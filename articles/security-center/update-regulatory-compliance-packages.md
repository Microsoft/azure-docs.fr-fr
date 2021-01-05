---
title: Utilisation du tableau de bord de conformité réglementaire dans Azure Security Center
description: Découvrez comment ajouter des normes réglementaires au tableau de bord de conformité réglementaire dans Security Center et comment les supprimer.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: c42d02e4-201d-4a95-8527-253af903a5c6
ms.service: security-center
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/15/2020
ms.author: memildin
ms.openlocfilehash: e7e1567a487dc6cadc94a42f02c597ff0e02665b
ms.sourcegitcommit: 65d518d1ccdbb7b7e1b1de1c387c382edf037850
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2020
ms.locfileid: "94372759"
---
# <a name="customizing-the-set-of-standards-in-your-regulatory-compliance-dashboard"></a>Personnalisation de l’ensemble de normes du tableau de bord de conformité réglementaire

Azure Security Center compare continuellement la configuration de vos ressources avec les exigences des normes, réglementations et tests d’évaluation du secteur. Le **tableau de bord de conformité réglementaire** fournit des analyses sur votre situation de conformité selon la façon dont vous répondez aux exigences et contrôles de conformité.


## <a name="overview-of-compliance-packages"></a>Vue d’ensemble des packages de conformité

Les standards de l’industrie, les normes réglementaires et les benchmarks sont représentés dans Security Center sous forme de *packages de conformité*.  Chaque package correspond à une initiative définie dans Azure Policy. Pour voir la correspondance entre les données de conformité et les évaluations dans votre tableau de bord, ajoutez un package de conformité à votre groupe d’administration ou à votre abonnement sur la page **Stratégie de sécurité**. (Pour plus d’informations sur Azure Policy et les initiatives, consultez [Utilisation des stratégies de sécurité](tutorial-security-policy.md).)

Après l’intégration dans l’étendue sélectionnée, le standard, la norme ou le benchmark affecte l’initiative à l’étendue et apparaît dans le tableau de bord de conformité réglementaire avec toutes les données de conformité associées sous forme d’évaluations. Vous pouvez également télécharger des rapports de synthèse pour toutes les normes intégrées.

Microsoft suit également les normes réglementaires et améliore automatiquement sa couverture dans certains packages au fur et à mesure. Lorsque Microsoft publie du nouveau contenu pour l’initiative (nouvelles stratégies qui se mappent à d’autres contrôles dans la norme), le contenu supplémentaire s’affiche automatiquement dans votre tableau de bord.

