---
title: Stratégies d’intégration Dapr de Gestion des API Azure | Microsoft Docs
description: Apprenez-en davantage sur les stratégies de Gestion des API Azure pour interagir avec des extensions de microservices Dapr.
author: vladvino
ms.author: vlvinogr
ms.date: 10/23/2020
ms.topic: article
ms.service: api-management
ms.openlocfilehash: 16788e3f547c5848893ba3867da4291c45b04408
ms.sourcegitcommit: 2a8a53e5438596f99537f7279619258e9ecb357a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94335483"
---
# <a name="api-management-dapr-integration-policies"></a>Stratégies d’intégration Dapr de Gestion des API

Cette rubrique est une ressource de référence pour les stratégies de Gestion des API en matière d’intégration Dapr. Dapr est un runtime portable permettant de créer des applications avec et sans état basées sur des microservices, avec n’importe quel langage ou infrastructure. Il codifie les modèles de microservice courants, tels que la découverte et l’appel de service avec une logique de nouvelle tentative intégrée, la publication et l’abonnement avec une sémantique de remise au moins une fois, ou des ressources de liaison enfichables pour faciliter la composition à l’aide de services externes. Accédez à [dapr.io](https://dapr.io) pour obtenir des informations et instructions détaillées sur la prise en main de Dapr. Pour plus d'informations sur l'ajout et la configuration des stratégies, consultez la page [Stratégies dans Gestion des API](api-management-howto-policies.md).

> [!CAUTION]
> Les stratégies référencées dans cette rubrique sont en préversion publique et peuvent faire l’objet d’un [Avenant aux conditions d’utilisation pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

> [!IMPORTANT]
> Les stratégies référencées dans cette rubrique fonctionnent uniquement dans la [version auto-hébergée de la passerelle de Gestion des API](self-hosted-gateway-overview.md) avec prise en charge de Dapr activée.

## <a name="enable-dapr-support-in-the-self-hosted-gateway"></a>Activer la prise en charge de Dapr dans la passerelle auto-hébergée

Pour activer la prise en charge de Dapr dans la passerelle auto-hébergée, ajoutez les [annotations Dapr](https://github.com/dapr/docs/blob/master/README.md) ci-dessous au [modèle de déploiement Kubernetes](how-to-deploy-self-hosted-gateway-kubernetes.md) en remplaçant « app-name » par le nom de votre choix. La procédure pas à pas complète de la configuration et de l’utilisation de la Gestion des API avec Dapr est disponible [ici](https://aka.ms/apim/dapr/walkthru).
```yml
template:
    metadata:
      labels:
        app: app-name
      annotations:
        dapr.io/enabled: "true"
        dapr.io/app-id: "app-name"
```


## <a name="distributed-application-runtime-dapr-integration-policies"></a>Stratégies d’intégration Distributed Application Runtime (Dapr)

-  [Envoyer une demande à un service](api-management-dapr-policies.md#invoke) : utilise le runtime Dapr pour localiser et communiquer de manière fiable avec un microservice Dapr. Pour en savoir plus sur l’appel de service dans Dapr, consultez la description fournie dans ce fichier [LISEZMOI](https://github.com/dapr/docs/blob/master/README.md#service-invocation).
-  [Envoyer un message à une rubrique Publier/S’abonner](api-management-dapr-policies.md#pubsub) : utilise le runtime Dapr pour publier un message dans une rubrique Publier/S’abonner. Pour plus d’informations sur les messages Publier/S’abonner dans Dapr, consultez la description fournie dans ce fichier [LISEZMOI](https://github.com/dapr/docs/blob/master/README.md).
-  [Déclencher la liaison de sortie](api-management-dapr-policies.md#bind) : utilise le runtime Dapr pour appeler un système externe par le biais d’une liaison de sortie. Pour en savoir plus sur les liaisons dans Dapr, consultez la description fournie dans ce fichier [LISEZMOI](https://github.com/dapr/docs/blob/master/README.md).

## <a name="send-request-to-a-service"></a><a name="invoke"></a> Envoyer une demande à un service

Cette stratégie définit l’URL cible de la demande actuelle sur `http://localhost:3500/v1.0/invoke/{app-id}/method/{method-name}`, en remplaçant les paramètres de modèle par les valeurs spécifiées dans la déclaration de stratégie.

La stratégie part du principe que Dapr s’exécute dans un conteneur sidecar se trouvant dans le même pod que la passerelle. Lors de la réception de la demande, le runtime Dapr effectue la détection du service et l’appel réel, en ce compris la traduction de protocole possible entre HTTP et gRPC, les nouvelles tentatives, le suivi distribué et la gestion des erreurs.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<set-backend-service backend-id="dapr" dapr-app-id="app-id" dapr-method="method-name" />
```

### <a name="examples"></a>Exemples

#### <a name="example"></a>Exemple

L’exemple suivant illustre l’appel de la méthode nommée « back » sur le microservice nommé « echo ». La stratégie `set-backend-service` définit l’URL de destination. La stratégie `forward-request` distribue la demande au runtime Dapr, qui le remet au microservice.

La stratégie `forward-request` est présentée ici par souci de clarté. La stratégie est généralement « héritée » de l’étendue globale via le mot clé `base`.

```xml
<policies>
    <inbound>
        <base />
        <set-backend-service backend-id="dapr" dapr-app-id="echo" dapr-method="back" />
    </inbound>
    <backend>
        <forward-request />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```

### <a name="elements"></a>Éléments

| Élément             | Description  | Obligatoire |
|---------------------|--------------|----------|
| set-backend-service | Élément racine | Oui      |

### <a name="attributes"></a>Attributs

| Attribut        | Description                     | Obligatoire | Default |
|------------------|---------------------------------|----------|---------|
| id de principal       | Doit être défini sur « dapr »           | Oui      | N/A     |
| dapr-app-id      | Nom du microservice cible. Correspond au paramètre [appId](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/service_invocation_api.md) dans Dapr.| Oui | N/A |
| dapr-method      | Nom de la méthode ou URL à appeler sur le microservice cible. Correspond au paramètre [method-name](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/service_invocation_api.md) dans Dapr.| Oui | N/A |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

- **Sections de la stratégie :** inbound
- **Étendues de la stratégie :** toutes les étendues

## <a name="send-message-to-pubsub-topic"></a><a name="pubsub"></a> Envoyer un message à une rubrique Publier/S’abonner

Cette stratégie indique à la passerelle de Gestion des API d’envoyer un message à une rubrique Publier/S’abonner de Dapr. La stratégie accomplit cela en adressant une requête HTTP POST à `http://localhost:3500/v1.0/publish/{{pubsub-name}}/{{topic}}` en remplaçant les paramètres du modèle et en ajoutant le contenu spécifié dans la déclaration de stratégie.

La stratégie part du principe que le runtime Dapr s’exécute dans un conteneur sidecar se trouvant dans le même pod que la passerelle. Le runtime Dapr implémente la sémantique Publier/S’abonner.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<publish-to-dapr pubsub-name="pubsub-name" topic=”topic-name” ignore-error="false|true" response-variable-name="resp-var-name" timeout="in seconds" template=”Liquid” content-type="application/json">
    <!-- message content -->
</publish-to-dapr>
```

### <a name="examples"></a>Exemples

#### <a name="example"></a>Exemple

L’exemple suivant montre comment envoyer le corps de la demande actuelle à la [rubrique](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/pubsub_api.md#url-parameters) « new » du [composant](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/pubsub_api.md#url-parameters) Publier/S’abonner d’« orders ». La réponse reçue du runtime Dapr est stockée dans l’entrée « dapr-response » de la collection Variables dans l’objet [contexte](api-management-policy-expressions.md#ContextVariables).

Si le runtime Dapr ne peut pas localiser la rubrique cible, par exemple, et répond avec une erreur, la section « on-error » est déclenchée. La réponse reçue du runtime Dapr est renvoyée au verbatim de l’appelant. Dans le cas contraire, la réponse `200 OK` par défaut est retournée.

La section « backend » est vide et la demande n’est pas transférée au serveur principal.

```xml
<policies>
     <inbound>
        <base />
        <publish-to-dapr
           pubsub-name="orders"
               topic="new"
               response-variable-name="dapr-response">
            @(context.Request.Body.As<string>())
        </publish-to-dapr>
    </inbound>
    <backend>
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
        <return-response response-variable-name="pubsub-response" />
    </on-error>
</policies>
```

### <a name="elements"></a>Éléments

| Élément             | Description  | Obligatoire |
|---------------------|--------------|----------|
| publish-to-dapr     | Élément racine | Oui      |

### <a name="attributes"></a>Attributs

| Attribut        | Description                     | Obligatoire | Default |
|------------------|---------------------------------|----------|---------|
| pubsub-name      | Nom du composant PubSub cible. Correspond au paramètre [pubsubname](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/pubsub_api.md) dans Dapr. À défaut, la valeur de l’attribut __rubric__ doit se présenter sous la forme `pubsub-name/topic-name`.    | Non       | None    |
| topic            | Nom de la rubrique. Correspond au paramètre [topic](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/pubsub_api.md) dans Dapr.               | Oui      | N/A     |
| ignore-error     | Si cette valeur est définie sur `true`, elle indique à la stratégie de ne pas déclencher la section [« on-error »](api-management-error-handling-policies.md) lors de la réception d’une erreur du runtime Dapr | Non | `false` |
| response-variable-name | Nom de l’entrée de collection [Variables](api-management-policy-expressions.md#ContextVariables) à utiliser pour stocker la réponse du runtime Dapr | Non | None |
| délai d'expiration | Durée (en secondes) d’attente de la réponse du runtime Dapr. Peut être comprise entre 1 et 240 secondes. | Non | 5 |
| template | Moteur de création de modèles à utiliser pour transformer le contenu du message. « Liquid » est la seule valeur prise en charge. | Non | None |
| content-type | Type de contenu du message. « application/json » est la seule valeur prise en charge. | Non | None |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

- **Sections de la stratégie :** inbound, outbound, on-error
- **Étendues de la stratégie :** toutes les étendues

## <a name="trigger-output-binding"></a><a name="bind"></a> Liaison de sortie du déclencheur

Cette stratégie indique à la passerelle de Gestion des API de déclencher une [liaison](https://github.com/dapr/docs/blob/master/README.md) Dapr sortante. La stratégie accomplit cela en adressant une requête HTTP POST à `http://localhost:3500/v1.0/bindings/{{bind-name}}` en remplaçant le paramètre du modèle et en ajoutant le contenu spécifié dans la déclaration de stratégie.

La stratégie part du principe que le runtime Dapr s’exécute dans un conteneur sidecar se trouvant dans le même pod que la passerelle. Le runtime Dapr est chargé d’appeler la ressource externe représentée par la liaison.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<invoke-dapr-binding name=”bind-name" operation="op-name" ignore-error="false|true" response-variable-name="resp-var-name" timeout="in seconds" template=”Liquid content-type="application/json">
    <metadata>
        <item key=”item-name”><!-- item-value --></item>
    </metadata>
    <data>
        <!-- message content -->
    </data>
</invoke-dapr-binding>
```

### <a name="examples"></a>Exemples

#### <a name="example"></a>Exemple

L’exemple suivant illustre le déclenchement d’une liaison sortante nommée « external-systems » avec le nom d’opération « create », les métadonnées étant composées de deux éléments clé/valeur nommés « source » et « client-ip », et le corps provenant de la demande d’origine. La réponse reçue du runtime Dapr est capturée dans l’entrée « bind-response » de la collection Variables dans l’objet [context](api-management-policy-expressions.md#ContextVariables).

Si le runtime Dapr échoue pour une raison quelconque et répond avec une erreur, la section « on-error » est déclenchée et la réponse reçue du runtime Dapr est renvoyée au verbatim de l’appelant. Dans le cas contraire, la réponse `200 OK` par défaut est retournée.

La section « backend » est vide et la demande n’est pas transférée au serveur principal.

```xml
<policies>
     <inbound>
        <base />
        <invoke-dapr-binding
                      name="external-system"
                      operation="create"
                      response-variable-name="bind-response">
            <metadata>
                <item key="source">api-management</item>
                <item key="client-ip">@( context.Request.IpAddress )</item>
            </metadata>
            <data>
                @( context.Request.Body.As<string>() )
            </data>
        </invoke-dapr-binding>
    </inbound>
    <backend>
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
        <return-response response-variable-name="bind-response" />
    </on-error>
</policies>
```

### <a name="elements"></a>Éléments

| Élément             | Description  | Obligatoire |
|---------------------|--------------|----------|
| invoke-dapr-binding | Élément racine | Oui      |
| metadata            | Liaison de métadonnées spécifiques sous la forme de paires clé/valeur. Correspond à la propriété [metadata](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/bindings_api.md#invoking-output-bindings) dans Dapr. | Non |
| Données            | Contenu du message. Correspond à la propriété [data](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/bindings_api.md#invoking-output-bindings) dans Dapr. | Non |


### <a name="attributes"></a>Attributs

| Attribut        | Description                     | Obligatoire | Default |
|------------------|---------------------------------|----------|---------|
| name            | Nom de la liaison cible. Doit correspondre au nom des liaisons [définies](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/bindings_api.md#bindings-structure) dans Dapr.           | Oui      | N/A     |
| operation       | Nom de l’opération cible (spécifique de la liaison). Correspond à la propriété [operation](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/bindings_api.md#invoking-output-bindings) dans Dapr. | Non | None |
| ignore-error     | Si cette valeur est définie sur `true`, elle indique à la stratégie de ne pas déclencher la section [« on-error »](api-management-error-handling-policies.md) lors de la réception d’une erreur du runtime Dapr | Non | `false` |
| response-variable-name | Nom de l’entrée de collection [Variables](api-management-policy-expressions.md#ContextVariables) à utiliser pour stocker la réponse du runtime Dapr | Non | None |
| délai d'expiration | Durée (en secondes) d’attente de la réponse du runtime Dapr. Peut être comprise entre 1 et 240 secondes. | Non | 5 |
| template | Moteur de création de modèles à utiliser pour transformer le contenu du message. « Liquid » est la seule valeur prise en charge. | Non | None |
| content-type | Type de contenu du message. « application/json » est la seule valeur prise en charge. | Non | None |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

- **Sections de la stratégie :** inbound, outbound, on-error
- **Étendues de la stratégie :** toutes les étendues
