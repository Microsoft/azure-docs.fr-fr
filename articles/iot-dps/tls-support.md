---
title: Prise en charge de TLS dans le service Azure IoT Device Provisioning (DPS)
description: Meilleures pratiques en matière d’utilisation de connexions TLS sécurisées pour les appareils et les services communiquant avec le service IoT Device Provisioning (DPS)
services: iot-dps
author: wesmc7777
ms.service: iot-dps
ms.topic: conceptual
ms.date: 09/14/2020
ms.author: wesmc
ms.openlocfilehash: 3a8910cf0e81bd041d74ef95f45220f1c1e0b34c
ms.sourcegitcommit: 6a4687b86b7aabaeb6aacdfa6c2a1229073254de
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/06/2020
ms.locfileid: "91761292"
---
# <a name="tls-support-in-azure-iot-hub-device-provisioning-service-dps"></a>Prise en charge de TLS dans le service Azure IoT Hub Device Provisioning (DPS)

DPS utilise le protocole [TLS (Transport Layer Security)](http://wikipedia.org/wiki/Transport_Layer_Security) pour sécuriser les connexions provenant d’appareils IoT. 

Les versions actuelles du protocole TLS prises en charge par DPS sont les suivantes : 
* TLS 1.2

Les protocoles TLS 1.0 et 1.1 sont considérés comme hérités et leur obsolescence est planifiée. Pour plus d’informations, consultez [Dépréciation de TLS 1.0 et 1.1 pour IoT Hub](../iot-hub/iot-hub-tls-deprecating-1-0-and-1-1.md). 

## <a name="restrict-connections-to-tls-12"></a>Restreindre les connexions à TLS 1.2

Pour renforcer la sécurité, il est recommandé de configurer vos instances DPS pour autoriser *uniquement* les connexions d’appareils clients qui utilisent la version 1.2 du protocole TLS et pour appliquer l’utilisation des [chiffrements recommandés](#recommended-ciphers).

Pour ce faire, provisionnez une nouvelle ressource DPS et affectez à la propriété `minTlsVersion` la valeur `1.2` dans la spécification de ressource DPS de votre modèle Azure Resource Manager. L’exemple de modèle JSON suivant spécifie la propriété `minTlsVersion` pour une nouvelle instance DPS.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.Devices/ProvisioningServices",
            "apiVersion": "2020-01-01",
            "name": "<provide-a-valid-DPS-resource-name>",
            "location": "<any-region>",
            "properties": {
                "minTlsVersion": "1.2"
            },
            "sku": {
                "name": "S1",
                "capacity": 1
            },
        }     
    ]
}
```

Vous pouvez déployer ce modèle en exécutant les commandes Azure CLI suivantes : 

```azurecli
az deployment group create -g <your resource group name> --template-file template.json
```

Pour plus d’informations sur la création de ressources DPS avec des modèles Resource Manager, consultez l’article [Configurer DPS avec un modèle Azure Resource Manager](quick-setup-auto-provision-rm.md).

La ressource DPS créée à l’aide de cette configuration refusera les appareils qui tentent de se connecter à l’aide des versions 1.0 et 1.1 du protocole TLS. De même, la négociation TLS est refusée si le message HELLO de l’appareil client ne répertorie pas les [chiffrements recommandés](#recommended-ciphers).

> [!NOTE]
> La propriété `minTlsVersion` est en lecture seule et ne peut pas être modifiée une fois votre ressource DPS créée. C’est pourquoi il est essentiel de tester et de valider correctement que *tous* vos appareils IoT sont compatibles avec le protocole TLS 1.2 et les [chiffrements recommandés](#recommended-ciphers) au préalable.


> [!NOTE]
> Lors des basculements, la propriété `minTlsVersion` de votre DPS reste effective dans la région jumelée géographiquement après le basculement.

## <a name="recommended-ciphers"></a>Chiffrements recommandés

Les instances de DPS configurées pour accepter uniquement le protocole TLS 1.2 imposent également l’utilisation des suites de chiffrement suivantes :


| Suites de chiffrement TLS 1.2 recommandées |
| :--- |
| `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`<br>`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`<br>`TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384`<br>`TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256` |


### <a name="legacy-cipher-suites"></a>Suites de chiffrement héritées 

Ces suites de chiffrement sont actuellement encore prises en charge par DPS, mais seront bientôt déconseillées. Utilisez les suites de chiffrement recommandées ci-dessus dans la mesure du possible.

| Option n° 1 (meilleure sécurité) |
| :--- |
| `TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA_P384   (uses SHA-1)`<br>`TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA_P256   (uses SHA-1)`<br>`TLS_RSA_WITH_AES_256_GCM_SHA384           (lack of Perfect Forward Secrecy)`<br>`TLS_RSA_WITH_AES_128_GCM_SHA256           (lack of Perfect Forward Secrecy)`<br>`TLS_RSA_WITH_AES_256_CBC_SHA256           (lack of Perfect Forward Secrecy)`<br>`TLS_RSA_WITH_AES_128_CBC_SHA256           (lack of Perfect Forward Secrecy)`<br>`TLS_RSA_WITH_AES_256_CBC_SHA              (uses SHA-1, lack of Perfect Forward Secrecy)`<br>`TLS_RSA_WITH_AES_128_CBC_SHA              (uses SHA-1, lack of Perfect Forward Secrecy)` |

| Option n° 2 (meilleur niveau de performance) |
| :--- |
| `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA_P256   (uses SHA-1)`<br>`TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA_P384   (uses SHA-1)`<br>`TLS_RSA_WITH_AES_128_GCM_SHA256           (lack of Perfect Forward Secrecy)`<br>`TLS_RSA_WITH_AES_256_GCM_SHA384           (lack of Perfect Forward Secrecy)`<br>`TLS_RSA_WITH_AES_128_CBC_SHA256           (lack of Perfect Forward Secrecy)`<br>`TLS_RSA_WITH_AES_256_CBC_SHA256           (lack of Perfect Forward Secrecy)`<br>`TLS_RSA_WITH_AES_128_CBC_SHA              (uses SHA-1, lack of Perfect Forward Secrecy)`<br>`TLS_RSA_WITH_AES_256_CBC_SHA              (uses SHA-1, lack of Perfect Forward Secrecy)` |


## <a name="use-tls-12-in-the-iot-sdks"></a>Utiliser TLS 1.2 dans les kits de développement logiciel (SDK) IoT

Utilisez les liens ci-dessous pour configurer TLS 1.2 et les chiffrements autorisés dans les Kits de développement logiciel (SDK) clients Azure IoT.

| Langage | Versions prenant en charge TLS 1.2 | Documentation |
|----------|------------------------------------|---------------|
| C        | Balise 2019-12-11 ou ultérieure            | [Lien](https://aka.ms/Tls_C_SDK_IoT) |
| Python   | Version 2.0.0 ou ultérieure             | [Lien](https://aka.ms/Tls_Python_SDK_IoT) |
| C#       | Version 1.21.4 ou ultérieure            | [Lien](https://aka.ms/Tls_CSharp_SDK_IoT) |
| Java     | Version 1.19.0 ou ultérieure            | [Lien](https://aka.ms/Tls_Java_SDK_IoT) |
| NodeJS   | Version 1.12.2 ou ultérieure            | [Lien](https://aka.ms/Tls_Node_SDK_IoT) |

## <a name="use-tls-12-with-iot-hub"></a>Utiliser TLS 1.2 avec IoT Hub

Vous pouvez configurer IoT Hub pour utiliser TLS 1.2 durant la communication avec les appareils. Pour plus d’informations, consultez [Dépréciation de TLS 1.0 et 1.1 pour IoT Hub](../iot-hub/iot-hub-tls-deprecating-1-0-and-1-1.md).

## <a name="use-tls-12-with-iot-edge"></a>Utiliser TLS 1.2 avec IoT Edge

Les appareils IoT Edge peuvent être configurés pour utiliser TLS 1.2 lors des communications avec IoT Hub et DPS. Pour plus d’informations, consultez la [page de documentation IoT Edge](https://github.com/Azure/iotedge/blob/master/edge-modules/edgehub-proxy/README.md).
