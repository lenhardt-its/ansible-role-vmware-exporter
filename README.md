# Ansible Role: vmware exporter

## Description

Deploy and manage prometheus [VMware exporter](https://github.com/pryorda/vmware_exporter) using ansible.

## Requirements

- Ansible >= 2.6 (It might work on previous versions, but we cannot guarantee it)

## Role Variables

All variables which can be overridden are stored in [defaults/main.yml](defaults/main.yml) file as well as in table below.

| Name           | Default Value | Description                        |
| -------------- | ------------- | -----------------------------------|
| `proxy_env` | {} | Proxy environment variables |
| `vmware_exporter_web_listen_port` | 9272 | Exporter port |
| `vmware_exporter_firewalld_state` | "disabled" | Enabled/Disabled Firewalld and open the port |
| `vmware_exporter_config_dir` | "/etc/vmware_exporter" | Configuration folder |
| `vmware_exporter_binary_local_dir` | "/usr/local/bin" | Exporter binary path |
| `vmware_exporter_create_consul_agent_service` | true | Add consul-agent service snipped |
| `vmware_exporter_system_user` | "{{ prometheus_user | default('prometheus') }}" | Exporter running user |
| `vmware_exporter_system_group` | "{{ prometheus_group | default('prometheus') }}" | Exporter running group |
| `vmware_exporter_targets` | "{}" | vCenter targets (see defaults/main.yml) |

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
    - ansible-role-snmp-exporter
```

## Contributing

See [contributor guideline](CONTRIBUTING.md).

## License

This project is licensed under MIT License. See [LICENSE](/LICENSE) for more details.

