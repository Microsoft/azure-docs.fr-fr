---
title: Afficher et configurer les métriques du connecteur Azure IoT pour FHIR (préversion)
description: Cet article explique comment afficher et configurer les métriques du Connecteur Azure IoT pour FHIR (préversion).
services: healthcare-apis
author: msjasteppe
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: how-to
ms.date: 11/13/2020
ms.author: jasteppe
ms.openlocfilehash: 79a7b18986f4bed503cd5493a7d74325a13fe535
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94630513"
---
# <a name="display-and-configure-azure-iot-connector-for-fhir-preview-metrics"></a>Afficher et configurer les métriques du connecteur Azure IoT pour FHIR (préversion) 

Dans cet article, vous allez découvrir comment afficher et configurer les métriques du Connecteur Azure IoT pour FHIR&#174;* (Fast Healthcare Interoperability Resources).

> [!TIP]
> Pour savoir comment configurer l’exportation des données de métriques, suivez les instructions dans [Exporter les métriques du connecteur Azure IoT pour FHIR (préversion) via les paramètres de diagnostic](./iot-metrics-diagnostics-export.md).

## <a name="display-metrics-for-azure-iot-connector-for-fhir-preview"></a>Afficher les métriques pour le Connecteur Azure IoT pour FHIR (préversion)

1. Connectez-vous au portail Azure, puis sélectionnez votre service API Azure pour FHIR. 

2. Dans le volet de gauche, sélectionnez **Métriques**. 

3. Sélectionnez l’onglet **Connecteur IoT**.

   :::image type="content" source="media/iot-metrics-display/iot-metrics-main.png" alt-text="Capture d’écran du volet « Connecteur IoT », avec des graphiques linéaires qui affichent le nombre de messages entrants et normalisés." lightbox="media/iot-metrics-display/iot-metrics-main.png"::: 

4. Sélectionnez un connecteur IoT pour en afficher les métriques. Par exemple, vous pouvez noter que quatre connecteurs IoT (*connecteur 1*, *connecteur 2*, etc.) sont ici associés à ce service API Azure pour FHIR.

   :::image type="content" source="media/iot-metrics-display/iot-metrics-select-connector.png" alt-text="Capture d’écran du volet « Connecteur IoT », affichant les onglets des connecteurs IoT 1, 2, 3 et 4." lightbox="media/iot-metrics-display/iot-metrics-select-connector.png"::: 

5. Sélectionnez la période (par exemple, **1 heure**, **24 heures**, **7 jours** ou **Personnalisé**) des métriques du connecteur IoT à afficher. En sélectionnant l’onglet **Personnalisé**, vous pouvez créer des combinaisons d’heures et de dates spécifiques pour afficher les métriques de connecteur IoT.

   :::image type="content" source="media/iot-metrics-display/iot-metrics-select-time.png" alt-text="Capture d’écran du volet « Connecteur IoT » affichant un graphique linéaire sur une période de « 1 heure » pour le « connecteur 1 »." lightbox="media/iot-metrics-display/iot-metrics-select-time.png"::: 
 
## <a name="metric-types-for-azure-iot-connector-for-fhir-preview"></a>Types de métriques pour le Connecteur Azure IoT pour FHIR (préversion) 

> [!TIP]
> Consultez [Flux de données du connecteur Azure IoT pour FHIR (préversion)](./iot-data-flow.md) pour en savoir plus sur le flux de données dans le connecteur Azure IoT pour FHIR et [Guide de dépannage du connecteur Azure IoT pour FHIR (préversion)](./iot-troubleshoot-guide.md) pour en savoir plus sur les messages d’erreur et les correctifs.

Les métriques du connecteur IoT que vous pouvez afficher sont listées dans le tableau suivant :

|Type de métrique|Objectif de la métrique| 
|-----------|--------------|
|Nombre de messages entrants|Affiche le nombre de messages entrants bruts reçus (par exemple, les événements d’appareil).|
|Nombre de messages normalisés|Affiche le nombre de messages normalisés.|
|Nombre de groupes de messages|Affiche le nombre de groupes dont les messages sont agrégés dans la fenêtre temporelle désignée.|
|Latence moyenne de la phase normalisée|Affiche la latence moyenne de la phase normalisée. La phase normalisée assure la normalisation des messages entrants bruts.|
|Latence moyenne de la phase de regroupement|Affiche la latence moyenne de la phase de regroupement. La phase de regroupement assure la mise en mémoire tampon, l’agrégation et le regroupement des messages normalisés.| 
|Nombre total d'erreurs|Affiche le nombre total d’erreurs.| 

## <a name="focus-on-and-configure-azure-iot-connector-for-fhir-preview-metrics"></a>Mettre en évidence et configurer les métriques du Connecteur Azure IoT pour FHIR (préversion)

Dans cet exemple, focalisons-nous sur la métrique **Nombre de messages entrants**.

1. Sélectionnez un point dans le temps sur lequel vous souhaitez vous concentrer.

   :::image type="content" source="media/iot-metrics-display/iot-metrics-focus.png" alt-text="Capture d’écran du volet de la métrique « Nombre de messages entrants », avec un point dans le temps unique mis en évidence dans le graphique linéaire." lightbox="media/iot-metrics-display/iot-metrics-focus.png"::: 

2. Dans le volet **Nombre de messages entrants**, vous pouvez personnaliser davantage la métrique en sélectionnant **Ajouter une métrique**, **Ajouter un filtre** ou **Appliquer le fractionnement**. 

   :::image type="content" source="media/iot-metrics-display/iot-metrics-add-options.png" alt-text="Capture d’écran du volet de la métrique « Nombre de messages entrants », avec les boutons « Ajouter une métrique », « Ajouter un filtre » et « Appliquer le fractionnement » mis en évidence." lightbox="media/iot-metrics-display/iot-metrics-add-options.png"::: 

## <a name="conclusion"></a>Conclusion 
L’accès aux métriques du plan de données est essentiel pour la supervision et la résolution des problèmes. Le Connecteur Azure IoT pour FHIR vous aide à effectuer ces actions via les métriques. 

## <a name="next-steps"></a>Étapes suivantes

Obtenez des réponses aux questions fréquentes sur le Connecteur Azure IoT pour FHIR.

>[!div class="nextstepaction"]
>[FAQ sur le Connecteur Azure IoT pour FHIR](fhir-faq.md)

*Sur le portail Azure, le Connecteur Azure IoT pour FHIR est appelé Connecteur IoT (préversion). Le sigle FHIR est une marque déposée de HL7 et est utilisé avec l’autorisation de HL7. 
