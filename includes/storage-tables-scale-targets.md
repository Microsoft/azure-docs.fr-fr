---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 03/09/2020
ms.author: tamram
ms.openlocfilehash: 27617da97ced9ac775beae71e4c25202944b2aba
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "78940980"
---
Le tableau suivant décrit les objectifs de capacité, de scalabilité et de performances pour le stockage Table.

| Ressource | Cible |
|----------|---------------|
| Nombre de tables dans un compte de stockage Azure | Limité uniquement par la capacité du compte de stockage |
| Nombre de partitions dans une table | Limité uniquement par la capacité du compte de stockage |
| Nombre d'entités dans une partition | Limité uniquement par la capacité du compte de stockage |
| Taille maximale d’une seule table | 500 Tio |
| Taille maximale d’une entité unique, y compris toutes les valeurs de propriété | 1 Mio |
| Nombre maximal de propriétés dans une entité de table | 255 (notamment les trois propriétés système : **PartitionKey**, **RowKey** et **Timestamp**) |
| Taille totale maximale d’une propriété individuelle dans une entité | Varie en fonction du type de propriété. Pour plus d'informations, consultez **Types de propriétés** dans [Présentation du modèle de données du service de Table](/rest/api/storageservices/understanding-the-table-service-data-model). |
| Taille de la **PartitionKey** | Chaîne jusqu’à 1 Kio |
| Taille de la **RowKey** | Chaîne jusqu’à 1 Kio |
| Taille d’une transaction ETG | Une transaction peut inclure au plus 100 entités, et la charge utile doit être inférieure à 4 Mio. Une transaction de groupe d’entités peut inclure une mise à jour d’une entité une seule fois. |
| Nombre maximal de stratégies d’accès stockées par table | 5 |
| Taux de requête maximal par compte de stockage | 20 000 transactions par seconde, en supposant que la taille d’entité est de 1 Kio |
| Débit cible pour une partition de table unique (entités de 1 Kio) | Jusqu'à 2 000 entités par seconde |