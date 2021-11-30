+++
tags = ["freebsd", "php"]
date = "2016-07-31"
title = "Upgrading from PHP 5.5 to 5.6 on FreeBSD"
+++

Recently PHP 5.5 got [EOL'd](http://php.net/supported-versions.php), but PHP 5.6 will be supported for another two years. On Debian, this is a just a matter of upgrading the php5 package, but FreeBSD splits it out into two packages: php55 and php56, not to mention that extensions are also split out this way. The fact that I've installed php via ports also complicates things.


## Doing the deed

This assumes [portmaster](https://www.freebsd.org/cgi/man.cgi?query=portmaster&apropos=0&sektion=8&manpath=FreeBSD+10.3-RELEASE+and+Ports&arch=default&format=html) is installed.

Listing the installed php55 packages:
```bash
$ pkg info | grep php55
php55-5.5.38                   PHP Scripting Language
php55-ctype-5.5.38             The ctype shared extension for php
etc...
```

Get the original ports path they were installed from ([pkg query](https://www.freebsd.org/cgi/man.cgi?query=pkg-query&sektion=8) is a fantastic command to have in your toolbelt), and convert any 5.5 refereces to 5.6:
```bash
$ pkg query "%o" $(pkg info | grep php55 | cut -f 1 -d ' ' | tr '\n' ' ') \
  | sed 's/55/56/g'
lang/php56
extproc/php56-ctype
etc...
```

For the sake of brevity, I'm going to place the list of packages into a temporary file
```bash
pkg query "%o" $(pkg info | grep php55 | cut -f 1 -d ' ' | tr '\n' ' ') \
  | sed 's/55/56/g' > /tmp/php56-packages.txt
```

I'm pretty sure that all the packages I care about have 5.6 equivalents, but just to be sure, let's check that those directories exist in the ports directory:
```bash
while read d; do [[ -d /usr/local/"$d" ]] && echo "$d does not exist"; done < /tmp/php56-packages.txt
```

Copy over the ports' options directory. (skip this step if you think any packages may need to be compiled differently):
```bash
mkdir -p /var/db/ports/lang_php56/ /var/db/ports/lang_php56-extensions/
cp /var/db/ports/lang_php55/options /var/db/ports/lang_php56/
cp /var/db/ports/lang_php55-extensions/options /var/db/ports/lang_php56-extensions/
```

Now begins the actual changes!
Build 5.6 over 5.5 with portmaster:
```bash
sudo portmaster -n -o /usr/ports/lang/php56 lang/php55
```

Do a quick dry run with portmaster (if there are any new options that can be set, portmaster will open up a prompt):
```bash
portmaster -n $(cat /tmp/php56-packages.txt | tr '\n' ' ')
```

Install the other packages sequentially, in the same manner as the main php56 package:
```bash
cat /tmp/php56-packages.txt | grep -v 'lang/php56$' | \
  while read p; do echo portmaster -D --no-confirm -o "/usr/ports/$p" "$(echo "$p" | sed 's/56/55/g')"; done
```

Restart any necessary daemons, and you're done! This was a learning process for me, I'm sure this could be compressed into a nice script. Of course, if you need to upgrade many machines, or downtime is an issue, building packages from the ports system (possible with portmaster's `-g` flag), and then installing them wholesale would cut down on the amount of time spent compiling everything.
