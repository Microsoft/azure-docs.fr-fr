---
title: Mise à niveau de Live Video Analytics sur IoT Edge de 1.0 à 2.0
description: Cet article aborde les différences et les différents aspects à prendre en compte lors de la mise à niveau du module Live Video Analytics (LVA) sur Azure IoT Edge.
author: naiteeks
ms.topic: how-to
ms.author: naiteeks
ms.date: 12/14/2020
ms.openlocfilehash: 7904269a8ca87d3d801c7ff78def1b60f242b9ab
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97417426"
---
# <a name="upgrading-live-video-analytics-on-iot-edge-from-10-to-20"></a>Mise à niveau de Live Video Analytics sur IoT Edge de 1.0 à 2.0

Cet article aborde les différences et les différents aspects à prendre en compte lors de la mise à niveau du module Live Video Analytics (LVA) sur Azure IoT Edge.

## <a name="change-list"></a>Liste des modifications

> [!div class="mx-tdCol4BreakAll"]
> |Titre|Live Video Analytics 1.0|Live Video Analytics 2.0|Description|
> |-------------|----------|---------|---------|
> |Image conteneur|mcr.microsoft.com/media/live-video-analytics:1.0.0|mcr.microsoft.com/media/live-video-analytics:2.0.0|Images Docker publiées par Microsoft pour le service Live Video Analytics sur Azure IoT Edge|
> |**Nœuds MediaGraph** |    |   |   |
> |Sources|:::image type="icon" source="./././media/upgrading-lva/check.png"::: Source RTSP </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Source de messages IoT Hub |:::image type="icon" source="./././media/upgrading-lva/check.png"::: Source RTSP </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Source de messages IoT Hub | Nœuds MediaGraph qui font office de sources pour l’ingestion de média et les messages.|
> |Processeurs|:::image type="icon" source="./././media/upgrading-lva/check.png"::: Processeur de détection de mouvement </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Processeur de filtre de fréquence d’images </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Processeur d’extension HTTP </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Processeur d’extension gRPC </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Processeur de porte de signal |:::image type="icon" source="./././media/upgrading-lva/check.png"::: Processeur de détection de mouvement </br>:::image type="icon" source="./././media/upgrading-lva/remove.png"::: **Processeur de filtre de fréquence d’images**</br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Processeur d’extension HTTP </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Processeur d’extension gRPC </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Processeur de porte de signal | Nœuds MediaGraph qui vous permettent de formater le média avant son envoi aux serveurs d’inférence d’intelligence artificielle.|
> |Récepteurs|:::image type="icon" source="./././media/upgrading-lva/check.png"::: Récepteur de ressources </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Récepteur de fichiers </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Récepteur de messages IoT Hub|:::image type="icon" source="./././media/upgrading-lva/check.png"::: Récepteur de ressources </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Récepteur de fichiers </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Récepteur de messages IoT Hub| Nœuds MediaGraph qui vous permettent de stocker le média traité.|
> |**MediaGraphs pris en charge** |    |   |   |
> |Topologies|:::image type="icon" source="./././media/upgrading-lva/check.png"::: Enregistrement vidéo en continu </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Analyse de mouvement et enregistrement vidéo continu </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Analyse externe et enregistrement vidéo continu </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Enregistrement basé sur les événements sur mouvement </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Enregistrement basé sur les événements sur intelligence artificielle</br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Enregistrement basé sur les événements sur événement externe </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Analyse de mouvement </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Analyse de mouvement suivie d’une inférence d’intelligence artificielle |:::image type="icon" source="./././media/upgrading-lva/check.png"::: Enregistrement vidéo en continu </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Analyse de mouvement et enregistrement vidéo continu </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Analyse externe et enregistrement vidéo continu </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Enregistrement basé sur les événements sur mouvement </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Enregistrement basé sur les événements sur intelligence artificielle</br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Enregistrement basé sur les événements sur événement externe </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Analyse de mouvement </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Analyse de mouvement suivie d’une inférence d’intelligence artificielle </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: **Composition d’intelligence artificielle** </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: **Enregistrement audio et vidéo** </br>  | Topologies MediaGraph vous permettant de définir le blueprint d’un graphe, avec des paramètres tels que des espaces réservés pour valeurs.|

