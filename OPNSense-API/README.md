
# OPNsense Firewall Rule Management with Python and Postman

## Introduction
This guide covers how to create, modify, and manage firewall rules on OPNsense using Python and Postman. It includes examples of JSON payloads, API calls, and configurations to facilitate rule creation.

## Table of Contents
- [Python Script for Firewall Rules](#python-script-for-firewall-rules)
- [Using `curl` for API Calls](#using-curl-for-api-calls)
- [Postman Setup](#postman-setup)
- [Example JSON Payloads](#example-json-payloads)

## Python Script for Firewall Rules
Below is an example Python script that interacts with the OPNsense API to add a firewall rule:

```python
#!/usr/bin/env python3.7
import requests
import json

# key + secret from downloaded apikey.txt
api_key = "YOUR_API_KEY"
api_secret = "YOUR_API_SECRET"

# Define the remote URI and rule description
rule_description = 'OPNsense_fw_api_testrule_1'
remote_uri = "https://192.168.1.1"

# Search for an existing rule
r = requests.get(
    f"{remote_uri}/api/firewall/filter/searchRule?current=1&rowCount=7&searchPhrase={rule_description}",
    auth=(api_key, api_secret), verify=False
)

if r.status_code == 200:
    response = json.loads(r.text)
    if len(response['rows']) == 0:
        # Create a new rule
        data = {
            "rule": {
                "description": rule_description,
                "source_net": "192.168.0.0/24",
                "protocol": "TCP",
                "destination_net": "10.0.0.0/24"
            }
        }
        r = requests.post(
            f"{remote_uri}/api/firewall/filter/addRule",
            auth=(api_key, api_secret), verify=False, json=data
        )
        if r.status_code == 200:
            print(f"Created: {json.loads(r.text)['uuid']}")
        else:
            print(f"Error: {r.text}")
    else:
        for row in response['rows']:
            print(f"Found UUID: {row['uuid']}")
```

## Using `curl` for API Calls
### Adding a Rule with `curl`:
Example `curl` command to add a rule from the `wan` interface to a specific destination network:

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
## Postman Setup
1. **Method**: Select `POST` or `GET` as needed.
2. **URL**: Example: `https://192.168.1.1/api/firewall/filter/addRule`.
3. **Authorization**: 
   - Choose `Basic Auth` and input the `API_KEY` and `API_SECRET`.
4. **Headers**:
   - Add `Content-Type: application/json`.
5. **Body**:
   - Use **raw** input for JSON payloads.

### Example JSON Payloads
#### Rule to Allow Port 80
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

#### Rule to Allow Port 22
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
        "action": "pass",
        "quick": true,
        "sequence": 2,
        "ip_protocol": "inet",
        "enabled": true
    }
}
```

## Conclusion
This guide helps set up and manage firewall rules in OPNsense using Python, `curl`, and Postman. Ensure the correct interface names and configurations are used for seamless integration.
