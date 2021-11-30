+++
tags = ["terraform", "powerdns"]
date = "2016-05-01"
title = "Making Terraform work with PowerDNS 4"
+++

Edit: This post has been made obsolete by a pull request I opened in the terraform repository: https://github.com/hashicorp/terraform/pull/7819

I've really enjoyed using [PowerDNS](https://www.powerdns.com/) as my DNS server at home. Most people only think of [BIND](https://www.isc.org/downloads/bind/) and [dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) when it comes to DNS, while ignoring this stable, scalable, secure database-backed offering that powers some really large deployments. But enough proselytizing! I'm in the middle of trying to migrate my infrastructure to be controlled via [Terraform](https://www.terraform.io/) (mostly). I figure this will help me consolidate and track most of my VPSs and the like.

## The problem

PowerDNS has a nice HTTP/JSON API that can be used, but it changed URL locations in version 4, from `/` to `/api/v1/`, which makes any Terraform changes return `powerdns_record.dns: Failed to create PowerDNS Record: Error creating record set: exmaple.com:::A, reason: "Not Found"`, which isn't exactly helpful (for the record, that's the error message returned by the API whenever a bad URL is requested).

## The solution

Unfortunately there's no easy fix, short of breaking compatibility for one thing or another. However, what I decided to do was to use [nginx](https://www.nginx.com/resources/wiki/) as a reverse proxy for the API. My configuration looks like this:
```
server {
    listen 8000;
    server_name _;

    location /api/v1/ {
        proxy_pass http://localhost:8081/;
    }

    location / {
        proxy_pass http://localhost:8081/api/v1/;
    }
}
```

This lets us use both the new and old locations simultaneously. I'm sure this can be done in Apache as well, but wasn't up for installing it just to test something so simple.
