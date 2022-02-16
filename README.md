cilium-kubernetes
=================

This Ansible role installs [Cilium](https://docs.cilium.io) network on a Kubernetes cluster. Behind the doors it uses the official [Helm chart](https://helm.cilium.io/). Currently procedures like installing, upgrading and deleting the Cilium deployment are supported.

Versions
--------

I tag every release and try to stay with [semantic versioning](http://semver.org). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `6.0.0+1.11.1` means this is release `6.0.0` of this role and it contains Cilium chart version `1.11.1`. If the role itself changes `X.Y.Z` before `+` will increase. If the Cilium chart version changes `X.Y.Z` after `+` will increase too. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Cilium release.

Requirements
------------

You need to have [Helm 3](https://helm.sh/) binary installed on that host where `ansible-playbook` runs. You can either try to use your favourite package manager if your distribution includes `helm` in its repository or use one of the Ansible `Helm` roles (e.g. https://galaxy.ansible.com/gantsign/helm) or directly download the binary from https://github.com/helm/helm/releases and put it into `/usr/local/bin/` directory e.g. For Archlinux Helm can be installed via `sudo pacman -S helm` e.g.

And of course you need a Kubernetes Cluster ;-)

Role Variables
--------------

```
# Helm chart version
cilium_chart_version: "1.11.1"

# Helm chart name
cilium_chart_name: "cilium"

# Helm chart URL
cilium_chart_url: "https://helm.cilium.io/"

# Kubernetes namespace where Cilium resources should be installed
cilium_namespace: "cilium"

#
# etcd settings. If "cilium_etcd_enabled" variable is defined and set to "true",
# Cilium etcd settings are generated and deployed. Otherwise all the following
# "cilium_etcd_*" settings are ignored.
#
cilium_etcd_enabled: "true"

# Interface where etcd daemons are listening. If etcd daemons are bound to
# a WireGuard interface this setting should be "wg0" (by default) e.g.
# You can also use a variable like "{{ etcd_interface }}" if you used
# my etcd role (https://github.com/githubixx/ansible-role-etcd)
cilium_etcd_interface: "eth0"

# Port where etcd daemons are listening
cilium_etcd_client_port: 2379

# Ansible etcd host group in Ansible's "hosts" file. This value is used in
# "templates/cilium_values_default.yml.j2" template to determine the IP
# addresses of the hosts where etcd daemons are listening.
cilium_etcd_nodes_group: "k8s_etcd"

# If this variable is defined a Kubernetes secret will be installed which
# contains the certificate files defined in "cilium_etcd_cafile",
# "cilium_etcd_certfile" and "cilium_etcd_keyfile"
#
# This causes that a secure connection (https) will be established to etcd.
# This of course requires that etcd is configured to use SSL/TLS.
#
# If this value is not defined (e.g. commented) the rest of the "cilium_etcd_*"
# settings below are ignored and connection to etcd will be established
# unsecured via "http".
cilium_etcd_secrets_name: "cilium-etcd-secrets"

# Where to find the certificate files defined below. If you used my
# Kubernetes Certificate Authority role (https://github.com/githubixx/ansible-role-kubernetes-ca)
# you may already have "k8s_ca_conf_directory" variable defined which you
# can re-use here. This role also generates the certificate files that can
# be used for the variables below.
# By default this will be "$HOME/k8s/certs" of the current user that runs
# "ansible-playbook" command.
cilium_etcd_cert_directory: "{{ '~/k8s/certs' | expanduser }}"

# etcd certificate authority file (file will be fetched in "cilium_etcd_cert_directory")
cilium_etcd_cafile: "ca-etcd.pem"

# etcd certificate file (file will be fetched in "cilium_etcd_cert_directory")
# Make sure that the certificate contains the IP addresses in the "Subject
# Alternative Name" (SAN) of the interfaces where etcd daemons listens on
# (that's the IP addresses of the interfaces defined in "cilium_etcd_interface").
# This is already handled by my Kubernetes Certificate Authority role
# (https://github.com/githubixx/ansible-role-kubernetes-ca) if you used that one.
cilium_etcd_certfile: "cert-cilium.pem"

# etcd certificate key file (file will be fetched in "cilium_etcd_cert_directory")
cilium_etcd_keyfile: "cert-cilium-key.pem"
```

Usage:
------

The first thing to do is to check `templates/cilium_values_default.yml.j2`. This file contains the values/settings for the Cilium Helm chart that are different to the default ones which are located [here](https://github.com/cilium/cilium/blob/master/install/kubernetes/cilium/values.yaml). The default values of this Ansible role are using a TLS enabled `etcd` cluster. If you have a self hosted/bare metal Kubernetes cluster chances are high that there is already running an `etcd` cluster for the Kubernetes API server which is the case for me. I'm using my Ansible [etcd role](https://github.com/githubixx/ansible-role-etcd) to install such an `etcd` cluster and my [Kubernetes Certificate Authority role](https://github.com/githubixx/ansible-role-kubernetes-ca) to generate the certificates. So if you used my roles you can use this Cilium role basically as is.

The `templates/cilium_values_default.yml.j2` template also contains some `if` clauses to use an `etcd` cluster that is not TLS enabled. See `defaults/main.yml` to check which values can be changed. You can also introduce your own variables and use it in `templates/cilium_values_user.yml.j2` if you want of course.

But nothing is made in stone ;-) To use your own values just create a file called `cilium_values_user.yml.j2` and put it into the `templates` directory. Then this Cilium role will use that file to render the Helm values. You can use `templates/cilium_values_default.yml.j2` as a template or just start from scratch. As mentioned above you can modify all settings for the Cilium Helm chart that are different to the default ones which are located [here](https://github.com/cilium/cilium/blob/master/install/kubernetes/cilium/values.yaml).

After the values file (`templates/cilium_values_default.yml.j2` or `templates/cilium_values_user.yml.j2`) is in place and the `defaults/main.yml` values are checked the role can be installed. Most of the role's tasks are executed locally so to say as quite a few tasks need to communicate with the Kubernetes API server or executing [Helm](https://helm.sh/) commands and it makes no sense to execute them on the `etcd` or Kubernetes worker nodes of course.

The default action is to just render the Kubernetes resources YAML file after replacing all Jinja2 variables and stuff like that. In the `Example Playbook` section below there is an `Example 2 (assign tag to role)`. The role `githubixx.cilium-kubernetes` has a tag `role-cilium-kubernetes` assigned. Assuming that the values for the Helm chart should be rendered (nothing will be installed in this case) and the playbook is called `k8s.yml` execute the following command:

```
ansible-playbook --tags=role-cilium-kubernetes k8s.yml
```

One of the final tasks is called `TASK [githubixx.cilium-kubernetes : Output rendered template]`. This allows to check the YAML file before Cilium gets deployed. As you might figure out the output isn't that "pretty". To get around this you can either set `ANSIBLE_STDOUT_CALLBACK=debug` environment variable or `stdout_callback = debug` in `ansible.cfg`. If you run the `ansible-playbook` command again now the YAML output should now look way better.

If the rendered output contains everything you need the role can be installed which finally deploys Cilium:

```
ansible-playbook --tags=role-cilium-kubernetes --extra-vars action=install k8s.yml
```

To check if everything was deployed use the usual `kubectl` commands like `kubectl -n <cilium_namespace> get pods -o wide`.

As [Cilium](https://docs.cilium.io) issues updates/upgrades every few weeks/months the role also can do upgrades. The role basically executes what is described in [Cilium upgrade guide](https://docs.cilium.io/en/v1.10/operations/upgrade/). That means the Cilium pre-flight check will be installed and some checks are executed before the update actually takes place. Have a look at `tasks/upgrade.yml` to see what's happening before, during and after the update. Of course you should consult [Cilium upgrade guide](https://docs.cilium.io/en/v1.10/operations/upgrade/) in general to check for major changes and stuff like that before upgrading. [Roll back](https://docs.cilium.io/en/v1.10/operations/upgrade/#step-3-rolling-back) is currently not implemented but can be easily done via `kubectl` as described in the upgrade document.

Before doing the upgrade you basically only need to change `cilium_chart_version` variable e.g. from `1.9.7` to `1.10.1` to upgrade from `1.9.7` to `1.10.1`. So to do the update run

```
ansible-playbook --tags=role-cilium-kubernetes --extra-vars action=upgrade k8s.yml
```

As already mentioned the role already includes some checks to make sure the upgrade runs smooth but you should again check with `kubectl` if all works as expected after the upgrade.

And finally if you want to get rid of Cilium you can delete all resources again:

```
ansible-playbook --tags=role-cilium-kubernetes --extra-vars action=delete k8s.yml
```

If you don't have any CNI plugins configured this will cause `kublet` process on the Kubernetes worker nodes to issue CNI errors every now and then because there is no CNI related stuff anymore and of course connectivity between pods on different hosts will be gone together with any network policies and stuff like that.

Example Playbook
----------------

Example 1 (without role tag):
```
- hosts: k8s_worker
  roles:
    - githubixx.cilium-kubernetes
```

Example 2 (assign tag to role):
```
-
  hosts: k8s_worker
  roles:
    -
      role: githubixx.cilium-kubernetes
      tags: role-cilium-kubernetes
```

License
-------

GNU GENERAL PUBLIC LICENSE Version 3

Author Information
------------------

[http://www.tauceti.blog](http://www.tauceti.blog)
