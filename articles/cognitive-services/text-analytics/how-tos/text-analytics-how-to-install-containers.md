---
title: Installer et exécuter des conteneurs Docker pour l’API Analyse de texte
titleSuffix: Azure Cognitive Services
description: Utilisez les conteneurs Docker pour l’API Analyse de texte pour effectuer un traitement en langage naturel, tel que l’analyse des sentiments, en local.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18, cog-serv-seo-aug-2020
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 09/28/2020
ms.author: aahi
keywords: local, Docker, conteneur, analyse des sentiments, traitement en langage naturel
ms.openlocfilehash: 088cf407320dcf8ef6705c57d86836531d9bdfe6
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965164"
---
# <a name="install-and-run-text-analytics-containers"></a>Installer et exécuter les conteneurs Analyse de texte

> [!NOTE]
> * Le conteneur Analyse des sentiments v3 est désormais en disposition générale. Les conteneurs Extraction de phrases clés et Détection de langue sont disponibles sous la forme d’une préversion publique non contrôlée.
> * Les fonctionnalités Liaison d’entités et Reconnaissance d'entité nommée ne sont actuellement pas disponibles en tant que conteneurs.
> * L’accès à l’analyse de texte pour le conteneur d’intégrité requiert un [formulaire de demande](https://aka.ms/csgate). Actuellement, vous ne serez pas facturé pour son utilisation.
> * Les emplacements de l’image du conteneur peuvent avoir été modifiés récemment. Lisez cet article pour voir l’emplacement mis à jour de ce conteneur.

Les conteneurs vous permettent d’exécuter les API d’analyse de texte dans votre propre environnement et sont très utiles pour répondre à vos besoins spécifiques en matière de sécurité et de gouvernance des données. Les conteneurs Analyse de texte fournissent un traitement en langage naturel avancé sur le texte brut et incluent trois fonctions principales : analyse des sentiments, extraction d’expressions clés et détection de la langue. 

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/) avant de commencer.

