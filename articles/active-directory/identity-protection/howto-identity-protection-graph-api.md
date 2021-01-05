---
title: API Microsoft Graph pour Azure Active Directory Identity Protection
description: Découvrez comment interroger Microsoft Graph pour obtenir les détections de risques et les informations associées à partir d’Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: how-to
ms.date: 10/06/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5367e5027bfae2fa3ed7e87a779e50e4048ba608
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861729"
---
# <a name="get-started-with-azure-active-directory-identity-protection-and-microsoft-graph"></a>Prise en main d’Azure Active Directory Identity Protection et de Microsoft Graph

Microsoft Graph est le point de terminaison d’API unifiée de Microsoft et accueille également les API [d’Azure Active Directory Identity Protection](./overview-identity-protection.md). Il existe trois API qui exposent des informations sur les connexions et les utilisateurs à risque. La première API, **riskDetection**, vous permet d’interroger Microsoft Graph pour obtenir la liste des détections de risque liées à l’utilisateur et à la connexion, ainsi que des informations associées sur la détection. La deuxième API, **riskyUsers**, vous permet d’interroger Microsoft Graph pour obtenir des informations sur les utilisateurs que le service Identity Protection a détecté comme étant à risque. La troisième API, **signIn**, vous permet d’interroger Microsoft Graph pour obtenir des informations sur des connexions Azure AD avec des propriétés spécifiques relatives au risque en termes de niveau, de détails et d’état. 

Cet article vous permet de vous familiariser avec la connexion à Microsoft Graph et l'interrogation de ces API. Pour obtenir des informations détaillées ainsi qu’un accès à l’Explorateur Graph, consultez le [site de Microsoft Graph](https://graph.microsoft.io/) ou la documentation de référence propre à ces API :

* [API riskDetection](/graph/api/resources/riskdetection?view=graph-rest-v1.0)
* [API riskyUsers](/graph/api/resources/riskyuser?view=graph-rest-v1.0)
* [API signIn](/graph/api/resources/signin?view=graph-rest-v1.0)

## <a name="connect-to-microsoft-graph"></a>Se connecter à Microsoft Graph

L’accès aux données d’Identity Protection par le biais de Microsoft Graph se fait en quatre étapes :

