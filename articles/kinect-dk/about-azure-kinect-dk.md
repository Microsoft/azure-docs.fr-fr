---
title: À propos d’Azure Kinect DK
description: Vue d’ensemble des outils du kit de développement Azure Kinect DK et des services intégrés.
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.topic: overview
ms.date: 06/26/2019
keywords: azure, kinect, vue d’ensemble, kit de développement, DK, appareil, profondeur, suivi des corps, voix, services cognitifs, kits de développement logiciel, SDK, microprogramme
ms.openlocfilehash: d188d37c0247aebb16f51b4404da81d469136468
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94359651"
---
# <a name="about-azure-kinect-dk"></a>À propos d’Azure Kinect DK

 ![Azure Kinect DK](./media/index/device-image.jpg)

Azure Kinect DK est un kit de développement doté de capteurs d’intelligence artificielle avancés qui fournissent des modèles vocaux et de vision informatique sophistiqués.  Kinect contient un capteur de profondeur, un réseau de microphones spatiaux avec une caméra vidéo et un capteur d’orientation, le tout présenté sous la forme d’un petit appareil tout-en-un doté de plusieurs modes, options et SDK. Il est vendu sur [Microsoft Store en ligne](https://www.microsoft.com/p/azure-kinect-dk/8pp5vxmd9nhq).

L’environnement de développement Azure Kinect DK comprend les kits SDK suivants :

- Capteur pour un accès aux appareils et capteurs de bas niveau.
- Suivi des corps pour suivre les corps en 3D.
- Speech Cognitive Services pour activer l’accès aux microphones ainsi que les services vocaux basés sur le cloud Azure.

Par ailleurs, les services Cognitive Vision peuvent être utilisés avec la caméra RVB de l’appareil.

   ![Diagramme des SDK Azure Kinect](./media/quickstarts/sdk-diagram.jpg)

## <a name="azure-kinect-sensor-sdk"></a>SDK Capteur d’Azure Kinect

Le SDK Capteur d’Azure Kinect donne accès aux capteurs de bas niveau pour la configuration des appareils et des capteurs matériels Azure Kinect DK.

Pour plus d’informations sur le SDK Capteur d’Azure Kinect, consultez [Utilisation du SDK Capteur](about-sensor-sdk.md).

### <a name="azure-kinect-sensor-sdk-features"></a>Fonctionnalités du SDK Capteur d’Azure Kinect

Les fonctionnalités du SDK Capteur ci-dessous fonctionnent dès lors que le SDK est installé et exécuté sur Azure Kinect DK :

- Accès aux caméras de profondeur et contrôle du mode (mode IR passif et modes de profondeur à champ de vision large et étroit) 
- Accès aux caméras RVB et contrôle de celles-ci (par exemple, exposition et balance des blancs) 
- Accès aux capteurs de mouvement (gyroscope et accéléromètre) 
- Diffusion synchronisée des données des caméras RVB de profondeur avec la possibilité de configurer le délai entre les caméras 
- Contrôle de synchronisation des appareils externes avec la possibilité de configurer le décalage entre les appareils 
- Accès aux métadonnées de vue de caméra pour la résolution d’image, l’horodatage, etc. 
- Accès aux données d’étalonnage des appareils 

### <a name="azure-kinect-sensor-sdk-tools"></a>Outils du SDK Capteur d’Azure Kinect

Le SDK Capteur propose les outils suivants :

- Outil de visualisation permettant de superviser les flux de données des appareils et de configurer les différents modes.
- Un outil d’enregistrement des données de capteur et une API de lecture utilisant le format de conteneur Matroska.
- Un outil de mise à jour de microprogramme Azure Kinect DK.

## <a name="azure-kinect-body-tracking-sdk"></a>SDK Suivi des corps d’Azure Kinect

Le SDK Suivi des corps comprend une bibliothèque et un runtime Windows qui permettent de suivre les corps en 3D avec le matériel Azure Kinect DK.

### <a name="azure-kinect-body-tracking-features"></a>Fonctionnalités du SDK Suivi des corps d’Azure Kinect

Le SDK fourni propose les fonctionnalités de suivi des corps suivantes :

- Assure une segmentation des corps.
- Contient un squelette à l’anatomie correcte pour chaque corps partiel ou complet présent dans le champ de vision.
- Attribue à chaque corps une identité unique.
- Peut assurer le suivi des corps dans le temps.

### <a name="azure-kinect-body-tracking-tools"></a>Outils du SDK Suivi des corps d’Azure Kinect

- Le dispositif de suivi des corps intègre un outil de visualisation destiné à suivre les corps en 3D.

## <a name="speech-cognitive-services-sdk"></a>SDK Speech Cognitive Services

Le SDK Speech permet d’activer les services vocaux connectés à Azure.

### <a name="speech-services"></a>Speech Services

- Reconnaissance vocale
- Traduction vocale
- Synthèse vocale

>[!NOTE]
>Azure Kinect DK n’est pas équipé de haut-parleurs.

Pour plus d’informations, consultez la [documentation Speech Service](../cognitive-services/speech-service/index.yml).

## <a name="vision-services"></a>Services de vision

[Azure Cognitive Vision Services](https://azure.microsoft.com/services/cognitive-services/directory/vision/) propose des services Azure capables d’identifier et d’analyser le contenu des images et des vidéos.

- [Vision par ordinateur](https://azure.microsoft.com/services/cognitive-services/computer-vision/)
- [Visage](https://azure.microsoft.com/services/cognitive-services/face/)
- [Video Indexer](https://azure.microsoft.com/services/media-services/video-indexer/)
- [Content Moderator](https://azure.microsoft.com/services/cognitive-services/content-moderator/)
- [Custom Vision](https://azure.microsoft.com/services/cognitive-services/custom-vision-service/)

Étant donné que les services évoluent et s’améliorent en permanence, pensez à vérifier régulièrement s’il n’existe pas de nouveaux services [Cognitive Services](https://azure.microsoft.com/services/cognitive-services/) pour améliorer votre application. Pour obtenir des informations à jour sur les derniers services disponibles, consultez la page des [labs Cognitive Services](https://labs.cognitive.microsoft.com/).

## <a name="azure-kinect-hardware-requirements"></a>Configuration matérielle requise pour Azure Kinect

Azure Kinect DK intègre les dernières technologies de capteur de Microsoft dans un même accessoire connecté par USB. Pour plus d’informations, consultez [Spécifications matérielles Azure Kinect DK](hardware-specification.md).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez une idée globale d’Azure Kinect DK, le moment est venu d’aller plus loin et de le configurer.

> [!div class="nextstepaction"]
>[Démarrage rapide : Configurer Azure Kinect DK](set-up-azure-kinect-dk.md)