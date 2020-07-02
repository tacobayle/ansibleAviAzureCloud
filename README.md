# aviAzureCloud

## Goals
Configure Avi win Azure through ansible playbooks

## Prerequisites:
1. Make sure Ansible in installed in the orchestrator VM
2. Make sure avisdk is installed:
```
pip install avisdk
ansible-galaxy install -f avinetworks.avisdk
```

## Environment:
Playbook(s) has/have been tested against:

### Ansible

```
avi@ansible:~/ansible/aviSlackAlerts$ ansible --version
ansible 2.9.5
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/avi/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /home/avi/.local/lib/python2.7/site-packages/ansible
  executable location = /home/avi/.local/bin/ansible
  python version = 2.7.12 (default, Oct  8 2019, 14:14:10) [GCC 5.4.0 20160609]
avi@ansible:~/ansible/aviSlackAlerts$
```

### Avi version

```
Avi 18.2.8
```

## Input/Parameters:

- All the paramaters/variables are stored in var/params.yml.
```
---
avi_systemconfiguration:
  global_tenant_config:
    se_in_provider_context: false
    tenant_access_to_provider_se: true
    tenant_vrf: true
  welcome_workflow_complete: true
  ntp_configuration:
    ntp_servers:
      - server:
          type: V4
          addr: 192.168.10.3
  dns_configuration:
    search_domain: ''
    server_list:
      - type: V4
        addr: 8.8.8.8
      - type: V4
        addr: 4.4.4.4
  email_configuration:
    from_email: test@avicontroller.net
    smtp_type: SMTP_LOCAL_HOST

serviceEngineGroup:
  - name: Default-Group
    ha_mode: HA_MODE_SHARED
    min_scaleout_per_vs: 1
    buffer_se: 0

cloudconnectoruser:
  name: user-azure

avi_cloud:
  name: cloudAzure
  tenant_ref: admin
  vtype: CLOUD_AZURE
  resource_group: rg-avi
  azure_configuration:
    use_managed_disks: true
    use_enhanced_ha: false
    cloud_credentials_ref: user-azure
    use_azure_dns: true
    location: westeurope
    network_info:
      - se_network_id: subnet1
        virtual_network_id: "/subscriptions/{{ azure.subscriptionId }}/resourceGroups/rg-avi/providers/Microsoft.Network/virtualNetworks/vnet-avi"
    use_standard_alb: false

avi_healthmonitor:
  - name: hm1
    # tenant_ref: team1
    receive_timeout: 1
    failed_checks: 2
    send_interval: 1
    successful_checks: 2
    type: HEALTH_MONITOR_HTTP
    http_request: "HEAD / HTTP/1.0"
    http_response_code:
      - HTTP_2XX
      - HTTP_3XX
      - HTTP_5XX

avi_pool:
  - name: pool1
    lb_algorithm: LB_ALGORITHM_ROUND_ROBIN
    health_monitor_refs: hm1
    servers:
      - ip:
          addr: 172.16.2.4
          type: 'V4'
      - ip:
          addr: 172.16.2.5
          type: 'V4'
    external_autoscale_groups: false

virtualservice:
  - name: vs1
    # tenant_ref: team1
    port: 443
    enable_ssl: true
    pool_ref: pool1
    vip:
    - subnet:
      ipam_network_subnet:
        subnet_uuid: subnet3
        subnet:
          mask: 24
          ip_addr:
            type: V4
            addr: 172.16.3.0
      avi_allocated_fip: true
      auto_allocate_ip: true
      auto_allocate_floating_ip: true
```

- A credential file (vars/creds.json)is required:
```
{"avi_credentials": {"username": "admin", "controller": "172.16.1.5", "password": "Avi_2020", "api_version": "18.2.8"}}
```

## Use the ansible playbook to:
1. Configure Avi System Parameters
2. Configure SE Default SE Group
3. Configure Azure Cloud
4. Configure Health Monitoring
5. Configure pool
6. Configure Virtual Service

## Run the playbook:
ansible-playbook main.yml

## Improvement:
