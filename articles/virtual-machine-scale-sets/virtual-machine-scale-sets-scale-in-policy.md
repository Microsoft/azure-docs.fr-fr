---
title: Utiliser des stratégies de scale-in personnalisées avec des groupes de machines virtuelles identiques Azure
description: Découvrez comment utiliser des stratégies de scale-in personnalisées avec des groupes de machines virtuelles identiques Azure qui utilisent la configuration de scale-in automatique pour gérer le nombre d’instances.
services: virtual-machine-scale-sets
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 02/26/2020
ms.reviewer: avverma
ms.custom: avverma, devx-track-azurecli
ms.openlocfilehash: 9ca6310705d54d563aae746ab2dbfe6cb412e6a9
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92747799"
---
# <a name="use-custom-scale-in-policies-with-azure-virtual-machine-scale-sets"></a>Utiliser des stratégies de scale-in personnalisées avec des groupes de machines virtuelles identiques Azure

Vous pouvez effectuer un scale-out ou un scale-in du déploiement d’un groupe de machines virtuelles identiques en fonction d’un ensemble de métriques, notamment des métriques personnalisées de plateforme et définies par l’utilisateur. Alors qu’un scale-out crée des machines virtuelles basées sur le modèle de groupe identique, un scale-in affecte les machines virtuelles en cours d’exécution qui peuvent avoir des configurations et/ou des fonctions différentes à mesure que la charge de travail du groupe identique évolue. 

La fonctionnalité de stratégie de scale-in offre aux utilisateurs un moyen de configurer l’ordre dans lequel les machines virtuelles font l’objet du scale-in, par le biais de trois configurations de mise à l’échelle : 

1. Default
2. NewestVM
3. OldestVM

### <a name="default-scale-in-policy"></a>Stratégie de scale-in par défaut

Par défaut, le groupe de machines virtuelles identiques applique cette stratégie pour déterminer la ou les instances qui feront l’objet du scale-in. Avec la stratégie *Default* , les machines virtuelles sont sélectionnées pour le scale-in dans l’ordre suivant :

1. Équilibrer les machines virtuelles parmi les zones de disponibilité (si le groupe identique est déployé dans une configuration zonale)
2. Équilibrer les machines virtuelles parmi les domaines d’erreur (meilleur effort)
3. Supprimer la machine virtuelle avec l’ID d’instance la plus élevée

Les utilisateurs n’ont pas besoin de spécifier de stratégie de scale-in s’ils veulent juste que l’ordre par défaut soit respecté.

Notez que l’équilibrage parmi les zones de disponibilité ou les domaines d’erreur ne déplace pas les instances entre les zones de disponibilité ou les domaines d’erreur. L’équilibrage est effectué par le biais de la suppression de machines virtuelles des zones de disponibilité ou domaines d’erreur déséquilibrés jusqu’à ce que la distribution des machines virtuelles soit équilibrée.

### <a name="newestvm-scale-in-policy"></a>Stratégie de scale-in NewestVM

Cette stratégie supprime la machine virtuelle créée la plus récente dans le groupe identique, après l’équilibrage des machines virtuelles parmi les zones de disponibilité (pour les déploiements de zones). L’activation de cette stratégie nécessite une modification de la configuration du modèle de groupe de machines virtuelles identiques.

### <a name="oldestvm-scale-in-policy"></a>Stratégie de scale-in OldestVM

Cette stratégie supprime la machine virtuelle créée la plus ancienne dans le groupe identique, après l’équilibrage des machines virtuelles parmi les zones de disponibilité (pour les déploiements de zones). L’activation de cette stratégie nécessite une modification de la configuration du modèle de groupe de machines virtuelles identiques.

## <a name="enabling-scale-in-policy"></a>Activation de la stratégie de scale-in

Une stratégie de scale-in est définie dans le modèle de groupe de machines virtuelles identiques. Comme indiqué dans les sections ci-dessus, une définition de stratégie de scale-in est nécessaire lors de l’utilisation des stratégies « NewestVM » et « OldestVM ». Le groupe de machines virtuelles identiques utilisera automatiquement la stratégie de scale-in « Default » si aucune définition de stratégie de scale-in n’est trouvée sur le modèle de groupe identique. 

Une stratégie de scale-in peut être définie sur le modèle de groupe de machines virtuelles identiques des manières suivantes :

### <a name="azure-portal"></a>Portail Azure
 
