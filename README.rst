Example TripleO Network Templates
=================================

This repository contains some example network templates that I use
with TripleO in my various development environments.  Below is a
description of the particular environments that the templates target
and instructions on how to use them.

The CIDRs and IP ranges in the respective network-environment.yaml files
are what I use in my environments, so you will most likely need to
customize them to match your environment (unless it already matches mine,
which is...unlikely ;-).

Note that the only node types I have tested these templates with are
Controller, Compute, and Ceph.  You may notice that the Cinder and Swift
types are commented out in the network-environment.yaml files, and this
is why.

Simple
------
About as basic as you can get and still be doing network isolation in
some form.  Runs the external API endpoints on an external network, and
everything else on the provisioning network.  This is mostly useful for
environments with two physical networks that don't have the ability to
run with vlans, because it moves the user-facing bits of OpenStack onto
a user-facing network and off the (in my case) unroutable provisioning
network.

Environment Details
~~~~~~~~~~~~~~~~~~~
I have used this configuration in my home baremetal environment before I
had a VLAN capable switch, but now I use it primarily with an
OpenStack Virtual Baremetal environment that is not VLAN capable either.
In the virtual environment, the target "baremetal" systems have their
first nic on a private, unroutable provisioning network, and their second
nic on a "public" network on which the undercloud also has an interface.

Usage Instructions
~~~~~~~~~~~~~~~~~~
::
    # Assumes this repo has been cloned to ~/tripleo-network-templates
    cp -r /usr/share/openstack-tripleo-heat-templates ~/simple-templates
    cp ~/tripleo-network-templates/simple/network-isolation.yaml ~/simple-templates/environments/network-isolation.yaml

Edit ~/tripleo-network-templates/simple/network-environment.yaml to reflect
your network setup.

Required parameters to deploy command (in addition to any others)::

    --templates ~/simple-templates -e ~/tripleo-network-templates/simple/network-environment.yaml -e ~/simple-templates/environments/network-isolation.yaml

Untagged External
-----------------
This is what I use for my home baremetal environment.  I'm not able to
run a standard single nic with vlans configuration because I only have
a single VLAN-capable switch, and additionally my provisioning network
is a completely separate, isolated switch connected to a dedicated port
on each server.

To address these issues, the templates in `untagged-external` add a
separate nic interface for the provisioning network, and configure the
OVS bridge so that the external network is untagged.  This allows me to
communicate with the external addresses from my VLAN-less home network
while still using VLAN isolation for all of the other networks.

.. note:: It seems to me that there should be some way to untag traffic
          that is outgoing from the switch port that is connected to the
          rest of my network, but my switch is pretty much VLAN for Dummies,
          so it doesn't expose more advanced configuration and I haven't
          figured out how to make it do that.  I would still need custom
          nic configs for the additional provisioning nic anyway, so I
          haven't spent much time investigating it.

Environment Details
~~~~~~~~~~~~~~~~~~~
Each target baremetal system has two network interfaces.  The first is
connected to a VLAN capable switch on my home network (11.0.0.0/8).
The second is connected to a dedicated, isolated provisioning switch,
which I have arbitrarily assigned the 9.1.1.0/24 network.

Usage Instructions
~~~~~~~~~~~~~~~~~~

Edit `untagged-external/network-environment.yaml` to reflect your network.

If you do not have any single digit CIDR masks in your network configuration
(which the example configuration does), the following parameters are all you
need to pass to the deploy command.  If you _do_ have a single digit CIDR
like I do, then see below for alternate instructions::

    # Assumes this repo has been cloned to ~/tripleo-network-templates
    -e ~/tripleo-network-templates/untagged-external/network-environment.yaml -e /usr/share/openstack-tripleo-heat-templates/environments/network-isolation.yaml

Single Digit CIDR Fix
~~~~~~~~~~~~~~~~~~~~~
The following instructions will enable a single digit CIDR mask on the
external network, as found in the example values for these templates::

    # Assumes this repo has been cloned to ~/tripleo-network-templates
    cp -r /usr/share/openstack-tripleo-heat-templates ~/untagged-external-templates
    sed -i 's/- {get_attr: \[ExternalPort, subnets, 0, cidr, -2\]}//' untagged-external-templates/network/ports/external.yaml

Required parameters to deploy command (in addition to any others)::

    --templates ~/untagged-external-templates -e ~/tripleo-network-templates/untagged-external/network-environment.yaml -e ~/untagged-external-templates/environments/network-isolation.yaml