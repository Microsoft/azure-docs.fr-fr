---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 09/30/2020
ms.author: tamram
ms.openlocfilehash: fe493ca4a2edf16bfda405e8b2c5581c0f745616
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91665596"
---
| Ressource | Cible | Cible (préversion) |
|-|-|-|
| Taille maximale du conteneur d’objets blob unique | Identique à la capacité maximale du compte de stockage |  |
| Nombre maximal de blocs dans un objet blob de blocs ou ajouter des objets blob | 50 000 blocs |  |
| Taille maximale d’un bloc dans un objet blob de blocs | 100 Mio | 4 000 Mio (préversion) |
| Taille maximale d’un objet blob de blocs | 50 000 x 100 Mio (environ 4,75 Tio) | 50 000 x 4 000 Mio (environ 190,7 Tio) (préversion) |
| Taille maximale d’un bloc dans un objet blob d’ajout | 4 Mio |  |
| Taille maximale d’un objet blob d’ajout | 50 000 x 4 Mio (environ 195 Gio) |  |
| Taille maximale d’un objet blob de pages | 8 Tio<sup>2</sup> |  |
| Nombre maximal de stratégies d’accès stockées par conteneur d’objets blob | 5 |  |
| Taux de requêtes cibles pour un seul objet blob | Jusqu’à 500 requêtes par seconde |  |
| Débit cible pour un blob de page unique | Jusqu’à 60 Mio par seconde<sup>2</sup> |  |
| Débit cible pour un objet blob de blocs unique | Jusqu’à la limite d’entrée/sortie du compte de stockage<sup>1</sup> |  |

<sup>1</sup> Le débit pour un seul objet blob dépend de plusieurs facteurs, dont et sans s’y limiter : concurrence, taille de la demande, niveau de performance, vitesse des chargements à la source et des téléchargements à la destination. Pour tirer parti des améliorations des performances des [objets blob de blocs à haut débit](https://azure.microsoft.com/blog/high-throughput-with-azure-blob-storage/), chargez des objets blob ou des blocs plus volumineux. En particulier, appelez l’opération [Put Blob](/rest/api/storageservices/put-blob) ou [Put Block](/rest/api/storageservices/put-block) avec une taille d’objet blob ou de bloc supérieure à 4 Mio pour les comptes de stockage standard. Pour les objets blob de blocs Premium ou pour les comptes de stockage Data Lake Storage Gen2, utilisez une taille de bloc ou d’objet blob supérieure à 256 Kio.

<sup>2</sup> Les objet blob de pages ne sont pas encore pris en charge dans les comptes qui ont le paramètre **Espace de noms hiérarchique** activé.

Le tableau suivant décrit les tailles maximales de blocs et d’objets blob autorisées par la version du service.

| Version du service | Taille de bloc maximale (via Put Block) | Taille de blob maximale (via Put Block List) | Taille de blob maximale via une seule opération d’écriture (via Put Blob) |
|-|-|-|-|
| Version 2019-12-12 et ultérieure | 4 000 Mio (préversion) | Environ 190,7 Tio (4 000 Mio X 50 000 blocs) (préversion) | 5 000 Mio (préversion) |
| De la version 2016-05-31 à la version 2019-07-07 | 100 Mio | Environ 4,75 Tio (100 Mio X 50 000 blocs) | 256 Mio |
| Versions antérieures à 2016-05-31 | 4 Mio | Environ 195 Gio (4 Mio X 50 000 blocs) | 64 Mio |