Les étapes suivantes définissent la stratégie de scale-in lors de la création d’un groupe identique. 
 
1. Accédez à **Groupes identiques de machines virtuelles** .
1. Sélectionnez **+ Ajouter** pour créer un groupe identique.
1. Accédez à l’onglet **Mise à l’échelle** . 
1. Recherchez la section **Stratégie de scale-in** .
1. Sélectionnez une stratégie de scale-in dans la liste déroulante.
1. Lorsque vous avez fini de créer le groupe identique, sélectionnez le bouton **Vérifier + créer** .

### <a name="using-api"></a>Utilisation de l’API

Exécutez une instruction PUT sur le groupe de machines virtuelles identiques à l’aide de l’API 2019-03-01 :

```
PUT
https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<myRG>/providers/Microsoft.Compute/virtualMachineScaleSets/<myVMSS>?api-version=2019-03-01

{ 
"location": "<VMSS location>", 
    "properties": { 
        "scaleInPolicy": {  
            "rules": ["OldestVM"]  
        } 
    }    
} 
```
### <a name="azure-powershell"></a>Azure PowerShell

Créez un groupe de ressources, puis un groupe identique avec une stratégie de scale-in définie comme *OldestVM* .

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName "myResourceGroup" -Location "<VMSS location>"
New-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Location "<VMSS location>" `
  -VMScaleSetName "myScaleSet" `
  -ScaleInPolicy “OldestVM”
```

### <a name="azure-cli-20"></a>Azure CLI 2.0

L’exemple suivant ajoute une stratégie de scale-in lors de la création d’un groupe identique. Commencez par créer un groupe de ressources, puis créez un groupe identique avec une stratégie de scale-in *OldestVM* . 

```azurecli-interactive
az group create --name <myResourceGroup> --location <VMSSLocation>
az vmss create \
  --resource-group <myResourceGroup> \
  --name <myVMScaleSet> \
  --image UbuntuLTS \
  --admin-username <azureuser> \
  --generate-ssh-keys \
  --scale-in-policy OldestVM
```

### <a name="using-template"></a>Utilisation d’un modèle

Dans votre modèle, sous « properties », ajoutez ce qui suit :

```json
"scaleInPolicy": {  
      "rules": ["OldestVM"]  
}
```

Les blocs ci-dessus spécifient que le groupe de machines virtuelles identiques supprimera la machine virtuelle la plus ancienne dans un groupe identique à équilibrage de zone quand un scale-in est déclenché (par le biais de la mise à l’échelle automatique ou d’une suppression manuelle).

Quand un groupe de machines virtuelles identiques n’est pas équilibré parmi les zones, le groupe identique supprime d’abord les machines virtuelles dans les zones déséquilibrées. Dans les zones déséquilibrées, le groupe identique utilise la stratégie de scale-in spécifiée ci-dessus pour déterminer la machine virtuelle devant faire l’objet d’un scale-in. Dans ce cas, dans une zone déséquilibrée, le groupe identique choisira de supprimer la machine virtuelle la plus ancienne dans cette zone.

Pour un groupe de machines virtuelles identiques non zonal, la stratégie choisira de supprimer la machine virtuelle la plus ancienne dans le groupe identique.

Le même processus s’applique lors de l’utilisation de « NewestVM » dans la stratégie de scale-in ci-dessus.

## <a name="modifying-scale-in-policies"></a>Modification des stratégies de scale-in

La modification de la stratégie de scale-in suit le même processus que l’application de la stratégie de scale-in. Par exemple, si, dans l’exemple ci-dessus, vous souhaitez remplacer la stratégie « OldestVM » par « NewestVM », vous pouvez le faire des manières suivantes :

### <a name="azure-portal"></a>Portail Azure

Vous pouvez modifier la stratégie de scale-in d’un groupe identique via le portail Azure. 
 
1. Dans un groupe de machines virtuelles identiques existant, dans le menu de gauche, sélectionnez **Mise à l’échelle** .
1. Sélectionnez l’onglet **Stratégie de scale-in** .
1. Sélectionnez une stratégie de scale-in dans la liste déroulante.
1. Lorsque vous avez terminé, sélectionnez **Enregistrer** . 

### <a name="using-api"></a>Utilisation de l’API

Exécutez une instruction PUT sur le groupe de machines virtuelles identiques à l’aide de l’API 2019-03-01 :

```
PUT
https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<myRG>/providers/Microsoft.Compute/virtualMachineScaleSets/<myVMSS>?api-version=2019-03-01 

