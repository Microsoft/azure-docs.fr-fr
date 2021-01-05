---
title: Déboguer des travaux récurrents dans Azure Data Lake Analytics
description: Découvrez comment utiliser Azure Data Lake Tools pour Visual Studio pour le débogage local d’une tâche périodique.
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 05/20/2018
ms.openlocfilehash: b95d4b0a082e8eac191d610c9516e5ce72180b4c
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92219171"
---
# <a name="troubleshoot-an-abnormal-recurring-job"></a>Résoudre des problèmes liés à une tâche périodique anormale

Cet article explique comment utiliser [Azure Data Lake Tools pour Visual Studio](https://aka.ms/adltoolsvs) afin de résoudre les problèmes liés à des tâches périodiques. Pour en savoir plus sur le pipeline et les tâches périodiques, consultez le [blog Azure Data Lake et Azure HDInsight](/archive/blogs/azuredatalake/managing-pipeline-recurring-jobs-in-azure-data-lake-analytics-made-easy).

Les tâches périodiques partagent généralement la même logique de requête et des données d’entrée similaires. Supposons, par exemple, qu’une tâche périodique s’exécute chaque lundi matin à 8 h 00 pour compter les utilisateurs actifs de la semaine précédente. Les scripts de ces tâches partagent un modèle qui contient la logique de requête. Les entrées de ces tâches sont les données d’utilisation de la semaine précédente. Dans la mesure où ces tâches partagent la même logique de requête et une entrée similaire, leurs performances sont en général équivalentes et stables. Si l’une de vos tâches périodiques se met à avoir un comportement anormal, échoue ou ralentit beaucoup, vous pouvez :

- Afficher les rapports statistiques des précédentes exécutions de la tâche périodique pour savoir ce qu’il s’est passé.
- Comparer la tâche anormale à une tâche normale pour déterminer ce qui a changé.

**La vue des tâches associées** dans Azure Data Lake Tools pour Visual Studio vous permet d’accélérer la résolution des problèmes dans les deux cas.

## <a name="step-1-find-recurring-jobs-and-open-related-job-view"></a>Étape 1 : Rechercher les tâches périodiques et ouvrir la vue des tâches associées

Pour utiliser l’affichage Tâches associées afin de résoudre un problème lié à une tâche périodique, vous devez tout d’abord rechercher la tâche périodique dans Visual Studio, puis ouvrir l’affichage Tâches associées.

### <a name="case-1-you-have-the-url-for-the-recurring-job"></a>Cas n°1 : Vous disposez de l’URL de la tâche périodique

Dans **Outils** > **Data Lake** > **Affichage Tâches**, vous pouvez coller l’URL de la tâche pour ouvrir l’affichage Tâches dans Visual Studio. Sélectionnez **Afficher les tâches associées** pour ouvrir l’affichage Tâches associées.

![Lien Afficher les tâches associées dans Data Lake Analytics Tools](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/view-related-job.png)
 
### <a name="case-2-you-have-the-pipeline-for-the-recurring-job-but-not-the-url"></a>Cas n°2 : Vous disposez du pipeline pour la tâche périodique, mais pas de l’URL

Dans Visual Studio, vous pouvez ouvrir le navigateur de pipeline dans Explorateur de serveurs > votre compte Azure Data Lake Analytics > **Pipelines**. (Si vous ne trouvez pas ce nœud dans l’Explorateur de serveurs, [téléchargez la dernière version du plug-in](https://aka.ms/adltoolsvs).) 

![Sélectionner le nœud Pipelines](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/pipeline-browser.png)

Dans l’explorateur de pipelines, tous les pipelines du compte Data Lake Analytics apparaissent à gauche. Vous pouvez développer les pipelines pour rechercher toutes les tâches périodiques, puis sélectionner celle qui pose problème. L’affichage Tâches associées s’ouvre à droite.

![Sélectionner un pipeline et ouvrir l’affichage Tâches associées](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/recurring-job-view.png)

## <a name="step-2-analyze-a-statistics-report"></a>Étape 2 : Analyser un rapport statistique

Un résumé et un rapport statistique s’affichent en haut de l’affichage Tâches associées. C’est là que vous trouverez la cause racine du problème. 

1.  Dans le rapport, l’axe X affiche l’heure de soumission de la tâche. Utilisez-le pour trouver la tâche anormale.
2.  Utilisez le processus du diagramme suivant pour vérifier les statistiques et obtenir des informations sur le problème et sur les solutions possibles.

![Diagramme du processus de vérification des statistiques](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/recurring-job-metrics-debugging-flow.png)

## <a name="step-3-compare-the-abnormal-job-to-a-normal-job"></a>Étape 3 : Comparer la tâche anormale à une tâche normale

Vous trouverez toutes les tâches périodiques soumises dans la liste de tâches en bas de l’affichage Tâches associées. Pour obtenir plus d’informations ainsi que des solutions potentielles, cliquez avec le bouton droit sur la tâche anormale. Utilisez l’affichage Différences entre les tâches pour comparer la tâche anormale avec une tâche antérieure normale.

![Menu contextuel de comparaison de tâches](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/compare-job.png)

Prêtez attention aux grandes différences entre ces deux tâches. Elles sont probablement responsables des problèmes de performances. Pour pousser plus loin la vérification, suivez les étapes du diagramme suivant :

![Diagramme du processus de vérification des différences entre les tâches](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/recurring-job-diff-debugging-flow.png)

## <a name="next-steps"></a>Étapes suivantes

* [Résoudre des problèmes d’asymétrie des données](data-lake-analytics-data-lake-tools-data-skew-solutions.md)
* [Déboguer du code C# défini par l’utilisateur pour des tâches U-SQL ayant échoué](data-lake-analytics-debug-u-sql-jobs.md)