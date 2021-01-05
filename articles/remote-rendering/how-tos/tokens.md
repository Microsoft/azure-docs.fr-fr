---
title: Obtenir des jetons d’accès au service
description: Décrit comment créer des jetons pour l’accès aux API REST ARR
author: florianborn71
ms.author: flborn
ms.date: 02/11/2020
ms.topic: how-to
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 7e8e2f3f9dd49693faa26eaaab309fcad58f6f9f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89076155"
---
# <a name="get-service-access-tokens"></a>Obtenir des jetons d’accès au service

L’accès aux API REST ARR est accordé uniquement aux utilisateurs autorisés. Vous devez envoyer un *jeton d’accès* avec les demandes REST comme preuve de votre autorisation. Ces jetons sont émis par le *service d’émission de jeton de sécurité* (STS, Secure Token Service) en échange d’une clé de compte. Les jetons ont une **durée de vie de 24 heures** et peuvent donc être fournis aux utilisateurs sans que ceux-ci ne disposent d’un accès complet au service.

Cet article explique comment créer ce type de jeton d’accès.

## <a name="prerequisites"></a>Prérequis

[Créez un compte ARR](create-an-account.md) si ce n’est déjà fait.

## <a name="token-service-rest-api"></a>API REST du service de jetons

Le *service d’émission de jeton de sécurité* fournit une API REST unique pour la création de jetons d’accès. L’URL du service STS ARR est https:\//sts.mixedreality.azure.com.

### <a name="get-token-request"></a>Demande d’obtention de jeton

| URI | Méthode |
|-----------|:-----------|
| /accounts/**accountId**/token | GET |

| En-tête | Valeur |
|--------|:------|
| Autorisation | "Bearer **accountId**:**accountKey**" |

Remplacez *accountId* et *accountKey* par vos propres données.

### <a name="get-token-response"></a>Réponse à une demande d’obtention de jeton

| Code d’état | Charge utile JSON | Commentaires |
|-----------|:-----------|:-----------|
| 200 | AccessToken : chaîne | Succès |

| En-tête | Objectif |
|--------|:------|
| MS-CV | Cette valeur peut être utilisée pour suivre l’appel dans le service. |

## <a name="getting-a-token-using-powershell"></a>Obtention d’un jeton avec PowerShell

Le code PowerShell ci-dessous montre comment envoyer la demande REST nécessaire au service STS. Il affiche ensuite le jeton dans l’invite PowerShell.

```PowerShell
$accountId = "<account_id_from_portal>"
$accountKey = "<account_key_from_portal>"

[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12;
$webResponse = Invoke-WebRequest -Uri "https://sts.mixedreality.azure.com/accounts/$accountId/token" -Method Get -Headers @{ Authorization = "Bearer ${accountId}:$accountKey" }
$response = ConvertFrom-Json -InputObject $webResponse.Content

Write-Output "Token: $($response.AccessToken)"
```

Le script affiche simplement le jeton dans la sortie. Vous pouvez y copier le jeton et le coller. Pour un projet réel, vous devez automatiser ce processus.

## <a name="next-steps"></a>Étapes suivantes

* [Exemples de scripts PowerShell](../samples/powershell-example-scripts.md)
* [API front-end Azure](../how-tos/frontend-apis.md)
* [API REST de gestion de session](../how-tos/session-rest-api.md)
