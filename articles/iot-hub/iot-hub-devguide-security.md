---
title: Présentation de la sécurité d’Azure IoT Hub | Microsoft Docs
description: 'Guide du développeur : comment contrôler l’accès à IoT Hub pour les applications d’appareil et applications principales. Inclut des informations sur les jetons de sécurité et la prise en charge des certificats X.509.'
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 07/18/2018
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
- 'Role: Operations'
- devx-track-js
- devx-track-csharp
ms.openlocfilehash: 3ddc8c78bac47ed85266037341328585e3c7cb1c
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825126"
---
# <a name="control-access-to-iot-hub"></a>Contrôler l’accès à IoT Hub

Cet article décrit les options de sécurisation de votre hub IoT. IoT Hub utilise des *autorisations* pour accorder l’accès à chaque point de terminaison IoT Hub. Les autorisations limitent l’accès à un hub IoT selon la fonctionnalité.

Cet article présente :

* Les différentes autorisations que vous pouvez accorder à une application d’appareil ou de serveur principal pour accéder à votre hub IoT.
* Le processus d’authentification et les jetons qu’il utilise pour vérifier les autorisations.
* Le mode de définition de l’étendue des informations d’identification pour limiter l’accès à des ressources spécifiques.
* La prise en charge par IoT Hub des certificats X.509.
* Les mécanismes d’authentification d’appareil personnalisés qui utilisent des registres d’identités d’appareil ou des schémas d’authentification existants.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

Pour accéder à tout point de terminaison IoT Hub, vous devez disposer des autorisations appropriées. Par exemple, un appareil doit inclure un jeton contenant des informations d’identification de sécurité dans chaque message envoyé à IoT Hub.

## <a name="access-control-and-permissions"></a>Contrôle d’accès et autorisations

