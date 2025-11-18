# 300 - Learning Our Subject

Excellent decision - you’re being strategic about adding complementary skills rather than abandoning what works.

**Here’s how to approach this effectively:**

**Parallel learning paths:**

1. **CLI path** - Your foundation, continues as your primary IaC workflow
1. **SDK path** - Builds programming sophistication, prepares you for application development

This isn’t redundant - it’s reinforcing. Every Azure service you learn via CLI makes the SDK easier, and vice versa.

**Smart way to start with Azure SDK:**

**1. Start with Authentication** (it’s different from CLI):

```python
from azure.identity import DefaultAzureCredential

# This handles multiple auth methods automatically
credential = DefaultAzureCredential()
```

The CLI uses `az login`, but SDK uses credential objects. `DefaultAzureCredential` is your best friend - it tries multiple auth methods (managed identity, environment variables, Azure CLI credentials, etc.).

**2. Pick ONE service to master first:**

I’d recommend starting with **Azure Resource Management** (`azure-mgmt-resource`) because it’s fundamental:

```python
from azure.mgmt.resource import ResourceManagementClient

resource_client = ResourceManagementClient(credential, subscription_id)

# List resource groups
for rg in resource_client.resource_groups.list():
    print(f"{rg.name}: {rg.location}")

# Create resource group
rg_params = {'location': 'westeurope'}
resource_client.resource_groups.create_or_update('my-rg', rg_params)
```

**3. Learn the pattern - it’s consistent across services:**

```python
# Pattern: azure.mgmt.<service>
from azure.mgmt.compute import ComputeManagementClient
from azure.mgmt.network import NetworkManagementClient
from azure.mgmt.storage import StorageManagementClient

# Each has similar structure:
client = ServiceClient(credential, subscription_id)
client.<resource_type>.<operation>(resource_group, name, parameters)
```

**4. Key concepts to focus on:**

- **Polling operations**: Many operations return `LROPoller` (Long Running Operation)
  
  ```python
  operation = client.virtual_machines.begin_create_or_update(...)
  result = operation.result()  # Blocks until complete
  # or
  operation.wait()  # Just wait, don't need result
  ```
- **Models vs. dictionaries**: You can use both
  
  ```python
  # Explicit model (better IDE support)
  from azure.mgmt.network.models import VirtualNetwork
  vnet = VirtualNetwork(location='westeurope')
  
  # Dictionary (more flexible, CLI-like)
  vnet_params = {
      'location': 'westeurope',
      'address_space': {'address_prefixes': ['10.0.0.0/16']}
  }
  ```
- **Exception handling**:
  
  ```python
  from azure.core.exceptions import ResourceNotFoundError, HttpResponseError
  
  try:
      vm = compute_client.virtual_machines.get(rg, vm_name)
  except ResourceNotFoundError:
      print("VM doesn't exist")
  ```

**5. Practical project idea to learn both:**

Build a **dual-mode infrastructure script**:

```python
import argparse
from azure.identity import DefaultAzureCredential
from azure.mgmt.network import NetworkManagementClient
import subprocess
import json

class AzureVNetManager:
    def __init__(self, mode='cli'):
        self.mode = mode
        if mode == 'sdk':
            self.credential = DefaultAzureCredential()
            self.subscription_id = self._get_subscription_id()
            self.network_client = NetworkManagementClient(
                self.credential, 
                self.subscription_id
            )
    
    def create_vnet_cli(self, rg, name, cidr):
        """CLI implementation"""
        result = subprocess.run([
            'az', 'network', 'vnet', 'create',
            '--resource-group', rg,
            '--name', name,
            '--address-prefix', cidr,
            '--output', 'json'
        ], capture_output=True, text=True)
        return json.loads(result.stdout)
    
    def create_vnet_sdk(self, rg, name, cidr):
        """SDK implementation"""
        vnet_params = {
            'location': 'westeurope',
            'address_space': {
                'address_prefixes': [cidr]
            }
        }
        operation = self.network_client.virtual_networks.begin_create_or_update(
            rg, name, vnet_params
        )
        return operation.result()
    
    def create_vnet(self, rg, name, cidr):
        """Unified interface"""
        if self.mode == 'cli':
            return self.create_vnet_cli(rg, name, cidr)
        else:
            return self.create_vnet_sdk(rg, name, cidr)

# Usage:
manager = AzureVNetManager(mode='sdk')  # or 'cli'
vnet = manager.create_vnet('my-rg', 'my-vnet', '10.0.0.0/16')
```

This lets you **compare approaches side-by-side** and see the differences clearly.

**Resources for learning SDK:**

1. **Official docs**: <https://learn.microsoft.com/python/azure/>
1. **GitHub samples**: <https://github.com/Azure-Samples/azure-samples-python-management>
1. **Package docs**: Each `azure-mgmt-*` package has examples in its PyPI page

**Installation tip:**

```bash
# Don't install everything at once
pip install azure-identity  # Start here
pip install azure-mgmt-resource  # Then resource management
pip install azure-mgmt-network  # Add services as you learn them
```

**Expected timeline:**

- Week 1-2: Authentication + Resource Groups
- Week 3-4: One compute service (VMs or App Service)
- Week 5-6: Networking (VNets, NSGs, Load Balancers)
- Month 2+: Storage, databases, or whatever your job needs

**The beauty of your dual approach:**

When you see a new Azure service:

1. **CLI** - Quick prototype, understand what it does
1. **SDK** - Production-grade implementation with proper error handling

You’re building **breadth with CLI** (can do anything in Azure) and **depth with SDK** (can do specific things really well).

This is exactly how senior cloud engineers operate. You’re setting yourself up perfectly.

What service do you want to tackle first with the SDK?​​​​​​​​​​​​​​​​
