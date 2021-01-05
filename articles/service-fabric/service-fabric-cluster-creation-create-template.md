---
title: Créer un modèle de cluster Azure Service Fabric
description: Découvrez comment créer un modèle Resource Manager pour un cluster Service Fabric. Configurez la sécurité, Azure Key Vault et Azure Active Directory (Azure AD) pour l’authentification client.
ms.topic: conceptual
ms.date: 08/16/2018
ms.openlocfilehash: b028b5c1f32733146c071e50b24202e5bad945c0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86260490"
---
# <a name="create-a-service-fabric-cluster-resource-manager-template"></a>Création d’un modèle Service Fabric Cluster Resource Manager

Un [cluster Azure Service Fabric](service-fabric-deploy-anywhere.md) est un ensemble de machines virtuelles connecté au réseau, sur lequel vos microservices sont déployés et gérés. Un cluster Service Fabric s’exécutant dans Azure est une ressource Azure, et il est déployé, géré et surveillé avec Resource Manager.  Cet article décrit comment créer un modèle Resource Manager pour un cluster Service Fabric s’exécutant dans Azure.  Quand le modèle est terminé, vous pouvez [déployer le cluster sur Azure](service-fabric-cluster-creation-via-arm.md).

La sécurité du cluster est configurée lors de la première configuration du cluster et elle ne peut pas être modifiée ultérieurement. Avant de configurer un cluster, lisez les [scénarios de sécurité du cluster Service Fabric][service-fabric-cluster-security]. Dans Azure, Service Fabric utilise un certificat x509 pour sécuriser votre cluster et ses points de terminaison, authentifier les clients et chiffrer les données. Azure Active Directory est également recommandé pour sécuriser l’accès aux points de terminaison de gestion. Les locataires et les utilisateurs Azure AD doivent être créés avant de créer le cluster.  Pour plus d’informations, consultez [Configurer Azure AD pour authentifier les clients](service-fabric-cluster-creation-setup-aad.md).

