---
title: Mettre à l’échelle un cluster Service Fabric dans Azure
description: Dans ce tutoriel, vous allez apprendre à effectuer un scale-out et un scale-in d’un cluster Service Fabric dans Azure, puis à nettoyer les ressources restantes.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 063c83818ec2e98491f9062e936b9a1e7b2c4356
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2020
ms.locfileid: "97702172"
---
# <a name="tutorial-scale-a-service-fabric-cluster-in-azure"></a>Tutoriel : Mettre à l’échelle un cluster Service Fabric dans Azure

Ce didacticiel constitue la troisième partie d’une série et montre comment diminuer ou augmenter la taille de votre cluster existant. À la fin de ce tutoriel, vous saurez comment mettre à l’échelle votre cluster et comment nettoyer les ressources restantes.  Pour plus d’informations sur la mise à l’échelle d’un cluster exécuté dans Azure, consultez [Mise à l’échelle de clusters Service Fabric](service-fabric-cluster-scaling.md).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Ajouter et supprimer des nœuds (scale-out et scale-in)
> * Ajouter et supprimer des types de nœuds (scale-out et scale-in)
> * Augmenter les ressources de nœud (scale up)

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * Créer un [cluster Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) sécurisé sur Azure à l’aide d’un modèle
> * [Superviser un cluster](service-fabric-tutorial-monitor-cluster.md)
> * Mettre à l’échelle un cluster
> * [Mettre à niveau le runtime d’un cluster](service-fabric-tutorial-upgrade-cluster.md)
> * [Supprimer un cluster](service-fabric-tutorial-delete-cluster.md)


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Installez [Azure PowerShell](/powershell/azure/install-az-ps) ou [Azure CLI](/cli/azure/install-azure-cli).
* Créer un [cluster Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) sécurisé sur Azure

## <a name="important-considerations-and-guidelines"></a>Considérations importantes et recommandations

