---
title: Vue d’ensemble des coffres de sauvegarde
description: Une vue d’ensemble des coffres de sauvegarde.
ms.topic: conceptual
ms.date: 08/17/2020
ms.openlocfilehash: c189997ecc4814917182246b35003649d317ac77
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92091285"
---
# <a name="backup-vaults-overview"></a>Vue d’ensemble des coffres de sauvegarde

Cet article décrit les fonctionnalités d’un coffre de sauvegarde. Un coffre de sauvegarde est une entité de stockage dans Azure qui héberge des données de sauvegarde pour certaines charges de travail plus récentes prises en charge par Sauvegarde Azure. Vous pouvez utiliser des coffres de sauvegarde afin de stocker des données de sauvegarde pour différents services Azure, comme des serveurs Azure Database pour PostgreSQL et des charges de travail plus récentes qui seront prises en charge par Sauvegarde Azure. Les coffres de sauvegarde facilitent l’organisation de vos données de sauvegarde tout en réduisant le temps nécessaire à leur gestion. Les coffres de sauvegarde sont basés sur le modèle Azure Resource Manager, qui fournit des fonctionnalités comme :

- **Fonctionnalités enrichies pour sécuriser les données de sauvegarde** : Avec les coffres de sauvegarde, Sauvegarde Azure fournit des fonctionnalités de sécurité pour protéger les sauvegardes cloud. Ces fonctionnalités de sécurité vous garantissent de pouvoir sécuriser vos sauvegardes et récupérer en toute sécurité des données même si des serveurs de production et de sauvegarde sont compromis. [En savoir plus](backup-azure-security-feature.md)

- **Contrôle d’accès en fonction du rôle Azure (Azure RBAC)**  : Azure RBAC fournit un contrôle de gestion des accès affiné dans Azure. [Azure offre différents rôles intégrés](../role-based-access-control/built-in-roles.md), et Sauvegarde Microsoft Azure comprend trois [rôles intégrés pour gérer les points de récupération](backup-rbac-rs-vault.md). Les coffres de sauvegarde sont compatibles avec Azure RBAC, qui restreint l’accès aux sauvegardes et restaurations à l’ensemble défini de rôles d’utilisateur. [En savoir plus](backup-rbac-rs-vault.md)

## <a name="storage-settings-in-the-backup-vault"></a>Paramètres de stockage dans le coffre de sauvegarde

Un coffre de sauvegarde est une entité qui stocke les sauvegardes et les points de récupération créés au fil du temps. Le coffre de sauvegarde contient également les stratégies de sauvegarde associées aux machines virtuelles protégées.

- La Sauvegarde Azure gère automatiquement le stockage du coffre. Choisissez la redondance de stockage qui correspond aux besoins de votre entreprise lors de la création du coffre de sauvegarde.

