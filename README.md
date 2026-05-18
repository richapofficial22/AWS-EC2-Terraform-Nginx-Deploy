# AWS-EC2-Terraform-Nginx-Deploy
Automated EC2 provisioning and Nginx web server deployment on AWS using Terraform. Covers VPC, subnets, security groups, IGW, and file provisioning via SSH.
##
Mistakes/Errors I made during the project :
 - At first created a separate variables file but on executing plan command it showed error in the aws_instance block mentioning that it is unable to find ami and instance_type. So had to include the variables in the main.tf file itself.
 - Faced few issues while deciding on which attributes to keep in the resource blocks while taking reference from 'Terraform Registry'
 - Messed up the cidr_ipv4 value in both aws_vpc_security_group_ingress_rule blocks
 - In 'connection' sub-block i mentioned user as 'Richa' but soon changed to 'ubuntu' as I decided I won't be creating any user in the ubuntu environment.
 - In the 'inline' sub-block I forgot to type forward slash before 'var' which stopped the execution of apply command due to which the file couldn't be copied to EC2 instance
 - I had also did a typing mistake in the 'inline' block writing 'ndex.html' instead of 'index.html'


##

# main.tf

```
   provider "aws" {
   region = "ap-south-1"
}
variable "ami" {
  description = "Input variable for ami"
  default = "ami-07a00cf47dbbc844c"
}
resource "aws_key_pair" "kp" {
  key_name = "richa-key-pair"
  public_key = file("/Users/richierich/.ssh/id_rsa.pub")
}
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}
resource "aws_subnet" "main" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
  availability_zone = "ap-south-1b"
  map_public_ip_on_launch = true
}
resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.main.id
}
resource "aws_route_table" "rtable" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.gw.id
  }
}
resource "aws_route_table_association" "rta" {
 subnet_id = aws_subnet.main.id
 route_table_id = aws_route_table.rtable.id
}
resource "aws_security_group" "allow_sg" {
  name        = "allow_tls"
  description = "Allow TLS inbound traffic and all outbound traffic"
  vpc_id      = aws_vpc.main.id
}
resource "aws_vpc_security_group_ingress_rule" "allow_tls_http" {
  security_group_id = aws_security_group.allow_sg.id
  cidr_ipv4         = "0.0.0.0/0"
  from_port         = 80
  ip_protocol       = "tcp"
  to_port           = 80
}
resource "aws_vpc_security_group_ingress_rule" "allow_tls_ssh" {
  security_group_id = aws_security_group.allow_sg.id
  cidr_ipv4         = "0.0.0.0/0"
  from_port         = 22
  ip_protocol       = "tcp"
  to_port           = 22
}
resource "aws_vpc_security_group_egress_rule" "allow_all_traffic_ipv4" {
  security_group_id = aws_security_group.allow_sg.id
  cidr_ipv4         = "0.0.0.0/0"
  ip_protocol       = "-1" # semantically equivalent to all ports
}
resource "aws_instance" "RPinstance" {
  ami = var.ami
  instance_type = "t2.micro"
  vpc_security_group_ids = [aws_security_group.allow_sg.id]
  subnet_id = aws_subnet.main.id
  key_name = aws_key_pair.kp.key_name

  connection {
    host = self.public_ip
    type = "ssh"
    user = "ubuntu"
    private_key = file("/Users/richierich/.ssh/id_rsa")
  }
  provisioner "file" {
    source = "index.html"
    destination = "/home/ubuntu/index.html"
  }
  provisioner "remote-exec" {
    inline = [
            "sudo apt update -y",
            "sudo apt-get install -y nginx",
            "sudo systemctl start nginx",
            "sudo systemctl enable nginx",
            "sudo cp /home/ubuntu/index.html /var/www/html/index.html"
      ]
  }
}

```

# output.tf
```
output "ami" {
  value = aws_instance.RPinstance.ami
}

```

# index.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Richa - Cloud Engineer in Training</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; padding: 50px; background: #f4f4f4; }
        h1 { color: #111; }
        .links a { display: inline-block; margin: 20px; padding: 15px 30px; background: #0077B5 ; color: white; text-decoration: none; border-radius: 5px; font-weight: bold; }
        
    </style>
</head>
<body>
    <h1>Hi, I'm Richa</h1>
    <p> AWS Certified Cloud Practitioner <br>Bengaluru, India</p>
    
    <div class="links">
        <a href="https://www.linkedin.com/in/richa-prabhakar-cloud/" target="_blank">Connect on LinkedIn</a>
        <a href="https://github.com/richapofficial22" target="_blank" >View on GitHub</a>
    </div>
    
    <p>Follow my journey in cloud computing and tech!</p>
</body>
</html>
```

