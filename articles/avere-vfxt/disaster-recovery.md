---
title: Guide pour la reprise d’activité pour Avere vFXT pour Azure
description: Guide pratique pour protéger les données dans Avere vFXT pour Azure contre les suppressions accidentelles ou les pannes
author: ekpgh
ms.service: avere-vfxt
ms.topic: conceptual
ms.date: 12/10/2019
ms.author: rohogue
ms.openlocfilehash: 5cc4678b082aa5a4a3f90518ff8fac448f414f1a
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92342244"
---
# <a name="disaster-recovery-guidance-for-avere-vfxt-for-azure"></a>Guide pour la reprise d’activité pour Avere vFXT pour Azure

Cet article décrit les stratégies de protection de votre workflow Avere vFXT pour Azure, et fournit de l’aide pour la sauvegarde des données pour vous permettre de récupérer après des accidents ou des pannes.

Avere vFXT pour Azure stocke temporairement les données dans son cache. Les données sont stockées à long terme dans des systèmes de stockage back-end : des systèmes matériels locaux, des conteneurs de stockage d’objets blob Azure, ou les deux.

Pour vous protéger contre les pannes et les possibles pertes de données, considérez les 4 aspects suivants :

* Protection contre les temps d’arrêt si le système Avere vFXT pour Azure devient indisponible
* Protection des données dans le cache du cluster
* Protection des données dans le stockage matériel du NAS back-end
* Protection des données dans le stockage cloud d’objets blob Azure

