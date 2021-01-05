---
title: Configurer la sécurité pour accorder l’accès aux données - Azure Time Series Insights | Microsoft Docs
description: Découvrez comment configurer la sécurité, les autorisations et gérer des stratégies d’accès aux données dans votre environnement Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: shipra1mishra
ms.author: shmishr
manager: dviso
ms.reviewer: v-mamcge, jasonh, kfile
ms.workload: big-data
ms.topic: conceptual
ms.date: 12/02/2020
ms.custom: seodec18
ms.openlocfilehash: 84b973dfa016b069b18fda47a4336fe952f73b3c
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96780856"
---
# <a name="grant-data-access-to-an-environment"></a>Accorder l’accès aux données dans un environnement

Cet article décrit les deux types de stratégies d’accès d’Azure Time Series Insights.

## <a name="sign-in-to-azure-time-series-insights"></a>Connectez-vous à Azure Time Series Insights

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Localisez votre environnement Azure Time Series Insights en entrant `Time Series Insights environments` dans la **boîte de recherche**. Sélectionnez `Time Series Insights environments` dans les résultats de la recherche.
1. Sélectionnez votre environnement Azure Time Series Insights dans la liste.

## <a name="grant-data-access"></a>Accorder l’accès aux données

Effectuez les étapes suivantes pour accorder l’accès aux données à un utilisateur principal.

