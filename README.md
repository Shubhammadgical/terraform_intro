# An Introduction to Terraform

This article would guide us about Introduction of terraform. We shall discuss the following:

1. Deploy a single server
2. Deploy a single web server
3. Deploy a configurable web server
4. Deploy a cluster of web servers
5. Deploy a load balancer
6. Clean up

I took inspiration from this [article](https://blog.gruntwork.io/an-introduction-to-terraform-f17df9c6d180#a9b0) and you should first read this blog to understand the steps properly. and the code explained in this [video part-1](https://shorthillstech-my.sharepoint.com/:v:/p/kapil_jain/Ef0H3oWHQyBMiocS2O9rVRsBm3RQpxwZBmJD-AlXmZsmnA?e=zO9UJ9) and [video part-2](https://shorthillstech-my.sharepoint.com/:v:/p/kapil_jain/EY7NTuItyotJlQpC_KdtEqgBT7I-A9fQF38hKiqzxZzOsA?e=kVkQBF)

# Installing / Getting started

Install Terraform [video](https://www.youtube.com/watch?v=Cn6xYf0QJME)

Setup your AWS account [video](https://www.youtube.com/watch?v=gA9pl-A9gDM)

create IAM user with progammatic access and administrator Access [video](https://www.youtube.com/watch?v=Xx_-IA9qnuI)

# Steps to run the code after we configure AWS account and Terraform on our machine.

1. Run terraform init command. It initializes a working directory containing Terraform configuration files.

```
terraform init
```

2. Run terraform plan command. It creates an execution plan, which lets you preview the changes that Terraform plans to make to your infrastructure.

```
terraform plan
```

3. Run terraform graph command(show you the dependency graph)

```
terraform graph
```

4. Run terraform apply command. I perform the plan.

```
terraform apply
```

5. Run curl <alb_dns_name> (After running terraform apply you got a url in alb_dns_name outputs. Example: curl terraform-asg-example-1288647294.us-east-2.elb.amazonaws.com).

6. Run terraform destroy command

```
terraform destroy
```

# For Deploy

Remove code from line no. 7 to 14 and from line no. 20 to 151(I not provide a image for it because it have so many lines of code).
![r1](https://user-images.githubusercontent.com/101810595/200256308-ea209eab-9fa8-4df2-be5e-b4822a7b7689.PNG)

## Deploy a single server

1. Run below commands:

```
   terraform init
   terraform plan
   terraform apply
   terraform destroy
```

## Deploy a single web server

1. Make change in main.tf.
![r2](https://user-images.githubusercontent.com/101810595/200258101-1c4da4b1-ac69-4b3f-bdec-49af16611dcb.PNG)

2. Run below commands:

```
   terraform init
   terraform plan
   terraform apply
```

3. Go to your aws instance and you’ll see your new EC2 Instance deploying. Click your new Instance, copy its public IP address in the description panel at the bottom of the screen. Give the Instance a minute or two to boot up, and then use a web browser or a tool like curl to make an HTTP request to this IP address at port 8080. Run below commands.

```
   curl http://<EC2_INSTANCE_PUBLIC_IP>:8080
   terraform destroy
```

## Deploy a configurable web server

1. Add below code in main.tf.
![r3](https://user-images.githubusercontent.com/101810595/200258868-45ab88cc-76ea-4df9-b8a7-9347bd62fdcb.PNG)

2. Run below commands:

```
   terraform init
   terraform plan
   terraform apply
   terraform destroy
```

## Deploy a cluster of web servers

1. Add below code in main.tf.
![r4](https://user-images.githubusercontent.com/101810595/200259316-e09f64de-588a-449e-b6b8-4c278d51d717.PNG)

2. Run below commands:

```
   terraform init
   terraform plan
   terraform apply
   terraform destroy
```

## Deploy a load balancer

1. Add below code in main.tf.
![r5](https://user-images.githubusercontent.com/101810595/200260120-324d96b5-118e-489d-9c3f-ec33274c5b7a.PNG)
![r6](https://user-images.githubusercontent.com/101810595/200260127-6ae98d63-ec4b-4304-a5f8-834b9187c95f.PNG)
![r7](https://user-images.githubusercontent.com/101810595/200260134-0c701b01-209f-467a-980b-c94b5eda764c.PNG)

2. Run below commands:

```
   terraform init
   terraform plan
   terraform apply
```

3. After running terraform apply you got a url in alb_dns_name outputs. Run below commands

```
   Run curl http://<alb_dns_name>
   terraform destroy
```
