---
title: Installer des conteneurs Docker OCR Read à partir de Vision par ordinateur
titleSuffix: Azure Cognitive Services
description: Utilisez les conteneurs Docker OCR Read de Vision par ordinateur pour extraire du texte d’images et de documents localement.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 11/23/2020
ms.author: aahi
ms.custom: seodec18, cog-serv-seo-aug-2020
keywords: local, OCR, Docker, conteneur
ms.openlocfilehash: b89d02107365872471f1dd5a7df07902b08f2031
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96006902"
---
# <a name="install-read-ocr-docker-containers-preview"></a>Installer des conteneurs Docker OCR Read (préversion) 

[!INCLUDE [container hosting on the Microsoft Container Registry](../containers/includes/gated-container-hosting.md)]

Les conteneurs vous permettent d’exécuter les API Vision par ordinateur dans votre propre environnement. Les conteneurs conviennent particulièrement bien à certaines exigences de sécurité et de gouvernance des données. Dans cet article, vous allez apprendre à télécharger, installer et exécuter des conteneurs Vision par ordinateur.

Le conteneur *Read* OCR vous permet d'extraire du texte imprimé et manuscrit à partir d'images et de documents avec la prise en charge des formats de fichier JPEG, PNG, BMP, PDF et TIFF. Pour plus d’informations, consultez la [documentation sur l’API Read](concept-recognizing-text.md#read-api).

## <a name="read-32-preview-container"></a>Conteneur Read 3.2-preview

> [!NOTE]
> Le conteneur Read 3.0-preview est déconseillé. 

Le conteneur Read 3.2-preview offre ce qui suit :
* Nouveaux modèles pour une précision accrue.
* Prise en charge de plusieurs langues dans le même document
* Prise en charge des langues suivantes : néerlandais, anglais, français, allemand, italien, portugais et espagnol.
* Une seule opération pour les documents et les images.
* Prise en charge des documents et des images de grande taille.
* Scores de confiance compris entre 0 et 1.
* Prise en charge des documents contenant à la fois du texte imprimé et du texte manuscrit
* Prise en charge du chinois simplifié et du japonais.
* Scores de confiance et étiquettes pour texte imprimé et texte manuscrit. 
* Possibilité d'extraire le texte de la page ou des pages sélectionnées dans un document.

Si vous utilisez actuellement des conteneurs Read 2.0, consultez le [guide de migration](read-container-migration-guide.md) pour en savoir plus sur les modifications apportées aux nouvelles versions.

## <a name="prerequisites"></a>Prérequis

Vous devez respecter les prérequis suivants avant d’utiliser les conteneurs :

|Obligatoire|Objectif|
|--|--|
|Moteur Docker| Vous avez besoin d’un moteur Docker installé sur un [ordinateur hôte](#the-host-computer). Docker fournit des packages qui configurent l’environnement Docker sur [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) et [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Pour apprendre les principes de base de Docker et des conteneurs, consultez la [vue d’ensemble de Docker](https://docs.docker.com/engine/docker-overview/).<br><br> Vous devez configurer Docker pour permettre aux conteneurs de se connecter à Azure et de lui envoyer des données de facturation. <br><br> **Sur Windows**, vous devez également configurer Docker pour prendre en charge les conteneurs Linux.<br><br>|
|Bonne connaissance de Docker | Vous devez avoir une compréhension élémentaire des concepts Docker, notamment les registres, référentiels, conteneurs et images conteneurs, ainsi qu’une maîtrise des commandes `docker` de base.| 
|Ressource Vision par ordinateur |Pour pouvoir utiliser le conteneur, vous devez disposer des éléments suivants :<br><br>Une ressource **Vision par ordinateur** Azure, la clé d’API associée et l’URI de point de terminaison. Les deux valeurs, disponibles dans les pages Vue d’ensemble et Clés de la ressource, sont nécessaires au démarrage du conteneur.<br><br>**{API_KEY}**  : L’une des deux clés de ressource disponibles à la page **Clés**<br><br>**{ENDPOINT_URI}**  : Le point de terminaison tel qu'il est fourni à la page **Vue d’ensemble**|

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/) avant de commencer.

## <a name="request-approval-to-run-the-container"></a>Demande d’approbation pour l’exécution du conteneur

Complétez le [formulaire de demande](https://aka.ms/csgate) et envoyez-le afin d’obtenir l’approbation d’exécuter le conteneur. 

[!INCLUDE [Request access to public preview](../../../includes/cognitive-services-containers-request-access.md)]

[!INCLUDE [Gathering required container parameters](../containers/includes/container-gathering-required-parameters.md)]

### <a name="the-host-computer"></a>L’ordinateur hôte

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="advanced-vector-extension-support"></a>Prise en charge d’Advanced Vector Extension

L’ordinateur **hôte** est l’ordinateur qui exécute le conteneur docker. L’hôte *doit prendre en charge* [Advanced Vector Extensions](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#CPUs_with_AVX2) (AVX2). Vous pouvez vérifier cette prise en charge AVX2 sur les hôtes Linux avec la commande suivante :

```console
grep -q avx2 /proc/cpuinfo && echo AVX2 supported || echo No AVX2 support detected
```

> [!WARNING]
> L’ordinateur hôte doit *obligatoirement* prendre en charge AVX2. Le conteneur ne fonctionnera *pas* correctement sans prise en charge AVX2.

### <a name="container-requirements-and-recommendations"></a>Exigences et suggestions relatives au conteneur

[!INCLUDE [Container requirements and recommendations](includes/container-requirements-and-recommendations.md)]

## <a name="get-the-container-image-with-docker-pull"></a>Obtenir l’image conteneur avec `docker pull`

Des images conteneurs sont disponibles pour le conteneur Lire.

| Conteneur | Nom de registre de conteneurs / référentiel / image |
|-----------|------------|
| Read 2.0 (préversion) | `mcr.microsoft.com/azure-cognitive-services/vision/read:2.0-preview` |
| Read 3.2-preview | `mcr.microsoft.com/azure-cognitive-services/vision/read:3.2-preview.1` |

Utilisez la commande [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) pour télécharger une image conteneur.

### <a name="docker-pull-for-the-read-container"></a>Commande docker pull du conteneur Lire

# <a name="version-32-preview"></a>[Version 3.2-preview](#tab/version-3-2)

```bash
docker pull mcr.microsoft.com/azure-cognitive-services/vision/read:3.2-preview.1
```

# <a name="version-20-preview"></a>[Version 2.0-preview](#tab/version-2)

```bash
docker pull mcr.microsoft.com/azure-cognitive-services/vision/read:2.0-preview
```

---

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

## <a name="how-to-use-the-container"></a>Comment utiliser le conteneur

Une fois que le conteneur est sur l’[ordinateur hôte](#the-host-computer), appliquez la procédure suivante pour travailler avec le conteneur.

1. [Exécutez le conteneur](#run-the-container-with-docker-run) avec les paramètres de facturation requis. D’autres [exemples](computer-vision-resource-container-config.md) de commande `docker run` sont disponibles. 
1. [Interrogez le point de terminaison de prédiction du conteneur](#query-the-containers-prediction-endpoint). 

## <a name="run-the-container-with-docker-run"></a>Exécuter le conteneur avec `docker run`

Utilisez la commande [docker run](https://docs.docker.com/engine/reference/commandline/run/) pour exécuter le conteneur. Pour plus d’informations sur la façon d’obtenir les valeurs `{ENDPOINT_URI}` et `{API_KEY}`, consultez [Collecte des paramètres requis](#gathering-required-parameters).

[Exemples ](computer-vision-resource-container-config.md#example-docker-run-commands) de la commande `docker run` sont disponibles.

# <a name="version-32-preview"></a>[Version 3.2-preview](#tab/version-3-2)

```bash
docker run --rm -it -p 5000:5000 --memory 18g --cpus 8 \
mcr.microsoft.com/azure-cognitive-services/vision/read:3.2-preview.1 \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Cette commande :

* Exécute un conteneur Lire à partir de l’image conteneur.
* Alloue 8 cœurs de processeur et 18 gigaoctets (Go) de mémoire.
* Expose le port TCP 5000 et alloue un pseudo-TTY pour le conteneur.
* Supprime automatiquement le conteneur après sa fermeture. L’image conteneur est toujours disponible sur l’ordinateur hôte.

# <a name="version-20-preview"></a>[Version 2.0-preview](#tab/version-2)

```bash
docker run --rm -it -p 5000:5000 --memory 16g --cpus 8 \
mcr.microsoft.com/azure-cognitive-services/vision/read:2.0-preview \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Cette commande :

* Exécute un conteneur Lire à partir de l’image conteneur.
* Alloue 8 cœurs de processeur et 16 gigaoctets (Go) de mémoire.
* Expose le port TCP 5000 et alloue un pseudo-TTY pour le conteneur.
* Supprime automatiquement le conteneur après sa fermeture. L’image conteneur est toujours disponible sur l’ordinateur hôte.

---


D’autres [exemples](./computer-vision-resource-container-config.md#example-docker-run-commands) de commande `docker run` sont disponibles. 

> [!IMPORTANT]
> Vous devez spécifier les options `Eula`, `Billing` et `ApiKey` pour exécuter le conteneur, sinon il ne démarrera pas.  Pour plus d'informations, consultez [Facturation](#billing).

Si vous avez besoin d’un débit plus élevé (par exemple, pour le traitement de fichiers multipages), vous pouvez envisagez de déployer plusieurs conteneurs [sur un cluster Kubernetes](deploy-computer-vision-on-premises.md) en utilisant [Stockage Azure](../../storage/common/storage-account-create.md) et [File d’attente Azure](../../storage/queues/storage-queues-introduction.md).

Si vous vous servez du stockage Azure pour stocker des images à traiter, vous pouvez créer une [chaîne de connexion](../../storage/common/storage-configure-connection-string.md) à utiliser au moment de l’appel du conteneur.

Pour rechercher votre chaîne de connexion :

1. Sur le Portail Azure, accédez à **Comptes de stockage** et recherchez votre compte.
2. Dans la liste de navigation de gauche, cliquez sur **Clés d’accès**.
3. Votre chaîne de connexion se trouve sous **Chaîne de connexion**.

[!INCLUDE [Running multiple containers on the same host](../../../includes/cognitive-services-containers-run-multiple-same-host.md)]

<!--  ## Validate container is running -->

[!INCLUDE [Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="query-the-containers-prediction-endpoint"></a>Interroger le point de terminaison de prédiction du conteneur

Le conteneur fournit des API de point de terminaison de prédiction de requête basées sur REST. 

# <a name="version-32-preview"></a>[Version 3.2-preview](#tab/version-3-2)

Utilisez l’hôte, `http://localhost:5000`, pour les API de conteneur. Le chemin d'accès à Swagger est visible à l'adresse : `http://localhost:5000/swagger/vision-v3.2-preview-read/swagger.json`.

# <a name="version-20-preview"></a>[Version 2.0-preview](#tab/version-2)

Utilisez l’hôte, `http://localhost:5000`, pour les API de conteneur. Le chemin d'accès à Swagger est visible à l'adresse : `http://localhost:5000/swagger/vision-v2.0-preview-read/swagger.json`.

---

### <a name="asynchronous-read"></a>Lecture asynchrone


# <a name="version-32-preview"></a>[Version 3.2-preview](#tab/version-3-2)

Vous pouvez utiliser conjointement les opérations `POST /vision/v3.2/read/analyze` et `GET /vision/v3.2/read/operations/{operationId}` pour lire de façon asynchrone une image, ce qui est similaire à la façon dont le service Vision par ordinateur utilise ces opérations REST correspondantes. La méthode POST asynchrone retourne un `operationId` qui est utilisé comme identificateur de la requête HTTP GET.


À partir de l’interface utilisateur Swagger, sélectionnez le `asyncBatchAnalyze` pour le développer dans le navigateur. Ensuite, sélectionnez **Faites un essai** > **Choisir un fichier**. Dans cet exemple, nous allons utiliser l’image suivante :

![Tabulations et espaces](media/tabs-vs-spaces.png)

Lorsque le POST asynchrone s’est correctement exécuté, il retourne le code d’état **HTTP 202**. Dans la réponse, un en-tête `operation-location` contient le point de terminaison de résultat de la requête.

```http
 content-length: 0
 date: Fri, 04 Sep 2020 16:23:01 GMT
 operation-location: http://localhost:5000/vision/v3.2/read/operations/a527d445-8a74-4482-8cb3-c98a65ec7ef9
 server: Kestrel
```

`operation-location` est l’URL complète qui est accessible via HTTP GET. Voici la réponse JSON à l’exécution de l’URL `operation-location` à partir de l’image précédente :

```json
{
  "status": "succeeded",
  "createdDateTime": "2020-09-02T10:30:14Z",
  "lastUpdatedDateTime": "2020-09-02T10:30:15Z",
  "analyzeResult": {
    "version": "3.2.0",
    "readResults": [
      {
        "page": 1,
        "angle": 2.12,
        "width": 502,
        "height": 252,
        "unit": "pixel",
        "language": "",
        "lines": [
          {
            "boundingBox": [58, 42, 314, 59, 311, 123, 56, 121],
            "text": "Tabs vs",
            "appearance": {
              "style": "handwriting",
              "styleConfidence": 0.999
            },
            "words": [
              {
                "boundingBox": [85, 45, 242, 62, 241, 122, 83, 123],
                "text": "Tabs",
                "confidence": 0.981
              },
              {
                "boundingBox": [258, 64, 314, 72, 314, 123, 256, 123],
                "text": "vs",
                "confidence": 0.958
              }
            ]
          },
          {
            "boundingBox": [286, 171, 415, 165, 417, 197, 287, 201],
            "text": "paces",
            "appearance": {
              "style": "print",
              "styleConfidence": 0.603
            },
            "words": [
              {
                "boundingBox": [303, 175, 415, 167, 415, 198, 306, 199],
                "text": "paces",
                "confidence": 0.918
              }
            ]
          }
        ]
      }
    ]
  }
}
```

# <a name="version-20-preview"></a>[Version 2.0-preview](#tab/version-2)

Vous pouvez utiliser conjointement les opérations `POST /vision/v2.0/read/core/asyncBatchAnalyze` et `GET /vision/v2.0/read/operations/{operationId}` pour lire de façon asynchrone une image, ce qui est similaire à la façon dont le service Vision par ordinateur utilise ces opérations REST correspondantes. La méthode POST asynchrone retourne un `operationId` qui est utilisé comme identificateur de la requête HTTP GET.

À partir de l’interface utilisateur Swagger, sélectionnez le `asyncBatchAnalyze` pour le développer dans le navigateur. Ensuite, sélectionnez **Faites un essai** > **Choisir un fichier**. Dans cet exemple, nous allons utiliser l’image suivante :

![Tabulations et espaces](media/tabs-vs-spaces.png)

Lorsque le POST asynchrone s’est correctement exécuté, il retourne le code d’état **HTTP 202**. Dans la réponse, un en-tête `operation-location` contient le point de terminaison de résultat de la requête.

```http
 content-length: 0
 date: Fri, 13 Sep 2019 16:23:01 GMT
 operation-location: http://localhost:5000/vision/v2.0/read/operations/a527d445-8a74-4482-8cb3-c98a65ec7ef9
 server: Kestrel
```

`operation-location` est l’URL complète qui est accessible via HTTP GET. Voici la réponse JSON à l’exécution de l’URL `operation-location` à partir de l’image précédente :

```json
{
  "status": "Succeeded",
  "recognitionResults": [
    {
      "page": 1,
      "clockwiseOrientation": 2.42,
      "width": 502,
      "height": 252,
      "unit": "pixel",
      "lines": [
        {
          "boundingBox": [ 56, 39, 317, 50, 313, 134, 53, 123 ],
          "text": "Tabs VS",
          "words": [
            {
              "boundingBox": [ 90, 43, 243, 53, 243, 123, 94, 125 ],
              "text": "Tabs",
              "confidence": "Low"
            },
            {
              "boundingBox": [ 259, 55, 313, 62, 313, 122, 259, 123 ],
              "text": "VS"
            }
          ]
        },
        {
          "boundingBox": [ 221, 148, 417, 146, 417, 206, 227, 218 ],
          "text": "Spaces",
          "words": [
            {
              "boundingBox": [ 230, 148, 416, 141, 419, 211, 232, 218 ],
              "text": "Spaces"
            }
          ]
        }
      ]
    }
  ]
}
```

---

> [!IMPORTANT]
> Si vous déployez plusieurs conteneurs de lecture derrière un équilibreur de charge, par exemple, sous Docker Compose ou Kubernetes, vous devez disposer d’un cache externe. Étant donné que le conteneur de traitement et le conteneur de requêtes GET peuvent être différents, un cache externe est utilisé pour stocker les résultats et les partager entre les conteneurs. Pour plus d’informations sur les paramètres de cache, consultez l’article [Configurer les conteneurs Docker Vision par ordinateur](./computer-vision-resource-container-config.md).

### <a name="synchronous-read"></a>Lecture synchrone

Vous pouvez utiliser l’opération suivante pour lire une image en mode synchrone. 

# <a name="version-32-preview"></a>[Version 3.2-preview](#tab/version-3-2)

`POST /vision/v3.2/read/syncAnalyze` 

# <a name="version-20-preview"></a>[Version 2.0-preview](#tab/version-2)

`POST /vision/v2.0/read/core/Analyze`

---

C’est uniquement lorsque l’image est lue dans son intégralité que l’API retourne une réponse JSON. La seule exception à ceci est si une erreur se produit. Lorsqu’une erreur se produit, le code JSON suivant est retourné :

```json
{
    "status": "Failed"
}
```

L’objet de réponse JSON contient le même graphe d’objets que la version asynchrone. Si vous êtes un utilisateur JavaScript et visez la cohérence des types, vous pouvez envisager d’utiliser TypeScript pour caster la réponse JSON.

Pour obtenir un exemple de cas d’utilisation, consultez <a href="https://aka.ms/ts-read-api-types" target="_blank" rel="noopener noreferrer">ce bac à sable TypeScript <span class="docon docon-navigate-external x-hidden-focus"></span></a>, puis sélectionnez **Exécuter** pour vous rendre compte de sa facilité d’utilisation.

## <a name="stop-the-container"></a>Arrêter le conteneur

[!INCLUDE [How to stop the container](../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>Dépannage

Si vous exécutez le conteneur avec un [montage](./computer-vision-resource-container-config.md#mount-settings) de sortie et la journalisation activée, il génère des fichiers journaux qui sont utiles pour résoudre les problèmes qui se produisent lors du démarrage ou de l’exécution du conteneur.

[!INCLUDE [Cognitive Services FAQ note](../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Facturation

Les conteneurs Cognitive Services envoient des informations de facturation à Azure à l’aide de la ressource correspondante dans votre compte Azure.

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Pour plus d’informations sur ces options, consultez [Configurer des conteneurs](./computer-vision-resource-container-config.md).

<!--blogs/samples/video course -->

[!INCLUDE [Discoverability of more container information](../../../includes/cognitive-services-containers-discoverability.md)]

## <a name="summary"></a>Résumé

Dans cet article, vous avez découvert des concepts et le flux de travail pour le téléchargement, l’installation et l’exécution des conteneurs Vision par ordinateur. En résumé :

* Vision par ordinateur fournit un conteneur Linux pour Docker, et encapsule le conteneur Lire.
* Les images conteneur sont téléchargées à partir du registre de conteneurs « Conteneur (préversion) » dans Azure.
* Les images conteneurs s’exécutent dans Docker.
* Vous pouvez utiliser l’API REST ou kit de développement logiciel (SDK) pour appeler des opérations dans des conteneurs Lire en spécifiant l’URI hôte du conteneur.
* Vous devez spécifier les informations de facturation lors de l’instanciation d’un conteneur.

> [!IMPORTANT]
> Les conteneurs Cognitives Services ne sont pas concédés sous licence pour s’exécuter sans être connectés à Azure pour le contrôle. Les clients doivent configurer les conteneurs de manière à ce qu’ils communiquent les informations de facturation au service de contrôle à tout moment. Les conteneurs Cognitive Services n’envoient pas de données relatives aux clients (par exemple, l’image ou le texte en cours d’analyse) à Microsoft.

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir les paramètres de configuration, passez en revue [Configurer des conteneurs](computer-vision-resource-container-config.md).
* Pour en savoir plus sur la reconnaissance du texte imprimé et manuscrit, passez en revue [Vue d’ensemble de Vision par ordinateur](overview.md).
* Pour plus d’informations sur les méthodes prises en charge par le conteneur, reportez-vous à l’[API Vision par ordinateur](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/56f91f2e778daf14a499f21b).
* Pour résoudre les problèmes liés à la fonctionnalité Vision par ordinateur, reportez-vous au [Forum aux questions (FAQ)](FAQ.md).
* Utiliser davantage de [conteneurs Cognitive Services](../cognitive-services-container-support.md)