- [Récupérez votre nom de domaine](#retrieve-your-domain-name)
- [Créez une inscription d’application](#create-a-new-app-registration)
- [Accédez aux autorisations d’API](#configure-api-permissions)
- [Configurez un nom d'informations d'identification valide](#configure-a-valid-credential)

### <a name="retrieve-your-domain-name"></a>Récupérer votre nom de domaine 

1. Connectez-vous au [portail Azure](https://portal.azure.com).  
1. Accédez à **Azure Active Directory** > **noms de domaine personnalisés**. 
1. Notez bien le domaine `.onmicrosoft.com`, vous aurez besoin de ces informations lors d’une prochaine étape.

### <a name="create-a-new-app-registration"></a>Créer une nouvelle inscription d’application

1. Dans le Portail Azure, cliquez sur **Azure Active Directory** > **Inscriptions d’applications**.
1. Sélectionnez **Nouvelle inscription**.
1. Dans la page **Créer**, effectuez les étapes suivantes :
   1. Dans la zone de texte **Nom**, entrez le nom de votre application (par exemple : API de détections des risques Azure AD).
   1. Sous **Types de comptes pris en charge**, sélectionnez le type de compte qui utilisera les API.
   1. Sélectionnez **Inscription**.
1. Copiez **l’ID de l’application**.

### <a name="configure-api-permissions"></a>Configurez les autorisations d’API

1. Depuis **l’application** que vous avez créée, puis sélectionnez **Autorisations de l’API**.
1. Dans la page **Autorisations configurées**, dans la barre d’outils en haut, cliquez sur **Ajouter une autorisation**.
1. Dans la page **Ajouter un accès d’API**, cliquez sur **Sélectionner une API**.
1. Dans la page **Sélectionner une API**, sélectionnez **Microsoft Graph**, puis cliquez sur **Sélectionner**.
1. Page Demander des **autorisations d’API** 
   1. Sélectionnez **Autorisations de l’application**.
   1. Cochez les cases en regard de `IdentityRiskEvent.Read.All` et `IdentityRiskyUser.Read.All`.
   1. Sélectionnez **Ajouter des autorisations**.
1. Sélectionner **Accorder le consentement de l’administrateur pour le domaine** 

### <a name="configure-a-valid-credential"></a>Configurez un nom d'informations d'identification valide

1. Depuis **l’application** que vous avez créée, sélectionnez **Certificats et secrets**.
1. Sous **Secrets client**, sélectionnez **Nouveau secret client**.
   1. Attribuez au secret client une **Description** et définissez le délai d’expiration en fonction des stratégies de votre organisation.
   1. Sélectionnez **Ajouter**.

   > [!NOTE]
   > Si vous perdez cette clé, vous devrez revenir à cette section et créer une nouvelle clé. Gardez cette clé secrète : toute personne la possédant peut accéder à vos données.

## <a name="authenticate-to-microsoft-graph-and-query-the-identity-risk-detections-api"></a>Authentifiez-vous auprès de Microsoft Graph et interrogez l’API de détections de risques concernant l’identité

À ce stade, vous devez avoir :

- le nom de domaine de votre client ;
- L’ID de l’application (client) 
- Le secret ou le certificat du client 

Pour l’authentification, envoyez une demande POST à `https://login.microsoft.com` , avec les paramètres suivants dans le corps :

- grant_type : « **client_informationsidentification** »
- resource : `https://graph.microsoft.com`
- client_id : \<your client ID\>
- client_secret : \<your key\>

Si l’opération réussit, la requête retourne un jeton d’authentification.  
Pour appeler l’API, créez un en-tête avec le paramètre suivant :

```
`Authorization`="<token_type> <access_token>"
```

Lors de l’authentification, le jeton retourné contient le type de jeton ainsi que le jeton d’accès.

Envoyez cet en-tête en tant que requête à l’URL de l’API suivante : `https://graph.microsoft.com/v1.0/identityProtection/riskDetections`

La réponse, en cas de réussite, consiste en une collection de détections de risques concernant l’identité ainsi que les données associées au format JSON OData, qui peuvent être analysées et gérées selon vos besoins.

### <a name="sample"></a>Exemple

Cet exemple montre l’utilisation d’un secret partagé pour l’authentification. Dans un environnement de production, le stockage des secrets dans du code est généralement mal vu. Les organisations peuvent utiliser des identités gérées pour les ressources Azure pour sécuriser ces informations d’identification. Pour en savoir plus sur les identités managées, consultez l’article [Que sont les identités managées pour les ressources Azure ?](../managed-identities-azure-resources/overview.md)

Voici un exemple de code pour l’authentification et l’appel de l’API par le biais de PowerShell.  
Il suffit d’ajouter l’ID client, la clé secrète, ainsi que le domaine du locataire.

```PowerShell
    $ClientID       = "<your client ID here>"        # Should be a ~36 hex character string; insert your info here
    $ClientSecret   = "<your client secret here>"    # Should be a ~44 character string; insert your info here
    $tenantdomain   = "<your tenant domain here>"    # For example, contoso.onmicrosoft.com

    $loginURL       = "https://login.microsoft.com"
    $resource       = "https://graph.microsoft.com"

    $body       = @{grant_type="client_credentials";resource=$resource;client_id=$ClientID;client_secret=$ClientSecret}
    $oauth      = Invoke-RestMethod -Method Post -Uri $loginURL/$tenantdomain/oauth2/token?api-version=1.0 -Body $body

    Write-Output $oauth

    if ($oauth.access_token -ne $null) {
        $headerParams = @{'Authorization'="$($oauth.token_type) $($oauth.access_token)"}

        $url = "https://graph.microsoft.com/v1.0/identityProtection/riskDetections"
        Write-Output $url

        $myReport = (Invoke-WebRequest -UseBasicParsing -Headers $headerParams -Uri $url)

        foreach ($event in ($myReport.Content | ConvertFrom-Json).value) {
            Write-Output $event
        }

    } else {
        Write-Host "ERROR: No Access Token"
    } 
```

## <a name="query-the-apis"></a>Interroger les API

Ces trois API fournissent une multitude d’opportunités de récupérer des informations sur les connexions et les utilisateurs à risque dans votre organisation. Voici quelques cas d’utilisation habituelle de ces API avec les exemples de requête associés. Vous pouvez exécuter ces requêtes à l’aide de l’exemple de code ci-dessus, ou par le biais de l’[Explorateur Graph](https://developer.microsoft.com/graph/graph-explorer).

### <a name="get-all-of-the-offline-risk-detections-riskdetection-api"></a>Récupérez toutes les détections de risque hors connexion (API riskDetection)

Avec les stratégies de risque de connexion d’Identity Protection, vous pouvez appliquer des conditions lorsque le risque est détecté en temps réel. Mais qu’en est-il des détections qui sont découvertes hors connexion ? Pour comprendre quelles détections ont eu lieu hors ligne et qui donc n'auraient pas déclenché la stratégie de risque de connexion, vous pouvez interroger l'API riskDetection.

```
GET https://graph.microsoft.com/v1.0/identityProtection/riskDetections?$filter=detectionTimingType eq 'offline'
```

### <a name="get-all-of-the-users-who-successfully-passed-an-mfa-challenge-triggered-by-risky-sign-ins-policy-riskyusers-api"></a>Obtenir tous les utilisateurs qui ont réussi l’authentification MFA déclenchée par la stratégie de connexion risquée (API riskyUsers)

Pour comprendre l’impact que les stratégies Identity Protection reposant sur les risques ont sur votre organisation, vous pouvez interroger tous les utilisateurs qui ont réussi une authentification MFA déclenchée par une stratégie de connexion à risque. Ces informations peuvent vous aider à comprendre quels utilisateurs Identity Protection ont peut-être été détectés, à tort, comme présentant des risques, et lesquels, parmi vos utilisateurs légitimes, peuvent effectuer des actions que l’intelligence artificielle considère à risque.

```
GET https://graph.microsoft.com/v1.0/identityProtection/riskyUsers?$filter=riskDetail eq 'userPassedMFADrivenByRiskBasedPolicy'
```

## <a name="next-steps"></a>Étapes suivantes

Félicitations, vous venez de créer votre premier appel à Microsoft Graph !  
Vous pouvez à présent interroger les détections de risques concernant l’identité et utiliser les données comme bon vous semble.

Pour en savoir plus sur Microsoft Graph et comment créer des applications à l’aide de l’API Graph, consultez la [documentation](/graph/overview) afférente et bien plus sur le [site de Microsoft Graph](https://developer.microsoft.com/graph). 

Pour plus d’informations, consultez :

- [Azure Active Directory Identity Protection](./overview-identity-protection.md)
- [Types de détections de risques détectées par Azure Active Directory Identity Protection](./overview-identity-protection.md)
- [Microsoft Graph](https://developer.microsoft.com/graph/)
- [Overview of Microsoft Graph (Vue d’ensemble de Microsoft Graph)](https://developer.microsoft.com/graph/docs)
- [Azure AD Identity Protection Service Root (Racine de service d’Azure AD Identity Protection)](/graph/api/resources/identityprotectionroot)
