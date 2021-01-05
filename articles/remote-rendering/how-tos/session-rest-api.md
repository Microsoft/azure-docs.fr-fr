---
title: API REST de gestion de session
description: Décrit comment gérer les sessions
author: florianborn71
ms.author: flborn
ms.date: 02/11/2020
ms.topic: article
ms.openlocfilehash: 0af9d6906e038a4b9285a2c302fc0c98345fdbd9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90023752"
---
# <a name="use-the-session-management-rest-api"></a>Utiliser l’API REST de gestion de session

Pour utiliser la fonctionnalité Azure Remote Rendering, vous devez créer une *session*. Chaque session correspond à une machine virtuelle (VM) allouée dans Azure et en attente de la connexion d’un appareil client. Quand un appareil se connecte, la machine virtuelle affiche les données demandées et présente le résultat sous la forme d’un flux vidéo. Pendant la création de la session, vous avez choisi le type de serveur sur lequel l’exécuter, ce qui détermine la tarification. Quand plus personne n’a besoin de la session, elle doit être arrêtée. Si elle n’est pas arrêtée manuellement, il l’est automatiquement arrêté lorsque la *durée de son bail* expire.

Dans le [référentiel d’exemples ARR](https://github.com/Azure/azure-remote-rendering) dans le dossier *Scripts*, nous fournissons un script PowerShell nommé *RenderingSession. ps1* qui illustre l’utilisation de notre service. Le script et sa configuration sont décrits ici : [Exemples de scripts PowerShell](../samples/powershell-example-scripts.md)

> [!TIP]
> Les commandes PowerShell répertoriées sur cette page sont destinées à se compléter mutuellement. Si vous exécutez tous les scripts dans l’ordre à l’intérieur de la même invite de commandes PowerShell, ils s’ajoutent les uns aux autres.

## <a name="regions"></a>Régions

Pour les URL de base auxquelles envoyer les demandes, consultez la [liste des régions disponibles](../reference/regions.md).

Pour les exemples de scripts ci-dessous, nous avons choisi la région *westus2*.

### <a name="example-script-choose-an-endpoint"></a>Exemple de script : choisir un point de terminaison

```PowerShell
$endPoint = "https://remoterendering.westus2.mixedreality.azure.com"
```

## <a name="accounts"></a>Comptes

Si vous n’avez pas de compte Remote Rendering, [créez-en un](create-an-account.md). Chaque ressource est identifiée par un *accountId* utilisé dans les API de session.

### <a name="example-script-set-accountid-and-accountkey"></a>Exemple de script : définir accountId et accountKey

```PowerShell
$accountId = "********-****-****-****-************"
$accountKey = "*******************************************="
```

## <a name="common-request-headers"></a>En-têtes de demande courants

* L’en-tête *Authorization* doit avoir la valeur « `Bearer TOKEN` », où « `TOKEN` » est le jeton d’authentification [retourné par le service STS (Secure Token Service)](tokens.md).

### <a name="example-script-request-a-token"></a>Exemple de script : Demander un jeton

```PowerShell
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12
$webResponse = Invoke-WebRequest -Uri "https://sts.mixedreality.azure.com/accounts/$accountId/token" -Method Get -ContentType "application/json" -Headers @{ Authorization = "Bearer ${accountId}:$accountKey" }
$response = ConvertFrom-Json -InputObject $webResponse.Content
$token = $response.AccessToken;
```

## <a name="common-response-headers"></a>En-têtes de réponse courants

* L’en-tête *MS-CV* peut être utilisé par l’équipe produit pour suivre l’appel au sein du service.

## <a name="create-a-session"></a>Créer une session

Cette commande crée une session. Elle retourne l’ID de la nouvelle session. Vous avez besoin de celui-ci pour toutes les autres commandes.

| URI | Méthode |
|-----------|:-----------|
| /v1/accounts/*accountId*/sessions/create | POST |

**Corps de la demande :**

* maxLeaseTime (timespan) : valeur de délai d’expiration lorsque la session est désactivée automatiquement.
* models (array) : URL de conteneur de ressources à précharger.
* size (string) : taille du serveur à configurer ([ **« Standard »** ](../reference/vm-sizes.md) ou [ **« Premium »** ](../reference/vm-sizes.md)). Voir les [limitations de taille](../reference/limits.md#overall-number-of-polygons) spécifiques.

**Réponses :**

| Code d’état | Charge utile JSON | Commentaires |
|-----------|:-----------|:-----------|
| 202 | - sessionId: GUID | Succès |

### <a name="example-script-create-a-session"></a>Exemple de script : créer une session

```PowerShell
Invoke-WebRequest -Uri "$endPoint/v1/accounts/$accountId/sessions/create" -Method Post -ContentType "application/json" -Body "{ 'maxLeaseTime': '4:0:0', 'models': [], 'size': 'standard' }" -Headers @{ Authorization = "Bearer $token" }
```

Exemple de sortie :

```PowerShell
StatusCode        : 202
StatusDescription : Accepted
Content           : {"sessionId":"d31bddca-dab7-498e-9bc9-7594bc12862f"}
RawContent        : HTTP/1.1 202 Accepted
                    MS-CV: 5EqPJ1VdTUakDJZc6/ZhTg.0
                    Content-Length: 52
                    Content-Type: application/json; charset=utf-8
                    Date: Thu, 09 May 2019 16:17:50 GMT
                    Location: accounts/11111111-1111-1111-11...
Forms             : {}
Headers           : {[MS-CV, 5EqPJ1VdTUakDJZc6/ZhTg.0], [Content-Length, 52], [Content-Type, application/json;
                    charset=utf-8], [Date, Thu, 09 May 2019 16:17:50 GMT]...}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 52
```

### <a name="example-script-store-sessionid"></a>Exemple de script : stocker sessionId

La réponse à la demande ci-dessus comprend un **sessionId** dont vous avez besoin pour toutes les demandes de suivi.

```PowerShell
$sessionId = "d31bddca-dab7-498e-9bc9-7594bc12862f"
```

## <a name="modify-and-query-session-properties"></a>Modifier et interroger des propriétés de session

Il existe quelques commandes permettant d’interroger ou de modifier les paramètres de sessions existantes.

> [!CAUTION]
> Comme pour tous les appels REST, une fréquence trop élevée d’envoi de ces commandes entraîne une limitation du serveur, et finit par retourner un échec. Dans ce cas, le code d’état est 429 (« trop de demandes »). En règle générale, il doit y avoir un délai de **5 à 10 secondes entre les appels successifs**.

### <a name="update-session-parameters"></a>Mettre à jour les paramètres de session

Cette commande met à jour les paramètres d’une session. Actuellement, vous pouvez uniquement étendre la durée du bail d’une session.

> [!IMPORTANT]
> La durée du bail est toujours donnée comme une durée totale depuis le début de la session. Cela signifie que, si vous avez créé une session avec une durée de bail d’une heure que vous souhaitez prolonger d’une heure, vous devez mettre à jour sa valeur maxLeaseTime en la définissant sur deux heures.

| URI | Méthode |
|-----------|:-----------|
| /v1/accounts/*accountID*/sessions/*sessionId* | PATCH |

**Corps de la demande :**

* maxLeaseTime (timespan) : valeur de délai d’expiration lorsque la session est désactivée automatiquement.

**Réponses :**

| Code d’état | Charge utile JSON | Commentaires |
|-----------|:-----------|:-----------|
| 200 | | Succès |

#### <a name="example-script-update-a-session"></a>Exemple de script : mettre à jour une session

```PowerShell
Invoke-WebRequest -Uri "$endPoint/v1/accounts/$accountId/sessions/$sessionId" -Method Patch -ContentType "application/json" -Body "{ 'maxLeaseTime': '5:0:0' }" -Headers @{ Authorization = "Bearer $token" }
```

Exemple de sortie :

```PowerShell
StatusCode        : 200
StatusDescription : OK
Content           : {}
RawContent        : HTTP/1.1 200 OK
                    MS-CV: Fe+yXCJumky82wuoedzDTA.0
                    Content-Length: 0
                    Date: Thu, 09 May 2019 16:27:31 GMT


Headers           : {[MS-CV, Fe+yXCJumky82wuoedzDTA.0], [Content-Length, 0], [Date, Thu, 09 May 2019 16:27:31 GMT]}
RawContentLength  : 0
```

### <a name="get-active-sessions"></a>Obtenir les sessions actives

Cette commande renvoie la liste des sessions actives.

| URI | Méthode |
|-----------|:-----------|
| /v1/accounts/*accountId*/sessions | GET |

**Réponses :**

| Code d’état | Charge utile JSON | Commentaires |
|-----------|:-----------|:-----------|
| 200 | -sessions : tableau des propriétés de session | Pour obtenir une description des propriétés d’une session, consultez la section « Obtenir les propriétés de session ». |

#### <a name="example-script-query-active-sessions"></a>Exemple de script : interroger les sessions actives

```PowerShell
Invoke-WebRequest -Uri "$endPoint/v1/accounts/$accountId/sessions" -Method Get -Headers @{ Authorization = "Bearer $token" }
```

Exemple de sortie :

```PowerShell
StatusCode        : 200
StatusDescription : OK
Content           : []
RawContent        : HTTP/1.1 200 OK
                    MS-CV: WfB9Cs5YeE6S28qYkp7Bhw.1
                    Content-Length: 15
                    Content-Type: application/json; charset=utf-8
                    Date: Thu, 25 Jul 2019 16:23:50 GMT

                    {"sessions":[]}
Forms             : {}
Headers           : {[MS-CV, WfB9Cs5YeE6S28qYkp7Bhw.1], [Content-Length, 2], [Content-Type, application/json;
                    charset=utf-8], [Date, Thu, 25 Jul 2019 16:23:50 GMT]}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 2
```

### <a name="get-sessions-properties"></a>Obtenir les propriétés de session

Cette commande retourne des informations sur une session, par exemple, son nom d’hôte de machine virtuelle.

| URI | Méthode |
|-----------|:-----------|
| /v1/accounts/*accountId*/sessions/*sessionId*/properties | GET |

**Réponses :**

| Code d’état | Charge utile JSON | Commentaires |
|-----------|:-----------|:-----------|
| 200 | - message: string<br/>- sessionElapsedTime: timespan<br/>- sessionHostname: string<br/>- sessionId: string<br/>- sessionMaxLeaseTime: timespan<br/>- sessionSize: enum<br/>- sessionStatus: enum | enum sessionStatus { starting, ready, stopping, stopped, expired, error}<br/>Si l’État est « error » ou « expired », le message contient plus d’informations. |

#### <a name="example-script-get-session-properties"></a>Exemple de script : Définit les propriétés de session

```PowerShell
Invoke-WebRequest -Uri "$endPoint/v1/accounts/$accountId/sessions/$sessionId/properties" -Method Get -Headers @{ Authorization = "Bearer $token" }
```

Exemple de sortie :

```PowerShell
StatusCode        : 200
StatusDescription : OK
Content           : {"message":null,"sessionElapsedTime":"00:00:01","sessionHostname":"5018fee8-817e-4366-9179-556af79a4240.remoterenderingvm.westeurope.mixedreality.azure.com","sessionId":"e13d2c44-63e0-4591-991e-f9e05e599a93","sessionMaxLeaseTime":"04:00:00","sessionStatus":"Ready"}
RawContent        : HTTP/1.1 200 OK
                    MS-CV: CMXegpZRMECH4pbOW2j5GA.0
                    Content-Length: 60
                    Content-Type: application/json; charset=utf-8
                    Date: Thu, 09 May 2019 16:30:38 GMT

                    {"message":null,...
Forms             : {}
Headers           : {[MS-CV, CMXegpZRMECH4pbOW2j5GA.0], [Content-Length, 60], [Content-Type, application/json;
                    charset=utf-8], [Date, Thu, 09 May 2019 16:30:38 GMT]}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 60
```

## <a name="stop-a-session"></a>Arrêter une session

Cette commande arrête une session. La machine virtuelle allouée sera récupérée peu après.

| URI | Méthode |
|-----------|:-----------|
| /v1/accounts/*accountId*/sessions/*sessionId* | Suppression |

**Réponses :**

| Code d’état | Charge utile JSON | Commentaires |
|-----------|:-----------|:-----------|
| 204 | | Succès |

### <a name="example-script-stop-a-session"></a>Exemple de script : arrêter une session

```PowerShell
Invoke-WebRequest -Uri "$endPoint/v1/accounts/$accountId/sessions/$sessionId" -Method Delete -Headers @{ Authorization = "Bearer $token" }
```

Exemple de sortie :

```PowerShell
StatusCode        : 204
StatusDescription : No Content
Content           : {}
RawContent        : HTTP/1.1 204 No Content
                    MS-CV: YDxR5/7+K0KstH54WG443w.0
                    Date: Thu, 09 May 2019 16:45:41 GMT


Headers           : {[MS-CV, YDxR5/7+K0KstH54WG443w.0], [Date, Thu, 09 May 2019 16:45:41 GMT]}
RawContentLength  : 0
```

## <a name="next-steps"></a>Étapes suivantes

* [Exemples de scripts PowerShell](../samples/powershell-example-scripts.md)
