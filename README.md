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
    ipv6_cidr_blocks = ["::/0"]
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    ipv6_cidr_blocks = ["::/0"]
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    ipv6_cidr_blocks = ["::/0"]
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    ipv6_cidr_blocks = ["::/0"]
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 53
    to_port     = 53
    protocol    = "udp"
    ipv6_cidr_blocks = ["::/0"]
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port       = 0
    to_port         = 0
    protocol        = "-1"
    ipv6_cidr_blocks = ["::/0"]
    cidr_blocks     = ["0.0.0.0/0"]
  }
}
```
 **Method 2 - Using count | meta argument**

> Declaring the variable

The default value is a list of integers [22, 80, 21, 443, 8080], representing the ports used for the frontend of an application or infrastructure.
The value of the "frontend-ports" variable can be customized according to your requirements.

```
variable "frontend-ports" {
type = list
default = [22,80,21,443,8080]
}
```
> Creating the SG

```
resource "aws_security_group" "frontend" {

name        = "frontend"
description = "Allow ssh, http,https and ftp connections"
  
  egress {
    from_port       = 0
    to_port         = 0
    protocol        = "-1"
    ipv6_cidr_blocks = ["::/0"]
    cidr_blocks     = ["0.0.0.0/0"]
  }
}
```
> Creating ingress rule

count = length(var.frontend-ports): This line sets the count of the resource based on the length of the "frontend-ports" variable. This allows creating multiple security group rules based on the number of ports specified in the variable.

from_port = var.frontend-ports[count.index] and to_port = var.frontend-ports[count.index]: These lines dynamically assign the from_port and to_port of each security group rule using the values from the "frontend-ports" variable. The count.index variable is used to iterate through the list of ports.

```
resource "aws_security_group_rule" "frontend" {
count = length (var.frontend-ports)

type = "ingress"
from_port = var.frontend-ports[count.index]
to_port = var.frontend-ports[count.index]
protocol          = "tcp"
ipv6_cidr_blocks = ["::/0"]
cidr_blocks     = ["0.0.0.0/0"]
security_group_id = aws_security_group.frontend.id
}
```

**Method-3-for_each**

You can easily add or remove ports in the var.frontend_ports variable, and Terraform will automatically create or delete the corresponding security group ingress rules. 

Variable holds the list of ports to be opened for ingress traffic.
The for_each argument is used to create a rule for each port in the frontend_ports list.
Each rule allows ingress traffic on the specified port (both from_port and to_port) with the TCP protocol.
```
variable "frontend_ports" {
    
  type    = list
  default = ["22","80","21","443"]
    
}

resource "aws_security_group" "frontend" {
        
  name        = "frontend"
  description = "frontend"

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }
}

resource "aws_security_group_rule" "frontend-rules" {
    
  for_each          = toset(var.frontend_ports)
        
  type              = "ingress"
  from_port         = each.key
  to_port           = each.key
  protocol          = "tcp"
  cidr_blocks       = ["0.0.0.0/0"]
  ipv6_cidr_blocks  = ["::/0"]
  security_group_id = aws_security_group.frontend.id
}
```

**Method4-DynamicBlock**

Dynamic block allows you to loop over a section of a resource and generate multiple instances of that section dynamically based on a given input. 

The for_each argument of the dynamic block is set to toset(var.ports), which converts the ports variable into a set. Using a set ensures that duplicate ports are removed, as each security group rule must have a unique combination of ports.

Inside the dynamic block, a nested content block is defined. This block represents the dynamic section that will be looped over.

The attributes within the content block, such as from_port, to_port, protocol, cidr_blocks, and ipv6_cidr_blocks, are set based on the current value of ingress during each iteration.

Each iteration of the dynamic block generates an ingress rule for a specific port.

```
variable "ports" {
  type = list
  default = ["80","443","8080,"22"]
}


resource "aws_security_group" "webserver-traffic" {
    
  name        = "dynamic-frontend"
  description = "allows http & https traffic"
    
  dynamic "ingress" {
    
     for_each = toset(var.ports)
     content {
         
        from_port        = ingress.value
        to_port          = ingress.value
        protocol         = "tcp"
        cidr_blocks      = ["0.0.0.0/0"]
        ipv6_cidr_blocks = ["::/0"]
         
    }
  }
    
 
  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }
  }
