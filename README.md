# security_group-creation-using-count-foreach-dynamic_block

Creating security groups including number of ports in your Terraform code can make it longer and potentially more complex. To simplify your Terraform code, you can consider using meta arguments like count, foreach function and Dynamic blocks.

Let us go throough the different Terraform code to create a security group with the specified ingress ports (22, 80, 443, 8080, and 53) and allow all traffic in egress

**Method1** 

This security group allows inbound traffic on ports 22 (SSH), 80 (HTTP), 443 (HTTPS), 8080, and UDP traffic on port 53 (DNS). It allows outbound traffic to any destination on any port.

You can see how long the code would be!
```
resource "aws_security_group" "frontend" {
  name        = "frontend"
  description = "Allow ssh, http,https and udp connections"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 53
    to_port     = 53
    protocol    = "udp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port       = 0
    to_port         = 0
    protocol        = "-1"
    cidr_blocks     = ["0.0.0.0/0"]
  }
}
```
 **Method 2 - Using count | meta argument**

> Declaring the variable

```
variable "frontend-ports" {
type = list
default = [22,80,53,443,8080]
}
```
> Creating the SG

```
resource "aws_security_group" "frontend" {

name        = "frontend"
description = "Allow ssh, http,https and udp connections"
  
  egress {
    from_port       = 0
    to_port         = 0
    protocol        = "-1"
    cidr_blocks     = ["0.0.0.0/0"]
  }
}
```
> Creating ingress rule

```
resource "aws_security_group_rule" "frontend" {
count = length (var.frontend-ports)
type = "ingress"
from_port = var.frontend-ports[count.index]
to_port = var.frontend-ports[count.index]
 cidr_blocks     = ["0.0.0.0/0"]
security_group_id = aws_security_group.frontend.id
}
```
