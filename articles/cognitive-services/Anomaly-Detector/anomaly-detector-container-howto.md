---
title: Installer et exécuter des conteneurs Docker pour l’API Détecteur d’anomalies
titleSuffix: Azure Cognitive Services
description: Utilisez les algorithmes de l’API Détecteur d’anomalies pour identifier les anomalies dans vos données, localement à l’aide d’un conteneur Docker.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: conceptual
ms.date: 09/28/2020
ms.author: mbullwin
ms.custom: cog-serv-seo-aug-2020
keywords: local, Docker, conteneur, diffusion en continu, algorithmes
ms.openlocfilehash: 911eb993ea5bb3dcce63057efc2d56d91d5a136b
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94364376"
---
# <a name="install-and-run-docker-containers-for-the-anomaly-detector-api"></a>Installer et exécuter des conteneurs Docker pour l’API Détecteur d’anomalies 

[!INCLUDE [container image location note](../containers/includes/image-location-note.md)]

Les conteneurs vous permettent d’utiliser l’API Détecteur d’anomalies dans votre propre environnement. Les conteneurs conviennent particulièrement bien à certaines exigences de sécurité et de gouvernance des données. Dans cet article, vous allez apprendre à télécharger, installer et exécuter un conteneur Détecteur d'anomalies.

Détecteur d’anomalies offre un conteneur Docker pour l’utilisation locale de l’API. Utilisez le conteneur pour :
* Utiliser les algorithmes de Détecteur d’anomalies sur vos données
* Surveiller les données de diffusion en continu et détecter les anomalies en temps réel.
* Détecter les anomalies tout au long de votre jeu de données par lots. 
* Détecter les points de changement de tendance dans votre jeu de données par lots.
* Ajuster la sensibilité de l’algorithme de détection des anomalies pour mieux ajuster vos données.

