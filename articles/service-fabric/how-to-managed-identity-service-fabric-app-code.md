---
title: Utiliser une identité gérée avec une application
description: Comment utiliser des identités managées dans du code d’application Azure Service Fabric pour accéder aux services Azure.
ms.topic: article
ms.date: 10/09/2019
ms.openlocfilehash: 07f960c01367ab42a434a8c2e1e276d9c5f7bd11
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86253641"
---
# <a name="how-to-leverage-a-service-fabric-applications-managed-identity-to-access-azure-services"></a>Tirer parti de l’identité managée d’une application Service Fabric pour accéder aux services Azure

Les applications Service Fabric peuvent tirer parti des identités managées pour accéder à d’autres ressources Azure qui prennent en charge l’authentification basée sur Azure Active Directory. Une application peut obtenir un [jeton d’accès](../active-directory/develop/developer-glossary.md#access-token) représentant son identité, qui peut être affectée par le système ou affectée par l’utilisateur, et l’utiliser en tant que jeton de « porteur » pour s’authentifier auprès d'un autre service (également appelé [serveur de ressources protégé](../active-directory/develop/developer-glossary.md#resource-server)). Le jeton représente l’identité assignée à l’application Service Fabric et sera uniquement délivré aux ressources Azure (y compris les applications DF) qui partagent cette identité. Reportez-vous à la [documentation de présentation des identités managées](../active-directory/managed-identities-azure-resources/overview.md) pour obtenir une description détaillée des identités managées, ainsi que la distinction entre les identités affectées par le système et celles affectées par l’utilisateur. Nous allons faire référence à une application avec identité managée Service Fabric comme [application cliente](../active-directory/develop/developer-glossary.md#client-application) dans cet article.

> [!IMPORTANT]
> Une identité managée représente l’association entre une ressource Azure et un principal de service dans le client Azure AD correspondant associé à l’abonnement contenant la ressource. Ainsi, dans le contexte de Service Fabric, les identités managées sont uniquement prises en charge pour les applications déployées en tant que ressources Azure. 

> [!IMPORTANT]
> Avant d’utiliser l’identité managée d’une application Service Fabric, l’accès à la ressource protégée doit être accordé à l’application cliente. Reportez-vous à la liste des [services Azure qui prennent en charge l’authentification Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-managed-identities-for-azure-resources) pour vérifier la prise en charge, puis à la documentation du service respectif pour les étapes spécifiques afin d’accorder un accès d’identité aux ressources intéressantes. 

## <a name="acquiring-an-access-token-using-rest-api"></a>Acquisition d’un jeton d’accès à l’aide de l’API REST
Dans les clusters activés pour l’identité managée, le runtime Service Fabric expose un point de terminaison localhost que les applications peuvent utiliser pour obtenir des jetons d’accès. Le point de terminaison est disponible sur chaque nœud du cluster et est accessible à toutes les entités sur ce nœud. Les appelants autorisés peuvent obtenir des jetons d’accès en appelant ce point de terminaison et en présentant un code d’authentification. Le code est généré par le runtime Service Fabric pour chaque activation de package de code de service distincte, et est lié à la durée de vie du processus hébergeant ce package de code de service.

Plus précisément, l’environnement d’un service Service Fabric avec identité managée est amorcé avec les variables suivantes :
- IDENTITY_ENDPOINT : point de terminaison localhost correspondant à l’identité managée du service
- IDENTITY_HEADER : code d’authentification unique représentant le service sur le nœud actuel
- IDENTITY_SERVER_THUMBPRINT : empreinte du serveur d’identités managées Service Fabric

> [!IMPORTANT]
> Le code d’application doit considérer la valeur de la variable d’environnement « IDENTITY_HEADER » comme des données sensibles. Il ne doit pas être consigné ou diffusé. Le code d’authentification n’a pas de valeur en dehors du nœud local, ou une fois que le processus qui héberge le service s’est arrêté, mais il représente l’identité du service Service Fabric et doit donc être traité avec les mêmes précautions que le jeton d’accès lui-même.

Pour obtenir un jeton, le client doit :
- former un URI en concaténant le point de terminaison de l’identité managée (valeur IDENTITY_ENDPOINT) avec la version de l’API et la ressource (audience) requise pour le jeton
- créer une requête HTTP(S) GET pour l’URI spécifié
- ajouter la logique de validation de certificat de serveur appropriée
- ajouter le code d’authentification (valeur IDENTITY_HEADER) en tant qu’en-tête à la demande
- soumettre la demande.

Une réponse correcte contient une charge utile JSON représentant le jeton d’accès obtenu, ainsi que des métadonnées qui le décrivent. Une réponse qui a échoué inclut également une explication de l’échec. Pour plus d’informations sur la gestion des erreurs, voir ci-dessous.

Les jetons d’accès sont mis en cache par Service Fabric à différents niveaux (nœud, cluster, service de fournisseur de ressources). Par conséquent, une réponse réussie n’implique pas nécessairement que le jeton a été émis directement en réponse à la demande de l’application utilisateur. Les jetons seront mis en cache pendant une durée inférieure à leur durée de vie. Une application est donc garantie de recevoir un jeton valide. Il est recommandé que le code de l’application mette en cache les jetons d’accès qu’il obtient ; la clé de mise en cache doit inclure (une dérivation de) l’audience. 

Exemple de demande :
```http
GET 'https://localhost:2377/metadata/identity/oauth2/token?api-version=2019-07-01-preview&resource=https://vault.azure.net/' HTTP/1.1 Secret: 912e4af7-77ba-4fa5-a737-56c8e3ace132
```
où :

| Élément | Description |
| ------- | ----------- |
| `GET` | Le verbe HTTP, indiquant votre souhait de récupérer des données du point de terminaison. Dans ce cas, un jeton d’accès OAuth. | 
| `https://localhost:2377/metadata/identity/oauth2/token` | Le point de terminaison de l’identité managée pour les applications Service Fabric, fourni via la variable d’environnement IDENTITY_ENDPOINT. |
| `api-version` | Un paramètre de chaîne de requête, spécifiant la version de l’API du service de jeton d’identité managée ; actuellement, la seule valeur acceptée est `2019-07-01-preview`, et peut faire l’objet de modifications. |
| `resource` | Un paramètre de chaîne de requête, indiquant l’URI ID d’application de la ressource cible. Il s’agit de la revendication `aud` (audience) du jeton émis. Cet exemple demande un jeton pour accéder à Azure Key Vault, dont l’URI ID d’application correspond à https:\//vault.azure.net/. |
| `Secret` | Un champ d’en-tête de demande HTTP, requis par le service de jeton d’identité managée Service Fabric pour que les services Service Fabric authentifient l’appelant. Cette valeur est fournie par le runtime SF via la variable d’environnement IDENTITY_HEADER. |


Exemple de réponse :
```json
HTTP/1.1 200 OK
Content-Type: application/json
{
    "token_type":  "Bearer",
    "access_token":  "eyJ0eXAiO...",
    "expires_on":  1565244611,
    "resource":  "https://vault.azure.net/"
}
```
où :

| Élément | Description |
| ------- | ----------- |
| `token_type` | Le type de jeton ; dans ce cas, un jeton d’accès de « porteur », ce qui signifie que le présentateur (« porteur ») de ce jeton est le sujet prévu du jeton. |
| `access_token` | Le jeton d’accès demandé. Lors de l’appel d’une API REST sécurisée, le jeton est incorporé dans le champ d’en-tête de requête `Authorization` comme un jeton « du porteur », autorisant l’API à authentifier l’appelant. | 
| `expires_on` | Le timestamp de l’expiration du jeton d’accès ; exprimé en nombre de secondes à partir de « 1970-01-01T0:0:0Z UTC » et correspond à la revendication `exp` du jeton. Dans ce cas, le jeton expire le 2019-08-08T06:10:11 + 00:00 (dans RFC 3339)|
| `resource` | La ressource pour laquelle le jeton d’accès a été émis, spécifiée via le paramètre de chaîne de requête `resource` de la demande, correspond à la revendication « aud » du jeton. |


## <a name="acquiring-an-access-token-using-c"></a>Acquisition d’un jeton d’accès à l’aide de C#
En C#, les éléments ci-dessus deviennent :

```C#
namespace Azure.ServiceFabric.ManagedIdentity.Samples
{
    using System;
    using System.Net.Http;
    using System.Text;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Web;
    using Newtonsoft.Json;

    /// <summary>
    /// Type representing the response of the SF Managed Identity endpoint for token acquisition requests.
    /// </summary>
    [JsonObject]
    public sealed class ManagedIdentityTokenResponse
    {
        [JsonProperty(Required = Required.Always, PropertyName = "token_type")]
        public string TokenType { get; set; }

        [JsonProperty(Required = Required.Always, PropertyName = "access_token")]
        public string AccessToken { get; set; }

        [JsonProperty(PropertyName = "expires_on")]
        public string ExpiresOn { get; set; }

        [JsonProperty(PropertyName = "resource")]
        public string Resource { get; set; }
    }

    /// <summary>
    /// Sample class demonstrating access token acquisition using Managed Identity.
    /// </summary>
    public sealed class AccessTokenAcquirer
    {
        /// <summary>
        /// Acquire an access token.
        /// </summary>
        /// <returns>Access token</returns>
        public static async Task<string> AcquireAccessTokenAsync()
        {
            var managedIdentityEndpoint = Environment.GetEnvironmentVariable("IDENTITY_ENDPOINT");
            var managedIdentityAuthenticationCode = Environment.GetEnvironmentVariable("IDENTITY_HEADER");
            var managedIdentityServerThumbprint = Environment.GetEnvironmentVariable("IDENTITY_SERVER_THUMBPRINT");
            // Latest api version, 2019-07-01-preview is still supported.
            var managedIdentityApiVersion = Environment.GetEnvironmentVariable("IDENTITY_API_VERSION");
            var managedIdentityAuthenticationHeader = "secret";
            var resource = "https://management.azure.com/";

            var requestUri = $"{managedIdentityEndpoint}?api-version={managedIdentityApiVersion}&resource={HttpUtility.UrlEncode(resource)}";

            var requestMessage = new HttpRequestMessage(HttpMethod.Get, requestUri);
            requestMessage.Headers.Add(managedIdentityAuthenticationHeader, managedIdentityAuthenticationCode);
            
            var handler = new HttpClientHandler();
            handler.ServerCertificateCustomValidationCallback = (httpRequestMessage, cert, certChain, policyErrors) =>
            {
                // Do any additional validation here
                if (policyErrors == SslPolicyErrors.None)
                {
                    return true;
                }
                return 0 == string.Compare(cert.GetCertHashString(), managedIdentityServerThumbprint, StringComparison.OrdinalIgnoreCase);
            };

            try
            {
                var response = await new HttpClient(handler).SendAsync(requestMessage)
                    .ConfigureAwait(false);

                response.EnsureSuccessStatusCode();

                var tokenResponseString = await response.Content.ReadAsStringAsync()
                    .ConfigureAwait(false);

                var tokenResponseObject = JsonConvert.DeserializeObject<ManagedIdentityTokenResponse>(tokenResponseString);

                return tokenResponseObject.AccessToken;
            }
            catch (Exception ex)
            {
                string errorText = String.Format("{0} \n\n{1}", ex.Message, ex.InnerException != null ? ex.InnerException.Message : "Acquire token failed");

                Console.WriteLine(errorText);
            }

            return String.Empty;
        }
    } // class AccessTokenAcquirer
} // namespace Azure.ServiceFabric.ManagedIdentity.Samples
```
## <a name="accessing-key-vault-from-a-service-fabric-application-using-managed-identity"></a>Accès à Key Vault à partir d’une application Service Fabric à l’aide d’une identité managée
Cet exemple s’appuie sur les éléments ci-dessus pour illustrer l’accès à un secret stocké dans un coffre de clés à l’aide d’une identité managée.

```C#
        /// <summary>
        /// Probe the specified secret, displaying metadata on success.  
        /// </summary>
        /// <param name="vault">vault name</param>
        /// <param name="secret">secret name</param>
        /// <param name="version">secret version id</param>
        /// <returns></returns>
        public async Task<string> ProbeSecretAsync(string vault, string secret, string version)
        {
            // initialize a KeyVault client with a managed identity-based authentication callback
            var kvClient = new Microsoft.Azure.KeyVault.KeyVaultClient(new Microsoft.Azure.KeyVault.KeyVaultClient.AuthenticationCallback((a, r, s) => { return AuthenticationCallbackAsync(a, r, s); }));

            Log(LogLevel.Info, $"\nRunning with configuration: \n\tobserved vault: {config.VaultName}\n\tobserved secret: {config.SecretName}\n\tMI endpoint: {config.ManagedIdentityEndpoint}\n\tMI auth code: {config.ManagedIdentityAuthenticationCode}\n\tMI auth header: {config.ManagedIdentityAuthenticationHeader}");
            string response = String.Empty;

            Log(LogLevel.Info, "\n== {DateTime.UtcNow.ToString()}: Probing secret...");
            try
            {
                var secretResponse = await kvClient.GetSecretWithHttpMessagesAsync(vault, secret, version)
                    .ConfigureAwait(false);

                if (secretResponse.Response.IsSuccessStatusCode)
                {
                    // use the secret: secretValue.Body.Value;
                    response = String.Format($"Successfully probed secret '{secret}' in vault '{vault}': {PrintSecretBundleMetadata(secretResponse.Body)}");
                }
                else
                {
                    response = String.Format($"Non-critical error encountered retrieving secret '{secret}' in vault '{vault}': {secretResponse.Response.ReasonPhrase} ({secretResponse.Response.StatusCode})");
                }
            }
            catch (Microsoft.Rest.ValidationException ve)
            {
                response = String.Format($"encountered REST validation exception 0x{ve.HResult.ToString("X")} trying to access '{secret}' in vault '{vault}' from {ve.Source}: {ve.Message}");
            }
            catch (KeyVaultErrorException kvee)
            {
                response = String.Format($"encountered KeyVault exception 0x{kvee.HResult.ToString("X")} trying to access '{secret}' in vault '{vault}': {kvee.Response.ReasonPhrase} ({kvee.Response.StatusCode})");
            }
            catch (Exception ex)
            {
                // handle generic errors here
                response = String.Format($"encountered exception 0x{ex.HResult.ToString("X")} trying to access '{secret}' in vault '{vault}': {ex.Message}");
            }

            Log(LogLevel.Info, response);

            return response;
        }

        /// <summary>
        /// KV authentication callback, using the application's managed identity.
        /// </summary>
        /// <param name="authority">The expected issuer of the access token, from the KV authorization challenge.</param>
        /// <param name="resource">The expected audience of the access token, from the KV authorization challenge.</param>
        /// <param name="scope">The expected scope of the access token; not currently used.</param>
        /// <returns>Access token</returns>
        public async Task<string> AuthenticationCallbackAsync(string authority, string resource, string scope)
        {
            Log(LogLevel.Verbose, $"authentication callback invoked with: auth: {authority}, resource: {resource}, scope: {scope}");
            var encodedResource = HttpUtility.UrlEncode(resource);

            // This sample does not illustrate the caching of the access token, which the user application is expected to do.
            // For a given service, the caching key should be the (encoded) resource uri. The token should be cached for a period
            // of time at most equal to its remaining validity. The 'expires_on' field of the token response object represents
            // the number of seconds from Unix time when the token will expire. You may cache the token if it will be valid for at
            // least another short interval (1-10s). If its expiration will occur shortly, don't cache but still return it to the 
            // caller. The MI endpoint will not return an expired token.
            // Sample caching code:
            //
            // ManagedIdentityTokenResponse tokenResponse;
            // if (responseCache.TryGetCachedItem(encodedResource, out tokenResponse))
            // {
            //     Log(LogLevel.Verbose, $"cache hit for key '{encodedResource}'");
            //
            //     return tokenResponse.AccessToken;
            // }
            //
            // Log(LogLevel.Verbose, $"cache miss for key '{encodedResource}'");
            //
            // where the response cache is left as an exercise for the reader. MemoryCache is a good option, albeit not yet available on .net core.

            var requestUri = $"{config.ManagedIdentityEndpoint}?api-version={config.ManagedIdentityApiVersion}&resource={encodedResource}";
            Log(LogLevel.Verbose, $"request uri: {requestUri}");

            var requestMessage = new HttpRequestMessage(HttpMethod.Get, requestUri);
            requestMessage.Headers.Add(config.ManagedIdentityAuthenticationHeader, config.ManagedIdentityAuthenticationCode);
            Log(LogLevel.Verbose, $"added header '{config.ManagedIdentityAuthenticationHeader}': '{config.ManagedIdentityAuthenticationCode}'");

            var response = await httpClient.SendAsync(requestMessage)
                .ConfigureAwait(false);
            Log(LogLevel.Verbose, $"response status: success: {response.IsSuccessStatusCode}, status: {response.StatusCode}");

            response.EnsureSuccessStatusCode();

            var tokenResponseString = await response.Content.ReadAsStringAsync()
                .ConfigureAwait(false);

            var tokenResponse = JsonConvert.DeserializeObject<ManagedIdentityTokenResponse>(tokenResponseString);
            Log(LogLevel.Verbose, "deserialized token response; returning access code..");

            // Sample caching code (continuation):
            // var expiration = DateTimeOffset.FromUnixTimeSeconds(Int32.Parse(tokenResponse.ExpiresOn));
            // if (expiration > DateTimeOffset.UtcNow.AddSeconds(5.0))
            //    responseCache.AddOrUpdate(encodedResource, tokenResponse, expiration);

            return tokenResponse.AccessToken;
        }

        private string PrintSecretBundleMetadata(SecretBundle bundle)
        {
            StringBuilder strBuilder = new StringBuilder();

            strBuilder.AppendFormat($"\n\tid: {bundle.Id}\n");
            strBuilder.AppendFormat($"\tcontent type: {bundle.ContentType}\n");
            strBuilder.AppendFormat($"\tmanaged: {bundle.Managed}\n");
            strBuilder.AppendFormat($"\tattributes:\n");
            strBuilder.AppendFormat($"\t\tenabled: {bundle.Attributes.Enabled}\n");
            strBuilder.AppendFormat($"\t\tnbf: {bundle.Attributes.NotBefore}\n");
            strBuilder.AppendFormat($"\t\texp: {bundle.Attributes.Expires}\n");
            strBuilder.AppendFormat($"\t\tcreated: {bundle.Attributes.Created}\n");
            strBuilder.AppendFormat($"\t\tupdated: {bundle.Attributes.Updated}\n");
            strBuilder.AppendFormat($"\t\trecoveryLevel: {bundle.Attributes.RecoveryLevel}\n");

            return strBuilder.ToString();
        }

        private enum LogLevel
        {
            Info,
            Verbose
        };

        private void Log(LogLevel level, string message)
        {
            if (level != LogLevel.Verbose
                || config.DoVerboseLogging)
            {
                Console.WriteLine(message);
            }
        }

```

## <a name="error-handling"></a>Gestion des erreurs
Le champ « Code d’état » de l’en-tête de réponse HTTP indique l’état de réussite de la demande ; un état « 200 OK » indique une réussite, et la réponse inclut le jeton d’accès comme décrit ci-dessus. Voici une énumération concise des réponses possibles aux erreurs.

| Code d’état | Motif de l’erreur | Procédure de gestion |
| ----------- | ------------ | ------------- |
| 404 Introuvable. | Code d’authentification inconnu, ou l’application n’a pas été assignée à une identité managée. | Corrigez le code d’installation de l’application ou d’acquisition de jeton. |
| 429 Trop de requêtes. |  Limitation atteinte, imposée par AAD ou SF. | Réessayer avec interruption exponentielle. Consultez les conseils ci-dessous. |
| Erreur 4xx dans la requête. | Un ou plusieurs des paramètres de la requête étaient incorrects. | Ne faites pas de nouvel essai.  Examinez les détails de l’erreur pour plus d’informations.  Les erreurs 4xx sont des erreurs au moment de la conception.|
| Erreur 5xx du service. | Le sous-système d’identité managée ou Azure Active Directory a renvoyé une erreur temporaire. | Il est possible d’effectuer une nouvelle tentative après quelques instants. Vous pouvez atteindre une condition de limitation (429) lors de la nouvelle tentative.|

Si une erreur se produit, le corps de réponse HTTP correspondant contient un objet JSON avec les détails de l’erreur :

| Élément | Description |
| ------- | ----------- |
| code | Code d’erreur. |
| correlationId | Un ID de corrélation qui peut être utilisé pour le débogage. |
| message | Description détaillée de l’erreur. **Les descriptions des erreurs peuvent changer à tout moment. Ne vous basez pas uniquement sur le message d’erreur lui-même.**|

Exemple de l’erreur :
```json
{"error":{"correlationId":"7f30f4d3-0f3a-41e0-a417-527f21b3848f","code":"SecretHeaderNotFound","message":"Secret is not found in the request headers."}}
```

Voici une liste d’erreurs courantes de Service Fabric spécifiques aux identités managées :

| Code | Message | Description | 
| ----------- | ----- | ----------------- |
| SecretHeaderNotFound | Le secret est introuvable dans les en-têtes de demande. | Le code d’authentification n’a pas été fourni avec la demande. | 
| ManagedIdentityNotFound | L’identité managée est introuvable pour l’hôte d’application spécifié. | L’application n’a pas d’identité, ou le code d’authentification est inconnu. |
| ArgumentNullOrEmpty | Le paramètre « resource » ne doit pas avoir une valeur null ou être une chaîne vide. | La ressource (audience) n’a pas été fournie dans la demande. |
| InvalidApiVersion | La version de l’API '' n’est pas prise en charge. La version prise en charge est « 2019-07-01-preview ». | Version d’API spécifiée manquante ou non prise en charge dans l’URI de la demande. |
| InternalServerError | Une erreur est survenue. | Une erreur s’est produite dans le sous-système d’identité managée, éventuellement en dehors de la pile de Service Fabric. La cause la plus probable est une valeur incorrecte spécifiée pour la ressource (recherchez «/» de fin) | 

## <a name="retry-guidance"></a>Conseils sur les nouvelles tentatives 

En général, le seul code d’erreur renouvelable est 429 (trop de requêtes). Les codes d’erreurs de serveur internes/d’erreurs 5xx peuvent être renouvelables, même si la cause peut être permanente. 

Les limitations de bande passante s’appliquent au nombre d’appels effectués au sous-système d’identité managée, en particulier les dépendances « en amont » (le service Azure d’identité managée ou le service de jeton sécurisé). Service Fabric met en cache les jetons à différents niveaux dans le pipeline, mais étant donné la nature distribuée des composants impliqués, l’appelant peut rencontrer des réponses de limitation incohérentes (c’est-à-dire être limité sur un nœud/une instance d’une application, mais pas sur un nœud différent lors de la demande d’un jeton pour la même identité.) Lorsque la condition de limitation est définie, les demandes ultérieures de la même application peuvent échouer avec le code d’état HTTP 429 (trop de requêtes) jusqu’à ce que la condition soit effacée.  

Il est recommandé de retenter les demandes échouées pour cause de limitation à l’aide d’une interruption exponentielle, comme suit : 

| Index d’appel | Action sur la réception de 429 | 
| --- | --- | 
| 1 | Attendre 1 seconde et réessayer |
| 2 | Attendre 2 seconde et réessayer |
| 3 | Attendre 4 seconde et réessayer |
| 4 | Attendre 8 seconde et réessayer |
| 4 | Attendre 8 seconde et réessayer |
| 5 | Attendre 16 seconde et réessayer |

## <a name="resource-ids-for-azure-services"></a>ID de ressource pour les services Azure
Consultez [Services Azure prenant en charge l’authentification de Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md) pour obtenir une liste des ressources qui prennent en charge Azure AD et leur ID de ressources respectif.

## <a name="next-steps"></a>Étapes suivantes
* [Déployer une application Azure Service Fabric avec une identité managée attribuée par le système](./how-to-deploy-service-fabric-application-system-assigned-managed-identity.md)
* [Déployer une application Azure Service Fabric avec une identité managée attribuée par l’utilisateur](./how-to-deploy-service-fabric-application-user-assigned-managed-identity.md)
* [Accorder à une application Azure Service Fabric l’accès à d’autres ressources Azure](./how-to-grant-access-other-resources.md)
