---
title: 'Tutoriel : Utiliser Azure Security Center pour les machines virtuelles dans Azure'
description: Dans ce tutoriel, vous allez découvrir les fonctionnalités Azure Security Center pour mieux protéger et sécuriser vos machines virtuelles dans Azure.
author: cynthn
manager: gwallace
ms.service: virtual-machines
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 06/11/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: aec223ceca242e37e25f9e7288c8e0c0e2ea7535
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94594792"
---
# <a name="tutorial-use-azure-security-center-to-monitor-virtual-machines"></a>Tutoriel : Utiliser Azure Security Center pour superviser les machines virtuelles

Azure Security Center peut vous aider à acquérir une meilleure visibilité des pratiques de sécurité de vos ressources Azure. Azure Security Center assure une surveillance intégrée de la sécurité. Il peut détecter des menaces qui sans cela pourraient passer inaperçues. Ce didacticiel décrit Azure Security Center et comment effectuer les opérations suivantes :
 
> [!div class="checklist"]
> * Configurer la collecte de données
> * Définir des stratégies de sécurité
> * Afficher et résoudre des problèmes d’intégrité de configuration
> * Examiner les menaces détectées

## <a name="security-center-overview"></a>Vue d’ensemble de Security Center

Security Center identifie des problèmes potentiels de configuration des machines virtuelles et des menaces de sécurité ciblées. Il peut s’agir de machines virtuelles dépourvues de groupes de sécurité réseau, de disques non chiffrés et d’attaques RDP (Remote Desktop Protocol) par force brute. Les informations s’affichent sur le tableau de bord de Security Center dans des graphiques aisément lisibles.

Pour accéder au tableau de bord de Security Center, dans le menu du portail Azure, sélectionnez **Security Center**. Le tableau de bord montre l’état d’intégrité de la sécurité de votre environnement Azure, différentes recommandations courantes, ainsi que l’état actuel des alertes de menace. Vous pouvez développer chaque graphique général pour afficher plus de détails.

![Tableau de bord de Security Center](./media/tutorial-azure-security/asc-dash.png)

Security Center va au-delà de la découverte de données et fournit des recommandations concernant les problèmes qu’il détecte. Par exemple, si une machine virtuelle a été déployée sans groupe de sécurité réseau associé, Security Center affiche une recommandation ainsi que des étapes de correction que vous pouvez suivre. Vous bénéficiez d’une correction automatisée sans quitter le contexte de Security Center.  

![La capture d’écran montre la page Recommandations, qui comprend une description, une ressource, un état, ainsi que le niveau de gravité des recommandations.](./media/tutorial-azure-security/recommendations.png)

## <a name="set-up-data-collection"></a>Configurer la collecte de données

Avant d’obtenir une visibilité des configurations de sécurité des machines virtuelles, vous devez configurer la collecte de données de Security Center. Pour ce faire, vous devez activer la collecte de données qui installe automatiquement Microsoft Monitoring Agent sur toutes les machines virtuelles de votre abonnement.

1. Dans le tableau de bord de Security Center, cliquez sur **Stratégie de sécurité** puis sélectionnez votre abonnement. 
2. Pour **Collecte de données**, dans **Approvisionnement automatique**, sélectionnez **Activer**.
3. Pour **Configuration de l’espace de travail par défaut**, conservez le paramètre **Utiliser un ou des espace(s) de travail créé(s) par Security Center (par défaut)** .
4. Pour **Événements de sécurité**, conservez l’option par défaut **Commun**.
4. Cliquez sur **Enregistrer** dans la partie supérieure de la page. 

L’agent de collecte de données de Security Center est alors installé sur toutes les machines virtuelles, puis la collecte de données commence. 

## <a name="set-up-a-security-policy"></a>Configurer une stratégie de sécurité

Les stratégies de sécurité permettent de définir les éléments pour lesquels Security Center collecte des données et formule des recommandations. Vous pouvez appliquer des stratégies de sécurité différentes à différents ensembles de ressources Azure. Bien que par défaut les ressources Azure soient évaluées pour tous les éléments de la stratégie, vous pouvez désactiver des éléments individuels de la stratégie pour toutes les ressources ou pour un groupe de ressources Azure. Pour obtenir des informations détaillées sur les stratégies de sécurité de Security Center, consultez [Définir des stratégies de sécurité dans Azure Security Center](../security-center/tutorial-security-policy.md). 

Pour configurer une stratégie de sécurité pour l’intégralité d’un abonnement :

1. Dans le tableau de bord de Security Center, cliquez sur **Stratégie de sécurité**, puis sélectionnez votre abonnement.
2. Dans le panneau **Stratégie de sécurité**, sélectionnez **Stratégie de sécurité**. 
3. Dans le panneau **Stratégie de sécurité – Stratégie de sécurité**, activez ou désactivez les éléments de stratégie que vous souhaitez appliquer à l’abonnement.
4. Lorsque vous avez terminé de sélectionner vos paramètres, sélectionnez **Enregistrer** dans la partie supérieure du panneau. 

![Stratégie unique](./media/tutorial-azure-security/unique-policy.png)

## <a name="view-vm-configuration-health"></a>Afficher l’état de configuration des machines virtuelles