Pour plus d’informations sur l’API, consultez :
* [En savoir plus sur le service API Détecteur d’anomalies](https://go.microsoft.com/fwlink/?linkid=2080698&clcid=0x409)

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

L’utilisation des conteneurs Détecteur d’anomalies est soumise aux prérequis suivants :

|Obligatoire|Objectif|
|--|--|
|Moteur Docker| Vous avez besoin d’un moteur Docker installé sur un [ordinateur hôte](#the-host-computer). Docker fournit des packages qui configurent l’environnement Docker sur [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) et [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Pour apprendre les principes de base de Docker et des conteneurs, consultez la [vue d’ensemble de Docker](https://docs.docker.com/engine/docker-overview/).<br><br> Vous devez configurer Docker pour permettre aux conteneurs de se connecter à Azure et de lui envoyer des données de facturation. <br><br> **Sur Windows** , vous devez également configurer Docker pour prendre en charge les conteneurs Linux.<br><br>|
|Bonne connaissance de Docker | Vous devez avoir une compréhension élémentaire des concepts Docker, notamment les registres, référentiels, conteneurs et images conteneurs, ainsi qu’une maîtrise des commandes `docker` de base.|
|Ressource Détecteur d’anomalies |Pour pouvoir utiliser ces conteneurs, vous devez avoir :<br><br>une ressource _Détecteur d'anomalies_ d’Azure afin d’obtenir la clé API et l’URI du point de terminaison associés. Les deux valeurs disponibles dans les pages Clés et Vue d’ensemble de **Détecteur d’anomalies** du Portail Azure sont nécessaires pour le démarrage du conteneur.<br><br>**{API_KEY}**  : L’une des deux clés de ressource disponibles à la page **Clés**<br><br>**{ENDPOINT_URI}**  : Le point de terminaison tel qu'il est fourni à la page **Vue d’ensemble**|

[!INCLUDE [Gathering required container parameters](../containers/includes/container-gathering-required-parameters.md)]

## <a name="the-host-computer"></a>L’ordinateur hôte

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

<!--* [Azure IoT Edge](../../iot-edge/index.yml). For instructions of deploying Anomaly Detector module in IoT Edge, see [How to deploy Anomaly Detector module in IoT Edge](how-to-deploy-anomaly-detector-module-in-iot-edge.md).-->

### <a name="container-requirements-and-recommendations"></a>Exigences et suggestions relatives au conteneur

Le tableau suivant décrit la quantité minimale et recommandée de cœurs de processeur et de mémoire à allouer pour le conteneur Détecteur d’anomalies.

| Requêtes par seconde (RPS) | Minimum | Recommandé |
|-----------|---------|-------------|
| 10 RPS | 4 cœurs, 1 Go de mémoire | 8 cœurs, 2 Go de mémoire |
| 20 RPS | 8 cœurs, 2 Go de mémoire | 16 cœurs, 4 Go de mémoire |

Chaque cœur doit être cadencé à au moins 2,6 gigahertz (GHz).

Le nombre de cœurs et la quantité de mémoire correspondent aux paramètres `--cpus` et `--memory` qui sont utilisés dans le cadre de la commande `docker run`.

## <a name="get-the-container-image-with-docker-pull"></a>Obtenir l’image conteneur avec `docker pull`

Utilisez la commande [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) pour télécharger une image conteneur.

| Conteneur | Référentiel |
|-----------|------------|
| cognitive-services-anomaly-detector | `mcr.microsoft.com/azure-cognitive-services/decision/anomaly-detector:latest` |

<!--
For a full description of available tags, such as `latest` used in the preceding command, see [anomaly-detector](https://go.microsoft.com/fwlink/?linkid=2083827&clcid=0x409) on Docker Hub.
-->
[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

### <a name="docker-pull-for-the-anomaly-detector-container"></a>Commande docker pull du conteneur Détecteur d’anomalies

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/anomaly-detector:latest
```

## <a name="how-to-use-the-container"></a>Comment utiliser le conteneur

Une fois que le conteneur est sur l’[ordinateur hôte](#the-host-computer), appliquez la procédure suivante pour travailler avec le conteneur.

1. [Exécutez le conteneur](#run-the-container-with-docker-run) avec les paramètres de facturation requis. D’autres [exemples](anomaly-detector-container-configuration.md#example-docker-run-commands) de commande `docker run` sont disponibles.
1. [Interrogez le point de terminaison de prédiction du conteneur](#query-the-containers-prediction-endpoint).

## <a name="run-the-container-with-docker-run"></a>Exécuter le conteneur avec `docker run`

Utilisez la commande [docker run](https://docs.docker.com/engine/reference/commandline/run/) pour exécuter le conteneur. Pour plus d’informations sur la façon d’obtenir les valeurs `{ENDPOINT_URI}` et `{API_KEY}`, consultez [Collecte des paramètres requis](#gathering-required-parameters).

[Exemples ](anomaly-detector-container-configuration.md#example-docker-run-commands) de la commande `docker run` sont disponibles.

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/decision/anomaly-detector:latest \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Cette commande :

* Exécute un conteneur Détecteur d’anomalies à partir de l’image conteneur
* Alloue un cœur de processeur et 4 gigaoctets (Go) de mémoire.
* Expose le port TCP 5000 et alloue un pseudo-TTY pour le conteneur
* Supprime automatiquement le conteneur après sa fermeture. L’image conteneur est toujours disponible sur l’ordinateur hôte.

> [!IMPORTANT]
> Vous devez spécifier les options `Eula`, `Billing` et `ApiKey` pour exécuter le conteneur, sinon il ne démarrera pas.  Pour plus d'informations, consultez [Facturation](#billing).

### <a name="running-multiple-containers-on-the-same-host"></a>Exécution de plusieurs conteneurs sur le même hôte

Si vous envisagez d’exécuter plusieurs conteneurs avec les ports exposés, veillez à exécuter chaque conteneur avec un port différent. Par exemple, exécutez le premier conteneur sur le port 5000 et le second conteneur sur le port 5001.

Remplacez `<container-registry>` et `<container-name>` par les valeurs des conteneurs que vous utilisez. Ils n’ont pas besoin d’être le même conteneur. Vous pouvez avoir le conteneur Détecteur d’anomalies et le conteneur LUIS qui s’exécutent ensemble sur l’HÔTE, ou plusieurs conteneurs Détecteur d’anomalies qui s’exécutent.

Exécutez le premier conteneur sur le port 5000.

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
<container-registry>/microsoft/<container-name> \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Exécutez le deuxième conteneur sur le port 5001.


```bash
docker run --rm -it -p 5000:5001 --memory 4g --cpus 1 \
<container-registry>/microsoft/<container-name> \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Tous les conteneurs suivants doivent être sur un port différent.

## <a name="query-the-containers-prediction-endpoint"></a>Interroger le point de terminaison de prédiction du conteneur

Le conteneur fournit des API de point de terminaison de prédiction de requête basées sur REST.

Utilisez l’hôte, http://localhost:5000, pour les API de conteneur.

<!--  ## Validate container is running -->

[!INCLUDE [Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>Arrêter le conteneur

[!INCLUDE [How to stop the container](../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>Dépannage

Si vous exécutez le conteneur avec un [montage](anomaly-detector-container-configuration.md#mount-settings) de sortie et la journalisation activée, il génère des fichiers journaux qui sont utiles pour résoudre les problèmes qui se produisent lors du démarrage ou de l’exécution du conteneur.

[!INCLUDE [Cognitive Services FAQ note](../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Facturation

Les conteneurs du Détecteur d’anomalies envoient les informations de facturation à Azure, en utilisant une ressource _Détecteur d’anomalie_ sur votre compte Azure.

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Pour plus d’informations sur ces options, consultez [Configurer des conteneurs](anomaly-detector-container-configuration.md).

<!--blogs/samples/video coures -->

[!INCLUDE [Discoverability of more container information](../../../includes/cognitive-services-containers-discoverability.md)]

## <a name="summary"></a>Résumé

Dans cet article, vous avez découvert les concepts et le workflow du téléchargement, de l’installation et de l’exécution des conteneurs Détecteur d’anomalies. En résumé :

* Détecteur d’anomalies fournit un conteneur Linux pour Docker, encapsulant la détection d’anomalies par lots ou en continu, l’inférence de plage attendue et le réglage de la sensibilité.
* Les images de conteneurs sont téléchargées à partir d’un registre Azure Container Registry privé pour obtenir des conteneurs.
* Les images conteneurs s’exécutent dans Docker.
* Vous pouvez utiliser l’API REST ou le Kit de développement logiciel (SDK) pour appeler des opérations dans les conteneurs Détecteur d’anomalies en spécifiant l’URI hôte du conteneur.
* Vous devez spécifier les informations de facturation lors de l’instanciation d’un conteneur.

> [!IMPORTANT]
> Les conteneurs Cognitives Services ne sont pas concédés sous licence pour s’exécuter sans être connectés à Azure pour le contrôle. Les clients doivent configurer les conteneurs de manière à ce qu’ils communiquent les informations de facturation au service de contrôle à tout moment. Les conteneurs Cognitive Services n’envoient pas les données des clients (p. ex., les données de série chronologique analysées) à Microsoft.

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir les paramètres de configuration, passez en revue [Configurer des conteneurs](anomaly-detector-container-configuration.md).
* [Déployer un conteneur Détecteur d’anomalies sur Azure Container Instances](how-to/deploy-anomaly-detection-on-container-instances.md)
* [En savoir plus sur le service API Détecteur d’anomalies](https://go.microsoft.com/fwlink/?linkid=2080698&clcid=0x409)