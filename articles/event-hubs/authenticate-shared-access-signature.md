---
title: Authentifier l’accès à Azure Event Hubs à l’aide de signatures d’accès partagé
description: Cet article explique comment authentifier l’accès aux ressources Event Hubs à l’aide de signatures d’accès partagé.
ms.topic: conceptual
ms.date: 06/23/2020
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 26c921213168e1028f311eabaa417efdb76f5c97
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97510088"
---
# <a name="authenticate-access-to-event-hubs-resources-using-shared-access-signatures-sas"></a>Authentifier l’accès aux ressources Event Hubs avec des signatures d’accès partagé
La signature d’accès partagé (SAS, shared access signature) vous offre un contrôle granulaire sur le type d’accès que vous octroyez aux clients qui la possèdent. Voici quelques-uns des contrôles que vous pouvez définir dans une signature d’accès partagé : 

- L’intervalle pendant lequel la signature d’accès partagé est valide, y compris l’heure de début et l’heure d’expiration.
- Les autorisations accordées par la SAP. Par exemple, une signature d’accès partagé pour un espace de noms Event Hubs peut octroyer l’autorisation d’écoute, mais pas l’autorisation d’envoi.
- Seuls les clients qui présentent des informations d’identification valides peuvent envoyer des données à un hub d’événements.
- Un client ne peut pas emprunter l’identité d’un autre client.
- Il est possible d’empêcher un client non autorisé d’envoyer des données à un hub d’événements.

Cet article traite de l’authentification de l’accès aux ressources Event Hubs à l’aide d’une signature d’accès partagé. Pour en savoir plus sur l’**autorisation** de l’accès aux ressources Event Hubs à l’aide d’une signature d’accès partagé, consultez [cet article](authorize-access-shared-access-signature.md). 

> [!NOTE]
> Comme bonne pratique de sécurité, Microsoft vous recommande d’utiliser si possible les informations d’identification Azure AD plutôt que les signatures d’accès partagé, qui peuvent être plus facilement compromises. Vous pouvez continuer à utiliser des signatures d’accès partagé pour octroyer un accès affiné aux ressources Event Hubs. Cependant, Azure AD offre des fonctionnalités similaires qui ne nécessitent pas de gérer les jetons SAS ou de révoquer des signatures d’accès partagé compromises.
> 
> Pour plus d’informations sur l’intégration d’Azure AD à Azure Event Hubs, consultez [Autoriser l’accès à Event hubs à l’aide d’Azure AD](authorize-access-azure-active-directory.md). 


## <a name="configuring-for-sas-authentication"></a>Configuration de l’authentification SAS
Vous pouvez configurer la règle d’autorisation d’accès partagé EventHubs sur un espace de noms Event Hubs ou une entité (instance d’Event Hub ou rubrique Kafka dans un Event Hub). La configuration d’une règle d’autorisation d’accès partagé sur un groupe de consommateurs n’est pas prise en charge pour l’instant. Cependant, vous pouvez utiliser les règles configurées sur un espace de noms ou une entité pour sécuriser l’accès au groupe de consommateurs. 

L’illustration suivante montre comment les règles d’autorisation s’appliquent à des exemples d’entités. 

![Configurer une règle d’autorisation](./media/authenticate-shared-access-signature/configure-sas-authorization-rule.png)

Dans cet exemple, l’exemple d’espace de noms Event Hubs (ExampleNamespace) a deux entités : eh1 et topic1. Les règles d’autorisation sont définies au niveau de l’entité et au niveau de l’espace de noms.  

Les règles d’autorisation manageRuleNS, sendRuleNS et listenRuleNS s’appliquent à la rubrique t1 et à l’instance eh1 du hub d’événements. Les règles d’autorisation listenRule-eh et sendRule-eh s’appliquent uniquement à l’instance eh1 du hub d’événements et la règle d’autorisation sendRuleT s’applique uniquement à la rubrique topic1. 

Quand la règle d’autorisation sendRuleNS est utilisée, les applications clientes peuvent effectuer des envois à eh1 et topic1. Quand la règle d’autorisation sendRuleT est utilisée, elle applique un accès granulaire à topic1 uniquement. Ainsi, les applications clientes qui utilisent cette règle pour l’accès ne peuvent plus effectuer des envois à eh1, mais uniquement à topic1.

