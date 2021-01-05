---
title: Déployer un cluster sur plusieurs Zones de disponibilité
description: Découvrez comment déployer un cluster Azure Service Fabric sur des zones de disponibilité.
author: peterpogorski
ms.topic: conceptual
ms.date: 04/25/2019
ms.author: pepogors
ms.openlocfilehash: 73a3be62e57991b63525372f008e15d8e4f36a74
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97401727"
---
# <a name="deploy-an-azure-service-fabric-cluster-across-availability-zones"></a>Déployer un cluster Azure Service Fabric sur des zones de disponibilité
Les zones de disponibilité dans Azure constituent une offre à haute disponibilité qui protège vos applications et données contre les pannes des centres de données. Une zone de disponibilité est un emplacement physique unique équipé d’une alimentation, d’un refroidissement et d’une mise en réseau indépendants dans une région Azure.

Service Fabric prend en charge les clusters qui s’étendent sur des zones de disponibilité en déployant des types de nœuds qui sont épinglés à des zones spécifiques. Ceci garantit la haute disponibilité de vos applications. Les zones de disponibilité Azure sont disponibles uniquement dans les régions sélectionnées. Pour plus d’informations, consultez [Vue d’ensemble des zones de disponibilité Azure](../availability-zones/az-overview.md).

Des exemples de modèles sont disponibles : [Modèle entre zones de disponibilité Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates)

## <a name="recommended-topology-for-primary-node-type-of-azure-service-fabric-clusters-spanning-across-availability-zones"></a>Topologie recommandée de type de nœud principal de clusters Azure Service Fabric s’étendant sur des zones de disponibilité
Un cluster Service Fabric distribué entre des zones de disponibilité garantit la haute disponibilité de l’état du cluster. Pour étendre un cluster Service Fabric sur des zones, vous devez créer un type de nœud principal dans chaque zone de disponibilité prise en charge par la région. Les nœuds initiaux seront ainsi distribués uniformément sur chacun des types de nœuds principaux.

La topologie recommandée du type de nœud principal nécessite les ressources décrites ci-dessous :

* Le niveau de fiabilité du cluster défini sur Platinum.
* Trois types de nœuds marqués comme principaux.
    * Chaque type de nœud doit être mappé à son propre groupe de machines virtuelles identiques situé dans différentes zones.
    * Chaque groupe de machines virtuelles identiques doit avoir au moins cinq nœuds (durabilité Silver).
* Une ressource d’adresse IP publique unique utilisant une référence SKU standard.
* Une ressource de Load Balancer unique utilisant une référence SKU standard.
* Un groupe de sécurité réseau (NSG) référencé par le sous-réseau dans lequel vous déployez vos groupes de machines virtuelles identiques.

>[!NOTE]
> La propriété de groupe de placement de groupe de machines virtuelles identiques doit être définie sur true (vrai) car Service Fabric ne prend pas en charge un groupe de machines virtuelles identiques couvrant des zones.

 ![Architecture de zone de disponibilité Azure Service Fabric][sf-architecture]

## <a name="networking-requirements"></a>Configuration requise du réseau
### <a name="public-ip-and-load-balancer-resource"></a>Ressource d’adresse IP publique et Load Balancer
Pour activer la propriété de zones sur une ressource de groupe de machines virtuelles identiques, les ressources d’équilibreur de charge et d’adresse IP référencées par ce groupe de machines virtuelles identiques doivent toutes deux utiliser une référence SKU *Standard*. La création d’une ressource d’équilibreur de charge ou d’adresse IP sans la propriété de référence SKU créera une référence SKU de base, qui ne prend pas en charge les zones de disponibilité. Un équilibreur de charge de référence SKU Standard bloque par défaut tout le trafic provenant de l’extérieur ; pour autoriser le trafic de l’extérieur, un groupe de sécurité réseau (NSG) doit être déployé sur le sous-réseau.

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
```

```json
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


### <a name="enabling-zones-on-a-virtual-machine-scale-set"></a>Activation de zones sur un groupe de machines virtuelles identiques
Pour activer une zone sur un groupe de machines virtuelles identiques, vous devez inclure les trois valeurs suivantes dans la ressource de groupe de machines virtuelles identiques.

