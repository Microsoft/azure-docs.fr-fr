---
title: Mettre à niveau vers Azure Defender – Azure Security Center
description: Ce guide de démarrage rapide explique comment effectuer une mise à niveau vers Azure Defender de Security Center pour renforcer la sécurité.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security-center
ms.devlang: na
ms.topic: quickstart
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/22/2020
ms.author: memildin
ms.openlocfilehash: bd8c5c31579ac22cd6c77dffd31103680e393f69
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96187168"
---
# <a name="quickstart-setting-up-azure-security-center"></a>Démarrage rapide : Configuration d’Azure Security Center

Azure Security Center propose des fonctionnalités unifiées de gestion de la sécurité et de protection contre les menaces sur l’ensemble des charges de travail cloud hybrides. Alors que les fonctionnalités gratuites offrent une sécurité limitée pour vos ressources Azure uniquement, l’activation d’Azure Defender étend ces fonctions aux ressources locales et à d’autres clouds. Azure Defender aide à rechercher et à corriger les failles de sécurité, applique des contrôles d’accès et d’application pour bloquer les activités malveillantes, détecte les menaces à l’aide de l’analytique et de l’analyse décisionnelle et répond rapidement en cas d’attaque. Vous pouvez essayer gratuitement Azure Defender. Pour en savoir plus, consultez la [page de tarification](https://azure.microsoft.com/pricing/details/security-center/).

Dans cet article, vous effectuez une mise à niveau vers Azure Defender pour renforcer la sécurité et installer l’agent Log Analytics sur vos machines afin de surveiller les menaces et les failles de sécurité.

## <a name="prerequisites"></a>Prérequis
Pour utiliser le Centre de sécurité, vous devez disposer d’un abonnement à Microsoft Azure. Si vous n’avez pas d’abonnement, vous pouvez vous inscrire pour avoir un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/).

Pour activer Azure Defender dans le cadre d’un abonnement, vous devez avoir le rôle de propriétaire de l’abonnement, de collaborateur de l’abonnement ou d’administrateur de la sécurité.


## <a name="enable-security-center-on-your-azure-subscription"></a>Activer Security Center sur votre abonnement Azure

