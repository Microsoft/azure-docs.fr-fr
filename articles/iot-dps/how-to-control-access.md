---
title: Points de terminaison de sécurité du service de provisionnement d’appareils IoT | Microsoft Docs
description: 'Concepts : comment contrôler l’accès au service IoT Device Provisioning Service (DPS) pour les applications back-end. Inclut des informations sur les jetons de sécurité.'
author: wesmc7777
manager: philmea
ms.service: iot-dps
services: iot-dps
ms.topic: conceptual
ms.date: 04/09/2019
ms.author: wesmc
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 024dbf6518748a4048873de4eb54a53f9d9a6362
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94954322"
---
# <a name="control-access-to-azure-iot-hub-device-provisioning-service"></a>Contrôler l’accès au service de provisionnement d’appareils Azure IoT Hub

Cet article décrit les options permettant de sécuriser votre service de provisionnement des appareils IoT. Le service de provisionnement utilise des *autorisations* pour accorder l’accès à chaque point de terminaison. Les autorisations limitent l’accès à une instance de service en fonction des fonctionnalités.

Cet article aborde les points suivants :

* Les différentes autorisations que vous pouvez accorder à une application principale pour lui permettre d’accéder à votre service de provisionnement.
* Le processus d’authentification et les jetons qu’il utilise pour vérifier les autorisations.

### <a name="when-to-use"></a>Quand l’utiliser

Vous devez disposer des autorisations appropriées pour accéder à l’un des points de terminaison de service de provisionnement. Par exemple, une application principale doit ajouter un jeton contenant des informations d’identification de sécurité à chaque message qu’elle envoie au service.

## <a name="access-control-and-permissions"></a>Contrôle d’accès et autorisations

