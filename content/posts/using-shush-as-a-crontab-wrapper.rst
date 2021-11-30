+++
title = "Using shush as a crontab wrapper"
date = "2015-04-11"
tags = ["cron"]
+++

.. title: Using shush as a crontab wrapper
.. slug: using-shush-as-a-crontab-wrapper
.. date: 2015-04-11 15:24:09 UTC-04:00
.. tags: cron
.. category: 
.. link: 
.. description: 
.. type: text

`Cron`_ is a great tool for linux servers, but it's a very limited in it's capabilities (since it follows the Unix philosophy), so when I started to run up against those limits, I began doing all sorts of bash trickery to accomplish what I needed to happen, but that swiftly started giving me even more problem. At work, I use the `Jenkins CI`_ tool as a cron replacement (great tool, allows for distributed runs, queuing tasks, emails on failure, etc), but it seemed rather heavy weight for a homelab. Thankfully, there are a lot of cron wrappers/replacements out there, but I settled on `shush`_, a neat little wrapper script for cron. 

.. TEASER_END

There are a couple reasons I like it:

- The ability to manually run scripts under shush before adding them to cron
- Automatic crontab management
- Simple C binary, very few dependencies
- Text based config file
- Locking and timeout handling

Basically how it works is that there's a designated directory that holds all the shush configuration files. These files can be named anything, although I usually have them be extensionless for simplicity. You can then run any of these files with ``shush -c <directory> <config_file_name>``. If you use the default directory of ``$HOME/.shush/``, you can just run ``shush <config_file_name>``. However, in order to have the config file run regularly, you need to set the ``schedule`` setting, and run ``shush -c <directory> -u`` to update the crontab file.

The `man page`_ has some more documentation, but I'll at least break down the example config file to make it more understandable.

Set the command ``shush -c /etc/shush -u`` to run every day at 9 PM:

.. code::

          command=shush -c /etc/shush -u
          schedule=0 9 * * *

Use a lockfile. If a lockfile already exists, send an email to root and root-logs, then abort the job:

.. code::

          lock=notify=root root-logs,abort

Send out an email notification if the script is still running after 5 minutes, but keep the script alive:

.. code::

          timeout=5m,notify=root root-logs

Print stderr output first, use the ``text`` format, and set the email subject:

.. code::

          stderr=first
          format=text
          Subject=Crontab Daily Update

Always send an email to root-logs:

.. code::

          [logs]
          to=root-logs

If one of the various failure conditions applies, send an email to root:

.. code::

          [readers]
          if=$exit != 0 || $outlines != 1 || $errsize > 0 || U
          to=root
          format=rich

Shush is a very powerful tool, and I'm very happy with the ways that I've been able to implement it in my homelab. However, it can be a bit confusing for a beginner, and the regexp matching features leave much to be desired. With the intention of fixing things up, I've create a repository at https://github.com/smuth4/shush, which I can hopefully use to clean up things, and get the project active again. Feel free to compile, try out and maybe even contribute to this neat little tool!

.. _Cron: http://linux.die.net/man/1/crontab

.. _Jenkins CI: https://jenkins-ci.org/

.. _shush: http://web.taranis.org/shush/

.. _man page: http://web.taranis.org/shush/shush.1.html