1. Connectez-vous au [portail Azure](https://azure.microsoft.com/features/azure-portal/).

1. Dans le menu du portail, sélectionnez **Centre de sécurité**. 

    La page de présentation de Security Center s’ouvre.

    :::image type="content" source="./media/security-center-get-started/overview.png" alt-text="Tableau de bord de présentation de Security Center" lightbox="./media/security-center-get-started/overview.png":::

**Security Center - Vue d’ensemble** donne un aperçu unifié de l’état de sécurité des charges de travail cloud hybrides, ce qui permet de détecter et d’évaluer leur sécurité, ainsi que d’identifier et d’atténuer les risques. Security Center active automatiquement et gratuitement les abonnements Azure qui n’ont pas été intégrés par vous ni par aucun autre utilisateur de l’abonnement.

Pour afficher et filtrer la liste des abonnements, sélectionnez l’élément de menu **Abonnements**. Security Center ajuste l’affichage pour refléter la position de sécurité des abonnements sélectionnés. 

Quelques minutes après le premier lancement de Security Center peuvent s’afficher :

- des **recommandations** de méthodes visant à améliorer la sécurité de vos ressources connectées ;
- un inventaire de vos ressources en cours d’évaluation par Security Center, avec la posture de sécurité de chacune.

Pour tirer pleinement parti de Security Center, vous devez suivre les étapes ci-dessous afin d’activer Azure Defender et d’installer l’agent Log Analytics.

> [!TIP]
> Pour activer Security Center sur tous les abonnements au sein d’un groupe d’administration, consultez [Activer Security Center sur plusieurs abonnements Azure](onboard-management-group.md).

## <a name="enable-azure-defender"></a>Activer Azure Defender

Pour suivre les guides de démarrage rapide et les tutoriels Security Center, vous devez activer Azure Defender. Une version d’évaluation gratuite de 30 jours est disponible. Pour en savoir plus, consultez la [page de tarification](https://azure.microsoft.com/pricing/details/security-center/). 

1. Dans la barre latérale de Security Center, sélectionnez **Bien démarrer**.

    :::image type="content" source="./media/security-center-get-started/get-started-upgrade-tab.png" alt-text="Onglet Mise à niveau de la page Prise en main"::: 

    L’onglet **Mise à jour** répertorie les abonnements et les espaces de travail éligibles à l’intégration.

1. Dans la liste **Sélectionner les espaces de travail où activer Azure Defender**, sélectionnez les espaces de travail à mettre à niveau.
   - Si vous sélectionnez des abonnements et des espaces de travail qui ne sont pas éligibles pour la version d’évaluation, l’étape suivante les met à niveau et des frais seront facturés depuis lors.
   - Si vous sélectionnez un espace de travail éligible à un essai gratuit, la prochaine étape activera un essai.
1. Sélectionnez **Mettre à niveau** pour activer Azure Defender.

## <a name="enable-automatic-data-collection"></a>Activer la collecte de données automatique
Security Center collecte des données à partir de vos machines pour surveiller les menaces et les failles de sécurité. Les données sont collectées à l’aide de l’agent Log Analytics, qui lit divers journaux d’événements et configurations liées à la sécurité de la machine et copie les données dans votre espace de travail à des fins d’analyse. Par défaut, Security Center vous crée un nouvel espace de travail.

Quand le provisionnement automatique est activé, Security Center installe l’agent Log Analytics sur toutes les machines prises en charge et sur toutes les nouvelles machines créées. L’approvisionnement automatique est vivement conseillé.

Pour activer le provisionnement automatique de l’agent Log Analytics :

1. Dans le menu de Security Center, sélectionnez **Tarification et paramètres**.
1. Sélectionnez l’abonnement approprié.
1. Dans la page **Collecte des données**, définissez **Provisionnement automatique** sur **On** (activé).
1. Sélectionnez **Enregistrer**.

    :::image type="content" source="./media/security-center-enable-data-collection/enable-automatic-provisioning.png" alt-text="Activation du provisionnement automatique de l’agent Log Analytics":::

>[!TIP]
> Si un espace de travail doit être approvisionné, l’installation de l’agent peut prendre jusqu’à 25 minutes.

Grâce à l’agent déployé sur vos machines, Security Center peut fournir des recommandations supplémentaires sur l’état de mise à jour du système, les configurations de la sécurité du système d’exploitation et la protection des points de terminaison, et générer des alertes de sécurité supplémentaires.

>[!NOTE]
> La définition du provisionnement automatique sur **Off** (désactivé) ne supprime pas l’agent Log Analytics des machines virtuelles Azure sur lesquelles l’agent a déjà été provisionné. La désactivation de l’approvisionnement automatique limite la surveillance de la sécurité pour vos ressources.



## <a name="next-steps"></a>Étapes suivantes
Dans cette procédure de démarrage rapide, vous avez activé Azure Defender et provisionné l’agent Log Analytics pour profiter de fonctionnalités unifiées de gestion de la sécurité et de protection contre les menaces sur l’ensemble de vos charges de travail cloud hybrides. Pour en savoir plus sur Security Center, enchaînez avec le guide de démarrage rapide sur l’intégration d’ordinateurs Windows en local et dans d’autres clouds.

> [!div class="nextstepaction"]
> [Démarrage rapide : Intégrer des machines non-Azure](quickstart-onboard-machines.md)

Vous souhaitez optimiser et réduire vos coûts de cloud ?

> [!div class="nextstepaction"]
> [Démarrer l’analyse des coûts avec Cost Management](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)

<!--Image references-->
[2]: ./media/security-center-get-started/overview.png
[4]: ./media/security-center-get-started/get-started.png
[5]: ./media/security-center-get-started/pricing.png
[7]: ./media/security-center-get-started/security-alerts.png
[8]: ./media/security-center-get-started/recommendations.png
[9]: ./media/security-center-get-started/select-subscription.png