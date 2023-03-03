## Move resources from one tfstate to another

Let’s say you have a folders structure like this: 
```
general/
  sg.tf   
  variables.tf   
  .terraform/terraform.tfstate
eks/   
  sg.tf   
  variables.tf   
  .terraform/terraform.tfstate
```

Now you realize that some resources that are defined in sg.tf file from folder eks dont belong there and should be in folder general.

You could just delete them from there and create them in another folder but that could impact a number of systems, especially if we are using it on production. So how can we move a sg resource from eks folder to general folder ?

* First we go into eks folder and pull the terraform state into the file , we call it moving_terraform.tfstate
```
cd eks
terraform state pull > moving_terraform.tfstate
```

* We do the same for the terraform state in general folder
```
cd general
terraform state pull > moving_terraform.tfstate
```

* Now we are going to move a resource from eks to general . In our case is a security group called “default”
```
cd ..
terraform state mv -state eks/moving_terraform.tfstate -state-out=general/moving_terraform.tfstate aws_security_group.default aws_security_group.default
```

* Now let’s push the changes to tfstate ( this is backed up by S3)
```
cd general
terraform state push moving_terraform.tfstate
```

* We also need to move the actual resource definition from eks folder to general folder. Cut/paste the resource block from the eks/sg.tf to general/sg.tf
```
resource "aws_security_group" "default" {
  name        = "default"
  description = "default"
  vpc_id = "vpc-12131312"
    
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```
* Finally, lets cleanup the resource definition from eks folder terraform state
```
cd eks
terraform state rm aws_security_group.default
```

* Here is an explanation on how to move a resource to a module https://ryaneschinger.com/blog/terraform-state-move/ \
You can move each individual resource into a module that contains multiple resources

```terraform state mv aws_iam_user.blablabla module.blablabla.aws_iam_user.this```

Where “this” is the name of the resource in the module
