---
title: Intégrer Windows Virtual Desktop à Azure Advisor - Azure
description: Découvrez comment utiliser Azure Advisor dans votre déploiement Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 08/28/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 76b7f97b6020a3a0d5571a3a105d15f7d7893485
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89147238"
---
# <a name="use-azure-advisor-with-windows-virtual-desktop"></a>Utiliser Azure Advisor avec Windows Virtual Desktop

Azure Advisor peut aider les utilisateurs à résoudre eux-mêmes des problèmes courants sans avoir à ouvrir des cas de support. Les recommandations limitent le recours aux demandes d’aide, ce qui vous fait faire des économies de temps et d’argent.

Cet article explique comment configurer Azure Advisor dans votre déploiement Windows Virtual Desktop pour aider vos utilisateurs.

## <a name="what-is-azure-advisor"></a>Présentation d’Azure Advisor

Azure Advisor analyse vos configurations et vos données de télémétrie en vue de fournir des recommandations personnalisées pour résoudre les problèmes courants. Avec ces recommandations, vous pouvez optimiser vos ressources Azure en matière de fiabilité, de sécurité, d’excellence opérationnelle, de performances et de coût. Pour plus d’informations, consultez le [site web Azure Advisor](https://azure.microsoft.com/services/advisor/).

## <a name="how-to-start-using-azure-advisor"></a>Comment démarrer avec Azure Advisor

Tout ce dont vous avez besoin pour démarrer est un compte Azure sur le portail Azure. Tout d’abord, ouvrez le portail Azure sur <https://portal.azure.com/#home>, puis sélectionnez **Advisor** sous **Services Azure**, comme illustré ci-dessous. Vous pouvez également entrer « Azure Advisor » dans la barre de recherche du portail Azure.

> [!div class="mx-imgBorder"]
> ![Capture d’écran du portail Azure. L’utilisateur pointe le curseur de la souris sur le lien Azure Advisor, ce qui entraîne l’affichage d’un menu déroulant.](media/azure-advisor.png)

Quand vous ouvrez Azure Advisor, vous voyez cinq catégories :

- Coût
- Sécurité
- Fiabilité
- Excellence opérationnelle
- Performances

> [!div class="mx-imgBorder"]
> ![Capture d’écran d’Azure Advisor montrant les cinq menus pour chaque catégorie. Les cinq éléments affichés dans des zones distinctes sont Coût, Sécurité, Fiabilité, Excellence opérationnelle et Performances.](media/advisor-categories.png)

Quand vous sélectionnez une catégorie, vous accédez à la page des recommandations actives correspondantes. Cette page présente les recommandations qui vous sont faites par Azure Advisor, comme dans l’image suivante.

> [!div class="mx-imgBorder"]
> ![Capture d’écran de la liste des recommandations actives pour la catégorie Excellence opérationnelle. La liste contient sept recommandations avec des niveaux de priorité différents.](media/active-suggestions.png)

## <a name="additional-tips-for-azure-advisor"></a>Conseils supplémentaires pour Azure Advisor

- Veillez à consulter régulièrement vos recommandations, c’est-à-dire au moins une fois par semaine. Azure Advisor met à jour ses recommandations actives plusieurs fois par jour. En consultant les nouvelles recommandations, vous pouvez éviter la survenue de problèmes plus graves, car les recommandations vous aident à repérer et à résoudre des problèmes mineurs.

- Essayez toujours de résoudre les problèmes avec le niveau de priorité le plus élevé dans Azure Advisor. Les problèmes de priorité élevée sont signalés en rouge. Ne pas appliquer les recommandations de haute priorité peut provoquer d’autres problèmes plus tard.

- Si une recommandation vous semble moins importante, vous pouvez l’ignorer ou la reporter. Pour ignorer ou reporter une recommandation, changez l’état de l’élément dans la colonne **Action**.

- N’ignorez pas les recommandations tant que vous n’avez pas compris pourquoi elles s’affichent et que vous n’avez pas écarté tout risque d’impact négatif pour vos utilisateurs ou vous-même. Sélectionnez toujours **En savoir plus** pour prendre connaissance du problème. Si vous résolvez un problème en suivant les instructions fournies dans Azure Advisor, ce problème est automatiquement retiré de la liste. Il vaut mieux résoudre les problèmes plutôt que différer sans cesse leur traitement.

- Dès que vous rencontrez un problème dans Windows Virtual Desktop, commencez par consulter Azure Advisor. Azure Advisor vous donnera des instructions pour résoudre le problème ou, à défaut, il vous redirigera vers une ressource utile.

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment appliquer les recommandations, consultez [Comment appliquer des recommandations Azure Advisor](azure-advisor-recommendations.md).

Si vous avez des suggestions de nouvelles recommandations, postez-les sur notre [forum des utilisateurs Azure Advisor](https://windowsvirtualdesktop.uservoice.com/forums/930847-azure-advisor-recommendations).
