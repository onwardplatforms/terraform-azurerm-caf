# Azure Cloud Adoption Framework (CAF) Terraform Module

> **Status**: This solution, offered by the Open-Source community, will no longer receive contributions from Microsoft. Customers are encouraged to transition to [Microsoft Azure Verified Modules](https://aka.ms/avm) for Microsoft support and updates.

A comprehensive Terraform module for deploying Azure infrastructure following the Cloud Adoption Framework (CAF) best practices. This module provides enterprise-grade infrastructure patterns with support for 35+ Azure service categories.

## 🏗️ Module Architecture

### Core Components

```
├── main.tf                 # Provider configuration and core data sources
├── variables.tf           # Input variable definitions (450+ variables)
├── locals.tf             # Complex object transformation logic
├── locals.combined_objects.tf # Cross-service object combination
├── versions.tf           # Terraform and provider version constraints
└── outputs/              # Service-specific outputs across 173 files
```

### Service Categories

| Category | Services | Root Files |
|----------|----------|------------|
| **Compute** | AKS, VMs, Container Apps, Batch, VMware, WVD | 15+ files |
| **Networking** | VNet, Firewall, Gateway, DNS, Load Balancer | 30+ files |
| **Security** | Key Vault, Managed Identity, Encryption, Sentinel | 13+ files |
| **Database** | SQL, PostgreSQL, MySQL, CosmosDB, Databricks | 15+ files |
| **Analytics** | Synapse, Data Factory, Purview, Search | 8+ files |
| **Storage** | Storage Account, NetApp, Backup Vault | 6+ files |
| **Identity** | Azure AD, AAD B2C, Service Principals | 12+ files |
| **Messaging** | Service Bus, Event Hub, SignalR | 8+ files |
| **Web Apps** | App Service, Function Apps, Logic Apps | 10+ files |
| **Monitoring** | Log Analytics, Application Insights, Alerts | 6+ files |

## 🚀 Quick Start

### Module Usage

```hcl
module "caf" {
  source  = "aztfmod/caf/azurerm"
  version = "~>5.7.0"

  # Core Configuration
  global_settings = {
    default_region = "region1"
    regions = {
      region1 = "southeastasia"
      region2 = "eastasia"
    }
    random_length = 4
  }

  # Resource Groups
  resource_groups = {
    rg1 = {
      name   = "my-resource-group"
      region = "region1"
    }
  }

  # Service-specific configurations
  networking = {
    virtual_networks = {
      vnet1 = {
        address_space = ["10.0.0.0/16"]
        subnets = {
          subnet1 = {
            address_prefixes = ["10.0.1.0/24"]
          }
        }
      }
    }
  }

  security = {
    keyvaults = {
      kv1 = {
        sku = "standard"
      }
    }
  }
}
```

### Provider Configuration

```hcl
terraform {
  required_version = ">= 1.3.5"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.114.0"
    }
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.43.0"
    }
  }
}

provider "azurerm" {
  features {}
}

# Required for cross-tenant scenarios
provider "azurerm" {
  alias = "vhub"
  features {}
}
```

## 🔧 Configuration Structure

### Global Settings

```hcl
variable "global_settings" {
  description = "Global configuration for the deployment"
  default = {
    passthrough    = false
    random_length  = 4
    default_region = "region1"
    regions = {
      region1 = "southeastasia"
      region2 = "eastasia"
    }
  }
}
```

### Service Configuration Pattern

Each service follows a consistent configuration pattern:

```hcl
# Example: Virtual Machine configuration
compute = {
  virtual_machines = {
    vm1 = {
      resource_group_key = "rg1"
      os_type           = "windows"
      size              = "Standard_D2s_v3"

      # Network configuration
      networking_interfaces = {
        nic1 = {
          vnet_key           = "vnet1"
          subnet_key         = "subnet1"
          primary            = true
        }
      }

      # Operating system
      virtual_machine_settings = {
        windows = {
          admin_username = "adminuser"
          # Additional Windows-specific settings
        }
      }
    }
  }
}
```

## 🏭 Module Organization

### Root-Level Resources (173 files)
Each Azure service has dedicated root-level `.tf` files following the pattern:
- `{service_category}_{service_name}.tf`
- Example: `compute_aks_clusters.tf`, `networking_virtual_wan.tf`

### Module Structure (35+ modules)
```
modules/
├── compute/
│   ├── aks_clusters/
│   ├── virtual_machines/
│   └── container_apps/
├── networking/
│   ├── virtual_networks/
│   ├── application_gateway/
│   └── firewall/
├── security/
│   ├── keyvault/
│   ├── managed_identity/
│   └── sentinel/
└── databases/
    ├── mssql_server/
    ├── cosmosdb/
    └── postgresql_server/
```

### Locals Pattern
Complex object transformation handled in `locals.tf`:

```hcl
# Example: Combined networking objects
combined_objects_networking = {
  for key, value in var.networking : key => {
    virtual_networks = module.virtual_networks[key]
    subnets         = local.combined_subnets[key]
    # Additional networking objects...
  }
}
```

## 🔐 Security Features

### Sensitive Data Handling
20+ outputs marked as sensitive:
```hcl
output "instrumentation_key" {
  value     = azurerm_application_insights.app_insights.instrumentation_key
  sensitive = true
}
```

### Key Vault Integration
```hcl
security = {
  keyvaults = {
    kv1 = {
      sku = "premium"

      # Network restrictions
      network_rules = {
        bypass                     = "AzureServices"
        default_action             = "Deny"
        ip_rules                  = ["10.0.0.0/8"]
        virtual_network_subnet_ids = [module.vnet.subnet_ids["subnet1"]]
      }

      # Access policies
      access_policies = {
        policy1 = {
          object_id = data.azuread_client_config.current.object_id
          key_permissions = ["Get", "List", "Create"]
          secret_permissions = ["Get", "List", "Set"]
        }
      }
    }
  }
}
```

## 📊 Dependency Management

### Cross-Service Dependencies
```hcl
# Application Gateway depends on networking
module "application_gateways" {
  depends_on = [module.networking]

  # Configuration...
}

# Explicit dependency ordering
module "keyvault_access_policies_azuread_apps" {
  depends_on = [module.keyvault_access_policies]

  # Prevents circular dependencies
}
```

### Time Delays for Azure Resource Propagation
```hcl
resource "time_sleep" "after_azurerm_firewall_policies" {
  depends_on = [azurerm_firewall_policy.policy]

  create_duration = "60s"
}
```

## 🎯 Naming Conventions

Uses Azure CAF naming conventions with `azurecaf_name`:

```hcl
data "azurecaf_name" "vm_name" {
  name          = var.name
  resource_type = "azurerm_windows_virtual_machine"
  prefixes      = var.global_settings.prefixes
  random_length = var.global_settings.random_length
  clean_input   = true
  passthrough   = var.global_settings.passthrough
}

resource "azurerm_windows_virtual_machine" "vm" {
  name = data.azurecaf_name.vm_name.result
  # Additional configuration...
}
```

## 🧪 Testing & Validation

### Pre-commit Hooks
```yaml
repos:
  - repo: https://github.com/antonbabenko/pre-commit-terraform
    hooks:
    - id: terraform_fmt      # Code formatting
    - id: terraform_docs     # Documentation generation
    - id: terraform_tflint   # Linting
    - id: terraform_validate # Syntax validation
    - id: terraform_tfsec    # Security scanning
    - id: checkov           # Compliance checking
```

### Example Configurations
The `examples/` directory provides 60+ configuration examples:
- Basic service deployments
- Advanced multi-service scenarios
- Security hardening patterns
- Network isolation examples

## 🔄 Version Management

### Current Requirements
- **Terraform**: >= 1.3.5
- **AzureRM Provider**: ~> 3.114.0
- **Azure AD Provider**: ~> 2.43.0

### Upgrade Path
See [UPGRADE.md](./UPGRADE.md) for version-specific migration instructions.

### Breaking Changes
See [CHANGELOG.md](./CHANGELOG.md) for detailed version history.

## 🚨 Known Issues & Limitations

### Deprecated Resources (Requires Action)
1. **azurerm_app_service_virtual_network_swift_connection** - 3 locations
2. **Public IP availability_zone** → **zones** migration needed
3. **VPN Gateway propagated_route_tables** → **propagated_route_table**

See [DEPRECATED_RESOURCES.md](./DEPRECATED_RESOURCES.md) for migration guidance.

### Current Limitations
- No built-in compliance policy enforcement
- Backend configuration must be externalized
- Complex locals may impact maintainability

## 📚 Documentation

- **[Examples](./examples/)** - 60+ service configuration examples
- **[Modules](./modules/)** - Individual module documentation
- **[UPGRADE.md](./UPGRADE.md)** - Version migration guide
- **[CHANGELOG.md](./CHANGELOG.md)** - Version history
- **[DEPRECATED_RESOURCES.md](./DEPRECATED_RESOURCES.md)** - Deprecation tracking

## 🤝 Contributing

This project welcomes contributions. Please check the [WIKI](https://github.com/aztfmod/terraform-azurerm-caf/wiki) for:
- Coding standards
- Common patterns
- PR checklist

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**Maintained by**: Open-Source Community
**Status**: Migration to [Azure Verified Modules](https://aka.ms/avm) recommended
**Architecture**: Enterprise-grade, production-ready Azure infrastructure patterns