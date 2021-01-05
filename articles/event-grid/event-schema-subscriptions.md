---
title: Abonnement Azure en tant que source Event Grid
description: Décrit les propriétés fournies pour les événements d’abonnement avec Azure Event Grid.
ms.topic: reference
ms.date: 07/07/2020
ms.openlocfilehash: 72b1a73bf418b417cd29f88063781e7b45979998
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86105895"
---
# <a name="azure-subscription-as-an-event-grid-source"></a>Abonnement Azure en tant que source Event Grid

Cet article fournit les propriétés et les schémas des événements d’abonnement Azure. Pour une présentation des schémas d’événements, consultez [Schéma d’événements Azure Event Grid](event-schema.md).

Les abonnements Azure et les groupes de ressources émettent les mêmes types d’événements. Les types d’événements sont liés aux modifications ou actions des ressources. La principale différence est que les groupes de ressources émettent des événements pour les ressources appartenant au groupe de ressources, alors que les abonnements Azure émettent des événements pour les ressources de l’abonnement.

Des événements de ressource sont créés pour les opérations PUT, PATCH, POST et DELETE envoyées à `management.azure.com`. Les opérations GET ne créent pas d’événements. Les opérations envoyées au plan de données (comme `myaccount.blob.core.windows.net`) ne créent pas d’événements. Les événements d’action fournissent des données d’événement pour des opérations comme l’énumération des clés pour une ressource.

Lorsque vous vous abonnez aux événements d’un abonnement Azure, votre point de terminaison reçoit tous les événements pour cet abonnement. Les événements peuvent inclure des événements que vous souhaitez visualiser, comme la mise à jour d’une machine virtuelle, mais également des événements qui ne sont peut-être pas importants pour vous, comme l’écriture d’une nouvelle entrée dans l’historique de déploiement. Vous pouvez recevoir tous les événements à votre point de terminaison et écrire du code qui traite les événements que vous souhaitez gérer. Vous pouvez aussi définir un filtre lors de la création de l’abonnement aux événements.

Pour gérer les événements par programmation, vous pouvez les trier selon la valeur `operationName`. Par exemple, votre point de terminaison d’événement peut traiter uniquement les événements pour les opérations correspondant à `Microsoft.Compute/virtualMachines/write` ou `Microsoft.Storage/storageAccounts/write`.

L’objet de l’événement est l’ID de la ressource cible de l’opération. Pour filtrer les événements pour une ressource, fournissez cet ID de ressource lors de la création de l’abonnement aux événements. Pour filtrer selon un type de ressource, utilisez une valeur au format suivant : `/subscriptions/<subscription-id>/resourcegroups/<resource-group>/providers/Microsoft.Compute/virtualMachines`


## <a name="event-grid-event-schema"></a>Schéma d’événement Event Grid

### <a name="available-event-types"></a>Types d’événement disponibles

Les abonnements Azure émettent des événements de gestion à partir d’Azure Resource Manager, par exemple lors de la création d’une machine virtuelle ou de la suppression d’un compte de stockage.

| Type d'événement | Description |
| ---------- | ----------- |
| Microsoft.Resources.ResourceActionCancel | Déclenché quand une action sur une ressource est annulée. |
| Microsoft.Resources.ResourceActionFailure | Déclenché quand une action sur une ressource échoue. |
| Microsoft.Resources.ResourceActionSuccess | Déclenché quand une action sur une ressource réussit. |
| Microsoft.Resources.ResourceDeleteCancel | Déclenché quand une opération de suppression est annulée. Cet événement se produit lorsque le déploiement d’un modèle est annulé. |
| Microsoft.Resources.ResourceDeleteFailure | Déclenché quand une opération de suppression échoue. |
| Microsoft.Resources.ResourceDeleteSuccess | Déclenché quand une opération de suppression réussit. |
| Microsoft.Resources.ResourceWriteCancel | Déclenché quand une opération de création ou de mise à jour est annulée. |
| Microsoft.Resources.ResourceWriteFailure | Déclenché quand une opération de création ou de mise à jour échoue. |
| Microsoft.Resources.ResourceWriteSuccess | Déclenché quand une opération de création ou de mise à jour réussit. |

### <a name="example-event"></a>Exemple d’événement

L’exemple suivant montre le schéma d’un événement **ResourceWriteSuccess**. Le même schéma est utilisé pour les événements **ResourceWriteFailure** et **ResourceWriteCancel** avec différentes valeurs pour `eventType`.

