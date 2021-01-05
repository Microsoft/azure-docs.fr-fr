---
title: Déployer des types de nœuds sans état uniquement dans un cluster Service Fabric
description: Découvrez comment créer et déployer des types de nœuds sans état dans un cluster Azure Service Fabric.
author: peterpogorski
ms.topic: conceptual
ms.date: 09/25/2020
ms.author: pepogors
ms.openlocfilehash: 266c04a049cab574576f781c397aee566efe5372
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516615"
---
# <a name="deploy-an-azure-service-fabric-cluster-with-stateless-only-node-types-preview"></a>Déployer un cluster Azure Service Fabric avec des types de nœuds sans état (préversion)
Les types de nœuds Service Fabric sont fournis en supposant qu’à un moment donné, des services avec état peuvent être placés sur les nœuds. Les types de nœuds sans état assouplissent cette hypothèse pour un type de nœud, ce qui permet au type de nœud d’utiliser d’autres fonctionnalités, telles que l’accélération des opérations de scale-out, la prise en charge des mises à niveau automatiques du système d’exploitation sur la durabilité Bronze et le scale-out de plus de 100 nœuds dans un même groupe de machines virtuelles identiques.

* Les types de nœuds principaux ne peuvent pas être configurés pour être sans état
* Les types de nœuds sans état ne sont pris en charge qu’avec des niveaux de durabilité Bronze
* Les types de nœuds sans état ne sont pris en charge que dans le runtime Service Fabric version 7.1.409 ou ultérieure.


Des exemples de modèles sont disponibles : [Modèle de types de nœuds sans état Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates)

## <a name="enabling-stateless-node-types-in-service-fabric-cluster"></a>Activation de types de nœuds sans état dans un cluster Service Fabric
Pour définir un ou plusieurs types de nœuds comme sans état dans une ressource de cluster, définissez la propriété **isStateless** sur « True ». Lors du déploiement d’un cluster Service Fabric avec des types de nœuds sans état, n’oubliez pas d’avoir au moins un type de nœud principal dans la ressource de cluster.

* La ressource de cluster Service Fabric apiVersion doit être « 2020-12-01-preview » ou une version supérieure.

```json
{
    "nodeTypes": [
    {
        "name": "[parameters('vmNodeType0Name')]",
        "applicationPorts": {
            "endPort": "[parameters('nt0applicationEndPort')]",
            "startPort": "[parameters('nt0applicationStartPort')]"
        },
        "clientConnectionEndpointPort": "[parameters('nt0fabricTcpGatewayPort')]",
        "durabilityLevel": "Bronze",
        "ephemeralPorts": {
            "endPort": "[parameters('nt0ephemeralEndPort')]",
            "startPort": "[parameters('nt0ephemeralStartPort')]"
        },
        "httpGatewayEndpointPort": "[parameters('nt0fabricHttpGatewayPort')]",
        "isPrimary": true,
        "isStateles": false,
        "vmInstanceCount": "[parameters('nt0InstanceCount')]"
    },
    {
        "name": "[parameters('vmNodeType1Name')]",
        "applicationPorts": {
            "endPort": "[parameters('nt1applicationEndPort')]",
            "startPort": "[parameters('nt1applicationStartPort')]"
        },
        "clientConnectionEndpointPort": "[parameters('nt1fabricTcpGatewayPort')]",
        "durabilityLevel": "Silver",
        "ephemeralPorts": {
            "endPort": "[parameters('nt1ephemeralEndPort')]",
            "startPort": "[parameters('nt1ephemeralStartPort')]"
        },
        "httpGatewayEndpointPort": "[parameters('nt1fabricHttpGatewayPort')]",
        "isPrimary": false,
        "isStateless": true,
        "vmInstanceCount": "[parameters('nt1InstanceCount')]"
    }    
    ],
}
```

## <a name="configuring-virtual-machine-scale-set-for-stateless-node-types"></a>Configuration du groupe de machines virtuelles identiques pour les types de nœuds sans état
Pour activer les types de nœuds sans état, vous devez configurer la ressource de groupe de machines virtuelles identiques sous-jacente de la façon suivante :

* La propriété **singlePlacementGroup** de la valeur doit être définie sur True/False en fonction de l’exigence de mise à l’échelle sur plus de 100 machines virtuelles.
* Le groupe identique **upgradeMode** doit être défini sur Continue.
* Le mode de mise à niveau Continue nécessite la configuration de l’extension Intégrité de l’application ou des sondes d’intégrité. Configurez la sonde d’intégrité avec la configuration par défaut pour les types de nœuds sans état comme indiqué ci-dessous. Une fois les applications déployées sur le nodetype, les ports d’extension de sonde d’intégrité/d’intégrité peuvent être modifiés pour surveiller l’intégrité de l’application.

