---
title: Préparer Azure Migrate à utiliser un outil ISV/Movere
description: Cet article explique comment préparer Azure Migrate à utiliser un outil ISV ou Movere, puis comment commencer à utiliser l’outil.
author: ms-psharma
ms.author: panshar
ms.manager: abhemraj
ms.topic: how-to
ms.date: 06/10/2020
ms.openlocfilehash: 1716db0476169e12822b3f47f7199bf6e2c4ee92
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753771"
---
# <a name="prepare-to-work-with-an-isv-tool-or-movere"></a>Se préparer à utiliser un outil ISV ou Movere

Si vous avez ajouté un [outil ISV](migrate-services-overview.md#isv-integration) ou Movere à un projet Azure Migrate, vous devez suivre quelques étapes avant de lier l’outil et d’envoyer des données à Azure Migrate. 

## <a name="check-azure-ad-permissions"></a>Vérifier les autorisations dans Azure AD

Votre compte d’utilisateur Azure a besoin des autorisations suivantes :

- Autorisation d’inscrire une application Azure Active Directory (Azure AD) auprès de votre locataire Azure
- Autorisation d’allouer un rôle à l’application Azure AD au niveau de l’abonnement


### <a name="set-permissions-to-register-an-azure-ad-app"></a>Définir des autorisations pour inscrire une application Azure AD

1. Dans Azure AD, vérifiez le rôle de votre compte.
2. Si vous avez le rôle d’utilisateur, sélectionnez **Paramètres utilisateur** à gauche et vérifiez si les utilisateurs peuvent inscrire des applications. Si la valeur est **Oui**, tous les utilisateurs dans le locataire Azure AD peuvent inscrire une application. Si la valeur est **Non**, seuls les utilisateurs administrateurs peuvent inscrire des applications.   
3. Si vous n’avez pas d’autorisations, un utilisateur administrateur peut fournir à votre compte d’utilisateur le rôle [d'administrateur de l’application](../active-directory/roles/permissions-reference.md#application-administrator), afin que vous puissiez inscrire l’application.
4. Une fois l’outil lié à Azure Migrate, l’administrateur peut supprimer le rôle de votre compte.

### <a name="set-permissions-to-assign-a-role-to-an-azure-ad-app"></a>Définir des autorisations pour affecter un rôle à une application Azure AD
 
Dans votre abonnement Azure, votre compte doit disposer de l’accès **Microsoft.Authorization/*/Write** pour attribuer un rôle à une application Azure AD. 

1. Sur le portail Azure, ouvrez **Abonnements**.
2. Sélectionnez l’abonnement approprié. Si vous ne le voyez pas, sélectionnez le **filtre d’abonnements généraux**. 
3. Sélectionner **Mes autorisations**. Ensuite, sélectionnez **Cliquer ici pour afficher les détails d’accès complet pour cet abonnement**.
4. Vérifiez les autorisations dans **Affectations de rôles** > **Affichage**. Si votre compte n’a pas d’autorisations, demandez à l’administrateur de l’abonnement de vous ajouter au rôle [Administrateur de l'accès utilisateur](../role-based-access-control/built-in-roles.md#user-access-administrator) ou au rôle [Propriétaire](../role-based-access-control/built-in-roles.md#owner).

## <a name="allow-access-to-urls"></a>Autoriser l'accès à des URL

Pour les outils ISV et l’Assistant Migration de données Azure, autorisez l’accès aux URL de cloud public résumées dans le tableau. Si vous utilisez un proxy basé sur les URL pour vous connecter à Internet, vérifiez que le proxy résout tous les enregistrements CNAME reçus lors de la recherche des URL. 

**URL** | **Détails**
--- | ---
*. portal.azure.com  | Accédez au portail Azure. 
\* .windows.net<br/> *.msftauth.net<br/> *.msauth.net <br/> *.microsoft.com<br/> *.live.com   | Connectez-vous à votre abonnement Azure. 
*.microsoftonline.com<br/> *.microsoftonline-p.com | Créez des applications Azure Active Directory (AD) pour que l’appliance communique avec Azure Migrate. 
management.azure.com | Faites en sorte qu’Azure Resource Manager appelle le projet Azure Migrate.
*.servicebus.windows.net | Communication entre l’appliance et EventHub pour l’envoi des messages.


## <a name="start-using-the-tool"></a>Commencer à utiliser le portail

1. Si vous n’avez pas encore de licence ou d’essai gratuit pour l’outil, accédez à l’accueil de l’outil dans Azure Migrate, sous **Inscrire**, sélectionnez **En savoir plus**.
2. Dans l’outil, suivez les instructions pour lier l’outil au projet Azure Migrate et pour envoyer des données à Azure Migrate.

## <a name="next-steps"></a>Étapes suivantes

Suivez les instructions de votre outil ISV ou Movere pour envoyer des données à Azure Migrate.

   
