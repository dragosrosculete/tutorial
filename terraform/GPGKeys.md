## GPG Keys and usage in Terraform

GnuPG is a complete and free implementation of the OpenPGP standard as defined by RFC4880 https://www.ietf.org/rfc/rfc4880.txt (also known as PGP) \
GnuPG allows you to encrypt and sign your data and communications; it features a versatile key management system, along with access modules for all kinds of public key directories. https://gnupg.org/ \
We need this for Terraform in order to create AWS IAM USERS.

* Install on MAC OS

```brew install gpg```

* Check for existing keys and initialize:

```gpg -k```

* Create a template file that will be used to generate the key. Let’s called it gpg_template.

```
Subkey-Type: default
Name-Real: devops
Name-Comment: Use PGP to Encrypt Your Terraform Secrets
Name-Email: devops@example.com
Expire-Date: 0
```

* Now we need to apply the template in order to generate a pub/secret key. When executed it will ask for a passphrase. Write it down  you will need it each time your decrpyt.

```gpg --batch --gen-key gpg_template```

* You should see an output like this:
```
gpg: key 792DAFC243E35A26 marked as ultimately trusted
gpg: directory '/Users/example/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/Users/example/.gnupg/openpgp-revocs.d/F511B6A31C9A694AD4FF0CD4792DAFC243E35A26.rev'
```

Cool. Now we need to make it available for Terraform and also for other Devops team members.

* So let’s export the Pub Key in binary format 

```gpg --output ~/public-key-binary-terraform.gpg --export devops@example.com```

* Now lets export the secret key, DONT SHARE THIS, DONT COMMIT IT TO GIT !!!

```gpg --output ~/secret-key-binary-terraform.gpg --export-secret-key devops@example.com```

* If you want to share the keys with other members that you trust:
```
gpg --import ~/public-key-binary-terraform.gpg 
gpg --import ~/secret-key-binary-terraform.gpg
```

## How to use in Terraform

Now that we have the key created we can use it in Terraform in order to create the AWS user.

* First we tell it where to find the Pub key . Replace the path.
```
data "local_file" "pgp_key" {
  filename = "/path/to/your/public-key-binary-terraform.gpg"
}
```

* Next we need to create the user. This creates the user but no login yet.
```
resource "aws_iam_user" "new_user" {
  name = "new_user"
  path = "/"

  tags = {
    Managed = "terraform"
    email = "new_user@example.com"
  }
}
```

* Now we create the login for the user . Here we can see we are passing the data(pub key) and tell it to base64 encode as required by Terraform .
```
resource "aws_iam_user_login_profile" "new_user" {
  user    = aws_iam_user.new_user.name
  pgp_key = data.local_file.pgp_key.content_base64
}
```

* We should also add an output so we can actually get the password once the user is created . This password will be retrieved encrypted
```
output "new_user_password" {
  value = aws_iam_user_login_profile.new_user.encrypted_password
}
```

* Now , we need to execute the standard commands. If we are ok with the changes we see in terraform plan we can then execute terraform apply.
```
terraform init
terraform plan
terraform apply
```

* We will get an output once it is applied, something like this : 
```
Outputs:
new_user_password = wcDMA3i3hz0G91B3AQwAFlnedByrasL7Ni3PmkSaaih7JwW7yEMbTrMHyIwDXeOqmXFJcG23iCZzcHDHAtWNTXBiLiXiPdptkssujFV9h2h/tBkqEsiUKFiA5z4pwf7zDkn/beZ4mhTLWmAFQEq2zcynvzAby9iXz5lq6sLbfeWn1n5tv/l9rqdUOdIObODw2r9y+3aDs9WVOkjHiSI70BFv7S30adrEYT6mn1xd5D0aPVCfzMRjjVNmwU4ScXwKJXshTCQvLLG8AqH8sAAo5H/djiW/39XNO1HeywDWIr/TsspcFW+iGZnTvsLgfPNDTSlJwL2iIdrpTnf6Dkpsp8inHsKZW4QkMybUzVLMR6RbKnUxR8QNzkMC+obQswmTUHPlgRtalTG/Z+ILnBXrLl4TlYfgpZ1SmJaiIYrH5Ork9gXh5NgeribT3cVxTYkCyPs8wy22qaYs+j4npGSloVjOUrL51HUS5oOVJpzxW2do/j/eZVQ3XqKm6MR8U/FpnJvtO/JdhDPC68CHNoUF0uAB5PB/yrQGoHdOqtgc7vpPsCXh+7/gZOBq4dOf4H3iljC5W+AX5AHb9Ieq/2579ibobcSj1Jfgt+J88oxm4Fzk9VRORwHpourJSxHILKC37OI65R2X4cF6AA==
```

* Nice. So now the user is created, it has a password set and it is encrypted.
But we want to give this password to the user so he/she can login on the AWS interface. The next step is to decrypt the password. \
Decrypt the password:
First we need to export GPG_TTY (By correctly setting the environment variable GPG_TTY, GPG clearly understands that we are going to pipe a secret into it.) and then execute the terraform command on that output

```
export GPG_TTY=$(tty)
terraform output new_user | base64 --decode | gpg --decrypt
```

* It will ask us for the passphrase that we’ve set when we’ve created the gpg keys. Once given it will return the unencrypted password
```
gpg: encrypted with 3072-bit RSA key, ID 78B7873D06F75077, created 2020-09-26
"vevops (How to Use PGP to Encrypt Your Terraform Secrets) <devops@example.com>"
#thisisthepassword
```