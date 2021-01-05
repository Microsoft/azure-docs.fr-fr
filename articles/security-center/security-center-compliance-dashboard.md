---
title: 'Tutoriel : Vérifications de conformité réglementaire - Azure Security Center'
description: 'Tutoriel : Découvrez comment améliorer la conformité aux réglementations à l’aide d’Azure Security Center.'
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 5f50c4dc-ea42-418d-9ea8-158ffeb93706
ms.service: security-center
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/11/2020
ms.author: memildin
ms.openlocfilehash: bbc36dbb2a17d379d31a9a235898500aea36247d
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96533908"
---
# <a name="tutorial-improve-your-regulatory-compliance"></a>Tutoriel : Améliorer votre conformité aux normes

Le **tableau de bord Conformité avec la réglementation** d’Azure Security Center simplifie le processus visant à respecter les exigences de conformité aux règlementations. Dans le tableau de bord, Security Center fournit des insights sur votre niveau de conformité sur la base d’évaluations continues de votre environnement Azure. Security Center analyse les facteurs de risque dans votre environnement de cloud hybride conformément aux bonnes pratiques en matière de sécurité. Ces évaluations sont comparées à des contrôles de conformité provenant d’un ensemble de normes prises en charge. Le tableau de bord de conformité réglementaire indique l’état de toutes les évaluations au sein de votre environnement dans le contexte d’une norme ou d’une réglementation particulières. Si vous suivez les recommandations et réduisez les facteurs de risque de votre environnement, votre niveau de conformité s’améliore.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Évaluer votre conformité aux réglementations à l’aide du tableau de bord de conformité réglementaire
> * Améliorer votre niveau de conformité en suivant les recommandations

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour parcourir les fonctionnalités couvertes dans ce tutoriel :

- [Azure Defender](azure-defender.md) doit être activé. Vous pouvez essayer gratuitement Azure Defender pendant 30 jours.
- Vous devez être connecté avec un compte qui dispose d’un accès de lecteur aux données de conformité de la stratégie (**lecteur Sécurité** est insuffisant). Le rôle **Lecteur général** pour l’abonnement fonctionnera. Au minimum, les rôles **Contributeur de stratégie de ressource** et **Administration de sécurité** doivent vous être affectés.

##  <a name="assess-your-regulatory-compliance"></a>Évaluer votre conformité aux réglementations

Security Center évalue en permanence la configuration de vos ressources pour identifier les vulnérabilités et les problèmes de sécurité. Ces évaluations sont présentées comme des recommandations visant à améliorer votre hygiène en matière de sécurité. Dans le tableau de bord de conformité réglementaire, vous pouvez voir un ensemble de normes de conformité et toutes leurs exigences, celles prises en charge étant comparées aux évaluations de sécurité applicables. Vous pouvez ainsi examiner votre niveau de conformité par rapport à la norme, en fonction de l’état de ces évaluations.

La vue du tableau de bord Conformité avec la réglementation peut vous aider à prendre conscience d’un manque de conformité à une norme ou à une réglementation qui est importante pour vous. Cette vue ciblée vous permet également de faire le suivi de votre score de conformité au fil du temps dans les environnements cloud et hybrides dynamiques.

>[!NOTE]
> Par défaut, Security Center prend en charge les normes réglementaires suivantes : Azure CIS, PCI DSS 3.2, ISO 27001 et SOC TSP. 
>
> La fonctionnalité [packages de conformité dynamiques (préversion)](update-regulatory-compliance-packages.md) permet de mettre à niveau les normes présentées dans votre tableau de bord de conformité réglementaire vers les nouveaux packages *dynamiques* Vous pouvez également utiliser la même fonctionnalité en préversion pour ajouter de nouveaux packages de conformité et surveiller votre conformité à des normes supplémentaires. 

1. Dans le menu de Security Center, sélectionnez **Conformité réglementaire**. <br>
Un tableau de bord apparaît en haut de l’écran avec une vue d’ensemble de votre état de conformité et l’ensemble des réglementations de conformité prises en charge. Vous pouvez voir votre score de conformité global et le nombre d’évaluations ayant réussi ou échoué pour chaque norme.

    :::image type="content" source="./media/security-center-compliance-dashboard/compliance-dashboard.png" alt-text="Tableau de bord de conformité réglementaire":::

1. Sélectionnez un onglet correspondant à une norme de conformité qui vous intéresse (1). Vous verrez sur quels abonnements la norme est appliquée (2) et la liste de tous les contrôles relatifs à cette norme (3). Pour les contrôles applicables, vous pouvez voir les détails des évaluations ayant réussi ou échoué associées à ce contrôle (4), ainsi que le nombre de ressources affectées (5). Certains contrôles sont grisés. Aucune évaluation Security Center n’est associée à ces contrôles. Vérifiez les exigences de ces contrôles et évaluez-les vous-même dans votre environnement. Certaines d’entre elles peuvent être liés au processus et ne pas être d’ordre technique.

    :::image type="content" source="./media/security-center-compliance-dashboard/compliance-drilldown.png" alt-text="Exploration des détails de la conformité à une norme spécifique":::

