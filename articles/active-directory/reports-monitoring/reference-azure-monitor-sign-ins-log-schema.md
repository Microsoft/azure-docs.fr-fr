---
title: Schéma des journaux de connexion dans Azure Monitor | Microsoft Docs
description: Décrire le schéma des journaux de connexion Azure AD pour une utilisation dans Azure Monitor
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 4b18127b-d1d0-4bdc-8f9c-6a4c991c5f75
ms.service: active-directory
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/18/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6a881dee50195fe4995c77d793b4f4b75091d20b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89231110"
---
# <a name="interpret-the-azure-ad-sign-in-logs-schema-in-azure-monitor"></a>Interpréter le schéma des journaux de connexion Azure Active Directory dans Azure Monitor

Cet article décrit le schéma de journal de connexion Azure Active Directory (Azure AD) dans Azure Monitor. La plupart des informations liées aux connexions sont fournies sous l’attribut *Propriétés* de l’objet `records`.


```json
{ 
    "time": "2019-03-12T16:02:15.5522137Z", 
    "resourceId": "/tenants/<TENANT ID>/providers/Microsoft.aadiam",
    "operationName": "Sign-in activity", 
    "operationVersion": "1.0", 
    "category": "SignInLogs", 
    "tenantId": "<TENANT ID>", 
    "resultType": "50140", 
    "resultSignature": "None", 
    "resultDescription": "This error occurred due to 'Keep me signed in' interrupt when the user was signing-in.", 
    "durationMs": 0, 
    "callerIpAddress": "<CALLER IP ADDRESS>", 
    "correlationId": "a75a10bd-c126-486b-9742-c03110d36262", 
    "identity": "Timothy Perkins", 
    "Level": 4, 
    "location": "US", 
    "properties": 
        {
            "id":"0231f922-93fa-4005-bb11-b344eca03c01",
            "createdDateTime":"2019-03-12T16:02:15.5522137+00:00",
            "userDisplayName":"Timothy Perkins",
            "userPrincipalName":"<USER PRINCIPAL NAME>",
            "userId":"<USER ID>",
            "appId":"<APPLICATION ID>",
            "appDisplayName":"Azure Portal",
            "ipAddress":"<IP ADDRESS>",
            "status":
            {
                "errorCode":50140,
                "failureReason":"This error occurred due to 'Keep me signed in' interrupt when the user was signing-in."
            },
            "clientAppUsed":"Browser",
            "deviceDetail":
            {
                "operatingSystem":"Windows 10",
                "browser":"Chrome 72.0.3626"
            },
            "location":
                {
                    "city":"Bellevue",
                    "state":"Washington",
                    "countryOrRegion":"US",
                    "geoCoordinates":
                    {
                        "latitude":45,
                        "longitude":122
                    }
                },
            "correlationId":"a75a10bd-c126-486b-9742-c03110d36262",
            "conditionalAccessStatus":"notApplied",
            "appliedConditionalAccessPolicies":
            [
                {
                    "id":"ae11ffaa-9879-44e0-972c-7538fd5c4d1a",
                    "displayName":"Hr app access policy",
                    "enforcedGrantControls":
                    [
                        "Mfa"
                    ],
                    "enforcedSessionControls":
                    [
                    ],
                    "result":"notApplied"
                },
                {
                    "id":"b915a70b-2eee-47b6-85b6-ff4f4a66256d",
                    "displayName":"MFA for all but global support access",
                    "enforcedGrantControls":[],
                    "enforcedSessionControls":[],
                    "result":"notEnabled"
                },
                {
                    "id":"830f27fa-67a8-461f-8791-635b7225caf1",
                    "displayName":"Header Based Application Control",
                    "enforcedGrantControls":["Mfa"],
                    "enforcedSessionControls":[],
                    "result":"notApplied"
                },
                {
                    "id":"8ed8d7f7-0a2e-437b-b512-9e47bed562e6",
                    "displayName":"MFA for everyones",
                    "enforcedGrantControls":[],
                    "enforcedSessionControls":[],
                    "result":"notEnabled"
                },
                {
                    "id":"52924e0f-798b-4afd-8c42-49055c7d6395",
                    "displayName":"Device compliant",
                    "enforcedGrantControls":[],
                    "enforcedSessionControls":[],
                    "result":"notEnabled"
                },
             ],
            "isInteractive":true,
            "tokenIssuerType":"AzureAD",
            "authenticationProcessingDetails":[],
            "networkLocationDetails":[],
            "processingTimeInMilliseconds":0,
            "riskDetail":"hidden",
            "riskLevelAggregated":"hidden",
            "riskLevelDuringSignIn":"hidden",
            "riskState":"none",
            "riskEventTypes":[],
            "resourceDisplayName":"windows azure service management api",
            "resourceId":"797f4846-ba00-4fd7-ba43-dac1f8f63013",
            "authenticationMethodsUsed":[]
        }
}
```


