```bash
    provider "aws" {
    region = "ap-south-1"
    shared_credentials_files = ["/root/.aws/credentials" ]
  }

  # Create VPC
  resource "aws_vpc" "main" {
    cidr_block = "10.0.0.0/16"

    tags = {
      Name = "main"
    }
  }

  # Create public subnet
  resource "aws_subnet" "public" {
    vpc_id            = aws_vpc.main.id
    cidr_block        = "10.0.1.0/24"
    availability_zone = "ap-south-1b"

    tags = {
      Name = "public"
    }
  }

  # Create another public subnet
  resource "aws_subnet" "public2" {
    vpc_id            = aws_vpc.main.id
    cidr_block        = "10.0.3.0/24"
    availability_zone = "ap-south-1c"

    tags = {
      Name = "public2"
    }
  }

  # Create private subnet
  resource "aws_subnet" "private" {
    vpc_id            = aws_vpc.main.id
    cidr_block        = "10.0.2.0/24"
    availability_zone = "ap-south-1a"

    tags = {
      Name = "private"
    }
  }

  # Create internet gateway
  resource "aws_internet_gateway" "gw" {
    vpc_id = aws_vpc.main.id

    tags = {
      Name = "gw"
    }
  }

  # Create route table for public subnet
  resource "aws_route_table" "public" {
    vpc_id = aws_vpc.main.id

    route {
      cidr_block = "0.0.0.0/0"
      gateway_id = aws_internet_gateway.gw.id
    }

    tags = {
      Name = "public"
    }
  }

  # Associate public subnet with public route table
  resource "aws_route_table_association" "public" {
    subnet_id      = aws_subnet.public.id
    route_table_id = aws_route_table.public.id
  }

  # Associate public2 subnet with public route table
  resource "aws_route_table_association" "public2" {
    subnet_id      = aws_subnet.public2.id
    route_table_id = aws_route_table.public.id
  }


  # Create security group
  resource "aws_security_group" "lb_sg" {
    name        = "lb-sg"
    description = "Security group for ALB"
    vpc_id      = aws_vpc.main.id

    ingress {
      from_port   = 0
      to_port     = 65535
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
    egress {
      from_port   = 0
      to_port     = 0
      protocol    = "-1"  # This indicates all protocols
      cidr_blocks = ["0.0.0.0/0"]  # Allow traffic to all destinations
  }
    tags = {
      Name = "lb-sg"
    }
  }

  # Create Application Load Balancer
  resource "aws_lb" "infra" {
    name               = "infra-alb"
    internal           = false
    load_balancer_type = "application"
    security_groups    = [aws_security_group.lb_sg.id]
    subnets            = [
      aws_subnet.public.id,
      aws_subnet.public2.id
    ]

    tags = {
      Name = "infra-alb"
    }
  }

  # Create target group
  resource "aws_lb_target_group" "infra" {
    name     = "infra-target-group"
    port     = 80
    protocol = "HTTP"
    vpc_id   = aws_vpc.main.id

    health_check {
      path                = "/"
      protocol            = "HTTP"
      port                = "traffic-port"
      interval            = 30
      timeout             = 5
      healthy_threshold   = 2
      unhealthy_threshold = 2
    }
  }

  # Create listener for ALB
  resource "aws_lb_listener" "infra" {
    load_balancer_arn = aws_lb.infra.arn
    port              = 80
    protocol          = "HTTP"

    default_action {
      type             = "forward"
      target_group_arn = aws_lb_target_group.infra.arn
    }
  }

  # Create autoscaling launch configuration
  resource "aws_launch_configuration" "example" {
    image_id        = "ami-0f58b397bc5c1f2e8"
    instance_type   = "t2.micro"
    key_name        = "vignesh"
    security_groups = ["${aws_security_group.lb_sg.id}"]
    associate_public_ip_address = true   # Assign a public IP address to instances

  
  }

  # Create autoscaling group
  resource "aws_autoscaling_group" "example" {
    launch_configuration = aws_launch_configuration.example.name
    min_size             = 1
    max_size             = 3
    desired_capacity     = 1

    vpc_zone_identifier = [
      aws_subnet.public.id, 
      aws_subnet.public2.id
    ]

    tag {
      key                 = "Name"
      value               = "example-instance"
      propagate_at_launch = true
    }

    lifecycle {
      create_before_destroy = true
    }

    target_group_arns = ["${aws_lb_target_group.infra.arn}"]
  }



# Create autoscaling policy for scaling out
resource "aws_autoscaling_policy" "scale_out_policy" {
  name                   = "scale-out-policy"
  adjustment_type        = "ChangeInCapacity"
  autoscaling_group_name = aws_autoscaling_group.example.name

  # Scale out when average CPU utilization exceeds 40% for 2 consecutive periods
  policy_type            = "TargetTrackingScaling"
  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
    target_value          = 40
    disable_scale_in      = false
  }
}

# Create autoscaling policy for scaling in
resource "aws_autoscaling_policy" "scale_in_policy" {
  name                   = "scale-in-policy"
  adjustment_type        = "ChangeInCapacity"
  autoscaling_group_name = aws_autoscaling_group.example.name

  # Scale in when average CPU utilization drops below 40% for 2 consecutive periods
  policy_type            = "TargetTrackingScaling"
  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
    target_value          = 40
    disable_scale_in      = false
  }
}



```