## <a name="generate-a-shared-access-signature-token"></a>Générer un jeton de signature d’accès partagé 
Les clients qui ont accès au nom d’une règle d’autorisation et à celui de ses clés de signature peuvent générer un jeton SAP. Le jeton est généré suite à l’élaboration d’une chaîne au format suivant :

- `se` - Délai d’expiration du jeton : entier reflétant les secondes depuis l’époque 00:00:00 UTC du 1er janvier 1970 (époque UNIX) quand le jeton expire
- `skn` - Nom de la règle d’autorisation, soit le nom de la clé SAS
- `sr` - URI de la ressource faisant l’objet de l’accès
- `sig` - Signature

La chaîne de signature est le hachage SHA-256 calculé sur l’URI de ressource (étendue comme décrit dans la section précédente) et la représentation de chaîne du délai d’expiration du jeton, séparée par CRLF.

Le calcul de hachage est similaire au code de pseudo suivant et retourne une valeur de hachage de 256 bits/32 octets. 

```
SHA-256('https://<yournamespace>.servicebus.windows.net/'+'\n'+ 1438205742)
```

Le jeton contient les valeurs non hachées afin que le destinataire puisse recalculer le hachage avec les mêmes paramètres, en vérifiant que l’émetteur est en possession d’une clé de signature valide.

L’URI de ressource est l’URI complet de la ressource Service Bus à laquelle vous souhaitez accéder. Par exemple, `http://contoso.servicebus.windows.net/eventhubs/eh1` pour http://<namespace>.servicebus.windows.net/<entityPath> ou `sb://<namespace>.servicebus.windows.net/<entityPath>;`.

L’URI doit être encodée en pourcentage.

La règle de l’autorisation d’accès partagé utilisée pour la signature doit être configurée sur l’entité spécifiée par cette URI, ou par un de ses parents hiérarchiques. Par exemple, `http://contoso.servicebus.windows.net/eventhubs/eh1` ou `http://contoso.servicebus.windows.net` dans l’exemple précédent.

Un jeton SAS est valide pour toutes les ressources avec le préfixe <resourceURI> utilisé dans la chaîne de signature.

