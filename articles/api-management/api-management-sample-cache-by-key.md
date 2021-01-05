---
title: Mise en cache personnalisée dans Azure API Management
description: Découvrez comment mettre en cache des éléments par clé dans Gestion des API Azure. Vous pouvez modifier la clé à l’aide des en-têtes de demande.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 772bc8dd-5cda-41c4-95bf-b9f6f052bc85
ms.service: api-management
ms.devlang: dotnet
ms.custom: devx-track-csharp
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/15/2016
ms.author: apimpm
ms.openlocfilehash: cf9901b4e49460dd2fb91dceaf239571058c5284
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88213315"
---
# <a name="custom-caching-in-azure-api-management"></a>Mise en cache personnalisée dans Azure API Management
Le service Azure API Management prend en charge la [mise en cache de réponses HTTP](api-management-howto-cache.md) en utilisant l’URL de la ressource comme clé. La clé peut être modifiée par les en-têtes de requête à l’aide des propriétés `vary-by` . Si cette approche permet de mettre en cache l’ensemble des réponses HTTP (également appelées représentations), elle peut être aussi parfois utile pour la mise en cache d’une simple partie d’une représentation. Les nouvelles stratégies [cache-lookup-value](./api-management-caching-policies.md#GetFromCacheByKey) et [cache-store-value](./api-management-caching-policies.md#StoreToCacheByKey) permettent de stocker et de récupérer des éléments de données arbitraires à partir des définitions de stratégie. Cette fonctionnalité apporte une valeur supplémentaire à la stratégie [send-request](./api-management-advanced-policies.md#SendRequest) présentée précédemment, puisqu’elle vous permet de mettre en cache les réponses à partir de services externes.

## <a name="architecture"></a>Architecture
Le service Gestion des API utilise un cache de données par locataire partagé, pour vous permettre de continuer à accéder aux mêmes données mises en cache lorsque vous déployez plusieurs unités. Lorsque vous gérez un déploiement dans plusieurs régions, chacune des régions comporte cependant des caches indépendants. Il est important de ne pas se servir du cache comme d’un magasin de données, où il constitue l’unique source de certains éléments d’informations. Si vous procédez ainsi, et que vous décidez ensuite de tirer parti du déploiement dans plusieurs régions, les clients ayant des utilisateurs nomades risquent de perdre l’accès à ces données mises en cache.

## <a name="fragment-caching"></a>Mise en cache de fragments
Il peut arriver que, dans les réponses renvoyées, une partie des données soit difficile à déterminer bien qu’elles restent à jour pendant un laps de temps raisonnable. Pensez, par exemple, à un service généré par une compagnie aérienne qui fournit des informations concernant les réservations de vol, l’état du vol, etc. Si l’utilisateur est membre du programme de points de fidélité de la compagnie aérienne, il obtiendrait également des informations relatives à l’état actuel de son adhésion et au nombre de miles cumulé. Ces informations relatives à l’utilisateur peuvent être stockées dans un autre système, mais il peut être souhaitable de les inclure dans les réponses renvoyées concernant les réservations et l’état du vol. Cette opération peut être effectuée à l’aide d’un processus appelé « mise en cache de fragments ». La représentation primaire peut être renvoyée par le serveur d’origine à l’aide d’un type de jeton pour indiquer l’emplacement où les informations relatives à l’utilisateur doivent être insérées. 

Observez la réponse JSON suivante renvoyée par une API du serveur principal.

```json
{
  "airline" : "Air Canada",
  "flightno" : "871",
  "status" : "ontime",
  "gate" : "B40",
  "terminal" : "2A",
  "userprofile" : "$userprofile$"
}  
```

Voici la ressource secondaire disponible à l’emplacement `/userprofile/{userid}` :

```json
{ "username" : "Bob Smith", "Status" : "Gold" }
```

Pour déterminer les informations utilisateur qu’il convient d’inclure, le service Gestion des API doit identifier l’utilisateur final. Ce mécanisme dépend de l’implémentation. Par exemple, j’utilise la revendication `Subject` d’un jeton `JWT`. 

```xml
<set-variable
  name="enduserid"
  value="@(context.Request.Headers.GetValueOrDefault("Authorization","").Split(' ')[1].AsJwt()?.Subject)" />
```

Le service Gestion des API stocke la valeur `enduserid` dans une variable contextuelle en vue d’une utilisation ultérieure. L’étape suivante consiste à déterminer si une demande précédente a déjà permis d’extraire les informations de l’utilisateur et de les stocker dans le cache. Pour ce faire, le service Gestion des API utilise la stratégie `cache-lookup-value`.

```xml
<cache-lookup-value
key="@("userprofile-" + context.Variables["enduserid"])"
variable-name="userprofile" />
```

Si aucune entrée du cache ne correspond à la valeur de clé, aucune variable contextuelle `userprofile` n’est créée. Pour vérifier si la recherche a abouti, le service Gestion des API utilise la stratégie de flux de contrôle `choose`.

```xml
<choose>
    <when condition="@(!context.Variables.ContainsKey("userprofile"))">
        <!-- If the userprofile context variable doesn’t exist, make an HTTP request to retrieve it.  -->
    </when>
</choose>
```

Si la variable contextuelle `userprofile` n’existe pas, le service Gestion des API doit exécuter une requête HTTP pour la récupérer.

```xml
<send-request
  mode="new"
  response-variable-name="userprofileresponse"
  timeout="10"
  ignore-error="true">

  <!-- Build a URL that points to the profile for the current end-user -->
  <set-url>@(new Uri(new Uri("https://apimairlineapi.azurewebsites.net/UserProfile/"),
      (string)context.Variables["enduserid"]).AbsoluteUri)
  </set-url>
  <set-method>GET</set-method>
</send-request>
```

Le service Gestion des API utilise `enduserid` pour construire l’URL de la ressource de profil utilisateur. Une fois la réponse obtenue, le service Gestion des API extrait le texte du corps de la réponse et le stocke dans une variable contextuelle.

```xml
<set-variable
    name="userprofile"
    value="@(((IResponse)context.Variables["userprofileresponse"]).Body.As<string>())" />
```

Pour éviter au service Gestion des API d’avoir à réexécuter cette requête HTTP, lorsque l’utilisateur effectue une autre requête, vous pouvez stocker le profil utilisateur dans le cache.

```xml
<cache-store-value
    key="@("userprofile-" + context.Variables["enduserid"])"
    value="@((string)context.Variables["userprofile"])" duration="100000" />
```

Le service Gestion des API stocke la valeur dans le cache en utilisant la même clé que celle avec laquelle il a initialement tenté de récupérer cette valeur. La durée du stockage de la valeur doit dépendre de la fréquence à laquelle les informations sont modifiées et du degré de tolérance des utilisateurs vis-à-vis des informations obsolètes. 

Il est important de comprendre que la récupération de données du cache est une requête réseau hors processus et qu’elle peut potentiellement ajouter des dizaines de millisecondes à la requête. On peut y voir des avantages lorsqu’il faut plus de temps que prévu pour déterminer les informations de profil utilisateur si l’on doit exécuter des requêtes de base de données ou agréger des informations à partir de plusieurs serveurs principaux.

La dernière étape du processus consiste à mettre à jour la réponse renvoyée avec les informations de profil utilisateur.

```xml
<!-- Update response body with user profile-->
<find-and-replace
    from='"$userprofile$"'
    to="@((string)context.Variables["userprofile"])" />
```

Vous pouvez choisir d’inclure les guillemets dans le jeton afin que même si l’opération de remplacement ne se produit pas, la réponse renvoyée prenne toujours la forme d’un script JSON valide.  

Une fois que l’on combine toutes ces étapes, on obtient une stratégie semblable à ce qui suit.

```xml
<policies>
    <inbound>
        <!-- How you determine user identity is application dependent -->
        <set-variable
          name="enduserid"
          value="@(context.Request.Headers.GetValueOrDefault("Authorization","").Split(' ')[1].AsJwt()?.Subject)" />

        <!--Look for userprofile for this user in the cache -->
        <cache-lookup-value
          key="@("userprofile-" + context.Variables["enduserid"])"
          variable-name="userprofile" />

        <!-- If API Management doesn’t find it in the cache, make a request for it and store it -->
        <choose>
            <when condition="@(!context.Variables.ContainsKey("userprofile"))">
                <!-- Make HTTP request to get user profile -->
                <send-request
                  mode="new"
                  response-variable-name="userprofileresponse"
                  timeout="10"
                  ignore-error="true">

                   <!-- Build a URL that points to the profile for the current end-user -->
                    <set-url>@(new Uri(new Uri("https://apimairlineapi.azurewebsites.net/UserProfile/"),(string)context.Variables["enduserid"]).AbsoluteUri)</set-url>
                    <set-method>GET</set-method>
                </send-request>

                <!-- Store response body in context variable -->
                <set-variable
                  name="userprofile"
                  value="@(((IResponse)context.Variables["userprofileresponse"]).Body.As<string>())" />

                <!-- Store result in cache -->
                <cache-store-value
                  key="@("userprofile-" + context.Variables["enduserid"])"
                  value="@((string)context.Variables["userprofile"])"
                  duration="100000" />
            </when>
        </choose>
        <base />
    </inbound>
    <outbound>
        <!-- Update response body with user profile-->
        <find-and-replace
              from='"$userprofile$"'
              to="@((string)context.Variables["userprofile"])" />
        <base />
    </outbound>
</policies>
```

Cette approche de mise en cache est principalement utilisée dans les sites web où le script HTML est composé côté serveur afin de pouvoir être restitué sur une seule page. Elle peut également se révéler utile dans les API où les clients ne peuvent pas effectuer de mise en cache HTTP côté client, ou lorsqu’il est préférable de ne pas confier cette responsabilité au client.

Ce même type de mise en cache de fragments peut également être effectué sur les serveurs web principaux qui utilisent un serveur de mise en cache Redis ; il est cependant utile de s’appuyer sur le service API Management pour exécuter cette opération lorsque les fragments mis en cache proviennent de serveurs principaux différents de ceux des réponses principales.

## <a name="transparent-versioning"></a>Contrôle de version transparent
Il n’est pas rare que plusieurs versions différentes d’une implémentation d’une API soient prises en charge simultanément. Cette prise en charge simultanée permet, par exemple, de gérer des environnements différents (environnements de développement, de test, de production, etc.) ou bien de prendre en charge les versions antérieures de l’API pour laisser aux consommateurs de l’API le temps d’évoluer vers des versions plus récentes. 

Au lieu de demander aux développeurs client de remplacer les URL `/v1/customers` par `/v2/customers`, il est envisageable de stocker dans les données de profil du consommateur la version de l’API qu’il souhaite actuellement utiliser et d’appeler l’URL de serveur principal appropriée. Pour déterminer l’URL de serveur principal qu’il convient d’appeler pour un client spécifique, il est nécessaire d’exécuter une requête portant sur certaines données de configuration. En mettant en cache ces données de configuration, le service Gestion des API peut limiter la baisse du niveau de performance associée à ce processus de recherche.

La première étape consiste à déterminer l’identificateur utilisé pour configurer la version souhaitée. Dans cet exemple, j’ai choisi d’associer la version à la clé d’abonnement du produit. 

```xml
<set-variable name="clientid" value="@(context.Subscription.Key)" />
```

Le service Gestion des API effectue alors une recherche dans le cache pour vérifier s’il a déjà récupéré la version client appropriée.

```xml
<cache-lookup-value
key="@("clientversion-" + context.Variables["clientid"])"
variable-name="clientversion" />
```

Ensuite, le service Gestion des API vérifie s’il n’a pas trouvé cette version dans le cache.

```xml
<choose>
    <when condition="@(!context.Variables.ContainsKey("clientversion"))">
```

S’il n’a pas trouvé la version, le service Gestion des API la récupère.

```xml
<send-request
    mode="new"
    response-variable-name="clientconfiguresponse"
    timeout="10"
    ignore-error="true">
            <set-url>@(new Uri(new Uri(context.Api.ServiceUrl.ToString() + "api/ClientConfig/"),(string)context.Variables["clientid"]).AbsoluteUri)</set-url>
            <set-method>GET</set-method>
</send-request>
```

Extrayez le texte du corps de réponse à partir de la réponse.

```xml
<set-variable
      name="clientversion"
      value="@(((IResponse)context.Variables["clientconfiguresponse"]).Body.As<string>())" />
```

Stockez-le de nouveau dans le cache pour une utilisation ultérieure.

```xml
<cache-store-value
      key="@("clientversion-" + context.Variables["clientid"])"
      value="@((string)context.Variables["clientversion"])"
      duration="100000" />
```

Pour finir, mettez à jour l’URL du serveur principal pour sélectionner la version du service souhaitée par le client.

```xml
<set-backend-service
      base-url="@(context.Api.ServiceUrl.ToString() + "api/" + (string)context.Variables["clientversion"] + "/")" />
```

La stratégie complète se présente comme suit :

```xml
<inbound>
    <base />
    <set-variable name="clientid" value="@(context.Subscription.Key)" />
    <cache-lookup-value key="@("clientversion-" + context.Variables["clientid"])" variable-name="clientversion" />

    <!-- If API Management doesn’t find it in the cache, make a request for it and store it -->
    <choose>
        <when condition="@(!context.Variables.ContainsKey("clientversion"))">
            <send-request mode="new" response-variable-name="clientconfiguresponse" timeout="10" ignore-error="true">
                <set-url>@(new Uri(new Uri(context.Api.ServiceUrl.ToString() + "api/ClientConfig/"),(string)context.Variables["clientid"]).AbsoluteUri)</set-url>
                <set-method>GET</set-method>
            </send-request>
            <!-- Store response body in context variable -->
            <set-variable name="clientversion" value="@(((IResponse)context.Variables["clientconfiguresponse"]).Body.As<string>())" />
            <!-- Store result in cache -->
            <cache-store-value key="@("clientversion-" + context.Variables["clientid"])" value="@((string)context.Variables["clientversion"])" duration="100000" />
        </when>
    </choose>
    <set-backend-service base-url="@(context.Api.ServiceUrl.ToString() + "api/" + (string)context.Variables["clientversion"] + "/")" />
</inbound>
```

En permettant aux consommateurs d’API de contrôler en toute transparence la version du serveur principal utilisée par les clients sans avoir à mettre à jour et redéployer les clients, nous proposons une solution pratique qui résout de nombreux problèmes liés au contrôle de version des API.

## <a name="tenant-isolation"></a>Isolation des locataires
Dans les déploiements mutualisés plus volumineux, certaines entreprises créent des groupes de locataires spécifiques sur des déploiements distincts de matériel du serveur principal. Cette approche contribue à réduire au minimum le nombre de clients affectés par un problème matériel sur le serveur principal. Elle permet également de déployer de nouvelles versions de logiciels de manière progressive. Dans l’idéal, cette architecture de serveur principal doit être transparente pour les consommateurs d’API. Cet objectif est réalisable en adoptant une approche similaire à celle du contrôle de version transparent, puisqu’il repose sur la même technique de manipulation de l’URL du serveur principal qui utilise un état de configuration par clé d’API.  

Au lieu de renvoyer une version par défaut de l’API pour chaque clé d’abonnement, vous devez retourner un identificateur qui associe un locataire au groupe matériel attribué. Cet identificateur peut être utilisé pour construire l’URL du serveur principal approprié.

## <a name="summary"></a>Récapitulatif
L’utilisation du cache du service Azure API Management pour le stockage de n’importe quel type de données permet d’accéder efficacement aux données de configuration susceptibles d’affecter le mode de traitement d’une demande entrante. Cette mise en cache peut également être utilisée pour stocker des fragments de données pouvant augmenter les réponses retournées à partir d’une API du serveur principal.
