---
title: Correctif rapide pour les recommandations Advisor
description: Effectuer une correction en bloc à l’aide d’un correctif rapide dans Advisor
ms.topic: article
ms.date: 03/13/2020
ms.author: sagupt
ms.openlocfilehash: a9c86a7ae510d9657f64c71db2aa8c4e3e558f52
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90968587"
---
# <a name="quick-fix-remediation-for-advisor"></a>Application d’un correctif rapide pour Advisor
Un **correctif rapide** offre un moyen plus rapide et plus facile de corriger les recommandations dans plusieurs ressources. Il permet d’apporter des corrections en bloc aux ressources et d’optimiser vos abonnements plus rapidement grâce à une correction à grande échelle.
Cette fonctionnalité est disponible uniquement pour certaines recommandations dans le portail Azure.


## <a name="steps-to-use-quick-fix"></a>Utilisation d’un correctif rapide

1. Dans la liste des recommandations portant l’étiquette **Correctif rapide**, cliquez sur la recommandation.

   :::image type="content" source="./media/quick-fix-1.png" alt-text="{Capture d’écran d’Azure Advisor montrant des étiquettes Correctif rapide dans les recommandations.}":::
   
   *Les prix indiqués dans l’image sont fournis uniquement à titre d’exemple*

2. Dans la page Détails de la recommandation, vous verrez la liste des ressources pour lesquelles vous avez cette recommandation. Sélectionnez toutes les ressources que vous souhaitez corriger.

   :::image type="content" source="./media/quick-fix-2.png" alt-text="{Capture d’écran d’Azure Advisor montrant des étiquettes Correctif rapide dans les recommandations.}":::
   
   *Les prix indiqués dans l’image sont fournis uniquement à titre d’exemple*

3. Une fois que vous avez sélectionné les ressources, cliquez sur le bouton **Correctif rapide** pour effectuer une correction en bloc.

   > [!NOTE]
   > Certaines des ressources listées peuvent être désactivées, ce qui est le cas si vous n’avez pas les autorisations appropriées pour les modifier.
   
   > [!NOTE]
   > S’il y a d’autres implications, en plus des avantages mentionnés dans Advisor, vous en serez informé afin de prendre des décisions éclairées concernant les corrections.
   
4. Vous recevrez une notification une fois la correction terminée. Vous verrez une erreur s’afficher si des ressources ne sont pas corrigées. Ces ressources s’afficheront dans la liste des ressources dans mode sélectionné.  


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les recommandations d’Advisor, consultez :
* [Présentation du conseiller Azure](advisor-overview.md)
* [Prise en main d’Advisor](advisor-get-started.md)
* [Recommandations du conseiller en matière de coûts](advisor-cost-recommendations.md)
* [Recommandations du conseiller en matière de performances](advisor-performance-recommendations.md)
* [Recommandations du conseiller en matière de sécurité](advisor-security-recommendations.md)
* [Recommandations d’excellence opérationnelle Advisor](advisor-operational-excellence-recommendations.md)
* [API REST Advisor](/rest/api/advisor/)
