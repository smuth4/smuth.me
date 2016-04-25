+++
title = "Check_mk and FreeNAS, Pt. 3"
date = "2014-07-10"
tags = ["FreeNSD", "check_mk", "monitoring"]
+++

.. title: Check_mk and FreeNAS, Pt. 3
.. slug: check_mk-and-freenas-pt-3
.. date: 2014-07-10 11:44:19 UTC-04:00
.. tags: check_mk,FreeNAS,monitoring
.. link: 
.. description: 
.. type: text

A continuation of :doc:`check_mk-and-freenas-pt-2`

Now that we have all our smart data nicely set up, let's see if we can't get some stats on I/O speed. I'm pretty sure FreeNAS is supposed to have a I/O section in its "Reports" section, but for whatever reason, it's not in my install, and I'd like to have the data in Nagios in any case.

Just like with the SMART data, we're going to write a small script that the check_mk agent can use. Unlike the SMART script, getting IO stats is incredibly easy.

.. listing:: iostat bash

Yep, that's all it is. We're only really interesting in the drives being used in ZFS, but you could open it up to all drives if you wanted to.

Next up is to let check_mk be able to recognize the agent's output. The check script that I use can be found `here </check_mk_freenas_iostat/iostat>`__. It should be placed in the "checks/" directory of your check_mk install.

I also created a quick template for pnp4nagios, found `here </check_mk_freenas_iostat/check_mk-iostat.php>`__, which should be placed in the "templates/" directory.

After all this, we've finally got a solid set up for tracking disks in FreeNAS. For each disk, there will be three associated services: SMART data, temperature (taken from the SMART data), and iostat data.
