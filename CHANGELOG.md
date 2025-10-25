# Changelog

All notable changes to the Azure Cloud Adoption Framework (CAF) Terraform module will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Version constraint for null provider (~> 3.2.0)
- Enabled security scanning tools in pre-commit configuration (TFSec, Checkov)
- Comprehensive security and validation tooling

### Changed
- Updated pre-commit configuration to enable terraform_tflint, terraform_validate, terraform_tfsec, and checkov

### Security
- Enhanced security posture with mandatory security scanning tools

## [5.7.0] - 2023

### Changed
- **BREAKING**: Migrated from `azurecaf_name` resource to data source for improved plan-time visibility
- **BREAKING**: Application Gateway `priority` field now mandatory for v2 SKUs
- **BREAKING**: Public IP `availability_zone` becomes `zones` (no longer supports "No-Zone", "Zone-Redundant")
- **BREAKING**: VPN Gateway connections - `propagated_route_tables` deprecated in favor of `propagated_route_table`

### Removed
- Multiple `azurecaf_name` resources will be destroyed during upgrade (expected behavior)

### Requirements
- Minimum rover version: 1.1.x (lower versions no longer supported)

## [5.6.0] - 2022

### Changed
- **BREAKING**: SignalR `features` block deprecated in favor of individual properties
- **BREAKING**: Data Factory `data_factory_name` replaced with `data_factory_id`
- **BREAKING**: Service Bus field deprecations: `namespace_name` ’ `namespace_id`, `topic_name` ’ `topic_id`, `subscription_name` ’ `subscription_id`
- **BREAKING**: API Management `proxy` block deprecated in favor of `gateways` for multiple gateway support
- **BREAKING**: Network Security Group refactoring causes recreation (temporary disconnection expected)

### Added
- Azure Virtual Desktop token method support via `azurerm_virtual_desktop_host_pool_registration_info`
- Network Watcher Flow Log `name` attribute (mandatory, forces recreation)

### Deprecated
- Data Factory literal name references (use `id` attribute instead)
- Service Bus namespace/topic/subscription name fields

## [5.5.0] - 2022

### Changed
- **BREAKING**: Dropped support for Terraform 0.13 and 0.14
- **BREAKING**: Requires provider configuration aliases (Terraform 0.15+)

### Added
- Provider configuration alias support for `azurerm.vhub`

### Requirements
- Terraform >= 0.15
- Landing zones version >= 2112.0

## [5.4.5] - 2021

### Deprecated
- Function App `client_affinity_enabled` attribute (no longer configurable)

### Added
- Support for azurerm provider 2.81.0

## [5.4.4] - 2021

### Known Issues
- Regression: Cross-tenant, cross-subscription peering between vhub and vwans not supported (fixed in 5.5.0)

## [5.4.0] - 2021

### Changed
- **BREAKING**: Azure Container Registry georeplications structure updated
- **BREAKING**: Azure Front Door custom HTTPS configuration moved to standalone resource
- **BREAKING**: Public IP `zone` parameter deprecated for `availability_zone`
- **BREAKING**: Updated RBAC structures (in-place updates, creates new assignments)

### Added
- Support for azurerm provider 2.64.0

## [5.3.0] - 2020

### Changed
- **BREAKING**: Network Security Group structure evolution
- Introduced NSG `version = 1` for standalone configuration and direct NIC stitching

## [4.21.0] - 2020

### Changed
- **BREAKING**: Virtual machines configuration structure updated
- `admin_user_key` replaced with `admin_username_key`

---

For detailed upgrade instructions, see [UPGRADE.md](./UPGRADE.md).

### Legend
- **BREAKING**: Changes that require configuration file updates
- **Security**: Security-related improvements
- **Deprecated**: Features marked for future removal