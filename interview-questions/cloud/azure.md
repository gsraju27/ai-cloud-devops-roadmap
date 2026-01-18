# Complete Microsoft Azure Interview Guide

Master Azure with 12 real-world interview questions covering identity, networking, AKS, serverless, data services, and DevOps. Practice scenarios that mirror actual senior cloud engineer challenges.

**Companies that ask these questions:** Microsoft | Accenture | Deloitte | PwC | Boeing | Walmart

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | Users can't access an application after Entra ID configurati... | Debugging | Identity & Security |
| 2 | Design a hub-spoke VNet architecture with private endpoints ... | Architecture | Networking |
| 3 | AKS pods can't pull images and nodes are NotReady. Diagnose ... | Debugging | AKS |
| 4 | Azure Storage costs are $30K/month. Implement tiering and li... | Practical | Storage |
| 5 | Azure Functions have 10-second cold starts causing API timeo... | Debugging | Serverless |
| 6 | Cosmos DB is consuming 50,000 RU/s and costing $3K/day. Opti... | Practical | Data Services |
| 7 | Azure SQL failover takes too long and connections drop. Impl... | Practical | Databases |
| 8 | Messages are lost and processed out of order in Service Bus.... | Practical | Messaging |
| 9 | Design a global application with Azure Front Door for traffi... | Architecture | Traffic Management |
| 10 | Build a monitoring solution with Azure Monitor that catches ... | Practical | Observability |
| 11 | Implement a CI/CD pipeline with Azure DevOps that deploys to... | Practical | DevOps |
| 12 | Your Azure bill increased 50% last month. Identify waste and... | Practical | Cost Optimization |

---

## What You'll Learn

- Debug Entra ID authentication and RBAC permission issues
- Design VNet architecture with private endpoints and hybrid connectivity
- Troubleshoot AKS cluster and workload failures
- Optimize Azure Storage costs with lifecycle management
- Fix Azure Functions cold starts and scaling issues
- Optimize Cosmos DB queries and reduce RU consumption
- Configure Azure SQL for high availability with failover groups
- Implement reliable messaging with Service Bus
- Design global traffic management with Front Door
- Build comprehensive monitoring with Azure Monitor
- Implement CI/CD pipelines with Azure DevOps
- Optimize Azure costs with reservations and policies

---

## Interview Questions

### Question 1: Users can't access an application after Entra ID configuration. Debug the authentication failure.

**Type:** Debugging | **Category:** Identity & Security

## The Scenario

After configuring Entra ID (formerly Azure AD) for a new web application, users see:

```
AADSTS700016: Application with identifier 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
was not found in the directory 'contoso.com'. This can happen if the application
has not been installed by the administrator of the tenant or consented to by
any user in the tenant.
```

Some users get a different error:
```
AADSTS65001: The user or administrator has not consented to use the application
with ID 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' named 'MyApp'.
```

## The Challenge

Debug the authentication flow, fix app registration issues, and implement proper consent and permission configuration.


### Step 1: Verify App Registration Exists

```bash
# Check if app registration exists
az ad app list --display-name "MyApp" --query "[].{Name:displayName,AppId:appId,Id:id}"

# If using service principal
az ad sp list --display-name "MyApp" --query "[].{Name:displayName,AppId:appId,Id:id}"

# Get detailed app registration info
az ad app show --id "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

### Step 2: Common Configuration Issues

```bash
# Check redirect URIs
az ad app show --id $APP_ID --query "web.redirectUris"

# Common mistakes:
# - HTTP instead of HTTPS (except localhost)
# - Missing trailing slash
# - Wrong port number
# - Case sensitivity issues

# Update redirect URIs
az ad app update --id $APP_ID \
  --web-redirect-uris "https://myapp.azurewebsites.net/signin-oidc" \
                       "https://myapp.azurewebsites.net/.auth/login/aad/callback"
```

### Step 3: Check API Permissions and Consent

```bash
# List required permissions
az ad app permission list --id $APP_ID

# Check granted permissions (admin consent)
az ad app permission list-grants --id $APP_ID

# Grant admin consent for Microsoft Graph permissions
az ad app permission admin-consent --id $APP_ID
```

### Step 4: Understanding Permission Types

```bicep
// Bicep template for app registration with proper permissions
resource appRegistration 'Microsoft.Graph/applications@v1.0' = {
  displayName: 'MyApp'
  signInAudience: 'AzureADMyOrg'  // Single tenant

  web: {
    redirectUris: [
      'https://myapp.azurewebsites.net/signin-oidc'
    ]
    implicitGrantSettings: {
      enableIdTokenIssuance: true
      enableAccessTokenIssuance: false  // Use auth code flow instead
    }
  }

  requiredResourceAccess: [
    {
      // Microsoft Graph
      resourceAppId: '00000003-0000-0000-c000-000000000000'
      resourceAccess: [
        {
          // User.Read - Delegated
          id: 'e1fe6dd8-ba31-4d61-89e7-88639da4683d'
          type: 'Scope'
        }
        {
          // Mail.Read - Delegated (requires admin consent)
          id: '570282fd-fa5c-430d-a7fd-fc8dc98a9dca'
          type: 'Scope'
        }
      ]
    }
  ]
}
```

### Step 5: Configure for Different Scenarios

**Single Tenant (Enterprise App):**
```json
{
  "signInAudience": "AzureADMyOrg",
  "accessTokenAcceptedVersion": 2
}
```

**Multi-Tenant (SaaS):**
```json
{
  "signInAudience": "AzureADMultipleOrgs",
  "accessTokenAcceptedVersion": 2
}
```

**B2C Scenario:**
```json
{
  "signInAudience": "AzureADandPersonalMicrosoftAccount",
  "accessTokenAcceptedVersion": 2
}
```

### Step 6: Implement Managed Identity (Best Practice)

```bicep
// Use Managed Identity to eliminate client secrets
resource webApp 'Microsoft.Web/sites@2022-03-01' = {
  name: 'myapp'
  location: location
  identity: {
    type: 'SystemAssigned'
  }
  properties: {
    // ...
  }
}

// Grant the managed identity access to Key Vault
resource keyVaultAccess 'Microsoft.KeyVault/vaults/accessPolicies@2022-07-01' = {
  parent: keyVault
  name: 'add'
  properties: {
    accessPolicies: [
      {
        tenantId: subscription().tenantId
        objectId: webApp.identity.principalId
        permissions: {
          secrets: ['get', 'list']
        }
      }
    ]
  }
}
```

```csharp
// C# code using Managed Identity
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

// No credentials needed - uses Managed Identity automatically
var client = new SecretClient(
    new Uri("https://myvault.vault.azure.net/"),
    new DefaultAzureCredential()
);

var secret = await client.GetSecretAsync("MySecret");
```

### Step 7: Debug Token Issues

```bash
# Decode and inspect JWT token
# Use https://jwt.ms or jwt.io

# Check token claims:
# - aud (audience) - must match your app ID
# - iss (issuer) - must be your tenant
# - exp (expiration) - must be in the future
# - scp or roles - must include required permissions

# Enable verbose logging in your app
export AZURE_LOG_LEVEL=verbose
```

### Step 8: Application Insights for Auth Debugging

```csharp
// Add detailed authentication logging
services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApp(options =>
    {
        Configuration.Bind("AzureAd", options);

        options.Events = new OpenIdConnectEvents
        {
            OnAuthenticationFailed = context =>
            {
                _logger.LogError(context.Exception,
                    "Authentication failed: {Message}",
                    context.Exception.Message);
                return Task.CompletedTask;
            },
            OnTokenValidated = context =>
            {
                _logger.LogInformation(
                    "Token validated for user: {User}",
                    context.Principal?.Identity?.Name);
                return Task.CompletedTask;
            }
        };
    });
```


## Common AADSTS Error Codes

| Error Code | Cause | Fix |
|------------|-------|-----|
| AADSTS700016 | App not found | Check app ID and tenant |
| AADSTS65001 | Consent required | Admin consent or user consent |
| AADSTS50011 | Redirect URI mismatch | Update redirect URIs |
| AADSTS7000218 | Missing client_assertion | Check client secret/cert |
| AADSTS90014 | Missing required field | Check request parameters |

## Authentication Flow Comparison

| Flow | Use Case | Token Location |
|------|----------|----------------|
| Auth Code | Web apps (server-side) | Backend only |
| Auth Code + PKCE | SPAs, mobile apps | Secure storage |
| Client Credentials | Service-to-service | Backend only |
| Implicit (deprecated) | Legacy SPAs | Don't use |

---

### Question 2: Design a hub-spoke VNet architecture with private endpoints and on-premises connectivity.

**Type:** Architecture | **Category:** Networking

## The Scenario

Your enterprise needs to migrate workloads to Azure while:
- Maintaining connectivity to on-premises data centers
- Keeping all traffic private (no public endpoints)
- Isolating workloads across multiple subscriptions
- Enabling shared services (DNS, firewalls, monitoring)
- Meeting compliance requirements for data residency

Current state: Each team has their own VNet with no connectivity, using public endpoints.

## The Challenge

Design a hub-spoke network architecture using Azure networking services: VNet peering, Private Link, ExpressRoute, and Azure Firewall.


### Architecture Overview

```
                    On-Premises Data Center
                            │
                            │ ExpressRoute / VPN
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Hub VNet (10.0.0.0/16)                       │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │  Gateway Subnet │  │  Firewall Subnet│  │  Bastion Subnet │ │
│  │  10.0.0.0/24    │  │  10.0.1.0/24    │  │  10.0.2.0/24    │ │
│  └────────┬────────┘  └────────┬────────┘  └─────────────────┘ │
│           │                    │                                 │
│           │           ┌────────┴────────┐                       │
│           │           │  Azure Firewall │                       │
│           │           └────────┬────────┘                       │
└───────────┼────────────────────┼────────────────────────────────┘
            │                    │
     ┌──────┴──────┐      ┌──────┴──────┐
     │   Peering   │      │   Peering   │
     ▼             ▼      ▼             ▼
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│ Spoke 1  │  │ Spoke 2  │  │ Spoke 3  │  │ Spoke 4  │
│  Prod    │  │   Dev    │  │ Shared   │  │   DMZ    │
│10.1.0/16 │  │10.2.0/16 │  │10.3.0/16 │  │10.4.0/16 │
└──────────┘  └──────────┘  └──────────┘  └──────────┘
```

### Step 1: Create Hub VNet with Subnets

```bicep
// hub-vnet.bicep
param location string = resourceGroup().location

resource hubVnet 'Microsoft.Network/virtualNetworks@2023-05-01' = {
  name: 'hub-vnet'
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: ['10.0.0.0/16']
    }
    subnets: [
      {
        name: 'GatewaySubnet'  // Must be named exactly this
        properties: {
          addressPrefix: '10.0.0.0/24'
        }
      }
      {
        name: 'AzureFirewallSubnet'  // Must be named exactly this
        properties: {
          addressPrefix: '10.0.1.0/24'
        }
      }
      {
        name: 'AzureBastionSubnet'  // Must be named exactly this
        properties: {
          addressPrefix: '10.0.2.0/24'
        }
      }
      {
        name: 'SharedServicesSubnet'
        properties: {
          addressPrefix: '10.0.3.0/24'
        }
      }
    ]
  }
}

// Azure Firewall
resource firewall 'Microsoft.Network/azureFirewalls@2023-05-01' = {
  name: 'hub-firewall'
  location: location
  properties: {
    sku: {
      name: 'AZFW_VNet'
      tier: 'Standard'
    }
    ipConfigurations: [
      {
        name: 'fw-ipconfig'
        properties: {
          subnet: {
            id: '${hubVnet.id}/subnets/AzureFirewallSubnet'
          }
          publicIPAddress: {
            id: firewallPublicIP.id
          }
        }
      }
    ]
    firewallPolicy: {
      id: firewallPolicy.id
    }
  }
}

// Firewall Policy
resource firewallPolicy 'Microsoft.Network/firewallPolicies@2023-05-01' = {
  name: 'hub-firewall-policy'
  location: location
  properties: {
    sku: {
      tier: 'Standard'
    }
    threatIntelMode: 'Alert'
    dnsSettings: {
      enableProxy: true  // Required for FQDN rules
    }
  }
}
```

### Step 2: Create Spoke VNets with Peering

```bicep
// spoke-vnet.bicep
param spokeName string
param spokeAddressPrefix string
param hubVnetId string
param firewallPrivateIP string

resource spokeVnet 'Microsoft.Network/virtualNetworks@2023-05-01' = {
  name: '${spokeName}-vnet'
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: [spokeAddressPrefix]
    }
    subnets: [
      {
        name: 'default'
        properties: {
          addressPrefix: cidrSubnet(spokeAddressPrefix, 24, 0)
          routeTable: {
            id: spokeRouteTable.id
          }
          networkSecurityGroup: {
            id: spokeNSG.id
          }
        }
      }
      {
        name: 'PrivateEndpoints'
        properties: {
          addressPrefix: cidrSubnet(spokeAddressPrefix, 24, 1)
          privateEndpointNetworkPolicies: 'Disabled'
        }
      }
    ]
  }
}

// Peering: Spoke to Hub
resource spokeToHubPeering 'Microsoft.Network/virtualNetworks/virtualNetworkPeerings@2023-05-01' = {
  parent: spokeVnet
  name: 'spoke-to-hub'
  properties: {
    remoteVirtualNetwork: {
      id: hubVnetId
    }
    allowVirtualNetworkAccess: true
    allowForwardedTraffic: true
    useRemoteGateways: true  // Use hub's gateway
  }
}

// Route table to force traffic through firewall
resource spokeRouteTable 'Microsoft.Network/routeTables@2023-05-01' = {
  name: '${spokeName}-rt'
  location: location
  properties: {
    routes: [
      {
        name: 'to-internet'
        properties: {
          addressPrefix: '0.0.0.0/0'
          nextHopType: 'VirtualAppliance'
          nextHopIpAddress: firewallPrivateIP
        }
      }
      {
        name: 'to-onprem'
        properties: {
          addressPrefix: '192.168.0.0/16'  // On-prem range
          nextHopType: 'VirtualAppliance'
          nextHopIpAddress: firewallPrivateIP
        }
      }
    ]
  }
}
```

### Step 3: Configure ExpressRoute for On-Premises

```bicep
// ExpressRoute Gateway
resource expressRouteGateway 'Microsoft.Network/virtualNetworkGateways@2023-05-01' = {
  name: 'hub-expressroute-gw'
  location: location
  properties: {
    gatewayType: 'ExpressRoute'
    sku: {
      name: 'ErGw2AZ'  // Zone-redundant
      tier: 'ErGw2AZ'
    }
    ipConfigurations: [
      {
        name: 'gwipconfig'
        properties: {
          subnet: {
            id: '${hubVnet.id}/subnets/GatewaySubnet'
          }
          publicIPAddress: {
            id: gatewayPublicIP.id
          }
        }
      }
    ]
  }
}

// ExpressRoute Connection
resource expressRouteConnection 'Microsoft.Network/connections@2023-05-01' = {
  name: 'hub-to-onprem'
  location: location
  properties: {
    connectionType: 'ExpressRoute'
    virtualNetworkGateway1: {
      id: expressRouteGateway.id
    }
    peer: {
      id: expressRouteCircuit.id
    }
    routingWeight: 0
  }
}
```

### Step 4: Private Endpoints and DNS

```bicep
// Private endpoint for Azure SQL
resource sqlPrivateEndpoint 'Microsoft.Network/privateEndpoints@2023-05-01' = {
  name: 'sql-private-endpoint'
  location: location
  properties: {
    subnet: {
      id: '${spokeVnet.id}/subnets/PrivateEndpoints'
    }
    privateLinkServiceConnections: [
      {
        name: 'sql-connection'
        properties: {
          privateLinkServiceId: sqlServer.id
          groupIds: ['sqlServer']
        }
      }
    ]
  }
}

// Private DNS Zone
resource privateDnsZone 'Microsoft.Network/privateDnsZones@2020-06-01' = {
  name: 'privatelink.database.windows.net'
  location: 'global'
}

// Link DNS zone to VNets
resource hubDnsLink 'Microsoft.Network/privateDnsZones/virtualNetworkLinks@2020-06-01' = {
  parent: privateDnsZone
  name: 'hub-link'
  location: 'global'
  properties: {
    virtualNetwork: {
      id: hubVnet.id
    }
    registrationEnabled: false
  }
}

