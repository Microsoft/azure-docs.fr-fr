---
title: Activer l’approvisionnement automatique d’utilisateurs pour les applications multilocataires – Azure AD
description: Guide d'activation de l'approvisionnement automatisé destiné aux éditeurs de logiciels indépendants
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: reference
ms.workload: identity
ms.date: 07/23/2019
ms.author: kenwith
ms.reviewer: zhchia
ms.openlocfilehash: d4c6c5b51e00f7f2ed7e29c27aa8e8bb88a2ea94
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861355"
---
# <a name="enable-automatic-user-provisioning-for-your-multi-tenant-application"></a>Activer l'approvisionnement automatique d'utilisateurs pour votre application multilocataire

Le processus d'approvisionnement automatique d'utilisateurs consiste à automatiser la création, la gestion et la suppression des identités utilisateurs dans des systèmes cibles tels que vos applications SaaS.

## <a name="why-enable-automatic-user-provisioning"></a>Pourquoi activer l'approvisionnement automatique d'utilisateurs ?

Les applications nécessitant la présence d'un enregistrement utilisateur dans l'application avant la première connexion d'un utilisateur requièrent un approvisionnement d'utilisateurs. Celui-ci présente des avantages pour vous, en tant que fournisseur de services, ainsi que pour vos clients.

### <a name="benefits-to-you-as-the-service-provider"></a>Avantages pour vous en tant que fournisseur de services

* Renforcez la sécurité de votre application en utilisant la Plateforme d'identités Microsoft.

* Réduisez les efforts réels et perçus des clients qui adoptent votre application.

* Réduisez vos coûts d'intégration à plusieurs fournisseurs d'identité (IdP) pour l'approvisionnement automatique d'utilisateurs en ayant recours à l'approvisionnement SCIM (Cross-Domain Identity Management).

* Réduisez les coûts liés au support en fournissant des journaux enrichis pour aider les clients à résoudre les problèmes d'approvisionnement des utilisateurs.

