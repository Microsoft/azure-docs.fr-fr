---
title: Exemples de modèle Resource Manager destinés à Azure Monitor pour conteneurs
description: Exemples de modèle Azure Resource Manager permettant de déployer et de configurer Azure Monitor pour conteneurs.
ms.subservice: logs
ms.topic: sample
author: bwren
ms.author: bwren
ms.date: 05/18/2020
ms.openlocfilehash: 0c32ecc45d57f76a675b156543fdc5019cba509f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83853236"
---
# <a name="resource-manager-template-samples-for-azure-monitor-for-containers"></a>Exemples de modèle Resource Manager destinés à Azure Monitor pour conteneurs
Cet article contient des exemples de [modèle Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md) pour déployer et configurer l’agent Log Analytics destiné à des machines virtuelles dans Azure Monitor. Chaque exemple comprend un fichier de modèle et un fichier de paramètres avec des exemples de valeurs à fournir au modèle.

[!INCLUDE [azure-monitor-samples](../../../includes/azure-monitor-resource-manager-samples.md)]


## <a name="enable-for-aks-cluster"></a>Activer pour un cluster AKS
L’exemple suivant active Azure Monitor pour conteneurs sur un cluster AKS.


### <a name="template-file"></a>Fichier de modèle

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "aksResourceId": {
      "type": "string",
      "metadata": {
        "description": "AKS Cluster Resource ID"
      }
    },
    "aksResourceLocation": {
      "type": "string",
      "metadata": {
        "description": "Location of the AKS resource e.g. \"East US\""
      }
    },
    "aksResourceTagValues": {
      "type": "object",
      "metadata": {
        "description": "Existing all tags on AKS Cluster Resource"
      }
    },
    "workspaceResourceId": {
      "type": "string",
      "metadata": {
        "description": "Azure Monitor Log Analytics Resource ID"
      }
    }
  },
  "resources": [
    {
      "name": "[split(parameters('aksResourceId'),'/')[8]]",
      "type": "Microsoft.ContainerService/managedClusters",
      "location": "[parameters('aksResourceLocation')]",
      "tags": "[parameters('aksResourceTagValues')]",
      "apiVersion": "2018-03-31",
      "properties": {
        "mode": "Incremental",
        "id": "[parameters('aksResourceId')]",
        "addonProfiles": {
          "omsagent": {
            "enabled": true,
            "config": {
              "logAnalyticsWorkspaceResourceID": "[parameters('workspaceResourceId')]"
            }
          }
        }
      }
    }
  ]
}
```

### <a name="parameter-file"></a>Fichier de paramètres

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "aksResourceId": {
      "value": "/subscriptions/<SubscriptionId>/resourcegroups/<ResourceGroup>/providers/Microsoft.ContainerService/managedClusters/<ResourceName>"
    },
    "aksResourceLocation": {
      "value": "<aksClusterLocation>"
    },
    "workspaceResourceId": {
      "value": "/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroup>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>"
    },
    "aksResourceTagValues": {
      "value": {
        "<existing-tag-name1>": "<existing-tag-value1>",
        "<existing-tag-name2>": "<existing-tag-value2>",
        "<existing-tag-nameN>": "<existing-tag-valueN>"
      }
    }
  }
}
```


## <a name="enable-for-new-azure-red-hat-openshift-v3-cluster"></a>Activer pour un nouveau cluster Azure Red Hat OpenShift v3

