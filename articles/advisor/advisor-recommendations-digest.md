---
title: Synthèse des recommandations pour Azure Advisor
description: Obtenir une synthèse périodique de vos recommandations actives
ms.topic: article
ms.date: 03/16/2020
ms.author: sagupt
ms.openlocfilehash: e446fca11e029e28e44ada884efd071f3142514c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86518077"
---
# <a name="configure-periodic-summary-for-recommendations"></a>Configurer une synthèse périodique des recommandations

Les **synthèses de recommandations** Advisor offrent un moyen simple et proactif de rester toujours à jour de vos recommandations actives, toutes catégories confondues. Cette fonctionnalité permet de configurer des notifications périodiques résumant vos recommandations actives, toutes catégories confondues. Vous pouvez choisir le canal des notifications, (e-mail, SMS ou autres) à l’aide de groupes d’actions. Cet article vous montre comment configurer des **synthèses de recommandations** pour vos recommandations Advisor.


## <a name="setting-up-your-recommendation-digest"></a>Configuration de votre synthèse de recommandations 

L’expérience de création d’une **synthèse de recommandations** vous aidera à configurer la synthèse. Vous pouvez sélectionner les paramètres ci-dessous pour les configurations :
1. Catégorie : Il existe des catégories de recommandations telles que les coûts, la haute disponibilité, les performances et l’excellence opérationnelle. Cette fonctionnalité n’est pas encore disponible pour les recommandations de sécurité.
2. Fréquence des synthèses : La fréquence des notifications de synthèse peut être hebdomadaire, bihebdomadaire ou mensuelle.
3. Groupe d’actions : Vous pouvez sélectionner un groupe d’actions existant ou en créer un. Pour plus d’informations sur les groupes d’actions, consultez [Créer et gérer des groupes d’actions](../azure-monitor/platform/action-groups.md).
4. Langue de la synthèse
5. Nom de la synthèse des recommandations : Vous pouvez utiliser une chaîne au nom explicite pour mieux suivre et superviser les synthèses.

## <a name="steps-to-create-recommendation-digest-in-azure-portal"></a>Étapes de création d’une synthèse de recommandations dans le portail Azure

Voici les étapes à suivre pour créer une **synthèse de recommandations** :
* **Étape 1 :** Dans le portail Azure, accédez à **Advisor** puis, sous la section **Supervision**, sélectionnez **Synthèse des recommandations**. 

   ![Point d’entrée de la synthèse de recommandations](./media/digest-0.png)

* **Étape 2 :** Sélectionnez **Nouvelle synthèse de recommandations** dans la barre supérieure, comme indiqué ci-dessous :

   ![Créer une synthèse de recommandations](./media/digest-5.png)

* **Étape 3 :** Dans la section **Étendue**, sélectionnez un **abonnement** pour votre synthèse.

   ![Fournir des entrées pour la synthèse des recommandations](./media/digest-1.png)

* **Étape 4 :** Dans la section **Condition**, sélectionnez des configurations comme **catégorie**, **fréquence** et **langue**.

   ![Fournir des entrées pour la synthèse des recommandations - Conditions](./media/digest-2.png)

* **Étape 5 :** Dans la section **Groupe d’actions**, sélectionnez un **groupe d’actions** pour la synthèse. Pour plus d’informations, consultez [Créer et gérer des groupes d’actions](../azure-monitor/platform/action-groups.md).

   ![Fournir des entrées pour la synthèse des recommandations - Groupe d’actions](./media/digest-3.png)

* **Étape 6 :** Dans cette section finale concernant les **détails de la synthèse**, vous pouvez attribuer un nom et un état à votre synthèse de recommandations. Appuyez sur **Créer une synthèse de recommandations** pour terminer la configuration.
   ![Terminer la création de la synthèse de recommandations](./media/digest-4.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les recommandations d’Advisor, consultez :
* [Présentation du conseiller Azure](advisor-overview.md)
* [Prise en main d’Advisor](advisor-get-started.md)
* [Recommandations du conseiller en matière de coûts](advisor-cost-recommendations.md)
* [Recommandations du conseiller en matière de performances](advisor-performance-recommendations.md)
* [Recommandations du conseiller en matière de sécurité](advisor-security-recommendations.md)
* [Recommandations d’excellence opérationnelle Advisor](advisor-operational-excellence-recommendations.md)
* [API REST Advisor](/rest/api/advisor/)