// DNS A record for private endpoint
resource sqlDnsRecord 'Microsoft.Network/privateDnsZones/A@2020-06-01' = {
  parent: privateDnsZone
  name: sqlServer.name
  properties: {
    ttl: 300
    aRecords: [
      {
        ipv4Address: sqlPrivateEndpoint.properties.customDnsConfigs[0].ipAddresses[0]
      }
    ]
  }
}
```

### Step 5: Azure Firewall Rules

```bicep
resource firewallRuleCollection 'Microsoft.Network/firewallPolicies/ruleCollectionGroups@2023-05-01' = {
  parent: firewallPolicy
  name: 'DefaultRuleCollectionGroup'
  properties: {
    priority: 100
    ruleCollections: [
      {
        ruleCollectionType: 'FirewallPolicyFilterRuleCollection'
        name: 'AllowAzureServices'
        priority: 100
        action: {
          type: 'Allow'
        }
        rules: [
          {
            ruleType: 'ApplicationRule'
            name: 'AllowAzureMonitor'
            protocols: [
              { protocolType: 'Https', port: 443 }
            ]
            targetFqdns: [
              '*.monitor.azure.com'
              '*.ods.opinsights.azure.com'
              '*.oms.opinsights.azure.com'
            ]
            sourceAddresses: ['10.0.0.0/8']
          }
        ]
      }
      {
        ruleCollectionType: 'FirewallPolicyFilterRuleCollection'
        name: 'AllowInternet'
        priority: 200
        action: {
          type: 'Allow'
        }
        rules: [
          {
            ruleType: 'ApplicationRule'
            name: 'AllowWindowsUpdate'
            protocols: [
              { protocolType: 'Https', port: 443 }
            ]
            targetFqdns: [
              '*.windowsupdate.com'
              '*.microsoft.com'
            ]
            sourceAddresses: ['10.0.0.0/8']
          }
        ]
      }
    ]
  }
}
```

### Step 6: Network Security Groups

```bicep
resource spokeNSG 'Microsoft.Network/networkSecurityGroups@2023-05-01' = {
  name: '${spokeName}-nsg'
  location: location
  properties: {
    securityRules: [
      {
        name: 'AllowHTTPSInbound'
        properties: {
          priority: 100
          direction: 'Inbound'
          access: 'Allow'
          protocol: 'Tcp'
          sourceAddressPrefix: 'VirtualNetwork'
          sourcePortRange: '*'
          destinationAddressPrefix: '*'
          destinationPortRange: '443'
        }
      }
      {
        name: 'DenyAllInbound'
        properties: {
          priority: 4096
          direction: 'Inbound'
          access: 'Deny'
          protocol: '*'
          sourceAddressPrefix: '*'
          sourcePortRange: '*'
          destinationAddressPrefix: '*'
          destinationPortRange: '*'
        }
      }
    ]
  }
}
```


## Hub-Spoke Design Decisions

| Decision | Option A | Option B | Recommendation |
|----------|----------|----------|----------------|
| Connectivity | VNet Peering | Virtual WAN | Peering for under 50 VNets, vWAN for global |
| Firewall | Azure Firewall | NVA | Azure Firewall for most cases |
| DNS | Azure DNS Private | Custom DNS | Azure Private DNS Zones |
| Gateway | Per-spoke | Shared in hub | Shared (cost efficient) |

## Private Endpoint DNS Zones

| Service | DNS Zone |
|---------|----------|
| Azure SQL | privatelink.database.windows.net |
| Storage Blob | privatelink.blob.core.windows.net |
| Key Vault | privatelink.vaultcore.azure.net |
| Cosmos DB | privatelink.documents.azure.com |
| App Service | privatelink.azurewebsites.net |


---

### Quick Check

**Why must you configure Private DNS Zones when using Private Endpoints?**

   A. Private Endpoints don't work without DNS Zones
-> B. **DNS resolution must return the private IP instead of the public IP for apps to connect via the private path**
   C. Azure requires DNS Zones for billing purposes
   D. Private DNS Zones provide encryption for the connection

<details>
<summary>See Answer</summary>

When you create a Private Endpoint, the Azure service still has its public FQDN (e.g., mydb.database.windows.net). By default, DNS resolves to the public IP. To use the private endpoint, DNS must resolve to the private IP. Private DNS Zones with A records for the private endpoint IPs achieve this. Without proper DNS, your app would still connect via the public internet.

</details>

---

### Question 3: AKS pods can't pull images and nodes are NotReady. Diagnose and fix the cluster.

**Type:** Debugging | **Category:** AKS

## The Scenario

Your AKS cluster is experiencing multiple issues:

```bash
$ kubectl get nodes
NAME                              STATUS     ROLES   AGE   VERSION
aks-nodepool1-12345678-vmss0000   NotReady   agent   2d    v1.27.3
aks-nodepool1-12345678-vmss0001   Ready      agent   2d    v1.27.3
aks-nodepool1-12345678-vmss0002   NotReady   agent   2d    v1.27.3

$ kubectl get pods
NAME                    READY   STATUS             RESTARTS   AGE
api-server-abc123       0/1     ImagePullBackOff   0          30m
worker-def456           0/1     Pending            0          25m
```

The cluster was working yesterday. You need to restore service quickly.

## The Challenge

Systematically diagnose node failures and pod issues. Understand AKS-specific debugging techniques and implement fixes.


### Step 1: Diagnose Node Issues

```bash
# Get detailed node status
kubectl describe node aks-nodepool1-12345678-vmss0000

# Look for conditions:
# - MemoryPressure
# - DiskPressure
# - PIDPressure
# - NetworkUnavailable
# - Ready

# Check node events
kubectl get events --field-selector involvedObject.kind=Node --sort-by='.lastTimestamp'

# Common NotReady causes:
# 1. kubelet not running
# 2. Network plugin issues (Azure CNI)
# 3. Disk pressure
# 4. Memory exhaustion
```

### Step 2: Check Node Health via Azure

```bash
# List VMSS instances
az vmss list-instances \
  --resource-group MC_myResourceGroup_myAKSCluster_eastus \
  --name aks-nodepool1-12345678-vmss \
  --output table

# Check instance health
az vmss get-instance-view \
  --resource-group MC_myResourceGroup_myAKSCluster_eastus \
  --name aks-nodepool1-12345678-vmss \
  --instance-id 0

# Reimage unhealthy node (last resort)
az vmss reimage \
  --resource-group MC_myResourceGroup_myAKSCluster_eastus \
  --name aks-nodepool1-12345678-vmss \
  --instance-id 0
```

### Step 3: Diagnose Image Pull Issues

```bash
# Check pod events
kubectl describe pod api-server-abc123

# Common ImagePullBackOff causes:
# 1. ACR authentication failure
# 2. Image doesn't exist
# 3. Network connectivity to registry
# 4. Private endpoint DNS resolution

# Check if AKS can authenticate to ACR
az aks check-acr \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --acr myacr.azurecr.io

# Attach ACR to AKS (if not done)
az aks update \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --attach-acr myacr
```

### Step 4: Fix ACR Authentication

```bicep
// Bicep: Proper ACR integration with AKS
resource aks 'Microsoft.ContainerService/managedClusters@2023-05-01' = {
  name: aksName
  location: location
  identity: {
    type: 'SystemAssigned'
  }
  properties: {
    // ... other properties
  }
}

// Grant AKS kubelet identity AcrPull role
resource acrPullRole 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid(acr.id, aks.id, 'acrpull')
  scope: acr
  properties: {
    principalId: aks.properties.identityProfile.kubeletidentity.objectId
    principalType: 'ServicePrincipal'
    roleDefinitionId: subscriptionResourceId('Microsoft.Authorization/roleDefinitions',
      '7f951dda-4ed3-4680-a7ca-43fe172d538d')  // AcrPull
  }
}
```

### Step 5: Debug Network Issues (Private Cluster)

```bash
# For private clusters, check DNS resolution
kubectl run debug-pod --image=busybox --rm -it --restart=Never -- nslookup myacr.azurecr.io

# Should resolve to private IP, not public
# If resolving to public IP, private endpoint DNS is misconfigured

# Check egress connectivity
kubectl run debug-pod --image=curlimages/curl --rm -it --restart=Never -- \
  curl -v https://myacr.azurecr.io/v2/

# Verify Private DNS Zone links
az network private-dns zone show \
  --resource-group myResourceGroup \
  --name "privatelink.azurecr.io"
```

### Step 6: Check Resource Constraints

```bash
# Check node resources
kubectl top nodes

# Check pod resource usage
kubectl top pods

# Check resource quotas
kubectl describe resourcequotas

# Check LimitRanges
kubectl describe limitranges

# Find pods causing resource pressure
kubectl get pods --all-namespaces -o json | \
  jq -r '.items[] | select(.spec.containers[].resources.requests != null) |
  "\(.metadata.namespace)/\(.metadata.name): \(.spec.containers[].resources.requests)"'
```

### Step 7: Fix Pending Pods

```bash
# Check why pod is pending
kubectl describe pod worker-def456

# Common Pending causes:
# 1. Insufficient CPU/memory
# 2. Node affinity/selectors don't match
# 3. Taints not tolerated
# 4. PVC not bound

# Check available resources per node
kubectl describe nodes | grep -A 5 "Allocated resources"

# Scale node pool if needed
az aks nodepool scale \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --name nodepool1 \
  --node-count 5
```

### Step 8: Enable AKS Diagnostics

```bicep
// Enable diagnostic settings for AKS
resource aksDiagnostics 'Microsoft.Insights/diagnosticSettings@2021-05-01-preview' = {
  name: 'aks-diagnostics'
  scope: aks
  properties: {
    workspaceId: logAnalyticsWorkspace.id
    logs: [
      {
        category: 'kube-apiserver'
        enabled: true
      }
      {
        category: 'kube-controller-manager'
        enabled: true
      }
      {
        category: 'kube-scheduler'
        enabled: true
      }
      {
        category: 'kube-audit'
        enabled: true
      }
      {
        category: 'cluster-autoscaler'
        enabled: true
      }
    ]
    metrics: [
      {
        category: 'AllMetrics'
        enabled: true
      }
    ]
  }
}
```

```kusto
// KQL query to find node issues
KubeNodeInventory
| where TimeGenerated > ago(1h)
| where Status != "Ready"
| project TimeGenerated, Computer, Status, Labels
| order by TimeGenerated desc

// KQL query to find pod failures
KubePodInventory
| where TimeGenerated > ago(1h)
| where PodStatus in ("Failed", "Unknown", "Pending")
| project TimeGenerated, Name, Namespace, PodStatus, ContainerStatusReason
| order by TimeGenerated desc
```

### Step 9: Implement Proper Health Monitoring

```yaml
# Deployment with proper probes
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-server
  template:
    spec:
      containers:
      - name: api
        image: myacr.azurecr.io/api:v1.2.3
        resources:
          requests:
            cpu: "250m"
            memory: "512Mi"
          limits:
            cpu: "500m"
            memory: "1Gi"
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
          failureThreshold: 3
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 20
          failureThreshold: 3
        startupProbe:
          httpGet:
            path: /health
            port: 8080
          failureThreshold: 30
          periodSeconds: 10
```


## AKS Debugging Cheatsheet

| Issue | Command | Fix |
|-------|---------|-----|
| Node NotReady | `kubectl describe node` | Check conditions, reimage if needed |
| ImagePullBackOff | `az aks check-acr` | Attach ACR, check DNS |
| Pending pods | `kubectl describe pod` | Scale nodepool, fix selectors |
| CrashLoopBackOff | `kubectl logs --previous` | Fix application code |
| OOMKilled | `kubectl describe pod` | Increase memory limits |

## Common AKS Issues

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| All pods pending | Cluster autoscaler disabled | Enable autoscaler |
| Can't pull from ACR | Missing AcrPull role | `az aks update --attach-acr` |
| DNS resolution fails | CoreDNS issues | Check CoreDNS pods |
| Private cluster no access | Jump box not configured | Deploy jump box in VNet |

---

### Question 4: Azure Storage costs are $30K/month. Implement tiering and lifecycle policies.

**Type:** Practical | **Category:** Storage

## The Scenario

Your Azure Storage bill analysis shows:

```
Storage Account: proddata (60TB total)
├── Hot tier: 60TB ($1,242/month storage + $1,500 transactions)
├── logs/ container: 30TB - 99% older than 30 days
├── backups/ container: 20TB - accessed monthly
├── media/ container: 10TB - frequently accessed
└── No lifecycle policies configured

Total: $30,000/month (including egress and transactions)
```

Most data is rarely accessed but stored in Hot tier.

## The Challenge

Implement a tiered storage strategy with lifecycle management to reduce costs by at least 50% while maintaining required access patterns.


> **Common Mistake:** A junior engineer might move everything to Archive tier, delete old data without checking retention requirements, or manually move blobs periodically. These approaches break access patterns, cause compliance violations, or create operational overhead.

> **Senior Engineer Approach:** A senior engineer analyzes access patterns per container, implements automated lifecycle policies for tier transitions, uses Cool for infrequently accessed data, Archive for compliance/backup data, and sets up access tier tracking in Azure Monitor.

### Step 1: Analyze Current Usage

```bash
# Get storage account metrics
az monitor metrics list \
  --resource /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/proddata \
  --metric "BlobCount,BlobCapacity" \
  --interval PT1H \
  --start-time 2024-01-01 \
  --end-time 2024-01-31

# List blobs with last access time
az storage blob list \
  --account-name proddata \
  --container-name logs \
  --query "[].{name:name,tier:properties.blobTier,lastAccess:properties.lastAccessedOn,size:properties.contentLength}" \
  --output table

# Enable last access time tracking
az storage account blob-service-properties update \
  --account-name proddata \
  --resource-group myRG \
  --enable-last-access-tracking true
```

### Step 2: Understand Azure Storage Tiers

| Tier | Storage Cost/GB | Access Cost | Min Duration | Use Case |
|------|----------------|-------------|--------------|----------|
| Hot | $0.0184 | Low | None | Frequent access |
| Cool | $0.01 | Medium | 30 days | Monthly access |
| Cold | $0.0036 | Higher | 90 days | Quarterly access |
| Archive | $0.00099 | High + delay | 180 days | Yearly/compliance |

### Step 3: Implement Lifecycle Policies

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "logs-lifecycle",
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["logs/"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 7
            },
            "tierToCold": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            },
            "delete": {
              "daysAfterModificationGreaterThan": 365
            }
          }
        }
      }
    },
    {
      "enabled": true,
      "name": "backups-lifecycle",
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["backups/"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 1
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 30
            },
            "delete": {
              "daysAfterModificationGreaterThan": 2555
            }
          }
        }
      }
    },
    {
      "enabled": true,
      "name": "cleanup-snapshots",
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"]
        },
        "actions": {
          "snapshot": {
            "tierToCool": {
              "daysAfterCreationGreaterThan": 30
            },
            "delete": {
              "daysAfterCreationGreaterThan": 90
            }
          }
        }
      }
    }
  ]
}
```

```bash
# Apply lifecycle policy
az storage account management-policy create \
  --account-name proddata \
  --resource-group myRG \
  --policy @lifecycle-policy.json
```

### Step 4: Bicep Implementation

```bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: 'proddata'
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    accessTier: 'Hot'  // Default tier for new blobs
    allowBlobPublicAccess: false
    minimumTlsVersion: 'TLS1_2'
  }
}

resource blobServices 'Microsoft.Storage/storageAccounts/blobServices@2023-01-01' = {
  parent: storageAccount
  name: 'default'
  properties: {
    lastAccessTimeTrackingPolicy: {
      enable: true
      name: 'AccessTimeTracking'
      trackingGranularityInDays: 1
      blobType: ['blockBlob']
    }
  }
}

resource lifecyclePolicy 'Microsoft.Storage/storageAccounts/managementPolicies@2023-01-01' = {
  parent: storageAccount
  name: 'default'
  properties: {
    policy: {
      rules: [
        {
          enabled: true
          name: 'logs-tier-and-delete'
          type: 'Lifecycle'
          definition: {
            filters: {
              blobTypes: ['blockBlob']
              prefixMatch: ['logs/']
            }
            actions: {
              baseBlob: {
                tierToCool: {
                  daysAfterLastAccessTimeGreaterThan: 7
                }
                tierToCold: {
                  daysAfterLastAccessTimeGreaterThan: 30
                }
                tierToArchive: {
                  daysAfterLastAccessTimeGreaterThan: 90
                }
                delete: {
                  daysAfterLastAccessTimeGreaterThan: 365
                }
              }
            }
          }
        }
        {
          enabled: true
          name: 'media-optimization'
          type: 'Lifecycle'
          definition: {
            filters: {
              blobTypes: ['blockBlob']
              prefixMatch: ['media/']
            }
            actions: {
              baseBlob: {
                tierToCool: {
                  daysAfterLastAccessTimeGreaterThan: 30
                }
              }
            }
          }
        }
      ]
    }
  }
}
```

### Step 5: Use Cold Tier for Additional Savings

```bash
# Cold tier (new in 2023) offers better economics than Cool for 90+ day retention
# Cold: $0.0036/GB storage, higher access costs
# Cool: $0.01/GB storage, lower access costs

# Break-even: ~2 reads per blob per month

# Move existing blobs to Cold
az storage blob set-tier \
  --account-name proddata \
  --container-name backups \
  --name "backup-2023-01.zip" \
  --tier Cold
```

### Step 6: Archive Tier for Compliance Data

```bash
# Archive tier: $0.00099/GB but requires rehydration
# Rehydration time: Standard (up to 15 hours), High priority (< 1 hour)

# Set blob to Archive
az storage blob set-tier \
  --account-name proddata \
  --container-name compliance \
  --name "audit-2020.zip" \
  --tier Archive

# Rehydrate when needed
az storage blob set-tier \
  --account-name proddata \
  --container-name compliance \
  --name "audit-2020.zip" \
  --tier Hot \
  --rehydrate-priority High
```

### Step 7: Monitor and Optimize

```bicep
// Alert on unexpected Hot tier growth
resource storageAlert 'Microsoft.Insights/metricAlerts@2018-03-01' = {
  name: 'hot-tier-growth-alert'
  location: 'global'
  properties: {
    description: 'Alert when Hot tier capacity grows unexpectedly'
    severity: 2
    enabled: true
    scopes: [storageAccount.id]
    evaluationFrequency: 'PT1H'
    windowSize: 'PT6H'
    criteria: {
      'odata.type': 'Microsoft.Azure.Monitor.SingleResourceMultipleMetricCriteria'
      allOf: [
        {
          name: 'HotCapacityGrowth'
          metricName: 'BlobCapacity'
          dimensions: [
            {
              name: 'Tier'
              operator: 'Include'
              values: ['Hot']
            }
          ]
          operator: 'GreaterThan'
          threshold: 15000000000000  // 15TB threshold
          timeAggregation: 'Average'
        }
      ]
    }
    actions: [
      {
        actionGroupId: actionGroup.id
      }
    ]
  }
}
```

### Cost Calculation

```
BEFORE:
- All 60TB in Hot tier
- Storage: 60TB × $0.0184 = $1,104/month
- Transactions + egress: ~$28,896/month
- Total: ~$30,000/month

AFTER (with lifecycle policies):
logs/ (30TB):
  - 2TB Hot (current week): $37
  - 5TB Cool (week 2-4): $50
  - 10TB Cold (month 2-3): $36
  - 13TB Archive (older): $13
  Subtotal: $136

backups/ (20TB):
  - 1TB Cool (current month): $10
  - 19TB Archive: $19
  Subtotal: $29

media/ (10TB):
  - 8TB Hot: $147
  - 2TB Cool: $20
  Subtotal: $167

Storage Total: $332/month (was $1,104)
Transactions reduced ~40%: $17,338
Total: ~$17,670/month

SAVINGS: ~$12,330/month (41% reduction)
```


## Lifecycle Policy Decision Guide

| Data Type | Access Pattern | Recommended Tier | Lifecycle Action |
|-----------|---------------|------------------|-----------------|
| Logs | 7-day analysis | Hot → Cool → Archive | Tiered transition |
| Backups | Monthly restore | Cool → Archive | After 30 days |
| Media | Varies | Use last access tracking | Dynamic tiering |
| Compliance | Yearly audit | Archive | Direct to Archive |

