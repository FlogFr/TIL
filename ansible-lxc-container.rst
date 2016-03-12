============================================
Ansible on LXC container
============================================
`return to home <https://github.com/aRkadeFR/TIL>`_

Add SSH on LXC container
================================================
Default lxc container should have ssh installed.
Install it either while creating the container or after with ``apt-get install
ssh``.

.. code-block:: shell
    
    # lxc-create -t debian -n ansible -- --release jessie --packages ssh,dnsutils,iputils-ping

Then add in the default rootfs the root/user ssh key to connect to container.
