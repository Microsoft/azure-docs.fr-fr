---
title: Tutoriel - Ajouter un appareil Azure IoT Edge à Azure IoT Central | Microsoft Docs
description: Tutoriel - Ajouter un appareil Azure IoT Edge à votre application Azure IoT Central en qualité d’opérateur
author: rangv
ms.author: rangv
ms.date: 05/29/2020
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.custom:
- mvc
- device-developer
- iot-edge
ms.openlocfilehash: 9b4bb462c94ab5a59dbd9d8fdd4cf619e311df56
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90987018"
---
# <a name="tutorial-add-an-azure-iot-edge-device-to-your-azure-iot-central-application"></a>Tutoriel : Ajouter un appareil Azure IoT Edge à votre application Azure IoT Central

*Cet article s’applique aux opérateurs, aux créateurs de solutions et aux développeurs d’appareils.*

Ce tutoriel vous montre comment configurer et ajouter un appareil Azure IoT Edge à votre application Azure IoT Central. Ce tutoriel utilise une machine virtuelle Linux activée pour IoT Edge afin de simuler un appareil IoT Edge. L’appareil IoT Edge utilise un module qui génère des données de télémétrie environnementales simulées. Vous visualisez la télémétrie sur un tableau de bord dans votre application Azure IoT Central.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un modèle d’appareil pour un appareil IoT Edge
> * Créer un appareil IoT Edge dans IoT Central
> * Déployer un appareil IoT Edge simulé sur une machine virtuelle Linux

## <a name="prerequisites"></a>Prérequis

Suivez le guide de démarrage rapide [Créer une application Azure IoT Central](./quick-deploy-iot-central.md) pour créer une application IoT Central avec le modèle **Application personnalisée > Application personnalisée**.

Pour effectuer les étapes de ce tutoriel, vous avez besoin d’un compte Azure actif.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

Téléchargez le fichier manifeste IoT Edge à partir de GitHub. Cliquez avec le bouton droit sur le lien, puis sélectionnez **Enregistrer le lien sous** : [EnvironmentalSensorManifest.json](https://raw.githubusercontent.com/Azure-Samples/iot-central-docs-samples/master/iotedge/EnvironmentalSensorManifest.json)

## <a name="create-device-template"></a>Créer un modèle d’appareil

Dans cette section, vous allez créer un modèle d’appareil IoT Central pour un appareil IoT Edge. Pour commencer, vous importez un manifeste IoT Edge, puis vous modifiez le modèle pour ajouter des définitions et des vues de télémétrie :

### <a name="import-manifest-to-create-template"></a>Importer le manifeste pour créer un modèle

Pour créer un modèle d’appareil à partir d’un manifeste IoT Edge :

1. Dans votre application Azure IoT Central, accédez à **Modèles d’appareil**, puis sélectionnez **+ Nouveau**.

1. Dans la page **Sélectionner un type de modèle**, sélectionnez la vignette **Azure IoT Edge**. Ensuite, sélectionnez **Next: Personnaliser**.

1. Sur la page **Charger un manifeste de déploiement Azure IoT Edge**, entrez le nom de modèle d’appareil *Périphérique de capteur d’environnement*. Ensuite, sélectionnez **Parcourir** pour charger le fichier **EnvironmentalSensorManifest.json** que vous avez téléchargé précédemment. Ensuite, sélectionnez **Next: Vérification**).

1. Dans la page **Vérifier**, sélectionnez **Créer**.

1. Sélectionnez l’interface **Gérer** dans le module **SimulatedTemperatureSensor** pour voir les deux propriétés définies dans le manifeste :

:::image type="content" source="media/tutorial-add-edge-as-leaf-device/imported-manifest.png" alt-text="Modèle d’appareil créé à partir du manifeste IoT Edge":::

### <a name="add-telemetry-to-manifest"></a>Ajouter de la télémétrie au manifeste

Un manifeste IoT Edge ne définit pas la télémétrie envoyée par un module. Ajoutez les définitions de télémétrie au modèle d’appareil dans IoT Central. Le module **SimulatedTemperatureSensor** envoie des messages de télémétrie qui se présentent comme le code JSON suivant :

