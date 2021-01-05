---
title: Créer un groupe d’administration avec le portail Azure
description: Dans ce guide de démarrage rapide, vous utilisez le portail Azure pour créer un groupe d’administration afin d’organiser vos ressources dans une hiérarchie de ressources.
ms.date: 08/31/2020
ms.topic: quickstart
ms.openlocfilehash: c8cb7b8bd3ad33d1f315670bfbb1782972e6e97f
ms.sourcegitcommit: f8d2ae6f91be1ab0bc91ee45c379811905185d07
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/10/2020
ms.locfileid: "89661748"
---
# <a name="quickstart-create-a-management-group"></a>Démarrage rapide : Créer un groupe d’administration

Les groupes d’administration sont des conteneurs qui vous aident à gérer l’accès, la stratégie et la conformité dans plusieurs abonnements. Créez ces conteneurs pour construire une hiérarchie efficace utilisable avec [Azure Policy](../policy/overview.md) et les [contrôles d’accès en fonction du rôle Azure](../../role-based-access-control/overview.md). Pour plus d’informations sur les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](overview.md).

La création du premier groupe d’administration dans l’annuaire peut nécessiter jusqu’à 15 minutes. En effet, des processus s’exécutent la première fois pour configurer le service des groupes d’administration dans Azure pour votre annuaire. Vous recevez une notification quand le processus est terminé. Pour plus d’informations, consultez la [configuration initiale des groupes d’administration](./overview.md#initial-setup-of-management-groups).

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

- Tout utilisateur Azure AD dans le locataire peut créer un groupe d’administration sans l’autorisation d’écriture du groupe d’administration attribuée à cet utilisateur si la [protection de la hiérarchie](./how-to/protect-resource-hierarchy.md#setting---require-authorization) n’est pas activée. Ce nouveau groupe d’administration devient alors un enfant du groupe d’administration racine ou le [groupe d’administration par défaut](./how-to/protect-resource-hierarchy.md#setting---default-management-group), et le rôle « Propriétaire » est attribué au créateur. Le service de groupe d’administration offre cette possibilité afin que les attributions de rôles ne soient pas nécessaires au niveau de la racine. Aucun utilisateur n’a accès au groupe d’administration racine lors de sa création. Pour éviter l’obstacle que constitue la recherche des administrateurs généraux Azure AD pour commencer à utiliser des groupes d’administration, nous autorisons la création des groupes d’administration initiaux au niveau racine.

### <a name="create-in-portal"></a>Créer dans le portail

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez **Tous les services** > **Gestion + gouvernance**.

1. Sélectionnez **Groupes d’administration**.

1. Sélectionnez **+ Ajouter un groupe d’administration**.

   :::image type="content" source="./media/main.png" alt-text="Capture d’écran de la page Groupes d’administration montrant les abonnements et les groupes d’administration enfants." border="false":::

1. Laissez l’option **Créer** sélectionnée et renseignez le champ ID du groupe d’administration.

   - L’**ID du groupe d’administration** est l’identificateur unique de l’annuaire utilisé pour envoyer des commandes sur ce groupe d’administration. Cet identificateur n’est pas modifiable après sa création car il est utilisé dans tout le système Azure pour identifier ce groupe. Le [groupe d’administration racine](./overview.md#root-management-group-for-each-directory) est automatiquement créé avec un ID qui correspond à l’ID Azure Active Directory. Pour tous les autres groupes d’administration, affectez un ID unique.
   - Le champ du nom d’affichage correspond au nom qui s’affiche dans le portail Azure. Un nom d’affichage distinct est un champ facultatif lors de la création du groupe d’administration. Il peut être modifié à tout moment.

   :::image type="content" source="./media/create_context_menu.png" alt-text="Capture d’écran de la page Groupes d’administration montrant les abonnements et les groupes d’administration enfants.":::

1. Sélectionnez **Enregistrer**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer le groupe d’administration créé, effectuez ces étapes :

1. Sélectionnez **Tous les services** > **Gestion + gouvernance**.

1. Sélectionnez **Groupes d’administration**.

1. Recherchez le groupe d’administration créé ci-dessus, sélectionnez-le, puis sélectionnez **Détails** en regard du nom.
   Ensuite, sélectionnez **Supprimer** et confirmez l’opération dans l’invite.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un groupe d’administration pour organiser votre hiérarchie de ressources. Le groupe d’administration peut contenir des abonnements ou d’autres groupes d’administration.

Pour en savoir plus sur les groupes d’administration et sur la gestion de votre hiérarchie de ressources, consultez :

> [!div class="nextstepaction"]
> Gérer vos ressources avec des groupes d’administration