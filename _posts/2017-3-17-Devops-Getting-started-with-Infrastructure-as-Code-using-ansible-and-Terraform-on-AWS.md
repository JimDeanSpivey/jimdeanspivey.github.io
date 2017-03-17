---
layout: post
title: Getting started with Infrastructure as Code using Ansible and Terraform. And testing it locally with Vagrant!
---

This post is a quick start set of default files and practices to get started storing all infrastructure (using Terraform) and how to provision it (using ansible). As opposed to just jumping in as a newbie (me) and exploring, I find it much faster to just jump in using a seasoned veterans (someone who knows this domain for awhile) defaults/best-practices. I also cover how I tested my infrastructure before deploying it, by using Vagrant. I also use Vagrant to test new incremental changes.

### Terraform quickstart with AWS

Overall, this part is quite simple. Terraform itself can be jumped into without having to worry much about best practices as it really is rather simply and just abstracts away a lot of details (from specific cloud providers). So this is really more of a very brief quick start into AWS common practices. I found Nick Charlton's [Terraform: AWS VPC with Private and Public Subnets](https://nickcharlton.net/posts/terraform-aws-vpc.html) blog post to be very helpful for using Terraform with AWS. This helped me to get started using AWS infrastructure design using a public and private virtual clouds (vpc). And also having a bastion access node inside the public vpc.

### Provisioning the created instances with ansible

So now that the bare cloud compute instances (ec2 sine we're on AWS) are created, they need to be provisioned to run a given app. I used ansible to install various depedencies such as redis and postgresql. This part really needs good defaults and solid best practices. For that, I found this github project [ansible-best-practices](https://github.com/enginyoyen/ansible-best-practises) to be very good. In brief, with ansible, you want to use roles for everything. Because they encompase the entire task (play) and force it to be seperated from the specific configuration values (ansible vairables) that you need to feed it. The "ansible-best-practices" repo is very helpful for managing external and internal roles. Just clone it and edit `roles/roles_requirements.yml`by adding exertnal roles. Then run the shell script `extensions/setup/role_update.sh` and it will download them (eg: a role to install postgresql). You can then put your own custom roles (eg: a role to seed tables in your postgresql database with values) into `roles/internal`. The project is fully documented so give it a try, great way to get started using ansible.

One thing I did differently was to use git-crypt instead of ansible-vault. Because with git-crypt, I can use it for things outside of ansible and I also have access to working git diffs and history.

### Time to test the ansible roles locally, using Vagrant

Once you have your ansible roles defined (externally, and maybe some internal ones you wrote yourself), it's a ~~good~~ great idea to test it locally (instead of deploying to AWS). The ansible provisiong step can be complex, there are many configurations files to edit and some might not work on all distros. In my limited experience, it's unlikely to just work and provision everything. And as infrastructure changes, having a local instance to apply incremental infrastructure changes would also be good. And better to get it started now than later for sure.

So I use Vagrant to test this. Here is my Vagrant file from a simple project where a twitter API client sits in the public VPC and a database node records calculated data that is sent from the public VPC instance to the private VPC database. "ti" is short for tweet ingestor.

```ruby
Vagrant.configure("2") do |config|
  config.vm.define "ti" do |ti|
    ti.vm.box = BOX
    ti.vm.hostname = 'ti'

    ti.vm.network :private_network, ip: "10.0.1.5"
    ti.vm.network :forwarded_port, guest: 22, host: 10122, id: "ssh"


    ti.vm.provider "parallels" do |pr|
      pr.name = "infra_tweetingestor"
      pr.cpus = 1
      pr.memory = 512
    end
  end

  config.vm.define "db" do |db|
    db.vm.box = BOX
    db.vm.hostname = 'db'

    db.vm.network :private_network, ip: "10.0.2.5"
    db.vm.network :forwarded_port, guest: 22, host: 10222, id: "ssh"
    db.vm.network :forwarded_port, guest: 6379, host: 6380, id: "redis"
    db.vm.network :forwarded_port, guest: 5432, host: 5433, id: "postgresql"

    db.vm.provider "parallels" do |pr|
      pr.name = "infra_db"
      pr.cpus = 1
      pr.memory = 512
    end
  end
  ```
  
And my ansible inventory file actually doesn't differ between `production.ini` and `development.ini` for ip address because both my Vagrant defined networks and AWS vpc use the same ip subnets (for Vagrant private networks) and also ip addresses as well. For example, `10.0.2.5` both point to the same instance in produciton AWS vpc and also the local Vagrant host. It's nice to have the least needed changes between production and development. Not just for replication but also less things to remember to re-config when going live. I also use parralels because virtualbox on OSX was causing the operating system to hang and freeze-up (requiring a hard reboot).
  
To test it all, I run ansible like this: `ansible-playbook -i plays/local_dev.ini db.yml -vv`. If I want to just test incremental changes, I make sure to tag my roles (in my playbook). So if I want to just re-popuate the database, I run `ansible-playbook -i plays/local_dev.ini db.yml --tags "dbload" -vv`
  
  
### Deploying it all to AWS
  
Here is my workflow to bring it all online. Every step to go from **only** code to live.
  
  1. Run `terraform apply`
  2. SSH into to the bastion node in the AWS VPC.
  3. Run a file called `install.sh` to bootstrap the bastion node so that it can provision everything. This file is versioned of course! This script installs ansible, git clones my "Infrastructure" repository, and also pulls down private keys needed to decrypt files in the "Infrastructure" repository (remember, I'm using git-crypt instead of ansible-valut). This script also runs `extensions/setup/role_update.sh` to download external roles. **The point of this file is so that I can perform all of these steps, without having to poke around, remember stuff off the top of my head, try to find some old lines, google stuff, etc.** This file should just work and bring everything up to speed for ansible to do the rest.
  4. Manually run the ansible playbook(s). I prefer to manually run them, instead of invoking them from `install.sh`
  
  
  
