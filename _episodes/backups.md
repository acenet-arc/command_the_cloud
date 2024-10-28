---
layout: episode
title: "Backups"
teaching: 20
exercises: 0
questions:
- "Should software and data be treated similarly with backups?"
- "Can software \"backups\" be more future proof?"
- "What are some good tools for backing up data?"
- "Is there a convenient place to store backups?"
objectives:
- ""
keypoints:
- ""

start: false
---

In the last episode we learned about the differences between snapshots and images with a focus on images. We learned that images are a copy of the contents of a volume and that we can download and upload images from and to a cloud. An image of the VMs boot volume contains the whole operating system and all the software that has been installed and configured on your VM as well as any data you may have put there. Could they be used as backups? In a pinch they could be used as backups, but they aren't the best solution for a variety of reasons.

* Saving your operating system and software in a backup means that backups will be larger than necessary.
* Restoring from an old backup will mean that the software will be outdated. Outdated software will be a security risk and should not be connected to the Internet.
* VMs contain special settings that allow them to work with a particular cloud, such as networking settings which may or may not be transferable between clouds.
* Less flexible in the sense that it will involve some work to pull needed data out of the image and use in a new VM.

There is a better approach. Backups of VMs can be considered in two main components:
* Software
* Data

## Software Backup
Backing up software should not be treated in the usually sense of saving the specific bits and bytes of the software. Instead save the process of installing and configuring the software. This allows you to re-create your VM with up-to-date versions of the software later.

As a first step towards a better solution when you create your VM initially, keep good notes about the software you install and configure to allow you to more easily install and configure software on a new VM. I usually keep specific commands I run and notes explaining what the commands do and what the I am using options are.

Taking notes is certainly a good place to start however there are other options which can allow you to benefit from the work of others and allow automation. The details of these options is beyond the scope of this workshop but I wanted to mention two main options with just enough of a description to let you know why you might want to use them and some of the benefits they bring.

