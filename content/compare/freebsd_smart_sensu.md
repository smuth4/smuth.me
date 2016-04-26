+++
title = "Sensu S.M.A.R.T. checks for FreeBSD"
tags = ["FreeBSD", "sensu", "smart" ]
date = "2015-04-25"
draft = true
+++
[//]: # (☐☑☒)

I'm looking to monitor my drives on a FreeBSD machine via sensu.

## Requirements

* Works on FreeBSD
* Can filter drives (alternatively if it forces you to specify a single drive, that's fine)

## Wishlist

* Supports metrics
* Can work with `/dev/diskid/`: smartctl itself doesn't support this, and I can always write a in-script wrapper that uses `glabel status -s` to translate disk IDs, but native support would be nice.

## Options

### [Sensu Plugins - disk-checks](https://github.com/sensu-plugins/sensu-plugins-disk-checks)
☒ Does not work on FreeBSD: tries to cat `/proc/partitions`.

### [check_smartmon](https://exchange.nagios.org/directory/Plugins/Operating-Systems/Linux/check_smartmon/details)
☑ Works on Freebsd: shebang is set to `/usr/local/bin/python`, but it works fine with `/usr/local/bin/python2`.  
☑ Checks one drive at a time.  

### [check_smart_attributes](https://exchange.nagios.org/directory/Plugins/Operating-Systems/Linux/check_smart_attributes/details)
Received `Can't locate utils.pm in @INC`, when I tried to run it, which is apparently due to the fact that I'm not using Nagios. Pass.

### [check_smart](https://github.com/Napsty/check_smart)
Fork of 2009's check_smart Nagios plugin by Kurt Yoder.

Same `utils.pm` issue. Pass.

## Selection

Inspired by a [ServerFault answer](http://serverfault.com/a/661381) I decided that monitoring all the drives via Sensu was impractical. Instead I'll just configure smartd, and use Sensu to make sure that process is alive. I can also extend that strategy to my Linux hosts as well.