{ 
"location": "<VMSS location>", 
    "properties": { 
        "scaleInPolicy": {  
            "rules": ["NewestVM"]  
        } 
    }    
}
```
### <a name="azure-powershell"></a>Azure PowerShell

Mettez à jour la stratégie de scale-in d’un groupe identique existant :

```azurepowershell-interactive
Update-AzVmss `
 -ResourceGroupName "myResourceGroup" `
 -VMScaleSetName "myScaleSet" `
 -ScaleInPolicy “OldestVM”
```

### <a name="azure-cli-20"></a>Azure CLI 2.0

Voici un exemple de mise à jour de la stratégie de scale-in d’un groupe identique existant : 

```azurecli-interactive
az vmss update \  
  --resource-group <myResourceGroup> \
  --name <myVMScaleSet> \
  --scale-in-policy OldestVM
```

### <a name="using-template"></a>Utilisation d’un modèle

Dans votre modèle, sous « properties », modifiez le modèle comme indiqué ci-dessous et redéployez : 

```json
"scaleInPolicy": {  
      "rules": ["NewestVM"]  
} 
```

Le même processus s’applique si vous décidez de remplacer « NewestVM » par « Default » ou « OldestVM ».

## <a name="instance-protection-and-scale-in-policy"></a>Protection d’instance et stratégie de scale-in

