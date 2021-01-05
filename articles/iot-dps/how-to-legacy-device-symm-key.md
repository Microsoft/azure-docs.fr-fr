---
title: Provisionner des appareils en utilisant des clés symétriques – Service Azure IoT Hub Device Provisioning
description: Guide pratique pour utiliser des clés symétriques afin de provisionner des appareils avec votre instance DPS (Device Provisioning Service)
author: wesmc7777
ms.author: wesmc
ms.date: 07/13/2020
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: eliotga
ms.openlocfilehash: dc33dcd2c80b2a6d4a1cc27778e49dc06ac48b34
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967310"
---
# <a name="how-to-provision-devices-using-symmetric-key-enrollment-groups"></a>Comment approvisionner des appareils à l’aide de groupes d’inscription avec des clés symétriques

Cet article explique comment approvisionner de façon sécurisée plusieurs appareils avec des clés symétriques sur un seul IoT Hub à l’aide d’un groupe d’inscription.

Certains appareils peuvent ne pas avoir de certificat, de module de plateforme sécurisée (TPM) ou d’autre fonctionnalité de sécurité utilisable pour identifier l’appareil de façon sécurisée. Le service Device Provisioning inclut l’[attestation de clé symétrique](concepts-symmetric-key-attestation.md). L’attestation de clé symétrique peut être utilisée pour identifier un appareil sur la base d’informations uniques comme l’adresse MAC ou un numéro de série.

