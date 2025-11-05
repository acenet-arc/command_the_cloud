---
layout: episode
title: "Creating a Keypair"
teaching: 10
exercises: 0
questions:
- "How can I see what OpenStack commands are possible?"
- "How can I get more information about a particular OpenStack command?"
- "How do you create a keypair?"
- "How do you use the OpenStack CLI to upload a public key to your cloud?"
objectives:
- "Navigate the OpenStack CLI help documentation."
- "Create a public/private keypair to use for authentication with a virtual machine."
- "Upload public key to OpenStack cloud so that it can be injected into new virtual machines."
keypoints:
- "OpenStack command can be used for one off commands by prefixing with `openstack` or a series of commands by first running `openstack` command without any sub commands giving an `(openstack)` prompt."
- "Virtual Machines usually use public/private keypairs to authenticate securely."
- "You can upload a public key to your OpenStack cloud that will be injected into a newly created virtual machine."

start: false
---

As a first task for using our newly setup OpenStack CLI, lets create a new virtual machine. However, in order to access it we will first need to have a keypair created and the public key setup with our OpenStack account so that it can be injected into newly created virtual machines.

We can generate a new key pair to use when we create new virtual machines using `ssh-keygen` command.

~~~
$ ssh-keygen -t ed25519
~~~
{: .bash}
~~~
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/user03/.ssh/id_ed25519):
~~~
{: .output}
Press enter to accept that filename.
~~~
Created directory '/home/user03/.ssh'.
Enter passphrase (empty for no passphrase): 
~~~
{: .output}
It is a good idea to enter a passphrase to protect your private keys. You will not see the characters you type for your passphrase. **Make sure it is something you can easily remember**, we will need to use it later.
~~~
Enter same passphrase again:
~~~
{: .output}
Enter your passphrase again go confirm.
~~~
Your public key has been saved in /home/user03/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:WVLtnV1/8/dp77wexgzexwwlkg6G0LIFLInDx5NbI9M user03@ctc
The key's randomart image is:
+--[ED25519 256]--+
| . o =oo  ..     |
|  + X E.oo  ..  .|
|   o B =o +.o..o+|
|    . .  = o..o++|
|        S   ... +|
|            . =+o|
|             . *B|
|              .++|
|              o+*|
+----[SHA256]-----+
~~~
{: .output}

Now that we have created a public/private keypair, we need to send the public key to OpenStack. Lets see if we can find an OpenStack CLI command to do that.

To find more information about the `openstack` command you can type `openstack help` which produces a huge amount of text describing how the command can be used, all of the available options, and a list of sub commands.
~~~
$ openstack help
~~~
{: .bash}
~~~
usage: openstack [--version] [-v | -q] [--log-file LOG_FILE] [-h]
                 [--debug] [--os-cloud <cloud-config-name>]
                 [--os-region-name <auth-region-name>]

...

options:
  --version             show program's version number and exit
  -v, --verbose         Increase verbosity of output. Can be repeated.
  -q, --quiet           Suppress output except warnings and errors.
  --log-file LOG_FILE
                        Specify a file to log output. Disabled by default.
  -h, --help            Show help message and exit.

...

Commands:
  access rule delete  Delete access rule(s)
  access rule list  List access rules

...

  keypair create  Create new public or private key for server ssh access

...
~~~
{: .output}

You can get additional information about a particular command by typing `openstack help` followed by the command.
~~~
$ openstack help keypair create
~~~
{: .bash}
~~~
usage: openstack keypair create [-h] [-f {json,shell,table,value,yaml}]
                                [-c COLUMN] [--noindent] [--prefix PREFIX]
                                [--max-width <integer>] [--fit-width] [--print-empty]
                                [--public-key <file> | --private-key <file>]
                                [--type <type>] [--user <user>]
                                [--user-domain <user-domain>]
                                <name>

Create new public or private key for server ssh access

positional arguments:
  <name>        New public or private key name


...

~~~
{: .output}

> ## OpenStack Command Permissions
> There are many commands listed with `openstack help` some of which, as a regular user of the cloud, you have permission to use and others require administrative permissions to use. If you attempt to use a command requiring administration permissions you might see error messages such as these below.
> 
> ~~~
> ForbiddenException: 403: Client Error for url: https://arbutus.cloud.computecanada.ca:####/v#.#/XXXXXX, Policy doesn't allow XXXXXX:XXXXXX to be performed.
> ~~~
> {: .output}
> ~~~
> ForbiddenException: 403: Client Error for url: https://arbutus.cloud.computecanada.ca:####/v#/XXXXXX, You are not authorized to perform the requested action:
> ~~~
> {: .output}
> ~~~
> You are not authorized to perform the requested action: XXXXXX:XXXXXX. (HTTP 403) (Request-ID: req-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX)
> ~~~
> {: .output}
{: .callout}

We can then upload our newly created public key to our OpenStack cloud using the `keypair create` command.
~~~
$ openstack keypair create --public-key ./.ssh/id_ed25519.pub ctc-workshop
~~~
{: .bash}
~~~
+-------------+------------------------------------------------------------------+
| Field       | Value                                                            |
+-------------+------------------------------------------------------------------+
| created_at  | None                                                             |
| fingerprint | ab:ea:87:86:03:61:74:da:7a:89:aa:4e:44:3e:9e:77                  |
| id          | ctc-workshop                                                     |
| is_deleted  | None                                                             |
| name        | ctc-workshop                                                     |
| type        | ssh                                                              |
| user_id     | XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX |
+-------------+------------------------------------------------------------------+
~~~
{: .output}
Public keys are specific to the OpenStack user account and not the OpenStack project.
