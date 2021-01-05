---
title: Interpréter le schéma des journaux d’audit Azure Active Directory dans Azure Monitor | Microsoft Docs
description: Décrire le schéma de journal d’audit Azure AD pour une utilisation dans Azure Monitor
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
ms.openlocfilehash: cc7a5e2e45cd3d76c0d8af612291ecaf9893b5b5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89231093"
---
# <a name="interpret-the-azure-ad-audit-logs-schema-in-azure-monitor-preview"></a>Interpréter le schéma des journaux d’audit Azure Active Directory dans Azure Monitor (préversion)

Cet article décrit le schéma de journal d’audit Azure Active Directory (Azure AD) dans Azure Monitor. Chaque entrée de journal est stockée sous en tant que texte mis en forme tel qu’un objet blob JSON, comme indiqué dans les deux exemples ci-dessous : 

```json
{ 
    "records": [ 
    { 
        "time": "2018-03-17T00:14:31.2585575Z", 
        "operationName": "Change password (self-service)",
        "operationVersion": "1.0",
        "category": "Audit", 
        "tenantId": "bf85dc9d-cb43-44a4-80c4-469e8c58249e", 
        "resultType": "Success", 
        "resultSignature": "-1", 
        "resultDescription": "None", 
        "durationMs": "-1", 
        "correlationId": "60d5e89a-b890-413f-9e25-a047734afe9f", 
        "identity": "sreens@wingtiptoysonline.com", 
        "Level": "Informational", 
        "location": "WUS", 
        "properties": { 
            "identityType": "UPN", 
            "operationType": "Update", 
            "additionalDetails": "None", 
            "additionalTargets": "", 
            "targetUpdatedProperties": "", 
            "targetResourceType": "UPN__TenantContextID__PUID__ObjectID__ObjectClass", 
            "targetResourceName": "sreens@wingtiptoysonline.com__bf85dc9d-cb43-44a4-80c4-469e8c58249e__1003BFFD9FEB17DB__7a408bdd-7d97-4574-8511-dd747b56465d__User", 
            "auditEventCategory": "UserManagement" 
        } 
    } 
    ] 
} 
```

```json
{ 
    "records": [ 
    { 
        "time": "2018-03-18T19:47:43.0368859Z", 
        "operationName": "Update service principal.", 
        "operationVersion": "1.0", 
        "category": "Audit", 
        "tenantId": "bf85dc9d-cb43-44a4-80c4-469e8c58249e", 
        "resultType": "Success", 
        "resultSignature": "-1", 
        "durationMs": "-1", 
        "callerIpAddress": "<null>", 
        "correlationId": "14916c7a-5a7d-44e8-9b06-74b49efb08ee", 
        "identity": "NA", 
        "Level": "Informational", 
        "properties": { 
            "identityType": "NA", 
            "operationType": "Update", 
            "additionalDetails": {}, 
            "additionalTargets": "", 
            "targetUpdatedProperties": [ 
            { 
                "Name": "Included Updated Properties", 
                "OldValue": null, 
                "NewValue": "" 
            }, 
            { 
                "Name": "TargetId.ServicePrincipalNames", 
                "OldValue": null, 
                "NewValue": "http://adapplicationregistry.onmicrosoft.com/salesforce.com/primary;cd3ed3de-93ee-400b-8b19-b61ef44a0f29" 
            } 
            ], 
        "targetResourceType": "Other__ObjectID__ObjectClass__Name__AppId__SPN", 
        "targetResourceName": "ServicePrincipal_ea70a262-4da3-440a-b396-9734ddfd9df2__ea70a262-4da3-440a-b396-9734ddfd9df2__ServicePrincipal__Salesforce__cd3ed3de-93ee-400b-8b19-b61ef44a0f29__http://adapplicationregistry.onmicrosoft.com/salesforce.com/primary;cd3ed3de-93ee-400b-8b19-b61ef44a0f29", 
        "auditEventCategory": "ApplicationManagement" 
      } 
    } 
    ] 
} 
```

