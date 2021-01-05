---
title: Gérer les environnements de service d’intégration dans Azure Logic Apps
description: Vérifiez l’intégrité du réseau et gérez les applications logiques, les connexions, les connecteurs personnalisés et les comptes d’intégration dans votre environnement de service d’intégration pour Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, jonfan, logicappspm
ms.topic: conceptual
ms.date: 10/25/2020
ms.openlocfilehash: 4df9543e5e747de640562b7e5be224e257e0cfd1
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92676109"
---
# <a name="manage-your-integration-service-environment-ise-in-azure-logic-apps"></a>Gérez votre environnement de service d’intégration dans Azure Logic Apps

Cet article explique comment effectuer des tâches de gestion pour votre [environnement de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), par exemple :

* gérer les ressources telles que les applications logiques, les connexions, les comptes d’intégration et les connecteurs dans votre ISE.

* Vérifier l’intégrité du réseau de votre ISE.

* Pour augmenter la capacité, redémarrer votre ISE, ou supprimer votre ISE, suivez les étapes de cette rubrique. Pour ajouter ces artefacts à votre environnement ISE, consultez [Ajouter des artefacts à votre environnement de service d’intégration](../logic-apps/add-artifacts-integration-service-environment-ise.md).

## <a name="view-your-ise"></a>Voir votre environnement ISE

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans la zone de recherche du portail, entrez « environnements de service d’intégration », puis sélectionnez **Environnements de service d’intégration**.

   ![Rechercher les environnements de service d’intégration](./media/ise-manage-integration-service-environment/find-integration-service-environment.png)

1. Dans la liste des résultats, sélectionnez votre environnement de service d’intégration.

   ![Sélection d’un environnement de service d’intégration](./media/ise-manage-integration-service-environment/select-integration-service-environment.png)

1. Passez aux sections suivantes pour rechercher des applications logiques, des connexions, des connecteurs ou des comptes d’intégration dans votre environnement ISE.

<a name="check-network-health"></a>

## <a name="check-network-health"></a>Vérifier l’intégrité du réseau

Dans le menu de votre environnement ISE, sous **Paramètres** , sélectionnez **Intégrité du réseau**. Ce volet présente l’état d’intégrité de vos sous-réseaux et les dépendances sortantes par rapport à d’autres services.

![Vérifier l’intégrité du réseau](./media/ise-manage-integration-service-environment/ise-check-network-health.png)

