---
title: FAQ sur l’accès conditionnel Azure Active Directory | Microsoft Docs
description: Trouvez les réponses aux questions les plus fréquentes sur l’accès conditionnel dans Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: troubleshooting
ms.date: 10/16/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: 8043e85db1ee1aadc814e98db12cab10ec17e129
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92145236"
---
# <a name="azure-active-directory-conditional-access-faqs"></a>Forum aux questions sur l’accès conditionnel dans Azure Active Directory

## <a name="which-applications-work-with-conditional-access-policies"></a>À quelles applications les stratégies d’accès conditionnel s’appliquent-elles ?

Pour en savoir plus sur les applications auxquelles s’appliquent les stratégies d’accès conditionnel, voir [Applications et navigateurs qui utilisent des règles d’accès conditionnel dans Azure Active Directory](concept-conditional-access-cloud-apps.md).

## <a name="are-conditional-access-policies-enforced-for-b2b-collaboration-and-guest-users"></a>Les stratégies d’accès conditionnel s’appliquent-elles à la collaboration B2B et aux utilisateurs invités ?

Les stratégies sont appliquées aux utilisateurs dans le cadre d’une collaboration business-to-business (B2B). Toutefois, dans certains cas, un utilisateur peut ne pas être en mesure de satisfaire aux exigences de la stratégie. Par exemple, il est possible que l’organisation d’un utilisateur invité ne prenne pas en charge l’authentification multifacteur. 

## <a name="does-a-sharepoint-online-policy-also-apply-to-onedrive-for-business"></a>La stratégie SharePoint Online s’applique-t-elle également à OneDrive Entreprise ?

Oui. Une stratégie SharePoint Online s’applique également à OneDrive Entreprise. Pour plus d’informations, consultez l’article [Dépendances du service d’accès conditionnel](service-dependencies.md) et envisagez de cibler les stratégies sur l’[application Office 365](concept-conditional-access-cloud-apps.md#office-365) à la place.

## <a name="why-cant-i-set-a-policy-directly-on-client-apps-like-word-or-outlook"></a>Pourquoi ne peut-on pas définir directement une stratégie sur des applications clientes, par exemple Word ou Outlook ?

Une stratégie d’accès conditionnel définit les conditions requises pour accéder à un service. Elle est appliquée quand l’authentification à ce service a lieu. La stratégie n’est pas définie directement sur une application cliente. Au lieu de cela, elle est appliquée quand un client appelle un service. Par exemple, une stratégie définie sur SharePoint s’applique aux clients qui appellent SharePoint. Une stratégie définie sur Exchange s’applique à Outlook. Pour plus d’informations, consultez l’article [Dépendances du service d’accès conditionnel](service-dependencies.md) et envisagez de cibler les stratégies sur l’[application Office 365](concept-conditional-access-cloud-apps.md#office-365) à la place.

## <a name="does-a-conditional-access-policy-apply-to-service-accounts"></a>Une stratégie d’accès conditionnel s’applique-t-elle aux comptes de service ?

Les stratégies d’accès conditionnel s’appliquent à tous les comptes d’utilisateur. Cela inclut les comptes d’utilisateur utilisés comme comptes de service. Souvent, un compte de service qui s’exécute sans assistance ne peut pas répondre aux exigences d’une stratégie d’accès conditionnel. Par exemple, l’authentification multifacteur peut être obligatoire. Les comptes de service peuvent être exclus d’une stratégie à l’aide d’une [exclusion d’utilisateurs ou de groupes](concept-conditional-access-users-groups.md#exclude-users). 

## <a name="what-is-the-default-exclusion-policy-for-unsupported-device-platforms"></a>Quelle est la stratégie d’exclusion par défaut pour les plateformes d’appareils non prises en charge ?

À l’heure actuelle, les stratégies d’accès conditionnel sont appliquées de manière sélective aux utilisateurs d’appareils iOS et Android. Les applications se trouvant sur d’autres plateformes d’appareils ne sont pas, par défaut, affectées par la stratégie d’accès conditionnel pour appareils iOS et Android. Un administrateur de locataires peut choisir de remplacer la stratégie globale pour interdire l’accès aux utilisateurs sur des plateformes non prises en charge.

## <a name="how-do-conditional-access-policies-work-for-microsoft-teams"></a>Comment les stratégies d’accès conditionnel fonctionnent-elles pour Microsoft Teams ?

Microsoft Teams s’appuie fortement sur Exchange Online et SharePoint Online pour les principaux scénarios de productivité, tels que les réunions, les calendriers et le partage de fichiers. Les stratégies d’accès conditionnel définies pour ces applications cloud s’appliquent à Microsoft Teams lorsqu’un utilisateur s’y connecte directement.

Le logiciel Microsoft Teams est également pris en charge séparément, en tant qu’application cloud dans les stratégies d’accès conditionnel. Les stratégies d’accès conditionnel définies pour une application cloud s’appliquent à Microsoft Teams lorsqu’un utilisateur se connecte. Toutefois, même sans que les stratégies appropriées ne soient appliquées dans les autres applications comme Exchange Online et SharePoint Online, les utilisateurs peuvent toujours être en mesure d’accéder directement à ces ressources.

Les clients de bureau Microsoft Teams pour Windows et Mac prennent en charge l’authentification moderne. L’authentification moderne permet d’utiliser la connexion basée sur la bibliothèque ADAL (Azure Active Directory Authentication Library) pour les applications clientes Microsoft Office sur plusieurs plateformes.

Pour plus d’informations, consultez l’article [Dépendances du service d’accès conditionnel](service-dependencies.md) et envisagez de cibler les stratégies sur l’[application Office 365](concept-conditional-access-cloud-apps.md#office-365) à la place.

## <a name="why-are-some-tabs-not-working-in-microsoft-teams-after-enabling-conditional-access-policies"></a>Pourquoi certains onglets ne fonctionnent-ils pas dans Microsoft Teams après l’activation des stratégies d’accès conditionnel ?

Après l’activation de certaines stratégies d’accès conditionnel sur le locataire dans Microsoft Teams, certains onglets peuvent ne plus fonctionner comme prévu dans le client de bureau. Toutefois, les onglets affectés fonctionnent lors de l’utilisation du client web Microsoft Teams. Les onglets affectés peuvent inclure Power BI, Formulaires, VSTS, PowerApps et Liste SharePoint.

Pour afficher les onglets affectés, vous devez utiliser le client web Teams dans Edge, Internet Explorer ou Chrome avec l’extension Comptes Windows 10 installée. Certains onglets dépendent de l’authentification web, qui ne fonctionne pas dans le client de bureau Microsoft Teams lorsque l’accès conditionnel est activé. Microsoft collabore avec des partenaires pour activer ces scénarios. À ce jour, nous avons activé des scénarios impliquant Planner, OneNote et Stream.

## <a name="next-steps"></a>Étapes suivantes

- Pour configurer des stratégies d’accès conditionnel pour votre environnement, consultez l’article [Planifier un déploiement d’accès conditionnel](plan-conditional-access.md). 
