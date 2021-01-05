---
title: Accès conditionnel pour les utilisateurs de B2B Collaboration - Azure AD
description: Découvrez comment appliquer des stratégies d’authentification multifacteur pour les utilisateurs d’Azure Active Directory B2B.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 09/11/2017
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: elisolMS
ms.collection: M365-identity-device-management
ms.openlocfilehash: eccbbb22814788aaf06fa6fd10d8c376203c1d49
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92892449"
---
# <a name="conditional-access-for-b2b-collaboration-users"></a>Accès conditionnel pour les utilisateurs de B2B Collaboration

## <a name="multi-factor-authentication-for-b2b-users"></a>Authentification MFA pour utilisateurs B2B
Avec Azure AD B2B Collaboration, les organisations peuvent appliquer des stratégies d’authentification multifacteur (MFA) pour les utilisateurs B2B. Ces stratégies peuvent être appliquées au niveau locataire, application ou utilisateur individuel, de la même façon qu’elles peuvent être activées pour les employés à plein temps et les membres de l’organisation. Les stratégies MFA sont appliquées à l’organisation de ressources.

Exemple :
1. L’administrateur ou le professionnel de l’information de la société A invite des utilisateurs de la société B pour l’application *Foo* dans la société A.
2. L'application *Foo* dans la société A est configurée de manière à exiger l’authentification MFA lors de l’accès.
3. Lorsque des utilisateurs de la société B tentent d’accéder à l’application *Foo* à partir d’un locataire de la société A, ils sont invités à effectuer une authentification MFA.
4. Les utilisateurs peuvent configurer leur authentification MFA avec la société A et choisir leur option d’authentification MFA.
5. Ce scénario fonctionne pour n’importe quelle identité (Azure AD ou MSA, par exemple, si les utilisateurs dans la société B s’authentifient à l’aide de leur ID social)
6. La société A doit avoir suffisamment de licences Azure AD Premium qui prennent en charge l’authentification multifacteur. L’utilisateur de la société B utilise cette licence à partir de la société A.

La location d’invitation est toujours responsable de l’authentification MFA pour les utilisateurs de l’organisation partenaire, même si l’organisation partenaire a des fonctionnalités d’authentification MFA.

### <a name="setting-up-mfa-for-b2b-collaboration-users"></a>Configuration de MFA pour les utilisateurs de B2B Collaboration
Pour découvrir combien il est facile de configurer l’authentification MFA pour les utilisateurs de B2B Collaboration, consultez la vidéo suivante :

