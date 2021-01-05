---
title: Présentation d’Azure Active Directory pour les développeurs (v1.0)
description: Cet article fournit une vue d’ensemble de la connexion des comptes professionnels et scolaires Microsoft à l’aide de la plateforme et du point de terminaison Azure Active Directory v1.0.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.topic: overview
ms.workload: identity
ms.date: 10/24/2018
ms.author: ryanwi
ms.reviewer: jmprieur
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: 8cb426239fe9f3dea207df06b6f2bafbdb54058a
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88117666"
---
# <a name="azure-active-directory-for-developers-v10-overview"></a>Présentation d’Azure Active Directory pour les développeurs (v1.0)

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

Azure Active Directory (Azure AD) est un service d’identité cloud qui permet aux développeurs de créer des applications qui connectent de façon sécurisée les utilisateurs disposant d’un compte Microsoft professionnel ou scolaire. Azure AD prend en charge les développeurs qui créent des applications à locataire unique et métiers, ainsi que les développeurs qui souhaitent développer des applications multi-locataires. En plus de la connexion basique, Azure AD permet également aux applications d’appeler des API Microsoft comme [Microsoft Graph](/graph/overview) et des API personnalisées reposant sur la plateforme Azure AD. Cette documentation explique comment ajouter une prise en charge Azure AD à votre application par le biais de protocoles standard du secteur, comme OAuth 2.0 et OpenID Connect.

> [!NOTE]
> Le contenu de cette page traite en grande partie de la plateforme et du point de terminaison v1.0, qui prennent seulement en charge les comptes Microsoft professionnels ou scolaires. Si vous voulez vous connecter à des comptes Microsoft consommateurs ou personnels, consultez les informations sur la [plateforme et le point de terminaison v2.0](../develop/v2-overview.md). Le point de terminaison v2.0 offre une expérience de développement unifiée pour les applications qui souhaitent connecter toutes les identités Microsoft.

- [Principes fondamentaux de l’authentification](v1-authentication-scenarios.md) Introduction à l’authentification avec Azure AD.
- [Types d’applications](app-types.md) Vue d’ensemble des scénarios d’authentification pris en charge par Azure AD.

## <a name="get-started"></a>Bien démarrer

Les didacticiels et démarrages rapides dédiés à la version v1.0 vous guident pas à pas dans les différentes étapes de création d’une application sur votre plateforme préférée à l’aide du Kit de développement logiciel (SDK) de la Bibliothèque d’authentification Active Directory (ADAL). Consultez les **démarrages rapides dédiés à la version v1.0** et les **didacticiels dédiés à la version v1.0** dans [Plateforme d’identités Microsoft (Azure Active Directory pour développeurs)](index.yml) pour commencer.

## <a name="how-to-guides"></a>Guides pratiques

Consultez les **guides pratiques dédiés à la version v1.0**  pour obtenir des informations détaillées et des procédures à suivre pour réaliser les tâches les plus courantes dans Azure AD.

## <a name="reference-topics"></a>Rubriques de référence

Les articles suivants fournissent des informations détaillées sur les API, les messages de protocole et les termes utilisés dans Azure AD.

- [Bibliothèques d’authentification](active-directory-authentication-libraries.md) Vue d’ensemble des bibliothèques et des kits SDK fournis par Azure AD.
- [Exemples de code](sample-v1-code.md) Liste de tous les exemples de code Azure AD.
- [Glossaire](../develop/developer-glossary.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json) Terminologie et définitions des termes utilisés dans cette documentation.

## <a name="videos"></a>Vidéos

Consultez [Vidéos sur la plateforme développeur d’Azure Active Directory](videos.md) pour obtenir de l’aide sur la migration vers la nouvelle plateforme d’identité Microsoft.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]