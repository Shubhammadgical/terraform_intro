Hello guys my self shubham Chaurasia and in this repo we discussed about Introduction to Terraform step-by-step.

# Set up your AWS account

First you need to make your AWS account and after login you need to create a new IAM user because when you first register for AWS, you initially sign in as the root user. Who has access permissions to do anything in the account and it is not a good idea to use root user thats why we create new IAM user where you manage user accounts as well as the permissions for each user.

- To create a new IAM user, go to the IAM Console, click Users, and then click the Add Users button. Enter a name for the user, and make sure “Access key — Programmatic access” is selected. Click the Next button.
- Add the AdministratorAccess Managed Policy to your IAM user.
- Click Next a couple more times and then the “Create user” button. AWS will show you the security credentials for that user, which consist of an Access Key ID and a Secret Access Key. These credentials give access to your AWS account, so store them somewhere secure.
- After you’ve saved your credentials, click the Close button. You’re now ready to move on to using Terraform.

# Install Terraform

Install Terraform manually by going to the Terraform home page, clicking the download link, selecting the appropriate package for your operating system, downloading the ZIP archive, and unzipping it into the directory where you want Terraform to be installed. The archive will extract a single binary called terraform, add the PATH of your directory where you unzip the package in environment variable.

Run terraform command for checking the terraform is working properly or not. You got terraform related commands if terraform is working properly.

Now, you will need to set the AWS credentials for the IAM user you created earlier as the environment variables AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY. For example,

here is how you can do it in a Unix/Linux/macOS terminal:
```
$ export AWS_ACCESS_KEY_ID=(your access key id)
$ export AWS_SECRET_ACCESS_KEY=(your secret access key)
```
here is how you can do it in a Windows command terminal:
```
$ set AWS_ACCESS_KEY_ID=(your access key id)
$ set AWS_SECRET_ACCESS_KEY=(your secret access key)
```
Note that these environment variables apply only to the current shell, so if you reboot your computer or open a new terminal window, you’ll need to export these variables again.

In addition to environment variables, Terraform supports the same authentication mechanisms as all AWS CLI and SDK tools. Therefore, it’ll also be able to use credentials in $HOME/.aws/credentials, which are automatically generated if you run aws configure, or IAM roles, which you can add to almost any resource in AWS.

# Deploy a single server

    I use Visual Studio Code but you can use any text editor like Sublime Text, Atom, and IntelliJ etc.

    1.  Create an empty folder and put a file in it called main.tf that contains the following contents:
        provider "aws" {
        region = "us-east-2"
        }
        This tells Terraform that you are going to be using AWS as your provider and that you want to deploy your infrastructure into the us-east-2 region.

    2.  For each type of provider, there are many different kinds of resources that you can create, such as servers, databases, and load balancers.
        To deploy a single (virtual) server in AWS, known as an EC2 Instance, use the aws_instance resource in main.tf as follows:
        resource "aws_instance" "example" {
        ami = "ami-0fb653ca2d3203ac1"
        instance_type = "t2.micro"
        }
    3.  In a terminal, go into the folder where you created main.tf and run the terraform init command. When you’re first starting to use Terraform, you need to run terraform init to tell Terraform to scan the code, figure out which providers you’re using, and download the code for them.

    4.  Run terraform plan command to see what Terraform will do before actually making any changes. anything with a plus sign (+) will be created, anything with a minus sign (–) will be deleted, and anything with a tilde sign (~) will be modified in place.

    5.  To actually create the Instance, run the terraform apply command. You’ll notice that the apply command shows you the same plan output and asks you to confirm whether you actually want to proceed with this plan.

    Type yes and hit Enter to deploy the EC2 Instance.
    You’ve just deployed an EC2 Instance in your AWS account using Terraform! To verify this, head over to the EC2 console, and you should see your instance is created and running.

    First, notice that the EC2 Instance doesn’t have a name. To add one, you can add tags to the aws_instance resource:
    resource "aws_instance" "example" {
    ami = "ami-0fb653ca2d3203ac1"
    instance_type = "t2.micro"
    tags = {
    Name = "terraform-example"
    }
    }
    Run terraform apply again. When you refresh your EC2 console, you’ll see now youre instance have a name.

    To store it in version control you can create a local Git repository and use it to store your Terraform configuration file and the lock file.
    git init
    git add main.tf .terraform.lock.hcl
    git commit -m "Initial commit"

    You should also create a .gitignore file with the following contents:
    .terraform
    _.tfstate
    _.tfstate.backup

    You should commit the .gitignore file, too:
    git add .gitignore
    git commit -m "Add a .gitignore file"

    Create an account if you don’t have one already, and create a new repository. Configure your local Git repository to use the new GitHub repository as a remote endpoint named origin, as follows:
    git remote add origin git@github.com:<YOUR_USERNAME>/<YOUR_REPO>.git

    Now, whenever you want to share your commits with your teammates, you can push them to origin:
    git push origin main

    And whenever you want to see changes your teammates have made, you can pull them from origin:
    git pull origin main