Avant de déployer un cluster de production pour exécuter des charges de travail de production, lisez bien la [liste de vérification pour la préparation à la production](service-fabric-production-readiness-checklist.md).


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="create-the-resource-manager-template"></a>Créer le modèle Resource Manager
Des exemples de modèles Resource Manager sont disponibles dans les [exemples Azure sur GitHub](https://github.com/Azure-Samples/service-fabric-cluster-templates). Ces modèles peuvent être utilisés comme point de départ pour votre modèle de cluster.

Cet article utilise l’exemple de modèle et les paramètres du modèle du [cluster sécurisé à 5 nœuds][service-fabric-secure-cluster-5-node-1-nodetype]. Téléchargez *azuredeploy.json* et *azuredeploy.parameters.json* sur votre ordinateur, et ouvrez les deux fichiers dans votre éditeur de texte préféré.

> [!NOTE]
> Pour les clouds nationaux (Azure Government, Azure Chine, Azure Allemagne), vous devez également ajouter les `fabricSettings` suivants à votre modèle : `AADLoginEndpoint`, `AADTokenEndpointFormat` et `AADCertEndpointFormat`.

## <a name="add-certificates"></a>Ajout de certificats
Pour ajouter les certificats à un modèle Resource Manager de cluster, vous devez référencer le Key Vault qui contient les clés de certificat. Ajoutez ces paramètres et valeurs du coffre de clés dans un fichier de paramètres de modèle Resource Manager (*azuredeploy.parameters.json*).

### <a name="add-all-certificates-to-the-virtual-machine-scale-set-osprofile"></a>Ajouter tous les certificats à l’osProfile du groupe de machines virtuelles identiques
Chaque certificat qui est installé dans le cluster doit être configuré dans la section **osProfile** de la ressource de groupe identique (Microsoft.Compute/virtualMachineScaleSets). Cela permet d’indiquer au fournisseur de ressources d’installer le certificat sur les machines virtuelles. Cette installation inclut le certificat de cluster et tous les certificats de sécurité d’application que vous projetez d’utiliser pour vos applications :

```json
{
  "apiVersion": "[variables('vmssApiVersion')]",
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  ...
  "properties": {
    ...
    "osProfile": {
      ...
      "secrets": [
        {
          "sourceVault": {
            "id": "[parameters('sourceVaultValue')]"
          },
          "vaultCertificates": [
            {
              "certificateStore": "[parameters('clusterCertificateStorevalue')]",
              "certificateUrl": "[parameters('clusterCertificateUrlValue')]"
            },
            {
              "certificateStore": "[parameters('applicationCertificateStorevalue')",
              "certificateUrl": "[parameters('applicationCertificateUrlValue')]"
            },
            ...
          ]
        }
      ]
    }
  }
}
```

### <a name="configure-the-service-fabric-cluster-certificate"></a>Configurer le certificat de cluster Service Fabric

Le certificat d’authentification de cluster doit être configuré à la fois dans la ressource de cluster Service Fabric (Microsoft.ServiceFabric/clusters) et dans l’extension de Service Fabric pour les groupes de machines virtuelles identiques dans la ressource associée à ces derniers. Cette disposition permet au fournisseur de ressources de Service Fabric de configurer le certificat pour l’authentification du cluster et l’authentification du serveur pour les points de terminaison de gestion.

#### <a name="add-the-certificate-information-the-virtual-machine-scale-set-resource"></a>Ajouter les informations de certificat à la ressource de groupe de machines virtuelles identiques

```json
{
  "apiVersion": "[variables('vmssApiVersion')]",
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  ...
  "properties": {
    ...
    "virtualMachineProfile": {
      "extensionProfile": {
        "extensions": [
          {
            "name": "[concat('ServiceFabricNodeVmExt_',variables('vmNodeType0Name'))]",
            "properties": {
              ...
              "settings": {
                ...
                "certificate": {
                  "commonNames": ["[parameters('certificateCommonName')]"],
                  "x509StoreName": "[parameters('clusterCertificateStoreValue')]"
                },
                ...
              }
            }
          }
        ]
      }
    }
  }
}
```

#### <a name="add-the-certificate-information-to-the-service-fabric-cluster-resource"></a>Ajouter les informations de certificat à la ressource de cluster Service Fabric

```json
{
  "apiVersion": "2018-02-01",
  "type": "Microsoft.ServiceFabric/clusters",
  "name": "[parameters('clusterName')]",
  "location": "[parameters('clusterLocation')]",
  "dependsOn": [
    "[concat('Microsoft.Storage/storageAccounts/', variables('supportLogStorageAccountName'))]"
  ],
  "properties": {
    "certificateCommonNames": {
        "commonNames": [
        {
            "certificateCommonName": "[parameters('certificateCommonName')]",
            "certificateIssuerThumbprint": ""
        }
        ],
        "x509StoreName": "[parameters('certificateStoreValue')]"
    },
    ...
  }
}
```

## <a name="add-azure-ad-configuration-to-use-azure-ad-for-client-access"></a>Ajouter la configuration Azure AD afin d’utiliser Azure AD pour l’accès client

Vous ajoutez la configuration Azure AD à un modèle Resource Manager de cluster en référençant le coffre de clés qui contient les clés de certificat. Ajoutez ces paramètres et valeurs Azure AD dans un fichier de paramètres de modèle Resource Manager (*azuredeploy.parameters.json*). 

> [!NOTE]
> Sur Linux, les locataires et les utilisateurs Azure AD doivent être créés avant de créer le cluster.  Pour plus d’informations, consultez [Configurer Azure AD pour authentifier les clients](service-fabric-cluster-creation-setup-aad.md).

```json
{
  "apiVersion": "2018-02-01",
  "type": "Microsoft.ServiceFabric/clusters",
  "name": "[parameters('clusterName')]",
  ...
  "properties": {
    "certificateCommonNames": {
        "commonNames": [
        {
            "certificateCommonName": "[parameters('certificateCommonName')]",
            "certificateIssuerThumbprint": ""
        }
        ],
        "x509StoreName": "[parameters('certificateStoreValue')]"
    },
    ...
    "azureActiveDirectory": {
      "tenantId": "[parameters('aadTenantId')]",
      "clusterApplication": "[parameters('aadClusterApplicationId')]",
      "clientApplication": "[parameters('aadClientApplicationId')]"
    },
    ...
  }
}
```

## <a name="populate-the-parameter-file-with-the-values"></a>Renseigner le fichier de paramètres avec les valeurs

Enfin, utilisez les valeurs de sortie des commandes PowerShell Azure AD et du coffre de clés pour renseigner le fichier de paramètres.

Si vous envisagez d’utiliser les modules PowerShell Azure Service Fabric RM, il est inutile de renseigner les informations de certificat de cluster. Si vous souhaitez que le système génère le certificat auto-signé pour la sécurité du cluster, il vous suffit de les conserver Null. 

> [!NOTE]
> Pour que les modules RM récupèrent et renseignent des valeurs de paramètres vides, les noms des paramètres doivent correspondre à ceux indiqués ci-dessous.

```json
"clusterCertificateThumbprint": {
    "value": ""
},
"certificateCommonName": {
    "value": ""
},
"clusterCertificateUrlValue": {
    "value": ""
},
"sourceVaultvalue": {
    "value": ""
},
```

Si vous utilisez des certificats d’application ou un cluster existant que vous avez chargé dans le coffre de clés, vous devez récupérer ces informations et les renseigner.

Les modules RM ne peuvent pas générer la configuration d’Azure AD pour vous, donc si vous envisagez d’utiliser Azure AD pour l’accès client, vous devez le renseigner.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        ...
        "clusterCertificateStoreValue": {
            "value": "My"
        },
        "clusterCertificateThumbprint": {
            "value": "<thumbprint>"
        },
        "clusterCertificateUrlValue": {
            "value": "https://myvault.vault.azure.net:443/secrets/myclustercert/4d087088df974e869f1c0978cb100e47"
        },
        "applicationCertificateStorevalue": {
            "value": "My"
        },
        "applicationCertificateUrlValue": {
            "value": "https://myvault.vault.azure.net:443/secrets/myapplicationcert/2e035058ae274f869c4d0348ca100f08"
        },
        "sourceVaultvalue": {
            "value": "/subscriptions/<guid>/resourceGroups/mycluster-keyvault/providers/Microsoft.KeyVault/vaults/myvault"
        },
        "aadTenantId": {
            "value": "<guid>"
        },
        "aadClusterApplicationId": {
            "value": "<guid>"
        },
        "aadClientApplicationId": {
            "value": "<guid>"
        },
        ...
    }
}
```

## <a name="test-your-template"></a>Tester votre modèle
Pour tester votre modèle Resource Manager avec un fichier de paramètres, utilisez la commande PowerShell suivante :

```powershell
Test-AzResourceGroupDeployment -ResourceGroupName "myresourcegroup" -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json
```

Si vous rencontrez des problèmes et obtenez des messages difficiles à déchiffrer, utilisez « -Debug » comme option.

```powershell
Test-AzResourceGroupDeployment -ResourceGroupName "myresourcegroup" -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json -Debug
```

Le diagramme suivant montre comment votre Key Vault et la configuration Azure AD s’appliquent à votre modèle Resource Manager.

![Mappage de dépendances de Resource Manager][cluster-security-arm-dependency-map]

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez un modèle pour votre cluster, découvrez comment [déployer le cluster sur Azure](service-fabric-cluster-creation-via-arm.md).  Si vous ne l’avez pas déjà fait, lisez la [liste de vérification de préparation à la production](service-fabric-production-readiness-checklist.md) avant de déployer un cluster de production.

Pour en savoir plus sur la syntaxe JSON et les propriétés pour les ressources déployées dans cet article, consultez :

* [Microsoft.ServiceFabric/clusters](/azure/templates/microsoft.servicefabric/clusters)
* [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts)
* [Microsoft.Network/virtualNetworks](/azure/templates/microsoft.network/virtualnetworks)
* [Microsoft.Network/publicIPAddresses](/azure/templates/microsoft.network/publicipaddresses)
* [Microsoft.Network/loadBalancers](/azure/templates/microsoft.network/loadbalancers)
* [Microsoft.Compute/virtualMachineScaleSets](/azure/templates/microsoft.compute/virtualmachinescalesets)

<!-- Links -->
[service-fabric-cluster-security]: service-fabric-cluster-security.md
[service-fabric-secure-cluster-5-node-1-nodetype]: https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure
[resource-group-template-deploy]: ../azure-resource-manager/templates/deploy-powershell.md

<!-- Images -->
[cluster-security-arm-dependency-map]: ./media/service-fabric-cluster-creation-create-template/cluster-security-arm-dependency-map.png
