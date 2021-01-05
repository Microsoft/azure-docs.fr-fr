---
title: Obtenir le consentement de plusieurs ressources (MSAL.NET) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment un utilisateur peut obtenir le consentement préalable pour plusieurs ressources à l’aide de la bibliothèque d’authentification Microsoft pour .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 04/30/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 6333d935e1a902ba173017f8149c098f44398955
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88165870"
---
# <a name="user-gets-consent-for-several-resources-using-msalnet"></a>L’utilisateur obtient le consentement pour plusieurs ressources à l’aide de MSAL.NET
Le point de terminaison de la plateforme d’identités Microsoft ne vous permet pas d’obtenir un jeton pour plusieurs ressources à la fois. Lorsque vous utilisez la bibliothèque d’authentification Microsoft pour .NET (MSAL.NET), le paramètre d’étendue dans la méthode d’acquisition de jetons doit contenir uniquement les étendues pour une seule ressource. Toutefois, vous pouvez accorder un consentement préalable à plusieurs ressources en spécifiant les étendues supplémentaires à l’aide de la méthode de générateur `.WithExtraScopeToConsent`.

> [!NOTE]
> L’obtention d’un consentement pour plusieurs ressources fonctionne pour la plateforme d’identités Microsoft, mais pas pour Azure AD B2C. Azure AD B2C prend en charge le consentement de l’administrateur uniquement, et pas le consentement de l’utilisateur.

Par exemple, si vous disposez de deux ressources ayant deux étendues chacune :

- https:\//mytenant.onmicrosoft.com/customerapi (avec 2 étendues `customer.read` et `customer.write`)
- https:\//mytenant.onmicrosoft.com/vendorapi (avec 2 étendues `vendor.read` et `vendor.write`)

Vous devez utiliser le modificateur `.WithExtraScopeToConsent` qui comporte le paramètre *extraScopesToConsent* comme indiqué dans l’exemple suivant :

```csharp
string[] scopesForCustomerApi = new string[]
{
  "https://mytenant.onmicrosoft.com/customerapi/customer.read",
  "https://mytenant.onmicrosoft.com/customerapi/customer.write"
};
string[] scopesForVendorApi = new string[]
{
 "https://mytenant.onmicrosoft.com/vendorapi/vendor.read",
 "https://mytenant.onmicrosoft.com/vendorapi/vendor.write"
};

var accounts = await app.GetAccountsAsync();
var result = await app.AcquireTokenInteractive(scopesForCustomerApi)
                     .WithAccount(accounts.FirstOrDefault())
                     .WithExtraScopeToConsent(scopesForVendorApi)
                     .ExecuteAsync();
```

Cela vous permet d’obtenir un jeton d’accès pour la première API web. Ensuite, lorsque vous avez besoin d’accéder à la deuxième API web, vous pouvez acquérir silencieusement le jeton à partir du cache du jeton :

```csharp
AcquireTokenSilent(scopesForVendorApi, accounts.FirstOrDefault()).ExecuteAsync();
```
