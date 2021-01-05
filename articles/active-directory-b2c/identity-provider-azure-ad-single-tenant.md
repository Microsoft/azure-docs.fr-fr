---
title: Configurer la connexion pour une organisation Azure AD
titleSuffix: Azure AD B2C
description: Configurez la connexion pour une organisation Azure Active Directory spécifique dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/20/2020
ms.author: mimart
ms.subservice: B2C
ms.custom: fasttrack-edit
ms.openlocfilehash: a079cfe155119a6afe8575767dd3e7c09a564f9a
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96445632"
---
# <a name="set-up-sign-in-for-a-specific-azure-active-directory-organization-in-azure-active-directory-b2c"></a>Configurer la connexion pour une organisation Azure Active Directory spécifique dans Azure Active Directory B2C

Pour utiliser Azure Active Directory (Azure AD) en tant que [fournisseur d’identité](authorization-code-flow.md) dans Azure AD B2C, vous devez créer une application qui la représente. Cet article explique comment autoriser la connexion d’utilisateurs à partir d’une organisation Azure AD spécifique en utilisant un flux utilisateur dans Azure AD B2C.

[!INCLUDE [active-directory-b2c-identity-provider-azure-ad](../../includes/active-directory-b2c-identity-provider-azure-ad.md)]

## <a name="configure-azure-ad-as-an-identity-provider"></a>Configurer Azure AD en tant que fournisseur d’identité

1. Veillez à bien utiliser le répertoire qui contient le locataire Azure AD B2C. Sélectionnez le filtre **Annuaire + abonnement** dans le menu du haut, puis choisissez l’annuaire qui contient votre locataire Azure AD B2C.
1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Fournisseurs d’identité**, puis **Nouveau fournisseur OpenID Connect**.
1. Saisissez un **Nom**. Par exemple, entrez *Contoso Azure AD*.
1. Dans **URL des métadonnées**, entrez l’URL suivante en remplaçant `{tenant}` par le nom de domaine de votre locataire Azure AD :

    ```
    https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration
    ```

    Par exemple : `https://login.microsoftonline.com/contoso.onmicrosoft.com/v2.0/.well-known/openid-configuration`.
    Par exemple : `https://login.microsoftonline.com/contoso.com/v2.0/.well-known/openid-configuration`.

1. Dans **ID client**, entrez l’ID d’application que vous avez enregistré précédemment.
1. Dans **Clé secrète client**, entrez la clé secrète client que vous avez enregistrée précédemment.
1. Pour l'**Étendue**, entrez `openid profile`.
1. Conservez les valeurs par défaut pour **Type de réponse** et **Mode de réponse**.
1. (Facultatif) Pour l'**Indication de domaine**, entrez `contoso.com`. Pour plus d’informations, consultez [Configurer la connexion directe avec Azure Active Directory B2C](direct-signin.md#redirect-sign-in-to-a-social-provider).
1. Sous **Mappage des revendications du fournisseur d’identité**, sélectionnez les revendications suivantes :

    * **ID d’utilisateur** : *oid*
    * **Nom d’affichage** : *name*
    * **Prénom** : *given_name*
    * **Nom** : *family_name*
    * **Adresse e-mail** : *preferred_username*

1. Sélectionnez **Enregistrer**.