## <a name="upgrading-the-live-video-analytics-on-iot-edge-module-from-10-to-20"></a>Mise à niveau du module Live Video Analytics sur IoT Edge de 1.0 à 2.0
Lors de la mise à niveau du module Live Video Analytics sur IoT Edge, veillez à mettre à jour les informations suivantes.
### <a name="container-image"></a>Image conteneur
Dans votre modèle de déploiement, recherchez votre module Live Video Analytics sur IoT Edge sous le nœud `modules`, et mettez à jour le champ `image` comme suit :
```
"image": "mcr.microsoft.com/media/live-video-analytics:2"
```
> [!TIP]
Si vous n’avez pas modifié le nom du module Live Video Analytics sur IoT Edge, recherchez `lvaEdge` sous le nœud du module.

### <a name="topology-file-changes"></a>Modifications des fichiers de topologie
Dans vos fichiers de topologie, assurez-vous que **`apiVersion`** est défini sur 2.0.

### <a name="mediagraph-node-changes"></a>Modifications du nœud MediaGraph


* L’audio de votre source de caméra peut désormais être passé en aval avec la vidéo. Vous pouvez passer l’**audio uniquement**, la **vidéo uniquement** ou **l’audio et la vidéo** à l’aide de **`outputSelectors`** à n’importe quel nœud de graphique. Par exemple, si vous souhaitez sélectionner la vidéo uniquement de la source RTSP et la passer en aval, ajoutez ce qui suit au nœud source RTSP :
    ```
    "outputSelectors": [
      {
        "property": "mediaType",
        "operator": "is",
        "value": "video"
      }
    ```
>[!NOTE]
>**`outputSelectors`** est une propriété facultative. Si elle n’est pas utilisée, le graphe de multimédia transmet l’audio (s’il est activé) et la vidéo de la caméra RTSP en aval. 

* Dans les processeurs `MediaGraphHttpExtension` et `MediaGraphGrpcExtension`, notez les modifications suivantes :  
    * **Propriétés de l’image**
        * `MediaGraphImageFormatEncoded` n’est plus pris en charge. 
        * Utilisez plutôt **`MediaGraphImageFormatBmp`** , **`MediaGraphImageFormatJpeg`** ou **`MediaGraphImageFormatPng`** . Par exemple,
        ```
        "image": {
                "scale": 
                {
                    "mode": "preserveAspectRatio",
                    "width": "416",
                    "height": "416"
                },
                "format": 
                {
                    "@type": "#Microsoft.Media.MediaGraphImageFormatJpeg"
                }
            }
        ```
        * Si vous souhaitez utiliser des images RAW, utilisez **`MediaGraphImageFormatRaw`** . Pour ce faire, mettez à jour le nœud d’image comme suit :
        ```
        "image": {
                "scale": 
                {
                    "mode": "preserveAspectRatio",
                    "width": "416",
                    "height": "416"
                },
                "format": 
                {
                    "@type": "#Microsoft.Media.MediaGraphImageFormatRaw",
                    "pixelFormat": "rgba"
                }
            }
        ```
        >[!NOTE]
        > Les valeurs possibles de pixelFormat sont les suivantes : `yuv420p`,`rgb565be`, `rgb565le`, `rgb555be`, `rgb555le`, `rgb24`, `bgr24`, `argb`, `rgba`, `abgr`, `bgra`  

    * **extensionConfiguration pour le processeur d’extension gRPC**  
        * Dans le processeur `MediaGraphGrpcExtension`, une nouvelle propriété nommée **`extensionConfiguration`** est disponible, qui est une chaîne facultative utilisable dans le contrat gRPC. Ce champ peut être utilisé pour passer des données au serveur d’inférence, et vous pouvez définir la façon dont celui-ci utilise ces données.  
        Un cas d’utilisation de cette propriété est lorsque vous avez plusieurs modèles d’intelligence artificielle empaquetés dans un seul serveur d’inférence. Avec cette propriété, vous n’avez pas besoin d’exposer un nœud pour chaque modèle d’intelligence artificielle. Au lieu de cela, pour une instance de graphe, en tant que fournisseur d’extensions, vous pouvez définir la manière de sélectionner les différents modèles d’intelligence artificielle à l’aide de la propriété **`extensionConfiguration`** , et pendant l’exécution, le service Live Video Analytics transmet cette chaîne au serveur d’inférence qui peut l’utiliser pour appeler le modèle d’intelligence artificielle souhaité.  

    * **Composition d’intelligence artificielle**
        * Le service Live Video Analytics 2.0 prend désormais en charge l’utilisation de plusieurs processeurs d’extension de graphe multimédia au sein d’une topologie. Vous pouvez transmettre les image de média de la caméra RTSP à différents modèles d’intelligence artificielle de manière séquentielle, en parallèle ou dans une combinaison des deux. Consultez un exemple de topologie présentant deux modèles d’intelligence artificielle utilisés de manière séquentielle.


