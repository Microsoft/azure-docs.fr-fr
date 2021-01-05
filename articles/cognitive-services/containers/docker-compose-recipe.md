---
title: Utiliser Docker Compose pour déployer plusieurs conteneurs
titleSuffix: Azure Cognitive Services
description: Découvrez comment déployer plusieurs conteneurs Cognitive Services. Cet article vous montre comment orchestrer plusieurs images conteneur Docker avec Docker Compose.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 10/29/2020
ms.author: aahi
ms.openlocfilehash: cedcf8a3fcd656c4af0ca7493c598791d35d20d9
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996121"
---
# <a name="use-docker-compose-to-deploy-multiple-containers"></a>Utiliser Docker Compose pour déployer plusieurs conteneurs

Cet article vous montre comment déployer plusieurs conteneurs Azure Cognitive Services. Plus précisément, vous allez découvrir comment utiliser Docker Compose pour orchestrer plusieurs images conteneur Docker.

> [Docker Compose](https://docs.docker.com/compose/) est un outil destiné à définir et exécuter des applications Docker à plusieurs conteneurs. Dans Compose, vous utilisez un fichier YAML pour configurer les services de votre application. Ensuite, vous créez et vous démarrez tous les services à partir de votre configuration en utilisant une seule commande.

Il peut être utile d’orchestrer plusieurs images conteneur sur un même ordinateur hôte. Dans cet article, nous allons regrouper les conteneurs Read et Form Recognizer.

## <a name="prerequisites"></a>Prérequis

Cette procédure nécessite plusieurs outils qui doivent être installés et exécutés localement :

* Un abonnement Azure. Si vous n’en avez pas, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services) avant de commencer.
* [Moteur Docker](https://www.docker.com/products/docker-engine). Vérifiez que l’interface CLI Docker fonctionne dans une fenêtre de console.
* Une ressource Azure avec le niveau de tarification approprié. Seuls les niveaux tarifaires suivants fonctionnent avec ce conteneur :
  * La ressource **Vision par ordinateur** avec le niveau tarifaire F0 ou Standard.
  * La ressource **Form Recognizer** avec le niveau tarifaire F0 ou Standard.
  * La ressource **Cognitive Services** avec un niveau tarifaire S0.
* Si vous utilisez un conteneur en préversion contrôlée, vous devez remplir le [formulaire de demande en ligne](https://aka.ms/csgate/) pour l’utiliser.

## <a name="docker-compose-file"></a>Fichier Docker Compose

Le fichier YAML définit tous les services à déployer. Ces services s’appuient sur `DockerFile` ou sur une image conteneur existante. Dans ce cas, nous allons utiliser deux images en préversion. Copiez et collez le fichier YAML suivant, puis enregistrez-le sous le nom *docker-compose.yaml*. Entrez les valeurs appropriées pour **apikey**, **billing** et **EndpointUri** dans le fichier.

```yaml
version: '3.7'
services:
  forms:
    image: "mcr.microsoft.com/azure-cognitive-services/form-recognizer/layout"
    environment:
       eula: accept
       billing: # < Your form recognizer billing URL >
       apikey: # < Your form recognizer API key >
       FormRecognizer__ComputerVisionApiKey: # < Your form recognizer API key >
       FormRecognizer__ComputerVisionEndpointUri: # < Your form recognizer URI >
    volumes:
       - type: bind
         source: E:\publicpreview\output
         target: /output
       - type: bind
         source: E:\publicpreview\input
         target: /input
    ports:
      - "5010:5000"

  ocr:
    image: "mcr.microsoft.com/azure-cognitive-services/vision/read:3.1-preview"
    environment:
      eula: accept
      apikey: # < Your computer vision API key >
      billing: # < Your computer vision billing URL >
    ports:
      - "5021:5000"
```

> [!IMPORTANT]
> Créez les répertoires qui sont spécifiés sous le nœud **volumes** sur la machine hôte. Cette approche est nécessaire, car les répertoires doivent exister avant une tentative pour monter une image en utilisant des liaisons de volume.

## <a name="start-the-configured-docker-compose-services"></a>Démarrer les services Docker Compose configurés

Un fichier Docker Compose permet de gérer tous les étapes du cycle de vie du service défini : démarrage, arrêt et reconstruction des services, affichage de l’état du service et streaming de journaux. Ouvrez une interface de ligne de commande à partir du répertoire du projet (où le fichier docker-compose.yaml se trouve).

> [!NOTE]
> Pour éviter les erreurs, vérifiez que la machine hôte partage correctement les lecteurs avec le moteur Docker. Par exemple, si *E:\publicpreview* est utilisé comme répertoire dans le fichier *docker-compose.yaml*, partagez le lecteur **E** avec Docker.

À partir de l’interface de ligne de commande, exécutez la commande suivante pour démarrer (ou redémarrer) tous les services définis dans le fichier *docker-compose.yaml* :

```console
docker-compose up
```

La première fois que Docker exécute la commande **docker-compose up** en utilisant cette configuration, il extrait les images configurées sous le nœud **services**, puis les télécharge et les monte :

```console
Pulling forms (mcr.microsoft.com/azure-cognitive-services/form-recognizer/layout:)...
latest: Pulling from azure-cognitive-services/form-recognizer/layout
743f2d6c1f65: Pull complete
72befba99561: Pull complete
2a40b9192d02: Pull complete
c7715c9d5c33: Pull complete
f0b33959f1c4: Pull complete
b8ab86c6ab26: Pull complete
41940c21ed3c: Pull complete
e3d37dd258d4: Pull complete
cdb5eb761109: Pull complete
fd93b5f95865: Pull complete
ef41dcbc5857: Pull complete
4d05c86a4178: Pull complete
34e811d37201: Pull complete
Pulling ocr (mcr.microsoft.com/azure-cognitive-services/vision/read:3.1-preview:)...
latest: Pulling from /azure-cognitive-services/vision/read:3.1-preview
f476d66f5408: Already exists
8882c27f669e: Already exists
d9af21273955: Already exists
f5029279ec12: Already exists
1a578849dcd1: Pull complete
45064b1ab0bf: Download complete
4bb846705268: Downloading [=========================================>         ]  187.1MB/222.8MB
c56511552241: Waiting
e91d2aa0f1ad: Downloading [==============================================>    ]  162.2MB/176.1MB
```

Une fois les images téléchargées, les services d’image sont démarrés :

```console
Starting docker_ocr_1   ... done
Starting docker_forms_1 ... doneAttaching to docker_ocr_1, docker_forms_1forms_1  | forms_1  | forms_1  | Notice: This Preview is made available to you on the condition that you agree to the Supplemental Terms of Use for Microsoft Azure Previews [https://go.microsoft.com/fwlink/?linkid=2018815], which supplement your agreement [https://go.microsoft.com/fwlink/?linkid=2018657] governing your use of Azure. If you do not have an existing agreement governing your use of Azure, you agree that your agreement governing use of Azure is the Microsoft Online Subscription Agreement [https://go.microsoft.com/fwlink/?linkid=2018755] (which incorporates the Online Services Terms [https://go.microsoft.com/fwlink/?linkid=2018760]). By using the Preview you agree to these terms.
forms_1  | 
forms_1  | 
forms_1  | Using '/input' for reading models and other read-only data.
forms_1  | Using '/output/forms/812d811d1bcc' for writing logs and other output data.
forms_1  | Logging to console.
forms_1  | Submitting metering to 'https://westus2.api.cognitive.microsoft.com/'.
forms_1  | WARNING: No access control enabled!
forms_1  | warn: Microsoft.AspNetCore.Server.Kestrel[0]
forms_1  |       Overriding address(es) 'http://+:80'. Binding to endpoints defined in UseKestrel() instead.
forms_1  | Hosting environment: Production
forms_1  | Content root path: /app/forms
forms_1  | Now listening on: http://0.0.0.0:5000
forms_1  | Application started. Press Ctrl+C to shut down.
ocr_1    | 
ocr_1    | 
ocr_1    | Notice: This Preview is made available to you on the condition that you agree to the Supplemental Terms of Use for Microsoft Azure Previews [https://go.microsoft.com/fwlink/?linkid=2018815], which supplement your agreement [https://go.microsoft.com/fwlink/?linkid=2018657] governing your use of Azure. If you do not have an existing agreement governing your use of Azure, you agree that your agreement governing use of Azure is the Microsoft Online Subscription Agreement [https://go.microsoft.com/fwlink/?linkid=2018755] (which incorporates the Online Services Terms [https://go.microsoft.com/fwlink/?linkid=2018760]). By using the Preview you agree to these terms.
ocr_1    |
ocr_1    | 
ocr_1    | Logging to console.
ocr_1    | Submitting metering to 'https://westcentralus.api.cognitive.microsoft.com/'.
ocr_1    | WARNING: No access control enabled!
ocr_1    | Hosting environment: Production
ocr_1    | Content root path: /
ocr_1    | Now listening on: http://0.0.0.0:5000
ocr_1    | Application started. Press Ctrl+C to shut down.
```

## <a name="verify-the-service-availability"></a>Vérifier la disponibilité du service

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

Voici un exemple de sortie :

```
IMAGE ID            REPOSITORY                                                                 TAG
2ce533f88e80        mcr.microsoft.com/azure-cognitive-services/form-recognizer/layout          latest
4be104c126c5        mcr.microsoft.com/azure-cognitive-services/vision/read:3.1-preview         latest
```

### <a name="test-containers"></a>Conteneurs de test

Ouvrez un navigateur sur la machine hôte, puis accédez à **localhost** en utilisant le port spécifié dans le fichier *docker-compose.yaml*, comme http://localhost:5021/swagger/index.html. Par exemple, vous pouvez utiliser la fonctionnalité **Essayer** de l’API pour tester le point de terminaison Form Recognizer. Les pages Swagger des deux conteneurs doivent être disponibles et testables.

![Conteneur Form Recognizer](media/form-recognizer-swagger-page.png)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Conteneurs Cognitive Services](../cognitive-services-container-support.md)
