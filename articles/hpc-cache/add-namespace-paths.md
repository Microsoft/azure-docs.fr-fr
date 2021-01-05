---
title: Configurer l’espace de noms agrégé d’Azure HPC Cache
description: Création de chemins d’accès côté client pour le stockage principal avec Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 09/30/2020
ms.author: v-erkel
ms.openlocfilehash: e525fc0705dffcd4765e6a1f6c5235bdef260fcd
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96339674"
---
# <a name="set-up-the-aggregated-namespace"></a>Configurer l’espace de noms agrégé

Après avoir créé des cibles de stockage, vous devez également créer des chemins d’espace de noms associés. Les ordinateurs clients utilisent ces chemins d’accès virtuels pour accéder aux fichiers via le cache au lieu de se connecter directement au stockage principal. Ce système permet aux administrateurs de cache de modifier les systèmes de stockage principal sans avoir à réécrire les instructions du client.

Pour en savoir plus sur cette fonctionnalité, consultez [Planifier l’espace de noms agrégé](hpc-cache-namespace.md).

La page **Espace de noms** du portail Azure présente les chemins d’accès que les clients utilisent pour accéder à vos données via le cache. Utilisez cette page pour créer, supprimer ou modifier les chemins d’espace de noms. Vous pouvez également configurer des chemins d’espace de noms à l’aide d’Azure CLI.

Tous les chemins d’accès côté client existants sont répertoriés dans la page **Espace de noms**. Si une cible de stockage n’a pas de chemin d’accès, elle n’apparaît pas dans le tableau.

Vous pouvez trier les colonnes du tableau en cliquant sur les flèches afin de mieux comprendre l’espace de noms agrégé de votre cache.

![Capture d’écran de la page Espace de noms du portail avec un tableau contenant deux chemins d’accès. En-têtes de colonne : Chemin d’espace de noms, Cible de stockage, Chemin d’exportation et Sous-répertoire d’exportation. Les éléments de la première colonne sont des liens sur lesquels vous pouvez cliquer. Boutons principaux : Ajouter un chemin d’espace de noms, Actualiser, Supprimer](media/namespace-page.png)

## <a name="add-or-edit-client-facing-namespace-paths"></a>Ajouter ou modifier des chemins d’espace de noms côté client

Vous devez créer au moins un chemin d’espace de noms pour que les clients puissent accéder à la cible de stockage. (Pour en savoir plus sur l’accès client, consultez [Monter le cache Azure HPC Cache](hpc-cache-mount.md).)

### <a name="blob-namespace-paths"></a>Chemins d’espace de noms d’objet blob

Une cible Stockage blob Azure ne peut avoir qu’un seul chemin d’espace de noms.

Suivez les instructions ci-dessous pour configurer ou modifier le chemin avec le portail Azure ou Azure CLI.

### <a name="portal"></a>[Portail](#tab/azure-portal)

Dans le portail Azure, chargez la page de paramètres **Espace de noms**. Vous pouvez ajouter, modifier ou supprimer des chemins d’espace de noms à partir de cette page.

* **Ajouter un nouveau chemin d’accès :** Cliquez sur le bouton **+ Ajouter** situé en haut et complétez les informations dans le panneau de modification.

  * Sélectionnez la cible de stockage dans la liste déroulante. (Dans cette capture d’écran, la cible de stockage blob ne peut pas être sélectionnée, car elle possède déjà un chemin d’espace de noms.)

    ![Capture d’écran des champs de modification du nouvel espace de noms avec le sélecteur de cible de stockage mis en évidence](media/namespace-select-storage-target.png)

  * Pour une cible Stockage blob Azure, les chemins d’exportation et de sous-répertoire sont automatiquement définis sur ``/``.

* **Modifier un chemin d’accès existant :** Cliquez sur le chemin d’espace de noms. Le panneau de modification apparaît et vous pouvez modifier le chemin.

  ![Capture d’écran de la page Espace de noms après sélection d’un chemin d’espace de noms Blob. Les champs de modification s’affichent dans un volet à droite.](media/edit-namespace-blob.png)

* **Suppression d’un chemin d’espace de noms :** Activez la case à cocher à gauche du chemin et cliquez sur le bouton **Supprimer**.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[Configurez Azure CLI pour Azure HPC Cache](./az-cli-prerequisites.md).

