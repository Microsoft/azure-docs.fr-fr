---
title: Gérer une stratégie d'enregistrement - Azure
description: Cette rubrique explique comment gérer une stratégie d'enregistrement.
ms.topic: how-to
ms.date: 04/27/2020
ms.openlocfilehash: d3a1be915dc1cc8714e49cc7b2fe68bbe9cad161
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87011479"
---
# <a name="manage-recording-policy"></a>Gérer une stratégie d’enregistrement

Vous pouvez utiliser Live Video Analytics sur IoT Edge pour l'[enregistrement de vidéo continu](continuous-video-recording-concept.md), qui permet d'enregistrer des vidéos dans le cloud pendant des semaines ou des mois. Vous pouvez gérer la durée (en jours) de cet archivage dans le cloud à l'aide des [outils de gestion du cycle de vie](../../storage/blobs/storage-lifecycle-management-concepts.md?tabs=azure-portal) intégrés au service Stockage Azure.  

Votre compte Media Service est lié à un compte Stockage Azure, et lorsque vous enregistrez une vidéo dans le cloud, le contenu est inscrit sur une ressource [Media Service](../latest/assets-concept.md). Chaque ressource est mappée à un conteneur du compte de stockage. La gestion du cycle de vie vous permet de définir une [stratégie](../../storage/blobs/storage-lifecycle-management-concepts.md?tabs=azure-portal#policy) applicable à un compte Stockage, dans laquelle vous pouvez spécifier une [règle](../../storage/blobs/storage-lifecycle-management-concepts.md?tabs=azure-portal#rules) telle que la suivante.

```
{
  "rules": [
    {
      "name": "NinetyDayRule",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": { "daysAfterModificationGreaterThan": 30 },
            "delete": { "daysAfterModificationGreaterThan": 90 }
          }
        }
      }
    }
  ]
}
```

La règle ci-dessus :

* s'applique à tous les objets blob de blocs du compte Stockage ;
* spécifie qu'au bout de 30 jours, les objets blob sont déplacés du [niveau d'accès chaud au niveau d'accès froid](../../storage/blobs/storage-blob-storage-tiers.md?tabs=azure-portal).
* En outre, lorsque les objets blob ont plus de 90 jours, ils doivent être supprimés.

Comme Live Video Analytics archive votre vidéo dans des unités de temps spécifiques, votre ressource contient une série d'objets blob, un objet blob par segment. Lorsque la stratégie de gestion du cycle de vie s'exécutera et supprimera les objets blob les plus anciens, vous pourrez continuer à accéder aux objets blob restants et à les lire via les API Media Service. Pour plus d'informations, consultez [Lecture des enregistrements](playback-recordings-how-to.md). 

## <a name="limitations"></a>Limites

Voici quelques limites connues de la gestion du cycle de vie :

* La stratégie peut contenir un maximum de 100 règles, et chaque règle peut spécifier 10 conteneurs. Ainsi, si vous avez besoin de différentes stratégies d'enregistrement (par exemple, 3 jours d'archivage pour la caméra située face au parking, 30 jours pour la caméra située sur le quai de chargement, et 180 jours pour la caméra située derrière la caisse), un seul compte Media Service vous permet de personnaliser les règles de 1 000 caméras.
* Les mises à jour de la stratégie de gestion du cycle de vie ne sont pas immédiates. Pour plus d'informations, consultez [cette section FAQ](../../storage/blobs/storage-lifecycle-management-concepts.md?tabs=azure-portal#faq).
* Si vous choisissez d'appliquer une stratégie qui déplace les objets blob vers le niveau froid, la lecture de cette partie de l'archive peut être affectée. Vous pouvez constater des latences supplémentaires, ou des erreurs sporadiques. Media Services ne prend pas en charge la lecture du contenu au niveau archive.

## <a name="next-steps"></a>Étapes suivantes

[Lecture des enregistrements](playback-recordings-how-to.md)
