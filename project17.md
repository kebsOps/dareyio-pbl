## Project 17

## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM. PART 2

This Project is a contuination of [Project 16](https://github.com/kebsOps/dareyio-pbl/blob/main/project16.md)

<img width="657" alt="image" src="https://user-images.githubusercontent.com/10085348/189526590-6befa92b-138a-4ba7-adc1-09747aeca0cd.png">

## Networking

### Private subnets and best practices

Create 4 private subnets keeping in mind following principles:

- Make sure you use variables or `length()` function to determine the number of AZs
- Use variables and `cidrsubnet()` function to allocate `vpc_cidr` for subnets
- Keep variables and resources in separate files for better code structure and readability
- Tags all the resources you have created so far. Explore how to use `format()` and `count` functions to automatically tag subnets with its respective number.


```
# Create private subnets
resource "aws_subnet" "private" {
  count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_private_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4, count.index + 2)
  map_public_ip_on_launch = false
  availability_zone       = element(data.aws_availability_zones.available.names, count.index)

  tags = merge(
    var.tags,
    {
      Name = format("PrivateSubnet-%s", count.index)
    },
  )

}
```

**Note:** You can add multiple tags as a default set. for example, in out `terraform.tfvars` file we can have default tags defined

```
tags = {
  Enviroment      = "production"
  Owner-Email     = "kebs@ops.com"
  Managed-By      = "Terraform"
  Billing-Account = "1234567890"
}
```

Now you can tag all you resources using the format below:

```
tags = merge(
    var.tags,
    {
      Name = "Name of the resource"
    },
  )
```

NOTE: Update the `variables.tf` to declare the variable tags used in the format above;

```
variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}
```

```
variable "name" {
  type = string
  default = "ACS"
}
```


## Internet Gateways & `format()` function

### Create an Internet Gateway in a separate Terraform file `internet_gateway.tf`

```
resource "aws_internet_gateway" "ig" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-%s!", aws_vpc.main.id,"IG")
    } 
  )
}
```

we have used `format()` function to dynamically generate a unique name for this resource? The first part of the `%s` takes the interpolated value of `aws_vpc.main.id` while the second `%s` appends a literal string IG and finally an exclamation mark is added in the end.

If any of the resources being created is either using the `count` function, or creating multiple resources using a loop, then a key-value pair that needs to be unique must be handled differently.


## NAT Gateways

### Create 1 NAT Gateways and 1 Elastic IP (EIP) addresses 

Create a new file called `natgateway.tf`

```
resource "aws_eip" "nat_eip" {
  vpc = true
  depends_on = [
  aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-EIP", var.name)
    },
  )
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = element(aws_subnet.public.*.id, 0)
  depends_on = [
    aws_internet_gateway.ig
  ]

  tags = merge(
    var.tags,
    {
      Name = format("%s-NAT", var.name)
    }
  )
}

```

## AWS ROUTES

Create a file called `route_tables.tf` and use it to create routes for both public and private subnets, create the below resources. Ensure they are properly tagged.

- aws_route_table
- aws_route
- aws_route_table_association

```
# Create private route table
resource "aws_route_table" "private-rtb" {
 vpc_id = aws_vpc.main.id

 tags = merge(
    var.tags,
    {
        Name = format("%s-Private-Route-Table", var.name)
    },
 )
}

# associate all private subnets to the private route table
resource "aws_route_table_association" "private-subnets-assoc" {
 count =  length(aws_subnet.private[*].id)
 subnet_id = element(aws_subnet.private[*].id, count.index)
 route_table_id = aws_route_table.private-rtb.id

}

# create route table for the public subnets
resource "aws_route_table" "public-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Public-Route-Table", var.name)
    },
  )
}

# create route for the public route table and attach the internet gateway
resource "aws_route" "public-rtb-route" {
    route_table_id = aws_route_table.public-rtb.id
    destination_cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.ig.id

}

# associate all public subnets to the public route table
resource "aws_route_table_association" "public-subnets-assoc" {
    count = length(aws_subnet.public[*].id)
    subnet_id = element(aws_subnet.public[*].id, count.index)
    route_table_id = aws_route_table.public-rtb.id
}

```

Now if you run ``terraform plan`` and ``terraform apply`` it will add the following resources to AWS in multi-az set up:

- Our main vpc
- 2 Public subnets
- 4 Private subnets
- 1 Internet Gateway
- 1 NAT Gateway
- 1 EIP
- 2 Route tables



<img width="1033" alt="image" src="https://user-images.githubusercontent.com/10085348/191476413-826c29c0-6f41-40af-a215-0b5092ef8437.png">

<img width="1047" alt="image" src="https://user-images.githubusercontent.com/10085348/191476655-c9710cf7-243f-4fbb-8834-7727ffa08761.png">


## AWS Identity and Access Management

### IaM and Roles

We want to pass an IAM role our EC2 instances to give them access to some specific resources, so we need to do the following:

- Create `AssumeRole`
- Assume Role uses Security Token Service (STS) API that returns a set of temporary security credentials that you can use to access AWS resources that you might not normally have access to. These temporary credentials consist of an `access key ID`, a `secret access key`, and a `security token`. Typically, you use `AssumeRole` within your account or for cross-account access.

Add the following code to a new file named `roles.tf`

```
resource "aws_iam_role" "ec2_instance_role" {
 name = "ec2_instance_role"
 assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
        {
            Action = "sts:AssumeRole"
            Effect = "Allow"
            Sid = ""
            Principal = {
                Service = "ec2.amazonaws.com"
            }
        },
    ]
 })

 tags = merge(
    var.tags,
    {
        Name = "aws assume role"
    }
 )

}
```


In this code we are creating `AssumeRole` with `AssumeRole policy`. It grants to an entity, in our case it is an EC2, permissions to assume the role.


### Create IAM policy for this role

This is where we need to define a required policy (i.e., permissions) according to our requirements. For example, allowing an IAM role to perform action `describe` applied to EC2 instances:


```
resource "aws_iam_policy" "policy" {
  name        = "ec2_instance_policy"
  description = "A test policy"
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = [
          "ec2:Describe*",
        ]
        Effect   = "Allow"
        Resource = "*"
      },
    ]
  })

  tags = merge(
    var.tags,
    {
      Name = "aws assume policy"
    },
  )

}
```

### Attach the Policy to the IAM Role

This is where, we will be attaching the policy which we created above, to the role we created in the first step.

```
resource "aws_iam_role_policy_attachment" "test-attach" {
  role = aws_iam_role.ec2_instance_role.name
  policy_arn = aws_iam_policy.policy.arn
}
```

Create an `Instance Profile` and interpolate the `IAM Role`

```
resource "aws_iam_instance_profile" "ip" {
 name = "aws_instance_profile_test"
 role = aws_iam_role.ec2_instance_role.name 
}
```

We are done with Identity and Management part for now, let us move on and create other resources required.


`roles.tf` file

<img width="602" alt="image" src="https://user-images.githubusercontent.com/10085348/191484900-f4492bac-8d36-4b85-9f75-76a7a101e3a4.png">


<img width="526" alt="image" src="https://user-images.githubusercontent.com/10085348/191484972-c0614634-f5a2-4043-9e7e-783dbee71369.png">




### Resources to be created

As per our architecture we need to do the following:

- Create Security Groups
- Create Target Group for Nginx, WordPress and Tooling
- Create certificate from AWS certificate manager
- Create an External Application Load Balancer and Internal Application Load Balancer.
- Create launch template for Bastion, Tooling, Nginx and WordPress
- Create an Auto Scaling Group (ASG) for Bastion, Tooling, Nginx and WordPress
- Create Elastic Filesystem
- Create Relational Database (RDS)


Let us create some Terraform configuration code to accomplish these tasks.

## Create Security Groups

We are going to create all the security groups in a single file, then we are going to refrence this security group within each resources that needs it.

IMPORTANT:

Check out the terraform documentation for [security group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group)

Check out the terraform documentation for [security group rule](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule)

Create a file and name it ``security.tf``, copy and paste the code below

```
# security group for alb, to allow acess from any where for HTTP and HTTPS traffic
resource "aws_security_group" "ext-alb-sg" {
  name        = "ext-alb-sg"
  vpc_id      = aws_vpc.main.id
  description = "Allow TLS inbound traffic"

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTPS"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "ext-alb-sg"
    },
  )

}

# security group for bastion, to allow access into the bastion host from you IP
resource "aws_security_group" "bastion_sg" {
  name        = "vpc_web_sg"
  vpc_id      = aws_vpc.main.id
  description = "Allow incoming HTTP connections."

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "Bastion-SG"
    },
  )
}

#security group for nginx reverse proxy, to allow access only from the extaernal load balancer and bastion instance
resource "aws_security_group" "nginx-sg" {
  name   = "nginx-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "nginx-SG"
    },
  )
}

resource "aws_security_group_rule" "inbound-nginx-http" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ext-alb-sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}

resource "aws_security_group_rule" "inbound-bastion-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}

# security group for ialb, to have acces only from nginx reverser proxy server
resource "aws_security_group" "int-alb-sg" {
  name   = "my-alb-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "int-alb-sg"
    },
  )

}

resource "aws_security_group_rule" "inbound-ialb-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.nginx-sg.id
  security_group_id        = aws_security_group.int-alb-sg.id
}

# security group for webservers, to have access only from the internal load balancer and bastion instance
resource "aws_security_group" "webserver-sg" {
  name   = "my-asg-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "webserver-sg"
    },
  )

}

resource "aws_security_group_rule" "inbound-web-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.int-alb-sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}

resource "aws_security_group_rule" "inbound-web-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}

# security group for datalayer to alow traffic from websever on nfs and mysql port and bastiopn host on mysql port
resource "aws_security_group" "datalayer-sg" {
  name   = "datalayer-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "datalayer-sg"
    },
  )
}

resource "aws_security_group_rule" "inbound-nfs-port" {
  type                     = "ingress"
  from_port                = 2049
  to_port                  = 2049
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-bastion" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-webserver" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}
```

**NOTE:** We used the `aws_security_group_rule` to refrence another security group in a security group.

### Creating Certificate From Amazon Certificate Manager

**NOTE: Read Through to change the domain name to your own domain name and every other name that needs to be changed**

Create ``cert.tf`` file and add the following code snippets to it.

```
# The entire section create a certificate, public zone, and validate the certificate using DNS method

# Create the certificate using a wildcard for all the domains created in toolingkb.xyz
resource "aws_acm_certificate" "toolingkb" {
 domain_name = "*.toolingkb.xyz"
 validation_method = "DNS" 
}

# calling the hosted zone
data "aws_route53_zone" "toolingkb" {
    name = "toolingkb.xyz"
    private_zone = false
}

# selecting validation method
resource "aws_route53_record" "toolingkb" {
  for_each = {
   for dvo in aws_acm_certificate.toolingkb.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }

  allow_overwrite = true
  name            = each.value.name
  records         = [each.value.record]
  ttl             = 60
  type            = each.value.type
  zone_id         = data.aws_route53_zone.toolingkb.zone_id
}

# validate the certificate through DNS method
resource "aws_acm_certificate_validation" "toolingkb" {
  certificate_arn         = aws_acm_certificate.toolingkb.arn
  validation_record_fqdns = [for record in aws_route53_record.toolingkb : record.fqdn]
}

# create records for tooling
resource "aws_route53_record" "tooling" {
  zone_id = data.aws_route53_zone.toolingkb.zone_id
  name    = "tooling.toolingkb.xyz"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}

# create records for wordpress
resource "aws_route53_record" "wordpress" {
  zone_id = data.aws_route53_zone.toolingkb.zone_id
  name    = "wordpress.toolingkb.xyz"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}
```

### Create an external (Internet facing) Application Load Balancer (ALB)

Create a file called `alb.tf`

We need to create an ALB to balance the traffic between the Instances:


```
resource "aws_lb" "ext-alb" {
 name = "ext-alb"
 internal = false
 security_groups = [
    aws_security_group.ext-alb-sg.id,
 ] 

 subnets = [
    aws_subnet.public[0].id,
    aws_subnet.public[1].id
  ]
  
 tags = merge(
    var.tags,
    {
        Name = "ACS-ext-alb"
    },
 )

 ip_address_type = "ipv4"
 load_balancer_type = "application"
}
```

```
# Inform our ALB to where route the traffic we need to create a Target Group to point to it's targets
resource "aws_lb_target_group" "nginx-tgt" {
  health_check {
    interval = 10
    path = "/healthstatus"
    protocol = "HTTPS"
    timeout = 5
    healthy_threshold = 5
    unhealthy_threshold = 2
  }
  name = "ngnix-tgt"
  port = 443
  protocol = "HTTPS"
  target_type = "instance"
  vpc_id = aws_vpc.main.id
}
```

```
# create a Listener for this target Group
resource "aws_lb_listener" "nginx-listener" {
  load_balancer_arn = aws_lb.ext-alb.arn
  port = 443
  protocol = "HTTPS"
  certificate_arn = aws_acm_certificate_validation.toolingkb.certificate_arn

  default_action {
    type = "forward"
    target_group_arn = aws_lb_target_group.nginx-tgt.arn
  }
}
```

Add the following outputs to `output.tf` to print them on screen

```
output "alb_dns_name" {
  value = aws_lb.ext-alb.dns_name
}

output "alb_target_group_arn" {
  value = aws_lb_target_group.nginx-tgt.arn
}
```

### Create an Internal (Internal) Application Load Balancer (ALB)

For the Internal Load balancer we will follow the same concepts with the external load balancer.

Add the code snippets inside the `alb.tf` file

```
# -------------------------------------
#Internal Load Balancers for webservers
#--------------------------------------

resource "aws_lb" "ialb" {
  name     = "ialb"
  internal = true
  security_groups = [
    aws_security_group.int-alb-sg.id,
  ]

  subnets = [
    aws_subnet.private[0].id,
    aws_subnet.private[1].id
  ]

  tags = merge(
    var.tags,
    {
      Name = "ACS-int-alb"
    },
  )

  ip_address_type    = "ipv4"
  load_balancer_type = "application"
}
```

To inform our ALB to where route the traffic we need to create a **Target Group** to point to its targets:

```
# --- target group  for wordpress -------
resource "aws_lb_target_group" "wordpress-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }

  name        = "wordpress-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}

# --- target group for tooling -------
resource "aws_lb_target_group" "tooling-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }

  name        = "tooling-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}
```

Then we will need to create a Listener for this target Group:

```
# For this aspect a single listener was created for the wordpress which is default,
# A rule was created to route traffic to tooling when the host header changes
resource "aws_lb_listener" "web-listener" {
  load_balancer_arn = aws_lb.ialb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.toolingkb.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.wordpress-tgt.arn
  }
}

# listener rule for tooling target
resource "aws_lb_listener_rule" "tooling-listener" {
  listener_arn = aws_lb_listener.web-listener.arn
  priority     = 99

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.tooling-tgt.arn
  }

  condition {
    host_header {
      values = ["tooling.toolingkb.xyz"]
    }
  }
}

```


### CREATING AUTOSCALING GROUPS

This Section we will create the Auto Scaling Group (ASG)

Now we need to configure our ASG to be able to scale the EC2s out and in depending on the application traffic.

Before we start configuring an ASG, we need to create the launch template and the the AMI needed. For now we are going to use a random AMI from AWS, then in project 19, we will use Packerto create our ami.

Based on our Architetcture we need for **Auto Scaling Groups for bastion, nginx, wordpress and tooling,** so we will create two files; `asg-bastion-nginx.tf` will contain Launch Template and Austoscaling froup for Bastion and Nginx, then `asg-wordpress-tooling.tf` will contain Launch Template and Austoscaling group for wordpress and tooling.


Create `asg-bastion-nginx.tf` and paste all the code snippet below:
