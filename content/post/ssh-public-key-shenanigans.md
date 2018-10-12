+++
date = "2016-10-19"
title = "SSH public key shenanigans"
tags = ["security", "SSH"]
+++

A fun little fact I discovered about SSH: when you specify a private key to use, it checks ${key}.pub for hints about how to parse the private key, without warning. Under normal operations this is never a problem, but you need to replace a private key in-place, and don't update the .pub file, authentication will fail:

{{< highlight diff >}}
$ ls -la
ssh.key ssh.key.pub
$ ssh user@host echo ping
user@host's password: ^C
$ mv ssh.key.pub ssh.key.pub.bak
$ ssh user@host echo ping
Last login: Tue Oct 19
ping
{{< / highlight >}}

This can be partially seen in the output of the ssh client's `-vvv` option. On the left is with the public key preset, on the right is without:

{{< highlight diff >}}
debug1: identity file ./ssh.key type 2                   | debug1: identity file ./ssh.key type -1
...
debug2: dh_gen_key: priv key bits set: 131/256           | debug2: dh_gen_key: priv key bits set: 125/256
debug2: bits set: 529/1024                               | debug2: bits set: 532/1024
...
debug2: bits set: 512/1024                               | debug2: bits set: 482/1024
...
debug2: key: ./ssh.key (0x7fdea492cb30)                  | debug2: key: ./ssh.key ((nil))
...
debug3: send_pubkey_test                                 | debug1: read PEM private key done: type DSA
                                                         > debug3: sign_and_send_pubkey
debug2: we sent a publickey packet, wait for reply         debug2: we sent a publickey packet, wait for reply
debug3: Wrote 528 bytes for a total of 1653              | debug3: Wrote 592 bytes for a total of 1717
debug1: Authentications that can continue: publickey,pas | debug1: Authentication succeeded (publickey).
debug2: we did not send a packet, disable method         | debug2: fd 5 setting O_NONBLOCK
{{< / highlight >}}

Interestingly, I wasn't able to find any official documentation that mentions this, and only figured it out after resorting to `strace`.
