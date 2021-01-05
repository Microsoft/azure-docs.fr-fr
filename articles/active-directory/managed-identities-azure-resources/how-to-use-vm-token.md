---
title: Utiliser des identités managées sur une machine virtuelle afin d’acquérir un jeton d’accès – Azure AD
description: Instructions et exemples détaillés concernant l’utilisation d’identités managées pour ressources Azure sur une machine virtuelle afin d’acquérir un jeton d’accès OAuth.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/03/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: bed64df921326ad4d219f934f7a7bc6860bfc7d8
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861899"
---
# <a name="how-to-use-managed-identities-for-azure-resources-on-an-azure-vm-to-acquire-an-access-token"></a>Guide pratique de l’utilisation d’identités managées pour ressources Azure sur une machine virtuelle Azure afin d’acquérir un jeton d’accès 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]  

Les identités managées pour ressources Azure fournissent des services Azure avec une identité managée automatiquement dans Azure Active Directory. Vous pouvez utiliser cette identité pour vous authentifier sur n’importe quel service prenant en charge l’authentification Azure AD, sans avoir d’informations d’identification dans votre code. 

Cet article fournit divers exemples de code et de script pour l’acquisition de jeton, ainsi que des conseils sur les rubriques importantes telles que la gestion des erreurs HTTP et des expirations de jeton. 

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

Si vous envisagez d’utiliser les exemples de Azure PowerShell dans cet article, veillez à installer la dernière version de [Azure PowerShell](/powershell/azure/install-az-ps).


> [!IMPORTANT]
> - Tous les exemples de code et de scripts présentés dans cet article partent du principe que le client s’exécute sur une machine virtuelle avec des identités managées pour ressources Azure. Utilisez la fonctionnalité « Se connecter » de la machine virtuelle dans le portail Azure, pour vous connecter à distance à votre machine virtuelle. Pour plus d’informations sur l’activation d’identités managées pour ressources Azure sur une machine virtuelle, voir [Configurer des identités managées pour ressources Azure sur une machine virtuelle en utilisant le portail Azure](qs-configure-portal-windows-vm.md), ou l’une des variantes de cet article (en utilisant PowerShell, CLI, un modèle ou un Kit de développement logiciel (SDK) Azure). 

> [!IMPORTANT]
> - La limite de sécurité des identités managées pour ressources Azure est la ressource sur laquelle elles sont utilisées. Tous les scripts et codes exécutés sur une machine virtuelle peuvent demander et récupérer des jetons pour les identités disponibles sur celle-ci. 

## <a name="overview"></a>Vue d’ensemble

