---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 03/29/2019
ms.author: erhopf
ms.openlocfilehash: 410b73d3f9011f9384fafa18394d9318e3eafa67
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97504489"
---
## <a name="authentication"></a>Authentication

Chaque requête nécessite un en-tête d’autorisation. Ce tableau présente les en-têtes pris en charge pour chaque service :

| En-têtes d'autorisation pris en charge | Reconnaissance vocale | Synthèse vocale |
|------------------------|----------------|----------------|
| Ocp-Apim-Subscription-Key | Oui | Non |
| Autorisation : Bearer | Oui | Oui |

Lorsque vous utilisez l’en-tête `Ocp-Apim-Subscription-Key`, vous devez uniquement fournir votre clé d’abonnement. Par exemple :

```http
'Ocp-Apim-Subscription-Key': 'YOUR_SUBSCRIPTION_KEY'
```

Lorsque vous utilisez l’en-tête `Authorization: Bearer`, vous devez envoyer une demande au point de terminaison `issueToken`. Dans cette requête, vous échangez votre clé d’abonnement contre un jeton d’accès valide pendant 10 minutes. Dans les sections suivantes, vous apprendrez à obtenir et utiliser un jeton.

### <a name="how-to-get-an-access-token"></a>Obtenir un jeton d’accès

Pour obtenir un jeton d’accès, vous devrez envoyer une demande au point de terminaison `issueToken` à l’aide de `Ocp-Apim-Subscription-Key` et de votre clé d’abonnement.

Le format du point de terminaison `issueToken` est le suivant :

```http
https://<REGION_IDENTIFIER>.api.cognitive.microsoft.com/sts/v1.0/issueToken
```

Remplacez `<REGION_IDENTIFIER>` par l’identificateur correspondant à la région de votre abonnement dans le tableau suivant :

[!INCLUDE [](cognitive-services-speech-service-region-identifier.md)]

Utilisez ces exemples pour créer votre demande de jeton d’accès.

#### <a name="http-sample"></a>Exemple HTTP

Cet exemple est une simple requête HTTP pour obtenir un jeton. Remplacez `YOUR_SUBSCRIPTION_KEY` par votre clé d’abonnement de service de reconnaissance vocale. Si votre abonnement n’est pas dans la région USA Ouest, remplacez l’en-tête `Host` par le nom d’hôte de votre région.

```http
POST /sts/v1.0/issueToken HTTP/1.1
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
Host: westus.api.cognitive.microsoft.com
Content-type: application/x-www-form-urlencoded
Content-Length: 0
```

Le corps de la réponse contient le jeton d’accès au format JSON Web Token (JWT).

#### <a name="powershell-sample"></a>Exemple de code PowerShell

Cet exemple est un simple script PowerShell pour obtenir un jeton d’accès. Remplacez `YOUR_SUBSCRIPTION_KEY` par votre clé d’abonnement de service de reconnaissance vocale. Veillez à utiliser le point de terminaison correct pour la région correspondant à votre abonnement. Cet exemple est actuellement configuré pour l’USA Ouest.

```powershell
$FetchTokenHeader = @{
  'Content-type'='application/x-www-form-urlencoded';
  'Content-Length'= '0';
  'Ocp-Apim-Subscription-Key' = 'YOUR_SUBSCRIPTION_KEY'
}

$OAuthToken = Invoke-RestMethod -Method POST -Uri https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken
 -Headers $FetchTokenHeader

# show the token received
$OAuthToken

```

#### <a name="curl-sample"></a>Exemple cURL

cURL est un outil en ligne de commande disponible dans Linux (ainsi que dans le sous-système Windows pour Linux). Cette commande cURL montre comment obtenir un jeton d’accès. Remplacez `YOUR_SUBSCRIPTION_KEY` par votre clé d’abonnement de service de reconnaissance vocale. Veillez à utiliser le point de terminaison correct pour la région correspondant à votre abonnement. Cet exemple est actuellement configuré pour l’USA Ouest.

```console
curl -v -X POST \
 "https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken" \
 -H "Content-type: application/x-www-form-urlencoded" \
 -H "Content-Length: 0" \
 -H "Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY"
```

#### <a name="c-sample"></a>Exemple de code C#

La classe C# montre comment obtenir un jeton d’accès. Transmettez votre clé d’abonnement du service Speech quand vous instanciez la classe. Si votre abonnement ne figure pas dans la région USA Ouest, modifiez la valeur `FetchTokenUri` afin qu’elle corresponde à la région de votre abonnement.

```csharp
public class Authentication
{
    public static readonly string FetchTokenUri =
        "https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken";
    private string subscriptionKey;
    private string token;

    public Authentication(string subscriptionKey)
    {
        this.subscriptionKey = subscriptionKey;
        this.token = FetchTokenAsync(FetchTokenUri, subscriptionKey).Result;
    }

    public string GetAccessToken()
    {
        return this.token;
    }

    private async Task<string> FetchTokenAsync(string fetchUri, string subscriptionKey)
    {
        using (var client = new HttpClient())
        {
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);
            UriBuilder uriBuilder = new UriBuilder(fetchUri);

            var result = await client.PostAsync(uriBuilder.Uri.AbsoluteUri, null);
            Console.WriteLine("Token Uri: {0}", uriBuilder.Uri.AbsoluteUri);
            return await result.Content.ReadAsStringAsync();
        }
    }
}
```

#### <a name="python-sample"></a>Exemple de code Python

```python
# Request module must be installed.
# Run pip install requests if necessary.
import requests

subscription_key = 'REPLACE_WITH_YOUR_KEY'


def get_token(subscription_key):
    fetch_token_url = 'https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken'
    headers = {
        'Ocp-Apim-Subscription-Key': subscription_key
    }
    response = requests.post(fetch_token_url, headers=headers)
    access_token = str(response.text)
    print(access_token)
```

### <a name="how-to-use-an-access-token"></a>Utiliser un jeton d’accès

Le jeton d’accès doit être envoyé au service en tant qu’en-tête `Authorization: Bearer <TOKEN>`. Chaque jeton d’accès est valide pour une durée de 10 minutes. Vous pouvez à tout moment obtenir un nouveau jeton, mais pour réduire la latence et le trafic réseau, nous recommandons d’utiliser le même jeton durant neuf minutes.

Voici un exemple de requête HTTP adressée à l’API REST de reconnaissance vocale pour audio court :

```http
POST /cognitiveservices/v1 HTTP/1.1
Authorization: Bearer YOUR_ACCESS_TOKEN
Host: westus.stt.speech.microsoft.com
Content-type: application/ssml+xml
Content-Length: 199
Connection: Keep-Alive

// Message body here...
```
