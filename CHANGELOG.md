# Changelog

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