## Important Considerations

1. **Minimum storage duration** - Early deletion incurs charges
2. **Rehydration time** - Archive needs 1-15 hours
3. **Access costs** - Cool/Cold reads are more expensive
4. **Last access tracking** - Enable for smarter policies

<InterviewQuiz
  question="Why is the Cold tier better than Cool tier for data accessed less than once per month?"
  options={[
    "Cold tier has faster access times",
    "Cold tier has no minimum storage duration",
    "Cold tier storage cost ($0.0036/GB) is much lower than Cool ($0.01/GB), offsetting higher access costs for infrequent access",
    "Cold tier supports larger blob sizes"
  ]}
  correctAnswer={2}
  explanation="Cold tier costs $0.0036/GB for storage (64% less than Cool at $0.01/GB) but has higher read costs. For data accessed less than ~2 times per month, the storage savings outweigh the higher access costs. Cold tier has a 90-day minimum storage duration vs 30 days for Cool, so it's best for data you know won't be accessed frequently for at least 90 days."
/>

---

### Question 5: Azure Functions have 10-second cold starts causing API timeouts. Optimize for low latency.

**Type:** Debugging | **Category:** Serverless

## The Scenario

Your Azure Functions HTTP API experiences severe cold starts:

```
Request timeline (cold start):
├── Function host initialization: 3s
├── .NET runtime startup: 2s
├── Dependency injection setup: 3s
├── Database connection pool: 2s
└── Actual function execution: 0.5s

Total: 10.5s (API Gateway timeout: 10s)
```

Users see intermittent timeouts, especially after periods of inactivity.

## The Challenge

Reduce cold start times to under 2 seconds while balancing cost. Understand the factors affecting startup time and implement targeted optimizations.


### Step 1: Understand Cold Start Components

```
Cold Start Breakdown:
┌─────────────────────────────────────────────────┐
│ Platform (Azure) - Cannot control               │
├─────────────────────────────────────────────────┤
│ • Instance allocation: ~500ms                   │
│ • Runtime initialization: ~500-1500ms           │
│ • Extension loading: ~200-500ms                 │
└─────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────┐
│ Application (Your code) - CAN optimize          │
├─────────────────────────────────────────────────┤
│ • Package restore/loading: ~500-2000ms          │
│ • Dependency injection: ~200-3000ms             │
│ • Static initialization: varies                 │
│ • Connection establishment: ~500-2000ms         │
└─────────────────────────────────────────────────┘
```

### Step 2: Use Premium Plan with Pre-Warmed Instances

```bicep
// Premium plan with pre-warmed instances
resource functionAppPlan 'Microsoft.Web/serverfarms@2022-09-01' = {
  name: 'func-plan-premium'
  location: location
  sku: {
    tier: 'ElasticPremium'
    name: 'EP1'
  }
  properties: {
    maximumElasticWorkerCount: 20
  }
}

resource functionApp 'Microsoft.Web/sites@2022-09-01' = {
  name: 'my-api-function'
  location: location
  kind: 'functionapp'
  properties: {
    serverFarmId: functionAppPlan.id
    siteConfig: {
      // Pre-warmed instances always ready
      preWarmedInstanceCount: 2

      // Minimum instances (never scale to zero)
      functionAppScaleLimit: 10

      // Runtime settings
      netFrameworkVersion: 'v8.0'
      use32BitWorkerProcess: false

      // Keep instances warm longer
      appSettings: [
        {
          name: 'WEBSITE_WARMUP_PATH'
          value: '/api/health'
        }
        {
          name: 'FUNCTIONS_WORKER_RUNTIME'
          value: 'dotnet-isolated'
        }
      ]
    }
  }
}
```

### Step 3: Optimize Application Code

```csharp
// BEFORE: Slow startup with synchronous initialization
public class Startup : FunctionsStartup
{
    public override void Configure(IFunctionsHostBuilder builder)
    {
        // Bad: Synchronous database connection
        var dbConnection = new SqlConnection(connString);
        dbConnection.Open();  // Blocks during cold start!

        // Bad: Loading all configuration upfront
        var config = LoadAllConfiguration();  // Blocks!

        builder.Services.AddSingleton(dbConnection);
    }
}

// AFTER: Lazy initialization
public class Startup : FunctionsStartup
{
    public override void Configure(IFunctionsHostBuilder builder)
    {
        // Good: Lazy connection factory
        builder.Services.AddSingleton<IDbConnectionFactory, LazyDbConnectionFactory>();

        // Good: Configuration loaded on-demand
        builder.Services.AddSingleton<IConfiguration>(sp =>
            new ConfigurationBuilder()
                .AddEnvironmentVariables()
                .Build());

        // Good: Use AddScoped for per-request services
        builder.Services.AddScoped<IMyService, MyService>();
    }
}

public class LazyDbConnectionFactory : IDbConnectionFactory
{
    private readonly Lazy<SqlConnection> _connection;

    public LazyDbConnectionFactory(IConfiguration config)
    {
        _connection = new Lazy<SqlConnection>(() =>
        {
            var conn = new SqlConnection(config["SqlConnectionString"]);
            conn.Open();
            return conn;
        });
    }

    public SqlConnection GetConnection() => _connection.Value;
}
```

### Step 4: Reduce Package Size

```xml
<!-- BEFORE: Large package with unused dependencies -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <AzureFunctionsVersion>v4</AzureFunctionsVersion>
    <!-- Bad: Not trimmed -->
  </PropertyGroup>

  <ItemGroup>
    <!-- Bad: Heavy dependencies -->
    <PackageReference Include="Entity.Framework.Core" Version="8.0.0" />
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
  </ItemGroup>
</Project>

<!-- AFTER: Optimized package -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <AzureFunctionsVersion>v4</AzureFunctionsVersion>

    <!-- Enable trimming -->
    <PublishTrimmed>true</PublishTrimmed>
    <TrimMode>link</TrimMode>

    <!-- Ahead-of-time compilation -->
    <PublishAot>true</PublishAot>

    <!-- Single file deployment -->
    <PublishSingleFile>true</PublishSingleFile>
  </PropertyGroup>

  <ItemGroup>
    <!-- Use lighter alternatives -->
    <PackageReference Include="Dapper" Version="2.1.24" />
    <PackageReference Include="System.Text.Json" Version="8.0.0" />
  </ItemGroup>
</Project>
```

### Step 5: Use HTTP-Triggered Warmup

```csharp
// Warmup function that initializes dependencies
[Function("Warmup")]
public async Task<HttpResponseData> Warmup(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "warmup")] HttpRequestData req)
{
    // Touch lazy-loaded services to initialize them
    _ = _dbConnectionFactory.GetConnection();
    _ = _cacheService.GetClient();

    var response = req.CreateResponse(HttpStatusCode.OK);
    await response.WriteStringAsync("Warmed up");
    return response;
}

// Configure warmup path in app settings
// WEBSITE_WARMUP_PATH = /api/warmup
```

### Step 6: Consider Durable Functions for Long Operations

```csharp
// For operations that might exceed timeout, use Durable Functions
[Function("ProcessOrder_Orchestrator")]
public async Task<OrderResult> ProcessOrderOrchestrator(
    [OrchestrationTrigger] TaskOrchestrationContext context)
{
    var orderId = context.GetInput<string>();

    // These run asynchronously, don't block the HTTP response
    var validation = await context.CallActivityAsync<bool>("ValidateOrder", orderId);
    var payment = await context.CallActivityAsync<PaymentResult>("ProcessPayment", orderId);
    var shipping = await context.CallActivityAsync<ShippingResult>("ScheduleShipping", orderId);

    return new OrderResult(validation, payment, shipping);
}

[Function("ProcessOrder_Http")]
public async Task<HttpResponseData> ProcessOrderHttp(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequestData req,
    [DurableClient] DurableTaskClient client)
{
    var orderId = await req.ReadFromJsonAsync<string>();

    // Start orchestration and return immediately
    var instanceId = await client.ScheduleNewOrchestrationInstanceAsync(
        "ProcessOrder_Orchestrator", orderId);

    // Return 202 Accepted with status URL
    return client.CreateCheckStatusResponse(req, instanceId);
}
```

### Step 7: Use Flex Consumption Plan (Preview)

```bicep
// New Flex Consumption plan offers faster cold starts
resource functionAppPlan 'Microsoft.Web/serverfarms@2023-01-01' = {
  name: 'func-plan-flex'
  location: location
  sku: {
    tier: 'FlexConsumption'
    name: 'FC1'
  }
  properties: {
    reserved: true
  }
}

// Flex benefits:
// - Faster cold starts (VNet integration without penalty)
// - Instance size options (2GB, 4GB memory)
// - Always-ready instances (like Premium but pay-per-use)
// - Concurrency control per instance
```

### Cold Start Comparison by Plan

| Plan | Typical Cold Start | Always-On Option | Cost |
|------|-------------------|------------------|------|
| Consumption | 2-10+ seconds | No | Pay per execution |
| Premium EP1 | Under 1 second | Yes (pre-warmed) | ~$150/month base |
| Premium EP2 | Under 1 second | Yes (pre-warmed) | ~$300/month base |
| Flex Consumption | 1-3 seconds | Yes (always-ready) | Pay per execution + always-ready |
| Dedicated | None | Always on | App Service pricing |


## Cold Start Optimization Checklist

| Optimization | Impact | Effort |
|-------------|--------|--------|
| Premium plan + pre-warmed | High | Low |
| Lazy dependency injection | High | Medium |
| Reduce package size | Medium | Medium |
| Use System.Text.Json | Low | Low |
| Enable AOT compilation | Medium | Medium |
| Warmup function | Medium | Low |

<InterviewQuiz
  question="Why is lazy initialization of database connections important for reducing Azure Functions cold starts?"
  options={[
    "Lazy initialization reduces the function package size",
    "Database connections are faster when created lazily",
    "Synchronous connection establishment blocks the cold start, while lazy loading defers it to the first actual request",
    "Azure Functions require lazy initialization for database connections"
  ]}
  correctAnswer={2}
  explanation="During cold start, everything in the Startup/Configure method runs synchronously before the function can serve requests. If you open database connections there, you add 500-2000ms to every cold start. With lazy initialization, the cold start completes quickly (the function becomes 'ready'), and the database connection is only established on the first request that actually needs it. This is a one-time cost that doesn't block the platform's readiness check."
/>

---

### Question 6: Cosmos DB is consuming 50,000 RU/s and costing $3K/day. Optimize the data model and queries.

**Type:** Practical | **Category:** Data Services

## The Scenario

Your Cosmos DB costs have exploded:

```
Current Usage:
├── Provisioned: 50,000 RU/s
├── Cost: ~$3,000/day (~$90,000/month)
├── Database: orders-db
├── Container: orders (1TB data)
├── Partition key: /userId
└── Most expensive query: GetOrdersByDate

Query analysis:
- GetOrdersByDate: 15,000 RU per execution, runs 100x/hour
- GetUserOrders: 500 RU per execution
- CreateOrder: 10 RU per execution
```

## The Challenge

Reduce RU consumption by 80% through data modeling, query optimization, indexing, and partition strategy improvements.


### Step 1: Analyze Current Query Costs

```javascript
// Enable query metrics in SDK
const { resource: item, requestCharge } = await container.items
    .query({
        query: "SELECT * FROM orders o WHERE o.orderDate >= @startDate",
        parameters: [{ name: "@startDate", value: "2024-01-01" }]
    }, {
        populateQueryMetrics: true
    })
    .fetchAll();

console.log(`Query cost: ${requestCharge} RUs`);

// In Azure Portal: Metrics > Total Request Units
// Filter by OperationType to find expensive queries
```

### Step 2: Fix the Partition Key Problem

```javascript
// CURRENT: Partition key is /userId
// Query: GetOrdersByDate - scans ALL partitions!

// Query that causes cross-partition fan-out (EXPENSIVE)
SELECT * FROM orders o
WHERE o.orderDate >= '2024-01-01'
// This must check EVERY partition because orderDate isn't the partition key

// SOLUTION 1: Hierarchical partition key (if using latest SDK)
{
    "partitionKey": {
        "paths": ["/tenantId", "/userId"],
        "kind": "MultiHash"
    }
}

// SOLUTION 2: Use composite partition key in data model
{
    "id": "order-123",
    "userId": "user-456",
    "orderDate": "2024-01-15",
    "pk": "2024-01-user-456",  // Composite partition key
    "yearMonth": "2024-01"     // For date-range queries
}
```

### Step 3: Create Composite Indexes

```json
{
    "indexingPolicy": {
        "automatic": true,
        "indexingMode": "consistent",
        "includedPaths": [
            { "path": "/*" }
        ],
        "excludedPaths": [
            { "path": "/largeDescription/*" },
            { "path": "/metadata/*" }
        ],
        "compositeIndexes": [
            [
                { "path": "/userId", "order": "ascending" },
                { "path": "/orderDate", "order": "descending" }
            ],
            [
                { "path": "/status", "order": "ascending" },
                { "path": "/orderDate", "order": "descending" }
            ]
        ]
    }
}
```

```bash
# Apply indexing policy
az cosmosdb sql container update \
    --account-name mycosmosdb \
    --database-name orders-db \
    --name orders \
    --idx @indexing-policy.json
```

### Step 4: Denormalize for Read-Heavy Patterns

```javascript
// BEFORE: Normalized (requires joins/lookups)
// Order document
{
    "id": "order-123",
    "userId": "user-456",
    "items": ["item-1", "item-2"]  // Just IDs, need lookup
}

// AFTER: Denormalized (single read)
{
    "id": "order-123",
    "userId": "user-456",
    "customerName": "John Doe",        // Embedded
    "customerEmail": "john@example.com", // Embedded
    "items": [
        {
            "productId": "prod-1",
            "productName": "Widget",    // Embedded
            "price": 29.99,
            "quantity": 2
        }
    ],
    "totalAmount": 59.98,
    "shippingAddress": {                // Embedded
        "street": "123 Main St",
        "city": "Seattle"
    }
}

// Trade-off: Larger documents, but single read instead of 5 reads
// 1 RU vs 5+ RUs per order retrieval
```

### Step 5: Use Change Feed for Materialized Views

```csharp
// Create materialized view container for date-based queries
// Partition key: /yearMonth for efficient date range queries

public class OrderMaterializedView
{
    public string id { get; set; }
    public string yearMonth { get; set; }  // Partition key: "2024-01"
    public string orderId { get; set; }
    public string userId { get; set; }
    public DateTime orderDate { get; set; }
    public decimal totalAmount { get; set; }
    public string status { get; set; }
}

// Change Feed processor to maintain the view
var changeFeedProcessor = container
    .GetChangeFeedProcessorBuilder<Order>("OrderViewBuilder", HandleChangesAsync)
    .WithInstanceName("OrderViewProcessor")
    .WithLeaseContainer(leaseContainer)
    .Build();

async Task HandleChangesAsync(
    IReadOnlyCollection<Order> changes,
    CancellationToken cancellationToken)
{
    foreach (var order in changes)
    {
        var view = new OrderMaterializedView
        {
            id = order.id,
            yearMonth = order.orderDate.ToString("yyyy-MM"),
            orderId = order.id,
            userId = order.userId,
            orderDate = order.orderDate,
            totalAmount = order.totalAmount,
            status = order.status
        };

        await viewContainer.UpsertItemAsync(view,
            new PartitionKey(view.yearMonth));
    }
}

// Now GetOrdersByDate queries the view container
// Single partition query: ~5 RU instead of 15,000 RU!
```

### Step 6: Optimize Query Patterns

```javascript
// BEFORE: Expensive query patterns
// 1. SELECT * (returns all properties)
SELECT * FROM orders o WHERE o.userId = 'user-123'

// 2. No partition key in query
SELECT * FROM orders o WHERE o.status = 'pending'

// 3. Inefficient pagination
SELECT * FROM orders o ORDER BY o.orderDate OFFSET 100 LIMIT 10

// AFTER: Optimized queries
// 1. Project only needed fields
SELECT o.id, o.orderDate, o.totalAmount
FROM orders o
WHERE o.userId = 'user-123'

// 2. Include partition key
SELECT o.id, o.status
FROM orders o
WHERE o.userId = 'user-123' AND o.status = 'pending'

// 3. Use continuation tokens
const { resources, continuationToken } = await container.items
    .query(query, { maxItemCount: 10, continuationToken: previousToken })
    .fetchNext();
```

### Step 7: Implement TTL for Transient Data

```json
// Container-level TTL
{
    "id": "orders",
    "partitionKey": { "paths": ["/userId"] },
    "defaultTtl": 7776000  // 90 days in seconds
}

// Document-level TTL override
{
    "id": "temp-cart-123",
    "userId": "user-456",
    "items": [...],
    "ttl": 3600  // This cart expires in 1 hour
}
```

### Step 8: Switch to Autoscale

```bicep
// Instead of fixed 50,000 RU/s, use autoscale
resource container 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers@2023-04-15' = {
  parent: database
  name: 'orders'
  properties: {
    resource: {
      id: 'orders'
      partitionKey: {
        paths: ['/userId']
        kind: 'Hash'
      }
    }
    options: {
      autoscaleSettings: {
        maxThroughput: 50000  // Scales between 5,000-50,000 RU/s
      }
    }
  }
}

// Cost savings: Only pay for what you use
// Peak: 50,000 RU/s during business hours
// Off-peak: 5,000 RU/s at night (90% cost reduction)
```

### Cost Comparison

```
BEFORE:
- 50,000 RU/s provisioned (constant)
- $2,920/day × 30 = ~$87,600/month

AFTER:
- Materialized view: GetOrdersByDate 5 RU (was 15,000)
- Composite indexes: Queries 50% faster
- Denormalization: Order reads 1 RU (was 5)
- Autoscale: Average 15,000 RU/s (was 50,000 constant)

- ~$876/day × 30 = ~$26,280/month
- SAVINGS: ~$61,320/month (70% reduction)
```


## Cosmos DB Optimization Checklist

