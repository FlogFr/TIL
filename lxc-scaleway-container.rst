========================================
LXC Container on Scaleway Instance
========================================
`return to home <https://github.com/aRkadeFR/TIL>`_

Enabled kernel module
========================================
The host doesn't provide the ``lsmod`` or ``modinfo`` information.

To enable it:

.. code-block:: shell

    # apt-get install kmod

Check that the bridge module is set:

.. code-block:: shell

    # modinfo bridge
    filename:       /lib/modules/4.4.4-std-3/kernel/net/bridge/bridge.ko
    alias:          rtnl-link-bridge
    version:        2.3
    license:        GPL
    srcversion:     67D4DE97250E3B2CDA2AC8D
    depends:        stp,llc
    intree:         Y
    vermagic:       4.4.4-std-3 SMP mod_unload modversions 

Now set the kernel option for IP forwarding:

.. code-block:: shell

    # uncomment net.ipv4.ip_forward=1 in /etc/sysctl.conf
    # sysctl -p /etc/sysctl.conf
    net.ipv4.ip_forward = 1


Setup the network for LXC
=========================================
Create the network bridge, and then the LXC container will use the network type
veth which create interface to this bridge. Please do it on runtime in order to
avoid network problem with ``/etc/network/interfaces``.

.. code-block:: shell

    # brctl addbr br0
    # brctl show br0
    bridge name     bridge id               STP enabled     interfaces
    br0             8000.000000000000       no


Problem with TX Offload on the veth (#717215 debian bug).

.. code-block:: shell
    
    # apt-get install ethtool
    # ethtool --offload br0 tx off


Setup the DNS & DHCP software
=======================================
The goal is to have DHCP for container & DNS with the container name.lxc. We use
``dnsmasq`` to do that.

.. code-block:: shell

    # apt-get install dnsmasq

Set up the dnsmasq configuration:

- interface=br0 # listen to
- domain=lxc.local,10.0.3.0/24
- dhcp-range=br0,10.0.3.10,10.0.3.120,1h


In order to use locally this dnsmasq server (and so the cache),
then you can add this configuration to your /etc/dhcp/dhclient.conf:

.. code-block::

    interface "eth0" {
        prepend domain-name-servers 127.0.0.1
    }

Restart all your services, and check now that when you create a container LXC
with:

..  code-block:: shell

    ``lxc-create -t debian -n #NAME_CONTAINER#-- --release jessie --packages ssh,python``

you can ping it with ping #NAME_CONTAINER#.lxc.local :)


Enable the connection to the internet from container
====================================================
Now that you have lxc container that set up on 10.0.3.XXX with DHCP and DNS, you
need them to access the internet to download and install packages. To do so, we
will use iptables.

.. code-block:: shell
    
    # iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    # iptables -A FORWARD -i eth0 -o br0 -m state --state RELATED,ESTABLISHED -j ACCEPT
    # iptables -A FORWARD -i br0 -o eth0 -j ACCEPT

Now we are all set to use the LXC containers.
