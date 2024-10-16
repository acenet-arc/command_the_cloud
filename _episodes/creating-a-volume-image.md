---
layout: episode
title: "Creating a volume image"
teaching: 20
exercises: 0
questions:
- "What is a volume snapshot?"
- "What is a volume image?"
- "What happens when I create a snapshot of a virtual machine?"
objectives:
- "Create a volume image from the boot volume of a virtual machine."
keypoints:
- ""

start: false
---

One common task, beyond creating a virtual machine in the first place, is creating an image or a snapshot of a virtual machine or volume. Images and snapshots are different things but it can be tricky to keep them straight. You can create snapshots and images from volumes, but you can also create a snapshot of a virtual machine. When you create a snapshot of a virtual machine different things can happen under the hood depending on the type of virtual machine it is. Lets start by talking about what happens when you create a snapshot or an image of a volume and then describe what happens when you create a snapshot of a virtual machine.

## Volume snapshot
A volume **snapshot** is a copy of a volume from a point in time. It can be used to create new volumes. When you create a snapshot of a volume, that snapshot then depends on that volume so that you can't delete the volume without first deleting any snapshots created from it. On some Alliance clouds, for example Arbutus, the newly created volumes also depend on the snapshot it was created from. This means there is a dependency from the newly created volume to the snapshot it was created from back to the original volume, such that the original volume can not be deleted until the newly created volume and all the snapshots have been deleted. On Beluga cloud, this dependency is broken when the new volume is created so that the original volume can be deleted after all the snapshots of it are also deleted but retain the newly created volume. Creating snapshots is fairly fast and light weight, however, the dependencies between volumes and snapshots can be problematic depending on your use case.