> [!CAUTION]
> Si le réseau de votre ISE devient défectueux, l’App Service Environment (ASE) interne qu’utilise votre ISE peut également devenir défectueux. Si l’ASE est défectueux pendant plus de sept jours, il est suspendu. Pour résoudre cet état, vérifiez la configuration de votre réseau virtuel. Résolvez tous les problèmes détectés, puis redémarrez votre ISE. Dans le cas contraire, après 90 jours, l’ASE suspendu est supprimé et votre ISE devient inutilisable. Veillez donc à garder votre ISE intègre pour permettre le trafic nécessaire.
> 
> Pour plus d’informations, consultez les rubriques suivantes :
>
> * [Présentation des diagnostics Azure App Service](../app-service/overview-diagnostics.md)
> * [Journalisation des messages pour Azure App Service Environment](../app-service/environment/using-an-ase.md#logging)

<a name="find-logic-apps"></a>

## <a name="manage-your-logic-apps"></a>Gérer les applications logiques

Vous pouvez voir et gérer les applications logiques qui se trouvent dans votre environnement ISE.

1. Dans le menu de votre environnement ISE, sous **Paramètres** , sélectionnez **Applications logiques**.

   ![Voir les applications logiques](./media/ise-manage-integration-service-environment/ise-find-logic-apps.png)

1. Pour supprimer les applications logiques dont vous n’avez plus besoin dans votre environnement ISE, sélectionnez-les, puis sélectionnez **Supprimer**. Pour confirmer la suppression, sélectionnez **Oui**.

> [!NOTE]
> Si vous supprimez et recréez une application logique enfant, vous devez réenregistrer l’application logique parente. L’application enfant recréée aura des métadonnées différentes.
> Si vous ne réenregistrez pas l’application logique parente après avoir recréé son enfant, vos appels à l’application logique enfant échouent et une erreur de type « non autorisé » s’affiche. Ce comportement s’applique aux applications logiques parent-enfant, par exemple celles qui utilisent des artefacts dans les comptes d’intégration ou qui appellent des fonctions Azure.

<a name="find-api-connections"></a>

## <a name="manage-api-connections"></a>Gérer les connexions d’API

Vous pouvez afficher et gérer les connexions qui ont été créées par les applications logiques qui s’exécutent dans votre environnement ISE.

1. Dans le menu de votre environnement ISE, sous **Paramètres** , sélectionnez **Connexions d’API**.

   ![Voir les connexions d’API](./media/ise-manage-integration-service-environment/ise-find-api-connections.png)

1. Pour supprimer les connexions dont vous n’avez plus besoin dans votre environnement ISE, sélectionnez-les, puis sélectionnez **Supprimer**. Pour confirmer la suppression, sélectionnez **Oui**.

<a name="manage-api-connectors"></a>

## <a name="manage-ise-connectors"></a>Gérer les connecteurs ISE

Vous pouvez voir et gérer les connecteurs d’API qui sont déployés dans votre environnement ISE.

1. Dans le menu de votre environnement ISE, sous **Paramètres** , sélectionnez **Connecteurs managés**.

   ![Voir les connecteurs managés](./media/ise-manage-integration-service-environment/ise-view-managed-connectors.png)

1. Pour supprimer les connecteurs que vous ne souhaitez pas rendre disponibles dans votre environnement ISE, sélectionnez-les, puis sélectionnez **Supprimer**. Pour confirmer la suppression, sélectionnez **Oui**.

<a name="find-custom-connectors"></a>

## <a name="manage-custom-connectors"></a>Gérer les connecteurs personnalisés

Vous pouvez voir et gérer les connecteurs personnalisés que vous avez déployés dans votre environnement ISE.

1. Dans le menu de votre environnement ISE, sous **Paramètres** , sélectionnez **Connecteurs personnalisés**.

   ![Trouver des connecteurs personnalisés](./media/ise-manage-integration-service-environment/ise-find-custom-connectors.png)

1. Pour supprimer les connecteurs personnalisés dont vous n’avez plus besoin dans votre environnement ISE, sélectionnez-les, puis sélectionnez **Supprimer**. Pour confirmer la suppression, sélectionnez **Oui**.

<a name="find-integration-accounts"></a>

## <a name="manage-integration-accounts"></a>Gérer les comptes d’intégration

1. Dans le menu de votre environnement ISE, sous **Paramètres** , sélectionnez **Comptes d’intégration**.

   ![Cherchez des comptes d’intégration](./media/ise-manage-integration-service-environment/ise-find-integration-accounts.png)

1. Pour supprimer des comptes d’intégration de votre environnement ISE quand vous n’en avez plus besoin, sélectionnez-les, puis sélectionnez **Supprimer**.

<a name="add-capacity"></a>

## <a name="add-ise-capacity"></a>Ajouter de la capacité à l’ISE

L’unité de base d’ISE Premium dispose d’une capacité fixe ; si vous avez besoin de davantage de débit, vous pouvez ajouter des unités d’échelle, pendant la création ou après. La référence SKU Développeur n’inclut pas la capacité à ajouter des unités d’échelle.

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre ISE.

1. Pour consulter les mesures de performances et d’utilisation pour votre environnement de service d’intégration (ISE), sélectionnez **Vue d’ensemble** dans le menu de votre ISE.

   ![Afficher l’utilisation pour l’ISE](./media/ise-manage-integration-service-environment/integration-service-environment-usage.png)

1. Sous **Paramètres** , sélectionnez **Scale-out**. Dans le volet **configurer** , opérez une sélection parmi les options suivantes :

   * [**Mise à l’échelle manuelle**](#manual-scale) : Mettez à l’échelle en fonction du nombre d’unités de traitement que vous voulez utiliser.
   * [**Mise à l’échelle automatique personnalisée**](#custom-autoscale) : Mettez à l’échelle en fonction des métriques de performances en opérant une sélection parmi différents critères et en spécifiant des conditions de seuil pour remplir ceux-ci.

   ![Capture d’écran montrant la page « Scale-out » avec l’option « Mise à l’échelle manuelle » sélectionnée](./media/ise-manage-integration-service-environment/select-scale-out-options.png)

<a name="manual-scale"></a>

### <a name="manual-scale"></a>Mise à l’échelle manuelle

1. Après avoir sélectionné **Mise à l’échelle manuelle** , pour **Capacité supplémentaire** , sélectionnez le nombre d’unités d’échelle que vous souhaitez utiliser.

   ![Sélectionner le type de mise à l’échelle souhaité](./media/ise-manage-integration-service-environment/select-manual-scale-out-units.png)

1. Quand vous avez terminé, sélectionnez **Enregistrer**.

<a name="custom-autoscale"></a>

### <a name="custom-autoscale"></a>Mise à l’échelle automatique personnalisée

1. Après avoir sélectionné **Mise à l’échelle automatique personnalisée** , pour **Nom du paramètre de mise à l’échelle automatique** , entrez un nom pour votre paramètre et, le cas échéant, sélectionnez le groupe de ressources Azure auquel le paramètre appartient.

   ![Fournir un nom pour le paramètre de mise à l’échelle automatique et sélectionner un groupe de ressources](./media/ise-manage-integration-service-environment/select-custom-autoscale.png)

1. Pour la condition **Par défaut** , sélectionnez soit **Mettre à l’échelle selon une métrique** soit **Mettre à l’échelle d’un nombre d’instances spécifique**.

   * Si vous choisissez la mise à l’échelle basée sur les instances, entrez un nombre d’unités de traitement, soit une valeur comprise entre 0 et 10.

   * Si vous choisissez la méthode basée sur les mesures, procédez comme suit :

     1. Dans la section **Règles** , sélectionnez **+Ajouter une règle**.

     1. Dans le volet **Règle de mise à l’échelle** , configurez vos critères et l’action à effectuer lorsque la règle se déclenche.

     1. Pour **Limites d’instance** , spécifiez les valeurs suivantes :

        * **Minimale**  : Nombre minimal d’unités de traitement à utiliser
        * **Maximum**  : Nombre maximal d’unités de traitement à utiliser
        * **Par défaut** : Si des problèmes surviennent lors de la lecture des métriques de ressources et que la capacité actuelle est inférieure à la capacité par défaut, la mise à l’échelle automatique augmente le nombre par défaut d’unités de traitement. En revanche, si la capacité actuelle dépasse la capacité par défaut, la mise à l’échelle automatique n’effectue pas de scale-in.

1. Pour ajouter une autre condition, sélectionnez **Ajouter une condition de mise à l’échelle**.

1. Lorsque vous avez terminé de configurer vos paramètres de mise à l’échelle, enregistrez vos modifications.

<a name="restart-ISE"></a>

## <a name="restart-ise"></a>Redémarrer l’ISE

Si vous changez votre serveur DNS ou modifiez ses paramètres, vous devez redémarrer votre ISE afin qu’il prenne en compte ces modifications. Le redémarrage d’un ISE de référence SKU Premium ne provoque pas de temps d’arrêt en raison de la redondance et des composants qui redémarrent un par un pendant le recyclage. En revanche, un ISE de référence SKU Développeur connaît un temps d’arrêt, car il n’existe aucune redondance. Pour plus d’informations, consultez [Références SKU d’ISE](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level).

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre ISE.

1. Dans le menu ISE, sélectionnez **Vue d’ensemble**. Dans la barre d’outils Vue d’ensemble, sélectionnez **Redémarrer**.

   ![Redémarrer l’environnement de service d’intégration](./media/connect-virtual-network-vnet-isolated-environment/restart-integration-service-environment.png)

<a name="delete-ise"></a>

## <a name="delete-ise"></a>Supprimer un environnement ISE

Avant de supprimer un environnement ISE dont vous n’avez plus besoin ou un groupe de ressources Azure qui contient un environnement ISE, vérifiez que vous n’avez pas de stratégies ni de verrous sur le groupe de ressources Azure qui contient ces ressources ou sur votre réseau virtuel Azure, car ces éléments peuvent bloquer la suppression.

Après avoir supprimé votre environnement ISE, vous devrez peut-être attendre jusqu’à 9 heures avant de tenter de supprimer votre réseau virtuel Azure ou vos sous-réseaux.

## <a name="next-steps"></a>Étapes suivantes

* [Ajouter des ressources à des environnements de service d'intégration](../logic-apps/add-artifacts-integration-service-environment-ise.md)
