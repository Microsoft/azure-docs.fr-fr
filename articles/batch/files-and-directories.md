---
title: Fichiers et répertoires dans Azure Batch
description: En savoir plus sur les fichiers et les répertoires et leur utilisation dans un workflow Azure Batch du point de vue du développeur.
ms.topic: conceptual
ms.date: 08/03/2020
ms.openlocfilehash: eafea6c234c3b261521f8a791b7a03e25388f02a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87552632"
---
# <a name="files-and-directories-in-azure-batch"></a>Fichiers et répertoires dans Azure Batch

Dans Azure Batch, chaque tâche possède un répertoire de travail sous lequel elle crée des fichiers et répertoires. Ce répertoire de travail peut être utilisé pour stocker le programme exécuté par la tâche, les données qu’il traite et la sortie du traitement qu’il effectue. Tous les fichiers et répertoires d’une tâche sont détenus par l’utilisateur de la tâche.

Le service Batch expose une partie du système de fichiers sur un nœud en tant que *répertoire racine*. Ce répertoire racine se trouve sur le lecteur de stockage temporaire de la machine virtuelle, pas directement sur le lecteur du système d’exploitation.

Les tâches peuvent accéder au répertoire racine en référençant la variable d’environnement `AZ_BATCH_NODE_ROOT_DIR` . Pour plus d’informations sur l’utilisation de variables d’environnement, consultez la section [Paramètres d’environnement des tâches](jobs-and-tasks.md#environment-settings-for-tasks).

## <a name="root-directory-structure"></a>Structure de répertoire racine

Le répertoire racine contient la structure de répertoires suivante :

![Capture d’écran de la structure de répertoires du nœud de calcul.](media\files-and-directories\node-folder-structure.png)

- **applications** : Contient des informations sur les détails des packages d’application installés sur le nœud de calcul. Les tâches peuvent accéder à ce répertoire en référençant la variable d’environnement `AZ_BATCH_APP_PACKAGE` .

- **fsmounts** : Le répertoire contient tous les systèmes de fichiers montés sur un nœud de calcul. Les tâches peuvent accéder à ce répertoire en référençant la variable d’environnement `AZ_BATCH_NODE_MOUNTS_DIR` . Pour plus d’informations, consultez [Monter un système de fichiers virtuel sur un pool Batch](virtual-file-mount.md).

- **shared** : ce répertoire fournit un accès en lecture/écriture à *toutes* les tâches qui s’exécutent sur un nœud. Les tâches qui s’exécutent sur le nœud peuvent créer, lire, mettre à jour et supprimer des fichiers dans ce répertoire. Les tâches peuvent accéder à ce répertoire en référençant la variable d’environnement `AZ_BATCH_NODE_SHARED_DIR` .

- **startup** : ce répertoire est utilisé comme répertoire de travail par une tâche de démarrage. Tous les fichiers téléchargés vers le nœud par la tâche de démarrage sont stockés ici. La tâche de démarrage peut créer, lire, mettre à jour et supprimer des fichiers dans ce répertoire. Les tâches peuvent accéder à ce répertoire en référençant la variable d’environnement `AZ_BATCH_NODE_STARTUP_DIR` .

- **volatile** : Ce répertoire n’est utilisé qu’à des fins internes. Il n’existe aucune garantie que les fichiers de ce répertoire ou que le répertoire lui-même existera à l’avenir.

- **workitems** : Ce répertoire contient les répertoires des travaux et de leurs tâches sur le nœud de calcul.

    Dans le répertoire **workitems**, un répertoire **Tasks** est créé pour chaque tâche qui s’exécute sur le nœud. Ce répertoire est accessible en référençant la variable d’environnement `AZ_BATCH_TASK_DIR`.

    Dans chaque répertoire **Tasks**, le service Batch crée un répertoire de travail (`wd`) dont le chemin unique est spécifié par la variable d’environnement `AZ_BATCH_TASK_WORKING_DIR`. Ce répertoire fournit un accès en lecture/écriture à la tâche. La tâche peut créer, lire, mettre à jour et supprimer des fichiers dans ce répertoire. Ce répertoire est conservé en fonction de la contrainte *RetentionTime* spécifiée pour la tâche.

    Les fichiers `stdout.txt` et `stderr.txt` sont écrits dans le dossier **Tasks** lors de l’exécution de la tâche.

> [!IMPORTANT]
> Lorsqu’un nœud est supprimé du pool, tous les fichiers stockés sur ce nœud sont supprimés.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur la [gestion et détection des erreurs](error-handling.md) dans Azure Batch.