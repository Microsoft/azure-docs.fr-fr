---
title: Tutoriel – Surveiller la vidéo à l’aide du modèle d’application d’analytique vidéo pour la détection d’objet et de mouvement Azure IoT Central
description: Ce tutoriel montre comment utiliser les tableaux de bord dans le modèle d’application d’analytique vidéo pour la détection d’objet et de mouvement afin de gérer vos caméras et de surveiller la vidéo.
services: iot-central
ms.service: iot-central
ms.subservice: iot-central-retail
ms.topic: tutorial
ms.author: nandab
author: KishorIoT
ms.date: 07/31/2020
ms.openlocfilehash: 1c1ddeb8cafd8aa7584da48a715139c5d12eeb10
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91874796"
---
# <a name="tutorial-monitor-and-manage-a-video-analytics---object-and-motion-detection-application"></a>Tutoriel : Surveiller et gérer une application d’analytique vidéo pour la détection d’objet et de mouvement

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * ajouter des caméras de détection d’objet et de mouvement à votre application IoT Central ;
> * gérez vos flux vidéo et les lire quand des événements intéressants sont détectés.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vous devez avoir lu :

* L’un des tutoriels précédents [Créer une application d’analytique vidéo en direct dans Azure IoT Central](./tutorial-video-analytics-create-app-yolo-v3.md) ou [Créer une analytique vidéo dans Azure IoT Central (OpenVINO)&trade;](tutorial-video-analytics-create-app-openvino.md).
* l’un des tutoriels précédents [Créer une instance IoT Edge d’analytique vidéo en direct (machine virtuelle Linux)](tutorial-video-analytics-iot-edge-vm.md) ou [Créer une instance IoT Edge d’analytique vidéo en direct (Intel NUC)](tutorial-video-analytics-iot-edge-nuc.md).

