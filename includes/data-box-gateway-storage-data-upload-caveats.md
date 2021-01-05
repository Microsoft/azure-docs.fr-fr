---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/15/2020
ms.author: alkohli
ms.openlocfilehash: 385ebffb4780543b532c81b89f1847f5c84cd0ac
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580903"
---
Les mises en garde suivantes s’appliquent pour les déplacements des données dans Azure.

- Nous vous suggérons de ne pas autoriser plusieurs appareils à écrire dans le même conteneur.
- Si un objet Azure existant (tel qu’un blob ou un fichier) présent dans le cloud a le même nom que l’objet qui est en train d’être copié, l’appareil remplace le fichier dans le cloud.
- Une hiérarchie de répertoires vides (sans fichiers) créée sous les dossiers de partage n’est pas chargée dans les conteneurs d’objets blob.
- Vous pouvez copier les données à l’aide de la fonction glisser-déposer de l’Explorateur de fichiers, ou via la ligne de commande. Si la taille globale des fichiers copiés est supérieure à 10 Go, nous vous recommandons d’utiliser un programme de copie en bloc tel que `Robocopy` ou `rsync`. Les outils de copie en bloc tentent à nouveau l’opération de copie en cas d’erreurs intermittentes et offrent une résilience supplémentaire.
- Si le partage associé au conteneur de stockage Azure télécharge des objets BLOB qui ne correspondent pas au type d’objets BLOB définis pour le partage au moment de la création, alors de tels objets BLOB ne sont pas mis à jour. Par exemple, si vous créez un partage d’objets blob de blocs sur l’appareil, associez le partage à un conteneur cloud existant qui possède des objets blob de pages, actualisez ce partage pour télécharger les fichiers, puis modifiez certains des fichiers actualisés déjà stockés en tant qu’objets blob de pages dans le cloud, vous verrez des échecs de téléchargement.
- Une fois qu’un fichier est créé dans les partages, vous ne pouvez plus le renommer.
- La suppression d’un fichier à partir d’un partage ne supprime pas l’entrée du compte de stockage.
- Si vous utilisez `rsync` pour copier des données, l’option `rsync -a` n’est pas prise en charge.