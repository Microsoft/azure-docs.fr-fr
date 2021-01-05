---
title: Utiliser différents mécanismes d’attestation avec le Kit de développement logiciel (SDK) client du service IoT Hub Device Provisioning Azure
description: 'Guide pratique Azure : Utilisation de différents mécanismes d’attestation avec le Kit de développement logiciel (SDK) client Device Provisioning Service (DPS) dans Azure'
author: wesmc7777
ms.author: wesmc
ms.date: 03/30/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
ms.custom:
- mvc
- amqp
ms.openlocfilehash: 0a32e2f055b2914fa0008e043e80092ac2da0814
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90531506"
---
# <a name="how-to-use-different-attestation-mechanisms-with-device-provisioning-service-client-sdk-for-c"></a>Utiliser différents mécanismes d’attestation avec le Kit de développement logiciel (SDK) Device Provisioning Service Client pour C

Cet article explique comment utiliser différents [mécanismes d’attestation](concepts-service.md#attestation-mechanism) avec le Kit de développement logiciel (SDK) Device Provisioning Service Client pour C. Vous pouvez choisir d’utiliser un appareil physique ou un simulateur. Le service de provisionnement prend en charge l’authentification pour deux types de mécanismes d’attestation : X.509 et Module de plateforme sécurisée (TPM).

## <a name="prerequisites"></a>Prérequis

Préparez votre environnement de développement en suivant la section intitulée « Préparer l’environnement de développement » du guide [Créer et approvisionner un appareil TPM simulé à l’aide du kit de développement logiciel C de périphérique pour le service IoT Hub Device Provisioning](./quick-create-simulated-device.md).

### <a name="choose-an-attestation-mechanism"></a>Choisir un mécanisme d’attestation

En tant que fabricant d’appareils, vous devez d’abord choisir un mécanisme d’attestation basé sur l’un des types pris en charge. Actuellement, le [Kit de développement logiciel (SDK) Device Provisioning Service Client pour C](https://github.com/Azure/azure-iot-sdk-c/tree/master/provisioning_client) prend en charge les mécanismes d’attestation suivants : 

- [Module de plateforme sécurisée (TPM)](https://en.wikipedia.org/wiki/Trusted_Platform_Module) : Le module TPM est une norme établie pour la plupart des plateformes d’appareils Windows, ainsi que pour quelques appareils Linux/Ubuntu. En tant que fabricant d’appareils, vous pouvez choisir ce mécanisme d’attestation si l’un de ces systèmes d’exploitation s’exécute sur vos appareils et si vous recherchez une norme établie. Avec les processeurs TPM, vous pouvez uniquement inscrire chaque appareil individuellement auprès du service Device Provisioning. À des fins de développement, vous pouvez utiliser le simulateur TPM sur votre machine de développement Windows ou Linux.

- [X.509](https://cryptography.io/en/latest/x509/) : Les certificats X.509 peuvent être stockés dans des processeurs relativement plus récents appelés [Modules de sécurité matériels (HSM)](concepts-service.md#hardware-security-module). Microsoft continue de travailler sur des processeurs RIoT ou DICE, qui implémentent les certificats X.509. Avec les processeurs X.509, vous pouvez effectuer une inscription d’appareils en bloc dans le portail. En outre, certains systèmes non-Windows, comme embedOS, sont pris en charge. À des fins de développement, le SDK client du service Device Provisioning prend en charge un simulateur d’appareil X.509. 

Pour plus d’informations, consultez les [mécanismes d’attestation](concepts-service.md#attestation-mechanism) du Service IoT Hub Device Provisioning.

## <a name="enable-authentication-for-supported-attestation-mechanisms"></a>Activer l’authentification pour les mécanismes d’attestation pris en charge

Le mode d’authentification du Kit de développement logiciel (SDK) (X.509 ou TPM) doit être activé pour le simulateur ou l’appareil physique avant qu’ils puissent être inscrits dans le portail Azure. Commencez par accéder au dossier racine pour azure-iot-sdk-c. Exécutez ensuite la commande spécifiée selon le mode d’authentification que vous choisissez :

### <a name="use-x509-with-simulator"></a>Utiliser X.509 avec le simulateur

Le service d’approvisionnement est fourni avec un émulateur de moteur de composition d’identité d’appareil (DICE) qui génère un certificat X **.** 509 pour l’authentification de l’appareil. Exécutez la commande suivante pour activer l’authentification **X.509** : 

```
cmake -Ddps_auth_type=x509 ..
```

Vous pouvez trouver des informations sur le matériel avec le moteur de composition d’identité d’appareil [ici](https://azure.microsoft.com/blog/azure-iot-supports-new-security-hardware-to-strengthen-iot-security/).

### <a name="use-x509-with-hardware"></a>Utiliser X.509 avec le matériel

Le service d’approvisionnement peut être utilisé avec **X.509** sur un autre matériel. Une interface entre le matériel et le kit de développement logiciel (SDK) est nécessaire pour établir la connexion. Contactez le fabricant de votre HSM pour plus d’informations sur l’interface.

### <a name="use-tpm"></a>Utiliser TPM

Le service d’approvisionnement peut se connecter aux processeurs TPM de matériels Windows et Linux avec un jeton SAP. Exécutez la commande suivante pour activer l’authentification TPM :

```
cmake -Ddps_auth_type=tpm ..
```

### <a name="use-tpm-with-simulator"></a>Utiliser TPM avec un simulateur

Si vous n’avez pas d’appareil avec des processeurs TPM, vous pouvez utiliser un simulateur à des fins de développement sur le système d’exploitation Windows. Exécutez la commande suivante pour activer l’authentification TPM et exécuter le simulateur TPM :

```
cmake -Ddps_auth_type=tpm_simulator ..
```

## <a name="build-the-sdk"></a>Générer le kit de développement logiciel (SDK)
Générez le kit de développement logiciel (SDK) avant de créer l’inscription d’appareils.

### <a name="linux"></a>Linux
- Pour générer le kit de développement logiciel (SDK) dans Linux :
    ```
    cd azure-iot-sdk-c
    mkdir cmake
    cd cmake
    cmake ..
    cmake --build .  # append '-- -j <n>' to run <n> jobs in parallel
    ```
- Pour générer des fichiers binaires de débogage, ajoutez l’option CMake correspondante à la commande de génération de projet, par exemple :
    ```
    cmake -DCMAKE_BUILD_TYPE=Debug ..
    ```

- Il existe de nombreuses [options de configuration CMake](https://cmake.org/cmake/help/v3.6/manual/cmake.1.html) disponibles pour générer le kit de développement logiciel (SDK). Par exemple, vous pouvez désactiver l’une des piles de protocoles disponibles en ajoutant un argument à la commande de génération de projet CMake :
    ```
    cmake -Duse_amqp=OFF ..
    ```
- Vous pouvez également générer et exécuter un test unitaire :
    ```
    cmake -Drun_unittests=ON ..
    cmake --build .
    ctest -C "debug" -V
    ```

### <a name="windows"></a>Windows
- Pour générer le kit de développement logiciel (SDK) dans Windows, procédez comme suit pour générer des fichiers de projet :
  - Ouvrez une « invite de commandes développeur pour VS2015 ».
  - Exécutez les commandes CMake suivantes à partir de la racine du référentiel :
    ```
    cd azure-iot-sdk-c
    mkdir cmake
    cd cmake
    cmake -G "Visual Studio 14 2015" ..
    ```
    Cette commande génère des bibliothèques x86. Pour effectuer une génération pour x64, modifiez l’argument de générateur Cmake : 
    ```
    cmake .. -G "Visual Studio 14 2015 Win64"
    ```

- Si la génération du projet se déroule correctement, vous devez voir un fichier de solution Visual Studio (.sln) sous le dossier `cmake`. Pour générer le kit de développement logiciel (SDK) :
    - Ouvrez **cmake\azure_iot_sdks.sln** dans Visual Studio et générez-le, **OU**
    - Exécutez la commande suivante dans l’invite de commandes que vous avez utilisée pour générer les fichiers de projet :
      ```
      cmake --build . -- /m /p:Configuration=Release
      ```
- Pour générer des fichiers binaires de débogage, utilisez l’argument MSBuild correspondant : 
    ```
    cmake --build . -- /m /p:Configuration=Debug`
    ```
- Il existe de nombreuses options de configuration CMake disponibles pour générer le kit de développement logiciel (SDK). Par exemple, vous pouvez désactiver l’une des piles de protocoles disponibles en ajoutant un argument à la commande de génération de projet CMake :
    ```
    cmake -G "Visual Studio 14 2015" -Duse_amqp=OFF ..
    ```
- Vous pouvez également générer et exécuter des tests unitaires :
    ```
    cmake -G "Visual Studio 14 2015" -Drun_unittests=ON ..
    cmake --build . -- /m /p:Configuration=Debug
    ctest -C "debug" -V
    ```
  
### <a name="libraries-to-include"></a>Bibliothèques à inclure
- Ces bibliothèques doivent être incluses dans votre kit de développement logiciel (SDK) :
    - Le service d’approvisionnement : dps_http_transport, dps_client, dps_security_client
    - IoTHub Security : iothub_security_client

## <a name="create-a-device-enrollment-entry-in-device-provisioning-services"></a>Créer une entrée d’inscription d’appareil dans le service Device Provisioning Service

### <a name="tpm"></a>Module de plateforme sécurisée (TPM)
Si vous utilisez TPM, suivez les instructions de [« Créer et approvisionner un appareil TPM simulé à l’aide du kit de développement logiciel C de périphérique pour le service IoT Hub Device Provisioning »](./quick-create-simulated-device.md) pour créer une entrée d’inscription d’appareil dans votre service Device Provisioning Service et simuler le premier démarrage.

### <a name="x509"></a>X.509

1. Pour inscrire un appareil dans le service d’approvisionnement, vous devez noter la paire de clé de type EK et l’ID d’inscription de chaque appareil, qui sont affichés dans l’outil d’approvisionnement fourni par le kit de développement logiciel (SDK) Client. Exécutez la commande suivante pour imprimer le certificat d’autorité de certification racine (pour les groupes d’inscription) et le certificat de nœud terminal (pour l’inscription individuelle) :
      ```
      ./azure-iot-sdk-c/dps_client/tools/x509_device_provision/x509_device_provision.exe
      ```
2. Connectez-vous au portail Azure, cliquez sur le bouton **Toutes les ressources** dans le menu de gauche et ouvrez votre instance Device Provisioning Service.
   - **Inscription individuelle X.509** : Dans le panneau de récapitulatif du service Device Provisioning, sélectionnez **Gérer les inscriptions**. Sélectionnez l’onglet **Inscriptions individuelles** et cliquez sur le bouton **Ajouter** dans la partie supérieure. Sélectionnez **X.509** en tant que *mécanisme* d’attestation d’identité, chargez le certificat de nœud terminal comme requis par le panneau. Cela fait, cliquez sur le bouton **Enregistrer**. 
   - **Inscription de groupe X.509** : Dans le panneau de récapitulatif du service de provisionnement, sélectionnez **Gérer les inscriptions**. Sélectionnez l’onglet **Inscriptions de groupe** et cliquez sur le bouton **Ajouter** dans la partie supérieure. Sélectionnez **X.509** en tant que *mécanisme* d’attestation d’identité, entrez un nom de groupe et un nom de certification, chargez le certificat d’autorité de certification/intermédiaire comme requis par le panneau. Cela fait, cliquez sur le bouton **Enregistrer**. 

## <a name="enable-authentication-for-devices-using-a-custom-attestation-mechanism-optional"></a>Activer l’authentification pour les appareils à l’aide d’un mécanisme d’attestation personnalisé (facultatif)

> [!NOTE]
> Cette section ne s’applique qu’aux appareils nécessitant la prise en charge d’une plateforme ou de mécanismes d’attestation personnalisés, non pris en charge actuellement par le Kit de développement logiciel (SDK) Device Provisioning Service Client SDK pour C. Notez également que le Kit de développement logiciel (SDK) utilise fréquemment le terme « HSM » comme substitut générique à la place de « mécanisme d’attestation ».

Vous devez d’abord développer un référentiel et une bibliothèque pour votre mécanisme d’attestation personnalisé :

1. Développez une bibliothèque pour accéder à votre mécanisme d’attestation. Ce projet doit produire une bibliothèque statique destinée au SDK Device Provisioning.

2. Implémentez les fonctions définies dans le fichier d’en-tête suivant, dans votre bibliothèque : 

    - Pour un module TPM personnalisé : implémentez les fonctions définies sous [HSM TPM API](https://github.com/Azure/azure-iot-sdk-c/blob/master/provisioning_client/devdoc/using_custom_hsm.md#hsm-tpm-api) (API TPM HSM).  
    - Pour un module X.509 personnalisé : implémentez les fonctions définies sous [HSM X509 API](https://github.com/Azure/azure-iot-sdk-c/blob/master/provisioning_client/devdoc/using_custom_hsm.md#hsm-x509-api) (API X509 HSM). 

Une fois votre bibliothèque générée, vous devez l’intégrer au Kit de développement logiciel (SDK) Device Provisioning Service Client en associant ce dernier à votre bibliothèque. :

1. Indiquez le référentiel GitHub personnalisé et la bibliothèque dans la commande `cmake` suivante :
    ```cmd/sh
    cmake -Duse_prov_client:BOOL=ON -Dhsm_custom_lib=<path_and_name_of_library> <PATH_TO_AZURE_IOT_SDK>
    ```
   
2. Ouvrez le fichier de solution Visual Studio généré par CMake (`\azure-iot-sdk-c\cmake\azure_iot_sdks.sln`) et générez-le. 

    - Le processus de génération compile la bibliothèque du Kit de développement logiciel (SDK).
    - Le Kit de développement logiciel (SDK) essaie d’établir un lien avec la bibliothèque personnalisée définie dans la commande `cmake`.

3. Pour vérifier que votre mécanisme d’attestation personnalisé est implémenté correctement, exécutez l’exemple d’application « prov_dev_client_ll_sample » sous « Provision_Samples » (sous `\azure-iot-sdk-c\cmake\provisioning_client\samples\prov_dev_client_ll_sample`).

## <a name="connecting-to-iot-hub-after-provisioning"></a>Connexion à IoT Hub après l’approvisionnement

Une fois l’appareil approvisionné avec le service d’approvisionnement, cette API utilise le mode d’authentification spécifié (**X.509** ou TPM) pour se connecter à IoT Hub : 
  ```
  IOTHUB_CLIENT_LL_HANDLE handle = IoTHubClient_LL_CreateFromDeviceAuth(iothub_uri, device_id, iothub_transport);
  ```
