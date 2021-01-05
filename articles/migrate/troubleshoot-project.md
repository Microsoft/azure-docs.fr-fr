---
title: Résoudre les problèmes des projets Azure Migrate
description: Vous aide à résoudre les problèmes liés à la création et à la gestion de projets Azure Migrate.
author: ms-psharma
ms.author: panshar
ms.manager: abhemraj
ms.topic: troubleshooting
ms.date: 01/01/2020
ms.openlocfilehash: f68a57d3780f388488d48835f322ff04ab7c7187
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753380"
---
# <a name="troubleshoot-azure-migrate-projects"></a>Résoudre les problèmes des projets Azure Migrate

Cet article vous aide à résoudre les problèmes liés à la création et à la gestion de projets [Azure Migrate](migrate-services-overview.md).

## <a name="how-to-add-new-project"></a>Comment ajouter un nouveau projet ?

Vous pouvez avoir plusieurs projets Azure Migrate par abonnement. [Découvrez comment](./create-manage-projects.md) créer un projet pour la première fois, ou [ajouter d’autres](create-manage-projects.md#create-additional-projects) projets.

## <a name="what-azure-permissions-are-needed"></a>Quelles sont les autorisations Azure nécessaires ?

Vous avez besoin d’autorisations Contributeur ou Propriétaire dans l’abonnement pour créer un projet Azure Migrate.

## <a name="cant-find-a-project"></a>Impossible de trouver un projet

La recherche d’un projet Azure Migrate existant varie selon que vous utilisez la version actuelle ou ancienne d’Azure Migrate. [Suivez](create-manage-projects.md#find-a-project).


## <a name="cant-find-a-geography"></a>Impossible de trouver une zone géographique

Vous pouvez créer un projet Azure Migrate dans des zones géographiques prises en charge pour les clouds [publics](migrate-support-matrix.md#supported-geographies-public-cloud) et du [secteur public](migrate-support-matrix.md#supported-geographies-azure-government).

## <a name="what-are-vm-limits"></a>Quelles sont les limites des machines virtuelles ?

Vous pouvez évaluer jusqu’à 35 000 machines virtuelles VMware ou jusqu’à 35 000 machines virtuelles Hyper-V au sein d’un même projet. Un projet peut inclure à la fois des machines virtuelles VMware et des machines virtuelles Hyper-V, jusqu’aux limites d’évaluation.

## <a name="can-i-upgrade-old-project"></a>Puis-je mettre à niveau un ancien projet ?

Les projets de la version précédente d’Azure Migrate ne peuvent pas être mis à jour. Vous devez [créer un nouveau projet](./create-manage-projects.md) et y ajouter des outils.

## <a name="cant-create-a-project"></a>Impossible de créer un projet

Si vous essayez de créer un projet et que vous rencontrez une erreur de déploiement :

- Essayez de créer à nouveau le projet s’il s’agit d’une erreur temporaire. Dans **Déploiements**, cliquez sur **Redéployer** pour réessayer.
- Vérifiez que vous disposez des autorisations de Contributeur ou de Propriétaire dans l’abonnement.
- Si vous déployez dans une nouvelle zone géographique ajoutée, attendez quelques instants et réessayez.
- Si vous recevez l’erreur « Les requêtes doivent contenir des en-têtes d’identité d’utilisateur », cela peut indiquer que vous n’avez pas accès au locataire Azure Active Directory (Azure AD) de l’organisation. Dans ce cas :
    - Quand vous êtes ajouté à un locataire Azure AD pour la première fois, vous recevez une invitation par e-mail pour rejoindre le locataire.
    - Acceptez l’invitation pour être ajouté au locataire.
    - Si vous ne voyez pas l’e-mail, contactez un utilisateur disposant d’un accès au locataire et demandez-lui de vous [renvoyer l’invitation](../active-directory/external-identities/add-users-administrator.md#resend-invitations-to-guest-users).
    - Après avoir reçu l’e-mail d’invitation, ouvrez-le et sélectionnez le lien pour accepter l’invitation. Ensuite, déconnectez-vous du Portail Azure et reconnectez-vous. (l’actualisation du navigateur ne fonctionnera pas). Vous pourrez ensuite créer le projet de migration.

## <a name="how-do-i-delete-a-project"></a>Comment supprimer un projet

[Suivez ces instructions](create-manage-projects.md#delete-a-project) pour supprimer un projet. Notez que lorsque vous supprimez un projet, le projet et les métadonnées concernant les machines découvertes dans le projet sont supprimés.

## <a name="added-tools-dont-show"></a>Les outils ajoutés ne s’affichent pas

Vérifiez que vous avez sélectionné le projet approprié. Dans le hub Azure Migrate > **Serveurs** ou dans **Bases de données**, cliquez sur **Modifier** en regard de **Migrate project (Change)** (Projet Migrate (Modifier)) dans l’angle supérieur droit de l’écran. Choisissez l’abonnement et le nom du projet appropriés > **OK**. La page doit s’actualiser avec les outils ajoutés du projet sélectionné.

## <a name="next-steps"></a>Étapes suivantes

Ajoutez des outils d’[évaluation](how-to-assess.md) ou de [migration](how-to-migrate.md) à des projets Azure Migrate.