## <a name="field-descriptions"></a>Descriptions de champs

| Nom du champ | Description |
|------------|-------------|
| Temps | Date et heure UTC. |
| ResourceId | Valeur non mappée, vous pouvez ignorer ce champ.  |
| NomOpération | Pour les connexions, cette valeur est toujours *Activité de connexion*. |
| operationVersion | Version d’API REST demandée par le client. |
| Category | Pour les connexions, cette valeur est toujours *SignIn*. | 
| TenantId | GUID de locataire associé aux journaux d’activité. |
| ResultType | Le résultat de l’opération de connexion peut être *Success* (Réussite) ou *Failure* (Échec). | 
| ResultSignature | Contient le code d’erreur éventuel de l’opération de connexion. |
| resultDescription | Fournit la description de l’erreur pour l’opération de connexion. |
| riskDetail | riskDetail | Fournit le motif de l’état spécifique d’un utilisateur à risque, d’une connexion ou d’une détection d’événement à risque. Les valeurs possibles sont : `none`, `adminGeneratedTemporaryPassword`, `userPerformedSecuredPasswordChange`, `userPerformedSecuredPasswordReset`, `adminConfirmedSigninSafe`, `aiConfirmedSigninSafe`, `userPassedMFADrivenByRiskBasedPolicy`, `adminDismissedAllRiskForUser`, `adminConfirmedSigninCompromised`, `unknownFutureValue`. La valeur `none` signifie qu’aucune action n’a été effectuée sur l’utilisateur ou la connexion jusqu’à présent. <br>**Remarque :** Les détails de cette propriété nécessitent une licence Azure AD Premium P2. Les autres licences retournent la valeur `hidden`. |
| riskEventTypes | riskEventTypes | Types de détections d’événements à risque associés à la connexion. Les valeurs possibles sont : `unlikelyTravel`, `anonymizedIPAddress`, `maliciousIPAddress`, `unfamiliarFeatures`, `malwareInfectedIPAddress`, `suspiciousIPAddress`, `leakedCredentials`, `investigationsThreatIntelligence`, `generic` et `unknownFutureValue`. |
| riskLevelAggregated | riskLevel | Niveau de risque agrégé. Les valeurs possibles sont : `none`, `low`, `medium`, `high`, `hidden` et `unknownFutureValue`. La valeur `hidden` indique que l’utilisateur ou la connexion n’a pas été activé pour Azure AD Identity Protection. **Remarque :** Les détails de cette propriété sont uniquement disponibles pour les clients Azure AD Premium P2. Tous les autres clients se verront retourner `hidden`. |
| riskLevelDuringSignIn | riskLevel | Niveau de risque pendant la connexion. Les valeurs possibles sont : `none`, `low`, `medium`, `high`, `hidden` et `unknownFutureValue`. La valeur `hidden` indique que l’utilisateur ou la connexion n’a pas été activé pour Azure AD Identity Protection. **Remarque :** Les détails de cette propriété sont uniquement disponibles pour les clients Azure AD Premium P2. Tous les autres clients se verront retourner `hidden`. |
| riskState | riskState | Signale l’état de l’utilisateur à risque, de la connexion ou d’une détection d’événement à risque. Les valeurs possibles sont : `none`, `confirmedSafe`, `remediated`, `dismissed`, `atRisk`, `confirmedCompromised`, `unknownFutureValue`. |
| DurationMs |  Valeur non mappée, vous pouvez ignorer ce champ. |
| callerIpAddress | Adresse IP du client à l’origine de la demande. | 
| CorrelationId | GUID facultatif transmis par le client. Cette valeur peut aider à corréler des opérations côté client avec des opérations côté serveur, et est utile lors du suivi de journaux d’activité couvrant plusieurs services. |
| Identité | Identité extraite du jeton présenté lors de la création de la demande. Il peut s’agir d’un compte d’utilisateur, d’un compte système ou d’un principal du service. |
| Level | Fournit le type de message. Pour l’audit, il s’agit toujours d’*Information*. |
| Emplacement | Indique l’emplacement de l’activité de connexion. |
| Propriétés | Répertorie toutes les propriétés associées aux connexions. Pour plus d’informations, voir la [documentation de référence sur l’API Microsoft Graph](/graph/api/resources/signin?view=graph-rest-beta). Ce schéma utilise les mêmes noms d’attribut que la ressource de connexion pour une meilleure lisibilité.

## <a name="next-steps"></a>Étapes suivantes

* [Interpréter le schéma des journaux d’audit dans Azure Monitor](reference-azure-monitor-audit-log-schema.md)
* [En savoir plus sur les journaux de la plateforme Azure](../../azure-monitor/platform/platform-logs-overview.md)