```json
{
  "machine": {
    "temperature": 75.0,
    "pressure": 40.2
  },
  "ambient": {
    "temperature": 23.0,
    "humidity": 30.0
  },
  "timeCreated": ""
}
```

Pour ajouter les définitions de télémétrie au modèle d’appareil :

1. Sélectionnez l’interface **Gérer** dans le modèle **Environmental Sensor Edge Device**.

1. Sélectionnez **+ Ajouter une capacité**. Entrez *machine* comme **Nom d’affichage** et vérifiez que le **Type de capacité** est **Télémétrie**.

1. Sélectionnez **Objet** comme type de schéma, puis sélectionnez **Définir**. Dans la page de définition de l’objet, ajoutez *temperature* (température) et *pressure* (pression) comme attributs de type **Double**, puis sélectionnez **Appliquer**.

1. Sélectionnez **+ Ajouter une capacité**. Entrez *ambient* (ambiant) comme **Nom d’affichage** et vérifiez que le **Type de capacité** est **Télémétrie**.

1. Sélectionnez **Objet** comme type de schéma, puis sélectionnez **Définir**. Dans la page de définition de l’objet, ajoutez *temperature* (température) et *humidity* (humidité) comme attributs de type **Double**, puis sélectionnez **Appliquer**.

1. Sélectionnez **+ Ajouter une capacité**. Entrez *timeCreated* comme **Nom d’affichage** et vérifiez que le **Type de capacité** est **Télémétrie**.

1. Sélectionnez **DateTime** comme type de schéma.

1. Sélectionnez **Enregistrer** pour mettre à jour le modèle.

L’interface **Gérer** comprend désormais les types de télémétrie **machine**, **ambient** et **timeCreated** :

:::image type="content" source="media/tutorial-add-edge-as-leaf-device/manage-interface.png" alt-text="Modèle d’appareil créé à partir du manifeste IoT Edge":::

### <a name="add-views-to-template"></a>Ajouter des vues au modèle

Le modèle d’appareil n’a pas encore de vue pour permettre à un opérateur de voir la télémétrie provenant de l’appareil IoT Edge. Pour ajouter une vue au modèle d’appareil :

1. Sélectionnez l’interface **Vues** dans le modèle **Environmental Sensor Edge Device**.

1. Dans la page **Sélectionner pour ajouter une nouvelle vue**, sélectionnez la vignette **Visualisation de l’appareil**.

1. Changez le nom de la vue en *View IoT Edge device telemetry* (Visualiser la télémétrie de l’appareil IoT Edge).

1. Sélectionnez les types de télémétrie **ambient** et **machine**. Sélectionnez ensuite **Ajouter une vignette**.

1. Sélectionnez **Enregistrer** pour enregistrer la vue **View IoT Edge device telemetry**.

:::image type="content" source="media/tutorial-add-edge-as-leaf-device/template-telemetry-view.png" alt-text="Modèle d’appareil créé à partir du manifeste IoT Edge":::

### <a name="publish-the-template"></a>Publier le modèle

Avant de pouvoir ajouter un appareil qui utilise le modèle **Environmental Sensor Edge Device**, vous devez publier le modèle.

Accédez au modèle **Environmental Sensor Edge Device**, puis sélectionnez **Publier**. Dans le panneau **Publier ce modèle d’appareil dans l’application**, sélectionnez **Publier** pour publier le modèle :

:::image type="content" source="media/tutorial-add-edge-as-leaf-device/publish-template.png" alt-text="Modèle d’appareil créé à partir du manifeste IoT Edge":::

## <a name="add-iot-edge-device"></a>Ajouter un appareil IoT Edge

Maintenant que vous avez publié le modèle **Environmental Sensor Edge Device**, vous pouvez ajouter un appareil à votre application IoT Central :

1. Dans votre application IoT Central, accédez à la page **Appareils**, puis sélectionnez **Environmental Sensor Edge Device** dans la liste des modèles disponibles.

1. Sélectionnez **+ Nouveau** pour ajouter un nouvel appareil à partir du modèle. Dans la page **Créer un appareil**, sélectionnez **Créer**.

Vous disposez maintenant d’un nouvel appareil avec l’état **Inscrit** :

