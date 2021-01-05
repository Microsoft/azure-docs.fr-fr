---
title: Applications managées avec notifications
description: Configurez des applications managées avec des points de terminaison webhook pour recevoir des notifications concernant les créations, les mises à jour, les suppressions et les erreurs sur les instances des applications managées.
ms.topic: conceptual
ms.author: ilahat
author: ilahat
ms.date: 11/01/2019
ms.openlocfilehash: cec17b98daa8eca31cda076921288e2838960511
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96434530"
---
# <a name="azure-managed-applications-with-notifications"></a>Applications managées Azure avec notifications

Les notifications d’applications managées Azure permettent aux éditeurs d’automatiser des actions en fonction des événements du cycle de vie des instances de l’application managée. Les éditeurs peuvent spécifier des points de terminaison webhook de notification personnalisés pour recevoir des notifications d’événements sur les instances nouvelles et existantes de l’application managée. Les éditeurs peuvent configurer des workflows personnalisés au moment du provisionnement, des mises à jour et de la suppression de l’application.

## <a name="getting-started"></a>Prise en main
Pour commencer à recevoir des applications managées, lancez un point de terminaison HTTPS public et spécifiez-le lors de la publication de la définition de l’application du catalogue de services ou de l’offre de la Place de marché Azure.

Voici les étapes recommandées pour bien démarrer :
1. Lancez un point de terminaison HTTPS public qui journalise les requêtes POST entrantes et retourne `200 OK`.
2. Ajoutez le point de terminaison à la définition de l’application du catalogue de services ou à l’offre de la Place de marché Azure, comme expliqué plus loin dans cet article.
3. Créez une instance d’application managée qui fait référence à la définition de l’application ou à l’offre de la Place de marché Azure.
4. Vérifiez que les notifications sont bien reçues.
5. Activez l’autorisation, comme expliqué dans la section **Authentification du point de terminaison** plus loin dans cet article.
6. Suivez les instructions de la section **Schéma de notification** dans cet article pour analyser les requêtes de notification et implémenter votre logique métier basée sur la notification.

## <a name="add-service-catalog-application-definition-notifications"></a>Ajouter des notifications de définition de l’application du catalogue de services
#### <a name="azure-portal"></a>Portail Azure
Pour bien démarrer, lisez [Publier une application du catalogue de services en utilisant le portail Azure](./publish-portal.md).

![Notifications de définition de l’application du catalogue de services dans le portail Azure](./media/publish-notifications/service-catalog-notifications.png)

#### <a name="rest-api"></a>API REST

> [!NOTE]
> Actuellement, vous ne pouvez fournir qu’un seul point de terminaison dans `notificationEndpoints` dans les propriétés de définition d’application.

``` JSON
    {
      "properties": {
        "isEnabled": true,
        "lockLevel": "ReadOnly",
        "displayName": "Sample Application Definition",
        "description": "Notification-enabled application definition.",
        "notificationPolicy": {
            "notificationEndpoints": [
                {
                    "uri": "https://isv.azurewebsites.net:1214?sig=unique_token"
                }
            ]
        },
        "authorizations": [
          {
            "principalId": "d6b7fbd3-4d99-43fe-8a7a-f13aef11dc18",
            "roleDefinitionId": "8e3af657-a8ff-443c-a75c-2fe8c4bcb635"
          },
        ...

```
## <a name="add-azure-marketplace-managed-application-notifications"></a>Ajouter des notifications d’application managée de la Place de marché Azure
Pour plus d’informations, consultez [Créer une offre d’application Azure](../../marketplace/create-new-azure-apps-offer.md).

![Notifications d’application managée de la Place de marché Azure dans le portail Azure](./media/publish-notifications/marketplace-notifications.png)
## <a name="event-triggers"></a>Déclencheurs d’événement
Le tableau suivant décrit toutes les combinaisons possibles d’EventType et ProvisioningState et leurs déclencheurs :

Type d’événement | ProvisioningState | Déclencheur pour la notification
---|---|---
PUT | Acceptée | Le groupe de ressources managé a été créé et projeté correctement après l’opération PUT de l’application (avant le lancement du déploiement dans le groupe de ressources managé).
PUT | Opération réussie | Le provisionnement complet de l’application managée a réussi après une opération PUT.
PUT | Échec | Échec de l’opération PUT de provisionnement d’une instance d’application à tout moment.
PATCH | Opération réussie | Après une opération PATCH réussie sur l’instance d’application managée pour mettre à jour les balises, la stratégie d’accès JIT ou l’identité managée.
Suppression | En cours de suppression | Dès que l’utilisateur lance une opération DELETE d’une instance d’application managée.
Suppression | Deleted | Après la suppression complète et réussie de l’application managée.
Suppression | Échec | Après une erreur pendant le processus de déprovisionnement qui bloque la suppression.
## <a name="notification-schema"></a>Schéma de notification
Lorsque vous lancez votre point de terminaison de webhook pour gérer les notifications, vous devez analyser la charge utile pour obtenir des propriétés importantes afin de pouvoir agir sur la notification. Les notifications d’application managée de catalogue de services et de Place de marché Azure fournissent un grand nombre des mêmes propriétés. Deux petites différences sont présentées dans le tableau qui suit les exemples.

