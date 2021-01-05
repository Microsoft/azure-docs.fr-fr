---
title: Mappage des flux de données
description: Vue d’ensemble du mappage de flux de données dans Azure Data Factory
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.custom: references_regions
ms.date: 09/02/2020
ms.openlocfilehash: 32f1602c91d17ef2ad3283d4bc03d00d432641aa
ms.sourcegitcommit: daab0491bbc05c43035a3693a96a451845ff193b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "93027599"
---
# <a name="mapping-data-flows-in-azure-data-factory"></a>Flux de données de mappage dans Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

## <a name="what-are-mapping-data-flows"></a>Que sont les flux de données de mappage ?

Les mappages de flux de données sont des transformations de données conçues de manière graphique dans Azure Data Factory. Les flux de données permettent aux ingénieurs de données de développer une logique de transformation des données sans rédiger de code. Les flux de données qui en résultent sont exécutés en tant qu'activités dans les pipelines Azure Data Factory qui utilisent des clusters Apache Spark faisant l'objet d'un scale-out. Les activités de flux de données peuvent être mises en œuvre à l’aide de fonctionnalités de planification, de contrôle, de flux et de supervision Azure Data Factory existantes.

Le flux de données de mappage fournit une expérience entièrement visuelle sans aucun codage. Vos flux de données sont exécutés sur les clusters d'exécution gérés par ADF pour un traitement des données faisant l'objet d'un scale-out. Azure Data Factory gère intégralement la traduction du code, l’optimisation du chemin et l'exécution de vos travaux de flux de données.

## <a name="getting-started"></a>Prise en main

Les flux de données sont créés à partir du volet Ressources de la fabrique, comme les pipelines et jeux de données. Pour créer un flux de données, sélectionnez le signe plus (+) en regard de **Ressources Factory** , puis sélectionnez **Flux de données**. 

![Nouveau flux de données](media/data-flow/new-data-flow.png "nouveau flux de données")

Vous accédez au canevas du flux de données dans lequel vous pouvez créer votre logique de transformation. Sélectionnez **Ajouter une source** pour commencer à configurer votre transformation de la source. Pour plus d’informations, consultez [Transformation de la source](data-flow-source.md).

## <a name="authoring-data-flows"></a>Création de flux de données

Le flux de données de mappage a un canevas de création unique conçu pour faciliter la création de logique de transformation. Le canevas de flux de données est divisé en trois parties : la barre supérieure, le graphe et le panneau de configuration. 

![Capture d’écran montrant le canevas du flux de données avec la barre supérieure, le graphique et le panneau de configuration étiquetés.](media/data-flow/canvas1.png "Canevas")

### <a name="graph"></a>Graph

Le graphe affiche le flux de transformation. Il montre la traçabilité des données sources à mesure qu’elles sont transmises à un ou plusieurs récepteurs. Pour ajouter une nouvelle source, sélectionnez **Ajouter une source**. Pour ajouter une nouvelle transformation, sélectionnez le signe plus (+) situé dans la partie inférieure droite d’une transformation existante. Apprenez à [gérer le graphique des flux de données](concepts-data-flow-manage-graph.md).

![Capture d’écran montrant la partie graphique du canevas avec une zone de texte de recherche.](media/data-flow/canvas2.png "Canevas")

### <a name="configuration-panel"></a>Panneau de configuration

Le panneau de configuration affiche les paramètres spécifiques à la transformation actuellement sélectionnée. Si aucune transformation n’est sélectionnée, le flux de données est affiché. Dans la configuration globale du flux de données, vous pouvez modifier le nom et la description sous l’onglet **Général** ou ajouter des paramètres à l’aide de l’onglet **Paramètres**. Pour plus d’informations, consultez [Paramètres du mappage de flux de données](parameters-data-flow.md).

Chaque transformation contient au moins quatre onglets de configuration.

#### <a name="transformation-settings"></a>Paramètres de transformation

Le premier onglet du volet de configuration de chaque transformation contient les paramètres spécifiques à cette transformation. Pour plus d’informations, reportez-vous à la page de documentation de cette transformation.

![Onglet Paramètres de la source](media/data-flow/source1.png "Onglet Paramètres de la source")

#### <a name="optimize"></a>Optimiser

L’onglet **Optimiser** contient des paramètres pour configurer des schémas de partitionnement. Pour en savoir plus sur l’optimisation de vos flux de données, consultez le [Guide des performances de flux de données de mappage](concepts-data-flow-performance.md).

![La capture d’écran montre l’onglet Optimiser, qui comprend l’option Partition, Type de partition et Nombre de partitions.](media/data-flow/optimize.png)

#### <a name="inspect"></a>Inspecter

Le volet **Inspecter** permet de visualiser les métadonnées du flux de données que vous êtes en train de transformer. Vous pouvez voir le nombre de colonnes, les colonnes modifiées, les colonnes ajoutées, les types de données, l'ordre des colonnes et les références des colonnes. **Inspecter** est un affichage en lecture seule de vos métadonnées. Il n’est pas nécessaire que le mode de débogage soit activé pour voir les métadonnées dans le volet **Inspecter**.

![Inspecter](media/data-flow/inspect1.png "Inspecter")

