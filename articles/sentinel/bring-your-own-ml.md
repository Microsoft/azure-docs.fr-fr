---
title: Intégrer votre propre Machine Learning dans Azure Sentinel | Microsoft Docs
description: Cet article explique comment créer et utiliser vos propres algorithmes d’apprentissage automatique pour l’analyse de données dans Azure Sentinel.
services: sentinel
cloud: na
documentationcenter: na
author: yelevin
manager: rkarlin
ms.assetid: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/23/2020
ms.author: yelevin
ms.openlocfilehash: 6a891a301d5869603a7d90d28bb9063d7d5bdb1d
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660574"
---
# <a name="bring-your-own-machine-learning-ml-into-azure-sentinel"></a>Intégrer votre propre Machine Learning (BYO-ML) dans Azure Sentinel

Le Machine Learning (ML ou apprentissage automatique) est l’un des fondements majeurs d’Azure Sentinel, et l’un des principaux attributs qui le distinguent. Azure Sentinel offre l’apprentissage automatique au travers de diverses expériences : intégré au moteur de corrélation [Fusion](fusion.md) et aux blocs-notes Jupyter, ainsi qu’à la nouvelle plateforme BYO ML (Construire votre propre ML). 

Les modèles de détection de ML peuvent s’adapter à des environnements individuels ainsi qu’à des modifications du comportement des utilisateurs, afin de réduire le nombre de faux positifs et d’identifier des menaces qu’une approche traditionnelle ne permettrait pas de détecter. Si bon nombre d’organisations de sécurité comprennent la valeur de l’apprentissage automatique pour la sécurité, la plupart d’entre elles n’ont pas les moyens de s’entourer de professionnels disposant d’une expertise tant en matière de sécurité qu’en matière d’apprentissage automatique. Nous avons conçu l’infrastructure présentée ici afin de permettre aux organisations et aux professionnels de la sécurité de se développer avec nous en lien avec l’apprentissage automatique. Les organisations qui découvrent l’apprentissage automatique ou qui ne disposent pas de l’expertise nécessaire peuvent trouver une protection significative dans les fonctionnalités intégrées d’apprentissage automatique d’Azure Sentinel.

:::image type="content" source="./media/bring-your-own-ml/machine-learning-framework.png" alt-text="Infrastructure d’apprentissage automatique":::

## <a name="what-is-the-bring-your-own-machine-learning-byo-ml-platform"></a>Qu’est-ce que la plateforme Intégrer votre propre Machine Learning (BYO-ML) ?

