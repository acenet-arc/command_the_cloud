---
layout: episode
title: "Creating a virtual machine"
teaching: 20
exercises: 0
questions:
- "How do you create a virtual machine using the OpenStack CLI?"
- "How do you find out which flavors, images, and networks are available?"
- "How do you check on the status of a virtual machine?"
objectives:
- "Create a virtual machine using the OpenStack CLI"
keypoints:
- "OpenStack CLI have commands to list flavors, images, and networks available to a project."
- "The `openstack server create` command can be used to create a virtual machine."
- "The `openstack server list` shows a list of virtual machines and various properties including their statuses."

start: false
---

We will use the `openstack server create` command to create our virtual machine. We will create a virtual machine booting from a volume as we did in the [Introduction to Cloud](https://acenet-arc.github.io/introduction_to_cloud/) workshop except this time using the OpenStack CLI instead of the OpenStack web dashboard.

~~~
openstack server create --flavor <flavor> --image <image> --boot-from-volume <volume-size> --network <network> --key-name <key-name> <vm-name>
~~~
{: .bash}

To use this command we need to following information:
* `<flavor>`: the flavor of the VM we want to create
* `<image>`: the operating system image to copy onto the newly created VMs boot drive
* `<volume-size>`: the size of the boot volume
* `<network>`: which network we want to connect the virtual machine to
* `<key-name>`: the name of the public key to inject into the VM
* `<vm-name>`: the name of the newly created VM. Use your username.

To figure out what these values should be, we can use some OpenStack CLI commands.

Since we are going to be running a series of OpenStack commands to get the above information, to save a little typing, lets start by entering into the OpenStack command prompt.

You can use the `openstack` command in one of two ways. First you can use it to run a single command once, like we have already done with `openstack image list` when we were checking to make sure our authentication worked, or you can run only the `openstack` command by its self and enter multiple OpenStack commands one after the other without having to first type `openstack` which can be useful if you know you want to run a number of different OpenStack commands at once. Lets quickly try it.

~~~
$ openstack
~~~
{: .bash}
~~~
(openstack)
~~~
{: .output}
We now have a new prompt `(openstack)` and we can type OpenStack commands at that prompt without having to type `openstack` first as we did previously.
~~~
(openstack) image list
~~~
{: .bash}
~~~
+--------------------------------------+-----------------------------------+--------+
| ID                                   | Name                              | Status |
+--------------------------------------+-----------------------------------+--------+
| 8fb60bbf-1b73-4339-8bab-7692fccee2cb | AlmaLinux-8.10-x64-2024-05        | active |
...
| cc683663-c2b6-4626-ae6a-f6129cf2f316 | Ubuntu-22.04.4-Jammy-x64-2024-06  | active |
| 241de10b-becc-4d4d-a622-61695e5cb94f | Ubuntu-24.04-Noble-x64-2024-06    | active |
+--------------------------------------+-----------------------------------+--------+
~~~
{: .output}

Now lets see the list of flavors we can pick from.
~~~
(openstack) flavor list
~~~
{: .bash}
~~~
+----------+----------+--------+------+-----------+-------+-----------+
| ID       | Name     |    RAM | Disk | Ephemeral | VCPUs | Is Public |
+----------+----------+--------+------+-----------+-------+-----------+
...
| 31ab2516 | p1-1.5gb |   1536 |   20 |         0 |     1 | False     |
| -c209-   |          |        |      |           |       |           |
| 470c-    |          |        |      |           |       |           |
| 87f1-    |          |        |      |           |       |           |
| e906725f |          |        |      |           |       |           |
| 458b     |          |        |      |           |       |           |
...
+----------+----------+--------+------+-----------+-------+-----------+
~~~
{: .output}
Everything in OpenStack has a unique `ID`. This ID can be used to reference the particular item. Often the name is sufficient and a bit easier than the IDs to keep track of, but the IDs ensure a unique reference to the item if there is some ambiguity when using the name.

For our flavor lets use the small `p1-1.5gb` flavor.

Now lets see what images we have available.
~~~
(openstack) image list
~~~
{: .bash}
~~~
+--------------------------------------+-----------------------------------+--------+
| ID                                   | Name                              | Status |
+--------------------------------------+-----------------------------------+--------+
...
| 241de10b-becc-4d4d-a622-61695e5cb94f | Ubuntu-24.04-Noble-x64-2024-06    | active |
...
+--------------------------------------+-----------------------------------+--------+
~~~
{: .output}
Lets use the `Ubuntu-24.04-Noble-x64-2024-06` image.

Now lets see what networks we have available.

~~~
(openstack) network list
~~~
{: .bash}
~~~
+-------------------------+-----------------+--------------------------+
| ID                      | Name            | Subnets                  |
+-------------------------+-----------------+--------------------------+
| XXXXXXXX-XXXX-XXXX-     | XXXXXXX-private | XXXXXXXX-XXXX-XXXX-XXXX- |
| XXXX-XXXXXXXXXXXX       |                 | XXXXXXXXXXXX             |
...
| 6621bf61-6094-4b24-     | Public-Network  | 1f6a472c-c1bf-42a4-9473- |
| a9a0-f5794c3a881e       |                 | 7ec1a543f0a8, 420e1bf0-  |
|                         |                 | 575c-4498-9bee-          |
|                         |                 | 85e24ce8c1fc             |
...
+-------------------------+-----------------+--------------------------+
~~~
{: .output}
We want to connect the VM to our local private network, so pick the network with the suffix `-private`. However, in order to access our VM from outside the private network we will need to create and allocated a floating IP form the `Public-Network`, so keep that network in mind too.

Finally lets double check our public key name.
~~~
(openstack) keypair list
~~~
{: .bash}
~~~
+----------------------+-------------------------------------------------+------+
| Name                 | Fingerprint                                     | Type |
+----------------------+-------------------------------------------------+------+
...
| ctc-workshop         | ab:ea:87:86:03:61:74:da:7a:89:aa:4e:44:3e:9e:77 | ssh  |
...
+----------------------+-------------------------------------------------+------+
~~~
{: .output}
Lets use the public key we just created in the previous episode, `ctc-workshop`.

For our VM name we will use our username to uniquely identify our virtual machine form those created by other students and we will give it a 20 GB volume to boot from. Putting all this information together we end up with the following command.

~~~
(openstack) server create --flavor 'p1-1.5gb' --image 'Ubuntu-24.04-Noble-x64-2024-06' --boot-from-volume 20 --network 'XXXXXXX-private' --key-name 'ctc-workshop' <your-user-name>
~~~
{: .bash}
Remember to replace `<your-user-name>` and `XXXXXXX-private`.

To see a list of virtual machines in the cloud project use the `server list` command.
~~~
(openstack) server list
~~~
{: .bash}
~~~
+-----------+-----------+--------+-----------+-----------+-------------+
| ID        | Name      | Status | Networks  | Image     | Flavor      |
+-----------+-----------+--------+-----------+-----------+-------------+
| 54eed12e- | user03    | ACTIVE | XXXXXXX-p | N/A       | p1-1.5gb    |
| fa7b-     |           |        | rivate=19 | (booted   |             |
| 4a6c-     |           |        | 2.168.0.1 | from      |             |
| 9b59-     |           |        | 22        | volume)   |             |
| ce05b75b7 |           |        |           |           |             |
| 39b       |           |        |           |           |             |
...
+-----------+-----------+--------+-----------+-----------+-------------+
~~~
{: .output}
Find the virtual machine with the same name as your username. This shows us the `ID` of our newly created virtual machine as well as the `Status` of the virtual machine.

In order to connect to the virtual machine we need to add a public IP, also called a floating IP, to the VM. First we need to allocate or create a new floating IP in the project that we can use.
~~~
(openstack) floating ip create Public-Network
~~~
{: .bash}
~~~
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| created_at          | ####-##-##T##:##:##Z                 |
| description         |                                      |
| dns_domain          |                                      |
| dns_name            |                                      |
| fixed_ip_address    | None                                 |
| floating_ip_address | ###.###.###.###                      |
| floating_network_id | XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX |
| id                  | XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX |
| name                | ###.###.###.###                      |
| port_details        | None                                 |
| port_id             | None                                 |
| project_id          | XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX     |
| qos_policy_id       | None                                 |
| revision_number     | 0                                    |
| router_id           | None                                 |
| status              | DOWN                                 |
| subnet_id           | None                                 |
| tags                | []                                   |
| updated_at          | ####-##-##T##:##:##Z                 |
+---------------------+--------------------------------------+
~~~
{: .output}
Then add the IP to our virtual machine. Remember your virtual machine was named after your user name.
~~~
(openstack) server add floating ip <your-user-name> ###.###.###.###
~~~
{: .bash}

> ## Add a security group rule?
> The virtual machine that we have been using to run the OpenStack CLI has already had its IP address added to the default security group in our cloud project. This means we can ssh from the VM we have been working on into our newly created VM without issue. If however, you need to add a new security group rule to allow ssh access from a different VM you could do the following.
> 
> First determine the public IP of the VM you are currently working from.
> ~~~
> $ curl ipv4.icanhazip.com
> ~~~
> {: .bash}
> ~~~
> 123.123.123.123
> ~~~
> {: .output}
> Then add a rule to the cloud projects default security group allowing ingress traffic using the TCP protocol on port 22 (the port used for ssh).
> ~~~
> $ openstack security group rule create --remote-ip 206.12.90.32 --ingress --protocol TCP --dst-port 22 --description 'ctc VM' default
> ~~~
> {: .bash}
>
> I always find a description on a security group rule handy, so later I know if I can delete the rule or not.
{: .callout}

Now we are all set to connect to our newly created VM. But first lets exit the OpenStack prompt.
~~~
(openstack) exit
~~~
{: .bash}
Now connect to our new virtual machine.
~~~
$ ssh ubuntu@###.###.###.###
~~~
{: .bash}
With Ubuntu images the default username is `ubuntu` and `###.###.###.###` is the floating IP we added to our VM with the `server add floating ip` above.
~~~
Enter passphrase for key '/home/user03/.ssh/id_ed25519':
~~~
{: .output}
Then enter your passphrase for the ssh keypair created in the last episode.
~~~
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.8.0-35-generic x86_64)

...

ubuntu@<your-user-name>:~$
~~~
{: .output}

Now lets exit our newly created VM and return to the VM where running our OpenStack CLI commands.
~~~
$ exit
~~~
{: .bash}

