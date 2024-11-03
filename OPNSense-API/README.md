
# OPNsense Firewall Rule Management Guide

## Introduction
This comprehensive guide explains how to create, modify, and manage firewall rules on OPNsense using Python, Postman, Ansible and `curl`. It includes detailed examples of JSON payloads, API calls, and configurations to simplify the process.

## Table of Contents
- [Python Script for Firewall Rules](#python-script-for-firewall-rules)
- [Using `curl` for API Calls](#using-curl-for-api-calls)
- [Configuring Postman](#configuring-postman)
- [Example JSON Payloads](#example-json-payloads)
- [Ansible Playbook Example](#ansible-playbook-example)
- [Conclusion](#conclusion)

## Python Script for Firewall Rules
Below is an example of a Python script that interacts with the OPNsense API to add a firewall rule.

```python
#!/usr/bin/env python3.7
import requests
import json

# Replace with your API key and secret
api_key = "YOUR_API_KEY"
api_secret = "YOUR_API_SECRET"

# Base URI for OPNsense
remote_uri = "https://192.168.1.1"
rule_description = 'OPNsense_fw_api_testrule_1'

# Search for an existing rule by description
response = requests.get(
    f"{remote_uri}/api/firewall/filter/searchRule?current=1&rowCount=7&searchPhrase={rule_description}",
    auth=(api_key, api_secret), verify=False
)

if response.status_code == 200:
    data = json.loads(response.text)
    if not data['rows']:
        # Create a new rule
        rule_data = {
            "rule": {
                "interface": "wan",
                "direction": "in",
                "source_net": "any",
                "destination_net": "192.168.2.0/24",
                "destination_port": "80",
                "protocol": "TCP",
                "description": "allow traffic on WAN to DMZ for port 80",
                "action": "pass",
                "enabled": True
            }
        }
        create_response = requests.post(
            f"{remote_uri}/api/firewall/filter/addRule",
            auth=(api_key, api_secret), verify=False, json=rule_data
        )
        if create_response.status_code == 200:
            print(f"Rule created: {json.loads(create_response.text)['uuid']}")
        else:
            print(f"Error: {create_response.text}")
    else:
        for row in data['rows']:
            print(f"Found existing rule UUID: {row['uuid']}")
else:
    print(f"Failed to search for rule: {response.status_code}")
```

## Using `curl` for API Calls
To make API calls using `curl`, you can use the following command to create a rule:

```bash
curl -k -u "API_KEY:API_SECRET" \
-X POST "https://192.168.1.1/api/firewall/filter/addRule" \
-H "Content-Type: application/json" \
-d '{
    "rule": {
        "interface": "wan",
        "direction": "in",
        "source_net": "any",
        "destination_net": "192.168.2.0/24",
        "destination_port": "80",
        "protocol": "TCP",
        "description": "allow traffic on WAN to DMZ for port 80",
        "action": "pass",
        "enabled": true
    }
}'
```

## Configuring Postman
To use Postman for managing OPNsense rules:

1. **Set the Method**: Select `POST` or `GET` as required.
2. **Enter the URL**: Example: `https://192.168.1.1/api/firewall/filter/addRule`.
3. **Authorization**: Use `Basic Auth` and input your `API_KEY` and `API_SECRET`.
4. **Headers**: Set `Content-Type` to `application/json`.
5. **Body**: Use raw input to include the JSON payload.

### Sample JSON Payloads

#### Allow Port 80 Traffic
```json
{
    "rule": {
        "interface": "wan",
        "direction": "in",
        "source_net": "any",
        "destination_net": "192.168.2.0/24",
        "destination_port": "80",
        "protocol": "TCP",
        "description": "allow traffic on WAN to DMZ for port 80",
        "action": "pass",
        "enabled": true
    }
}
```

#### Allow Port 22 Traffic
```json
{
    "rule": {
        "interface": "wan",
        "direction": "in",
        "source_net": "any",
        "destination_net": "192.168.2.0/24",
        "destination_port": "22",
        "protocol": "TCP",
        "description": "POSTMAN - allow traffic on WAN to DMZ for port 22",
        "quick": true,
        "sequence": 2,
        "ip_protocol": "inet",
        "action": "pass",
        "enabled": true
    }
}
```

## Ansible Playbook Example
For those using Ansible, below is an example to automate rule management:<br>
Ref.: https://github.com/ansibleguy/collection_opnsense

```yaml
---
- hosts: firewall
  connection: local
  gather_facts: no

  tasks:
    - name: Allow traffic on WAN to DMZ for port 80
      ansibleguy.opnsense.rule:
        source_net: 'any'
        destination_net: '192.168.2.0/24'
        destination_port: 80
        protocol: 'TCP'
        description: 'allow traffic on WAN to DMZ for port 80'
        interface: 'wan'
        direction: 'in'
        action: 'pass'
        state: 'present'
        enabled: true

    - name: Allow traffic on WAN to DMZ for port 22
      ansibleguy.opnsense.rule:
        source_net: 'any'
        destination_net: '192.168.2.0/24'
        destination_port: 22
        protocol: 'TCP'
        description: 'POSTMAN - allow traffic on WAN to DMZ for port 22'
        interface: 'wan'
        direction: 'in'
        quick: true
        sequence: 2
        ip_protocol: 'inet'
        action: 'pass'
        state: 'present'
        enabled: true
```

### Ansible Execution Output
```bash
PLAY [firewall] ********************************************************************

TASK [Allow traffic on WAN to DMZ for port 80] *************************************
changed: [192.168.1.1]

TASK [Allow traffic on WAN to DMZ for port 22] *************************************
changed: [192.168.1.1]

PLAY RECAP **************************************************************************
192.168.1.1                : ok=2    changed=2    unreachable=0    failed=0
```

## Conclusion
This guide provides a step-by-step process for managing firewall rules in OPNsense using Python, `curl`, Postman, and Ansible. Proper use of these tools ensures efficient and accurate rule management.
