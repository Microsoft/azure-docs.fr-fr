---
title: Utiliser des proxys dans Azure Functions
description: Présentation de l’utilisation de Azure Functions Proxies
ms.topic: conceptual
ms.date: 01/22/2018
ms.openlocfilehash: fb263239f99bcb4ec4c893b700d5c1cce078659f
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96601371"
---
# <a name="work-with-azure-functions-proxies"></a>Utilisation d’Azure Functions Proxies

Cet article vous explique comment configurer et utiliser Azure Functions Proxies. Cette fonctionnalité vous permet de spécifier des points de terminaison sur votre Function App implémentés par une autre ressource. Vous pouvez utiliser ces proxys pour diviser une API de grande taille en plusieurs applications Function (comme dans une architecture microservice), tout en continuant à présenter une surface API unique aux clients.

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

> [!NOTE] 
> Une facturation standard s’applique à l’exécution des proxys. Pour plus d’informations, consultez [Tarification d’Azure Functions](https://azure.microsoft.com/pricing/details/functions/).

## <a name="create-a-proxy"></a><a name="create"></a>Création d’un proxy

Cette section vous explique comment créer un proxy dans le portail Functions.

1. Ouvrez le [Azure portal] et accédez à votre Function App.
2. Dans le volet gauche, sélectionnez **Nouveau proxy**.
3. Entrez un nom pour votre proxy.
4. Configurez le point de terminaison exposé sur cette Function App en spécifiant le **modèle de routage** et les **méthodes HTTP**. Ces paramètres se comportent selon les règles des [déclencheurs HTTP].
5. Définissez l’**URL principale** sur un autre point de terminaison. Il peut s’agir d’une fonction dans une autre Function App ou bien de n’importe quelle autre API. La valeur ne doit pas nécessairement être statique et peut faire référence aux [paramètres de l’application] et aux [paramètres de la demande client d’origine].
6. Cliquez sur **Créer**.

Votre proxy existe désormais sous la forme d’un nouveau point de terminaison de votre application de fonction. Du point de vue du client, cela équivaut à un HttpTrigger dans Azure Functions. Vous pouvez essayer votre nouveau proxy en copiant l’URL de proxy et en le testant avec le client HTTP de votre choix.

## <a name="modify-requests-and-responses"></a><a name="modify-requests-responses"></a>Modification de demandes et de réponses

La fonctionnalité Azure Functions Proxies vous permet de modifier les demandes envoyées au backend et les réponses reçues de ce dernier. Ces transformations peuvent impliquer l’utilisation de variables, comme décrit dans la section [Utilisation de variables].

### <a name="modify-the-back-end-request"></a><a name="modify-backend-request"></a>Modification de la demande du serveur principal

Par défaut, la demande du serveur principal est initialisée comme une copie de la demande d’origine. Outre la définition de l’URL du serveur principal, vous pouvez apporter des modifications à la méthode HTTP, aux en-têtes et aux paramètres de chaîne de requête. Les valeurs modifiées peuvent faire référence aux [paramètres de l’application] et aux [paramètres de la demande client d’origine].

Les demandes de serveur principal peuvent être modifiées dans le portail en développant la section *remplacement de la requête* de la page de détails du proxy. 

### <a name="modify-the-response"></a><a name="modify-response"></a>Modification de la réponse

Par défaut, la réponse client est initialisée comme une copie de la réponse du serveur principal. Vous pouvez apporter des modifications au code d’état, au motif, aux en-têtes et au corps. Les valeurs modifiées peuvent faire référence aux [paramètres de l’application], aux [paramètres de la demande client d’origine] et aux [paramètres de la réponse du serveur principal].

Les demandes de serveur principal peuvent être modifiées dans le portail en développant la section *remplacement de la réponse* de la page de détails du proxy. 

## <a name="use-variables"></a><a name="using-variables"></a>Utilisation de variables

La configuration d’un proxy ne doit pas nécessairement être statique. Vous pouvez définir comme condition l’utilisation des variables de la demande client d’origine, de la réponse du backend ou des paramètres de l’application.

### <a name="reference-local-functions"></a><a name="reference-localhost"></a>Fonctions locales de référence
Vous pouvez utiliser `localhost` pour faire référence à une fonction au sein de la même application de fonction directement, sans demande proxy en aller-retour.

`"backendUri": "https://localhost/api/httptriggerC#1"` fait référence à une fonction HTTP locale à l’itinéraire `/api/httptriggerC#1`

 
>[!Note]  
>Si votre fonction utilise des niveaux d’autorisation de *fonction, administrateur ou système*, vous devez fournir le code et l’ID client, conformément à l’URL d’origine de la fonction. Dans ce cas, la référence doit ressembler à : `"backendUri": "https://localhost/api/httptriggerC#1?code=<keyvalue>&clientId=<keyname>"` Nous recommandons de stocker ces clés dans les [paramètres de l’application] et de les référencer dans vos proxies. Cela évite de stocker des secrets dans votre code source. 

### <a name="reference-request-parameters"></a><a name="request-parameters"></a>Référencement des paramètres de la demande

Les paramètres de la demande peuvent être entrés au niveau de la propriété d’URL du serveur principal ou peuvent être utilisés lors de la modification des demandes et des réponses. Certains paramètres peuvent être liés à partir du modèle de routage spécifié dans la configuration du proxy de base, alors que d’autres proviennent des propriétés de la demande entrante.

#### <a name="route-template-parameters"></a>Paramètres de modèle de routage
Les paramètres utilisés dans le modèle de routage peuvent être référencés par nom. Les noms des paramètres sont placés entre accolades ({}).

Par exemple, si un proxy dispose d’un modèle de routage du type `/pets/{petId}`, l’URL du serveur principal peut inclure la valeur de `{petId}`, comme dans `https://<AnotherApp>.azurewebsites.net/api/pets/{petId}`. Si le modèle de routage prend fin dans un caractère générique, tel que `/api/{*restOfPath}`, la valeur `{restOfPath}` est une chaîne représentant les autres segments de chemin d’accès de la demande entrante.

#### <a name="additional-request-parameters"></a>Paramètres de demande supplémentaires
Outre les paramètres de modèle de routage, les valeurs suivantes peuvent être utilisées dans les valeurs de configuration :

* **{request.method}** : méthode HTTP utilisée lors de la demande d’origine.
* **{request.headers.\<HeaderName\>}**  : un en-tête qui peut être lu à partir de la demande d’origine. Remplacez *\<HeaderName\>* par le nom de l’en-tête que vous souhaitez lire. Si l’en-tête n’est pas inclus dans la demande, la valeur sera une chaîne vide.
* **{request.querystring.\<ParameterName\>}**  : un paramètre de chaîne de requête qui peut être lu à partir de la demande d’origine. Remplacez *\<ParameterName\>* par le nom du paramètre que vous souhaitez lire. Si le paramètre n’est pas inclus dans la demande, la valeur sera une chaîne vide.

### <a name="reference-back-end-response-parameters"></a><a name="response-parameters"></a>Référencement des paramètres de réponse du serveur principal

Les paramètres de réponse peuvent être utilisés lors de la modification de la réponse au client. Les valeurs suivantes peuvent être utilisées dans la configuration :

* **{backend.response.statusCode}** : code d’état HTTP renvoyé dans la réponse du serveur principal.
* **{backend.response.statusReason}** : motif HTTP renvoyé dans la réponse du serveur principal.
* **{backend.response.headers.\<HeaderName\>}**  : un en-tête qui peut être lu à partir de la réponse du serveur principal. Remplacez *\<HeaderName\>* par le nom de l’en-tête que vous souhaitez lire. Si l’en-tête n’est pas inclus dans la réponse, la valeur sera une chaîne vide.

### <a name="reference-application-settings"></a><a name="use-appsettings"></a>Référencement des paramètres de l’application

Vous pouvez également référencer les [paramètres de l’application définis pour la Function App](./functions-how-to-use-azure-function-app-settings.md) en mettant le nom du paramètre entre signes de pourcentage (%).

Par exemple, dans une URL de serveur principal de *https://%ORDER_PROCESSING_HOST%/api/orders* , « %ORDER_PROCESSING_HOST% » sera remplacé par la valeur du paramètre ORDER_PROCESSING_HOST.

> [!TIP] 
> Utilisez des paramètres d’application pour les hôtes de serveur principal lorsque vous avez plusieurs déploiements ou environnements de test. De cette façon, vous avez l’assurance de toujours parler au backend adapté à cet environnement.

## <a name="troubleshoot-proxies"></a><a name="debugProxies"></a>Résolution des problèmes de proxy

En ajoutant l’indicateur `"debug":true` à tout proxy de votre instance `proxies.json`, vous activez la journalisation du débogage. Les journaux d’activité sont stockés dans `D:\home\LogFiles\Application\Proxies\DetailedTrace` et accessibles via les outils avancés (kudu). Toute réponse HTTP comporte également un en-tête `Proxy-Trace-Location` avec une URL dirigeant vers le fichier journal.

Pour déboguer un proxy du côté client, ajoutez un jeu d’en-têtes `Proxy-Trace-Enabled` à `true`. Ce faisant, vous enregistrez également une trace sur le système de fichiers et renvoyez l’URL de suivi en tant qu’en-tête dans la réponse.

### <a name="block-proxy-traces"></a>Bloquer les traces de proxy

Pour des raisons de sécurité, vous pouvez interdire tout appel à votre service et ainsi éviter toute génération de trace. Le cas échéant, les utilisateurs ne pourront pas accéder aux contenu de suivi sans vos informations de connexion. Notez toutefois que la génération de trace consomme des ressources et expose votre utilisation des proxys de fonction.

Désactivez les traces en ajoutant `"debug":false` à tout proxy de votre instance `proxies.json`.

## <a name="advanced-configuration"></a>Configuration avancée

Les serveurs proxy que vous configurez sont stockés dans un fichier *proxies.json* situé à la racine d’un répertoire de Function App. Vous pouvez modifier manuellement ce fichier et le déployer dans le cadre de votre application lors de l’utilisation de l’une des [méthodes de déploiement](./functions-continuous-deployment.md) prises en charge par Functions. 

> [!TIP] 
> Si vous n’avez pas configuré l’une des méthodes de déploiement, vous pouvez également utiliser le fichier *proxies.json* dans le portail. Accédez à votre Function App et sélectionnez **Fonctionnalités de la plateforme**, puis **Éditeur App Service**. Cela vous permettra d’afficher l’ensemble de la structure de fichiers de votre Function App et d’y apporter des modifications.

*Proxies.json* est défini par un objet proxy, composé de proxys nommés et de leurs définitions. Vous pouvez éventuellement référencer un [schéma JSON](http://json.schemastore.org/proxies) de complétion de code si votre éditeur est compatible. Voici un exemple de fichier :

```json
{
    "$schema": "http://json.schemastore.org/proxies",
    "proxies": {
        "proxy1": {
            "matchCondition": {
                "methods": [ "GET" ],
                "route": "/api/{test}"
            },
            "backendUri": "https://<AnotherApp>.azurewebsites.net/api/<FunctionName>"
        }
    }
}
```

Chaque proxy a un nom convivial, tel que *proxy1* dans l’exemple ci-dessus. L’objet de définition de proxy correspondant est défini par les propriétés suivantes :

* **matchCondition** : Obligatoire - un objet définissant les demandes qui déclenchent l’exécution de ce proxy. Il contient deux propriétés partagées avec les [déclencheurs HTTP] :
    * _methods_ : un tableau des méthodes HTTP auxquelles le proxy répond. À défaut de spécification, le proxy répond à toutes les méthodes HTTP sur le routage.
    * _route_ : Obligatoire - définit le modèle de routage, en contrôlant les URL de demande auxquelles votre proxy répond. Contrairement aux déclencheurs HTTP, il n’y a pas de valeur par défaut.
* **backendUri** : l’URL de la ressource principale à laquelle la demande doit être transmise par proxy. Cette valeur peut faire référence aux paramètres de l’application et à ceux de la demande client d’origine. Si cette propriété n’est pas incluse, Azure Functions répond avec un message HTTP 200 OK.
* **requestOverrides** : un objet définissant les transformations apportées à la demande du serveur principal. Consultez la section [Définition d’un objet requestOverrides].
* **responseOverrides** : un objet définissant les transformations apportées à la réponse du client. Consultez la section [Définition d’un objet responseOverrides].

> [!NOTE] 
> La propriété *route* dans Azure Functions Proxies n’honore pas la propriété *routePrefix* de la configuration d’hôte Function App. Si vous souhaitez inclure un préfixe tel que `/api`, il doit être inclus dans la propriété *route*.

### <a name="disable-individual-proxies"></a><a name="disableProxies"></a> Désactiver des proxys individuels

Pour désactivez des proxys individuels, ajoutez `"disabled": true` au proxy considéré dans le fichier `proxies.json`. Ainsi, toute requête correspondant à matchCondition renverra une erreur 404.
```json
{
    "$schema": "http://json.schemastore.org/proxies",
    "proxies": {
        "Root": {
            "disabled":true,
            "matchCondition": {
                "route": "/example"
            },
            "backendUri": "https://<AnotherApp>.azurewebsites.net/api/<FunctionName>"
        }
    }
}
```

### <a name="application-settings"></a><a name="applicationSettings"></a> Paramètres de l’application

Le comportement du proxy peut être contrôlé via plusieurs paramètres d’application. Ceux-ci sont décrits dans les [Informations de référence sur les paramètres d’application d’Azure Functions](./functions-app-settings.md)

* [AZURE_FUNCTION_PROXY_DISABLE_LOCAL_CALL](./functions-app-settings.md#azure_function_proxy_disable_local_call)
* [AZURE_FUNCTION_PROXY_BACKEND_URL_DECODE_SLASHES](./functions-app-settings.md#azure_function_proxy_backend_url_decode_slashes)

### <a name="reserved-characters-string-formatting"></a><a name="reservedChars"></a> Caractères réservés (mise en forme de chaînes)

Les proxies lisent toutes les chaînes d’un fichier JSON en utilisant \ en tant que symbole d’échappement. Les proxies interprètent également les accolades. Consultez un ensemble complet d’exemples ci-dessous.

|Caractère|Caractère d’échappement|Exemple|
|-|-|-|
|{ ou }|{{ ou }}|`{{ example }}` --> `{ example }`
| \ | \\\\ | `example.com\\text.html` --> `example.com\text.html`
|"|\\\"| `\"example\"` --> `"example"`

### <a name="define-a-requestoverrides-object"></a><a name="requestOverrides"></a>Définition d’un objet requestOverrides

L’objet requestOverrides définit les modifications apportées à la demande lors de l’appel de la ressource du serveur principal. L’objet est défini par les propriétés suivantes :

* **backend.request.method** : méthode HTTP utilisée pour appeler le backend.
* **backend.request.querystring.\<ParameterName\>**  : paramètre de chaîne de requête pouvant être défini pour l’appel au backend. Remplacez *\<ParameterName\>* par le nom du paramètre que vous souhaitez définir. Notez que si une chaîne vide est fournie, le paramètre reste inclus dans la demande du serveur principal.
* **backend.request.headers.\<HeaderName\>**  : en-tête pouvant être défini pour l’appel au backend. Remplacez *\<HeaderName\>* par le nom de l’en-tête que vous souhaitez définir. Notez que si une chaîne vide est fournie, le paramètre reste inclus dans la demande du serveur principal.

Les valeurs peuvent faire référence aux paramètres de l’application et aux paramètres de la demande client d’origine.

Voici un exemple de configuration :

```json
{
    "$schema": "http://json.schemastore.org/proxies",
    "proxies": {
        "proxy1": {
            "matchCondition": {
                "methods": [ "GET" ],
                "route": "/api/{test}"
            },
            "backendUri": "https://<AnotherApp>.azurewebsites.net/api/<FunctionName>",
            "requestOverrides": {
                "backend.request.headers.Accept": "application/xml",
                "backend.request.headers.x-functions-key": "%ANOTHERAPP_API_KEY%"
            }
        }
    }
}
```

### <a name="define-a-responseoverrides-object"></a><a name="responseOverrides"></a>Définition d’un objet responseOverrides

L’objet requestOverrides définit les modifications apportées à la réponse retransmise au client. L’objet est défini par les propriétés suivantes :

* **response.statusCode** : code d’état HTTP à renvoyer au client.
* **response.statusReason** : motif HTTP à renvoyer au client.
* **response.body** : représentation sous forme de chaîne du corps à renvoyer au client.
* **response.headers.\<HeaderName\>**  : en-tête pouvant être défini pour la réponse au client. Remplacez *\<HeaderName\>* par le nom de l’en-tête que vous souhaitez définir. Si vous fournissez une chaîne vide, l’en-tête n’est pas inclus dans la réponse.

Les valeurs peuvent faire référence aux paramètres de l’application, aux paramètres de la demande client d’origine et aux paramètres de la réponse du serveur principal.

Voici un exemple de configuration :

```json
{
    "$schema": "http://json.schemastore.org/proxies",
    "proxies": {
        "proxy1": {
            "matchCondition": {
                "methods": [ "GET" ],
                "route": "/api/{test}"
            },
            "responseOverrides": {
                "response.body": "Hello, {test}",
                "response.headers.Content-Type": "text/plain"
            }
        }
    }
}
```
> [!NOTE] 
> Dans cet exemple, le corps de la réponse est défini directement. Aucune propriété `backendUri` n’est nécessaire. Cet exemple illustre comment utiliser les Proxys Azure Functions pour simuler des API.

[Azure portal]: https://portal.azure.com
[Déclencheurs HTTP]: ./functions-bindings-http-webhook.md
[Modify the back-end request]: #modify-backend-request
[Modify the response]: #modify-response
[Définition d’un objet requestOverrides]: #requestOverrides
[Définition d’un objet responseOverrides]: #responseOverrides
[paramètres de l’application]: #use-appsettings
[Utilisation de variables]: #using-variables
[paramètres de la demande client d’origine]: #request-parameters
[paramètres de la réponse du serveur principal]: #response-parameters
