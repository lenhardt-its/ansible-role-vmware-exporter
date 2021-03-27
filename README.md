# Ansible Role: VMware Exporter

[![ubuntu-18](https://img.shields.io/badge/ubuntu-18.x-orange?style=flat&logo=ubuntu)](https://ubuntu.com/)
[![ubuntu-20](https://img.shields.io/badge/ubuntu-20.x-orange?style=flat&logo=ubuntu)](https://ubuntu.com/)
[![debian-9](https://img.shields.io/badge/debian-9.x-orange?style=flat&logo=debian)](https://www.debian.org/)
[![debian-10](https://img.shields.io/badge/debian-10.x-orange?style=flat&logo=debian)](https://www.debian.org/)
[![centos-7](https://img.shields.io/badge/centos-7.x-orange?style=flat&logo=centos)](https://www.centos.org/)
[![centos-8](https://img.shields.io/badge/centos-8.x-orange?style=flat&logo=centos)](https://www.centos.org/)

[![License](https://img.shields.io/badge/license-MIT%20License-brightgreen.svg?style=flat)](https://opensource.org/licenses/MIT)
[![GitHub issues](https://img.shields.io/github/issues/OnkelDom/ansible-role-vmware-exporter?style=flat)](https://github.com/OnkelDom/ansible-role-vmware-exporter/issues)
[![GitHub tag](https://img.shields.io/github/tag/OnkelDom/ansible-role-vmware-exporter.svg?style=flat)](https://github.com/OnkelDom/ansible-role-vmware-exporter/tags)
[![GitHub action](https://github.com/OnkelDom/ansible-role-vmware-exporter/workflows/ansible-lint/badge.svg)](https://github.com/OnkelDom/ansible-role-vmware-exporter)

## Description

Deploy and manage prometheus [VMware exporter](https://github.com/pryorda/vmware_exporter) using ansible.

## Requirements

- Ansible >= 2.9 (It might work on previous versions, but we cannot guarantee it)
- Community Packages: `ansible-galaxy collection install community.general`

## Role Variables

All variables which can be overridden are stored in [defaults/main.yml](defaults/main.yml) file as well as in table below.

| Name           | Default Value | Description                        |
| -------------- | ------------- | -----------------------------------|
| `proxy_env` | {} | Proxy environment variables |
| `vmware_exporter_web_listen_port` | 9272 | Exporter port |
| `vmware_exporter_allow_firewall` | false | Enabled/Disabled Firewalld and open the port |
| `vmware_exporter_config_dir` | /etc/vmware_exporter | Configuration folder |
| `vmware_exporter_binary_local_dir` | /usr/local/bin | Exporter binary path |
| `vmware_exporter_create_consul_agent_service` | true | Add consul-agent service snipped |
| `vmware_exporter_system_user` | prometheus | Exporter running user |
| `vmware_exporter_system_group` | prometheus | Exporter running group |
| `vmware_exporter_targets` | {} | vCenter targets (see defaults/main.yml) |

## Example Prometheus config
```yaml
# You can use the following parameters in the Prometheus configuration file. The params section is used to manage multiple login/passwords.
- job_name: 'vmware_vcenter'
  metrics_path: '/metrics'
  static_configs:
    - targets:
      - 'vcenter.company.com'
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: localhost:9272

- job_name: 'vmware_esx'
  metrics_path: '/metrics'
  file_sd_configs:
    - files:
      - /etc/prometheus/esx.yml
  params:
    section: [esx]
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: localhost:9272

# Example of Multiple vCenter usage per #23
- job_name: vmware_export
    metrics_path: /metrics
    static_configs:
    - targets:
      - vcenter01
      - vcenter02
      - vcenter03
    relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: exporter_ip:9272
```

### Playbook

```yaml
- hosts: all
  become: yes
  roles:
    - onkeldom.vmware_exporter
```

## Contributing

See [contributor guideline](CONTRIBUTING.md).

## License

This project is licensed under MIT License. See [LICENSE](/LICENSE) for more details.