* Dans votre nœud **Récepteur de fichiers**, vous pouvez maintenant spécifier la quantité d’espace disque que le module Live Video Analytics sur IoT Edge peut utiliser pour stocker les images traitées. Pour ce faire, ajoutez le champ **`maximumSizeMiB`** au nœud Récepteur de fichiers. Voici un exemple de nœud Récepteur de fichiers :
    ```
    "sinks": [
      {
        "@type": "#Microsoft.Media.MediaGraphFileSink",
        "name": "fileSink",
        "inputs": [
          {
            "nodeName": "signalGateProcessor",
            "outputSelectors": [
              {
                "property": "mediaType",
                "operator": "is",
                "value": "video"
              }
            ]
          }
        ],
        "fileNamePattern": "sampleFiles-${System.DateTime}",
        "maximumSizeMiB":"512",
        "baseDirectoryPath":"/var/media"
      }
    ]
    ```
* Dans votre nœud **Récepteur de ressources**, vous pouvez spécifier la quantité d’espace disque que le module Live Video Analytics sur IoT Edge peut utiliser pour stocker les images traitées. Pour ce faire, ajoutez le champ **`localMediaCacheMaximumSizeMiB`** au nœud Récepteur de ressources. Voici un exemple de nœud Récepteur de ressources :
    ```
    "sinks": [
      {
        "@type": "#Microsoft.Media.MediaGraphAssetSink",
        "name": "AssetSink",
        "inputs": [
          {
            "nodeName": "signalGateProcessor",
            "outputSelectors": [
              {
                "property": "mediaType",
                "operator": "is",
                "value": "video"
              }
            ]
          }
        ],
        "assetNamePattern": "sampleAsset-${System.GraphInstanceName}",
        "segmentLength": "PT30S",
        "localMediaCacheMaximumSizeMiB":"200",
        "localMediaCachePath":"/var/lib/azuremediaservices/tmp/"
      }
    ]
    ```
    >[!NOTE]
    >  Le chemin d’accès du **Récepteur de fichiers** est fractionné en chemin d’accès du répertoire de base et en modèle de nom de fichier, tandis que le chemin d’accès du **Récepteur de ressources** contient le chemin du répertoire de base.  

* **`MediaGraphFrameRateFilterProcessor`** est déconseillé dans le module **Live Video Analytics sur IoT Edge 2.0**.
    * Pour échantillonner la vidéo entrante à des fins de traitement, ajoutez la propriété **`samplingOptions`** aux processeurs d’extension MediaGraph (`MediaGraphHttpExtension` ou `MediaGraphGrpcExtension`).  
     ```
          "samplingOptions": 
          {
            "skipSamplesWithoutAnnotation": "false",  // If true, limits the samples submitted to the extension to only samples which have associated inference(s) 
            "maximumSamplesPerSecond": "20"   // Maximum rate of samples submitted to the extension
          },
    ```
### <a name="module-metrics-in-the-prometheus-format-using-telegraf"></a>Métriques de module au format Prometheus utilisant Telegraf
Avec cette mise en production, Telegraf peut être utilisé pour envoyer des métriques à Azure Monitor. À partir de là, les métriques peuvent être dirigées vers Log Analytics ou un Event Hub.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/telemetry-schema/telegraf.png" alt-text="Taxonomie des événements":::

Vous pouvez produire une image Telegraf avec une configuration personnalisée facilement à l’aide de Docker. Pour en savoir plus à ce sujet, consultez la page [Surveillance et journalisation](monitoring-logging.md#azure-monitor-collection-using-telegraf).

## <a name="next-steps"></a>Étapes suivantes

[Bien démarrer avec Live Video Analytics sur IoT Edge](get-started-detect-motion-emit-events-quickstart.md)