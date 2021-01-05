---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 09/25/2020
ms.author: alkohli
ms.openlocfilehash: 6dc201af2271909de15af9bac1a2e2bb68faed1a
ms.sourcegitcommit: 4313e0d13714559d67d51770b2b9b92e4b0cc629
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/27/2020
ms.locfileid: "91401049"
---
Les mises en garde suivantes s’appliquent pour les déplacements des données dans Azure.

- Nous vous suggérons de ne pas autoriser plusieurs appareils à écrire dans le même conteneur.
- Si un objet Azure existant (tel qu’un objet blob ou un fichier) présent dans le cloud a le même nom que l’objet qui est en train d’être copié, l’appareil remplace le fichier dans le cloud.
- Une hiérarchie de répertoires vides (sans fichiers) créée sous les dossiers de partage n’est pas chargée dans les conteneurs d’objets blob.
- Vous pouvez copier les données à l’aide de la fonction glisser-déposer de l’Explorateur de fichiers, ou via la ligne de commande. Si la taille globale des fichiers copiés est supérieure à 10 Go, nous vous recommandons d’utiliser un programme de copie en bloc tel que Robocopy ou rsync. Les outils de copie en bloc tentent à nouveau l’opération de copie en cas d’erreurs intermittentes et offrent une résilience supplémentaire.
- Si le partage associé au conteneur de stockage Azure télécharge des objets BLOB qui ne correspondent pas au type d’objets BLOB définis pour le partage au moment de la création, alors de tels objets BLOB ne sont pas mis à jour. Par exemple, vous pouvez créer un partage d’objets blob de blocs sur l’appareil. Associez le partage avec un conteneur cloud existant présentant des objets blob de pages. Actualisez ce partage pour télécharger les fichiers. Modifiez quelques-uns des fichiers actualisés qui sont déjà stockés en tant qu’objets blob de pages dans le cloud. Vous verrez des échecs de chargement.
- Une fois qu’un fichier est créé dans les partages, vous ne pouvez plus le renommer.
- La suppression d’un fichier à partir d’un partage ne supprime pas l’entrée du compte de stockage.
- Si vous utilisez rsync pour copier des données, l’option `rsync -a` n’est pas prise en charge.

