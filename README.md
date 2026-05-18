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

# Execution of code

