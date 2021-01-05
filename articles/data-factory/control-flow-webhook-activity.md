---
title: Activité Webhook dans Azure Data Factory
description: L’activité de webhook interrompt l’exécution du pipeline jusqu’à ce qu’elle valide le jeu de données attaché avec certains critères spécifiés par l’utilisateur.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 03/25/2019
ms.openlocfilehash: 144006c3d0722bc3211f542b7059bba0bb0cbdbf
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499405"
---
# <a name="webhook-activity-in-azure-data-factory"></a>Activité Webhook dans Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Une activité de webhook peut contrôler l’exécution des pipelines dans votre code personnalisé. Grâce à l’activité de webhook, le code des clients peut appeler un point de terminaison et lui passer une URL de rappel. L’exécution du pipeline attend l’appel du rappel avant de passer à l’activité suivante.

## <a name="syntax"></a>Syntaxe

```json

{
    "name": "MyWebHookActivity",
    "type": "WebHook",
    "typeProperties": {
        "method": "POST",
        "url": "<URLEndpoint>",
        "headers": {
            "Content-Type": "application/json"
        },
        "body": {
            "key": "value"
        },
        "timeout": "00:03:00",
        "authentication": {
            "type": "ClientCertificate",
            "pfx": "****",
            "password": "****"
        }
    }
}

```

## <a name="type-properties"></a>Propriétés type

Propriété | Description | Valeurs autorisées | Obligatoire
-------- | ----------- | -------------- | --------
**name** | Nom de l’activité de webhook. | String | Oui |
**type** | Doit avoir la valeur « WebHook ». | String | Oui |
**method** | Méthode d’API REST pour le point de terminaison cible. | Chaîne. Le type pris en charge est « POST ». | Oui |
**url** | Point de terminaison cible et chemin d’accès. | Chaîne ou expression avec la valeur **resultType** d’une chaîne. | Oui |
**headers** | En-têtes envoyés à la demande. Voici un exemple qui définit la langue et le type d’une demande : `"headers" : { "Accept-Language": "en-us", "Content-Type": "application/json" }`. | Chaîne ou expression avec la valeur **resultType** d’une chaîne. | Oui. Un en-tête `Content-Type` comme `"headers":{ "Content-Type":"application/json"}` est requis. |
**body** | Représente la charge utile envoyée au point de terminaison. | Code JSON valide ou expression avec la valeur **resultType** du code JSON. Consultez [Schéma de charge utile de demande](./control-flow-web-activity.md#request-payload-schema) pour obtenir le schéma de la charge utile de demande. | Oui |
**authentification** | Méthode d’authentification utilisée pour appeler le point de terminaison. Les types pris en charge sont « Basic » et « ClientCertificate ». Pour en savoir plus, consultez [Authentification](./control-flow-web-activity.md#authentication). Si l’authentification n’est pas obligatoire, excluez cette propriété. | Chaîne ou expression avec la valeur **resultType** d’une chaîne. | Non |
**timeout** | Durée pendant laquelle l’activité attend que le rappel spécifié par **callBackUri** soit appelé. La valeur par défaut est 10 minutes (« 00:10:00 »). Les valeurs ont le format TimeSpan *j*.*hh*:*mm*:*ss*. | String | Non |
**Signaler l’état lors du rappel** | Permet à un utilisateur de signaler l’état d’échec d’une activité de webhook. | Boolean | Non |

## <a name="authentication"></a>Authentification

Une activité de webhook prend en charge les types d’authentification suivants.

### <a name="none"></a>None

Si l’authentification n’est pas requise, n’incluez pas la propriété **authentication**.

### <a name="basic"></a>De base

Spécifiez le nom d’utilisateur et le mot de passe à utiliser avec l’authentification de base.

```json
"authentication":{
   "type":"Basic",
   "username":"****",
   "password":"****"
}
```

### <a name="client-certificate"></a>Certificat client

Spécifiez le contenu encodé en Base64 d’un fichier PFX et un mot de passe.

```json
"authentication":{
   "type":"ClientCertificate",
   "pfx":"****",
   "password":"****"
}
```

### <a name="managed-identity"></a>Identité managée

Utilisez l’identité managée de la fabrique de données pour spécifier l’URI de ressource pour lequel le jeton d’accès est demandé. Pour appeler l’API Gestion des ressources Azure, utilisez `https://management.azure.com/`. Pour plus d’informations sur le fonctionnement des identités managées, consultez la [vue d’ensemble des identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/overview.md).

```json
"authentication": {
    "type": "MSI",
    "resource": "https://management.azure.com/"
}
```

> [!NOTE]
> Si votre fabrique de données est configurée avec un dépôt Git, vous devez stocker vos informations d’identification dans Azure Key Vault pour utiliser l’authentification de base ou de certificat client. Azure Data Factory ne stocke pas les mots de passe dans Git.

## <a name="additional-notes"></a>Remarques supplémentaires

Data Factory transmet la propriété supplémentaire **callBackUri** dans le corps envoyé au point de terminaison de l’URL. Data Factory s’attend à ce que cet URI soit appelé avant la valeur de délai d’attente spécifiée. Si l’URI n’est pas appelé, l’activité échoue avec l’état « TimedOut ».

L’activité de webhook échoue quand l’appel au point de terminaison personnalisé échoue. Tout message d’erreur peut être ajouté au corps du rappel et utilisé dans une activité ultérieure.

Pour chaque appel d’API REST, le client expire si le point de terminaison ne répond pas dans un délai d’une minute. Ce comportement est une bonne pratique standard pour le protocole HTTP. Pour résoudre ce problème, implémentez un modèle 202. Dans le cas actuel, le point de terminaison retourne 202 (accepté) et le client interroge.

Le délai d’expiration d’une minute de la demande n’a rien à voir avec le délai d’expiration de l’activité. Ce dernier est utilisé pour attendre le rappel spécifié par **callbackUri**.

Le corps retransmis à l’URI de rappel doit être du code JSON valide. Attribuez à l’en-tête `Content-Type` la valeur `application/json`.

Si vous utilisez la propriété **Signaler l’état lors du rappel**, vous devez ajouter le code suivant au corps quand vous effectuez le rappel :

```json
{
    "Output": {
        // output object is used in activity output
        "testProp": "testPropValue"
    },
    "Error": {
        // Optional, set it when you want to fail the activity
        "ErrorCode": "testErrorCode",
        "Message": "error message to show in activity error"
    },
    "StatusCode": "403" // when status code is >=400, activity is marked as failed
}
```

## <a name="next-steps"></a>Étapes suivantes

Consultez les activités de flux de contrôle suivantes prises en charge par Data Factory :

- [Activité IfCondition](control-flow-if-condition-activity.md)
- [Activité d’exécution du pipeline](control-flow-execute-pipeline-activity.md)
- [Pour chaque activité](control-flow-for-each-activity.md)
- [Activité d’obtention des métadonnées](control-flow-get-metadata-activity.md)
- [Activité de recherche](control-flow-lookup-activity.md)
- [Activité Web](control-flow-web-activity.md)
- [Activité jusqu’à](control-flow-until-activity.md)