# Deploy a single web server

    Since the dummy web server in this example is just a one-liner that uses busybox, You pass a shell script to User Data by setting the user_data argument in your Terraform code as follows:

    resource "aws_instance" "example" {
    ami = "ami-0fb653ca2d3203ac1"
    instance_type = "t2.micro"
    user_data = <<-EOF
    #!/bin/bash
    echo "Hello, World" > index.html
    nohup busybox httpd -f -p 8080 &
    EOF
    user_data_replace_on_change = true
    tags = {
    Name = "terraform-example"
    }
    }

    Two things to notice about the preceding code:

    1.  The <<-EOF and EOF are Terraform’s heredoc syntax, which allows you to create multiline strings without having to insert \n characters all over the place. 2. The user_data_replace_on_change parameter is set to true so that when you change the user_data parameter and run apply, Terraform will terminate the original instance and launch a totally new one.

    You need to do one more thing before this web server works. By default, AWS does not allow any incoming or outgoing traffic from an EC2 Instance. To allow the EC2 Instance to receive traffic on port 8080, you need to create a security group:

    resource "aws_security_group" "instance" {
    name = "terraform-example-instance"
    ingress {
    from_port = 8080
    to_port = 8080
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    }
    }

    This code creates a new resource called aws_security_group and specifies that this group allows incoming TCP requests on port 8080 from the CIDR block 0.0.0.0/0.

    Simply creating a security group isn’t enough; you need to tell the EC2 Instance to actually use it by passing the ID of the security group into the vpc_security_group_ids argument of the aws_instance resource.

    To access the ID of the security group resource, you are going to need to use a resource attribute reference.
    Syntax:<PROVIDER>\_<TYPE>.<NAME>.<ATTRIBUTE>
    aws_security_group.instance.id

    You can use this security group ID in the vpc_security_group_ids argument of the aws_instance as follows:

    resource "aws_instance" "example" {
    ami = "ami-0fb653ca2d3203ac1"
    instance_type = "t2.micro"
    vpc_security_group_ids = [aws_security_group.instance.id]
    user_data = <<-EOF
    #!/bin/bash
    echo "Hello, World" > index.html
    nohup busybox httpd -f -p 8080 &
    EOF
    user_data_replace_on_change = true
    tags = {
    Name = "terraform-example"
    }
    }

    When you add a reference from one resource to another, you create an implicit dependency. Terraform parses these dependencies, builds a dependency graph from them, and uses that to automatically determine in which order it should create resources. You can even get Terraform to show you the dependency graph by running the graph command.

    Run the apply command, you’ll see that Terraform wants to create a security group and replace the EC2 Instance with a new one that has the new user data.

    Since the plan looks good, enter yes, and you’ll see your new EC2 Instance deploying. Search for instance in your aws console and click your new Instance, you can find its public IP address in the description panel at the bottom of the screen. Give the Instance a minute or two to boot up, and then use a web browser or a tool like curl to make an HTTP request to this IP address at port 8080:

    $ curl http://<EC2_INSTANCE_PUBLIC_IP>:8080
    Hello, World

# Deploy a configurable web server

    To allow you to make your code more DRY and more configurable, Terraform allows you to define input variables. Here’s the syntax for declaring a variable:
    variable "NAME" {
    [CONFIG ...]
    }

    I use default value to deal with extra command-line arguments every time you run plan or apply.
    variable "server_port" {
    description = "The port the server will use for HTTP requests"
    type = number
    default = 8080
    }

    In addition to input variables, Terraform also allows you to define output variables by using the following syntax:
    output "<NAME>" {
    value = <VALUE>
    [CONFIG ...]
    }

    If you want to show something in output message you use output variable.
    output "public_ip" {
    value = aws_instance.example.public_ip
    description = "The public IP address of the web server"
    }
    Run terraform apply command and you will see the output message which you defined.

