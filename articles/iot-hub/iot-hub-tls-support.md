---
title: Prise en charge du protocole TLS dans Azure IoT Hub
description: En savoir plus sur l’utilisation de connexions TLS sécurisées pour les appareils et les services communiquant avec IoT Hub
services: iot-hub
author: jlian
ms.service: iot-fundamentals
ms.topic: conceptual
ms.date: 11/25/2020
ms.author: jlian
ms.openlocfilehash: f4438aebcb81d665a19a595ac7ade4fea27fc43f
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96621006"
---
# <a name="transport-layer-security-tls-support-in-iot-hub"></a>Prise en charge de TLS (Transport Layer Security) dans IoT Hub

IoT Hub utilise le protocole TLS (Transport Layer Security) pour sécuriser les connexions provenant d’appareils et de services IoT. Trois versions du protocole TLS sont actuellement prises en charge, à savoir les versions 1.0, 1.1 et 1.2.

Les protocoles TLS 1.0 et 1.1 sont considérés comme hérités et leur obsolescence est planifiée. Pour plus d’informations, consultez [Dépréciation de TLS 1.0 et 1.1 pour IoT Hub](iot-hub-tls-deprecating-1-0-and-1-1.md). Pour éviter tout problème futur, utilisez TLS 1.2 comme seule version TLS lors de la connexion à IoT Hub.

## <a name="iot-hubs-server-tls-certificate"></a>Certificat TLS du serveur d’IoT Hub