```json
[{
  "subject": "/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
  "eventType": "Microsoft.Resources.ResourceWriteSuccess",
  "eventTime": "2018-07-19T18:38:04.6117357Z",
  "id": "4db48cba-50a2-455a-93b4-de41a3b5b7f6",
  "data": {
    "authorization": {
      "scope": "/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
      "action": "Microsoft.Storage/storageAccounts/write",
      "evidence": {
        "role": "Subscription Admin"
      }
    },
    "claims": {
      "aud": "{audience-claim}",
      "iss": "{issuer-claim}",
      "iat": "{issued-at-claim}",
      "nbf": "{not-before-claim}",
      "exp": "{expiration-claim}",
      "_claim_names": "{\"groups\":\"src1\"}",
      "_claim_sources": "{\"src1\":{\"endpoint\":\"{URI}\"}}",
      "http://schemas.microsoft.com/claims/authnclassreference": "1",
      "aio": "{token}",
      "http://schemas.microsoft.com/claims/authnmethodsreferences": "rsa,mfa",
      "appid": "{ID}",
      "appidacr": "2",
      "http://schemas.microsoft.com/2012/01/devicecontext/claims/identifier": "{ID}",
      "e_exp": "{expiration}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname": "{last-name}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname": "{first-name}",
      "ipaddr": "{IP-address}",
      "name": "{full-name}",
      "http://schemas.microsoft.com/identity/claims/objectidentifier": "{ID}",
      "onprem_sid": "{ID}",
      "puid": "{ID}",
      "http://schemas.microsoft.com/identity/claims/scope": "user_impersonation",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "{ID}",
      "http://schemas.microsoft.com/identity/claims/tenantid": "{ID}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name": "{user-name}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn": "{user-name}",
      "uti": "{ID}",
      "ver": "1.0"
    },
    "correlationId": "{ID}",
    "resourceProvider": "Microsoft.Storage",
    "resourceUri": "/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
    "operationName": "Microsoft.Storage/storageAccounts/write",
    "status": "Succeeded",
    "subscriptionId": "{subscription-id}",
    "tenantId": "{tenant-id}"
  },
  "dataVersion": "2",
  "metadataVersion": "1",
  "topic": "/subscriptions/{subscription-id}"
}]
```

L’exemple suivant montre le schéma d’un événement **ResourceDeleteSuccess**. Le même schéma est utilisé pour les événements **ResourceDeleteFailure** et **ResourceDeleteCancel** avec différentes valeurs pour `eventType`.

```json
[{
  "subject": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
  "eventType": "Microsoft.Resources.ResourceDeleteSuccess",
  "eventTime": "2018-07-19T19:24:12.763881Z",
  "id": "19a69642-1aad-4a96-a5ab-8d05494513ce",
  "data": {
    "authorization": {
      "scope": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
      "action": "Microsoft.Storage/storageAccounts/delete",
      "evidence": {
        "role": "Subscription Admin"
      }
    },
    "claims": {
      "aud": "{audience-claim}",
      "iss": "{issuer-claim}",
      "iat": "{issued-at-claim}",
      "nbf": "{not-before-claim}",
      "exp": "{expiration-claim}",
      "_claim_names": "{\"groups\":\"src1\"}",
      "_claim_sources": "{\"src1\":{\"endpoint\":\"{URI}\"}}",
      "http://schemas.microsoft.com/claims/authnclassreference": "1",
      "aio": "{token}",
      "http://schemas.microsoft.com/claims/authnmethodsreferences": "rsa,mfa",
      "appid": "{ID}",
      "appidacr": "2",
      "http://schemas.microsoft.com/2012/01/devicecontext/claims/identifier": "{ID}",
      "e_exp": "262800",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname": "{last-name}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname": "{first-name}",
      "ipaddr": "{IP-address}",
      "name": "{full-name}",
      "http://schemas.microsoft.com/identity/claims/objectidentifier": "{ID}",
      "onprem_sid": "{ID}",
      "puid": "{ID}",
      "http://schemas.microsoft.com/identity/claims/scope": "user_impersonation",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "{ID}",
      "http://schemas.microsoft.com/identity/claims/tenantid": "{ID}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name": "{user-name}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn": "{user-name}",
      "uti": "{ID}",
      "ver": "1.0"
    },
    "correlationId": "{ID}",
    "httpRequest": {
      "clientRequestId": "{ID}",
      "clientIpAddress": "{IP-address}",
      "method": "DELETE",
      "url": "https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}?api-version=2018-02-01"
    },
    "resourceProvider": "Microsoft.Storage",
    "resourceUri": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
    "operationName": "Microsoft.Storage/storageAccounts/delete",
    "status": "Succeeded",
    "subscriptionId": "{subscription-id}",
    "tenantId": "{tenant-id}"
  },
  "dataVersion": "2",
  "metadataVersion": "1",
  "topic": "/subscriptions/{subscription-id}"
}]
```

L’exemple suivant montre le schéma d’un événement **ResourceActionSuccess**. Le même schéma est utilisé pour les événements **ResourceActionFailure** et **ResourceActionCancel** avec différentes valeurs pour `eventType`.

