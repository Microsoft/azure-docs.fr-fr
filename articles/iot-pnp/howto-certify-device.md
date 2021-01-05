---
title: Certifier les appareils IoT Plug-and-Play | Microsoft Docs
description: En tant que concepteur d’appareils, découvrez comment exécuter des tests et soumettre un appareil à des fins de certification.
author: konichi3
ms.author: koichih
ms.date: 08/21/2020
ms.topic: how-to
ms.custom: mvc, devx-track-azurecli
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 1b17729437b238b98f60e2cd97664fa6aafcccd4
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462459"
---
# <a name="how-to-certify-iot-plug-and-play-devices"></a>Certifier des appareils IoT Plug-and-Play

Le programme de certification d’appareil IoT Plug-and-Play comprend des outils permettant de vérifier qu’un appareil répond aux exigences de certification IoT Plug-and-Play. Ces outils aident également les organisations à sensibiliser sur la disponibilité de leurs appareils IoT Plug-and-Play. Ces appareils certifiés sont adaptés aux solutions IoT et contribuent à réduire le délai de commercialisation.

Cet article vous montre comment :

- Installer l’extension de l’outil de ligne de commande Azure IoT pour Azure CLI
- Exécuter les tests IoT Plug-and-Play pour valider votre application d’appareil pendant la phase de développement  
- Utiliser le portail Azure Certified Device pour valider l’application de l’appareil

## <a name="prepare-your-device"></a>Préparer votre appareil

Le code d’application qui s’exécute sur votre appareil IoT Plug-and-Play doit :

- Se connecter à Azure IoT Hub à l’aide du [Device Provisioning Service (DPS)](../iot-dps/about-iot-dps.md)
- Suivez les [conventions IoT Plug-and-Play](concepts-convention.md) pour implémenter les données de télémétrie, les propriétés et les commandes.

L’application est un logiciel installé indépendamment du système d’exploitation ou fourni avec le système d’exploitation dans une image de microprogramme flashée sur l’appareil.

