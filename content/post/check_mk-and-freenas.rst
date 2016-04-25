+++
title = "Check_mk and FreeNAS"
date = "2014-02-23"
tags = ["FreeNAS", "python", "check_mk"]
+++

.. title: Check_mk and FreeNAS
.. slug: check_mk-and-freenas
.. date: 2014/02/23 15:56:35
.. tags: FreeNAS, python, check_mk
.. link: 
.. description: 
.. type: text

.. note:: Software involved:

     - FreeNAS 9.2.0
     - OMD 1.10 (check_mk 1.2.2p3)

FreeNAS is great, and the web interface makes it easy and simple to understand my NAS's overall structure. However, my favored method of monitoring in my homelab is OMD with Check_mk, while FreeNAS prefers a self-contained collectd solution. We're in luck however, in that FreeNAS is heavily based on FreeBSD, which check_mk happens to have a plugin for, so it shouldn't be too hard to set things up the way I like them. There are two possible ways to do this:

* Enable inetd and point it to the check_mk_agent
* Call check_mk_agent over ssh through as shown in the `datasource programs`_ documentation

I decided to go the second way, as I prefer to avoid making manual changes to FreeNAS if I can avoid it.

If you do decided to go the inetd route, `this thread`_ may come in useful.

Agent Setup
===========

The first thing we need to do is set up a user with a home directory where we can store the check_mk_agent program. If you already have a non-root user set up for yourself (which is good practice), that will work perfectly fine (they may need root access to collect certain data points). If you want to be more secure, you can set up a check_mk-only user, and limit it to just the agent command, which I will explain below.

Once the user is set up with a writeable home directory, it's as simple as copying check_mk_agent.freebsd into the home directory. Run it once or twice to make sure it's collecting data correctly.

Check_mk setup
==============

From here it's basically following the instructions on the `datasource programs`_ documentation link. Here's a quick overview of the steps involved:

1. Add datasource programs configuration entry to main.mk. It will looks something like this:

   .. code:: python
   
     datasource_programs = [
       ( "ssh -l omd_user <IP> check_mk_agent", ['ssh'], ALL_HOSTS ),
     ]

2. Set up password-less key authentication ssh access for omd_user to FreeNAS
3. (Optional) Limit omd_user to onyl check_mk_agent command by placing command="check_mk_agent" 
4. Add ssh tag to FreeNAS host, through WATO or configuration files
5. Enjoy the pretty graphs and trends!

Tweaks
======

So we've got everything set up, but not everything is perfect. 

Network
~~~~~~~

The first thing that I noticed was missing was a network interface counter. check_mk_agent was outputting a 'netctr' section, which seemed to have all the necessary information, but it wasn't being recognized in check-mk inventory, as it's been superseeded by lnx_if. It's possible to re-enable netctr, but not on a per-host basis.

.. _`datasource programs`: http://mathias-kettner.com/checkmk_datasource_programs.html

.. _`this thread`: http://forums.freenas.org/index.php?threads/activation-of-inetd-server.3926/