Au cours de l’établissement d’une liaison TLS, IoT Hub présente des certificats de serveur à clé RSA pour la connexion des clients. Sa racine est l’autorité de certification racine Baltimore Cybertrust. Récemment, nous avons apporté une modification à notre certificat de serveur TLS, de sorte qu’il est désormais délivré par de nouvelles autorités de certification intermédiaires (ICA). Pour plus d’informations, consultez [Mise à jour des certificats TLS pour IoT Hub](https://azure.microsoft.com/updates/iot-hub-tls-certificate-update/).

### <a name="elliptic-curve-cryptography-ecc-server-tls-certificate-preview"></a>Certificat TLS (version préliminaire) du serveur ECC (Elliptic Curve Cryptography)

Le certificat TLS du serveur ECC d’IoT Hub est disponible en préversion publique. Tout en offrant une sécurité similaire aux certificats RSA, la validation des certificats ECC (avec les suites de chiffrement exclusivement ECC) utilise jusqu’à 40 % moins de calcul, de mémoire et de bande passante. Ces économies sont importantes pour les appareils IoT en raison de leur profil et de leur mémoire plus petits, et pour prendre en charge les cas d’usage dans des environnements où la bande passante est limitée. 

Pour afficher un aperçu du certificat de serveur ECC d’IoT Hub :

1. [Créez un hub IoT avec le mode aperçu activé](iot-hub-preview-mode.md).
1. [Configurez votre client](#tls-configuration-for-sdk-and-iot-edge) pour qu’il inclue *uniquement* des suites de chiffrement ECDSA et *exclue* toute suite RSA. Les suites de chiffrement prises en charge pour la préversion publique du certificat ECC sont les suivantes :
    - `TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256`
    - `TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384`
    - `TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256`
    - `TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384`
1. Connectez votre client à la préversion d’IoT Hub.

## <a name="tls-12-enforcement-available-in-select-regions"></a>TLS 1.2 n’est disponible que dans certaines régions

Pour renforcer la sécurité, configurez vos hubs IoT pour autoriser *uniquement* les connexions client qui utilisent la version 1.2 du protocole TLS et pour appliquer l’utilisation des [suites de chiffrement](#cipher-suites). Cette fonctionnalité n'est prise en charge que dans ces régions :

* USA Est
* États-Unis - partie centrale méridionale
* USA Ouest 2
* Gouvernement des États-Unis – Arizona
* Gouvernement américain - Virginie

À cet effet, approvisionnez un nouveau hub IoT dans l’une des régions prises en charge et `minTlsVersion`définissez la propriété`1.2` sur dans la spécification de ressource IoT Hub de votre modèle Azure Resource Manager :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.Devices/IotHubs",
            "apiVersion": "2020-01-01",
            "name": "<provide-a-valid-resource-name>",
            "location": "<any-of-supported-regions-below>",
            "properties": {
                "minTlsVersion": "1.2"
            },
            "sku": {
                "name": "<your-hubs-SKU-name>",
                "tier": "<your-hubs-SKU-tier>",
                "capacity": 1
            }
        }
    ]
}
```

La ressource IoT Hub créée à l’aide de cette configuration refusera les clients d’appareil et de service qui tentent de se connecter à l’aide des versions 1.0 et 1.1 du protocole TLS. De même, la négociation TLS est refusée si le message `ClientHello` ne liste aucun des [chiffrements recommandés](#cipher-suites).

> [!NOTE]
> La propriété `minTlsVersion` est en lecture seule et ne peut pas être modifiée une fois votre ressource IoT Hub créée. C’est pourquoi il est essentiel de tester et de valider correctement que *tous* vos appareils et services IoT sont compatibles avec le protocole TLS 1.2 et les [chiffrements recommandés](#cipher-suites) au préalable.
> 
> Lors des basculements, la propriété `minTlsVersion` de votre IoT Hub reste effective dans la région jumelée géographiquement après le basculement.

## <a name="cipher-suites"></a>Suites de chiffrement

Les hubs IoT configurés pour accepter uniquement TLS 1.2 appliquent également l’utilisation des suites de chiffrement recommandées suivantes :

* `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`
* `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`
* `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256`
* `TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384`

Pour les hubs IoT non configurés pour l’application de TLS 1.2, ce protocole fonctionne quand même avec les suites de chiffrement suivantes :

* `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256`
* `TLS_DHE_RSA_WITH_AES_256_GCM_SHA384`
* `TLS_DHE_RSA_WITH_AES_128_GCM_SHA256`
* `TLS_RSA_WITH_AES_256_GCM_SHA384`
* `TLS_RSA_WITH_AES_128_GCM_SHA256`
* `TLS_RSA_WITH_AES_256_CBC_SHA256`
* `TLS_RSA_WITH_AES_128_CBC_SHA256`
* `TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA`
* `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA`
* `TLS_RSA_WITH_AES_256_CBC_SHA`
* `TLS_RSA_WITH_AES_128_CBC_SHA`
* `TLS_RSA_WITH_3DES_EDE_CBC_SHA`

Un client peut suggérer une liste de suites de chiffrement plus élevées à utiliser pendant `ClientHello`. Toutefois, certaines d’entre elles peuvent ne pas être prises en charge par IoT Hub (par exemple, `ECDHE-ECDSA-AES256-GCM-SHA384`). Dans ce cas, IoT Hub tente de suivre la préférence du client, mais finit par négocier seulement la suite de chiffrement avec `ServerHello`.

## <a name="tls-configuration-for-sdk-and-iot-edge"></a>Configuration TLS pour le Kit de développement logiciel (SDK) et IoT Edge

Utilisez les liens ci-dessous pour configurer TLS 1.2 et les chiffrements autorisés dans les Kits de développement logiciel (SDK) clients IoT Hub.

| Langage | Versions prenant en charge TLS 1.2 | Documentation |
|----------|------------------------------------|---------------|
| C        | Balise 2019-12-11 ou ultérieure            | [Lien](https://aka.ms/Tls_C_SDK_IoT) |
| Python   | Version 2.0.0 ou ultérieure             | [Lien](https://aka.ms/Tls_Python_SDK_IoT) |
| C#       | Version 1.21.4 ou ultérieure            | [Lien](https://aka.ms/Tls_CSharp_SDK_IoT) |
| Java     | Version 1.19.0 ou ultérieure            | [Lien](https://aka.ms/Tls_Java_SDK_IoT) |
| NodeJS   | Version 1.12.2 ou ultérieure            | [Lien](https://aka.ms/Tls_Node_SDK_IoT) |

Les appareils IoT Edge peuvent être configurés pour utiliser TLS 1.2 lors des communications avec IoT Hub. À cet effet, consultez la [page de documentation d’IoT Edge](https://github.com/Azure/iotedge/blob/master/edge-modules/edgehub-proxy/README.md).

## <a name="device-authentication"></a>Authentification des appareils

Après l’établissement de la liaison TLS, IoT Hub peut authentifier un appareil à l’aide d’une clé symétrique ou d’un certificat X.509. Pour l’authentification par certificat, il peut s’agir de n’importe quel certificat X.509, y compris ECC. IoT Hub valide le certificat par rapport à l’empreinte ou à l’autorité de certification (CA) que vous fournissez. Pour plus d’informations, consultez [Certificats X.509 pris en charge](iot-hub-devguide-security.md#supported-x509-certificates).

## <a name="tls-maximum-fragment-length-negotiation-preview"></a>Négociation de la longueur maximale des fragments TLS (préversion)

IoT Hub prend également en charge la négociation de longueur maximale des fragments TLS, parfois appelée négociation de la taille de images TLS. Cette fonctionnalité est en version préliminaire publique. 

Utilisez cette fonctionnalité pour spécifier la longueur maximale du fragment de texte en clair sur une valeur inférieure à 2^14 octets par défaut. Une fois la négociation terminée, IoT Hub et le client commencent à fragmenter les messages pour s’assurer que tous les fragments sont plus petits que la longueur négociée. Ce comportement est utile pour le calcul ou les appareils ayant une mémoire restreinte. Pour plus d’informations, consultez la [spécification officielle de l’extension TLS](https://tools.ietf.org/html/rfc6066#section-4).

La prise en charge officielle du Kit de développement logiciel (SDK) pour cette fonctionnalité d’évaluation publique n’est pas encore disponible. Pour commencer

1. [Créez un hub IoT avec le mode aperçu activé](iot-hub-preview-mode.md).
1. Quand vous utilisez OpenSSL, appelez [SSL_CTX_set_tlsext_max_fragment_length](https://manpages.debian.org/testing/libssl-doc/SSL_CTX_set_max_send_fragment.3ssl.en.html) pour spécifier la taille du fragment.
1. Connectez votre client à la préversion d’IoT Hub.

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur la sécurité et le contrôle d’accès d’IoT Hub, consultez [Contrôler l’accès à IoT Hub](iot-hub-devguide-security.md).
- Pour en savoir plus sur l’utilisation du certificat X.509 pour l’authentification des appareils, consultez [Authentification des appareils à l’aide de certificats d’autorité de certification X.509](iot-hub-x509ca-overview.md).