1. Pour générer et télécharger un rapport PDF résumant votre état de compatibilité actuel pour une norme particulière, cliquez sur **Télécharger un rapport**.

    Le rapport fournit un résumé détaillé de votre état de conformité pour la norme sélectionnée en fonction des données des évaluations de Security Center, et il est organisé selon les contrôles de cette norme particulière. Le rapport peut être partagé avec les parties prenantes concernées et servir de preuve aux auditeurs internes et externes.

    :::image type="content" source="./media/security-center-compliance-dashboard/download-report.png" alt-text="Télécharger le rapport de conformité":::

## <a name="improve-your-compliance-posture"></a>Améliorer votre niveau de conformité

Compte tenu des informations figurant dans le tableau de bord de conformité réglementaire, vous pouvez améliorer votre niveau de conformité en suivant les recommandations directement dans le tableau de bord.

1.  Cliquez sur l’une des évaluations ayant échoué dans le tableau de bord pour voir les détails de cette recommandation. Chaque recommandation comprend un ensemble d’étapes de correction que vous devez suivre pour résoudre le problème.

1.  Vous pouvez sélectionner une ressource particulière pour voir plus de détails et suivre la recommandation associée à cette ressource. <br>Par exemple, dans la **norme Azure CIS 1.1.0 (nouveau)** , vous pouvez sélectionner la recommandation **Le chiffrement de disque doit être appliqué sur les machines virtuelles**.

    :::image type="content" source="./media/security-center-compliance-dashboard/sample-recommendation.png" alt-text="La sélection d’une recommandation concernant une norme vous amène directement à la page des détails de la recommandation":::

1. Dans cet exemple, quand vous sélectionnez **Entreprendre une action** dans la page des détails de la recommandation, vous arrivez dans les pages relatives aux machines virtuelles Azure du portail Azure, où vous pouvez ouvrir l’onglet **Sécurité** et activer le chiffrement :

    :::image type="content" source="./media/security-center-compliance-dashboard/encrypting-vm-disks.png" alt-text="Le bouton Entreprendre une action de la page des détails de la recommandation vous amène aux options de correction":::

    Pour plus d’informations sur la façon d’appliquer des recommandations, consultez [Implémentation des recommandations de sécurité dans Azure Security Center](security-center-recommendations.md).

1.  Quand vous prenez des mesures pour résoudre les recommandations, vous pouvez constater une amélioration de votre score de conformité dans le rapport du tableau de bord de conformité.

    > [!NOTE]
    > Les évaluations s’exécutent toutes les 12 heures environ. L’impact sur vos données de conformité n’est donc visible qu’après l’exécution suivante de l’évaluation correspondante.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à utiliser le tableau de bord de conformité réglementaire de Security Center pour effectuer les tâches suivantes :

-   Afficher et superviser votre niveau de conformité par rapport à des normes et à des réglementations qui sont importantes pour vous.
-   Améliorer votre état de conformité en suivant les recommandations appropriées et en augmentant le score de conformité.

Le tableau de bord de conformité réglementaire peut grandement simplifier le processus de conformité et réduire considérablement le temps nécessaire à la collecte des preuves de conformité pour votre environnement Azure et hybride.

Pour en savoir plus, consultez les articles connexes suivants :

-   [Mise à jour des packages de conformité dynamique dans votre tableau de bord de conformité réglementaire (préversion)](update-regulatory-compliance-packages.md) : découvrez cette fonctionnalité d’évaluation qui vous permet de mettre à jour les normes affichées dans votre tableau de bord de conformité réglementaire en fonction des nouveaux packages *dynamiques*. Vous pouvez également utiliser la même fonctionnalité en préversion pour ajouter de nouveaux packages de conformité et surveiller votre conformité avec des normes supplémentaires. 
-   [Surveillance de l’intégrité de la sécurité dans Azure Security Center](security-center-monitoring.md) : découvrez comment surveiller l’intégrité de vos ressources Azure.
-   [Gestion des recommandations de sécurité dans Azure Security Center](security-center-recommendations.md) : découvrez comment utiliser les recommandations proposées par Azure Security Center pour protéger vos ressources Azure.
-   [Améliorer votre degré de sécurisation dans Azure Security Center](secure-score-security-controls.md) : découvrez comment classer par ordre de priorité les vulnérabilités et les recommandations pour améliorer votre niveau de sécurité.
