---
title: En-têtes HTTP X-EC-Debug pour le moteur de règles Azure CDN | Microsoft Docs
description: L’en-tête de requête de cache de débogage X-EC-Debug fournit des informations supplémentaires sur la stratégie de cache qui est appliquée à la ressource demandée. Ces en-têtes sont spécifiques à Verizon.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/12/2018
ms.author: allensu
ms.openlocfilehash: 4154c6a1e739f935022271e7a101f39d3ee5c500
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84343018"
---
# <a name="x-ec-debug-http-headers-for-azure-cdn-rules-engine"></a>En-têtes HTTP X-EC-Debug pour le moteur de règles Azure CDN
L’en-tête de requête de cache de débogage `X-EC-Debug` fournit des informations supplémentaires sur la stratégie de cache qui est appliquée à la ressource demandée. Ces en-têtes sont spécifiques aux produits **Azure CDN Premium de Verizon**.

## <a name="usage"></a>Usage
La réponse envoyée à partir des serveurs POP à un utilisateur comprend l’en-tête `X-EC-Debug` uniquement lorsque les conditions suivantes sont remplies :

- La [fonctionnalité En-têtes de réponse de cache de débogage](https://docs.vdms.com/cdn/Content/HRE/F/Debug-Cache-Response-Headers.htm) a été activée sur le moteur de règles pour la requête spécifiée.
- La requête spécifiée définit l’ensemble des en-têtes de réponse de cache de débogage qui seront inclus dans la réponse.

## <a name="requesting-debug-cache-information"></a>Demander des informations sur le cache de débogage
Utilisez les directives suivantes dans la requête spécifiée pour définir les informations sur le cache de débogage qui seront incluses dans la réponse :

En-tête de requête | Description |
---------------|-------------|
X-EC-Debug : x-ec-cache | [Code d’état du cache](#cache-status-code-information)
X-EC-Debug : x-ec-cache-remote | [Code d’état du cache](#cache-status-code-information)
X-EC-Debug : x-ec-check-cacheable | [Pouvant être mis en cache](#cacheable-response-header)
X-EC-Debug : x-ec-cache-key | [Clé du cache](#cache-key-response-header)
X-EC-Debug : x-ec-cache-state | [État du cache](#cache-state-response-header)

### <a name="syntax"></a>Syntaxe

Des en-têtes de réponse de cache de débogage peuvent être demandés en incluant l’en-tête suivant et les directives spécifiées dans la requête :

`X-EC-Debug: Directive1,Directive2,DirectiveN`

### <a name="sample-x-ec-debug-header"></a>Exemple d’en-tête X-EC-Debug

`X-EC-Debug: x-ec-cache,x-ec-check-cacheable,x-ec-cache-key,x-ec-cache-state`

## <a name="cache-status-code-information"></a>Informations de code d’état du cache
L’en-tête de réponse X-EC-Debug peut identifier un serveur et la façon dont il a traité la réponse via les directives suivantes :

En-tête | Description
-------|------------
X-EC-Debug : x-ec-cache | Cet en-tête est signalé lorsque le contenu est acheminé via le CDN. Il identifie le serveur POP qui a traité la requête.
X-EC-Debug : x-ec-cache-remote | Cet en-tête est signalé uniquement lorsque le contenu de la requête a été mis en cache sur un serveur de protection d’origine ou un serveur de passerelle ADN.

### <a name="response-header-format"></a>Format d’en-tête de réponse

L’en-tête X-EC-Debug fournit des informations de code d’état du cache au format suivant :

- `X-EC-Debug: x-ec-cache: <StatusCode from Platform (POP/ID)>`

- `X-EC-Debug: x-ec-cache-remote: <StatusCode from Platform (POP/ID)>`

Les termes utilisés dans la syntaxe de l’en-tête de réponse ci-dessus sont définis comme suit :
- StatusCode : Indique la façon dont le contenu demandé a été traité par le CDN, qui est représenté par un code d’état du cache.
    
    Le code d’état TCP_DENIED peut être signalé à la place de NONE lorsqu’une requête non autorisée est refusée en raison de l’authentification basée sur jeton. Toutefois, le code d’état NONE continuera à être utilisé lors de l’affichage des rapports sur l’état du cache ou des données brutes du journal.

- Platform : Indique la plateforme sur laquelle le contenu a été demandé. Les codes suivants sont valides pour ce champ :

    Code  | Plateforme
    ------| --------
    ECAcc | HTTP Large
    ECS   | HTTP Small
    ECD   | ADN (Application delivery network)

- POP : Indique le [POP](cdn-pop-abbreviations.md) qui a traité la demande. 

### <a name="sample-response-headers"></a>Exemples d’en-têtes de réponse

Les exemples d’en-têtes suivants fournissent des informations de code d’état du cache pour une requête :

- `X-EC-Debug: x-ec-cache: TCP_HIT from ECD (lga/0FE8)`

- `X-EC-Debug: x-ec-cache-remote: TCP_HIT from ECD (dca/EF00)`

## <a name="cacheable-response-header"></a>En-tête de réponse Cacheable
L’en-tête de réponse `X-EC-Debug: x-ec-check-cacheable` indique si le contenu demandé pourrait avoir été mis en cache.

Cet en-tête de réponse n’indique pas si la mise en cache a eu lieu. Au lieu de cela, il indique si la requête était éligible pour la mise en cache.

### <a name="response-header-format"></a>Format d’en-tête de réponse

L’en-tête de réponse `X-EC-Debug` qui indique si une requête peut avoir été mise en cache a le format suivant :

`X-EC-Debug: x-ec-check-cacheable: <cacheable status>`

Le terme utilisé dans la syntaxe de l’en-tête de réponse ci-dessus est défini comme suit :

Valeur  | Description
-------| --------
YES    | Indique que le contenu demandé était éligible pour la mise en cache.
Non     | Indique que le contenu demandé n’était pas éligible pour la mise en cache. Cet état peut être dû à l’une des raisons suivantes : <br /> - Configuration spécifique au client : Une configuration spécifique à votre compte peut empêcher les serveurs POP de mettre en cache une ressource. Par exemple, le moteur de règles peut empêcher la mise en cache d’une ressource en activant la fonctionnalité de contournement du cache pour les requêtes applicables.<br /> - En-têtes de réponse du cache : Les en-têtes Cache-Control et Expires de la ressource demandée peuvent empêcher les serveurs POP de la mettre en cache.
UNKNOWN | Indique que les serveurs n’ont pas pu déterminer si la ressource demandée pouvait être mise en cache. Cet état se produit généralement lorsque la requête est refusée en raison de l’authentification basée sur jeton.

### <a name="sample-response-header"></a>Exemple d’en-tête de réponse

L’exemple d’en-tête de réponse suivant indique si le contenu demandé pourrait avoir été mis en cache :

`X-EC-Debug: x-ec-check-cacheable: YES`

## <a name="cache-key-response-header"></a>En-tête de réponse Cache-Key
L’en-tête de réponse `X-EC-Debug: x-ec-cache-key` indique la clé de cache physique associée au contenu demandé. Une clé de cache physique consiste en un chemin d’accès qui identifie une ressource pour les besoins de la mise en cache. En d’autres termes, les serveurs recherchent une version mise en cache d’une ressource en fonction de son chemin, tel que défini par sa clé de cache.

Cette clé de cache physique commence par une double barre oblique (//) suivie du protocole utilisé pour demander le contenu (HTTP ou HTTPS). Ce protocole est suivi du chemin d’accès relatif à la ressource demandée, qui commence par le point d’accès au contenu (par exemple, _/000001/_ ).

Par défaut, les plateformes HTTP sont configurées pour utiliser *standard-cache*, ce qui signifie que les chaînes de requête sont ignorées par le mécanisme de mise en cache. Ce type de configuration empêche la clé de cache d’inclure les données de la chaîne de requête.

Si une chaîne de requête est enregistrée dans la clé de cache, elle est convertie en son équivalent de hachage et insérée entre le nom de la ressource demandée et son extension de fichier (par exemple, ressource&lt;valeur de hachage&gt;.html).

### <a name="response-header-format"></a>Format d’en-tête de réponse

L’en-tête de réponse `X-EC-Debug` fournit des informations sur la clé de cache physique au format suivant :

`X-EC-Debug: x-ec-cache-key: CacheKey`

### <a name="sample-response-header"></a>Exemple d’en-tête de réponse

L’exemple d’en-tête de réponse suivant indique la clé de cache physique associée au contenu demandé :

`X-EC-Debug: x-ec-cache-key: //http/800001/origin/images/foo.jpg`

## <a name="cache-state-response-header"></a>En-tête de réponse Cache state
L’en-tête de réponse `X-EC-Debug: x-ec-cache-state` indique l’état du cache du contenu demandé au moment où il a été demandé.

### <a name="response-header-format"></a>Format d’en-tête de réponse

L’en-tête de réponse `X-EC-Debug` fournit des informations sur l’état du cache au format suivant :

`X-EC-Debug: x-ec-cache-state: max-age=MASeconds (MATimePeriod); cache-ts=UnixTime (ddd, dd MMM yyyy HH:mm:ss GMT); cache-age=CASeconds (CATimePeriod); remaining-ttl=RTSeconds (RTTimePeriod); expires-delta=ExpiresSeconds`

Les termes utilisés dans la syntaxe de l’en-tête de réponse ci-dessus sont définis comme suit :

- MASeconds : Indique la durée de vie maximale (en secondes) comme définie par les en-têtes Cache-Control du contenu demandé.

- MATimePeriod : Convertit la valeur de durée de vie maximale (autrement dit, MASeconds) en valeur équivalente approximative d’une unité supérieure (par exemple, jours). 

- UnixTime : Indique l’horodatage du cache du contenu demandé en temps Unix (également appelé heure POSIX ou époque Unix). L’horodatage du cache indique la date/heure de début à partir de laquelle la durée de vie d’une ressource sera calculée. 

    Si le serveur d’origine n’utilise pas de serveur de mise en cache HTTP tiers ou si ce serveur ne retourne pas l’en-tête de réponse Age, alors l’horodatage du cache sera toujours la date/heure à laquelle la ressource a été récupérée ou revalidée. Sinon, les serveurs POP utilisent le champ Age pour calculer la durée de vie de la ressource comme suit : Retrieval/RevalidateDateTime - Age.

- ddd, dd MMM yyyy HH:mm:ss GMT : Indique l’horodatage du cache du contenu demandé. Pour plus d’informations, consultez le terme UnixTime ci-dessus.

- CASeconds : Indique le nombre de secondes qui se sont écoulées depuis l’horodatage de mise en cache.

- RTSeconds : Indique le nombre de secondes restantes où le contenu mis en cache sera considéré nouveau. Cette valeur est calculée comme suit : RTSeconds = max-age - cache age.

- RTTimePeriod : Convertit la valeur de durée de vie restante (autrement dit, RTSeconds) en valeur équivalente approximative d’une unité supérieure (par exemple, jours).

- ExpiresSeconds : Indique le nombre de secondes restantes avant la date/l’heure spécifiée dans l’en-tête de réponse `Expires`. Si l’en-tête de la réponse `Expires` n’était pas inclus dans la réponse, alors la valeur de ce terme est *none*.

### <a name="sample-response-header"></a>Exemple d’en-tête de réponse

L’exemple d’en-tête de réponse suivant indique l’état du cache du contenu demandé au moment où il a été demandé :

```X-EC-Debug: x-ec-cache-state: max-age=604800 (7d); cache-ts=1341802519 (Mon, 09 Jul 2012 02:55:19 GMT); cache-age=0 (0s); remaining-ttl=604800 (7d); expires-delta=none```