Les groupes identiques de machines virtuelles fournissent deux types de [protection d’instance](./virtual-machine-scale-sets-instance-protection.md#types-of-instance-protection) :

1. Protection contre les opérations scale-in
2. Protection contre les actions de groupe identique

Une machine virtuelle protégée n’est pas supprimée par le biais d’une action de scale-in, quelle que soit la stratégie de scale-in appliquée. Par exemple, si VM_0 (machine virtuelle la plus ancienne dans le groupe identique) est protégée contre le scale-in et que le groupe identique a la stratégie de scale-in « OldestVM » activée, VM_0 n’est pas pris en compte pour le scale-in, même s’il s’agit de la machine virtuelle la plus ancienne dans le groupe identique. 

Une machine virtuelle protégée peut être supprimée manuellement par l’utilisateur à tout moment, quelle que soit la stratégie de scale-in activée sur le groupe identique. 

## <a name="usage-examples"></a>Exemples d'utilisation 

Les exemples ci-dessous illustrent comment un groupe de machines virtuelles identiques sélectionnera les machines virtuelles à supprimer lors du déclenchement d’un événement de scale-in. Les machines virtuelles avec les ID d’instances les plus élevés sont supposées être les machines virtuelles les plus récentes dans le groupe identique, et les machines virtuelles avec les ID d’instances les plus petits sont supposées être les machines virtuelles les plus anciennes dans le groupe identique. 

### <a name="oldestvm-scale-in-policy"></a>Stratégie de scale-in OldestVM

| Événement                 | ID d’instances dans Zone1  | ID d’instances dans Zone2  | ID d’instances dans Zone3  | Sélection de scale-in                                                                                                               |
|-----------------------|------------------------|------------------------|------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| Initial               | 3, 4, 5, 10            | 2, 6, 9, 11            | 1, 7, 8                |                                                                                                                                  |
| Scale-in              | 3, 4, 5, 10            | **_2_* _, 6, 9, 11      | 1, 7, 8                | Choisir entre Zone 1 et 2, bien que Zone 3 ait la machine virtuelle la plus ancienne. Supprimer VM2 de Zone 2, car il s’agit de la machine virtuelle la plus ancienne dans cette zone.   |
| Scale-in              | _*_3_*_ , 4, 5, 10      | 6, 9, 11               | 1, 7, 8                | Choisir Zone 1 bien que Zone 3 ait la machine virtuelle la plus ancienne. Supprimer VM3 de Zone 1, car il s’agit de la machine virtuelle la plus ancienne dans cette zone.                  |
| Scale-in              | 4, 5, 10               | 6, 9, 11               | _*_1_*_ , 7, 8          | Les zones sont équilibrées. Supprimer VM1 de Zone 3, car il s’agit de la machine virtuelle la plus ancienne dans le groupe identique.                                               |
| Scale-in              | _*_4_*_ , 5, 10         | 6, 9, 11               | 7, 8                   | Choisir entre Zone 1 et Zone 2. Supprimer VM4 de Zone 1, car il s’agit de la machine virtuelle la plus ancienne parmi les deux zones.                              |
| Scale-in              | 5, 10                  | _*_6_*_ , 9, 11         | 7, 8                   | Choisir Zone 2 bien que Zone 1 ait la machine virtuelle la plus ancienne. Supprimer VM6 de Zone 1, car il s’agit de la machine virtuelle la plus ancienne dans cette zone.                    |
| Scale-in              | _*_5_*_ , 10            | 9, 11                  | 7, 8                   | Les zones sont équilibrées. Supprimer VM5 de Zone 1, car il s’agit de la machine virtuelle la plus ancienne dans le groupe identique.                                                |

Pour un groupe de machines virtuelles identiques non zonal, la stratégie choisira de supprimer la machine virtuelle la plus ancienne dans le groupe identique. Toute machine virtuelle protégée est ignorée pour la suppression.

### <a name="newestvm-scale-in-policy"></a>Stratégie de scale-in NewestVM

| Événement                 | ID d’instances dans Zone1  | ID d’instances dans Zone2  | ID d’instances dans Zone3  | Sélection de scale-in                                                                                                               |
|-----------------------|------------------------|------------------------|------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| Initial               | 3, 4, 5, 10            | 2, 6, 9, 11            | 1, 7, 8                |                                                                                                                                  |
| Scale-in              | 3, 4, 5, 10            | 2, 6, 9, _*_11_*_      | 1, 7, 8                | Choisir entre Zone 1 et Zone 2. Supprimer VM11 de Zone 2, car il s’agit de la machine virtuelle la plus récente parmi les deux zones.                                |
| Scale-in              | 3, 4, 5, _*_10_*_      | 2, 6, 9                | 1, 7, 8                | Choisir Zone 1, car elle contient plus de machines virtuelles que les deux autres zones. Supprimer VM10 de Zone 1, car il s’agit de la machine virtuelle la plus récente dans cette zone.          |
| Scale-in              | 3, 4, 5                | 2, 6, _*_9_*_          | 1, 7, 8                | Les zones sont équilibrées. Supprimer VM9 de Zone 2, car il s’agit de la machine virtuelle la plus récente dans le groupe identique.                                                |
| Scale-in              | 3, 4, 5                | 2, 6                   | 1, 7, _*_8_*_          | Choisir entre Zone 1 et Zone 3. Supprimer VM8 de Zone 3, car il s’agit de la machine virtuelle la plus récente dans cette zone.                                      |
| Scale-in              | 3, 4, _*_5_*_          | 2, 6                   | 1, 7                   | Choisir Zone 1 bien que Zone 3 ait la machine virtuelle la plus récente. Supprimer VM5 de Zone 1, car il s’agit de la machine virtuelle la plus récente dans cette zone.                    |
| Scale-in              | 3, 4                   | 2, 6                   | 1, _ *_7_**             | Les zones sont équilibrées. Supprimer VM7 de Zone 3, car il s’agit de la machine virtuelle la plus récente dans le groupe identique.                                                |

Pour un groupe de machines virtuelles identiques non zonal, la stratégie choisira de supprimer la machine virtuelle la plus récente dans le groupe identique. Toute machine virtuelle protégée est ignorée pour la suppression. 

## <a name="troubleshoot"></a>Dépanner

1. Échec d’activation de scaleInPolicy Si vous obtenez une erreur « BadRequest » avec un message d’erreur indiquant que le membre « scaleInPolicy » est introuvable sur l’objet de type « properties », vérifiez la version de l’API utilisée pour le groupe de machines virtuelles identiques. La version d’API 2019-03-01 ou une version ultérieure est requise pour cette fonctionnalité.

2. Sélection incorrecte des machines virtuelles pour le scale-in : reportez-vous aux exemples ci-dessus. Si votre groupe de machines virtuelles identiques est un déploiement zonal, la stratégie de scale-in est appliquée en premier aux zones déséquilibrées, puis dans tout le groupe identique une fois l’équilibrage des zones effectué. Si l’ordre de scale-in n’est pas cohérent avec les exemples ci-dessus, soumettez une requête auprès de l’équipe du groupe de machines virtuelles identiques afin d’obtenir une assistance pour la résolution des problèmes.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [déployer votre application](virtual-machine-scale-sets-deploy-app.md) sur des groupes de machines virtuelles identiques.