:::image type="content" source="media/tutorial-add-edge-as-leaf-device/new-device.png" alt-text="Modèle d’appareil créé à partir du manifeste IoT Edge":::

### <a name="get-the-device-credentials"></a>Obtenir les informations d’identification de l’appareil

Quand vous déployez l’appareil IoT Edge plus loin dans ce tutoriel, vous avez besoin des informations d’identification qui permettent à l’appareil de se connecter à votre application IoT Central. Pour obtenir les informations d’identification de l’appareil :

1. Dans la page **Appareils**, sélectionnez l’appareil que vous avez créé.

1. Sélectionnez **Connecter**.

1. Dans la page **Connexion de l’appareil**, notez les valeurs de **Étendue de l’ID**, **ID de l’appareil** et **Clé principale**. Vous utiliserez ces valeurs plus tard.

1. Sélectionnez **Fermer**.

Vous avez maintenant terminé la configuration de votre application IoT Central pour permettre à un appareil IoT Edge de se connecter.

## <a name="deploy-an-iot-edge-device"></a>Déployer un appareil IoT Edge

Dans ce tutoriel, vous utilisez une machine virtuelle Linux activée pour Azure IoT Edge créée sur Azure pour simuler un appareil IoT Edge. Pour créer la machine virtuelle prenant en charge IoT Edge dans votre abonnement Azure, cliquez sur :

