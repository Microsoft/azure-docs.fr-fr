---
title: Appels PUT pour les opérations de création ou de mise à jour
description: Appels PUT pour les opérations de création ou de mise à jour sur des ressources de calcul
author: mimckitt
ms.author: mimckitt
ms.reviewer: cynthn
ms.topic: conceptual
ms.service: virtual-machines
ms.date: 08/4/2020
ms.custom: avverma
ms.openlocfilehash: d6ee4179dce905d637e933743ade7452a2484077
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91978558"
---
# <a name="put-calls-for-creation-or-updates-on-compute-resources"></a>Appels PUT pour la création ou les mises à jour de ressources de calcul

Les ressources `Microsoft.Compute` ne prennent pas en charge la définition conventionnelle de la sémantique *HTTP PUT*. Au lieu de cela, ces ressources utilisent la sémantique PATCH pour les verbes PUT et PATCH.

Les opérations **de création** appliquent les valeurs par défaut si nécessaire. Toutefois, les **mises à jour** de ressources effectuées via PUT ou PATCH n’appliquent aucune propriété par défaut. Les opérations **de mise à jour** appliquent la stricte sémantique PATCH.

Par exemple, la propriété de disque `caching` d’une machine virtuelle est définie par défaut sur `ReadWrite` si la ressource est un disque de système d’exploitation.

```json
    "storageProfile": {
      "osDisk": {
        "name": "myVMosdisk",
        "image": {
          "uri": "http://{existing-storage-account-name}.blob.core.windows.net/{existing-container-name}/{existing-generalized-os-image-blob-name}.vhd"
        },
        "osType": "Windows",
        "createOption": "FromImage",
        "caching": "ReadWrite",
        "vhd": {
          "uri": "http://{existing-storage-account-name}.blob.core.windows.net/{existing-container-name}/myDisk.vhd"
        }
      }
    },
```

Toutefois, pour les opérations **de mise à jour** lorsqu’une propriété est omise ou qu’une valeur *Null* est transmise, elle reste inchangée et il n’y a pas de valeurs par défaut.

Cela est important lors de l’envoi d’opérations de mise à jour à une ressource dans le but de supprimer une association. Si cette ressource est une ressource `Microsoft.Compute`, la propriété correspondante que vous souhaitez supprimer doit être explicitement appelée et une valeur doit être attribuée. Pour ce faire, les utilisateurs peuvent transmettre une chaîne vide, par exemple **" "** . Cela permet d’indiquer à la plateforme de supprimer cette association.

> [!IMPORTANT]
> Il n’existe aucune prise en charge de la « mise à jour corrective » d’un élément de tableau. Au lieu de cela, le client doit effectuer une requête PUT ou PATCH avec la totalité du contenu du tableau mis à jour. Par exemple, pour détacher un disque de données d’une machine virtuelle, effectuez une requête GET pour obtenir le modèle de machine virtuelle actuel, supprimez le disque à détacher de `properties.storageProfile.dataDisks` et effectuez une requête PUT avec l’entité de machine virtuelle mise à jour.

## <a name="examples"></a>Exemples

### <a name="correct-payload-to-remove-a-proximity-placement-groups-association"></a>Charge utile correcte pour supprimer une association de groupes de placement de proximité

`
{ "location": "westus", "properties": { "platformFaultDomainCount": 2, "platformUpdateDomainCount": 20, "proximityPlacementGroup": "" } }
`

### <a name="incorrect-payloads-to-remove-a-proximity-placement-groups-association"></a>Charges utiles incorrectes pour supprimer une association de groupes de placement de proximité

`
{ "location": "westus", "properties": { "platformFaultDomainCount": 2, "platformUpdateDomainCount": 20, "proximityPlacementGroup": null } }
`

`
{ "location": "westus", "properties": { "platformFaultDomainCount": 2, "platformUpdateDomainCount": 20 } }
`

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur les appels de création ou de mise à jour pour les [machines virtuelles](/rest/api/compute/virtualmachines/createorupdate) et les [groupes de machines virtuelles identiques](/rest/api/compute/virtualmachinescalesets/createorupdate).