| Optimization | RU Impact | Implementation |
|-------------|-----------|----------------|
| Partition key design | 10-100x | Data model change |
| Composite indexes | 2-10x | Index policy update |
| Project specific fields | 1.5-3x | Query change |
| Denormalization | 2-5x | Data model change |
| Materialized views | 10-1000x | New container + change feed |
| TTL for temp data | Cost reduction | Container setting |

---

### Question 7: Azure SQL failover takes too long and connections drop. Implement proper HA.

**Type:** Practical | **Category:** Databases

## The Scenario

Your mission-critical application uses Azure SQL Database:

```
Current Setup:
├── Single Azure SQL Database (S3 tier)
├── Location: East US
├── Connection string: hardcoded in app.config
├── No geo-replication configured
├── RTO requirement: < 5 minutes
├── RPO requirement: < 30 seconds
└── Recent outage: 45 minutes (regional issue)
```

Business is demanding zero-downtime database access across regions.

## The Challenge

Implement auto-failover groups with proper connection handling, understand the failover process, and design for multi-region high availability.


### Step 1: Understand Failover Options

```
Azure SQL HA Options:
┌─────────────────────────────────────────────────────────────────┐
│ Zone Redundancy (Same Region)                                    │
│ • Automatic within availability zones                            │
│ • No configuration needed (just enable)                          │
│ • RTO: ~0 (automatic)                                           │
└─────────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────────┐
│ Active Geo-Replication (Cross-Region)                           │
│ • Up to 4 readable secondaries                                   │
│ • Manual failover required                                       │
│ • Different connection strings per replica                       │
└─────────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────────┐
│ Auto-Failover Groups (Cross-Region + Automatic)                 │
│ • Automatic failover with grace period                          │
│ • Single listener endpoint (DNS-based)                          │
│ • Read-write and read-only endpoints                            │
│ • Recommended for production                                    │
└─────────────────────────────────────────────────────────────────┘
```

### Step 2: Create Failover Group with Bicep

```bicep
// Parameters
param primaryLocation string = 'eastus'
param secondaryLocation string = 'westus'
param sqlAdminLogin string
@secure()
param sqlAdminPassword string

// Primary SQL Server
resource primaryServer 'Microsoft.Sql/servers@2023-05-01-preview' = {
  name: 'sql-primary-${uniqueString(resourceGroup().id)}'
  location: primaryLocation
  properties: {
    administratorLogin: sqlAdminLogin
    administratorLoginPassword: sqlAdminPassword
    minimalTlsVersion: '1.2'
  }
}

// Secondary SQL Server (in different region)
resource secondaryServer 'Microsoft.Sql/servers@2023-05-01-preview' = {
  name: 'sql-secondary-${uniqueString(resourceGroup().id)}'
  location: secondaryLocation
  properties: {
    administratorLogin: sqlAdminLogin
    administratorLoginPassword: sqlAdminPassword
    minimalTlsVersion: '1.2'
  }
}

// Database on primary server
resource database 'Microsoft.Sql/servers/databases@2023-05-01-preview' = {
  parent: primaryServer
  name: 'appdb'
  location: primaryLocation
  sku: {
    name: 'S3'
    tier: 'Standard'
  }
  properties: {
    zoneRedundant: true  // Enable zone redundancy
    readScale: 'Enabled' // Enable read scale-out
  }
}

// Auto-failover group
resource failoverGroup 'Microsoft.Sql/servers/failoverGroups@2023-05-01-preview' = {
  parent: primaryServer
  name: 'fog-appdb'
  properties: {
    readWriteEndpoint: {
      failoverPolicy: 'Automatic'
      failoverWithDataLossGracePeriodMinutes: 1  // Wait 1 min before auto-failover
    }
    readOnlyEndpoint: {
      failoverPolicy: 'Enabled'  // Failover read endpoint too
    }
    partnerServers: [
      {
        id: secondaryServer.id
      }
    ]
    databases: [
      database.id
    ]
  }
}

// Outputs - Use these endpoints in your application
output readWriteEndpoint string = 'fog-appdb.database.windows.net'
output readOnlyEndpoint string = 'fog-appdb.secondary.database.windows.net'
```

### Step 3: Configure Connection Strings

```csharp
// appsettings.json - Use failover group endpoints
{
  "ConnectionStrings": {
    // Read-write traffic (points to current primary)
    "DefaultConnection": "Server=fog-appdb.database.windows.net;Database=appdb;Authentication=Active Directory Default;Encrypt=True;",

    // Read-only traffic (can use secondary for read scale)
    "ReadOnlyConnection": "Server=fog-appdb.secondary.database.windows.net;Database=appdb;Authentication=Active Directory Default;Encrypt=True;ApplicationIntent=ReadOnly;"
  }
}

// Connection with retry logic
public class ResilientDbContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlServer(connectionString, sqlOptions =>
        {
            // Enable connection resiliency
            sqlOptions.EnableRetryOnFailure(
                maxRetryCount: 5,
                maxRetryDelay: TimeSpan.FromSeconds(30),
                errorNumbersToAdd: new List<int> {
                    4060,   // Cannot open database
                    40197,  // Error processing request
                    40501,  // Service busy
                    40613,  // Database unavailable
                    49918,  // Not enough resources
                    49919,  // Cannot process request
                    49920   // Too many requests
                });

            sqlOptions.CommandTimeout(60);
        });
    }
}
```

### Step 4: Implement Connection Retry Logic

```csharp
// Polly-based retry policy for transient failures
public static class SqlRetryPolicy
{
    public static IAsyncPolicy GetRetryPolicy()
    {
        return Policy
            .Handle<SqlException>(ex => IsTransientError(ex))
            .Or<TimeoutException>()
            .WaitAndRetryAsync(
                retryCount: 5,
                sleepDurationProvider: attempt =>
                    TimeSpan.FromSeconds(Math.Pow(2, attempt)),  // Exponential backoff
                onRetry: (exception, timeSpan, retryCount, context) =>
                {
                    Log.Warning(
                        "SQL transient error. Retry {RetryCount} after {Delay}ms. Error: {Error}",
                        retryCount, timeSpan.TotalMilliseconds, exception.Message);
                });
    }

    private static bool IsTransientError(SqlException ex)
    {
        // Transient error numbers during failover
        int[] transientErrors = {
            4060, 40197, 40501, 40613, 40143, 40166,
            49918, 49919, 49920, 10929, 10928, 10060,
            10054, 10053, 233, -1, -2
        };

        return transientErrors.Contains(ex.Number);
    }
}

// Usage
public async Task<Order> GetOrderAsync(int orderId)
{
    return await SqlRetryPolicy.GetRetryPolicy().ExecuteAsync(async () =>
    {
        using var connection = new SqlConnection(_connectionString);
        await connection.OpenAsync();

        return await connection.QuerySingleOrDefaultAsync<Order>(
            "SELECT * FROM Orders WHERE Id = @Id",
            new { Id = orderId });
    });
}
```

### Step 5: Monitor Failover Group Health

```bicep
// Alert on failover events
resource failoverAlert 'Microsoft.Insights/metricAlerts@2018-03-01' = {
  name: 'sql-failover-alert'
  location: 'global'
  properties: {
    description: 'Alert when failover group role changes'
    severity: 1
    enabled: true
    scopes: [primaryServer.id]
    evaluationFrequency: 'PT1M'
    windowSize: 'PT5M'
    criteria: {
      'odata.type': 'Microsoft.Azure.Monitor.SingleResourceMultipleMetricCriteria'
      allOf: [
        {
          name: 'FailoverGroupRole'
          metricName: 'sql_server_failover_group_role'
          dimensions: []
          operator: 'GreaterThan'
          threshold: 0  // 0 = Primary, 1 = Secondary
          timeAggregation: 'Maximum'
        }
      ]
    }
    actions: [
      {
        actionGroupId: actionGroup.id
      }
    ]
  }
}
```

```bash
# Check failover group status
az sql failover-group show \
  --name fog-appdb \
  --resource-group myResourceGroup \
  --server sql-primary

# Manual failover (for testing/planned maintenance)
az sql failover-group set-primary \
  --name fog-appdb \
  --resource-group myResourceGroup \
  --server sql-secondary

# Forced failover (data loss possible)
az sql failover-group set-primary \
  --name fog-appdb \
  --resource-group myResourceGroup \
  --server sql-secondary \
  --allow-data-loss
```

### Step 6: Read Scale-Out for Performance

```csharp
// Route read queries to secondary replicas
public class ReadScaleDbContext : DbContext
{
    private readonly bool _useReadReplica;

    public ReadScaleDbContext(bool useReadReplica = false)
    {
        _useReadReplica = useReadReplica;
    }

    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        var connectionString = _useReadReplica
            ? "Server=fog-appdb.secondary.database.windows.net;Database=appdb;ApplicationIntent=ReadOnly;..."
            : "Server=fog-appdb.database.windows.net;Database=appdb;...";

        options.UseSqlServer(connectionString);
    }
}

// Service using read replicas for reports
public class ReportService
{
    public async Task<SalesReport> GetMonthlySalesAsync()
    {
        // Use read replica for heavy read queries
        using var context = new ReadScaleDbContext(useReadReplica: true);

        return await context.Orders
            .Where(o => o.OrderDate >= startOfMonth)
            .GroupBy(o => o.ProductCategory)
            .Select(g => new { Category = g.Key, Total = g.Sum(o => o.Amount) })
            .ToListAsync();
    }
}
```

### Failover Behavior Summary

```
Failover Timeline:
┌────────────────────────────────────────────────────────────────┐
│ T+0:00   Primary becomes unavailable                           │
│ T+0:00   Azure detects health check failures                   │
│ T+0:30   Grace period begins (configured: 1 minute)            │
│ T+1:00   Grace period expires                                  │
│ T+1:00   Automatic failover initiated                          │
│ T+1:05   Secondary promoted to primary                         │
│ T+1:10   DNS updated (fog-appdb.database.windows.net)          │
│ T+1:15   Apps reconnect to new primary (with retry logic)      │
│ T+1:30   Full service restored                                 │
└────────────────────────────────────────────────────────────────┘

Total RTO: ~90 seconds (within 5 minute requirement)
RPO: Near-zero with sync replication (within 30 second requirement)
```


## Failover Group Checklist

| Component | Configuration | Purpose |
|-----------|--------------|---------|
| Grace period | 1 hour (default), 0-1 hour | Avoid false failovers |
| Read-only endpoint | Enabled | Route read traffic to secondary |
| Listener endpoints | Use .database.windows.net | Automatic DNS failover |
| Connection retry | Exponential backoff | Handle transient errors |
| Zone redundancy | Enable | HA within region |

## Connection String Endpoints

| Endpoint | Format | Routes To |
|----------|--------|-----------|
| Read-write | `fog-name.database.windows.net` | Current primary |
| Read-only | `fog-name.secondary.database.windows.net` | Current secondary |


---

### Quick Check

**Why should you use the failover group listener endpoint instead of the primary server endpoint in connection strings?**

   A. The listener endpoint provides better performance
   B. The listener endpoint is required for encryption
-> C. **The listener endpoint automatically routes to the current primary after failover, while the server endpoint would still point to the old (now secondary) server**
   D. The listener endpoint supports more concurrent connections

<details>
<summary>See Answer</summary>

The failover group listener endpoint (fog-name.database.windows.net) is a DNS alias that always points to the current primary server. When failover occurs, Azure updates the DNS to point to the new primary. If you hardcode the server name (sql-primary.database.windows.net), your app would still try to connect to that server even after it becomes the secondary, causing write failures. The listener endpoint abstracts the failover, requiring no connection string changes.

</details>

---

### Question 8: Messages are lost and processed out of order in Service Bus. Implement reliable messaging.

**Type:** Practical | **Category:** Messaging

## The Scenario

Your order processing system is losing messages:

```
Service Bus Configuration:
├── Namespace: Standard tier
├── Queue: orders-queue
├── Current issues:
│   ├── ~5% of orders never processed
│   ├── Some orders processed multiple times
│   ├── Dead letter queue has 10,000 messages
│   └── No monitoring or alerting configured
└── Message flow: Web API → Queue → Order Processor → Database
```

The business is losing revenue due to unprocessed orders.

## The Challenge

Implement reliable messaging with proper error handling, dead letter management, duplicate detection, and monitoring to achieve zero message loss.


> **Common Mistake:** A junior engineer might use auto-complete mode without proper error handling, ignore the dead letter queue, skip duplicate detection, or use ReceiveAndDelete mode for performance. These approaches cause message loss, duplicate processing, and silent failures.

> **Senior Engineer Approach:** A senior engineer implements PeekLock with explicit completion, configures dead letter handling with monitoring, enables duplicate detection, implements idempotent message handlers, and sets up proper retry policies with circuit breakers.

### Step 1: Understand Message Loss Scenarios

```
Why Messages Get Lost:
┌─────────────────────────────────────────────────────────────────┐
│ 1. ReceiveAndDelete Mode                                        │
│    Message deleted immediately on receive                       │
│    If processor crashes before completing work = LOST           │
└─────────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────────┐
│ 2. PeekLock Timeout                                             │
│    Lock expires before processing completes                     │
│    Message becomes visible again = DUPLICATES                   │
└─────────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────────┐
│ 3. MaxDeliveryCount Exceeded                                    │
│    Failed processing > 10 times (default)                       │
│    Message moved to Dead Letter Queue                           │
└─────────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────────┐
│ 4. TTL Expired                                                   │
│    Message not processed within TimeToLive                      │
│    Message discarded or dead-lettered                           │
└─────────────────────────────────────────────────────────────────┘
```

### Step 2: Configure Queue Properly

```bicep
// Service Bus namespace and queue with reliability settings
resource serviceBusNamespace 'Microsoft.ServiceBus/namespaces@2022-10-01-preview' = {
  name: 'sb-orders-${uniqueString(resourceGroup().id)}'
  location: location
  sku: {
    name: 'Premium'  // Premium for production (partitioning, geo-DR)
    tier: 'Premium'
    capacity: 1
  }
  properties: {
    zoneRedundant: true  // High availability
  }
}

resource ordersQueue 'Microsoft.ServiceBus/namespaces/queues@2022-10-01-preview' = {
  parent: serviceBusNamespace
  name: 'orders-queue'
  properties: {
    // Reliability settings
    requiresDuplicateDetection: true
    duplicateDetectionHistoryTimeWindow: 'PT10M'  // 10 minute window

    // Dead letter configuration
    deadLetteringOnMessageExpiration: true
    maxDeliveryCount: 10

    // Message settings
    defaultMessageTimeToLive: 'P14D'  // 14 days
    lockDuration: 'PT5M'  // 5 minute lock (extend for long processing)

    // Ordering
    requiresSession: false  // Enable if order matters

    // Size
    maxSizeInMegabytes: 5120  // 5GB
    enablePartitioning: false  // Disable for ordering guarantees
  }
}

// Dead letter queue alert
resource dlqAlert 'Microsoft.Insights/metricAlerts@2018-03-01' = {
  name: 'orders-dlq-alert'
  location: 'global'
  properties: {
    description: 'Alert when dead letter queue has messages'
    severity: 2
    enabled: true
    scopes: [serviceBusNamespace.id]
    evaluationFrequency: 'PT5M'
    windowSize: 'PT15M'
    criteria: {
      'odata.type': 'Microsoft.Azure.Monitor.SingleResourceMultipleMetricCriteria'
      allOf: [
        {
          name: 'DeadLetteredMessages'
          metricName: 'DeadletteredMessages'
          dimensions: [
            {
              name: 'EntityName'
              operator: 'Include'
              values: ['orders-queue']
            }
          ]
          operator: 'GreaterThan'
          threshold: 0
          timeAggregation: 'Total'
        }
      ]
    }
    actions: [
      {
        actionGroupId: actionGroup.id
      }
    ]
  }
}
```

### Step 3: Implement Reliable Message Sender

```csharp
// Reliable message sender with retry and duplicate detection
public class ReliableOrderSender
{
    private readonly ServiceBusSender _sender;
    private readonly ILogger<ReliableOrderSender> _logger;

    public ReliableOrderSender(ServiceBusClient client, ILogger<ReliableOrderSender> logger)
    {
        _sender = client.CreateSender("orders-queue");
        _logger = logger;
    }

    public async Task SendOrderAsync(Order order)
    {
        var message = new ServiceBusMessage(JsonSerializer.SerializeToUtf8Bytes(order))
        {
            // Unique ID for duplicate detection
            MessageId = $"order-{order.Id}-{order.Version}",

            // Business correlation
            CorrelationId = order.CorrelationId,

            // Content type for deserialization
            ContentType = "application/json",

            // Custom properties for filtering/routing
            ApplicationProperties =
            {
                ["OrderType"] = order.Type,
                ["Priority"] = order.Priority,
                ["CustomerId"] = order.CustomerId
            },

            // Schedule for later if needed
            // ScheduledEnqueueTime = DateTimeOffset.UtcNow.AddMinutes(5)
        };

        // Retry policy for transient failures
        var retryPolicy = Policy
            .Handle<ServiceBusException>(ex => ex.IsTransient)
            .WaitAndRetryAsync(3, attempt => TimeSpan.FromSeconds(Math.Pow(2, attempt)),
                (ex, delay, attempt, ctx) =>
                {
                    _logger.LogWarning(ex,
                        "Send attempt {Attempt} failed, retrying in {Delay}ms",
                        attempt, delay.TotalMilliseconds);
                });

        await retryPolicy.ExecuteAsync(async () =>
        {
            await _sender.SendMessageAsync(message);
            _logger.LogInformation("Order {OrderId} sent successfully", order.Id);
        });
    }

    // Batch sending for high throughput
    public async Task SendOrderBatchAsync(IEnumerable<Order> orders)
    {
        using ServiceBusMessageBatch batch = await _sender.CreateMessageBatchAsync();

        foreach (var order in orders)
        {
            var message = new ServiceBusMessage(JsonSerializer.SerializeToUtf8Bytes(order))
            {
                MessageId = $"order-{order.Id}-{order.Version}"
            };

            if (!batch.TryAddMessage(message))
            {
                // Batch is full, send it and create new batch
                await _sender.SendMessagesAsync(batch);
                batch = await _sender.CreateMessageBatchAsync();

                if (!batch.TryAddMessage(message))
                {
                    throw new Exception($"Message too large for batch: {order.Id}");
                }
            }
        }

        if (batch.Count > 0)
        {
            await _sender.SendMessagesAsync(batch);
        }
    }
}
```

