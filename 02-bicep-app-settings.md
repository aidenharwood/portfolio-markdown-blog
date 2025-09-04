---
title: Avoiding Azure Web App environment variable overwrites in Bicep
date: 04-09-2025
slug: bicep-app-settings
excerpt: How I solved a known gripe with the way environment variables are handled in Azure Web Apps
---
## The Problem

Bicep is inherently an incremental deployment, meaning that changes are generally additive. There are exceptions to this, when making changes to nested resources; for example, changing
<table>
<tr>
<td> From </td> <td> To </td>
</tr>
<tr>
<td>
```bicep
resource someResource 'Microsoft.Provider/resource@2025-04-09' = {
  name: 'resource-name'
  properties: {
    propertyA: 'valueA'
  }
}
```
</td>
<td>    
```bicep
resource someResource 'Microsoft.Provider/resource@2025-04-09' = {
  name: 'resource-name'
  properties: {
    propertyB: 'valueB'
  }
```
</td>
</tr>
</table>
would still leave `propertyA` in place. However, changing
<table>
<tr>
<td> From </td> <td> To </td>
</tr>
<tr>
<td>
```bicep
resource someResource 'Microsoft.Provider/resource@2025-04-09' = {
  name: 'resource-name'
  properties: {
    property: {
        propertyA: 'valueA'
    }
  }
}
```
</td>
<td>    
```bicep
resource someResource 'Microsoft.Provider/resource@2025-04-09' = {
  name: 'resource-name'
  properties: {
    property: {
        propertyB: 'valueB'
    }
  }
}
```
</td>
</tr>
</table>
would replace `property.propertyA` with `property.propertyB`. This is particularly a problem with Azure Web App environment variables, as often, environment variables may be paired with app code rather than infra code (or even entirely separately); meaning that subsequent bicep runs may completely wipe a developer's configuration.

## The Solution
In order to avoid overwriting existing environment variables when deploying updates with Bicep, you can use a helper module to retrieve the current app settings and merge them with your new values. This approach ensures that variables are set once with future changes ignored.

### list-helper.bicep
```bicep
targetScope = 'subscription'

param resourceId string

param apiVersion string

var list = az.list(resourceId, apiVersion)

output list object = list
```

### app-settings.bicep
```bicep
targetScope = 'resourceGroup'

param appName string
param appSettings object
param appResourceId string

module listHelper 'list-helper.bicep' = {
    name: '${deployment().name}-appsettings'    
    scope: subscription()
    params: {
        apiVersion: '2023-12-01'
        resourceId: appResourceId
    }
}

resource appSettingsConfig 'Microsoft.Web/sites/config@2023-12-01' = if (slot == null) {
  name: '${appName}/appsettings'
  kind: 'string'
  properties: union(appSettings, listHelper.outputs.list.properties)
}
```

> This is based on assumption, we are utilising the [union](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/bicep-functions-object#union) function, which overwrites any common properties of `n` with the value in `n+1`.
> This module can be further adapted to make exceptions but this is intended as a starting point