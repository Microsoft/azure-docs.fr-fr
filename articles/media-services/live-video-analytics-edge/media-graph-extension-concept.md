---
title: Présentation de l’extension de graphe multimédia – Azure
description: Live Video Analytics sur IoT Edge vous permet d’étendre les fonctionnalités de traitement de graphe multimédia via un nœud d’extension de graphe.
ms.topic: overview
ms.date: 09/14/2020
ms.openlocfilehash: 6735148bf453cfe0afb58d51451dea65f06705d6
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97401115"
---
# <a name="media-graph-extension"></a>Extension de graphe multimédia

Live Video Analytics sur IoT Edge vous permet d’étendre les fonctionnalités de traitement de graphe multimédia via un nœud d’extension de graphe. Votre plug-in d’extension Analytics peut utiliser des techniques de traitement d’image traditionnelles ou des modèles d’IA de vision par ordinateur. Les extensions de graphe sont activées en incluant un nœud de processeur d’extension dans un graphe multimédia. Le nœud de processeur d’extension relaie les images vidéo vers le point de terminaison configuré et agit en tant qu’interface de votre extension. La connexion peut être établie à un point de terminaison local ou distant. Elle peut être sécurisée par une authentification et un chiffrement TLS, si nécessaire. En outre, le nœud de processeur d’extension de graphe permet la mise à l’échelle et l’encodage facultatifs des images vidéo avant leur envoi à votre extension personnalisée. 

Live Video Analytics prend en charge deux types de processeurs d’extension de graphe multimédia :

* [Processeur d’extension HTTP](media-graph-concept.md#http-extension-processor)
* [Processeur d’extension gRPC](media-graph-concept.md#grpc-extension-processor)

Le nœud d’extension Graph s’attend à ce que le plug-in d’extension Analytics retourne les résultats au format JSON. Dans l’idéal, les résultats doivent suivre le [modèle d’objet du schéma des métadonnées d’inférence](https://review.docs.microsoft.com/en-us/azure/media-services/live-video-analytics-edge/inference-metadata-schema?branch=release-lva-dec-update).

## <a name="http-extension-processor"></a>Processeur d’extension HTTP

Le processeur d’extension HTTP permet des scénarios d’extensibilité à l’aide du [protocole HTTP](https://review.docs.microsoft.com/en-us/azure/media-services/live-video-analytics-edge/http-extension-protocol?branch=release-lva-dec-update), dans lesquels les performances et/ou l’utilisation optimale des ressources ne constituent pas la préoccupation principale. Vous pouvez exposer votre propre IA à un graphe multimédia via un point de terminaison REST HTTP. 

Utilisez le nœud de processeur d’extension HTTP lorsque :

* Vous souhaitez une meilleure interopérabilité avec les systèmes d’inférence HTTP existants.
* Un transfert de données à faibles performances est acceptable.
* Vous voulez utiliser une interface de requête-réponse simple avec Live Video Analytics.

## <a name="grpc-extension-processor"></a>Processeur d’extension gRPC

Le processeur d’extension gRPC permet des scénarios d’extensibilité à l’aide d’un [protocole structuré](https://review.docs.microsoft.com/en-us/azure/media-services/live-video-analytics-edge/grpc-extension-protocol?branch=release-lva-dec-update) hautement performant, s’appuyant sur gRPC. Il est idéal pour les scénarios où les performances et/ou l’utilisation optimale des ressources sont une priorité. Le processeur d’extension gRPC permet de tirer pleinement parti des définitions de données structurées. gRPC offre de hautes performances de transfert de contenu avec :

* la [mémoire partagée intégrée](https://en.wikipedia.org/wiki/Shared_memory) ou 
* l’incorporation directe du contenu dans le corps des messages gRPC. 

Le processeur d’extension gRPC peut être utilisé pour envoyer les propriétés multimédias tout en échangeant les messages d’inférence.
Par conséquent, utilisez un nœud de processeur d’extension gRPC lorsque :

* Vous voulez utiliser un contrat structuré (par exemple, des messages structurés pour les requêtes et les réponses)
* Vous voulez utiliser des mémoires tampons de protocole ([protobuf](https://developers.google.com/protocol-buffers)) comme format d’échange de messages sous-jacent pour la communication.
* Vous voulez communiquer avec un serveur gRPC dans une session de flux unique à la place du modèle de requête-réponse traditionnel nécessitant un gestionnaire de demandes personnalisé pour analyser les requêtes entrantes et appeler les fonctions d’implémentation appropriées. 
* Vous voulez une faible latence et une communication à haut débit entre Live Video Analytics et votre module.

## <a name="use-your-inferencing-model-with-live-video-analytics"></a>Utiliser votre modèle d’inférence avec Live Video Analytics

Les extensions de graphe multimédia vous permettent d’exécuter les modèles d’inférence de votre choix sur un runtime d’inférence disponible quelconque, comme ONNX, TensorFlow, PyTorch ou autres, dans votre propre conteneur Docker. L’extension personnalisée d’inférence doit être déployée en même temps que le module Live Video Analytics de périphérie pour optimiser les performances. Elle est alors appelée via le processeur d’extension HTTP ou le processeur d’extension gRPC inclus dans votre topologie de graphe. De plus, la fréquence des appels dans votre extension personnalisée peut être limitée par l’ajout facultatif d’un [processeur de détection de mouvement](media-graph-concept.md#motion-detection-processor) en amont du processeur d’extension multimédia.

Le diagramme ci-dessous illustre le flux de données général :

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/media-graph-extension/analyze-live-video-with-AI-inference-service.svg" alt-text="Service d’inférence d’IA":::

## <a name="samples"></a>Exemples

Vous pouvez commencer par utiliser l’un de nos guides de démarrage rapide qui illustrent l’analytique vidéo en direct avec un service d’extension prédéfini, à des fréquences d’images faibles en utilisant le [processeur d’extension HTTP](https://review.docs.microsoft.com/en-us/azure/media-services/live-video-analytics-edge/use-your-model-quickstart?branch=release-lva-dec-update&pivots=programming-language-csharp) ou à des fréquences d’images élevées en utilisant le [processeur d’extension gRPC](https://review.docs.microsoft.com/en-us/azure/media-services/live-video-analytics-edge/analyze-live-video-use-your-grpc-model-quickstart?branch=release-lva-dec-update&pivots=programming-language-csharp).

Pour les utilisateurs expérimentés, vous pouvez passer en revue quelques-uns de nos exemples de [notebook Jupyter](https://github.com/Azure/live-video-analytics/blob/master/utilities/video-analysis/notebooks/readme.md) pour Live Video Analytics. Ces notebooks fournissent des instructions pas à pas pour **les extensions de graphe multimédia** sur :

* La création d’une image conteneur Docker d’un service d’extension
* Le déploiement du service d’extension en tant que conteneur avec le conteneur Live Video Analytics
* L’utilisation d’un graphe multimédia Live Video Analytics avec un client d’extension et son pointage vers le point de terminaison de l’extension (HTTP/gRPC)