Après que vous avez activé la collecte de données et défini une stratégie de sécurité, Security Center commence à fournir des alertes et des recommandations. À mesure que des machines virtuelles sont déployées, l’agent de collecte de données est installé. Security Center reçoit ensuite des données relatives aux nouvelles machines virtuelles. Pour des informations détaillées sur l’intégrité de la configuration des machines virtuelles, consultez [Protéger vos machines virtuelles dans Security Center](../security-center/asset-inventory.md). 

À mesure que des données sont collectées, les informations sur l’état d’intégrité des ressources de chaque machine virtuelle et des ressources Azure associées sont agrégées. Les informations sont affichées dans un graphique aisément lisible. 

Pour afficher l’intégrité des ressources :

1.  Dans le tableau de bord de Security Center, sous **Prévention**, sélectionnez **Calcul**. 
2.  Dans le panneau **Calcul**, sélectionnez **Machines virtuelles et ordinateurs**. Cette vue fournit un récapitulatif de l’état de la configuration de toutes vos machines virtuelles.

![Calculer l’état d’intégrité](./media/tutorial-azure-security/compute-health.png)

Pour voir toutes les recommandations relatives à une machine virtuelle, sélectionnez celle-ci. 

## <a name="remediate-configuration-issues"></a>Corriger les problèmes de configuration

Quand Security Center commence à recevoir des données de configuration, des recommandations sont formulées en fonction de la stratégie de sécurité configurée. Par exemple, si une machine virtuelle a été configurée sans groupe de sécurité réseau associé, une recommandation préconise d’en créer un. 

Pour afficher la liste de toutes les recommandations : 

1. Dans le tableau de bord de Security Center, sélectionnez **Recommandations**.
2. Sélectionnez une recommandation spécifique. La liste de toutes les ressources auxquelles la recommandation s’applique s’affiche.
3. Pour appliquer une recommandation, sélectionnez la ressource. 
4. Suivez les instructions pour les étapes de correction. 

Dans de nombreux cas, Security Center propose des mesures que vous pouvez prendre pour suivre une recommandation sans quitter Security Center. Dans l’exemple suivant, Security Center détecte un groupe de sécurité réseau qui a une règle de trafic entrant non restreinte. Dans la page de recommandation, vous pouvez sélectionner le bouton **Modifier les règles de trafic entrant**. L’interface utilisateur nécessaire pour modifier la règle apparaît. 

![Recommandations](./media/tutorial-azure-security/remediation.png)

Les recommandations sont marquées comme étant résolues à mesure qu’elles sont appliquées. 

## <a name="view-detected-threats"></a>Consulter les menaces détectées

En plus des recommandations concernant la configuration des ressources, Security Center affiche des alertes de détection de menaces. La fonctionnalité d’alertes de sécurité agrège les données collectées à partir de chaque machine virtuelle, des journaux d’activité de réseau Azure et des solutions partenaires connectées pour détecter les menaces de sécurité au niveau des ressources Azure. Pour obtenir des informations détaillées sur les fonctionnalités de détection des menaces de Security Center, consultez [Comment Security Center détecte-t-il les menaces ?](../security-center/security-center-alerts-overview.md#detect-threats).

La fonctionnalité d’alertes de sécurité nécessite que le niveau tarifaire de Security Center soit porté de *Gratuit* à *Standard*. Une **version d’essai gratuit** est disponible quand vous passez à ce niveau tarifaire supérieur. 

Pour changer de niveau tarifaire :  

1. Dans le tableau de bord de Security Center, cliquez sur **Stratégie de sécurité** puis sélectionnez votre abonnement.
2. Sélectionnez **Niveau tarifaire**.
3. Sélectionnez **Standard**, puis cliquez sur **Enregistrer** dans la partie supérieure du panneau.


Après que vous avez changé le niveau tarifaire, le graphique des alertes de sécurité commence à se remplir à mesure que des menaces de sécurité sont détectées.

![Alertes de sécurité](./media/tutorial-azure-security/security-alerts.png)

Sélectionnez une alerte pour afficher des informations la concernant. Par exemple, vous pouvez voir une description de la menace, l’heure de la détection, toutes les tentatives de la menace et la correction recommandée. Dans l’exemple suivant, une attaque RDP par force brute a été détectée, avec 294 tentatives RDP qui ont échoué. Une solution recommandée est fournie.

![Attaque RDP](./media/tutorial-azure-security/rdp-attack.png)

## <a name="next-steps"></a>Étapes suivantes
Ce didacticiel vous a montré comment configurer Azure Security Center, puis examiner les machines virtuelles dans Azure Security Center. Vous avez appris à :

> [!div class="checklist"]
> * Configurer la collecte de données
> * Définir des stratégies de sécurité
> * Afficher et résoudre des problèmes d’intégrité de configuration
> * Examiner les menaces détectées

Passez au didacticiel suivant pour en savoir plus sur la création d’un pipeline d’intégration continue/de livraison continue avec Jenkins, GitHub et Docker.

> [!div class="nextstepaction"]
> [Créer une infrastructure d’intégration continue/de livraison continue avec Jenkins, GitHub et Docker](/azure/developer/jenkins/pipeline-with-github-and-docker)