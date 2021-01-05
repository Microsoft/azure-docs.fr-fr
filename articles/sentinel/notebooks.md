---
title: Utiliser des notebooks avec Azure Sentinel pour la chasse sécuritaire
description: Cet article décrit comment utiliser des notebooks avec les fonctionnalités de recherche Azure Sentinel.
services: sentinel
author: yelevin
ms.author: yelevin
ms.assetid: 1721d0da-c91e-4c96-82de-5c7458df566b
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: conceptual
ms.custom: mvc
ms.date: 09/06/2020
ms.openlocfilehash: 43d7a697b3cb013a73a0b14db8ec1758244ae3b9
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97092188"
---
# <a name="use-jupyter-notebook-to-hunt-for-security-threats"></a>Utiliser Jupyter Notebook pour rechercher des menaces de sécurité

Azure Sentinel repose sur le magasin de données. Il associe des capacités d’interrogation hautes performances, un schéma dynamique et s’adapte aux volumes de données importants. Le portail Azure et tous les outils Azure Sentinel utilisent une API commune pour accéder à ce magasin de données. La même API est également disponible pour les outils externes tels que les notebooks [Jupyter](https://jupyter.org/) et Python. Bien que de nombreuses tâches courantes puissent être effectuées dans le portail, Jupyter étend l’étendue de ce que vous pouvez faire avec ces données. Il combine une programmabilité totale à une vaste collection de bibliothèques pour l’apprentissage automatique, la visualisation et l’analyse des données. Ces attributs transforment Jupyter en un outil essentiel pour les scénarios de recherche et d’enquête relatifs à la sécurité.

![exemple de notebook](./media/notebooks/sentinel-notebooks-map.png)

Nous avons intégré l’expérience de Jupyter dans le portail Azure , pour que vous puissiez créer et exécuter facilement des notebooks pour analyser vos données. La bibliothèque *Kqlmagic* fournit le ciment qui vous permet d’extraire des requêtes d’Azure Sentinel et de les exécuter directement à l’intérieur d’un notebook. Les requêtes utilisent le [langage de requête Kusto](https://kusto.azurewebsites.net/docs/kusto/query/index.html). Plusieurs notebooks, développés par certains des analystes de sécurité de Microsoft, sont intégrés à Azure Sentinel. Quelques-uns d’entre eux sont conçus pour un scénario spécifique et peuvent être utilisés en tant que tel. D’autres sont des exemples pour illustrer les techniques et les fonctionnalités que vous pouvez copier ou adapter pour les utiliser dans vos propres notebooks. D’autres blocs-notes peuvent également être importés à partir du GitHub de la communauté Azure Sentinel.

L’expérience Jupyter intégrée utilise [Azure Notebooks](https://notebooks.azure.com/) pour stocker, partager et exécuter des notebooks. Vous pouvez également exécuter ces notebooks localement (si vous avez un environnement Python et Jupyter sur votre ordinateur) ou dans d’autres environnements JupterHub tels qu’Azure Databricks.

Notebooks a deux composants :

- l’interface basée sur un navigateur où vous entrez et exécutez des requêtes et du code, et où les résultats de l’exécution sont affichés.
- Un *noyau* qui est responsable de l’analyse et de l’exécution du code.

Le noyau du bloc-notes Azure Sentinel s’exécute sur une machine virtuelle Azure. Plusieurs options de licence existent pour tirer parti de machines virtuelles plus puissantes si vos blocs-notes incluent des modèles d’apprentissage automatique complexes.

Les notebooks Azure Sentinel utilisent de nombreuses bibliothèques Python populaires telles que pandas, matplotlib, bokeh et bien d’autres. Vous avez le choix entre un grand nombre d’autres packages Python, couvrant des domaines tels que ceux énoncés ci-dessous :

- visualisations et graphiques
- traitement de données et analyse
- statistiques et calculs numériques
- Machine Learning et Deep Learning

Nous avons également publié des outils de sécurité open source Jupyter dans un package nommé [msticpy](https://github.com/Microsoft/msticpy/). Ce package est utilisé dans la plupart des notebooks inclus. Les outils de msticpy sont conçus spécialement pour faciliter la création de notebooks pour la recherche et l’investigation, et nous travaillons activement pour vous proposer de nouvelles fonctionnalités et améliorations.

Le [référentiel GitHub de la communauté Azure Sentinel](https://github.com/Azure/Azure-Sentinel) est l’emplacement de tous les futurs notebooks Azure Sentinel créés par Microsoft ou fournis par la communauté.

Pour utiliser les blocs-notes, vous devez commencer par créer un espace de travail Azure Machine Learning (ML).

## <a name="create-an-azure-ml-workspace"></a>Création d’un espace de travail Azure ML

1. À partir du portail Azure, accédez à **Azure Sentinel** > **Gestion des menaces** > **Blocs-notes**, puis sélectionnez **Lancer un bloc-notes**.

    > [!div class="mx-imgBorder"]
    > ![lancer un bloc-notes pour démarrer l’espace de travail azure ml](./media/notebooks/sentinel-notebooks-launch.png)

1. Sous **Espace de travail AzureML**, sélectionnez **Créer**.

    > [!div class="mx-imgBorder"]
    > ![créer un espace de travail](./media/notebooks/sentinel-notebooks-azureml-create.png)

1. Dans la page **Machine Learning**, fournissez les informations suivantes, puis sélectionnez **Vérifier + créer**.

    |Champ|Description|
    |--|--|
    |Abonnement|Sélectionnez l’abonnement Azure que vous souhaitez utiliser.|
    |Resource group|Utilisez un groupe de ressources existant dans votre abonnement, ou entrez un nom pour créer un groupe de ressources. Un groupe de ressources contient les ressources associées d’une solution Azure. Dans cet exemple, nous utilisons **AzureMLRG**.|
    |Nom de l’espace de travail|Entrez un nom unique qui identifie votre espace de travail. Dans cet exemple, nous utilisons **testworkspace1**. Dans le groupe de ressources, les noms doivent être uniques. Utilisez un nom dont il est facile de se rappeler et que vous pouvez facilement différencier des autres espaces de travail.|
    |Region|Sélectionnez l’emplacement le plus proche de vos utilisateurs et des ressources de données pour créer votre espace de travail.|
    |Édition de l’espace de travail|Dans cet exemple, sélectionnez **De base** comme type d’espace de travail. Le type d’espace de travail (De base et Entreprise) détermine les fonctionnalités auxquelles vous aurez accès et la tarification.|

    > [!div class="mx-imgBorder"]
    > ![fournir des détails d’espace de travail](./media/notebooks/sentinel-notebooks-azureml-basics.png)

1. Passez en revue les informations, vérifiez qu’elles sont correctes, puis sélectionnez **Créer** pour démarrer le déploiement de votre espace de travail.

    > [!div class="mx-imgBorder"]
    > ![examiner les détails de l’espace de travail](./media/notebooks/sentinel-notebooks-azureml-review.png)

    La création de votre espace de travail dans le cloud peut prendre plusieurs minutes au cours desquelles la page **Vue d’ensemble** affiche l’état actuel du déploiement.

    > [!div class="mx-imgBorder"]
    > ![Déploiement d’espace de travail](./media/notebooks/sentinel-notebooks-azureml-deploy.png)

Une fois votre déploiement terminé, vous pouvez lancer des blocs-notes dans votre nouvel espace de travail Azure ML.

> [!div class="mx-imgBorder"]
> ![déploiement d’espace de travail réussi](./media/notebooks/sentinel-notebooks-azureml-complete.png)

## <a name="launch-a-notebook-using-your-azure-ml-workspace"></a>Lancer un bloc-notes à l’aide de votre espace de travail Azure ML

1. À partir du portail Azure, accédez à **Azure Sentinel** > **Gestion des menaces** > **Notebooks**, où vous pouvez voir les notebooks fournis par Azure Sentinel.

    > [!TIP]
    > Sélectionnez **Guides & commentaires** pour ouvrir un volet contenant de l’aide et des conseils supplémentaires sur les blocs-notes.
    > ![afficher des repères de bloc-notes](./media/notebooks/sentinel-azure-notebooks-guides.png)

1. Sélectionnez des blocs-notes individuels pour afficher leurs descriptions, les types de données requis et les sources de données.

    > [!div class="mx-imgBorder"]
    > ![afficher les détails d’un bloc-notes](./media/notebooks/sentinel-azure-notebooks-view.png)

1. Sélectionnez le bloc-notes que vous souhaitez utiliser, puis choisissez **Lancer le bloc-notes** afin de cloner et de configurer le bloc-notes dans un nouveau projet Azure Notebooks qui se connecte à votre espace de travail Azure Sentinel. Une fois le processus terminé, le notebook s’ouvre dans Azure Notebooks pour que vous l’exécutiez.

    > [!div class="mx-imgBorder"]
    > ![sélectionner un bloc-notes](./media/notebooks/sentinel-azure-notebooks-select.png)

1. Sous espace de travail AzureML, sélectionnez votre espace de travail Azure ML, puis **Lancer**.

    > [!div class="mx-imgBorder"]
    > ![lancer le notebook](./media/notebooks/sentinel-azure-notebooks-launch.png)

1. Sélectionnez une instance de calcul. Si vous n’avez pas d’instance de calcul, procédez comme suit :
    1. Sélectionnez le signe plus (+) pour démarrer l’Assistant **Nouvelle instance de calcul**.

        > [!div class="mx-imgBorder"]
        > ![démarrer l’assistant instance de calcul](./media/notebooks/sentinel-azure-notebooks-compute-wizard.png)

    1. Sur la page **Nouvelle instance de calcul**, fournissez les informations requises, puis sélectionnez **Créer**.

        > [!div class="mx-imgBorder"]
        > ![créer une instance de calcul](./media/notebooks/sentinel-azure-notebooks-compute-create.png)

1. Une fois votre serveur de bloc-notes créé, dans chaque cellule, sélectionnez l’icône exécuter pour exécuter le code dans les blocs-notes.

    > [!div class="mx-imgBorder"]
    > ![exécuter un bloc-notes](./media/notebooks/sentinel-azure-notebooks-run.png)

Recommandations :

- Pour une présentation rapide de l’interrogation des données dans Azure Sentinel, consultez le guide [Prise en main des blocs-notes Azure ML et d’Azure Sentinel](https://github.com/Azure/Azure-Sentinel-Notebooks/blob/master/A%20Getting%20Started%20Guide%20For%20Azure%20Sentinel%20ML%20Notebooks.ipynb).

- Vous trouverez des exemples supplémentaires de blocs-notes dans le sous-dossier GitHub [**Sample-Notebooks**](https://github.com/Azure/Azure-Sentinel-Notebooks/tree/master/Sample-Notebooks). Les exemples de notebooks ont été enregistrés avec des données pour que vous puissiez voir plus facilement le résultat prévu. Nous vous recommandons de les afficher dans [nbviewer](https://nbviewer.jupyter.org/).

- Le sous-dossier GitHub [**HowTos**](https://github.com/Azure/Azure-Sentinel-Notebooks/tree/master/HowTos) contient des blocs-notes décrivant, par exemple : la configuration de votre version Python par défaut, la configuration d’une DSVM, la création de signets Azure Sentinel dans un notebook et bien d’autres sujets.

Les notebooks fournis sont conçus en tant qu’outils utiles, illustrations et exemples de code que vous pouvez utiliser dans le développement de vos propres notebooks.

Nous apprécions les commentaires, qu’il s’agisse de suggestions, de demandes de fonctionnalités, de contribution aux notebooks, de signalements de bogues ou d’améliorations et d’ajouts aux notebooks existants. Rendez-vous sur le [GitHub de la communauté Azure Sentinel](https://github.com/Azure/Azure-Sentinel) pour créer un problème ou dupliquer et charger une contribution.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à bien démarrer avec les notebooks Jupyter dans Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :

- [Hunt for threats with in Azure Sentinel Preview](hunting.md) (Rechercher des menaces dans Azure Sentinel en préversion)
- [Keep track of data during hunting](bookmarks.md) (Suivre des données pendant la recherche)
