---
title: Accéder aux journaux d’audit et les consulter
titleSuffix: Azure AD B2C
description: Comment accéder aux journaux d’audit Azure AD B2C par programme et dans le Portail Azure.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.date: 02/20/2020
ms.author: mimart
ms.subservice: B2C
ms.custom: fasttrack-edit
ms.openlocfilehash: 9485fb09d3ac7684ba91bcd37720a0e55603404b
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94952724"
---
# <a name="accessing-azure-ad-b2c-audit-logs"></a>Accès aux journaux d’audit Azure AD B2C

Azure Active Directory B2C (Azure AD B2C) produit des journaux d’audit contenant des informations sur les activités liées aux ressources, aux jetons émis et à l’accès administrateur B2C. Cet article fournit une rapide vue d’ensemble des informations disponibles dans les journaux d’audit, ainsi que des instructions pour accéder à ces données pour votre locataire Azure AD B2C.

Les événements des journaux d’audit sont uniquement conservés pendant **sept jours**. Envisagez de télécharger et stocker vos journaux d’activité en utilisant l’une des méthodes indiquées ci-dessous si vous avez besoin d’une période de rétention plus longue.

> [!NOTE]
> Vous ne voyez pas de connexions d’utilisateur pour des applications Azure AD B2C individuelles dans la section **Utilisateurs** des pages **Azure Active Directory** ou **Azure AD B2C** dans le portail Azure. Les événements de connexion affichent l’activité de l’utilisateur, mais ne peuvent pas être corrélés avec l’application B2C à laquelle l’utilisateur s’est connecté. Pour cela, vous devez utiliser les journaux d’audit, comme expliqué dans cet article.

## <a name="overview-of-activities-available-in-the-b2c-category-of-audit-logs"></a>Vue d’ensemble des activités disponibles dans la catégorie B2C des journaux d’audit

La catégorie **B2C** des journaux d’audit contient les types d’activités suivants :

|Type d’activité |Description  |
|---------|---------|
|Autorisation |Activités relatives à l’autorisation d’un utilisateur d’accéder à des ressources B2C (par exemple, un administrateur qui accède à la liste des stratégies B2C).         |
|Répertoire |Activités relatives aux attributs d’annuaire récupérés quand un administrateur se connecte en utilisant le Portail Microsoft Azure. |
|Application | Opérations de création, lecture, mise à jour et suppression (CRUD) sur les applications B2C. |
|Clé |Opérations CRUD sur les clés stockées dans le conteneur de clé B2C. |
|Ressource |Opérations CRUD sur les ressources B2C. Par exemple, les stratégies et les fournisseurs d’identité.
|Authentication |Validation des informations d’identification des utilisateurs et de l’émission des jetons.|

Pour les activités CRUD des objets utilisateur, reportez-vous à la catégorie **Annuaire principal**.

## <a name="example-activity"></a>Exemple d’activité

Cet exemple d’image du portail Azure montre les données capturées lorsqu’un utilisateur se connecte avec un fournisseur d’identité externe, dans ce cas, Facebook :

![Exemple de page Détails de l’activité du journal d’audit du Portail Microsoft Azure](./media/view-audit-logs/audit-logs-example.png)

Le volet des détails de l’activité contient les informations pertinentes suivantes :

|Section|Champ|Description|
|-------|-----|-----------|
| Activité | Name | Activité qui a eu lieu. Par exemple, *Émettre un id_token pour l’application*, qui conclut la connexion d’utilisateur réelle. |
| Initié par (intervenant) | ObjectId | **ID d’objet** de l’application B2C à laquelle l’utilisateur se connecte. Cet identificateur n’est pas visible dans le portail Azure, mais il est accessible via l’API Microsoft Graph. |
| Initié par (intervenant) | Nom de principal de service | **ID d’Application** de l’application B2C à laquelle l’utilisateur se connecte. |
| Cible(s) | ObjectId | **ID d’objet** de l’utilisateur qui se connecte. |
| Détails supplémentaires | TenantId | **ID de locataire** du locataire Azure AD B2C. |
| Détails supplémentaires | PolicyId | **ID de stratégie** du flux d’utilisateur (stratégie) utilisé pour la connexion de l’utilisateur. |
| Détails supplémentaires | ApplicationId | **ID d’Application** de l’application B2C à laquelle l’utilisateur se connecte. |