# Deploy a cluster of web servers

    An ASG takes care of a lot of tasks for you completely automatically, including launching a cluster of EC2 Instances, monitoring the health of each Instance, replacing failed Instances, and adjusting the size of the cluster in response to load.

    The first step in creating an ASG is to create a launch configuration, which specifies how to configure each EC2 Instance in the ASG.

    Replace aws_instance with aws_launch_configuration as follows:
    resource "aws_launch_configuration" "example" {
    image_id = "ami-0fb653ca2d3203ac1"
    instance_type = "t2.micro"
    security_groups = [aws_security_group.instance.id]
    user_data = <<-EOF
    #!/bin/bash
    echo "Hello, World" > index.html
    nohup busybox httpd -f -p ${var.server_port} &
    EOF
    }

    Now you can create the ASG itself using the aws_autoscaling_group resource:

    resource "aws_autoscaling_group" "example" {
    launch_configuration = aws_launch_configuration.example.name
    min_size = 2
    max_size = 10
    tag {
    key = "Name"
    value = "terraform-asg-example"
    propagate_at_launch = true
    }
    }

    This ASG will run between 2 and 10 EC2 Instances (defaulting to 2 for the initial launch), each tagged with the name terraform-asg-example. Note that the ASG uses a reference to fill in the launch configuration name. This leads to a problem: launch configurations are immutable, so if you change any parameter of your launch configuration, Terraform will try to replace it. Normally, when replacing a resource, Terraform would delete the old resource first and then creates its replacement, but because your ASG now has a reference to the old resource, Terraform won’t be able to delete it.

    To solve this problem, you can use a lifecycle setting. Every Terraform resource supports several lifecycle settings that configure how that resource is created, updated, and/or deleted. A particularly useful lifecycle setting is create_before_destroy. If you set create_before_destroy to true, Terraform will invert the order in which it replaces resources, creating the replacement resource first (including updating any references that were pointing at the old resource to point to the replacement) and then deleting the old resource. Add the lifecycle block to your aws_launch_configuration as follows:

    resource "aws_launch_configuration" "example" {
    image_id = "ami-0fb653ca2d3203ac1"
    instance_type = "t2.micro"
    security_groups = [aws_security_group.instance.id]

    user_data = <<-EOF
    #!/bin/bash
    echo "Hello, World" > index.html
    nohup busybox httpd -f -p ${var.server_port} &
    EOF

    #Required when using a launch configuration with an ASG.

    lifecycle {
    create_before_destroy = true
    }
    }

    There’s also one other parameter that you need to add to your ASG to make it work: subnet_ids. This parameter specifies to the ASG into which VPC subnets the EC2 Instances should be deployed.

    ou can use the aws_vpc data source to look up the data for your Default VPC:

    data "aws_vpc" "default" {
    default = true
    }

    Note that with data sources, the arguments you pass in are typically search filters that indicate to the data source what information you’re looking for. With the aws_vpc data source, the only filter you need is default = true, which directs Terraform to look up the Default VPC in your AWS account.

    To get the ID of the VPC from the aws_vpc data source, you would use the following:
    data.aws_vpc.default.id

    You can combine this with another data source, aws_subnets, to look up the subnets within that VPC:

    data "aws_subnets" "default" {
    filter {
    name = "vpc-id"
    values = [data.aws_vpc.default.id]
    }
    }

    Finally, you can pull the subnet IDs out of the aws_subnets data source and tell your ASG to use those subnets via the (somewhat oddly named) vpc_zone_identifier argument:

    resource "aws_autoscaling_group" "example" {
    launch_configuration = aws_launch_configuration.example.name
    vpc_zone_identifier = data.aws_subnets.default.ids
    min_size = 2
    max_size = 10

    tag {
    key = "Name"
    value = "terraform-asg-example"
    propagate_at_launch = true
    }
    }

    At this point, you can deploy your ASG, but you’ll have a small problem: you now have multiple servers, each with its own IP address, but you typically want to give your end users only a single IP to use. One way to solve this problem is to deploy a load balancer.