### Provisioning tools
Taking this approach further, a provisioning tool can be used. A provisioning tool is software that allows you to save configurations and automate the installation and configuration of software. Some commonly used provisioning tools are:
* [ansible](https://docs.ansible.com/ansible/latest/index.html)
* [puppet](https://www.puppet.com/)
* [salt](https://docs.saltproject.io/en/latest/topics/index.html)
* [chef](https://www.chef.io/)

These tools do have a bit of a learning curve however do have significant benefits especially if you are managing more than a couple virtual machines. I have a tried out the top three tools listed, and have most experience with ansible. In my experience ansible is the simplest and easiest to use.

Many of these tools have configurations for common software that are maintained by others which can help ease the burden of developing and maintaining these installation processes. For example the ansible provisioning tool has the [Ansible Galaxy](https://galaxy.ansible.com/ui/) and as an example of a common configuration here is an [apache](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/apache/documentation/) web server role. There are usually various configuration and variable options allowing you to customize configurations while still benefitting from best practices when using configurations from reputable and experienced developers.

### Containers
Another option, which can be used either separate from or in conjunction with provisioning tools, are containers. Containers can be though of like very light weight virtual machines. They share some parts of the underlying operating system hosting the containers yet are also isolated from the underlying operating system to some degree. This isolation can have security benefits. Containers usually contain some software and optionally data. Containers being light weight allows one to pull down new containers and run them in place of older outdated containers. This provides a way to easily update software and configurations. Some popular containerization tools are:
* [Docker](https://docs.docker.com/get-started/docker-overview/)
* [Apptainer](https://apptainer.org/)
* [Podman](https://podman.io/)

As with the provisioning tools there are repositories of different containers providing different software configurations. In the case of docker there is [docker hub](https://hub.docker.com/). As an example here is the [mysql](https://hub.docker.com/_/mysql) database container on docker hub. When updates are made new containers can be pulled down and services can be restarted to run with these new updated containers.

## Data Backup
In contrast backing up data can be much more straight forward, just copy the bits and bytes to some where safe. It can be that simple using something like [sftp](https://en.wikipedia.org/wiki/SSH_File_Transfer_Protocol) or slightly more sophisticated with [rsync](https://en.wikipedia.org/wiki/Rsync) which will only transfer updated files. However there are even more feature rich backup tools that allow you to keep multiple snapshots of your data, remove older versions when storage gets tight, and optionally compression and encryption of data. A couple I have used are:

* [Duplicity](https://duplicity.gitlab.io/)
* [Restic](https://restic.net/)

Duplicity is a command line tool that works great, but it can be a bit cumbersome to use. There is a nice GUI front end available for Duplicity, [Deja Dup](https://apps.gnome.org/en-GB/DejaDup/) which is simple and easy to use, this is what I use on my own linux laptop to do my backups. On VMs you likely don't want a GUI interface. I have used Duplicity directly on the command line and it works just fine, though I found myself writing some python scripts to "drive" it. I found the command line usage of Restic to be a little simpler than duplicity.

Both Duplicity and Restic support the S3 protocol used with many different object store providers including the Alliance OpenStack clouds. Object store can be a convenient place to store backups.

<!-- You don't need to create a container, restic will do that for you
Create a container
~~~
(openstack) container create cgeroux-workshop-container
~~~
{: .bash}
~~~
+-------------------+-------------------+---------------------+
| account           | container         | x-trans-id          |
+-------------------+-------------------+---------------------+
| AUTH_############ | cgeroux-workshop- | tx################# |
| ################# | contianer         | ####-##########-    |
| ###               |                   | ######-default      |
+-------------------+-------------------+---------------------+
~~~
{: .output}
-->

Create credentials to access project's object store with tools other than the OpenStack CLI.
~~~
(openstack) ec2 credentials create
~~~
{: .bash}

~~~
+------------+------------------------------------------------+
| Field      | Value                                          |
+------------+------------------------------------------------+
| access     | ################################               |
| links      | {'self': 'https://arbutus.cloud.computecanada. |
|            | ca:5000/v3/users/############################# |
|            | ca:5000/v3/users/############################# |
|            | ###################################/credential |
|            | s/OS-EC2/################################'}    |
| project_id | ################################               |
| secret     | ################################               |
| trust_id   | None                                           |
| user_id    | ############################################## |
|            | ##################                             |
+------------+------------------------------------------------+
~~~
{: .output}

Leave the OpenStack prompt.
~~~
(openstack) exit
~~~
{: .bash}

Set environment variables to allow tools like s3cmd and restic to operate on containers.
~~~
$ export RESTIC_BACKUP_URL="s3:https://object-arbutus.cloud.computecanada.ca:443/<container-name>"
$ export AWS_ACCESS_KEY_ID="################################"
$ export AWS_SECRET_ACCESS_KEY="################################"
~~~
{: .bash}

~~~
$ restic -r $RESTIC_BACKUP_URL init
~~~
{: .bash}
~~~
enter password for new repository:
enter password again:
created restic repository 103f316734 at s3:https://object-arbutus.cloud.computecanada.ca:443/cgeroux-workshop-container

Please note that knowledge of your password is required to access
the repository. Losing your password means that your data is
irrecoverably lost.
~~~
{: .output}

**Characters you enter as password are not shown.**

Create some things to backup.
~~~
$ mkdir work
$ echo "stuff to save">work/important_file.txt
$ cat work/important_file.txt
~~~
{: .bash}
~~~
stuff to save
~~~
{: .output}

Now create a backup of the `work` folder.
~~~
$ restic backup -r $RESTIC_BACKUP_URL ./work/
~~~
{: .bash}
~~~
enter password for repository:
repository 103f3167 opened (version 2, compression level auto)
created new cache in /home/user03/.cache/restic
no parent snapshot found, will read all files


Files:           1 new,     0 changed,     0 unmodified
Dirs:            1 new,     0 changed,     0 unmodified
Added to the repository: 755 B (697 B stored)

processed 1 files, 14 B in 0:00
snapshot b765d737 saved
~~~
{: .output}

List snapshots in backup
~~~
$ restic snapshots -r $RESTIC_BACKUP_URL
~~~
{: .bash}
~~~
enter password for repository: 
repository 103f3167 opened (version 2, compression level auto)
ID        Time                 Host        Tags        Paths
------------------------------------------------------------------------
b765d737  2024-10-18 17:57:37  ctc                     /home/user03/work
------------------------------------------------------------------------
1 snapshots
~~~
{: .output}
~~~
$ rm -rf ./work/
$ ls 
~~~
{: .bash}
~~~
-rw-rw-r-- 1 user03 user03       1959 Sep 13 13:28  openstack-rc.sh
drwxrwxr-x 4 user03 user03       4096 Sep  5 18:20  pyenvopst
~~~
{: .output}

Then restore it from backup.
~~~
$ restic restore -r $RESTIC_BACKUP_URL --target=./ b765d737
~~~
{: .bash}
~~~
enter password for repository: 
repository 103f3167 opened (version 2, compression level auto)

restoring <Snapshot b765d737 of [/home/user03/work] at 2024-10-18 17:57:37.919267415 +0000 UTC by user03@ctc> to ./
Summary: Restored 2 files/dirs (14 B) in 0:00
~~~
{: .output}
~~~
$ ls
~~~
{: .bash}
~~~
-rw-rw-r-- 1 user03 user03       1959 Sep 13 13:28  openstack-rc.sh
drwxrwxr-x 4 user03 user03       4096 Sep  5 18:20  pyenvopst
drwxrwxr-x 2 user03 user03       4096 Oct 18 17:56  work
~~~
{: .output}

Lots more to learn about using restic, see [restic documentation](https://restic.readthedocs.io/en/stable/index.html).


