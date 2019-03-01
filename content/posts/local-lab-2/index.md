---                                                                                                                                                                                   
title: "Local Lab 2"                                                                                                                                                                  
date: 2015-04-23T23:53:00-06:00                                                                                                                                                       
tags: ["setup"]                                                                                                                                                                       
categories: ["homelab"]                                                                                                                                                               
type: "post"                                                                                                                                                                          
toc: true                                                                                                                                                                             
body_classes: "blog"                                                                                                                                                                  
---

## Getting Started
Finally, I had some time to start playing with my Vagrant local lab setup. I decided that the best place to start would be to get a small cluster of machines up and running so I could begin configuring them via ansible. I thought about some servers I want to set up, both for learning and also eventually using on my local net, so those will the be the first few machines I work on. A local DNS set up was up on my list, so I figured I'd work on a server for unbound. Centralized logging management is also on there, so I'm going to try to set up a machine for graylog.

## Vagrant Setup
The first step in vagrant set up is deciding what you want the OS to be. This was a pretty easy one for me since I use the same OS both at work and on my server at home. I grabbed my CentOS box from vagrantbox.es. I used "CentOS 6.5 x86_64" which can be downloaded here. I wanted the newest version of CentOS 6 (to stay consistent with my environments) with the least amount of customization done. This box pretty much only has VirtualBox guest additions.

## Vagrantfile
Now that we've chosen our box, it's time to work on our Vagrantfile. This is where we tell Vagrant (which tells VirtualBox in my case) what machine(s) to install. We want to use version 2 as it is the current version with all the features and yet still very stable (it's been around for a while now). I will put a full copy of this Vagrantfile at the end for the sake of understanding.

```YAML 
VAGRANTFILEAPIVERSION = "2"

Vagrant.configure(2) do |config|
```

Now we can define our boxes. The first box I'm going to create is the host I plan on using for my ansible "master." Even though ansible doesn't necessarily depend on a master server, I still like the idea of one machine to do one task. I don't like the idea of pushing ansible playbooks from the unbound or graylog servers.

This is a pretty standard configuration. I point out the box I want to use, which now means it will be available for my other machines as well. I also added a hostname to keep things easier than if all my boxes ended up being named "Vagrant." I wanted to keep these machines all on their own network just so we can talk between them and yet keep them away from any other network.

```YAML
config.vm.define "ansible", primary: true do |ansible| 
	ansible.vm.box = "centos65-x86_64-20140116" 
	ansible.vm.box_url = "https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box" 
	ansible.vm.hostname = "ansible" 
	ansible.vm.network "private_network", ip: "10.0.0.3" 
end
```

Since we already know we have the box downloaded thanks to it be explicitly set in the ansible box defintion, we can just point the other boxes to the name instead of the url as well. Nothing else new here; a hostname and private IP.

```YAML
config.vm.define "unbound" do |unbound| 
	unbound.vm.box = "centos65-x8664-20140116"
	unbound.vm.hostname = "unbound" 
	unbound.vm.network "privatenetwork", ip: "10.0.0.2" 
end

config.vm.define "graylog" do |graylog| 
	graylog.vm.box = "centos65-x8664-20140116"
	graylog.vm.hostname = "graylog"
	graylog.vm.network "privatenetwork", ip: "10.0.0.6" 
end
```

A quick vagrant up from the directory with this vagrantfile, and now we've soon got three CentOS 6.5 machines all on a small private network where they can talk to each other. Perfect. Step 1 done.

## Full Vagrantfile:

```YAML
VAGRANTFILEAPIVERSION = "2"

Vagrant.configure(2) do |config|

	config.vm.define "ansible", primary: true do |ansible| 
		ansible.vm.box = "centos65-x8664-20140116" 
		ansible.vm.boxurl = "https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x8664-20140116.box" 
		ansible.vm.hostname = "ansible" 
		ansible.vm.network "privatenetwork", ip: "10.0.0.3" end

	config.vm.define "unbound" do |unbound|
		unbound.vm.box = "centos65-x8664-20140116"
		unbound.vm.hostname = "unbound"
		unbound.vm.network "privatenetwork", ip: "10.0.0.2" end

	config.vm.define "graylog" do |graylog| 
		graylog.vm.box = "centos65-x8664-20140116" 
		graylog.vm.hostname = "graylog"
		graylog.vm.network "privatenetwork", ip: "10.0.0.6" end

end
```

## Dive into Ansible
Now we want to start playing with ansible. Since this is where I will build my playbooks that will eventually be pushed into machines in "production" at home, I want to get this all into versioning. Inside of the same directory where the Vagrantfile is, create a directory called ansible. Then cd into it and create a git repository (I won't go into details on this, but here some good instructions if you use bitbucket.org).

I worked mainly off of these instructions from ansible's best practices page. To give a practical example, here is my ansible tree (which I have now pushed up to bitbucket as you hopefully have as well, if you're following along).

```bash
ansible/ 
├── README.md 
├── contributors.txt
├── filter_plugins
├── group_vars
├── host_vars
├── library
├── common.yml
├── production
├── roles 
│   └── common 
│	   ├── defaults 
│	   │   └── main.yml
│	   ├── files 
│	   │   ├── id_rsa.pub 
│	   ├── handlers 
│	   │   └── main.yml 
│	   ├── meta 
│	   │   └── main.yml
│	   ├── tasks
│	   │   └── main.yml 
│	   ├── templates
│	   └── vars 
│	   └── main.yml 
├── site.yml 
└── staging
```

So far, the only role I've started is my "common" role which should be installed everywhere. Currently, I have it setting up my user and installing my ssh key. I don't like mixing my code with my data, so I searched around for the best way to handle that set up. I ended up having to put together a couple of ideas to get it to work.

### Common Role
My main tasks file looks like this:

```YAML
- name: create my user 
  user: 
	 name: "{{ usernm }}"
	 groups: "wheel"
	 append: "true" 
	 shell: "/bin/bash"

- name: add auth key
  authorizedkey:
	 user: "{{ item.name }}"
	 key: "{{ item.key }}"
	 state: present
	withitems: ssh_users 
```

I found a post that was used as a basis of my user role. A few things I didn't like about it though: 

1) it sets up passwords and keys where I would prefer keys only 
2) It tells you to install the public key in the variables file. That seems messy to me having the big blob of a pubkey in my variables file. 
3) It explicitly calls out the username, where I would prefer to use a variable in there.

