---
title: Événement de fin de suppression de pool Azure Batch
description: Référence pour l’événement de fin de suppression de pool Batch. Cet événement est émis quand une opération de suppression de pool est terminée.
ms.topic: reference
ms.date: 12/28/2020
ms.openlocfilehash: be6411a150ae6be424c0621eed768157154c7408
ms.sourcegitcommit: 7e97ae405c1c6c8ac63850e1b88cf9c9c82372da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/29/2020
ms.locfileid: "97803729"
---
# <a name="pool-delete-complete-event"></a>Événement de fin de suppression de pool

 Cet événement est émis quand une opération de suppression de pool est terminée.

 L’exemple suivant montre le corps d’un événement de fin de suppression de pool.

```
{
   "id": "myPool1",
   "startTime": "2016-09-09T22:13:48.579Z",
   "endTime": "2016-09-09T22:14:08.836Z"
}
```

|Élément|Type|Notes|
|-------------|----------|-----------|
|`id`|String|ID du pool.|
|`startTime`|DateTime|Heure de début de la suppression du pool.|
|`endTime`|DateTime|Heure de fin de la suppression du pool.|

## <a name="remarks"></a>Notes

Pour plus d’informations sur les états et les codes d’erreur pour l’opération de redimensionnement de pool, voir [Supprimer un pool d’un compte](/rest/api/batchservice/delete-a-pool-from-an-account).
