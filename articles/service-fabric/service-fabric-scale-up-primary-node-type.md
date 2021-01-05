---
title: Effectuer un scale-up sur un type de nœud primaire Azure Service Fabric
description: Découvrez comment mettre à l’échelle un cluster Service Fabric en ajoutant un type de nœud.
ms.topic: article
ms.date: 08/06/2020
ms.author: pepogors
ms.openlocfilehash: a18a40cc9e467b089ea9d6be3d0ca81a21d2c474
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89228713"
---
# <a name="scale-up-a-service-fabric-cluster-primary-node-type-by-adding-a-node-type"></a>Effectuer un scale-up sur un type de nœud principal de cluster Service Fabric en ajoutant un type de nœud
Cet article explique comment effectuer un scale-up sur un type de nœud principal de cluster Service Fabric en ajoutant un type de nœud supplémentaire au cluster. Un cluster Service Fabric est un groupe de machines virtuelles ou physiques connectées au réseau, sur lequel vos microservices sont déployés et gérés. Une machine ou une machine virtuelle faisant partie d’un cluster est appelée un nœud. Les groupes de machines virtuelles identiques constituent une ressource de calcul Azure que vous utilisez pour déployer et gérer une collection de machines virtuelles en tant que groupe. Chaque type de nœud défini dans un cluster Azure est [ configuré comme un groupe identique distinct](service-fabric-cluster-nodetypes.md). Chaque type de nœud peut alors faire l’objet d’une gestion séparée.

Les exemples de modèles dans le didacticiel suivant peuvent être trouvés ici : [Exemples de mise à l’échelle de type de nœud principal Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/Primary-NodeType-Scaling-Sample)

> [!WARNING]
> N’essayez pas une procédure de mise à l’échelle de type de nœud principal si l’état du cluster n’est pas sain, car cela déstabilisera uniquement le cluster.
>

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="process-to-upgrade-the-size-and-operating-system-of-the-primary-node-type"></a>Procéder à la mise à niveau de la taille et du système d’exploitation du type de nœud principal
Vous trouverez ci-après la procédure pour mettre à jour la taille et le système d’exploitation des machines virtuelles du type de nœud principal.  Après la mise à niveau, la taille des machines virtuelles du type de nœud principal sera Standard D4_V2 et leur système d’exploitation sera Windows Server 2019 Datacenter avec Conteneurs.

> [!WARNING]
> Avant de tenter cette procédure sur un cluster de production, nous vous recommandons d’examiner les exemples de modèles et de vérifier la procédure sur un cluster de test. Le cluster peut également être indisponible pendant une brève période de temps. 

### <a name="deploy-the-initial-service-fabric-cluster"></a>Déploiement du cluster Service Fabric initial 
Si vous souhaitez suivre l’exemple donné, déployez le cluster initial avec un seul type de nœud principal et un seul groupe identique [Service Fabric - Initial Cluster](https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/Primary-NodeType-Scaling-Sample/AzureDeploy-1.json). Vous pouvez ignorer cette étape si vous avez déjà déployé un cluster Service Fabric. 

1. Connectez-vous à votre compte Azure. 
```powershell
# sign in to your Azure account and select your subscription
Login-AzAccount -SubscriptionId "<your subscription ID>"
```
2. Créez un groupe de ressources. 
```powershell
# create a resource group for your cluster deployment
$resourceGroupName = "myResourceGroup"
$location = "WestUS"

New-AzResourceGroup `
    -Name $resourceGroupName `
    -Location $location
```
3. Renseignez les valeurs des paramètres dans les fichiers de modèle. 
4. Déployez le cluster dans le groupe de ressources créé à l’étape 2. 
```powershell
# deploy the template files to the resource group created above
$templateFilePath = "C:\AzureDeploy-1.json"
$parameterFilePath = "C:\AzureDeploy.Parameters.json"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath
```

