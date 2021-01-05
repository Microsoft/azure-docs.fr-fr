---
title: Notes de publication pour Live Video Analytics sur IoT Edge – Azure
description: Cette rubrique fournit des notes de publication sur les versions, les améliorations, les correctifs de bogues et les problèmes connus de Live Video Analytics sur IoT Edge.
ms.topic: conceptual
ms.date: 08/19/2020
ms.openlocfilehash: 18da50fe763ef7c5fc3c06ac00052679c488cae7
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92015632"
---
# <a name="live-video-analytics-on-iot-edge-release-notes"></a>Notes de publication pour Live Video Analytics sur IoT Edge

>Soyez notifié de la disponibilité des mises à jour sur cette page en faisant un copier-coller de cette URL : `https://docs.microsoft.com/api/search/rss?search=%22Live+Video+Analytics+on+IoT+Edge+release+notes%22&locale=en-us` dans votre lecteur de flux RSS.

Cet article vous fournit des informations sur :

* Versions les plus récentes
* Problèmes connus
* Résolution des bogues
* Fonctionnalités dépréciées

<hr width=100%>

## <a name="september-22-2020"></a>22 septembre 2020

Cette balise de version pour l’actualisation du module en septembre 2020 est la suivante :

```
mcr.microsoft.com/media/live-video-analytics:1.0.4
```

> [!NOTE]
> Dans les démarrages rapides et les didacticiels, les manifestes de déploiement utilisent une balise 1 (live-video-analytics:1). Par conséquent, le simple redéploiement de tels manifestes doit mettre à jour le module sur votre application Edge > Appareils.

### <a name="module-updates"></a>Mises à jour de module

* Un nouveau nœud d’extension de graphique, [MediaGraphCognitiveServicesVisionExtension](spatial-analysis-tutorial.md), est disponible pour une intégration au module [Analyse spatiale](/legal/cognitive-services/computer-vision/intro-to-spatial-analysis-public-preview) (préversion) à partir de Cognitive Services.
* Ajout de la prise en charge des appareils Linux ARM64 : utilisez les [étapes manuelles](deploy-iot-edge-device.md) pour déployer cette fonctionnalité sur ces appareils.

### <a name="documentation-updates"></a>Mises à jour de la documentation

* [Des instructions](deploy-azure-stack-edge-how-to.md) relatives à l’utilisation de Live Video Analytics sur IoT Edge sur des appareils Azure Stack Edge sont disponibles.
* Nouveau tutoriel sur le développement de modèles de vision par ordinateur spécifiques à l’aide du [service Custom Vision](https://azure.microsoft.com/services/cognitive-services/custom-vision-service/) et l’utilisation de ce dernier pour [analyser des vidéos en direct](custom-vision-tutorial.md) en temps réel.

<hr width=100%>

## <a name="august-19-2020"></a>19 août 2020

Cette balise de version pour l’actualisation du module en août 2020 est la suivante :

```
mcr.microsoft.com/media/live-video-analytics:1.0.3
```

> [!NOTE]
> Dans les démarrages rapides et les didacticiels, les manifestes de déploiement utilisent une balise 1 (live-video-analytics:1). Par conséquent, le simple redéploiement de tels manifestes doit mettre à jour le module sur votre application Edge > Appareils.

### <a name="module-updates"></a>Mises à jour de module

* Vous pouvez désormais bénéficier de performances de transfert de contenu de données élevées entre Live Video Analytics sur IoT Edge et votre extension personnalisée à l’aide de l’infrastructure gRPC. Consultez [ce](analyze-live-video-use-your-grpc-model-quickstart.md) pour vous lancer.
* Déploiement régional plus étendu de Live Video Analytics et seul le service cloud a été mis à jour.  
* Live Video Analytics est désormais disponible dans 25 régions supplémentaires dans le monde. Voici la [liste](https://azure.microsoft.com/global-infrastructure/services/?products=media-services) de toutes les régions disponibles.  
* La [configuration](https://aka.ms/lva-edge/setup-resources-for-samples) pour les démarrages rapides a également été mise à jour avec la prise en charge de nouvelles régions.
    * Aucun appel à l’action n’est effectué pour quiconque a déjà configuré des ressources

### <a name="bug-fixes"></a>Résolution des bogues 

* Supprimer l’utilisation d’une extension Azure déconseillée dans le script de configuration

<hr width=100%>

## <a name="july-13-2020"></a>13 juillet 2020

Cette balise de version pour l’actualisation du module en juillet 2020 est la suivante :

```
mcr.microsoft.com/media/live-video-analytics:1.0.2
```

> [!NOTE]
> Dans les démarrages rapides et les didacticiels, les manifestes de déploiement utilisent une balise 1 (live-video-analytics:1). Par conséquent, le simple redéploiement de tels manifestes doit mettre à jour le module sur votre application Edge > Appareils.

### <a name="module-updates"></a>Mises à jour de module

* Vous pouvez désormais créer des topologies de graphiques qui ont un nœud récepteur de ressources, ainsi qu’un nœud récepteur de fichiers en aval d’un nœud processeur de porte de signal. Pour obtenir un exemple, consultez [cette page](https://github.com/Azure/live-video-analytics/tree/master/MediaGraph/topologies/evr-motion-assets-files).

### <a name="bug-fixes"></a>Résolution des bogues

* Améliorations de la validation des propriétés souhaitées

<hr width=100%>

## <a name="june-1-2020"></a>1er juin 2020

Cette version est la première préversion publique de Live Video Analytics sur IoT Edge. La balise de version est

```
     mcr.microsoft.com/media/live-video-analytics:1.0.0
```

### <a name="supported-functionalities"></a>Fonctionnalités prises en charge

* Analyser les flux vidéo en direct à l’aide des modules d’intelligence artificielle de votre choix et enregistrer éventuellement une vidéo sur le périphérique ou dans le cloud
* Utiliser sur les systèmes d’exploitation Linux AMD64 [pris en charge](../../iot-edge/support.md) par IoT Edge
* Déployer et configurer le module via le hub IoT à l’aide de Portail Azure ou Visual Studio Code
* Gérer [les topologies et instances de graphe](media-graph-concept.md#media-graph-topologies-and-instances) à distance ou localement par le biais des appels de méthode directe suivants

    *   GraphTopologyGet
    *   GraphTopologySet
    *   GraphTopologyDelete
    *   GraphTopologyList
    *   GraphInstanceGet
    *   GraphInstanceSet
    *   GraphInstanceDelete
    *   GraphInstanceList

## <a name="next-steps"></a>Étapes suivantes

[Vue d'ensemble](overview.md)