---
title: Expressions de stratégie de la Gestion des API Azure | Microsoft Docs
description: Découvrez les expressions de stratégie dans la Gestion des API Azure. Consultez des exemples et affichez des ressources disponibles supplémentaires.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: ea160028-fc04-4782-aa26-4b8329df3448
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 03/22/2019
ms.author: apimpm
ms.openlocfilehash: ab83344f779f93107b59ca28348da3a66f1efc1a
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92076859"
---
# <a name="api-management-policy-expressions"></a>Expressions de stratégie de la Gestion des API
Cet article décrit la syntaxe des expressions de stratégie dans C# 7. Chaque expression a accès à la variable de [contexte](api-management-policy-expressions.md#ContextVariables) fournie implicitement et à un [sous-ensemble](api-management-policy-expressions.md#CLRTypes) autorisé de types .NET Framework.

Pour plus d'informations :

- Découvrez comment fournir des informations de contexte à votre service backend. Utilisez les stratégies [Set query string parameter](api-management-transformation-policies.md#SetQueryStringParameter) et [Set HTTP header](api-management-transformation-policies.md#SetHTTPheader) pour fournir ces informations.
- Découvrez comment utiliser la stratégie [Validate JWT](api-management-access-restriction-policies.md#ValidateJWT) pour pré-autoriser l’accès aux opérations à partir de revendications de jetons.
- Découvrez comment utiliser une trace [Inspecteur d’API](./api-management-howto-api-inspector.md) pour voir comment les stratégies sont évaluées et connaître les résultats des évaluations.
- Découvrez comment utiliser les expressions avec les stratégies [Get from cache](api-management-caching-policies.md#GetFromCache) et [Store to cache](api-management-caching-policies.md#StoreToCache) pour configurer la durée de mise en cache des réponses de Gestion des API. Définissez une durée qui correspond à la mise en cache des réponses du service principal comme le spécifie la directive `Cache-Control` du service principal.
- Découvrez comment effectuer le filtrage de contenu. Supprimez des éléments de données de la réponse reçue du service principal à l’aide des stratégies [Control flow](api-management-advanced-policies.md#choose) et [Set body](api-management-transformation-policies.md#SetBody).
- Pour télécharger les instructions de stratégie, consultez le référentiel GitHub [api-management-samples/policies](https://github.com/Azure/api-management-samples/tree/master/policies).


## <a name="syntax"></a><a name="Syntax"></a> Syntaxe
Les expressions à instruction unique sont entre `@(expression)`, où `expression` est une instruction d’expression C# bien formée.

Les expressions à instructions multiples sont entre `@{expression}`. Tous les chemins d’accès de code au sein des expressions à instructions multiples doivent se terminer par une instruction `return`.

## <a name="examples"></a><a name="PolicyExpressionsExamples"></a> Exemples

```
@(true)

@((1+1).ToString())

@("Hi There".Length)

@(Regex.Match(context.Response.Headers.GetValueOrDefault("Cache-Control",""), @"max-age=(?<maxAge>\d+)").Groups["maxAge"]?.Value)

@(context.Variables.ContainsKey("maxAge") ? int.Parse((string)context.Variables["maxAge"]) : 3600)

@{
  string[] value;
  if (context.Request.Headers.TryGetValue("Authorization", out value))
  {
      if(value != null && value.Length > 0)
      {
          return Encoding.UTF8.GetString(Convert.FromBase64String(value[0]));
      }
  }
  return null;

}
```

## <a name="usage"></a><a name="PolicyExpressionsUsage"></a>Utilisation
Les expressions peuvent être utilisées comme valeurs d’attribut ou valeurs de texte dans l’une des [stratégies](api-management-policies.md) de la Gestion des API, sauf si la référence de la stratégie le spécifie autrement.

> [!IMPORTANT]
> Lorsque vous utilisez des expressions de stratégie, elles ne font l’objet que d’une vérification limitée lors de la définition de la stratégie. Les expressions sont exécutées par la passerelle lors de l’exécution, et toutes les exceptions générées par les expressions de stratégie entraînent une erreur d’exécution.

## <a name="net-framework-types-allowed-in-policy-expressions"></a><a name="CLRTypes"></a> Types .NET Framework autorisés dans les expressions de stratégie
Le tableau suivant liste les types .NET Framework et leurs membres qui sont autorisés dans les expressions de stratégie.

|Type|Membres pris en charge|
|--------------|-----------------------|
|Newtonsoft.Json.Formatting|Tous|
|Newtonsoft.Json.JsonConvert|SerializeObject, DeserializeObject|
|Newtonsoft.Json.Linq.Extensions|Tous|
|Newtonsoft.Json.Linq.JArray|Tous|
|Newtonsoft.Json.Linq.JConstructor|Tous|
|Newtonsoft.Json.Linq.JContainer|Tous|
|Newtonsoft.Json.Linq.JObject|Tous|
|Newtonsoft.Json.Linq.JProperty|Tous|
|Newtonsoft.Json.Linq.JRaw|Tous|
|Newtonsoft.Json.Linq.JToken|Tous|
|Newtonsoft.Json.Linq.JTokenType|Tous|
|Newtonsoft.Json.Linq.JValue|Tous|
|System.Array|Tous|
|System.BitConverter|Tous|
|System.Boolean|Tous|
|System.Byte|Tous|
|System.Char|Tous|
|System.Collections.Generic.Dictionary<TKey, TValue>|Tous|
|System.Collections.Generic.HashSet\<T>|Tous|
|System.Collections.Generic.ICollection\<T>|Tous|
|System.Collections.Generic.IDictionary<TKey, TValue>|Tous|
|System.Collections.Generic.IEnumerable\<T>|Tous|
|System.Collections.Generic.IEnumerator\<T>|Tous|
|System.Collections.Generic.IList\<T>|Tous|
|System.Collections.Generic.IReadOnlyCollection\<T>|Tous|
|System.Collections.Generic.IReadOnlyDictionary<TKey, TValue>|Tous|
|System.Collections.Generic.ISet\<T>|Tous|
|System.Collections.Generic.KeyValuePair<TKey, TValue>|Tous|
|System.Collections.Generic.IList\<T>|Tous|
|System.Collections.Generic.Queue\<T>|Tous|
|System.Collections.Generic.Stack\<T>|Tous|
|System.Convert|Tous|
|System.DateTime|(Constructor), Add, AddDays, AddHours, AddMilliseconds, AddMinutes, AddMonths, AddSeconds, AddTicks, AddYears, Date, Day, DayOfWeek, DayOfYear, DaysInMonth, Hour, IsDaylightSavingTime, IsLeapYear, MaxValue, Millisecond, Minute, MinValue, Month, Now, Parse, Second, Subtract, Ticks, TimeOfDay, Today, ToString, UtcNow, Year|
|System.DateTimeKind|Utc|
|System.DateTimeOffset|Tous|
|System.Decimal|Tous|
|System.Double|Tous|
|System.Exception|Tous|
|System.Guid|Tous|
|System.Int16|Tous|
|System.Int32|Tous|
|System.Int64|Tous|
|System.IO.StringReader|Tous|
|System.IO.StringWriter|Tous|
|System.Linq.Enumerable|Tous|
|System.Math|Tous|
|System.MidpointRounding|Tous|
|System.Net.WebUtility|Tous|
|System.Nullable|Tous|
|System.Random|Tous|
|System.SByte|Tous|
|System.Security.Cryptography.AsymmetricAlgorithm|Tous|
|System.Security.Cryptography.CipherMode|Tous|
|System.Security.Cryptography.HashAlgorithm|Tous|
|System.Security.Cryptography.HashAlgorithmName|Tous|
|System.Security.Cryptography.HMAC|Tous|
|System.Security.Cryptography.HMACMD5|Tous|
|System.Security.Cryptography.HMACSHA1|Tous|
|System.Security.Cryptography.HMACSHA256|Tous|
|System.Security.Cryptography.HMACSHA384|Tous|
|System.Security.Cryptography.HMACSHA512|Tous|
|System.Security.Cryptography.KeyedHashAlgorithm|Tous|
|System.Security.Cryptography.MD5|Tous|
|System.Security.Cryptography.Oid|Tous|
|System.Security.Cryptography.PaddingMode|Tous|
|System.Security.Cryptography.RNGCryptoServiceProvider|Tous|
|System.Security.Cryptography.RSA|Tous|
|System.Security.Cryptography.RSAEncryptionPadding|Tous|
|System.Security.Cryptography.RSASignaturePadding|Tous|
|System.Security.Cryptography.SHA1|Tous|
|System.Security.Cryptography.SHA1Managed|Tous|
|System.Security.Cryptography.SHA256|Tous|
|System.Security.Cryptography.SHA256Managed|Tous|
|System.Security.Cryptography.SHA384|Tous|
|System.Security.Cryptography.SHA384Managed|Tous|
|System.Security.Cryptography.SHA512|Tous|
|System.Security.Cryptography.SHA512Managed|Tous|
|System.Security.Cryptography.SymmetricAlgorithm|Tous|
|System.Security.Cryptography.X509Certificates.PublicKey|Tous|
|System.Security.Cryptography.X509Certificates.RSACertificateExtensions|Tous|
|System.Security.Cryptography.X509Certificates.X500DistinguishedName|Nom|
|System.Security.Cryptography.X509Certificates.X509Certificate|Tous|
|System.Security.Cryptography.X509Certificates.X509Certificate2|Tous|
|System.Security.Cryptography.X509Certificates.X509ContentType|Tous|
|System.Security.Cryptography.X509Certificates.X509NameType|Tous|
|System.Single|Tous|
|System.String|Tous|
|System.StringComparer|Tous|
|System.StringComparison|Tous|
|System.StringSplitOptions|Tous|
|System.Text.Encoding|Tous|
|System.Text.RegularExpressions.Capture|Index, Length, Value|
|System.Text.RegularExpressions.CaptureCollection|Count, Item|
|System.Text.RegularExpressions.Group|Captures, Success|
|System.Text.RegularExpressions.GroupCollection|Count, Item|
|System.Text.RegularExpressions.Match|Empty, Groups, Result|
|System.Text.RegularExpressions.Regex|(Constructor), IsMatch, Match, Matches, Replace, Unescape, Split|
|System.Text.RegularExpressions.RegexOptions|Tous|
|System.Text.StringBuilder|Tous|
|System.TimeSpan|Tous|
|System.TimeZone|Tous|
|System.TimeZoneInfo.AdjustmentRule|Tous|
|System.TimeZoneInfo.TransitionTime|Tous|
|System.TimeZoneInfo|Tous|
|System.Tuple|Tous|
|System.UInt16|Tous|
|System.UInt32|Tous|
|System.UInt64|Tous|
|System.Uri|Tous|
|System.UriPartial|Tous|
|System.Xml.Linq.Extensions|Tous|
|System.Xml.Linq.XAttribute|Tous|
|System.Xml.Linq.XCData|Tous|
|System.Xml.Linq.XComment|Tous|
|System.Xml.Linq.XContainer|Tous|
|System.Xml.Linq.XDeclaration|Tous|
|System.Xml.Linq.XDocument|Toutes, sauf : Load|
|System.Xml.Linq.XDocumentType|Tous|
|System.Xml.Linq.XElement|Tous|
|System.Xml.Linq.XName|Tous|
|System.Xml.Linq.XNamespace|Tous|
|System.Xml.Linq.XNode|Tous|
|System.Xml.Linq.XNodeDocumentOrderComparer|Tous|
|System.Xml.Linq.XNodeEqualityComparer|Tous|
|System.Xml.Linq.XObject|Tous|
|System.Xml.Linq.XProcessingInstruction|Tous|
|System.Xml.Linq.XText|Tous|
|System.Xml.XmlNodeType|Tous|

## <a name="context-variable"></a><a name="ContextVariables"></a> Variable de contexte
Une variable nommée `context` est implicitement disponible dans toutes les [expressions](api-management-policy-expressions.md#Syntax) de stratégie. Ses membres fournissent des informations pertinentes pour la `\request`. Tous les membres `context` sont en lecture seule.

|Variable de contexte|Méthodes, propriétés et valeurs de paramètres autorisées|
|----------------------|-------------------------------------------------------|
|contexte|[Api](#ref-context-api) : [IApi](#ref-iapi)<br /><br /> [Déploiement](#ref-context-deployment)<br /><br /> Elapsed: TimeSpan (intervalle de temps entre la valeur Timestamp et l’heure actuelle)<br /><br /> [LastError](#ref-context-lasterror)<br /><br /> [opération](#ref-context-operation)<br /><br /> [Produit](#ref-context-product)<br /><br /> [Requête](#ref-context-request)<br /><br /> RequestId: Guid (identificateur de requête unique)<br /><br /> [Réponse](#ref-context-response)<br /><br /> [Abonnement](#ref-context-subscription)<br /><br /> Timestamp : DateTime (point dans le temps où la requête a été reçue)<br /><br /> Tracing: bool (indique si le traçage est activé ou désactivé) <br /><br /> [Utilisateur](#ref-context-user)<br /><br /> [Variables](#ref-context-variables) : IReadOnlyDictionary<string, object><br /><br /> void Trace(message: string)|
|<a id="ref-context-api"></a>context.Api|Id: string<br /><br /> IsCurrentRevision: bool<br /><br />  Name: string<br /><br /> Path: string<br /><br /> Revision: string<br /><br /> ServiceUrl: [IUrl](#ref-iurl)<br /><br /> Version: string |
|<a id="ref-context-deployment"></a>context.Deployment|Region: string<br /><br /> ServiceName: string<br /><br /> Certificats : IReadOnlyDictionary<string, X509Certificate2>|
|<a id="ref-context-lasterror"></a>context.LastError|Source: string<br /><br /> Reason: string<br /><br /> Message: string<br /><br /> Scope: string<br /><br /> Section: string<br /><br /> Path: string<br /><br /> PolicyId: string<br /><br /> Pour plus d’informations sur context.LastError, consultez la page [Gestion des erreurs](api-management-error-handling-policies.md).|
|<a id="ref-context-operation"></a>context.Operation|Id: string<br /><br /> Method: string<br /><br /> Name: string<br /><br /> UrlTemplate: string|
|<a id="ref-context-product"></a>context.Product|Apis: IEnumerable<[IApi](#ref-iapi)\><br /><br /> ApprovalRequired: bool<br /><br /> Groups: IEnumerable<[IGroup](#ref-igroup)\><br /><br /> Id: string<br /><br /> Name: string<br /><br /> State: enum ProductState {NotPublished, Published}<br /><br /> SubscriptionLimit: int?<br /><br /> SubscriptionRequired: bool|
|<a id="ref-context-request"></a>context.Request|Corps : [IMessageBody](#ref-imessagebody) ou `null` si la requête n’a pas de corps.<br /><br /> Certificate: System.Security.Cryptography.X509Certificates.X509Certificate2<br /><br /> [Headers](#ref-context-request-headers) : IReadOnlyDictionary<string, string[]><br /><br /> IpAddress: string<br /><br /> MatchedParameters: IReadOnlyDictionary<string, string><br /><br /> Method: string<br /><br /> OriginalUrl: [IUrl](#ref-iurl)<br /><br /> Url: [IUrl](#ref-iurl)|
|<a id="ref-context-request-headers"></a>string context.Request.Headers.GetValueOrDefault(headerName: string, defaultValue: string)|headerName: string<br /><br /> defaultValue: string<br /><br /> Renvoie des valeurs d’en-tête de demande séparées par des virgules ou `defaultValue` si l’en-tête est introuvable.|
|<a id="ref-context-response"></a>context.Response|Corps : [IMessageBody](#ref-imessagebody)<br /><br /> [Headers](#ref-context-response-headers) : IReadOnlyDictionary<string, string[]><br /><br /> StatusCode: int<br /><br /> StatusReason: string|
|<a id="ref-context-response-headers"></a>string context.Response.Headers.GetValueOrDefault(headerName: string, defaultValue: string)|headerName: string<br /><br /> defaultValue: string<br /><br /> Renvoie des valeurs d’en-tête de réponse séparées par des virgules ou `defaultValue` si l’en-tête est introuvable.|
|<a id="ref-context-subscription"></a>context.Subscription|CreatedTime: DateTime<br /><br /> EndDate: DateTime?<br /><br /> Id: string<br /><br /> Key: string<br /><br /> Name: string<br /><br /> PrimaryKey: string<br /><br /> SecondaryKey: string<br /><br /> StartDate: DateTime?|
|<a id="ref-context-user"></a>context.User|Email: string<br /><br /> FirstName: string<br /><br /> Groups: IEnumerable<[IGroup](#ref-igroup)\><br /><br /> Id: string<br /><br /> Identities: IEnumerable<[IUserIdentity](#ref-iuseridentity)\><br /><br /> LastName: string<br /><br /> Note: string<br /><br /> RegistrationDate: DateTime|
|<a id="ref-iapi"></a>IApi|Id: string<br /><br /> Name: string<br /><br /> Path: string<br /><br /> Protocols: IEnumerable<string\><br /><br /> ServiceUrl: [IUrl](#ref-iurl)<br /><br /> SubscriptionKeyParameterNames: [ISubscriptionKeyParameterNames](#ref-isubscriptionkeyparameternames)|
|<a id="ref-igroup"></a>IGroup|Id: string<br /><br /> Name: string|
|<a id="ref-imessagebody"></a>IMessageBody|As<T\>(preserveContent: bool = false): Où T: string, JObject, JToken, JArray, XNode, XElement, XDocument<br /><br /> Les méthodes `context.Request.Body.As<T>` et `context.Response.Body.As<T>` sont utilisées pour lire un corps de message de demande et de réponse dans un type `T` spécifié. Par défaut, la méthode utilise le flux du corps du message d’origine et le rend indisponible après son retour. Pour éviter cela en faisant en sorte que la méthode travaille sur une copie du flux du corps, définissez le paramètre `preserveContent` sur `true`. Accédez [ici](api-management-transformation-policies.md#SetBody) pour voir un exemple.|
|<a id="ref-iurl"></a>IUrl|Host: string<br /><br /> Path: string<br /><br /> Port: int<br /><br /> [Requête](#ref-iurl-query) : IReadOnlyDictionary<string, string[]><br /><br /> QueryString: string<br /><br /> Scheme: string|
|<a id="ref-iuseridentity"></a>IUserIdentity|Id: string<br /><br /> Provider: string|
|<a id="ref-isubscriptionkeyparameternames"></a>ISubscriptionKeyParameterNames|Header: string<br /><br /> Query: string|
|<a id="ref-iurl-query"></a>string IUrl.Query.GetValueOrDefault(queryParameterName: string, defaultValue: string)|queryParameterName: string<br /><br /> defaultValue: string<br /><br /> Renvoie des valeurs de paramètre de requête séparées par des virgules ou `defaultValue` si le paramètre est introuvable.|
|<a id="ref-context-variables"></a>T context.Variables.GetValueOrDefault<T\>(variableName: string, defaultValue: T)|variableName: string<br /><br /> defaultValue: T<br /><br /> Renvoie la valeur de variable avec conversion de type vers `T` ou `defaultValue` si la variable est introuvable.<br /><br /> Cette méthode lève une exception si le type spécifié ne correspond pas au type réel de la variable renvoyée.|
|BasicAuthCredentials AsBasic(input: this string)|input: string<br /><br /> Si le paramètre d’entrée contient une valeur valide d’en-tête de demande d’authentification de base HTTP, la méthode renvoie un objet de type `BasicAuthCredentials` ; sinon, la méthode renvoie Null.|
|bool TryParseBasic(input: this string, result: out BasicAuthCredentials)|input: string<br /><br /> result: out BasicAuthCredentials<br /><br /> Si le paramètre d’entrée contient une valeur valide dans l’en-tête de demande d’authentification de base HTTP, la méthode renvoie `true` et le paramètre de résultats contient une valeur de type `BasicAuthCredentials` ; sinon, la méthode renvoie `false`.|
|BasicAuthCredentials|Password: string<br /><br /> UserId: string|
|Jwt AsJwt(input: this string)|input: string<br /><br /> Si le paramètre d’entrée contient une valeur valide d’en-tête de jeton JWT, la méthode renvoie un objet de type `Jwt` ; sinon, la méthode renvoie `null`.|
|bool TryParseJwt(input: this string, result: out Jwt)|input: string<br /><br /> result: out Jwt<br /><br /> Si le paramètre d’entrée contient une valeur valide d’en-tête de jeton JWT, la méthode renvoie `true` et le paramètre de résultats contient une valeur de type `Jwt` ; sinon, la méthode renvoie `false`.|
|Jwt|Algorithm: string<br /><br /> Audiences: IEnumerable<string\><br /><br /> Claims: IReadOnlyDictionary<string, string[]><br /><br /> ExpirationTime: DateTime?<br /><br /> Id: string<br /><br /> Issuer: string<br /><br /> IssuedAt: DateTime?<br /><br /> NotBefore: DateTime?<br /><br /> Subject: string<br /><br /> Type: string|
|string Jwt.Claims.GetValueOrDefault(claimName: string, defaultValue: string)|claimName: string<br /><br /> defaultValue: string<br /><br /> Renvoie des valeurs de revendication séparées par des virgules ou `defaultValue` si l’en-tête est introuvable.|
|byte[] Encrypt(input: this byte[], alg: string, key:byte[], iv:byte[])|input - texte brut à déchiffrer<br /><br />alg - nom de l'algorithme de chiffrement symétrique<br /><br />key - clé de chiffrement<br /><br />iv - vecteur d'initialisation<br /><br />Renvoie un texte brut chiffré.|
|byte[] Encrypt(input: this byte[], alg: System.Security.Cryptography.SymmetricAlgorithm)|input - texte brut à déchiffrer<br /><br />alg - algorithme de chiffrement<br /><br />Renvoie un texte brut chiffré.|
|byte[] Encrypt(input: this byte[], alg: System.Security.Cryptography.SymmetricAlgorithm, key:byte[], iv:byte[])|input - texte brut à déchiffrer<br /><br />alg - algorithme de chiffrement<br /><br />key - clé de chiffrement<br /><br />iv - vecteur d'initialisation<br /><br />Renvoie un texte brut chiffré.|
|byte[] Decrypt(input: this byte[], alg: string, key:byte[], iv:byte[])|input - texte à déchiffrer<br /><br />alg - nom de l'algorithme de chiffrement symétrique<br /><br />key - clé de chiffrement<br /><br />iv - vecteur d'initialisation<br /><br />Renvoie un texte brut.|
|byte[] Decrypt(input: this byte[], alg: System.Security.Cryptography.SymmetricAlgorithm)|input - texte à déchiffrer<br /><br />alg - algorithme de chiffrement<br /><br />Renvoie un texte brut.|
|byte[] Decrypt(input: this byte[], alg: System.Security.Cryptography.SymmetricAlgorithm, key:byte[], iv:byte[])|input - texte à déchiffrer<br /><br />alg - algorithme de chiffrement<br /><br />key - clé de chiffrement<br /><br />iv - vecteur d'initialisation<br /><br />Renvoie un texte brut.|
|bool VerifyNoRevocation(entrée : ce certificat System.Security.Cryptography.X509Certificates.X509Certificate2)|Effectue une validation de la chaîne X.509 sans vérifier l’état de révocation du certificat.<br /><br />Entrée - Objet de certificat<br /><br />Renvoie `true` si la validation réussit et `false` si elle échoue.|


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’utilisation de stratégies, consultez les pages :

+ [Stratégies dans Gestion des API](api-management-howto-policies.md)
+ [Transform and protect your API](transform-api.md) (Transformer et protéger votre API)
+ [Référence de stratégie](./api-management-policies.md) pour obtenir la liste complète des instructions et des paramètres de stratégie
+ [Exemples de stratégie](./policy-reference.md)