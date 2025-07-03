provider "aws" {
  region = "us-east-1"
}

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">=6.0.0"
    }
  }
  required_version = ">=1.12.2"
}

resource "aws_vpc" "vp" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = {
    Name = "main-webserver-vpc"
  }
}

resource "aws_subnet" "pubsubnet1" {
  vpc_id                  = aws_vpc.vp.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
  tags = {
    Name = "public-subnet-1"
  }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.vp.id
  tags = {
    Name = "main-igw"
  }
}

resource "aws_route_table" "pubrt" {
  vpc_id = aws_vpc.vp.id
  tags = {
    Name = "public-route-table"
  }
}

locals {
  ingress_rules = [
    { port = 80 },  # HTTP
    { port = 22 }  # SSH
  ]
}

resource "aws_security_group" "web_sg" {
  name        = "web-sg"
  description = "Allow HTTP and SSH traffic"
  vpc_id      = aws_vpc.vp.id

  dynamic "ingress" {
    for_each = local.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "web-sg"
  }
}

resource "aws_instance" "webserver" {
  ami                    = "ami-020cba7c55df1f615" #ubuntu server
  instance_type          = "t2.micro"
  key_name = "mysecond"
  subnet_id              = aws_subnet.pubsubnet1.id
  vpc_security_group_ids = [aws_security_group.web_sg.id]
  user_data = file("webdeploy.sh")

  tags = {
    Name = "webserver-instance"
  }
}

resource "aws_route_table_association" "pubsubnet_assoc" {
  subnet_id      = aws_subnet.pubsubnet1.id
  route_table_id = aws_route_table.pubrt.id
}

resource "aws_route" "internet_access" {
  route_table_id         = aws_route_table.pubrt.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.igw.id
}

output "values" {
  value = {
    ip_address       = aws_instance.webserver.public_ip
    public_dns       = aws_instance.webserver.public_dns
    vpc_id            = aws_vpc.vp.id
    subnet_id         = aws_subnet.pubsubnet1.id
    internet_gateway  = aws_internet_gateway.igw.id
    route_table_id    = aws_route_table.pubrt.id
    security_group_id = aws_security_group.web_sg.id
    instance_id       = aws_instance.webserver.id
  }
}
