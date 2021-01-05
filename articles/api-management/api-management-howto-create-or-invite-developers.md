---
title: Gestion des comptes d’utilisateur dans Gestion des API Azure | Microsoft Docs
description: Apprenez à créer ou à inviter des utilisateurs dans Gestion des API Azure. Affichez les ressources supplémentaires à utiliser après la création d’un compte de développeur.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 02/13/2018
ms.author: apimpm
ms.openlocfilehash: 92e032eb104835788f515cc7800fe5dacfa8adaa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88566129"
---
# <a name="how-to-manage-user-accounts-in-azure-api-management"></a>Gestion des comptes d’utilisateur dans Gestion des API Azure

Dans Gestion des API Azure, les développeurs sont les utilisateurs des API que vous exposez via Gestion des API. Ce guide montre comment créer et inviter des développeurs à utiliser les API et les produits que vous mettez à leur disposition dans votre instance de Gestion des API. Pour plus d’informations sur la gestion des comptes d’utilisateur par programme, consultez la documentation [Entité utilisateur](/rest/api/apimanagement/2019-12-01/user) dans la référence [API REST de gestion](/rest/api/apimanagement/).

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="prerequisites"></a>Prérequis

Effectuez les tâches indiquées dans cet article : [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md).

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="create-a-new-developer"></a><a name="create-developer"> </a>Création d’un développeur

Pour ajouter un nouvel utilisateur, procédez comme suit :

1. Sélectionnez l’onglet **Utilisateurs** à gauche de l’écran.
2. Appuyez sur **+ Ajouter**.
3. Entrez les informations appropriées concernant l’utilisateur.
4. Cliquez sur **Ajouter**.

    ![Ajouter un nouvel utilisateur](./media/api-management-howto-create-or-invite-developers/api-management-create-developer.png)

Par défaut, les comptes de développeurs nouvellement créés sont **actifs**. Ils sont associés au groupe **Développeurs**. Les comptes de développeurs dont l'état est **actif** peuvent être utilisés pour accéder à toutes les API auxquelles ils sont abonnés. Pour associer les développeurs nouvellement créés à d'autres groupes, consultez la rubrique [Association de groupes à des développeurs][How to associate groups with developers].

## <a name="invite-a-developer"></a><a name="invite-developer"> </a>Invitation d’un développeur
Pour inviter un développeur, procédez comme suit :

1. Sélectionnez l’onglet **Utilisateurs** à gauche de l’écran.
2. Appuyez sur **+Inviter**.

Un message de confirmation s’affiche, mais le développeur qui vient d’être invité n’apparaît pas dans la liste tant qu’il n’a pas accepté l’invitation. 

Quand un développeur est invité, un message lui est envoyé. Ce message est généré à partir d'un modèle. Il est personnalisable. Pour plus d’informations, consultez la page [Configuration des modèles de courrier électronique][Configure email templates].

Une fois l'invitation acceptée, le compte est activé.

## <a name="deactivate-or-reactivate-a-developer-account"></a><a name="block-developer"> </a> Désactiver ou réactiver un compte de développeur

Par défaut, les comptes de développeur nouvellement créés ou invités sont **actifs**. Pour désactiver un compte de développeur, cliquez sur **Bloquer**. Pour réactiver un compte de développeur bloqué, cliquez sur **Activer**. Les comptes de développeurs bloqués ne peuvent pas accéder au portail des développeurs, ni appeler les API. Pour supprimer un compte d’utilisateur, cliquez sur **Supprimer**.

Pour bloquer un utilisateur, procédez comme suit.

1. Sélectionnez l’onglet **Utilisateurs** à gauche de l’écran.
2. Cliquez sur l’utilisateur que vous souhaitez bloquer.
3. Appuyez sur **Bloquer**.

## <a name="reset-a-user-password"></a>Réinitialiser le mot de passe d’un utilisateur

Pour utiliser les comptes d’utilisateur par programme, consultez la documentation Entité utilisateur dans la référence [API REST Gestion des API](/rest/api/apimanagement/). Pour réinitialiser le mot de passe d’un compte d’utilisateur sur une valeur spécifique, vous pouvez utiliser l’opération [Mettre à jour un utilisateur](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-user-entity#UpdateUser) et spécifier le mot de passe nécessaire.

## <a name="next-steps"></a><a name="next-steps"> </a>Étapes suivantes
Une fois le compte de développeur créé, vous pouvez l'associer à des rôles et l'abonner à des produits et des API. Pour plus d'informations, consultez la page [Création et utilisation de groupes][How to create and use groups].

[api-management-management-console]: ./media/api-management-howto-create-or-invite-developers/api-management-management-console.png
[api-management-add-new-user]: ./media/api-management-howto-create-or-invite-developers/api-management-add-new-user.png
[api-management-create-developer]: ./media/api-management-howto-create-or-invite-developers/api-management-create-developer.png
[api-management-invite-developer]: ./media/api-management-howto-create-or-invite-developers/api-management-invite-developer.png
[api-management-new-developer]: ./media/api-management-howto-create-or-invite-developers/api-management-new-developer.png
[api-management-invite-developer-window]: ./media/api-management-howto-create-or-invite-developers/api-management-invite-developer-window.png
[api-management-invite-developer-confirmation]: ./media/api-management-howto-create-or-invite-developers/api-management-invite-developer-confirmation.png
[api-management-pending-verification]: ./media/api-management-howto-create-or-invite-developers/api-management-pending-verification.png
[api-management-view-developer]: ./media/api-management-howto-create-or-invite-developers/api-management-view-developer.png
[api-management-reset-password]: ./media/api-management-howto-create-or-invite-developers/api-management-reset-password.png


[Create a new developer]: #create-developer
[Invite a developer]: #invite-developer
[Deactivate or reactivate a developer account]: #block-developer
[Next steps]: #next-steps
[How to create and use groups]: api-management-howto-create-groups.md
[How to associate groups with developers]: api-management-howto-create-groups.md#associate-group-developer

[Get started with Azure API Management]: get-started-create-service-instance.md
[Create an API Management service instance]: get-started-create-service-instance.md
[Configure email templates]: api-management-howto-configure-notifications.md#email-templates