# Deploy a load balancer

    AWS offers three types of load balancers:
    Application Load Balancer (ALB).
    Network Load Balancer (NLB).
    Classic Load Balancer (CLB).

    We use ALB. The first step is to create the ALB itself using the aws_lb resource:

    resource "aws_lb" "example" {
    name = "terraform-asg-example"
    load_balancer_type = "application"
    subnets = data.aws_subnets.default.ids
    }

    The next step is to define a listener for this ALB using the aws_lb_listener resource:

    resource "aws_lb_listener" "http" {
    load_balancer_arn = aws_lb.example.arn
    port = 80
    protocol = "HTTP"

    #By default, return a simple 404 page

    default_action {
    type = "fixed-response"

    fixed_response {
    content_type = "text/plain"
    message_body = "404: page not found"
    status_code = 404
    }
    }
    }

    This listener configures the ALB to listen on the default HTTP port, port 80, use HTTP as the protocol, and send a simple 404 page as the default response for requests that don’t match any listener rules.

    By default, all AWS resources, including ALBs, don’t allow any incoming or outgoing traffic, so you need to create a new security group specifically for the ALB.

    resource "aws_security_group" "alb" {
    name = "terraform-example-alb"

    #Allow inbound HTTP requests

    ingress {
    from_port = 80
    to_port = 80
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    }

    # Allow all outbound requests

    egress {
    from_port = 0
    to_port = 0
    protocol = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    }
    }

    You’ll need to tell the aws_lb resource to use this security group via the security_groups argument:

    resource "aws_lb" "example" {
    name = "terraform-asg-example"
    load_balancer_type = "application"
    subnets = data.aws_subnets.default.ids
    security_groups = [aws_security_group.alb.id]
    }

    Next, you need to create a target group for your ASG using the aws_lb_target_group resource:

    resource "aws_lb_target_group" "asg" {
    name = "terraform-asg-example"
    port = var.server_port
    protocol = "HTTP"
    vpc_id = data.aws_vpc.default.id

    health_check {
    path = "/"
    protocol = "HTTP"
    matcher = "200"
    interval = 15
    timeout = 3
    healthy_threshold = 2
    unhealthy_threshold = 2
    }
    }

    Go back to the aws_autoscaling_group resource, and set its target_group_arns argument to point at your new target group and You should also update the health_check_type to “ELB”:

    resource "aws_autoscaling_group" "example" {
    launch_configuration = aws_launch_configuration.example.name
    vpc_zone_identifier = data.aws_subnets.default.ids
    target_group_arns = [aws_lb_target_group.asg.arn]
    health_check_type = "ELB"
    min_size = 2
    max_size = 10

    tag {
    key = "Name"
    value = "terraform-asg-example"
    propagate_at_launch = true
    }
    }

    Finally, it’s time to tie all these pieces together by creating listener rules using the aws_lb_listener_rule resource:

    resource "aws_lb_listener_rule" "asg" {
    listener_arn = aws_lb_listener.http.arn
    priority = 100

    condition {
    path_pattern {
    values = ["*"]
    }
    }

    action {
    type = "forward"
    target_group_arn = aws_lb_target_group.asg.arn
    }
    }

    There’s one last thing to do before you deploy the load balancer — replace the old public_ip output of the single EC2 Instance you had before with an output that shows the DNS name of the ALB:

    output "alb_dns_name" {
    value = aws_lb.example.dns_name
    description = "The domain name of the load balancer"
    }

    Run terraform apply. When apply completes, you should see the alb_dns_name output:

    Outputs:
    alb_dns_name = "terraform-asg-example-1.us-east-2.elb.amazonaws.com"

    Copy down this URL. It’ll take a couple minutes for the Instances to boot and show up as healthy in the ALB. In the meantime, you can inspect what you’ve deployed.

    After some time, test the alb_dns_name output you copied earlier:

    $ curl http://<alb_dns_name>
    Hello, World

    Success! The ALB is routing traffic to your EC2 Instances. Each time you access the URL, it’ll pick a different Instance to handle the request. You now have a fully working cluster of web servers!

    For example, go to the Instances tab and terminate one of the Instances by selecting its checkbox, clicking the Actions button at the top, and then setting the Instance State to Terminate. Continue to test the ALB URL, and you should get a 200 OK for each request, even while terminating an Instance, because the ALB will automatically detect that the Instance is down and stop routing to it.

    add, commit, and push your change in your github repo.

# Clean up

    It’s a good idea to remove all of the resources you created so that AWS doesn’t charge you for them. Because Terraform keeps track of what resources you created, cleanup is simple. All you need to do is run the destroy command.

    It goes without saying that you should rarely, if ever, run destroy in a production environment! There’s no “undo” for the destroy command, so Terraform gives you one final chance to review what you’re doing, showing you the list of all the resources you’re about to delete, and prompting you to confirm the deletion. If everything looks good, type yes and hit Enter; Terraform will build the dependency graph and delete all of the resources in the correct order, using as much parallelism as possible. In a minute or two, your AWS account should be clean again.

# Steps to run the code after we configure AWS account and Terraform on our machine.

1. terraform init
2. terraform plan
3. terraform graph (show you the dependency graph)
4. terraform apply
5. terraform destroy
