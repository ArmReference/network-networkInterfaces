# network-networkInterfaces
Reference deployment
```
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "subnetId": {
      "type": "string"
    },
    "StorageAccountName": {
      "type": "string"
    },
    "ContainerName": {
      "type": "string"
    },
    "SasToken": {
      "type": "string"
    }
  },
  "variables": {
    "Provider": "/Microsoft.Network",
    "Resource": "/networkInterfaces",
    "templateUri": "[concat('https://',parameters('StorageAccountName'),'.blob.core.windows.net/',parameters('ContainerName'),variables('Provider'),variables('Resource'))]"
  },
  "resources": [
    {
      "name": "BuildIpConfigurations",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('templateUri'), '/ipConfigurations.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "name": {
            "value": "ipConfig1"
          },
          "subnet": {
            "value": "[parameters('subnetId')]"
          }
        }
      }
    },
    {
      "name": "DeployNetworkInterface",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [
        "[concat('Microsoft.Resources/deployments/','BuildIpConfigurations')]"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('templateUri'), '/networkInterfaces.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "name": {
            "value": "NetworkInterface"
          },
          "ipConfigurations": {
            "value": "[createarray(reference('BuildIpConfigurations').outputs.ipConfigurations.value)]"
          },
          "tags": {
            "value": "[json('{\"TagName\": \"TagValue\"}')]"
          }
        }
      }
    }
  ],
  "outputs": {
    "networkInterfaces": {
      "type": "object",
      "value": "[reference('DeployNetworkInterface').outputs.networkInterfaces.value]"
    }
  }
}
```
