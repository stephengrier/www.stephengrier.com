---
layout: post
title: Reducing the cost of AWS NAT gateways
categories: [AWS,terraform,Cost efficiency]
---

Like many people these days I have various bits of infrastructure and software
that I run to support internet services for my own personal domains, mostly
email, DNS and web. I have a physical server colocated in a datacentre that
hosted much of these services, but the colo costs have become quite expensive
and running my own hardware is a bit of a liability. So I have been slowly
moving all my stuff to AWS, which is the cloud hosting provider I am most
familiar with.

Although part of the rationale for moving to AWS is to make my services more
reliable and easier to maintain, for me the number one requirement was to keep
costs low. For my purposes AWS's `t3.nano` and `t3.micro` instance types are
more than adequate. In the London region the on-demand price of a `t3.nano`
comes in at around $4.31 per month, and with spot rates typically 60-70% below
on-demand pricing a `t3.nano` could cost as little as $1.70 per month, well
within what I'm prepared to pay.

So, off I went and wrote bunch of terraform to provision a number of spot
instances to host my various bits-and-bobs, along with a typical AWS pattern of
VPC, subnets, security groups and load balancers. And like many AWS deployments
I decided I needed [NAT
gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
in my VPC to allow my instances on private subnets to connect out to the
internet. As is generally considered [best
prctice](https://docs.aws.amazon.com/quickstart/latest/vpc/architecture.html#best-practices)
I spread my infra across two availability zones and to be resilient to
availability zone failures I created a NAT gateway in both availability zones.

Having built and deployed the infrastructure I was happy to have everything
working nicely, everything was nicely defined in terraform code and
reproducible, and with spot instances to keep the costs nice and low. Consider
my shock, then, when I looked at AWS Cost Explorer and saw this:

![](/images/nat-cost-1.png)

What in the name of all that is holy was going on here? I looked at the Monthly
costs by service graph and noticed that although the `EC2-Instances` cost had
increased very slightly, the `EC2-Other` cost had gone through the roof.

![](/images/nat-cost-2.png)

But what exactly was `EC2-Other`? I looked at the daily costs by Usage Type and
found the answer: although `LoadBalancerUsage` and an old `t1.micro` were a
significant chunk of the cost, by far the largest part was due to
`NATGateway-Hours`. **In fact the NAT gateways were 61% of the daily cost!**

![](/images/nat-cost-3.png)

> "In fact the NAT gateways were 61% of the daily cost!"

## NAT Gateway Costs

NAT gateways are *not* cheap. AWS currently charge $0.05 per hour (in the London
region) for each NAT gateway running in your VPC, which is more than a
`t3.medium` on-demand. Unlike EC2 instances, AWS do not provide the option of
[Reserved Instances](https://aws.amazon.com/ec2/pricing/reserved-instances/) or
[spot pricing](https://aws.amazon.com/ec2/spot/pricing/) on NAT gateways, so
there is no way to reduce the cost of running them in your VPC.

As well as this, AWS charge $0.05 per Gigabyte of data processed through a NAT
gateway. For me this is not an issue as my use cases are all low traffic, but
for some this can push the cost up even further.

Fortunately, there is an alternative to NAT gateways that are much cheaper.

## NAT instances to the rescue!

NAT gateways were introduced by AWS in 2015. Prior to that, customers had to
provision [NAT
instances](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html),
which are just EC2 instances configured to perform NAT.

Helpfully, Amazon provides ready-made AMIs that are configured to run as NAT
instances so it is really easy to set these up. The AMIs have names that are
prefixed with `amzn-ami-vpc-nat` so are easy to find. They come configured with
IPv4 forwarding enabled and iptables configured to perform IP masquerading, so
no post-initialisation configuration with cloud-init is necessary.

Because NAT instances are just EC2 instances, you are free to decide what size
of instance is appropriate for your use case. With NAT all the processing is
done by the linux kernel networking stack, which runs in kernel space so is
efficient and does not require a huge amount of resources. For most use cases a
`t3.nano` will be perfectly adequate as a NAT instance.

And most importantly, because NAT instances are just EC2 instances, it is
possible to use Reserved Instance and spot pricing to reduce the cost of running
them. Even with on-demand pricing the cost of running a `t3.nano` performing NAT
is around 1/8 cost of a NAT gateway.

## Using spot instances for NAT

As mentioned already, spot pricing is typically 60-70% cheaper than on-demand
pricing. However, the downside of spot intances is that AWS can reclaim your
instances to serve spikes in demand for certain instances types. Although rare,
it is possible you might get a two minute warning before your spot instance is
terminated. This is not great if this happens to your NAT instance because
everything in your private subnets will lose connectivity to the internet.

However, a solution to this is to use autoscaling groups to recreate your NAT
instances if the spot instance is withdrawn. You will want to create a separate
autoscaling group for each availability zone because you want to guarantee there
is always a NAT instance in each AZ.

The terraform to create this might look like so:

```terraform
resource "aws_autoscaling_group" "nat" {
  count              = "${local.az_count}"
  name               = "nat-asg-${count.index}"
  desired_capacity   = 1
  min_size           = 1
  max_size           = 1
  availability_zones = ["${element(data.aws_availability_zones.azs.names, count.index)}"]

  mixed_instances_policy {
    instances_distribution {
      on_demand_base_capacity                  = 0
      on_demand_percentage_above_base_capacity = 0
    }

    launch_template {
      launch_template_specification {
        launch_template_id = "${element(aws_launch_template.nat_instance.*.id, count.index)}"
        version            = "$Latest"
      }

      dynamic "override" {
        for_each = ["t3.nano", "t3a.nano"]

        content {
          instance_type = "${override.value}"
        }
      }
    }
  }
}
```

The `on_demand_base_capacity` and `on_demand_percentage_above_base_capacity`
arguments above indicate we want 0% of our capacity to be served by on-demand
instances, meaning they should all be spot instances. The `count` argument will
cause one ASG to be created for each availability zone, and the
`desired_capacity`, `min_size` and `max_size` arguments indicate we want exactly
one instance to be provisioned in each AZ.

## Routing traffic to your NAT instance

Unfortunately there is one further problem with this setup. When setting up
NAT for a private subnet in AWS the default route target must be the ID of the
device that is performing the NAT. In the case of a NAT instance, this can be
either the ID of a network interface or an instance ID. Terraform will helpfully
set the ID in the route table. However, if the instance is terminated for some
reason, for example if the spot instance is reclaimed, the autoscaling group
will recreate the NAT instance, but the ID of the instance or the network
interface will be different to the one referenced in the route table, and things
will be broken and won't recover without intervention.

Thankfully, there is a nifty fix for this problem too. We can create [Elastic
Network Interfaces
(ENI)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html)
separately to the NAT instances, which can be attached and reattached to each
of the NAT instances as they are created. In the launch template for the
autoscaling group we simply need to set the `delete_on_termination` argument to
false for the network interface to stop the ENI being destroyed whenever the
instance is terminated, allowing it to be reused by the next instance.

The terraform for this might look like so:

```terraform
resource "aws_network_interface" "nat_eni" {
  count             = "${local.az_count}"
  security_groups   = ["${aws_security_group.nat_instance.id}"]
  subnet_id         = "${element(aws_subnet.public.*.id, count.index)}"
  source_dest_check = false
}

resource "aws_launch_template" "nat_instance" {
  count       = "${local.az_count}"
  name_prefix = "nat-instance-${count.index}"
  image_id    = "${data.aws_ami.amazon_nat.id}"

  network_interfaces {
    delete_on_termination = false
    network_interface_id  = "${element(aws_network_interface.nat_eni.*.id, count.index)}"
  }
}
```

This will create an ENI and launch template for each availability zone. There
needs to be separate launch templates for each AZ because each one references a
specific network interface ID. The `source_dest_check` argument to the
`aws_network_interface` resource is important because by default AWS will check
the source or destination IP addresses of IP packets match that of the EC2
instance and will drop the packet if not. But for NAT instances this is not
necessarily the case so we must disable this behaviour.

Finally, we can now route all traffic from our private subnet destined for the
internet to our network interface. In terrform this looks like:

```terraform
resource "aws_route_table" "private" {
  count  = "${local.az_count}"
  vpc_id = "${aws_vpc.vpc.id}"
}

resource "aws_route" "private_default" {
  count                  = "${local.az_count}"
  destination_cidr_block = "0.0.0.0/0"
  route_table_id         = "${element(aws_route_table.private.*.id, count.index)}"
  network_interface_id   = "${element(aws_network_interface.nat_eni.*.id, count.index)}"
}
```

## Final thoughts

On paper NAT gateways have a lot to offer and for many use cases offer a simple,
fully-managed way of providing NAT to private subnets in AWS VPCs. However, for
smaller personal or hobbyist uses they are prohibitively expensive and cause a
nasty surprise for many who, like me, are not initially aware of the potential
cost of running them.

However, by running NAT instances on spot pricing in the way described here it
is possible to run NAT boxes for as little as 1/20 of the cost of a NAT gateway.

Of course NAT instances are not the only way to avoid the cost of NAT gateways.
It is also possible to place instances on a public subnet with a public IP or
Elastic IP and use an AWS Internet gateway to route traffic to and from the
internet. You will need to rely on security groups to control ingress into your
instances with this setup as they will be addressable from the internet, but
with no need to run NAT instances at all this is likely the cheapest and
simplest solution for running workloads in AWS.