À mesure que vous modifiez la forme de vos données par le biais de transformations, les changements de métadonnées sont visibles dans le volet **Inspecter**. Si votre transformation de la source ne comporte pas de schéma défini, les métadonnées ne sont pas visibles dans le volet **Inspecter**. L’absence de métadonnées est fréquent dans les scénarios de dérive de schéma.

#### <a name="data-preview"></a>Aperçu des données

Si le mode de débogage est activé, l’onglet **Aperçu des données**  vous donne une capture instantanée interactive des données à chaque transformation. Pour en savoir plus, consultez [Aperçu des données en mode de débogage](concepts-data-flow-debug-mode.md#data-preview).

### <a name="top-bar"></a>Barre supérieure

La barre supérieure contient des actions qui affectent l’ensemble du flux de données comme l’enregistrement et la validation. Vous pouvez également afficher le code JSON sous-jacent et le script de flux de données de votre logique de transformation. Pour plus d’informations, consultez le [script de flux de données](data-flow-script.md).

## <a name="available-transformations"></a>Transformations disponibles

Pour obtenir la liste des transformations disponibles, consultez la [Vue d’ensemble de la transformation de flux de données de mappage](data-flow-transformation-overview.md).

## <a name="data-flow-activity"></a>Activité de flux de données

Les flux de données de mappage sont mis en œuvre dans les pipelines ADF à l’aide de l’[activité de flux de données](control-flow-execute-data-flow-activity.md). Il suffit à l’utilisateur de spécifier le runtime d’intégration à utiliser et de passer des valeurs de paramètre. Pour plus d’informations, consultez [Runtime d’intégration Azure](concepts-integration-runtime.md#azure-integration-runtime).

## <a name="debug-mode"></a>Mode débogage

Le mode débogage vous permet de voir de manière interactive les résultats de chaque étape de transformation pendant que vous générez et déboguez vos flux de données. La session de débogage peut être utilisée à la fois dans lors de la génération de votre logique de flux de données et l’exécution des exécutions de débogage de pipeline avec les activités de flux de données. Pour plus d’informations, consultez la [documentation relative au mode de débogage](concepts-data-flow-debug-mode.md).

## <a name="monitoring-data-flows"></a>Surveillance des flux de données

Le mappage du flux de données s’intègre aux fonctionnalités d’analyse existantes d’Azure Data Factory. Pour savoir comment comprendre la sortie de la surveillance du flux de données, consultez [Supervision des flux de données de mappage](concepts-data-flow-monitoring.md).

L’équipe Azure Data Factory a créé un [guide de réglage des performances](concepts-data-flow-performance.md) pour vous aider à optimiser le temps d’exécution de vos flux de données après avoir généré votre logique métier.

## <a name="available-regions"></a>Régions disponibles

Le mappage des flux de données est disponible dans les régions suivantes :

| Région Azure | Flux de données dans ADF | Flux de données dans Synapse Studio |
| ------------ | ----------------- | ---------------------------- |
|  Centre de l’Australie | | |  
| Centre de l’Australie 2 | | |
| Australie Est | ✓ |  ✓ |
| Sud-Australie Est   | ✓ | ✓ |
| Brésil Sud  | ✓ |  |
| Centre du Canada | ✓ |  |
| Inde centrale | ✓ |   ✓ |
| USA Centre    | ✓ |   ✓ |
| Chine orientale |      | ✓ |
| Chine orientale 2  |   |    |
| Chine Non-régional | | |
| Chine du Nord |     | |
| Chine Nord 2 | |  |
| Asie Est | ✓ | |
| USA Est   | ✓ | ✓ |
| USA Est 2 | ✓ | ✓ |
| France Centre | ✓ | ✓ |
| France Sud  | | |
| Allemagne Centre (souverain) | | |
| Allemagne Non-régional (souverain) | | |
| Allemagne Nord (public) | | |
| Allemagne Nord-Est (souverain) | | |
| Allemagne Centre-Ouest (public) |  | ✓ |
| Japon Est | ✓ |  |
| OuJapon Est |  | |
| Centre de la Corée | ✓ |  |
| Corée du Sud | | |
| Centre-Nord des États-Unis  | ✓ | ✓ |
| Europe Nord  | ✓ |    |
| Norvège Est | | |
| Norvège Ouest | | |
| Afrique du Sud Nord    | ✓ | |
| Afrique du Sud Ouest |  |    |
| États-Unis - partie centrale méridionale  | | ✓ |
| Inde Sud | | |
| Asie Sud-Est    | ✓ | ✓ |
| Suisse Nord |   |  |
| Suisse Ouest | | |
| Émirats arabes unis Centre | | |
| Émirats arabes unis Nord |  |    |
| Sud du Royaume-Uni  | ✓ |   | ✓ |
| Ouest du Royaume-Uni |     | ✓ |
| Centre des États-Unis – US DoD | |  |
| Est des États-Unis – US DoD | |  |
| Gouvernement des États-Unis – Arizona |      |  |
| US Gov Non-Régional | |  |
| Gouvernement des États-Unis – Texas | |  |
| Gouvernement américain - Virginie |     |  |
| Centre-USA Ouest |     | ✓ |
| Europe Ouest   | ✓ |   ✓ |
| Inde Ouest | | |
| USA Ouest   | ✓ |   |
| USA Ouest 2 | ✓ |   ✓ | 

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment créer une [transformation de la source](data-flow-source.md).
* Découvrez comment créer vos flux de données en [mode de débogage](concepts-data-flow-debug-mode.md).