### Step 4: Implement Reliable Message Processor

```csharp
// Reliable processor with proper error handling
public class OrderProcessor : BackgroundService
{
    private readonly ServiceBusProcessor _processor;
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger<OrderProcessor> _logger;

    public OrderProcessor(ServiceBusClient client, IServiceProvider serviceProvider,
        ILogger<OrderProcessor> logger)
    {
        _serviceProvider = serviceProvider;
        _logger = logger;

        _processor = client.CreateProcessor("orders-queue", new ServiceBusProcessorOptions
        {
            // PeekLock mode (default) - explicit completion required
            ReceiveMode = ServiceBusReceiveMode.PeekLock,

            // Process multiple messages concurrently
            MaxConcurrentCalls = 10,

            // Auto-renew lock for long processing
            MaxAutoLockRenewalDuration = TimeSpan.FromMinutes(30),

            // Prefetch for performance
            PrefetchCount = 20
        });
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _processor.ProcessMessageAsync += ProcessMessageAsync;
        _processor.ProcessErrorAsync += ProcessErrorAsync;

        await _processor.StartProcessingAsync(stoppingToken);

        // Keep running until cancellation
        await Task.Delay(Timeout.Infinite, stoppingToken);
    }

    private async Task ProcessMessageAsync(ProcessMessageEventArgs args)
    {
        var orderId = args.Message.MessageId;
        var deliveryCount = args.Message.DeliveryCount;

        _logger.LogInformation(
            "Processing order {OrderId}, attempt {DeliveryCount}",
            orderId, deliveryCount);

        try
        {
            var order = JsonSerializer.Deserialize<Order>(args.Message.Body.ToArray());

            using var scope = _serviceProvider.CreateScope();
            var orderService = scope.ServiceProvider.GetRequiredService<IOrderService>();

            // Process with idempotency check
            await orderService.ProcessOrderAsync(order);

            // Explicitly complete the message
            await args.CompleteMessageAsync(args.Message);

            _logger.LogInformation("Order {OrderId} completed successfully", orderId);
        }
        catch (DuplicateOrderException ex)
        {
            // Already processed - complete the message to remove it
            _logger.LogWarning("Duplicate order {OrderId}, completing message", orderId);
            await args.CompleteMessageAsync(args.Message);
        }
        catch (InvalidOrderException ex)
        {
            // Business validation failed - dead letter immediately
            _logger.LogError(ex, "Invalid order {OrderId}, dead-lettering", orderId);
            await args.DeadLetterMessageAsync(args.Message,
                deadLetterReason: "InvalidOrder",
                deadLetterErrorDescription: ex.Message);
        }
        catch (TransientException ex) when (deliveryCount < 5)
        {
            // Transient error - abandon to retry
            _logger.LogWarning(ex,
                "Transient error processing {OrderId}, abandoning for retry", orderId);
            await args.AbandonMessageAsync(args.Message);
        }
        catch (Exception ex)
        {
            // Unexpected error - decide based on delivery count
            if (deliveryCount >= 10)
            {
                _logger.LogError(ex,
                    "Max delivery count reached for {OrderId}, dead-lettering", orderId);
                await args.DeadLetterMessageAsync(args.Message,
                    deadLetterReason: "MaxDeliveryCountExceeded",
                    deadLetterErrorDescription: ex.Message);
            }
            else
            {
                _logger.LogWarning(ex,
                    "Error processing {OrderId}, abandoning for retry", orderId);
                await args.AbandonMessageAsync(args.Message);
            }
        }
    }

    private Task ProcessErrorAsync(ProcessErrorEventArgs args)
    {
        _logger.LogError(args.Exception,
            "Service Bus error. Source: {Source}, Entity: {Entity}",
            args.ErrorSource, args.EntityPath);
        return Task.CompletedTask;
    }
}
```

### Step 5: Implement Idempotent Processing

```csharp
// Idempotent order processing to handle duplicates
public class OrderService : IOrderService
{
    private readonly ApplicationDbContext _context;

    public async Task ProcessOrderAsync(Order order)
    {
        // Check if already processed (idempotency key)
        var existingOrder = await _context.Orders
            .FirstOrDefaultAsync(o => o.Id == order.Id);

        if (existingOrder != null)
        {
            if (existingOrder.Version >= order.Version)
            {
                throw new DuplicateOrderException(order.Id);
            }
            // Newer version - update
            _context.Entry(existingOrder).CurrentValues.SetValues(order);
        }
        else
        {
            // New order
            _context.Orders.Add(order);
        }

        // Use database transaction for atomicity
        using var transaction = await _context.Database.BeginTransactionAsync();

        try
        {
            await _context.SaveChangesAsync();

            // Record processing in idempotency table
            await _context.ProcessedMessages.AddAsync(new ProcessedMessage
            {
                MessageId = $"order-{order.Id}-{order.Version}",
                ProcessedAt = DateTime.UtcNow
            });

            await _context.SaveChangesAsync();
            await transaction.CommitAsync();
        }
        catch
        {
            await transaction.RollbackAsync();
            throw;
        }
    }
}
```

### Step 6: Dead Letter Queue Processing

```csharp
// Process dead letter queue for investigation and retry
public class DeadLetterProcessor : BackgroundService
{
    private readonly ServiceBusReceiver _dlqReceiver;
    private readonly ServiceBusSender _sender;
    private readonly ILogger<DeadLetterProcessor> _logger;

    public DeadLetterProcessor(ServiceBusClient client, ILogger<DeadLetterProcessor> logger)
    {
        _logger = logger;

        // Create receiver for dead letter sub-queue
        _dlqReceiver = client.CreateReceiver(
            "orders-queue",
            new ServiceBusReceiverOptions
            {
                SubQueue = SubQueue.DeadLetter,
                ReceiveMode = ServiceBusReceiveMode.PeekLock
            });

        _sender = client.CreateSender("orders-queue");
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            var messages = await _dlqReceiver.ReceiveMessagesAsync(
                maxMessages: 10,
                maxWaitTime: TimeSpan.FromSeconds(30),
                cancellationToken: stoppingToken);

            foreach (var message in messages)
            {
                var reason = message.DeadLetterReason;
                var description = message.DeadLetterErrorDescription;

                _logger.LogWarning(
                    "Dead letter message {MessageId}. Reason: {Reason}, Description: {Description}",
                    message.MessageId, reason, description);

                // Decide whether to retry or discard
                if (ShouldRetry(reason))
                {
                    // Create new message for retry
                    var retryMessage = new ServiceBusMessage(message.Body)
                    {
                        MessageId = $"{message.MessageId}-retry",
                        ApplicationProperties = { ["RetryCount"] = GetRetryCount(message) + 1 }
                    };

                    await _sender.SendMessageAsync(retryMessage);
                    _logger.LogInformation("Retrying message {MessageId}", message.MessageId);
                }
                else
                {
                    // Log for manual investigation
                    await LogToInvestigationTableAsync(message);
                }

                await _dlqReceiver.CompleteMessageAsync(message);
            }

            await Task.Delay(TimeSpan.FromMinutes(1), stoppingToken);
        }
    }

    private bool ShouldRetry(string reason)
    {
        return reason switch
        {
            "TransientError" => true,
            "MaxDeliveryCountExceeded" => true,  // May have been a temporary issue
            "InvalidOrder" => false,              // Business error, won't succeed
            _ => false
        };
    }
}
```

### Message Flow Summary

```
Reliable Message Flow:
                              ┌─────────────────────────────┐
                              │     Dead Letter Queue       │
                              │  (Monitor + Alert + Retry)  │
                              └──────────────▲──────────────┘
                                             │ Max Delivery
                                             │ or Explicit DLQ
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────────────┐   ┌─────────┐
│   API   │──▶│  Queue  │──▶│ PeekLock│──▶│    Processor    │──▶│Database │
│ (Send)  │   │         │   │         │   │ (Idempotent)    │   │         │
└─────────┘   └─────────┘   └────┬────┘   └────────┬────────┘   └─────────┘
     │              ▲            │                 │
     │              │            │   Complete      │
     │              └────────────┴─────────────────┘
     │                           │
     │        Duplicate          │   Abandon (retry)
     └──────Detection────────────┘
```


## Service Bus Reliability Checklist

| Setting | Value | Purpose |
|---------|-------|---------|
| ReceiveMode | PeekLock | Don't delete until processed |
| DuplicateDetection | Enabled | Prevent duplicate sends |
| MaxDeliveryCount | 10 | Retry before dead-letter |
| LockDuration | 5 min | Match processing time |
| DeadLetterOnExpiration | true | Don't lose expired messages |

---

### Question 9: Design a global application with Azure Front Door for traffic management and WAF protection.

**Type:** Architecture | **Category:** Traffic Management

## The Scenario

Your e-commerce platform needs global reach:

```
Current State:
├── Single region: East US
├── Average latency: 200ms (US), 400ms (Europe), 600ms (Asia)
├── No CDN for static assets
├── Single origin = single point of failure
└── Compliance: GDPR requires EU data to stay in EU

Business Requirements:
├── < 100ms latency globally for static content
├── < 200ms latency for API responses
├── 99.99% availability (multi-region)
├── Automatic failover between regions
└── WAF protection against OWASP Top 10
```

## The Challenge

Design a multi-region architecture using Azure Front Door Premium with intelligent traffic routing, WAF protection, and private link to origins.


### Architecture Overview

```
                           Azure Front Door Premium
                    ┌──────────────────────────────────────┐
                    │         Global Entry Point           │
                    │     app.contoso.com (Anycast)        │
                    │                                       │
                    │  ┌─────────────────────────────────┐ │
                    │  │           WAF Policy             │ │
                    │  │  OWASP 3.2 + Custom Rules        │ │
                    │  └─────────────────────────────────┘ │
                    │                                       │
                    │  ┌─────────────────────────────────┐ │
                    │  │     CDN / Edge Caching          │ │
                    │  │  Static: /images/*, /css/*      │ │
                    │  └─────────────────────────────────┘ │
                    └──────────────────────────────────────┘
                                      │
                    ┌─────────────────┼─────────────────┐
                    ▼                 ▼                 ▼
          ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
          │   East US       │ │   West Europe   │ │   Southeast Asia│
          │                 │ │                 │ │                 │
          │ ┌─────────────┐ │ │ ┌─────────────┐ │ │ ┌─────────────┐ │
          │ │App Service  │ │ │ │App Service  │ │ │ │App Service  │ │
          │ │(Private EP) │ │ │ │(Private EP) │ │ │ │(Private EP) │ │
          │ └─────────────┘ │ │ └─────────────┘ │ │ └─────────────┘ │
          │                 │ │                 │ │                 │
          │ ┌─────────────┐ │ │ ┌─────────────┐ │ │ ┌─────────────┐ │
          │ │  Azure SQL  │ │ │ │  Azure SQL  │ │ │ │  Azure SQL  │ │
          │ │  (Read Rep) │ │ │ │  (Primary)  │ │ │ │  (Read Rep) │ │
          │ └─────────────┘ │ │ └─────────────┘ │ │ └─────────────┘ │
          └─────────────────┘ └─────────────────┘ └─────────────────┘
```

### Step 1: Create Front Door Profile

```bicep
// Azure Front Door Premium with WAF and Private Link
resource frontDoorProfile 'Microsoft.Cdn/profiles@2023-05-01' = {
  name: 'fd-contoso-global'
  location: 'global'
  sku: {
    name: 'Premium_AzureFrontDoor'  // Required for Private Link and WAF
  }
}

// Custom domain endpoint
resource frontDoorEndpoint 'Microsoft.Cdn/profiles/afdEndpoints@2023-05-01' = {
  parent: frontDoorProfile
  name: 'contoso-endpoint'
  location: 'global'
  properties: {
    enabledState: 'Enabled'
  }
}

// Custom domain
resource customDomain 'Microsoft.Cdn/profiles/customDomains@2023-05-01' = {
  parent: frontDoorProfile
  name: 'app-contoso-com'
  properties: {
    hostName: 'app.contoso.com'
    tlsSettings: {
      certificateType: 'ManagedCertificate'
      minimumTlsVersion: 'TLS12'
    }
  }
}
```

### Step 2: Configure Origin Groups with Health Probes

```bicep
// Origin group for multi-region backends
resource originGroup 'Microsoft.Cdn/profiles/originGroups@2023-05-01' = {
  parent: frontDoorProfile
  name: 'app-origin-group'
  properties: {
    loadBalancingSettings: {
      sampleSize: 4
      successfulSamplesRequired: 2
      additionalLatencyInMilliseconds: 50  // Latency tolerance
    }
    healthProbeSettings: {
      probePath: '/health'
      probeRequestType: 'GET'
      probeProtocol: 'Https'
      probeIntervalInSeconds: 30
    }
    sessionAffinityState: 'Disabled'  // Enable if needed for stateful apps
  }
}

// East US origin (App Service with Private Link)
resource originEastUS 'Microsoft.Cdn/profiles/originGroups/origins@2023-05-01' = {
  parent: originGroup
  name: 'origin-eastus'
  properties: {
    hostName: 'app-eastus.azurewebsites.net'
    httpPort: 80
    httpsPort: 443
    originHostHeader: 'app-eastus.azurewebsites.net'
    priority: 1
    weight: 1000
    enabledState: 'Enabled'
    sharedPrivateLinkResource: {
      privateLink: {
        id: appServiceEastUS.id
      }
      groupId: 'sites'
      privateLinkLocation: 'eastus'
      requestMessage: 'Front Door Private Link'
    }
  }
}

// West Europe origin
resource originWestEurope 'Microsoft.Cdn/profiles/originGroups/origins@2023-05-01' = {
  parent: originGroup
  name: 'origin-westeurope'
  properties: {
    hostName: 'app-westeurope.azurewebsites.net'
    httpPort: 80
    httpsPort: 443
    originHostHeader: 'app-westeurope.azurewebsites.net'
    priority: 1
    weight: 1000
    enabledState: 'Enabled'
    sharedPrivateLinkResource: {
      privateLink: {
        id: appServiceWestEurope.id
      }
      groupId: 'sites'
      privateLinkLocation: 'westeurope'
      requestMessage: 'Front Door Private Link'
    }
  }
}

// Southeast Asia origin
resource originSEAsia 'Microsoft.Cdn/profiles/originGroups/origins@2023-05-01' = {
  parent: originGroup
  name: 'origin-seasia'
  properties: {
    hostName: 'app-seasia.azurewebsites.net'
    httpPort: 80
    httpsPort: 443
    originHostHeader: 'app-seasia.azurewebsites.net'
    priority: 1
    weight: 1000
    enabledState: 'Enabled'
    sharedPrivateLinkResource: {
      privateLink: {
        id: appServiceSEAsia.id
      }
      groupId: 'sites'
      privateLinkLocation: 'southeastasia'
      requestMessage: 'Front Door Private Link'
    }
  }
}
```

### Step 3: Configure Routes with Caching

```bicep
// Static content route with caching
resource staticRoute 'Microsoft.Cdn/profiles/afdEndpoints/routes@2023-05-01' = {
  parent: frontDoorEndpoint
  name: 'static-route'
  properties: {
    customDomains: [
      {
        id: customDomain.id
      }
    ]
    originGroup: {
      id: originGroup.id
    }
    supportedProtocols: ['Https']
    patternsToMatch: [
      '/images/*'
      '/css/*'
      '/js/*'
      '/fonts/*'
    ]
    forwardingProtocol: 'HttpsOnly'
    linkToDefaultDomain: 'Enabled'
    httpsRedirect: 'Enabled'
    cacheConfiguration: {
      compressionSettings: {
        isCompressionEnabled: true
        contentTypesToCompress: [
          'text/html'
          'text/css'
          'application/javascript'
          'application/json'
          'image/svg+xml'
        ]
      }
      queryStringCachingBehavior: 'IgnoreQueryString'
      cacheBehavior: 'OverrideAlways'
      cacheDuration: 'P7D'  // 7 days for static assets
    }
  }
}

// API route without caching
resource apiRoute 'Microsoft.Cdn/profiles/afdEndpoints/routes@2023-05-01' = {
  parent: frontDoorEndpoint
  name: 'api-route'
  properties: {
    customDomains: [
      {
        id: customDomain.id
      }
    ]
    originGroup: {
      id: originGroup.id
    }
    supportedProtocols: ['Https']
    patternsToMatch: ['/api/*']
    forwardingProtocol: 'HttpsOnly'
    linkToDefaultDomain: 'Enabled'
    httpsRedirect: 'Enabled'
    cacheConfiguration: {
      queryStringCachingBehavior: 'UseQueryString'
      cacheBehavior: 'HonorOrigin'  // Respect Cache-Control headers
    }
  }
}

// Default route
resource defaultRoute 'Microsoft.Cdn/profiles/afdEndpoints/routes@2023-05-01' = {
  parent: frontDoorEndpoint
  name: 'default-route'
  properties: {
    customDomains: [
      {
        id: customDomain.id
      }
    ]
    originGroup: {
      id: originGroup.id
    }
    supportedProtocols: ['Https']
    patternsToMatch: ['/*']
    forwardingProtocol: 'HttpsOnly'
    linkToDefaultDomain: 'Enabled'
    httpsRedirect: 'Enabled'
  }
}
```

### Step 4: Configure WAF Policy

