+++
tags = ["netbox", "oauth"]
date = "2022-06-25"
title = "Integrating netbox with Authentik"
+++

Netbox has support for SSO integration out of the box, however some extra work
is required to make it work with authentik correctly.

## Setting up Authentik

1. In authentik, create an OAuth2/OpenID Provider (under Resources/Providers) with these settings:
   * Name: Netbox
   * Signing Key: Select any available key
2. Take note of the client ID & secret for later usage
3. Create an application with these settings:
   * Name: Netbox
   * Slug: netbox-slug
   * Provider: Netbox

## Setting up Netbox

### Building the image

This step is only required for docker. Netbox comes with the SSO python package (`social-auth-core`)
pre-installed, however not all the optional depedencies are installed due to relying on libraries that
may not be present[^1].

Luckily the image is made to be easily extendable:

```Dockerfile
FROM netboxcommunity/netbox:v3.2.5

RUN /opt/netbox/venv/bin/python -m pip install --upgrade 'social-auth-core[openidconnect]'
```

### Configuration

For the python configuration file, we'll combine the netbox documentation for
connecting to Okta[^2] with the generic OpenID connection backend[^3] from
social-core

```python
REMOTE_AUTH_BACKEND = 'social_core.backends.open_id_connect.OpenIdConnectAuth'
SOCIAL_AUTH_OIDC_OIDC_ENDPOINT = 'https://authentik.company/application/o/netbox-slug/'
SOCIAL_AUTH_OIDC_KEY = '<ID from step 2>'
SOCIAL_AUTH_OIDC_SECRET = '<secret from step 2>'
SOCIAL_AUTH_PROTECTED_USER_FIELDS = ['groups']
```

If `groups` is not set to be protected, you'll receive a an error from Django about
not being able to set a many-to-many field.

## Caveats

Currently this setup does not handle groups or superuser status. If that functionality
is required, an authentik LDAP outpost can be used instead.

[^1]: https://github.com/netbox-community/netbox/pull/8503
[^2]: https://docs.netbox.dev/en/stable/administration/authentication/okta/
[^3]: https://python-social-auth.readthedocs.io/en/latest/backends/oidc.html
