---
title: Alertes de sécurité concernant les environnements dans Azure DevTest Labs
description: Cet article explique comment afficher des alertes de sécurité pour un environnement dans DevTest Labs, et prendre les mesures nécessaires.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: c9f7cf9fe7ab0e3f573470228ee1962aa92ccaef
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91308688"
---
# <a name="security-alerts-for-environments-in-azure-devtest-labs"></a>Alertes de sécurité concernant les environnements dans Azure DevTest Labs
En tant qu’utilisateur de laboratoire, vous pouvez désormais afficher les alertes Azure Security Center pour vos environnements lab. Le Centre de sécurité collecte, analyse et intègre automatiquement les données de journaux provenant de vos ressources Azure, du réseau et des solutions partenaires connectées, telles que les solutions de protection des points de terminaison et des pare-feu, pour détecter les menaces réelles et réduire le nombre de faux positifs. Une liste hiérarchisée d’alertes de sécurité est affichée dans le Centre de sécurité, ainsi que les informations nécessaires pour trouver rapidement la cause d’une attaque et des recommandations sur la façon d’y remédier. [Apprenez-en davantage sur les alertes de sécurité dans Azure Security Center](../security-center//security-center-alerts-overview.md).  


## <a name="prerequisites"></a>Conditions préalables requises
Vous pouvez afficher des alertes de sécurité uniquement pour les environnements PaaS (Platform as a Service) qui sont déployés dans votre lab. Pour tester ou utiliser cette fonctionnalité, [déployez un environnement dans votre laboratoire](devtest-lab-create-environment-from-arm.md). 

## <a name="view-security-alerts-for-an-environment"></a>Voir les alertes de sécurité d’un environnement

1. Dans la page d’accueil de votre laboratoire, sélectionnez **Alertes de sécurité** dans le menu de gauche. Le nombre d’alertes de sécurité (de niveau élevé, moyen ou faible) doit s’afficher. Apprenez-en davantage sur [la façon dont les alertes sont classées](../security-center/security-center-alerts-overview.md#how-are-alerts-classified).

    ![Alertes de sécurité - Vue d’ensemble](./media/environment-security-alerts/security-alerts-overview-page.png)
2. Cliquez avec le bouton droit sur les trois points (...) de la dernière colonne, puis sélectionnez **Voir les alertes de sécurité**. 

    ![Capture d’écran montrant la page Alertes de sécurité avec l’option « Afficher les alertes de sécurité » sélectionnée.](./media/environment-security-alerts/view-security-alerts-menu.png)
    
3. Vous verrez des informations détaillées sur les alertes et les recommandations Advisor. Apprenez-en davantage sur la [gestion et la résolution des alertes de sécurité dans Azure Security Center](../security-center/security-center-managing-and-responding-alerts.md).

    ![Afficher les alertes de sécurité](./media/environment-security-alerts/advisor-recommendations.png)


## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les environnements, consultez les articles suivants :

- [Créer des environnements de plusieurs machines virtuelles et des ressources PaaS avec les modèles Azure Resource Manager](devtest-lab-create-environment-from-arm.md)
- [Configurer et utiliser des environnements publics](devtest-lab-configure-use-public-environments.md)