#### <a name="service-catalog-application-notification-schema"></a>Schéma de notification de l’application du catalogue de services
Voici un exemple de notification du catalogue de services après le provisionnement réussi d’une instance d’application managée :
``` HTTP
POST https://{your_endpoint_URI}/resource?{optional_parameter}={optional_parameter_value} HTTP/1.1

{
    "eventType": "PUT",
    "applicationId": "subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applications/<applicationName>",
    "eventTime": "2019-08-14T19:20:08.1707163Z",
    "provisioningState": "Succeeded",
    "applicationDefinitionId": "subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applicationDefinitions/<appDefName>"    
}

```

Si le provisionnement échoue, une notification avec les détails de l’erreur est envoyée au point de terminaison spécifié.

``` HTTP
POST https://{your_endpoint_URI}/resource?{optional_parameter}={optional_parameter_value} HTTP/1.1

{
    "eventType": "PUT",
    "applicationId": "subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applications/<applicationName>",
    "eventTime": "2019-08-14T19:20:08.1707163Z",
    "provisioningState": "Failed",
    "applicationDefinitionId": "subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applicationDefinitions/<appDefName>",
    "error": {
        "code": "ErrorCode",
        "message": "error message",
        "details": [
            {
                "code": "DetailedErrorCode",
                "message": "error message"
            }
        ]
    }
}

```

#### <a name="azure-marketplace-application-notification-schema"></a>Schéma de notification de l’application de la Place de marché Azure

Voici un exemple de notification du catalogue de services après le provisionnement réussi d’une instance d’application managée :
``` HTTP
POST https://{your_endpoint_URI}/resource?{optional_parameter}={optional_parameter_value} HTTP/1.1

{
    "eventType": "PUT",
    "applicationId": "subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applications/<applicationName>",
    "eventTime": "2019-08-14T19:20:08.1707163Z",
    "provisioningState": "Succeeded",
    "billingDetails": {
        "resourceUsageId":"<resourceUsageId>"
    },
    "plan": {
        "publisher": "publisherId",
        "product": "offer",
        "name": "skuName",
        "version": "1.0.1"
    }
}

```

Si le provisionnement échoue, une notification avec les détails de l’erreur est envoyée au point de terminaison spécifié.

``` HTTP
POST https://{your_endpoint_URI}/resource?{optional_parameter}={optional_parameter_value} HTTP/1.1

{
    "eventType": "PUT",
    "applicationId": "subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applications/<applicationName>",
    "eventTime": "2019-08-14T19:20:08.1707163Z",
    "provisioningState": "Failed",
    "billingDetails": {
        "resourceUsageId":"<resourceUsageId>"
    },
    "plan": {
        "publisher": "publisherId",
        "product": "offer",
        "name": "skuName",
        "version": "1.0.1"
    },
    "error": {
        "code": "ErrorCode",
        "message": "error message",
        "details": [
            {
                "code": "DetailedErrorCode",
                "message": "error message"
            }
        ]
    }
}

```

Paramètre | Description
---|---
eventType | Type d’événement ayant déclenché la notification. (Par exemple PUT, PATCH et DELETE.)
applicationId | Identificateur de ressource complet de l’application managée pour laquelle la notification a été déclenchée.
eventTime | Horodatage de l’événement ayant déclenché la notification. (Date et heure au format UTC ISO 8601.)
provisioningState | État de provisionnement de l’instance d’application managée. (Par exemple Succeeded, Failed, Deleting, Deleted.)
error | *Spécifié uniquement quand provisioningState est égal à Failed*. Contient le code d’erreur, le message et les détails relatifs au problème à l’origine de l’échec.
applicationDefinitionId | *Spécifié uniquement pour les applications managées du catalogue de services*. Représente l’identificateur de ressource complet de la définition d’application pour laquelle l’instance d’application managée a été provisionnée.
Plan | *Spécifié uniquement pour les applications managées de la Place de marché Azure*. Représente l’éditeur, l’offre, la référence (SKU) et la version de l’instance de l’application managée.
billingDetails | *Spécifié uniquement pour les applications managées de la Place de marché Azure.* Détails de facturation de l’instance d’application managée. Contient le resourceUsageId que vous pouvez utiliser pour interroger la Place de marché Azure afin d’obtenir des détails sur l’utilisation.

## <a name="endpoint-authentication"></a>Authentification du point de terminaison
Pour sécuriser le point de terminaison webhook et garantir l’authenticité de la notification :
1. Fournissez un paramètre de requête au-dessus de l’URI webhook, comme ceci : https\://your-endpoint.com?sig=Guid. Avec chaque notification, vérifiez que le paramètre de requête `sig` a la valeur attendue `Guid`.
2. Émettez une opération GET sur l’instance d’application managée à l’aide d’applicationId. Vérifiez que provisioningState correspond au provisioningState de la notification pour garantir la cohérence.

## <a name="notification-retries"></a>Nouvelles tentatives de notification

Le service de notification d’application managée attend une réponse `200 OK` du point de terminaison webhook à la notification. Le service de notification effectue une nouvelle tentative si le point de terminaison webhook retourne un code d’erreur HTTP supérieur ou égal à 500, s’il retourne un code d’erreur 429 ou si le point de terminaison est temporairement inaccessible. Si le point de terminaison webhook n’est pas disponible dans un délai de 10 heures, le message de notification est abandonné et les nouvelles tentatives s’arrêtent.
