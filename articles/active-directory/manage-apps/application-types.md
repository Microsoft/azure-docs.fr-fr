---
title: Affichage des applications qui utilisent votre locataire Azure Active Directory pour la gestion des identités
description: Découvrez comment afficher toutes les applications qui utilisent votre locataire Azure Active Directory pour la gestion des identités
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 07/11/2017
ms.author: kenwith
ms.openlocfilehash: 49dd8b65db771a182138f13218adda2f0e7f7396
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96858227"
---
# <a name="viewing-apps-using-your-azure-ad-tenant-for-identity-management"></a>Affichage des applications qui utilisent votre locataire Azure AD pour la gestion des identités
La [série de guides de démarrage rapide sur la gestion des applications](view-applications-portal.md) vous guide parmi les concepts de base. Vous y apprenez comment afficher toutes les applications qui utilisent votre locataire Azure AD pour la gestion des identités. Cet article aborde plus en détail les types d’applications que vous trouverez.

## <a name="why-does-a-specific-application-appear-in-my-all-applications-list"></a>Pourquoi une application spécifique apparaît-elle dans ma liste de toutes les applications ?
Lors du filtrage avec **Toutes les applications**, la **liste** **Toutes les applications** affiche chaque objet Principal de service de votre locataire. Les objets Principal de service peuvent apparaître dans cette liste de différentes façons :
- Lorsque vous ajoutez une application à partir de la galerie d’applications, y compris :
   - **Azure AD – Applications d’entreprise** – Applications ajoutées à votre locataire à l’aide de l’option **Applications d’entreprise** sur le portail Azure AD. Applications généralement intégrées à l’aide de la norme SAML.
   - **Azure AD – Inscriptions d’applications** – Applications ajoutées à votre locataire à l’aide de l’option **Inscriptions d’applications** sur le portail Azure AD. En général, les applications personnalisées développées à l’aide des normes Open ID Connect et OAuth.
   - **Applications du proxy d’application** : applications s’exécutant dans votre environnement local, pour lesquelles vous souhaitez utiliser l’authentification unique en externe.
- Lors de l’inscription ou de la connexion, une application tierce intégrée à Azure Active Directory. Exemple : [Smartsheet](https://app.smartsheet.com/b/home) ou [DocuSign](https://www.docusign.net/member/MemberLogin.aspx).
- Applications Microsoft telles que Microsoft 365.
- Lorsque vous ajoutez une inscription pour une nouvelle application en créant une application personnalisée à l’aide du [Registre d’application](../develop/quickstart-register-app.md).
- Lorsque vous ajoutez une inscription pour une nouvelle application en créant une application personnalisée à l’aide du [Portail d’inscription des applications V2.0](../develop/quickstart-register-app.md).
- Lorsque vous ajoutez une application que vous développez à l’aide des [méthodes d’authentification ASP.NET](https://www.asp.net/visual-studio/overview/2013/creating-web-projects-in-visual-studio#orgauthoptions) ou des [services connectés](https://devblogs.microsoft.com/visualstudio/connecting-to-cloud-services/) de Visual Studio.
- Lorsque vous créez un objet Principal de service à l’aide du [module Azure AD PowerShell](/powershell/azure/active-directory/install-adv2).
- Lorsque vous [donnez votre consentement à une application](../develop/howto-convert-app-to-be-multi-tenant.md) en tant qu’administrateur concernant l’utilisation des données de votre locataire.
- Lorsqu’un [utilisateur donne son consentement à une application](../develop/howto-convert-app-to-be-multi-tenant.md) concernant l’utilisation des données de votre locataire.
- Lorsque vous activez certains services qui stockent des données dans votre client. Un exemple correspond à l’opération de réinitialisation du mot de passe, qui est modélisée en tant que principal de service pour stocker de façon sécurisée votre stratégie de réinitialisation du mot de passe.

Pour plus d’informations sur la façon dont les applications sont ajoutées à votre annuaire et sur les raisons de cela, lisez [Comment et pourquoi les applications sont ajoutées à Azure AD](../develop/active-directory-how-applications-are-added.md).

## <a name="next-steps"></a>Étapes suivantes
[Gestion des applications avec Azure Active Directory](what-is-application-management.md)
