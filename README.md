# AWS-EC2-Terraform-Nginx-Deploy
Automated EC2 provisioning and Nginx web server deployment on AWS using Terraform. Covers VPC, subnets, security groups, IGW, and file provisioning via SSH.

<img width="739" height="621" alt="Screenshot 2026-05-18 at 11 16 19 PM" src="https://github.com/user-attachments/assets/448e3a27-ca30-4ed4-b6ca-c6243477189d" />

## What I learned?

Concepts covered :
- VPC, subnet, IGW, route table — building a network from scratch
- Security groups — ingress vs egress rules, CIDR blocks
- Provisioners — file transfer and remote-exec over SSH
- nohup and process management on Linux
- terraform fmt, validate, plan, apply, destroy lifecycle

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

##

# Execution of terraform code

## terraform init
- <img width="587" height="309" alt="Screenshot 2026-05-18 at 10 55 58 PM" src="https://github.com/user-attachments/assets/70c86288-99ff-4486-a62e-ee54b253aafb" />

## terraform plan
- <img width="975" height="151" alt="Screenshot 2026-05-18 at 10 56 38 PM" src="https://github.com/user-attachments/assets/00ecfc18-c154-4452-b754-534afb406058" />

## terraform apply
- <img width="511" height="172" alt="Screenshot 2026-05-18 at 10 57 40 PM" src="https://github.com/user-attachments/assets/441e8538-d317-42bd-8b26-07db9a063b19" />
- <img width="553" height="101" alt="Screenshot 2026-05-18 at 10 59 14 PM" src="https://github.com/user-attachments/assets/1d3c215e-8eb6-4302-88fd-55bb073ba4c4" />

## EC2 instance created on console
- <img width="1166" height="678" alt="Screenshot 2026-05-18 at 11 00 30 PM" src="https://github.com/user-attachments/assets/8923f648-e484-44a0-918b-905618958f3e" />

## SSH connection to the ubuntu instance using the public IP
- <img width="632" height="27" alt="Screenshot 2026-05-18 at 11 02 42 PM" src="https://github.com/user-attachments/assets/e8bbacfc-53dd-4c31-b3d9-36ffa79fed66" />

## Checking index.html file has been copied and nginx version installed in the ubuntu instance 
- <img width="314" height="139" alt="Screenshot 2026-05-18 at 11 04 06 PM" src="https://github.com/user-attachments/assets/d4830a30-3748-40ad-9333-125f2c4391d7" />

## Copying the public IP of the ubuntu instance on chrome as 'http://65.1.95.226' to check if the webpage is getting hosted or not. 
- <img width="1440" height="900" alt="Screenshot 2026-05-18 at 11 06 26 PM" src="https://github.com/user-attachments/assets/9b218873-b542-467b-afb7-cac0ca81dff9" />

## Final stage : terraform destroy
   After verification, infrastructure was destroyed to avoid AWS charges.
- <img width="514" height="53" alt="Screenshot 2026-05-18 at 11 08 06 PM" src="https://github.com/user-attachments/assets/082fca72-98d5-4f66-9921-35dc056d90d2" />

##

Following are the snapshots of the reference from 'Terraform registry' that I used during the creation of the infrastructure as code.

- <img width="691" height="178" alt="Screenshot 2026-05-18 at 7 38 11 PM" src="https://github.com/user-attachments/assets/fe8c0aaa-1d77-4bba-8526-b4378c7b8da3" />
- <img width="672" height="160" alt="Screenshot 2026-05-18 at 7 31 20 PM" src="https://github.com/user-attachments/assets/47ceb296-c7ee-46fb-b2a2-373de48a6a36" />
- <img width="682" height="442" alt="Screenshot 2026-05-18 at 7 23 18 PM" src="https://github.com/user-attachments/assets/8b480b54-a95e-4176-be1d-9876f03c00e3" />
- <img width="679" height="239" alt="Screenshot 2026-05-18 at 7 21 11 PM" src="https://github.com/user-attachments/assets/41dc9d24-8862-4f19-8f59-ad65a6810877" />
- <img width="687" height="262" alt="Screenshot 2026-05-18 at 6 48 46 PM" src="https://github.com/user-attachments/assets/8f55ac50-62b2-4b55-b095-bbf39a7e96c3" />
- <img width="670" height="200" alt="Screenshot 2026-05-18 at 6 39 48 PM" src="https://github.com/user-attachments/assets/e72df53b-a90a-46f2-b814-744ccbe93022" />
- <img width="680" height="660" alt="Screenshot 2026-05-18 at 11 11 09 PM" src="https://github.com/user-attachments/assets/f71efd56-bc0b-4262-932d-ca56728f5e72" />



##

