## Volume image
Like a volume snapshot a volume image is also copy of a volume. However, unlike volume snapshots, it is a completely separate copy with no dependency on the original volume. This means that you can delete the original volume as soon as you make an image of it. Images can also be downloaded to your local machine and uploaded to other clouds. However, when uploading images to other clouds you are not always guaranteed that the image will work similarly as there maybe difference in the way clouds are configured. I have been successful in moving images between Alliance OpenStack clouds in the past, however, it should not be assumed that an image is directly transferable between all clouds. While it maybe transferable to a particular cloud it may not to another, it is always best to test first before relying on it. Images can also be used to start a virtual machine on your local machine using something like [virtual box](https://www.virtualbox.org/), though you may need to ensure you are creating an image with a compatible format such as [vmdk](https://en.wikipedia.org/wiki/VMDK).

## Virtual machine snapshot
There are two main types of virtual machines, persistent virtual machines and compute virtual machines. Persistent virtual machines are what we have created so far and boot from a volume. Compute virtual machines do not boot from a volume and instead have an ephemeral disk that resides on the local hypervisor (server that runs virtual machines). Compute virtual machines also have an extra ephemeral disk which also resides on the local hypervisor. What happens when you create a virtual machine snapshot differs depending on which of these two cases applies.

### Creating a snapshot of a persistent virtual machine
In this case a volume snapshot is made of the boot disk and a zero size image is created. The volume snapshot that is made works just like the volume snapshots described above. The image on the other hand is very different. In this case the image just points to the volume snapshot. It can't be downloaded and used in virtualbox or uploaded to another cloud.

### Creating a snapshot of a compute virtual machine
In the case of a compute virtual machine, there is no volume to create a snapshot of. So in this case it instead creates an image. This image behaves similarly to the volume image described above. In this case the extra ephemeral disk local to the hypervisor is not captured only the ephemeral boot disk.

> ## Confusing CLI command name
> There is an OpenStack CLI command `openstack server image create <server-name>` however, this does exactly what the GUI Dashboard does when creating a snapshot of a virtual machine. Meaning it creates a volume snapshot of the boot disk if the VM boots from a volume, or if it has an ephemeral disk (e.g. a compute virtual machine) it creates an image of the ephemeral boot disk. Both the GUI Dashboard "create snapshot" instance action and `server create image` can create either a volume snapshot or an image depending on the nature of the virtual machine.
{: .callout}

## Shutdown virtual machine
Whenever creating a snapshot of a virtual machine or a snapshot or image of a volume it is important to shutdown the virtual machine first. If the virtual machine is running while the snapshot or image is created it is possible that it could be writing something to the disk at the time the snapshot or image are created. This can result in a corrupted and unusable image or snapshot. Lets shutdown the virtual machine in the dashboard. Go to "Project" -> "Compute" -> "Instances" then find the row with the virtual machine with the same name as your username. In the far right hand column of this row under the "Actions" column in the drop down menu select "Shut Off Instance".

> ## Create an image from the Dashboard
> Go to "Project" -> "Volumes" -> "Volumes". Then in the "Attached To" column find the name of your virtual machine. Then in the far right column "Actions" for the row for that volume select "Upload to Image". Try with or without "Force" checked. What happens?
> > ## Solution
> > `Error: Unable to upload volume to image`. This error message occurs regardless of weather "Force" is checked or not.
> {: .solution}
{: .challenge}

The reason this is happening is because it is the volume that the virtual machine boots from, though it doesn't tell you this in the error message unfortunately. The volume must be detached form the virtual machine before you can create an image of the volume. However, you can not just detach a boot volume form a virtual machine, instead you would need to delete the virtual machine. Unless otherwise specified, volumes will persist beyond a particular virtual machine and can be used to create a new virtual machine booting form the same volume retaining the data on the volume.

> ## Delete on terminate
> When creating a virtual machine it is possible to specify that the volume should be deleted when the virtual machine is also deleted. It is possible to check if this is the case for a particular virtual machine using the OpenStack CLI.
> ~~~
> $ openstack server show <server-name-or-id>
> ~~~
> {: .bash}
> ~~~
> +--------------------------+---------------------------+
> | Field                    | Value                     |
> +--------------------------+---------------------------+
> ...
> | volumes_attached         | delete_on_termination='Fa |
> |                          | lse', id='9fbae2de-0e45-  |
> |                          | 4fb0-b2c2-806d1da8135e'   |
> +--------------------------+---------------------------+
> ~~~
> {: .output}
> Notice at the bottom of the output under the `volumes_attached` field that is says `delete_on_terminatoin='False'` indicating that it will not delete the attached volume when the virtual machine is deleted.
{: .callout}

## Cloning a volume
Deleting a virtual machine to create an image of the volume isn't the most desirable way to create an image of a volume. Another way to do this is to **clone** the volume, which you can only do using the OpenStack CLI, and then create an image from this cloned volume. This does mean that you need to have the storage quota available for the additional cloned volume but allows you to avoid having to delete your virtual machine first.

To create a clone of a volume we need to know the volume's ID and its size so that the newly created volume is the same size as the original. Lets re-enter the OpenStack shell.

~~~
$ openstack
~~~
{: .bash}
~~~
(openstack) 
~~~
{: .output}
Now lets list the volumes in our project
~~~
(openstack) volume list
~~~
{: .bash}
~~~
+--------------+--------------+------------+------+----------------+
| ID           | Name         | Status     | Size | Attached to    |
+--------------+--------------+------------+------+----------------+
...
| 0e493f24-    |              | in-use     |   20 | Attached to    |
| 3b0b-4c76-   |              |            |      | user03 on      |
| 94c3-        |              |            |      | /dev/vda       |
| a7e6211628bc |              |            |      |                |
...
+--------------+--------------+------------+------+----------------+
~~~
{: .output}
Look through the list of volume untel you find the one that says `Attached to <your-vm-name> on /dev/vda'. This provides the volume ID in the "ID" column that we need to know as well as the size of the volume in the "Size" column.

> ## Copying the volume ID
> It can be tricky to copy the volume ID when it is split over multiple lines. If possible make your terminal wider so you can get the ID all on one line to make copying it easier. Then re-run your `volume list` command and it will make use of the extra width.
{: .callout}

Now we can create the clone of the volume.
~~~
(openstack) volume create --source 0e493f24-3b0b-4c76-94c3-a7e6211628bc --size 20 user03-volume-clone
~~~
{: .bash}
~~~
+---------------------+----------------------------------------+
| Field               | Value                                  |
+---------------------+----------------------------------------+
| attachments         | []                                     |
| availability_zone   | nova                                   |
| bootable            | true                                   |
| consistencygroup_id | None                                   |
| created_at          | 2024-10-16T14:48:12.000000             |
| description         | None                                   |
| encrypted           | False                                  |
| id                  | e5accde4-0ecc-4eba-b2cd-b15ba66b0bd3   |
| multiattach         | False                                  |
| name                | user03-volume-clone                    |
| properties          |                                        |
| replication_status  | None                                   |
| size                | 20                                     |
| snapshot_id         | None                                   |
| source_volid        | 0e493f24-3b0b-4c76-94c3-a7e6211628bc   |
| status              | creating                               |
| type                | Default                                |
| updated_at          | None                                   |
| user_id             | XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX |
+---------------------+----------------------------------------+
~~~
{: .output}

Lets check for our newly created volume.
~~~
(openstack) volume list
~~~
{: .bash}
~~~
+----------+----------+----------+------+-------------+
| ID       | Name     | Status   | Size | Attached to |
+----------+----------+----------+------+-------------+
| e5accde4 | user03-  | availabl |   20 |             |
| -0ecc-   | volume-  | e        |      |             |
| 4eba-    | clone    |          |      |             |
| b2cd-    |          |          |      |             |
| b15ba66b |          |          |      |             |
| 0bd3     |          |          |      |             |
...
+----------+----------+----------+------+-------------+
~~~
{: .output}
Notice how it isn't attached to any VMs, this means it isn't a boot volume for any VMs and we can create an image of it and it contains all the data that the original volume we cloned it from does.

## Creating an image of a volume

~~~
(openstack) image create --disk-format qcow2 --volume user03-volume-clone user03-image
~~~
{: .bash}
~~~
+---------------------+-----------------------------------+
| Field               | Value                             |
+---------------------+-----------------------------------+
| container_format    | bare                              |
| disk_format         | qcow2                             |
| display_description | None                              |
| id                  | e5accde4-0ecc-4eba-b2cd-          |
|                     | b15ba66b0bd3                      |
| image_id            | 40905fd2-3f8c-4fb4-a2b3-          |
|                     | 6707bffe2bb5                      |
| image_name          | user03-image                      |
| protected           | False                             |
| size                | 20                                |
| status              | uploading                         |
| updated_at          | 2024-10-16T14:48:14.000000        |
| visibility          | shared                            |
| volume_type         | Default                           |
+---------------------+-----------------------------------+
~~~
{: .output}
Here we choose the `qcow2` volume format, this image format allows the image to be smaller than for example the `raw` format which copies all the bytes of the volume 1:1 into the image. `qcow2` is the standard image format used on the Arbutus cloud, while the `raw` image format is typically used on the Beluga cloud. The `raw` format can be faster when creating new VMs as there are fewer if any format conversions that need to happen when new volumes are created form the image, however, that comes at the cost of greater image sizes.  As an example, on Arbutus the 'Rocky-9.1-x64-2023-02' image is 885.94MB, while the same image on Beluga is 10GB. Similarly the 'Ubuntu-24.04-Noble-x64-2024-06' image is 556.38MB on Arbutus but is 3.5GB on Beluga.

I have also found the `vmdk` format useful if I want to download the image and use it with [virtual box](https://www.virtualbox.org/), though other formats may also work. Like the `qcow2` image format `vmdk` also does not copy all the bytes of the volume so again will be smaller than then `raw` format which is helpful if you are copying it across a network. Virtual box is a program you can use to run virtual machines locally on your laptop or desktop computer.

## Moving images

It is possible to download and upload images. In a pinch it is possible to use images like a backup, but it is not the ideal way to do it. We will talk more about backups in the next episode. Since even small image files are likely a few GB in size we won't actually run the commands to download and upload images but just show you what they look like and talk about them briefly.

To download an image you use the `image save` command with the `--file` option.
~~~
openstack image save --file ./user03-image.qcow2 40905fd2-3f8c-4fb4-a2b3-6707bffe2bb5
~~~
{: .bash}
The `.qcow2` extension on the file name isn't reqiured but is a nice way to remind you of the image format that the file contains.

To upload an image file you would use the `image create` command with the `--file` option.
~~~
openstack image create --file ./user03-image.qcow2 --disk-foramt qcow2 new-user03-image
~~~
{: .bash}
Note that the `--disk-format` provided must match the actual format of the image file you upload, if not your image will not work correctly. This the main reason I specify the `.qcow2` extension on my image file so that I can remember what format the image file is.
