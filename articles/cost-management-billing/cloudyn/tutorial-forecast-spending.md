---
title: Tutoriel - Prévoir les dépenses avec Cloudyn dans Azure
description: Dans ce didacticiel, vous allez apprendre à prévoir les dépenses à l’aide de l’historique d’utilisation et des données de dépenses.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: tutorial
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.custom: seodec18
ms.reviewer: benshy
ROBOTS: NOINDEX
ms.openlocfilehash: 70a5a4bf6981aa115c923559112d3533eebf3086
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92132174"
---
# <a name="tutorial-forecast-future-spending"></a>Tutoriel : Prévoir les dépenses futures

Cloudyn vous aide à prévoir les dépenses à venir via l’historique d’utilisation et les données relatives aux dépenses. Les rapports Cloudyn vous permettent d’afficher toutes les données de projection des coûts. Les exemples de ce didacticiel vous guident tout au long de l’étude des projections de coûts à l’aide de rapports. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Prévoir les dépenses futures

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’un compte Azure.
- Vous devez disposer d’une inscription à un essai gratuit ou d’un abonnement payant pour Cloudyn.

## <a name="forecast-future-spending"></a>Prévoir les dépenses futures

Cloudyn inclut des rapports de prévision des coûts pour vous aider à prévoir les dépenses au fil du temps, en fonction de votre utilisation. Ces rapports vous aident principalement à vous assurer que vos tendances de coûts n’excèdent pas les attentes de votre organisation. Vous utilisez un rapport du coût projeté pour le mois en cours (Current Month Projected Cost) et un rapport du coût projeté annuel (Annual Projected Cost). Ces deux rapports indiquent les dépenses futures projetées si votre utilisation reste relativement cohérente durant vos 30 derniers jours d’utilisation.

Le rapport du coût projeté pour le mois en cours affiche les coûts de vos services. Il prend en compte les coûts depuis le début du mois en cours et du mois précédent pour afficher le coût projeté. Dans le menu des rapports en haut du portail, cliquez sur **Costs** > **Projection and Budget** > **Current Month Projected Cost**. L’image suivante en montre un exemple.

![Informations d’exemples illustrées dans le rapport des coûts prévisionnels pour le mois en cours](./media/tutorial-forecast-spending/project-month01.png)

Dans l’exemple, vous pouvez voir les services qui ont le plus dépensé. Les coûts Azure sont inférieurs aux coûts AWS. Si vous souhaitez afficher les détails de la projection des coûts pour les machines virtuelles Azure, dans la liste **Filter**, sélectionnez **Azure/VM**.

![Exemple illustrant le coût prévisionnel des machines virtuelles Azure pour le mois en cours](./media/tutorial-forecast-spending/project-month02.png)

Suivez les mêmes étapes précédentes pour examiner les projections de coûts mensuels d’autres services qui vous intéressent.

Le rapport du coût projeté annuel (Annual Projected Cost) affiche le coût extrapolé de vos services pour les 12 prochains mois.

Dans le menu des rapports en haut du portail, cliquez sur **Costs** > **Projection and Budget** > **Annual Projected Cost**. L’image suivante en montre un exemple.

![Exemple illustrant le rapport du coût prévisionnel annuel](./media/tutorial-forecast-spending/project-annual01.png)

Dans l’exemple, vous pouvez voir les services qui ont le plus dépensé. Comme pour l’exemple mensuel, les coûts Azure ont été inférieurs aux coûts AWS. Si vous souhaitez afficher les détails de la projection des coûts pour les machines virtuelles Azure, dans la liste **Filter**, sélectionnez **Azure/VM**.

![Exemple illustrant le rapport du coût prévisionnel annuel des machines virtuelles](./media/tutorial-forecast-spending/project-annual02.png)

Dans l’image ci-dessus, le coût projeté annuel des machines virtuelles Azure est de 28 374 USD.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Prévoir les dépenses futures


Passez au didacticiel suivant pour découvrir comment gérer les coûts avec les rapports de facturation et d’allocation des coûts.

> [!div class="nextstepaction"]
> [Gérer les coûts grâce à des rapports de facturation et d’allocation des coûts](./tutorial-manage-costs.md)