* Améliorez la visibilité de votre application dans la [galerie d'applications Azure AD](https://azuremarketplace.microsoft.com/marketplace/apps).

* Bénéficiez d'une liste hiérarchisée sur la page Didacticiels des applications.

### <a name="benefits-to-your-customers"></a>Avantages pour vos clients

* Renforcez la sécurité en supprimant automatiquement l'accès à votre application pour les utilisateurs qui changent de rôle ou quittent l'organisation.

* Simplifiez la gestion des utilisateurs de votre application en évitant les erreurs humaines et les tâches répétitives associées à l'approvisionnement manuel.

* Réduisez les coûts d'hébergement et de gestion des solutions d'approvisionnement personnalisées.

## <a name="choose-a-provisioning-method"></a>Choisir une méthode d'approvisionnement

Azure AD propose différents chemins d'intégration afin d'activer l'approvisionnement automatique d'utilisateurs pour votre application.

* Le [service d'approvisionnement Azure AD](../app-provisioning/user-provisioning.md) gère l'approvisionnement et le déprovisionnement d'utilisateurs d'Azure AD vers votre application (approvisionnement sortant) et de votre application vers Azure AD (approvisionnement entrant). Le service se connecte aux points de terminaison de l'API de gestion des utilisateurs SCIM (System for Inter-Domain Identity Management) fournis par votre application.

* Lorsque vous utilisez [Microsoft Graph](/graph/), votre application gère l'approvisionnement entrant et sortant des utilisateurs et des groupes d'Azure AD vers votre application en interrogeant l'API Microsoft Graph.

* L'approvisionnement d'utilisateurs SAML JIT (Security Assertion Markup Language Just in Time) peut être activé si votre application utilise SAML pour la fédération. Les informations de revendication envoyées dans le jeton SAML sont utilisées pour approvisionner les utilisateurs.

Pour savoir quelle option d'intégration utiliser pour votre application, reportez-vous au tableau comparatif général, puis consultez les informations plus détaillées sur chaque option.

| Fonctionnalités activées ou améliorées par l'approvisionnement automatique| Service d'approvisionnement Azure AD (SCIM 2.0)| API Microsoft Graph (OData v4.0)| SAML JIT |
|---|---|---|---|
| Gestion des utilisateurs et des groupes dans Azure AD| √| √| Utilisateur uniquement |
| Gérer les utilisateurs et les groupes synchronisés à partir de l'instance locale d'Active Directory| √*| √*| Utilisateur uniquement* |
| Accéder à des données autres que celles des utilisateurs et des groupes lors de l'approvisionnement Accéder aux données Microsoft 365 (Teams, SharePoint, Courrier, Calendrier, Documents, etc.)| X+| √| X |
| Créer, lire et mettre à jour les utilisateurs en fonction de règles d'entreprise| √| √| √ |
| Supprimer les utilisateurs en fonction de règles d'entreprise| √| √| X |
| Gérer l'approvisionnement automatique d'utilisateurs de toutes les applications à partir du Portail Azure| √| X| √ |
| Prendre en charge plusieurs fournisseurs d'identité| √| X| √ |
| Prendre en charge des comptes Invité (B2B)| √| √| √ |
| Prise en charge des comptes qui n'appartiennent pas à l'entreprise (B2C)| X| √| √ |

<sup>*</sup> - Azure AD Connect doit être installé pour synchroniser les utilisateurs d'AD vers Azure AD.  
<sup>+</sup > - L'utilisation de SCIM pour l'approvisionnement ne vous empêche pas d'intégrer votre application à Microsoft Graph à d'autres fins.

## <a name="azure-ad-provisioning-service-scim"></a>Service de provisionnement Azure AD (SCIM)

Les services de provisionnement Azure AD utilisent [SCIM](https://aka.ms/SCIMOverview), une norme industrielle pour le provisionnement qui est prise en charge par de nombreux fournisseurs d’identité (IdP) et de nombreuses applications (comme Slack, G Suite, Dropbox). Nous vous recommandons d'utiliser le service d'approvisionnement Azure AD si vous souhaitez prendre en charge des IdP en plus d'Azure AD, car tout IdP compatible avec SCIM peut se connecter à votre point de terminaison SCIM. En générant un simple point de terminaison /User, vous pouvez activer le provisionnement sans avoir à gérer votre propre moteur de synchronisation. 

Pour plus d'informations sur le service d'approvisionnement Azure AD et SCIM, consultez : 

* [En savoir plus sur la norme SCIM](https://aka.ms/SCIMOverview)

* [Utilisation du protocole SCIM (System for Cross-Domain Identity Management) pour configurer automatiquement des utilisateurs et groupes d'Azure Active Directory dans des applications](../app-provisioning/use-scim-to-provision-users-and-groups.md)

* [Comprendre l'implémentation de SCIM d'Azure AD](../app-provisioning/use-scim-to-provision-users-and-groups.md)

## <a name="microsoft-graph-for-provisioning"></a>Microsoft Graph pour l'approvisionnement

Lorsque vous utilisez Microsoft Graph pour l'approvisionnement, vous avez accès à toutes les données utilisateur enrichies disponibles dans Graph. Outre les détails relatifs aux utilisateurs et aux groupes, vous pouvez également récupérer des informations supplémentaires telles que les rôles des utilisateurs, leurs supérieurs et collaborateurs directs, les appareils possédés et enregistrés, et des centaines d'autres données disponibles dans [Microsoft Graph](/graph/api/overview). 

Plus de 15 millions d'organisations et 90 % des entreprises figurant dans le classement Fortune 500 utilisent Azure AD tout en s'abonnant à des services cloud Microsoft tels que Microsoft 365, Microsoft Azure ou Enterprise Mobility Suite. Vous pouvez utiliser Microsoft Graph pour intégrer votre application à des flux de travail administratifs, tels que l'intégration (et le licenciement) des employés, la maintenance des profils, etc. 

Apprenez-en plus sur l'utilisation de Microsoft Graph pour l'approvisionnement :

* [Page d'accueil de Microsoft Graph](https://developer.microsoft.com/graph)

* [Overview of Microsoft Graph (Vue d’ensemble de Microsoft Graph)](/graph/overview)

* [Présentation de l'authentification de Microsoft Graph](/graph/auth/)

* [Prise en main de Microsoft Graph](https://developer.microsoft.com/graph/get-started)

## <a name="using-saml-jit-for-provisioning"></a>Utilisation de SAML JIT pour l'approvisionnement

Si vous ne souhaitez approvisionner les utilisateurs que lors de la première connexion à votre application et que vous n'avez pas besoin de les déprovisionner automatiquement, SAML JIT est une option. Votre application doit prendre en charge SAML 2.0 comme protocole de fédération pour utiliser SAML JIT.

SAML JIT utilise les informations de revendication contenues dans le jeton SAML pour créer et mettre à jour les informations utilisateur dans l'application. Les clients peuvent configurer ces revendications requises dans l'application Azure AD selon leurs besoins. Parfois, l'approvisionnement JIT doit être activé du côté de l'application pour que le client puisse utiliser cette fonctionnalité. SAML JIT permet de créer et de mettre à jour des utilisateurs, mais pas de supprimer ou de désactiver ceux-ci dans l'application.

## <a name="next-steps"></a>Étapes suivantes

* [Activer l'authentification unique pour votre application](../develop/v2-howto-app-gallery-listing.md)

* [Soumettez votre liste d'applications](https://microsoft.sharepoint.com/teams/apponboarding/Apps/SitePages/Default.aspx) et établissez un partenariat avec Microsoft pour créer de la documentation sur le site de Microsoft.

* [Rejoignez le Microsoft Partner Network (gratuit) et élaborez votre plan de commercialisation](https://partner.microsoft.com/explore/commercial).
