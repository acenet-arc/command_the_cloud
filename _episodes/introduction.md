---
layout: episode
title: "Introduction"
teaching: 15
exercises: 0
questions:
- "What is the OpenStack CLI?"
- "Why use the OpenStack CLI?"
- "Are their things the OpenStack CLI can do that the Web Dashboard can't?"
objectives:
- ""
keypoints:
- "[**Cloud computing**](../reference#cloud-computing) is very flexible and has many diverse uses."
- "Using the OpenStack dashboard is an easy way to perform many common cloud related tasks, however it doesn't provide access to all the functionality of the cloud."
- "The OpenStack CLI can be used to run commands to obtain information and perform tasks similar to the OpenStack dashboard."
- "The OpenStack CLI can also provide information and perform tasks not available through the OpenStack dashboard."
start: true
start_time: 780
---

<!--
> ## Office hours
> 
> | Day of the week | Date | Time (ADT) |
> | :-- | :-- | :-- |
> | Friday    | TBD | 1:00 - 3:00pm |
> | Friday    | TBD | 1:00 - 3:00pm |
{: .callout}
-->

Cloud computing offers a wide range of usage possibilities. From running basic HTML sites for publishing works, to collaborative wikis, to running persistent web scrapers, automating data collection and processing, to providing platforms to help support whole research communities. Possible use cases are varied and wide ranging. One particular example using the Alliance cloud is [Voyant Tools](https://voyant-tools.org/), a web-based text reading and analysis environment. It is a scholarly project that is designed to facilitate reading and interpretive practises for digital humanities students and scholars as well as for the general public. In order to make good use of the cloud you need to know how to interact with it to do the things you need to do. In this workshop we are going to introduce a new way of interacting with OpenStack clouds, the OpenStack Command Line Interface (CLI).

## Overview

In this workshop we will ... ***TODO***


## What is the OpenStack CLI?

If you have been using the OpenStack dashboard either in our [Introduction to Cloud](https://acenet-arc.github.io/introduction_to_cloud/) course or from your own experience working with the Alliance or similar OpenStack based clouds you should have some idea of the sorts of things you can do on the OpenStack Horizon (web) dashboard, such as:

* creating virtual machines booting from a volume,
* editing security groups, and
* checking project quotas.

The dashboard is an easy way to do many common tasks you need to perform when working with the cloud. However, it is not the only way to interact with your cloud project. Another very common way to interact with your cloud project is using the OpenStack CLI. This is a command line tool that allows you to run commands in a local (or remote) terminal and interact with your remote cloud projects on different OpenStack based clouds. To do this you will need to install the tools and setup the necessary credentials to authenticate with the particular remote cloud project you want to work with. We will get to that in the next episode.

## Why use the OpenStack CLI?

While the OpenStack dashboard is easy to perform many common OpenStack cloud tasks, there are some limitations. Not all possible cloud functionality is exposed to the dashboard. Instead some tasks and information can only be performed or obtained using the OpenStack CLI. For example, with the `openstack quota show --usage` command used for a project on Arbutus cloud, you can see what your storage quota for 'OS or Database' volumes is and how much you have used. On the web dashboard you only see the total volume storage limits and usage, which is not the important limit when creating specialized types of volumes. Then when you go to create an "OS or Database" volume you will know what the maximum size of that volume type you can create is. Otherwise in the dashboard it might look like you are able to create a 200GB 'OS and Database' volume for example, but when you click "create volume" you get the message "Error: Unable to create volume." and you have no idea why.

Here is some example output of the `openstack quota show --usage` command.
~~~
$ openstack quota show --usage
~~~
{: .bash}
~~~
+---------------------------+---------+--------+----------+
| Resource                  |   Limit | In Use | Reserved |
+---------------------------+---------+--------+----------+
...
| gigabytes                 |    1000 |    340 |        0 |
...
| gigabytes_OS or Database  |     100 |     20 |        0 |
...
+---------------------------+---------+--------+----------+
~~~
{: .output}
Here you can see that the total volume storage limit is 1000 GB and 340 GB of that is used, while the limit for 'OS or Database' volumes is 100 GB and 20 GB are used. A similar situation exists on Beluga cloud between "EC" volumes ([erasure encoded](https://en.wikipedia.org/wiki/Erasure_code) and backed by HDD) versus, "SSD" backed by solid state drives. Some other tasks that require use of the OpenStack CLI are: cloning a volume, downloading an image, and creating object store credentials. In addition you can perform the same tasks you perform in the web dashboard using the CLI. For example creating new virtual machines, volumes, images, security groups, and security group rules.

The OpenStack CLI also makes it easier to automate common tasks by placing commands into shell scripts. There are other ways to automate common OpenStack tasks such as using the [OpenStack Python SDK](https://docs.openstack.org/mitaka/user-guide/sdk.html) to write Python scripts, [OpenStack Orchestration](https://docs.openstack.org/heat/2023.1/index.html) using [template files](https://docs.openstack.org/heat/2023.1/template_guide/index.html), or [Terraform](https://www.terraform.io/). While these other methods of automation provide many different benefits over using OpenStack CLI commands in a shell script they do have significantly larger learning curves. These methods of automation are beyond the scope of this workshop.