### <a name="add-a-new-primary-node-type-to-the-cluster"></a>Ajouter un type de nœud primaire au cluster
> [!Note]
> Les ressources créées dans les étapes suivantes deviennent le nouveau type de nœud principal de votre cluster une fois l’opération de mise à l’échelle terminée. Veillez à utiliser des noms différents du sous-réseau initial, de l’adresse IP publique, du Load Balancer, du groupe de machines virtuelles identiques et du type de nœud. 

Vous pouvez trouver un modèle avec toutes les étapes suivantes terminées ici : [Service Fabric - New Node Type Cluster](https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/Primary-NodeType-Scaling-Sample/AzureDeploy-2.json). Les étapes suivantes contiennent des extraits de ressources partielles qui mettent en évidence les modifications apportées aux nouvelles ressources.  

1. Créez un sous-réseau dans votre réseau virtuel existant.
```json
{
    "name": "[variables('subnet1Name')]",
    "properties": {
        "addressPrefix": "[variables('subnet1Prefix')]"
    }
}
```
2. Créez une nouvelle ressource IP publique avec un domainNameLabel unique. 
```json
{
    "apiVersion": "[variables('publicIPApiVersion')]",
    "type": "Microsoft.Network/publicIPAddresses",
    "name": "[concat(variables('lbIPName'),'-',variables('vmNodeType1Name'))]",
    "location": "[variables('computeLocation')]",
    "properties": {
    "dnsSettings": {
        "domainNameLabel": "[concat(variables('dnsName'),'-','nt2')]"
    },
    "publicIPAllocationMethod": "Dynamic"
    },
    "tags": {
    "resourceType": "Service Fabric",
    "clusterName": "[parameters('clusterName')]"
    }
}
```
3. Créez une nouvelle ressource Load Balancer qui dépend de l’adresse IP publique créée ci-dessus. 
```json
"dependsOn": [
    "[concat('Microsoft.Network/publicIPAddresses/',concat(variables('lbIPName'),'-',variables('vmNodeType1Name')))]"
]
```
4. Créez un autre groupe de machines virtuelles identiques qui utilise les nouvelles références SKU de la machine virtuelle et du système d’exploitation devant faire l’objet d’un scale-up. 

Ref type de nœud 
```json
"nodeTypeRef": "[variables('vmNodeType1Name')]"
```

Référence de la machine virtuelle
```json
"sku": {
    "name": "[parameters('vmNodeType1Size')]",
    "capacity": "[parameters('nt1InstanceCount')]",
    "tier": "Standard"
}
```

Référence (SKU) du système d’exploitation 
```json
"imageReference": {
    "publisher": "[parameters('vmImagePublisher1')]",
    "offer": "[parameters('vmImageOffer1')]",
    "sku": "[parameters('vmImageSku1')]",
    "version": "[parameters('vmImageVersion1')]"
}
```

L’extrait de code suivant est un exemple de nouvelle ressource de groupe de machines virtuelles identiques qui est utilisée dans le cadre de la création d’un nouveau type de nœud pour un cluster Service Fabric. Assurez-vous que vous incluez toutes les extensions supplémentaires requises pour votre charge de travail. 