* La première valeur est la propriété **zones**, qui spécifie la zone de disponibilité sur laquelle le groupe de machines virtuelles identiques sera déployé.
* La deuxième valeur est la propriété « singlePlacementGroup », qui doit être définie sur true (vrai).
* La troisième valeur est la propriété « faultDomainOverride » dans l’extension de groupe de machines virtuelles identiques Service Fabric. La valeur de cette propriété doit uniquement inclure la zone dans lesquelles ce groupe de machines virtuelles identiques sera placé. Exemple : « faultDomainOverride » : « az1 ». Toutes les ressources de groupe de machines virtuelles identiques doivent être placées dans la même région car les clusters Azure Service Fabric ne prennent pas en charge entre régions.

```json
{
    "apiVersion": "2018-10-01",
    "type": "Microsoft.Compute/virtualMachineScaleSets",
    "name": "[parameters('vmNodeType1Name')]",
    "location": "[parameters('computeLocation')]",
    "zones": ["1"],
    "properties": {
        "singlePlacementGroup": "true",
    },
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
            "faultDomainOverride": "az1"
        },
        "typeHandlerVersion": "1.0"
    }
}
```

### <a name="enabling-multiple-primary-node-types-in-the-service-fabric-cluster-resource"></a>Activation de plusieurs types de nœuds principaux dans la ressource de cluster Service Fabric
Pour définir un ou plusieurs types de nœuds comme principaux dans une ressource de cluster, définissez la propriété « isPrimary » sur « true ». Lorsque vous déployez un cluster Service Fabric entre des zones de disponibilité, vous devez avoir trois types de nœuds dans des zones distinctes.

```json
{
    "reliabilityLevel": "Platinum",
    "nodeTypes": [
    {
        "name": "[parameters('vmNodeType0Name')]",
        "applicationPorts": {
            "endPort": "[parameters('nt0applicationEndPort')]",
            "startPort": "[parameters('nt0applicationStartPort')]"
        },
        "clientConnectionEndpointPort": "[parameters('nt0fabricTcpGatewayPort')]",
        "durabilityLevel": "Silver",
        "ephemeralPorts": {
            "endPort": "[parameters('nt0ephemeralEndPort')]",
            "startPort": "[parameters('nt0ephemeralStartPort')]"
        },
        "httpGatewayEndpointPort": "[parameters('nt0fabricHttpGatewayPort')]",
        "isPrimary": true,
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
        "isPrimary": true,
        "vmInstanceCount": "[parameters('nt1InstanceCount')]"
    },
    {
        "name": "[parameters('vmNodeType2Name')]",
        "applicationPorts": {
            "endPort": "[parameters('nt2applicationEndPort')]",
            "startPort": "[parameters('nt2applicationStartPort')]"
        },
        "clientConnectionEndpointPort": "[parameters('nt2fabricTcpGatewayPort')]",
        "durabilityLevel": "Silver",
        "ephemeralPorts": {
            "endPort": "[parameters('nt2ephemeralEndPort')]",
            "startPort": "[parameters('nt2ephemeralStartPort')]"
        },
        "httpGatewayEndpointPort": "[parameters('nt2fabricHttpGatewayPort')]",
        "isPrimary": true,
        "vmInstanceCount": "[parameters('nt2InstanceCount')]"
    }
    ],
}
```

## <a name="migrate-to-using-availability-zones-from-a-cluster-using-a-basic-sku-load-balancer-and-a-basic-sku-ip"></a>Migrer à l’aide de zones de disponibilité à partir d’un cluster à l’aide d’un Load Balancer de référence SKU de base et d’une adresse IP de référence SKU de base
Pour migrer un cluster qui utilisait un Load Balancer et une adresse IP avec une référence SKU de base, vous devez tout d’abord créer une toute nouvelle ressource de Load Balancer et d’adresse à l’aide de la référence SKU standard. Il n’est pas possible de mettre à jour ces ressources sur place.

Les nouveaux équilibreur de charge et adresse IP doivent être référencés dans les nouveaux types de nœuds entre zones de disponibilité que vous souhaitez utiliser. Dans l’exemple ci-dessus, trois nouvelles ressources de groupe de machines virtuelles identiques ont été ajoutées dans les zones 1, 2 et 3. Ces groupes de machines virtuelles identiques font référence aux nouveaux équilibreur de charge et adresse IP et sont marqués comme des types de nœuds principaux dans la ressource de cluster Service Fabric.

