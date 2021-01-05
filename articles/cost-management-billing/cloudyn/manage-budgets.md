---
title: Gérer des budgets Cloudyn dans Azure
description: Cet article vous aide à créer des budgets rapidement et à commencer à les gérer dans Cloudyn.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.reviewer: vitavor
ms.custom: seodec18
ROBOTS: NOINDEX
ms.openlocfilehash: b7a26e7d94bbfea77bfc8121923377a086aef037
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690865"
---
# <a name="manage-azure-budgets-with-cloudyn"></a>Gérer les budgets Azure avec Cloudyn

La mise en place de budgets et d’alertes basées sur un budget permet d’améliorer la gouvernance et la transparence du cloud. Cet article vous aide à créer des budgets rapidement et à commencer à les gérer dans Cloudyn.

Si vous avez un compte Entreprise ou MSP, vous pouvez utiliser votre structure d’entité de coût hiérarchique pour attribuer des quotas de budget mensuel à différentes divisions, services ou autre entité de coût. Si vous avez un compte Premium, vous pouvez utiliser la fonctionnalité de gestion de budget, qui est ensuite appliquée à l’ensemble de vos dépenses cloud. Tous les budgets sont attribués manuellement.

Selon les budgets attribués, vous pouvez définir des alertes de seuil en fonction du pourcentage de votre budget qui est consommé et définir la gravité de chaque seuil.

Les rapports de budget indiquent le budget attribué. Les utilisateurs peuvent voir quand leurs dépenses sont au-dessus, au-dessous ou au même niveau que leur consommation au fil du temps. Quand vous sélectionnez **Afficher/masquer les champs** en haut d’un rapport de budget, vous pouvez voir le coût, le budget, le coût cumulé ou le budget total.

Azure Cost Management offre des fonctionnalités similaires à Cloudyn. Azure Cost Management est une solution native de gestion des coûts Azure. Il vous permet d’analyser les coûts, de créer et de gérer des budgets, d’exporter des données, et de consulter des recommandations d’optimisation et d’agir en conséquence pour dépenser moins. Pour plus d’informations sur les budgets dans Cost Management, consultez [Créer et gérer des budgets](../costs/tutorial-acm-create-budgets.md).

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="create-budgets"></a>Créer des budgets

Quand vous créez un budget, vous le définissez pour l’exercice fiscal et il s’applique à une entité spécifique.

Pour créer un budget et l’attribuer à une entité :

1. Accédez à **Coûts** &gt; **Gestion des coûts** &gt; **Budget**.
2. Dans la page Gestion budgétaire, sous **Entités**, sélectionnez l’entité où vous souhaitez créer le budget.
3. Pour l’année budgétaire, sélectionnez l’année où vous souhaitez créer le budget.
4. Pour chaque mois, définissez une valeur de budget. Quand vous avez terminé, cliquez sur **Enregistrer**.
Dans cet exemple, le budget mensuel pour juin 2018 est défini sur 135 000 $. Le budget total pour l’année s’élève à 1 615 000,00 $.
![Créer une page de budget pour définir un budget mensuel](./media/manage-budgets/set-budget.png)


Pour importer un fichier pour le budget annuel :

1. Sous **Actions**, sélectionnez **Exporter** afin de télécharger un modèle CSV vide sur lequel baser le budget.
2. Remplissez le fichier CSV avec les entrées de votre budget, puis enregistrez-le localement.
3. Sous **Actions**, sélectionnez **Importer**.
4. Sélectionnez votre fichier enregistré, puis cliquez sur **OK**.

Pour exporter votre budget terminé dans un fichier CSV, sous **Actions**, sélectionnez **Exporter** afin de télécharger le fichier.

## <a name="view-budget-in-reports"></a>Voir le budget dans des rapports

Une fois terminé, votre budget apparaît dans la plupart des rapports de coûts sous **Coûts** &gt; **Analyse du coût** et dans le rapport des coûts par rapport au budget dans le temps. Vous pouvez également planifier des rapports basés sur des seuils de budget en utilisant des **Actions**.

Voici un exemple du rapport Analyse du coût. Il montre le budget total et le coût par charge de travail et types d’utilisation depuis le début de l’année.

![Exemple du rapport Analyse du coût avec un budget](./media/manage-budgets/cost-analysis-budget-example.png)

Dans cet exemple, supposons que la date du jour est le 22 juin. Le coût pour juin 2018 est de 71 611,28 $ alors que le budget mensuel est de 135 000 $. Le coût est beaucoup moins élevé que le budget mensuel, car il reste encore huit jours de dépenses jusqu’à la fin du mois.

Une autre façon d’afficher le rapport consiste à regarder le coût cumulé par rapport au budget. Pour voir les coûts cumulés, sous **Afficher/masquer les champs**, sélectionnez **Coût cumulé** et **Budget total**. Voici un exemple montrant le coût cumulé depuis le début de l’année.

![Exemple de coût cumulé et budget total indiqué dans le rapport relatif au coût et du budget dans le temps](./media/manage-budgets/accumulated-budget.png)

Parfois, votre coût cumulé peut être appelé à dépasser votre budget. Vous pouvez plus facilement le voir si vous choisissez _graphique en courbes_ comme type d’affichage.

![Budget indiqué dans un graphique en courbes au sein du rapport relatif aux coûts mensuels](./media/manage-budgets/budget-line.png)

## <a name="create-budget-alerts-for-a-filter"></a>Créer des alertes de budget pour un filtre

Dans l’exemple précédent, vous pouvez voir que le coût cumulé s’approche du budget. Vous pouvez créer des alertes de budget automatiques afin d’être averti quand les dépenses s’approchent de votre budget ou le dépassent. Fondamentalement, l’alerte est un rapport planifié comportant un seuil. Les métriques d’un seuil d’alerte pour un budget sont les suivantes :

- Coût restant par rapport au budget : spécifier un seuil de valeur en devise
- Pourcentage du coût par rapport au budget : spécifier un seuil de valeur en pourcentage

Intéressons-nous à un exemple.

Dans le rapport des coûts par rapport au budget dans le temps, cliquez sur **Actions**, puis sélectionnez **Planifier le rapport**. Sous l’onglet Seuil, sélectionnez une métrique de seuil. Par exemple, **Pourcentage du coût par rapport au budget**. Sélectionnez un type d’alerte, puis entrez une valeur du budget en pourcentage. Si vous souhaitez être averti une seule fois, sélectionnez **Nombre d’alertes consécutives**, puis tapez _1_. Cliquez sur **Enregistrer**.

![Création d’une alerte de budget dans la zone Enregistrer ou planifier ce rapport](./media/manage-budgets/budget-alert.png)

## <a name="next-steps"></a>Étapes suivantes

- Si vous n’avez pas encore suivi le premier tutoriel de Cloudyn, consultez-le dans [Réviser l’utilisation et les coûts](tutorial-review-usage.md).
- Découvrez-en plus sur les [rapports disponibles dans Cloudyn](use-reports.md).
