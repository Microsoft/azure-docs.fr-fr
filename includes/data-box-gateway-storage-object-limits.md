---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/15/2020
ms.author: alkohli
ms.openlocfilehash: 20d30935fe03bc002ab63f2f8ca1ce890ef9e3b4
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580898"
---
Voici les tailles des objets Azure qui peuvent être écrits. Assurez-vous que tous les fichiers chargés sont en conformité avec ces limites.

| Type d’objet Azure | Limite de chargement                                             |
|-------------------|-----------------------------------------------------------|
| Objet blob de blocs        | ~ 4,75 To                                                 |
| Objet blob de pages         | 1 To <br> Chaque fichier chargé au format d’objet blob de pages doit être de 512 octets alignés (un multiple entier), sinon le chargement échoue. <br> Les disques VHD et VHDX sont de 512 octets alignés. |
| Azure Files         | 1 To <br> Chaque fichier chargé au format d’objet blob de pages doit être de 512 octets alignés (un multiple entier), sinon le chargement échoue. <br> Les disques VHD et VHDX sont de 512 octets alignés. |

> [!IMPORTANT]
> La création de fichiers (quel que soit le type de stockage) est autorisée jusqu’à 5 To. Toutefois, si vous créez un fichier dont la taille est supérieure à la limite de chargement définie dans le tableau précédent, le fichier n’est pas chargé. Vous devez supprimer manuellement le fichier pour récupérer l’espace.