```json
    {
      "apiVersion": "[variables('vmssApiVersion')]",
      "type": "Microsoft.Compute/virtualMachineScaleSets",
      "name": "[variables('vmNodeType1Name')]",
      "location": "[variables('computeLocation')]",
      "dependsOn": [
        "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]",
        "[concat('Microsoft.Network/loadBalancers/', concat('LB','-', parameters('clusterName'),'-',variables('vmNodeType1Name')))]",
        "[concat('Microsoft.Storage/storageAccounts/', variables('supportLogStorageAccountName'))]",
        "[concat('Microsoft.Storage/storageAccounts/', variables('applicationDiagnosticsStorageAccountName'))]"
      ],
      "properties": {
        "overprovision": "[variables('overProvision')]",
        "upgradePolicy": {
          "mode": "Automatic"
        },
        "virtualMachineProfile": {
          "extensionProfile": {
            "extensions": [
              {
                "name": "[concat('ServiceFabricNodeVmExt_',variables('vmNodeType1Name'))]",
                "properties": {
                  "type": "ServiceFabricNode",
                  "autoUpgradeMinorVersion": true,
                  "protectedSettings": {
                    "StorageAccountKey1": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('supportLogStorageAccountName')),'2015-05-01-preview').key1]",
                    "StorageAccountKey2": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('supportLogStorageAccountName')),'2015-05-01-preview').key2]"
                  },
                  "publisher": "Microsoft.Azure.ServiceFabric",
                  "settings": {
                    "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
                    "nodeTypeRef": "[variables('vmNodeType1Name')]",
                    "dataPath": "D:\\SvcFab",
                    "durabilityLevel": "Bronze",
                    "enableParallelJobs": true,
                    "nicPrefixOverride": "[variables('subnet1Prefix')]",
                    "certificate": {
                      "thumbprint": "[parameters('certificateThumbprint')]",
                      "x509StoreName": "[parameters('certificateStoreValue')]"
                    }
                  },
                  "typeHandlerVersion": "1.0"
                }
              }
            ]
          },
          "networkProfile": {
            "networkInterfaceConfigurations": [
              {
                "name": "[concat(variables('nicName'), '-1')]",
                "properties": {
                  "ipConfigurations": [
                    {
                      "name": "[concat(variables('nicName'),'-',1)]",
                      "properties": {
                        "loadBalancerBackendAddressPools": [
                          {
                            "id": "[variables('lbPoolID1')]"
                          }
                        ],
                        "loadBalancerInboundNatPools": [
                          {
                            "id": "[variables('lbNatPoolID1')]"
                          }
                        ],
                        "subnet": {
                          "id": "[variables('subnet1Ref')]"
                        }
                      }
                    }
                  ],
                  "primary": true
                }
              }
            ]
          },
          "osProfile": {
            "adminPassword": "[parameters('adminPassword')]",
            "adminUsername": "[parameters('adminUsername')]",
            "computernamePrefix": "[variables('vmNodeType1Name')]",
            "secrets": [
              {
                "sourceVault": {
                  "id": "[parameters('sourceVaultValue')]"
                },
                "vaultCertificates": [
                  {
                    "certificateStore": "[parameters('certificateStoreValue')]",
                    "certificateUrl": "[parameters('certificateUrlValue')]"
                  }
                ]
              }
            ]
          },
          "storageProfile": {
            "imageReference": {
              "publisher": "[parameters('vmImagePublisher1')]",
              "offer": "[parameters('vmImageOffer1')]",
              "sku": "[parameters('vmImageSku1')]",
              "version": "[parameters('vmImageVersion1')]"
            },
            "osDisk": {
              "caching": "ReadOnly",
              "createOption": "FromImage",
              "managedDisk": {
                "storageAccountType": "[parameters('storageAccountType')]"
              }
            }
          }
        }
      },
      "sku": {
        "name": "[parameters('vmNodeType1Size')]",
        "capacity": "[parameters('nt1InstanceCount')]",
        "tier": "Standard"
      },
      "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
      }
    },

```