>[!VIDEO https://channel9.msdn.com/Blogs/Azure/b2b-conditional-access-setup/Player]

### <a name="b2b-users-mfa-experience-for-offer-redemption"></a>Expérience MFA d'utilisation de l'invitation par des utilisateurs B2B
Regardez l’animation suivante pour découvrir comment utiliser l’invitation :

>[!VIDEO https://channel9.msdn.com/Blogs/Azure/MFA-redemption/Player]

### <a name="mfa-reset-for-b2b-collaboration-users"></a>Réinitialisation de l'authentification MFA pour les utilisateurs de B2B de Collaboration
Actuellement, l’administrateur peut exiger que les utilisateurs B2B Collaboration s’authentifient à nouveau uniquement par le biais des applets de commande PowerShell suivantes :

1. Se connecter à Azure AD

   ```
   $cred = Get-Credential
   Connect-MsolService -Credential $cred
   ```
2. Obtenir tous les utilisateurs avec des méthodes d'authentification

   ```
   Get-MsolUser | where { $_.StrongAuthenticationMethods} | select UserPrincipalName, @{n="Methods";e={($_.StrongAuthenticationMethods).MethodType}}
   ```
   Voici un exemple :

   ```
   Get-MsolUser | where { $_.StrongAuthenticationMethods} | select UserPrincipalName, @{n="Methods";e={($_.StrongAuthenticationMethods).MethodType}}
   ```

3. Réinitialisez la méthode d’authentification multifacteur pour qu’un utilisateur spécifique oblige l’utilisateur B2B Collaboration à définir de nouveau des méthodes d’authentification. Exemple :

   ```
   Reset-MsolStrongAuthenticationMethodByUpn -UserPrincipalName gsamoogle_gmail.com#EXT#@ WoodGroveAzureAD.onmicrosoft.com
   ```

### <a name="why-do-we-perform-mfa-at-the-resource-tenancy"></a>Pourquoi effectuer l’authentification MFA au niveau de la location de la ressource ?

Dans la version actuelle, l’authentification MFA s’effectue toujours dans la location de la ressource, pour des raisons de prévisibilité. Par exemple, un utilisateur de Contoso (Catherine) est invité à Fabrikam, et Fabrikam a activé l’authentification MFA pour les utilisateurs B2B.

Si Contoso utilise la stratégie d’authentification multifacteur sur App1 mais pas sur App2, et si nous examinons la revendication MFA Contoso dans le jeton, nous pourrions constater le problème suivant :

* Jour 1 : Un utilisateur dispose de l’authentification multifacteur dans Contoso et accède à App1, mais aucune invite MFA supplémentaire ne s’affiche dans Fabrikam.

* Jour 2 : L’utilisateur a accédé à App2 dans Contoso, et à présent, lorsqu’il accède à Fabrikam, il doit s’inscrire à l’authentification multifacteur.

Ce processus peut prêter à confusion et entraîner l’abandon de connexion.

En outre, même si Contoso dispose de la fonctionnalité MFA, il n’est pas toujours certain que Fabrikam approuve la stratégie MFA de Contoso.

Enfin, l’authentification MFA du locataire de la ressource fonctionne également pour les MSA et les ID sociaux ainsi que pour les organisations partenaires au sein desquelles l’authentification MFA n’est pas configurée.

Par conséquent, la recommandation d’authentification MFA pour les utilisateurs B2B consiste à toujours demander l’authentification MFA dans le locataire à l’origine de l’invitation. Cette exigence peut entraîner une double authentification multifacteur dans certains cas, mais à chaque accès au locataire à l’origine de l’invitation, l’expérience des utilisateurs finaux est prévisible : Catherine doit s’inscrire à MFA avec le locataire à l’origine de l’invitation.

### <a name="device-based-location-based-and-risk-based-conditional-access-for-b2b-users"></a>Accès conditionnel en fonction des appareils, des emplacements et des risques pour les utilisateurs B2B

Lorsque Contoso active les stratégies d’accès conditionnel en fonction des appareils pour ses données d’entreprise, l’accès est protégé contre les appareils non gérés par Contoso et non conformes aux stratégies d’appareils de Contoso.

Si l’appareil de l’utilisateur B2B n’est pas géré par Contoso, l’accès des utilisateurs B2B des organisations partenaires est bloqué quel que soit le contexte d’application de ces stratégies. Cependant, Contoso peut créer des listes d’exclusions contenant des utilisateurs partenaires spécifiques afin de les exclure de la stratégie d’accès conditionnel en fonction des appareils.

#### <a name="mobile-application-management-policies-for-b2b"></a>Stratégies de gestion des applications mobiles pour B2B

Les stratégies de protection des applications à accès conditionnel ne peuvent pas être appliquées aux utilisateurs B2B, car l’organisation qui lance l’invitation n’a aucune visibilité sur l’organisation de l’utilisateur B2B.

#### <a name="location-based-conditional-access-for-b2b"></a>Accès conditionnel en fonction des emplacements pour B2B

Les stratégies d’accès conditionnel en fonction des emplacements peuvent être appliquées pour les utilisateurs B2B si l’organisation à l’origine de l’invitation est en mesure de créer une plage d’adresses IP approuvée qui définit leurs organisations partenaires.

#### <a name="risk-based-conditional-access-for-b2b"></a>Accès conditionnel en fonction des risques pour B2B

Actuellement, les stratégies de connexion en fonction des risques ne peuvent pas être appliquées aux utilisateurs B2B, car l’évaluation des risques s’effectue au niveau de l’organisation d’origine de l’utilisateur B2B.

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants sur Azure AD B2B Collaboration :

* [Qu'est-ce que la collaboration B2B d'Azure AD ?](what-is-b2b.md)
* [Prix des identités externes](external-identities-pricing.md)
* [Questions fréquemment posées (FAQ) sur Azure Active Directory B2B Collaboration](faq.md)