Chaque client Avere vFXT pour Azure doit créer son propre plan de reprise d’activité complet qui comprend des plans pour ces éléments. Vous pouvez également mettre en place de la résilience dans les applications que vous utilisez avec le cluster vFXT. Pour obtenir de l’aide, suivez les liens figurant dans [Étapes suivantes](#next-steps).

## <a name="protect-against-downtime"></a>Protéger contre les temps d’arrêt

La redondance est intégrée au produit Avere vFXT pour Azure :

* Le cluster a une haute disponibilité et les nœuds de cluster individuels peuvent basculer avec une interruption minimale.
* Les données modifiées dans le cache sont écrites régulièrement dans les serveurs de fichiers principaux du back-end (matériel NAS ou Stockage Blob Azure) pour le stockage à long terme.

Chaque cluster Avere vFXT pour Azure doit se trouver dans une seule zone de disponibilité, mais vous pouvez utiliser des clusters redondants situés dans des zones différentes ou des régions différentes pour fournir un accès rapide en cas de panne d’une région.

Vous pouvez également positionner des conteneurs de stockage dans plusieurs régions si vous craignez de perdre l’accès aux données. Cependant, gardez à l’esprit que les transactions entre régions ont une latence plus élevée et un coût plus élevé que les transactions qui se font au sein d’une même région.

## <a name="protect-data-in-the-cluster-cache"></a>Protéger les données dans le cache du cluster

Les données mises en cache sont toujours écrites dans les serveurs de fichiers principaux avant un arrêt normal, mais dans un arrêt non contrôlé, les données modifiées dans le cache peuvent être perdues.

Si vous utilisez le cluster pour optimiser seulement les lectures de fichiers, il n’y a pas de modifications à perdre. Si vous utilisez aussi le cluster pour mettre en cache les modifications des fichiers (écritures), déterminez s’il convient ou non d’ajuster les [stratégies de cache](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_manage_cache_policies.html) des serveurs de fichiers principaux<!-- link to legacy doc --> pour personnaliser la fréquence à laquelle les données sont écrites dans un stockage à long terme.

D’une façon générale, votre plan de récupération doit être centré sur la sauvegarde des systèmes de stockage back-end, qui contiennent plus de données et sont généralement plus importants pour le rétablissement de votre workflow après une défaillance.

## <a name="protect-data-in-nas-core-filers"></a>Protéger des données dans des serveurs de fichiers principaux de type NAS

Utilisez les méthodes acceptées pour protéger les données stockées dans un serveur de fichiers principal sur du matériel NAS local, notamment des captures instantanées et des sauvegardes complètes, comme recommandé par le fournisseur du NAS. La reprise d’activité pour ces serveurs de fichiers principaux n’est pas couverte par cet article.

## <a name="protect-data-in-azure-blob-storage"></a>Protéger des données dans Stockage Blob Azure

Avere vFXT pour Azure utilise un stockage localement redondant (LRS) pour les serveurs de fichiers principaux de Stockage Blob Azure. Cela signifie que les données de vos conteneurs d’objets blob sont copiées automatiquement de façon à offrir une protection contre les défaillances matérielles temporaires au sein d’un centre de données.

Cette section donne des conseils sur la protection de vos données contre les rares pannes qui peuvent se produire à l’échelle d’une région ou contre les suppressions accidentelles .

Les bonnes pratiques pour la protection des données dans Stockage Blob Azure sont les suivantes :

* Copiez fréquemment vos données critiques vers un autre compte de stockage dans une autre région (à la fréquence déterminée par votre plan de reprise d’activité).
* Contrôlez l’accès aux données sur tous les systèmes cibles pour empêcher la suppression ou l’endommagement accidentel. Envisagez d’utiliser des [verrous de ressources](../azure-resource-manager/management/lock-resources.md) sur le stockage de données.
* Activez la fonctionnalité d’[instantané cloud](<https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_cloud_snapshot_policies.html>) d’Avere vFXT pour Azure pour vos serveurs de fichiers principaux d’objets blob.

### <a name="copy-avere-vfxt-core-filer-data-to-a-backup-account"></a>Copier les données du serveur de fichiers principal Avere vFXT vers un compte de sauvegarde

Suivez ces étapes pour mettre en place une sauvegarde des données dans un autre compte.

1. Si nécessaire, générez une nouvelle clé de chiffrement et stockez-la de façon sécurisée en dehors des systèmes affectés.

   Si vos données sont chiffrées par le cluster Avere vFXT pour Azure, vous devez générer une nouvelle clé de chiffrement avant de copier les données vers un autre compte de stockage. Stockez cette clé et votre mot de passe dans une installation sécurisée et qui ne sera pas affectée par une défaillance régionale.

   Vous devez fournir cette clé lors de l’ajout du conteneur à un cluster, même si vous l’ajoutez à nouveau à son cluster d’origine.

   Lisez [Paramètres de chiffrement cloud](<https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_cloud_encryption_settings.html>)<!-- link to legacy doc site --> pour obtenir des informations détaillées.

   Si votre conteneur utilise seulement le chiffrement intégré d’Azure, vous pouvez ignorer cette étape.

1. Supprimez le serveur de fichiers principal du système. Ceci force le cluster à écrire toutes les données modifiées dans le stockage back-end.

   Même si vous devrez ajouter à nouveau le serveur de fichiers principal après la sauvegarde, la suppression de ce dernier est la meilleure façon de garantir que toutes les données sont entièrement écrites dans le back end. (L’option « Suspendre » peut parfois laisser des données modifiées dans le cache.) <!-- xxx true? or just metadata? -->

   Notez le nom et les informations de jonction du serveur de fichiers principal (listées dans la page **Espace de noms** dans le Panneau de configuration) pour pouvoir le répliquer quand vous ajoutez à nouveau le conteneur après la sauvegarde.

   Utilisez le Panneau de configuration du cluster pour supprimer le serveur de fichiers principal. [Ouvrez le Panneau de configuration du cluster](avere-vfxt-cluster-gui.md) et choisissez **Serveur de fichiers principal** > **Gérer les serveurs de fichiers principaux**. Recherchez le système de stockage que vous voulez sauvegarder et utilisez le bouton **Supprimer** pour le supprimer du cluster.

1. Créez un conteneur de stockage d’objets blob vide dans un autre compte de stockage dans une autre région.

1. Utilisez n’importe quel outil de copie pratique pour copier les données du serveur de fichiers principal vers le nouveau conteneur. La copie doit répliquer les données sans modifications et sans perturber le format du système de fichiers cloud propriétaire utilisé par Avere vFXT pour Azure. Les outils basés sur Azure incluent [AzCopy](../storage/common/storage-use-azcopy-v10.md), [Azure PowerShell](../data-lake-store/data-lake-store-get-started-powershell.md) et [Azure Data Factory](../data-factory/connector-azure-data-lake-store.md).

1. Après avoir copié les données dans le conteneur de sauvegarde, ajoutez de nouveau le conteneur d’origine au cluster, comme décrit dans [Configurer le stockage](avere-vfxt-add-storage.md).

   * Utilisez le même nom et les mêmes informations de jonction du serveur de fichiers principal de façon à ne pas devoir modifier les workflows clients.
   * Définissez la valeur de **Contenu du compartiment** sur l’option de données existante.
   * Si le conteneur a été chiffré par le cluster, vous devez entrer la clé de chiffrement actuelle pour son contenu. (Il s’agit de la clé que vous avez mise à jour à l’étape 1.)

Pour les sauvegardes qui suivent la première, vous n’avez pas besoin de créer un nouveau conteneur de stockage. Cependant, vous pouvez envisager de générer une nouvelle clé de chiffrement chaque fois que vous effectuez une sauvegarde, pour être sûr que la clé actuelle est stockée dans un endroit dont vous vous rappelez.

### <a name="access-a-backup-data-source-during-an-outage"></a>Accéder à une source de données de sauvegarde lors d’une panne

Pour accéder au conteneur de sauvegarde à partir d’un cluster Avere vFXT pour Azure, procédez comme suit :

1. Si nécessaire, créez un cluster Avere vFXT pour Azure dans une région non affectée.

   > [!TIP]
   > Quand vous créez un cluster Avere vFXT pour Azure, vous pouvez enregistrer une copie de son modèle et de ses paramètres de création. Si vous enregistrez ces informations lors de la création de votre cluster principal, vous pouvez les utiliser pour créer un cluster de remplacement avec les mêmes propriétés. Dans la page [Récapitulatif](avere-vfxt-deploy.md#validation-and-purchase), cliquez sur le lien **Télécharger le modèle et les paramètres**. Enregistrez les informations dans un fichier avant de créer le cluster.

1. Ajoutez un nouveau serveur de fichiers principal cloud qui pointe vers le conteneur d’objets blob dupliqué.

   Veillez à spécifier que le conteneur cible contient déjà des données dans le paramètre **Contenu du compartiment** de l’Assistant Création de serveur de fichiers principal. (Le système doit vous alerter si vous laissez accidentellement ce paramètre défini sur **Vide**.)  <!-- you can't add a populated volume at cluster creation time via template, only create a fresh one -->

1. Si nécessaire, mettez à jour les clients afin qu’ils montent le nouveau cluster ou le nouveau serveur de fichiers principal au lieu de l’original. (Si vous ajoutez le serveur de fichiers principal de remplacement avec le même nom et le même chemin que le conteneur d’origine, vous n’avez pas besoin de mettre à jour les processus clients, sauf si vous devez monter le nouveau cluster à une nouvelle adresse IP.)

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la personnalisation des paramètres pour Avere vFXT pour Azure, consultez [Optimisation du cluster](avere-vfxt-tuning.md).
* Découvrez plus d’informations sur la reprise d’activité et la création d’applications résilientes dans Azure :

  * [Guide technique de la résilience Azure](/azure/architecture/framework/resiliency/overview)
  * [Récupérer après une interruption de service à l’échelle de la région](/azure/architecture/resiliency/recovery-loss-azure-region)
  * [Récupération d’urgence et haute disponibilité pour les applications Azure](/azure/architecture/framework/resiliency/backup-and-recovery)
  <!-- can't find these in the source tree to use relative links -->