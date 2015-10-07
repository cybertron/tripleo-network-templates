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
run with vlans because it moves the user-facing bits of OpenStack onto
a user-facing network and off the (in my case) unroutable provisioning
network.

To use::

    # Assumes this repo has been cloned to ~/tripleo-network-templates
    cp -r /usr/share/openstack-tripleo-heat-templates ~/simple-templates
    cp ~/tripleo-network-templates/simple/network-isolation.yaml ~/simple-templates/environments/network-isolation.yaml

Required parameters to deploy command (in addition to any others)::

    --templates ~/simple-templates -e ~/tripleo-network-templates/simple/network-environment.yaml -e ~/simple-templates/environments/network-isolation.yaml