```json
{
    "apiVersion": "2018-10-01",
    "type": "Microsoft.Compute/virtualMachineScaleSets",
    "name": "[parameters('vmNodeType1Name')]",
    "location": "[parameters('computeLocation')]",
    "properties": {
        "overprovision": "[variables('overProvision')]",
        "upgradePolicy": {
          "mode": "Rolling",
          "automaticOSUpgradePolicy": {
            "enableAutomaticOSUpgrade": true
          }
        }
    }
    "virtualMachineProfile": {
    "extensionProfile": {
    "extensions": [
    {
    "name": "[concat(parameters('vmNodeType1Name'),'_ServiceFabricNode')]",
    "properties": {
        "type": "ServiceFabricNode",
        "autoUpgradeMinorVersion": false,
        "publisher": "Microsoft.Azure.ServiceFabric",
        "settings": {
            "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
            "nodeTypeRef": "[parameters('vmNodeType1Name')]",
            "dataPath": "D:\\\\SvcFab",
            "durabilityLevel": "Silver",
            "certificate": {
                "thumbprint": "[parameters('certificateThumbprint')]",
                "x509StoreName": "[parameters('certificateStoreValue')]"
            },
            "systemLogUploadSettings": {
                "Enabled": true
            },
        },
        "typeHandlerVersion": "1.0"
    }
    },
    {
        "type": "extensions",
        "name": "HealthExtension",
        "properties": {
            "publisher": "Microsoft.ManagedServices",
            "type": "ApplicationHealthWindows",
            "autoUpgradeMinorVersion": true,
            "typeHandlerVersion": "1.0",
            "settings": {
            "protocol": "tcp",
            "port": "19000"
            }
            }
        },
    ]
}
```

## <a name="networking-requirements"></a>Configuration requise du réseau
### <a name="public-ip-and-load-balancer-resource"></a>Ressource d’adresse IP publique et Load Balancer
Pour activer la mise à l'échelle de plus de 100 machines virtuelles sur une ressource de groupe de machines virtuelles identiques, les ressources d’équilibreur de charge et d’adresse IP référencées par ce groupe de machines virtuelles identiques doivent toutes deux utiliser une référence SKU *Standard*. La création d’une ressource d’équilibreur de charge ou d’adresse IP sans la propriété de référence SKU crée une référence SKU de base, qui ne prend pas en charge plus de 100 machines virtuelles. Un équilibreur de charge de référence SKU Standard bloque par défaut tout le trafic provenant de l’extérieur ; pour autoriser le trafic de l’extérieur, un groupe de sécurité réseau (NSG) doit être déployé sur le sous-réseau.

```json
{
    "apiVersion": "2018-11-01",
    "type": "Microsoft.Network/publicIPAddresses",
    "name": "[concat('LB','-', parameters('clusterName')]",
    "location": "[parameters('computeLocation')]",
    "sku": {
        "name": "Standard"
    }
}
{
    "apiVersion": "2018-11-01",
    "type": "Microsoft.Network/loadBalancers",
    "name": "[concat('LB','-', parameters('clusterName')]", 
    "location": "[parameters('computeLocation')]",
    "dependsOn": [
        "[concat('Microsoft.Network/networkSecurityGroups/', concat('nsg', parameters('subnet0Name')))]"
    ],
    "properties": {
        "addressSpace": {
            "addressPrefixes": [
                "[parameters('addressPrefix')]"
            ]
        },
        "subnets": [
        {
            "name": "[parameters('subnet0Name')]",
            "properties": {
                "addressPrefix": "[parameters('subnet0Prefix')]",
                "networkSecurityGroup": {
                "id": "[resourceId('Microsoft.Network/networkSecurityGroups', concat('nsg', parameters('subnet0Name')))]"
              }
            }
          }
        ]
    },
    "sku": {
        "name": "Standard"
    }
}
```

>[!NOTE]
> Il n’est pas possible d’effectuer une modification sur place de référence SKU sur les ressources d’adresse IP publique et d’équilibreur de charge. Si vous effectuez une migration à partir de ressources existantes ayant une référence SKU de base, consultez la section relative à la migration de cet article.

### <a name="virtual-machine-scale-set-nat-rules"></a>Règles NAT de groupe de machines virtuelles identiques
Les règles NAT de trafic entrant de l’équilibreur de charge doivent correspondre aux pools NAT du groupe de machines virtuelles identiques. Chaque groupe de machines virtuelles identiques doit avoir un pool NAT entrant unique.