[![Bouton déployer sur Azure pour iotedge-vm-deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fazure%2Fiotedge-vm-deploy%2Fmaster%2FedgeDeploy.json)

Sur la page **Déploiement personnalisé** :

1. Sélectionnez votre abonnement Azure.

1. Sélectionnez **Créer** pour créer un groupe de ressources appelé *central-edge-rg*.

1. Sélectionnez une région proche de vous.

1. Ajoutez un **préfixe d’étiquette DNS** unique tel que *contoso-central-edge*.

1. Choisissez le nom d’utilisateur administrateur de la machine virtuelle.

1. Entrez *temp* comme chaîne de connexion. Plus tard, vous configurerez l’appareil pour qu’il se connecte à l’aide de DPS.

1. Acceptez les valeurs par défaut pour la taille de machine virtuelle, la version Ubuntu et l’emplacement.

1. Sélectionnez le type d’authentification **password**.

1. Entrez un mot de passe pour la machine virtuelle.

1. Sélectionnez ensuite **Vérifier + créer**.

1. Passez en revue vos choix, puis sélectionnez **Créer** :

    :::image type="content" source="media/tutorial-add-edge-as-leaf-device/vm-deployment.png" alt-text="Modèle d’appareil créé à partir du manifeste IoT Edge":::

Quelques minutes sont nécessaires pour achever le déploiement. Une fois le déploiement terminé, accédez au groupe de ressources **central-edge-rg** dans le portail Azure.

### <a name="configure-the-iot-edge-vm"></a>Configurer la machine virtuelle IoT Edge

Pour configurer la machine virtuelle IoT Edge pour qu’elle utilise DPS pour s’inscrire et se connecter à votre application IoT Central :

1. Dans le groupe de ressources **contoso-edge-rg**, sélectionnez l’instance de machine virtuelle.

1. Dans la section **Support + dépannage**, sélectionnez **Console série**. Si vous êtes invité à configurer les diagnostics de démarrage, suivez les instructions qui s’affichent dans le portail.

1. Appuyez sur **Entrée** pour voir l’invite `login:`. Entrez votre nom d’utilisateur et votre mot de passe pour vous connecter.

1. Exécutez la commande suivante pour vérifier la version du runtime IoT Edge. Au moment de la rédaction de cette documentation, la version est 1.0.9.1 :

    ```bash
    sudo iotedge --version
    ```

1. Utilisez l’éditeur `nano` pour ouvrir le fichier config.yaml d’IoT Edge :

    ```bash
    sudo nano /etc/iotedge/config.yaml
    ```

1. Faites défiler jusqu’à faire apparaître `# Manual provisioning configuration`. Placez les trois lignes suivantes en commentaire comme indiqué dans l’extrait de code suivant :

    ```yaml
    # Manual provisioning configuration
    #provisioning:
    #  source: "manual"
    #  device_connection_string: "temp"
    ```

1. Faites défiler jusqu’à faire apparaître `# DPS symmetric key provisioning configuration`. Décommentez les huit lignes suivantes comme indiqué dans l’extrait de code suivant :

    ```yaml
    # DPS symmetric key provisioning configuration
    provisioning:
      source: "dps"
      global_endpoint: "https://global.azure-devices-provisioning.net"
      scope_id: "{scope_id}"
      attestation:
        method: "symmetric_key"
        registration_id: "{registration_id}"
        symmetric_key: "{symmetric_key}"
    ```

    > [!TIP]
    > Vérifiez qu’il n’y a pas d’espace restant devant `provisioning:`.

1. Remplacez `{scope_id}` par l’**étendue d’ID** que vous avez notée précédemment.

1. Remplacez `{registration_id}` par l’**ID de l’appareil** que vous avez noté précédemment.

1. Remplacez `{symmetric_key}` par la **Clé principale** que vous avez notée précédemment.

1. Enregistrez les modifications (**Ctrl+O**) et quittez (**Ctrl+X**) l’éditeur `nano`.

1. Exécutez la commande suivante pour redémarrer le démon IoT Edge :

    ```bash
    sudo systemctl restart iotedge
    ```

1. Pour vérifier l’état des modules IoT Edge, exécutez la commande suivante :

    ```bash
    iotedge list
    ```

    L’exemple de sortie suivant montre les modules en cours d’exécution :

    ```bash
    NAME                        STATUS           DESCRIPTION      CONFIG
    SimulatedTemperatureSensor  running          Up 20 seconds    mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0
    edgeAgent                   running          Up 27 seconds    mcr.microsoft.com/azureiotedge-agent:1.0
    edgeHub                     running          Up 22 seconds    mcr.microsoft.com/azureiotedge-hub:1.0
    ```

    > [!TIP]
    > Vous devrez peut-être attendre que tous les modules commencent à s’exécuter.

## <a name="view-the-telemetry"></a>Afficher les données de télémétrie

L’appareil IoT Edge simulé est maintenant en cours d’exécution sur la machine virtuelle. Dans votre application IoT Central, l’état de l’appareil est maintenant **Provisionné** dans la page **Appareils** :

:::image type="content" source="media/tutorial-add-edge-as-leaf-device/provisioned-device.png" alt-text="Modèle d’appareil créé à partir du manifeste IoT Edge":::

Vous pouvez voir la télémétrie de l’appareil dans la page **Afficher la télémétrie de l’appareil IoT Edge** :

:::image type="content" source="media/tutorial-add-edge-as-leaf-device/device-telemetry-view.png" alt-text="Modèle d’appareil créé à partir du manifeste IoT Edge":::

La page **Modules** affiche l’état des modules IoT Edge sur l’appareil :

:::image type="content" source="media/tutorial-add-edge-as-leaf-device/edge-module-status.png" alt-text="Modèle d’appareil créé à partir du manifeste IoT Edge":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez de continuer à utiliser la machine virtuelle IoT Edge, vous pouvez conserver et réutiliser les ressources que vous avez utilisées dans ce tutoriel. Sinon, vous pouvez supprimer les ressources que vous avez créées dans ce tutoriel afin d’éviter des frais supplémentaires :

* Pour supprimer la machine virtuelle IoT Edge et les ressources qui lui sont associées, supprimez le groupe de ressources **contoso-edge-rg** dans le portail Azure.
* Pour supprimer l’application IoT Central, accédez à la page **Votre application** dans la section **Administration** de l’application, puis sélectionnez **Supprimer**.

En tant que développeur de solutions ou opérateur, maintenant que vous avez appris à utiliser et gérer les appareils IoT Edge dans IoT Central, nous vous suggérons ensuite de lire :

> [!div class="nextstepaction"]
> [Utiliser des groupes d’appareils pour analyser les données de télémétrie des appareils](./tutorial-use-device-groups.md)

## <a name="next-steps"></a>Étapes suivantes

En tant que développeur d’appareils, maintenant que vous avez appris à utiliser et gérer les appareils IoT Edge dans IoT Central, nous vous suggérons ensuite de lire :

> [!div class="nextstepaction"]
> [Développer des modules IoT Edge](../../iot-edge/tutorial-develop-for-linux.md)