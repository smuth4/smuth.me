+++
title = "Flashing LSI SAS 9211-8i with EFI"
tags = ["storage", "firmware"]
date = "2015-02-08"
+++

.. title: Flashing LSI SAS 9211-8i with EFI
.. slug: flashing-lsi-sas-9211-8i-with-efi
.. date: 2015-02-08 15:00:17 UTC-05:00
.. tags: storage,firmare
.. category: 
.. link: 
.. description: 
.. type: text

I recently went on an upgrade crusade to my homelab, and as part of that, upgraded FreeNAS to 9.3. When I did, there was a non-urgent alert about a driver mismatch for my LSI HBA (FreeNAS expected 16, LSI had 12). Thus, I decided to upgrade the firmware.

Directions
==========

This assumes that your server can boot directly into an EFI shell. It might require a Shell.efi for some motherboards, but I can't tell you much more than that, as it boots straight to EFI on mine. I'm going to be flashing mine to IT mode (so it passes the disks through directly to ZFS), but the steps to upgrading the firmware in IR mode is very similar.

To do this, you're going to need a USB drive (formatted with something like Rufus_) and 3 files:

- sas2flash.efi - The executable that will be doing the flashing
- mptsas2.rom - The BIOS ROM
- 2118it.bin/2118ir.bin - The firmware binary

All three of these files can be found on `LSI's website`_. The two downloads you need are Installer_PXX_for_UEFI and 9211-8i_Package_PXX_IR_IT_Firmware_BIOS_for_MSDOS_Windows, where XX is the version you want. If you need an older version like I did, you can find them through the `download search page`_.

Now open up Installer_PXX_for_UEFI.zip, find sas2flash_efi_ebc_rel/sas2flash.efi, and put that on the USB drive. Similarly, open up 9211-8i_Package_PXX_IR_IT_Firmware_BIOS_for_MSDOS_Windows.zip, extract Firmware/HBA_9211_8i_IT/2118it.bin and sasbios_rel/mptsas2.rom, and put those on the USB drive as well.

Alright, now that we have our files, we can boot into the EFI shell.

Verify you're in the USB root directory with ``ls``. If not, try ``map`` to list the devices, ``mount <device>`` and then ``<device>:`` to mount and enter the USB.

.. note::

   For this next step, apparenlty it's possible to upgrade multiple cards at once. I prefer the slow but safe way, since there's a (slim) chance of bricking a card, and flashed each card one at a time, manually pulling them in and out.

Verify we can see the hardware, and that it's a version we want to update::

  sas2flash.efi -listall

First we erase the old firmware::

  sas2flash.efi -o -e 6

Then we install the new one::

  sas2flash.efi -o -f 2118it.bin -b mptsas2.rom

And Bob's your uncle! Enjoy your new and exciting passthrough disk I/O.
  
Useful links:

http://digitalcardboard.com/blog/2014/07/09/flashing-it-firmware-to-the-lsi-sas-9211-8i-hba-2014-efi-recipe/

http://brycv.com/blog/2012/flashing-it-firmware-to-lsi-sas9211-8i/

http://linustechtips.com/main/topic/104425-flashing-an-lsi-9211-8i-raid-card-to-it-mode-for-zfssoftware-raid-tutorial/

.. _LSI's website: http://www.lsi.com

.. _download search page: http://www.lsi.com/support/pages/download-search.aspx

.. _Rufus: http://rufus.akeo.ie/
