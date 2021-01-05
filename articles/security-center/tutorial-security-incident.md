---
title: Tutoriel sur la réponse aux alertes - Azure Security Center
description: Dans ce tutoriel, vous allez apprendre à trier les alertes de sécurité et à déterminer la cause racine et l’étendue d’une alerte.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 181e3695-cbb8-4b4e-96e9-c4396754862f
ms.service: security-center
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/30/2020
ms.author: memildin
ms.openlocfilehash: b9f0b3219a75900a44a73ca0fc3e453f023bddb8
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92787157"
---
# <a name="tutorial-triage-investigate-and-respond-to-security-alerts"></a>Tutoriel : Trier les alertes de sécurité, les examiner et y répondre
Security Center analyse continuellement vos charges de travail de cloud hybride à l’aide de l’analytique avancée et du renseignement sur les menaces, afin de vous avertir des activités potentiellement malveillantes détectées sur vos ressources cloud. Vous pouvez également intégrer à Security Center des alertes provenant d’autres produits et services de sécurité. Lorsqu’une alerte est déclenchée, vous devez agir rapidement pour examiner et résoudre le problème de sécurité potentiel. 

Dans ce didacticiel, vous apprendrez à :

> [!div class="checklist"]
> * Trier les alertes de sécurité
> * Examiner une alerte de sécurité pour déterminer la cause racine
> * Répondre à une alerte de sécurité et atténuer cette cause racine

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis
Pour parcourir les fonctionnalités couvertes dans ce tutoriel, vous devez avoir activé Azure Defender. Vous pouvez essayer gratuitement Azure Defender. Pour en savoir plus, consultez la [page de tarification](https://azure.microsoft.com/pricing/details/security-center/). Le guide de démarrage rapide [Bien démarrer avec Security Center](security-center-get-started.md) vous guide tout au long de la procédure de mise à niveau.


## <a name="triage-security-alerts"></a>Trier les alertes de sécurité
Security Center fournit une vue unifiée de toutes les alertes de sécurité. Les alertes de sécurité sont classées en fonction de la gravité de l’activité détectée. 

Triez vos alertes à partir de la page **Alertes de sécurité**  :

:::image type="content" source="./media/tutorial-security-incident/alerts-list.png" alt-text="Page de la liste des alertes de sécurité" lightbox="./media/tutorial-security-incident/alerts-list.png":::

Utilisez cette page pour examiner les alertes de sécurité actives dans votre environnement pour décider de l’alerte à examiner en premier.

Quand vous triez les alertes de sécurité, hiérarchisez-les en fonction de leur gravité en traitant d’abord celles qui présentent un niveau de gravité supérieur. Découvrez-en plus sur la gravité des alertes dans [Comment les alertes sont-elles classifiées ?](security-center-alerts-overview.md#how-are-alerts-classified).

> [!TIP]
> Vous pouvez connecter Azure Security Center aux solutions SIEM les plus populaires, y compris Azure Sentinel, et consommer les alertes à partir de l’outil de votre choix. Apprenez-en davantage dans [Diffuser des alertes vers un système SIEM, SOAR ou une solution de gestion des services informatiques](export-to-siem.md).


## <a name="investigate-a-security-alert"></a>Examiner une alerte de sécurité

Quand vous avez décidé de l’alerte à examiner en premier :

1. Sélectionnez l’alerte souhaitée.
1. Dans la page de vue d’ensemble de l’alerte, sélectionnez la ressource à examiner en premier.
1. Commencez votre investigation dans le volet gauche, qui affiche les informations générales de l’alerte de sécurité.

    :::image type="content" source="./media/tutorial-security-incident/alert-details-left-pane.png" alt-text="Volet gauche de la page Détails de l’alerte mettant en évidence les informations générales":::

    Ce volet affiche les éléments suivants :
    - Gravité de l’alerte, état et durée d’activité
    - Description qui explique l’activité précise détectée
    - Ressources affectées
    - Intention de la chaîne d’arrêt de l’activité sur la matrice MITRE ATT&CK

1. Pour obtenir des informations plus détaillées qui peuvent vous aider à examiner l’activité suspecte, consultez l’onglet **Détails de l’alerte**.

1. Une fois que vous avez consulté cette page, vous avez peut-être suffisamment d’informations pour passer à l’élaboration d’une réponse. Si vous avez besoin de plus de détails :

    - Contactez le propriétaire de la ressource pour vérifier si l’activité détectée est un faux positif.
    - Examinez les journaux bruts générés par la ressource attaquée.

## <a name="respond-to-a-security-alert"></a>Répondre à une alerte de sécurité
Après avoir examiné une alerte de sécurité et compris son étendue, vous pouvez y répondre à partir d’Azure Security Center :

1.  Ouvrez l’onglet **Entreprendre une action** pour voir les réponses recommandées.

    :::image type="content" source="./media/tutorial-security-incident/alert-details-take-action.png" alt-text="Onglet Entreprendre une action pour les alertes de sécurité" lightbox="./media/tutorial-security-incident/alert-details-take-action.png":::

1.  Consultez la section **Atténuer la menace** , qui indique les étapes d’investigation manuelle nécessaires à l’atténuation du problème.
1.  Pour renforcer vos ressources et empêcher les futures attaques de ce genre, appliquez les recommandations de sécurité indiquées dans la section **Empêcher les attaques futures**.
1.  Pour déclencher une application logique avec des étapes de réponse automatisée, utilisez la section **Déclencher une réponse automatisée**.
1.  Si l’activité détectée *n’est pas* malveillante, vous pouvez supprimer les alertes futures de ce genre à l’aide de la section **Supprimer les alertes similaires**.

1.  Quand vous avez terminé d’examiner l’alerte et y avez répondu de manière appropriée, changez l’état en **Ignoré**.

    :::image type="content" source="./media/tutorial-security-incident/set-status-dismissed.png" alt-text="Définition de l’état d’une alerte":::

    L’alerte est alors supprimée de la liste des alertes principales. Vous pouvez utiliser le filtre de la page de la liste des alertes pour voir toutes les alertes ayant l’état **Ignoré**.

1.  Nous vous encourageons à fournir à Microsoft des commentaires sur l’alerte,
    1. en la signalant comme étant **Utile** ou **Inutile**.
    1. Sélectionnez une raison et ajoutez un commentaire.

        :::image type="content" source="./media/tutorial-security-incident/alert-feedback.png" alt-text="Fournir des commentaires à Microsoft sur l’utilité d’une alerte":::

    > [!TIP]
    > Nous examinons vos commentaires afin d’améliorer nos algorithmes et de fournir de meilleures alertes de sécurité.

## <a name="end-the-tutorial"></a>Terminer le tutoriel

D’autres guides de démarrage rapide et didacticiels de cette collection reposent sur ce guide. Si vous envisagez de suivre les tutoriels et guides de démarrage rapide suivants, maintenez activés le provisionnement automatique et Azure Defender. 

Si vous n’envisagez pas de continuer ou si vous souhaitez désactiver l’une de ces fonctionnalités :

1. Revenez au menu principal de Security Center et sélectionnez **Tarifs et paramètres**.
1. Sélectionnez l’abonnement approprié.
1. Pour passer à une version antérieure, sélectionnez **Azure Defender désactivé**.
1. Pour désactiver le provisionnement automatique, ouvrez la page **Collecte de données** et définissez **Provisionnement automatique** sur **Désactivé**.
4. Sélectionnez **Enregistrer**.

>[!NOTE]
> La désactivation du provisionnement automatique ne supprime pas l’agent Log Analytics des machines virtuelles Azure sur lesquelles il est déjà installé. La désactivation de l’approvisionnement automatique limite la surveillance de la sécurité pour vos ressources.
>

## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez appris les fonctionnalités de Security Center à utiliser pour répondre à une alerte de sécurité. Pour des documents connexes, consultez :

- [Répondre aux alertes Azure Defender pour Key Vault](defender-for-key-vault-usage.md)
- [Alertes de sécurité - guide de référence](alerts-reference.md)
- [Présentation d’Azure Defender](azure-defender.md)
