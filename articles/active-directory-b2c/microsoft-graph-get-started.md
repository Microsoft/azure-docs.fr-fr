---
title: Gérer les ressources avec Microsoft Graph
titleSuffix: Azure AD B2C
description: Préparez la gestion des ressources Azure AD B2C avec Microsoft Graph en inscrivant une application disposant des autorisations API Graph requises.
services: B2C
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/14/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 61e32f50e02e45487830c5ea7ff476a0571960f6
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94953999"
---
# <a name="manage-azure-ad-b2c-with-microsoft-graph"></a>Gérer Azure AD B2C avec Microsoft Graph

[Microsoft Graph][ms-graph] vous permet de gérer de nombreuses ressources au sein de votre locataire Azure AD B2C, y compris des comptes d’utilisateur client et des stratégies personnalisées. En écrivant des scripts ou applications appelant l’[API Microsoft Graph][ms-graph-api], vous pouvez automatiser les tâches de gestion du locataire, notamment :

* Migrer un magasin d’utilisateurs existant vers un locataire Azure AD B2C
* Déployer des stratégies personnalisées avec un pipeline Azure dans Azure DevOps et gérer des clés de stratégie personnalisées
* Héberger l’inscription des utilisateurs sur votre page et créer des comptes d’utilisateur dans votre répertoire Azure AD B2C en arrière-plan
* Automatiser l'inscription d’application
* Obtenir des journaux d’audit

Les sections suivantes vous préparent à utiliser l’API Microsoft Graph pour automatiser la gestion des ressources dans votre répertoire Azure AD B2C.

## <a name="microsoft-graph-api-interaction-modes"></a>Modes d’interaction de l’API Microsoft Graph

Vous pouvez utiliser deux modes de communication lorsque vous utilisez l’API Microsoft Graph afin de gérer les ressources de votre locataire Azure AD B2C :

* **Interactif** : adapté aux tâches à exécution unique, vous utilisez un compte d’administrateur dans le locataire B2C pour effectuer les tâches de gestion. Ce mode implique qu'un administrateur se connecte à l’aide de ses informations d’identification avant d’appeler l’API Microsoft Graph.

* **Automatisé** : adapté aux tâches planifiées ou exécutées en continu, ce mode utilise un compte de service que vous configurez avec les autorisations requises pour effectuer des tâches de gestion. Vous créez le « compte de service » dans Azure AD B2C en inscrivant une application que vos applications et scripts utilisent à des fins d’authentification à l’aide de leur *ID d’application (client)* et de l’octroi des **informations d’identification du client OAuth 2.0**. Dans ce cas, l’application, et non plus l'utilisateur administrateur comme décrit dans le mode interactif, appelle l’API Microsoft Graph.

Vous activez le scénario d’interaction **Automatisé** en créant une inscription d’application comme indiqué dans les sections suivantes.

Bien que le flux d’octroi d’informations d’identification du client OAuth 2.0 ne soit pas directement pris en charge par le service d’authentification Azure AD B2C, vous pouvez configurer le flux d’informations d’identification du client à l’aide d’Azure AD et de la Plateforme d’identités Microsoft ou du point de terminaison de jeton pour une application dans votre locataire Azure AD B2C. Un locataire Azure AD B2C partage certaines fonctionnalités avec les locataires d’entreprise Azure AD.

## <a name="register-management-application"></a>Inscrire une application de gestion

Pour permettre à vos scripts et applications d'interagir avec l’[API Microsoft Graph][ms-graph-api] afin de gérer les ressources Azure AD B2C, vous devez créer une inscription d’application dans votre locataire Azure AD B2C, qui octroie les autorisations d’API requises.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire Azure AD B2C.
1. Dans le portail Azure, recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Inscriptions d’applications**, puis **Nouvelle inscription**.
1. Entrez un **Nom** pour l’application. Par exemple, *managementapp1*.
1. Sélectionnez **Comptes dans ce répertoire organisationnel uniquement**.
1. Sous **Autorisations**, décochez la case *Accorder le consentement administrateur aux autorisations openid et offline_access*.
1. Sélectionnez **Inscription**.
1. Enregistrez l’**ID d’application (client)** indiqué dans la page d’aperçu de l’application. Vous utiliserez cette valeur ultérieurement.

### <a name="grant-api-access"></a>Accorder un accès à l’API

Accordez ensuite les autorisations d’application inscrites pour manipuler les ressources de locataire par le biais d’appels à l’API Microsoft Graph.

[!INCLUDE [active-directory-b2c-permissions-directory](../../includes/active-directory-b2c-permissions-directory.md)]

### <a name="create-client-secret"></a>Créer un secret client

[!INCLUDE [active-directory-b2c-client-secret](../../includes/active-directory-b2c-client-secret.md)]

Vous disposez maintenant d’une application autorisée à *créer*, *lire*, *mettre à jour* et *supprimer* des utilisateurs dans votre locataire Azure AD B2C. Passez à la section suivante pour ajouter des autorisations de *mise à jour des mots de passe*.

## <a name="enable-user-delete-and-password-update"></a>Activer la suppression d’utilisateurs et la mise à jour des mots de passe

L’autorisation *Accéder en lecture et en écriture aux données de l’annuaire* n’inclut **PAS** la possibilité de supprimer des utilisateurs ou de mettre à jour des mots de passe pour les comptes d'utilisateur.

Si votre application ou script doit supprimer des utilisateurs ou mettre à jour leur mot de passe, attribuez le rôle *Administrateur d'utilisateurs* à votre application :

1. Connectez-vous au [portail Azure](https://portal.azure.com) et utilisez le filtre **Répertoire + abonnement** pour basculer vers votre locataire Azure AD B2C.
1. Recherchez et sélectionnez **Azure AD B2C**.
1. Sous **Gérer**, sélectionnez **Rôles et administrateurs**.
1. Sélectionnez le rôle **Administrateur d’utilisateurs**.
1. Sélectionnez **Ajouter des affectations**.
1. Dans la zone de texte **Sélectionner**, entrez le nom de l’application que vous avez inscrite, par exemple *managementapp1*. Sélectionnez votre application quand elle apparaît dans les résultats de la recherche.
1. Sélectionnez **Ajouter**. La propagation complète des autorisations peut prendre quelques minutes.

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez inscrit votre application de gestion et lui avez accordé les autorisations requises, vos applications et services (par exemple, Azure Pipelines) peuvent utiliser ses informations d’identification et autorisations pour interagir avec l’API Microsoft Graph. 

* [Obtenir un jeton d’accès d’Azure AD](/graph/auth-v2-service#4-get-an-access-token)
* [Utilisez le jeton d’accès pour appeler Microsoft Graph](/graph/auth-v2-service#4-get-an-access-token)
* [Opérations B2C prises en charge par Microsoft Graph](microsoft-graph-operations.md)
* [Gérer les comptes d’utilisateur Azure AD B2C avec Microsoft Graph](manage-user-accounts-graph-api.md)
* [Obtenir les journaux d’audit avec l’API de création de rapports Azure AD](view-audit-logs.md#get-audit-logs-with-the-azure-ad-reporting-api)

<!-- LINKS -->
[ms-graph]: /graph/
[ms-graph-api]: https://docs.microsoft.com/graph/api/overview