Vous pouvez accorder des [autorisations](#device-provisioning-service-permissions) de différentes manières :

* **Stratégies d’autorisation d’accès partagé**. Les stratégies d’accès partagé peuvent accorder n’importe quelle combinaison d’[autorisations](#device-provisioning-service-permissions). Vous pouvez définir des stratégies dans le [portail Azure][lnk-management-portal], ou programmatiquement avec des [API REST du service Device Provisioning][lnk-resource-provider-apis]. Tout service de provisionnement créé comporte la stratégie par défaut suivante :

* **provisioningserviceowner** : stratégie avec toutes les autorisations.

> [!NOTE]
> Pour plus d’informations, consultez la page [Autorisations](#device-provisioning-service-permissions).

## <a name="authentication"></a>Authentication

Le service de provisionnement d’appareils Azure IoT Hub accorde l’accès aux points de terminaison en vérifiant la validité d’un jeton par rapport aux stratégies d’accès partagé. Les informations d’identification de sécurité telles que les clés symétriques ne sont jamais envoyées sur le réseau.

> [!NOTE]
> Le fournisseur de ressources du service Device Provisioning est sécurisé par le biais de votre abonnement Azure, comme le sont tous les fournisseurs dans [Azure Resource Manager][lnk-azure-resource-manager].

Pour plus d’informations sur la façon de construire et d’utiliser les jetons de sécurité, consultez la section suivante.

HTTP est le seul protocole pris en charge. Il implémente l’authentification en incluant un jeton valide dans l’en-tête de demande d’**autorisation**.

#### <a name="example"></a>Exemple
```csharp
SharedAccessSignature sr = 
   mydps.azure-devices-provisioning.net&sig=kPszxZZZZZZZZZZZZZZZZZAhLT%2bV7o%3d&se=1487709501&skn=provisioningserviceowner`\
```

> [!NOTE]
> Les [kits SDK du service Azure IoT Device Provisioning][lnk-sdks] génèrent automatiquement des jetons durant la connexion au service.

## <a name="security-tokens"></a>Jetons de sécurité

Le service de provisionnement d’appareils utilise des jetons de sécurité pour authentifier les services et éviter l’envoi de clés. En outre, la validité et la portée des jetons sont limitées dans le temps. Les [kits SDK du service Azure IoT Device Provisioning][lnk-sdks] génèrent automatiquement des jetons sans nécessiter de configuration particulière. Certains scénarios nécessitent toutefois que vous génériez et utilisiez directement des jetons de sécurité. Ces scénarios incluent l’utilisation directe du protocole HTTP.

### <a name="security-token-structure"></a>Structure du jeton de sécurité

Vous utilisez des jetons de sécurité pour permettre aux services d’accéder de manière limitée dans le temps à des caractéristiques spécifiques du service de provisionnement d’appareils IoT. Pour obtenir l’autorisation de se connecter au service de provisionnement, les services doivent envoyer des jetons de sécurité signés avec une clé d’accès partagé ou une clé symétrique.

Un jeton signé avec une clé d’accès partagé accorde un accès à toutes les fonctionnalités associées aux autorisations de stratégie d’accès partagé. 

Le jeton de sécurité présente le format suivant :

`SharedAccessSignature sig={signature}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}`

Voici les valeurs attendues :

| Valeur | Description |
| --- | --- |
| {signature} |Une chaîne de signature HMAC-SHA256 sous la forme : `{URL-encoded-resourceURI} + "\n" + expiry`. **Important**: la clé est décodée à partir de base64 et utilisée comme clé pour effectuer le calcul HMAC-SHA256.|
| {expiry} |Chaînes UTF8 pour le nombre de secondes depuis l’époque 00:00:00 UTC 1er janvier 1970. |
| {URL-encoded-resourceURI} | Encodage de l’URL en minuscules à partir de l’URI de ressource en minuscules. Préfixe URI (par segment) des points de terminaison accessibles avec ce jeton, en commençant par le nom d’hôte du service de provisionnement d’appareils IoT (sans protocole). Par exemple : `mydps.azure-devices-provisioning.net`. |
| {policyName} |Le nom de la stratégie d’accès partagé à laquelle ce jeton fait référence. |

**Remarque sur le préfixe**: le préfixe URI est calculé par segment et non par caractère. Par exemple `/a/b` est un préfixe de `/a/b/c`, mais pas de `/a/bc`.

L’extrait de code Node.js suivant illustre une fonction appelée **generateSasToken** qui calcule le jeton à partir des entrées `resourceUri, signingKey, policyName, expiresInMins`. Les sections suivantes décrivent en détail comment initialiser les différentes entrées pour les différents cas d’utilisation des jetons.

```javascript
var generateSasToken = function(resourceUri, signingKey, policyName, expiresInMins) {
    resourceUri = encodeURIComponent(resourceUri);

    // Set expiration in seconds
    var expires = (Date.now() / 1000) + expiresInMins * 60;
    expires = Math.ceil(expires);
    var toSign = resourceUri + '\n' + expires;

    // Use crypto
    var hmac = crypto.createHmac('sha256', new Buffer(signingKey, 'base64'));
    hmac.update(toSign);
    var base64UriEncoded = encodeURIComponent(hmac.digest('base64'));

    // Construct authorization string
    var token = "SharedAccessSignature sr=" + resourceUri + "&sig="
    + base64UriEncoded + "&se=" + expires + "&skn="+ policyName;
    return token;
};
```

À titre de comparaison, le code Python équivalent pour générer un jeton de sécurité est :

```python
from base64 import b64encode, b64decode
from hashlib import sha256
from time import time
from urllib import quote_plus, urlencode
from hmac import HMAC

def generate_sas_token(uri, key, policy_name, expiry=3600):
    ttl = time() + expiry
    sign_key = "%s\n%d" % ((quote_plus(uri)), int(ttl))
    print sign_key
    signature = b64encode(HMAC(b64decode(key), sign_key, sha256).digest())

    rawtoken = {
        'sr' :  uri,
        'sig': signature,
        'se' : str(int(ttl)),
        'skn' : policy_name
    }

    return 'SharedAccessSignature ' + urlencode(rawtoken)
```

> [!NOTE]
> Dans la mesure où la validité temporelle du jeton est vérifiée sur les machines du service de provisionnement d’appareils IoT, la dérive de l’horloge de la machine qui génère le jeton doit être minimale.

### <a name="use-security-tokens-from-service-components"></a>Utilisation de jetons de sécurité de composants du service

Les composants de service peuvent uniquement créer des jetons de sécurité utilisant des stratégies d’accès partagé pour accorder les autorisations adaptées, comme expliqué précédemment.

Voici les fonctions de service exposées sur les points de terminaison :

| Point de terminaison | Fonctionnalités |
| --- | --- |
| `{your-service}.azure-devices-provisioning.net/enrollments` |Fournit les opérations relatives à l’inscription d’appareils auprès du service de provisionnement d’appareils. |
| `{your-service}.azure-devices-provisioning.net/enrollmentGroups` |Fournit les opérations relatives à la gestion des groupes d’inscription d’appareils. |
| `{your-service}.azure-devices-provisioning.net/registrations/{id}` |Fournit les opérations relatives à la récupération et à la gestion de l’état des inscriptions d’appareils. |


Par exemple, un service généré à l’aide d’une stratégie d’accès partagé précréée appelée **enrollmentread** crée un jeton avec les paramètres suivants :

* URI de ressource : `{mydps}.azure-devices-provisioning.net`,
* clé de signature : une des clés de la stratégie `enrollmentread` ,
* nom de la stratégie : `enrollmentread`,
* n’importe quel délai d’expiration time.backn

![Créer une stratégie d’accès partagé pour votre instance de service de provisionnement des appareils sur le portail][img-add-shared-access-policy]

```javascript
var endpoint ="mydps.azure-devices-provisioning.net";
var policyName = 'enrollmentread'; 
var policyKey = '...';

var token = generateSasToken(endpoint, policyKey, policyName, 60);
```

Le résultat, qui revient à accorder l’accès en lecture à tous les dossiers d’inscription, est le suivant :

`SharedAccessSignature sr=mydps.azure-devices-provisioning.net&sig=JdyscqTpXdEJs49elIUCcohw2DlFDR3zfH5KqGJo4r4%3D&se=1456973447&skn=enrollmentread`

## <a name="reference-topics"></a>Rubriques de référence :

Les rubriques de référence suivantes fournissent des informations supplémentaires sur le contrôle de l’accès à votre service de provisionnement d’appareils IoT.

### <a name="device-provisioning-service-permissions"></a>Autorisations liées au service de provisionnement d’appareils

Le tableau suivant répertorie les autorisations qui vous permettent de contrôler l’accès à votre service de provisionnement d’appareils IoT.

| Autorisation | Notes |
| --- | --- |
| **ServiceConfig** |Accorde l’accès pour changer les configurations du service. <br/>Cette autorisation est utilisée par les services cloud principaux. |
| **EnrollmentRead** |Accorde l’accès en lecture aux inscriptions d’appareils et aux groupes d’inscription. <br/>Cette autorisation est utilisée par les services cloud principaux. |
| **EnrollmentWrite** |Accorde l’accès en écriture aux inscriptions d’appareils et aux groupes d’inscription. <br/>Cette autorisation est utilisée par les services cloud principaux. |
| **RegistrationStatusRead** |Accorde l’accès en lecture à l’état d’inscription de l’appareil. <br/>Cette autorisation est utilisée par les services cloud principaux. |
| **RegistrationStatusWrite**  |Accorde l’accès permettant de supprimer l’état d’inscription de l’appareil. <br/>Cette autorisation est utilisée par les services cloud principaux. |

<!-- links and images -->

[img-add-shared-access-policy]: ./media/how-to-control-access/how-to-add-shared-access-policy.PNG
[lnk-sdks]: ../iot-hub/iot-hub-devguide-sdks.md
[lnk-management-portal]: https://portal.azure.com
[lnk-azure-resource-manager]: ../azure-resource-manager/management/overview.md
[lnk-resource-provider-apis]: /rest/api/iot-dps/