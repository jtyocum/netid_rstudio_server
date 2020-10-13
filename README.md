# R Studio Server on NETID

This playbook is designed to simplify the setup of an R Studio Server on the UW NETID domain. It assumes the server is running a fresh install of Ubuntu 20.04, and will be dedicated to the role. The playbook performs the following operations:

* basic system hardening
* install of R, R Studio, and dependencies
* install of Samba, Winbind, autofs, etc.
* install of Apache as a reverse proxy, along with Shibboleth for Weblogin

## Requirements

### System Setup

* Fresh Ubuntu 20.04 system, with SSH installed
* Root SSH key installed

### Additional Roles

The playbook depends on the following external roles and collections:

* https://github.com/jtyocum/debuntu_security_baseline
* community.crypto

## Hosts / Inventory

Please see the ansible_hosts.example file for an example of the various playbook variables you'll need to set.

## Manual Operations

There are some tasks the playbook doesn't perform for you, such as requesting a SSL certificate from UW-IT. Details on these steps can be found below.

### Request SSL Certificate

1. CSR is written to /etc/ssl/{{ rstudio_server_name }}.csr
2. Submit an InCommon certificate request via https://iam-tools.u.washington.edu/cs/
3. Once certificate is received, save it to /etc/ssl/certs/{{ rstudio_server_name }}_cert.pem
4. Deactivate the default virtual host ``` a2dissite 000-default ```
5. Activate the proxy virtual host ``` a2ensite {{ rstudio_server_name }} ```
6. Restart Apache ``` systemctl restart apache2 ```

### AD Join

NETID domain join requires a delegated OU. Below is an example of joining the "EXAMPLE" OU.

```
net ads join "createcomputer=Delegated/EXAMPLE" -U netid\\sadm_example -d1
```

### Shibboleth Registration

Register the application via https://iam-tools.u.washington.edu/spreg, using the entity ID "https://{{ rstudio_server_name }}/shibboleth".
