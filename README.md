\# Background

The purpose of this repository is to automate the process of configuring and creating an AWS EC2 instance, as well as downloading and configuring the resources required to create and start up a Minecraft server. This will be accomplished by using Terraform and Ansible files to automate these steps. Specifically, the process of creating the AWS EC2 instance will be accomplished through Terraform files, and the process of creating the Minecraft server will be accomplished through an Ansible playbook and a supporting inventory.ini file. 

\# Requirements

For the repository to function properly, the following is required to have:

\* A computer with a Linux operating system

\* Terraform installed on the computer

\* Ansible installed on the computer

\* AWS account credentials

\* An SSH key pair file (e.g. Minecraft\_Key.pem)

\# Diagram

File Path

\## Terraform Files

\### variable.tf

The variable.tf file is written with the following commands:

> variable "ami\_id" {

&#x20; description = "AMI ID for the EC2 instance (Ubuntu 26.04 in us-east-1)"

&#x20; type        = string

&#x20; default     = "ami-0d13e2317a7e75c95"

}

> variable "instance\_type" {

&#x20; description = "EC2 instance type"

&#x20; type        = string

&#x20; default     = "t4g.small"

}

> variable "Minecraft\_Key" {

&#x20; description = "Name of the SSH key pair (must already exist in AWS)"

&#x20; type        = string

&#x20; default     = "Minecraft\_Key.pem"

}



This establishes the following variables:

\* \*\*AMI ID\*\*: This determines the operating system the instance uses

\* \*\*Instance type\*\*: This determines the type of instance the EC2 instance will be

\* \*\*Mincraft Key\*\*: This determines the name of the SSH key pair file



These variables will be used in the main.tf file.

\### main.tf

The main.tf file is written with the following commands:

>terraform {

&#x20; required\_providers {

&#x20;   aws = {

&#x20;     source  = "hashicorp/aws"

&#x20;     version = "\~> 5.0"

&#x20;   }

&#x20; }

}

>

>provider "aws" {

&#x20; region = "us-east-1"

}

>data "aws\_vpc" "default" {

&#x20; default = true

}

>

>resource "aws\_security\_group" "minecraft" {

&#x20; name        = "Minecraft Security Group"

&#x20; vpc\_id      = data.aws\_vpc.default.id

>

&#x20; >ingress {

&#x20;   from\_port   = 22

&#x20;   to\_port     = 22

&#x20;   protocol    = "tcp"

&#x20;   cidr\_blocks = \["0.0.0.0/0"]

&#x20; }

>

&#x20; >ingress {

&#x20;   from\_port   = 25565

&#x20;   to\_port     = 25565

&#x20;   protocol    = "tcp"

&#x20;   cidr\_blocks = \["0.0.0.0/0"]

&#x20; }

>

&#x20; >egress {

&#x20;   from\_port   = 0

&#x20;   to\_port     = 0

&#x20;   protocol    = "-1"

&#x20;   cidr\_blocks = \["0.0.0.0/0"]

&#x20; }

}

>

>resource "aws\_instance" "minecraft" {

&#x20; ami                    = var.ami\_id

&#x20; instance\_type          = var.instance\_type

&#x20; key\_name               = var.key\_name

&#x20; vpc\_security\_group\_ids = \[aws\_security\_group.control.id]

&#x20; iam\_instance\_profile   = "InstanceProfile"

>

&#x20; >tags = {

&#x20;   Name = "minecrat\_server"

&#x20; }

}

>

>resource "local\_file" "ansible\_inventory" {

&#x20; content = "\[Minecraft]\\n${aws\_instance.minecraft\_server.public\_ip} ansible\_user=ec2-user ansible\_ssh\_private\_key\_file=\~\\mc\_server"

&#x20; filename = "inventory.ini"

}



This is what each block does:

\* \*\*Terraform\*\* - Specifies the provider to use

\* \*\*Provider\*\* - Lists specific details of the provider

\* \*\*Data\*\* - Reads the existing VPC without creating it

\* \*\*resource "aws\_security\_group" "minecraft\_group"\*\* - Creates the security group of the instance, specifying open ports

\* \*\*resource "aws\_instance" "minecraft\_instance"\*\* - Establishes the details of the AWS EC2 instance

\* \*\*resource "local\_file" "ansible\_inventory"\*\* - Creates the inventory.ini file to be used by the Ansible playbook

\### outputs.tf

The outputs.tf file is written with the following commands:

>output "public\_ip" {

&#x20; value       = aws\_instance.minecraft\_server.public\_ip

}



This block outputs the public IP of the instance to the terminal.

\## Ansible Playbook

The Ansible playbook, \*playbook.yml\*, is written with the following commands:

Screenshot 1

Screenshot 2

This is what each block does:

\* \*\*Create Minecraft Server\*\* - References the Minecraft group in the inventory.ini file and enables the tasks to be ran with sudo (admin privileges)

\* \*\*Create Minecraft User\*\* - Creates user to associate with the directory and server

\* \*\*Create Minecraft Directory\*\* - Creates the Minecraft directory and associates the owner and group

\* \*\*Download Java File\*\* - Retrieves the java file from the provided link and downloads it to the specified destination and establishes permissions for the file

\* \*\*Extract Java File\*\* - Extracts the file to the specified destination and tells ansible that the file is already on the host so that it doesn't get retrieved again

\* \*\*Download Minecraft Server\*\* - Retrieves the server file from the provided link and downloads it to the specified destination while also establishing the owner, group, and permissions

\* \*\*Create the systemd file\*\* - Creates the \*minecraft.service\* file and writes the following commands to the file:

> \[Unit]

> Description=Minecraft Server

> After=network.target

>

>\[Service]

>Type=simple

>

>User=ec2-user

>Group=ec2-user

>WorkingDirectory=/home/ec2-user/mc\_server

>

>ExecStart=/home/ec2-user/mc\_server/amazon-corretto-25.0.3.9.1-linux-aarch64/bin/java -Xmx1300M -Xms1300M -jar server.jar nogui

>

>\[Install]

>WantedBy=multi-user.target



\* \*\*Reload systemd\*\* - Reload systemd to ensure its working

\* \*\*Enable minecraft.service\*\* - Enables the systemd file to automatically start up the Minecraft server

\## Execution Commands

\*\*Terraform:\*\*

> terraform init



This command creates the directory that stores the Terraform plugins

> terraform plan -var="key\_name=Minecraft\_key.pem"



This command shows what the terraform files will output before executing the command

> terraform apply -var="key\_name=Minecraft\_key.pem"



This command shows the plan again and asks for confirmation. When yes is typed into the terminal, the plan is executed.



\*\*Ansible:\*\*

> ansible-playbook -i inventory.ini playbook.yml



This command executes the playbook while also referencing the inventory.ini file



\*\*Nmap:\*\*

> nmap -sV -Pn -p T:25565 <instance\_public\_ip>



This command connects to the Minecraft server instance with nmap



\* 34. Execute the following systemctl commands to run the server:

>sudo systemctl daemon-reload

>sudo systemctl enable minecraft.service

>sudo systemctl start minecraft.service

\* 35. The server should automatically start whenever the instance is started

\* 36. Optional: Type the command \*\*sudo systemctl status minecraft.service\*\* to check if the server is successfully running

\## How to connect to the server from the Minecraft client

\* Open your Minecraft client, select \*\*Multiplayer\*\*, then select the \*\*Direct Connection\*\* button

\* Type in the elastic IP, then press the \*\*Join Server\*\* button

