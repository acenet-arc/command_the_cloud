---
layout: episode
title: "Installing CLI"
teaching: 20
exercises: 0
questions:
- "How do you install the OpenStack CLI?"
- "How do you authenticate with an OpenStack cloud?"
objectives:
- "Install the OpenStack CLI"
- "Configure authentication for the OpenStack CLI"
- "Run an OpenStack CLI command to verify authentication works"
keypoints:
- Install the OpenStack CLI using the Python package tool pip
- It is recommended to install the OpenStack CLI into a Python virtual environment.
- Use the `openstack-rc.sh` file to easily and consistently set the needed environment variables.
start: false
---

OpenStack's Command Line Interface (CLI) is written using the scripting language [Python](https://www.python.org/) and under the hood uses the [Openstack Python SDK](https://docs.openstack.org/mitaka/user-guide/sdk.html), which allows you to write OpenStack automation scripts using Python.

It can be difficult to install the OpenStack CLI on some operating systems, in particular I have had difficulties installing them on Windows machines. That isn't to say you can't, but it may not be as straight forward. However, since we have access to an OpenStack cloud we can instead install the OpenStack CLI on a virtual machine running the linux operating system Ubuntu and use the tool from there. This also means we are all working in the same environment removing many possible variations that might exist between everyone's different laptops or desktops.

I have already setup virtual machines which we can ssh into and use for this workshop using the provided usernames and passwords. If you haven't already, open up a terminal either by starting a local terminal on MobaXterm on Windows or opening up your operating systems built in terminal as described on our [setup page](/setup).

~~~
$ ssh <username>@<ip>
~~~
{: .bash}
~~~
<username>@<ip>'s password:
~~~
{: .output}
At the `password:` prompt enter the password you have been given. **The password characters you type will not be shown** and it will look like nothing is happening as you enter the password characters. Not displaying the password characters is a security feature.

