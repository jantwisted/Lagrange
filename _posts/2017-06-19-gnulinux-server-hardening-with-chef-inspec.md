---
layout: post
title: "GNU/Linux Server Hardening with Chef & Inspec"
author: "jantwisted"
categories: tech
tags: [Automation,DevOps,InSpec]
image:
---

With my passion on DevOps techniques and learning new tools, I'm urged to use Chef tools for automation tasks. In this post, I'm using inspec and chef to do a server hardening on GNU/Linux box.

The name itself describes what `inspec` does. It gives a compliance report based on the checklist you have provided. [how-to-install-inspec](https://github.com/chef/inspec)

This is a sample of compliance code.

```Ruby
control'sshd-21'
	title 'Set SSH Protocol to 2'
	desc 'A detailed description'
	impact 1.0 #this is critical
	ref 'compliance guide, section 2.1'
	describe sshd_config do
		its ('Protocol') {should cmp 2}
	end
end
```

Let's find out what operating system we are dealing with,

```Shell
inspec detect -t ssh://root@192.168.56.103 --password 'cookies'
```
```
== Operating System Details

Name:      redhat
Family:    redhat
Release:   6.4
Arch:      x86_64
```

Now, I'm downloading `linux-baseline` compliance code which is provided by dev-sec (A hardening framework)

```Shell
git clone https://github.com/dev-sec/linux-baseline
inspec exec linux-baseline -t ssh://root@192.168.56.103 --password 'cookies'
```
and my result is not promising :disappointed:

```
Profile Summary: 23 successful, 27 failures, 0 skipped
```

If you scroll up the result, you will see what things you need to fix for a better result. However, it's a painful task to fix everything by your own. So I'm using some tools and scripts to do that task for me.

To remediate the targeted operating system, I'm using `knife-zero` with `os-hardening` cookbook. [how-to-install-knife-zero](http://knife-zero.github.io/10_install/)

```
knife zero bootstrap 192.168.56.103 --ssh-user root --node-name production-server
```

This installs chef client on the target. After the installation, I'm going to download and prepare `os-hardening` cookbook and run the run-list.

```Shell
knife cookbook site download compat_resource
knife cookbook site download ohai
knife cookbook site download sysctl
knife cookbook site download os-hardening

mkdir cookbooks

tar xvf *.tar.gz -C cookbooks/

knife zero converge "name:production-server" --ssh-user root --override-runlist os-hardening
```

Once it executed successfully, my server os will be hardened. To verify the status, let's call inspec again.

```
inspec exec linux-baseline -t ssh://root@192.168.56.103 --password 'cookies'
```

And the result is awesome, :wink:

```
Profile Summary: 48 successful, 2 failures, 0 skipped
```

#### References


* Install Inspec, https://github.com/chef/inspec

* Install knife-zero, http://knife-zero.github.io/10_install/

* Hardening Framework, http://dev-sec.io/