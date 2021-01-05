---
title: Élément d’interface utilisateur SizeSelector
description: Décrit l’élément d’interface utilisateur Microsoft.Compute.SizeSelector pour le Portail Azure. Permet de sélectionner la taille d’une machine virtuelle.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: tomfitz
ms.openlocfilehash: d6408f8c08694ae681d302ae35f5778894091733
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87063624"
---
# <a name="microsoftcomputesizeselector-ui-element"></a>Élément d’interface utilisateur Microsoft.Compute.SizeSelector

Contrôle permettant de sélectionner une taille pour une ou plusieurs instances de machine virtuelle.

## <a name="ui-sample"></a>Exemple d’interface utilisateur

L’utilisateur voit un sélecteur comportant des valeurs par défaut issues de la définition d’élément.

![Microsoft.Compute.SizeSelector](./media/managed-application-elements/microsoft-compute-sizeselector.png)

Après avoir sélectionné le contrôle, l’utilisateur voit une vue développée des tailles disponibles.

![Microsoft.Compute.SizeSelector développé](./media/managed-application-elements/microsoft-compute-sizeselector-expanded.png)

## <a name="schema"></a>schéma

```json
{
  "name": "element1",
  "type": "Microsoft.Compute.SizeSelector",
  "label": "Size",
  "toolTip": "",
  "recommendedSizes": [
    "Standard_D1",
    "Standard_D2",
    "Standard_D3"
  ],
  "constraints": {
    "allowedSizes": [],
    "excludedSizes": [],
    "numAvailabilityZonesRequired": 3,
    "zone": "3"
  },
  "options": {
    "hideDiskTypeFilter": false
  },
  "osPlatform": "Windows",
  "imageReference": {
    "publisher": "MicrosoftWindowsServer",
    "offer": "WindowsServer",
    "sku": "2012-R2-Datacenter"
  },
  "count": 2,
  "visible": true
}
```

## <a name="sample-output"></a>Exemple de sortie

```json
"Standard_D1"
```

## <a name="remarks"></a>Notes

- `recommendedSizes` doit disposer au moins d’une taille. La première taille recommandée est utilisée comme valeur par défaut. La liste des tailles disponibles n’est pas triée par état recommandé. L’utilisateur peut sélectionner cette colonne pour trier par état recommandé.
- Si une taille recommandée n’est pas disponible à l’emplacement sélectionné, la taille est automatiquement ignorée. La taille recommandée suivante est alors utilisée.
- `constraints.allowedSizes` et `constraints.excludedSizes` sont tous deux facultatifs, mais ne peuvent pas être utilisés simultanément. La liste des tailles disponibles peut être déterminée en appelant [Répertorier les tailles de machines virtuelles disponibles pour un abonnement](/rest/api/compute/virtualmachines/virtualmachines-list-sizes-region). Toute taille non spécifiée dans `constraints.allowedSizes` est masquée et toute taille non spécifiée dans `constraints.excludedSizes` s’affiche.
- `osPlatform` doit être spécifié. Il peut s’agir de **Windows** ou de **Linux**. Il permet de déterminer le coût du matériel des machines virtuelles.
- `imageReference` est omis pour les images internes, mais est indiqué pour les images issues de tiers. Il permet de déterminer le coût des logiciels des machines virtuelles.
- `count` permet de définir le multiplicateur approprié pour l’élément. Il prend en charge une valeur statique, telle que **2**, ou une valeur dynamique issue d’un autre élément, comme `[steps('step1').vmCount]`. La valeur par défaut est **1**.
- `numAvailabilityZonesRequired` peut être 1, 2 ou 3.
- Par défaut, `hideDiskTypeFilter` est défini sur **false**. Le filtre du type de disque permet à l’utilisateur de voir tous les types de disques ou uniquement les disques SSD.

## <a name="next-steps"></a>Étapes suivantes

* Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).
* Pour obtenir une description des propriétés communes des éléments d’interface utilisateur, consultez la page [Éléments de CreateUiDefinition](create-uidefinition-elements.md).
