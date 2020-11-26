Changelog
---------

**3.0.0+1.9.0**

- upgrade to Cilium v1.9.0
- refactor cilium_values_default.yml.j2 because re-scoped in Cilium v1.9 (see [1.9 Upgrade Notes](https://docs.cilium.io/en/v1.9/operations/upgrade/#upgrade-notes) and [values.yaml](https://github.com/cilium/cilium/blob/master/install/kubernetes/cilium/values.yaml))

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