Une application cliente peut demander un [jeton d’accès pour l’application uniquement](../develop/developer-glossary.md#access-token) des identités managées pour ressources Azure afin d’accéder à une ressource donnée. Le jeton est [basé sur le principal du service des identités managées pour ressources Azure](overview.md#managed-identity-types). Par conséquent, il n’est pas nécessaire que le client s’inscrive pour obtenir un jeton d’accès sous son propre principal du service. Le jeton peut être utilisé comme un jeton du porteur dans [les appels de service à service nécessitant des informations d’identification du client](../develop/v2-oauth2-client-creds-grant-flow.md).

| Lien | Description |
| -------------- | -------------------- |
| [Obtenir un jeton par HTTP](#get-a-token-using-http) | Détails du protocole pour le point de terminaison de jeton d’identités managées pour ressources Azure |
| [Obtenir un jeton à l’aide de la bibliothèque Microsoft.Azure.Services.AppAuthentication pour .NET](#get-a-token-using-the-microsoftazureservicesappauthentication-library-for-net) | Exemple d’utilisation de la bibliothèque Microsoft.Azure.Services.AppAuthentication à partir d’un client .NET
| [Obtenir un jeton par C#](#get-a-token-using-c) | Exemple d’utilisation d’un point de terminaison REST d’identités managées pour ressources Azure à partir d’un client C# |
| [Obtenir un jeton par le biais de Java](#get-a-token-using-java) | Exemple d’utilisation d’un point de terminaison REST d’identités managées pour ressources Azure à partir d’un client Java |
| [Obtenir un jeton par Go](#get-a-token-using-go) | Exemple d’utilisation d’un point de terminaison REST d’identités managées pour ressources Azure à partir d’un client Go |
| [Obtenir un jeton par Azure PowerShell](#get-a-token-using-azure-powershell) | Exemple d’utilisation d’un point de terminaison REST d’identités managées pour ressources Azure à partir d’un client PowerShell |
| [Obtenir un jeton par CURL](#get-a-token-using-curl) | Exemple d’utilisation d’un point de terminaison REST d’identités managées pour ressources Azure à partir d’un client Bash/CURL |
| Gestion de la mise en cache des jetons | Conseils pour la gestion des jetons d’accès expirés |
| [Gestion des erreurs](#error-handling) | Conseils pour la gestion des erreurs HTTP retournées par le point de terminaison de jeton d’identités managées pour ressources Azure |
| [ID de ressource pour les services Azure](#resource-ids-for-azure-services) | Où obtenir les ID de ressource pour les services Azure pris en charge |

## <a name="get-a-token-using-http"></a>Obtenir un jeton par HTTP 

L’interface fondamentale pour l’acquisition d’un jeton d’accès est basée sur REST, le rendant accessible à toute application cliente en cours d’exécution sur la machine virtuelle et pouvant effectuer des appels REST par HTTP. Cela est similaire au modèle de programmation Azure AD, sauf si le client utilise un point de terminaison sur la machine virtuelle (à la place d’un point de terminaison Azure AD).

Exemple de requête utilisant le point de terminaison IMDS (Instance Metadata Service) Azure *(recommandé)*  :

```
GET 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/' HTTP/1.1 Metadata: true
```

| Élément | Description |
| ------- | ----------- |
| `GET` | Le verbe HTTP, indiquant votre souhait de récupérer des données du point de terminaison. Dans ce cas, un jeton d’accès OAuth. | 
| `http://169.254.169.254/metadata/identity/oauth2/token` | Le point de terminaison d’identités managées pour ressources Azure pour Instance Metadata Service. |
| `api-version`  | Un paramètre de chaîne de requête qui indique la version d’API pour le point de terminaison IMDS. Veuillez utiliser la version d’API `2018-02-01` ou une version ultérieure. |
| `resource` | Un paramètre de chaîne de requête, indiquant l’URI ID d’application de la ressource cible. Il apparaît également dans la revendication `aud` (audience) du jeton émis. Cet exemple demande un jeton pour accéder à Azure Resource Manager, qui possède un URI ID d’application, `https://management.azure.com/`. |
| `Metadata` | Un champ d’en-tête de requête HTTP, requis par les identités managées pour ressources Azure afin d’atténuer une attaque par falsification de requête côté serveur (SSRF). Cette valeur doit être définie sur « true », en minuscules. |
| `object_id` | (Facultatif) Un paramètre de chaîne de requête, indiquant l’élément object_id de l’identité managée pour laquelle vous souhaitez obtenir le jeton. Obligatoire, si votre machine virtuelle possède plusieurs identités managées affectées par l’utilisateur.|
| `client_id` | (Facultatif) Un paramètre de chaîne de requête, indiquant l’élément client_id de l’identité managée pour laquelle vous souhaitez obtenir le jeton. Obligatoire, si votre machine virtuelle possède plusieurs identités managées affectées par l’utilisateur.|
| `mi_res_id` | (Facultatif) Un paramètre de chaîne de requête, indiquant l’élément mi_res_id (ID de la ressource Azure) de l’identité managée pour laquelle vous souhaitez obtenir le jeton. Obligatoire, si votre machine virtuelle possède plusieurs identités managées affectées par l’utilisateur. |

Exemple de requête utilisant le point de terminaison d’extension de machine virtuelle d’identités managées pour ressources Azure *(dont l’abandon est prévu en janvier 2019)* :

```http
GET http://localhost:50342/oauth2/token?resource=https%3A%2F%2Fmanagement.azure.com%2F HTTP/1.1
Metadata: true
```

| Élément | Description |
| ------- | ----------- |
| `GET` | Le verbe HTTP, indiquant votre souhait de récupérer des données du point de terminaison. Dans ce cas, un jeton d’accès OAuth. | 
| `http://localhost:50342/oauth2/token` | Le point de terminaison d’identités managées pour ressources Azure, où 50342 est le port par défaut configurable. |
| `resource` | Un paramètre de chaîne de requête, indiquant l’URI ID d’application de la ressource cible. Il apparaît également dans la revendication `aud` (audience) du jeton émis. Cet exemple demande un jeton pour accéder à Azure Resource Manager, qui possède un URI ID d’application, `https://management.azure.com/`. |
| `Metadata` | Un champ d’en-tête de requête HTTP, requis par les identités managées pour ressources Azure afin d’atténuer une attaque par falsification de requête côté serveur (SSRF). Cette valeur doit être définie sur « true », en minuscules.|
| `object_id` | (Facultatif) Un paramètre de chaîne de requête, indiquant l’élément object_id de l’identité managée pour laquelle vous souhaitez obtenir le jeton. Obligatoire, si votre machine virtuelle possède plusieurs identités managées affectées par l’utilisateur.|
| `client_id` | (Facultatif) Un paramètre de chaîne de requête, indiquant l’élément client_id de l’identité managée pour laquelle vous souhaitez obtenir le jeton. Obligatoire, si votre machine virtuelle possède plusieurs identités managées affectées par l’utilisateur.|

Exemple de réponse :

```json
HTTP/1.1 200 OK
Content-Type: application/json
{
  "access_token": "eyJ0eXAi...",
  "refresh_token": "",
  "expires_in": "3599",
  "expires_on": "1506484173",
  "not_before": "1506480273",
  "resource": "https://management.azure.com/",
  "token_type": "Bearer"
}
```

| Élément | Description |
| ------- | ----------- |
| `access_token` | Le jeton d’accès demandé. Lors de l’appel d’une API REST sécurisée, le jeton est incorporé dans le champ d’en-tête de requête `Authorization` comme un jeton « du porteur », autorisant l’API à authentifier l’appelant. | 
| `refresh_token` | Non utilisé par les identités managées pour ressources Azure. |
| `expires_in` | Le nombre de secondes pendant lesquelles le jeton d’accès est toujours valide, avant d’expirer, à partir de son émission. L’heure d’émission est accessible dans la revendication `iat` du jeton. |
| `expires_on` | L’intervalle de temps lorsque le jeton d’accès expire. La date est exprimée en nombre de secondes à partir de « 1970-01-01T0:0:0Z UTC » (correspond à la revendication `exp` du jeton). |
| `not_before` | L’intervalle de temps pendant lequel le jeton d’accès prend effet, et peut être accepté. La date est exprimée en nombre de secondes à partir de « 1970-01-01T0:0:0Z UTC » (correspond à la revendication `nbf` du jeton). |
| `resource` | La ressource pour laquelle le jeton d’accès a été demandé, correspondant au paramètre de chaîne de requête `resource` de la requête. |
| `token_type` | Le type de jeton, qui est un jeton d’accès « du porteur », ce qui signifie que la ressource peut donner l’accès au porteur de ce jeton. |

## <a name="get-a-token-using-the-microsoftazureservicesappauthentication-library-for-net"></a>Obtenir un jeton à l’aide de la bibliothèque Microsoft.Azure.Services.AppAuthentication pour .NET

Pour les applications et fonctions .NET, la façon la plus simple d’utiliser des identités managées pour ressources Azure consiste à se servir du package Microsoft.Azure.Services.AppAuthentication. Cette bibliothèque vous permet également de tester votre code localement sur votre machine de développement, à l’aide de votre compte d’utilisateur à partir de Visual Studio, [d’Azure CLI](/cli/azure) ou de l’authentification intégrée à Active Directory. Pour plus d’informations sur les options de développement local avec cette bibliothèque, consultez le [Guide de référence technique sur Microsoft.Azure.Services.AppAuthentication](../../key-vault/general/service-to-service-authentication.md). Cette section vous montre comment prendre en main la bibliothèque dans votre code.

1. Ajoutez des références aux packages NuGet [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication) et [Microsoft.Azure.KeyVault](https://www.nuget.org/packages/Microsoft.Azure.KeyVault) à votre application.

2.  Ajoutez le code suivant à votre application :

    ```csharp
    using Microsoft.Azure.Services.AppAuthentication;
    using Microsoft.Azure.KeyVault;
    // ...
    var azureServiceTokenProvider = new AzureServiceTokenProvider();
    string accessToken = await azureServiceTokenProvider.GetAccessTokenAsync("https://management.azure.com/");
    // OR
    var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider.KeyVaultTokenCallback));
    ```
    
Pour en savoir plus sur Microsoft.Azure.Services.AppAuthentication et les opérations qu’il expose, voir le [Guide de référence technique sur Microsoft.Azure.Services.AppAuthentication](../../key-vault/general/service-to-service-authentication.md) et [l’exemple .NET d’utilisation d’App Service et de Key Vault avec des identités managées pour ressources Azure](https://github.com/Azure-Samples/app-service-msi-keyvault-dotnet).

## <a name="get-a-token-using-c"></a>Obtenir un jeton par C#

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Net;
using System.Web.Script.Serialization; 

// Build request to acquire managed identities for Azure resources token
HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/");
request.Headers["Metadata"] = "true";
request.Method = "GET";

try
{
    // Call /token endpoint
    HttpWebResponse response = (HttpWebResponse)request.GetResponse();

    // Pipe response Stream to a StreamReader, and extract access token
    StreamReader streamResponse = new StreamReader(response.GetResponseStream()); 
    string stringResponse = streamResponse.ReadToEnd();
    JavaScriptSerializer j = new JavaScriptSerializer();
    Dictionary<string, string> list = (Dictionary<string, string>) j.Deserialize(stringResponse, typeof(Dictionary<string, string>));
    string accessToken = list["access_token"];
}
catch (Exception e)
{
    string errorText = String.Format("{0} \n\n{1}", e.Message, e.InnerException != null ? e.InnerException.Message : "Acquire token failed");
}

```

## <a name="get-a-token-using-java"></a>Obtenir un jeton par le biais de Java

Utilisez cette [bibliothèque JSON](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core/2.9.4) pour récupérer un jeton à l’aide de Java.

```Java
import java.io.*;
import java.net.*;
import com.fasterxml.jackson.core.*;
 
class GetMSIToken {
    public static void main(String[] args) throws Exception {
 
        URL msiEndpoint = new URL("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/");
        HttpURLConnection con = (HttpURLConnection) msiEndpoint.openConnection();
        con.setRequestMethod("GET");
        con.setRequestProperty("Metadata", "true");
 
        if (con.getResponseCode()!=200) {
            throw new Exception("Error calling managed identity token endpoint.");
        }
 
        InputStream responseStream = con.getInputStream();
 
        JsonFactory factory = new JsonFactory();
        JsonParser parser = factory.createParser(responseStream);
 
        while(!parser.isClosed()){
            JsonToken jsonToken = parser.nextToken();
 
            if(JsonToken.FIELD_NAME.equals(jsonToken)){
                String fieldName = parser.getCurrentName();
                jsonToken = parser.nextToken();
 
                if("access_token".equals(fieldName)){
                    String accesstoken = parser.getValueAsString();
                    System.out.println("Access Token: " + accesstoken.substring(0,5)+ "..." + accesstoken.substring(accesstoken.length()-5));
                    return;
                }
            }
        }
    }
}
```

## <a name="get-a-token-using-go"></a>Obtenir un jeton par Go

```
package main

import (
  "fmt"
  "io/ioutil"
  "net/http"
  "net/url"
  "encoding/json"
)

type responseJson struct {
  AccessToken string `json:"access_token"`
  RefreshToken string `json:"refresh_token"`
  ExpiresIn string `json:"expires_in"`
  ExpiresOn string `json:"expires_on"`
  NotBefore string `json:"not_before"`
  Resource string `json:"resource"`
  TokenType string `json:"token_type"`
}

func main() {
    
    // Create HTTP request for a managed services for Azure resources token to access Azure Resource Manager
    var msi_endpoint *url.URL
    msi_endpoint, err := url.Parse("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01")
    if err != nil {
      fmt.Println("Error creating URL: ", err)
      return 
    }
    msi_parameters := url.Values{}
    msi_parameters.Add("resource", "https://management.azure.com/")
    msi_endpoint.RawQuery = msi_parameters.Encode()
    req, err := http.NewRequest("GET", msi_endpoint.String(), nil)
    if err != nil {
      fmt.Println("Error creating HTTP request: ", err)
      return 
    }
    req.Header.Add("Metadata", "true")

    // Call managed services for Azure resources token endpoint
    client := &http.Client{}
    resp, err := client.Do(req) 
    if err != nil{
      fmt.Println("Error calling token endpoint: ", err)
      return
    }

    // Pull out response body
    responseBytes,err := ioutil.ReadAll(resp.Body)
    defer resp.Body.Close()
    if err != nil {
      fmt.Println("Error reading response body : ", err)
      return
    }

    // Unmarshall response body into struct
    var r responseJson
    err = json.Unmarshal(responseBytes, &r)
    if err != nil {
      fmt.Println("Error unmarshalling the response:", err)
      return
    }

    // Print HTTP response and marshalled response body elements to console
    fmt.Println("Response status:", resp.Status)
    fmt.Println("access_token: ", r.AccessToken)
    fmt.Println("refresh_token: ", r.RefreshToken)
    fmt.Println("expires_in: ", r.ExpiresIn)
    fmt.Println("expires_on: ", r.ExpiresOn)
    fmt.Println("not_before: ", r.NotBefore)
    fmt.Println("resource: ", r.Resource)
    fmt.Println("token_type: ", r.TokenType)
}
```

## <a name="get-a-token-using-azure-powershell"></a>Obtenir un jeton par Azure PowerShell

L’exemple suivant montre comment utiliser le point de terminaison REST d’identités managées pour ressources Azure à partir d’un client PowerShell :

1. Obtenez un jeton d’accès.
2. Utilisez le jeton d’accès pour appeler une API REST de Azure Resource Manager et obtenez des informations sur la machine virtuelle. Indiquez votre ID d’abonnement, le nom de groupe de ressources et le nom de la machine virtuelle respectivement à la place de `<SUBSCRIPTION-ID>`, `<RESOURCE-GROUP>`, et `<VM-NAME>`.

```azurepowershell
Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -Headers @{Metadata="true"}
```

Exemple d’extraction du jeton d’accès de la réponse :
```azurepowershell
# Get an access token for managed identities for Azure resources
$response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' `
                              -Headers @{Metadata="true"}
$content =$response.Content | ConvertFrom-Json
$access_token = $content.access_token
echo "The managed identities for Azure resources access token is $access_token"

# Use the access token to get resource information for the VM
$vmInfoRest = (Invoke-WebRequest -Uri 'https://management.azure.com/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/<RESOURCE-GROUP>/providers/Microsoft.Compute/virtualMachines/<VM-NAME>?api-version=2017-12-01' -Method GET -ContentType "application/json" -Headers @{ Authorization ="Bearer $access_token"}).content
echo "JSON returned from call to get VM info:"
echo $vmInfoRest

```

## <a name="get-a-token-using-curl"></a>Obtenir un jeton par CURL

```bash
curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -H Metadata:true -s
```


Exemple d’extraction du jeton d’accès de la réponse :

```bash
response=$(curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -H Metadata:true -s)
access_token=$(echo $response | python -c 'import sys, json; print (json.load(sys.stdin)["access_token"])')
echo The managed identities for Azure resources access token is $access_token
```

## <a name="token-caching"></a>Mise en cache de jeton

Si le sous-système d’identités managées pour ressources Azure utilisé (extension de machine virtuelle IMDS/d’identités managées pour ressources Azure) met en cache des jetons, nous vous recommandons également d’implémenter la mise en cache des jetons dans votre code. Vous devez donc vous préparer pour les scénarios dans lesquels la ressource indique que le jeton a expiré. 

Les appels sur le câble à Azure AD réussissent uniquement quand :
- une absence de mise en cache se produit en raison de l’absence de jeton dans le cache du sous-système d’identités managées pour ressources Azure
- Le jeton mis en cache a expiré.

## <a name="error-handling"></a>Gestion des erreurs

Le point de terminaison d’identités managées pour ressources Azure signale des erreurs par le biais du champ de code d’état de l’en-tête du message de la réponse HTTP en tant qu’erreurs 4xx ou 5xx :

| Code d’état | Motif de l’erreur | Procédure de gestion |
| ----------- | ------------ | ------------- |
| 404 Introuvable. | Le point de terminaison IMDS est en cours de mise à jour. | Réessayer avec interruption exponentielle. Consultez les conseils ci-dessous. |
| 429 Trop de requêtes. |  Limite IMDS atteinte. | Réessayer avec interruption exponentielle. Consultez les conseils ci-dessous. |
| Erreur 4xx dans la requête. | Un ou plusieurs des paramètres de la requête étaient incorrects. | Ne faites pas de nouvel essai.  Examinez les détails de l’erreur pour plus d’informations.  Les erreurs 4xx sont des erreurs au moment de la conception.|
| Erreur 5xx temporaire de service. | Le sous-système d’identités managées pour ressources Azure ou Azure Active Directory ont retourné une erreur temporaire. | Il est possible de faire une nouvelle tentative après un délai d’une seconde.  Si vous réessayez trop souvent ou trop rapidement, IMDS et/ou Azure AD peut renvoyer une erreur de limite de débit (429).|
| délai d'expiration | Le point de terminaison IMDS est en cours de mise à jour. | Réessayer avec interruption exponentielle. Consultez les conseils ci-dessous. |

Si une erreur se produit, le corps de réponse HTTP correspondant contient des données JSON avec les détails de l’erreur :

| Élément | Description |
| ------- | ----------- |
| error   | Identificateur de l’erreur. |
| error_description | Description détaillée de l’erreur. **Les descriptions des erreurs peuvent changer à tout moment. N’écrivez pas de codes créant des branches en fonction des valeurs dans la description de l’erreur.**|

### <a name="http-response-reference"></a>Référence de réponse HTTP

Cette section documente les réponses possibles aux erreurs. Un état « 200 OK » est une réponse correcte, et le jeton d’accès est contenu au sein du corps de réponse JSON, dans l’élément access_token.

| Code d’état | Error | Description de l’erreur | Solution |
| ----------- | ----- | ----------------- | -------- |
| 400 Demande incorrecte | invalid_resource | AADSTS50001 : L’application nommée *\<URI\>* est introuvable dans le locataire nommé *\<TENANT-ID\>* . Cela peut se produire si l’application n’a pas été installée par l’administrateur du locataire ni acceptée par un utilisateur dans le locataire. Vous avez peut-être envoyé votre requête d’authentification au locataire incorrect.\ | (Linux uniquement) |
| 400 Demande incorrecte | bad_request_102 | En-tête de métadonnées requis non spécifié | Le champ d’en-tête de métadonnées `Metadata` est absent de votre requête, ou bien il n’est pas correctement formaté. La valeur spécifiée doit être `true`, en minuscules. Un « Exemple de requête » est disponible à la section REST précédente.|
| 401 Non autorisé | unknown_source | Source inconnue *\<URI\>* | Vérifiez que votre URI de requête HTTP GET est correctement mise en forme. La partie `scheme:host/resource-path` doit être spécifiée comme `http://localhost:50342/oauth2/token`. Un « Exemple de requête » est disponible à la section REST précédente.|
|           | invalid_request | Il manque un paramètre nécessaire à la requête, elle comprend une valeur de paramètre non valide, plus d’un paramètre à la fois, ou bien elle est incorrecte. |  |
|           | unauthorized_client | Le client n’est pas autorisé à demander un jeton d’accès avec cette méthode. | Occasionné par une demande n’ayant pas utilisé de bouclage local pour appeler l’extension, ou sur une machine virtuelle dont les identités managées pour ressources Azure ne sont pas correctement configurées. Si vous avez besoin d’aide pour configurer une machine virtuelle, voir [Configurer des identités managées pour ressources Azure sur une machine virtuelle en utilisant le portail Azure](qs-configure-portal-windows-vm.md). |
|           | access_denied | Le propriétaire de la ressource ou le serveur d’autorisation a refusé la requête. |  |
|           | unsupported_response_type | Le serveur d’autorisation ne prend pas en charge l’obtention d’un jeton d’accès par cette méthode. |  |
|           | invalid_scope | L’étendue demandée est incorrecte, inconnue ou non valide. |  |
| Erreur interne 500 du serveur | unknown | Impossible de récupérer le jeton depuis Active Directory. Pour plus d’informations, consultez les journaux d’activité dans *\<file path\>* | Vérifiez que les identités managées pour ressources Azure ont été activées sur la machine virtuelle. Si vous avez besoin d’aide pour configurer une machine virtuelle, voir [Configurer des identités managées pour ressources Azure sur une machine virtuelle en utilisant le portail Azure](qs-configure-portal-windows-vm.md).<br><br>Vérifiez également que votre URI de requête HTTP GET est correctement mise en forme, en particulier l’URI de la ressource spécifiée dans la chaîne de requête. Un « Exemple de requête » est disponible à la section REST précédente. Pour obtenir la liste des services et leur ID de ressource respectif, consultez [Services Azure prenant en charge l'authentification Azure AD](./services-support-managed-identities.md).

## <a name="retry-guidance"></a>Conseils sur les nouvelles tentatives 

Il est recommandé d’effectuer une nouvelle tentative si vous recevez un code d’erreur 404, 429 ou 5xx (consultez la section [Gestion des erreurs](#error-handling) ci-dessus).

La limitation s’applique au nombre d’appels effectués au point de terminaison IMDS. Quand le seuil de limitation est dépassé, le point de terminaison IMDS limite toutes les autres requêtes pendant que la limitation est appliquée. Pendant cette période, le point de terminaison IMDS retourne le code d’état HTTP 429 (« Trop de requêtes ») et les requêtes échouent. 

Pour une nouvelle tentative, nous vous recommandons la stratégie suivante : 

| **Stratégie de nouvelle tentative** | **Paramètres** | **Valeurs** | **Fonctionnement** |
| --- | --- | --- | --- |
|ExponentialBackoff |Nombre de tentatives<br />Temporisation min<br />Temporisation max<br />Temporisation delta<br />Première nouvelle tentative rapide |5<br />0 seconde<br />60 secondes<br />2 secondes<br />false |Tentative 1 - délai 0 s<br />Tentative 2 - délai ~2 s<br />Tentative 3 - délai ~6 s<br />Tentative 4 - délai ~14 s<br />Tentative 5 - délai ~30 s |

## <a name="resource-ids-for-azure-services"></a>ID de ressource pour les services Azure

Pour obtenir la liste des ressources qui prennent en charge Azure AD et qui ont été testées avec des identités managées pour ressources Azure et leurs ID de ressource respectifs, voir [Services Azure prenant en charge Azure AD Authentication](./services-support-managed-identities.md).


## <a name="next-steps"></a>Étapes suivantes

- Pour activer les identités managées pour ressources Azure sur une machine virtuelle Azure, voir [Configurer des identités managées pour ressources Azure sur une machine virtuelle en utilisant le portail Azure](qs-configure-portal-windows-vm.md).