- Pour en savoir plus sur la redondance du stockage, consultez les articles suivants sur la redondance [géographique](../storage/common/storage-redundancy.md#geo-redundant-storage) et la redondance [locale](../storage/common/storage-redundancy.md#locally-redundant-storage).

## <a name="encryption-settings-in-the-backup-vault"></a>Paramètres de chiffrement dans le coffre de sauvegarde

Cette section décrit les options disponibles pour le chiffrement de vos données de sauvegarde stockées dans le coffre de sauvegarde.

### <a name="encryption-of-backup-data-using-platform-managed-keys"></a>Chiffrement des données de sauvegarde à l’aide de clés gérées par la plateforme

Par défaut, toutes vos données sont chiffrées à l’aide de clés gérées par la plateforme. Vous n’avez aucune action explicite à effectuer pour activer ce chiffrement. Il s’applique à toutes les charges de travail sauvegardées dans votre coffre de sauvegarde.

## <a name="create-a-backup-vault"></a>Créer un coffre de sauvegarde

Un coffre de sauvegarde est une entité de gestion qui stocke les points de récupération créés au fil du temps et il fournit une interface permettant d’effectuer des opérations liées à la sauvegarde. Cela inclut l’exécution de sauvegardes à la demande, l’exécution de restaurations et la création de stratégies de sauvegarde.

Pour créer un coffre de sauvegarde, effectuez ces étapes.

### <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur <https://portal.azure.com>.

### <a name="create-backup-vault"></a>Créer un coffre de sauvegarde

1. Tapez **Coffres de sauvegarde** dans la zone de recherche.
1. Sous **Services** , sélectionnez **Coffres de sauvegarde** .
1. Dans la page **Coffres de sauvegarde** , sélectionnez **Ajouter** .
1. Sous l’onglet **Général** , sous **Détails du projet** , vérifiez que l’abonnement approprié est sélectionné, puis choisissez **Créer un groupe de ressources** . Tapez *myResourceGroup* comme nom.

  ![Créer un groupe de ressources](./media/backup-vault-overview/new-resource-group.png)

1. Sous **Détails de l’instance** , tapez *myVault* pour **Nom du coffre de sauvegarde** , puis choisissez votre région, dans le cas présent *USA Est* pour **Région** .
1. Choisissez maintenant votre **Redondance du stockage** . La redondance du stockage ne peut pas être changée une fois que des éléments sont protégés dans le coffre.
1. Si vous utilisez Azure comme principal point de terminaison du stockage de sauvegarde, nous vous recommandons de continuer à utiliser le paramètre **Géoredondant** par défaut.
1. Sinon, choisissez l’option **Redondant en local** qui réduit les coûts de stockage Azure.
1. Informez-vous sur la redondance [géo](../storage/common/storage-redundancy.md#geo-redundant-storage) et [locale](../storage/common/storage-redundancy.md#locally-redundant-storage).

  ![Choisir la redondance du stockage](./media/backup-vault-overview/storage-redundancy.png)

1. Sélectionnez le bouton Vérifier + créer dans le bas de la page.

    ![Sélectionner Vérifier + créer](./media/backup-vault-overview/review-and-create.png)

## <a name="delete-a-backup-vault"></a>Supprimer un coffre de sauvegarde

Cette section décrit comment supprimer un coffre de sauvegarde. Il indique comment supprimer des dépendances, puis un coffre.

### <a name="before-you-start"></a>Avant de commencer

Vous ne pouvez pas supprimer un coffre de sauvegarde ayant une des dépendances suivantes :

- Vous ne pouvez pas supprimer un coffre qui contient des sources de données protégées (par exemple des serveurs Azure Database pour PostgreSQL).
- Vous ne pouvez pas supprimer un coffre qui contient des données de sauvegarde.

Si vous essayez de supprimer le coffre sans supprimer les dépendances, vous obtenez les messages d’erreur suivants :

>Impossible de supprimer le coffre de sauvegarde, car des instances de sauvegarde ou des stratégies de sauvegarde existent dans le coffre. Supprimez toutes les instances de sauvegarde et toutes les stratégies de sauvegarde présentes dans le coffre, puis essayez de supprimer le coffre.

### <a name="proper-way-to-delete-a-vault"></a>Méthode appropriée pour supprimer un coffre

>[!WARNING]
L’opération suivante est destructrice et ne peut pas être annulée. Toutes les données de sauvegarde et tous les éléments de sauvegarde associés au serveur protégé seront définitivement supprimés. Procédez avec prudence.

Pour supprimer correctement un coffre, vous devez suivre les étapes ci-après dans l’ordre indiqué :

- Vous devez vérifier s’il existe des éléments protégés :
  - Accédez à **Instances de sauvegarde** dans la barre de navigation de gauche. Tous les éléments listés ici doivent d’abord être supprimés.

Une fois ces étapes terminées, vous pouvez poursuivre la suppression du coffre.

### <a name="delete-the-backup-vault"></a>Supprimer le coffre de sauvegarde

Lorsqu’il n’y a plus d’éléments dans le coffre, sélectionnez **Supprimer** dans le tableau de bord du coffre. Un texte de confirmation s’affiche, vous demandant si vous voulez supprimer le coffre.

![Supprimer un coffre](./media/backup-vault-overview/delete-vault.png)

1. Sélectionnez **Oui** pour confirmer la suppression du coffre. Le coffre est supprimé. Le portail revient au menu de service **Nouveau** .

## <a name="monitor-and-manage-the-backup-vault"></a>Superviser et gérer le coffre de sauvegarde

Cette section explique comment utiliser le tableau de bord **Vue d’ensemble** des coffres de sauvegarde pour superviser et gérer vos coffres de sauvegarde. Le volet Vue d’ensemble contient deux vignettes : **Travaux** et **Instances** .

![Vue d’ensemble du tableau de bord](./media/backup-vault-overview/overview-dashboard.png)

### <a name="manage-backup-instances"></a>Gérer les instances de sauvegarde

Dans la vignette **Travaux** , vous voyez un récapitulatif de tous les travaux liés à la sauvegarde et à la restauration dans votre coffre de sauvegarde. La sélection de l’un des nombres dans cette vignette vous permet de voir des informations supplémentaires sur les travaux pour un type de source de données, un type d’opération et un état spécifiques.

![Instances de sauvegarde](./media/backup-vault-overview/backup-instances.png)

### <a name="manage-backup-jobs"></a>Gérer les travaux de sauvegarde

La vignette **Instances de sauvegarde** vous montre un récapitulatif de toutes les instances de sauvegarde dans votre coffre de sauvegarde. La sélection de l’un des nombres figurant dans cette vignette vous permet d’afficher des informations supplémentaires sur les instances de sauvegarde pour un type de source de données et un état de protection spécifiques.

![Travaux de sauvegarde](./media/backup-vault-overview/backup-jobs.png)

## <a name="next-steps"></a>Étapes suivantes

- [Configurer la sauvegarde sur des bases de données Azure PostgreSQL](backup-azure-database-postgresql.md#configure-backup-on-azure-postgresql-databases)