Generally password authentication is a bad idea unless you take special precautions such as running [fail2ban](https://github.com/fail2ban/fail2ban/wiki) to block brute force attacks. A better method is to use ssh key pairs as described in our [Introduction to Cloud](https://acenet-arc.github.io/introduction_to_cloud/creating-a-keypair/) workshop.

> ## On your own fresh Ubuntu VM?
> 
> The below packages have already been installed on the course VMs for us. However, if you are using a VM you setup yourself, you will need to install the below packages to gain access to both `pip3` and `virtualenv`. At least this is the case in the most recent Ubuntu 24.04 version of the operating system.
> 
> ~~~
> sudo apt install python3-pip python3-virtualenv
> ~~~
> {: .bash}
{: .callout}

# Installing OpenStack CLI

We will install the OpenStack CLI into a Python virtual environment. Virtual environments allow you to isolate multiple python environments from each other to avoid different versions of the same packages clashing with each other. It also makes it easy to remove no longer needed python modules and applications by just removing the virtual environment folder. Lets create a new virtual environment into which we will install the OpenStack CLI. 

~~~
$ virtualenv pyenvopst
~~~
{: .bash}
~~~
created virtual environment CPython3.12.3.final.0-64 in 8881ms
...
~~~
{: .output}
~~~
$ ls
~~~
{: .bash}
~~~
pyenvopst
~~~
{: .output}
This created a new folder called `pyenvopst` where the virtual environment is stored.

Lets activate the environment.

~~~
$ source ./pyenvopst/bin/activate
~~~
{: .bash}

Finally lets use the Python package installer [pip](https://pypi.org/project/pip/) to install the OpenStack CLI into our virtual environment.

~~~
(pyenvopst) $ pip3 install python-openstackclient
~~~
{: .bash}

> ## Exiting a virtual environment?
> 
> Once a virtual environment has been activated it can be deactivated with the below command.
> ~~~
> (pyenvopst) $ deactivate
> ~~~
> {: .bash}
{: .callout}

# Configuring Authentication

In order to authenticate a number of environment variables need to be set in your shell which the OpenStack CLI will reference to authenticate you with your OpenStack project. These are things like the project ID, the URL to the cloud's authentication service, project name and so on. To avoid needing to know all the settings for a particular cloud and project you can download a file called an "OpenStack RC" file which contains most of the settings needed to authenticate, with the exception of your password. Then when you source this file, as we did when activating our python virtual environment above, it will ask you for your OpenStack cloud password which you can type in. This password gets saved in an environment variable that is stored for the lifetime of your shell. If your shell is on a compromised machine, it maybe possible for someone to gain access to your shell environment variables and thus your password. There for it is recommended to use the command line clients on a well secured non-publicly shared machine.

To find this "OpenStack RC" file log into the OpenStack dashboard, navigate to the particular project you want to use the OpenStack CLI with (drop down in top left), and go to "Project" -> "API Access" and click "Download OpenStack RC File". This will save the file on your local machine where other items you would normaly download from the Internet go, for example a "Downloads" folder, or perhaps your "Desktop" depending on where you configured your web browser to save files. Open this file in a plain text editor such as notepad on Windows or TextEdit on Mac select all and copy the contents of this file. Then back on our VM create a new file using the nano text editor and paste in the contents of the downloaded file.

~~~
$ nano openstack-rc.sh
~~~
{: .bash}
~~~
#!/usr/bin/env bash
# To use an OpenStack cloud you need to authenticate against the Identity
# service named keystone, which returns a **Token** and **Service Catalog**.
# The catalog contains the endpoints for all services the user/tenant has
# access to - such as Compute, Image Service, Identity, Object Storage, Block
# Storage, and Networking (code-named nova, glance, keystone, swift,
# cinder, and neutron).
#
# *NOTE*: Using the 3 *Identity API* does not necessarily mean any other
# OpenStack API is version 3. For example, your cloud provider may implement
# Image API v1.1, Block Storage API v2, and Compute API v2.0. OS_AUTH_URL is
# only for the Identity API served through keystone.
export OS_AUTH_URL=https://arbutus.cloud.computecanada.ca:5000
# With the addition of Keystone we have standardized on the term **project**
# as the entity that owns the resources.
export OS_PROJECT_ID=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
export OS_PROJECT_NAME="project-name"
export OS_USER_DOMAIN_NAME="domain"
if [ -z "$OS_USER_DOMAIN_NAME" ]; then unset OS_USER_DOMAIN_NAME; fi
export OS_PROJECT_DOMAIN_ID="XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
if [ -z "$OS_PROJECT_DOMAIN_ID" ]; then unset OS_PROJECT_DOMAIN_ID; fi
# unset v2.0 items in case set
unset OS_TENANT_ID
unset OS_TENANT_NAME
# In addition to the owning entity (tenant), OpenStack stores the entity
# performing the action as the **user**.
export OS_USERNAME="your-user-name"
# With Keystone you pass the keystone password.
echo "Please enter your OpenStack Password for project $OS_PROJECT_NAME as user $OS_USERNAME: "
read -sr OS_PASSWORD_INPUT
export OS_PASSWORD=$OS_PASSWORD_INPUT
# If your configuration has multiple regions, we set that information here.
# OS_REGION_NAME is optional and only valid in certain environments.
export OS_REGION_NAME="RegionOne"
# Don't leave a blank variable, unset it if it was empty
if [ -z "$OS_REGION_NAME" ]; then unset OS_REGION_NAME; fi
export OS_INTERFACE=public
export OS_IDENTITY_API_VERSION=3
~~~
{: .output}
Then save and exit nano by pressing `ctrl`+`x`, answering `Y` when asked if you want to 'Save modified buffer?', and pressing your return key to accept the file name.

Note that I have replaced some strings with 'X' and other descriptive strings, rather than leave the actual contents of those strings for security reasons.

Then to set those environment variables source the OpenStack RC file.
~~~
$ source ./openstack-rc.sh
Please enter your OpenStack Password for project project-name as user your-user-name:
~~~
{: .bash}
And enter the password you use to log into the OpenStack dashboard which matches the username specified in your rc file. Once this is complete issue the following command to verify you can indeed authenticate with the project.
~~~
$ openstack image list
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

If everything worked you should see a list of images you can use to create VMs from.

> ## Old OpenStack CLI
> You may or may not know, that OpenStack is actually composed of a number of different components or services. For example the **Nova** service manages computes services like virtual machine creation, services like **Cinder** manage block storage or volumes and so on. See this list of [openstack services](https://www.openstack.org/software/project-navigator/openstack-components#openstack-services) for a more complete list of OpenStack services not all of which will appear on all OpenStack clouds.
>
> Originally the OpenStack command line interface tools had a separate tool for each OpenStack component, however more recently they have been moving toward using the `openstack` command to perform tasks across all the different OpenStack services, which is what we are focusing on in this workshop. However, not all functionality is available even under the new `openstack` CLI and in some special cases it maybe helpful to use the older separate CLI tools. For example to work with OpenStack volumes you would use the `cider` tool.
> ~~~
> $ cinder list
> ~~~
> {: .bash}
> ~~~
+--------------------------------------+------------+------+------+----------------+----------+--------------------------------------+
| ID                                   | Status     | Name | Size | Volume Type    | Bootable | Attached to                          |
+--------------------------------------+------------+------+------+----------------+----------+--------------------------------------+
| XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX | available  | test | 20   | OS or Database | false    |                                      |
| XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX | in-use     |      | 20   | Default        | true     | XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX |
+--------------------------------------+------------+------+------+----------------+----------+--------------------------------------+
> ~~~
> {: .output}
> The `cinder` command happens to already be installed with the openstack tool we have already installed. Other command tools such as `nova` might need to be installed separately.
{: .callout}
