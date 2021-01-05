---
author: normesta
ms.service: storage
ms.topic: include
ms.date: 09/28/2020
ms.author: normesta
ms.openlocfilehash: e3732823be1c8391f2bec9018a094200c7c93a5e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91710955"
---
| Propriété | Description |
|:--- |:---|
|**identity / type** | Type d’authentification qui a été utilisé pour effectuer la demande. Par exemple : `OAuth`, `SAS Key`, `Account Key` ou `Anonymous` |
|**identity / tokenHash**|Ce champ est réservé à un usage interne. |
|**authorization / action** | Action affectée à la demande. |
|**authorization / roleAssignmentId** | ID d’attribution de rôle. Par exemple : `4e2521b7-13be-4363-aeda-111111111111`.|
|**authorization / roleDefinitionId** | ID de définition de rôle. Par exemple : `ba92f5b4-2d11-453d-a403-111111111111"`.|
|**principals / id** | ID du principal de sécurité. Par exemple : `a4711f3a-254f-4cfb-8a2d-111111111111`.|
|**principals / type** | Type du principal de sécurité. Par exemple : `ServicePrincipal`. |
|**requester / appID** | ID d’application Open Authorization (OAuth) utilisé comme demandeur. <br> Par exemple : `d3f7d5fe-e64a-4e4e-871d-333333333333`.|
|**requester / audience** | Audience OAuth de la demande. Par exemple : `https://storage.azure.com`. |
|**requester / objectId** | ID d’objet OAuth du demandeur. Dans le cas de l’authentification Kerberos, représente l’identificateur d’objet de l’utilisateur authentifié Kerberos. Par exemple : `0e0bf547-55e5-465c-91b7-2873712b249c`. |
|**requester / tenantId** | ID de locataire OAuth de l’identité. Par exemple : `72f988bf-86f1-41af-91ab-222222222222`.|
|**requester / tokenIssuer** | Émetteur de jeton OAuth. Par exemple : `https://sts.windows.net/72f988bf-86f1-41af-91ab-222222222222/`.|
|**requester / upn** | Nom d’utilisateur principal (UPN) du demandeur. Par exemple : `someone@contoso.com`. |
|**requester / userName** | Ce champ est réservé à un usage interne.|