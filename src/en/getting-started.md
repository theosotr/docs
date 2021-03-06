Title: Getting started with Juju 2.0
TODO: remove ppa/devel after release
      simplify ZFS install


# Getting started with Juju 2.0

The instructions here will get you up and running and deliver the best-possible
experience. At the moment, that means using the very latest release of 
Ubuntu, [16.04LTS (Xenial)](http://www.ubuntu.com/download/).

If you are using a different OS, please read the 
[general install instructions here](./getting-started-general.html).

To get the best experience, as well as Juju, this guide will also set up:
   
- LXD - a hypervisor for LXC, providing fast, secure containers.
- ZFS - a combined filesystem/LVM which gives great performance.

Both the above are provided with Ubuntu 16.04LTS.


## Install the software

Run the following commands to install the required software:

```bash
sudo add-apt-repository ppa:juju/devel
sudo apt update
sudo apt install juju zfsutils-linux lxd
newgrp -
```

!!! Note: The `newgrp` command should only be necessary if package 'lxd' got
installed (it may be already installed). This command will update your group
memberships but will also start a new shell.


## Prepare ZFS

Using ZFS we can create sparse backing-storage for any of the containers which
LXD creates for Juju. You can create this storage anywhere (e.g. the fastest
drive you have). This example creates a 32G file to use:

```bash
sudo mkdir /var/lib/zfs
sudo truncate -s 32G /var/lib/zfs/lxd.img
sudo zpool create lxd /var/lib/zfs/lxd.img
```

As this is sparse storage, it won't actually take up disk space until it is 
actually being used. You can check the file has been added to a ZFS pool with 
the following command:
  
```bash
sudo zpool iostat -v
```


## Initialise LXD

Now we need to tell LXD about this storage:

```bash
sudo lxd init --auto --storage-backend zfs --storage-pool lxd
```


## Create a controller

Juju needs a controller instance to manage your models and the `juju bootstrap`
command is used to create one.

However, if this is the first time using LXD then you'll need to configure it
and expose a "network bridge" that Juju requires:

```bash
sudo dpkg-reconfigure -p medium lxd
```

!!! Note: During the configuration dialog, the bridge must be 'lxdbr0'. You
can accept all other prompts although the last question (IPv6) is probably not
needed.

Proceed with the creation of the controller. For use with our LXD "cloud", we
will make a controller called 'lxd-test':

```bash
juju bootstrap lxd-test lxd
```

This may take a few minutes as LXD must download an image for Xenial. A cache
will be used for subsequent containers.

Once the process has completed you can check that the controller has been
created:

```bash
juju list-controllers 
```

This will return a list of the controllers known to Juju, which at the moment is
the one we just created:
  
```no-highlight
CONTROLLER       MODEL    USER         SERVER
local.lxd-test*  default  admin@local  10.0.3.124:17070
```

Notice that the prefix 'local.' is added to the controller name we specified.

Confirm ZFS is working by looking for changes to the storage pool:
  
```bash
sudo zpool iostat -v
```

Newly-created controllers come bundled with two models: The 'admin' model,
which should be used only by Juju for internal management, and a 'default'
model, which is ready for actual use.

The following command shows the currently active controller and model:

```bash 
juju switch
```

In our example, the output should look like this:

```no-highlight
local.lxd-test:default
```

The format is 'controller:model'.


## Deploy

Juju is now ready to deploy any services from the hundreds included in the
[juju charm store](https://jujucharms.com). It is a good idea to test your new 
model. How about a Mediawiki site?

```bash
juju deploy mediawiki-single
```
This will fetch a 'bundle' from the Juju store. A bundle is a pre-packaged set
of services, in this case the 'Mediawiki' service, and a database to run it 
with. Juju will install both these services and add a relation between them - 
this is part of the magic of Juju: it isn't just about deploying services, Juju 
also knows how to connect them together.

Installing shouldn't take long. You can check on how far Juju has got by running
the command:
 
```bash
juju status
```
When the services have been installed the output to the above command will look
something like this:

![juju status](./media/juju-mediawiki-status.png)

There is quite a lot of information there but the important parts for now are 
the [Services] section, which show that Mediawiki and MySQL are installed, and
the [Units] section, which crucially shows the IP addresses allocated to them.

By default, Juju is secure - you won't be able to connect to any services 
unless they are specifically exposed. This adjusts the relevant firewall 
controls (on any cloud, not just LXD) to allow external access. To make
our Mediawiki visible, we run the command:

```bash
juju expose mediawiki
```

From the status output, we can see that the Mediawiki service is running on 
10.0.3.60 (your IP may vary). If we open up Firefox now and point it at that 
address, you should see the site running.

!["mediawiki site"](./media/juju-mediawiki-site.png)

Congratulations, you have just deployed a service with Juju!

!!! Note: To remove all the services in the model you just created, it is often
quickest to destroy the model with the command 'juju destroy-model default` and
then [create a new model][models].


## Next Steps

Now that you have a Juju-powered cloud, it is time to explore the amazing
things you can do with it! 

We suggest you take the time to read the following:

- [Clouds][clouds] goes into detail about configuring other clouds, including the 
  public clouds like Azure, AWS, Google Compute Engine and Rackspace.
- [Models][models] - Learn how to create, destroy and manage models.
- [Charms/Services][charms] - find out how to construct complicated workloads 
  in next to no time.


[clouds]: ./clouds.html  "Configuring Juju Clouds"
[charm store]: https://jujucharms.com "Juju Charm Store"
[releases]: reference-releases.html 
[keygen]: ./getting-started-keygen-win.html "How to generate an SSH key with Windows"
[concepts]: ./juju-concepts.html "Juju concepts"
[charms]: ./charms.html
[models]: ./models.html
