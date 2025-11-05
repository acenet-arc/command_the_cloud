---
layout: page
title: "Instructor Notes"
permalink: /guide/
---

To create a VM for students to use for installing and running OpenStack CLI use `cloud_init_guest_accounts.yml` file in the [cloud-init-files](https://github.com/acenet-arc/command_the_cloud/tree/main/cloud-init-files) folder when creating the VM to perform setup. Copy and paste the contents of that file into "Customization Script" under the "Configuration" tab in the popup while creating a VM in OpenStack. Under the `users` section add the name for an admin user and their public key. Multiple students can share a VM as setup will happen in python virtual environments in users home directory. They also don't need admin access. A large flavor like a `p16-24gb` flavor should be good for around 30 users. Each user will only need enough space for their Python virtual environment with the OpenStack CLI installed.

Once setup has completed have a look at the VM log to find the login password for guest accounts. Search for "Guest accounts passphrase:".

While working with OpenStack using the CLI, students will create a small persistent VM booting from a 20GB volume, clone that volume, and create an image of it. There needs to be enough storage quota for each student to have two 20 GB volumes and enough quota for one small virtual machine. Also the last episode makes use of OpenStack object store to backup a small text file, so Object storage is also required.
