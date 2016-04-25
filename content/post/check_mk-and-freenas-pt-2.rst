+++
title = "Check_mk and FreeNAS, Pt. 2"
date = "2014-04-25"
tags = ["check_mk", "FreeNAS", "monitoring"]
+++

.. title: Check_mk and FreeNAS, Pt. 2
.. slug: check_mk-and-freenas-pt-2
.. date: 2014/04/25 11:22:01
.. tags: check_mk,FreeNAS,monitoring
.. link: 
.. description: 
.. type: text

A continuation of :doc:`check_mk-and-freenas`

So I've got my check_mk on set up on my NAS, and it's monitoring stuff beautifully. However, it's not monitoring something very near and dear to my heart for this server: S.M.A.R.T. data. FreeNAS comes with smartctl, and there's already S.M.A.R.T. data plugins for the linux agents, so I figured this wouldn't be a big deal. And I was right! All I had to do was add the following script to my plugins/ folder for check_mk to find, and the server picked it up automatically. 

.. NOTE::
   The linux script has a lot fancier checking for edge cases. I figured FreeNAS would be homogeneous enough that it wouldn't be worth converting all those edge cases, so this is a very simple script, shared on a "works for me" basis.

.. listing:: smart bash
   
