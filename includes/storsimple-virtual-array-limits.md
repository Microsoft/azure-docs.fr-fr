---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 658fd9178495f14274c85eab2129c9dcd3be7693
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177094"
---
| **Identificateur de la limite** | **Limite** | **Commentaires** |
| --- | --- | --- |
| Capacité totale (y compris le cloud) |Jusqu'à 64 To par appareil virtuel |Vous pouvez basculer une baie virtuelle StorSimple Virtual Array vers une autre baie vide. Si vous essayez de restaurer sur le même appareil, assurez-vous de disposer de suffisamment d'espace sur l’appareil pour terminer cette opération. Après avoir dépassé 32 To, vous ne pouvez pas restaurer sur le même appareil. |
| Nombre maximal d'informations d'identification du compte de stockage par appareil |1 | |
| Nombre maximal de volumes/partages |16 | |
| Taille minimale d’un partage hiérarchisé |500 Go | |
| Taille minimale d’un volume hiérarchisé |500 Go | |
| Taille maximale d’un partage hiérarchisé |20 To | |
| Taille maximale d’un volume hiérarchisé |5 To | |
| Taille minimale d'un partage épinglé localement |50 Go | |
| Taille minimale d'un volume épinglé localement |50 Go | |
| Taille maximale d'un partage épinglé localement |2 To | |
| Taille maximale d'un volume épinglé localement |200 Go | |
| Nombre maximal de connexions iSCSI à partir d'initiateurs |512 | |
| Nombre maximal d'enregistrements de contrôle d'accès par appareil |64 | |
| Nombre maximal de sauvegardes conservées par l'appareil virtuel dans le dossier *.backups* pour le serveur de fichiers |5 |Cela inclut les sauvegardes les plus récentes planifiées (générées par la stratégie de sauvegarde par défaut) et les sauvegardes manuelles. |
| Nombre maximal de sauvegardes planifiées conservées par l'appareil |55 |30 sauvegardes quotidiennes<br>12 sauvegardes mensuelles<br>13 sauvegardes annuelles |
| Nombre maximal de sauvegardes manuelles conservées par l'appareil |45 | |
| Nombre maximal de fichiers par partage pour un serveur de fichiers |1 million |Lorsque vous effectuez une restauration d’appareil, les délais de restauration sont proportionnels au nombre de fichiers se trouvant sur tous les partages de l’appareil. |
| Nombre maximal de fichiers par volume pour un serveur iSCSI |1 million | |
| Nombre maximal de fichiers par baie virtuelle |4 millions | |
| Délai de récupération après restauration |Restauration rapide |La restauration est basée sur la carte thermique et dépend de la taille du volume.<br>Les opérations de sauvegarde peuvent se produire lorsqu’une opération de restauration est en cours. |