## <a name="view-audit-logs-in-the-azure-portal"></a>Afficher des journaux d’audit dans le portail Azure

Le portail Azure permet d’accéder aux événements du journal d’audit dans votre locataire Azure AD B2C.

1. Connectez-vous au [portail Azure](https://portal.azure.com)
1. Basculez vers le répertoire qui contient votre locataire Azure AD B2C, puis accédez à **Azure AD B2C**.
1. Sous **Activités** dans le menu de gauche, sélectionnez **Journaux d’audit**.

Une liste des événements d’activité journalisés au cours des sept derniers jours s’affiche.

![Exemple de filtre avec deux événements d’activité dans le portail Azure](./media/view-audit-logs/audit-logs-example-filter.png)

Plusieurs options de filtrage sont disponibles, notamment :

* **Type de ressource d’activité** : filtrez les types d’activités indiqués dans la section [Vue d’ensemble des activités disponibles](#overview-of-activities-available-in-the-b2c-category-of-audit-logs) de la table.
* **Date** : filtre la plage de dates des activités affichées.

Si vous sélectionnez une ligne dans la liste, les détails de l’activité de l’événement s’affichent.

Pour télécharger la liste des événements d’activité dans un fichier de valeurs séparées par des virgules (CSV), sélectionnez **Télécharger**.

## <a name="get-audit-logs-with-the-azure-ad-reporting-api"></a>Obtenir les journaux d’audit avec l’API de création de rapports Azure AD

Les journaux d’audit sont publiés dans le même pipeline que les autres activités Azure Active Directory, afin d’être accessibles par le biais de l’[API de création de rapports Azure Active Directory](/graph/api/directoryaudit-list). Pour plus d’informations, consultez [Prise en main de l’API de création de rapports Azure Active Directory](../active-directory/reports-monitoring/concept-reporting-api.md).

### <a name="enable-reporting-api-access"></a>Activer l’accès à l’API de rapports

Pour autoriser l’accès basé sur un script ou une application à l’API de création de rapports Azure AD, vous avez besoin d’une application inscrite dans votre locataire Azure AD B2C avec les autorisations d’API suivantes. Vous pouvez activer ces autorisations sur une inscription d’application existante au sein de votre locataire B2C, ou en créer une autre spécifiquement pour une utilisation avec l’automatisation des journaux d’audit.

* Microsoft Graph > Autorisations d’application > AuditLog > AuditLog.Read.All

Suivez les étapes de l’article suivant pour inscrire une application avec les autorisations requises :

[Gérer Azure AD B2C avec Microsoft Graph](microsoft-graph-get-started.md)

Après avoir inscrit une application avec les autorisations appropriées, voir la section Script PowerShell plus loin dans cet article pour obtenir un exemple de la façon dont vous pouvez obtenir des événements d’activité à l’aide d’un script.

### <a name="access-the-api"></a>Accéder à l’API

Pour télécharger les événements du journal d’audit Azure AD B2C via l’API, filtrez les journaux avec la catégorie `B2C`. Pour filtrer par catégorie, utilisez le paramètre de chaîne de requête `filter` lors de l’appel du point de terminaison de l’API de création de rapport Azure AD.

```http
https://graph.microsoft.com/v1.0/auditLogs/directoryAudits?$filter=loggedByService eq 'B2C' and activityDateTime gt 2019-09-10T02:28:17Z
```

### <a name="powershell-script"></a>Script PowerShell

Le script PowerShell suivant montre un exemple d’interrogation de l’API de rapports Azure AD. Après l’interrogation de l’API, le script imprime les événements journalisés dans la sortie standard, puis écrit la sortie JSON dans un fichier.

Vous pouvez essayer ce script dans [Azure Cloud Shell](overview.md). Veillez à le mettre à jour avec votre ID d’application, votre clé secrète client et le nom de votre abonné Azure AD B2C.

```powershell
# This script requires an application registration that's granted Microsoft Graph API permission
# https://docs.microsoft.com/azure/active-directory-b2c/microsoft-graph-get-started

# Constants
$ClientID       = "your-client-application-id-here"       # Insert your application's client ID, a GUID
$ClientSecret   = "your-client-application-secret-here"   # Insert your application's client secret
$tenantdomain   = "your-b2c-tenant.onmicrosoft.com"       # Insert your Azure AD B2C tenant domain name

$loginURL       = "https://login.microsoftonline.com"
$resource       = "https://graph.microsoft.com"           # Microsoft Graph API resource URI
$7daysago       = "{0:s}" -f (get-date).AddDays(-7) + "Z" # Use 'AddMinutes(-5)' to decrement minutes, for example
Write-Output "Searching for events starting $7daysago"

# Create HTTP header, get an OAuth2 access token based on client id, secret and tenant domain
$body       = @{grant_type="client_credentials";resource=$resource;client_id=$ClientID;client_secret=$ClientSecret}
$oauth      = Invoke-RestMethod -Method Post -Uri $loginURL/$tenantdomain/oauth2/token?api-version=1.0 -Body $body

# Parse audit report items, save output to file(s): auditX.json, where X = 0 through n for number of nextLink pages
if ($oauth.access_token -ne $null) {
    $i=0
    $headerParams = @{'Authorization'="$($oauth.token_type) $($oauth.access_token)"}
    $url = "https://graph.microsoft.com/v1.0/auditLogs/directoryAudits?`$filter=loggedByService eq 'B2C' and activityDateTime gt  " + $7daysago

    # loop through each query page (1 through n)
    Do {
        # display each event on the console window
        Write-Output "Fetching data using Uri: $url"
        $myReport = (Invoke-WebRequest -UseBasicParsing -Headers $headerParams -Uri $url)
        foreach ($event in ($myReport.Content | ConvertFrom-Json).value) {
            Write-Output ($event | ConvertTo-Json)
        }

        # save the query page to an output file
        Write-Output "Save the output to a file audit$i.json"
        $myReport.Content | Out-File -FilePath audit$i.json -Force
        $url = ($myReport.Content | ConvertFrom-Json).'@odata.nextLink'
        $i = $i+1
    } while($url -ne $null)
} else {
    Write-Host "ERROR: No Access Token"
}
```

Voici la représentation JSON de l’exemple d’événement d’activité illustré plus haut dans l’article :

```json
{
    "id": "B2C_DQO3J_4984536",
    "category": "Authentication",
    "correlationId": "00000000-0000-0000-0000-000000000000",
    "result": "success",
    "resultReason": "N/A",
    "activityDisplayName": "Issue an id_token to the application",
    "activityDateTime": "2019-09-14T18:13:17.0618117Z",
    "loggedByService": "B2C",
    "operationType": "",
    "initiatedBy": {
        "user": null,
        "app": {
            "appId": "00000000-0000-0000-0000-000000000000",
            "displayName": null,
            "servicePrincipalId": null,
            "servicePrincipalName": "00000000-0000-0000-0000-000000000000"
        }
    },
    "targetResources": [
        {
            "id": "00000000-0000-0000-0000-000000000000",
            "displayName": null,
            "type": "User",
            "userPrincipalName": null,
            "groupType": null,
            "modifiedProperties": []
        }
    ],
    "additionalDetails": [
        {
            "key": "TenantId",
            "value": "test.onmicrosoft.com"
        },
        {
            "key": "PolicyId",
            "value": "B2C_1A_signup_signin"
        },
        {
            "key": "ApplicationId",
            "value": "00000000-0000-0000-0000-000000000000"
        },
        {
            "key": "Client",
            "value": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.132 Safari/537.36"
        },
        {
            "key": "IdentityProviderName",
            "value": "facebook"
        },
        {
            "key": "IdentityProviderApplicationId",
            "value": "0000000000000000"
        },
        {
            "key": "ClientIpAddress",
            "value": "127.0.0.1"
        }
    ]
}
```

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez automatiser d’autres tâches d’administration, par exemple, [gérer les comptes d’utilisateurs Azure AD B2C avec Microsoft Graph](manage-user-accounts-graph-api.md).