Les charges de travail d’application évoluent au fil du temps. Vos services existants ont-ils besoin de plus (ou moins) de ressources ?  [Ajoutez ou supprimez des nœuds](#add-nodes-to-or-remove-nodes-from-a-node-type) d’un type de nœud pour augmenter ou diminuer les ressources de cluster.

Vous devez ajouter plus de 100 nœuds à votre cluster ?  Un groupe identique/type de nœud Service Fabric unique ne peut pas contenir plus de 100 nœuds/machines virtuelles.  Pour mettre à l’échelle un cluster au-delà de 100 nœuds, [ajoutez des types de nœuds supplémentaires](#add-nodes-to-or-remove-nodes-from-a-node-type).

Votre application inclut-elle plusieurs services ? Si oui, ces services doivent-ils être publics ou accessibles sur Internet ?  Les applications standard contiennent un service de passerelle frontale qui reçoit des entrées d’un client, et un ou plusieurs services principaux qui communiquent avec les services frontaux. Dans ce cas, nous vous recommandons d’[ajouter au moins deux types de nœuds](#add-nodes-to-or-remove-nodes-from-a-node-type) au cluster.  

Vos services ont-ils des besoins d’infrastructure différents tels qu’une RAM plus volumineuse ou des cycles processeur plus élevés ? Par exemple, votre application contient un service front-end et un service back-end. Le service frontal peut s’exécuter sur des machines virtuelles plus petites (par exemple, de taille D2) ayant des ports ouverts sur Internet. Toutefois, le service principal est gourmand en calculs et doit s’exécuter sur des machines virtuelles plus grandes (par exemple, de taille D4, D6, D15) qui ne sont pas connectées à Internet. Dans ce cas, nous vous recommandons d’[ajouter plusieurs types de nœuds](#add-nodes-to-or-remove-nodes-from-a-node-type) à votre cluster. Ainsi, chaque type de nœud a des propriétés distinctes comme la connectivité Internet ou la taille de machine virtuelle. Le nombre de machines virtuelles peut être mis à l’échelle indépendamment, également.

Lors de la mise à l’échelle d’un cluster Azure, gardez les instructions suivantes à l’esprit :

* Un groupe identique/type de nœud Service Fabric unique ne peut pas contenir plus de 100 nœuds/machines virtuelles.  Pour mettre à l’échelle un cluster au-delà de 100 nœuds, ajoutez des types de nœuds supplémentaires.
* Les types de nœuds principaux exécutant des charges de travail de production doivent avoir un [niveau de durabilité][durability] Or ou Argent et toujours comporter au moins cinq nœuds.
* Les types de nœuds non principaux exécutant des charges de travail de production avec état doivent toujours comporter au moins cinq nœuds.
* Les types de nœuds non principaux exécutant des charges de travail de production sans état doivent toujours comporter au moins deux nœuds.
* Tout type de nœud dont le [niveau de durabilité][durability] est Gold ou Silver doit toujours comporter au moins cinq nœuds.
* En cas de mise à l’échelle (suppression de nœuds) d’un type de nœud principal, vous ne devez jamais réduire le nombre d’instances à un chiffre inférieur à celui requis par le [niveau de fiabilité][reliability].

Pour plus d’informations, consultez [Instructions concernant la capacité du cluster](service-fabric-cluster-capacity.md).

## <a name="export-the-template-for-the-resource-group"></a>Exporter le modèle pour le groupe de ressources

Après avoir créé un [cluster Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) sécurisé et configuré votre groupe de ressources, exportez le modèle Resource Manager pour le groupe de ressources. L’exportation du modèle vous permet d’automatiser les futurs déploiements du cluster et de ses ressources, car le modèle contient l’infrastructure complète.  Pour plus d’informations sur l’exportation de modèles, consultez [Gérer des groupes de ressources Azure Resource Manager à l’aide du portail Azure](../azure-resource-manager/management/manage-resource-groups-portal.md).

1. Dans le [portail Azure](https://portal.azure.com), accédez au groupe de ressources contenant le cluster (**sfclustertutorialgroup**, si vous suivez ce tutoriel). 

2. Dans le volet gauche, sélectionnez **Déploiements**, ou sélectionnez le lien situé sous **Déploiements**. 

3. Sélectionnez le dernier déploiement ayant réussi dans la liste.

4. Dans le volet gauche, sélectionnez **Modèle**, puis **Télécharger** pour exporter le modèle en tant que fichier ZIP.  Enregistrez le modèle et les paramètres sur votre ordinateur local.

## <a name="add-nodes-to-or-remove-nodes-from-a-node-type"></a>Ajouter ou supprimer des nœuds d’un type de nœud

Le scale in ou scale out, ou mise à l’échelle horizontale, modifie le nombre de nœuds du cluster. Quand vous effectuez un scale-in ou un scale-out, vous ajoutez des instances de machines virtuelles au groupe identique. Ces instances deviennent les nœuds que Service Fabric utilise. Service Fabric sait quand des instances sont ajoutées au groupe identique et réagit automatiquement. Une mise à l’échelle peut s’effectuer à tout moment, même lorsque des charges de travail sont en cours d’exécution sur le cluster.

### <a name="update-the-template"></a>Mettre à jour le modèle

[Exportez un fichier de modèle et de paramètres](#export-the-template-for-the-resource-group) à partir du groupe de ressources pour le déploiement le plus récent.  Ouvrez le fichier *parameters.json*.  Si vous avez déployé le cluster à l’aide de l’[exemple de modèle][template] dans ce tutoriel, il existe trois types de nœuds dans le cluster et trois paramètres qui définissent le nombre de nœuds pour chaque type de nœud :  *nt0InstanceCount*, *nt1InstanceCount* et *nt2InstanceCount*.  Le paramètre *nt1InstanceCount*, par exemple, définit le nombre d’instances du deuxième type de nœud et le nombre de machines virtuelles dans le groupe de machines virtuelles identiques associé.

Ainsi, en mettant à jour la valeur de *nt1InstanceCount*, vous modifiez le nombre de nœuds dans le deuxième type de nœud.  N’oubliez pas que vous ne pouvez pas effectuer de scale out d’un type de nœud à plus de 100 nœuds.  Les types de nœuds non principaux exécutant des charges de travail de production avec état doivent toujours comporter au moins cinq nœuds. Les types de nœuds non principaux exécutant des charges de travail de production sans état doivent toujours comporter au moins deux nœuds.

Si vous effectuez un scale-in et supprimez des nœuds à partir d’un type de nœud de [niveau de durabilité][durability] Bronze, vous devez [effacer manuellement l’état de ces nœuds](service-fabric-cluster-scale-in-out.md#manually-remove-vms-from-a-node-typevirtual-machine-scale-set).  Pour les niveaux de durabilité Silver et Gold, ces étapes sont effectuées automatiquement par la plateforme.

### <a name="deploy-the-updated-template"></a>Déployer le modèle mis à jour
Enregistrez les modifications apportées aux fichiers *template.json* et *parameters.json*.  Pour déployer le modèle mis à jour, exécutez la commande suivante :

```powershell
New-AzResourceGroupDeployment -ResourceGroupName sfclustertutorialgroup -TemplateFile c:\temp\template.json -TemplateParameterFile c:\temp\parameters.json -Name "ChangingInstanceCount"
```
Ou la commande Azure CLI suivante :
```azurecli
az deployment group create --resource-group sfclustertutorialgroup --template-file c:\temp\template.json --parameters c:\temp\parameters.json
```

## <a name="add-a-node-type-to-the-cluster"></a>Ajouter un type de nœud au cluster

Chaque type de nœud qui est défini dans un cluster Service Fabric exécuté dans Azure est configuré en tant que [groupe de machines virtuelles identiques distinct](service-fabric-cluster-nodetypes.md). Chaque type de nœud peut alors faire l’objet d’une gestion séparée. Vous pouvez faire monter ou descendre en puissance chaque type de nœud de manière indépendante, avoir différents ensembles de ports ouverts et utiliser différentes métriques de capacité. Vous pouvez également changer indépendamment la référence SKU du système d’exploitation en cours d’exécution sur chaque nœud de cluster, mais notez que vous ne pouvez avoir une combinaison de Windows et Linux en cours d’exécution dans l’exemple de cluster. Un même type de nœud/groupe identique ne peut pas contenir plus de 100 nœuds.  Vous pouvez mettre à l’échelle un cluster horizontalement à plus de 100 nœuds en ajoutant des types de nœuds/groupes identiques supplémentaires. Une mise à l’échelle peut s’effectuer à tout moment, même lorsque des charges de travail sont en cours d’exécution sur le cluster.

### <a name="update-the-template"></a>Mettre à jour le modèle

[Exportez un fichier de modèle et de paramètres](#export-the-template-for-the-resource-group) à partir du groupe de ressources pour le déploiement le plus récent.  Ouvrez le fichier *parameters.json*.  Si vous avez déployé le cluster à l’aide de l’[exemple de modèle][template] dans ce tutoriel, il existe trois types de nœuds dans le cluster.  Dans cette section, vous allez ajouter un quatrième type de nœud en mettant à jour et en déployant un modèle Resource Manager. 

Outre le nouveau type de nœud, vous allez également ajouter le groupe de machines virtuelles identiques (qui s’exécute sur un sous-réseau distinct du réseau virtuel) et le groupe de sécurité réseau associés.  Vous pouvez choisir d’ajouter des adresses IP publiques et des ressources d’équilibreur de charge Azure nouvelles ou existantes pour le nouveau groupe identique.  Le nouveau type de nœud a un [niveau de durabilité][durability] Silver et une taille « Standard_D2_V2 ».

Dans le fichier *template.json*, ajoutez les nouveaux paramètres suivants :
```json
"nt3InstanceCount": {
    "defaultValue": 5,
    "type": "Int",
    "metadata": {
        "description": "Instance count for node type"
    }
},
"vmNodeType3Size": {
    "defaultValue": "Standard_D2_V2",
    "type": "String"
},
```

Dans le fichier *template.json*, ajoutez les nouvelles variables suivantes :
```json
"lbID3": "[resourceId('Microsoft.Network/loadBalancers',concat('LB','-', parameters('clusterName'),'-',variables('vmNodeType3Name')))]",
"lbIPConfig3": "[concat(variables('lbID3'),'/frontendIPConfigurations/LoadBalancerIPConfig')]",
"lbPoolID3": "[concat(variables('lbID3'),'/backendAddressPools/LoadBalancerBEAddressPool')]",
"lbProbeID3": "[concat(variables('lbID3'),'/probes/FabricGatewayProbe')]",
"lbHttpProbeID3": "[concat(variables('lbID3'),'/probes/FabricHttpGatewayProbe')]",
"lbNatPoolID3": "[concat(variables('lbID3'),'/inboundNatPools/LoadBalancerBEAddressNatPool')]",
"vmNodeType3Name": "[toLower(concat('NT4', variables('vmName')))]",
"vmStorageAccountName3": "[toLower(concat(uniqueString(resourceGroup().id), '1', '3' ))]",
"nt3applicationStartPort": "20000",
"nt3applicationEndPort": "30000",
"nt3ephemeralStartPort": "49152",
"nt3ephemeralEndPort": "65534",
"nt3fabricTcpGatewayPort": "19000",
"nt3fabricHttpGatewayPort": "19080",
"nt3reverseProxyEndpointPort": "19081",
"subnet3Name": "Subnet-3",
"subnet3Prefix": "10.0.3.0/24",
"subnet3Ref": "[concat(variables('vnetID'),'/subnets/',variables('subnet3Name'))]",
```

Dans le fichier *template.json*, ajoutez un nouveau sous-réseau à la ressource de réseau virtuel :
```json
{
    "type": "Microsoft.Network/virtualNetworks",
    "name": "[variables('virtualNetworkName')]",
    "apiVersion": "2018-08-01",
    "location": "[variables('computeLocation')]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "addressSpace": {
            "addressPrefixes": [
                "[variables('addressPrefix')]"
            ]
        },
        "subnets": [
            ...
            {
                "name": "[variables('subnet3Name')]",
                "properties": {
                    "addressPrefix": "[variables('subnet3Prefix')]",
                    "networkSecurityGroup": {
                        "id": "[resourceId('Microsoft.Network/networkSecurityGroups', concat('nsg', variables('subnet3Name')))]"
                    }
                }
            }
        ]
    },
    "dependsOn": [
        ...
        "[concat('Microsoft.Network/networkSecurityGroups/', concat('nsg', variables('subnet3Name')))]"
    ]
},
```

Dans le fichier *template.json*, ajoutez de nouvelles adresses IP publiques et ressources d’équilibreur de charge :
```json
{
    "type": "Microsoft.Network/publicIPAddresses",
    "name": "[concat(variables('lbIPName'),'-',variables('vmNodeType3Name'))]",
    "apiVersion": "2018-08-01",
    "location": "[variables('computeLocation')]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "dnsSettings": {
            "domainNameLabel": "[concat(variables('dnsName'),'-','nt4')]"
        },
        "publicIPAllocationMethod": "Dynamic"
    }
},
        {
    "type": "Microsoft.Network/loadBalancers",
    "name": "[concat('LB','-', parameters('clusterName'),'-',variables('vmNodeType3Name'))]",
    "apiVersion": "2018-08-01",
    "location": "[variables('computeLocation')]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "frontendIPConfigurations": [
            {
                "name": "LoadBalancerIPConfig",
                "properties": {
                    "publicIPAddress": {
                        "id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(variables('lbIPName'),'-',variables('vmNodeType3Name')))]"
                    }
                }
            }
        ],
        "backendAddressPools": [
            {
                "name": "LoadBalancerBEAddressPool",
                "properties": {}
            }
        ],
        "loadBalancingRules": [
            {
                "name": "LBRule",
                "properties": {
                    "backendAddressPool": {
                        "id": "[variables('lbPoolID3')]"
                    },
                    "backendPort": "[variables('nt3fabricTcpGatewayPort')]",
                    "enableFloatingIP": "false",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig3')]"
                    },
                    "frontendPort": "[variables('nt3fabricTcpGatewayPort')]",
                    "idleTimeoutInMinutes": "5",
                    "probe": {
                        "id": "[variables('lbProbeID3')]"
                    },
                    "protocol": "tcp"
                }
            },
            {
                "name": "LBHttpRule",
                "properties": {
                    "backendAddressPool": {
                        "id": "[variables('lbPoolID3')]"
                    },
                    "backendPort": "[variables('nt3fabricHttpGatewayPort')]",
                    "enableFloatingIP": "false",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig3')]"
                    },
                    "frontendPort": "[variables('nt3fabricHttpGatewayPort')]",
                    "idleTimeoutInMinutes": "5",
                    "probe": {
                        "id": "[variables('lbHttpProbeID3')]"
                    },
                    "protocol": "tcp"
                }
            },
            {
                "name": "AppPortLBRule1",
                "properties": {
                    "backendAddressPool": {
                        "id": "[variables('lbPoolID3')]"
                    },
                    "backendPort": "[parameters('loadBalancedAppPort1')]",
                    "enableFloatingIP": "false",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig3')]"
                    },
                    "frontendPort": "[parameters('loadBalancedAppPort1')]",
                    "idleTimeoutInMinutes": "5",
                    "probe": {
                        "id": "[concat(variables('lbID3'),'/probes/AppPortProbe1')]"
                    },
                    "protocol": "tcp"
                }
            },
            {
                "name": "AppPortLBRule2",
                "properties": {
                    "backendAddressPool": {
                        "id": "[variables('lbPoolID3')]"
                    },
                    "backendPort": "[parameters('loadBalancedAppPort2')]",
                    "enableFloatingIP": "false",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig3')]"
                    },
                    "frontendPort": "[parameters('loadBalancedAppPort2')]",
                    "idleTimeoutInMinutes": "5",
                    "probe": {
                        "id": "[concat(variables('lbID3'),'/probes/AppPortProbe2')]"
                    },
                    "protocol": "tcp"
                }
            }
        ],
        "probes": [
            {
                "name": "FabricGatewayProbe",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port": "[variables('nt3fabricTcpGatewayPort')]",
                    "protocol": "tcp"
                }
            },
            {
                "name": "FabricHttpGatewayProbe",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port": "[variables('nt3fabricHttpGatewayPort')]",
                    "protocol": "tcp"
                }
            },
            {
                "name": "AppPortProbe1",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port": "[parameters('loadBalancedAppPort1')]",
                    "protocol": "tcp"
                }
            },
            {
                "name": "AppPortProbe2",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port": "[parameters('loadBalancedAppPort2')]",
                    "protocol": "tcp"
                }
            }
        ],
        "inboundNatPools": [
            {
                "name": "LoadBalancerBEAddressNatPool",
                "properties": {
                    "backendPort": "3389",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig3')]"
                    },
                    "frontendPortRangeEnd": "4500",
                    "frontendPortRangeStart": "3389",
                    "protocol": "tcp"
                }
            }
        ]
    },
    "dependsOn": [
        "[concat('Microsoft.Network/publicIPAddresses/',concat(variables('lbIPName'),'-',variables('vmNodeType3Name')))]"
    ]
},
```

Dans le fichier *template.json*, ajoutez les nouvelles ressources de groupe de machines virtuelles identiques et de groupe de sécurité réseau.  La propriété NodeTypeRef dans les propriétés d’extension Service Fabric du groupe de machines virtuelles identiques mappe le type de nœud spécifié au groupe identique.

```json
{
    "type": "Microsoft.Network/networkSecurityGroups",
    "name": "[concat('nsg', variables('subnet3Name'))]",
    "apiVersion": "2018-08-01",
    "location": "[resourceGroup().location]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "securityRules": [
            {
                "name": "allowSvcFabSMB",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "445",
                    "direction": "Inbound",
                    "priority": 3950,
                    "protocol": "*",
                    "sourceAddressPrefix": "VirtualNetwork",
                    "sourcePortRange": "*",
                    "description": "allow SMB traffic within the net, used by fabric to move packages around"
                }
            },
            {
                "name": "allowSvcFabCluser",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "1025-1027",
                    "direction": "Inbound",
                    "priority": 3920,
                    "protocol": "*",
                    "sourceAddressPrefix": "VirtualNetwork",
                    "sourcePortRange": "*",
                    "description": "allow ports within vnet that are used by the fabric to talk between nodes"
                }
            },
            {
                "name": "allowSvcFabEphemeral",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[concat(variables('nt3ephemeralStartPort'), '-', variables('nt3ephemeralEndPort'))]",
                    "direction": "Inbound",
                    "priority": 3930,
                    "protocol": "*",
                    "sourceAddressPrefix": "VirtualNetwork",
                    "sourcePortRange": "*",
                    "description": "allow fabric ephemeral ports within the vnet"
                }
            },
            {
                "name": "allowSvcFabPortal",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[variables('nt3fabricHttpGatewayPort')]",
                    "direction": "Inbound",
                    "priority": 3900,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow port used to access the fabric cluster web portal"
                }
            },
            {
                "name": "allowSvcFabClient",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[variables('nt3fabricTcpGatewayPort')]",
                    "direction": "Inbound",
                    "priority": 3910,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow port used by the fabric client (includes powershell)"
                }
            },
            {
                "name": "allowSvcFabApplication",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[concat(variables('nt3applicationStartPort'), '-', variables('nt3applicationEndPort'))]",
                    "direction": "Inbound",
                    "priority": 3940,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow fabric application ports within the vnet"
                }
            },
            {
                "name": "blockAll",
                "properties": {
                    "access": "Deny",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "*",
                    "direction": "Inbound",
                    "priority": 4095,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "block all traffic except what we've explicitly allowed"
                }
            },
            {
                "name": "allowVNetRDP",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "3389",
                    "direction": "Inbound",
                    "priority": 3960,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow RDP within the net"
                }
            },
            {
                "name": "allowSvcFabReverseProxy",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[variables('nt3reverseProxyEndpointPort')]",
                    "direction": "Inbound",
                    "priority": 3980,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow port used to access the fabric cluster using reverse proxy"
                }
            },
            {
                "name": "allowAppPort1",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[parameters('loadBalancedAppPort1')]",
                    "direction": "Inbound",
                    "priority": 2001,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow public application port 1"
                }
            },
            {
                "name": "allowAppPort2",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[parameters('loadBalancedAppPort2')]",
                    "direction": "Inbound",
                    "priority": 2002,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow public application port 2"
                }
            }
        ]
    }
},
{
    "type": "Microsoft.Compute/virtualMachineScaleSets",
    "sku": {
        "name": "[parameters('vmNodeType3Size')]",
        "capacity": "[parameters('nt3InstanceCount')]",
        "tier": "Standard"
    },
    "name": "[variables('vmNodeType3Name')]",
    "apiVersion": "2018-10-01",
    "location": "[variables('computeLocation')]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "overprovision": "[variables('overProvision')]",
        "upgradePolicy": {
            "mode": "Automatic"
        },
        "virtualMachineProfile": {
            "extensionProfile": {
                "extensions": [
                    {
                        "name": "[concat(variables('vmNodeType3Name'),'OMS')]",
                        "properties": {
                            "publisher": "Microsoft.EnterpriseCloud.Monitoring",
                            "type": "MicrosoftMonitoringAgent",
                            "typeHandlerVersion": "1.0",
                            "autoUpgradeMinorVersion": true,
                            "settings": {
                                "workspaceId": "[reference(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')), '2015-11-01-preview').customerId]"
                            },
                            "protectedSettings": {
                                "workspaceKey": "[listKeys(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')),'2015-11-01-preview').primarySharedKey]"
                            }
                        }
                    },
                    {
                        "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType3Name')]",
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
                                "nodeTypeRef": "[variables('vmNodeType3Name')]",
                                "dataPath": "D:\\SvcFab",
                                "durabilityLevel": "Silver",
                                "enableParallelJobs": true,
                                "nicPrefixOverride": "[variables('subnet3Prefix')]",
                                "certificate": {
                                    "thumbprint": "[parameters('certificateThumbprint')]",
                                    "x509StoreName": "[parameters('certificateStoreValue')]"
                                }
                            },
                            "typeHandlerVersion": "1.0"
                        }
                    },
                    {
                        "name": "[concat('VMDiagnosticsVmExt','_vmNodeType3Name')]",
                        "properties": {
                            "type": "IaaSDiagnostics",
                            "autoUpgradeMinorVersion": true,
                            "protectedSettings": {
                                "storageAccountName": "[variables('applicationDiagnosticsStorageAccountName')]",
                                "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('applicationDiagnosticsStorageAccountName')),'2015-05-01-preview').key1]",
                                "storageAccountEndPoint": "https://core.windows.net/"
                            },
                            "publisher": "Microsoft.Azure.Diagnostics",
                            "settings": {
                                "WadCfg": {
                                    "DiagnosticMonitorConfiguration": {
                                        "overallQuotaInMB": "50000",
                                        "EtwProviders": {
                                            "EtwEventSourceProviderConfiguration": [
                                                {
                                                    "provider": "Microsoft-ServiceFabric-Actors",
                                                    "scheduledTransferKeywordFilter": "1",
                                                    "scheduledTransferPeriod": "PT5M",
                                                    "DefaultEvents": {
                                                        "eventDestination": "ServiceFabricReliableActorEventTable"
                                                    }
                                                },
                                                {
                                                    "provider": "Microsoft-ServiceFabric-Services",
                                                    "scheduledTransferPeriod": "PT5M",
                                                    "DefaultEvents": {
                                                        "eventDestination": "ServiceFabricReliableServiceEventTable"
                                                    }
                                                }
                                            ],
                                            "EtwManifestProviderConfiguration": [
                                                {
                                                    "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8",
                                                    "scheduledTransferLogLevelFilter": "Information",
                                                    "scheduledTransferKeywordFilter": "4611686018427387904",
                                                    "scheduledTransferPeriod": "PT5M",
                                                    "DefaultEvents": {
                                                        "eventDestination": "ServiceFabricSystemEventTable"
                                                    }
                                                }
                                            ]
                                        }
                                    }
                                },
                                "StorageAccount": "[variables('applicationDiagnosticsStorageAccountName')]"
                            },
                            "typeHandlerVersion": "1.5"
                        }
                    },
                    {
                        "name": "[concat('VMIaaSAntimalware','_vmNodeType3Name')]",
                        "properties": {
                            "publisher": "Microsoft.Azure.Security",
                            "type": "IaaSAntimalware",
                            "typeHandlerVersion": "1.5",
                            "settings": {
                                "AntimalwareEnabled": "true",
                                "Exclusions": {
                                    "Paths": "D:\\SvcFab;D:\\SvcFab\\Log;C:\\Program Files\\Microsoft Service Fabric",
                                    "Processes": "Fabric.exe;FabricHost.exe;FabricInstallerService.exe;FabricSetup.exe;FabricDeployer.exe;ImageBuilder.exe;FabricGateway.exe;FabricDCA.exe;FabricFAS.exe;FabricUOS.exe;FabricRM.exe;FileStoreService.exe"
                                },
                                "RealtimeProtectionEnabled": "true",
                                "ScheduledScanSettings": {
                                    "isEnabled": "true",
                                    "scanType": "Quick",
                                    "day": "7",
                                    "time": "120"
                                }
                            },
                            "protectedSettings": null
                        }
                    }
                ]
            },
            "networkProfile": {
                "networkInterfaceConfigurations": [
                    {
                        "name": "[concat(variables('nicName'), '-2')]",
                        "properties": {
                            "ipConfigurations": [
                                {
                                    "name": "[concat(variables('nicName'),'-',2)]",
                                    "properties": {
                                        "loadBalancerBackendAddressPools": [
                                            {
                                                "id": "[variables('lbPoolID3')]"
                                            }
                                        ],
                                        "loadBalancerInboundNatPools": [
                                            {
                                                "id": "[variables('lbNatPoolID3')]"
                                            }
                                        ],
                                        "subnet": {
                                            "id": "[variables('subnet3Ref')]"
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
                "computernamePrefix": "[variables('vmNodeType3Name')]",
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
                    "publisher": "[parameters('vmImagePublisher')]",
                    "offer": "[parameters('vmImageOffer')]",
                    "sku": "[parameters('vmImageSku')]",
                    "version": "[parameters('vmImageVersion')]"
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
    "dependsOn": [
        "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]",
        "[concat('Microsoft.Network/loadBalancers/', concat('LB','-', parameters('clusterName'),'-',variables('vmNodeType3Name')))]",
        "[concat('Microsoft.Storage/storageAccounts/', variables('supportLogStorageAccountName'))]",
        "[concat('Microsoft.Storage/storageAccounts/', variables('applicationDiagnosticsStorageAccountName'))]"
    ]
},
```

Dans le fichier *template.json*, mettez à jour la ressource de cluster et ajoutez un nouveau type de nœud :
```json
{
    "type": "Microsoft.ServiceFabric/clusters",
    "name": "[parameters('clusterName')]",
    "apiVersion": "2018-02-01",
    "location": "[parameters('clusterLocation')]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "nodeTypes": [
            ...
            {
                "name": "[variables('vmNodeType3Name')]",
                "applicationPorts": {
                    "endPort": "[variables('nt3applicationEndPort')]",
                    "startPort": "[variables('nt3applicationStartPort')]"
                },
                "clientConnectionEndpointPort": "[variables('nt3fabricTcpGatewayPort')]",
                "durabilityLevel": "Silver",
                "ephemeralPorts": {
                    "endPort": "[variables('nt3ephemeralEndPort')]",
                    "startPort": "[variables('nt3ephemeralStartPort')]"
                },
                "httpGatewayEndpointPort": "[variables('nt3fabricHttpGatewayPort')]",
                "isPrimary": false,
                "reverseProxyEndpointPort": "[variables('nt3reverseProxyEndpointPort')]",
                "vmInstanceCount": "[parameters('nt3InstanceCount')]"
            }
        ],    
    }
}                
```

Dans le fichier *parameters.json*, ajoutez les nouveaux paramètres et valeurs suivants :
```json
"nt3InstanceCount": {
    "Value": 5    
},
"vmNodeType3Size": {
    "Value": "Standard_D2_V2"
},
```

### <a name="deploy-the-updated-template"></a>Déployer le modèle mis à jour
Enregistrez les modifications apportées aux fichiers *template.json* et *parameters.json*.  Pour déployer le modèle mis à jour, exécutez la commande suivante :

```powershell
New-AzResourceGroupDeployment -ResourceGroupName sfclustertutorialgroup -TemplateFile c:\temp\template.json -TemplateParameterFile c:\temp\parameters.json -Name "AddingNodeType"
```
Ou la commande Azure CLI suivante :
```azurecli
az deployment group create --resource-group sfclustertutorialgroup --template-file c:\temp\template.json --parameters c:\temp\parameters.json
```

## <a name="remove-a-node-type-from-the-cluster"></a>Supprimer un type de nœud du cluster
Après avoir créé un cluster Service Fabric, vous pouvez rendre scalable un cluster horizontalement en supprimant un type de nœud (groupe de machines virtuelles identiques) et tous ses nœuds. Une mise à l’échelle peut s’effectuer à tout moment, même lorsque des charges de travail sont en cours d’exécution sur le cluster. Lorsque vous mettez vos nœuds à l’échelle, vos applications sont automatiquement mises à l’échelle.

> [!WARNING]
> Nous vous recommandons de ne pas utiliser fréquemment Remove-AzServiceFabricNodeType pour supprimer un type de nœud d’un cluster de production. Il s’agit d’une commande dangereuse, car elle supprime le groupe de machines virtuelles identiques derrière le type de nœud. 

Pour supprimer le type de nœud, exécutez l’applet de commande [Remove-AzServiceFabricNodeType](/powershell/module/az.servicefabric/remove-azservicefabricnodetype).  Le type de nœud doit être de [niveau de durabilité][durability] Silver ou Gold. L’applet de commande supprime le groupe identique associé au type de nœud et son exécution prend un certain temps.  Exécutez ensuite l’applet de commande [Remove-ServiceFabricNodeState](/powershell/module/servicefabric/remove-servicefabricnodestate?view=azureservicefabricps) sur chacun des nœuds à supprimer. Elle efface l’état du nœud et supprime les nœuds du cluster. S’il existe des services sur le nœuds, ils sont tout d’abord déplacés vers un autre nœud. Si le Gestionnaire du cluster ne trouve pas de nœud pour le réplica/service, l’opération est retardée/bloquée.

```powershell
$groupname = "sfclustertutorialgroup"
$nodetype = "nt4vm"
$clustername = "mysfcluster123"

Remove-AzServiceFabricNodeType -Name $clustername  -NodeType $nodetype -ResourceGroupName $groupname

Connect-ServiceFabricCluster -ConnectionEndpoint mysfcluster123.eastus.cloudapp.azure.com:19000 `
          -KeepAliveIntervalInSec 10 `
          -X509Credential -ServerCertThumbprint <thumbprint> `
          -FindType FindByThumbprint -FindValue <thumbprint> `
          -StoreLocation CurrentUser -StoreName My

$nodes = Get-ServiceFabricNode | Where-Object {$_.NodeType -eq $nodetype} | Sort-Object { $_.NodeName.Substring($_.NodeName.LastIndexOf('_') + 1) } -Descending

Foreach($node in $nodes)
{
    Remove-ServiceFabricNodeState -NodeName $node.NodeName -TimeoutSec 300 -Force 
}
```

## <a name="increase-node-resources"></a>Augmenter les ressources du nœud 
Une fois que vous avez créé un cluster Service Fabric, vous pouvez mettre à l’échelle le type de nœud d’un cluster verticalement (changement des ressources des nœuds) ou mettre à niveau le système d’exploitation des machines virtuelles du type de nœud en remplaçant le type de nœud d’origine par un nouveau type de nœud (avec image du système d’exploitation ou référence SKU de machine virtuelle mise à jour). Pour plus d’informations, consultez [Effectuer un scale-up d’un type de nœud Azure Service Fabric](service-fabric-scale-up-primary-node-type.md).

> [!IMPORTANT]
> Ne tentez jamais une modification sur place d’une référence SKU de machine virtuelle ou d’une image de système d’exploitation, car cette opération est dangereuse et non prise en charge.

Si cela n’est pas possible, créez un cluster et [restaurez l’état des applications](service-fabric-reliable-services-backup-restore.md) (le cas échéant) de votre ancien cluster. Vous n’avez pas besoin de restaurer l’état des services système ; ceux-ci sont recréés pendant le déploiement de vos applications sur le nouveau cluster. Si les applications que vous exécutiez sur votre cluster étaient sans état, tout ce que vous avez à faire, c’est déployer vos applications sur le nouveau cluster, sans rien avoir à restaurer.

### <a name="update-the-template"></a>Mettre à jour le modèle

[Exportez un fichier de modèle et de paramètres](#export-the-template-for-the-resource-group) à partir du groupe de ressources pour le déploiement le plus récent.  Ouvrez le fichier *parameters.json*.  Si vous avez déployé le cluster à l’aide de l’[exemple de modèle][template] dans ce tutoriel, il existe trois types de nœuds dans le cluster.  

La taille des machines virtuelles dans le deuxième type de nœud est définie dans le paramètre *vmNodeType1Size*.  Changez la valeur du paramètre *vmNodeType1Size* de Standard_D2_V2 en [Standard_D3_V2](../virtual-machines/dv2-dsv2-series.md) afin de doubler les ressources de chaque instance de machine virtuelle.

La référence SKU des machines virtuelles pour les trois types de nœuds est défini dans le paramètre *vmImageSku*.  Là encore, la modification de la référence SKU des machines virtuelles d’un type de nœud doit être appréhendée avec précaution, et n’est pas recommandée pour le type de nœud principal.

### <a name="deploy-the-updated-template"></a>Déployer le modèle mis à jour
Enregistrez les modifications apportées aux fichiers *template.json* et *parameters.json*.  Pour déployer le modèle mis à jour, exécutez la commande suivante :

```powershell
New-AzResourceGroupDeployment -ResourceGroupName sfclustertutorialgroup -TemplateFile c:\temp\template.json -TemplateParameterFile c:\temp\parameters.json -Name "ScaleUpNodeType"
```
Ou la commande Azure CLI suivante :
```azurecli
az deployment group create --resource-group sfclustertutorialgroup --template-file c:\temp\template.json --parameters c:\temp\parameters.json
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Ajouter et supprimer des nœuds (scale-out et scale-in)
> * Ajouter et supprimer des types de nœuds (scale-out et scale-in)
> * Augmenter les ressources de nœud (scale up)

Maintenant, passez au didacticiel suivant pour savoir comment mettre à niveau le runtime d’un cluster.
> [!div class="nextstepaction"]
> [Mettre à niveau le runtime d’un cluster](service-fabric-tutorial-upgrade-cluster.md)

[durability]: service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster
[reliability]: service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster
[template]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Windows-3-NodeTypes-Secure-NSG/AzureDeploy.json
[parameters]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Windows-3-NodeTypes-Secure-NSG/AzureDeploy.Parameters.json