Pour commencer, vous devez ajouter les nouvelles ressources à votre modèle Resource Manager existant. Ces ressources incluent :
* Une ressource d’adresse IP publique utilisant une référence SKU standard.
* Une ressource de Load Balancer utilisant une référence SKU standard.
* Un groupe de sécurité réseau (NSG) référencé par le sous-réseau dans lequel vous déployez vos groupes de machines virtuelles identiques.
* Trois types de nœuds marqués comme principaux.
    * Chaque type de nœud doit être mappé à son propre groupe de machines virtuelles identiques situé dans différentes zones.
    * Chaque groupe de machines virtuelles identiques doit avoir au moins cinq nœuds (durabilité Silver).

Vous trouverez un exemple de ces ressources dans l’[exemple de modèle](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/10-VM-Ubuntu-2-NodeType-Secure).

```powershell
New-AzureRmResourceGroupDeployment `
    -ResourceGroupName $ResourceGroupName `
    -TemplateFile $Template `
    -TemplateParameterFile $Parameters
```

Une fois le déploiement des ressources terminé, vous pouvez commencer à désactiver les nœuds dans le type de nœud principal du cluster d’origine. Comme les nœuds sont désactivés, les services système migrent vers le nouveau type de nœud principal qui a été déployé à l’étape précédente.

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint $ClusterName `
    -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCertThumbprint $thumb  `
    -FindType FindByThumbprint `
    -FindValue $thumb `
    -StoreLocation CurrentUser `
    -StoreName My 

Write-Host "Connected to cluster"

$nodeNames = @("_nt0_0", "_nt0_1", "_nt0_2", "_nt0_3", "_nt0_4")

Write-Host "Disabling nodes..."
foreach($name in $nodeNames) {
    Disable-ServiceFabricNode -NodeName $name -Intent RemoveNode -Force
}
```

Lorsque tous les nœuds sont désactivés, les services système sont exécutés sur le type de nœud principal, qui s’étend entre des zones. Vous pouvez ensuite supprimer les nœuds désactivés du cluster. Lorsque les nœuds sont supprimés, vous pouvez supprimer les ressources d’adresse IP, de Load Balancer et de groupe de machines virtuelles identiques d’origine.

```powershell
foreach($name in $nodeNames){
    # Remove the node from the cluster
    Remove-ServiceFabricNodeState -NodeName $name -TimeoutSec 300 -Force
    Write-Host "Removed node state for node $name"
}

$scaleSetName="nt0"
Remove-AzureRmVmss -ResourceGroupName $groupname -VMScaleSetName $scaleSetName -Force

$lbname="LB-cluster-nt0"
$oldPublicIpName="LBIP-cluster-0"
$newPublicIpName="LBIP-cluster-1"

Remove-AzureRmLoadBalancer -Name $lbname -ResourceGroupName $groupname -Force
Remove-AzureRmPublicIpAddress -Name $oldPublicIpName -ResourceGroupName $groupname -Force
```

Vous devez ensuite supprimer les références à ces ressources du modèle Resource Manager que vous avez déployé.

La dernière étape implique de mettre à jour le nom DNS et l’adresse IP publique.

```powershell
$oldprimaryPublicIP = Get-AzureRmPublicIpAddress -Name $oldPublicIpName  -ResourceGroupName $groupname
$primaryDNSName = $oldprimaryPublicIP.DnsSettings.DomainNameLabel
$primaryDNSFqdn = $oldprimaryPublicIP.DnsSettings.Fqdn

Remove-AzureRmLoadBalancer -Name $lbname -ResourceGroupName $groupname -Force
Remove-AzureRmPublicIpAddress -Name $oldPublicIpName -ResourceGroupName $groupname -Force

$PublicIP = Get-AzureRmPublicIpAddress -Name $newPublicIpName  -ResourceGroupName $groupname
$PublicIP.DnsSettings.DomainNameLabel = $primaryDNSName
$PublicIP.DnsSettings.Fqdn = $primaryDNSFqdn
Set-AzureRmPublicIpAddress -PublicIpAddress $PublicIP