Pour les organisations disposant de ressources de Machine Learning (ML) et désireuses de créer des modèles ML personnalisés répondant à leurs besoins spécifiques, nous proposons la **plateforme BYO-ML**. La plateforme utilise l’environnement [Azure Databricks](/azure/databricks/scenarios/what-is-azure-databricks)/[Apache Spark](http://spark.apache.org/) et des blocs-notes Jupyter pour produire l’environnement ML. Elle fournit les composants suivants :

- un package BYO-ML comprenant des bibliothèques destinées à vous aider à accéder aux données et à renvoyer les résultats à Log Analytics (LA) afin que vous puissiez les intégrer avec vos activités de détection, d’investigation et de repérage ; 

- des modèles d’algorithmes ML que vous pouvez personnaliser pour résoudre des problèmes de sécurité spécifiques dans votre organisation ; 

- des exemples de blocs-notes pour effectuer l’apprentissage du modèle et planifier son scoring. 

En outre, vous pouvez apporter vos propres modèles ML et/ou votre propre environnement Spark en vue d’une intégration avec Azure Sentinel.

Avec la plateforme BYO-ML, vous pouvez commencer à créer vos propres modèles ML : 

- Le bloc-notes avec des exemples de données vous permet d’acquérir une expérience pratique de bout en bout, sans vous soucier de la gestion des données de production.

- Le package intégré avec l’environnement Spark réduit les difficultés et les frictions en lien avec la gestion de l’infrastructure.

- Les bibliothèques prennent en charge les déplacements des données. Les blocs-notes de formation et de scoring illustrent l’expérience de bout en bout et font office de modèle que vous pouvez adapter à votre environnement.

### <a name="use-cases"></a>Cas d'utilisation
 
La plateforme et le package BYO-ML réduisent sensiblement le temps et les efforts que vous devrez consacrer à la création de vos propres détections ML, et facilitent la résolution de problèmes de sécurité spécifiques dans Azure Sentinel. La plateforme prend en charge les cas d’utilisation suivants :

**Effectuer l’apprentissage d’un algorithme ML pour obtenir un modèle personnalisé :** vous pouvez prendre un algorithme ML existant (partagé par Microsoft ou la communauté des utilisateurs) et en effectuer l’apprentissage facilement sur vos propres données afin d’obtenir un modèle ML personnalisé mieux adapté à vos données et à votre environnement.

**Modifier un modèle d’algorithme ML pour obtenir un modèle personnalisé :** vous pouvez modifier un modèle d’algorithme ML (partagé par Microsoft ou la communauté d’utilisateurs) et effectuer l’apprentissage de l’algorithme modifié sur vos propres données, afin de dériver un modèle personnalisé adapté à votre problème spécifique.

**Créer votre propre modèle :** vous pouvez créer votre propre modèle de toutes pièces en vous servant de la plateforme et des utilitaires BYO-ML d’Azure Sentinel.

**Intégrer votre environnement Databricks/Spark :** vous pouvez intégrer votre environnement Databricks/Spark existant dans Azure Sentinel, et utiliser des bibliothèques et modèles BYO-ML pour créer des modèles ML adaptés à des situations uniques.

**Importer votre propre modèle ML :** vous pouvez importer vos propres modèles ML et utiliser la plateforme et les utilitaires BYO-ML pour les intégrer avec Azure Sentinel.

**Partager un algorithme ML :** vous pouvez partager un algorithme ML afin que la communauté l’adopte et l’adapte.

**Utiliser ML pour consolider les SecOP :** vous pouvez utiliser votre propre modèle ML personnalisé et ses résultats à des fins de repérage, de détection, d’investigation et de réponse.

Cet article présente les composants de la plateforme BYO-ML, et explique comment tirer parti de celle-ci ainsi que de l’algorithme d’accès anormal aux ressources pour fournir une détection ML personnalisée avec Azure Sentinel.

## <a name="azure-databricksspark-environment"></a>Environnement Azure Databricks/Spark

[Apache Spark™](http://spark.apache.org/) a fait un bond en avant en simplifiant le Big Data par la fourniture d’une infrastructure unifiée pour la création de pipelines de données. Azure Databricks va plus loin en fournissant une plateforme cloud basée sur Spark, ne nécessitant aucune administration. Nous vous recommandons d’utiliser Databricks pour votre plateforme BYO-ML afin de pouvoir vous concentrer sur la recherche de réponses qui ont un impact immédiat sur votre activité, au lieu de vous attaquer aux problèmes de plateforme et pipelines de données.
Si vous disposez déjà de Databricks ou d’un autre environnement Spark, et préférez utiliser l’installation existante, le package BYO-ML fonctionnera correctement sur ces derniers. 

## <a name="byo-ml-package"></a>Package BYO-ML

Le package BYO-ML inclut les meilleures pratiques et les recherches de Microsoft dans la partie frontale du ML pour la sécurité. Ce package fournit la liste suivante d’utilitaires, de blocs-notes et de modèles d’algorithmes en lien avec les problèmes de sécurité.

| Nom de fichier | Description |
| --------- | ----------- |
| azure_sentinel_utilities.whl | Contient des utilitaires pour la lecture de blobs d’Azure et l’écriture dans Log Analytics. |
| AnomalousRASampleData | Bloc-notes montrant l’utilisation d’un modèle d’accès anormal aux ressources dans Sentinel avec des exemples de données d’apprentissage et de test générés. |
| AnomalousRATraining.ipynb | Bloc-notes pour effectuer l’apprentissage de l’algorithme, générer et enregistrer les modèles. |
| AnomalousRAScoring.ipynb | Bloc-notes pour planifier l’exécution du modèle, visualiser le résultat et écrire le score dans Azure Sentinel. |
|

Le premier modèle d’algorithme ML que nous proposons est destiné à la [détection d’accès anormal aux ressources](https://github.com/Azure/Azure-Sentinel/tree/master/BYOML). Il est basé sur un algorithme de filtrage collaboratif et son apprentissage est effectué avec les journaux d’accès au partage de fichiers Windows (événements de sécurité avec l’ID d’événement 5140). Les informations clés dont vous avez besoin pour ce modèle dans le journal sont le couplage d’utilisateurs et de ressources utilisées. 

## <a name="example-walkthrough-anomalous-file-share-access-detection"></a>Exemple de procédure pas à pas : détection d’accès anormal au partage de fichiers 

À présent que vous êtes familiarisé avec les principaux composants de la plateforme BYO-ML, voici un exemple montrant comment utiliser celle-ci et ses composants pour fournir une détection ML personnalisée.

### <a name="setup-the-databricksspark-environment"></a>Configurer l’environnement Databricks/Spark

Si vous ne disposez pas encore de votre propre environnement Databricks, vous devez l’installer. Pour obtenir des instructions, consultez le document [Démarrage rapide d’Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal?tabs=azure-portal).

### <a name="auto-export-instruction"></a>Instruction d’exportation automatique

Pour créer des modèles ML personnalisés basés sur vos propres données dans Sentinel, vous devez exporter vos données de Log Analytics vers une ressource Stockage Blob ou Event Hub, afin que le modèle ML puisse y accéder à partir de Databricks. Découvrez comment [ingérer des données dans Azure Sentinel](connect-data-sources.md).

Pour cet exemple, vous devez disposer de vos données d’apprentissage pour le journal d’accès au partage de fichiers dans le stockage d’objets blob Azure. Le format des données est documenté dans le bloc-notes et les bibliothèques.

Vous pouvez exporter automatiquement vos données de Log Analytics à l’aide de l’[interface de ligne de commande (CLI) Azure](/cli/azure/monitor/log-analytics). 

Pour exécuter les commandes, vous devez être titulaire du rôle **Contributeur** dans votre espace de travail Log Analytics, votre compte de stockage et votre ressource EventHub. 

Voici un exemple de jeu de commandes pour configurer l’exportation automatique :

```azurecli

az –version

# Login with Azure CLI
 az login

# List all Log Analytics clusters
 az monitor log-analytics cluster list

# Set to specific subscription
 az account set --subscription "SUBSCRIPTION_NAME"
 
# Export to Storage - all tables
 az monitor log-analytics workspace data-export create --resource-group "RG_NAME" --workspace-name "WS_NAME" -n LAExportCLIStr --destination “DESTINATION_NAME" --enable "true" --tables SecurityEvent
 
# Export to EventHub - all tables
 az monitor log-analytics workspace data-export create --resource-group "RG_NAME" --workspace-name "WS_NAME" -n LAExportCLIEH --destination “DESTINATION_NAME" --enable "true" --tables SecurityEvent Heartbeat"]

# List export settings
az monitor log-analytics workspace data-export list --resource-group "RG_NAME" --workspace-name "WS_NAME"

# Delete export setting
 az monitor log-analytics workspace data-export delete --resource-group "RG_NAME" --workspace-name "WS_NAME" --name "NAME"
```

### <a name="export-custom-data"></a>Exporter des données personnalisées

Concernant vos données personnalisées qui ne sont pas prises en charge par l’exportation automatique de Log Analytics, vous pouvez utiliser une application logique ou d’autres solutions pour les déplacer. Vous pouvez vous référer au blog et au script [Exportation de données de Log Analytics vers un magasin d’objets blob](https://www.borninthecloud.com/exporting-log-analytics-data-to-blob-store/?preview=true).

### <a name="correlate-with-data-outside-of-azure-sentinel"></a>Établir une corrélation avec des données en dehors d’Azure Sentinel

Vous pouvez également importer des données extérieures à Azure Sentinel vers le Stockage blob ou l’Event Hub, et les mettre en corrélation avec les données de Sentinel pour créer vos modèles ML. 
 
### <a name="copy-and-install-the-related-packages"></a>Copier et installer les packages associés

Copiez le package BYO-ML à partir du dépôt GitHub Azure Sentinel mentionné plus haut dans votre environnement Databricks. Ouvrez ensuite les blocs-notes et suivez les instructions qu’ils contiennent pour installer les bibliothèques requises sur vos clusters.

### <a name="model-training-and-scoring"></a>Apprentissage et scoring du modèle

Suivez les instructions figurant dans les deux blocs-notes pour modifier les configurations en fonction de votre environnement et de vos ressources, suivez les étapes d’apprentissage et de génération de votre modèle, puis planifiez le modèle pour effectuer le scoring des journaux d’accès entrant au partage de fichiers.

### <a name="write-results-to-log-analytics"></a>Écrire les résultats dans Log Analytics

Une fois que vous avez planifié le scoring, vous pouvez utiliser le module figurant dans le bloc-notes de scoring pour écrire les résultats du scoring dans l’espace de travail Log Analytics associé à votre instance Azure Sentinel.

### <a name="check-results-in-azure-sentinel"></a>Vérifier les résultats dans Azure Sentinel

Pour afficher vos résultats de scoring avec les détails du journal associés, revenez à votre portail Azure Sentinel. Dans **Journaux** > Journaux personnalisés, vous verrez les résultats dans la table **AnomalousResourceAccessResult_CL** (ou la table portant votre nom personnalisé). Vous pouvez utiliser ces résultats pour améliorer vos expériences d’investigation et de repérage.

:::image type="content" source="./media/bring-your-own-ml/anomalous-resource-access-logs.png" alt-text="Journaux d’accès anormaux aux ressources":::

### <a name="build-custom-analytics-rule-with-ml-results"></a>Créer une règle d’analytique personnalisée avec des résultats de ML

Après avoir vérifié que les résultats de ML figurent dans la table des journaux personnalisés, lorsque vous êtes satisfait de la fidélité du scoring, vous pouvez créer une détection basée sur les résultats. Accédez à **Analytics** à partir du portail Azure Sentinel, et [créez une règle de détection](tutorial-detect-threats-custom.md). Vous trouverez ci-dessous un exemple de requête utilisée pour créer la détection.

:::image type="content" source="./media/bring-your-own-ml/create-byo-ml-analytics-rule.png" alt-text="Créer une règle d’analytique personnalisée pour les détections BYO-ML":::

### <a name="view-and-respond-to-incidents"></a>Afficher et traiter les incidents
Une fois que vous avez configuré la règle d’analytique basée sur les résultats de ML, si des résultats dépassent le seuil que vous avez défini dans la requête, un incident est généré et affiché dans la page **Incidents** sur Azure Sentinel. 

## <a name="next-steps"></a>Étapes suivantes

Ce document vous a montré comment utiliser la plateforme BYO-ML d’Azure Sentinel pour créer ou importer vos propres algorithmes d’apprentissage automatique afin d’analyser des données et de détecter des menaces.

- Pour accéder à des billets relatifs à l’apprentissage automatique ainsi qu’à de nombreuses autres rubriques pertinentes, consultez le [Blog Azure Sentinel](https://aka.ms/azuresentinelblog).