---
layout: post
title: "Docker with Systemd"
author: "jantwisted"
categories: docker, centos, systemd
tags: [tech]
image:
---

I've been really getting into Docker after moving my blog into a container. The containers are pretty impressive, easy to deploy and of course they are flexible. For the very first time I tried to do an assignment on Docker and the task was simple, which is to set up Apache server in a container that is created from a base image (CentOS). Everything was fine, I pulled an image (centos:latest) and created a container, ran it with bash to login and then I installed `httpd` (apache server). As the final step, I started the `httpd.service` by issuing `systemctl start httpd`. And I got the following error.

```
Failed to get D-Bus connection: No connection to service
```

So, this is what I learned after a some consideration of the issue I faced. A docker container isn't a VM, so it's more like a *NIX process. Easiest way to run httpd is run it with my own process manager.

```shell
/usr/sbin/httpd -k start
```
Now, I'm going to show how it can be run with systemd tools. This is a pain and insecure in certain ways. But let's do it for fun !

1. Create a customized image from the base image, here we delete some unit files of systemd to avoid some issues.
```shell
FROM centos:latest
ENV container docker
RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == \
systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*;\
rm -f /etc/systemd/system/*.wants/*;\
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*;\
rm -f /lib/systemd/system/anaconda.target.wants/*;
VOLUME [ "/sys/fs/cgroup" ]
CMD ["/usr/sbin/init"]
```

2. Build the customized based image.
```shell
docker build -t local/centos-systemd .
```

3. Create another docker image from the customized base image.
```shell
FROM local/c-systemd
MAINTAINER janith <janith@member.fsf.org>
RUN yum -y update && yum -y install httpd && yum clean all
RUN systemctl enable httpd.service
#optional
EXPOSE 80
CMD ["/usr/sbin/init"]
```

4. Build the new image.
```shell
   docker build -t local/centos-httpd .
```
5. Create a container based on the new image. While creating, we need to mount cgroups volumes from the host in read only mode (ro).
```shell
docker run -itd --privileged -v /sys/fs/cgroup:/sys/fs/cgroup:ro -p 80:80 local/centos-httpd
```

6. Finally, the apache server must be started, and you can check it from a browser (for terminal based, use elinks).
```shell
elinks http://localhost
```

7. Let's check the apache server status.
```shell
docker exec -it <container_name> /bin/bash
systemctl status httpd
```

That's it, and I hope this will help someone who's struggling with systemd containers.

#### References:

- https://github.com/docker-library/docs/tree/master/centos#systemd-integration
- https://github.com/moby/moby/issues/7459