```json
{
"inboundNatPools": [
    {
        "name": "LoadBalancerBEAddressNatPool0",
        "properties": {
            "backendPort": "3389",
            "frontendIPConfiguration": {
                "id": "[variables('lbIPConfig0')]"
            },
            "frontendPortRangeEnd": "50999",
            "frontendPortRangeStart": "50000",
            "protocol": "tcp"
        }
    },
    {
        "name": "LoadBalancerBEAddressNatPool1",
        "properties": {
            "backendPort": "3389",
            "frontendIPConfiguration": {
                "id": "[variables('lbIPConfig0')]"
            },
            "frontendPortRangeEnd": "51999",
            "frontendPortRangeStart": "51000",
            "protocol": "tcp"
        }
    },
    {
        "name": "LoadBalancerBEAddressNatPool2",
        "properties": {
            "backendPort": "3389",
            "frontendIPConfiguration": {
                "id": "[variables('lbIPConfig0')]"
            },
            "frontendPortRangeEnd": "52999",
            "frontendPortRangeStart": "52000",
            "protocol": "tcp"
        }
    }
    ]
}
```

### <a name="standard-sku-load-balancer-outbound-rules"></a>Règles de trafic sortant dans Load Balancer de la référence SKU standard
Si on les compare à l’utilisation de références SKU de base, Standard Load Balancer et les adresses IP publiques Standard introduisent de nouvelles fonctionnalités et des comportements différents pour la connectivité sortante. Si vous souhaitez une connectivité sortante lorsque vous travaillez avec des références SKU Standard, vous devez explicitement la définir avec des adresses IP publiques Standard ou l’équilibreur de charge public Standard. Pour plus d’informations, consultez [Connexions sortantes](../load-balancer/load-balancer-outbound-connections.md) et [Azure Standard Load Balancer](../load-balancer/load-balancer-overview.md).

>[!NOTE]
> Le modèle standard fait référence à un groupe de sécurité réseau (NSG) qui autorise tout le trafic sortant par défaut. Le trafic entrant est limité aux ports qui sont requis pour les opérations de gestion de Service Fabric. Les règles NSG peuvent être modifiées pour répondre à vos besoins.

>[!NOTE]
> Tout cluster Service Fabric utilisant la référence SLB SKU Standard doit vérifier que chaque type de nœud a une règle autorisant le trafic sortant sur le port 443. Cette opération est nécessaire pour terminer l’installation du cluster, et tout déploiement sans règle de ce type échouera.



### <a name="migrate-to-using-stateless-node-types-from-a-cluster-using-a-basic-sku-load-balancer-and-a-basic-sku-ip"></a>Migrer à l’aide de types de nœuds sans état à partir d’un cluster à l’aide d’un Load Balancer de référence SKU de base et d’une adresse IP de référence SKU de base
Pour tous les scénarios de migration, un nouveau type de nœud sans état doit être ajouté. Impossible de migrer un type de nœud existant pour qu’il soit sans état uniquement.

Pour migrer un cluster qui utilisait un Load Balancer et une adresse IP avec une référence SKU de base, vous devez tout d’abord créer une toute nouvelle ressource de Load Balancer et d’adresse à l’aide de la référence SKU standard. Il n’est pas possible de mettre à jour ces ressources sur place.

Les nouveaux équilibreur de charge et adresse IP doivent être référencés dans les nouveaux types de nœuds sans état que vous souhaitez utiliser. Dans l’exemple ci-dessus, une nouvelle ressource de groupe de machines virtuelles identiques est ajoutée pour être utilisée pour les types de nœuds sans état. Ces groupes de machines virtuelles identiques font référence aux nouveaux équilibreur de charge et adresse IP, et sont marqués comme des types de nœuds sans état dans la ressource de cluster Service Fabric.

Pour commencer, vous devez ajouter les nouvelles ressources à votre modèle Resource Manager existant. Ces ressources incluent :
* Une ressource d’adresse IP publique utilisant une référence SKU standard.
* Une ressource de Load Balancer utilisant une référence SKU standard.
* Un groupe de sécurité réseau (NSG) référencé par le sous-réseau dans lequel vous déployez vos groupes de machines virtuelles identiques.

Une fois le déploiement des ressources terminé, vous pouvez commencer à désactiver les nœuds dans le type de nœud que vous souhaitez supprimer du cluster d’origine.


## <a name="next-steps"></a>Étapes suivantes 
* [Services fiables (Reliable Services)](service-fabric-reliable-services-introduction.md)
* [Types de nœud et groupes de machines virtuelles identiques](service-fabric-cluster-nodetypes.md)