5. Ajoutez un nouveau type de nœud au cluster, qui fait référence au groupe de machines virtuelles identiques créé ci-dessus. La propriété **isPrimary** sur ce type de nœud doit être définie sur true. 
```json
"name": "[variables('vmNodeType1Name')]",
"applicationPorts": {
    "endPort": "[variables('nt0applicationEndPort')]",
    "startPort": "[variables('nt0applicationStartPort')]"
},
"clientConnectionEndpointPort": "[variables('nt0fabricTcpGatewayPort')]",
"durabilityLevel": "Bronze",
"ephemeralPorts": {
    "endPort": "[variables('nt0ephemeralEndPort')]",
    "startPort": "[variables('nt0ephemeralStartPort')]"
},
"httpGatewayEndpointPort": "[variables('nt0fabricHttpGatewayPort')]",
"isPrimary": true,
"reverseProxyEndpointPort": "[variables('nt0reverseProxyEndpointPort')]",
"vmInstanceCount": "[parameters('nt1InstanceCount')]"
```
6. Déployez le modèle ARM mis à jour 
```powershell
# deploy the updated template files to the existing resource group
$templateFilePath = "C:\AzureDeploy-2.json"
$parameterFilePath = "C:\AzureDeploy.Parameters.json"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath `
```

Le cluster Service Fabric aura désormais deux types de nœuds une fois le déploiement terminé. 

### <a name="remove-the-existing-node-type"></a>Supprimer le type de nœud existant. 
Une fois le déploiement des ressources terminé, vous pouvez commencer à désactiver les nœuds dans le type de nœud principal original. Comme les nœuds sont désactivés, les services système migrent vers le nouveau type de nœud principal qui a été déployé à l’étape précédente.

1. Affectez la valeur false à la propriété type de nœud principal de la ressource de cluster Service Fabric. 
```json
{
    "name": "[variables('vmNodeType0Name')]",
    "applicationPorts": {
        "endPort": "[variables('nt0applicationEndPort')]",
        "startPort": "[variables('nt0applicationStartPort')]"
    },
    "clientConnectionEndpointPort": "[variables('nt0fabricTcpGatewayPort')]",
    "durabilityLevel": "Bronze",
    "ephemeralPorts": {
        "endPort": "[variables('nt0ephemeralEndPort')]",
        "startPort": "[variables('nt0ephemeralStartPort')]"
    },
    "httpGatewayEndpointPort": "[variables('nt0fabricHttpGatewayPort')]",
    "isPrimary": false,
    "reverseProxyEndpointPort": "[variables('nt0reverseProxyEndpointPort')]",
    "vmInstanceCount": "[parameters('nt0InstanceCount')]"
}
```
2. Déployez le modèle avec la propriété isPrimary mise à jour sur le type de nœud d’origine. Vous pouvez trouver un modèle avec l’indicateur principal défini sur false sur le type de nœud d’origine ici : [Service Fabric - Primary Node Type False](https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/Primary-NodeType-Scaling-Sample/AzureDeploy-3.json).

```powershell
# deploy the updated template files to the existing resource group
$templateFilePath = "C:\AzureDeploy-3.json"
$parameterFilePath = "C:\AzureDeploy.Parameters.json"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath `
```

3. Désactivez les nœuds dans le type de nœud 0. 
```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint $ClusterConnectionEndpoint `
    -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCertThumbprint $thumb  `
    -FindType FindByThumbprint `
    -FindValue $thumb `
    -StoreLocation CurrentUser `
    -StoreName My 

Write-Host "Connected to cluster"


$nodeType = "nt1vm" # specify the name of node type
$nodes = Get-ServiceFabricNode 

Write-Host "Disabling nodes..."
foreach($node in $nodes)
{
  if ($node.NodeType -eq $nodeType)
  {
    $node.NodeName

    Disable-ServiceFabricNode -Intent RemoveNode -NodeName $node.NodeName -Force
  }
}
```
* Pour la durabilité Bronze, attendez que tous les nœuds soient désactivés.
* Pour la durabilité Silver ou Gold, certains nœuds seront désactivés et les autres seront en cours de désactivation. Consultez l'onglet des détails des nœuds en cours de désactivation. S'ils sont tous bloqués pour garantir le quorum des partitions du service d'infrastructure, il est possible de continuer en toute sécurité.

> [!Note]
> Cette étape peut prendre un certain temps. 

4. Arrêtez les données sur le type de nœud 0. 
```powershell
foreach($node in $nodes)
{
  if ($node.NodeType -eq $nodeType)
  {
    $node.NodeName

    Start-ServiceFabricNodeTransition -Stop -OperationId (New-Guid) -NodeInstanceId $node.NodeInstanceId -NodeName $node.NodeName -StopDurationInSeconds 10000
  }
}
```
5. Libérer des nœuds dans le groupe de machines virtuelles identiques d’origine 
```powershell
$scaleSetName="nt1vm"
$scaleSetResourceType="Microsoft.Compute/virtualMachineScaleSets"

