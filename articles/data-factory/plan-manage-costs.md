---
title: Planifier et gérer les coûts d’Azure Data Factory
description: Cet article explique comment vous pouvez planifier et gérer les coûts pour Azure Data Factory.
documentationcenter: ''
author: shirleywangmsft
ms.author: shwang
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 05/14/2020
ms.openlocfilehash: 6271b83749c078b64c168a27f887ed7055e37d96
ms.sourcegitcommit: 66b0caafd915544f1c658c131eaf4695daba74c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97678902"
---
# <a name="plan-and-manage-costs-for-azure-data-factory"></a>Planifier et gérer les coûts d’Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Azure Data Factory est un service d’intégration de données élastique et serverless, conçu à l’échelle du cloud.  Ainsi, il n’existe aucun calcul de taille fixe à planifier pour la charge maximale ; vous spécifiez plutôt la quantité de ressources à allouer sur demande par opération, ce qui vous permet de concevoir les processus ETL d’une manière beaucoup plus scalable. De plus, Azure Data Factory est facturé sur la base d’un plan de consommation, ce qui signifie que vous payez uniquement ce que vous utilisez.

Cet article explique comment vous pouvez planifier et gérer les coûts d’Azure Data Factory.

*   Tout d’abord, au début du projet ETL, vous menez une preuve de concept et utilisez une combinaison de la consommation par pipeline avec la calculatrice de prix pour estimer les coûts.
*   Une fois que vous avez déployé vos pipelines en production, vous utilisez les fonctionnalités de gestion des coûts pour définir des budgets et superviser les coûts. Vous pouvez également examiner les coûts prévisionnels et déterminer les tendances des dépenses.
*   De plus, vous pouvez afficher les informations de consommation par pipeline et par activité pour comprendre quels pipelines et activités sont les plus coûteux, et identifier ceux qui sont susceptibles de réduire leurs coûts.

## <a name="estimate-costs-using-pipeline-and-activity-run-consumption-and-pricing-calculator"></a>Estimer les coûts à l’aide de la consommation d’exécutions d'activité et de pipeline et de la calculatrice de prix

