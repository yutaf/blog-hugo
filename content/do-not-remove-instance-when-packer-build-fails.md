+++
date = "2017-07-04T13:00:56+09:00"
draft = false
title = "Do not remove instance when packer build fails"
tags = ["packer", "aws", "ec2"]

+++

<!--more-->

I want to run commands manually inside the instance for investigation when packer build fails.  
Packer remove the instance by default when the build fails.  
We can change the behavior by `-on-error=abort` option.  

```
$ AWS_ACCESS_KEY=*** AWS_SECRET_KEY=*** packer build -on-error=abort app.json
```

Then you can ssh to the instance after packer build exited with error.  

---

### ssh into EC2 instance built by packer

This time I use `AMI Builder (EBS backed)` as builder.  
Packer allocates ssh key randomly by default and I don't know where it is.  
So I can't ssh into the instance.  
To avoid this, we can specify ssh keys by `ssh_keypair_name` and `ssh_private_key_file` in the packer config file.  
<https://www.packer.io/docs/builders/amazon-ebs.html#ssh_keypair_name>  

```
{
  "variables": {
    "aws_access_key": "",
    "aws_secret_key": ""
  },
  "builders": [{
    "type": "amazon-ebs",
    "access_key": "{{user `aws_access_key`}}",
    "secret_key": "{{user `aws_secret_key`}}",
    "region": "us-east-1",
    "vpc_id": "***",
    "subnet_id": "***",
    "source_ami": "ami-427a392a",
    "instance_type": "c3.large",
    "ssh_username": "ubuntu",
    "ssh_keypair_name": "salt",
    "ssh_private_key_file": "/Users/me/.ssh/salt.pem",
    "ami_name": "image-app {{timestamp}}"
  }],
  "provisioners": [
    ...
    {
      "type": "shell",
      "inline": [
        "sudo curl -o bootstrap-salt.sh -L https://bootstrap.saltstack.com",
        "sudo sh bootstrap-salt.sh -P git v2016.11.6"
      ]
    },
    {
      "type": "salt-masterless",
      "skip_bootstrap": true,
      "local_state_tree": "../file_roots",
      "local_pillar_roots": "../pillar_roots",
      "minion_config": "minion-app"
    },
    ...
  ]
}
```

Then I can ssh into the instance with the key I specified.  
