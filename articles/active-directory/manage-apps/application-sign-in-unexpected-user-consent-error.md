---
title: Erreur inattendue lors du consentement à une application | Microsoft Docs
description: Aborde les erreurs qui peuvent se produire durant le processus de consentement à une application et ce que vous pouvez faire
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 07/11/2017
ms.author: kenwith
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 558c6dc24f6d0d17c9a82bbc79f39649f63dc7f4
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658483"
---
# <a name="unexpected-error-when-performing-consent-to-an-application"></a>Erreur inattendue lors du consentement à une application

Cet article traite des erreurs qui peuvent se produire durant le processus de consentement à une application. Si vous rencontrez des invites de consentement inattendues qui ne contiennent aucun message d’erreur, consultez [Scénarios d’authentification pour Azure AD](../develop/authentication-vs-authorization.md) pour résoudre le problème.

Pour fonctionner, de nombreuses applications qui s’intègrent à Azure Active Directory nécessitent des autorisations à d’autres ressources. Quand ces ressources sont également intégrées à Azure Active Directory, vous devez souvent utiliser l’infrastructure de consentement commune pour demander les autorisations nécessaires pour y accéder. Une invite de consentement s’affiche, généralement lors de la première utilisation de l’application, mais parfois lors d’une utilisation ultérieure.

Certaines conditions doivent être réunies pour qu’un utilisateur consente aux autorisations exigées par une application. Si ces conditions ne sont pas remplies, les erreurs suivantes peuvent se produire.

## <a name="requesting-not-authorized-permissions-error"></a>Erreur : demande d’autorisations dont l’octroi n’est pas autorisé
* **AADSTS90093 :** &lt;clientAppDisplayName&gt; demande une ou plusieurs autorisations que vous ne pouvez pas accorder. Contactez un administrateur qui peut donner son consentement à cette application en votre nom.
* **AADSTS90094 :** pour accéder aux ressources de votre organisation, &lt;clientAppDisplayName&gt; a besoin d’une autorisation que seul un administrateur peut accorder. Avant d’utiliser cette application, demandez à un administrateur d’accorder une autorisation à celle-ci.

Cette erreur se produit quand un utilisateur, qui n’est pas un administrateur d’entreprise, tente d’utiliser une application qui demande des autorisations pouvant uniquement être accordées par un administrateur. Pour résoudre cette erreur, un administrateur peut accorder l’accès à l’application au nom de son organisation.

Cette erreur peut également se produire quand un utilisateur est empêché de donner son consentement à une application parce que Microsoft a détecté que la demande d’autorisation présente un risque. Dans ce cas, un événement d’audit est également journalisé avec la catégorie « ApplicationManagement », le type d’activité « Consent to application » et la raison de l’état « Risky application detected ».

Cette erreur peut également se produire lorsque l’affectation de l’utilisateur est requise pour l’application, mais qu’aucun consentement de l’administrateur n’a été fourni. Dans ce cas, l’administrateur doit d’abord fournir son consentement.   

## <a name="policy-prevents-granting-permissions-error"></a>Erreur : stratégie empêchant l’octroi d’autorisations
* **AADSTS90093 :** Un administrateur de &lt;tenantDisplayName&gt; a défini une stratégie qui vous empêche d’octroyer à &lt;nom_application&gt; les autorisations qu’elle demande. Contactez un administrateur de &lt;tenantDisplayName&gt;, qui peut accorder des autorisations à cette application en votre nom.

Cette erreur se produit quand un utilisateur non-administrateur tente d’utiliser une application qui nécessite son consentement et que l’administrateur d’entreprise l’en empêche. Pour résoudre cette erreur, un administrateur peut accorder l’accès à l’application au nom de son organisation.

## <a name="intermittent-problem-error"></a>Erreur : problème intermittent
* **AADSTS90090 :** Un problème intermittent s’est apparemment produit au cours du processus de connexion, lors de l’enregistrement des autorisations que vous avez tenté d’accorder à &lt;clientAppDisplayName&gt;. Réessayez plus tard.

Cette erreur indique qu’un problème intermittent côté service s’est produit. Vous pouvez le résoudre en essayant de donner à nouveau votre consentement à l’application.

