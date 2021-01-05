---
title: Stratégies de restriction des accès de la Gestion des API Azure | Microsoft Docs
description: Découvrez les stratégies de restriction des accès disponibles dans la Gestion des API Azure.
services: api-management
documentationcenter: ''
author: vladvino
ms.assetid: 034febe3-465f-4840-9fc6-c448ef520b0f
ms.service: api-management
ms.topic: article
ms.date: 11/23/2020
ms.author: apimpm
ms.openlocfilehash: e38dcf1e12629405ae5f28a987ba20557037ee67
ms.sourcegitcommit: e0ec3c06206ebd79195d12009fd21349de4a995d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97683447"
---
# <a name="api-management-access-restriction-policies"></a>Stratégies de restriction des accès de la Gestion des API

Cette rubrique est une ressource de référence au sujet des stratégies Gestion des API suivantes. Pour plus d'informations sur l'ajout et la configuration des stratégies, consultez la page [Stratégies dans Gestion des API](./api-management-policies.md).

## <a name="access-restriction-policies"></a><a name="AccessRestrictionPolicies"></a> Stratégies de restriction des accès

-   [Check HTTP header](#CheckHTTPHeader) : applique l’existence et/ou la valeur d’un en-tête HTTP.
-   [Limit call rate by subscription](#LimitCallRate) : empêche les pics d’utilisation de l’API en limitant le débit d’appels par abonnement.
-   [Limit call rate by key](#LimitCallRateByKey) : empêche les pics d’utilisation de l’API en limitant le débit d’appels par clé.
-   [Restrict caller IPs](#RestrictCallerIPs) : filtre (autorise/rejette) les appels de certaines adresses IP spécifiques et/ou de certaines plages d’adresses.
-   [Set usage quota by subscription](#SetUsageQuota) : vous permet d’appliquer un volume d’appel et/ou un quota de bande passante renouvelable ou illimité par abonnement.
-   [Set usage quota by key](#SetUsageQuotaByKey) : vous permet d’appliquer un volume d’appel et/ou un quota de bande passante renouvelable ou illimité par clé.
-   [Validate JWT](#ValidateJWT) : applique l’existence et la validité d’un JWT extrait d’un en-tête HTTP ou d’un paramètre de requête spécifié.

> [!TIP]
> Vous pouvez utiliser des stratégies de restriction d’accès dans différentes étendues à des fins différentes. Par exemple, vous pouvez sécuriser l’intégralité de l’API avec l’authentification AAD en appliquant la stratégie `validate-jwt` au niveau de l’API ou vous pouvez l’appliquer au niveau de l’opération d’API et utiliser `claims` pour un contrôle plus granulaire.

## <a name="check-http-header"></a><a name="CheckHTTPHeader"></a> Check HTTP header

Utilisez la stratégie `check-header` pour imposer un en-tête HTTP donné à une demande. Vous pouvez éventuellement vérifier si l’en-tête a une certaine valeur ou une valeur comprise dans une plage de valeurs autorisées. Si la vérification échoue, la stratégie met fin au traitement de la demande et renvoie le message d’erreur et le code d’état HTTP spécifiés par la stratégie.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<check-header name="header name" failed-check-httpcode="code" failed-check-error-message="message" ignore-case="true">
    <value>Value1</value>
    <value>Value2</value>
</check-header>
```

### <a name="example"></a>Exemple

```xml
<check-header name="Authorization" failed-check-httpcode="401" failed-check-error-message="Not authorized" ignore-case="false">
    <value>f6dc69a089844cf6b2019bae6d36fac8</value>
</check-header>
```

### <a name="elements"></a>Éléments

| Nom         | Description                                                                                                                                   | Obligatoire |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| check-header | Élément racine.                                                                                                                                 | Oui      |
| value        | Valeur autorisée de l’en-tête HTTP. Lorsque plusieurs éléments de valeurs sont spécifiés, la vérification est considérée comme réussie si l’une des valeurs correspond. | Non       |

### <a name="attributes"></a>Attributs

| Nom                       | Description                                                                                                                                                            | Obligatoire | Default |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| failed-check-error-message | Message d’erreur à renvoyer dans le corps de la réponse HTTP si l’en-tête n’existe pas ou a une valeur non valide. Les éventuels caractères spéciaux de ce message doivent être correctement placés dans une séquence d’échappement. | Oui      | N/A     |
| failed-check-httpcode      | Code d’état HTTP à renvoyer si l’en-tête n’existe pas ou a une valeur non valide.                                                                                        | Oui      | N/A     |
| header-name                | Nom de l’en-tête HTTP à vérifier.                                                                                                                                  | Oui      | N/A     |
| ignore-case                | Peut avoir la valeur True ou False. S’il a la valeur True, la casse est ignorée lors de la comparaison de la valeur de l’en-tête à l’ensemble des valeurs acceptables.                                    | Oui      | N/A     |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound

-   **Étendues de la stratégie :** toutes les étendues

## <a name="limit-call-rate-by-subscription"></a><a name="LimitCallRate"></a> Limit call rate by subscription

La stratégie `rate-limit` évite les pics d’utilisation des API par abonnement en limitant le débit d’appels à un nombre spécifié pour une période donnée. Lorsque cette stratégie est déclenchée, l’appelant reçoit le code d’état de réponse `429 Too Many Requests`.

> [!IMPORTANT]
> Cette stratégie ne peut être utilisée qu’une seule fois par document de stratégie.
>
> Les [expressions de stratégie](api-management-policy-expressions.md) ne peuvent être utilisées dans aucun attribut de cette stratégie.

> [!CAUTION]
> En raison de la nature distribuée de l’architecture de limitation, la limitation du débit n’est jamais totalement exacte. La différence entre le nombre configuré et le nombre réel de requêtes autorisées varie en fonction du volume et du débit des requêtes, de la latence du backend et d’autres facteurs.

> [!NOTE]
> Pour comprendre la différence entre les limites de taux et les quotas, consultez [Limites de taux et quotas.](./api-management-sample-flexible-throttling.md#rate-limits-and-quotas)

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<rate-limit calls="number" renewal-period="seconds">
    <api name="API name" id="API id" calls="number" renewal-period="seconds" />
        <operation name="operation name" id="operation id" calls="number" renewal-period="seconds" />
    </api>
</rate-limit>
```

### <a name="example"></a>Exemple

```xml
<policies>
    <inbound>
        <base />
        <rate-limit calls="20" renewal-period="90" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```

### <a name="elements"></a>Éléments

| Nom       | Description                                                                                                                                                                                                                                                                                              | Obligatoire |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| rate-limit | Élément racine.                                                                                                                                                                                                                                                                                            | Oui      |
| api        | Ajoutez un ou plusieurs éléments de ce type pour imposer une limite de débit d’appels aux API au sein du produit. Les limites de débit d’appels au niveau du produit et de l’API s’appliquent indépendamment les unes des autres. L’API peut être référencée via `name` ou `id`. Si les deux attributs sont fournis, `id` sera utilisé et `name` sera ignoré.                    | Non       |
| operation  | Ajoutez un ou plusieurs éléments de ce type pour imposer une limite de débit d’appels aux opérations au sein d’une API. Les limites de débit d’appels au niveau du produit, de l’API et de l’opération s’appliquent indépendamment les unes des autres. L’opération peut être référencée via `name` ou `id`. Si les deux attributs sont fournis, `id` sera utilisé et `name` sera ignoré. | Non       |

### <a name="attributes"></a>Attributs

| Nom           | Description                                                                                           | Obligatoire | Default |
| -------------- | ----------------------------------------------------------------------------------------------------- | -------- | ------- |
| name           | Nom de l’API à laquelle la limite de débit s’applique.                                                | Oui      | N/A     |
| calls          | Nombre maximal d’appels autorisés au cours de l’intervalle de temps spécifié dans le paramètre `renewal-period`. | Oui      | N/A     |
| renewal-period | Période en secondes après laquelle le quota se réinitialise.                                              | Oui      | N/A     |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound

-   **Étendues de la stratégie :** produit, API, opération

## <a name="limit-call-rate-by-key"></a><a name="LimitCallRateByKey"></a> Limite de débit d’appels par clé

> [!IMPORTANT]
> Cette fonctionnalité n’est pas disponible dans le niveau **Consommation** du service Gestion des API.

La stratégie `rate-limit-by-key` évite les pics d’utilisation des API par clé en limitant le débit d’appels à un nombre spécifié pour une période donnée. La clé peut avoir une valeur de chaîne arbitraire ; elle est généralement fournie par le biais d’une expression de stratégie. Une condition d’incrément facultative peut être ajoutée pour spécifier quelles demandes doivent être comptées dans la limite. Lorsque cette stratégie est déclenchée, l’appelant reçoit le code d’état de réponse `429 Too Many Requests`.

Pour plus d’informations et d’exemples sur cette stratégie, consultez la page [Limitation avancée des demandes dans la Gestion des API Azure](./api-management-sample-flexible-throttling.md).

> [!CAUTION]
> En raison de la nature distribuée de l’architecture de limitation, la limitation du débit n’est jamais totalement exacte. La différence entre le nombre configuré et le nombre réel de requêtes autorisées varie en fonction du volume et du débit des requêtes, de la latence du backend et d’autres facteurs.

> [!NOTE]
> Pour comprendre la différence entre les limites de taux et les quotas, consultez [Limites de taux et quotas.](./api-management-sample-flexible-throttling.md#rate-limits-and-quotas)

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<rate-limit-by-key calls="number"
                   renewal-period="seconds"
                   increment-condition="condition"
                   counter-key="key value" />

```

### <a name="example"></a>Exemple

Dans l’exemple suivant, la limite de débit est indexée par l’adresse IP de l’appelant.

```xml
<policies>
    <inbound>
        <base />
        <rate-limit-by-key  calls="10"
              renewal-period="60"
              increment-condition="@(context.Response.StatusCode == 200)"
              counter-key="@(context.Request.IpAddress)"/>
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```

### <a name="elements"></a>Éléments

| Nom              | Description   | Obligatoire |
| ----------------- | ------------- | -------- |
| rate-limit-by-key | Élément racine. | Oui      |

### <a name="attributes"></a>Attributs

| Nom                | Description                                                                                           | Obligatoire | Default |
| ------------------- | ----------------------------------------------------------------------------------------------------- | -------- | ------- |
| calls               | Nombre maximal d’appels autorisés au cours de l’intervalle de temps spécifié dans le paramètre `renewal-period`. | Oui      | N/A     |
| counter-key         | Clé à utiliser pour la stratégie de limite de débit.                                                             | Oui      | N/A     |
| increment-condition | Expression booléenne spécifiant si la demande doit être comptée dans le quota (`true`).        | Non       | N/A     |
| renewal-period      | Période en secondes après laquelle le quota se réinitialise.                                              | Oui      | N/A     |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound

-   **Étendues de la stratégie :** toutes les étendues

## <a name="restrict-caller-ips"></a><a name="RestrictCallerIPs"></a> Restrict caller IPs

La stratégie `ip-filter` filtre (autorise/rejette) les appels de certaines adresses IP et/ou de certaines plages d’adresses.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<ip-filter action="allow | forbid">
    <address>address</address>
    <address-range from="address" to="address" />
</ip-filter>
```

### <a name="example"></a>Exemple

Dans l’exemple suivant, la stratégie autorise uniquement les requêtes entrantes à partir de l’adresse IP unique ou de la plage d’adresses IP spécifiée

```xml
<ip-filter action="allow">
    <address>13.66.201.169</address>
    <address-range from="13.66.140.128" to="13.66.140.143" />
</ip-filter>
```

### <a name="elements"></a>Éléments

| Nom                                      | Description                                         | Obligatoire                                                       |
| ----------------------------------------- | --------------------------------------------------- | -------------------------------------------------------------- |
| ip-filter                                 | Élément racine.                                       | Oui                                                            |
| address                                   | Spécifie une adresse IP unique à filtrer.   | Au moins un élément `address` ou `address-range` est requis. |
| address-range from="address" to="address" | Spécifie une plage d’adresses IP à filtrer. | Au moins un élément `address` ou `address-range` est requis. |

### <a name="attributes"></a>Attributs

| Nom                                      | Description                                                                                 | Obligatoire                                           | Default |
| ----------------------------------------- | ------------------------------------------------------------------------------------------- | -------------------------------------------------- | ------- |
| address-range from="address" to="address" | Plage d'adresses IP pour lesquelles autoriser ou refuser l'accès.                                        | Obligatoire lorsque l’élément `address-range` est utilisé. | N/A     |
| ip-filter action="allow &#124; forbid"    | Spécifie si les appels doivent être autorisés ou non pour les adresses IP et plages spécifiées. | Oui                                                | N/A     |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound
-   **Étendues de la stratégie :** toutes les étendues

## <a name="set-usage-quota-by-subscription"></a><a name="SetUsageQuota"></a> Set usage quota by subscription

La stratégie `quota` applique un volume d’appels et/ou un quota de bande passante renouvelable ou illimité par abonnement.

> [!IMPORTANT]
> Cette stratégie ne peut être utilisée qu’une seule fois par document de stratégie.
>
> Des [expressions de stratégie](api-management-policy-expressions.md) ne peuvent pas être utilisées dans les attributs de stratégie pour cette stratégie.

> [!NOTE]
> Pour comprendre la différence entre les limites de taux et les quotas, consultez [Limites de taux et quotas.](./api-management-sample-flexible-throttling.md#rate-limits-and-quotas)

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<quota calls="number" bandwidth="kilobytes" renewal-period="seconds">
    <api name="API name" id="API id" calls="number" renewal-period="seconds" />
        <operation name="operation name" id="operation id" calls="number" renewal-period="seconds" />
    </api>
</quota>
```

### <a name="example"></a>Exemple

```xml
<policies>
    <inbound>
        <base />
        <quota calls="10000" bandwidth="40000" renewal-period="3600" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```

### <a name="elements"></a>Éléments

| Nom      | Description                                                                                                                                                                                                                                                                                  | Obligatoire |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| quota     | Élément racine.                                                                                                                                                                                                                                                                                | Oui      |
| api       | Ajoutez un ou plusieurs éléments de ce type pour imposer un quota d’appel aux API au sein du produit. Les quotas d’appel au niveau du produit et de l’API s’appliquent indépendamment les uns des autres. L’API peut être référencée via `name` ou `id`. Si les deux attributs sont fournis, `id` sera utilisé et `name` sera ignoré.                    | Non       |
| operation | Ajoutez un ou plusieurs éléments de ce type pour imposer un quota d’appel aux opérations au sein d’une API. Les quotas d’appel au niveau du produit, de l’API et de l’opération s’appliquent indépendamment les uns des autres. L’opération peut être référencée via `name` ou `id`. Si les deux attributs sont fournis, `id` sera utilisé et `name` sera ignoré. | Non       |

### <a name="attributes"></a>Attributs

| Nom           | Description                                                                                               | Obligatoire                                                         | Default |
| -------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ------- |
| name           | Nom de l’API ou de l’opération à laquelle s’applique le quota.                                             | Oui                                                              | N/A     |
| bandwidth      | Nombre maximal de kilo-octets autorisés au cours de l’intervalle de temps spécifié dans le paramètre `renewal-period`. | Il est obligatoire de spécifier `calls`, `bandwidth` ou les deux. | N/A     |
| calls          | Nombre maximal d’appels autorisés au cours de l’intervalle de temps spécifié dans le paramètre `renewal-period`.     | Il est obligatoire de spécifier `calls`, `bandwidth` ou les deux. | N/A     |
| renewal-period | Période en secondes après laquelle le quota se réinitialise.                                                  | Oui                                                              | N/A     |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound
-   **Étendues de la stratégie :** product

## <a name="set-usage-quota-by-key"></a><a name="SetUsageQuotaByKey"></a> Set usage quota by key

> [!IMPORTANT]
> Cette fonctionnalité n’est pas disponible dans le niveau **Consommation** du service Gestion des API.

La stratégie `quota-by-key` applique un volume d’appels et/ou un quota de bande passante renouvelable ou illimité par clé. La clé peut avoir une valeur de chaîne arbitraire ; elle est généralement fournie par le biais d’une expression de stratégie. Une condition d’incrément facultative peut être ajoutée pour spécifier quelles demandes doivent être comptées dans le quota. Si plusieurs stratégies incrémentent la même valeur de clé, celle-ci est incrémentée une seule fois par demande. Quand la limite d’appels est atteinte, l’appelant reçoit le code d’état de réponse `403 Forbidden`.

Pour plus d’informations et d’exemples sur cette stratégie, consultez la page [Limitation avancée des demandes dans la Gestion des API Azure](./api-management-sample-flexible-throttling.md).

> [!NOTE]
> Pour comprendre la différence entre les limites de taux et les quotas, consultez [Limites de taux et quotas.](./api-management-sample-flexible-throttling.md#rate-limits-and-quotas)

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<quota-by-key calls="number"
              bandwidth="kilobytes"
              renewal-period="seconds"
              increment-condition="condition"
              counter-key="key value" />

```

### <a name="example"></a>Exemple

Dans l’exemple suivant, le quota est indexé par l’adresse IP de l’appelant.

```xml
<policies>
    <inbound>
        <base />
        <quota-by-key calls="10000" bandwidth="40000" renewal-period="3600"
                      increment-condition="@(context.Response.StatusCode >= 200 && context.Response.StatusCode < 400)"
                      counter-key="@(context.Request.IpAddress)" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```

### <a name="elements"></a>Éléments

| Nom  | Description   | Obligatoire |
| ----- | ------------- | -------- |
| quota | Élément racine. | Oui      |

### <a name="attributes"></a>Attributs

| Nom                | Description                                                                                               | Obligatoire                                                         | Default |
| ------------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ------- |
| bandwidth           | Nombre maximal de kilo-octets autorisés au cours de l’intervalle de temps spécifié dans le paramètre `renewal-period`. | Il est obligatoire de spécifier `calls`, `bandwidth` ou les deux. | N/A     |
| calls               | Nombre maximal d’appels autorisés au cours de l’intervalle de temps spécifié dans le paramètre `renewal-period`.     | Il est obligatoire de spécifier `calls`, `bandwidth` ou les deux. | N/A     |
| counter-key         | Clé à utiliser pour la stratégie de quota.                                                                      | Oui                                                              | N/A     |
| increment-condition | Expression booléenne spécifiant si la demande doit être comptée dans le quota (`true`).             | Non                                                               | N/A     |
| renewal-period      | Période en secondes après laquelle le quota se réinitialise.                                                  | Oui                                                              | N/A     |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound
-   **Étendues de la stratégie :** toutes les étendues

## <a name="validate-jwt"></a><a name="ValidateJWT"></a> Validate JWT

La stratégie `validate-jwt` applique l’existence et la validité d’un jeton JWT (JSON Web Token) extrait à partir de l’en-tête HTTP spécifié ou du paramètre de requête spécifié.

> [!IMPORTANT]
> La stratégie `validate-jwt` exige que la revendication inscrite `exp` soit incluse dans le jeton JWT, sauf si l’attribut `require-expiration-time` est spécifié et a la valeur `false`.
> La stratégie `validate-jwt` prend en charge les algorithmes de signature HS256 et RS256. Pour HS256, la clé doit être fournie en ligne au sein de la stratégie au format encodé en base 64. Pour l’algorithme RS256, la clé peut être fournie soit via un point de terminaison de configuration OpenID, soit en spécifiant l’ID d’un certificat chargé qui contient la clé publique ou la paire modulo-exposant de la clé publique.
> La stratégie `validate-jwt` prend en charge les jetons chiffrés avec des clés symétriques à l’aide des algorithmes de chiffrement suivants : A128CBC-HS256, A192CBC-HS384, A256CBC-HS512.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<validate-jwt
    header-name="name of http header containing the token (use query-parameter-name attribute if the token is passed in the URL)"
    failed-validation-httpcode="http status code to return on failure"
    failed-validation-error-message="error message to return on failure"
    token-value="expression returning JWT token as a string"
    require-expiration-time="true|false"
    require-scheme="scheme"
    require-signed-tokens="true|false"
    clock-skew="allowed clock skew in seconds"
    output-token-variable-name="name of a variable to receive a JWT object representing successfully validated token">
  <openid-config url="full URL of the configuration endpoint, e.g. https://login.constoso.com/openid-configuration" />
  <issuer-signing-keys>
    <key>base64 encoded signing key</key>
    <!-- if there are multiple keys, then add additional key elements -->
  </issuer-signing-keys>
  <decryption-keys>
    <key>base64 encoded signing key</key>
    <!-- if there are multiple keys, then add additional key elements -->
  </decryption-keys>
  <audiences>
    <audience>audience string</audience>
    <!-- if there are multiple possible audiences, then add additional audience elements -->
  </audiences>
  <issuers>
    <issuer>issuer string</issuer>
    <!-- if there are multiple possible issuers, then add additional issuer elements -->
  </issuers>
  <required-claims>
    <claim name="name of the claim as it appears in the token" match="all|any" separator="separator character in a multi-valued claim">
      <value>claim value as it is expected to appear in the token</value>
      <!-- if there is more than one allowed values, then add additional value elements -->
    </claim>
    <!-- if there are multiple possible allowed values, then add additional value elements -->
  </required-claims>
</validate-jwt>

```

### <a name="examples"></a>Exemples

#### <a name="simple-token-validation"></a>Validation simple du jeton

```xml
<validate-jwt header-name="Authorization" require-scheme="Bearer">
    <issuer-signing-keys>
        <key>{{jwt-signing-key}}</key>  <!-- signing key specified as a named value -->
    </issuer-signing-keys>
    <audiences>
        <audience>@(context.Request.OriginalUrl.Host)</audience>  <!-- audience is set to API Management host name -->
    </audiences>
    <issuers>
        <issuer>http://contoso.com/</issuer>
    </issuers>
</validate-jwt>
```

#### <a name="token-validation-with-rsa-certificate"></a>Validation de jeton avec certificat RSA

```xml
<validate-jwt header-name="Authorization" require-scheme="Bearer">
    <issuer-signing-keys>
        <key certificate-id="my-rsa-cert" />  <!-- signing key specified as certificate ID, enclosed in double-quotes -->
    </issuer-signing-keys>
    <audiences>
        <audience>@(context.Request.OriginalUrl.Host)</audience>  <!-- audience is set to API Management host name -->
    </audiences>
    <issuers>
        <issuer>http://contoso.com/</issuer>
    </issuers>
</validate-jwt>
```

#### <a name="azure-active-directory-token-validation"></a>Validation d’un jeton Azure Active Directory

```xml
<validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
    <openid-config url="https://login.microsoftonline.com/contoso.onmicrosoft.com/.well-known/openid-configuration" />
    <audiences>
        <audience>25eef6e4-c905-4a07-8eb4-0d08d5df8b3f</audience>
    </audiences>
    <required-claims>
        <claim name="id" match="all">
            <value>insert claim here</value>
        </claim>
    </required-claims>
</validate-jwt>
```

#### <a name="azure-active-directory-b2c-token-validation"></a>Validation d’un jeton Azure Active Directory B2C

```xml
<validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
    <openid-config url="https://login.microsoftonline.com/tfp/contoso.onmicrosoft.com/b2c_1_signin/v2.0/.well-known/openid-configuration" />
    <audiences>
        <audience>d313c4e4-de5f-4197-9470-e509a2f0b806</audience>
    </audiences>
    <required-claims>
        <claim name="id" match="all">
            <value>insert claim here</value>
        </claim>
    </required-claims>
</validate-jwt>
```

#### <a name="authorize-access-to-operations-based-on-token-claims"></a>Autoriser l’accès aux opérations à partir de revendications de jetons

Cet exemple montre comment utiliser la stratégie [Validate JWT](api-management-access-restriction-policies.md#ValidateJWT) pour autoriser l’accès aux opérations à partir de revendications de jetons.

```xml
<validate-jwt header-name="Authorization" require-scheme="Bearer" output-token-variable-name="jwt">
    <issuer-signing-keys>
        <key>{{jwt-signing-key}}</key> <!-- signing key is stored in a named value -->
    </issuer-signing-keys>
    <audiences>
        <audience>@(context.Request.OriginalUrl.Host)</audience>
    </audiences>
    <issuers>
        <issuer>contoso.com</issuer>
    </issuers>
    <required-claims>
        <claim name="group" match="any">
            <value>finance</value>
            <value>logistics</value>
        </claim>
    </required-claims>
</validate-jwt>
<choose>
    <when condition="@(context.Request.Method == "POST" && !((Jwt)context.Variables["jwt"]).Claims["group"].Contains("finance"))">
        <return-response>
            <set-status code="403" reason="Forbidden" />
        </return-response>
    </when>
</choose>
```

### <a name="elements"></a>Éléments

| Élément             | Description                                                                                                                                                                                                                                                                                                                                           | Obligatoire |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| validate-jwt        | Élément racine.                                                                                                                                                                                                                                                                                                                                         | Oui      |
| audiences           | Contient la liste des revendications d’audience acceptables qui peuvent être présentes sur le jeton. Si plusieurs valeurs d’audience sont présentes, chacune est tentée jusqu’à ce que toutes soient épuisées (auquel cas la validation échoue) ou que l’une d’elles réussisse. Au moins une audience doit être spécifiée.                                                                     | Non       |
| issuer-signing-keys | Liste de clés de sécurité encodées en base 64 utilisé pour valider les jetons signés. Si plusieurs clés de sécurité sont présentes, chacune est tentée jusqu’à ce qu’il n’en reste plus (ce qui entraîne un échec de validation) ou jusqu’à ce que l’une d’elles soit la clé appropriée (utile pour la substitution de jeton). Les éléments clés ont un attribut `id` facultatif utilisé pour comparer à la revendication `kid`. <br/><br/>Vous pouvez également fournir une clé de signature d’émetteur en utilisant :<br/><br/> - `certificate-id` au format `<key certificate-id="mycertificate" />` pour spécifier l’identificateur d’une entité de certificat [chargée](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-certificate-entity#Add) vers le service Gestion des API<br/>- Paire RSA du modulo `n` et de l’exposant `e` au format `<key n="<modulus>" e="<exponent>" />` pour spécifier les paramètres RSA au format d’encodage base64url               | Non       |
| decryption-keys     | Liste de clés codée en Base64 utilisée pour déchiffrer les jetons. Si plusieurs clés de sécurité sont présentes, chacune est tentée jusqu’à ce qu’il n’en reste plus (ce qui entraîne un échec de validation) ou jusqu’à ce que l’une d’elles soit la clé appropriée. Les éléments clés ont un attribut `id` facultatif utilisé pour comparer à la revendication `kid`.<br/><br/>Vous pouvez également fournir une clé de déchiffrement en utilisant :<br/><br/> - `certificate-id` au format `<key certificate-id="mycertificate" />` pour spécifier l’identificateur d’une entité de certificat [chargée](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-certificate-entity#Add) vers le service Gestion des API                                                 | Non       |
| issuers             | Liste des services principaux acceptables qui ont émis le jeton. Si plusieurs valeurs d’émetteur sont présentes, chacune est tentée jusqu’à ce que toutes soient épuisées (auquel cas la validation échoue) ou que l’une d’elles réussisse.                                                                                                                                         | Non       |
| openid-config       | Élément utilisé pour spécifier un point de terminaison de configuration Open ID conforme à partir duquel l’émetteur et les clés de signature peuvent être obtenus.                                                                                                                                                                                                                        | Non       |
| required-claims     | Contient une liste de revendications censées être présentes sur le jeton pour qu’il soit considéré comme valide. Si l’attribut `match` a la valeur `all`, toutes les valeurs de revendication de la stratégie doivent être présentes dans le jeton pour que la validation réussisse. Si l’attribut `match` a la valeur `any`, au moins une revendication doit être présente dans le jeton pour que la validation réussisse. | Non       |

### <a name="attributes"></a>Attributs

| Nom                            | Description                                                                                                                                                                                                                                                                                                                                                                                                                                            | Obligatoire                                                                         | Default                                                                           |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| clock-skew                      | Intervalle de temps. Permet de spécifier l’écart maximal de durée estimée entre les horloges système de l’émetteur du jeton et l’instance de gestion des API.                                                                                                                                                                                                                                                                                                               | Non                                                                               | 0 seconde                                                                         |
| failed-validation-error-message | Message d’erreur à renvoyer dans le corps de la réponse HTTP si le JWT n’est pas validé. Les éventuels caractères spéciaux de ce message doivent être correctement placés dans une séquence d’échappement.                                                                                                                                                                                                                                                                                                 | Non                                                                               | Le message d’erreur par défaut dépend du problème de validation, par exemple « JWT absent ». |
| failed-validation-httpcode      | Code d’état HTTP à renvoyer si le JWT n’est pas validé.                                                                                                                                                                                                                                                                                                                                                                                         | Non                                                                               | 401                                                                               |
| header-name                     | Nom de l’en-tête HTTP contenant le jeton.                                                                                                                                                                                                                                                                                                                                                                                                         | `header-name`, `query-parameter-name` ou `token-value` doit être spécifié. | N/A                                                                               |
| query-parameter-name            | Nom du paramètre de la requête contenant le jeton.                                                                                                                                                                                                                                                                                                                                                                                                     | `header-name`, `query-parameter-name` ou `token-value` doit être spécifié. | N/A                                                                               |
| token-value                     | Expression renvoyant une chaîne contenant le jeton JWT                                                                                                                                                                                                                                                                                                                                                                                                     | `header-name`, `query-parameter-name` ou `token-value` doit être spécifié. | N/A                                                                               |
| id                              | L’attribut `id` sur l’élément `key` vous permet de spécifier la chaîne qui sera comparée à la revendication `kid` dans le jeton (le cas échéant) pour déterminer la clé appropriée à utiliser pour la validation de la signature.                                                                                                                                                                                                                                           | Non                                                                               | N/A                                                                               |
| match                           | L’attribut `match` sur l’élément `claim` spécifie si toutes les valeurs de revendication de la stratégie doivent être présentes dans le jeton pour que la validation réussisse. Les valeurs possibles sont les suivantes :<br /><br /> - `all` : toutes les valeurs de revendication de la stratégie doivent être présentes dans le jeton pour que la validation réussisse.<br /><br /> - `any` : au moins une valeur de revendication doit être présente dans le jeton pour que la validation réussisse.                                                       | Non                                                                               | all                                                                               |
| require-expiration-time         | Propriété booléenne. Spécifie si une revendication d’expiration est requise dans le jeton.                                                                                                                                                                                                                                                                                                                                                                               | Non                                                                               | true                                                                              |
| require-scheme                  | Le nom du schéma de jeton, par ex. « Support ». Lorsque cet attribut est défini, la stratégie garantit que le schéma spécifié est présent dans la valeur d’en-tête d’autorisation.                                                                                                                                                                                                                                                                                    | Non                                                                               | N/A                                                                               |
| require-signed-tokens           | Propriété booléenne. Spécifie si un jeton doit être signé.                                                                                                                                                                                                                                                                                                                                                                                           | Non                                                                               | true                                                                              |
| separator                       | Chaîne. Spécifie un séparateur (par exemple, « , ») à utiliser pour extraire un ensemble de valeurs à partir d’une revendication à valeurs multiples.                                                                                                                                                                                                                                                                                                                                          | Non                                                                               | N/A                                                                               |
| url                             | URL du point de terminaison de configuration Open ID à partir de laquelle les métadonnées de configuration Open ID peuvent être récupérées. La réponse devrait correspondre aux spécifications définies dans l’URL :`https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderMetadata`. Pour Azure Active Directory, utilisez l’URL suivante : `https://login.microsoftonline.com/{tenant-name}/.well-known/openid-configuration`, en remplaçant par le nom de votre client d’annuaire, par exemple, `contoso.onmicrosoft.com`. | Oui                                                                              | N/A                                                                               |
| output-token-variable-name      | Chaîne. Nom de variable de contexte qui reçoit la valeur du jeton en tant qu’objet de type [`Jwt`](api-management-policy-expressions.md) à la validation du jeton                                                                                                                                                                                                                                                                                     | Non                                                                               | N/A                                                                               |

### <a name="usage"></a>Usage

Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound
-   **Étendues de la stratégie :** toutes les étendues

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’utilisation de stratégies, consultez les pages :

-   [Stratégies dans Gestion des API](api-management-howto-policies.md)
-   [Transform and protect your API](transform-api.md) (Transformer et protéger votre API)
-   [Référence de stratégie](./api-management-policies.md) pour obtenir la liste complète des instructions et des paramètres de stratégie
-   [Exemples de stratégie](./policy-reference.md)
