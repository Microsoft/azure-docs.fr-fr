---
title: Mises à niveau automatiques d'images de système d'exploitation avec des groupes de machines virtuelles identiques Azure
description: Découvrez comment mettre à niveau automatiquement l’image de système d’exploitation sur des instances de machines virtuelles dans un groupe identique
author: avirishuv
ms.author: avverma
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 06/26/2020
ms.reviewer: jushiman
ms.custom: avverma, devx-track-azurecli
ms.openlocfilehash: 334e0c745257354d9548a6f9c8cee4d43fa8da6d
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92744744"
---
# <a name="azure-virtual-machine-scale-set-automatic-os-image-upgrades"></a>Mises à niveau automatiques d’images de système d’exploitation de groupes de machines virtuelles identiques Azure

L’activation des mises à niveau automatiques des images de système d’exploitation pour un groupe identique facilite la gestion des mises à jour en appliquant automatiquement et en toute sécurité chaque mise à niveau du disque du système d’exploitation à toutes les instances dans le groupe identique.

La fonctionnalité de mise à niveau automatique du système d’exploitation présente les caractéristiques suivantes :

- Une fois configurée, la dernière image du système d’exploitation publiée par les éditeurs de l’image est automatiquement appliquée au groupe identique, sans aucune intervention de l’utilisateur.
- Elle effectue une mise à niveau propagée de lots d’instances chaque fois qu’une nouvelle image est publiée par l’éditeur.
- Elle s’intègre avec les sondes d’intégrité d’application et l’[extension Intégrité de l’application](virtual-machine-scale-sets-health-extension.md).
- Elle fonctionne pour toutes les tailles de machine virtuelle et pour les images tant Windows que Linux.
- Vous pouvez désactiver les mises à niveau automatiques à tout moment (les mises à niveau du système d’exploitation peuvent également être démarrées manuellement).
- Le disque du système d’exploitation d’une machine virtuelle est remplacé par le nouveau disque de système d’exploitation créé avec la dernière version de l’image. Les extensions configurées et les scripts de données personnalisés sont exécutés. Les disques de données persistantes sont conservés.
- Le [séquencement d’extensions](virtual-machine-scale-sets-extension-sequencing.md) est pris en charge.
- La mise à niveau automatique de l’image du système d’exploitation peut être activée sur un groupe identique de n’importe quelle taille.

## <a name="how-does-automatic-os-image-upgrade-work"></a>Comment la mise à niveau automatique de l’image de système d’exploitation fonctionne-t-elle ?

Une mise à niveau fonctionne en remplaçant le disque du système d’exploitation d’une machine virtuelle par un nouveau disque créé à l’aide de la dernière version de l’image. Les extensions et scripts de données personnalisés configurés sont exécutés sur le disque du système d’exploitation, tandis que les disques de données persistantes sont conservés. Pour réduire au minimum le temps d’arrêt de l’application, les mises à niveau sont effectuées par lots, avec au maximum 20 % du groupe identique mis à niveau à la fois. Vous pouvez également intégrer une sonde d’intégrité d’application Azure Load Balancer ou une [extension Intégrité de l’application](virtual-machine-scale-sets-health-extension.md). Nous recommandons d’incorporer une pulsation de l’application et de valider la réussite de la mise à niveau pour chaque lot dans le processus de mise à niveau.

