---
title: Changer les valeurs par défaut de durée de vie des jetons pour les applications Azure AD personnalisées
description: Comment mettre à jour les stratégies de durée de vie des jetons pour l’application que vous développez sur Azure AD
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 10/23/2020
ms.author: ryanwi
ms.custom: aaddev, seoapril2019
ms.openlocfilehash: 41dd1ae62e67fe7a23ad62927e1a67989d684bdb
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516475"
---
# <a name="how-to-change-the-token-lifetime-defaults-for-a-custom-developed-application"></a>Modifier les valeurs par défaut de la durée de vie des jetons pour une application personnalisée

Cet article explique comment utiliser Azure AD PowerShell pour définir une stratégie de durée de vie des jetons d’accès. Azure AD Premium permet aux développeurs d’applications et aux administrateurs de locataires de configurer la durée de vie des jetons émis pour les clients non confidentiels. Les stratégies de durée de vie des jetons sont définies à l’échelle d’un locataire ou en fonction des ressources accessibles.

> [!IMPORTANT]
> Après mai 2020, les locataires ne seront plus en mesure de configurer la durée de vie des jetons d’actualisation et de session.  Azure Active Directory cessera d’honorer la configuration actuelle des jetons d’actualisation et de session dans les stratégies après le 30 janvier 2021. Vous pourrez toujours configurer la durée de vie des jetons d'accès après la dépréciation. Pour plus d’informations, consultez [Durées de vie des jetons configurables dans Azure Active Directory](./active-directory-configurable-token-lifetimes.md).
> Nous avons implémenté des  [capacités de gestion des sessions d’authentification](../conditional-access/howto-conditional-access-session-lifetime.md) dans l’accès conditionnel Azure AD. Vous pouvez utiliser cette nouvelle fonctionnalité pour configurer les durées de vie des jetons d’actualisation en définissant la fréquence de connexion.  

Pour définir une stratégie de durée de vie des jetons d’accès, vous devez télécharger le [module Azure AD PowerShell](https://www.powershellgallery.com/packages/AzureADPreview).
Exécutez la commande **Connect-AzureAD -Confirm**.

Voici un exemple de stratégie qui oblige les utilisateurs à s’authentifier plus fréquemment dans votre application web. Cette stratégie définit la durée de vie de l’accès au principal du service de votre application web. Créez la stratégie et attribuez-la à votre principal de service. Vous devez également obtenir l’ObjectId de votre principal du service.

```powershell
$policy = New-AzureADPolicy -Definition @('{"TokenLifetimePolicy":{"Version":1,"AccessTokenLifetime":"02:00:00"}}') -DisplayName "WebPolicyScenario" -IsOrganizationDefault $false -Type "TokenLifetimePolicy"

$sp = Get-AzureADServicePrincipal -Filter "DisplayName eq '<service principal display name>'"

Add-AzureADServicePrincipalPolicy -Id $sp.ObjectId -RefObjectId $policy.Id
```

## <a name="next-steps"></a>Étapes suivantes

* Consultez l’article [Configurable token lifetimes in Azure AD](./active-directory-configurable-token-lifetimes.md) (Durées de vie configurables des jetons dans Azure AD) pour découvrir comment configurer les durées de vie des jetons émis par Azure AD, notamment comment définir les durées de vie des jetons pour toutes les applications de votre organisation, pour une application mutualisée ou pour un principal de service spécifique de votre organisation. 
* [Référence sur les jetons Azure AD](./id-tokens.md)
