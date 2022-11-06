# An Introduction to Terraform

This article would guide us about Introduction of terraform. We shall discuss the following:

1. Set up your AWS account
2. Install Terraform
3. Deploy a single server
4. Deploy a single web server
5. Deploy a configurable web server
6. Deploy a cluster of web servers
7. Deploy a load balancer
8. Clean up

I took inspiration from this [article](https://blog.gruntwork.io/an-introduction-to-terraform-f17df9c6d180#a9b0) and you should first read this blog to understand the steps properly.

Install Terraform [video](https://www.youtube.com/watch?v=Cn6xYf0QJME)

Setup your AWS account [video](https://www.youtube.com/watch?v=gA9pl-A9gDM)

create IAM user with progammatic access and administrator Access [video](https://www.youtube.com/watch?v=Xx_-IA9qnuI)

# Steps to run the code after we configure AWS account and Terraform on our machine.

1. Run terraform init command

```
terraform init
```

2. Run terraform plan command

```
terraform plan
```

3. Run terraform graph command(show you the dependency graph)

```
terraform graph
```

4. Run terraform apply command

```
terraform apply
```

5. Run curl <alb_dns_name> (After running terraform apply you got a url in alb_dns_name outputs. Example: curl terraform-asg-example-1288647294.us-east-2.elb.amazonaws.com).

6. Run terraform destroy command

```
terraform destroy
```
