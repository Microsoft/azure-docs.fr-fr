---
title: 'Démarrage rapide : Supprimer une application de votre locataire Azure Active Directory (Azure AD)'
description: Ce guide de démarrage rapide utilise le portail Azure pour supprimer une application de votre locataire Azure Active Directory (Azure AD).
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: quickstart
ms.workload: identity
ms.date: 07/01/2020
ms.author: kenwith
ms.openlocfilehash: 677c9c0123c174fdf2cf8e94f9c8110656d21306
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94649396"
---
# <a name="quickstart-delete-an-application-from-your-azure-active-directory-azure-ad-tenant"></a>Démarrage rapide : Supprimer une application de votre locataire Azure Active Directory (Azure AD)

Ce guide de démarrage rapide utilise le portail Azure pour supprimer une application qui a été ajoutée à votre locataire Azure Active Directory (Azure AD).

## <a name="prerequisites"></a>Prérequis

Pour supprimer une application de votre locataire Azure AD, vous avez besoin des éléments suivants :

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un des rôles suivants : Administrateur général, Administrateur d’application cloud, Administrateur d’application ou propriétaire du principal de service.
- Facultatif : effectuer [Voir vos applications](view-applications-portal.md).
- Facultatif : effectuer [Ajouter une application](add-application-portal.md).
- Facultatif : effectuer [Configurer une application](add-application-portal-configure.md).
- Facultatif : suivre [Affecter des utilisateurs à une application](add-application-portal-assign-users.md).
- Facultatif : effectuer [Configurer l’authentification unique](add-application-portal-setup-sso.md).

>[!IMPORTANT]
>Pour tester les étapes de ce guide de démarrage rapide, utilisez un environnement de non-production.

## <a name="delete-an-application-from-your-azure-ad-tenant"></a>Supprimer une application de votre locataire Azure AD

Pour supprimer une application de votre locataire Azure AD :

1. Dans le portail Azure AD, sélectionnez **Applications d’entreprise**. Ensuite, recherchez et sélectionnez l’application que vous souhaitez supprimer. En l’occurrence, nous allons supprimer l’application **GitHub_test** que nous avons ajoutée à l’occasion du guide de démarrage rapide précédent.
1. À la section **Gérer** dans le volet gauche, sélectionnez **Propriétés**.
1. Sélectionnez **Supprimer**, puis sélectionnez **Oui** pour confirmer que vous souhaitez supprimer l’application de votre locataire Azure AD.

> [!TIP]
> Vous pouvez automatiser la gestion des applications à l’aide de l’API Graph. Consultez [Automatiser la gestion des applications avec l’API Microsoft Graph](/graph/application-saml-sso-configure-api).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous aurez terminé cette série de guides de démarrage rapide, envisagez de supprimer l’application pour nettoyer votre locataire de test. La suppression d’application est traitée dans ce guide de démarrage rapide.

## <a name="next-steps"></a>Étapes suivantes

Vous avez terminé la série de guides de démarrage rapide. Pour continuer, apprenez-en davantage sur les bonnes pratiques relatives à la gestion des applications.
> [!div class="nextstepaction"]
> [Bonnes pratiques relatives à la gestion des applications](application-management-fundamentals.md)