```bicep
// WAF Policy with managed rules
resource wafPolicy 'Microsoft.Network/FrontDoorWebApplicationFirewallPolicies@2022-05-01' = {
  name: 'waf-contoso-global'
  location: 'global'
  sku: {
    name: 'Premium_AzureFrontDoor'
  }
  properties: {
    policySettings: {
      mode: 'Prevention'  // Block attacks (use Detection for testing)
      requestBodyCheck: 'Enabled'
      maxRequestBodySizeInKb: 128
    }
    managedRules: {
      managedRuleSets: [
        {
          ruleSetType: 'Microsoft_DefaultRuleSet'
          ruleSetVersion: '2.1'
          ruleGroupOverrides: []
        }
        {
          ruleSetType: 'Microsoft_BotManagerRuleSet'
          ruleSetVersion: '1.0'
        }
      ]
    }
    customRules: {
      rules: [
        {
          name: 'BlockBadBots'
          priority: 1
          ruleType: 'MatchRule'
          action: 'Block'
          matchConditions: [
            {
              matchVariable: 'RequestHeaders'
              selector: 'User-Agent'
              operator: 'Contains'
              matchValue: ['bot', 'spider', 'crawler']
              transforms: ['Lowercase']
            }
          ]
        }
        {
          name: 'RateLimitByIP'
          priority: 2
          ruleType: 'RateLimitRule'
          rateLimitThreshold: 1000
          rateLimitDurationInMinutes: 1
          action: 'Block'
          matchConditions: [
            {
              matchVariable: 'RequestUri'
              operator: 'Contains'
              matchValue: ['/api/']
            }
          ]
        }
        {
          name: 'GeoBlockRule'
          priority: 3
          ruleType: 'MatchRule'
          action: 'Block'
          matchConditions: [
            {
              matchVariable: 'RemoteAddr'
              operator: 'GeoMatch'
              matchValue: ['CN', 'RU']  // Block specific countries
            }
          ]
        }
      ]
    }
  }
}

// Associate WAF with Front Door security policy
resource securityPolicy 'Microsoft.Cdn/profiles/securityPolicies@2023-05-01' = {
  parent: frontDoorProfile
  name: 'waf-security-policy'
  properties: {
    parameters: {
      type: 'WebApplicationFirewall'
      wafPolicy: {
        id: wafPolicy.id
      }
      associations: [
        {
          domains: [
            {
              id: customDomain.id
            }
          ]
          patternsToMatch: ['/*']
        }
      ]
    }
  }
}
```

### Step 5: Geo-Routing for Compliance

```bicep
// Rule set for geo-based routing (GDPR compliance)
resource ruleSet 'Microsoft.Cdn/profiles/ruleSets@2023-05-01' = {
  parent: frontDoorProfile
  name: 'geo-routing-rules'
}

resource euRoutingRule 'Microsoft.Cdn/profiles/ruleSets/rules@2023-05-01' = {
  parent: ruleSet
  name: 'route-eu-to-eu-origin'
  properties: {
    order: 1
    conditions: [
      {
        name: 'RemoteAddress'
        parameters: {
          typeName: 'DeliveryRuleRemoteAddressConditionParameters'
          operator: 'GeoMatch'
          matchValues: [
            'AT', 'BE', 'BG', 'HR', 'CY', 'CZ', 'DK', 'EE', 'FI',
            'FR', 'DE', 'GR', 'HU', 'IE', 'IT', 'LV', 'LT', 'LU',
            'MT', 'NL', 'PL', 'PT', 'RO', 'SK', 'SI', 'ES', 'SE'
          ]
        }
      }
    ]
    actions: [
      {
        name: 'RouteConfigurationOverride'
        parameters: {
          typeName: 'DeliveryRuleRouteConfigurationOverrideActionParameters'
          originGroupOverride: {
            originGroup: {
              id: euOriginGroup.id  // EU-only origin group
            }
          }
        }
      }
    ]
  }
}
```

### Step 6: Monitor and Alert

```bicep
// Diagnostic settings
resource frontDoorDiagnostics 'Microsoft.Insights/diagnosticSettings@2021-05-01-preview' = {
  name: 'fd-diagnostics'
  scope: frontDoorProfile
  properties: {
    workspaceId: logAnalyticsWorkspace.id
    logs: [
      {
        category: 'FrontDoorAccessLog'
        enabled: true
      }
      {
        category: 'FrontDoorHealthProbeLog'
        enabled: true
      }
      {
        category: 'FrontDoorWebApplicationFirewallLog'
        enabled: true
      }
    ]
    metrics: [
      {
        category: 'AllMetrics'
        enabled: true
      }
    ]
  }
}

// Alert on origin health
resource originHealthAlert 'Microsoft.Insights/metricAlerts@2018-03-01' = {
  name: 'fd-origin-health-alert'
  location: 'global'
  properties: {
    description: 'Alert when origin health drops below threshold'
    severity: 1
    enabled: true
    scopes: [frontDoorProfile.id]
    evaluationFrequency: 'PT1M'
    windowSize: 'PT5M'
    criteria: {
      'odata.type': 'Microsoft.Azure.Monitor.SingleResourceMultipleMetricCriteria'
      allOf: [
        {
          name: 'OriginHealthPercentage'
          metricName: 'OriginHealthPercentage'
          operator: 'LessThan'
          threshold: 80
          timeAggregation: 'Average'
        }
      ]
    }
    actions: [
      {
        actionGroupId: actionGroup.id
      }
    ]
  }
}
```

```kusto
// KQL query for WAF blocked requests
AzureDiagnostics
| where Category == "FrontDoorWebApplicationFirewallLog"
| where action_s == "Block"
| summarize BlockedRequests = count() by
    ruleName_s,
    clientIp_s,
    requestUri_s
| order by BlockedRequests desc
| take 100

// KQL query for origin latency
AzureDiagnostics
| where Category == "FrontDoorAccessLog"
| summarize
    AvgLatency = avg(timeTaken_d),
    P95Latency = percentile(timeTaken_d, 95),
    P99Latency = percentile(timeTaken_d, 99)
by originName_s, bin(TimeGenerated, 5m)
| render timechart
```


## Front Door vs Other Services

| Feature | Front Door | Traffic Manager | Application Gateway |
|---------|-----------|-----------------|---------------------|
| Layer | 7 (HTTP) | DNS | 7 (HTTP) |
| Scope | Global | Global | Regional |
| CDN | Built-in | No | No |
| WAF | Yes (Premium) | No | Yes |
| Private Link | Yes (Premium) | No | Yes |
| SSL Offload | Yes | No | Yes |
| Latency-based | Yes | Yes | No |

## Routing Methods

| Method | Use Case |
|--------|----------|
| Latency | Route to fastest origin |
| Priority | Active-passive failover |
| Weighted | Blue-green deployments |
| Session Affinity | Stateful applications |


---

### Quick Check

**Why use Azure Front Door Premium with Private Link to origins instead of public endpoints?**

   A. Private Link is required for custom domains
   B. Private Link provides better caching performance
-> C. **Private Link ensures origin traffic never traverses the public internet, reducing attack surface and enabling origins to block all public access**
   D. Private Link is cheaper than public endpoints

<details>
<summary>See Answer</summary>

With Private Link, Front Door connects to your origins (App Service, Storage, etc.) over the Microsoft backbone network using private IP addresses. This means you can completely block public access to your origins (via access restrictions or private endpoints) while still serving traffic through Front Door. Without Private Link, your origins must have public endpoints, creating a potential attack vector where attackers could bypass Front Door and hit your origins directly.

</details>

---

### Question 10: Build a monitoring solution with Azure Monitor that catches issues before users notice.

**Type:** Practical | **Category:** Observability

## The Scenario

Your production environment lacks proper observability:

```
Current State:
├── Azure resources spread across 3 subscriptions
├── No centralized logging
├── Alerts: only CPU > 90% email (often ignored)
├── MTTD (Mean Time to Detect): 2-4 hours
├── MTTR (Mean Time to Recover): 4-8 hours
├── Last week: customers reported issues before team noticed
└── No correlation between logs, metrics, and traces
```

Business requirement: Detect production issues within 5 minutes.

## The Challenge

Implement a comprehensive monitoring strategy using Azure Monitor, Log Analytics, Application Insights, and proper alerting to achieve proactive incident detection.


### Step 1: Design Monitoring Architecture

```
Centralized Monitoring Architecture:
┌────────────────────────────────────────────────────────────────────────┐
│                        Azure Monitor                                    │
│  ┌──────────────────────────────────────────────────────────────────┐ │
│  │              Log Analytics Workspace (Central)                    │ │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌────────────┐ │ │
│  │  │ App Insights│ │Platform Logs│ │ Custom Logs │ │ Security   │ │ │
│  │  │   (APM)     │ │(Diagnostic) │ │(Application)│ │  Logs      │ │ │
│  │  └─────────────┘ └─────────────┘ └─────────────┘ └────────────┘ │ │
│  └──────────────────────────────────────────────────────────────────┘ │
│                               │                                        │
│  ┌────────────────────────────┼────────────────────────────────────┐  │
│  │                      Alert Rules                                 │  │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐               │  │
│  │  │   Metric    │ │     Log     │ │  Activity   │               │  │
│  │  │   Alerts    │ │   Alerts    │ │   Alerts    │               │  │
│  │  └─────────────┘ └─────────────┘ └─────────────┘               │  │
│  └──────────────────────────────────────────────────────────────────┘ │
│                               │                                        │
│  ┌────────────────────────────┼────────────────────────────────────┐  │
│  │                     Action Groups                                │  │
│  │     Email │ SMS │ PagerDuty │ Logic Apps │ Azure Functions      │  │
│  └──────────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────────┘
```

### Step 2: Create Centralized Log Analytics Workspace

```bicep
// Centralized Log Analytics workspace
resource logAnalyticsWorkspace 'Microsoft.OperationalInsights/workspaces@2022-10-01' = {
  name: 'law-central-${environment}'
  location: location
  properties: {
    sku: {
      name: 'PerGB2018'
    }
    retentionInDays: 90
    features: {
      enableLogAccessUsingOnlyResourcePermissions: true
    }
    workspaceCapping: {
      dailyQuotaGb: 10  // Cost control
    }
  }
}

// Application Insights connected to workspace
resource appInsights 'Microsoft.Insights/components@2020-02-02' = {
  name: 'appi-${appName}-${environment}'
  location: location
  kind: 'web'
  properties: {
    Application_Type: 'web'
    WorkspaceResourceId: logAnalyticsWorkspace.id
    IngestionMode: 'LogAnalytics'
    publicNetworkAccessForIngestion: 'Enabled'
    publicNetworkAccessForQuery: 'Enabled'
    RetentionInDays: 90
  }
}

// Enable diagnostic settings for all resources
resource appServiceDiagnostics 'Microsoft.Insights/diagnosticSettings@2021-05-01-preview' = {
  name: 'diag-appservice'
  scope: appService
  properties: {
    workspaceId: logAnalyticsWorkspace.id
    logs: [
      {
        category: 'AppServiceHTTPLogs'
        enabled: true
      }
      {
        category: 'AppServiceConsoleLogs'
        enabled: true
      }
      {
        category: 'AppServiceAppLogs'
        enabled: true
      }
      {
        category: 'AppServicePlatformLogs'
        enabled: true
      }
    ]
    metrics: [
      {
        category: 'AllMetrics'
        enabled: true
      }
    ]
  }
}
```

### Step 3: Implement Application Insights

```csharp
// Program.cs - Configure Application Insights
var builder = WebApplication.CreateBuilder(args);

// Add Application Insights telemetry
builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.ConnectionString = builder.Configuration["ApplicationInsights:ConnectionString"];
    options.EnableAdaptiveSampling = true;
    options.EnableQuickPulseMetricStream = true;  // Live Metrics
});

// Add custom telemetry initializer
builder.Services.AddSingleton<ITelemetryInitializer, CustomTelemetryInitializer>();

// Custom telemetry initializer for enriching data
public class CustomTelemetryInitializer : ITelemetryInitializer
{
    public void Initialize(ITelemetry telemetry)
    {
        // Add custom properties to all telemetry
        telemetry.Context.Cloud.RoleName = "OrderService";
        telemetry.Context.GlobalProperties["Environment"] = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT");
        telemetry.Context.GlobalProperties["Version"] = Assembly.GetExecutingAssembly().GetName().Version?.ToString();
    }
}
```

```csharp
// Custom telemetry for business metrics
public class OrderService
{
    private readonly TelemetryClient _telemetry;
    private readonly ILogger<OrderService> _logger;

    public OrderService(TelemetryClient telemetry, ILogger<OrderService> logger)
    {
        _telemetry = telemetry;
        _logger = logger;
    }

    public async Task<Order> ProcessOrderAsync(CreateOrderRequest request)
    {
        using var operation = _telemetry.StartOperation<RequestTelemetry>("ProcessOrder");

        try
        {
            // Track custom event
            _telemetry.TrackEvent("OrderReceived", new Dictionary<string, string>
            {
                ["OrderId"] = request.OrderId,
                ["CustomerId"] = request.CustomerId,
                ["Region"] = request.Region
            });

            var stopwatch = Stopwatch.StartNew();
            var order = await CreateOrderInternalAsync(request);
            stopwatch.Stop();

            // Track custom metric
            _telemetry.TrackMetric("OrderProcessingTime", stopwatch.ElapsedMilliseconds);
            _telemetry.TrackMetric("OrderValue", (double)order.TotalAmount);

            // Track dependency call
            using (_telemetry.StartOperation<DependencyTelemetry>("PaymentService"))
            {
                await ProcessPaymentAsync(order);
            }

            operation.Telemetry.Success = true;
            return order;
        }
        catch (Exception ex)
        {
            operation.Telemetry.Success = false;

            _telemetry.TrackException(ex, new Dictionary<string, string>
            {
                ["OrderId"] = request.OrderId,
                ["ErrorType"] = ex.GetType().Name
            });

            // Structured logging with correlation
            _logger.LogError(ex, "Failed to process order {OrderId} for customer {CustomerId}",
                request.OrderId, request.CustomerId);

            throw;
        }
    }
}
```

### Step 4: Create Intelligent Alerts

```bicep
// Action group for notifications
resource actionGroup 'Microsoft.Insights/actionGroups@2023-01-01' = {
  name: 'ag-production-critical'
  location: 'global'
  properties: {
    groupShortName: 'ProdCrit'
    enabled: true
    emailReceivers: [
      {
        name: 'OnCallTeam'
        emailAddress: 'oncall@contoso.com'
        useCommonAlertSchema: true
      }
    ]
    smsReceivers: [
      {
        name: 'OnCallSMS'
        countryCode: '1'
        phoneNumber: '5551234567'
      }
    ]
    webhookReceivers: [
      {
        name: 'PagerDuty'
        serviceUri: 'https://events.pagerduty.com/integration/xxx/enqueue'
        useCommonAlertSchema: true
      }
    ]
  }
}

// Metric alert: High error rate
resource errorRateAlert 'Microsoft.Insights/metricAlerts@2018-03-01' = {
  name: 'alert-high-error-rate'
  location: 'global'
  properties: {
    description: 'Alert when error rate exceeds 5%'
    severity: 1  // Critical
    enabled: true
    scopes: [appInsights.id]
    evaluationFrequency: 'PT1M'
    windowSize: 'PT5M'
    criteria: {
      'odata.type': 'Microsoft.Azure.Monitor.SingleResourceMultipleMetricCriteria'
      allOf: [
        {
          name: 'FailedRequests'
          metricName: 'requests/failed'
          operator: 'GreaterThan'
          threshold: 5
          timeAggregation: 'Average'
          criterionType: 'StaticThresholdCriterion'
        }
      ]
    }
    actions: [
      {
        actionGroupId: actionGroup.id
      }
    ]
  }
}

// Dynamic threshold alert: Anomaly detection
resource anomalyAlert 'Microsoft.Insights/metricAlerts@2018-03-01' = {
  name: 'alert-response-time-anomaly'
  location: 'global'
  properties: {
    description: 'Alert when response time deviates from normal'
    severity: 2
    enabled: true
    scopes: [appInsights.id]
    evaluationFrequency: 'PT5M'
    windowSize: 'PT15M'
    criteria: {
      'odata.type': 'Microsoft.Azure.Monitor.MultipleResourceMultipleMetricCriteria'
      allOf: [
        {
          name: 'ResponseTimeAnomaly'
          metricName: 'requests/duration'
          operator: 'GreaterOrLessThan'
          alertSensitivity: 'Medium'  // Low, Medium, High
          failingPeriods: {
            numberOfEvaluationPeriods: 4
            minFailingPeriodsToAlert: 3
          }
          timeAggregation: 'Average'
          criterionType: 'DynamicThresholdCriterion'
        }
      ]
    }
    actions: [
      {
        actionGroupId: actionGroup.id
      }
    ]
  }
}

// Log alert: Exception spike
resource exceptionAlert 'Microsoft.Insights/scheduledQueryRules@2022-06-15' = {
  name: 'alert-exception-spike'
  location: location
  properties: {
    displayName: 'Exception Spike Detected'
    description: 'Alert when exception count exceeds normal levels'
    severity: 2
    enabled: true
    scopes: [logAnalyticsWorkspace.id]
    evaluationFrequency: 'PT5M'
    windowSize: 'PT15M'
    criteria: {
      allOf: [
        {
          query: '''
            exceptions
            | where timestamp > ago(15m)
            | summarize ExceptionCount = count() by type, bin(timestamp, 5m)
            | where ExceptionCount > 10
          '''
          timeAggregation: 'Count'
          operator: 'GreaterThan'
          threshold: 0
          failingPeriods: {
            numberOfEvaluationPeriods: 1
            minFailingPeriodsToAlert: 1
          }
        }
      ]
    }
    actions: {
      actionGroups: [actionGroup.id]
      customProperties: {
        Severity: 'High'
        Runbook: 'https://wiki.contoso.com/runbooks/exception-investigation'
      }
    }
  }
}
```

### Step 5: Create Azure Workbook for Dashboard

```json
// Workbook template for production dashboard
{
  "version": "Notebook/1.0",
  "items": [
    {
      "type": 1,
      "content": {
        "json": "# Production Health Dashboard\n---"
      }
    },
    {
      "type": 10,
      "content": {
        "chartId": "workbook-health-overview",
        "version": "MetricsItem/2.0",
        "size": 1,
        "chartType": 2,
        "resourceType": "microsoft.insights/components",
        "metricScope": 0,
        "resourceParameter": "AppInsights",
        "timeContext": {
          "durationMs": 3600000
        },
        "metrics": [
          {
            "namespace": "microsoft.insights/components",
            "metric": "requests/count",
            "aggregation": 1
          },
          {
            "namespace": "microsoft.insights/components",
            "metric": "requests/failed",
            "aggregation": 1
          }
        ]
      }
    },
    {
      "type": 3,
      "content": {
        "version": "KqlItem/1.0",
        "query": "requests\n| where timestamp > ago(1h)\n| summarize \n    Requests = count(),\n    AvgDuration = avg(duration),\n    P95Duration = percentile(duration, 95),\n    FailedPercent = 100.0 * countif(success == false) / count()\nby bin(timestamp, 5m)\n| render timechart",
        "size": 0,
        "title": "Request Performance (Last Hour)"
      }
    }
  ]
}
```

