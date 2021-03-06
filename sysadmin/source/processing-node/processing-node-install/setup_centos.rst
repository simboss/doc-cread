.. _setup_centos.rst

Installing basic packages
=========================

We are re going to install a minimal CentOS 7 distribution. You can get a copy
of the .iso the image used for the installation `here
<http://mi.mirror.garr.it/mirrors/CentOS/7/isos/x86_64/CentOS-7-x86_64-Minimal-1503-01.iso>`_.

Installing the Operating System
-------------------------------

Boot up the installation DVD and start the `CentOS 7` Installation wizard.

    - Under `Select Date & Time` an set appropriate Date and Time settings
    - Under `Keyboard` and choose the keyboard layout
    - Under `Installation Destination` select the hard disk where CentOS will
      be installed. 
      
      Create a custom partitioning scheme as follows:
      
      +-----------------+----------------+-----------+-------------+
      | Partition Label | Partition Type | Size      | Mount Point |
      +=================+================+===========+=============+
      | boot            | ext3           |   700 MB  | /boot       |
      +-----------------+----------------+-----------+-------------+
      | root            | ext4           |    35 GB  | /           |
      +-----------------+----------------+-----------+-------------+
      | swap            | swap           |     4 GB  |             |
      +-----------------+----------------+-----------+-------------+
    - Under `Networking` configure your network interface according to your infrastructure
      you can either set it to `DHCP` to automatically get all the settings from
      a local DHCP server or configure it by hand.
    - Enable the network interface, then go back to `Select Date & Time` and enable
      NTP synchronization periodically get date and time settings from CentOS servers
    - Click on `Begin Installation`
    - Now set the password for the `root` user. Also click on `User Creation` to
      create the `geosolutions` user.
    -  Wait for the installation process to finish, then reboot your machine

Setup SSH authentication
------------------------

Login as `root` user and give the `geosolutions` user administrative privileges
by adding him to the `wheel` group: ::

    usermod -aG wheel geosolutions


Allow SSH connections through the firewall
''''''''''''''''''''''''''''''''''''''''''

On CentOS 7 the firewall is enabled by default. To allow SSH clients to connect
to the machine allow incoming connections on port 22::

    firewall-cmd --zone=public --add-port=22/tcp --permanent
    firewall-cmd --zone=public --add-service=ssh --permanent
    firewall-cmd --reload

Disable SSH login for the `root` user
'''''''''''''''''''''''''''''''''''''
.. warning::
    Before you disable root login make sure you are able to login via SSH with
    `geosolutions` user account and you have the privileges to run `sudo su` to
    switch to the `root` user account.

Edit file `/etc/ssh/sshd_config` to disable `root` login via SSH::

    PermitRootLogin no

Public key authentication
'''''''''''''''''''''''''

`Public key authentication`_. is generally consider a safer way to authenticate
users for SSH access. Let's set it up and disable password based authentication

.. _a link: https://en.wikipedia.org/wiki/Public-key_cryptography

First generate a public/private key pair using `ssh-keygen`::

    ssh-keygen

Folllow the procedure, you will end up with your newly generated key under `~/.ssh`
Now copy your `public` (by default it is called id_rsa.pub) key over the CentOS
machine in `/home/geosolutions/.ssh/authorized_keys`. There are several ways to do
it, we are going to use the `ssh-copy-id` tool::

        ssh-copy-id -i ~/.ssh/id_rsa.pub geosolutions@<server-ip-address>

You should now be able to login via SSH as `geosolutions` without been asked for
the password::


    ssh geosolutions@<server-ip-address>

You can now disable password based login over SSH

.. warning::
    Before disabling password authentication make sure you' ve installed your
    public key on the server and you are able to login without password

Edit `/etc/ssh/sshd_config` as follows::

    ...
    RSAAuthentication yes
    ...
    PubkeyAuthentication yes
    ...
    PasswordAuthentication no
    ...
    UsePAM no
    ...