```json
[{   
  "subject": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.EventHub/namespaces/{namespace}/AuthorizationRules/RootManageSharedAccessKey",
  "eventType": "Microsoft.Resources.ResourceActionSuccess",
  "eventTime": "2018-10-08T22:46:22.6022559Z",
  "id": "{ID}",
  "data": {
    "authorization": {
      "scope": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.EventHub/namespaces/{namespace}/AuthorizationRules/RootManageSharedAccessKey",
      "action": "Microsoft.EventHub/namespaces/AuthorizationRules/listKeys/action",
      "evidence": {
        "role": "Contributor",
        "roleAssignmentScope": "/subscriptions/{subscription-id}",
        "roleAssignmentId": "{ID}",
        "roleDefinitionId": "{ID}",
        "principalId": "{ID}",
        "principalType": "ServicePrincipal"
      }     
    },
    "claims": {
      "aud": "{audience-claim}",
      "iss": "{issuer-claim}",
      "iat": "{issued-at-claim}",
      "nbf": "{not-before-claim}",
      "exp": "{expiration-claim}",
      "aio": "{token}",
      "appid": "{ID}",
      "appidacr": "2",
      "http://schemas.microsoft.com/identity/claims/identityprovider": "{URL}",
      "http://schemas.microsoft.com/identity/claims/objectidentifier": "{ID}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "{ID}",       "http://schemas.microsoft.com/identity/claims/tenantid": "{ID}",
      "uti": "{ID}",
      "ver": "1.0"
    },
    "correlationId": "{ID}",
    "httpRequest": {
      "clientRequestId": "{ID}",
      "clientIpAddress": "{IP-address}",
      "method": "POST",
      "url": "https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.EventHub/namespaces/{namespace}/AuthorizationRules/RootManageSharedAccessKey/listKeys?api-version=2017-04-01"
    },
    "resourceProvider": "Microsoft.EventHub",
    "resourceUri": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.EventHub/namespaces/{namespace}/AuthorizationRules/RootManageSharedAccessKey",
    "operationName": "Microsoft.EventHub/namespaces/AuthorizationRules/listKeys/action",
    "status": "Succeeded",
    "subscriptionId": "{subscription-id}",
    "tenantId": "{tenant-id}"
  },
  "dataVersion": "2",
  "metadataVersion": "1",
  "topic": "/subscriptions/{subscription-id}" 
}]
```

### <a name="event-properties"></a>Propriétés d’événement

Un événement contient les données générales suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| topic | string | Chemin d’accès complet à la source de l’événement. Ce champ n’est pas modifiable. Event Grid fournit cette valeur. |
| subject | string | Chemin de l’objet de l’événement, défini par le serveur de publication. |
| eventType | string | Un des types d’événements inscrits pour cette source d’événement. |
| eventTime | string | L’heure à quelle l’événement est généré selon l’heure UTC du fournisseur. |
| id | string | Identificateur unique de l’événement. |
| data | object | Données d’événement d’abonnement. |
| dataVersion | string | Version du schéma de l’objet de données. Le serveur de publication définit la version du schéma. |
| metadataVersion | string | Version du schéma des métadonnées d’événement. Event Grid définit le schéma des propriétés de niveau supérieur. Event Grid fournit cette valeur. |

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| autorisation | object | Autorisation demandée pour l’opération. |
| réclamations | object | Propriétés des revendications. Pour en savoir plus, consultez la [Spécification JWT](https://self-issued.info/docs/draft-ietf-oauth-json-web-token.html). |
| correlationId | string | ID d’opération pour le dépannage. |
| httpRequest | object | Détails de l’opération. Cet objet est inclus uniquement lors de la mise à jour ou de la suppression d’une ressource existante. |
| resourceProvider | string | Fournisseur de ressources pour l’opération. |
| resourceUri | string | URI de la ressource dans l’opération. |
| operationName | string | Opération effectuée. |
| status | string | L’état de l’opération. |
| subscriptionId | string | ID d’abonnement de la ressource. |
| tenantId | string | ID de locataire de la ressource. |

## <a name="tutorials-and-how-tos"></a>Tutoriels et articles de procédures
|Intitulé |Description  |
|---------|---------|
| [Tutoriel : Intégrer Azure Automation à Event Grid et Microsoft Teams](ensure-tags-exists-on-new-virtual-machines.md) |Créez une machine virtuelle, qui envoie un événement. L’événement déclenche un runbook Automation qui balise la machine virtuelle et déclenche un message qui est envoyé à un canal Microsoft Teams. |
| [Guide pratique pour s’abonner aux événements via le portail](subscribe-through-portal.md) | Utilisez le portail pour vous abonner aux événements d’un abonnement Azure. |
| [Azure CLI : S’abonner aux événements d’un abonnement Azure](./scripts/event-grid-cli-azure-subscription.md) |Exemple de script qui crée un abonnement Event Grid à un abonnement Azure, et envoie les événements à un webhook. |
| [PowerShell : S’abonner aux événements d’un abonnement Azure](./scripts/event-grid-powershell-azure-subscription.md)| Exemple de script qui crée un abonnement Event Grid à un abonnement Azure, et envoie les événements à un webhook. |

## <a name="next-steps"></a>Étapes suivantes

* Pour découvrir Azure Event Grid, consultez [Présentation d’Event Grid](overview.md).
* Pour plus d’informations sur la création d’un abonnement Azure Event Grid, consultez [Schéma d’abonnement à Event Grid](subscription-creation-schema.md).