### Step 6: KQL Queries for Investigation

```kusto
// Find slow requests
requests
| where timestamp > ago(1h)
| where duration > 5000  // > 5 seconds
| project timestamp, name, duration, resultCode, operation_Id
| order by duration desc
| take 100

// Exception breakdown by type
exceptions
| where timestamp > ago(24h)
| summarize Count = count() by type, outerMessage
| order by Count desc
| take 20

// Dependency failures
dependencies
| where timestamp > ago(1h)
| where success == false
| summarize FailureCount = count() by target, name, resultCode
| order by FailureCount desc

// End-to-end transaction trace
union requests, dependencies, exceptions, traces
| where operation_Id == "abc123"
| project timestamp, itemType, name, duration, success, message
| order by timestamp asc

// Availability by region
requests
| where timestamp > ago(24h)
| summarize
    TotalRequests = count(),
    FailedRequests = countif(success == false),
    AvailabilityPercent = 100.0 * countif(success == true) / count()
by client_CountryOrRegion
| order by TotalRequests desc

// Performance degradation detection
requests
| where timestamp > ago(7d)
| summarize AvgDuration = avg(duration) by bin(timestamp, 1h)
| render timechart
```

### Step 7: Implement Smart Detection

```bicep
// Enable Smart Detection for Application Insights
resource smartDetection 'Microsoft.Insights/components/ProactiveDetectionConfigs@2018-05-01-preview' = {
  parent: appInsights
  name: 'slowpageloadtime'
  properties: {
    RuleDefinitions: {
      Name: 'slowpageloadtime'
      DisplayName: 'Slow page load time'
      Description: 'Smart Detection rules notify you of performance anomalies'
      HelpUrl: 'https://docs.microsoft.com/azure/application-insights/app-insights-proactive-performance-diagnostics'
      IsHidden: false
      IsEnabledByDefault: true
      IsInPreview: false
      SupportsEmailNotifications: true
    }
    Enabled: true
    SendEmailsToSubscriptionOwners: true
    CustomEmails: ['platform-team@contoso.com']
  }
}
```


## Monitoring Best Practices

| Area | Recommendation | Purpose |
|------|---------------|---------|
| Workspace | Single centralized workspace | Correlation across resources |
| Retention | 90 days (export to Storage for longer) | Cost vs compliance balance |
| Alerts | Dynamic thresholds | Reduce false positives |
| Sampling | Adaptive sampling | Cost control for high-volume |
| Dashboards | Azure Workbooks | Interactive investigation |

## Alert Severity Guidelines

| Severity | Response Time | Examples |
|----------|--------------|----------|
| Sev 0 - Critical | Immediate (page) | Service down, data loss |
| Sev 1 - Error | 15 minutes | High error rate, degraded |
| Sev 2 - Warning | 1 hour | Elevated latency, capacity |
| Sev 3 - Info | Next business day | Anomalies, recommendations |

 500ms') don't account for normal variations. During a sale event, latency might normally increase from 200ms to 400ms due to higher load - a static 300ms threshold would fire unnecessarily. Dynamic thresholds use ML to learn normal patterns (including daily/weekly seasonality) and only alert when behavior truly deviates from expectations. This dramatically reduces alert fatigue while catching real anomalies."
/>

---

### Question 11: Implement a CI/CD pipeline with Azure DevOps that deploys to AKS with blue-green releases.

**Type:** Practical | **Category:** DevOps

## The Scenario

Your team needs a robust deployment pipeline:

```
Current State:
├── Manual deployments via Portal/CLI
├── No automated testing
├── Secrets hardcoded in config files
├── Same deployment process for all environments
├── No rollback capability
├── Deployment frequency: 1-2 per month (fear of breaking things)
└── Production incidents during 30% of deployments
```

Target: Multiple deployments per day with zero-downtime and automatic rollback.

## The Challenge

Build a production-ready CI/CD pipeline in Azure DevOps with proper branching strategy, security scanning, multi-stage deployments, approval gates, and rollback capabilities.


> **Common Mistake:** A junior engineer might create a single pipeline that deploys directly to production, store secrets in pipeline variables, skip testing stages, use the same configuration for all environments, or ignore security scanning. These approaches cause production incidents, security vulnerabilities, and lack auditability.

> **Senior Engineer Approach:** A senior engineer implements multi-stage pipelines with environment-specific configurations, integrates security scanning (SAST/DAST/SCA), uses Azure Key Vault for secrets, implements proper approval gates, and designs for zero-downtime deployments with rollback capabilities.

### Step 1: Pipeline Architecture

```
CI/CD Pipeline Architecture:
┌─────────────────────────────────────────────────────────────────────────┐
│                        CI Pipeline (Build)                               │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐       │
│  │ Restore │→ │  Build  │→ │  Test   │→ │Security │→ │Artifact │       │
│  │         │  │         │  │(Unit/   │  │ Scan    │  │Publish  │       │
│  │         │  │         │  │ Int)    │  │(SAST)   │  │         │       │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └─────────┘       │
└────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        CD Pipeline (Release)                             │
│                                                                          │
│  ┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐ │
│  │   Development   │  →   │     Staging     │  →   │   Production    │ │
│  │                 │      │                 │      │                 │ │
│  │ • Auto deploy   │      │ • Auto deploy   │      │ • Manual gate   │ │
│  │ • Smoke tests   │      │ • Integration   │      │ • Blue-green    │ │
│  │ • No approval   │      │ • DAST scan     │      │ • Canary option │ │
│  └─────────────────┘      │ • Load test     │      │ • Rollback      │ │
│                           │ • 1 approval    │      │ • 2 approvals   │ │
│                           └─────────────────┘      └─────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

### Step 2: Multi-Stage YAML Pipeline

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - release/*
  paths:
    exclude:
      - docs/*
      - '*.md'

pr:
  branches:
    include:
      - main

variables:
  - group: 'global-variables'  # Variable group linked to Key Vault
  - name: buildConfiguration
    value: 'Release'
  - name: dotnetVersion
    value: '8.0.x'

stages:
  # ==================== BUILD STAGE ====================
  - stage: Build
    displayName: 'Build and Test'
    jobs:
      - job: BuildJob
        displayName: 'Build, Test, and Scan'
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: UseDotNet@2
            displayName: 'Install .NET SDK'
            inputs:
              version: $(dotnetVersion)

          - task: DotNetCoreCLI@2
            displayName: 'Restore packages'
            inputs:
              command: 'restore'
              projects: '**/*.csproj'
              feedsToUse: 'select'

          - task: DotNetCoreCLI@2
            displayName: 'Build solution'
            inputs:
              command: 'build'
              projects: '**/*.csproj'
              arguments: '--configuration $(buildConfiguration) --no-restore'

          - task: DotNetCoreCLI@2
            displayName: 'Run unit tests'
            inputs:
              command: 'test'
              projects: '**/*Tests.csproj'
              arguments: '--configuration $(buildConfiguration) --collect:"XPlat Code Coverage" --results-directory $(Agent.TempDirectory)'
              publishTestResults: true

          - task: PublishCodeCoverageResults@1
            displayName: 'Publish code coverage'
            inputs:
              codeCoverageTool: 'Cobertura'
              summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'

          # Security scanning
          - task: CredScan@3
            displayName: 'Credential Scan'
            inputs:
              toolMajorVersion: 'V2'

          - task: SdtReport@2
            displayName: 'Security Analysis Report'
            inputs:
              GdnExportAllTools: true

          # Build and push Docker image
          - task: Docker@2
            displayName: 'Build Docker image'
            inputs:
              containerRegistry: 'acr-connection'
              repository: 'myapp/api'
              command: 'build'
              Dockerfile: '**/Dockerfile'
              tags: |
                $(Build.BuildId)
                $(Build.SourceBranchName)
                latest

          - task: Docker@2
            displayName: 'Push to ACR'
            inputs:
              containerRegistry: 'acr-connection'
              repository: 'myapp/api'
              command: 'push'
              tags: |
                $(Build.BuildId)
                $(Build.SourceBranchName)

          # Publish artifacts
          - task: PublishBuildArtifacts@1
            displayName: 'Publish artifacts'
            inputs:
              pathToPublish: '$(Build.ArtifactStagingDirectory)'
              artifactName: 'drop'

  # ==================== DEV STAGE ====================
  - stage: DeployDev
    displayName: 'Deploy to Development'
    dependsOn: Build
    condition: succeeded()
    variables:
      - group: 'dev-variables'
    jobs:
      - deployment: DeployDev
        displayName: 'Deploy to Dev'
        environment: 'development'
        pool:
          vmImage: 'ubuntu-latest'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebAppContainer@1
                  displayName: 'Deploy to App Service'
                  inputs:
                    azureSubscription: 'azure-dev-connection'
                    appName: 'app-myapi-dev'
                    containers: 'myacr.azurecr.io/myapp/api:$(Build.BuildId)'

                - task: AzureAppServiceManage@0
                  displayName: 'Start slot swap warmup'
                  inputs:
                    azureSubscription: 'azure-dev-connection'
                    Action: 'Start Azure App Service'
                    WebAppName: 'app-myapi-dev'

      - job: SmokeTest
        displayName: 'Smoke Tests'
        dependsOn: DeployDev
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - script: |
              response=$(curl -s -o /dev/null -w "%{http_code}" https://app-myapi-dev.azurewebsites.net/health)
              if [ $response -ne 200 ]; then
                echo "Health check failed with status $response"
                exit 1
              fi
              echo "Health check passed"
            displayName: 'Health check'

  # ==================== STAGING STAGE ====================
  - stage: DeployStaging
    displayName: 'Deploy to Staging'
    dependsOn: DeployDev
    condition: succeeded()
    variables:
      - group: 'staging-variables'
    jobs:
      - deployment: DeployStaging
        displayName: 'Deploy to Staging'
        environment: 'staging'
        pool:
          vmImage: 'ubuntu-latest'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebAppContainer@1
                  displayName: 'Deploy to staging slot'
                  inputs:
                    azureSubscription: 'azure-staging-connection'
                    appName: 'app-myapi-staging'
                    deployToSlotOrASE: true
                    slotName: 'staging'
                    containers: 'myacr.azurecr.io/myapp/api:$(Build.BuildId)'

      - job: IntegrationTests
        displayName: 'Integration Tests'
        dependsOn: DeployStaging
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: DotNetCoreCLI@2
            displayName: 'Run integration tests'
            inputs:
              command: 'test'
              projects: '**/*IntegrationTests.csproj'
              arguments: '--configuration $(buildConfiguration)'
            env:
              API_BASE_URL: 'https://app-myapi-staging-staging.azurewebsites.net'

      - job: LoadTest
        displayName: 'Load Test'
        dependsOn: IntegrationTests
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: AzureLoadTest@1
            displayName: 'Run load test'
            inputs:
              azureSubscription: 'azure-staging-connection'
              loadTestConfigFile: 'tests/load/config.yaml'
              resourceGroup: 'rg-loadtest'
              loadTestResource: 'lt-myapi'
              env: |
                [
                  { "name": "webapp", "value": "app-myapi-staging-staging.azurewebsites.net" }
                ]

      - job: SecurityScan
        displayName: 'DAST Security Scan'
        dependsOn: DeployStaging
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: owabormuwa.owasp-zap-azure-devops.owasp-zap-scan.owaspzap@1
            displayName: 'OWASP ZAP Scan'
            inputs:
              scanType: 'targetedScan'
              zapApiUrl: 'https://app-myapi-staging-staging.azurewebsites.net'

  # ==================== PRODUCTION STAGE ====================
  - stage: DeployProduction
    displayName: 'Deploy to Production'
    dependsOn: DeployStaging
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    variables:
      - group: 'prod-variables'
    jobs:
      - deployment: DeployProd
        displayName: 'Deploy to Production'
        environment: 'production'  # Has approval gates configured
        pool:
          vmImage: 'ubuntu-latest'
        strategy:
          runOnce:
            deploy:
              steps:
                # Deploy to staging slot first (blue-green)
                - task: AzureWebAppContainer@1
                  displayName: 'Deploy to production staging slot'
                  inputs:
                    azureSubscription: 'azure-prod-connection'
                    appName: 'app-myapi-prod'
                    deployToSlotOrASE: true
                    slotName: 'staging'
                    containers: 'myacr.azurecr.io/myapp/api:$(Build.BuildId)'

                # Warm up the staging slot
                - task: AzureAppServiceManage@0
                  displayName: 'Warm up staging slot'
                  inputs:
                    azureSubscription: 'azure-prod-connection'
                    Action: 'Start Azure App Service'
                    WebAppName: 'app-myapi-prod'
                    SpecifySlotOrASE: true
                    ResourceGroupName: 'rg-myapi-prod'
                    Slot: 'staging'

                - script: |
                    echo "Warming up staging slot..."
                    for i in {1..5}; do
                      curl -s https://app-myapi-prod-staging.azurewebsites.net/health
                      sleep 2
                    done
                  displayName: 'Warmup requests'

                # Swap slots (zero-downtime deployment)
                - task: AzureAppServiceManage@0
                  displayName: 'Swap slots'
                  inputs:
                    azureSubscription: 'azure-prod-connection'
                    Action: 'Swap Slots'
                    WebAppName: 'app-myapi-prod'
                    ResourceGroupName: 'rg-myapi-prod'
                    SourceSlot: 'staging'
                    PreserveVnet: true

            on:
              failure:
                steps:
                  # Automatic rollback on failure
                  - task: AzureAppServiceManage@0
                    displayName: 'Rollback - Swap back'
                    inputs:
                      azureSubscription: 'azure-prod-connection'
                      Action: 'Swap Slots'
                      WebAppName: 'app-myapi-prod'
                      ResourceGroupName: 'rg-myapi-prod'
                      SourceSlot: 'staging'

      - job: PostDeployValidation
        displayName: 'Post-Deployment Validation'
        dependsOn: DeployProd
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - script: |
              # Validate production health
              for i in {1..10}; do
                response=$(curl -s -o /dev/null -w "%{http_code}" https://app-myapi-prod.azurewebsites.net/health)
                if [ $response -eq 200 ]; then
                  echo "Production health check passed"
                  exit 0
                fi
                echo "Attempt $i failed, retrying..."
                sleep 10
              done
              echo "Production health check failed"
              exit 1
            displayName: 'Production health check'
```

### Step 3: Environment Configuration

```yaml
# environments/production.yml
# Azure DevOps Environment with approvals and checks

# Configure in Azure DevOps UI:
# 1. Go to Pipelines > Environments > production
# 2. Add Approvals and Checks:
#    - Approvals: Require 2 approvers from release-approvers group
#    - Business Hours: Only deploy Mon-Thu, 9AM-4PM
#    - Branch Control: Only from main branch
#    - Required Template: Must use approved pipeline template

# Variable group linked to Azure Key Vault
# azure-keyvault-prod:
#   - ConnectionString (secret)
#   - ApiKey (secret)
#   - AppInsightsKey (secret)
```

### Step 4: Secure Secrets with Key Vault

```yaml
# azure-pipelines.yml - Key Vault integration
variables:
  - group: 'azure-keyvault-prod'  # Linked to Key Vault

# In variable group settings:
# 1. Link to Azure Key Vault
# 2. Select secrets to expose
# 3. Secrets automatically rotated

steps:
  - task: AzureKeyVault@2
    displayName: 'Get secrets from Key Vault'
    inputs:
      azureSubscription: 'azure-prod-connection'
      KeyVaultName: 'kv-myapi-prod'
      SecretsFilter: 'SqlConnectionString,ApiKey'
      RunAsPreJob: true

  - task: AzureCLI@2
    displayName: 'Deploy with secrets'
    inputs:
      azureSubscription: 'azure-prod-connection'
      scriptType: 'bash'
      scriptLocation: 'inlineScript'
      inlineScript: |
        az webapp config appsettings set \
          --name app-myapi-prod \
          --resource-group rg-myapi-prod \
          --settings "SqlConnectionString=$(SqlConnectionString)"
```

### Step 5: Canary Deployment Strategy

```yaml
# Canary deployment using Traffic Manager
- stage: DeployProductionCanary
  displayName: 'Canary Deployment'
  jobs:
    - deployment: CanaryDeploy
      environment: 'production-canary'
      strategy:
        canary:
          increments: [10, 50, 100]  # Traffic percentages
          preDeploy:
            steps:
              - script: echo "Preparing canary deployment"
          deploy:
            steps:
              - task: AzureWebAppContainer@1
                inputs:
                  appName: 'app-myapi-prod-canary'
                  containers: 'myacr.azurecr.io/myapp/api:$(Build.BuildId)'
          routeTraffic:
            steps:
              - task: AzureCLI@2
                displayName: 'Route traffic to canary'
                inputs:
                  azureSubscription: 'azure-prod-connection'
                  scriptType: 'bash'
                  inlineScript: |
                    az webapp traffic-routing set \
                      --distribution canary=$(strategy.increment) \
                      --name app-myapi-prod \
                      --resource-group rg-myapi-prod
          postRouteTraffic:
            steps:
              - script: |
                  # Monitor error rate for 5 minutes
                  sleep 300
                  # Query Application Insights for error rate
                  error_rate=$(az monitor app-insights query \
                    --app appi-myapi-prod \
                    --analytics-query "requests | where timestamp > ago(5m) | summarize failure_rate = 100.0 * countif(success == false) / count()" \
                    --query "tables[0].rows[0][0]" -o tsv)

                  if (( $(echo "$error_rate > 1" | bc -l) )); then
                    echo "Error rate too high: $error_rate%"
                    exit 1
                  fi
                displayName: 'Monitor canary health'
          on:
            failure:
              steps:
                - task: AzureCLI@2
                  displayName: 'Rollback canary'
                  inputs:
                    azureSubscription: 'azure-prod-connection'
                    scriptType: 'bash'
                    inlineScript: |
                      az webapp traffic-routing clear \
                        --name app-myapi-prod \
                        --resource-group rg-myapi-prod
```

### Step 6: Pipeline Templates for Reusability

