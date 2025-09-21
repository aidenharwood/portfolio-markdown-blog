---
title: Dynamic Object Property Traversal in Bice
date: 08-21-2025
slug: bicep-dynamic-object-traversal
excerpt: How I devised a way to traverse object properties on-the-fly in Bicep
---

# The Problem
When creating Key Vault Secrets, I found referencing resource properties to be fairly cumbersome, particularly when the resource is managed elsewhere due to several factors:
1. `existing` resources can only be used once per resource per parent deployment
2. Usage of `reference` or `list` functions must validate property references at compile-time
3. Either approach must be declarative within the `.bicep` file

# The Solution
Taking inspiration from how it is done in JavaScript, I set about finding a way to pass a resource ID, API version (because this is mandatory within the `reference` and `list` function) and dot-path property notation into a function, and return either the value or null if the operation failed.

```bicep

@export()
func findProperty(obj object, path string) string? =>
  reduce(
    map(split(path, '.'), (item) => {
      '${item}': null
    }),
    obj,
    (cur, seg) =>
      cur[?objectKeys(seg)[0]] ?? shallowMerge(map(array(cur), (val, i) => { '${i}': val }))[?objectKeys(seg)[0]]
  )

output example = findProperty(reference('/some/resource/id', '2025-01-01', 'full'), 'properties.array.1.property')
### Assume the returned property from reference() is:
### {
###     "properties": {
###         "array": [
###             { "property": "example1" },
###             { "property": "example2" },
###         ]
###     }    
### }
###
### The example output above would return "example2"
```
## What does this do?
1. Splits the property reference into its constituent parts
2. Use a reducer to traverse the object, attempting to either extract an array index or object property
3. Return the last property in the chain, or null upon failure

## Why?
So that instead of declaring references or existing resources in a length Bicep file, I can now do something along the lines of:
```bicep
param keyVaultSecrets = {
    'some-resource-fqdn': {
        valueRef: {
            resourceId: '${subscriptionId}/resourceGroups/${resourceGroupName}/Some.Provider/${someResourceName}'
            apiVersion: '2025-01-01'
            propertyPath: 'properties.fullyQualifiedDomainName'
            refType: 'reference' # | 'list'
        }
    }
}
...
```

as opposed to

```bicep
existing resource someResource 'Some.Provider' = {
    name: someResourceName
    scope: resourceGroup(subscriptionId, resourceGroupName)
}

param keyVaultSecrets = {
    'some-resource-fqdn': {
        value: someResource.properties.fqdn
    }
}
...
```

This is particularly useful as my current preferred way to deploy resources, is to avoid `.bicep` files entirely, and deploy directly via `.bicepparam` files via [using statements](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/bicep-using#the-using-statement) that reference modules in a container registry, meaning the readily-available options are no longer appropriate or realistic.