```

## <a name="preview-enable-multiple-availability-zones-in-single-virtual-machine-scale-set"></a>(Préversion) Activer plusieurs zones de disponibilité dans un seul groupe de machines virtuelles identiques

La solution mentionnée précédemment utilise un nodeType par AZ. La solution suivante permet aux utilisateurs de déployer 3 AZ dans le même nodeType.

L’exemple de modèle complet se trouve [ici](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/15-VM-Windows-Multiple-AZ-Secure).

![Architecture de zone de disponibilité Azure Service Fabric][sf-multi-az-arch]

### <a name="configuring-zones-on-a-virtual-machine-scale-set"></a>Configuration de zones sur un groupe de machines virtuelles identiques
Pour activer des zones sur un groupe de machines virtuelles identiques, vous devez inclure les trois valeurs suivantes dans la ressource de groupe de machines virtuelles identiques.

* La première valeur est la propriété **zones** qui spécifie les zones de disponibilité présentes dans le groupe de machines virtuelles identiques.
* La deuxième valeur est la propriété « singlePlacementGroup », qui doit être définie sur true (vrai).
* La troisième valeur, « zoneBalance » est facultative. Elle garantit un strict équilibrage strict des zones si la valeur est true. Découvrez [zoneBalancing](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-availability-zones#zone-balancing).
* Les remplacements FaultDomain et UpgradeDomain ne doivent pas être configurés.

```json
{
    "apiVersion": "2018-10-01",
    "type": "Microsoft.Compute/virtualMachineScaleSets",
    "name": "[parameters('vmNodeType1Name')]",
    "location": "[parameters('computeLocation')]",
    "zones": ["1", "2", "3"],
    "properties": {
        "singlePlacementGroup": "true",
        "zoneBalance": false
    }
}
```

>[!NOTE]
> * **Les clusters DF doivent avoir au moins un nodeType principal. La valeur de durabilityLevel des nodeTypes principaux doit être Silver ou une valeur supérieure.**
> * Le groupe de machines virtuelles identiques couvrant AZ doit être configuré avec au moins 3 zones de disponibilité, quel que soit la valeur de durabilityLevel.
> * Un groupe de machines virtuelles identiques couvrant AZ défini avec une durabilité Silver (ou supérieure) doit compter au moins 15 machines virtuelles.
> * Un groupe de machines virtuelles identiques couvrant AZ défini avec une durabilité Bronze doit compter au moins 6 machines virtuelles.

### <a name="enabling-the-support-for-multiple-zones-in-the-service-fabric-nodetype"></a>Activation de la prise en charge de plusieurs zones dans le nodeType Service Fabric
Le nodeType Service Fabric doit être activé pour prendre en charge plusieurs zones de disponibilité.

* La première valeur, **multipleAvailabilityZones**, doit être définie sur true pour le nodeType.
* La deuxième valeur, **sfZonalUpgradeMode**, est facultative. Il n’est pas possible de modifier cette propriété si un nodeType avec plusieurs AZ est déjà présent dans le cluster.
      La propriété contrôle le regroupement logique des machines virtuelles dans les domaines de mise à niveau.
          Si la valeur est définie sur false (mode plat) : les machines virtuelles sous le type de nœud seront regroupées en UD en ignorant les informations de zone dans 5 UD.
          Si la valeur est omise ou définie sur true (mode hiérarchique) : les machines virtuelles sont regroupées pour refléter la distribution zonale jusqu’à 15 UD. Chacune des 3 zones aura 5 UD.
          Cette propriété définit uniquement le comportement de mise à niveau pour l’application ServiceFabric et les mises à niveau de code. Les mises à niveau du groupe de machines virtuelles identiques sous-jacentes seront toujours parallèles dans tous les AZ.
      Cette propriété n’a aucun impact sur la distribution d’UD pour les types de nœuds pour lesquels plusieurs zones ne sont pas activées.
* La troisième valeur est **vmssZonalUpgradeMode = Parallel**. Il s’agit d’une propriété *obligatoire* à configurer dans le cluster en cas d’ajout d’un nodeType avec plusieurs AZ. Cette propriété définit le mode de mise à niveau pour les mises à jour du groupe de machines virtuelles identiques qui se produiront en parallèle simultanément dans tous les AZ.
      Actuellement, cette propriété ne peut être définie que sur parallèle.
* La ressource de cluster Service Fabric apiVersion doit être « 2020-12-01-preview » ou une version ultérieure.
* La version de code du cluster doit être « 7.2.445 » ou une version ultérieure.

```json
{
    "apiVersion": "2020-12-01-preview",
    "type": "Microsoft.ServiceFabric/clusters",
    "name": "[parameters('clusterName')]",
    "location": "[parameters('clusterLocation')]",
    "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/', parameters('supportLogStorageAccountName'))]"
    ],
    "properties": {
        "SFZonalUpgradeMode": "Hierarchical",
        "VMSSZonalUpgradeMode": "Parallel",
        "nodeTypes": [
          {
                "name": "[parameters('vmNodeType0Name')]",
                "multipleAvailabilityZones": true,
          }
        ]
}
```

>[!NOTE]
> * Les ressources d’adresse IP publique et d’équilibreur de charge doivent utiliser la référence (SKU) Standard, comme décrit précédemment dans l’article.
> * la propriété « multipleAvailabilityZones » sur le nodeType ne peut être définie qu’au moment de la création du nodeType, et ne peut pas être modifiée ultérieurement. Par conséquent, des nodeTypes existants ne peuvent pas être configurés avec cette propriété.
> * Quand « sfZonalUpgradeMode » est omis ou défini sur « Hierachical », les déploiements de cluster et d’application sont plus lents, car il y a plus de domaines de mise à niveau dans le cluster. Il est important d’ajuster correctement les délais d’expiration de stratégie de mise à niveau à incorporer pendant la durée de mise à niveau pour 15 domaines de mise à niveau.
> * Il est recommandé de définir le niveau de fiabilité du cluster sur Platinum pour garantir que le cluster résiste au scénario à une seule zone.

>[!NOTE]
> En guise de meilleure pratique, nous recommandons de définir sfZonalUpgradeMode sur Hierarchical ou de l’omettre. Le déploiement suivra la distribution zonale des machines virtuelles ayant un impact sur un plus petit nombre de réplicas et/ou d’instances, rendant celles-ci plus sûres.
> Utilisez sfZonalUpgradeMode défini sur Parallel si la vitesse de déploiement est une priorité ou si seule une charge de travail sans état s’exécute sur le type de nœud avec plusieurs AZ. Cela aura pour effet que le parcours d’UD se passera en parallèle dans tous les AZ.

### <a name="migration-to-the-node-type-with-multiple-availability-zones"></a>Migration vers le type de nœud avec plusieurs zones de disponibilité
Pour tous les scénarios de migration, un nouveau nodeType doit être ajouté, qui aura plusieurs zones de disponibilité prises en charge. Un nodeType existant ne peut pas être migré pour prendre en charge plusieurs zones.
L’article [ici](https://docs.microsoft.com/azure/service-fabric/service-fabric-scale-up-primary-node-type ) décrit en détail les étapes d’ajout d’un nouveau nodeType, ainsi que d’ajout d’autres ressources requises pour le nouveau nodeType, telles que les ressources IP et LB. Le même article décrit également comment retirer le nodeType existant après l’ajout du nodeType avec plusieurs zones de disponibilité au cluster.

* Migration à partir d’un nodeType qui utilise des ressources LB et IP de base : Cette procédure est déjà décrite [ici](https://docs.microsoft.com/azure/service-fabric/service-fabric-cross-availability-zones#migrate-to-using-availability-zones-from-a-cluster-using-a-basic-sku-load-balancer-and-a-basic-sku-ip) pour la solution avec un seul type de nœud par AZ. 
    Pour le nouveau type de nœud, la seule différence est qu’il n’y a qu’un seul groupe de machines virtuelles identiques et 1 nodeType pour tous les AZ au lieu de 1 par AZ.
* Migration à partir d’un nodeType utilisant les ressources LB et IP de référence (SKU) Standard avec un groupe de sécurité réseau :   Suivez la procédure décrite ci-dessus, sauf qu’il n’est pas nécessaire d’ajouter de nouvelles ressources LB, IP et NSG, et que les mêmes ressources peuvent être réutilisées dans le nouveau nodeType.


[sf-architecture]: ./media/service-fabric-cross-availability-zones/sf-cross-az-topology.png
[sf-multi-az-arch]: ./media/service-fabric-cross-availability-zones/sf-multi-az-topology.png
