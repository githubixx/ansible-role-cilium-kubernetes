# cilium-kubernetes

This Ansible role installs [Cilium](https://docs.cilium.io) network on a Kubernetes cluster. Behind the doors it uses the official [Helm chart](https://helm.cilium.io/). Currently procedures like installing, upgrading and deleting the Cilium deployment are supported.

## Versions

I tag every release and try to stay with [semantic versioning](http://semver.org). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `13.0.0+1.15.3` means this is release `13.0.0` of this role and it contains Cilium chart version `1.15.3`. If the role itself changes `X.Y.Z` before `+` will increase. If the Cilium chart version changes `X.Y.Z` after `+` will increase too. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Cilium release.

## Requirements

You need to have [Helm 3](https://helm.sh/) binary installed on that host where `ansible-playbook` is executed or on that host where you delegated the playbooks to (e.g. by using `cilium_delegate_to` variable). You can either

- use your favorite package manager if your distribution includes `helm` in its repository (for Archlinux use `sudo pacman -S helm` e.g.)
- or use one of the Ansible `Helm` roles (e.g. [helm](https://galaxy.ansible.com/gantsign/helm) - which gets also installed if you use `ansible-galaxy role install -vr requirements.yml`
- or directly download the binary from [Helm releases](https://github.com/helm/helm/releases) and put it into `/usr/local/bin/` directory e.g.

A properly configured `KUBECONFIG` is also needed (which is located at `${HOME}/.kube/config` by default). Normally if `kubectl` works with your cluster then everything should be already fine in this regards.

Additionally the Ansible `kubernetes.core` collection needs to be installed. This can be done by using the `collections.yml` file included in this role: `ansible-galaxy install -r collections.yml`.

And of course you need a Kubernetes Cluster ;-)

## Installation

- Directly download from Github (Change into Ansible roles directory before cloning. You can figure out the role path by using `ansible-config dump | grep DEFAULT_ROLES_PATH` command):
`git clone https://github.com/githubixx/ansible-role-cilium-kubernetes.git githubixx.cilium_kubernetes`

- Via `ansible-galaxy` command and download directly from Ansible Galaxy:
`ansible-galaxy install role githubixx.cilium_kubernetes`

- Create a `requirements.yml` file with the following content (this will download the role from Github) and install with `ansible-galaxy role install -r requirements.yml` (change `version` if needed):

```yaml
---
roles:
  - name: githubixx.cilium_kubernetes
    src: https://github.com/githubixx/ansible-role-cilium-kubernetes.git
    version: 13.0.0+1.15.3
```

## Changelog

**Change history:**

See full [CHANGELOG.md](https://github.com/githubixx/ansible-role-kubernetes-worker/blob/master/CHANGELOG.md)

**Recent changes:**

## 13.1.0+1.15.8

### Update

- upgrade to Cilium `v1.15.8

### Molecule

- replace Vagrant `alvistack/ubuntu-22.04` boxes with `alvistack/ubuntu-24.04`

## 13.0.0+1.15.3

### Breaking

- changes in `templates/cilium_values_default.yml.j2`:
  - added `kubeProxyReplacement`, `nodePort` and `socketLB` (this is needed because BPF masquerade requires NodePort)

### Update

- upgrade to Cilium `v1.15.3`

### Molecule

- replace Vagrant `generic/ubuntu2204` boxes with `alvistack/ubuntu-22.04`

## 12.0.0+1.15.0

- upgrade to Cilium `v1.15.0`
- refactor Molecule setup
- introduce `cilium_chart_values_directory` variable

## Role Variables

```yaml
# Helm chart version
cilium_chart_version: "1.15.8"

# Helm chart name
cilium_chart_name: "cilium"

# Helm chart URL
cilium_chart_url: "https://helm.cilium.io/"

# Kubernetes namespace where Cilium resources should be installed
cilium_namespace: "cilium"

# Directory that contains Helm chart values file. Ansible will try to locate
# a file called "values.yml.j2" or "values.yaml.j2" in the specified directory
# (".j2" because you can use the usual Jinja2 template stuff there).
# If not found the default "templates/cilium_values_default.yml.j2" will be
# used (which can be used as a template BTW). The content of this file
# will be provided to "helm install/template" command as values file.
cilium_chart_values_directory: "/tmp/cilium/helm"

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

# By default all tasks that needs to communicate with the Kubernetes
# cluster are executed on your local host (127.0.0.1). But if that one
# doesn't have direct connection to this cluster or should be executed
# elsewhere this variable can be changed accordingly. 
cilium_delegate_to: 127.0.0.1

# Shows the "helm" command that was executed if a task uses Helm to
# install, update/upgrade or deletes such a resource.
cilium_helm_show_commands: false

# Without "cilium_action" variable defined this role will only render a file
# with all the resources that will be installed or upgraded. The rendered
# file with the resources will be called "template.yml" and will be
# placed in the directory specified below.
cilium_template_output_directory: "{{ '~/cilium/template' | expanduser }}"
```

## Usage

The first thing to do is to check `templates/cilium_values_default.yml.j2`. This file contains the values/settings for the Cilium Helm chart that are different to the default ones which are located [here](https://github.com/cilium/cilium/blob/master/install/kubernetes/cilium/values.yaml). The default values of this Ansible role are using a TLS enabled `etcd` cluster. If you have a self hosted/bare metal Kubernetes cluster chances are high that there is already running an `etcd` cluster for the Kubernetes API server which is the case for me. I'm using my Ansible [etcd role](https://github.com/githubixx/ansible-role-etcd) to install such an `etcd` cluster and my [Kubernetes Certificate Authority role](https://github.com/githubixx/ansible-role-kubernetes-ca) to generate the certificates. So if you used my roles you can use this Cilium role basically as is.

The `templates/cilium_values_default.yml.j2` template also contains some `if` clauses to use an `etcd` cluster that is not TLS enabled. See `defaults/main.yml` to check which values can be changed. You can also introduce your own variables. To use your own values just create a file called `values.yml.j2` or `values.yaml.j2` and put it into the directory specified in `cilium_chart_values_directory`. Then this role will use that file to render the Helm values.

After the values file is in place and the `defaults/main.yml` values are checked the role can be installed. Most of the role's tasks are executed locally by default so to say as quite a few tasks need to communicate with the Kubernetes API server or executing [Helm](https://helm.sh/) commands. But you can delegate this kind of tasks to a different host by using `cilium_delegate_to` variable (see above). Just make sure that the host you delegate these kind of tasks has connection to the Kubernetes API server and the user a valid `KUBECONFIG` file.

The default action is to just render the Kubernetes resources YAML file after replacing all Jinja2 variables and stuff like that. In the `Example Playbook` section below there is an `Example 2 (assign tag to role)`. The role `githubixx.cilium_kubernetes` has a tag `role-cilium-kubernetes` assigned. Assuming that the values for the Helm chart should be rendered (nothing will be installed in this case) and the playbook is called `k8s.yml` execute the following command:

```bash
ansible-playbook --tags=role-cilium-kubernetes k8s.yml
```

To render the template into a different directory use `cilium_template_output_directory` variable e.g.:

```bash
ansible-playbook --tags=role-cilium-kubernetes --extra-vars cilium_template_output_directory="/tmp/cilium" k8s.yml
```

If you want to see the `helm` commands and the parameters which were executed in the logs you can also specify `--extra-vars cilium_helm_show_commands=true`.

One of the final tasks is called `TASK [githubixx.cilium_kubernetes : Write templates to file]`. This renders the template with the resources that will be created into the directory specified in `cilium_template_output_directory`. The file will be called `template.yml`. The directory/file will be placed on your local machine to be able to inspect it.

If the rendered output contains everything you need the role can be installed which finally deploys Cilium:

```bash
ansible-playbook --tags=role-cilium-kubernetes --extra-vars cilium_action=install k8s.yml
```

To check if everything was deployed use the usual `kubectl` commands like `kubectl -n <cilium_namespace> get pods -o wide`.

As [Cilium](https://docs.cilium.io) issues updates/upgrades every few weeks/months the role also can do upgrades. The role basically executes what is described in [Cilium upgrade guide](https://docs.cilium.io/en/v1.15/operations/upgrade/). That means the Cilium pre-flight check will be installed and some checks are executed before the update actually takes place. Have a look at `tasks/upgrade.yml` to see what's happening before, during and after the update. Of course you should consult [Cilium upgrade guide](https://docs.cilium.io/en/v1.15/operations/upgrade/) in general to check for major changes and stuff like that before upgrading. Also make sure to check the [Upgrade Notes](https://docs.cilium.io/en/stable/operations/upgrade/#current-release-required-changes)!

If a upgrade wasn't successful a [Roll back](https://docs.cilium.io/en/v1.15/operations/upgrade/#step-3-rolling-back) to a previous version can be basically initiated by just changing `cilium_chart_version` variable. But you should definitely read the Cilium [roll back guide](https://docs.cilium.io/en/v1.15/operations/upgrade/#step-3-rolling-back). Switching between minor releases is normally not an issue but switching from one major release to a previous one might be not so easy.

Also check `templates/cilium_values_default_pre_flight_check.yml.j2`. If you need to adjust values for the `pre-flight` check you can either change that file or create a file `templates/cilium_values_user_pre_flight_check.yml.j2` with your own values.

Before doing the upgrade you basically only need to change `cilium_chart_version` variable e.g. from `1.13.4` to `1.14.5` to upgrade from `1.13.4` to `1.14.5`. So to do the update run

```bash
ansible-playbook --tags=role-cilium-kubernetes --extra-vars cilium_action=upgrade k8s.yml
```

As already mentioned the role already includes some checks to make sure the upgrade runs smooth but you should again check with `kubectl` if all works as expected after the upgrade.

And finally if you want to get rid of Cilium you can delete all resources again:

```bash
ansible-playbook --tags=role-cilium-kubernetes --extra-vars cilium_action=delete k8s.yml
```

If you don't have any CNI plugins configured this will cause `kubelet` process on the Kubernetes worker nodes to issue CNI errors every now and then because there is no CNI related stuff anymore and of course connectivity between pods on different hosts will be gone together with any network policies and stuff like that.

## Example Playbook

Example 1 (without role tag):

```yaml
- hosts: k8s_worker
  roles:
    - githubixx.cilium_kubernetes
```

Example 2 (assign tag to role):

```yaml
-
  hosts: k8s_worker
  roles:
    -
      role: githubixx.cilium_kubernetes
      tags: role-cilium-kubernetes
```

## Testing

This role has a small test setup that is created using [Molecule](https://github.com/ansible-community/molecule), libvirt (vagrant-libvirt) and QEMU/KVM. Please see my blog post [Testing Ansible roles with Molecule, libvirt (vagrant-libvirt) and QEMU/KVM](https://www.tauceti.blog/posts/testing-ansible-roles-with-molecule-libvirt-vagrant-qemu-kvm/) how to setup. The test configuration is [here](https://github.com/githubixx/ansible-role-cilium-kubernetes/tree/master/molecule/default).

Afterwards molecule can be executed. The following command will do a basic setup and create a template of the resources (default action see above) that will be created:

```bash
molecule converge
```

Installing `Cilium` and the required resources. This will setup a few virtual machines (VM) and installs a Kubernetes cluster. That setup will be used to install `Cilium` by using this role.

```bash
molecule converge -- --extra-vars cilium_action=install
```

The following command can be used to install [CoreDNS](https://github.com/githubixx/ansible-kubernetes-playbooks/tree/master/coredns) for Kubernetes DNS stuff and taints controller nodes to only run Cilium pods:

```bash
molecule converge -- --extra-vars cilium_setup_networking=install
```

Upgrading `Cilium` or changing parameters:

```bash
molecule converge -- --extra-vars cilium_action=upgrade
```

Deleting `Cilium` and its resources:

```bash
molecule converge -- --extra-vars cilium_action=delete
```

To run a few tests use (optionally add `-v` for more output):

```bash
molecule verify
```

To clean up run

```bash
molecule destroy
```

## License

GNU GENERAL PUBLIC LICENSE Version 3

## Author Information

[http://www.tauceti.blog](http://www.tauceti.blog)
