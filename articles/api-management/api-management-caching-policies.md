---
title: Stratégies de mise en cache dans Gestion des API Azure | Microsoft Docs
description: Découvrez les stratégies de mise en cache disponibles dans Gestion des API Azure. Consultez des exemples et affichez des ressources disponibles supplémentaires.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 8147199c-24d8-439f-b2a9-da28a70a890c
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/13/2020
ms.author: apimpm
ms.openlocfilehash: 4db42d8fa8c676b20b236577ce6646b909df7c3a
ms.sourcegitcommit: 18046170f21fa1e569a3be75267e791ca9eb67d0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/16/2020
ms.locfileid: "94638884"
---
# <a name="api-management-caching-policies"></a>Stratégies de mise en cache dans Gestion des API
Cette rubrique est une ressource de référence au sujet des stratégies Gestion des API suivantes. Pour plus d'informations sur l'ajout et la configuration des stratégies, consultez la page [Stratégies dans Gestion des API](./api-management-policies.md).

> [!IMPORTANT]
> Le cache intégré est volatile et partagé par toutes les unités de la même région dans le même service de gestion des API.

## <a name="caching-policies"></a><a name="CachingPolicies"></a> Stratégies de mise en cache

- Stratégies de mise en cache des réponses
    - [Get from cache](api-management-caching-policies.md#GetFromCache) : effectue une recherche dans le cache et renvoie une réponse mise en cache valide si elle est disponible.
    - [Store to cache](api-management-caching-policies.md#StoreToCache) : met en cache la réponse en fonction de la configuration de contrôle de cache spécifiée.
- Stratégies de mise en cache des valeurs
    - [Get value from cache](#GetFromCacheByKey) : récupère un élément mis en cache par clé.
    - [Store value in cache](#StoreToCacheByKey) : stocke un élément mis en cache par clé.
    - [Supprimer la valeur du cache](#RemoveCacheByKey) : supprime un élément du cache par clé.

## <a name="get-from-cache"></a><a name="GetFromCache"></a> Get from cache
La stratégie `cache-lookup` permet d’effectuer une recherche dans le cache et de renvoyer une réponse mise en cache valide si elle est disponible. Cette stratégie peut être appliquée dans les cas où le contenu de la réponse reste statique pendant un certain temps. La mise en cache de la réponse réduit les besoins en bande passante et en calcul imposés par le serveur web principal et limite la latence perçue par les consommateurs de l’API.

> [!NOTE]
> Cette stratégie doit avoir une stratégie [Store to cache](api-management-caching-policies.md#StoreToCache) correspondante.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<cache-lookup vary-by-developer="true | false" vary-by-developer-groups="true | false" caching-type="prefer-external | external | internal" downstream-caching-type="none | private | public" must-revalidate="true | false" allow-private-response-caching="@(expression to evaluate)">
  <vary-by-header>Accept</vary-by-header>
  <!-- should be present in most cases -->
  <vary-by-header>Accept-Charset</vary-by-header>
  <!-- should be present in most cases -->
  <vary-by-header>Authorization</vary-by-header>
  <!-- should be present when allow-private-response-caching is "true"-->
  <vary-by-header>header name</vary-by-header>
  <!-- optional, can repeated several times -->
  <vary-by-query-parameter>parameter name</vary-by-query-parameter>
  <!-- optional, can repeated several times -->
</cache-lookup>
```

### <a name="examples"></a>Exemples

#### <a name="example"></a>Exemple

```xml
<policies>
    <inbound>
        <base />
        <cache-lookup vary-by-developer="false" vary-by-developer-groups="false" downstream-caching-type="none" must-revalidate="true" caching-type="internal" >
            <vary-by-query-parameter>version</vary-by-query-parameter>
        </cache-lookup>
    </inbound>
    <outbound>
        <cache-store duration="seconds" />
        <base />
    </outbound>
</policies>
```

#### <a name="example-using-policy-expressions"></a>Exemple utilisant des expressions de stratégie
Cet exemple montre comment configurer la durée de mise en cache des réponses de Gestion des API qui correspond à la mise en cache de la réponse du service principal, comme spécifié par la directive `Cache-Control` du service sauvegardé. Pour une démonstration de la configuration et de l’utilisation de cette stratégie, consultez [Cloud Cover Episode 177: More API Management Features](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/) (Plus de fonctionnalités de Gestion des API) avec Vlad Vinogradsky et rendez-vous directement à 25’25’’.

```xml
<!-- The following cache policy snippets demonstrate how to control API Management response cache duration with Cache-Control headers sent by the backend service. -->

<!-- Copy this snippet into the inbound section -->
<cache-lookup vary-by-developer="false" vary-by-developer-groups="false" downstream-caching-type="public" must-revalidate="true" >
  <vary-by-header>Accept</vary-by-header>
  <vary-by-header>Accept-Charset</vary-by-header>
</cache-lookup>

<!-- Copy this snippet into the outbound section. Note that cache duration is set to the max-age value provided in the Cache-Control header received from the backend service or to the default value of 5 min if none is found  -->
<cache-store duration="@{
    var header = context.Response.Headers.GetValueOrDefault("Cache-Control","");
    var maxAge = Regex.Match(header, @"max-age=(?<maxAge>\d+)").Groups["maxAge"]?.Value;
    return (!string.IsNullOrEmpty(maxAge))?int.Parse(maxAge):300;
  }"
 />
```

Pour plus d’informations, consultez les pages [Expressions de stratégie](api-management-policy-expressions.md) et [Variable de contexte](api-management-policy-expressions.md#ContextVariables).

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|cache-lookup|Élément racine.|Oui|
|vary-by-header|Commence par mettre en cache les réponses selon la valeur d’en-tête spécifiée, telle que Accept, Accept-Charset, Accept-Encoding, Accept-Language, Authorization, Expect, From, Host, If-Match.|Non|
|vary-by-query-parameter|Commencer la mise en cache des réponses selon la valeur des paramètres de requête spécifiés. Entrez un ou plusieurs paramètres. Utilisez un point-virgule comme séparateur. Si vous n’en spécifiez aucun, tous les paramètres de la requête sont utilisés.|Non|

### <a name="attributes"></a>Attributs

| Nom                           | Description                                                                                                                                                                                                                                                                                                                                                 | Obligatoire | Default           |
|--------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|-------------------|
| allow-private-response-caching | Lorsque l’attribut est défini sur `true`, permet la mise en cache des requêtes qui contiennent un en-tête d’autorisation.                                                                                                                                                                                                                                                                        | Non       | false             |
| caching-type               | Choisissez entre les valeurs suivantes de l’attribut :<br />- `internal` pour utiliser le cache Gestion des API intégré ;<br />- `external` pour utiliser le cache externe (voir [Utiliser un Cache Azure pour Redis externe dans Gestion des API Azure](api-management-howto-cache-external.md)),<br />- `prefer-external` pour utiliser un cache externe (si configuré) ou un cache interne sinon. | Non       | `prefer-external` |
| downstream-caching-type        | Cet attribut doit avoir l’une des valeurs suivantes.<br /><br /> - none : la mise en cache en aval n’est pas autorisée.<br />- private : la mise en cache privée en aval est autorisée.<br />- public : la mise en cache privée et partagée en aval est autorisée.                                                                                                          | Non       | Aucun              |
| must-revalidate                | Lorsque la mise en cache en aval est activée, cet attribut active ou désactive la directive de contrôle de cache `must-revalidate` dans les réponses de la passerelle.                                                                                                                                                                                                                      | Non       | true              |
| vary-by-developer              | Définissez la valeur `true` pour mettre en cache les réponses par compte de développeur possédant une [clé d’abonnement](./api-management-subscriptions.md) incluse dans la demande.                                                                                                                                                                                                                                                                                                  | Oui      |         False          |
| vary-by-developer-groups       | Attribut défini sur `true` pour mettre en cache des réponses par [groupe d’utilisateurs](./api-management-howto-create-groups.md).                                                                                                                                                                                                                                                                                                             | Oui      |       False            |

### <a name="usage"></a>Usage
Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

- **Sections de la stratégie :** inbound
- **Étendues de la stratégie :** toutes les étendues

## <a name="store-to-cache"></a><a name="StoreToCache"></a> Store to cache
La stratégie `cache-store` met en cache la réponse en fonction des paramètres de cache spécifiés. Cette stratégie peut être appliquée dans les cas où le contenu de la réponse reste statique pendant un certain temps. La mise en cache de la réponse réduit les besoins en bande passante et en calcul imposés par le serveur web principal et limite la latence perçue par les consommateurs de l’API.

> [!NOTE]
> Cette stratégie avoir une stratégie [Get from cache](api-management-caching-policies.md#GetFromCache) correspondante.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<cache-store duration="seconds" />
```

### <a name="examples"></a>Exemples

#### <a name="example"></a>Exemple

```xml
<policies>
    <inbound>
        <base />
        <cache-lookup vary-by-developer="true | false" vary-by-developer-groups="true | false" downstream-caching-type="none | private | public" must-revalidate="true | false">
            <vary-by-query-parameter>parameter name</vary-by-query-parameter> <!-- optional, can repeated several times -->
        </cache-lookup>
    </inbound>
    <outbound>
        <base />
        <cache-store duration="3600" />
    </outbound>
</policies>
```

#### <a name="example-using-policy-expressions"></a>Exemple utilisant des expressions de stratégie
Cet exemple montre comment configurer la durée de mise en cache des réponses de Gestion des API qui correspond à la mise en cache de la réponse du service principal, comme spécifié par la directive `Cache-Control` du service sauvegardé. Pour une démonstration de la configuration et de l’utilisation de cette stratégie, consultez [Cloud Cover Episode 177: More API Management Features](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/) (Plus de fonctionnalités de Gestion des API) avec Vlad Vinogradsky et rendez-vous directement à 25’25’’.

```xml
<!-- The following cache policy snippets demonstrate how to control API Management response cache duration with Cache-Control headers sent by the backend service. -->

<!-- Copy this snippet into the inbound section -->
<cache-lookup vary-by-developer="false" vary-by-developer-groups="false" downstream-caching-type="public" must-revalidate="true" >
  <vary-by-header>Accept</vary-by-header>
  <vary-by-header>Accept-Charset</vary-by-header>
</cache-lookup>

<!-- Copy this snippet into the outbound section. Note that cache duration is set to the max-age value provided in the Cache-Control header received from the backend service or to the default value of 5 min if none is found  -->
<cache-store duration="@{
    var header = context.Response.Headers.GetValueOrDefault("Cache-Control","");
    var maxAge = Regex.Match(header, @"max-age=(?<maxAge>\d+)").Groups["maxAge"]?.Value;
    return (!string.IsNullOrEmpty(maxAge))?int.Parse(maxAge):300;
  }"
 />
```

Pour plus d’informations, consultez les pages [Expressions de stratégie](api-management-policy-expressions.md) et [Variable de contexte](api-management-policy-expressions.md#ContextVariables).

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|cache-store|Élément racine.|Oui|

### <a name="attributes"></a>Attributs

| Nom             | Description                                                                                                                                                                                                                                                                                                                                                 | Obligatoire | Default           |
|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|-------------------|
| duration         | Durée de vie des entrées mises en cache (en secondes).                                                                                                                                                                                                                                                                                                   | Oui      | N/A               |

### <a name="usage"></a>Usage
Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

- **Sections de la stratégie :** outbound (sortant)
- **Étendues de la stratégie :** toutes les étendues

## <a name="get-value-from-cache"></a><a name="GetFromCacheByKey"></a> Get value from cache
La stratégie `cache-lookup-value` permet d’effectuer une recherche dans le cache par clé et de renvoyer une valeur mise en cache. La clé peut avoir une valeur de chaîne arbitraire ; elle est généralement fournie par le biais d’une expression de stratégie.

> [!NOTE]
> Cette stratégie doit avoir une stratégie [Store value in cache](#StoreToCacheByKey) correspondante.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<cache-lookup-value key="cache key value"
    default-value="value to use if cache lookup resulted in a miss"
    variable-name="name of a variable looked up value is assigned to"
    caching-type="prefer-external | external | internal" />
```

### <a name="example"></a>Exemple
Pour plus d’informations et d’exemples sur cette stratégie, consultez [Mise en cache personnalisée dans Gestion des API Azure](./api-management-sample-cache-by-key.md).

```xml
<cache-lookup-value
    key="@("userprofile-" + context.Variables["enduserid"])"
    variable-name="userprofile" />

```

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|cache-lookup-value|Élément racine.|Oui|

### <a name="attributes"></a>Attributs

| Nom             | Description                                                                                                                                                                                                                                                                                                                                                 | Obligatoire | Default           |
|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|-------------------|
| caching-type | Choisissez entre les valeurs suivantes de l’attribut :<br />- `internal` pour utiliser le cache Gestion des API intégré ;<br />- `external` pour utiliser le cache externe (voir [Utiliser un Cache Azure pour Redis externe dans Gestion des API Azure](api-management-howto-cache-external.md)),<br />- `prefer-external` pour utiliser un cache externe (si configuré) ou un cache interne sinon. | Non       | `prefer-external` |
| default-value    | Valeur attribuée à la variable si la recherche de clés de cache a échoué. Si cet attribut n’est pas spécifié, `null` est attribué.                                                                                                                                                                                                           | Non       | `null`            |
| key              | Valeur de clé de cache à utiliser dans la recherche.                                                                                                                                                                                                                                                                                                                       | Oui      | N/A               |
| variable-name    | Nom de la [variable contextuelle](api-management-policy-expressions.md#ContextVariables) à laquelle la valeur recherchée est attribuée, si la recherche réussit. Si la recherche aboutit à un échec, la variable reçoit la valeur de l’attribut `default-value` ou `null`, si l’attribut `default-value` est omis.                                       | Oui      | N/A               |

### <a name="usage"></a>Usage
Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

- **Sections de la stratégie :** inbound, outbound, backend, on-error
- **Étendues de la stratégie :** toutes les étendues

## <a name="store-value-in-cache"></a><a name="StoreToCacheByKey"></a> Store value in cache
La stratégie `cache-store-value` effectue le stockage du cache par clé. La clé peut avoir une valeur de chaîne arbitraire ; elle est généralement fournie par le biais d’une expression de stratégie.

> [!NOTE]
> Cette stratégie avoir une stratégie [Get value from cache](#GetFromCacheByKey) correspondante.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<cache-store-value key="cache key value" value="value to cache" duration="seconds" caching-type="prefer-external | external | internal" />
```

### <a name="example"></a>Exemple
Pour plus d’informations et d’exemples sur cette stratégie, consultez [Mise en cache personnalisée dans Gestion des API Azure](./api-management-sample-cache-by-key.md).

```xml
<cache-store-value
    key="@("userprofile-" + context.Variables["enduserid"])"
    value="@((string)context.Variables["userprofile"])" duration="100000" />

```

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|cache-store-value|Élément racine.|Oui|

### <a name="attributes"></a>Attributs

| Nom             | Description                                                                                                                                                                                                                                                                                                                                                 | Obligatoire | Default           |
|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|-------------------|
| caching-type | Choisissez entre les valeurs suivantes de l’attribut :<br />- `internal` pour utiliser le cache Gestion des API intégré ;<br />- `external` pour utiliser le cache externe (voir [Utiliser un Cache Azure pour Redis externe dans Gestion des API Azure](api-management-howto-cache-external.md)),<br />- `prefer-external` pour utiliser un cache externe (si configuré) ou un cache interne sinon. | Non       | `prefer-external` |
| duration         | La valeur est mise en cache pendant la durée spécifiée en secondes.                                                                                                                                                                                                                                                                                 | Oui      | N/A               |
| key              | Clé de cache sous laquelle la valeur est stockée.                                                                                                                                                                                                                                                                                                                   | Oui      | N/A               |
| value            | Valeur à mettre en cache.                                                                                                                                                                                                                                                                                                                                     | Oui      | N/A               |
### <a name="usage"></a>Usage
Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

- **Sections de la stratégie :** inbound, outbound, backend, on-error
- **Étendues de la stratégie :** toutes les étendues

## <a name="remove-value-from-cache"></a><a name="RemoveCacheByKey"></a> Remove value from cache
`cache-remove-value` supprime un élément en cache identifié par sa clé. La clé peut avoir une valeur de chaîne arbitraire ; elle est généralement fournie par le biais d’une expression de stratégie.

#### <a name="policy-statement"></a>Instruction de la stratégie

```xml

<cache-remove-value key="cache key value" caching-type="prefer-external | external | internal"  />

```

#### <a name="example"></a>Exemple

```xml

<cache-remove-value key="@("userprofile-" + context.Variables["enduserid"])"/>

```

#### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|cache-remove-value|Élément racine.|Oui|

#### <a name="attributes"></a>Attributs

| Nom             | Description                                                                                                                                                                                                                                                                                                                                                 | Obligatoire | Default           |
|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|-------------------|
| caching-type | Choisissez entre les valeurs suivantes de l’attribut :<br />- `internal` pour utiliser le cache Gestion des API intégré ;<br />- `external` pour utiliser le cache externe (voir [Utiliser un Cache Azure pour Redis externe dans Gestion des API Azure](api-management-howto-cache-external.md)),<br />- `prefer-external` pour utiliser un cache externe (si configuré) ou un cache interne sinon. | Non       | `prefer-external` |
| key              | Clé de la valeur précédemment mise en cache à supprimer du cache.                                                                                                                                                                                                                                                                                        | Oui      | N/A               |

#### <a name="usage"></a>Usage
Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

- **Sections de la stratégie :** inbound, outbound, backend, on-error
- **Étendues de la stratégie :** toutes les étendues

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’utilisation de stratégies, consultez les pages :

+ [Stratégies dans Gestion des API](api-management-howto-policies.md)
+ [Transform and protect your API](transform-api.md) (Transformer et protéger votre API)
+ [Référence de stratégie](./api-management-policies.md) pour obtenir la liste complète des instructions et des paramètres de stratégie
+ [Exemples de stratégie](./policy-reference.md)