### <a name="template-file"></a>Fichier de modèle

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "metadata": {
        "description": "Location"
      }
    },
    "clusterName": {
      "type": "string",
      "metadata": {
        "description": "Unique name for the cluster"
      }
    },
    "masterNodeCount": {
      "type": "int",
      "defaultValue": 3,
      "metadata": {
        "description": "number of master nodes"
      }
    },
    "computeNodeCount": {
      "type": "int",
      "defaultValue": 3,
      "metadata": {
        "description": "number of compute nodes"
      }
    },
    "infraNodeCount": {
      "type": "int",
      "defaultValue": 3,
      "metadata": {
        "description": "number of infra nodes"
      }
    },
    "aadTenantId": {
      "type": "string",
      "metadata": {
        "description": "The ID of an Azure Active Directory tenant"
      }
    },
    "aadClientId": {
      "type": "string",
      "metadata": {
        "description": "The ID of an Azure Active Directory client application"
      }
    },
    "aadClientSecret": {
      "type": "securestring",
      "metadata": {
        "description": "The secret of an Azure Active Directory client application"
      }
    },
    "aadCustomerAdminGroupId": {
      "type": "string",
      "metadata": {
        "description": "The Object ID of an Azure Active Directory Group that memberships will get synced into the OpenShift group 'osa-customer-admins'. If not specified, no cluster admin access will be granted."
      }
    },
    "workspaceResourceId": {
      "type": "string",
      "metadata": {
        "description": "Azure ResourceId of an existing Log Analytics Workspace"
      }
    }
  },
  "resources": [
    {
      "location": "[parameters('location')]",
      "name": "[parameters('clusterName')]",
      "type": "Microsoft.ContainerService/openShiftManagedClusters",
      "apiVersion": "2019-09-30-preview",
      "properties": {
        "openShiftVersion": "v3.11",
        "fqdn": "[concat(parameters('clusterName'), '.', parameters('location'), '.', 'cloudapp.azure.com')]",
        "networkProfile": {
          "vnetCidr": "10.0.0.0/8"
        },
        "authProfile": {
          "identityProviders": [
            {
              "name": "Azure AD",
              "provider": {
                "kind": "AADIdentityProvider",
                "clientId": "[parameters('aadClientId')]",
                "secret": "[parameters('aadClientSecret')]",
                "tenantId": "[parameters('aadTenantId')]",
                "customerAdminGroupId": "[parameters('aadCustomerAdminGroupId')]"
              }
            }
          ]
        },
        "masterPoolProfile": {
          "name": "master",
          "count": "[parameters('masterNodeCount')]",
          "subnetCidr": "10.0.0.0/24",
          "vmSize": "Standard_D4s_v3",
          "osType": "Linux"
        },
        "agentPoolProfiles": [
          {
            "role": "compute",
            "name": "compute",
            "count": "[parameters('computeNodeCount')]",
            "subnetCidr": "10.0.0.0/24",
            "vmSize": "Standard_D4s_v3",
            "osType": "Linux"
          },
          {
            "role": "infra",
            "name": "infra",
            "count": "[parameters('infraNodeCount')]",
            "subnetCidr": "10.0.0.0/24",
            "vmSize": "Standard_D4s_v3",
            "osType": "Linux"
          }
        ],
        "routerProfiles": [
          {
            "name": "default"
          }
        ],
        "monitorProfile": {
          "workspaceResourceID": "[parameters('workspaceResourceId')]",
          "enabled": true
        }
      }
    }
  ]
}
```

### <a name="parameter-file"></a>Fichier de paramètres

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "value": "<azure region of the cluster e.g. westcentralus>"
        },
        "clusterName": {
            "value": "<name of the aro cluster>"
        },
        "aadTenantId": {
            "value": "<id of an azure active directory tenant>"
        },
        "aadClientId": {
            "value": "<id of an azure active directory client application>"
        },
        "aadClientSecret": {
            "value": "<secret of an azure active directory client application  >"
        },
        "aadCustomerAdminGroupId": {
            "value": "<customer admin group id>"
        },
        "workspaceResourceId": {
            "value": "<resource id of an existing log analytics workspace>"
        },
        "masterNodeCount": {
            "value": "<number of master node e.g. 3>"
        },
        "computeNodeCount": {
            "value": "<number of compute nodes in agent pool profile e.g. 3>"
        },
        "infraNodeCount": {
            "value": "<number of infra nodes in agent pool profile e.g. 3>"
        }
    }
}
```

## <a name="enable-for-existing-azure-red-hat-openshift-v3-cluster"></a>Activer pour un cluster Azure Red Hat OpenShift v3 existant

### <a name="template-file"></a>Fichier de modèle

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "aroResourceId": {
      "type": "string",
      "metadata": {
        "description": "ARO Cluster Resource ID"
      }
    },
    "aroResourceLocation": {
      "type": "string",
      "metadata": {
        "description": "Location of the aro cluster resource e.g. westcentralus"
      }
    },
    "workspaceResourceId": {
      "type": "string",
      "metadata": {
        "description": "Azure Monitor Log Analytics Resource ID"
      }
    }
  },
  "resources": [
    {
      "name": "[split(parameters('aroResourceId'),'/')[8]]",
      "type": "Microsoft.ContainerService/openShiftManagedClusters",
      "location": "[parameters('aroResourceLocation')]",
      "apiVersion": "2019-09-30-preview",
      "properties": {
        "mode": "Incremental",
        "id": "[parameters('aroResourceId')]",
        "monitorProfile": {
          "enabled": true,
          "workspaceResourceID": "[parameters('workspaceResourceId')]"
        }
      }
    }
  ]
}
```

### <a name="parameter-file"></a>Fichier de paramètres

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "aroResourceId": {
      "value": "/subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.ContainerService/openShiftManagedClusters/<clusterName>"
    },
    "aroResourceLocation": {
      "value": "<azure region of the cluster e.g. westcentralus>"
    },
    "workspaceResourceId": {
      "value": "/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroup>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>"
    }
  }
}
```

## <a name="next-steps"></a>Étapes suivantes

* [Obtenir d’autres exemples de modèle pour Azure Monitor](resource-manager-samples.md).
* [En savoir plus sur Azure Monitor pour conteneurs](../insights/container-insights-overview.md).
