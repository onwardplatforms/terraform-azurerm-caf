# Deprecated Resources Documentation

This document tracks deprecated Terraform resources that need to be updated to maintain compatibility with current Azure provider versions.

## Critical Deprecations Requiring Immediate Action

### 1. azurerm_app_service_virtual_network_swift_connection

**Status**: DEPRECATED since azurerm provider v3.0
**Impact**: High - Used in 3 locations
**Timeline**: Must be updated before azurerm provider v4.0

#### Current Usage Locations:
1. **`app_services.tf:43`** - Root-level app service VNet integration
2. **`modules/logic_app/standard/module.tf:47`** - Logic App Standard VNet integration
3. **`modules/webapps/function_app/module.tf:198`** - Function App VNet integration

#### Migration Path:
Replace with `azurerm_app_service_virtual_network_swift_connection` inline configuration within the app service resource.

**Before (Deprecated):**
```hcl
resource "azurerm_app_service_virtual_network_swift_connection" "vnet_config" {
  app_service_id = azurerm_linux_web_app.example.id
  subnet_id      = azurerm_subnet.example.id
}
```

**After (Current):**
```hcl
resource "azurerm_linux_web_app" "example" {
  # ... other configuration

  virtual_network_subnet_id = azurerm_subnet.example.id
}
```

#### Action Required:
1. **App Services**: Update `modules/webapps/appservice/module.tf` to include `virtual_network_subnet_id` parameter
2. **Logic App Standard**: Update `modules/logic_app/standard/module.tf` to use inline VNet integration
3. **Function Apps**: Update `modules/webapps/function_app/module.tf` to use inline VNet integration
4. **Root Configuration**: Remove standalone `azurerm_app_service_virtual_network_swift_connection` resources from root files

#### Compatibility Notes:
- Requires azurerm provider >= 3.0
- Configuration structure remains the same (subnet_id mapping)
- No breaking changes to module interface expected

## Other Known Deprecations

### 2. Public IP availability_zone → zones
**Status**: Deprecated in azurerm provider 2.99+
**Files**: Multiple public IP configurations
**Action**: Update `availability_zone` string to `zones` list

### 3. VPN Gateway propagated_route_tables → propagated_route_table
**Status**: Deprecated
**Files**: VPN gateway connection configurations
**Action**: Rename field in configuration files

### 4. Data Factory data_factory_name → data_factory_id
**Status**: Deprecated in azurerm provider 2.98+
**Files**: Data Factory linked services and datasets
**Action**: Update reference method from name to ID

## Migration Priority

1. **HIGH**: azurerm_app_service_virtual_network_swift_connection (blocking for provider v4.0)
2. **MEDIUM**: Public IP zones configuration (deprecated but functional)
3. **MEDIUM**: VPN Gateway route table configuration
4. **LOW**: Data Factory references (backward compatible)

## Testing Strategy

Before implementing migrations:
1. Enable all pre-commit hooks for validation
2. Run `terraform plan` to identify affected resources
3. Test in development environment first
4. Document expected resource recreations

## Provider Compatibility Matrix

| Resource Type | Deprecated In | Removed In | Current Alternative |
|---------------|---------------|------------|-------------------|
| app_service_virtual_network_swift_connection | v3.0 | v4.0 | Inline virtual_network_subnet_id |
| availability_zone (Public IP) | v2.99 | TBD | zones parameter |
| propagated_route_tables | v3.0 | TBD | propagated_route_table |
| data_factory_name | v2.98 | TBD | data_factory_id |

---

**Next Review**: Update this document after completing migrations
**Owner**: Infrastructure Team
**Last Updated**: Current deployment