Le processus de mise à niveau se déroule comme suit :
1. Avant de commencer le processus de mise à niveau, l’orchestrateur vérifie qu’il n’y a pas plus de 20 % des instances dans tout le groupe identique qui présentent un état non sain (pour une raison ou une autre).
2. L’orchestrateur de mise à niveau identifie le lot d’instances de machines virtuelles à mettre à niveau, chaque lot devant compter au maximum 20 % du nombre total d’instances, sujet à une taille de lot maximale d’une machine virtuelle.
3. Le disque du système d’exploitation du lot sélectionné d’instances de machine virtuelle est remplacé par un nouveau disque de système d’exploitation créé à partir de l’image la plus récente. Toutes les extensions et configurations spécifiées dans le modèle de groupe identique sont appliquées à l’instance mise à niveau.
4. Pour les groupes identiques configurés avec des sondes d’intégrité d’application ou l’extension Intégrité de l’application, la mise à niveau attend (jusqu’à 5 minutes) que l’instance passe à l’état sain avant de commencer la mise à niveau du lot suivant. Si une instance ne récupère pas son intégrité en 5 minutes après une mise à niveau, le disque du système d’exploitation précédent pour l’instance est restauré par défaut.
5. L’orchestrateur de mise à niveau suit également le pourcentage d’instances qui deviennent non saines après une mise à niveau. La mise à niveau s’arrête si plus de 20 % des instances mises à niveau passent à l’état non sain pendant le processus de mise à niveau.
6. Le processus ci-dessus se poursuit jusqu’à ce que toutes les instances dans le groupe identique aient été mises à niveau.

L’orchestrateur de mise à niveau du système d’exploitation du groupe identique vérifie l’intégrité de tout le groupe identique avant de procéder à la mise à niveau de chaque lot. Durant la mise à niveau d’un lot, il peut arriver que d’autres activités de maintenance planifiées ou non planifiées aient lieu en même temps et impactent l’intégrité des instances du groupe identique. Si c’est le cas et que plus de 20 % des instances du groupe identique passent à l’état non sain, la mise à niveau du groupe identique s’arrête à la fin du lot en cours.