Vous pouvez accorder des [autorisations](#iot-hub-permissions) de différentes manières :

* **Stratégies d’accès partagé au niveau d’IoT Hub**. Les stratégies d’accès partagé peuvent accorder n’importe quelle combinaison d’[autorisations](#iot-hub-permissions). Vous pouvez définir des stratégies dans le [portail Azure](https://portal.azure.com), par programmation à l’aide des [API REST de ressources IoT Hub](/rest/api/iothub/iothubresource) ou en utilisant l’interface CLI [az iot hub policy](/cli/azure/iot/hub/policy). Un hub IoT qui vient d’être créé a les stratégies par défaut suivantes :
  
  | Stratégie d’accès partagé | Autorisations |
  | -------------------- | ----------- |
  | iothubowner | Toutes les autorisations |
  | service | Autorisations **ServiceConnect** |
  | device | Autorisations **DeviceConnect** |
  | registryRead | Autorisations **RegistryRead** |
  | registryReadWrite | Autorisations **RegistryRead** et **RegistryWrite** |

* **Informations d’identification de sécurité par appareil**. Chaque IoT Hub contient un [registre des identités](iot-hub-devguide-identity-registry.md). Pour chaque appareil dans ce registre des identités, vous pouvez configurer des informations d’identification de sécurité qui accordent des autorisations **DeviceConnect** incluses dans l’étendue des points de terminaison des appareils correspondants.

Par exemple, dans une solution IoT classique :

* Le composant de gestion des appareils utilise la stratégie *registryReadWrite* .
* Le composant de processeur d’événements utilise la stratégie *service* .
* Le composant de logique métier des appareils d’exécution utilise la stratégie *service*.
* Les appareils individuels se connectent à l’aide d’informations d’identification stockées dans le registre des identités du hub IoT.

> [!NOTE]
> Pour plus d’informations, consultez la page [Autorisations](#iot-hub-permissions).

## <a name="authentication"></a>Authentification

Azure IoT Hub accorde l’accès aux points de terminaison en vérifiant un jeton par rapport aux stratégies d’accès partagé et aux informations d’identification de sécurité du registre des identités.

Les informations d’identification de sécurité telles que les clés symétriques ne sont jamais envoyées sur le réseau.

> [!NOTE]
> Le fournisseur de ressources Azure IoT Hub est sécurisé par votre abonnement Azure, comme tous les fournisseurs dans [Azure Resource Manager](../azure-resource-manager/management/overview.md).

Pour plus d’informations sur la construction et l’utilisation des jetons de sécurité, consultez [Jetons de sécurité IoT Hub](iot-hub-devguide-security.md#security-tokens).

### <a name="protocol-specifics"></a>Spécifications de protocole

Chaque protocole pris en charge (par exemple, MQTT, AMQP et HTTPS) transporte des jetons de différentes façons.

Quand vous utilisez MQTT, le paquet CONNECT utilise deviceid en tant que ClientId, `{iothubhostname}/{deviceId}` dans le champ Nom d’utilisateur et un jeton SAP dans le champ Mot de passe. `{iothubhostname}` doit être le nom canonique (CNAME) complet du hub IoT (par exemple, contoso.azure-devices.net).

Lorsque vous utilisez [AMQP](https://www.amqp.org/), IoT Hub prend en charge [SASL PLAIN](https://tools.ietf.org/html/rfc4616) et la [sécurité basée sur des revendications AMQP](https://www.oasis-open.org/committees/download.php/50506/amqp-cbs-v1%200-wd02%202013-08-12.doc).

Si vous utilisez une sécurité basée sur des revendications AMQP, la norme indique comment transmettre les jetons répertoriés ci-dessus.

Pour SASL PLAIN, le **nom d’utilisateur** peut être :

* `{policyName}@sas.root.{iothubName}` si vous utilisez des jetons au niveau du hub IoT.
* `{deviceId}@sas.{iothubname}` si vous utilisez des jetons à l’échelle de l’appareil.

Dans les deux cas, le champ de mot de passe contient le jeton, comme indiqué dans [Jetons de sécurité IoT Hub](iot-hub-devguide-security.md#security-tokens).

Le protocole HTTPS implémente l’authentification en incluant un jeton valide dans l’en-tête de requête **Authorization**.

#### <a name="example"></a>Exemple

Nom d’utilisateur (DeviceId respecte la casse) : `iothubname.azure-devices.net/DeviceId`

Mot de passe (vous pouvez générer un jeton SAS à l'aide de la commande d'extension CLI [az iot hub generate-sas-token](/cli/azure/ext/azure-iot/iot/hub#ext-azure-iot-az-iot-hub-generate-sas-token) ou d'[Azure IoT Tools pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)) :

`SharedAccessSignature sr=iothubname.azure-devices.net%2fdevices%2fDeviceId&sig=kPszxZZZZZZZZZZZZZZZZZAhLT%2bV7o%3d&se=1487709501`

> [!NOTE]
> Les [kis de développement logiciel (SDK) Azure IoT](iot-hub-devguide-sdks.md) génèrent automatiquement des jetons lors de la connexion au service. Dans certains cas, les kits Azure IoT SDK ne prennent pas en charge l’ensemble des protocoles ou méthodes d’authentification.

### <a name="special-considerations-for-sasl-plain"></a>Considérations spécifiques concernant SASL PLAIN

Lorsque vous utilisez SASL PLAIN avec AMQP, un client qui se connecte à un IoT Hub peut utiliser un jeton unique pour chaque connexion TCP. Quand le jeton expire, la connexion TCP est déconnectée du service, ce qui déclenche une reconnexion. Bien que non problématique pour une application principale, ce comportement peut créer des dommages pour une application d’appareil pour les motifs suivants :

* Les passerelles se connectent généralement au nom de nombreux appareils. Lorsque vous utilisez SASL PLAIN, elles doivent créer une connexion TCP distincte pour chaque appareil se connectant à un hub IoT. Ce scénario augmente considérablement la consommation des ressources d’alimentation et de mise en réseau, ainsi que la latence de chaque connexion d’appareil.

* Les appareils avec des contraintes de ressources sont affectés par l’utilisation accrue des ressources pour se reconnecter après chaque expiration du jeton.

## <a name="scope-iot-hub-level-credentials"></a>Étendue des informations d’identification au niveau du hub IoT

Vous pouvez étendre les stratégies de sécurité au niveau du hub IoT en créant des jetons avec un URI de ressource restreint. Par exemple, le point de terminaison pour l’envoi de messages appareil-à-cloud est **/devices/{deviceId}/messages/events**. Vous pouvez également utiliser une stratégie d’accès partagé au niveau du hub IoT avec des autorisations **DeviceConnect** pour signer un jeton dont l’URI de ressource est **/devices/{deviceId}** . Cette approche crée un jeton utilisable uniquement pour envoyer des messages au nom de l’appareil **deviceId**.

Ce mécanisme similaire à la [Stratégie de publication Event Hubs](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-99ce67ab) vous permet d’implémenter des méthodes d’authentification personnalisées.

## <a name="security-tokens"></a>Jetons de sécurité

IoT Hub utilise des jetons de sécurité pour authentifier les appareils et les services afin d’éviter d’envoyer des clés sur le réseau. En outre, la validité et la portée des jetons sont limitées dans le temps. Les [kits de développement logiciel (SDK) Azure IoT](iot-hub-devguide-sdks.md) génèrent automatiquement des jetons, sans nécessiter de configuration particulière. Certains scénarios nécessitent toutefois que vous génériez et utilisiez directement des jetons de sécurité. Il s’agit entre autres des scénarios suivants :

* Utilisation directe des surfaces MQTT, AMQP ou HTTPS.

* Implémentation du modèle de service de jeton, comme indiqué dans [Authentification d’appareil personnalisée](iot-hub-devguide-security.md#custom-device-and-module-authentication).

IoT Hub permet également aux appareils de s’authentifier avec IoT Hub à l’aide de [certificats X.509](iot-hub-devguide-security.md#supported-x509-certificates).

### <a name="security-token-structure"></a>Structure du jeton de sécurité

Vous utilisez des jetons de sécurité pour accorder un accès limité dans le temps aux appareils et aux services à des fonctionnalités spécifiques dans IoT Hub. Pour obtenir l’autorisation de se connecter à IoT Hub, les appareils et services doivent envoyer des jetons de sécurité signés avec une clé symétrique ou d’accès partagé. Ces clés sont stockées avec l’identité de l’appareil dans le registre de l’identité.

Un jeton signé avec une clé d’accès partagé accorde un accès à toutes les fonctionnalités associées aux autorisations de stratégie d’accès partagé. Un jeton signé avec la clé symétrique de l’identité de l’appareil accorde uniquement l’autorisation **DeviceConnect** à l’identité de l’appareil associée.

Le jeton de sécurité présente le format suivant :

`SharedAccessSignature sig={signature-string}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}`

Voici les valeurs attendues :

| Valeur | Description |
| --- | --- |
| {signature} |Une chaîne de signature HMAC-SHA256 sous la forme : `{URL-encoded-resourceURI} + "\n" + expiry`. **Important !** la clé est décodée à partir de base64 et utilisée comme clé pour effectuer le calcul HMAC-SHA256. |
| {resourceURI} |Préfixe URI (par segment) des points de terminaison qui sont accessibles avec ce jeton, en commençant par le nom d’hôte IoT Hub (sans protocole). Par exemple : `myHub.azure-devices.net/devices/device1` |
| {expiry} |Chaînes UTF8 pour le nombre de secondes depuis l’époque 00:00:00 UTC 1er janvier 1970. |
| {URL-encoded-resourceURI} |Encodage en URL minuscules de l’URI de ressource en minuscules |
| {policyName} |Le nom de la stratégie d’accès partagé à laquelle ce jeton fait référence. Il n’y en a pas si le jeton fait référence aux informations d’identification de registre des appareils. |

**Remarque sur le préfixe** : le préfixe URI est calculé par segment et non par caractère. Par exemple `/a/b` est un préfixe de `/a/b/c`, mais pas de `/a/bc`.

L’extrait de code Node.js suivant illustre une fonction appelée **generateSasToken** qui calcule le jeton à partir des entrées `resourceUri, signingKey, policyName, expiresInMins`. Les sections suivantes décrivent en détail comment initialiser les différentes entrées pour les différents cas d’utilisation des jetons.

```javascript
var generateSasToken = function(resourceUri, signingKey, policyName, expiresInMins) {
    resourceUri = encodeURIComponent(resourceUri);

    // Set expiration in seconds
    var expires = (Date.now() / 1000) + expiresInMins * 60;
    expires = Math.ceil(expires);
    var toSign = resourceUri + '\n' + expires;

    // Use crypto
    var hmac = crypto.createHmac('sha256', Buffer.from(signingKey, 'base64'));
    hmac.update(toSign);
    var base64UriEncoded = encodeURIComponent(hmac.digest('base64'));

    // Construct authorization string
    var token = "SharedAccessSignature sr=" + resourceUri + "&sig="
    + base64UriEncoded + "&se=" + expires;
    if (policyName) token += "&skn="+policyName;
    return token;
};
```

À titre de comparaison, le code Python équivalent pour générer un jeton de sécurité est :

```python
from base64 import b64encode, b64decode
from hashlib import sha256
from time import time
from urllib import parse
from hmac import HMAC

def generate_sas_token(uri, key, policy_name, expiry=3600):
    ttl = time() + expiry
    sign_key = "%s\n%d" % ((parse.quote_plus(uri)), int(ttl))
    print(sign_key)
    signature = b64encode(HMAC(b64decode(key), sign_key.encode('utf-8'), sha256).digest())

    rawtoken = {
        'sr' :  uri,
        'sig': signature,
        'se' : str(int(ttl))
    }

    if policy_name is not None:
        rawtoken['skn'] = policy_name

    return 'SharedAccessSignature ' + parse.urlencode(rawtoken)
```

La fonctionnalité en C# pour générer un jeton de sécurité est la suivante :

```csharp
using System;
using System.Globalization;
using System.Net;
using System.Net.Http;
using System.Security.Cryptography;
using System.Text;

public static string generateSasToken(string resourceUri, string key, string policyName, int expiryInSeconds = 3600)
{
    TimeSpan fromEpochStart = DateTime.UtcNow - new DateTime(1970, 1, 1);
    string expiry = Convert.ToString((int)fromEpochStart.TotalSeconds + expiryInSeconds);

    string stringToSign = WebUtility.UrlEncode(resourceUri) + "\n" + expiry;

    HMACSHA256 hmac = new HMACSHA256(Convert.FromBase64String(key));
    string signature = Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(stringToSign)));

    string token = String.Format(CultureInfo.InvariantCulture, "SharedAccessSignature sr={0}&sig={1}&se={2}", WebUtility.UrlEncode(resourceUri), WebUtility.UrlEncode(signature), expiry);

    if (!String.IsNullOrEmpty(policyName))
    {
        token += "&skn=" + policyName;
    }

    return token;
}
```

Pour Java :
```java
    public static String generateSasToken(String resourceUri, String key) throws Exception {
        // Token will expire in one hour
        var expiry = Instant.now().getEpochSecond() + 3600;

        String stringToSign = URLEncoder.encode(resourceUri, StandardCharsets.UTF_8) + "\n" + expiry;
        byte[] decodedKey = Base64.getDecoder().decode(key);

        Mac sha256HMAC = Mac.getInstance("HmacSHA256");
        SecretKeySpec secretKey = new SecretKeySpec(decodedKey, "HmacSHA256");
        sha256HMAC.init(secretKey);
        Base64.Encoder encoder = Base64.getEncoder();

        String signature = new String(encoder.encode(
            sha256HMAC.doFinal(stringToSign.getBytes(StandardCharsets.UTF_8))), StandardCharsets.UTF_8);

        String token = "SharedAccessSignature sr=" + URLEncoder.encode(resourceUri, StandardCharsets.UTF_8)
                + "&sig=" + URLEncoder.encode(signature, StandardCharsets.UTF_8.name()) + "&se=" + expiry;
            
        return token;
    }
```


> [!NOTE]
> Étant donné que la validité du jeton est validée sur les ordinateurs IoT Hub, la dérive de l’horloge de l’ordinateur qui génère le jeton doit être minime.

### <a name="use-sas-tokens-in-a-device-app"></a>Utiliser des jetons SPA dans une application d’appareil

Il existe deux façons d’obtenir des autorisations **DeviceConnect** avec IoT Hub au moyen de jetons de sécurité : utiliser une [clé d’appareil symétrique extraite du registre des identités](#use-a-symmetric-key-in-the-identity-registry) ou une [clé d’accès partagé](#use-a-shared-access-policy).

N’oubliez pas que toutes les fonctionnalités accessibles à partir des appareils sont exposées par définition sur les points de terminaison avec le préfixe `/devices/{deviceId}`.

> [!IMPORTANT]
> La seule façon dont IoT Hub authentifie un appareil est à l’aide de la clé symétrique d’identité de l’appareil. Dans le cas où une stratégie d’accès partagé est utilisée pour accéder aux fonctionnalités de l’appareil, la solution doit prendre en compte le composant émettant le jeton de sécurité en tant que sous-composant approuvé.

Les points de terminaison côté appareil sont (quel que soit le protocole) :

| Point de terminaison | Fonctionnalités |
| --- | --- |
| `{iot hub host name}/devices/{deviceId}/messages/events` |Envoyer des messages appareil-à-cloud. |
| `{iot hub host name}/devices/{deviceId}/messages/devicebound` |Recevoir des messages cloud-à-appareil. |

### <a name="use-a-symmetric-key-in-the-identity-registry"></a>Utilisation d’une clé symétrique dans le registre d’identité

Lorsqu’une clé symétrique d’identité de l’appareil est utilisée pour générer un jeton, l’élément PolicyName (`skn`) du jeton est omis.

Par exemple, un jeton créé pour accéder à toutes les fonctionnalités de l’appareil doit avoir les paramètres suivants :

* URI de ressource : `{IoT hub name}.azure-devices.net/devices/{device id}`,
* clé de signature : n’importe quelle clé symétrique pour l’identité `{device id}` ,
* pas de nom de stratégie,
* n’importe quelle heure d’expiration.

Un exemple d’utilisation de la fonction Node.js précédente serait :

```javascript
var endpoint ="myhub.azure-devices.net/devices/device1";
var deviceKey ="...";

var token = generateSasToken(endpoint, deviceKey, null, 60);
```

Le résultat, qui accorde l’accès à toutes les fonctionnalités de device1, est alors :

`SharedAccessSignature sr=myhub.azure-devices.net%2fdevices%2fdevice1&sig=13y8ejUk2z7PLmvtwR5RqlGBOVwiq7rQR3WZ5xZX3N4%3D&se=1456971697`

> [!NOTE]
> Il est possible de générer un jeton SAS à l'aide de la commande d'extension CLI [az iot hub generate-sas-token](/cli/azure/ext/azure-iot/iot/hub#ext-azure-iot-az-iot-hub-generate-sas-token) ou d'[Azure IoT Tools pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).

### <a name="use-a-shared-access-policy"></a>Utilisation d’une stratégie d’accès partagé

Quand vous créez un jeton à partir d’une stratégie d’accès partagé, spécifiez le nom de la stratégie dans le champ `skn`. Cette stratégie doit accorder l’autorisation **DeviceConnect**.

Les deux principaux scénarios pour l’utilisation de stratégies d’accès partagé pour accéder aux fonctionnalités de l’appareil sont :

* [passerelles de protocole cloud](iot-hub-devguide-endpoints.md),
* [services de jeton](iot-hub-devguide-security.md#custom-device-and-module-authentication) utilisés pour implémenter des schémas d’authentification personnalisés.

Étant donné que la stratégie d’accès partagé peut potentiellement accorder un accès pour se connecter comme n’importe quel appareil, il est important d’utiliser l’URI de ressource correct lors de la création de jetons de sécurité. Ce paramètre est particulièrement important pour les services de jeton dont il faut définir l’étendue pour un appareil spécifique à l’aide de l’URI de ressource. Ce point est moins important pour les passerelles de protocole, car elles interceptent déjà le trafic pour tous les appareils.

Par exemple, un service de jeton utilisant la stratégie d’accès partagé précréée appelée **device** crée un jeton avec les paramètres suivants :

* URI de ressource : `{IoT hub name}.azure-devices.net/devices/{device id}`,
* clé de signature : une des clés de la stratégie `device` ,
* nom de la stratégie : `device`,
* n’importe quelle heure d’expiration.

Un exemple d’utilisation de la fonction Node.js précédente serait :

```javascript
var endpoint ="myhub.azure-devices.net/devices/device1";
var policyName = 'device';
var policyKey = '...';

var token = generateSasToken(endpoint, policyKey, policyName, 60);
```

Le résultat, qui accorde l’accès à toutes les fonctionnalités de device1, est alors :

`SharedAccessSignature sr=myhub.azure-devices.net%2fdevices%2fdevice1&sig=13y8ejUk2z7PLmvtwR5RqlGBOVwiq7rQR3WZ5xZX3N4%3D&se=1456971697&skn=device`

Une passerelle de protocole peut utiliser le même jeton pour tous les appareils en définissant simplement l’URI de ressource sur `myhub.azure-devices.net/devices`.

### <a name="use-security-tokens-from-service-components"></a>Utilisation de jetons de sécurité de composants du service

Les composants de service peuvent uniquement créer des jetons de sécurité utilisant des stratégies d’accès partagé pour accorder les autorisations adaptées, comme expliqué précédemment.

Voici les fonctions de service exposées sur les points de terminaison :

| Point de terminaison | Fonctionnalités |
| --- | --- |
| `{iot hub host name}/devices` |Créer, mettre à jour, récupérer et supprimer les identités des appareils. |
| `{iot hub host name}/messages/events` |Recevoir des messages appareil-à-cloud. |
| `{iot hub host name}/servicebound/feedback` |Recevoir des commentaires pour les messages cloud-à-appareil. |
| `{iot hub host name}/devicebound` |Envoyer des messages cloud-à-appareil. |

Par exemple, un service qui génère à l’aide de la stratégie d’accès partagé précréée appelée **registryRead** crée un jeton avec les paramètres suivants :

* URI de ressource : `{IoT hub name}.azure-devices.net/devices`,
* clé de signature : une des clés de la stratégie `registryRead` ,
* nom de la stratégie : `registryRead`,
* n’importe quelle heure d’expiration.

```javascript
var endpoint ="myhub.azure-devices.net/devices";
var policyName = 'registryRead';
var policyKey = '...';

var token = generateSasToken(endpoint, policyKey, policyName, 60);
```

Le résultat, qui revient à accorder l’accès en lecture à toutes les identités des appareils, est alors :

`SharedAccessSignature sr=myhub.azure-devices.net%2fdevices&sig=JdyscqTpXdEJs49elIUCcohw2DlFDR3zfH5KqGJo4r4%3D&se=1456973447&skn=registryRead`

## <a name="supported-x509-certificates"></a>Certificats X.509 pris en charge

Vous pouvez utiliser n’importe quel certificat X.509 pour authentifier un appareil avec IoT Hub en chargeant une empreinte de certificat ou une autorité de certification (CA) sur Azure IoT Hub. L’authentification à l’aide d’empreintes de certificat vérifie que l’empreinte présentée correspond à l’empreinte configurée. L’authentification à l’aide de l’autorité de certification valide la chaîne de certificats. Dans les deux cas, la négociation TLS nécessite que l’appareil dispose d’un certificat et d’une clé privée valides. Pour plus d’informations, voir la spécification TLS, par exemple : [RFC 5246 – Protocole TLS version 1.2](https://tools.ietf.org/html/rfc5246/).

Les certificats pris en charge incluent :

* **un certificat X.509 existant**. Un appareil peut être déjà associé à un certificat X.509. L’appareil peut utiliser ce certificat pour s’authentifier sur IoT Hub ; Fonctionne avec l’authentification à l’aide de l’empreinte ou de l’autorité de certification. 

* **un certificat X.509 signé par une autorité de certification**. Pour identifier un appareil et l’authentifier auprès de IoT Hub, vous pouvez utiliser un certificat X.509 généré et signé par une autorité de certification. Fonctionne avec l’authentification à l’aide de l’empreinte ou de l’autorité de certification.

* **un certificat X-509 généré et signé automatiquement**. Un fabricant d’appareils ou un technicien de déploiement en interne peut générer ces certificats et stocker la clé privée correspondante (ainsi que le certificat) sur l’appareil. Vous pouvez utiliser des outils tels que [OpenSSL](https://www.openssl.org/) ou l’utilitaire [Windows SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate) à cette fin. Fonctionne uniquement avec l’authentification à l’aide de l’empreinte.

Un appareil peut utiliser un certificat X.509 ou un jeton de sécurité pour l’authentification, mais pas les deux. Avec l’authentification par certificat X.509, veillez à mettre en place une stratégie pour gérer la substitution de certificat à l’expiration d’un certificat existant.

Les fonctionnalités suivantes ne sont pas prises en charge pour les appareils qui utilisent l’authentification de l’autorité de certification X.509 :

* protocoles HTTPS, MQTT sur WebSockets, et AMQP sur WebSockets.
* Chargements de fichiers (tous les protocoles).

Pour plus d’informations sur l’authentification à l’aide de l’autorité de certification, consultez [Authentification des appareils à l’aide de certificats d’autorité de certification X.509](iot-hub-x509ca-overview.md). Pour plus d’informations sur le chargement et la vérification d’une autorité de certification avec votre IoT Hub, consultez [Configurer la sécurité X.509 dans votre Azure IoT Hub](iot-hub-security-x509-get-started.md).

### <a name="register-an-x509-certificate-for-a-device"></a>Inscrire un certificat X.509 pour un appareil

[Azure IoT service SDK pour C#](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/iothub/service) (version 1.0.8+) prend en charge l’inscription d’un appareil qui utilise un certificat X.509 pour s’authentifier. D’autres API telles que l’importation/exportation d’appareils prennent également en charge les certificats X.509.

Vous pouvez également utiliser la commande d’extension CLI [az iot hub device-identity](/cli/azure/ext/azure-iot/iot/hub/device-identity) pour configurer des certificats X.509 pour les appareils.

### <a name="c-support"></a>Prise en charge de C\#

La classe **RegistryManager** offre un moyen d’inscrire un appareil dans le cadre d’un programme. Les méthodes **AddDeviceAsync** et **UpdateDeviceAsync** vous permettent de vous inscrire et mettre à jour un appareil dans le registre des identités IoT Hub. Ces deux méthodes utilisent une instance **Device** comme entrée. La classe **Device** inclut une propriété **Authentication** qui vous permet de spécifier les empreintes de certificats X.509 primaires et secondaires. L’empreinte numérique représente un hachage SHA256 du certificat X.509 (stocké à l’aide d’un codage DER binaire). Vous pouvez spécifier une empreinte numérique principale et/ou une empreinte numérique secondaire. Les empreintes numériques principales et secondaires sont prises en charge pour la gestion des scénarios de substitution de certificat.

Voici un exemple d’extrait de code C\# permettant d’inscrire un appareil à l’aide d’une empreinte de certificat X.509 :

```csharp
var device = new Device(deviceId)
{
  Authentication = new AuthenticationMechanism()
  {
    X509Thumbprint = new X509Thumbprint()
    {
      PrimaryThumbprint = "B4172AB44C28F3B9E117648C6F7294978A00CDCBA34A46A1B8588B3F7D82C4F1"
    }
  }
};
RegistryManager registryManager = RegistryManager.CreateFromConnectionString(deviceGatewayConnectionString);
await registryManager.AddDeviceAsync(device);
```

### <a name="use-an-x509-certificate-during-run-time-operations"></a>Utiliser un certificat X.509 pendant les opérations d’exécution

[Azure IoT device SDK pour .NET](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/iothub/device) (version 1.0.11+) prend en charge l’utilisation des certificats X.509.

### <a name="c-support"></a>Prise en charge de C\#

La classe **DeviceAuthenticationWithX509Certificate** prend en charge la création d’instances **DeviceClient** à l’aide d’un certificat X.509. Le certificat X.509 doit être au format PFX (également appelé PKCS #12) qui inclut la clé privée.

Voici un exemple d’extrait de code :

```csharp
var authMethod = new DeviceAuthenticationWithX509Certificate("<device id>", x509Certificate);

var deviceClient = DeviceClient.Create("<IotHub DNS HostName>", authMethod);
```

## <a name="custom-device-and-module-authentication"></a>Authentification d’appareil et de module personnalisée

Vous pouvez utiliser le [registre des identités](iot-hub-devguide-identity-registry.md) IoT Hub pour configurer les informations d’identification de sécurité et le contrôle d’accès par appareil/module à l’aide de [jetons](iot-hub-devguide-security.md#security-tokens). Si une solution IoT a déjà un registre des identités et/ou un schéma d’authentification personnalisé, vous pouvez créer un *service de jeton* pour intégrer cette infrastructure existante à IoT Hub. De cette façon, vous pouvez utiliser d'autres fonctionnalités IoT dans votre solution.

Un service de jeton est un service cloud personnalisé. Il utilise une *stratégie d’accès partagé* IoT Hub avec des autorisations **DeviceConnect** ou **ModuleConnect** pour créer des jetons *device-scoped* ou *module-scoped*. Ces jetons permettent à un appareil et à un module de se connecter à votre hub IoT.

![Étapes du modèle de service de jeton](./media/iot-hub-devguide-security/tokenservice.png)

Voici les principales étapes du schéma de service de jeton :

1. Créez une stratégie d’accès partagé IoT Hub avec des autorisations **DeviceConnect** ou **ModuleConnect** pour votre hub IoT. Vous pouvez créer cette stratégie dans le [Portail Azure](https://portal.azure.com) ou par programme. Le service de jetons utilise cette stratégie pour signer les jetons qu'elle crée.

2. Quand un appareil/module doit accéder à votre hub IoT, il demande à votre service de jetons un jeton signé. L’appareil peut s’authentifier avec votre registre des identités personnalisé/schéma d’authentification pour déterminer l’identité d’appareil/de module que le service de jeton utilise pour créer le jeton.

3. Le service de jeton renvoie un jeton. Le jeton est créé à partir de `/devices/{deviceId}` ou `/devices/{deviceId}/module/{moduleId}` en tant que `resourceURI`, avec `deviceId` en tant qu’appareil en cours d’authentification ou `moduleId` en tant que module en cours d’authentification. Le service de jeton utilise la stratégie d'accès partagé pour construire le jeton.

4. L’appareil/le module utilise le jeton directement avec le hub IoT.

> [!NOTE]
> Vous pouvez utiliser la classe .NET [SharedAccessSignatureBuilder](/dotnet/api/microsoft.azure.devices.common.security.sharedaccesssignaturebuilder) ou la classe Java [IotHubServiceSasToken](/java/api/com.microsoft.azure.sdk.iot.service.auth.iothubservicesastoken) pour créer un jeton dans votre service de jeton.

Le service de jetons peut définir l’expiration du jeton comme vous le souhaitez. Quand le jeton expire, le hub IoT interrompt la connexion. L’appareil/le /module doit ensuite demander un nouveau jeton au service de jeton. Un délai d’expiration court accroît la charge de l’appareil/du module et du service de jeton.

Pour qu’un appareil/module se connecte à votre hub, vous devez l’ajouter au registre des identités IoT Hub, même s’il utilise un jeton et non une clé pour se connecter. Ainsi, vous pouvez continuer d’utiliser le contrôle d’accès par appareil/par module en activant ou désactivant les identités des appareils/modules dans le [registre des identités](iot-hub-devguide-identity-registry.md). Cette approche réduit les risques liés à l'utilisation de jetons avec des délais d'expiration longs.

### <a name="comparison-with-a-custom-gateway"></a>Comparaison avec une passerelle personnalisée

Le modèle de service de jeton est la méthode recommandée pour implémenter un schéma de registre d’identité/d’authentification avec IoT Hub. Ce modèle est recommandé car il laisse IoT Hub gérer la plupart du trafic de la solution. Toutefois, si l’entrelacement entre le schéma d’authentification personnalisé et le protocole SSL est conséquent, vous aurez peut-être besoin d’une *passerelle personnalisée* pour traiter tout le trafic. Le [protocole TLS (Transport Layer Security) et les clés prépartagées (PSK)](https://tools.ietf.org/html/rfc4279) en sont un exemple. Pour plus d’informations, consultez l’article [Prise en charge de protocoles supplémentaires pour IoT Hub](iot-hub-protocol-gateway.md).

## <a name="reference-topics"></a>Rubriques de référence :

Les rubriques de référence suivantes vous fournissent des informations supplémentaires sur le contrôle de l’accès à votre hub IoT.

## <a name="iot-hub-permissions"></a>Autorisations IoT Hub

Le tableau suivant répertorie les autorisations que vous pouvez utiliser pour contrôler l’accès à votre IoT Hub.

| Autorisation | Notes |
| --- | --- |
| **RegistryRead**. |Accorde l’accès en lecture au registre des identités. Pour plus d’informations, consultez [Registre des identités](iot-hub-devguide-identity-registry.md). <br/>Cette autorisation est utilisée par les services cloud principaux. |
| **RegistryReadWrite**. |Accorde l’accès en lecture et en écriture au registre des identités. Pour plus d’informations, consultez [Registre des identités](iot-hub-devguide-identity-registry.md). <br/>Cette autorisation est utilisée par les services cloud principaux. |
| **ServiceConnect**. |Accorde l’accès à la communication de services cloud et à la surveillance des points de terminaison. <br/>Accorde l’autorisation de recevoir des messages appareil-à-cloud, d’envoyer des messages cloud-à-appareil et de récupérer les accusés de remise correspondants. <br/>Accorde l’autorisation de récupérer les accusés de remise pour les chargements de fichiers. <br/>Accorde l’autorisation d’accéder aux jumeaux pour mettre à jour les balises et les propriétés voulues, de récupérer les propriétés signalées et d’exécuter des requêtes. <br/>Cette autorisation est utilisée par les services cloud principaux. |
| **DeviceConnect**. |Accorde l’accès aux points de terminaison côté appareil. <br/>Accorde l’autorisation d’envoyer des messages appareil-à-cloud et de recevoir des messages cloud-à-appareil. <br/>Accorde l’autorisation d’effectuer un chargement de fichiers à partir d’un appareil. <br/>Accorde l’autorisation de recevoir des notifications de propriétés voulues pour les jumeaux d’appareil et de mettre à jour les propriétés signalées pour les jumeaux d’appareil. <br/>Accorde l’autorisation d’effectuer des chargements de fichiers. <br/>Cette autorisation est utilisée par les appareils. |

## <a name="additional-reference-material"></a>Matériel de référence supplémentaire

Les autres rubriques de référence dans le Guide du développeur IoT Hub comprennent :

* La rubrique [Points de terminaison IoT Hub](iot-hub-devguide-endpoints.md), qui décrit les différents points de terminaison que chaque hub IoT expose pour les opérations d’exécution et de gestion.

* La rubrique [Quotas et limitation IoT Hub](iot-hub-devguide-quotas-throttling.md) décrit les quotas et le comportement de limitation qui s’appliquent au service IoT Hub.

* La rubrique [SDK des services et appareils Azure IoT](iot-hub-devguide-sdks.md), qui répertorie les SDK en différents langages que vous pouvez utiliser pour le développement d’applications d’appareil et de service qui interagissent avec IoT Hub.

* La rubrique [Langage de requête IoT Hub](iot-hub-devguide-query-language.md) décrit le langage de requête permettant de récupérer à partir d’IoT Hub des informations sur les jumeaux d’appareil et les travaux.

* La rubrique [Prise en charge de MQTT au niveau d’IoT Hub](iot-hub-mqtt-support.md), qui fournit des informations supplémentaires sur la prise en charge du protocole MQTT par IoT Hub.

* [RFC 5246 – Le protocole TLS version 1.2](https://tools.ietf.org/html/rfc5246/) fournit plus d’informations sur l’authentification TLS.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment contrôler l’accès à IoT Hub, vous serez peut-être intéressé par les rubriques suivantes du Guide du développeur IoT Hub :

* [Utiliser des jumeaux d’appareil pour synchroniser les données d’état et de configuration](iot-hub-devguide-device-twins.md)
* [Appeler une méthode directe sur un appareil](iot-hub-devguide-direct-methods.md)
* [Planifier des travaux sur plusieurs appareils](iot-hub-devguide-jobs.md)

Si vous souhaitez tenter de mettre en pratique certains des concepts décrits dans cet article, consultez les didacticiels IoT Hub suivants :

* [Mise en route de Azure IoT Hub](quickstart-send-telemetry-node.md)
* [Envoyer des messages du cloud à votre appareil avec IoT Hub](iot-hub-csharp-csharp-c2d.md)
* [Configurer le routage des messages avec IoT Hub](tutorial-routing.md)