Lorsque vous utilisez Azure CLI, vous devez ajouter un chemin d’espace de noms lors de la création de la cible de stockage. Pour en savoir plus, consultez [Ajouter une cible Stockage blob Azure](hpc-cache-add-storage.md?tabs=azure-cli#add-a-new-azure-blob-storage-target).

Pour mettre à jour le chemin d’espace de noms d’une cible, utilisez la commande [az hpc-cache blob-storage-target update](/cli/azure/ext/hpc-cache/hpc-cache/blob-storage-target#ext-hpc-cache-az-hpc-cache-blob-storage-target-update). Les arguments de la commande de mise à jour sont similaires à ceux de la commande de création, sauf que vous ne transmettez pas le nom du conteneur ou le compte de stockage.

Vous ne pouvez pas supprimer le chemin d’espace de noms d’une cible Stockage blob avec Azure CLI, mais vous pouvez le remplacer par une autre valeur.

---

### <a name="nfs-namespace-paths"></a>Chemin d'espaces de noms NFS

Une cible de stockage NFS peut avoir plusieurs chemins d’accès virtuels, à condition que chaque chemin d’accès représente une exportation ou un sous-répertoire différent sur le même système de stockage.

Lors de la planification de votre espace de noms pour une cible de stockage NFS, souvenez-vous que chaque chemin doit être unique et ne peut pas être un sous-répertoire d’un autre chemin d’espace de noms. Par exemple, si vous avez un chemin d’espace de noms appelé ``/parent-a``, vous ne pouvez pas créer de chemins d’espace de noms tels que ``/parent-a/user1`` et ``/parent-a/user2``. Ces chemins d’accès de répertoire sont déjà accessibles dans l’espace de noms en tant que sous-répertoires de ``/parent-a``.

Tous les chemins d’espace de noms pour un système de stockage NFS sont créés sur une seule cible de stockage. La plupart des configurations de cache peuvent prendre en charge jusqu’à dix chemins d’espace de noms par cible de stockage, mais les configurations plus volumineuses peuvent en accepter jusqu’à 20.

Cette liste montre le nombre maximum de chemins d’espace de noms par configuration.

* Débit jusqu’à 2 Go/s :

  * Cache de 3 To - 10 chemins d’espace de noms
  * Cache de 6 To - 10 chemins d’espace de noms
  * Cache de 23 To - 20 chemins d’espace de noms

* Débit jusqu’à 4 Go/s :

  * Cache de 6 To - 10 chemins d’espace de noms
  * Cache de 12 To - 10 chemins d’espace de noms
  * Cache de 24 To - 20 chemins d’espace de noms

* Débit jusqu’à 8 Go/s :

  * Cache de 12 To - 10 chemins d’espace de noms
  * Cache de 24 To - 10 chemins d’espace de noms
  * Cache de 48 To - 20 chemins d’espace de noms

Pour chaque chemin d’espace de noms NFS, indiquez le chemin côté client, le chemin d’exportation du système de stockage et éventuellement un sous-répertoire d’exportation.

### <a name="portal"></a>[Portail](#tab/azure-portal)

Dans le portail Azure, chargez la page de paramètres **Espace de noms**. Vous pouvez ajouter, modifier ou supprimer des chemins d’espace de noms à partir de cette page.

* **Pour ajouter un nouveau chemin d’accès :** Cliquez sur le bouton **+ Ajouter** situé en haut et complétez les informations dans le panneau de modification.
* **Pour modifier un chemin d’accès existant :** Cliquez sur le chemin d’espace de noms. Le panneau de modification apparaît et vous pouvez modifier le chemin.
* **Pour supprimer un chemin d’espace de noms :** Activez la case à cocher à gauche du chemin et cliquez sur le bouton **Supprimer**.

Renseignez ces valeurs pour chaque chemin d’espace de noms :

* **Chemin d’espace de noms** - Chemin d’accès du fichier côté client.

* **Cible de stockage** - Si vous créez un nouveau chemin d’espace de noms, sélectionnez une cible de stockage dans le menu déroulant.

* **Chemin d’exportation** - Entrez le chemin de l’exportation NFS. Veillez à ne pas faire d’erreur dans le nom du chemin d’exportation. Le portail valide la syntaxe pour ce champ mais ne vérifie pas le chemin d’exportation tant que vous n’avez pas validé la modification.

* **Sous-répertoire d’exportation** - Si vous souhaitez que ce chemin monte un sous-répertoire spécifique de l’exportation, entrez-le ici. Sinon, laissez ce champ vide.

![Capture d’écran de la page Espace de noms du portail avec la page de mise à jour ouverte à droite](media/update-namespace-nfs.png)

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[Configurez Azure CLI pour Azure HPC Cache](./az-cli-prerequisites.md).

Lorsque vous utilisez Azure CLI, vous devez ajouter au moins un chemin d’espace de noms lors de la création de la cible de stockage. Pour en savoir plus, consultez [Ajouter une cible de stockage NFS](hpc-cache-add-storage.md?tabs=azure-cli#add-a-new-nfs-storage-target).

Pour mettre à jour le chemin d’espace de noms d’une cible ou ajouter des chemins supplémentaires, utilisez la commande [az hpc-cache nfs-storage-target update](/cli/azure/ext/hpc-cache/hpc-cache/nfs-storage-target#ext-hpc-cache-az-hpc-cache-nfs-storage-target-update). Utilisez l’option ``--junction`` pour spécifier tous les chemins d’espace de noms souhaités.

Les options utilisées pour la commande de mise à jour sont similaires à celles de la commande « create », sauf que vous ne transmettez pas les informations du système de stockage (adresse IP ou nom d’hôte) et le modèle d’utilisation est facultatif. Pour en savoir plus sur la syntaxe de l’option ``--junction``, consultez [Ajouter une nouvelle cible de stockage NFS](hpc-cache-add-storage.md?tabs=azure-cli#add-a-new-nfs-storage-target).

---

## <a name="next-steps"></a>Étapes suivantes

Après avoir créé l’espace de noms agrégé pour vos cibles de stockage, vous pouvez monter des clients sur le cache. Pour en savoir plus, consultez les articles suivants.

* [Monter le cache Azure HPC Cache](hpc-cache-mount.md)
* [Déplacer les données dans le stockage d’objets blob Azure](hpc-cache-ingest.md)
