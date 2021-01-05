---
title: Stratégies de transformation de la Gestion des API Azure | Microsoft Docs
description: Découvrez les stratégies de transformation disponibles dans la Gestion des API Azure.
services: api-management
documentationcenter: ''
author: miaojiang
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 03/11/2019
ms.author: apimpm
ms.openlocfilehash: 212a6b0786b371bfb92f2e193e67d9accd432bf8
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97657241"
---
# <a name="api-management-transformation-policies"></a>Stratégies de transformation de la Gestion des API
Cette rubrique est une ressource de référence au sujet des stratégies Gestion des API suivantes. Pour plus d'informations sur l'ajout et la configuration des stratégies, consultez la page [Stratégies dans Gestion des API](./api-management-policies.md).

##  <a name="transformation-policies"></a><a name="TransformationPolicies"></a> Stratégies de transformation

-   [Convert JSON to XML](api-management-transformation-policies.md#ConvertJSONtoXML) : convertit le corps de la demande ou de la réponse de JSON en XML.

-   [Convert XML to JSON](api-management-transformation-policies.md#ConvertXMLtoJSON) :convertit le corps de la demande ou de la réponse de XML en JSON.

-   [Find and replace string in body](api-management-transformation-policies.md#Findandreplacestringinbody) : recherche une sous-chaîne de demande ou de réponse et la remplace par une autre sous-chaîne.

-   [Mask URLs in content](api-management-transformation-policies.md#MaskURLSContent) : réécrit (masque) les liens dans le corps de la réponse afin qu’ils pointent vers un lien équivalent via la passerelle.

-   [Set backend service](api-management-transformation-policies.md#SetBackendService) : modifie le service principal pour une demande entrante.

-   [Set body](api-management-transformation-policies.md#SetBody) : définit le corps du message pour les demandes entrantes et sortantes.

-   [Set HTTP header](api-management-transformation-policies.md#SetHTTPheader) : affecte une valeur à un en-tête de réponse et/ou de demande existant ou bien ajoute un nouvel en-tête de réponse et/ou de demande.

-   [Set query string parameter](api-management-transformation-policies.md#SetQueryStringParameter) : ajoute, supprime un paramètre de chaîne de requête de la demande ou le remplace par une autre valeur.

-   [URL de réécriture](api-management-transformation-policies.md#RewriteURL) : convertit une URL de demande de sa forme publique en une forme attendue par le service web.

-   [Transform XML using an XSLT](api-management-transformation-policies.md#XSLTransform) : applique une transformation de XSL en XML dans le corps de la réponse ou de la demande.

##  <a name="convert-json-to-xml"></a><a name="ConvertJSONtoXML"></a> Convert JSON to XML
 La stratégie `json-to-xml` convertit le corps de la demande ou de la réponse de JSON en XML.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<json-to-xml apply="always | content-type-json" consider-accept-header="true | false" parse-date="true | false"/>
```

### <a name="example"></a>Exemple

```xml
<policies>
    <inbound>
        <base />
    </inbound>
    <outbound>
        <base />
        <json-to-xml apply="always" consider-accept-header="false" parse-date="false"/>
    </outbound>
</policies>
```

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|json-to-xml|Élément racine.|Oui|

### <a name="attributes"></a>Attributs

|Nom|Description|Obligatoire|Default|
|----------|-----------------|--------------|-------------|
|apply|L’attribut doit avoir l’une des valeurs suivantes.<br /><br /> -   always : toujours appliquer la conversion.<br />-   content-type-json : ne convertir que si l’en-tête de réponse Content-Type indique la présence de JSON.|Oui|N/A|
|consider-accept-header|L’attribut doit avoir l’une des valeurs suivantes.<br /><br /> -   true : appliquer la conversion si le format XML est demandé dans l’en-tête d’acceptation de la demande.<br />-   false : toujours appliquer la conversion.|Non|true|
|date de l’analyse|Quand les valeurs de date sont définies sur `false`, elles sont simplement copiées lors de la transformation|Non|true|

### <a name="usage"></a>Usage
 Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound, on-error

-   **Étendues de la stratégie :** toutes les étendues

##  <a name="convert-xml-to-json"></a><a name="ConvertXMLtoJSON"></a> Convert XML to JSON
 La stratégie `xml-to-json` convertit le corps de la demande ou de la réponse de XML en JSON. Cette stratégie peut être utilisée pour moderniser les API basées sur des services web exclusivement en XML.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<xml-to-json kind="javascript-friendly | direct" apply="always | content-type-xml" consider-accept-header="true | false"/>
```

### <a name="example"></a>Exemple

```xml
<policies>
    <inbound>
        <base />
    </inbound>
    <outbound>
        <base />
        <xml-to-json kind="direct" apply="always" consider-accept-header="false" />
    </outbound>
</policies>
```

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|xml-to-json|Élément racine.|Oui|

### <a name="attributes"></a>Attributs

|Nom|Description|Obligatoire|Default|
|----------|-----------------|--------------|-------------|
|kind|L’attribut doit avoir l’une des valeurs suivantes.<br /><br /> -   javascript-friendly : le JSON converti présente un format familier aux développeurs JavaScript.<br />-   direct : le JSON converti reflète la structure d’origine du document XML.|Oui|N/A|
|apply|L’attribut doit avoir l’une des valeurs suivantes.<br /><br /> -   always : toujours convertir.<br />-   content-type-xml : ne convertir que si l’en-tête de réponse Content-Type indique la présence de XML.|Oui|N/A|
|consider-accept-header|L’attribut doit avoir l’une des valeurs suivantes.<br /><br /> -   true : appliquer la conversion si le format JSON est demandé dans l’en-tête d’acceptation de la demande.<br />-   false : toujours appliquer la conversion.|Non|true|

### <a name="usage"></a>Usage
 Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound, on-error

-   **Étendues de la stratégie :** toutes les étendues

##  <a name="find-and-replace-string-in-body"></a><a name="Findandreplacestringinbody"></a> Find and replace string in body
 La stratégie `find-and-replace` recherche une sous-chaîne de demande ou de réponse et la remplace par une autre sous-chaîne.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<find-and-replace from="what to replace" to="replacement" />
```

### <a name="example"></a>Exemple

```xml
<find-and-replace from="notebook" to="laptop" />
```

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|find-and-replace|Élément racine.|Oui|

### <a name="attributes"></a>Attributs

|Nom|Description|Obligatoire|Default|
|----------|-----------------|--------------|-------------|
|de|Chaîne à rechercher.|Oui|N/A|
|to|Chaîne de remplacement. Spécifiez une chaîne de remplacement nulle pour supprimer la chaîne de recherche.|Oui|N/A|

### <a name="usage"></a>Usage
 Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound, backend, on-error

-   **Étendues de la stratégie :** toutes les étendues

##  <a name="mask-urls-in-content"></a><a name="MaskURLSContent"></a> Mask URLs in content
 La stratégie `redirect-content-urls` réécrit (masque) les liens dans le corps de la réponse afin qu’ils pointent vers un lien équivalent par l’intermédiaire de la passerelle. À utiliser dans la section outbound pour réécrire les liens du corps de réponse afin qu’ils pointent vers la passerelle. À utiliser dans la section inbound pour obtenir l’effet opposé.

> [!NOTE]
>  Cette stratégie ne change pas les valeurs d’en-têtes, notamment des en-têtes `Location`. Pour modifier les valeurs d’en-têtes, utilisez la stratégie [set-header](api-management-transformation-policies.md#SetHTTPheader).

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<redirect-content-urls />
```

### <a name="example"></a>Exemple

```xml
<redirect-content-urls />
```

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|redirect-content-urls|Élément racine.|Oui|

### <a name="usage"></a>Usage
 Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound

-   **Étendues de la stratégie :** toutes les étendues

##  <a name="set-backend-service"></a><a name="SetBackendService"></a> Set backend service
 Utilisez la stratégie `set-backend-service` pour rediriger une demande entrante vers un service principal autre que celui qui est spécifié dans les paramètres d’API de cette opération. Cette stratégie remplace l’URL de base du service principal de la demande entrante par celle qui est spécifiée dans la stratégie.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<set-backend-service base-url="base URL of the backend service" />
```

or

```xml
<set-backend-service backend-id="identifier of the backend entity specifying base URL of the backend service" />
```

> [!NOTE]
> Les entités de back-end peuvent être gérées via l’[API](/rest/api/apimanagement/2019-12-01/backend) de gestion et [PowerShell](https://www.powershellgallery.com/packages?q=apimanagement).

### <a name="example"></a>Exemple

```xml
<policies>
    <inbound>
        <choose>
            <when condition="@(context.Request.Url.Query.GetValueOrDefault("version") == "2013-05")">
                <set-backend-service base-url="http://contoso.com/api/8.2/" />
            </when>
            <when condition="@(context.Request.Url.Query.GetValueOrDefault("version") == "2014-03")">
                <set-backend-service base-url="http://contoso.com/api/9.1/" />
            </when>
        </choose>
        <base />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```
Dans cet exemple, la stratégie du service principal définie achemine les demandes en fonction de la valeur de version transmise dans la chaîne de requête à un service principal autre que celui qui est spécifié dans l’API.

À l’origine, l’URL de base du service principal est dérivée des paramètres d’API. Par conséquent, l’URL de la demande `https://contoso.azure-api.net/api/partners/15?version=2013-05&subscription-key=abcdef` devient `http://contoso.com/api/10.4/partners/15?version=2013-05&subscription-key=abcdef`, où `http://contoso.com/api/10.4/` est l’URL du service principal spécifiée dans les paramètres d’API.

Lorsque la déclaration de stratégie [<choose\>](api-management-advanced-policies.md#choose) est appliquée, l’URL de base du service principal être de nouveau remplacée par `http://contoso.com/api/8.2` ou `http://contoso.com/api/9.1`, selon la valeur du paramètre de requête de la demande de version. Par exemple, si la valeur est `"2013-15"`, l’URL de demande finale devient `http://contoso.com/api/8.2/partners/15?version=2013-05&subscription-key=abcdef`.

Pour effectuer davantage de transformations de la demande, il est possible d’utiliser d’autres [Stratégies de transformation](api-management-transformation-policies.md#TransformationPolicies). Par exemple, pour supprimer le paramètre de requête de la version maintenant que la demande est acheminée vers un service principal propre à la version, la stratégie [Set query string parameter](api-management-transformation-policies.md#SetQueryStringParameter) peut être utilisée afin de supprimer l’attribut de version désormais superflu.

### <a name="example"></a>Exemple

```xml
<policies>
    <inbound>
        <set-backend-service backend-id="my-sf-service" sf-partition-key="@(context.Request.Url.Query.GetValueOrDefault("userId","")" sf-replica-type="primary" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```
Dans cet exemple, la stratégie permet d’acheminer la requête vers un serveur principal Service Fabric, à l’aide de la chaîne de requêtes userid en tant que clé de partition et à l’aide du réplica primaire de la partition.

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|set-backend-service|Élément racine.|Oui|

### <a name="attributes"></a>Attributs

|Nom|Description|Obligatoire|Default|
|----------|-----------------|--------------|-------------|
|base-url|Nouvelle URL de base du service principal.|`base-url` ou `backend-id` doit être présent.|N/A|
|id de principal|Identificateur du serveur principal pour l’acheminement. (Les entités de back-end sont gérées via une [API](/rest/api/apimanagement/2019-12-01/backend) et [PowerShell](https://www.powershellgallery.com/packages?q=apimanagement).)|`base-url` ou `backend-id` doit être présent.|N/A|
|clé de partition SF|Applicable uniquement lorsque le serveur principal est un service Service Fabric et qu’il est spécifié à l’aide de « id principal ». Utilisé pour résoudre une partition spécifique à partir du service de résolution des noms.|Non|N/A|
|type de réplica SF|Applicable uniquement lorsque le serveur principal est un service Service Fabric et qu’il est spécifié à l’aide de « id principal ». Contrôle si la requête doit atteindre le réplica principal ou secondaire d’une partition. |Non|N/A|
|condition de résolution SF|Applicable uniquement lorsque le serveur principal est un service Service Fabric. Condition identifiant si l’appel au serveur principal Service Fabric doit être répété avec une nouvelle résolution.|Non|N/A|
|nom d’instance de service DF|Applicable uniquement lorsque le serveur principal est un service Service Fabric. Permet de modifier les instances de service lors de l’exécution. |Non|N/A|
|sf-listener-name|S’applique uniquement lorsque le back end est un service Service Fabric et qu’il est spécifié comme « backend-id ». Service Fabric Reliable Services permet de créer plusieurs écouteurs dans un service. Cet attribut permet de sélectionner un écouteur spécifique lorsqu’un service fiable principal dispose de plusieurs écouteurs. S’il n’est pas spécifié, la Gestion des API tentera d’utiliser un écouteur sans nom. Les écouteurs sans nom sont courants pour les services fiables qui n'ont qu’un seul écouteur. |Non|N/A|

### <a name="usage"></a>Usage
 Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, backend

-   **Étendues de la stratégie :** toutes les étendues

##  <a name="set-body"></a><a name="SetBody"></a> Set body
 Utilisez la stratégie `set-body` pour définir le corps du message pour les demandes entrantes et sortantes. Pour accéder au corps du message, vous pouvez utiliser la propriété `context.Request.Body` ou `context.Response.Body`, selon que la stratégie se trouve dans la section inbound ou outbound.

> [!IMPORTANT]
>  Notez que, par défaut, lorsque vous accédez au corps du message avec `context.Request.Body` ou `context.Response.Body`, le corps du message d’origine est perdu et doit être défini en renvoyant le corps dans l’expression. Pour conserver le contenu du corps, donnez la valeur `true` au paramètre `preserveContent` lorsque vous accédez au message. Si `preserveContent` a la valeur `true` et qu’un autre corps est renvoyé par l’expression, le corps renvoyé est utilisé.
>
>  Notez les points suivants lorsque vous utilisez la stratégie `set-body`.
>
> - Si vous utilisez la stratégie `set-body` pour renvoyer un nouveau corps ou un corps mis à jour, vous n’avez pas besoin de donner la valeur `true` à `preserveContent` parce que vous fournissez explicitement le nouveau contenu du corps.
>   -   Conserver le contenu d’une réponse dans le pipeline inbound n’est pas judicieux, car il n’existe encore aucune réponse.
>   -   Conserver le contenu d’une demande dans le pipeline de sortie n’est pas judicieux, car la demande a déjà été envoyée au service principal à ce stade.
>   -   Si cette stratégie est utilisée en l’absence de corps de message, par exemple dans une requête GET inbound, une exception est levée.

 Pour plus d’informations, consultez les sections `context.Request.Body`, `context.Response.Body` et `IMessage` dans le tableau [Variable contextuelle](api-management-policy-expressions.md#ContextVariables).

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<set-body>new body value as text</set-body>
```

### <a name="examples"></a>Exemples

#### <a name="literal-text-example"></a>Exemple de texte littéral

```xml
<set-body>Hello world!</set-body>
```

#### <a name="example-accessing-the-body-as-a-string-note-that-we-are-preserving-the-original-request-body-so-that-we-can-access-it-later-in-the-pipeline"></a>Exemple d’accès au corps sous forme de chaîne. Notez que nous conservons le corps de la demande d’origine et que nous pouvons ainsi y accéder plus tard dans le pipeline.

```xml
<set-body>
@{ 
    string inBody = context.Request.Body.As<string>(preserveContent: true); 
    if (inBody[0] =='c') { 
        inBody[0] = 'm'; 
    } 
    return inBody; 
}
</set-body>
```

#### <a name="example-accessing-the-body-as-a-jobject-note-that-since-we-are-not-reserving-the-original-request-body-accessing-it-later-in-the-pipeline-will-result-in-an-exception"></a>Exemple d’accès au corps sous forme de JObject. Notez que puisque nous ne conservons pas le corps de la requête d’origine, une tentative pour y accéder ultérieurement dans le pipeline entraîne une exception.

```xml
<set-body> 
@{ 
    JObject inBody = context.Request.Body.As<JObject>(); 
    if (inBody.attribute == <tag>) { 
        inBody[0] = 'm'; 
    } 
    return inBody.ToString(); 
} 
</set-body>

```

#### <a name="filter-response-based-on-product"></a>Filtrer la réponse en fonction du produit
 Cet exemple montre comment effectuer un filtrage du contenu en supprimant des éléments de données de la réponse reçue du service principal en cas d’utilisation du produit `Starter`. Pour une démonstration de la configuration et de l’utilisation de cette stratégie, consultez [Cloud Cover Episode 177: More API Management Features with Vlad Vinogradsky](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/) (Cloud Cover, épisode 177 : Plus de fonctionnalités de la Gestion des API avec Vlad Vinogradsky) et rendez-vous directement à 34 min 30 s. Commencez à 31 min 50 s pour voir une présentation de [l’API The Dark Sky Forecast](https://developer.forecast.io/) utilisée pour cette démonstration.

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

### <a name="using-liquid-templates-with-set-body"></a>Utilisation de modèles Liquid avec Set body
La stratégie `set-body` peut être configurée pour utiliser le langage de modèle [Liquid](https://shopify.github.io/liquid/basics/introduction/) pour transformer le corps d’une requête ou réponse. Cela peut être très efficace si vous avez besoin de modifier complètement le format de votre message.

> [!IMPORTANT]
> L’implémentation Liquid utilisée dans la stratégie `set-body` est configurée en mode « C# ». Cela est particulièrement important lors d’opérations telles que le filtrage. Par exemple, l’utilisation d’un filtre de date nécessite l’emploi de la casse Pascal et de la mise en forme de date de C#, par exemple :
>
> {{body.foo.startDateTime| Date:"yyyyMMddTHH:mm:ddZ"}}

> [!IMPORTANT]
> Pour établir correctement une liaison avec un corps XML à l’aide du modèle Liquid, utilisez une stratégie `set-header` pour définir Content-Type pour application/xml, text/xml (ou n’importe quel type se terminant par +xml). Pour un corps JSON, la valeur doit être application/json, text/json (ou n’importe quel type se terminant par +json).

#### <a name="convert-json-to-soap-using-a-liquid-template"></a>Conversion de JSON en SOAP à l’aide d’un modèle Liquid
```xml
<set-body template="liquid">
    <soap:Envelope xmlns="http://tempuri.org/" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
        <soap:Body>
            <GetOpenOrders>
                <cust>{{body.getOpenOrders.cust}}</cust>
            </GetOpenOrders>
        </soap:Body>
    </soap:Envelope>
</set-body>
```

#### <a name="transform-json-using-a-liquid-template"></a>Transformation de JSON à l’aide d’un modèle Liquid
```xml
{
"order": {
    "id": "{{body.customer.purchase.identifier}}",
    "summary": "{{body.customer.purchase.orderShortDesc}}"
    }
}
```

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|set-body|Élément racine. Contient le corps du texte ou une expression qui renvoie un corps.|Oui|

### <a name="properties"></a>Propriétés

|Nom|Description|Obligatoire|Default|
|----------|-----------------|--------------|-------------|
|template|Permet de modifier le mode de création du modèle dans lequel la stratégie Set body sera exécutée. Actuellement, la seule valeur possible est :<br /><br />- liquid - la stratégie Set body utilisera le moteur de création de modèle Liquid |Non||

Pour accéder aux informations sur la requête et la réponse, le modèle Liquid peut lier à un objet de contexte aux propriétés suivantes : <br />
<pre>context.
    Request.
        Url
        Method
        OriginalMethod
        OriginalUrl
        IpAddress
        MatchedParameters
        HasBody
        ClientCertificates
        Headers

    Response.
        StatusCode
        Method
        Headers
Url.
    Scheme
    Host
    Port
    Path
    Query
    QueryString
    ToUri
    ToString

OriginalUrl.
    Scheme
    Host
    Port
    Path
    Query
    QueryString
    ToUri
    ToString
</pre>



### <a name="usage"></a>Usage
 Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound, backend

-   **Étendues de la stratégie :** toutes les étendues

##  <a name="set-http-header"></a><a name="SetHTTPheader"></a> Set HTTP header
 La stratégie `set-header` affecte une valeur à un en-tête de réponse et/ou de demande existant ou bien ajoute un nouvel en-tête de réponse et/ou de demande.

 Insère une liste d'en-têtes HTTP dans un message HTTP. Lorsqu'elle est placée dans un pipeline entrant, cette stratégie définit les en-têtes HTTP pour la demande transmise au service cible. Lorsqu’elle est placée dans un pipeline outbound, cette stratégie définit les en-têtes HTTP pour la réponse envoyée au client de la passerelle.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<set-header name="header name" exists-action="override | skip | append | delete">
    <value>value</value> <!--for multiple headers with the same name add additional value elements-->
</set-header>
```

### <a name="examples"></a>Exemples

#### <a name="example---adding-header-override-existing"></a>Exemple : ajout d’un en-tête, remplacement existant

```xml
<set-header name="some header name" exists-action="override">
    <value>20</value>
</set-header>
```
#### <a name="example---removing-header"></a>Exemple : suppression d’un en-tête

```xml
 <set-header name="some header name" exists-action="delete" />
```



#### <a name="forward-context-information-to-the-backend-service"></a>Transférer des informations de contexte au service principal
 Cet exemple montre comment appliquer la stratégie au niveau de l’API pour fournir des informations de contexte au service principal. Pour une démonstration de la configuration et de l’utilisation de cette stratégie, consultez [Cloud Cover Episode 177: More API Management Features with Vlad Vinogradsky](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/) (Cloud Cover, épisode 177 : Plus de fonctionnalités de la Gestion des API avec Vlad Vinogradsky) et rendez-vous directement à 10 min 30 s. À 12 min 10 s, une démonstration de l’appel d’une opération dans le portail des développeurs montre la stratégie à l’œuvre.

```xml
<!-- Copy this snippet into the inbound element to forward some context information, user id and the region the gateway is hosted in, to the backend service for logging or evaluation -->
<set-header name="x-request-context-data" exists-action="override">
  <value>@(context.User.Id)</value>
  <value>@(context.Deployment.Region)</value>
</set-header>
```

 Pour plus d’informations, consultez les pages [Expressions de stratégie](api-management-policy-expressions.md) et [Variable de contexte](api-management-policy-expressions.md#ContextVariables).

> [!NOTE]
> Les valeurs multiples d’un en-tête sont concaténées pour former une chaîne CSV, par exemple : `headerName: value1,value2,value3`
>
> Parmi les exceptions figurent les en-têtes normalisés dont les valeurs sont normalisées :
> - peuvent contenir des virgules (`User-Agent`, `WWW-Authenticate`, `Proxy-Authenticate`),
> - peuvent contenir une date (`Cookie`, `Set-Cookie`, `Warning`),
> - contiennent une date (`Date`, `Expires`, `If-Modified-Since`, `If-Unmodified-Since`, `Last-Modified`, `Retry-After`).
>
> Dans ces cas d’exception, les valeurs d’en-tête multiples ne sont pas concaténées en une seule chaîne et sont transmises comme en-têtes distincts, par exemple : `User-Agent: value1`
>`User-Agent: value2`
>`User-Agent: value3`

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|set-header|Élément racine.|Oui|
|value|Spécifie la valeur de l'en-tête à définir. Si plusieurs en-têtes portent le même nom, ajoutez d’autres éléments `value`.|Non|

### <a name="properties"></a>Propriétés

|Nom|Description|Obligatoire|Default|
|----------|-----------------|--------------|-------------|
|exists-action|Spécifie l’action à entreprendre lorsque l’en-tête est déjà spécifié. Cet attribut doit avoir une des valeurs suivantes.<br /><br /> - override : remplace la valeur de l’en-tête actuel.<br />- skip : ne remplace pas la valeur de l’en-tête actuel.<br />- append : ajoute la valeur à celle de l’en-tête actuel.<br />- delete : supprime l’en-tête de la demande.<br /><br /> S’il a la valeur `override`, l’inscription de plusieurs entrées portant le même nom fait que l’en-tête est défini selon toutes les entrées (qui figurent plusieurs fois) ; seules les valeurs listées seront définies dans le résultat.|Non|override|
|name|Spécifie le nom de l'en-tête à définir.|Oui|N/A|

### <a name="usage"></a>Usage
 Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound, backend, on-error

-   **Étendues de la stratégie :** toutes les étendues

##  <a name="set-query-string-parameter"></a><a name="SetQueryStringParameter"></a> Set query string parameter
 La stratégie `set-query-parameter` ajoute, supprime un paramètre de chaîne de requête de la demande ou le remplace par une autre valeur. Peut être utilisée pour transmettre les paramètres de requête attendus par le service principal qui sont facultatifs ou ne sont jamais présents dans la demande.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<set-query-parameter name="param name" exists-action="override | skip | append | delete">
    <value>value</value> <!--for multiple parameters with the same name add additional value elements-->
</set-query-parameter>
```

#### <a name="example"></a>Exemple

```xml

<set-query-parameter name="api-key" exists-action="skip">
  <value>12345678901</value>
</set-query-parameter>

```

#### <a name="forward-context-information-to-the-backend-service"></a>Transférer des informations de contexte au service principal
 Cet exemple montre comment appliquer la stratégie au niveau de l’API pour fournir des informations de contexte au service principal. Pour une démonstration de la configuration et de l’utilisation de cette stratégie, consultez [Cloud Cover Episode 177: More API Management Features with Vlad Vinogradsky](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/) (Cloud Cover, épisode 177 : Plus de fonctionnalités de la Gestion des API avec Vlad Vinogradsky) et rendez-vous directement à 10 min 30 s. À 12 min 10 s, une démonstration de l’appel d’une opération dans le portail des développeurs montre la stratégie à l’œuvre.

```xml
<!-- Copy this snippet into the inbound element to forward a piece of context, product name in this example, to the backend service for logging or evaluation -->
<set-query-parameter name="x-product-name" exists-action="override">
  <value>@(context.Product.Name)</value>
</set-query-parameter>

```

 Pour plus d’informations, consultez les pages [Expressions de stratégie](api-management-policy-expressions.md) et [Variable de contexte](api-management-policy-expressions.md#ContextVariables).

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|set-query-parameter|Élément racine.|Oui|
|value|Fournissez une valeur au paramètre de requête à définir. Si plusieurs paramètres de requête portent le même nom, ajoutez d’autres éléments `value`.|Oui|

### <a name="properties"></a>Propriétés

|Nom|Description|Obligatoire|Default|
|----------|-----------------|--------------|-------------|
|exists-action|Spécifie l’action à entreprendre lorsque le paramètre de requête est déjà spécifié. Cet attribut doit avoir une des valeurs suivantes.<br /><br /> - override : remplace la valeur du paramètre actuel.<br />- skip : ne remplace pas la valeur du paramètre de requête actuel.<br />- append : ajoute la valeur à celle du paramètre de requête actuel.<br />- delete : supprime le paramètre de requête de la demande.<br /><br /> S’il a la valeur `override`, l’ajout de plusieurs entrées portant le même nom fait que le paramètre de requête est défini selon toutes les entrées (qui figurent plusieurs fois) ; seules les valeurs listées seront définies dans le résultat.|Non|override|
|name|Spécifie le nom du paramètre de requête à définir.|Oui|N/A|

### <a name="usage"></a>Usage
 Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, backend

-   **Étendues de la stratégie :** toutes les étendues

##  <a name="rewrite-url"></a><a name="RewriteURL"></a> Rewrite URL
 La stratégie `rewrite-uri` convertit une URL de demande de sa forme publique en une forme attendue par le service web, comme le montre l’exemple suivant.

- URL publique : `http://api.example.com/storenumber/ordernumber`

- URL de la demande : `http://api.example.com/v2/US/hardware/storenumber&ordernumber?City&State`

  Cette stratégie peut être utilisée lorsqu’une URL compréhensible par un humain et/ou un navigateur doit être transformée au format d’URL attendu par le service web. Cette stratégie ne doit être appliquée qu’en cas d’exposition d’un autre format d’URL, comme les URL propres, les URL RESTful, les URL conviviales ou les URL adaptées au SEO qui sont des URL purement structurelles ne contenant pas de chaîne de requête et ne contenant que le chemin d’accès de la ressource (après le schéma et l’autorité). C'est souvent le cas pour des raisons d'esthétique, de simplicité d'utilisation ou bien pour l'optimisation des résultats de recherche (SEO).

> [!NOTE]
>  Vous ne pouvez ajouter que des paramètres de chaîne de requête avec la stratégie. Vous ne pouvez pas ajouter de paramètres de chemin d’accès de modèle dans l’URL de réécriture.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<rewrite-uri template="uri template" copy-unmatched-params="true | false" />
```

### <a name="example"></a>Exemple

```xml
<policies>
    <inbound>
        <base />
        <rewrite-uri template="/v2/US/hardware/{storenumber}&{ordernumber}?City=city&State=state" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```
```xml
<!-- Assuming incoming request is /get?a=b&c=d and operation template is set to /get?a={b} -->
<policies>
    <inbound>
        <base />
        <rewrite-uri template="/put" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
<!-- Resulting URL will be /put?c=d -->
```
```xml
<!-- Assuming incoming request is /get?a=b&c=d and operation template is set to /get?a={b} -->
<policies>
    <inbound>
        <base />
        <rewrite-uri template="/put" copy-unmatched-params="false" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
<!-- Resulting URL will be /put -->
```

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|rewrite-uri|Élément racine.|Oui|

### <a name="attributes"></a>Attributs

|Attribut|Description|Obligatoire|Default|
|---------------|-----------------|--------------|-------------|
|template|URL de service web réelle avec les paramètres de chaîne de requête. Lorsque vous utilisez des expressions, la valeur entière doit être une expression.|Oui|N/A|
|copy-unmatched-params|Spécifie si les paramètres de requête dans la requête entrante non présents dans le modèle d’URL d’origine sont ajoutés à l’URL définie par le modèle de réécriture|Non|true|

### <a name="usage"></a>Usage
 Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound

-   **Étendues de la stratégie :** toutes les étendues

##  <a name="transform-xml-using-an-xslt"></a><a name="XSLTransform"></a> Transform XML using an XSLT
 La stratégie `Transform XML using an XSLT` applique une transformation de XSL en XML dans le corps de la réponse ou de la demande.

### <a name="policy-statement"></a>Instruction de la stratégie

```xml
<xsl-transform>
    <parameter name="User-Agent">@(context.Request.Headers.GetValueOrDefault("User-Agent","non-specified"))</parameter>
    <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
        <xsl:output method="xml" indent="yes" />
        <xsl:param name="User-Agent" />
        <xsl:template match="* | @* | node()">
            <xsl:copy>
                <xsl:if test="self::* and not(parent::*)">
                    <xsl:attribute name="User-Agent">
                        <xsl:value-of select="$User-Agent" />
                    </xsl:attribute>
                </xsl:if>
                <xsl:apply-templates select="* | @* | node()" />
            </xsl:copy>
        </xsl:template>
    </xsl:stylesheet>
  </xsl-transform>
```

### <a name="example"></a>Exemple

```xml
<policies>
  <inbound>
      <base />
  </inbound>
  <outbound>
      <base />
      <xsl-transform>
          <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
            <xsl:output omit-xml-declaration="yes" method="xml" indent="yes" />
            <!-- Copy all nodes directly-->
            <xsl:template match="node()| @*|*">
                <xsl:copy>
                    <xsl:apply-templates select="@* | node()|*" />
                </xsl:copy>
            </xsl:template>
          </xsl:stylesheet>
    </xsl-transform>
  </outbound>
</policies>
```

### <a name="elements"></a>Éléments

|Nom|Description|Obligatoire|
|----------|-----------------|--------------|
|xsl-transform|Élément racine.|Oui|
|paramètre|Permet de définir des variables utilisées dans la transformation|Non|
|xsl:stylesheet|Élément feuille de style racine. Tous les éléments et attributs définis à l’intérieur respectent la [Spécification XSLT](https://www.w3.org/TR/xslt) standard.|Oui|

### <a name="usage"></a>Usage
 Cette stratégie peut être utilisée dans les [sections](./api-management-howto-policies.md#sections) et [étendues](./api-management-howto-policies.md#scopes) de stratégie suivantes.

-   **Sections de la stratégie :** inbound, outbound

-   **Étendues de la stratégie :** toutes les étendues

## <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations, voir les rubriques suivantes :

+ [Stratégies dans Gestion des API](api-management-howto-policies.md)
+ [Référence de stratégie](./api-management-policies.md) pour obtenir la liste complète des instructions et des paramètres de stratégie
+ [Exemples de stratégie](./policy-reference.md)