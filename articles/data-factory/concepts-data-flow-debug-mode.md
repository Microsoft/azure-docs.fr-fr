---
title: Mode de débogage du mappage de flux de données
description: Démarrer une session de débogage interactive lors de la génération de flux de données
ms.author: makromer
author: kromerm
ms.reviewer: douglasl
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 12/04/2020
ms.openlocfilehash: 87465ff381c62343a11c54130378b48580ca40e2
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96621640"
---
# <a name="mapping-data-flow-debug-mode"></a>Mode de débogage du mappage de flux de données

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

## <a name="overview"></a>Vue d’ensemble

Le mode de débogage du flux de données de mappage d’Azure Data Factory permet de suivre de manière interactive la transformation de la forme des données à mesure que vous générez et déboguez vos flux de données. La session de débogage peut être utilisée dans des sessions de conception de flux de données et lors de l’exécution du débogage de flux de données dans un pipeline. Pour activer le mode de débogage, utilisez le bouton « Débogage de flux de données » en haut de l’aire de conception.

![Curseur de débogage](media/data-flow/debugbutton.png "Curseur de débogage")

Quand vous activez le curseur, vous êtes invité à sélectionner la configuration de runtime d’intégration à utiliser. Si vous choisissez AutoResolveIntegrationRuntime, un cluster comprenant 8 cœurs de calcul général avec une durée de vie par défaut de 60 minutes est lancé. Si vous souhaitez autoriser un plus grand nombre d’équipes inactives avant l’expiration de votre session, vous pouvez choisir un paramètre TTL plus élevé. Pour plus d’informations sur les runtimes d’intégration de flux de données, consultez [Performances de flux de données](concepts-data-flow-performance.md#ir).

![Déboguer la sélection IR](media/data-flow/debug-new-1.png "Déboguer la sélection IR")

Lorsque le mode débogage est activé, vous allez générer de manière interactive votre flux de données avec un cluster Spark actif. La session se ferme dès que vous désactivez le débogage dans Azure Data Factory. Prenez connaissance des frais horaires engendrés par Azure Databricks pendant la durée d’activation de la session de débogage.

Dans la plupart des cas, nous vous recommandons de créer vos flux de données en mode débogage pour que vous puissiez valider votre logique métier et afficher vos transformations de données avant de publier votre travail dans Azure Data Factory. Utilisez le bouton « Débogage » sur le panneau du pipeline pour tester votre flux de données dans un pipeline.

![Afficher les sessions de débogage du flux de données](media/iterative-development-debugging/view-dataflow-debug-sessions.png)

> [!NOTE]
> Chaque session de débogage qu’un utilisateur démarre à partir de l’interface utilisateur de son navigateur ADF est une nouvelle session avec son propre cluster Spark. Vous pouvez utiliser la vue de supervision pour les sessions de débogage ci-dessus pour afficher et gérer les sessions de débogage par fabrique. Vous êtes facturé pour chaque heure d’exécution de chaque session de débogage, y compris la durée de vie.

## <a name="cluster-status"></a>État du cluster

L’indicateur d’état du cluster en haut de l’aire de conception devient vert quand le cluster est prêt pour le débogage. Si votre cluster est déjà chaud, l’indicateur vert apparaît presque instantanément. Si votre cluster n’est pas en cours d’exécution quand vous entrez en mode débogage, vous devez attendre 5 à 7 minutes avant que le cluster ne soit opérationnel. L’indicateur tournera jusqu’à être prêt.

Une fois le débogage terminé, désactivez-le à l’aide du commutateur pour que votre cluster Azure Databricks puisse se terminer. Vous n’êtes alors plus facturé pour l’activité de débogage.

## <a name="debug-settings"></a>Paramètres de débogage

Une fois que vous activez le mode débogage, vous pouvez modifier la façon dont un flux de données affiche un aperçu des données. Les paramètres de débogage peuvent être modifiés en cliquant sur « Paramètres de débogage » sur la barre d’outils du canevas Flux de données. Vous pouvez sélectionner la limite de ligne ou la source de fichier à utiliser pour chacune de vos transformations Source ici. Les limites de lignes de ce paramètre s’appliquent uniquement à la session de débogage actuelle. Vous pouvez aussi sélectionner le service lié intermédiaire à utiliser pour une source Azure Synapse Analytics. 

![Paramètres de débogage](media/data-flow/debug-settings.png "Paramètres de débogage")

Si vous avez des paramètres dans votre flux de données ou dans l’un de ses jeux de données référencés, vous pouvez spécifier les valeurs à utiliser pendant le débogage en sélectionnant l'onglet **Paramètres**.

![Configuration des paramètres de débogage](media/data-flow/debug-settings2.png "Configuration des paramètres de débogage")

Le runtime d’intégration par défaut utilisé pour le mode débogage dans les flux de données ADF est un petit nœud Worker à 4 cœurs avec un nœud de pilote unique à 4 cœurs. Ceci fonctionne correctement avec des échantillons de données réduits quand vous testez la logique de votre flux de données. Si vous étendez les limites du nombre de lignes dans vos paramètres de débogage pour l’aperçu des données ou que vous définissez un nombre plus élevé de lignes échantillonnées dans votre source pour le débogage du pipeline, vous pouvez envisager de définir un environnement de calcul plus grand dans un nouvel Azure Integration Runtime. Vous pouvez ensuite redémarrer votre session de débogage et utiliser l’environnement de calcul agrandi.

## <a name="data-preview"></a>Aperçu des données

Quand le débogage est activé, l’onglet d’aperçu des données s’allume dans le panneau inférieur. Si le mode débogage n’est pas activé, Data Flow affiche uniquement les métadonnées entrantes et sortantes pour chacune de vos transformations sous l’onglet Inspecter. L’aperçu des données interroge uniquement le nombre de lignes que vous avez défini comme limite dans vos paramètres de débogage. Cliquez sur **Actualiser** pour récupérer l’aperçu des données.

![Aperçu des données](media/data-flow/datapreview.png "Aperçu des données")

> [!NOTE]
> Les sources de fichier limitent uniquement les lignes que vous voyez et non celles qui sont en cours de lecture. Pour les jeux de données très volumineux, il est recommandé de prendre une petite partie de ce fichier et de l’utiliser pour votre test. Vous pouvez sélectionner un fichier temporaire dans les paramètres de débogage pour chaque source correspondant à un type de jeu de données de fichier.

Quand vous exécutez le mode débogage dans Data Flow, vos données ne sont pas écrites dans la transformation Sink. Le but d’une session de débogage est de servir d’atelier de test pour vos transformations. Les récepteurs ne sont pas obligatoires durant le débogage et sont ignorés dans votre flux de données. Si vous souhaitez tester l’écriture des données dans votre récepteur, exécutez le flux de données à partir d’un pipeline Azure Data Factory et utilisez l’exécution Débogage à partir d’un pipeline.

L’aperçu des données est un instantané de vos données transformées qui utilise les limites du nombre de lignes et l’échantillonnage des données provenant des trames de données dans la mémoire Spark. Par conséquent, les pilotes du récepteur ne sont pas utilisés ou testés dans ce scénario.

### <a name="testing-join-conditions"></a>Test des conditions de jointure

Lors du test unitaire des transformations de jointure, de recherche ou Exists, veillez à utiliser un petit ensemble de données connues pour votre test. Vous pouvez utiliser l’option Paramètres de débogage évoquée plus haut pour définir un fichier temporaire à utiliser pour votre test. Cette opération est nécessaire, car lors de la limitation ou de l’échantillonnage de lignes à partir d’un jeu de données volumineux, vous ne pouvez pas prédire quelles lignes et quelles clés seront lues dans le flux dans le cadre du test. Le résultat n’est pas déterministe, ce qui signifie que vos conditions de jointure peuvent échouer.

### <a name="quick-actions"></a>Actions rapides

Une fois que vous voyez l’aperçu des données, vous pouvez générer une transformation rapide pour convertir en typecast, supprimer ou modifier une colonne. Cliquez sur l’en-tête de colonne, puis sélectionnez l’une des options dans la barre d’outils d’aperçu des données.

![Capture d’écran montrant la barre d’outils d’aperçu des données avec les options : Typecast, Modifier, Statistiques et Supprimer.](media/data-flow/quick-actions1.png "Actions rapides")

Une fois que vous avez sélectionné une modification, l’aperçu des données est immédiatement actualisé. Cliquez sur **Confirmer** dans le coin supérieur droit pour générer une nouvelle transformation.

![Capture d’écran montrant le bouton Confirmer.](media/data-flow/quick-actions2.png "Actions rapides")

**Typecast** et **Modifier** permettent de générer une transformation de colonne dérivée et **Supprimer** de générer une transformation de sélection (Select).

![Capture d’écran montrant les paramètres de la colonne dérivée.](media/data-flow/quick-actions3.png "Actions rapides")

> [!NOTE]
> Si vous modifiez votre flux de données, vous devez extraire à nouveau l’aperçu des données avant d’ajouter une transformation rapide.

### <a name="data-profiling"></a>Profilage des données

Quand vous sélectionnez une colonne sous l’onglet d’aperçu des données et cliquez sur **Statistiques** dans la barre d’outils de l’aperçu des données, un graphique s’affiche à l’extrême droite de votre grille de données avec des statistiques détaillées sur chaque champ. Azure Data Factory détermine le type de graphique à afficher en fonction de l’échantillonnage des données. Les champs à cardinalité élevée affichent par défaut des graphiques NULL/NOT NULL tandis que les données catégoriques et numériques à cardinalité faible affichent des graphiques à barres indiquant la fréquence des valeurs de données. Les champs suivants sont également disponibles : longueur max/len des chaînes, valeurs min/max dans les champs numériques, écart type, centiles, comptes et moyenne.

![Statistiques de colonne](media/data-flow/stats.png "Statistiques de colonne")

## <a name="next-steps"></a>Étapes suivantes

* Une fois que vous avez terminé la création et le débogage de votre flux de données, [exécutez-le depuis un pipeline](control-flow-execute-data-flow-activity.md).
* Lors du test de votre pipeline avec un flux de données, utilisez l’[option Exécution du débogage](iterative-development-debugging.md) du pipeline.
