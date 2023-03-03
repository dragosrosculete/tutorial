## Import resources into Terraform

We may have resources that already exist in AWS so we need to import them rather than create them from scratch. \
Thankfuly we have this option with Terraform.
Let’s look at importing an iam user. https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_user

* Defining the resource we want to import
```
resource "aws_iam_user" "myusername" {
  # ... other configuration ...
}
```

* Now we can run the command. The myusername is the existing aws iam user
```
terraform import aws_iam_user.myusername myusername
```

* We should get this output 
```
Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
```

> Sometimes the resource might be more complex, so after importing it, we should run **terraform show** in order to see all the resource definition. \
The terraform show command is used to provide human-readable output from a state or plan file. https://developer.hashicorp.com/terraform/cli/commands/show

* Now the resources is imported and we can modify or delete it. Let’s add a tag for it and also add it to a group.
```
resource "aws_iam_user" "myusername" {
  name = "myusername"
  tags = {
    Managed = "terraform"
    email = "myusername@example.com"
  }
}
```

* Add the user to a group
```
resource "aws_iam_user_group_membership" "existing_group" {
  user = aws_iam_user.myusername.name

  groups = [
    aws_iam_group.existing_group.name
  ]
}
```

* Import it into a module
```
terraform import module.myusername.aws_iam_user.this myusername
```
