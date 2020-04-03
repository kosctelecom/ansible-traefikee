# Traefikee

Install and configure a [TraefikEE](https://containo.us/traefikee/) on-prem cluster

## Role Variables

From [the defaults](./defaults/main.yml) (skipping boring stuff, see the file for fine tweaks such
as install path, systemd service name, startup command lines...):

```yaml

# Name of the groups in the inventory for the controllers and the proxies
traefikee_controller_group: traefikee_controller
traefikee_proxy_group: traefikee_proxy

# Those are set to true at cluster installation only.
traefikee_install: false  # true to install traefikee binaries
traefikee_configure: false  # true to configure and start the systemd services

# Dynamic configuration
# each entry is a configuration file. If the config evaluates to false
# the config file is deleted
#
# example:

# traefikee_cluster_dynamic_config:
#   my_site:
#     http:
#       routers:
#         my-site-router:
#           rule: "Host('www.example.org')"
#           entryPoints:
#             - https
#           service: site
#       services:
#         site:
#           loadBalancer:
#             servers:
#               - url: "http://10.0.0.42:8000"
#   my_old_site: null

traefikee_cluster_dynamic_config: {}

# Get a license key getting in touch with Containous sales team
traefikee_license_key: ""

# binaries download info
traefikee_version: 2.0.2
traefikee_arch: linux_amd64

# At startup, traefikee calls home for license check. This is added as
# systemd service env vars.
traefikee_http_proxy: ""
traefikee_https_proxy: "{{ traefikee_http_proxy }}"

# Add additional env vars here.
# e.g. for Lego configuration (https://docs.traefik.io/https/acme/) :
# traefikee_environment_extra: |
#   OVH_ENDPOINT=ovh-eu
#   OVH_APPLICATION_KEY=123456
#   OVH_APPLICATION_SECRET=123456
#   OVH_CONSUMER_KEY=abcdef1234
traefikee_environment_extra: ""

# The static configuration is a verbatim copy of this mapping
traefikee_cluster_config:
  entryPoints:
    http:
      address: ":80"
    https:
      address: ":443"
  providers:
    # this provider is required if one wants to use traefikee_cluster_dynamic_config
    file:
      directory: "{{ traefikee_cluster_dynamic_config_dir }}"
      watch: true
```

## Example Playbook

Given this inventory:

```yaml
all:
  children:
    traefikee:
      vars:
        # TODO: ATM this is hard-coded, we must get this from the facts 
        traefikee_controller_listen_address: 10.108.0.18
        # obvouisly this should be vaulted
        traefikee_license_key: !vault |
           **********
        traefikee_http_proxy: http://mysquid:3128
      children:
        traefikee_controller:
          hosts:
            traefik-ctl-01.example.net:
        traefikee_proxy:
          hosts:
            traefik-proxy-01.example.net:
            traefik-proxy-02.example.net:
```

### Install and configure the cluster

```yaml
- hosts: traefikee
  become: yes
  environment:
    http_proxy: http://mysquid:3128
    https_proxy: http://mysquid:3128
  vars:
    traefikee_install: yes
    traefikee_configure: yes
  roles:
    - traefikee
```

### Update the dynamic configuration

```yaml
- hosts: traefikee
  become: yes
  vars:
    traefikee_cluster_dynamic_config:
      remove_me: null
      my_site:
        http:
          routers:
            my-site-router:
              rule: "Host('www.example.org')"
              entryPoints:
                - https
              service: site
          services:
            site:
              loadBalancer:
                servers:
                  - url: "http://10.0.0.42:8000"

  roles:
    - traefikee
```

## License

© 2020 – Kosc Telecom

Distributed under the terms of GPLV3