```json
{
    "records": [
    {
        "time": "2018-12-10T00:03:46.6161822Z",
        "resourceId": "/tenants/7918d4b5-0442-4a97-be2d-36f9f9962ece/providers/Microsoft.aadiam",
        "operationName": "Update policy",
        "operationVersion": "1.0",
        "category": "AuditLogs",
        "tenantId": "7918d4b5-0442-4a97-be2d-36f9f9962ece",
        "resultSignature": "None",
        "durationMs": 0,
        "callerIpAddress": "<null>",
        "correlationId": "192298c1-0994-4dd6-b05a-a6c5984c31cb",
        "identity": "MS-PIM",
        "level": "Informational",
        "properties": {
            "id": "Directory_VNXV4_28148892",
            "category": "Policy",
            "correlationId": "192298c1-0994-4dd6-b05a-a6c5984c31cb",
            "result": 0,
            "resultReason": "",
            "activityDisplayName": "Update policy",
            "activityDateTime": "2018-12-10T00:03:46.6161822+00:00",
            "loggedByService": "Core Directory",
            "operationType": "Update",
            "initiatedBy": {},
            "targetResources": [
            {
                "id": "5e7a8ae7-165d-44a4-a4f4-6141f8c8ef40",
                "displayName": "Default Policy",
                "type": "Policy",
                "modifiedProperties": []
            }
            ],
            "additionalDetails": []
        }
    }
    ]
}

```

## <a name="field-and-property-descriptions"></a>Descriptions de champ et de propriété

| Nom du champ | Description |
|------------|-------------|
| time       | Date et heure (UTC). |
| operationName | Nom de l’opération. |
| operationVersion | Version d’API REST demandée par le client. |
| catégorie | Actuellement, *Audit* est la seule valeur prise en charge. |
| tenantId | GUID de locataire associé aux journaux d’activité. |
| resultType | Résultat de l'opération. Le résultat peut être *Success* (Réussite) ou *Failure* (Échec). |
| resultSignature |  Ce champ n’étant pas mappé, vous pouvez l’ignorer en toute sécurité. | 
| resultDescription | Description supplémentaire du résultat, si disponible. | 
| durationMS |  Ce champ n’étant pas mappé, vous pouvez l’ignorer en toute sécurité. |
| callerIpAddress | Adresse IP du client à l’origine de la demande. | 
| correlationId | GUID facultatif passé par le client. Il peut aider à corréler des opérations côté client avec des opérations côté serveur, et est utile lors du suivi de journaux d’activité couvrant plusieurs services. |
| identité | Identité extraite du jeton présenté lors de la création de la demande. Il peut s’agir d’un compte d’utilisateur, d’un compte système ou d’un principal du service. |
| level | Type de message. Pour les journaux d’audit, le niveau est toujours *informatif*. |
| location | Emplacement du centre de données. |
| properties | Répertorie les propriétés prises en charge en lien avec un journal d’audit. Pour plus d’informations, voir le tableau suivant. | 

<br>

| Nom de la propriété | Description |
|---------------|-------------|
| AuditEventCategory | Type d’événement d’audit. Il peut s’agir de *Gestion des utilisateurs*, de *Gestion des applications*, ou d’un autre type.|
| Type d'identité | Il peut s’agir d’*Application* ou d’*Utilisateur*. |
| Type d’opération | Il peut s’agir d’*Ajouter*, de *Mettre à jour*, de *Supprimer*, ou d’*autre* chose. |
| Type de ressource cible | Spécifie le type de ressource cible sur lequel a été effectuée l’opération. Il peut s’agir d’*Application*, d’*Utilisateur*, de *Rôle* ou de *Stratégie*. | 
| Nom de ressource cible | Nom de la ressource cible. Il peut s’agir d’un nom d’application, d’un nom de rôle, d’un nom d’utilisateur principal ou d’un nom de principal du service. |
| additionalTargets | Répertorie les propriétés supplémentaires pour des opérations spécifiques. Par exemple, pour une opération de mise à jour, les valeurs anciennes et nouvelles sont répertoriées sous *targetUpdatedProperties*. | 

## <a name="next-steps"></a>Étapes suivantes

* [Interpréter le schéma des journaux d’activité de connexion dans Azure Monitor](reference-azure-monitor-sign-ins-log-schema.md)
* [Journaux de diagnostics Azure](../../azure-monitor/platform/platform-logs-overview.md)
* [Questions fréquentes et problèmes connus](concept-activity-logs-azure-monitor.md#frequently-asked-questions)