### Variables
I ended up finding another post which shows a better way of moving the pubkeys out of the actual variables file. Basically you add the pubkey file to the files directory inside of the common role. Then you have the variable file read in that info upon execution. That looks like this:

```YAML
usernm: "soehlert"

sshusers: 
	- name: "{{ usernm }}" 
	  key: "{{ lookup('file', 'idrsa.pub') }}" 
```

You'll also notice this is where I set my username.

### Dependencies
The only other file I edited for this so far was to put the dependencies in the meta/main.yml file. We don't have any dependencies at the moment, but I still like making that explicit so my file looks like this:

```YAML
dependencies: [] 
```

### Playbook File
Now we have to create a new file in the base of our ansible tree. I called it common, as this is the playbook we will use to install all common users, files, packages, etc. That is the file we point to when running ansible and tells ansible what we want to happen.

```YAML
---
- hosts: private 
  user: vagrant 
  sudo: True 
  sudo_user: root 
  roles:
  	- role: common
```

### Ansible Hosts
You might have noticed that in my playbook file, I specified "- hosts: private" and wondered where that comes from. When setting up ansible, there is a hosts inventory you can set "groups" for ansible to work on more information can be found here.

I have a private group in mine (for the hosts on my vagrant based private network). I was able to use the hostnames because of editing /etc/hosts on my ansible machine, but you can also just use the IPs.

```YAML
[private]
127.0.0.1
unbound
graylog
```

### Ansible Roles Config
There is one last step before we can use this playbook. We have to tell ansible where to find the roles directory. This is done in the /etc/ansible/ansible.cfg file. It should look like this:

```bash 
[defaults]

roles_path = /vagrant/ansible/roles 
```

Now let's run ansible:

```bash
ansible-playbook common.yml
```

The output is easy to parse, but you should see it create your user and add the key to each host.

## Wrapup
That should be it for now. We now have a small network with three machines to start playing with. As far as ansible, since I'm still learning it, I may have messed up the terminology. If so, please correct me so I can learn the proper words. I also have seen many different methods for setting up the ansible tree, feel free to share if you have another method that you think is better, or you just want to discuss why you do it that vs this way.