Remove-AzResource -ResourceName $scaleSetName -ResourceType $scaleSetResourceType -ResourceGroupName $resourceGroupName -Force
```
> [!Note]
> Les étapes 6 et 7 sont facultatives si vous utilisez déjà une adresse IP publique de référence SKU standard et un équilibreur de charge de référence SKU standard. Dans ce cas, vous pouvez avoir plusieurs types de nœuds/groupes de machines virtuelles identiques sous le même équilibreur de charge. 

6. Vous pouvez maintenant supprimer l’adresse IP d’origine et les ressources de Load Balancer. Au cours de cette étape, vous allez également mettre à jour le nom DNS. 

```powershell
$lbname="LB-cluster-name-nt1vm"
$lbResourceType="Microsoft.Network/loadBalancers"
$ipResourceType="Microsoft.Network/publicIPAddresses"
$oldPublicIpName="PublicIP-LB-FE-nt1vm"
$newPublicIpName="PublicIP-LB-FE-nt2vm"

$oldprimaryPublicIP = Get-AzPublicIpAddress -Name $oldPublicIpName  -ResourceGroupName $resourceGroupName
$primaryDNSName = $oldprimaryPublicIP.DnsSettings.DomainNameLabel
$primaryDNSFqdn = $oldprimaryPublicIP.DnsSettings.Fqdn

Remove-AzResource -ResourceName $lbname -ResourceType $lbResourceType -ResourceGroupName $resourceGroupName -Force
Remove-AzResource -ResourceName $oldPublicIpName -ResourceType $ipResourceType -ResourceGroupName $resourceGroupName -Force

$PublicIP = Get-AzPublicIpAddress -Name $newPublicIpName  -ResourceGroupName $resourceGroupName
$PublicIP.DnsSettings.DomainNameLabel = $primaryDNSName
$PublicIP.DnsSettings.Fqdn = $primaryDNSFqdn
Set-AzPublicIpAddress -PublicIpAddress $PublicIP
``` 

7. Mettez à jour le point de terminaison de gestion sur le cluster pour faire référence à la nouvelle adresse IP. 
```json
  "managementEndpoint": "[concat('https://',reference(concat(variables('lbIPName'),'-',variables('vmNodeType1Name'))).dnsSettings.fqdn,':',variables('nt0fabricHttpGatewayPort'))]",