## <a name="resource-not-available-error"></a>Erreur : ressource non disponible
* **AADSTS65005 :** L’application &lt;clientAppDisplayName&gt; a demandé des autorisations pour accéder à une ressource &lt;resourceAppDisplayName&gt; qui n’est pas disponible. 

Contactez le développeur de l’application.

##  <a name="resource-not-available-in-tenant-error"></a>Erreur : ressource non disponible dans le locataire
* **AADSTS65005 :** &lt;clientAppDisplayName&gt; demande l’accès à une ressource &lt;resourceAppDisplayName&gt; qui n’est pas disponible dans votre organisation &lt;tenantDisplayName&gt;. 

Vérifiez que cette ressource est disponible ou contactez un administrateur de &lt;tenantDisplayName&gt;.

## <a name="permissions-mismatch-error"></a>Erreur : incompatibilité des autorisations
* **AADSTS65005 :** L’application a fait une demande de consentement pour accéder à la ressource &lt;resourceAppDisplayName&gt;. Cette demande a échoué car elle ne correspond pas à la façon dont l’application a été préconfigurée pendant son inscription. Contactez le fournisseur de l’application.**

Ces erreurs se produisent quand l’application à laquelle un utilisateur tente de donner son consentement demande des autorisations pour accéder à une application de ressources introuvable dans l’annuaire (locataire) de l’organisation. Cette situation peut se produire pour plusieurs raisons :

-   Le développeur de l’application cliente a configuré l’application de manière incorrecte, la forçant à demander l’accès à une ressource non valide. Dans ce cas, le développeur de l’application doit mettre à jour la configuration de l’application cliente pour résoudre ce problème.

-   Un principal de service représentant l’application de ressources cible n’existe pas dans l’organisation, ou il a existé mais a été supprimé. Pour résoudre ce problème, un principal de service représentant l’application de ressources doit être approvisionné dans l’organisation pour que l’application cliente puisse demander des autorisations d’accès à ce principal de service. Il existe plusieurs manières de configurer le principal de service selon le type d’application, notamment :

    -   Acquisition d’un abonnement pour l’application de ressources (applications publiées par Microsoft)

    -   Consentement à l’application de ressources

    -   Accorder les autorisations de l’application par le biais du Portail Azure

    -   Ajout de l’application à partir de la galerie d’applications Azure AD

## <a name="risky-app-error-and-warning"></a>Erreur et avertissement : application à risque
* **AADSTS900941 :** Le consentement de l’administrateur est obligatoire. L’application est considérée comme risquée. (AdminConsentRequiredDueToRiskyApp)
* Cette application peut présenter un risque. Si vous faites confiance à cette application, demandez à votre administrateur de vous accorder l’accès.
* **AADSTS900981 :** Une demande de consentement administrateur a été reçue pour une application risquée. (AdminConsentRequestRiskyAppWarning)
* Cette application peut présenter un risque. Continuez uniquement si vous faites confiance à cette application.

Ces deux messages s’affichent lorsque Microsoft a déterminé que la demande de consentement peut présenter un risque. Parmi d’autres facteurs, cela peut se produire si aucun [éditeur vérifié](../develop/publisher-verification-overview.md) n’a été ajouté à l’inscription de l’application. Les utilisateurs finaux verront le premier code d’erreur et le premier message lorsque le [workflow de consentement administrateur](configure-admin-consent-workflow.md) est désactivé. Les administrateurs ainsi que les utilisateurs finaux verront le second code et le second message lorsque le workflow de consentement administrateur est activé. 

Les utilisateurs finaux ne pourront pas donner leur consentement aux applications qui ont été détectées comme présentant un risque. Les administrateurs peuvent le faire, mais ils doivent évaluer l’application avec beaucoup de soin et procéder avec prudence. Si l’application semble suspecte après un examen plus approfondi, elle peut être signalée à Microsoft sur l’écran de consentement. 

## <a name="next-steps"></a>Étapes suivantes 

[Applications, autorisations et consentement dans Azure Active Directory (point de terminaison v1)](../develop/quickstart-register-app.md)<br>

[Étendues, autorisations et consentement dans Azure Active Directory (point de terminaison v2.0)](../develop/v2-permissions-and-consent.md)