Vous pouvez utiliser la [calculatrice de prix Azure Data Factory](https://azure.microsoft.com/pricing/calculator/?service=data-factory) pour estimer le coût d’exécution de votre charge de travail ETL dans Azure Data Factory.  Pour utiliser la calculatrice, vous devez entrer des détails, tels que le nombre d’exécutions d’activité, le nombre d’heures d’unités d’intégration de données, le type de calcul utilisé pour Data Flow, le nombre de cœurs, le nombre d’instances, la durée d’exécution, etc.

L’une des questions les plus fréquemment posées sur la calculatrice de prix se rapporte aux valeurs à utiliser comme entrées.  Pendant la phase de preuve de concept, vous pouvez effectuer des exécutions d’évaluation à l’aide d’exemples de jeux de données pour comprendre la consommation de divers compteurs Azure Data Factory.  Ensuite, en fonction de la consommation de l’exemple de jeu de données, vous pouvez prévoir la consommation du jeu de données complet et la planification de mise en œuvre.

> [!NOTE]
> Les prix mentionnés dans les exemples ci-dessous sont hypothétiques et ne sont pas destinés à indiquer la tarification réelle.

Supposons, par exemple, que vous deviez déplacer quotidiennement 1 To de données, de AWS S3 vers Azure Data Lake Gen2.  Vous pouvez effectuer la preuve de concept du déplacement de 100 Go de données pour mesurer le débit d’ingestion de données, et comprendre la consommation de facturation correspondante.

Voici un exemple d’exécution de l’activité de copie (votre consommation réelle variera en fonction de la forme de votre jeu de données spécifique, des vitesses de réseau, des limites de sortie sur le compte S3, des limites d’entrée sur ADLS Gen2, entre autres facteurs).

![Exécution de la copie de S3](media/plan-manage-costs/s3-copy-run-details.png)

En tirant parti de la [supervision de la consommation au niveau de l’exécution du pipeline](#monitor-consumption-at-pipeline-run-level), vous pouvez voir les quantités de consommation des compteurs de déplacement de données correspondantes :

![Consommation du pipeline de copie de S3](media/plan-manage-costs/s3-copy-pipeline-consumption.png)

Par conséquent, le nombre total d’heures d’unités d’intégration de données (DIU) nécessaires au déplacement de 1 To par jour, et pour le mois entier, est le suivant :

1,2667 (heures DIU) * (1 To / 100 Go) * 30 (jours dans un mois) = 380 heures DIU

Vous pouvez maintenant intégrer 30 exécutions d’activité et 380 heures DIU dans la calculatrice de prix Azure Data Factory pour connaître l’estimation de votre facture mensuelle :

![Calculatrice de prix de la copie de S3](media/plan-manage-costs/s3-copy-pricing-calculator.png)

## <a name="use-budgets-and-cost-alerts"></a>Utiliser des budgets et des alertes de coût

Vous pouvez créer des [budgets](../cost-management-billing/costs/tutorial-acm-create-budgets.md) pour gérer les coûts, et des alertes permettant d’avertir automatiquement les parties prenantes en cas d’anomalies de dépenses et de risques de dépenses excessives.  Les alertes sont basées sur les dépenses par rapport aux seuils de budget et de coût.  Lorsque vous créez un budget, vous pouvez le faire au niveau de l’abonnement, ou à une précision inférieure, en ajoutant des filtres supplémentaires, tels que l’ID de ressource et le nom du compteur.  Toutefois, vous ne pouvez pas créer de budget pour les pipelines individuels d’une fabrique.

## <a name="monitor-costs-at-factory-level"></a>Superviser les coûts au niveau de la fabrique

Lorsque vous commencez à utiliser Azure Data Factory, vous pouvez examiner les coûts engendrés dans le volet d’[analyse des coûts](../cost-management-billing/costs/quick-acm-cost-analysis.md) du portail Azure.

1. Pour voir l’[analyse des coûts](../cost-management-billing/costs/quick-acm-cost-analysis.md), ouvrez la fenêtre **Cost Management + facturation**, sélectionnez **Cost Management** dans le menu, puis sélectionnez **Ouvrir l’analyse des coûts**.
2. La vue par défaut affiche les coûts cumulés pour le mois en cours.  Vous pouvez basculer vers une autre période et une autre précision, par exemple quotidienne ou mensuelle.
3. Pour limiter les coûts à un seul service, comme Azure Data Factory, sélectionnez **Ajouter un filtre**, puis sélectionnez **Nom du service**.  Choisissez ensuite **Azure Data Factory v2** dans la liste.
4. Vous pouvez ajouter des filtres supplémentaires pour analyser les coûts d’une instance de fabrique particulière, et d’une précision de compteur Azure Data Factory spécifique.

   ![Analyse des coûts](media/plan-manage-costs/cost-analysis.png)

## <a name="monitor-consumption-at-pipeline-run-level"></a>Superviser la consommation au niveau de l’exécution du pipeline

En fonction des types d’activités que vous développez dans votre pipeline, de la quantité de données déplacées et transformées, et de la complexité de la transformation, l’exécution d’un pipeline fera tourner différents compteurs de facturation dans Azure Data Factory.

Vous pouvez afficher la quantité de consommation de différents compteurs pour des exécutions de pipeline individuels dans l’expérience utilisateur Azure Data Factory. Pour ouvrir l’expérience de surveillance, sélectionnez la vignette **Surveiller et gérer** dans le panneau de fabrique de données du [Portail Azure](https://portal.azure.com/). Si vous êtes déjà dans l’expérience utilisateur ADF, cliquez sur l’icône **Surveiller** dans la barre latérale gauche. Une liste d’exécutions de pipeline constitue l’affichage de la supervision par défaut.

En cliquant sur le bouton **Consommation** situé en regard du nom du pipeline, vous affichez une fenêtre contextuelle ; celle-ci vous renseigne sur la consommation de l’exécution de votre pipeline en regroupant toutes les activités au sein du pipeline.

![Consommation de l’exécution du pipeline](media/plan-manage-costs/pipeline-run-consumption.png)

![Détails de la consommation du pipeline](media/plan-manage-costs/pipeline-consumption-details.png)

L’affichage de la consommation se rapportant à l’exécution du pipeline indique la quantité consommée sur chaque compteur Azure Data Factory de l’exécution du pipeline spécifique, mais il n’affiche pas le prix réel facturé, car le montant qui vous est facturé dépend du type de compte Azure dont vous disposez, et du type de devise utilisé.  Pour accéder à la liste complète des types de comptes pris en charge, voir [Comprendre les données de Cost Management](../cost-management-billing/costs/understand-cost-mgt-data.md).

## <a name="monitor-consumption-at-activity-run-level"></a>Superviser la consommation au niveau de l’exécution de l’activité
Une fois la consommation agrégée au niveau de l’exécution du pipeline comprise, il reste des scénarios dans lesquels vous devez approfondir l’exploration et identifier l’activité la plus coûteuse au sein du pipeline.

Pour consulter la consommation au niveau de l’exécution de l’activité, accédez à l’IU **Créer et surveiller** de votre fabrique de données. À partir de l’onglet **Superviser**, dans lequel vous affichez la liste des exécutions de pipeline, cliquez sur le lien du **nom du pipeline** pour accéder à la liste des exécutions d’activités dans l’exécution de pipeline.  Cliquez sur le bouton **Sortie** en regard du nom de l’activité et recherchez la propriété **billableDuration** dans la sortie JSON :

Voici un exemple extrait d’une exécution de l’activité de copie :

![Sortie de la copie](media/plan-manage-costs/copy-output.png)

Et voici un exemple extrait d’une exécution de l’activité de mappage de Data Flow :

![Sortie de Data Flow](media/plan-manage-costs/dataflow-output.png)

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour en savoir plus sur l’application des tarifs dans Azure Data Factory :

- [Page de tarification d’Azure Data Factory](https://azure.microsoft.com/pricing/details/data-factory/ssis/)
- [Comprendre Azure Data Factory par des exemples](./pricing-concepts.md)
- [Calculatrice de prix d’Azure Data Factory](https://azure.microsoft.com/pricing/calculator/?service=data-factory)