1. Sélectionnez **Stratégies d’accès aux données**, puis **+Ajouter**.

    [![Sélectionner et ajouter une stratégie d’accès aux données](media/data-access/data-access-select-add-button.png)](media/data-access/data-access-select-add-button.png#lightbox)

1. Choisissez **Sélectionner un utilisateur**. Recherchez le nom d’utilisateur ou l’adresse e-mail pour accéder à l’utilisateur que vous voulez ajouter. Sélectionnez **Sélectionner** pour confirmer la sélection.

    [![Sélectionner un utilisateur à ajouter](media/data-access/data-access-select-user-to-confirm.png)](media/data-access/data-access-select-user-to-confirm.png#lightbox)

1. Choisissez **Sélectionner un rôle**. Choisissez le rôle d’accès approprié pour l’utilisateur :

    * Sélectionnez **Contributeur** si vous voulez autoriser l’utilisateur à changer les données de référence, et à partager des requêtes et des perspectives enregistrées avec d’autres utilisateurs de l’environnement.

    * Sinon, sélectionnez **Lecteur** pour autoriser l’utilisateur à interroger les données dans l’environnement et à enregistrer des requêtes personnelles non partagées dans l’environnement.

   Sélectionnez **OK** pour confirmer le choix du rôle.

    [![Confirmer le rôle sélectionné](media/data-access/data-access-select-a-role.png)](media/data-access/data-access-select-a-role.png#lightbox)

1. Sélectionnez **OK** dans la page **Sélectionner un rôle utilisateur**.

    [![Sélectionner OK sur la page Sélectionner un rôle utilisateur](media/data-access/data-access-confirm-user-and-role.png)](media/data-access/data-access-confirm-user-and-role.png#lightbox)

1. Confirmez que la page **Stratégies d’accès aux données** répertorie les utilisateurs, et les rôles pour chaque utilisateur.

    [![Vérifier les utilisateurs et les rôles appropriés](media/data-access/data-access-verify-and-confirm-assignments.png)](media/data-access/data-access-verify-and-confirm-assignments.png#lightbox)

## <a name="provide-guest-access-from-another-azure-ad-tenant"></a>Fournir l’accès invité à partir d’un autre locataire Azure AD

Le rôle `Guest` n’est pas à un rôle de gestion. Il est utilisé pour désigner un compte qui a été invité par un autre locataire. Une fois que le compte invité est invité dans le répertoire du locataire, le même contrôle d’accès peut lui être appliqué comme pour tout autre compte. Vous pouvez accorder à l’administration l’accès à un environnement Azure Time Series Insights en utilisant le Panneau Contrôle d’accès (IAM). Vous pouvez également autoriser l’accès aux données de l’environnement via le panneau Stratégies d’accès aux données. Pour plus d’informations sur l’accès invité Azure Active Directory (Azure AD), lisez [Ajouter des utilisateurs Azure Active Directory B2B Collaboration dans le Portail Azure](../active-directory/external-identities/add-users-administrator.md).

Suivez ces étapes pour que l’accès invité à un environnement Azure Time Series Insights soit accordé à un utilisateur Azure AD par un autre locataire.

1. Accédez à Portail Azure, cliquez sur **Azure Active Directory**, faites défiler l’onglet **Présentation**, puis sélectionnez **Utilisateur invité**.

    [![Sélectionner Stratégies d’accès aux données, puis +Inviter](media/data-access/data-access-invite-another-aad-tenant.png)](media/data-access/data-access-invite-another-aad-tenant.png#lightbox)

1. Entrez l’adresse de messagerie de l’utilisateur que vous souhaitez inviter. Cette adresse e-mail doit être associée à Azure AD. Vous pouvez inclure un message personnel dans l’invitation.

    [![Entrer l’adresse e-mail pour rechercher l’utilisateur sélectionné](media/data-access/data-access-invite-guest-by-email.png)](media/data-access/data-access-invite-guest-by-email.png#lightbox)

1. Recherchez la bulle de confirmation qui s’affiche sur l’écran. Vous pouvez également cliquer sur **Notifications** pour confirmer que l’utilisateur invité a été ajouté.

    [![Rechercher la bulle de confirmation qui doit s’afficher](media/data-access/data-access-confirmation-bubble.png)](media/data-access/data-access-confirmation-bubble.png#lightbox)

1. Revenez à votre environnement Time Series Insights pour ajouter l’utilisateur invité nouvellement créé. Cliquez sur **Stratégies d’accès aux données** comme décrit sous **Accorder l’accès aux données**. **Sélectionnez l’utilisateur**. Pour ajouter l’utilisateur que vous avez invité, recherchez son adresse e-mail. Cliquez ensuite sur **Sélectionner** pour confirmer la sélection.

    [![Sélectionner l’utilisateur et confirmer la sélection](media/data-access/data-access-select-invited-person-confirmation.png)](media/data-access/data-access-select-invited-person-confirmation.png#lightbox)

1. Choisissez **Sélectionner un rôle**. Choisissez le rôle d’accès approprié pour l’utilisateur invité :

    * Sélectionnez **Contributeur** si vous voulez autoriser l’utilisateur à changer les données de référence, et à partager des requêtes et des perspectives enregistrées avec d’autres utilisateurs de l’environnement.

    * Sinon, sélectionnez **Lecteur** pour autoriser l’utilisateur à interroger les données dans l’environnement et à enregistrer des requêtes personnelles non partagées dans l’environnement.

   Sélectionnez **OK** pour confirmer le choix du rôle.

    [![Confirmer le choix du rôle](media/data-access/data-access-select-ok-and-confirm.png)](media/data-access/data-access-select-ok-and-confirm.png#lightbox)

1. Sélectionnez **OK** dans la page **Sélectionner un rôle utilisateur**.

1. Confirmez que la page **Stratégies d’accès aux données** liste l’utilisateur invité et les rôles pour chaque utilisateur invité.

    [![Vérifier que les utilisateurs et les rôles sont correctement attribués](media/data-access/data-access-confirm-invited-users-and-roles.png)](media/data-access/data-access-confirm-invited-users-and-roles.png#lightbox)

1. À présent, l’utilisateur invité recevra un e-mail d’invitation à l’adresse e-mail spécifiée ci-dessus. L’utilisateur invité sélectionnera **Démarrer** pour confirmer qu’il accepte et se connecter à Azure Cloud.

    [![L’invité sélectionne Démarrer pour accepter ](media/data-access/data-access-email-invitation.png)](media/data-access/data-access-email-invitation.png#lightbox)

1. Une fois que vous avez sélectionné **Démarrer**, l’utilisateur invité verra s’afficher une zone d’autorisations associée à l’organisation de l’administrateur. Une fois l’autorisation accordée en sélectionnant **Accepter**, il sera connecté.

    [![L’invité passe en revue les autorisations et les accepte](media/data-access/data-access-grant-permission-sign-in.png)](media/data-access/data-access-grant-permission-sign-in.png#lightbox)

1. L’administrateur [partage l’URL d’environnement](time-series-insights-parameterized-urls.md) avec son invité.

1. Une fois que l’utilisateur invité est connecté à l’adresse e-mail que vous avez utilisée pour l’inviter et qu’il accepte l’invitation, il sera dirigé vers le Portail Azure.

1. L’invité peut maintenant accéder à l’environnement partagé à l’aide de l’URL d’environnement fournie par l’administrateur. Pour un accès immédiat, il peut entrer cette URL dans son navigateur web.

1. L’utilisateur invité verra le locataire de l’administrateur en sélectionnant son icône de profil en haut à droite de l’écran dans l’Explorateur Time Series.

    [![Sélection d’un avatar sur insights.azure.com](media/data-access/data-access-select-tenant-and-instance.png)](media/data-access/data-access-select-tenant-and-instance.png#lightbox)

    Une fois que l’utilisateur invité a sélectionné le locataire de l’administrateur, il a la possibilité de sélectionner l’environnement Azure Time Series Insights partagé.

    Il dispose désormais de toutes les fonctionnalités associées au rôle que vous lui avez fourni à l’**étape 5**.

    [![L’utilisateur invité sélectionne votre locataire Azure dans la liste déroulante](media/data-access/data-access-all-capabilities.png)](media/data-access/data-access-all-capabilities.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes

* Lisez [Authentification et autorisation](time-series-insights-authentication-and-authorization.md) pour connaître les étapes d’inscription de l’application Azure Active Directory.

* Affichez [votre environnement dans l’Explorateur Azure Time Series Insights](./concepts-ux-panels.md).
