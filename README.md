# Image Builder

Based on the RH-COP Repositories:

[rhis-builder](https://github.com/redhat-cop/rhis-builder) and [infra.osbuild](https://github.com/redhat-cop/infra.osbuild)

## Goal

Build up knowledge of [osbuild composer](https://osbuild.org/) and [rpm-ostree](https://coreos.github.io/rpm-ostree/).  

Also I want to build the [selfhosted version](https://osbuild.org/docs/on-premises/overview) of the [Red Hat Hybrid Cloud Console](https://console.redhat.com/insights/image-builder) and test its practical capabilities.  


## Getting started manually:  

### Install osbuild composer

```bash
sudo dnf install osbuild-composer composer-cli -y
sudo systemctl enable --now osbuild-composer.socket
sudo usermod -a -G weldr $USER
newgrp weldr
```


### Check via CLI
```bash
composer-cli status show
```

## Install Web UI (cockpit)
```bash
sudo dnf install cockpit-composer -y
sudo systemctl enable --now cockpit.socket
```

### Check via Web UI

http://localhost:9090/composer

## Uninstall

```bash
sudo dnf remove osbuild-composer composer-cli cockpit-composer -y
sudo usermod -r -G weldr $USER
sudo systemctl daemon-reload
sudo rm -rf /var/lib/osbuild-composer
sudo rm -rf /var/cache/osbuild-composer
```

If you run into issues, you might need to reboot due to systemd magic.

```bash
reboot
```


## Getting Started via Ansible
Official collection

```bash
ansible-galaxy collection install infra.osbuild
```

[Fork](https://github.com/daparm/infra.osbuild) for creating OVA images
```bash
ansible-galaxy collection install git+https://github.com/daparm/infra.osbuild.git --force
```

Create Inventory to install it locally:

```bash
cat <<EOF> hosts
---
all:
  hosts:
    localhost:
  vars:
    ansible_connection: local
...
EOF
```

### Installing the osbuild server

I am running into issues though, could'nt yet figured out what is going wrong, may consider installing it with above commands, if the role fails.  

```bash
ansible-playbook infra.osbuild.osbuild_setup_server.yml -i hosts -K
```

### Prep

create a ssh key pair for testing purposes:

```bash
ssh-keygen -t rsa -N "" -f ./image_builder_test
```

### Building images

Building images with default variable set:

#### RHEL
```bash
ansible-playbook infra.osbuild.osbuild_builder.yml -e builder_compose_type=image-installer -i hosts -K
```
### VMware - OVA (needs fork)
```bash
ansible-playbook infra.osbuild.osbuild_builder.yml -e builder_compose_type=ova -i hosts -K
```
#### Fedora
```bash
ansible-playbook infra.osbuild.osbuild_builder.yml -e builder_compose_type=iot-installer -i hosts -K
```

Building images with customized variable set:

#### RHEL
```bash
ansible-playbook infra.osbuild.osbuild_builder.yml -e "@image-installer.yml" -i hosts -K
```
#### Fedora
```bash
ansible-playbook infra.osbuild.osbuild_builder.yml -e "@iot-installer.yml" -i hosts -K
```
### VMware - OVA (needs fork)
```bash
ansible-playbook infra.osbuild.osbuild_builder.yml -e "@ova.yml" -i hosts -K
```

### Aftermatch - inspect / investigati file structure

Apache Server is running on /var/www/html

tree -L 3 /var/www/html/
```
/var/www/html/
├── fedora_image_installer_build
│   ├── images
│   │   └── 0.0.1
│   ├── kickstart.ks
│   └── repo
│       ├── config
│       ├── extensions
│       ├── objects
│       ├── refs
│       ├── state
│       └── tmp
└── rhel_image_installer_build
    ├── images
    │   └── 0.0.2
    ├── kickstart.ks
    └── repo
        ├── config
        ├── extensions
        ├── objects
        ├── refs
        ├── state
        └── tmp
```

You can see the kickstart.ks file which get injected later on, the blueprint file will be in /tmp/blueprint.toml based on this variable:

```yaml
builder_blueprint_src_path: /tmp/blueprint.toml
```

Example Blueprint for [rhel-9-latest-CIS-L2-Server.toml](https://github.com/redhat-cop/infra.osbuild/blob/main/blueprints/rhel-9-latest-CIS-L2-Server.toml):

With Filesystem - Parttion Schema

```ini
###############################################################################
#
# Blueprint for CIS Red Hat Enterprise Linux 9 Benchmark for Level 2 - Server
#
# Profile Description:
# This profile defines a baseline that aligns to the "Level 2 - Server"
# configuration from the Center for Internet Security® Red Hat Enterprise
# Linux 9 Benchmark™, v1.0.0, released 2022-11-28.
# This profile includes Center for Internet Security®
# Red Hat Enterprise Linux 9 CIS Benchmarks™ content.
#
# Profile ID:  xccdf_org.ssgproject.content_profile_cis
# Benchmark ID:  xccdf_org.ssgproject.content_benchmark_RHEL-9
# Benchmark Version:  0.1.69
# XCCDF Version:  1.2
#
# This file was generated by OpenSCAP 1.3.8 using:
# oscap xccdf generate fix --profile xccdf_org.ssgproject.content_profile_cis --fix-type blueprint xccdf-file.xml
#
# This Blueprint is generated from an OpenSCAP profile without preliminary evaluation.
# It attempts to fix every selected rule, even if the system is already compliant.
#
# How to apply this Blueprint:
# composer-cli blueprints push blueprint.toml
#
# See RHEL documentation for more customization options. 
# https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/composing_a_customized_rhel_system_image/index#composer-blueprint-format_creating-system-images-with-composer-command-line-interface
#
# oscap info /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
# oscap info --profiles /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
# oscap info --profile xccdf_org.ssgproject.content_profile_cis /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
# oscap xccdf generate fix --profile xccdf_org.ssgproject.content_profile_cis --fix-type blueprint /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
# 
# Note:  When importing into image builder, all comment lines will be removed.
###############################################################################

name = "RHEL-9-latest-CIS-L2-Server"
description = "Blueprint for CIS Red Hat Enterprise Linux 9 Benchmark for Level 2 - Server from profile xccdf_org.ssgproject.content_profile_cis and auditable by Red Hat Insights."
version = "0.0.1"
modules = []
groups = []
distro = ""

[customizations.openscap]
datastream = "/usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml"
profile_id = "xccdf_org.ssgproject.content_profile_cis"

[[packages]]
name = "aide"

[[packages]]
name = "audit"

[[packages]]
name = "firewalld"

[[packages]]
name = "rsyslog"

[[packages]]
name = "libselinux"

[[packages]]
name = "scap-security-guide"

[[packages]]
name = "sudo"

[[packages]]
name = "insights-client"

[[packages]]
name = "rhc"

[[packages]]
name = "rhc-worker-playbook"

[[packages]]
name = "glibc-langpack-en"

[[packages]]
name = "langpacks-en"

[[packages]]
name = "ansible-core"

[[packages]]
name = "rhel-system-roles"

[customizations.kernel]
append = "audit_backlog_limit=8192 audit=1"

[customizations.firewall]
[customizations.firewall.services]
enabled = ["ssh"]

[customizations.services]
enabled = ["sshd", "crond", "firewalld", "systemd-journald", "rsyslog", "auditd"]
disabled = ["nfs-server", "rpcbind"]

# The below add up to a 10 GB disk size.  Adjust according to deployment environment.
[[customizations.filesystem]]
mountpoint = "/"
minsize = "4 GB"

[[customizations.filesystem]]
mountpoint = "/dev/shm"
size =  "1 GB"  # typical default is tmpfs, half of RAM

[[customizations.filesystem]]
mountpoint = "/home"
size =  "1 GB"

[[customizations.filesystem]]
mountpoint = "/tmp"
size =  "1 GB"

[[customizations.filesystem]]
mountpoint = "/var"
size =  "1 GB"

[[customizations.filesystem]]
mountpoint = "/var/log"
size =  "1 GB"

[[customizations.filesystem]]
mountpoint = "/var/log/audit"
size =  "1 GB"

[[customizations.filesystem]]
mountpoint = "/var/tmp"
size =  "1 GB"
```
