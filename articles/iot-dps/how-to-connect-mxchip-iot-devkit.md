---
title: Utilisation de l’approvisionnement automatique du service Azure IoT Hub Device Provisioning pour inscrire le DevKit IoT MXChip auprès d’IoT Hub | Microsoft Docs
description: Guide pratique pour utiliser le provisionnement automatique du service Azure IoT Hub Device Provisioning (DPS) afin d’inscrire le DevKit IoT MXChip auprès d’IoT Hub.
author: wesmc7777
ms.author: wesmc
ms.date: 06/25/2019
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: eliotgra
ms.openlocfilehash: d6b6649d03da319171b24baa24983972bf270679
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94954543"
---
# <a name="use-azure-iot-hub-device-provisioning-service-auto-provisioning-to-register-the-mxchip-iot-devkit-with-iot-hub"></a>Utiliser l’approvisionnement automatique du service Azure IoT Hub Device Provisioning pour inscrire le DevKit IoT MXChip auprès d’IoT Hub

Cet article décrit la procédure d’utilisation de [l’approvisionnement automatique](about-iot-dps.md#provisioning-process) du service Azure IoT Hub Device Provisioning du DevKit IoT MXChip auprès d’IoT Hub. Dans ce tutoriel, vous allez apprendre à :

* configurer le point de terminaison global du service Device Provisioning sur un appareil.
* utiliser un secret d’appareil unique (UDS) pour générer un certificat X.509 ;
* inscrire un appareil particulier ;
* vérifier que l’appareil est enregistré.

Le [IoT MXChip DevKit](https://aka.ms/iot-devkit) est une carte tout-en-un compatible Arduino qui est équipée de périphériques et de capteurs élaborés. Vous pouvez développer pour ce kit à l’aide du pack d’extension [Azure IoT Device Workbench](https://aka.ms/iot-workbench) ou [Azure IoT Tools](https://aka.ms/azure-iot-tools) dans Visual Studio Code. Par ailleurs, il s’accompagne d’un [catalogue de projets](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/) en plein développement pour vous guider dans la réalisation de prototypes de solutions IoT (Internet des objets) qui exploitent les services Azure.

## <a name="before-you-begin"></a>Avant de commencer

Pour effectuer les étapes de ce didacticiel, commencez par exécuter les tâches suivantes :

* Configurez le Wi-Fi de votre DevKit et préparez votre environnement de développement en suivant les étapes de la section « Préparer l’environnement de développement » dans l’article [Connecter IoT DevKit AZ3166 à Azure IoT Hub dans le cloud](../iot-hub/iot-hub-arduino-iot-devkit-az3166-get-started.md#prepare-the-development-environment).
* Procédez à la mise à niveau vers la dernière version du microprogramme (1.3.0 ou ultérieure) à l’aide du didacticiel [Update DevKit firmware](https://microsoft.github.io/azure-iot-developer-kit/docs/firmware-upgrading/) (Mettre à jour le microprogramme du DevKit).
* Créez et liez un IoT Hub avec une instance du service Device Provisioning en suivant les étapes décrites dans [Configurer le service IoT Hub Device Provisioning avec le portail Azure](./quick-setup-auto-provision.md).

## <a name="open-sample-project"></a>Ouvrir un exemple de projet

1. Assurez-vous que votre IoT DevKit n’est **pas connecté** à votre ordinateur. Démarrez d’abord VS Code, puis connectez le DevKit à votre ordinateur.

1. Cliquez sur `F1` pour ouvrir la palette de commandes, puis tapez et sélectionnez **Azure IoT Device Workbench : Ouvrir des exemples...** Puis sélectionnez **IoT DevKit** en tant que carte.

1. Dans la page des exemples IoT Workbench, recherchez **Inscription de l’appareil avec DPS**, puis cliquez sur **Ouvrir l’exemple**. Sélectionnez ensuite le chemin d’accès par défaut pour télécharger l’exemple de code.
    ![Ouvrir l’exemple](media/how-to-connect-mxchip-iot-devkit/open-sample.png)

## <a name="save-a-unique-device-secret-on-device-security-storage"></a>Enregistrer un secret d’appareil unique dans un stockage de sécurité d’appareil

L’approvisionnement automatique peut être configuré sur un appareil en fonction du [mécanisme d’attestation](concepts-service.md#attestation-mechanism) de l’appareil. Le MXChip IoT DevKit utilise le moteur [DICE (Device Identity Composition Engine)](https://trustedcomputinggroup.org/wp-content/uploads/Foundational-Trust-for-IOT-and-Resource-Constrained-Devices.pdf) de [Trusted Computing Group (TCG)](https://trustedcomputinggroup.org). Un **secret d’appareil unique** (UDS) enregistré dans un processeur de sécurité STSAFE ([STSAFE-A100](https://microsoft.github.io/azure-iot-developer-kit/docs/understand-security-chip/)) du DevKit sert à générer le [certificat X.509](concepts-x509-attestation.md) unique de l’appareil. Ce certificat est utilisé par la suite pour le processus d’inscription du service Device Provisioning, ainsi que lors de l’inscription au moment de l’exécution.

Par défaut, un secret d’appareil unique (UDS) est une chaîne de 64 caractères, comme illustré dans l’exemple suivant :

```
19e25a259d0c2be03a02d416c05c48ccd0cc7d1743458aae1cb488b074993eae
```

Pour enregistrer un UDS sur le DevKit :

1. Dans VS Code, cliquez sur la barre d’état pour sélectionner le port COM pour le DevKit.
  ![Sélectionner le port COM](media/how-to-connect-mxchip-iot-devkit/select-com.png)

1. Dans DevKit, maintenez le **bouton A** enfoncé, appuyez sur le bouton de **réinitialisation** et relâchez-le, puis relâchez le **bouton A**. Votre DevKit passe en mode de configuration.

1. Cliquez sur `F1` pour ouvrir la palette de commandes, puis tapez et sélectionnez **Azure IoT Device Workbench : Configurer les paramètres de l’appareil... > Configurer la chaîne d’appareil unique (UDS)** .
  ![Configurer une UDS](media/how-to-connect-mxchip-iot-devkit/config-uds.png)

1. Notez la chaîne UDS générée. Vous en aurez besoin pour générer le certificat X.509. Appuyez ensuite sur `Enter`.
  ![Copier l’UDS](media/how-to-connect-mxchip-iot-devkit/copy-uds.png)

1. Vérifiez dans la notification que l’UDS a été configuré correctement sur le STSAFE.
  ![Réussite de la configuration de l’UDS](media/how-to-connect-mxchip-iot-devkit/config-uds-success.png)

> [!NOTE]
> Vous pouvez également configurer l’UDS par le biais du port série à l’aide d’utilitaires tels que Putty. Pour ce faire, suivez les instructions de la rubrique [Utiliser le mode de configuration](https://microsoft.github.io/azure-iot-developer-kit/docs/use-configuration-mode/).

## <a name="update-the-global-device-endpoint-and-id-scope"></a>Mettre à jour le point de terminaison d’appareil global et l’étendue d’ID

Dans le code de l’appareil, vous devez spécifier le [point de terminaison de provisionnement des appareils](./concepts-service.md#device-provisioning-endpoint) et l’étendue d’ID pour garantir l’isolation des locataires.

1. Dans le portail Azure, sélectionnez le volet **Vue d’ensemble** de votre instance du service Device Provisioning et notez les valeurs des paramètres **Point de terminaison d’appareil global** et **Étendue de l’ID**.
  ![Point de terminaison global de service Device Provisioning et étendue de l’ID](media/how-to-connect-mxchip-iot-devkit/dps-global-endpoint.png)

1. Ouvrez **DevKitDPS.ino**. Recherchez `[Global Device Endpoint]` et `[ID Scope]`, puis remplacez-les par les valeurs que vous venez de noter.
  ![Point de terminaison du service Device Provisioning ](media/how-to-connect-mxchip-iot-devkit/endpoint.png)

1. Remplissez la variable `registrationId` dans le code. Seule une combinaison de caractères alphanumériques, de minuscules et de tirets d’une longueur maximale de 128 caractères est autorisée. Notez également la valeur.
  ![ID d'inscription](media/how-to-connect-mxchip-iot-devkit/registration-id.png)

1. Cliquez sur `F1`, puis tapez et sélectionnez **Azure IoT Device Workbench : Charger le code de l’appareil**. Cette opération démarre la compilation et le chargement du code dans le DevKit.
  ![Chargement de l’appareil](media/how-to-connect-mxchip-iot-devkit/device-upload.png)

## <a name="generate-x509-certificate"></a>Générer un certificat X.509

Le [mécanisme d’attestation](./concepts-service.md#attestation-mechanism) utilisé par cet exemple est le certificat X.509. Vous devez utiliser un utilitaire pour le générer.

1. Dans VS Code, cliquez sur `F1`, puis tapez et sélectionnez **Ouvrir un nouveau Terminal** pour ouvrir une fenêtre de terminal.

1. Exécutez `dps_cert_gen.exe` dans le dossier `tool`.

1. Spécifiez l’emplacement du fichier binaire compilé sous la forme `..\.build\DevKitDPS`. Collez ensuite les valeurs **UDS** et **registrationId** que vous venez de noter. 
  ![Générer un certificat X.509](media/how-to-connect-mxchip-iot-devkit/gen-x509.png)

1. Un certificat X.509 `.pem` est généré dans le même dossier.
  ![Fichier X.509](media/how-to-connect-mxchip-iot-devkit/pem-file.png)

## <a name="create-a-device-enrollment-entry"></a>Créer une entrée d’inscription d’appareil

1. Dans le portail Azure, ouvrez votre service de provisionnement des appareils, accédez à la section Gérer les inscriptions, puis cliquez sur **Ajouter une inscription individuelle**.
  ![Ajouter une inscription individuelle](media/how-to-connect-mxchip-iot-devkit/add-enrollment.png)

1. Cliquez sur l’icône de fichier en regard de **Fichier .pem ou .cer du certificat principal** pour charger le fichier `.pem` généré.
  ![Charger le fichier .pem](media/how-to-connect-mxchip-iot-devkit/upload-pem.png)

## <a name="verify-the-devkit-is-registered-with-azure-iot-hub"></a>Vérifier que le DevKit est inscrit auprès d’Azure IoT Hub

Appuyez sur le bouton de **réinitialisation** de DevKit. Vous devriez voir s’afficher **DPS connecté !** sur l’écran du DevKit. Une fois l’appareil redémarré, les actions suivantes se produisent :

1. L’appareil envoie une demande d’enregistrement à votre service Device Provisioning.
1. Le service Device Provisioning retourne une vérification d’inscription à laquelle votre appareil répond.
1. Une fois l’inscription réussie, le service Device Provisioning retourne l’URI d’IoT Hub, l’ID de l’appareil et la clé chiffrée à l’appareil.
1. L’application cliente IoT Hub sur l’appareil peut se connecter à votre hub.
1. Une fois la connexion avec le hub établie, l’appareil s’affiche dans Device Explorer, dans IoT Hub.
  ![Appareil inscrit](./media/how-to-connect-mxchip-iot-devkit/device-registered.png)

## <a name="problems-and-feedback"></a>Problèmes et commentaires

Si vous rencontrez des problèmes, consultez les [FAQ](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) du MXChip IoT DevKit ou contactez le support via les canaux ci-dessous :

* [Gitter.im](https://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à inscrire un appareil en toute sécurité dans le service Device Provisioning en utilisant le moteur DICE afin que l’appareil puisse s’enregistrer automatiquement auprès d’Azure IoT Hub. 

En résumé, vous avez découvert comment :

> [!div class="checklist"]
> * configurer le point de terminaison global du service Device Provisioning sur un appareil.
> * utiliser un secret d’appareil unique pour générer un certificat X.509 ;
> * inscrire un appareil particulier ;
> * vérifier que l’appareil est enregistré.

Découvrez comment [Créer et provisionner un appareil simulé](./quick-create-simulated-device.md).