Changelog
---------

**4.0.1+1.9.7**

- upgrade to Cilium v1.9.7

**4.0.0+1.9.1**

- introduce variables `cilium_release_name` and `cilium_repo_name`
- `cilium_chart_name` had the wrong value. The `cilium_chart_name` was actually the `cilium_release_name`. If you used the default `cilium_chart_name: "cilium"` nothing will change for you. Otherwise you may need `cilium_release_name`, `cilium_repo_name` and `cilium_chart_name` accordingly.

**3.0.0+1.9.1**

- upgrade to Cilium v1.9.1
- refactor `cilium_values_default.yml.j2` because re-scoped in Cilium v1.9 (see [1.9 Upgrade Notes](https://docs.cilium.io/en/v1.9/operations/upgrade/#upgrade-notes) and [values.yaml](https://github.com/cilium/cilium/blob/master/install/kubernetes/cilium/values.yaml)). Esp. value names like `agent.*`, `config.*` and `global.*` have changed.
- small changes in Helm values for pre-flight check
- rename extra vars `cilium_(install|upgrade|delete)=true` to `action=(install|upgrade|delete)`

**2.0.0+1.8.4**

- upgrade to Cilium v1.8.4

**2.0.0+1.8.1**

- upgrade to Cilium v1.8.1
- handle Cilium pre-flight check leftover
- add new options to `cilium_values_default.yml.j2` as recommended in upgrade guide
- `upgradeCompatibility` variable value needs to be string
- increase retries for pre-flight-check from 30 to 60

**1.0.1+1.7.4**

- formatting / make ansible-lint happy

**1.0.0+1.7.4**

- initial commit
