---
title: Stratégies avancées de la Gestion des API Azure | Microsoft Docs
description: Découvrez les stratégies avancées disponibles dans la Gestion des API Azure. Consultez des exemples et affichez des ressources disponibles supplémentaires.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/13/2020
ms.author: apimpm
ms.openlocfilehash: 3a37cde79cef59eaf9c3ef130bfbae9cff958bd7
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96919438"
---
# <a name="api-management-advanced-policies"></a>Stratégies avancées de la Gestion des API

Cette rubrique est une ressource de référence au sujet des stratégies Gestion des API suivantes. Pour plus d'informations sur l'ajout et la configuration des stratégies, consultez la page [Stratégies dans Gestion des API](./api-management-policies.md).

## <a name="advanced-policies"></a><a name="AdvancedPolicies"></a> Stratégies avancées

-   [Control flow](api-management-advanced-policies.md#choose) : applique de manière conditionnelle les instructions des stratégies en fonction des résultats de l’évaluation des [expressions](api-management-policy-expressions.md) booléennes.
-   [Forward request](#ForwardRequest) : transfère la demande vers le service principal.
-   [Limit concurrency](#LimitConcurrency) : empêche les stratégies incluses d’exécuter plus de requêtes simultanées que le nombre spécifié.
-   [Log to Event Hub](#log-to-eventhub) : envoie des messages au format spécifié à un Event Hub défini par une entité Enregistreur d’événements.
-   [Mock response](#mock-response) : abandonne l’exécution du pipeline et renvoie une réponse factice indiquée directement à l’appelant.
-   [Retry](#Retry) : effectue une nouvelle tentative d’exécution des instructions de stratégie incluses, si la condition est remplie et jusqu’à ce qu’elle le soit. L’exécution se répète à intervalles réguliers et ce jusqu’au nombre de tentatives défini.
-   [Return response](#ReturnResponse) : abandonne l’exécution du pipeline et renvoie la réponse indiquée directement à l’appelant.
-   [Send one way request](#SendOneWayRequest) : envoie une demande à l’URL indiquée sans attendre de réponse.
-   [Send request](#SendRequest) : envoie une demande à l’URL indiquée.
-   [Définir le proxy HTTP](#SetHttpProxy) : vous permet de router les demandes transférées via un proxy HTTP.
-   [Set request method](#SetRequestMethod) : permet de modifier la méthode HTTP d’une demande.
-   [Set status code](#SetStatus) : permet de donner la valeur spécifiée au code d’état HTTP.
-   [Set variable](api-management-advanced-policies.md#set-variable) : conserve une valeur dans une variable de [contexte](api-management-policy-expressions.md#ContextVariables) nommée pour permettre d’y accéder ultérieurement.
-   [Trace](#Trace) : ajoute des traces personnalisées à la sortie [API Inspector](./api-management-howto-api-inspector.md), aux données de télémétrie Application Insights et aux journaux de ressources.
-   [Wait](#Wait) : attend l’exécution des stratégies [Send request](api-management-advanced-policies.md#SendRequest), [Get value from cache](api-management-caching-policies.md#GetFromCacheByKey) ou [Control flow](api-management-advanced-policies.md#choose) pour continuer.

## <a name="control-flow"></a><a name="choose"></a> Control flow

La stratégie `choose` applique les instructions de stratégie incluses en fonction du résultat de l’évaluation d’expressions booléennes, de façon similaire à une construction de type if-then-else ou commutateur dans un langage de programmation.

### <a name="policy-statement"></a><a name="ChoosePolicyStatement"></a> Instruction de la stratégie

```xml
<choose>
    <when condition="Boolean expression | Boolean constant">
        <!— one or more policy statements to be applied if the above condition is true  -->
    </when>
    <when condition="Boolean expression | Boolean constant">
        <!— one or more policy statements to be applied if the above condition is true  -->
    </when>
    <otherwise>
        <!— one or more policy statements to be applied if none of the above conditions are true  -->
</otherwise>
</choose>
```

La stratégie de flux de contrôle doit contenir au moins un élément `<when/>`. L’élément `<otherwise/>` est facultatif. Les conditions dans les éléments `<when/>` sont évaluées par ordre d’apparition dans la stratégie. La ou les déclarations de stratégie incluses dans le premier élément `<when/>` avec attribut de condition égal à `true` seront appliquées. Les stratégies incluses dans l’élément `<otherwise/>`, le cas échéant, seront appliquées si tous les attributs de condition de l’élément `<when/>` sont `false`.

### <a name="examples"></a>Exemples

#### <a name="example"></a><a name="ChooseExample"></a> Exemple

L'exemple suivant présente une stratégie [set-variable](api-management-advanced-policies.md#set-variable) et deux stratégies de flux de contrôle.

La stratégie set variable se trouve dans la section inbound et crée une variable de [contexte](api-management-policy-expressions.md#ContextVariables)`isMobile` booléenne qui a la valeur true si l’en-tête de demande `User-Agent` contient le texte `iPad` ou `iPhone`.

La première stratégie de flux de contrôle se trouve également dans la section inbound et applique de manière conditionnelle une des deux stratégies [Set query string parameter](api-management-transformation-policies.md#SetQueryStringParameter) selon la valeur de la variable de contexte `isMobile`.

La deuxième stratégie de flux de contrôle se trouve dans la section outbound et applique de manière conditionnelle la stratégie [Convert XML to JSON](api-management-transformation-policies.md#ConvertXMLtoJSON) si `isMobile` a la valeur `true`.

```xml
<policies>
    <inbound>
        <set-variable name="isMobile" value="@(context.Request.Headers.GetValueOrDefault("User-Agent","").Contains("iPad") || context.Request.Headers.GetValueOrDefault("User-Agent","").Contains("iPhone"))" />
        <base />
        <choose>
            <when condition="@(context.Variables.GetValueOrDefault<bool>("isMobile"))">
                <set-query-parameter name="mobile" exists-action="override">
                    <value>true</value>
                </set-query-parameter>
            </when>
            <otherwise>
                <set-query-parameter name="mobile" exists-action="override">
                    <value>false</value>
                </set-query-parameter>
            </otherwise>
        </choose>
    </inbound>
    <outbound>
        <base />
        <choose>
            <when condition="@(context.Variables.GetValueOrDefault<bool>("isMobile"))">
                <xml-to-json kind="direct" apply="always" consider-accept-header="false"/>
            </when>
        </choose>
    </outbound>
</policies>
```

#### <a name="example"></a>Exemple

Cet exemple montre comment effectuer un filtrage du contenu en supprimant des éléments de données de la réponse reçue du service principal en cas d’utilisation du produit `Starter`. Pour une démonstration de la configuration et de l’utilisation de cette stratégie, consultez [Cloud Cover Episode 177: More API Management Features with Vlad Vinogradsky](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/) (Cloud Cover, épisode 177 : Plus de fonctionnalités de la Gestion des API avec Vlad Vinogradsky) et rendez-vous directement à 34 min 30 s. Commencez à 31 min 50 s pour voir une présentation de [l’API The Dark Sky Forecast](https://developer.forecast.io/) utilisée pour cette démonstration.

```xml
<!-- Copy this snippet into the outbound section to remove a number of data elements from the response received from the backend service based on the name of the api product -->
<choose>
  <when condition="@(context.Response.StatusCode == 200 && context.Product.Name.Equals("Starter"))">
    <set-body>@{
        var response = context.Response.Body.As<JObject>();
        foreach (var key in new [] {"minutely", "hourly", "daily", "flags"}) {
          response.Property (key).Remove ();
        }
        return response.ToString();
      }
    </set-body>
  </when>
</choose>
```

### <a name="elements"></a>Éléments

| Élément   | Description                                                                                                                                                                                                                                                               | Obligatoire |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| choose    | Élément racine.                                                                                                                                                                                                                                                             | Oui      |
| when      | Condition à utiliser pour les parties `if` ou `ifelse` de la stratégie `choose`. Si la stratégie `choose` possède plusieurs sections `when`, elles sont évaluées de façon séquentielle. Une fois la `condition` d’un élément when évaluée à `true`, aucune autre condition `when` n’est évaluée. | Oui      |
| otherwise | Contient l’extrait de stratégie à utiliser si aucune des conditions `when` n’est évaluée à `true`.                                                                                                                                                                               | Non       |

### <a name="attributes"></a>Attributs

| Attribut                                              | Description                                                                                               | Obligatoire |
| ------------------------------------------------------ | --------------------------------------------------------------------------------------------------------- | -------- |
| condition="Boolean expression &#124; Boolean constant" | Constante ou expression booléenne à évaluer lorsque la déclaration de stratégie `when` qui l’englobe est évaluée. | Oui      |

### <a name="usage"></a><a name="ChooseUsage"></a> Utilisation

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound, backend, on-error

-   **Étendues de la stratégie :** toutes les étendues

## <a name="forward-request"></a><a name="ForwardRequest"></a> Forward request

La stratégie `forward-request` transfère la demande entrante au service principal spécifié dans le [contexte](api-management-policy-expressions.md#ContextVariables) de la demande. L’URL du service back-end est spécifiée dans les [paramètres](./import-and-publish.md) de l’API et peut être modifiée à l’aide de la stratégie [set backend service](api-management-transformation-policies.md).

> [!NOTE]
> En cas de suppression cette stratégie, la demande n’est pas transférée au service principal et les stratégies de la section outbound sont évaluées immédiatement après la réussite des stratégies de la section inbound.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<forward-request timeout="time in seconds" follow-redirects="false | true" buffer-request-body="false | true" buffer-response="true | false" fail-on-error-status-code="false | true"/>
```

### <a name="examples"></a>Exemples

#### <a name="example"></a>Exemple

La stratégie au niveau de l’API suivante transfère toutes les demandes d’API au service back-end avec un délai d’expiration de 60 secondes.

```xml
<!-- api level -->
<policies>
    <inbound>
        <base/>
    </inbound>
    <backend>
        <forward-request timeout="60"/>
    </backend>
    <outbound>
        <base/>
    </outbound>
</policies>

```

#### <a name="example"></a>Exemple

Cette stratégie au niveau de l’opération utilise l’élément `base` pour hériter de la stratégie backend de l’étendue au niveau de l’API parente.

```xml
<!-- operation level -->
<policies>
    <inbound>
        <base/>
    </inbound>
    <backend>
        <base/>
    </backend>
    <outbound>
        <base/>
    </outbound>
</policies>

```

#### <a name="example"></a>Exemple

Cette stratégie au niveau de l’opération transfère explicitement toutes les demandes au service principal avec un délai d’expiration de 120 secondes et n’hérite pas de la stratégie principale au niveau de l’API parente. Si le service principal répond avec un code d’état d’erreur compris entre 400 et 599, la section [on-error](api-management-error-handling-policies.md) sera déclenchée.

```xml
<!-- operation level -->
<policies>
    <inbound>
        <base/>
    </inbound>
    <backend>
        <forward-request timeout="120" fail-on-error-status-code="true" />
        <!-- effective policy. note the absence of <base/> -->
    </backend>
    <outbound>
        <base/>
    </outbound>
</policies>

```

#### <a name="example"></a>Exemple

Cette stratégie au niveau de l’opération ne transmet pas de demandes au service principal.

```xml
<!-- operation level -->
<policies>
    <inbound>
        <base/>
    </inbound>
    <backend>
        <!-- no forwarding to backend -->
    </backend>
    <outbound>
        <base/>
    </outbound>
</policies>

```

### <a name="elements"></a>Éléments

| Élément         | Description   | Obligatoire |
| --------------- | ------------- | -------- |
| forward-request | Élément racine. | Oui      |

### <a name="attributes"></a>Attributs

| Attribut                                     | Description                                                                                                                                                                                                                                                                                                    | Obligatoire | Default |
| --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| timeout="integer"                             | Durée, en secondes, de l’attente du retour des en-têtes de réponse HTTP par le service back-end avant de déclencher une erreur de délai d’expiration. La valeur minimale est 0 seconde. Il est possible que les valeurs supérieures à 240 secondes ne soient pas prises en compte, car l’infrastructure réseau sous-jacente peut supprimer des connexions inactives après ce délai. | Non       | None    |
| follow-redirects="false &#124; true"          | Indique si les redirections à partir du service principal sont suivies par la passerelle ou renvoyées à l’appelant.                                                                                                                                                                                                    | Non       | false   |
| buffer-request-body="false &#124; true"       | Quand la valeur est « true », la demande est mise en mémoire tampon et sera réutilisée lors d’une [nouvelle tentative](api-management-advanced-policies.md#Retry).                                                                                                                                                                                               | Non       | false   |
| buffer-response="false &#124; true" | Affecte le traitement des réponses mémorisées en bloc. Quand la valeur est « false », chaque bloc reçu du back-end est immédiatement retourné à l’appelant. Quand la valeur est définie sur « true », les blocs sont mis en mémoire tampon (8 ko, sauf si la fin du flux est détectée) puis retournés à l’appelant. | Non | true |
| fail-on-error-status-code="false &#124; true" | Quand la valeur est true, la section [on-error](api-management-error-handling-policies.md) est déclenchée pour les codes de réponse dans la plage comprise entre 400 et 599.                                                                                                                                                                      | Non       | false   |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** backend
-   **Étendues de la stratégie :** toutes les étendues

## <a name="limit-concurrency"></a><a name="LimitConcurrency"></a> Limit concurrency

La stratégie `limit-concurrency` empêche les stratégies incluses d’exécuter plus de demandes simultanées que le nombre spécifié. En cas de dépassement de ce nombre, les nouvelles requêtes échouent immédiatement avec le code d’état 429 Trop de requêtes.

### <a name="policy-statement"></a><a name="LimitConcurrencyStatement"></a> Instruction de la stratégie

```xml
<limit-concurrency key="expression" max-count="number">
        <!— nested policy statements -->
</limit-concurrency>
```

### <a name="examples"></a>Exemples

#### <a name="example"></a>Exemple

L’exemple suivant montre comment limiter le nombre de requêtes transmises à un serveur principal en fonction de la valeur d’une variable contextuelle.

```xml
<policies>
  <inbound>…</inbound>
  <backend>
    <limit-concurrency key="@((string)context.Variables["connectionId"])" max-count="3">
      <forward-request timeout="120"/>
    </limit-concurrency>
  </backend>
  <outbound>…</outbound>
</policies>
```

### <a name="elements"></a>Éléments

| Élément           | Description   | Obligatoire |
| ----------------- | ------------- | -------- |
| limit-concurrency | Élément racine. | Oui      |

### <a name="attributes"></a>Attributs

| Attribut | Description                                                                                        | Obligatoire | Default |
| --------- | -------------------------------------------------------------------------------------------------- | -------- | ------- |
| key       | Chaîne. Expression autorisée. Spécifie l’étendue de la simultanéité. Peut être partagée par plusieurs stratégies. | Oui      | N/A     |
| max-count | Entier. Spécifie le nombre maximal de requêtes autorisées à entrer dans la stratégie.           | Oui      | N/A     |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound, backend, on-error

-   **Étendues de la stratégie :** toutes les étendues

## <a name="log-to-event-hub"></a><a name="log-to-eventhub"></a> Log to Event Hub

La stratégie `log-to-eventhub` envoie des messages au format spécifié à un Event Hub défini par une entité Enregistreur d’événements. Comme son nom l’indique, la stratégie est utilisée pour enregistrer certaines informations sur le contexte de la réponse ou de la demande à des fins d’analyse en ligne ou hors ligne.

> [!NOTE]
> Vous trouverez un guide de configuration étape par étape d’un Event Hub et des événements de journalisation à la page [Guide pratique de l’enregistrement d’événements de la Gestion des API avec Azure Event Hubs](./api-management-howto-log-event-hubs.md).

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<log-to-eventhub logger-id="id of the logger entity" partition-id="index of the partition where messages are sent" partition-key="value used for partition assignment">
  Expression returning a string to be logged
</log-to-eventhub>

```

### <a name="example"></a>Exemple

Toute chaîne peut être utilisée comme valeur à consigner dans Event Hubs. Dans cet exemple, la date et l’heure, le nom de service de déploiement, l’ID de la demande, l’adresse IP et le nom de l’opération de tous les appels entrants sont consignés dans l’Enregistreur d’événements Event Hub avec l’ID `contoso-logger`

```xml
<policies>
  <inbound>
    <log-to-eventhub logger-id ='contoso-logger'>
      @( string.Join(",", DateTime.UtcNow, context.Deployment.ServiceName, context.RequestId, context.Request.IpAddress, context.Operation.Name) )
    </log-to-eventhub>
  </inbound>
  <outbound>
  </outbound>
</policies>
```

### <a name="elements"></a>Éléments

| Élément         | Description                                                                     | Obligatoire |
| --------------- | ------------------------------------------------------------------------------- | -------- |
| log-to-eventhub | Élément racine. La valeur de cet élément est la chaîne à consigner dans votre Event Hub. | Oui      |

### <a name="attributes"></a>Attributs

| Attribut     | Description                                                               | Obligatoire                                                             |
| ------------- | ------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| logger-id     | ID de l’Enregistreur d’événements inscrit auprès de votre service Gestion des API.         | Oui                                                                  |
| partition-id  | Spécifie l’index de la partition où les messages sont envoyés.             | facultatif. Cet attribut peut ne pas être utilisé si `partition-key` est utilisé. |
| partition-key | Spécifie la valeur utilisée pour l’affectation de partitions lorsque des messages sont envoyés. | facultatif. Cet attribut peut ne pas être utilisé si `partition-id` est utilisé.  |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound, backend, on-error

-   **Étendues de la stratégie :** toutes les étendues

## <a name="mock-response"></a><a name="mock-response"></a> Réponse factice

La `mock-response`, comme le nom l’indique, est utilisée pour simuler des API et des opérations. Elle interrompt l’exécution du pipeline et retourne une réponse factice à l’appelant. La stratégie essaie toujours de renvoyer des réponses de la plus haute fidélité. Elle préfère les exemples de contenu de réponse, le cas échéant. Elle génère des exemples de réponses à partir de schémas, lorsque les schémas sont fournis et les exemples ne le sont pas. Si aucun exemple ou schéma n’est trouvé, des réponses sans contenu sont retournées.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<mock-response status-code="code" content-type="media type"/>

```

### <a name="examples"></a>Exemples

```xml
<!-- Returns 200 OK status code. Content is based on an example or schema, if provided for this
status code. First found content type is used. If no example or schema is found, the content is empty. -->
<mock-response/>

<!-- Returns 200 OK status code. Content is based on an example or schema, if provided for this
status code and media type. If no example or schema found, the content is empty. -->
<mock-response status-code='200' content-type='application/json'/>
```

### <a name="elements"></a>Éléments

| Élément       | Description   | Obligatoire |
| ------------- | ------------- | -------- |
| mock-response | Élément racine. | Oui      |

### <a name="attributes"></a>Attributs

| Attribut    | Description                                                                                           | Obligatoire | Default |
| ------------ | ----------------------------------------------------------------------------------------------------- | -------- | ------- |
| status-code  | Spécifie le code d’état de réponse et permet de sélectionner l’exemple ou le schéma correspondant.                 | Non       | 200     |
| content-type | Spécifie la valeur d’état de réponse `Content-Type` et permet de sélectionner l’exemple ou le schéma correspondant. | Non       | None    |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound, on-error

-   **Étendues de la stratégie :** toutes les étendues

## <a name="retry"></a><a name="Retry"></a> Retry

La stratégie `retry` exécute ses stratégies enfants une fois, puis retente leur exécution jusqu’à ce que la `condition` de la nouvelle tentative devienne `false` ou que le `count` de nouvelles tentatives soit épuisé.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml

<retry
    condition="boolean expression or literal"
    count="number of retry attempts"
    interval="retry interval in seconds"
    max-interval="maximum retry interval in seconds"
    delta="retry interval delta in seconds"
    first-fast-retry="boolean expression or literal">
        <!-- One or more child policies. No restrictions -->
</retry>

```

### <a name="example"></a>Exemple

Dans l’exemple suivant, le transfert de la demande est retenté jusqu’à dix fois suivant un algorithme de nouvelles tentatives exponentiel. Étant donné que `first-fast-retry` a la valeur false, toutes les nouvelles tentatives sont soumises à l’algorithme de nouvelles tentatives exponentiel.

```xml

<retry
    condition="@(context.Response.StatusCode == 500)"
    count="10"
    interval="10"
    max-interval="100"
    delta="10"
    first-fast-retry="false">
        <forward-request buffer-request-body="true" />
</retry>

```

### <a name="elements"></a>Éléments

| Élément | Description                                                         | Obligatoire |
| ------- | ------------------------------------------------------------------- | -------- |
| retry   | Élément racine. Peut contenir n’importe quelle autre stratégie sous forme d’élément enfant. | Oui      |

### <a name="attributes"></a>Attributs

| Attribut        | Description                                                                                                                                           | Obligatoire | Default |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| condition        | [Expression](api-management-policy-expressions.md) ou littéral booléen spécifiant si les nouvelles tentatives doivent être arrêtées (`false`) ou poursuivies (`true`).      | Oui      | N/A     |
| count            | Nombre positif spécifiant le nombre maximal de nouvelles tentatives à effectuer.                                                                                | Oui      | N/A     |
| interval         | Nombre positif en secondes spécifiant le délai d’attente entre les tentatives.                                                                 | Oui      | N/A     |
| max-interval     | Nombre positif en secondes spécifiant le délai d’attente maximal entre les tentatives. Il est utilisé pour implémenter un algorithme de nouvelles tentatives exponentiel. | Non       | N/A     |
| delta            | Nombre positif en secondes spécifiant l’incrément du délai d’attente. Il est utilisé pour implémenter les algorithmes de nouvelles tentatives linéaires et exponentiels.             | Non       | N/A     |
| first-fast-retry | S’il a la valeur `true`, la première des nouvelles tentatives est effectuée immédiatement.                                                                                  | Non       | `false` |

> [!NOTE]
> Lorsque seul `interval` est spécifié, les nouvelles tentatives sont effectuées à intervalles **fixes**.
> Lorsque seuls `interval` et `delta` sont spécifiés, un algorithme de nouvelles tentatives à intervalle **linéaire** est utilisé, suivant lequel le temps d’attente entre les tentatives est calculé selon la formule suivante : `interval + (count - 1)*delta`.
> Lorsque `interval`, `max-interval` et `delta` sont spécifiés, l’algorithme de nouvelles tentatives à intervalle **exponentiel** s’applique, suivant lequel le temps d’attente entre les tentatives augmente de façon exponentielle entre la valeur `interval` et la valeur `max-interval` selon la formule suivante : `min(interval + (2^count - 1) * random(delta * 0.8, delta * 1.2), max-interval)`.

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes. Notez que des restrictions d’utilisation des stratégies enfants seront héritées par cette stratégie.

-   **Sections de la stratégie :** inbound, outbound, backend, on-error

-   **Étendues de la stratégie :** toutes les étendues

## <a name="return-response"></a><a name="ReturnResponse"></a> Return response

La stratégie `return-response` abandonne l’exécution du pipeline et renvoie une réponse par défaut ou personnalisée à l’appelant. La réponse par défaut est `200 OK` sans corps. La réponse personnalisée peut être spécifiée par le biais d’instructions de stratégie ou d’une variable de contexte. Lorsque les deux sont fournies, la réponse contenue dans la variable de contexte est modifiée par les instructions de stratégie avant d’être renvoyée à l’appelant.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<return-response response-variable-name="existing context variable">
  <set-header/>
  <set-body/>
  <set-status/>
</return-response>

```

### <a name="example"></a>Exemple

```xml
<return-response>
   <set-status code="401" reason="Unauthorized"/>
   <set-header name="WWW-Authenticate" exists-action="override">
      <value>Bearer error="invalid_token"</value>
   </set-header>
</return-response>

```

### <a name="elements"></a>Éléments

| Élément         | Description                                                                               | Obligatoire |
| --------------- | ----------------------------------------------------------------------------------------- | -------- |
| return-response | Élément racine.                                                                             | Oui      |
| set-header      | Instruction de stratégie [set-header](api-management-transformation-policies.md#SetHTTPheader). | Non       |
| set-body        | Instruction de stratégie [set-body](api-management-transformation-policies.md#SetBody).         | Non       |
| set-status      | Instruction de stratégie [set-status](api-management-advanced-policies.md#SetStatus).           | Non       |

### <a name="attributes"></a>Attributs

| Attribut              | Description                                                                                                                                                                          | Obligatoire  |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------- |
| response-variable-name | Nom de la variable de contexte référencée à partir, par exemple, d’une stratégie [send-request](api-management-advanced-policies.md#SendRequest) en amont et contenant un objet `Response`. | facultatif. |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound, backend, on-error

-   **Étendues de la stratégie :** toutes les étendues

## <a name="send-one-way-request"></a><a name="SendOneWayRequest"></a> Send one way request

La stratégie `send-one-way-request` envoie une demande à l’URL indiquée sans attendre de réponse.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<send-one-way-request mode="new | copy">
  <url>...</url>
  <method>...</method>
  <header name="" exists-action="override | skip | append | delete">...</header>
  <body>...</body>
  <authentication-certificate thumbprint="thumbprint" />
</send-one-way-request>

```

### <a name="example"></a>Exemple

Cet exemple de stratégie montre un exemple d’utilisation de la stratégie `send-one-way-request` pour envoyer un message à une salle de conversation Slack si le code de la réponse HTTP est supérieur ou égal à 500. Pour plus d’informations sur cet exemple, consultez la page [Utilisation de services externes à partir du service Gestion des API Azure](./api-management-sample-send-request.md).

```xml
<choose>
    <when condition="@(context.Response.StatusCode >= 500)">
      <send-one-way-request mode="new">
        <set-url>https://hooks.slack.com/services/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</set-url>
        <set-method>POST</set-method>
        <set-body>@{
                return new JObject(
                        new JProperty("username","APIM Alert"),
                        new JProperty("icon_emoji", ":ghost:"),
                        new JProperty("text", String.Format("{0} {1}\nHost: {2}\n{3} {4}\n User: {5}",
                                                context.Request.Method,
                                                context.Request.Url.Path + context.Request.Url.QueryString,
                                                context.Request.Url.Host,
                                                context.Response.StatusCode,
                                                context.Response.StatusReason,
                                                context.User.Email
                                                ))
                        ).ToString();
            }</set-body>
      </send-one-way-request>
    </when>
</choose>

```

### <a name="elements"></a>Éléments

| Élément                    | Description                                                                                                 | Obligatoire                        |
| -------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------------------------- |
| send-one-way-request       | Élément racine.                                                                                               | Oui                             |
| url                        | URL de la demande.                                                                                     | Non si mode=copy ; sinon, oui. |
| method                     | Méthode HTTP de la demande.                                                                            | Non si mode=copy ; sinon, oui. |
| en-tête                     | En-tête de demande. Utilisez un élément d’en-tête pour chaque en-tête de demande.                                  | Non                              |
| body                       | Corps de la demande.                                                                                           | Non                              |
| authentication-certificate | [Certificat à utiliser pour l’authentification du client](api-management-authentication-policies.md#ClientCertificate) | Non                              |

### <a name="attributes"></a>Attributs

| Attribut     | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Obligatoire | Default  |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | -------- |
| mode="string" | Détermine s’il s’agit d’une nouvelle demande ou d’une copie de la demande actuelle. En mode outbound, mode=copy n’initialise pas le corps de la demande.                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Non       | Nouveau      |
| name          | Spécifie le nom de l’en-tête à définir.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Oui      | N/A      |
| exists-action | Spécifie l’action à entreprendre lorsque l’en-tête est déjà spécifié. Cet attribut doit avoir une des valeurs suivantes.<br /><br /> - override : remplace la valeur de l’en-tête actuel.<br />- skip : ne remplace pas la valeur de l’en-tête actuel.<br />- append : ajoute la valeur à celle de l’en-tête actuel.<br />- delete : supprime l’en-tête de la demande.<br /><br /> S’il a la valeur `override`, l’inscription de plusieurs entrées portant le même nom fait que l’en-tête est défini selon toutes les entrées (qui figurent plusieurs fois) ; seules les valeurs listées seront définies dans le résultat. | Non       | override |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound, backend, on-error

-   **Étendues de la stratégie :** toutes les étendues

## <a name="send-request"></a><a name="SendRequest"></a> Send request

La stratégie `send-request` envoie la demande fournie à l’URL spécifiée, sans attendre plus longtemps que la valeur du délai d’expiration définie.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<send-request mode="new|copy" response-variable-name="" timeout="60 sec" ignore-error
="false|true">
  <set-url>...</set-url>
  <set-method>...</set-method>
  <set-header name="" exists-action="override|skip|append|delete">...</set-header>
  <set-body>...</set-body>
  <authentication-certificate thumbprint="thumbprint" />
</send-request>

```

### <a name="example"></a>Exemple

Cet exemple montre un moyen de vérifier un jeton de référence avec un serveur d’autorisation. Pour plus d’informations sur cet exemple, consultez la page [Utilisation de services externes à partir du service Gestion des API Azure](./api-management-sample-send-request.md).

```xml
<inbound>
  <!-- Extract Token from Authorization header parameter -->
  <set-variable name="token" value="@(context.Request.Headers.GetValueOrDefault("Authorization","scheme param").Split(' ').Last())" />

  <!-- Send request to Token Server to validate token (see RFC 7662) -->
  <send-request mode="new" response-variable-name="tokenstate" timeout="20" ignore-error="true">
    <set-url>https://microsoft-apiappec990ad4c76641c6aea22f566efc5a4e.azurewebsites.net/introspection</set-url>
    <set-method>POST</set-method>
    <set-header name="Authorization" exists-action="override">
      <value>basic dXNlcm5hbWU6cGFzc3dvcmQ=</value>
    </set-header>
    <set-header name="Content-Type" exists-action="override">
      <value>application/x-www-form-urlencoded</value>
    </set-header>
    <set-body>@($"token={(string)context.Variables["token"]}")</set-body>
  </send-request>

  <choose>
        <!-- Check active property in response -->
        <when condition="@((bool)((IResponse)context.Variables["tokenstate"]).Body.As<JObject>()["active"] == false)">
            <!-- Return 401 Unauthorized with http-problem payload -->
            <return-response>
                <set-status code="401" reason="Unauthorized" />
                <set-header name="WWW-Authenticate" exists-action="override">
                    <value>Bearer error="invalid_token"</value>
                </set-header>
            </return-response>
        </when>
    </choose>
  <base />
</inbound>

```

### <a name="elements"></a>Éléments

| Élément                    | Description                                                                                                 | Obligatoire                        |
| -------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------------------------- |
| send-request               | Élément racine.                                                                                               | Oui                             |
| url                        | URL de la demande.                                                                                     | Non si mode=copy ; sinon, oui. |
| method                     | Méthode HTTP de la demande.                                                                            | Non si mode=copy ; sinon, oui. |
| en-tête                     | En-tête de demande. Utilisez un élément d’en-tête pour chaque en-tête de demande.                                  | Non                              |
| body                       | Corps de la demande.                                                                                           | Non                              |
| authentication-certificate | [Certificat à utiliser pour l’authentification du client](api-management-authentication-policies.md#ClientCertificate) | Non                              |

### <a name="attributes"></a>Attributs

| Attribut                       | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Obligatoire | Default  |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | -------- |
| mode="string"                   | Détermine s’il s’agit d’une nouvelle demande ou d’une copie de la demande actuelle. En mode outbound, mode=copy n’initialise pas le corps de la demande.                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Non       | Nouveau      |
| response-variable-name="string" | Nom de la variable contextuelle qui recevra un objet Response. Si la variable n’existe pas, elle est créée après l’exécution réussie de la stratégie, et devient accessible par le biais de la collection [`context.Variable`](api-management-policy-expressions.md#ContextVariables).                                                                                                                                                                                                                                                                                                                          | Oui      | N/A      |
| timeout="integer"               | Délai d’expiration en secondes avant l’échec de l’appel à l’URL.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | Non       | 60       |
| ignore-error                    | S’il a la valeur true et que la demande aboutit à une erreur :<br /><br /> - Si response-variable-name a été spécifié, il contiendra une valeur Null.<br />- Si response-variable-name n’est pas spécifié, context.Request ne sera pas mis à jour.                                                                                                                                                                                                                                                                                                                                                                                   | Non       | false    |
| name                            | Spécifie le nom de l’en-tête à définir.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Oui      | N/A      |
| exists-action                   | Spécifie l’action à entreprendre lorsque l’en-tête est déjà spécifié. Cet attribut doit avoir une des valeurs suivantes.<br /><br /> - override : remplace la valeur de l’en-tête actuel.<br />- skip : ne remplace pas la valeur de l’en-tête actuel.<br />- append : ajoute la valeur à celle de l’en-tête actuel.<br />- delete : supprime l’en-tête de la demande.<br /><br /> S’il a la valeur `override`, l’inscription de plusieurs entrées portant le même nom fait que l’en-tête est défini selon toutes les entrées (qui figurent plusieurs fois) ; seules les valeurs listées seront définies dans le résultat. | Non       | override |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound, backend, on-error

-   **Étendues de la stratégie :** toutes les étendues

## <a name="set-http-proxy"></a><a name="SetHttpProxy"></a> Définir le proxy HTTP

La stratégie `proxy` vous permet de router les demandes transférées aux back-ends via un proxy HTTP. Seul HTTP (et pas HTTPS) est pris en charge entre la passerelle et le proxy. Authentification de base et NTLM uniquement.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<proxy url="http://hostname-or-ip:port" username="username" password="password" />

```

### <a name="example"></a>Exemple

Notez l’utilisation de [propriétés](api-management-howto-properties.md) en tant que valeurs du nom d’utilisateur et du mot de passe pour éviter de stocker des informations sensibles dans le document de stratégie.

```xml
<proxy url="http://192.168.1.1:8080" username={{username}} password={{password}} />

```

### <a name="elements"></a>Éléments

| Élément | Description  | Obligatoire |
| ------- | ------------ | -------- |
| proxy   | Élément racine | Oui      |

### <a name="attributes"></a>Attributs

| Attribut         | Description                                            | Obligatoire | Default |
| ----------------- | ------------------------------------------------------ | -------- | ------- |
| url="chaîne"      | URL du proxy sous la forme http://host:port.             | Oui      | N/A     |
| username="chaîne" | Nom d’utilisateur à utiliser pour l’authentification auprès du proxy. | Non       | N/A     |
| password="chaîne" | Mot de passe à utiliser pour l’authentification auprès du proxy. | Non       | N/A     |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound

-   **Étendues de la stratégie :** toutes les étendues

## <a name="set-request-method"></a><a name="SetRequestMethod"></a> Set request method

La stratégie `set-method` permet de modifier la méthode d’une requête HTTP.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<set-method>METHOD</set-method>

```

### <a name="example"></a>Exemple

Cet exemple de stratégie, qui utilise la stratégie `set-method`, montre un exemple d’envoi d’un message à une salle de conversation Slack si le code de la réponse HTTP est supérieur ou égal à 500. Pour plus d’informations sur cet exemple, consultez la page [Utilisation de services externes à partir du service Gestion des API Azure](./api-management-sample-send-request.md).

```xml
<choose>
    <when condition="@(context.Response.StatusCode >= 500)">
      <send-one-way-request mode="new">
        <set-url>https://hooks.slack.com/services/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</set-url>
        <set-method>POST</set-method>
        <set-body>@{
                return new JObject(
                        new JProperty("username","APIM Alert"),
                        new JProperty("icon_emoji", ":ghost:"),
                        new JProperty("text", String.Format("{0} {1}\nHost: {2}\n{3} {4}\n User: {5}",
                                                context.Request.Method,
                                                context.Request.Url.Path + context.Request.Url.QueryString,
                                                context.Request.Url.Host,
                                                context.Response.StatusCode,
                                                context.Response.StatusReason,
                                                context.User.Email
                                                ))
                        ).ToString();
            }</set-body>
      </send-one-way-request>
    </when>
</choose>

```

### <a name="elements"></a>Éléments

| Élément    | Description                                                       | Obligatoire |
| ---------- | ----------------------------------------------------------------- | -------- |
| set-method | Élément racine. La valeur de l’élément spécifie la méthode HTTP. | Oui      |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, on-error

-   **Étendues de la stratégie :** toutes les étendues

## <a name="set-status-code"></a><a name="SetStatus"></a> Set status code

La stratégie `set-status` permet de donner la valeur spécifiée au code d’état HTTP.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<set-status code="" reason=""/>

```

### <a name="example"></a>Exemple

Cet exemple montre comment renvoyer une réponse 401 si le jeton d’autorisation n’est pas valide. Pour plus d’informations, consultez la page [Utiliser des services externes à partir du service Gestion des API Azure](./api-management-sample-send-request.md).

```xml
<choose>
  <when condition="@((bool)((IResponse)context.Variables["tokenstate"]).Body.As<JObject>()["active"] == false)">
    <return-response response-variable-name="existing response variable">
      <set-status code="401" reason="Unauthorized" />
      <set-header name="WWW-Authenticate" exists-action="override">
        <value>Bearer error="invalid_token"</value>
      </set-header>
    </return-response>
  </when>
</choose>

```

### <a name="elements"></a>Éléments

| Élément    | Description   | Obligatoire |
| ---------- | ------------- | -------- |
| set-status | Élément racine. | Oui      |

### <a name="attributes"></a>Attributs

| Attribut       | Description                                                | Obligatoire | Default |
| --------------- | ---------------------------------------------------------- | -------- | ------- |
| code="integer"  | Code d’état HTTP à renvoyer.                            | Oui      | N/A     |
| reason="string" | Description du motif pour lequel le code d’état est renvoyé. | Oui      | N/A     |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** outbound, backend, on-error
-   **Étendues de la stratégie :** toutes les étendues

## <a name="set-variable"></a><a name="set-variable"></a> Set variable

La stratégie `set-variable` déclare une variable de [contexte](api-management-policy-expressions.md#ContextVariables) et lui affecte une valeur spécifiée par le biais d’une [expression](api-management-policy-expressions.md) ou d’un littéral chaîne. Si l’expression contient un littéral, il sera converti en chaîne et le type de la valeur sera `System.String`.

### <a name="policy-statement"></a><a name="set-variablePolicyStatement"></a> Instruction de la stratégie

```xml
<set-variable name="variable name" value="Expression | String literal" />
```

### <a name="example"></a><a name="set-variableExample"></a> Exemple

L’exemple suivant montre une stratégie set variable dans la section inbound. Cette stratégie de variable définie crée une variable `isMobile` booléenne [définie](api-management-policy-expressions.md#ContextVariables) sur true si l'en-tête de demande `User-Agent` contient le texte `iPad` ou `iPhone`.

```xml
<set-variable name="IsMobile" value="@(context.Request.Headers.GetValueOrDefault("User-Agent","").Contains("iPad") || context.Request.Headers.GetValueOrDefault("User-Agent","").Contains("iPhone"))" />
```

### <a name="elements"></a>Éléments

| Élément      | Description   | Obligatoire |
| ------------ | ------------- | -------- |
| set-variable | Élément racine. | Oui      |

### <a name="attributes"></a>Attributs

| Attribut | Description                                                              | Obligatoire |
| --------- | ------------------------------------------------------------------------ | -------- |
| name      | Nom de la variable.                                                | Oui      |
| value     | Valeur de la variable. Peut être une expression ou une valeur littérale. | Oui      |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound, backend, on-error
-   **Étendues de la stratégie :** toutes les étendues

### <a name="allowed-types"></a><a name="set-variableAllowedTypes"></a> Types autorisés

Les expressions utilisées dans la stratégie `set-variable` doivent renvoyer un des types de base suivants.

-   System.Boolean
-   System.SByte
-   System.Byte
-   System.UInt16
-   System.UInt32
-   System.UInt64
-   System.Int16
-   System.Int32
-   System.Int64
-   System.Decimal
-   System.Single
-   System.Double
-   System.Guid
-   System.String
-   System.Char
-   System.DateTime
-   System.TimeSpan
-   System.Byte?
-   System.UInt16?
-   System.UInt32?
-   System.UInt64?
-   System.Int16?
-   System.Int32?
-   System.Int64?
-   System.Decimal?
-   System.Single?
-   System.Double?
-   System.Guid?
-   System.String?
-   System.Char?
-   System.DateTime?

## <a name="trace"></a><a name="Trace"></a> Trace

La stratégie `trace` ajoute une trace personnalisée à la sortie API Inspector, aux données de télémétrie Application Insights et/ou aux journaux de ressources.

-   La stratégie ajoute une trace personnalisée à la sortie [API Inspector](./api-management-howto-api-inspector.md) quand le suivi est déclenché, c.-à-d. que l’en-tête de demande `Ocp-Apim-Trace` est présent et a la valeur true et que l’en-tête de requête `Ocp-Apim-Subscription-Key` est présent et contient une clé valide qui autorise le suivi.
-   La stratégie crée des données de télémétrie [Trace](../azure-monitor/app/data-model-trace-telemetry.md) dans Application Insights, quand l’intégration à [Application Insights](./api-management-howto-app-insights.md) est activée et que le niveau `severity` spécifié dans la stratégie est supérieur ou égal au niveau `verbosity` spécifié dans le paramètre du diagnostic.
-   La stratégie ajoute une propriété dans l’entrée du journal quand les [journaux de ressources](./api-management-howto-use-azure-monitor.md#activity-logs) sont activés et que le niveau de gravité spécifié dans la stratégie est supérieur ou égal au niveau de verbosité spécifié dans le paramètre du diagnostic.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml

<trace source="arbitrary string literal" severity="verbose|information|error">
    <message>String literal or expressions</message>
    <metadata name="string literal or expressions" value="string literal or expressions"/>
</trace>

```

### <a name="example"></a><a name="traceExample"></a> Exemple

```xml
<trace source="PetStore API" severity="verbose">
    <message>@((string)context.Variables["clientConnectionID"])</message>
    <metadata name="Operation Name" value="New-Order"/>
</trace>
```

### <a name="elements"></a>Éléments

| Élément  | Description                                                                                                                                          | Obligatoire |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| trace    | Élément racine.                                                                                                                                        | Oui      |
| message  | Chaîne ou expression à journaliser.                                                                                                                 | Oui      |
| metadata | Ajoute une propriété personnalisée aux données de télémétrie [Trace](../azure-monitor/app/data-model-trace-telemetry.md) Application Insights. | Non       |

### <a name="attributes"></a>Attributs

| Attribut | Description                                                                                                               | Obligatoire | Default |
| --------- | ------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| source    | Littéral chaîne significatif pour la visionneuse de trace, qui spécifie la source du message.                                   | Oui      | N/A     |
| severity  | Spécifie le niveau de gravité de la trace. Les valeurs autorisées sont `verbose`, `information` et `error` (de la plus petite à la plus élevée). | Non       | Commentaires |
| name      | Nom de la propriété.                                                                                                     | Oui      | N/A     |
| value     | Valeur de la propriété.                                                                                                    | Oui      | N/A     |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound, backend, on-error

-   **Étendues de la stratégie :** toutes les étendues

## <a name="wait"></a><a name="Wait"></a> Wait

La stratégie `wait` exécute ses stratégies enfants immédiates en parallèle et attend la fin de la totalité ou de l’une de ses stratégies enfants immédiates pour se terminer. La stratégie d’attente peut avoir comme stratégies enfants immédiates les stratégies [Send request](api-management-advanced-policies.md#SendRequest), [Get value from cache](api-management-caching-policies.md#GetFromCacheByKey) et [Control flow](api-management-advanced-policies.md#choose).

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<wait for="all|any">
  <!--Wait policy can contain send-request, cache-lookup-value,
        and choose policies as child elements -->
</wait>

```

### <a name="example"></a>Exemple

Dans l’exemple suivant, deux stratégies `choose` sont les stratégies enfants immédiates de la stratégie `wait`. Ces deux stratégies `choose` s’exécutent en parallèle. Chaque stratégie `choose` essaie de récupérer une valeur en cache. En cas d’échec de cache, un service principal est appelé pour fournir la valeur. Dans cet exemple, la stratégie `wait` ne se termine pas tant que toutes ses stratégies enfants immédiates ne sont pas terminées, car l’attribut `for` a la valeur `all`. Dans cet exemple, les variables de contexte (`execute-branch-one`, `value-one`, `execute-branch-two` et `value-two`) sont déclarées hors de l’étendue de cet exemple de stratégie.

```xml
<wait for="all">
  <choose>
    <when condition="@((bool)context.Variables["execute-branch-one="])">
      <cache-lookup-value key="key-one" variable-name="value-one" />
      <choose>
        <when condition="@(!context.Variables.ContainsKey("value-one="))">
          <send-request mode="new" response-variable-name="value-one">
            <set-url>https://backend-one</set-url>
            <set-method>GET</set-method>
          </send-request>
        </when>
      </choose>
    </when>
  </choose>
  <choose>
    <when condition="@((bool)context.Variables["execute-branch-two="])">
      <cache-lookup-value key="key-two" variable-name="value-two" />
      <choose>
        <when condition="@(!context.Variables.ContainsKey("value-two="))">
          <send-request mode="new" response-variable-name="value-two">
            <set-url>https://backend-two</set-url>
            <set-method>GET</set-method>
          </send-request>
        </when>
      </choose>
    </when>
  </choose>
</wait>

```

### <a name="elements"></a>Éléments

| Élément | Description                                                                                                   | Obligatoire |
| ------- | ------------------------------------------------------------------------------------------------------------- | -------- |
| wait    | Élément racine. Ne peut contenir comme éléments enfants que les stratégies `send-request`, `cache-lookup-value` et `choose`. | Oui      |

### <a name="attributes"></a>Attributs

| Attribut | Description                                                                                                                                                                                                                                                                                                                                                                                                            | Obligatoire | Default |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| for       | Détermine si la stratégie `wait` attend la fin de toutes les stratégies enfants immédiates ou d’une seule. Les valeurs autorisées sont les suivantes :<br /><br /> - `all` : attend la fin de toutes les stratégies enfants immédiates.<br />- any : attend la fin d’une stratégie enfant immédiate. Une fois la première stratégie enfant immédiate terminée, la stratégie `wait` se termine et l’exécution de toutes les autres stratégies enfants immédiates est arrêtée. | Non       | all     |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound, backend
-   **Étendues de la stratégie :** toutes les étendues

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’utilisation de stratégies, consultez les pages :

-   [Stratégies dans Gestion des API](api-management-howto-policies.md)
-   [Expressions de stratégie](api-management-policy-expressions.md)
-   [Référence de stratégie](./api-management-policies.md) pour obtenir la liste complète des instructions et des paramètres de stratégie
-   [Exemples de stratégie](./policy-reference.md)