> [!NOTE]
> Vous générez un jeton d’accès pour Event Hubs à l’aide de la stratégie d’accès partagé. Pour plus d’informations, consultez [Stratégies d’autorisation d’accès partagé](authorize-access-shared-access-signature.md#shared-access-authorization-policies).

### <a name="generating-a-signaturetoken-from-a-policy"></a>Génération d’une signature (jeton) à partir d’une stratégie 
La section suivante explique comment générer un jeton SAS à l’aide de stratégies de signature d’accès partagé.

#### <a name="nodejs"></a>NodeJS

```javascript
function createSharedAccessToken(uri, saName, saKey) { 
    if (!uri || !saName || !saKey) { 
            throw "Missing required parameter"; 
        } 
    var encoded = encodeURIComponent(uri); 
    var now = new Date(); 
    var week = 60*60*24*7;
    var ttl = Math.round(now.getTime() / 1000) + week;
    var signature = encoded + '\n' + ttl; 
    var signatureUTF8 = utf8.encode(signature); 
    var hash = crypto.createHmac('sha256', saKey).update(signatureUTF8).digest('base64'); 
    return 'SharedAccessSignature sr=' + encoded + '&sig=' +  
        encodeURIComponent(hash) + '&se=' + ttl + '&skn=' + saName; 
```

#### <a name="java"></a>Java

```java
private static String GetSASToken(String resourceUri, String keyName, String key)
  {
      long epoch = System.currentTimeMillis()/1000L;
      int week = 60*60*24*7;
      String expiry = Long.toString(epoch + week);

      String sasToken = null;
      try {
          String stringToSign = URLEncoder.encode(resourceUri, "UTF-8") + "\n" + expiry;
          String signature = getHMAC256(key, stringToSign);
          sasToken = "SharedAccessSignature sr=" + URLEncoder.encode(resourceUri, "UTF-8") +"&sig=" +
                  URLEncoder.encode(signature, "UTF-8") + "&se=" + expiry + "&skn=" + keyName;
      } catch (UnsupportedEncodingException e) {

          e.printStackTrace();
      }

      return sasToken;
  }


public static String getHMAC256(String key, String input) {
    Mac sha256_HMAC = null;
    String hash = null;
    try {
        sha256_HMAC = Mac.getInstance("HmacSHA256");
        SecretKeySpec secret_key = new SecretKeySpec(key.getBytes(), "HmacSHA256");
        sha256_HMAC.init(secret_key);
        Encoder encoder = Base64.getEncoder();

        hash = new String(encoder.encode(sha256_HMAC.doFinal(input.getBytes("UTF-8"))));

    } catch (InvalidKeyException e) {
        e.printStackTrace();
    } catch (NoSuchAlgorithmException e) {
        e.printStackTrace();
   } catch (IllegalStateException e) {
        e.printStackTrace();
    } catch (UnsupportedEncodingException e) {
        e.printStackTrace();
    }

    return hash;
}
```
#### <a name="php"></a>PHP

```php
function generateSasToken($uri, $sasKeyName, $sasKeyValue) 
{ 
    $targetUri = strtolower(rawurlencode(strtolower($uri))); 
    $expires = time();  
    $expiresInMins = 60; 
    $week = 60*60*24*7;
    $expires = $expires + $week; 
    $toSign = $targetUri . "\n" . $expires; 
    $signature = rawurlencode(base64_encode(hash_hmac('sha256',             
     $toSign, $sasKeyValue, TRUE))); 
    
    $token = "SharedAccessSignature sr=" . $targetUri . "&sig=" . $signature . "&se=" . $expires .      "&skn=" . $sasKeyName; 
    return $token; 
}
```

#### <a name="c"></a>C#

```csharp
private static string createToken(string resourceUri, string keyName, string key)
{
    TimeSpan sinceEpoch = DateTime.UtcNow - new DateTime(1970, 1, 1);
    var week = 60 * 60 * 24 * 7;
    var expiry = Convert.ToString((int)sinceEpoch.TotalSeconds + week);
    string stringToSign = HttpUtility.UrlEncode(resourceUri) + "\n" + expiry;
    HMACSHA256 hmac = new HMACSHA256(Encoding.UTF8.GetBytes(key));
    var signature = Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(stringToSign)));
    var sasToken = String.Format(CultureInfo.InvariantCulture, "SharedAccessSignature sr={0}&sig={1}&se={2}&skn={3}", HttpUtility.UrlEncode(resourceUri), HttpUtility.UrlEncode(signature), expiry, keyName);
    return sasToken;
}
```

## <a name="authenticating-event-hubs-publishers-with-sas"></a>Authentification des éditeurs Event Hubs avec une signature d’accès partagé 
Un éditeur d’événements définit un point de terminaison virtuel pour un hub d’événements. L’éditeur peut être utilisé uniquement pour envoyer des messages à un hub d’événements et non pour en recevoir.

En règle générale, un hub d’événements utilise un seul éditeur par client. Tous les messages qui sont envoyés à l’un des éditeurs d’un hub d’événements sont empilés dans celui-ci. Les éditeurs permettent un contrôle d’accès précis.

Un jeton unique, qui est téléchargé sur le client, est affecté à chaque hub d’événement. Les jetons sont produits de façon à ce que chaque jeton unique donne accès à un éditeur unique différent. Un client qui possède un jeton ne peut envoyer qu’à un seul éditeur et à aucun autre. Si plusieurs clients partagent le même jeton, alors ils partagent l’éditeur.

Des clés SAS sont attribuées à tous les jetons. En règle générale, tous les jetons sont signés avec la même clé. Les clients n’ont pas connaissance de la clé, ce qui les empêche de fabriquer des jetons. Les clients utilisent les mêmes jetons jusqu’à leur expiration.

Par exemple, pour définir des règles d’autorisation limitées à l’envoi et à la publication sur Event Hubs, vous devez définir une règle d’autorisation d’envoi. Cela peut être effectué au niveau de l’espace de noms. Il est également possible d’attribuer à une entité particulière (rubrique ou instance de hubs d’événements) une étendue plus granulaire. Un client ou une application dont l’étendue est définie avec ce type d’accès granulaire est appelé éditeur Event Hubs. Pour ce faire, procédez comme suit :

1. Créez une clé SAS sur l’entité que vous souhaitez publier pour lui attribuer l’étendue d’**envoi**. Pour plus d’informations, consultez [Stratégies d’autorisation d’accès partagé](authorize-access-shared-access-signature.md#shared-access-authorization-policies).
2. Générez un jeton SAS avec un délai d’expiration pour un éditeur spécifique à l’aide de la clé générée à l’étape 1.

    ```csharp
    var sasToken = SharedAccessSignatureTokenProvider.GetPublisherSharedAccessSignature(
                new Uri("Service-Bus-URI"),
                "eventub-name",
                "publisher-name",
                "sas-key-name",
                "sas-key",
                TimeSpan.FromMinutes(30));
    ```
3. Fournissez le jeton au client de l’éditeur, qui peut uniquement effectuer des envois à l’entité et à l’éditeur auxquels le jeton octroie l’accès.

    Quand le jeton expire, le client perd l’accès lui permettant d’effectuer des envois et des publications vers l’entité. 


> [!NOTE]
> Même si cela est déconseillé, il est possible d’équiper les appareils de jetons qui octroient un accès à un hub d’événements ou à un espace de noms. N’importe quel appareil qui détient ce jeton peut envoyer des messages directement à ce hub d’événements. En outre, il est impossible d’empêcher l’appareil d’effectuer des envois à ce hub d’événements.
> 
> Il est toujours recommandé de fournir des étendues spécifiques et granulaires.

> [!IMPORTANT]
> Une fois les jetons créés, chaque client est configuré avec son propre jeton unique.
>
> Quand le client envoie des données à un hub d’événements, il balise sa requête avec le jeton. Pour empêcher un intrus de procéder à des écoutes clandestines et de voler le jeton, la communication entre le client et le hub d’événements doit avoir lieu sur un canal chiffré.
> 
> Si un jeton est volé par un intrus, celui-ci peut emprunter l’identité du client à qui le jeton a été volé. L’inscription d’un serveur de publication dans une liste de refus rend le client inutilisable, jusqu’à ce qu’il reçoive un nouveau jeton qui utilise un serveur de publication différent.


## <a name="authenticating-event-hubs-consumers-with-sas"></a>Authentification des consommateurs Event Hubs avec une signature d’accès partagé 
Pour authentifier les applications back-end qui consomment des données générées par des producteurs Event hubs, l’authentification par jeton Event Hubs nécessite que les clients disposent des droits de **gestion** ou des privilèges d’**écoute** attribués à l’espace de noms Event hubs ou à l’instance ou la rubrique de hub d’événements. Les données sont consommées à partir de Event Hubs par le biais de groupes de consommateurs. Même si la stratégie SAS vous donne une étendue granulaire, cette étendue est définie uniquement au niveau de l’entité et non au niveau du consommateur. Cela signifie que les privilèges définis au niveau de l’espace de noms ou au niveau de la rubrique ou de l’instance du hub d’événements seront appliqués aux groupes de consommateurs de cette entité.

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- [Autorisation avec des signatures d’accès partagé](authenticate-shared-access-signature.md)
- [Autorisation avec le contrôle d’accès en fonction du rôle Azure (Azure RBAC)](authenticate-shared-access-signature.md)
- [En savoir plus sur Event Hubs](event-hubs-about.md)

Consultez les articles associés suivants :

- [Authentifier les requêtes adressées à Azure Event Hubs à partir d’une application à l’aide d’Azure Active Directory](authenticate-application.md)
- [Authentifier une identité managée avec Azure Active Directory pour accéder aux ressources Event Hubs](authenticate-managed-identity.md)
- [Autoriser l’accès aux ressources Event Hubs à l’aide d’Azure Active Directory](authorize-access-azure-active-directory.md)
- [Authentifier l’accès aux ressources Event Hubs avec des signatures d’accès partagé](authorize-access-shared-access-signature.md)