```yaml
# templates/build-template.yml
parameters:
  - name: dotnetVersion
    default: '8.0.x'
  - name: projectPath
    default: '**/*.csproj'
  - name: runTests
    type: boolean
    default: true

jobs:
  - job: Build
    pool:
      vmImage: 'ubuntu-latest'
    steps:
      - task: UseDotNet@2
        inputs:
          version: ${{ parameters.dotnetVersion }}

      - task: DotNetCoreCLI@2
        displayName: 'Build'
        inputs:
          command: 'build'
          projects: ${{ parameters.projectPath }}

      - ${{ if eq(parameters.runTests, true) }}:
          - task: DotNetCoreCLI@2
            displayName: 'Test'
            inputs:
              command: 'test'
              projects: '**/*Tests.csproj'

# Usage in main pipeline:
stages:
  - stage: Build
    jobs:
      - template: templates/build-template.yml
        parameters:
          dotnetVersion: '8.0.x'
          runTests: true
```


## Pipeline Security Checklist

| Check | Implementation | Purpose |
|-------|---------------|---------|
| Secrets | Azure Key Vault + variable groups | Never in code |
| Branch protection | Required reviewers + status checks | Prevent direct pushes |
| Environment approvals | Manual gates for prod | Human verification |
| Security scanning | SAST/DAST/SCA in pipeline | Find vulnerabilities |
| Audit trail | Pipeline logs + Git history | Compliance |

## Deployment Strategies

| Strategy | Downtime | Rollback Speed | Complexity |
|----------|----------|----------------|------------|
| Basic | Yes | Slow | Low |
| Blue-Green (Slots) | Zero | Instant | Medium |
| Canary | Zero | Fast | High |
| Rolling | Minimal | Medium | Medium |


---

### Quick Check

**Why should you deploy to a staging slot first and then swap slots for production deployments?**

   A. Staging slots are faster for deployments
-> B. **It enables zero-downtime deployment by warming up the new version before routing traffic, with instant rollback by swapping back**
   C. Azure requires slot deployment for production
   D. Staging slots provide better security

<details>
<summary>See Answer</summary>

Slot swapping enables zero-downtime deployments. When you deploy to a staging slot: 1) The new version warms up (loads assemblies, caches, opens connections) without serving traffic, 2) When ready, the swap operation atomically routes all traffic to the new slot, 3) If issues occur, swap back instantly - the old version is still warm in the other slot. Without slots, deployment causes downtime during app restart and makes rollback slow (requires redeployment).

</details>

---

### Question 12: Your Azure bill increased 50% last month. Identify waste and implement cost controls.

**Type:** Practical | **Category:** Cost Optimization

## The Scenario

Your Azure spending is out of control:

```
Monthly Bill Breakdown:
├── Virtual Machines: $45,000 (30%)
│   └── Many D-series running 24/7 including dev/test
├── Azure SQL: $30,000 (20%)
│   └── Premium tier for all databases
├── Storage: $22,500 (15%)
│   └── All data in Hot tier
├── AKS: $25,500 (17%)
│   └── Overprovisioned node pools
├── App Services: $15,000 (10%)
│   └── Premium V3 for all environments
└── Other: $12,000 (8%)

Total: $150,000/month
YoY Growth: 40%
Reserved Instance Coverage: 0%
```

Finance is asking for a 30% cost reduction without impacting performance.

## The Challenge

Implement a comprehensive cost optimization strategy using Azure Cost Management, reservations, right-sizing, and architectural improvements.


### Step 1: Analyze Current Spending

```bash
# Get cost breakdown by resource group
az consumption usage list \
  --start-date 2024-01-01 \
  --end-date 2024-01-31 \
  --query "[].{ResourceGroup:resourceGroup,Cost:pretaxCost}" \
  --output table

# Get Azure Advisor recommendations
az advisor recommendation list \
  --category Cost \
  --output table

# Export cost data for analysis
az costmanagement query \
  --type Usage \
  --scope "/subscriptions/{subscription-id}" \
  --timeframe MonthToDate \
  --dataset-grouping name=ResourceGroup type=Dimension \
  --dataset-aggregation '{"totalCost":{"name":"Cost","function":"Sum"}}'
```

### Step 2: Implement Reserved Instances

```bicep
// Reserved Instance savings calculator
// Standard D4s_v5 (4 vCPU, 16GB) in East US:
// - Pay-as-you-go: $140.16/month
// - 1-year reserved: $89.79/month (36% savings)
// - 3-year reserved: $57.67/month (59% savings)

// For 10 production VMs running 24/7:
// - Current: 10 × $140.16 = $1,401.60/month
// - With 3-year RI: 10 × $57.67 = $576.70/month
// - Annual savings: $9,899/year

// Purchase recommendations based on usage patterns
resource reservationOrder 'Microsoft.Capacity/reservationOrders@2022-11-01' = {
  name: 'ro-production-vms'
  location: 'global'
  properties: {
    reservedResourceType: 'VirtualMachines'
    billingScopeId: subscription().id
    term: 'P3Y'  // 3-year term
    billingPlan: 'Monthly'
    quantity: 10
    displayName: 'Production VM Reservations'
    appliedScopes: [subscription().id]
    appliedScopeType: 'Shared'  // Apply across subscriptions
    renew: true
  }
}

// Azure SQL Database reservations
// vCore reservations apply across all SQL products
resource sqlReservation 'Microsoft.Capacity/reservationOrders@2022-11-01' = {
  name: 'ro-sql-vcores'
  location: 'global'
  properties: {
    reservedResourceType: 'SqlDatabases'
    term: 'P1Y'
    quantity: 24  // Total vCores
    displayName: 'SQL vCore Reservations'
    appliedScopeType: 'Shared'
  }
}
```

### Step 3: Right-Size Virtual Machines

```bicep
// Enable Azure Monitor for VM metrics analysis
resource vmInsights 'Microsoft.Insights/dataCollectionRules@2022-06-01' = {
  name: 'dcr-vm-performance'
  location: location
  properties: {
    dataSources: {
      performanceCounters: [
        {
          name: 'VMPerformance'
          streams: ['Microsoft-Perf']
          samplingFrequencyInSeconds: 60
          counterSpecifiers: [
            '\\Processor Information(_Total)\\% Processor Time'
            '\\Memory\\% Committed Bytes In Use'
            '\\LogicalDisk(_Total)\\% Disk Read Time'
            '\\LogicalDisk(_Total)\\% Disk Write Time'
          ]
        }
      ]
    }
    destinations: {
      logAnalytics: [
        {
          workspaceResourceId: logAnalytics.id
          name: 'vmLogs'
        }
      ]
    }
  }
}
```

```kusto
// KQL query to find oversized VMs
Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| where TimeGenerated > ago(30d)
| summarize AvgCPU = avg(CounterValue),
            MaxCPU = max(CounterValue),
            P95CPU = percentile(CounterValue, 95)
by Computer
| where P95CPU < 20  // VMs with P95 CPU < 20% are oversized
| order by AvgCPU asc

// Memory utilization
Perf
| where ObjectName == "Memory" and CounterName == "% Committed Bytes In Use"
| where TimeGenerated > ago(30d)
| summarize AvgMemory = avg(CounterValue),
            MaxMemory = max(CounterValue),
            P95Memory = percentile(CounterValue, 95)
by Computer
| where P95Memory < 40  // VMs with P95 Memory < 40% can be downsized
```

```bash
# Resize VM based on analysis
az vm resize \
  --resource-group rg-production \
  --name vm-web-01 \
  --size Standard_D2s_v5  # Downsize from D4s_v5

# Savings: D4s_v5 ($140/mo) → D2s_v5 ($70/mo) = 50% per VM
```

### Step 4: Auto-Shutdown for Non-Production

```bicep
// Auto-shutdown for dev/test VMs
resource autoShutdown 'Microsoft.DevTestLab/schedules@2018-09-15' = {
  name: 'shutdown-computevm-${vmName}'
  location: location
  properties: {
    status: 'Enabled'
    taskType: 'ComputeVmShutdownTask'
    dailyRecurrence: {
      time: '1900'  // 7 PM
    }
    timeZoneId: 'Eastern Standard Time'
    notificationSettings: {
      status: 'Enabled'
      timeInMinutes: 30
      emailRecipient: 'team@contoso.com'
    }
    targetResourceId: vm.id
  }
}

// Start VMs on schedule using Automation
resource automationRunbook 'Microsoft.Automation/automationAccounts/runbooks@2022-08-08' = {
  parent: automationAccount
  name: 'Start-DevVMs'
  location: location
  properties: {
    runbookType: 'PowerShell'
    logProgress: true
    logVerbose: false
    publishContentLink: {
      uri: 'https://raw.githubusercontent.com/contoso/runbooks/main/Start-DevVMs.ps1'
    }
  }
}

resource startSchedule 'Microsoft.Automation/automationAccounts/schedules@2022-08-08' = {
  parent: automationAccount
  name: 'StartDevVMsWeekday'
  properties: {
    startTime: '2024-01-01T08:00:00+00:00'
    frequency: 'Week'
    interval: 1
    timeZone: 'Eastern Standard Time'
    advancedSchedule: {
      weekDays: ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday']
    }
  }
}

// Savings: Dev VMs running 10hrs/day × 5 days = 50hrs vs 720hrs
// = 93% cost reduction for dev VMs
```

### Step 5: Optimize Azure SQL

```sql
-- Identify unused indexes
SELECT
    OBJECT_NAME(i.object_id) AS TableName,
    i.name AS IndexName,
    ius.user_seeks,
    ius.user_scans,
    ius.user_lookups,
    ius.user_updates
FROM sys.indexes i
JOIN sys.dm_db_index_usage_stats ius
    ON i.object_id = ius.object_id AND i.index_id = ius.index_id
WHERE OBJECTPROPERTY(i.object_id, 'IsUserTable') = 1
    AND ius.user_seeks = 0
    AND ius.user_scans = 0
    AND ius.user_lookups = 0
ORDER BY ius.user_updates DESC;

-- Check DTU/vCore utilization
SELECT
    AVG(avg_cpu_percent) as AvgCPU,
    MAX(avg_cpu_percent) as MaxCPU,
    AVG(avg_data_io_percent) as AvgIO,
    AVG(avg_memory_usage_percent) as AvgMemory
FROM sys.dm_db_resource_stats
WHERE end_time > DATEADD(day, -14, GETUTCDATE());
```

```bicep
// Right-size based on analysis
resource sqlDatabase 'Microsoft.Sql/servers/databases@2023-05-01-preview' = {
  parent: sqlServer
  name: 'appdb'
  location: location
  sku: {
    // Before: Premium P4 (500 DTU) - $1,860/month
    // After: Standard S3 (100 DTU) - $150/month
    // Or: General Purpose 2 vCore - $370/month

    name: 'GP_S_Gen5'  // Serverless for variable workloads
    tier: 'GeneralPurpose'
    family: 'Gen5'
    capacity: 2
  }
  properties: {
    autoPauseDelay: 60  // Pause after 1 hour of inactivity
    minCapacity: 0.5     // Minimum 0.5 vCores when active
    zoneRedundant: false // Disable for non-prod
  }
}

// Use Elastic Pools for multiple databases
resource elasticPool 'Microsoft.Sql/servers/elasticPools@2023-05-01-preview' = {
  parent: sqlServer
  name: 'pool-shared'
  location: location
  sku: {
    name: 'GP_Gen5'
    tier: 'GeneralPurpose'
    family: 'Gen5'
    capacity: 4  // 4 vCores shared across databases
  }
  properties: {
    perDatabaseSettings: {
      minCapacity: 0
      maxCapacity: 2
    }
  }
}
// 10 databases × $370/month = $3,700
// vs Elastic Pool 4 vCore: $740/month = 80% savings
```

### Step 6: Optimize AKS

```bicep
// Right-size AKS node pools
resource aksCluster 'Microsoft.ContainerService/managedClusters@2023-05-01' = {
  name: aksName
  location: location
  properties: {
    agentPoolProfiles: [
      {
        name: 'system'
        count: 2  // Reduced from 3
        vmSize: 'Standard_D2s_v5'  // Reduced from D4s_v5
        mode: 'System'
        enableAutoScaling: true
        minCount: 2
        maxCount: 3
      }
      {
        name: 'workload'
        count: 3
        vmSize: 'Standard_D4s_v5'
        mode: 'User'
        enableAutoScaling: true
        minCount: 2
        maxCount: 10  // Scale up only when needed

        // Use spot instances for non-critical workloads
        scaleSetPriority: 'Spot'
        spotMaxPrice: -1  // Pay up to on-demand price
        scaleSetEvictionPolicy: 'Delete'

        nodeLabels: {
          'workload-type': 'batch'
        }
        nodeTaints: [
          'kubernetes.azure.com/scalesetpriority=spot:NoSchedule'
        ]
      }
    ]
  }
}

// Spot instance savings: ~60-90% vs regular VMs
```

```yaml
# Kubernetes resource optimization
apiVersion: v1
kind: LimitRange
metadata:
  name: resource-limits
  namespace: production
spec:
  limits:
    - default:
        cpu: "500m"
        memory: "512Mi"
      defaultRequest:
        cpu: "100m"
        memory: "128Mi"
      type: Container

---
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

### Step 7: Set Up Cost Alerts and Budgets

```bicep
// Budget with alerts
resource budget 'Microsoft.Consumption/budgets@2023-05-01' = {
  name: 'monthly-budget'
  properties: {
    category: 'Cost'
    amount: 120000  // Target: $120K (20% reduction)
    timeGrain: 'Monthly'
    timePeriod: {
      startDate: '2024-01-01'
      endDate: '2025-12-31'
    }
    filter: {
      dimensions: {
        name: 'ResourceGroup'
        operator: 'In'
        values: ['rg-production', 'rg-staging', 'rg-development']
      }
    }
    notifications: {
      Actual_GreaterThan_80_Percent: {
        enabled: true
        operator: 'GreaterThan'
        threshold: 80
        contactEmails: ['finance@contoso.com', 'platform@contoso.com']
        contactRoles: ['Owner', 'Contributor']
        thresholdType: 'Actual'
      }
      Forecasted_GreaterThan_100_Percent: {
        enabled: true
        operator: 'GreaterThan'
        threshold: 100
        contactEmails: ['finance@contoso.com', 'cto@contoso.com']
        thresholdType: 'Forecasted'
      }
    }
  }
}

// Resource group level budget
resource rgBudget 'Microsoft.Consumption/budgets@2023-05-01' = {
  name: 'dev-budget'
  scope: resourceGroup('rg-development')
  properties: {
    category: 'Cost'
    amount: 5000
    timeGrain: 'Monthly'
    notifications: {
      Actual_GreaterThan_90_Percent: {
        enabled: true
        operator: 'GreaterThan'
        threshold: 90
        contactEmails: ['dev-lead@contoso.com']
      }
    }
  }
}
```

### Step 8: Implement Cost Tagging

```bicep
// Enforce tagging policy
resource taggingPolicy 'Microsoft.Authorization/policyDefinitions@2021-06-01' = {
  name: 'require-cost-tags'
  properties: {
    policyType: 'Custom'
    mode: 'Indexed'
    displayName: 'Require cost center and environment tags'
    policyRule: {
      if: {
        anyOf: [
          {
            field: 'tags[CostCenter]'
            exists: 'false'
          }
          {
            field: 'tags[Environment]'
            exists: 'false'
          }
          {
            field: 'tags[Owner]'
            exists: 'false'
          }
        ]
      }
      then: {
        effect: 'deny'
      }
    }
  }
}

// Apply tags to all resources
resource tagPolicy 'Microsoft.Resources/tags@2021-04-01' = {
  name: 'default'
  properties: {
    tags: {
      Environment: environment
      CostCenter: costCenter
      Owner: ownerEmail
      Project: projectName
      CreatedBy: 'Bicep'
      CreatedDate: utcNow('yyyy-MM-dd')
    }
  }
}
```

### Cost Optimization Summary

```
Optimization Results:

BEFORE ($150,000/month):
├── VMs: $45,000
├── SQL: $30,000
├── Storage: $22,500
├── AKS: $25,500
├── App Services: $15,000
└── Other: $12,000

AFTER ($100,000/month):
├── VMs: $25,000 (-44%)
│   ├── Reserved instances: -$10,000
│   ├── Right-sizing: -$5,000
│   └── Auto-shutdown dev: -$5,000
├── SQL: $18,000 (-40%)
│   ├── Elastic pools: -$8,000
│   └── Serverless: -$4,000
├── Storage: $15,000 (-33%)
│   └── Lifecycle policies: -$7,500
├── AKS: $18,000 (-29%)
│   ├── Spot instances: -$5,000
│   └── Autoscaling: -$2,500
├── App Services: $12,000 (-20%)
│   └── Right-size non-prod: -$3,000
└── Other: $12,000

TOTAL SAVINGS: $50,000/month (33% reduction)
ANNUAL SAVINGS: $600,000
```


## Cost Optimization Strategies

| Strategy | Savings | Effort | Risk |
|----------|---------|--------|------|
| Reserved Instances | 30-60% | Low | Commitment |
| Spot Instances | 60-90% | Medium | Interruption |
| Right-sizing | 20-50% | Medium | Performance |
| Auto-shutdown | 50-90% | Low | Availability |
| Serverless | Variable | High | Architecture |

## Quick Wins Checklist

| Action | Expected Savings |
|--------|-----------------|
| Delete unattached disks | $5-50/disk/month |
| Stop idle VMs | 100% of compute |
| Resize oversized VMs | 30-50% per VM |
| Enable auto-shutdown | 60% for dev/test |
| Use reserved instances | 30-60% for prod |
| Implement lifecycle policies | 30-90% on storage |


---

### Quick Check

**Why should you analyze at least 14-30 days of metrics before right-sizing a VM?**

   A. Azure requires 14 days of data for recommendations
-> B. **To capture workload patterns including peaks, batch jobs, and weekly cycles that might not appear in shorter timeframes**
   C. VMs take 14 days to stabilize after deployment
   D. Reserved instances require 14 days of usage data

<details>
<summary>See Answer</summary>

Workloads often have patterns that only appear over time: monthly batch processing, weekly reports, end-of-month spikes, or marketing campaigns. If you right-size based on just a few days of low usage, you might downgrade a VM that needs more resources during peak periods. Analyzing 14-30 days (or longer for monthly patterns) captures the true resource requirements and helps avoid performance issues after right-sizing.

</details>

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*
