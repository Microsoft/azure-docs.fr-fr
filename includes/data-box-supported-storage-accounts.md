---
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: include
ms.date: 09/git14/2020
ms.author: alkohli
ms.openlocfilehash: 25c2ea04cd062554a975c63aae9b97846e646d68
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96025777"
---
Voici une liste des comptes de stockage et des types de stockage pris en charge pour l’appareil Data Box. Pour obtenir la liste complète des différents types de comptes de stockage et de leurs fonctionnalités, consultez [Types de comptes de stockage](../articles/storage/common/storage-account-overview.md#types-of-storage-accounts).

Pour les commandes d’importation, le tableau suivant indique les comptes de stockage pris en charge.

| **Compte de stockage / Type de stockage pris en charge** | **Objet blob de blocs** |**Objet blob de pages** _ |_ *Fichiers Azure** |**Remarques**|
| --- | --- | -- | -- | -- |
| Standard classique | O | O | O |
| Édition Standard v1 à usage général  | O | O | O | Les objets chauds et froids sont pris en charge.|
| Comptes de stockage à usage général v1 Premium  |  | O| | |
| Comptes de stockage à usage général v2 Standard  | O | O | O | Les objets chauds et froids sont pris en charge.|
| Comptes de stockage à usage général v2 Premium  |  |O | | |
| Stockage d'objets blob Standard |O | | |Les objets chauds et froids sont pris en charge. |

\* *Les données chargées dans des objets blob de pages doivent être alignées sur 512 octets (disques durs virtuels, par exemple).*

Pour les commandes d’exportation, le tableau suivant indique les comptes de stockage pris en charge.

| **Compte de stockage / Type de stockage pris en charge** | **Objet blob de blocs** |**Objet blob de pages** _ |_ *Fichiers Azure** |**Niveaux d’accès pris en charge**|
| --- | --- | -- | -- | -- |
| Standard classique | O | O | O | |
| Édition Standard v1 à usage général  | O | O | O | Chaud, froid|
| Comptes de stockage à usage général v1 Premium  |  | O| | |
| Comptes de stockage à usage général v2 Standard  | O | O | O | Chaud, froid|
| Comptes de stockage à usage général v2 Premium  |  |O | | |
| Stockage d'objets blob Standard |O | | |Chaud, froid |
| Stockage d’objets blob de blocs Premium |O | | |Chaud, froid |
| Stockage d’objets blob de pages Premium | |O | | |

> [!IMPORTANT]
> - Pour les comptes à usage général, Data Box ne prend pas en charge les types de stockage File d’attente, Table et Disque pour les commandes d’importation. Pour les commandes d’exportation, Data Box ne prend pas en charge les types de stockage File d’attente, Table, Disque et Azure Data Lake Gen 2 pour les comptes à usage général.
> - Data Box ne prend pas en charge l’ajout (append) d’objets blob pour les comptes de Stockage Blob et de stockage d’objets blob de blocs.
> - Data Box ne prend pas en charge les comptes de Stockage Fichier Premium.
> - Les données chargées dans des objets blob de pages doivent être alignées sur 512 octets (p. ex., disques durs virtuels).
> - Le volume maximal de données exportées est de 80 To.
> - L’historique des fichiers et les instantanés d’objets blob ne sont pas exportés.