> [!TIP]
> Parmi les normes qui s’améliorent à mesure que Microsoft publie de nouveaux contenus, citons **Azure CIS 1.1.0 (nouveau)** (plus formellement, la [version 1.1.0 de CIS Microsoft Azure Foundations Benchmark](https://www.cisecurity.org/benchmark/azure/)). Vous devez l’ajouter à votre tableau de bord avec « Azure CIS 1.1.0 », la représentation d’Azure CIS configurée par défaut dans chaque environnement Security Center. Ce package s’appuie sur un ensemble de règles statique. Il est automatiquement mis à jour au fil du temps, chaque version comportant plus de stratégies que la précédente. Mettez à jour vers le nouveau package dynamique en procédant comme suit.


## <a name="available-packages"></a>Packages disponibles

Vous pouvez ajouter des normes et des standards comme NIST SP 800-53 R4, SWIFT CSP CSCF-v2020, UK Official et UK NHS, Canada Federal PBMM et Azure CIS 1.1.0 (nouveau), une représentation plus complète d’Azure CIS 1.1.0. 

De plus, vous pouvez ajouter le **Benchmark de sécurité Azure**, des directives propres à Azure et créées par Microsoft qui donnent les meilleures pratiques de sécurité et de conformité pour des frameworks de conformité courantes. ([En savoir plus sur le Benchmark de sécurité Azure](../security/benchmarks/introduction.md).)

Des normes et standards supplémentaires seront pris en charge dans le tableau de bord dès qu’ils seront disponibles. 


## <a name="add-a-regulatory-standard-to-your-dashboard"></a>Ajouter une norme réglementaire au tableau de bord

La procédure suivante permet d’ajouter un package pour surveiller la conformité avec l’une des normes réglementaires prises en charge.

> [!NOTE]
> Seuls les utilisateurs qui sont propriétaires ou contributeurs de stratégie disposent des autorisations nécessaires pour ajouter des normes de conformité. 

1. Dans la barre latérale de Security Center, sélectionnez **Conformité réglementaire** pour ouvrir le tableau de bord de conformité réglementaire. Ici, vous pouvez voir les normes de conformité actuellement attribuées aux abonnements actuellement sélectionnés.   

1. En haut de la page, sélectionnez **Gérer les stratégies de conformité**. La page Gestion des stratégies s’affiche.

1. Sélectionnez l’abonnement ou le groupe d’administration pour lequel vous souhaitez gérer la situation de conformité réglementaire. 

    > [!TIP]
    > Nous vous recommandons de sélectionner l’étendue la plus élevée pour laquelle la norme est applicable afin que les données de conformité soient agrégées et suivies pour toutes les ressources imbriquées. 

1. Pour ajouter les normes et standards correspondant à votre organisation, cliquez sur **Ajouter d’autres normes et standards**. 

1. Sur la page **Ajouter des normes de conformité réglementaire**, vous pouvez rechercher des packages pour tous les standards et normes disponibles, par exemple :

    - **Benchmark de sécurité Azure**
    - **NIST SP 800-53 R4**
    - **NIST SP 800 171 R2**
    - **SWIFT CSP CSCF-v2020**
    - **UKO et UK NHS**
    - **Canada PBMM**
    
    ![Ajout de packages réglementaires au tableau de bord de conformité réglementaire d’Azure Security Center](./media/update-regulatory-compliance-packages/dynamic-regulatory-compliance-additional-standards.png)

1. Sélectionnez **Ajouter** et entrez tous les détails nécessaires pour l’initiative spécifique, comme l’étendue, les paramètres et la correction.

1. Dans la barre latérale de Security Center, sélectionnez à nouveau **Conformité réglementaire** pour revenir au tableau de bord de conformité réglementaire.
    * Votre nouvelle norme s’affiche à présent dans la liste des normes réglementaires et des standards du secteur. 
    * Si vous avez ajouté **Azure CIS 1.1.0 (nouveau)** , la vue *statique* d’origine de votre conformité Azure CIS 1.1.0 restera également à ses côtés. Elle pourrait être supprimée automatiquement à l’avenir.

    > [!NOTE]
    > L’affichage d’une norme nouvellement ajoutée dans le tableau de bord de conformité peut prendre quelques heures.

    [![Tableau de bord de conformité réglementaire présentant l’ancienne et la nouvelle version d’Azure CIS](media/update-regulatory-compliance-packages/regulatory-compliance-dashboard-with-benchmark-small.png)](media/update-regulatory-compliance-packages/regulatory-compliance-dashboard-with-benchmark.png#lightbox)


## <a name="removing-a-standard-from-your-dashboard"></a>Suppression d’une norme de votre tableau de bord

Si certaines des normes réglementaires fournies ne sont pas pertinentes pour votre organisation, il est facile de les supprimer de l’interface utilisateur. Cela vous permet de personnaliser davantage le tableau de bord de conformité réglementaire et de vous concentrer uniquement sur les normes qui vous sont applicables.

Pour supprimer une norme :

1. Dans le menu de Security Center, sélectionnez **Stratégie de sécurité**.

1. Sélectionnez l’abonnement approprié à partir duquel vous souhaitez supprimer une norme.

    > [!NOTE]
    > Vous pouvez supprimer une norme d’un abonnement, mais pas d’un groupe d’administration. 

    La page Stratégie de sécurité s’ouvre. Pour l’abonnement sélectionné, il affiche la stratégie par défaut, le secteur et les normes réglementaires, ainsi que les initiatives personnalisées que vous avez créées.

    :::image type="content" source="./media/update-regulatory-compliance-packages/remove-standard.png" alt-text="Suppression d’une norme réglementaire de votre tableau de bord de conformité réglementaire dans Azure Security Center":::

1. Pour la norme que vous souhaitez supprimer, sélectionnez **Désactiver**. Une fenêtre de confirmation s’ouvre.

    :::image type="content" source="./media/update-regulatory-compliance-packages/remove-standard-confirm.png" alt-text="Confirmez que vous voulez vraiment supprimer la norme réglementaire que vous avez sélectionnée":::.

1. Sélectionnez **Oui**. La norme est supprimée. 


## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert comment **ajouter des packages de conformité** pour surveiller votre conformité avec des normes et des standards supplémentaires. 

Pour d’autres informations connexes, consultez les articles suivants : 

- [Benchmark de sécurité Azure](../security/benchmarks/introduction.md)
- [Tableau de bord de conformité réglementaire de Security Center](security-center-compliance-dashboard.md)
- [Utilisation de stratégies de sécurité](tutorial-security-policy.md)