```
8. Supprimez l’état du nœud du type de nœud 0.
```powershell
foreach($node in $nodes)
{
  if ($node.NodeType -eq $nodeType)
  {
    $node.NodeName

    Remove-ServiceFabricNodeState -NodeName $node.NodeName -Force
  }
}
```
9. Supprimez la référence de type de nœud d’origine de la ressource Service Fabric dans le modèle ARM. 
```json
"name": "[variables('vmNodeType0Name')]",
"applicationPorts": {
    "endPort": "[variables('nt0applicationEndPort')]",
    "startPort": "[variables('nt0applicationStartPort')]"
},
"clientConnectionEndpointPort": "[variables('nt0fabricTcpGatewayPort')]",
"durabilityLevel": "Bronze",
"ephemeralPorts": {
    "endPort": "[variables('nt0ephemeralEndPort')]",
    "startPort": "[variables('nt0ephemeralStartPort')]"
},
"httpGatewayEndpointPort": "[variables('nt0fabricHttpGatewayPort')]",
"isPrimary": true,
"reverseProxyEndpointPort": "[variables('nt0reverseProxyEndpointPort')]",
"vmInstanceCount": "[parameters('nt0InstanceCount')]"
```
Pour les clusters Silver et de durabilité supérieure uniquement, mettez à jour la ressource de cluster dans le modèle et configurez les stratégies d’intégrité de manière à ignorer l’intégrité de l’application fabric:/System en ajoutant applicationDeltaHealthPolicies sous les propriétés de la ressource de cluster comme indiqué ci-dessous. La stratégie ci-dessous doit ignorer les erreurs existantes, mais ne pas autoriser de nouvelles erreurs d'intégrité.
```json
"upgradeDescription":  
{ 
 "forceRestart": false, 
 "upgradeReplicaSetCheckTimeout": "10675199.02:48:05.4775807", 
 "healthCheckWaitDuration": "00:05:00", 
 "healthCheckStableDuration": "00:05:00", 
 "healthCheckRetryTimeout": "00:45:00", 
 "upgradeTimeout": "12:00:00", 
 "upgradeDomainTimeout": "02:00:00", 
 "healthPolicy": { 
   "maxPercentUnhealthyNodes": 100, 
   "maxPercentUnhealthyApplications": 100 
 }, 
 "deltaHealthPolicy":  
 { 
   "maxPercentDeltaUnhealthyNodes": 0, 
   "maxPercentUpgradeDomainDeltaUnhealthyNodes": 0, 
   "maxPercentDeltaUnhealthyApplications": 0, 
   "applicationDeltaHealthPolicies":  
   { 
       "fabric:/System":  
       { 
           "defaultServiceTypeDeltaHealthPolicy":  
           { 
                   "maxPercentDeltaUnhealthyServices": 0 
           } 
       } 
   } 
 } 
}
```
10. Supprimez toutes les autres ressources associées au type de nœud d’origine à partir du modèle Resource Manager. Consultez [Service Fabric - New Node Type Cluster](https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/Primary-NodeType-Scaling-Sample/AzureDeploy-4.json) pour un modèle avec l’ensemble de ces ressources d’origine supprimées.

11. Déployez le modèle Azure Resource Manager modifié. ** Cette étape prendra un certain temps, mais généralement pas plus de deux heures. Comme cette mise à niveau modifie les paramètres d’InfrastructureService, un redémarrage du nœud est nécessaire. Dans ce cas, forceRestart est ignoré. Le paramètre upgradeReplicaSetCheckTimeout spécifie la durée maximale pendant laquelle Service Fabric doit attendre qu’une partition soit sécurisée, si ce n’est pas encore le cas. Une fois les contrôles de sécurité réussis pour toutes les partitions d'un nœud, Service Fabric procède à la mise à niveau sur ce nœud. La valeur du paramètre upgradeTimeout peut être réduite à 6 heures, mais pour une sécurité maximale, il convient d’utiliser 12 heures.
Vérifiez ensuite que la ressource Service Fabric dans le portail semble prête. 

```powershell
# deploy the updated template files to the existing resource group
$templateFilePath = "C:\AzureDeploy-4.json"
$parameterFilePath = "C:\AzureDeploy.Parameters.json"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath `
```

Le type de nœud principal du cluster a été mis à niveau. Vérifiez que toutes les applications déployées fonctionnent correctement et que le cluster est sain.

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [ajouter un type de nœud à un cluster](virtual-machine-scale-set-scale-node-type-scale-out.md).
* Découvrez-en plus sur l’[extensibilité des applications](service-fabric-concepts-scalability.md).
* [Procédez à une montée et une descente en puissance d’un cluster Azure](service-fabric-tutorial-scale-cluster.md).
* [Mettez à l’échelle un cluster Azure par programmation](service-fabric-cluster-programmatic-scaling.md), à l’aide du kit de développement logiciel de calcul Azure.
* [Augmentez ou diminuez la taille des instances d’un cluster autonome](service-fabric-cluster-windows-server-add-remove-nodes.md).