> [!IMPORTANT]
> Le compte gratuit est limité à 5 000 transactions par mois et seuls les niveaux tarifaires **Gratuit** et **Standard** <a href="https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics" target="_blank">sont <span class="docon docon-navigate-external x-hidden-focus"></span></a> valides pour les conteneurs. Pour plus d’informations sur les taux de requête de transaction, consultez [Tarification - Transfert de données](../overview.md#data-limits).

## <a name="prerequisites"></a>Prérequis

Pour exécuter l’un des conteneurs Analyse de texte, vous devez disposer de l’ordinateur hôte et des environnements de conteneur.

## <a name="preparation"></a>Préparation

Vous devez respecter les prérequis suivants avant d’utiliser les conteneurs Analyse de texte :

|Obligatoire|Objectif|
|--|--|
|Moteur Docker| Vous avez besoin d’un moteur Docker installé sur un [ordinateur hôte](#the-host-computer). Docker fournit des packages qui configurent l’environnement Docker sur [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) et [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Pour apprendre les principes de base de Docker et des conteneurs, consultez la [vue d’ensemble de Docker](https://docs.docker.com/engine/docker-overview/).<br><br> Vous devez configurer Docker pour permettre aux conteneurs de se connecter à Azure et de lui envoyer des données de facturation. <br><br> **Sur Windows**, vous devez également configurer Docker pour prendre en charge les conteneurs Linux.<br><br>|
|Bonne connaissance de Docker | Vous devez avoir une compréhension élémentaire des concepts Docker, notamment les registres, référentiels, conteneurs et images conteneurs, ainsi qu’une maîtrise des commandes `docker` de base.| 
|Ressource Analyse de texte |Pour pouvoir utiliser le conteneur, vous devez disposer des éléments suivants :<br><br>Ressource [Analyse de texte](../../cognitive-services-apis-create-account.md) d’Azure afin d’obtenir la clé API et l’URI du point de terminaison associés. Les deux valeurs, disponibles dans les pages Vue d’ensemble d’Analyse de texte et Clés du portail Azure, sont nécessaires au démarrage du conteneur.<br><br>**{API_KEY}**  : L’une des deux clés de ressource disponibles à la page **Clés**<br><br>**{ENDPOINT_URI}**  : Le point de terminaison tel qu'il est fourni à la page **Vue d’ensemble**|

[!INCLUDE [Gathering required parameters](../../containers/includes/container-gathering-required-parameters.md)]

## <a name="the-host-computer"></a>L’ordinateur hôte

[!INCLUDE [Host Computer requirements](../../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="container-requirements-and-recommendations"></a>Exigences et suggestions relatives au conteneur

Le tableau suivant décrit les spécifications minimales et recommandées pour les conteneurs Analyse de texte. Au moins 2 gigaoctets (Go) de mémoire sont requis, et chaque UC doit être cadencé à au moins 2,6 gigahertz (GHz). Les transactions par section (TPS) autorisées sont également indiquées.

|  | Configuration minimale pour l’hôte | Configuration recommandée pour l’hôte | TPS minimales | TPS maximales|
|---|---------|-------------|--|--|
| **Détection de la langue, Extraction de phrases clés**   | 1 cœur, 2 Go de mémoire | 1 cœur, 4 Go de mémoire |15 | 30|
| **Analyse des sentiments v3**   | 1 cœur, 2 Go de mémoire | 4 cœurs, 8 Go de mémoire |15 | 30|
| **Analyse de texte pour l’intégrité – 1 document/demande**   |  4 cœurs, 10 Go de mémoire | 6 cœurs, 12 Go de mémoire |15 | 30|
| **Analyse de texte pour l’intégrité – 10 documents/demande**   |  6 cœurs, 16 Go de mémoire | 8 cœurs, 20 Go de mémoire |15 | 30|

Le cœur d’UC et la quantité de mémoire correspondent aux paramètres `--cpus` et `--memory` qui sont utilisés dans le cadre de la commande `docker run`.

## <a name="get-the-container-image-with-docker-pull"></a>Obtenir l’image conteneur avec `docker pull`

[!INCLUDE [Tip for using docker list](../../../../includes/cognitive-services-containers-docker-list-tip.md)]

Images conteneur pour Analyse de texte disponibles dans Microsoft Container Registry.

# <a name="sentiment-analysis-v3"></a>[Analyse des sentiments v3](#tab/sentiment)

[!INCLUDE [docker-pull-sentiment-analysis-container](../includes/docker-pull-sentiment-analysis-container.md)]

# <a name="key-phrase-extraction-preview"></a>[Extraction de phrases clés (préversion)](#tab/keyphrase)

[!INCLUDE [docker-pull-key-phrase-extraction-container](../includes/docker-pull-key-phrase-extraction-container.md)]

# <a name="language-detection-preview"></a>[Détection de langue (préversion)](#tab/language)

[!INCLUDE [docker-pull-language-detection-container](../includes/docker-pull-language-detection-container.md)]

# <a name="text-analytics-for-health-preview"></a>[Analyse de texte pour l’intégrité (préversion)](#tab/healthcare)

[!INCLUDE [docker-pull-health-container](../includes/docker-pull-health-container.md)]

**_

## <a name="how-to-use-the-container"></a>Comment utiliser le conteneur

Une fois que le conteneur est sur l’[ordinateur hôte](#the-host-computer), appliquez la procédure suivante pour travailler avec le conteneur.

1. [Exécutez le conteneur](#run-the-container-with-docker-run) avec les paramètres de facturation requis.
1. [Interrogez le point de terminaison de prédiction du conteneur](#query-the-containers-prediction-endpoint).

## <a name="run-the-container-with-docker-run"></a>Exécuter le conteneur avec `docker run`

Utilisez la commande [docker run](https://docs.docker.com/engine/reference/commandline/run/) (Exécution Docker) pour exécuter les conteneurs. Chaque conteneur continuera à s’exécuter jusqu’à ce que vous l’arrêtiez.

> [!IMPORTANT]
> _ Les commandes Docker dans les sections suivantes utilisent la barre oblique inverse, `\`, comme caractère de continuation de ligne. Remplacez-la ou supprimez-la en fonction des exigences de votre système d’exploitation hôte. 
> * Vous devez spécifier les options `Eula`, `Billing` et `ApiKey` pour exécuter le conteneur, sinon il ne démarrera pas.  Pour plus d'informations, consultez [Facturation](#billing).
> * Le conteneur Analyse des sentiments v3 est désormais en disposition générale. Il renvoie des [étiquettes de sentiment](../how-tos/text-analytics-how-to-sentiment-analysis.md#sentiment-analysis-versions-and-features) dans la réponse. Les conteneurs Extraction de phrases clés et Détection de langue utilisent la v2 de l’API. Ils sont en préversion.

# <a name="sentiment-analysis-v3"></a>[Analyse des sentiments v3](#tab/sentiment)

[!INCLUDE [docker-run-sentiment-analysis-container](../includes/docker-run-sentiment-analysis-container.md)]

# <a name="key-phrase-extraction-preview"></a>[Extraction de phrases clés (préversion)](#tab/keyphrase)

[!INCLUDE [docker-run-key-phrase-extraction-container](../includes/docker-run-key-phrase-extraction-container.md)]

# <a name="language-detection-preview"></a>[Détection de langue (préversion)](#tab/language)

[!INCLUDE [docker-run-language-detection-container](../includes/docker-run-language-detection-container.md)]

# <a name="text-analytics-for-health-preview"></a>[Analyse de texte pour l’intégrité (préversion)](#tab/healthcare)

[!INCLUDE [docker-run-health-container](../includes/docker-run-health-container.md)]

***

[!INCLUDE [Running multiple containers on the same host](../../../../includes/cognitive-services-containers-run-multiple-same-host.md)]

## <a name="query-the-containers-prediction-endpoint"></a>Interroger le point de terminaison de prédiction du conteneur

Le conteneur fournit des API de point de terminaison de prédiction de requête basées sur REST.

Utilisez l’hôte, `http://localhost:5000`, pour les API de conteneur.

<!--  ## Validate container is running -->

[!INCLUDE [Container's API documentation](../../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>Arrêter le conteneur

[!INCLUDE [How to stop the container](../../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>Dépannage

Si vous exécutez le conteneur avec un [montage](../text-analytics-resource-container-config.md#mount-settings) de sortie et la journalisation activée, il génère des fichiers journaux qui sont utiles pour résoudre les problèmes qui se produisent lors du démarrage ou de l’exécution du conteneur.

[!INCLUDE [Cognitive Services FAQ note](../../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Facturation

Les conteneurs Analyse de texte envoient des informations de facturation à Azure à l’aide d’une ressource _Analyse de texte_ sur votre compte Azure. 

[!INCLUDE [Container's Billing Settings](../../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Pour plus d’informations sur ces options, consultez [Configurer des conteneurs](../text-analytics-resource-container-config.md).

<!--blogs/samples/video course -->

[!INCLUDE [Discoverability of more container information](../../../../includes/cognitive-services-containers-discoverability.md)]

## <a name="summary"></a>Résumé

Dans cet article, vous avez découvert des concepts et le flux de travail pour le téléchargement, l’installation et l’exécution des conteneurs Analyse de texte. En résumé :

* Analyse de texte fournit trois conteneurs Linux pour Docker, encapsulant diverses fonctionnalités :
   * *Analyse des sentiments*
   * *Extraction de phrases clés (préversion)* 
   * *Détection de langue (préversion)*
   * *Analyse de texte pour l’intégrité (préversion)*
* Les images conteneur sont téléchargées à partir de Microsoft Container Registry (MCR) ou du référentiel de conteneur de préversion.
* Les images conteneurs s’exécutent dans Docker.
* Vous pouvez utiliser l’API REST ou le kit SDK pour appeler des opérations dans des conteneurs Analyse de texte en spécifiant l’URI hôte du conteneur.
* Vous devez spécifier les informations de facturation lors de l’instanciation d’un conteneur.

> [!IMPORTANT]
> Les conteneurs Cognitives Services ne sont pas concédés sous licence pour s’exécuter sans être connectés à Azure pour le contrôle. Les clients doivent configurer les conteneurs de manière à ce qu’ils communiquent les informations de facturation au service de contrôle à tout moment. Les conteneurs Cognitive Services ne transmettent aucune donnée client (p. ex., le texte analysé) à Microsoft.

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir les paramètres de configuration, passez en revue [Configurer des conteneurs](../text-analytics-resource-container-config.md).
* Reportez-vous à [Forum aux questions (FAQ)](../text-analytics-resource-faq.md) pour résoudre les problèmes liés à la fonctionnalité.