## <a name="supported-os-images"></a>Images de système d’exploitation prises en charge
Seules certaines images de plateforme de système d’exploitation sont actuellement prises en charge. Les images personnalisées [sont prises en charge](virtual-machine-scale-sets-automatic-upgrade.md#automatic-os-image-upgrade-for-custom-images) si le groupe identique utilise des images personnalisées via [Shared Image Gallery](shared-image-galleries.md).

Les références SKU de plateforme suivantes sont prises en charge (et d’autres sont régulièrement ajoutées) :

| Serveur de publication               | Offre de système d’exploitation      |  Sku               |
|-------------------------|---------------|--------------------|
| Canonical               | UbuntuServer  | 16.04-LTS          |
| Canonical               | UbuntuServer  | 18.04-LTS          |
| Rogue Wave (OpenLogic)  | CentOS        | 7.5                |
| CoreOS                  | CoreOS        | Stable             |
| Microsoft Corporation   | WindowsServer | 2012-R2-Datacenter |
| Microsoft Corporation   | WindowsServer | 2016-centre-de-données    |
| Microsoft Corporation   | WindowsServer | 2016-Datacenter-Smalldisk |
| Microsoft Corporation   | WindowsServer | 2016-centre-de-données-avec-conteneurs |
| Microsoft Corporation   | WindowsServer | 2019-Datacenter |
| Microsoft Corporation   | WindowsServer | 2019-Datacenter-Smalldisk |
| Microsoft Corporation   | WindowsServer | 2019-Datacenter-with-Containers |
| Microsoft Corporation   | WindowsServer | Datacenter-Core-1903-with-Containers-smalldisk |


## <a name="requirements-for-configuring-automatic-os-image-upgrade"></a>Conditions requises pour la configuration de la mise à niveau automatique d’image de système d’exploitation

- La propriété *version* de l’image doit être définie sur *latest* .
- Utilisez des sondes d’intégrité d’application ou l’[extension Intégrité de l’application](virtual-machine-scale-sets-health-extension.md) pour des groupes identiques autres que Service Fabric.
- Utilisez l’API de calcul de la version 2018-10-01 ou ultérieure.
- Assurez-vous que les ressources externes spécifiées dans le modèle de groupe identique sont disponibles et à jour. Les ressources concernées sont l’URI SAS pour l’amorçage de la charge utile dans les propriétés d’extension de machine virtuelle et dans le compte de stockage, les références à des secrets dans le modèle, etc.
- Pour les groupes identiques utilisant des machines virtuelles Windows, à partir de la version 2019-03-01 de l’API de calcul, la propriété *virtualMachineProfile.osProfile.windowsConfiguration.enableAutomaticUpdates* doit avoir la valeur *false* dans la définition du modèle de groupe identique. La propriété ci-dessus active les mises à niveau dans les machines virtuelles où « Windows Update » applique les correctifs du système d’exploitation sans remplacer le disque du système d’exploitation. Lorsque les mises à niveau automatiques de l’image du système d’exploitation sont activées sur votre groupe identique, une mise à jour supplémentaire effectuée par le biais de « Windows Update » n’est pas nécessaire.

### <a name="service-fabric-requirements"></a>Exigence pour Service Fabric

Si vous utilisez Service Fabric, assurez-vous que les conditions suivantes sont remplies :
-   Le [niveau de durabilité](../service-fabric/service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster) de Service Fabric est Silver ou Gold, et non Bronze.
-   L’extension Service Fabric sur la définition du modèle de groupe identique doit avoir TypeHandlerVersion 1.1 ou version ultérieure.
-   Le niveau de durabilité doit être identique au cluster Service Fabric et à l’extension de Service Fabric de la définition du modèle de groupe identique.
- Une sonde d’intégrité supplémentaire ou l’utilisation de l’extension d’intégrité d’application n’est pas nécessaire.

Assurez-vous que les paramètres de durabilité ne sont pas incompatibles avec le cluster Service Fabric et l’extension Service Fabric, car une incompatibilité entraînera des erreurs de mise à niveau. Les niveaux de durabilité peuvent être modifiés selon les instructions indiquées sur[cette page](../service-fabric/service-fabric-cluster-capacity.md#changing-durability-levels).


## <a name="automatic-os-image-upgrade-for-custom-images"></a>Mise à niveau automatique de l’image du système d’exploitation pour les images personnalisées

La mise à niveau automatique de l’image du système d’exploitation est prise en charge pour les images personnalisées déployées via [Shared Image Gallery](shared-image-galleries.md). Les autres images personnalisées ne sont pas prises en charge pour les mises à niveau automatiques de l’image du système d’exploitation.

### <a name="additional-requirements-for-custom-images"></a>Exigences supplémentaires pour les images personnalisées
- Le processus de paramétrage et de configuration de la mise à niveau automatique de l’image du système d’exploitation est le même pour tous les groupes identiques, comme indiqué dans la [section Configuration](virtual-machine-scale-sets-automatic-upgrade.md#configure-automatic-os-image-upgrade) de cette page.
- Les instances de groupes identiques configurées pour les mises à niveau automatiques de l’image du système d’exploitation sont mises à niveau vers la dernière version de l’image de la Galerie d’images partagées lorsqu’une nouvelle version de l’image est publiée et [répliquée](shared-image-galleries.md#replication) dans la région de ce groupe identique. Si la nouvelle image n’est pas répliquée vers la région où le groupe est déployé, les instances du groupe identique ne seront pas mises à niveau vers la dernière version. La réplication régionale d’images vous permet de contrôler le déploiement de la nouvelle image pour vos groupes identiques.
- La nouvelle version de l’image ne doit pas être exclue de la version la plus récente pour cette image de la galerie. Les versions d’image exclues de la dernière version de l’image de la galerie ne sont pas déployées dans le groupe identique via la mise à niveau automatique de l’image du système d’exploitation.

> [!NOTE]
>Un groupe identique peut nécessiter jusqu’à 3 heures pour lancer le premier déploiement de mise à niveau d’une image après la première configuration du groupe identique pour les mises à niveau automatiques du système d’exploitation. Il s’agit d’un délai unique par groupe identique. Les déploiements d’images suivants sont déclenchés sur le groupe identique dans un délai de 30 à 60 minutes.


## <a name="configure-automatic-os-image-upgrade"></a>Configurer la mise à niveau automatique d’image de système d’exploitation
Pour configurer la mise à niveau automatique d’image de système d’exploitation, vérifiez que la propriété *automaticOSUpgradePolicy.enableAutomaticOSUpgrade* est définie sur *true* dans la définition du modèle de groupe identique.

### <a name="rest-api"></a>API REST
L’exemple suivant montre comment activer les mises à niveau automatiques du système d’exploitation pour un modèle de groupe identique :

```
PUT or PATCH on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet?api-version=2019-12-01`
```

```json
{
  "properties": {
    "upgradePolicy": {
      "automaticOSUpgradePolicy": {
        "enableAutomaticOSUpgrade":  true
      }
    }
  }
}
```

### <a name="azure-powershell"></a>Azure PowerShell
Utilisez la cmdlet [Update-AzVmss](/powershell/module/az.compute/update-azvmss) pour configurer les mises à niveau automatiques d’image de système d’exploitation pour votre groupe identique. L’exemple suivant configure les mises à niveau automatiques pour le groupe identique nommé *myScaleSet* dans le groupe de ressources appelé *myResourceGroup*  :

```azurepowershell-interactive
Update-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -AutomaticOSUpgrade $true
```

### <a name="azure-cli-20"></a>Azure CLI 2.0
Utilisez la cmdlet [az vmss update](/cli/azure/vmss#az-vmss-update) pour configurer les mises à niveau automatiques d’image de système d’exploitation pour votre groupe identique. Utilisez Azure CLI 2.0.47 ou une version ultérieure. L’exemple suivant configure les mises à niveau automatiques pour le groupe identique nommé *myScaleSet* dans le groupe de ressources appelé *myResourceGroup*  :

```azurecli-interactive
az vmss update --name myScaleSet --resource-group myResourceGroup --set UpgradePolicy.AutomaticOSUpgradePolicy.EnableAutomaticOSUpgrade=true
```

> [!NOTE]
>Après la configuration des mises à niveau automatiques d’image de système d’exploitation pour votre groupe identique, vous devez également appliquer aux machines virtuelles du groupe identique le dernier modèle de groupe identique si votre groupe identique utilise la [stratégie de mise à jour](virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model) « Manuel ».

## <a name="using-application-health-probes"></a>Utilisation de sondes d’intégrité d’application

Durant une mise à niveau du système d’exploitation, les instances de machine virtuelle dans un groupe identique sont mises à niveau par lot. La mise à niveau doit se poursuivre uniquement si l’application du client est intègre sur les instances de machine virtuelle mises à niveau. Nous recommandons que l’application envoie des signaux d’intégrité au moteur de mise à niveau du système d’exploitation pour le groupe identique. Par défaut, pendant les mises à niveau du système d’exploitation, la plateforme vérifie l’état d’alimentation des machines virtuelles et l’état de provisionnement des extensions pour déterminer l’intégrité d’une instance de machine virtuelle après sa mise à niveau. Lors de la mise à niveau du système d’exploitation d’une instance de machine virtuelle, le disque du système d’exploitation sur l’instance de machine virtuelle est remplacé par un nouveau disque basé sur la dernière version de l’image. Après la mise à niveau du système d’exploitation, les extensions configurées sont exécutées sur ces machines virtuelles. L’application est considérée comme saine uniquement quand toutes les extensions sur l’instance ont été correctement provisionnées.

Un groupe identique peut éventuellement être configuré avec des sondes d’intégrité d’application pour fournir à la plateforme des informations précises sur l’état actuel de l’application. Les sondes d’intégrité d’application sont des sondes d’équilibreur de charge personnalisées qui sont utilisées comme signal d’intégrité. L’application exécutée sur une instance de machine virtuelle d’un groupe identique peut répondre à des requêtes HTTP ou TCP externes en indiquant si elle est saine. Pour plus d’informations sur le fonctionnement des sondes d’équilibreur de charge personnalisées, consultez [Comprendre les sondes d’équilibreur de charge](../load-balancer/load-balancer-custom-probe-overview.md). Les sondes d’intégrité d’application ne sont pas prises en charge pour des groupes identiques Service Fabric. Pour les groupes identiques autres que Service Fabric, vous devez utiliser des sondes d’intégrité d’application Load Balancer ou l’[extension Intégrité de l’application](virtual-machine-scale-sets-health-extension.md).

Si le groupe identique est configuré pour utiliser plusieurs groupes de sélection élective, vous devez utiliser des sondes avec un [équilibreur de charge standard](../load-balancer/load-balancer-overview.md).

### <a name="configuring-a-custom-load-balancer-probe-as-application-health-probe-on-a-scale-set"></a>Configuration d’une sonde d’équilibreur de charge personnalisée comme sonde d’intégrité d’application dans un groupe identique
La bonne pratique est de créer une sonde d’équilibreur de charge de manière explicite pour déterminer l’intégrité du groupe identique. Vous pouvez utiliser le même point de terminaison pour une sonde HTTP ou TCP existante, mais une sonde d’intégrité n’a pas forcément le même comportement qu’une sonde d’équilibreur de charge standard. Par exemple, une sonde d’équilibreur de charge standard signale un état non sain quand la charge sur l’instance est trop élevée, mais cela ne détermine pas forcément l’intégrité de l’instance durant une mise à niveau automatique du système d’exploitation. Configurez la sonde pour que le taux de sondage élevé soit inférieur à deux minutes.

La sonde d’équilibreur de charge peut être référencée dans la propriété *networkProfile* du groupe identique et être associée à un équilibreur de charge interne ou public, comme suit :

```json
"networkProfile": {
  "healthProbe" : {
    "id": "[concat(variables('lbId'), '/probes/', variables('sshProbeName'))]"
  },
  "networkInterfaceConfigurations":
  ...
}
```

> [!NOTE]
> Lors de l’utilisation de mises à niveau automatiques du système d’exploitation avec Service Fabric, la nouvelle image du système d’exploitation est déployée, un domaine de mise à jour après l’autre, pour maintenir la haute disponibilité des services en cours d’exécution dans Service Fabric. Pour utiliser les mises à niveau automatiques du système d’exploitation dans Service Fabric, votre cluster doit être configuré pour utiliser le niveau de durabilité Silver ou une version supérieure. Pour plus d’informations sur les caractéristiques de durabilité des clusters Service Fabric, voir [cette documentation](../service-fabric/service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster).

### <a name="keep-credentials-up-to-date"></a>Tenez à jour toutes les informations d’identification
Si votre groupe identique utilise des informations d’identification pour accéder à des ressources externes, telles qu’une extension de machine virtuelle configurée pour utiliser un jeton SAS pour le compte de stockage, assurez-vous que les informations d’identification sont mises à jour. Si les informations d’identification, y compris les certificats et les jetons, ont expiré, la mise à niveau échouera, et le premier lot de machines virtuelles restera dans un état d’échec.

Pour récupérer les machines virtuelles et réactiver la mise à niveau automatique du système d’exploitation après un échec d’authentification des ressources, effectuez les étapes recommandées suivantes :

* Regénérez le jeton (ou d’autres informations d’identification) qui a été passé à vos extensions.
* Vérifiez que toutes les informations d’identification utilisées pour les communications entre les machines virtuelles et les entités externes sont à jour.
* Mettez à jour chaque extension dans le modèle de groupe identique avec les nouveaux jetons.
* Déployez le groupe identique mis à jour. Cette opération met à jour toutes les instances de machine virtuelle, y compris celles en échec.

## <a name="using-application-health-extension"></a>Utilisation de l’extension Intégrité de l’application
L’extension Intégrité de l’application est déployée à l’intérieur d’une instance de groupe de machines virtuelles identiques et rend compte de l’intégrité des machines virtuelles à partir de l’instance de groupe identique. Vous pouvez configurer l’extension pour sonder un point de terminaison d’application et mettre à jour l’état de l’application sur cette instance. Cet état de l’instance est vérifié par Azure pour déterminer si une instance est éligible pour des opérations de mise à niveau.

Étant donné que l’extension rend compte de l’intégrité à partir d’une machine virtuelle, elle peut être utilisée dans les situations où les sondes externes telles que les sondes d’intégrité d’application (qui utilisent des [sondes](../load-balancer/load-balancer-custom-probe-overview.md) Azure Load Balancer personnalisées) ne peuvent pas être utilisées.

Il existe plusieurs façons de déployer l’extension Intégrité de l’application dans vos groupes identiques, comme cela est expliqué dans les exemples de [cet article](virtual-machine-scale-sets-health-extension.md#deploy-the-application-health-extension).

## <a name="get-the-history-of-automatic-os-image-upgrades"></a>Obtenir l’historique des mises à niveau automatiques d’image de système d’exploitation
Vous pouvez vérifier l’historique de la dernière mise à niveau du système d’exploitation effectuée dans un groupe identique à l’aide d’Azure PowerShell, d’Azure CLI 2.0 ou des API REST. Vous pouvez obtenir l’historique des cinq dernières tentatives de mise à niveau du système d’exploitation au cours des deux derniers mois.

### <a name="rest-api"></a>API REST
L’exemple suivant utilise l’ [API REST](/rest/api/compute/virtualmachinescalesets/getosupgradehistory) pour vérifier l’état du groupe identique nommé *myScaleSet* dans le groupe de ressources appelé *myResourceGroup*  :

```
GET on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/osUpgradeHistory?api-version=2019-12-01`
```

L’appel de Get retourne des propriétés similaires à l’exemple de sortie suivant :

```json
{
    "value": [
        {
            "properties": {
        "runningStatus": {
          "code": "RollingForward",
          "startTime": "2018-07-24T17:46:06.1248429+00:00",
          "completedTime": "2018-04-21T12:29:25.0511245+00:00"
        },
        "progress": {
          "successfulInstanceCount": 16,
          "failedInstanceCount": 0,
          "inProgressInstanceCount": 4,
          "pendingInstanceCount": 0
        },
        "startedBy": "Platform",
        "targetImageReference": {
          "publisher": "MicrosoftWindowsServer",
          "offer": "WindowsServer",
          "sku": "2016-Datacenter",
          "version": "2016.127.20180613"
        },
        "rollbackInfo": {
          "successfullyRolledbackInstanceCount": 0,
          "failedRolledbackInstanceCount": 0
        }
      },
      "type": "Microsoft.Compute/virtualMachineScaleSets/rollingUpgrades",
      "location": "westeurope"
    }
  ]
}
```

### <a name="azure-powershell"></a>Azure PowerShell
Utilisez l’applet de commande [Get-AzVmss](/powershell/module/az.compute/get-azvmss) afin de vérifier l’historique de mise à niveau du système d’exploitation pour votre groupe identique. L’exemple suivant montre comment vérifier l’état de la mise à niveau du système d’exploitation pour un groupe identique nommé *myScaleSet* dans le groupe de ressources appelé *myResourceGroup*  :

```azurepowershell-interactive
Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -OSUpgradeHistory
```

### <a name="azure-cli-20"></a>Azure CLI 2.0
Utilisez [az vmss get-os-upgrade-history](/cli/azure/vmss#az-vmss-get-os-upgrade-history) afin de vérifier l’historique de mise à niveau du système d’exploitation pour votre groupe identique Utilisez Azure CLI 2.0.47 ou une version ultérieure. L’exemple suivant montre comment vérifier l’état de la mise à niveau du système d’exploitation pour un groupe identique nommé *myScaleSet* dans le groupe de ressources appelé *myResourceGroup*  :

```azurecli-interactive
az vmss get-os-upgrade-history --resource-group myResourceGroup --name myScaleSet
```

## <a name="how-to-get-the-latest-version-of-a-platform-os-image"></a>Comment obtenir la dernière version d’une image de système d’exploitation de plateforme ?

Vous pouvez obtenir les versions disponibles d’image des références SKU prises en charge pour la mise à niveau automatique du système d’exploitation en utilisant les exemples ci-dessous :

### <a name="rest-api"></a>API REST
```
GET on `/subscriptions/subscription_id/providers/Microsoft.Compute/locations/{location}/publishers/{publisherName}/artifacttypes/vmimage/offers/{offer}/skus/{skus}/versions?api-version=2019-12-01`
```

### <a name="azure-powershell"></a>Azure PowerShell
```azurepowershell-interactive
Get-AzVmImage -Location "westus" -PublisherName "Canonical" -Offer "UbuntuServer" -Skus "16.04-LTS"
```

### <a name="azure-cli-20"></a>Azure CLI 2.0
```azurecli-interactive
az vm image list --location "westus" --publisher "Canonical" --offer "UbuntuServer" --sku "16.04-LTS" --all
```

## <a name="manually-trigger-os-image-upgrades"></a>Déclencher manuellement les mises à niveau d’images du système d’exploitation
Lorsque la mise à niveau automatique de l’image du système d’exploitation est activée sur votre groupe identique, vous n’avez pas besoin de déclencher manuellement les mises à jour. L’orchestrateur de mise à niveau du système d’exploitation applique automatiquement la dernière version de l’image disponible à vos instances de groupe identique sans aucune intervention manuelle.

Pour des cas spécifiques où vous ne souhaitez pas attendre que l’orchestrateur applique la dernière image, vous pouvez déclencher manuellement une mise à niveau de l’image du système d’exploitation à l’aide des exemples ci-dessous.

> [!NOTE]
> Le déclencheur manuel des mises à niveau d’images du système d’exploitation ne fournit pas de fonctionnalités de restauration automatique. Si une instance ne récupère pas son intégrité après une opération de mise à niveau, son disque de système d’exploitation précédent ne peut pas être restauré.

### <a name="rest-api"></a>API REST
Utilisez l’appel démarrer l’API de [mise à niveau du système d’exploitation](/rest/api/compute/virtualmachinescalesetrollingupgrades/startosupgrade) pour démarrer une mise à niveau propagée afin de déplacer toutes les instances du groupe de machines virtuelles identiques vers la dernière version disponible du système d’exploitation. Les instances qui exécutent déjà la dernière version du système d’exploitation disponible ne sont pas affectées. L’exemple suivant explique en détail comment vous pouvez démarrer une mise à niveau propagée du système d’exploitation sur un groupe identique nommé *myScaleSet* dans le groupe de ressources nommé *myResourceGroup*  :

```
POST on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/osRollingUpgrade?api-version=2019-12-01`
```

### <a name="azure-powershell"></a>Azure PowerShell
Utilisez le cmdlet de [Start-AzVmssRollingOSUpgrade](/powershell/module/az.compute/Start-AzVmssRollingOSUpgrade) afin de vérifier l’historique de mise à niveau du système d’exploitation pour votre groupe identique. L’exemple suivant explique en détail comment vous pouvez démarrer une mise à niveau propagée du système d’exploitation sur un groupe identique nommé *myScaleSet* dans le groupe de ressources nommé *myResourceGroup*  :

```azurepowershell-interactive
Start-AzVmssRollingOSUpgrade -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```

### <a name="azure-cli-20"></a>Azure CLI 2.0
Utilisez [az vmss rolling-upgrade start](/cli/azure/vmss/rolling-upgrade#az-vmss-rolling-upgrade-start) afin de vérifier l’historique de mise à niveau du système d’exploitation pour votre groupe identique. Utilisez Azure CLI 2.0.47 ou une version ultérieure. L’exemple suivant explique en détail comment vous pouvez démarrer une mise à niveau propagée du système d’exploitation sur un groupe identique nommé *myScaleSet* dans le groupe de ressources nommé *myResourceGroup*  :

```azurecli-interactive
az vmss rolling-upgrade start --resource-group "myResourceGroup" --name "myScaleSet" --subscription "subscriptionId"
```

## <a name="deploy-with-a-template"></a>Déployer les mises à niveau avec un modèle

Vous pouvez utiliser des modèles pour déployer un groupe identique avec les mises à niveau automatiques du système d’exploitation pour les images prises en charge comme [Ubuntu 16.04-LTS](https://github.com/Azure/vm-scale-sets/blob/master/preview/upgrade/autoupdate.json).

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fvm-scale-sets%2Fmaster%2Fpreview%2Fupgrade%2Fautoupdate.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png" alt="Button to Deploy to Azure." /></a>

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir d’autres exemples d’utilisation des mises à niveau automatiques du système d’exploitation avec des groupes identiques, consultez le [dépôt GitHub](https://github.com/Azure/vm-scale-sets/tree/master/preview/upgrade).
