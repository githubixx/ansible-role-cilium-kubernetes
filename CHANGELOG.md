# Changelog

## 14.0.1+1.16.5

- **Update**
  - upgrade to Cilium `v1.16.5`

## 14.0.0+1.16.2

**NOTE:** Upgrading from Cilium `1.15.x` to `1.16.x` is a major release upgrade! Please read the [1.16 Upgrade Notes](https://docs.cilium.io/en/v1.16/operations/upgrade/#current-release-required-changes) carefully and adjust your settings accordingly!

All new features are explained in the Cilium `1.16` announcement blog: [Cilium 1.16 – High-Performance Networking With Netkit, Gateway API Gamma Support, BGPV2 and More!](https://isovalent.com/blog/post/cilium-1-16/).

In general it makes sense to update to the latest Cilium `1.15.x` version first before upgrading to `0.16.x`. If you've used the default (or slightly adjusted) settings that this Ansible role provides then the upgrade should be pretty straight forward.

- **Update**
  - upgrade to Cilium `v1.16.2`

- **Other**
  - update `.yamllint`

- **Molecule**
  - add a few more checks in `verify.yml`

## 13.1.0+1.15.8

- **Update**
  - upgrade to Cilium `v1.15.8`

- **Molecule**
  - replace Vagrant `alvistack/ubuntu-22.04` boxes with `alvistack/ubuntu-24.04`

## 13.0.0+1.15.3

- **Breaking**
  - changes in `templates/cilium_values_default.yml.j2`: added `kubeProxyReplacement`, `nodePort` and `socketLB` (this is needed because BPF masquerade requires NodePort)

- **Update**
  - upgrade to Cilium `v1.15.3`

- **Molecule**
  - replace Vagrant `generic/ubuntu2204` boxes with `alvistack/ubuntu-22.04`

## 12.0.0+1.15.0

- upgrade to Cilium `v1.15.0`
- refactor Molecule setup
- introduce `cilium_chart_values_directory` variable

## 11.0.6+1.14.5

- fix Github action

## 11.0.5+1.14.5

- upgrade to Cilium `v1.14.5`

## 11.0.4+1.14.4

- upgrade to Cilium `v1.14.4`

## 11.0.3+1.14.3

- upgrade to Cilium `v1.14.3`

## 11.0.2+1.14.1

- rename `githubixx.harden-linux` to `githubixx.harden_linux`

## 11.0.1+1.14.1

- rename `githubixx.kubernetes-ca` to `githubixx.kubernetes_ca`

## 11.0.0+1.14.1

- upgrade to Cilium `v1.14.1`
- refactor Molecule tests

## 10.0.2+1.13.4

- upgrade to Cilium `v1.13.4`

## 10.0.1+1.13.2

- upgrade to Cilium `v1.13.2`

## 10.0.0+1.13.0

- upgrade to Cilium `v1.13.0`. Please check [](https://docs.cilium.io/en/v1.13/operations/upgrade/#current-release-required-changes) for possible incompatible changes and deprecations.
- added `force:true` in case of Helm chart upgrade to make sure the upgrade happens no matter what
- `tasks/pre_flight_check.yml` + `upgrade.yml`: fix ansible-lint issues

## 9.0.0+1.12.5

- **BREAKING**: The `action` variable was renamed to `cilium_action` variable. This avoids variable collisions with other roles.
- **BREAKING**: Allow to set Helm values for pre-flight-check in `cilium_values_user_pre_flight_check.yml.j2` or `cilium_values_default_pre_flight_check.yml.j2` templates. In previous versions these values were hard-coded in `tasks/upgrade.yml`. This change comes together with the next one:
- **BREAKING**: In previous versions the Helm values for pre-flight-check were:

```yaml
agent: false
preflight:
  enabled: true
operator:
  enabled: false
```

This was changed to:

```yaml
agent: false
preflight:
  enabled: true
  tolerations:
    - operator: Exists
operator:
  enabled: false
```

By default Cilium gets installed on all nodes including the control plane nodes. That's because the `toleration` is set to `operator: Exists` which ignores all taints set on all Kubernetes nodes. Since the "pre-flight-check" expects that the number of current Cilium pods equals the number of "pre-flight-check" pods, the "pre-flight-check" pods needs to run on all nodes where the current Cilium pods are running. The default Helm chart values disallow running "pre-flight-check" pods running on the control-plane nodes. If you've different taints you might need to adjust the "pre-flight-check" tolerations.

- upgrade to Cilium `v1.12.5`
- added `.gitignore`
- added `.yamlint`

## 8.0.0+1.12.3

- upgrade to Cilium `v1.12.3`
- introduce `cilium_delegate_to` variable. Previously this was hard coded to `127.0.0.1` and it's also the default of this variable.
- introduce `cilium_helm_show_commands` variable (see README for more information)
- introduce `cilium_template_output_directory` variable (see README for more information)
- introduce `Molecule` tests
- don't use `shell` module anymore to execute `helm` command. Now `kubernetes.core.helm*` modules are used.
- use two underscores for internal variables
- ansible-lint: fix various issues like using FQDN Ansible module names now
- add `requirements.yml`
- add `collections.yml`

## 7.1.1+1.12.1

- fix YAML syntax in `tasks/main.yml`
- use FQN Ansible module names for `include_tasks`

## 7.1.0+1.12.1

- fix various ansible-lint issues

## 7.0.0+1.12.1

- upgrade to Cilium `v1.12.1`
- add Github release action to push new release to Ansible Galaxy

## 6.0.4+1.11.6

- upgrade to Cilium `v1.11.6`

## 6.0.3+1.11.5

- upgrade to Cilium `v1.11.5`

## 6.0.2+1.11.4

- upgrade to Cilium `v1.11.4`

## 6.0.1+1.11.1

- upgrade to Cilium `v1.11.1`
- make sure Cilium Helm chart is installed before rendering (contribution by @tiagoblackcode)

## 6.0.0+1.11.0

- upgrade to Cilium `v1.11.0`
- remove unneeded directories
- fix some linting issues

## 5.1.0+1.10.4

- upgrade to Cilium v1.10.4
- add `bpf.masquerade: true` option to enable native IP masquerade support in eBPF. The eBPF-based masquerading implementation is the most efficient implementation. It requires Linux kernel >= 4.19. See: [Cilium Masquerading](https://docs.cilium.io/en/stable/concepts/networking/masquerading/)

## 5.0.0+1.10.1

- upgrade to Cilium v1.10.1

## 4.0.1+1.9.7

- upgrade to Cilium v1.9.7

## 4.0.0+1.9.1

- introduce variables `cilium_release_name` and `cilium_repo_name`
- `cilium_chart_name` had the wrong value. The `cilium_chart_name` was actually the `cilium_release_name`. If you used the default `cilium_chart_name: "cilium"` nothing will change for you. Otherwise you may need `cilium_release_name`, `cilium_repo_name` and `cilium_chart_name` accordingly.

## 3.0.0+1.9.1

- upgrade to Cilium v1.9.1
- refactor `cilium_values_default.yml.j2` because re-scoped in Cilium v1.9 (see [1.9 Upgrade Notes](https://docs.cilium.io/en/v1.9/operations/upgrade/#upgrade-notes) and [values.yaml](https://github.com/cilium/cilium/blob/master/install/kubernetes/cilium/values.yaml)). Esp. value names like `agent.*`, `config.*` and `global.*` have changed.
- small changes in Helm values for pre-flight check
- rename extra vars `cilium_(install|upgrade|delete)=true` to `action=(install|upgrade|delete)`

## 2.0.0+1.8.4

- upgrade to Cilium v1.8.4

## 2.0.0+1.8.1

- upgrade to Cilium v1.8.1
- handle Cilium pre-flight check leftover
- add new options to `cilium_values_default.yml.j2` as recommended in upgrade guide
- `upgradeCompatibility` variable value needs to be string
- increase retries for pre-flight-check from 30 to 60

## 1.0.1+1.7.4

- formatting / make ansible-lint happy

## 1.0.0+1.7.4

- initial commit
