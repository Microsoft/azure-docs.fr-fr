---
title: Liaisons de sortie HTTP Azure Functions
description: Découvrez comment retourner des réponses HTTP dans Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/21/2020
ms.author: cshoe
ms.openlocfilehash: a25658677e436edf4d001599bb4981f527016596
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84697440"
---
# <a name="azure-functions-http-output-bindings"></a>Liaisons de sortie HTTP Azure Functions

Utilisez la liaison de sortie HTTP pour répondre à l’expéditeur de la demande HTTP. Cette liaison requiert un déclencheur HTTP, et vous permet de personnaliser la réponse associée à la demande du déclencheur.

La valeur de retour par défaut pour une fonction déclenchée par HTTP est :

- `HTTP 204 No Content` avec un corps vide dans Functions 2.x et versions ultérieures
- `HTTP 200 OK` avec un corps vide dans Functions 1.x

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json*. Pour les bibliothèques de classes C#, aucune propriété d’attribut ne correspond à ces propriétés de *function.json*.

|Propriété  |Description  |
|---------|---------|
| **type** |Cette propriété doit être définie sur `http`. |
| **direction** | Cette propriété doit être définie sur `out`. |
| **name** | Nom de variable utilisé dans le code de fonction pour la réponse, ou `$return` pour utiliser la valeur renvoyée. |

## <a name="usage"></a>Usage

Pour envoyer une réponse HTTP, utilisez les modèles de réponse standard du langage. Dans un script C# ou C#, choisissez le type de retour de fonction `IActionResult` ou `Task<IActionResult>`. En C#, aucun attribut de valeur renvoyée n’est requis.

Par obtenir des exemples de réponse, consultez [l’exemple de déclencheur](./functions-bindings-http-webhook-trigger.md#example).

## <a name="hostjson-settings"></a>Paramètres host.json

Cette section décrit les paramètres de configuration globaux disponibles pour cette liaison dans les versions 2.x et ultérieures. L’exemple de fichier host.json ci-dessous contient seulement les paramètres des versions 2.x et ultérieures pour cette liaison. Pour plus d’informations sur les paramètres de configuration globaux dans les versions 2.x et ultérieures, consultez [Informations de référence sur le fichier host.json pour Azure Functions](functions-host-json.md).

> [!NOTE]
> Pour obtenir une référence de host.json dans Functions 1.x, consultez [Informations de référence sur le fichier host.json pour Azure Functions 1.x](functions-host-json-v1.md#http).

```json
{
    "extensions": {
        "http": {
            "routePrefix": "api",
            "maxOutstandingRequests": 200,
            "maxConcurrentRequests": 100,
            "dynamicThrottlesEnabled": true,
            "hsts": {
                "isEnabled": true,
                "maxAge": "10"
            },
            "customHeaders": {
                "X-Content-Type-Options": "nosniff"
            }
        }
    }
}
```

|Propriété  |Default | Description |
|---------|---------|---------| 
| customHeaders|Aucun|Vous permet de définir des en-têtes personnalisés dans la réponse HTTP. L’exemple précédent ajoute l’en-tête `X-Content-Type-Options` à la réponse pour éviter la détection du type de contenu. |
|dynamicThrottlesEnabled|true<sup>\*</sup>|Quand il est activé, ce paramètre provoque la vérification périodique des compteurs de performances système, comme `connections/threads/processes/memory/cpu/etc`, par le pipeline de traitement des requêtes. Si un de ces compteurs dépasse un seuil supérieur intégré (80 %), les requêtes sont rejetées avec une réponse `429 "Too Busy"` jusqu’à ce que le ou les compteurs reviennent à un niveau normal.<br/><sup>\*</sup>La valeur par défaut d’un plan Consommation est `true`. La valeur par défaut dans un plan dédié est `false`.|
|hsts|non activé|Lorsque `isEnabled` est défini sur `true`, le [comportement HTTP Strict Transport Security (HSTS) de .NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-3.0&tabs=visual-studio#hsts) est appliqué, comme défini dans la [classe `HstsOptions`](/dotnet/api/microsoft.aspnetcore.httpspolicy.hstsoptions?view=aspnetcore-3.0). L’exemple ci-dessus définit également la propriété [`maxAge`](/dotnet/api/microsoft.aspnetcore.httpspolicy.hstsoptions.maxage?view=aspnetcore-3.0#Microsoft_AspNetCore_HttpsPolicy_HstsOptions_MaxAge) sur 10 jours. Les propriétés prises en charge de `hsts` sont : <table><tr><th>Propriété</th><th>Description</th></tr><tr><td>excludedHosts</td><td>Tableau de chaînes des noms d’hôtes pour lesquels l’en-tête HSTS n’est pas ajouté.</td></tr><tr><td>includeSubDomains</td><td>Valeur booléenne qui indique si le paramètre includeSubDomain de l’en-tête Strict-Transport-Security est activé.</td></tr><tr><td>maxAge</td><td>Chaîne qui définit le paramètre max-age de l’en-tête Strict-Transport-Security.</td></tr><tr><td>preload</td><td>Valeur booléenne qui indique si le paramètre preload de l’en-tête Strict-Transport-Security est activé.</td></tr></table>|
|maxConcurrentRequests|100<sup>\*</sup>|Nombre maximal de fonctions HTTP exécutées en parallèle. Cette valeur vous permet de contrôler la concurrence et facilite ainsi la gestion de l’utilisation des ressources. Par exemple, vous pouvez avoir une fonction HTTP qui utilise un grand nombre de ressources système (mémoire/processeur/sockets) et qui provoque par conséquent des situations où la concurrence est trop élevée. Vous pouvez également avoir une fonction qui effectue des requête sortantes vers un service tiers et qui émet à ce titre des appels dont le débit doit être limité. Dans ces cas, il peut être pertinent d’appliquer une limitation. <br/><sup>*</sup>La valeur par défaut d’un plan Consommation est 100. La valeur par défaut d’un plan dédié est illimitée (`-1`).|
|maxOutstandingRequests|200<sup>\*</sup>|Nombre maximal de requêtes en attente qui ont lieu à un moment donné. La limite inclut les requêtes qui sont en file d’attente, mais dont l’exécution n’a pas démarré, ainsi que les exécutions en cours. Toutes les requêtes entrantes au-delà de cette limite sont rejetées avec une réponse 429 « Trop occupé ». Ainsi, les appelants peuvent appliquer des stratégies temporelles de nouvelle tentative et vous êtes en mesure de contrôler les latences maximales de requêtes. Ce paramètre contrôle uniquement la mise en file d’attente qui se produit dans le chemin d’accès d’exécution de l’hôte de script. Les autres files d’attente, telles que la file d’attente des requêtes ASP.NET, sont toujours actives et ne sont pas concernées par ce paramètre. <br/><sup>\*</sup>La valeur par défaut d’un plan Consommation est 200. La valeur par défaut d’un plan dédié est illimitée (`-1`).|
|routePrefix|api|Préfixe d’itinéraire qui s’applique à tous les itinéraires. Utilisez une chaîne vide pour supprimer le préfixe par défaut. |

## <a name="next-steps"></a>Étapes suivantes

- [Exécuter une fonction à partir d’une requête HTTP](./functions-bindings-http-webhook-trigger.md)