L’outil [Docker](https://www.docker.com/products/docker-desktop) doit être installé sur votre machine locale pour exécuter l’application de visionneuse vidéo.

## <a name="add-an-object-detection-camera"></a>Ajouter une caméra de détection d’objet

Dans votre application IoT Central, accédez à l’appareil **Passerelle d’analytique vidéo en direct 001** que vous avez créé précédemment. Sélectionnez l’onglet **Commandes**.

Utilisez les valeurs du tableau suivant pour paramétrer la commande de **demande d’ajout de caméra**. Les valeurs indiquées dans le tableau supposent que vous utilisez la caméra simulée de la machine virtuelle Azure. Si vous utilisez une caméra réelle, modifiez les valeurs de manière appropriée :

| Champ| Description| Exemple de valeur|
|---------|---------|---------|
| ID de la caméra      | ID d’appareil pour l’approvisionnement | caméra-003 |
| Nom de la caméra    | Nom convivial           | Caméra de détection d’objet |
| URL RTSP       | Adresse du flux.   | RTSP://10.0.0.4:554/media/camera-300s.mkv|
| Nom d’utilisateur RTSP  |                         | utilisateur    |
| Mot de passe RTSP  |                         | mot de passe    |
| Type de détection | Liste déroulante                | Détection d’objets       |

Sélectionnez **Exécuter** pour ajouter la caméra :

:::image type="content" source="media/tutorial-video-analytics-manage/add-camera.png" alt-text="Ajouter une caméra":::

> [!NOTE]
> Le modèle d’appareil **LVA Edge Object Detector** (détecteur d’objet périphérique d’analytique vidéo en direct) existe déjà dans l’application.

## <a name="add-a-motion-detection-camera-optional"></a>Ajouter une caméra de détection de mouvement (facultatif)

Si vous avez deux caméras connectées à votre appareil de passerelle IoT Edge, répétez les étapes précédentes pour ajouter une caméra de détection de mouvement à l’application. Utilisez des valeurs différentes pour les paramètres **ID de la caméra**, **Nom de la caméra** et **URL RTSP**.

## <a name="view-the-downstream-devices"></a>Afficher les appareils en aval

Sélectionnez l’onglet **Appareils en aval** pour l’appareil **Passerelle d’analytique vidéo en direct 001** afin de voir les caméras que vous venez d’ajouter :

:::image type="content" source="media/tutorial-video-analytics-manage/inspect-downstream.png" alt-text="Ajouter une caméra":::

Les caméras figurent également dans la liste de la page **Appareils** dans l’application.

## <a name="configure-and-manage-the-camera"></a>Configurer et gérer la caméra

Accédez à l’appareil **camera-003** et sélectionnez l’onglet **Gérer**.

Utilisez les valeurs par défaut ou modifiez-les si vous devez personnaliser les propriétés de l’appareil :

**Paramètres de la caméra**

| Propriété | Description | Valeur suggérée |
|-|-|-|
| Hôte de lecture vidéo | Hôte pour la visionneuse du Lecteur multimédia Azure | http://localhost:8094 |

**Opérations et diagnostics d’analytique vidéo en direct**

| Propriété | Description | Valeur suggérée |
|-|-|-|
| Démarrage automatique | Démarrer la détection d’objet au redémarrage de la passerelle d’analytique vidéo en direct | Activée |
| Télémétrie de débogage | Suivis d’événements | Facultatif |
|Délai d’expiration d’inférence (s)| Temps nécessaire pour déterminer si les inférences ont été arrêtées | 20 |

**Détection d’objet d’IA**

| Propriété | Description | Valeur suggérée |
|-|-|-|
| Temps maximal de capture d’inférence (s) | Temps maximal de capture | 15 |
| Classes de détection | Chaînes délimitées par des virgules, avec les balises de détection. Pour plus d’informations, consultez la [liste des balises prises en charge](https://github.com/Azure/live-video-analytics/blob/master/utilities/video-analysis/yolov3-onnx/tags.txt). | camion, bus |
| Seuil de confiance | Pourcentage de qualification pour déterminer si la détection d’objet est valide | 70 |
| Taux d’échantillonnage d’image d’inférence (i/s) | [Description ici] | 2 |

Sélectionnez **Enregistrer**.

Au bout de quelques secondes, vous voyez s’afficher le message de confirmation **Accepté** pour chaque paramètre :

:::image type="content" source="media/tutorial-video-analytics-manage/object-detection.png" alt-text="Ajouter une caméra":::

## <a name="start-lva-processing"></a>Démarrer le traitement d’analytique vidéo en direct

Accédez à l’appareil **caméra-003** et sélectionnez l’onglet **Commandes**.

Exécutez la commande **Démarrer le traitement d’analytique vidéo en direct**.

Une fois l’exécution de la commande terminée, affichez l’historique des commandes pour vous assurer qu’il n’y a pas d’erreurs :

:::image type="content" source="media/tutorial-video-analytics-manage/start-processing.png" alt-text="Ajouter une caméra":::

## <a name="monitor-the-cameras"></a>Surveiller les caméras

Accédez à l’appareil **caméra-003** et sélectionnez l’onglet **Tableau de bord** :

:::image type="content" source="media/tutorial-video-analytics-manage/camera-dashboard.png" alt-text="Ajouter une caméra":::

La vignette **Nombre de détections** affiche le nombre moyen de détections pour chacun des objets classes de détection sélectionnés pendant un intervalle de détection d’une seconde.

Le graphique à secteurs **Classes de détection** affiche le pourcentage de détections pour chaque type de classe.

La **vidéo d’événement d’inférence** fournit la liste des liens vers les ressources dans Azure Media Services qui contiennent les détections. Le lien utilise le lecteur hôte décrit dans la section suivante.

## <a name="start-the-streaming-endpoint"></a>Démarrer le point de terminaison de streaming

Démarrez le point de terminaison Media Services pour permettre à votre application de Lecteur multimédia cliente de diffuser en continu la vidéo enregistrée :

* Dans le portail Azure, accédez au groupe de ressources **lva-rg**.
* Ouvrez la ressource **Point de terminaison de streaming**.
* Dans la page **Détails du point de terminaison de streaming**, sélectionnez **Démarrer**. Un avertissement s’affiche, indiquant que la facturation commence dès le démarrage du point de terminaison.

## <a name="view-stored-video"></a>Afficher une vidéo stockée

L’époque où l’on visionnait les images des caméras et réagissait à celles qui semblaient suspectes est révolue. Grâce à l’étiquetage automatique des événements et aux liens directs vers les vidéos stockées en raison de la détection inférée, les opérateurs de sécurité peuvent trouver des événements d’intérêt dans une liste, puis suivre le lien pour afficher la vidéo correspondante.

Vous pouvez utiliser le [Lecteur multimédia Azure](https://github.com/Azure/live-video-analytics/tree/master/utilities/amp-viewer) pour afficher la vidéo stockée dans votre compte Azure Media Services.

L’application IoT Central stocke la vidéo sur la plateforme Azure Media Services à partir de laquelle vous pouvez la diffuser. Vous avez besoin d’un lecteur vidéo pour lire la vidéo stockée dans Azure Media Services.

Assurez-vous que **Docker** est en cours d’exécution sur votre ordinateur local.

Ouvrez une invite de commandes et utilisez la commande suivante pour exécuter le lecteur vidéo dans un conteneur Docker sur votre ordinateur local. Remplacez les espaces réservés dans la commande par les valeurs que vous avez notées précédemment dans le fichier *scratchpad.txt*. Vous avez noté les valeurs `amsAadClientId`, `amsAadSecret`, `amsAadTenantId`, `amsSubscriptionId` et `amsAccountName` quand vous avez créé le principal du service pour votre compte Media Services :

```bash
docker run -it --rm -e amsAadClientId="<FROM_AZURE_PORTAL>" -e amsAadSecret="<FROM_AZURE_PORTAL>" -e amsAadTenantId="<FROM_AZURE_PORTAL>" -e amsArmAadAudience="https://management.core.windows.net" -e amsArmEndpoint="https://management.azure.com" -e amsAadEndpoint="https://login.microsoftonline.com" -e amsSubscriptionId="<FROM_AZURE_PORTAL>" -e amsResourceGroup="<FROM_AZURE_PORTAL>" -e amsAccountName="<FROM_AZURE_PORTAL>" -p 8094:8094 mcr.microsoft.com/lva-utilities/amp-viewer:1.0-amd64
```

|Paramètre de Docker | Accès à l’API AMS (JSON)|
|-|-|
|amsAadClientId| AadClientId|
|amsAadSecret| AadSecret |
|amsAadTenantId| AadTenantId |
|amsSubscriptionId| SubscriptionId|
|amsResourceGroup| ResourceGroup |
|amsAccountName| AccountName|

Accédez à l’appareil **caméra-003** et sélectionnez l’onglet **Tableau de bord**. Cliquez ensuite sur l’un des liens hypertexte de détection d’objet capturés dans la vignette **Vidéo d’événement d’inférence**. La vidéo apparaît sur une page affichée dans le lecteur vidéo local :

:::image type="content" source="media/tutorial-video-analytics-manage/video-snippet.png" alt-text="Ajouter une caméra":::

## <a name="change-the-simulated-devices-in-application-dashboards"></a>Changer les appareils simulés dans les tableaux de bord d’application

Les tableaux de bord d’application affichent initialement la télémétrie et les propriétés générées à partir d’appareils IoT Central simulés. Pour configurer les vignettes pour la télémétrie de caméras réelles ou du simulateur Live555, procédez comme suit :

1. Accédez au tableau de bord d’application **(exemple) Moniteur de caméra réelle**.
1. Sélectionnez **Modifier**.
1. Sélectionnez la vignette **Note** et supprimez-la.
1. Modifiez le titre du tableau de bord en spécifiant *Moniteur de caméra réelle*.
1. Dans la vignette **Nombre d’inférences**, sélectionnez l’icône Configurer.
1. Dans la section **Configurer un graphique**, dans le groupe d’appareils **LVA Edge Object Detector** (détecteur d’objet périphérique d’analytique vidéo en direct) sélectionnez une ou plusieurs caméras réelles.
1. Sélectionnez le champ de télémétrie `AI Inference Interface/Inference Count`.
1. Sélectionnez **Update**.

1. Répétez les étapes pour les vignettes suivantes :
    1. Le graphique à secteurs **Détection** utilise le type de données de télémétrie `AI Inference Interface/Inference/entity/tag/value`.
    1. La vignette **Inférence** affiche la dernière valeur connue d’`AI Inference Interface/Inference/entity/tag/value`.
    1. La vignette **% Confiance** affiche la dernière valeur connue d’`AI Inference Interface/Inference/entity/tag/confidence`.
    1. La vignette **Capture instantanée** affiche la valeur `AI Inference Interface/Inference Image` sous la forme d’une image.
    1. La vignette **Vidéo d’événement d’inférence** affiche la `AI Inference Interface/Inference Event Video` sous forme d’un lien.
1. Sélectionnez **Enregistrer**.

Le tableau de bord **Moniteur de caméra réelle** affiche maintenant les valeurs de votre caméra réelle :

:::image type="content" source="media/tutorial-video-analytics-manage/update-real-cameras.png" alt-text="Ajouter une caméra":::

## <a name="pause-processing"></a>Suspendre le traitement

Vous pouvez suspendre le traitement de l’analytique vidéo en direct dans l’application :

1. Dans votre application, accédez à la page **Commandes** de la caméra de détection d’objet. Exécutez la commande **Arrêter le traitement d’analytique vidéo en direct**.

1. Arrêtez le point de terminaison de streaming pour votre compte Media Services :
    * Dans le portail Azure, accédez au groupe de ressources **lva-rg**.
    * Cliquez sur la ressource **Point de terminaison de streaming**.
    * Dans la page **Détails du point de terminaison de streaming**, sélectionnez **Arrêter**.

## <a name="tidy-up"></a>Nettoyer

Si vous en avez terminé avec l’application, vous pouvez supprimer toutes les ressources que vous avez créées comme suit :

1. Dans l’application IoT Central, dans la section **Administration**, accédez à la page **Votre application** . Puis sélectionnez **Supprimer**.
1. Dans le portail Azure, supprimez le groupe de ressources **lva-rg**.
1. Sur votre ordinateur local, arrêtez le conteneur Docker **amp-Viewer**.

## <a name="next-steps"></a>Étapes suivantes

Vous savez désormais comment à ajouter des caméras à l’application IoT Central, et les configurer pour la détection d’objet et de mouvement.

Pour savoir comment personnaliser le code source des modules IoT Edge, consultez :

> [!div class="nextstepaction"]
> [Modifier et créer des modules de passerelle d’analytique vidéo en direct](./tutorial-video-analytics-build-module.md)
