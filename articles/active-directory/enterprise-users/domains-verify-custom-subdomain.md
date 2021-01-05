---
title: Modifier le type d’authentification du sous-domaine à l’aide de PowerShell et de Graph – Azure Active Directory | Microsoft Docs
description: Modifier les paramètres d’authentification par défaut du sous-domaine hérités des paramètres du domaine racine dans Azure Active Directory.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 11/15/2020
ms.author: curtand
ms.reviewer: sumitp
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 734e6824f13e62ad080500eff18c4892e1f76807
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95503677"
---
# <a name="change-subdomain-authentication-type-in-azure-active-directory"></a>Modifier le type d’authentification du sous-domaine dans Azure Active Directory

Après l’ajout d’un domaine racine à Azure Active Directory (Azure AD), tous les sous-domaines ultérieurs ajoutés à cette racine dans votre organisation Azure AD héritent automatiquement du paramètre d’authentification du domaine racine. Toutefois, si vous souhaitez gérer les paramètres d’authentification du domaine indépendamment des paramètres du domaine racine, vous pouvez désormais utiliser l’API Microsoft Graph. Par exemple, si vous avez un domaine racine fédéré, tel que contoso.com, cet article peut vous aider à vérifier qu’un sous-domaine tel que child.contoso.com est géré au lieu d’être fédéré.

Dans le portail Azure AD, lorsque le domaine parent est fédéré et que l’administrateur tente de vérifier un sous-domaine géré sur la page **Noms de domaines personnalisés**, vous obtenez l’erreur « Échec de l’ajout du domaine » avec la raison « Une ou plusieurs propriétés contiennent des valeurs non valides ». Si vous essayez d’ajouter ce sous-domaine à partir du centre d’administration Microsoft 365, vous recevrez une erreur similaire. Pour plus d’informations sur l’erreur, consultez [Un domaine enfant n’hérite pas des modifications du domaine parent dans Office 365, Azure ou Intune](/office365/troubleshoot/administration/child-domain-fails-inherit-parent-domain-changes).

## <a name="how-to-verify-a-custom-subdomain"></a>Comment vérifier un sous-domaine personnalisé

Comme les sous-domaines héritent par défaut du type d’authentification du domaine racine, vous devez promouvoir le sous-domaine en domaine racine dans Azure AD à l’aide de Microsoft Graph afin de pouvoir définir le type d’authentification de votre choix.

### <a name="add-the-subdomain-and-view-its-authentication-type"></a>Ajouter le sous-domaine et afficher son type d’authentification

1. Utilisez PowerShell pour ajouter le nouveau sous-domaine, qui a le type d’authentification par défaut de son domaine racine. Les centres d’administration Azure AD et Microsoft 365 ne prennent pas encore en charge cette opération.

   ```powershell
   New-MsolDomain -Name "child.mydomain.com" -Authentication Federated
   ```

1. Utilisez [Afficheur Graph Azure AD](https://graphexplorer.azurewebsites.net) pour effectuer une requête GET sur le domaine. Étant donné que le domaine n’est pas un domaine racine, il hérite du type d’authentification du domaine racine. Votre commande et vos résultats peuvent se présenter comme suit, en utilisant votre propre ID de locataire :

   ```http
   GET https://graph.windows.net/{tenant_id}/domains?api-version=1.6
   
   Return:
     {
         "authenticationType": "Federated",
         "availabilityStatus": null,
         "isAdminManaged": true,
         "isDefault": false,
         "isDefaultForCloudRedirections": false,
         "isInitial": false,
         "isRoot": false,          <---------------- Not a root domain, so it inherits parent domain's authentication type (federated)
         "isVerified": true,
         "name": "child.mydomain.com",
         "supportedServices": [],
         "forceDeleteState": null,
         "state": null,
         "passwordValidityPeriodInDays": null,
         "passwordNotificationWindowInDays": null
     },
   ```

### <a name="use-azure-ad-graph-explorer-api-to-make-this-a-root-domain"></a>Utiliser l’API Afficheur Graph Azure AD pour en faire un domaine racine

Utilisez la commande suivante pour promouvoir le sous-domaine :

```http
POST https://graph.windows.net/{tenant_id}/domains/child.mydomain.com/promote?api-version=1.6
```

### <a name="change-the-subdomain-authentication-type"></a>Modifier le type d’authentification du sous-domaine

1. Utilisez la commande suivante pour modifier le type d’authentification du sous-domaine :

   ```powershell
   Set-MsolDomainAuthentication -DomainName child.mydomain.com -Authentication Managed
   ```

1. Vérifiez par le biais d’une requête GET dans Afficheur Graph Azure AD que le type d’authentification du sous-domaine est désormais géré :

   ```http
   GET https://graph.windows.net/{{tenant_id} }/domains?api-version=1.6
   
   Return:
     {
         "authenticationType": "Managed",   <---------- Now this domain is successfully added as Managed and not inheriting Federated status
         "availabilityStatus": null,
         "isAdminManaged": true,
         "isDefault": false,
         "isDefaultForCloudRedirections": false,
         "isInitial": false,
         "isRoot": true,   <------------------------------ Also a root domain, so not inheriting from parent domain any longer
         "isVerified": true,
         "name": "child.mydomain.com",
         "supportedServices": [
             "Email",
             "OfficeCommunicationsOnline",
             "Intune"
         ],
         "forceDeleteState": null,
         "state": null,
         "passwordValidityPeriodInDays": null,
         "passwordNotificationWindowInDays": null }
   ```

## <a name="next-steps"></a>Étapes suivantes

- [Ajouter des noms de domaines personnalisés](../fundamentals/add-custom-domain.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context)
- [Gérer les noms de domaine](domains-manage.md)
- [ForceDelete a custom domain name with Microsoft Graph API](/graph/api/domain-forcedelete?view=graph-rest-beta&preserve-view=true) (Appliquer une opération ForceDelete sur un nom de domaine personnalisé avec l’API Microsoft Graph)