Si vous pouvez facilement installer un [module de sécurité matériel](concepts-service.md#hardware-security-module) et un certificat, cette approche peut être meilleure pour identifier et provisionner vos appareils. En effet, cette approche peut vous permettre de contourner la mise à jour du code déployé sur tous vos appareils et vous n’avez pas de clé secrète incorporée dans l’image de votre appareil.

Cet article suppose que ni un module de sécurité matériel ni un certificat ne sont des options viables. Il est cependant supposé que vous disposez d’une méthode de mise à jour du code de l’appareil pour utiliser le service Device Provisioning afin de provisionner ces appareils. 

Cet article suppose également que la mise à jour de l’appareil a lieu dans un environnement sécurisé, pour empêcher tout accès non autorisé à la clé de groupe principale ou à la clé d’appareil dérivée.

Cet article traite d’une station de travail Windows. Toutefois, vous pouvez effectuer les procédures sur Linux. Pour obtenir un exemple sur Linux, consultez [Guide pratique du provisionnement pour la multilocation](how-to-provision-multitenant.md).

> [!NOTE]
> L’exemple utilisé dans cet article est écrit en C. Un [exemple de clé symétrique d’approvisionnement d’appareil en C#](https://github.com/Azure-Samples/azure-iot-samples-csharp/tree/master/provisioning/Samples/device/SymmetricKeySample) est également disponible. Pour utiliser cet exemple, téléchargez ou clonez le référentiel [azure-iot-samples-csharp ](https://github.com/Azure-Samples/azure-iot-samples-csharp) et suivez les instructions en ligne de l’exemple de code. Vous pouvez suivre les instructions de cet article pour créer un groupe d’inscription de clé symétrique à l’aide du portail et pour rechercher l’étendue de l’ID et les clés primaires et secondaires du groupe d’inscription nécessaires à l’exécution de l’exemple. Vous pouvez également créer des inscriptions individuelles à l’aide de l’exemple.

## <a name="overview"></a>Vue d’ensemble

Un ID d’inscription unique est défini pour chaque appareil en fonction des informations qui identifient cet appareil. Par exemple, l’adresse MAC ou un numéro de série.

Un groupe d’inscription qui utilise [l’attestation de clé symétrique](concepts-symmetric-key-attestation.md) est créé avec le service Device Provisioning. Le groupe d’inscription inclut une clé de groupe principale. Cette clé principale est utilisée pour hacher chaque ID d’inscription unique de façon à produire une clé d’appareil unique pour chaque appareil. L’appareil utilise cette clé d’appareil dérivée avec son ID d’inscription unique pour attester avec le service Device Provisioning et être affecté à un hub IoT.

Le code de l’appareil montré dans cet article suit le même modèle que le [Démarrage rapide : provisionner un appareil simulé avec des clés symétriques](quick-create-simulated-device-symm-key.md). Le code simule un appareil en utilisant un exemple provenant du [SDK C Azure IoT](https://github.com/Azure/azure-iot-sdk-c). Comme illustré dans le guide de démarrage rapide, l’appareil simulé atteste avec un groupe d’inscription au lieu d’une inscription individuelle.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]


## <a name="prerequisites"></a>Prérequis

* Avoir effectué les étapes décrites dans le guide de démarrage rapide [Configurer le service IoT Hub Device Provisioning avec le portail Azure](./quick-setup-auto-provision.md).

Les prérequis suivants s’appliquent à un environnement de développement Windows. Pour Linux ou macOS, consultez la section appropriée de [Préparer votre environnement de développement](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md) dans la documentation du kit de développement logiciel (SDK).

* [Visual Studio](https://visualstudio.microsoft.com/vs/) 2019 avec la charge de travail [« Développement Desktop en C++ »](/cpp/ide/using-the-visual-studio-ide-for-cpp-desktop-development) activée. Visual Studio 2015 et Visual Studio 2017 sont également pris en charge.

* Dernière version de [Git](https://git-scm.com/download/) installée.

## <a name="prepare-an-azure-iot-c-sdk-development-environment"></a>Préparer un environnement de développement pour le SDK Azure IoT pour C

Dans cette section, vous allez préparer un environnement de développement pour générer le [SDK Azure IoT pour C](https://github.com/Azure/azure-iot-sdk-c). 

Le SDK inclut l’exemple de code pour l’appareil simulé. Cet appareil simulé tente le provisionnement durant la séquence de démarrage de l’appareil.

1. Téléchargez le [système de génération CMake](https://cmake.org/download/).

    Il est important que les composants requis Visual Studio (Visual Studio et la charge de travail « Développement Desktop en C++ ») soient installés sur votre machine, **avant** de commencer l’installation de l’élément `CMake`. Une fois les composants requis en place et le téléchargement effectué, installez le système de génération de CMake.

2. Recherchez le nom de balise de la [version la plus récente](https://github.com/Azure/azure-iot-sdk-c/releases/latest) du kit de développement logiciel (SDK).

3. Ouvrez une invite de commandes ou l’interpréteur de commandes Git Bash. Exécutez les commandes suivantes pour cloner la dernière version du dépôt GitHub du [Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c). Utilisez l’étiquette obtenue à l’étape précédente comme valeur pour le paramètre `-b` :

    ```cmd/sh
    git clone -b <release-tag> https://github.com/Azure/azure-iot-sdk-c.git
    cd azure-iot-sdk-c
    git submodule update --init
    ```

    Attendez-vous à ce que cette opération prenne plusieurs minutes.

4. Créez un sous-répertoire `cmake` dans le répertoire racine du référentiel Git et accédez à ce dossier. Exécutez les commandes suivantes à partir du répertoire `azure-iot-sdk-c` :

    ```cmd/sh
    mkdir cmake
    cd cmake
    ```

5. Exécutez la commande suivante qui génère une version du SDK propre à votre plateforme cliente de développement. Une solution Visual Studio pour l’appareil simulé est générée dans le répertoire `cmake`. 

    ```cmd
    cmake -Dhsm_type_symm_key:BOOL=ON -Duse_prov_client:BOOL=ON  ..
    ```
    
    Si `cmake` ne trouve pas votre compilateur C++, vous obtiendrez peut-être des erreurs de build lors de l’exécution de la commande ci-dessus. Si cela se produit, essayez d’exécuter cette commande dans [l’invite de commandes de Visual Studio](/dotnet/framework/tools/developer-command-prompt-for-vs). 

    Une fois la génération terminée, les dernières lignes de sortie doivent ressembler à la sortie suivante :

    ```cmd/sh
    $ cmake -Dhsm_type_symm_key:BOOL=ON -Duse_prov_client:BOOL=ON  ..
    -- Building for: Visual Studio 15 2017
    -- Selecting Windows SDK version 10.0.16299.0 to target Windows 10.0.17134.
    -- The C compiler identification is MSVC 19.12.25835.0
    -- The CXX compiler identification is MSVC 19.12.25835.0

    ...

    -- Configuring done
    -- Generating done
    -- Build files have been written to: E:/IoT Testing/azure-iot-sdk-c/cmake
    ```


## <a name="create-a-symmetric-key-enrollment-group"></a>Créer un groupe d’inscription de clé symétrique

1. Connectez-vous au [portail Azure](https://portal.azure.com) et ouvrez votre instance du service Device Provisioning.

2. Sélectionnez l’onglet **Gérer les inscriptions**, puis cliquez sur le bouton **Ajouter un groupe d’inscriptions** en haut de la page. 

3. Dans **Ajouter un groupe d’inscriptions**, entrez les informations suivantes, puis cliquez sur le bouton **Enregistrer**.

   - **Nom du groupe** : entrez **mylegacydevices**.

   - **Type d’attestation** : sélectionnez **Clé symétrique**.

   - **Générer automatiquement les clés** : cochez cette case.

   - **Sélectionner le mode d’affectation des appareils aux hubs** : sélectionnez **Configuration statique** pour pouvoir affecter à un hub spécifique.

   - **Sélectionner les hubs IoT auxquels ce groupe peut être affecté** : sélectionnez un de vos hubs.

     ![Ajouter un groupe d’inscription pour l’attestation de clé symétrique](./media/how-to-legacy-device-symm-key/symm-key-enrollment-group.png)

4. Une fois que vous avez enregistré votre inscription, la **Clé primaire** et la **Clé secondaire** sont générées et ajoutées à l’entrée d’inscription. Votre groupe d’inscription de clé symétrique apparaît en tant que **mylegacydevices** sous la colonne *Nom du groupe* dans l’onglet *Groupes d’inscription*. 

    Ouvrez l’inscription et copiez la valeur de votre **Clé primaire** générée. Cette clé est votre clé de groupe principale.


## <a name="choose-a-unique-registration-id-for-the-device"></a>Choisir un ID d’inscription unique pour l’appareil

Un ID d’inscription unique doit être défini pour identifier chaque appareil. Vous pouvez utiliser l’adresse MAC, le numéro de série ou toute information unique provenant de l’appareil. 

Dans cet exemple, nous utilisons une combinaison d’une adresse MAC et du numéro de série, qui forment la chaîne suivante pour un ID d’inscription.

```
sn-007-888-abc-mac-a1-b2-c3-d4-e5-f6
```

Créez un ID d’inscription unique pour votre appareil. Les caractères valides sont les caractères alphanumériques minuscules et les tirets (« - »).


## <a name="derive-a-device-key"></a>Dériver une clé d’appareil 

Pour générer la clé de l’appareil, calculez le code [HMAC-SHA256](https://wikipedia.org/wiki/HMAC) de l’ID d’inscription unique de l’appareil en utilisant la clé de groupe principale, puis convertissez le résultat au format Base64.

> [!WARNING]
> Votre code d’appareil doit inclure uniquement la clé d’appareil dérivée pour l’appareil individuel. N’incluez pas votre clé de groupe principale dans le code de l’appareil. Une clé principale compromise est susceptible de compromettre la sécurité de tous les appareils qui y sont authentifiés.


#### <a name="linux-workstations"></a>Stations de travail Linux

Si vous utilisez une station de travail Linux, vous pouvez utiliser openssl pour générer votre clé d’appareil dérivée, comme indiqué dans l’exemple suivant.

Remplacez la valeur **KEY** par la **clé primaire** que vous avez notée précédemment.

Remplacez la valeur de **REG_ID** avec votre ID d’inscription.

```bash
KEY=8isrFI1sGsIlvvFSSFRiMfCNzv21fjbE/+ah/lSh3lF8e2YG1Te7w1KpZhJFFXJrqYKi9yegxkqIChbqOS9Egw==
REG_ID=sn-007-888-abc-mac-a1-b2-c3-d4-e5-f6

keybytes=$(echo $KEY | base64 --decode | xxd -p -u -c 1000)
echo -n $REG_ID | openssl sha256 -mac HMAC -macopt hexkey:$keybytes -binary | base64
```

```bash
Jsm0lyGpjaVYVP2g3FnmnmG9dI/9qU24wNoykUmermc=
```


#### <a name="windows-based-workstations"></a>Stations de travail Windows

Si vous utilisez une station de travail Windows, utilisez PowerShell pour générer votre clé d’appareil dérivée, comme indiqué dans l’exemple suivant.

Remplacez la valeur **KEY** par la **clé primaire** que vous avez notée précédemment.

Remplacez la valeur de **REG_ID** avec votre ID d’inscription.

```powershell
$KEY='8isrFI1sGsIlvvFSSFRiMfCNzv21fjbE/+ah/lSh3lF8e2YG1Te7w1KpZhJFFXJrqYKi9yegxkqIChbqOS9Egw=='
$REG_ID='sn-007-888-abc-mac-a1-b2-c3-d4-e5-f6'

$hmacsha256 = New-Object System.Security.Cryptography.HMACSHA256
$hmacsha256.key = [Convert]::FromBase64String($KEY)
$sig = $hmacsha256.ComputeHash([Text.Encoding]::ASCII.GetBytes($REG_ID))
$derivedkey = [Convert]::ToBase64String($sig)
echo "`n$derivedkey`n"
```

```powershell
Jsm0lyGpjaVYVP2g3FnmnmG9dI/9qU24wNoykUmermc=
```


Votre appareil utilise la clé d’appareil dérivée avec votre ID d’inscription unique pour effectuer l’attestation de clé symétrique avec le groupe d’inscription lors du provisionnement.



## <a name="create-a-device-image-to-provision"></a>Créer une image d’appareil à provisionner

Dans cette section, vous allez mettre à jour un exemple de provisionnement nommé **prov\_dev\_client\_sample** provenant du SDK Azure IoT pour C que vous avez configuré précédemment. 

Cet exemple de code simule une séquence de démarrage d’un appareil qui envoie la demande de provisionnement à votre instance du service Device Provisioning. La séquence de démarrage entraîne la reconnaissance de l’appareil et son affectation au hub IoT que vous avez configuré sur le groupe d’inscription.

1. Dans le portail Azure, sélectionnez l’onglet **Vue d’ensemble** de votre service Device Provisioning et notez les valeurs de **_Étendue de l’ID_**.

    ![Extraction des informations de point de terminaison du service Device Provisioning à partir du panneau du Portail](./media/quick-create-simulated-device-x509/extract-dps-endpoints.png) 

2. Dans Visual Studio, ouvrez le fichier solution **azure_iot_sdks.sln** généré précédemment en exécutant CMake. Le fichier solution doit être à l’emplacement suivant :

    ```
    \azure-iot-sdk-c\cmake\azure_iot_sdks.sln
    ```

3. Dans la fenêtre *Explorateur de solutions* de Visual Studio, accédez au dossier **Provision\_Samples**. Développez l’exemple de projet nommé **prov\_dev\_client\_sample**. Développez **Fichiers sources**, puis ouvrez **prov\_dev\_client\_sample.c**.

4. Recherchez la constante `id_scope` et remplacez la valeur par la valeur **Étendue de l’ID** que vous avez copiée précédemment. 

    ```c
    static const char* id_scope = "0ne00002193";
    ```

5. Recherchez la définition de la fonction `main()` dans le même fichier. Vérifiez que la variable `hsm_type` a la valeur `SECURE_DEVICE_TYPE_SYMMETRIC_KEY`, comme indiqué ci-dessous :

    ```c
    SECURE_DEVICE_TYPE hsm_type;
    //hsm_type = SECURE_DEVICE_TYPE_TPM;
    //hsm_type = SECURE_DEVICE_TYPE_X509;
    hsm_type = SECURE_DEVICE_TYPE_SYMMETRIC_KEY;
    ```

6. Recherchez l'appel vers `prov_dev_set_symmetric_key_info()` dans **prov\_dev\_client\_sample.c** qui est commenté.

    ```c
    // Set the symmetric key if using they auth type
    //prov_dev_set_symmetric_key_info("<symm_registration_id>", "<symmetric_Key>");
    ```

    Supprimez les commentaires de l’appel de fonction et remplacez les valeurs d’espace réservé (y compris les crochets) par l’ID d’inscription unique de votre appareil et la clé d’appareil dérivée que vous avez générée.

    ```c
    // Set the symmetric key if using they auth type
    prov_dev_set_symmetric_key_info("sn-007-888-abc-mac-a1-b2-c3-d4-e5-f6", "Jsm0lyGpjaVYVP2g3FnmnmG9dI/9qU24wNoykUmermc=");
    ```
   
    Enregistrez le fichier.

7. Cliquez avec le bouton droit sur le projet **prov\_dev\_client\_sample** et sélectionnez **Définir comme projet de démarrage**. 

8. Dans le menu Visual Studio, sélectionnez **Déboguer** > **Exécuter sans débogage** pour exécuter la solution. Dans l’invite pour régénérer le projet, cliquez sur **Oui** pour régénérer le projet avant l’exécution.

    La sortie suivante est un exemple illustrant le démarrage réussi de l’appareil simulé et sa connexion à une instance de service Device Provisioning pour l’affecter à un hub IoT :

    ```cmd
    Provisioning API Version: 1.2.8

    Registering Device

    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING

    Registration Information received from service: 
    test-docs-hub.azure-devices.net, deviceId: sn-007-888-abc-mac-a1-b2-c3-d4-e5-f6

    Press enter key to exit:
    ```

9. Dans le portail, accédez au hub IoT auquel votre appareil simulé a été affecté, puis cliquez sur l’onglet **Appareils IoT**. En cas de réussite du provisionnement de l’appareil simulé sur le hub, son ID d’appareil s’affiche sur le panneau **Appareils IoT**, avec un *ÉTAT* **activé**. Vous devrez peut-être cliquer sur le bouton **Actualiser** dans la partie supérieure. 

    ![L’appareil est inscrit avec le hub IoT](./media/how-to-legacy-device-symm-key/hub-registration.png) 



## <a name="security-concerns"></a>Considérations sur la sécurité

Ne perdez pas de vue que ceci laisse la clé d’appareil dérivée incluse dans l’image, ce qui n’est pas une bonne pratique de sécurité recommandée. C’est une raison pour laquelle la sécurité et la facilité d’utilisation sont des compromis. 





## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur le reprovisionnement, consultez [Concepts du reprovisionnement d’appareils IoT Hub](concepts-device-reprovision.md) 
* [Démarrage rapide : provisionner un appareil simulé avec des clés symétriques](quick-create-simulated-device-symm-key.md)
* Pour en savoir plus sur le déprovisionnement, consultez [Guide pratique pour déprovisionner des appareils auparavant provisionnés automatiquement](how-to-unprovision-devices.md)