Le processus de certification s’assure que l’appareil est compatible avec IoT Plug-and-Play en vérifiant que l’implémentation de l’appareil correspond aux données de télémétrie, aux propriétés et aux commandes définies dans le modèle d’appareil [DTDL (Digital Twins Definition Language)](https://github.com/Azure/opendigitaltwins-dtdl) et que le modèle est disponible dans le [référentiel de modèles public Azure IoT](concepts-model-repository.md).

Pour répondre aux exigences de certification, votre appareil doit :

- Se connecter à Azure IoT Hub à l’aide du [DPS](../iot-dps/about-iot-dps.md).
- Implémenter les données de télémétrie, les propriétés et les commandes suivant les conventions IoT Plug-and-Play.
- Décrire les interactions de l’appareil avec un modèle [DTDL v2](https://aka.ms/dtdl).
- Publier le modèle et toutes les interfaces requises dans le [référentiel de modèles public IoT Azure](https://devicemodels.azureiotsolutions.com/)
- Envoyer l’ID de modèle lors de l’[inscription DPS](./concepts-developer-guide-device.md#dps-payload) dans la charge utile de configuration DPS.
- Annoncer l’ID de modèle lors de la [connexion MQTT](./concepts-developer-guide-device.md#model-id-announcement).
- Tous les modèles d’appareils doivent être compatibles avec [Azure IoT Central](../iot-central/core/overview-iot-central-developer.md).

> [!NOTE]
> Actuellement, Azure IoT Central ne prend pas entièrement en charge les types de données DTDL **Array** et **Geospatial**.

## <a name="test-with-the-azure-iot-extension-cli"></a>Tester avec la CLI de l’extension Azure IoT

L'[extension de l’interface CLI Azure IoT](/cli/azure/ext/azure-iot/iot/product?preserve-view=true&view=azure-cli-latest) vous permet de vérifier que l’implémentation de l’appareil correspond au modèle avant de soumettre l’appareil à des fins de certification par le biais du portail Azure Certified Device.

La procédure suivante vous montre comment préparer et exécuter les tests de certification à l’aide de l’interface CLI :

### <a name="install-the-azure-iot-extension-for-the-azure-cli"></a>Installer l’extension Azure IoT pour Azure CLI

Consultez les instructions d’installation pour configurer [Azure CLI](/cli/azure/?preserve-view=true&view=azure-cli-latest) dans votre environnement.

Pour installer l’extension Azure IoT, exécutez la commande suivante :

```azurecli
az extension add --name azure-iot
```

Pour plus d’informations, consultez [Azure CLI pour Azure IoT](/cli/azure/azure-cli-reference-for-iot?preserve-view=true&view=azure-cli-latest).

### <a name="create-a-new-product-test"></a>Créer un test de produit

La commande suivante crée un test à l’aide du DPS avec une méthode d’attestation de clé symétrique :

- Crée un nouveau produit à tester et génère une configuration de test. La sortie affiche les informations du DPS que l’appareil doit utiliser à des fins d’approvisionnement : la clé primaire, l’ID de l’appareil et l’étendue de l’ID.
- Spécifie le dossier contenant les fichiers DTDL décrivant votre modèle.

```azurecli
az iot product test create --badge-type Pnp --at SymmetricKey --device-type FinishedProduct --models {local folder name}
```

> [!NOTE]
> Vous devez [vous connecter](/cli/azure/authenticate-azure-cli?preserve-view=true&view=azure-cli-latest) à votre abonnement lorsque vous utilisez l’interface CLI.

La sortie JSON de la commande contient les `primaryKey`, `registrationId`et `scopeID` à utiliser lorsque vous connectez votre appareil.

Sortie attendue :

```json
"deviceType": "FinishedProduct",
"id": "d45d53d9-656d-4be7-9bbf-140bc87e98dc",
"provisioningConfiguration": {
  "symmetricKeyEnrollmentInformation": {
    "primaryKey":"Ci/Ghpqp0n4m8FD5PTicr6dEikIfM3AtVWzAhObU7yc=",
    "registrationId": "d45d53d9-656d-4be7-9bbf-140bc87e98dc",
    "scopeId": "0ne000FFA42"
  }
}
```

### <a name="connect-your-device"></a>Connexion de votre appareil

Utilisez la sortie des informations DPS de la commande précédente pour connecter votre appareil à l’instance de test IoT Hub.

### <a name="manage-and-configure-the-product-tests"></a>Gérer et configurer les tests de produit

Lorsque l’appareil est connecté et prêt à interagir avec IoT Hub, générez un fichier de configuration de test de produit. Pour créer le fichier :

- Utilisez le test `id` de la sortie de la commande précédente.
- Utilisez le paramètre `--wait` pour récupérer le cas de test.

```azurecli
az iot product test task create --type GenerateTestCases --test-id d45d53d9-656d-4be7-9bbf-140bc87e98dc --wait
```

Sortie attendue :

```json
{
  "deviceTestId": "d45d53d9-656d-4be7-9bbf-140bc87e98dc",
  "error": null,
  "id": "526da38e-91fc-4e20-a761-4f04b392c42b",
  "resultLink": "/deviceTests/d45d53d9-656d-4be7-9bbf-140bc87e98dc/TestCases",
  "status": "Completed",
  "type": "GenerateTestCases"
}
```

Vous pouvez utiliser la commande `az iot product test case update` pour modifier le fichier de configuration de test.

### <a name="run-the-tests"></a>Exécuter les tests

Une fois la configuration de test générée, l’étape suivante consiste à exécuter les tests. Utilisez le même `devicetestId` que celui des commandes précédentes en tant que paramètre pour exécuter les tests. Vérifiez les résultats des tests pour vous assurer que leur état correspond à `Passed`.

```azurecli
az iot product test task create --type QueueTestRun --test-id d45d53d9-656d-4be7-9bbf-140bc87e98dc --wait
```

Exemple de sortie d’exécution de test

```json
      "validationTasks": [
        {
          "componentName": "Default component",
          "endTime": "2020-08-25T05:18:49.5224772+00:00",
          "interfaceId": "dtmi:com:example:TemperatureController;1",
          "logs": [
            {
              "message": "Waiting for telemetry from the device",
              "time": "2020-08-25T05:18:37.3862586+00:00"
            },
            {
              "message": "Validating PnP properties",
              "time": "2020-08-25T05:18:37.3875168+00:00"
            },
            {
              "message": "Validating PnP commands",
              "time": "2020-08-25T05:18:37.3894343+00:00"
            },
            {
              "message": "{\"propertyName\":\"serialNumber\",\"expectedSchemaType\":null,\"actualSchemaType\":null,\"message\":\"Property is successfully validated\",\"passed\":true,\"time\":\"2020-08-25T05:18:37.4205985+00:00\"}",
              "time": "2020-08-25T05:18:37.4205985+00:00"
            },
            {
              "message": "PnP interface properties validation passed",
              "time": "2020-08-25T05:18:37.4206964+00:00"
            },
```

## <a name="test-using-the-azure-certified-device-portal"></a>Tester à l’aide du portail Azure Certified Device

La procédure suivante vous montre comment utiliser le [portail Azure Certified Device](https://aka.ms/acdp) pour intégrer, inscrire les détails du produit, soumettre un guide de prise en main et exécuter les tests de certification.

### <a name="onboarding"></a>Mise en route

Pour utiliser le [portail de certification](https://aka.ms/acdp), vous devez utiliser un annuaire Azure Active Directory de votre locataire professionnel ou scolaire.

Pour publier les modèles dans le référentiel de modèles public Azure IoT, votre compte doit être membre de [Microsoft Partner Network](https://partner.microsoft.com). Le système vérifie que l’ID de Microsoft Partner Network existe et que le compte est entièrement approuvé avant la publication sur le catalogue d’appareils.

### <a name="company-profile"></a>Profil de l’entreprise

Vous pouvez gérer votre profil d’entreprise dans le menu de navigation de gauche. Le profil d’entreprise comprend l’URL, l’adresse e-mail et le logo de l’entreprise. Le contrat du programme doit être accepté sur cette page avant d’exécuter toute opération de certification.

Les informations du profil d’entreprise sont utilisées dans la description de l’appareil présentée dans le catalogue d’appareils.

### <a name="create-new-project"></a>Création d’un projet

Pour certifier un appareil, vous devez d’abord créer un projet.

Accédez au [portail de certification](https://aka.ms/acdp). Sur la page **Projets**, sélectionnez *+ Créer un projet*. Entrez ensuite le nom du projet, le nom de l’appareil, puis sélectionnez une classe d’appareil.

Les informations produit que vous fournissez lors du processus de certification se répartissent en quatre catégories :

- Informations sur l'appareil. Collecte des informations sur l’appareil, telles que son nom, sa description, ses certifications et son système d’exploitation.
- Guide de **prise en main**. Vous devez soumettre ce guide sous forme de document PDF pour qu’il soit approuvé par l’administrateur système avant de publier l’appareil.
- Détails marketing Fournissez des informations marketing adaptées aux clients pour votre appareil. Les informations marketing comprennent une description, une photo et les distributeurs.
- Autres certifications de l’industrie. Cette section facultative vous permet de fournir des informations supplémentaires sur les autres certifications dont dispose l’appareil.

### <a name="connect-and-test"></a>Connecter et tester

L’étape de connexion et de test vérifie que votre appareil répond aux exigences de certification IoT Plug-and-Play.

Trois étapes sont à effectuer :

1. Connecter et découvrir des interfaces. L’appareil doit se connecter au service de certification Azure IoT via DPS. Choisissez la méthode d’authentification (certificat X.509, clés symétriques ou module de plateforme sécurisée) pour utiliser et mettre à jour l’application de l’appareil avec les informations du DPS.
1. Passer en revue les interfaces. Passez en revue l’interface et assurez-vous que chacune dispose d’entrées de charge utile adaptées aux tests.
1. Effectuer des tests. Le système teste chaque modèle d’appareil pour vérifier que les données de télémétrie, les propriétés et les commandes décrites dans le modèle suivent les conventions IoT Plug-and-Play. Une fois le test terminé, sélectionnez le lien **afficher les journaux** pour afficher les données de télémétrie de l’appareil et les données brutes envoyées aux propriétés de jumeau d’appareil IoT Hub.

### <a name="submit-and-publish"></a>Soumettre et publier

La dernière étape requise consiste à soumettre le projet pour revue. Cette étape indique à un membre de l’équipe Azure Certified Device de vérifier que votre projet contient toutes les informations requises, notamment les détails de l’appareil, les données marketing et le guide de démarrage. Il est possible qu’un membre de l’équipe vous envoie des questions à l’adresse e-mail professionnelle précédemment fournie ou modifie les requêtes avant l’approbation.

Si votre appareil nécessite une validation manuelle supplémentaire dans le cadre de la certification, vous recevrez une notification au moment opportun.

Lorsqu’un appareil est certifié, vous pouvez choisir de publier les détails de votre produit dans le catalogue Azure Certified Device à l’aide de la fonctionnalité **Publier dans le catalogue** dans la page de résumé du produit.

## <a name="next-steps"></a>Étapes suivantes

Une fois la soumission de l’appareil terminée, vous pouvez contacter l’équipe de certification de l’appareil à l’adresse [iotcert@microsoft.com](mailto:iotcert@microsoft.com) pour passer aux étapes suivantes, notamment la validation de l’appartenance à Microsoft Partner Network et l’examen des guides de prise en main. Lorsque toutes les conditions requises sont satisfaites, vous pouvez choisir de faire figurer votre appareil dans le [catalogue des appareils certifiés pour Azure IoT](https://aka.ms/devicecatalog).