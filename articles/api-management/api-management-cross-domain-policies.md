---
title: Stratégies inter-domaines dans Gestion des API Azure | Microsoft Docs
description: Découvrez les stratégies inter-domaines disponibles dans Gestion des API Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 7689d277-8abe-472a-a78c-e6d4bd43455d
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 07/14/2020
ms.author: apimpm
ms.openlocfilehash: 77d9d20f3321aa5bb6c5ea47a3949a82bdd1ad75
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92131239"
---
# <a name="api-management-cross-domain-policies"></a>Gestion des API dans les stratégies de domaine
Cette rubrique est une ressource de référence au sujet des stratégies Gestion des API suivantes. Pour plus d'informations sur l'ajout et la configuration des stratégies, consultez la page [Stratégies dans Gestion des API](./api-management-policies.md).

## <a name="cross-domain-policies"></a><a name="CrossDomainPolicies"></a> Stratégies inter-domaines

- [Allow cross-domain calls](api-management-cross-domain-policies.md#AllowCrossDomainCalls) : rend l'API accessible depuis les navigateurs clients utilisant Adobe Flash et Microsoft Silverlight.
- [CORS](api-management-cross-domain-policies.md#CORS) : ajoute une prise en charge partage des ressources cross-origin (CORS) à une opération ou une API afin de permettre les appels interdomaines depuis les navigateurs clients.
- [JSONP](api-management-cross-domain-policies.md#JSONP) : ajoute une prise en charge de JSON avec remplissage (JSONP) à une opération ou une API afin de permettre les appels interdomaines depuis les navigateurs clients utilisant JavaScript.

## <a name="allow-cross-domain-calls"></a><a name="AllowCrossDomainCalls"></a> Allow cross-domain calls
La stratégie `cross-domain` rend l’API accessible depuis les navigateurs clients utilisant Adobe Flash et Microsoft Silverlight.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<cross-domain>
    <!-Policy configuration is in the Adobe cross-domain policy file format,
        see https://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html-->
</cross-domain>
```

### <a name="example"></a>Exemple

```xml
<cross-domain>
        <allow-http-request-headers-from domain='*' headers='*' />
</cross-domain>
```

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|inter-domaines|Élément racine. Les éléments enfants doivent être conformes à la [spécification de fichier de stratégie inter-domaines Adobe](https://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html).|Oui|

### <a name="usage"></a>Usage
Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

- **Sections de la stratégie :** inbound
- **Étendues de la stratégie :** toutes les étendues

## <a name="cors"></a><a name="CORS"></a> CORS
La stratégie `cors` ajoute la prise en charge du partage des ressources cross-origin (CORS) à une opération ou une API afin de permettre les appels inter-domaines à partir des navigateurs clients.

CORS permet à un navigateur et à un serveur d'interagir et de déterminer si les demandes cross-origin doivent être autorisées ou non, par exemple dans le cas d'appels XMLHttpRequests passés via JavaScript sur une page web vers d'autres domaines). Cette stratégie offre plus de flexibilité que de simplement autoriser les demandes de même origine, mais elle est plus sûre que d'autoriser toutes les demandes cross-origin.

Vous devez appliquer la stratégie CORS pour activer la console interactive dans le portail des développeurs. Pour plus d’informations, reportez-vous à la [documentation du portail des développeurs](./api-management-howto-developer-portal.md#cors).

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<cors allow-credentials="false|true">
    <allowed-origins>
        <origin>origin uri</origin>
    </allowed-origins>
    <allowed-methods preflight-result-max-age="number of seconds">
        <method>http verb</method>
    </allowed-methods>
    <allowed-headers>
        <header>header name</header>
    </allowed-headers>
    <expose-headers>
        <header>header name</header>
    </expose-headers>
</cors>
```

### <a name="example"></a>Exemple
Cet exemple montre comment prendre en charge les demandes en amont, telles que celles comportant des en-têtes personnalisés ou des méthodes autres que GET et POST. Pour prendre en charge les en-têtes personnalisés et autres verbes HTTP, utilisez les sections `allowed-methods` et `allowed-headers` comme indiqué dans l’exemple suivant.

```xml
<cors allow-credentials="true">
    <allowed-origins>
        <!-- Localhost useful for development -->
        <origin>http://localhost:8080/</origin>
        <origin>http://example.com/</origin>
    </allowed-origins>
    <allowed-methods preflight-result-max-age="300">
        <method>GET</method>
        <method>POST</method>
        <method>PATCH</method>
        <method>DELETE</method>
    </allowed-methods>
    <allowed-headers>
        <!-- Examples below show Azure Mobile Services headers -->
        <header>x-zumo-installation-id</header>
        <header>x-zumo-application</header>
        <header>x-zumo-version</header>
        <header>x-zumo-auth</header>
        <header>content-type</header>
        <header>accept</header>
    </allowed-headers>
    <expose-headers>
        <!-- Examples below show Azure Mobile Services headers -->
        <header>x-zumo-installation-id</header>
        <header>x-zumo-application</header>
    </expose-headers>
</cors>
```

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|Default|
|----------|-----------------|--------------|-------------|
|cors|Élément racine.|Oui|N/A|
|allowed-origins|Contient des éléments `origin` qui décrivent les origines autorisées pour les demandes inter-domaines. `allowed-origins` peut contenir un seul élément `origin` qui spécifie `*` pour autoriser toute origine, ou un ou plusieurs éléments `origin` contenant un URI.|Oui|N/A|
|origin|La valeur peut être `*` pour autoriser toutes les origines, ou un URI qui spécifie une origine unique. L'URI doit comprendre un modèle, un hôte et un port.|Oui|Si le port n’est pas spécifié dans l’URI, le port 80 est utilisé pour HTTP et le port 443 pour HTTPS.|
|allowed-methods|Cet élément est requis si les méthodes autres que GET ou POST sont autorisées. Contient des éléments `method` qui spécifient les verbes HTTP pris en charge. La valeur `*` indique toutes les méthodes.|Non|Si cette section n’est pas présente, les méthodes GET et POST sont prises en charge.|
|method|Spécifie un verbe HTTP.|Au moins un élément `method` est requis si la section `allowed-methods` est présente.|N/A|
|allowed-headers|Cet élément contient des éléments `header` spécifiant les noms des en-têtes qui peuvent être inclus dans la demande.|Non|N/A|
|expose-headers|Cet élément contient des éléments `header` spécifiant les noms des en-têtes accessibles par le client.|Non|N/A|
|en-tête|Spécifie un nom d’en-tête.|Au moins un élément `header` est requis dans `allowed-headers` ou `expose-headers` si la section est présente.|N/A|

### <a name="attributes"></a>Attributs

|Nom|Description|Obligatoire|Default|
|----------|-----------------|--------------|-------------|
|allow-credentials|L’en-tête `Access-Control-Allow-Credentials` de la réponse en amont est défini sur la valeur de cet attribut et influe sur la capacité du client à envoyer des informations d’identification dans les demandes inter-domaines.|Non|false|
|preflight-result-max-age|L’en-tête `Access-Control-Max-Age` de la réponse en amont est défini sur la valeur de cet attribut et influe sur la capacité de l’agent utilisateur à mettre en cache la réponse en amont.|Non|0|

### <a name="usage"></a>Usage
Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

- **Sections de la stratégie :** inbound
- **Étendues de la stratégie :** toutes les étendues

## <a name="jsonp"></a><a name="JSONP"></a> JSONP
La stratégie `jsonp` ajoute la prise en charge de JSON avec remplissage (JSONP) à une opération ou une API afin de permettre les appels inter-domaines à partir des navigateurs clients utilisant JavaScript. JSONP est une méthode utilisée par les programmes JavaScript pour demander des données à un serveur se trouvant dans un autre domaine. JSONP passe outre la limite appliquée par la plupart des navigateurs web, selon laquelle l'accès aux pages web doit se trouver dans le même domaine.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<jsonp callback-parameter-name="callback function name" />
```

### <a name="example"></a>Exemple

```xml
<jsonp callback-parameter-name="cb" />
```

Si vous appelez la méthode sans le paramètre de rappel ?cb=XXX, elle renvoie un code JSON simple (sans enveloppe d’appel de fonction).

Si vous ajoutez le paramètre de rappel `?cb=XXX`, il renvoie un résultat JSONP, enveloppant les résultats JSON d’origine autour de la fonction de rappel, comme `XYZ('<json result goes here>');`

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|jsonp|Élément racine.|Oui|

### <a name="attributes"></a>Attributs

|Nom|Description|Obligatoire|Default|
|----------|-----------------|--------------|-------------|
|callback-parameter-name|Appel de fonction JavaScript interdomaines avec comme préfixe le nom de domaine complet de l'emplacement de la fonction.|Oui|N/A|

### <a name="usage"></a>Usage
Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

- **Sections de la stratégie :** outbound (sortant)
- **Étendues de la stratégie :** toutes les étendues

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’utilisation de stratégies, consultez les pages :

+ [Stratégies dans Gestion des API](api-management-howto-policies.md)
+ [Transform and protect your API](transform-api.md) (Transformer et protéger votre API)
+ [Référence de stratégie](./api-management-policies.md) pour obtenir la liste complète des instructions et des paramètres de stratégie
+ [Exemples de stratégie](./policy-reference.md)