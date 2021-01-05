---
title: Exporter ou supprimer les données d’un espace de travail
titleSuffix: Azure Machine Learning
description: Apprenez à exporter ou supprimer votre espace de travail avec Azure Machine Learning Studio, l’interface CLI, le kit SDK et les API REST authentifiées.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: jmartens
author: lobrien
ms.author: laobri
ms.date: 04/24/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 82e128ffdc215a084110a68170ac12accd8fca0e
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93308129"
---
# <a name="export-or-delete-your-machine-learning-service-workspace-data"></a>Exporter ou supprimer vos données d’espace de travail Machine Learning



Dans Azure Machine Learning, vous pouvez exporter ou supprimer les données de votre espace de travail à l’aide du kit SDK Python ou de l’interface graphique du portail. Cet article décrit ces deux options.

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-dsr-and-stp-note.md)]

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

## <a name="control-your-workspace-data"></a>Contrôler les données de votre espace de travail

Les données intégrées au produit, stockées par Azure Machine Learning, sont disponibles pour exportation et suppression. Vous pouvez les exporter et les supprimer à l’aide d’Azure Machine Learning Studio, de l’interface CLI et du kit SDK. Les données de télémétrie sont accessibles via le portail de confidentialité Azure. 

Dans Azure Machine Learning, les données personnelles se composent d’informations utilisateur dans des documents d’historique des exécutions. 

## <a name="delete-high-level-resources-using-the-portal"></a>Supprimer des ressources de haut niveau à l’aide du portail

Lorsque vous créez un espace de travail, Azure crée un certain nombre de ressources au sein du groupe de ressources :

- L’espace de travail lui-même
- Un compte de stockage
- Un registre de conteneurs
- Une instance Application Insights
- Un coffre de clés

Il est possible de supprimer ces ressources en les sélectionnant dans la liste et en choisissant **Supprimer**. 

:::image type="content" source="media/how-to-export-delete-data/delete-resource-group-resources.png" alt-text="Capture d’écran du portail, avec l’icône Supprimer mise en évidence":::

Les documents d’historique des exécutions, qui peuvent contenir des informations utilisateur personnelles, sont stockés dans le compte de stockage dans le stockage d’objets blob, dans les sous-dossiers de `/azureml`. Vous pouvez télécharger et supprimer ces données à partir du portail.

:::image type="content" source="media/how-to-export-delete-data/storage-account-folders.png" alt-text="Capture d’écran du répertoire azureml dans le compte de stockage, dans le portail":::

## <a name="export-and-delete-machine-learning-resources-using-azure-machine-learning-studio"></a>Exporter et supprimer des ressources Machine Learning à l’aide d’Azure Machine Learning Studio

Azure Machine Learning Studio fournit une vue unifiée de vos ressources Machine Learning, telles que les notebooks, les jeux de données, les modèles et les expériences. Azure Machine Learning Studio met l’accent sur la conservation d’un enregistrement de vos données et de vos expériences. Les ressources de calcul telles que les pipelines et autres peuvent être supprimées à l’aide du navigateur. Pour ces ressources, accédez à la ressource en question et choisissez **Supprimer**. 

Les jeux de données peuvent être désinscrits et les expériences peuvent être archivées, mais ces opérations ne suppriment pas les données. Pour supprimer entièrement les données, les jeux de données et les données d’exécution doivent être supprimés au niveau du stockage. La suppression au niveau du stockage s’effectue à l’aide du portail, comme décrit précédemment.

Vous pouvez télécharger des artefacts d’entraînement à partir d’exécutions expérimentales à l’aide de Studio. Choisissez l’ **expérience** et l’ **exécution** qui vous intéressent. Choisissez **Sortie + journaux** et accédez aux artefacts spécifiques que vous souhaitez télécharger. Choisissez **...** et **Télécharger**.

Vous pouvez télécharger un modèle inscrit en accédant au **modèle** souhaité et en choisissant **Télécharger**. 

:::image type="contents" source="media/how-to-export-delete-data/model-download.png" alt-text="Capture d’écran de la page de modèle Studio avec l’option Télécharger mise en évidence":::

## <a name="export-and-delete-resources-using-the-python-sdk"></a>Exporter et supprimer des ressources à l’aide du kit SDK Python

Vous pouvez télécharger les sorties d’une exécution particulière avec : 

```python
# Retrieved from Azure Machine Learning web UI
run_id = 'aaaaaaaa-bbbb-cccc-dddd-0123456789AB'
experiment = ws.experiments['my-experiment']
run = next(run for run in ex.get_runs() if run.id == run_id)
metrics_output_port = run.get_pipeline_output('metrics_output')
model_output_port = run.get_pipeline_output('model_output')

metrics_output_port.download('.', show_progress=True)
model_output_port.download('.', show_progress=True)
```

Les ressources Machine Learning suivantes peuvent être supprimées à l’aide du kit SDK Python : 

| Type | Appel de fonction | Notes | 
| --- | --- | --- |
| `Workspace` | [`delete`](/python/api/azureml-core/azureml.core.workspace.workspace?preserve-view=true&view=azure-ml-py#&preserve-view=truedelete-delete-dependent-resources-false--no-wait-false-) | Utilisez `delete-dependent-resources` pour effectuer une suppression en cascade |
| `Model` | [`delete`](/python/api/azureml-core/azureml.core.model%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=truedelete--) | | 
| `ComputeTarget` | [`delete`](/python/api/azureml-core/azureml.core.computetarget?preserve-view=true&view=azure-ml-py#&preserve-view=truedelete--) | |
| `WebService` | [`delete`](/python/api/azureml-core/azureml.core.webservice%28class%29?preserve-view=true&view=azure-ml-py) | |