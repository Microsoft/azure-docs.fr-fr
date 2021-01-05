---
title: Gérer les applications logiques dans le portail Azure
description: Modifiez, activez, désactivez ou supprimez des applications logiques à l’aide du portail Azure.
services: logic-apps
ms.suite: integration
author: lauradolan
ms.author: ladolan
ms.reviewer: estfan, jonfan, logicappspm
ms.topic: article
ms.custom: mvc
ms.date: 07/20/2020
ms.openlocfilehash: d50f577a7170982be004cc8957114f79675fbc6e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006096"
---
# <a name="manage-logic-apps-in-the-azure-portal"></a>Gérer les applications logiques dans le portail Azure

Vous pouvez gérer les applications logiques à l’aide du [portail Azure](https://portal.azure.com) ou de [Visual Studio](manage-logic-apps-with-visual-studio.md). Cet article explique comment modifier, désactiver, activer ou supprimer des applications logiques dans le portail Azure. Si vous débutez avec Azure Logic Apps, consultez [Qu’est-ce qu’Azure Logic Apps ?](logic-apps-overview.md).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Une application logique existante. Pour en savoir plus sur la création d’une application logique par le biais du portail Azure, consultez [Démarrage rapide : Créer votre premier workflow à l’aide d’Azure Logic Apps - Portail Azure](quickstart-create-first-logic-app-workflow.md).

<a name="find-logic-app"></a>

## <a name="find-your-logic-apps"></a>Trouver vos applications logiques

Pour rechercher et ouvrir votre application logique, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec votre compte Azure.

1. Dans la barre de recherche Azure, entrez `logic apps`, puis sélectionnez **Logic Apps**.

   ![Capture d’écran du menu du portail Azure, avec l’expression Logic Apps recherchée et sélectionnée dans la barre de recherche.](./media/manage-logic-apps-with-azure-portal/find-select-logic-apps.png)

1. Dans la page **Logic Apps**, recherchez et sélectionnez l’application logique à gérer.

   Une fois que le volet **Vue d’ensemble** de l’application logique s’ouvre, vous pouvez filtrer la liste qui s’affiche dans la page **Logic Apps** des manières suivantes :

   * Rechercher des applications logiques par nom
   * Filtrer les applications logiques par abonnement, groupe de ressources, emplacement et étiquettes
   * Regrouper les applications logiques par groupe de ressources, type, abonnement et emplacement

## <a name="view-logic-app-properties"></a>Afficher les propriétés de l’application logique

1. Dans le portail Azure, [recherchez et ouvrez votre application logique](#find-logic-app).

1. Dans le menu de votre application logique, sous **Paramètres**, sélectionnez **Propriétés**.

1. Dans le volet **Propriétés**, vous pouvez afficher et copier les informations suivantes sur votre application logique :

   * **Nom**
   * **ID de ressource**
   * **Groupe de ressources**
   * **Lieu**
   * **Type** 
   * **Nom d’abonnement**
   * **Identifiant d’abonnement**
   * **Point de terminaison de l’accès**
   * **Adresses IP sortantes du runtime**
   * **Accéder aux adresses IP du point de terminaison**
   * **Adresses IP sortantes du connecteur**

## <a name="disable-or-enable-logic-apps"></a>Activer ou désactiver des applications logiques

Vous pouvez activer ou désactiver une [application logique spécifique](#disable-enable-single-logic-app) ou [plusieurs applications logiques en même temps](#disable-or-enable-multiple-logic-apps) dans le portail Azure. Vous pouvez également [activer ou désactiver des applications logiques dans Visual Studio](manage-logic-apps-with-visual-studio.md#disable-or-enable-logic-app).

La désactivation de votre application logique affecte vos instances et exécutions de workflow des manières suivantes :

* Toutes les exécutions en cours et en attente continuent jusqu’à ce qu’elles soient terminées. En fonction du nombre de ces exécutions, ce processus peut prendre un certain temps.

* Le moteur Logic Apps ne crée pas ou n’exécute pas de nouvelles instances de workflow.

* Le déclencheur ne se déclenche pas la prochaine fois que ses conditions sont remplies.

* L’état du déclencheur mémorise le point auquel l’application logique a été arrêtée. Ainsi, si vous réactivez l’application logique, le déclencheur se déclenche pour tous les éléments non traités depuis la dernière exécution.

  Pour empêcher votre application logique de se déclencher sur des éléments non traités depuis la dernière exécution, effacez l’état du déclencheur avant de réactiver l’application logique :

  1. Dans le portail Azure, [recherchez et ouvrez votre application logique](#find-logic-app).

  1. Modifiez une partie du déclencheur de l’application logique.

  1. Enregistrez vos modifications. Cette étape réinitialise l’état actuel de votre déclencheur.

  1. [Réactivez votre application logique](#disable-enable-single-logic-app).

<a name="disable-enable-single-logic-app"></a>

### <a name="disable-or-enable-single-logic-app"></a>Activer ou désactiver une application logique spécifique

1. Dans le portail Azure, [recherchez et ouvrez votre application logique](#find-logic-app).

1. Dans le menu de votre application logique, sélectionnez **Vue d’ensemble**. Choisissez l’une des options suivantes :

   * Dans la barre d’outils, sélectionnez **Désactiver**.

     ![Capture d’écran de la barre d’outils de l’application logique, avec le bouton Désactiver sélectionné.](./media/manage-logic-apps-with-azure-portal/disable-single-logic-app.png)

     Si votre application logique est déjà désactivée, vous ne voyez que l’option **Activer**.

   * Dans la barre d’outils, sélectionnez **Activer**.

     ![Capture d’écran de la barre d’outils de l’application logique, avec le bouton Activer sélectionné.](./media/manage-logic-apps-with-azure-portal/enable-single-logic-app.png)

     Si votre application logique est déjà activée, vous ne voyez que l’option **Désactiver**. 

   Le portail Azure affiche une notification dans la barre d’outils Azure principale qui confirme si votre opération a réussi ou échoué.

   ![Capture d’écran du portail Azure, montrant la notification pour confirmer l’état de l’opération](./media/manage-logic-apps-with-azure-portal/operation-confirmation-notification.png)

<a name="disable-or-enable-multiple-logic-apps"></a>

### <a name="disable-or-enable-multiple-logic-apps"></a>Activer ou désactiver plusieurs applications logiques

1. Dans le portail Azure, [recherchez les applications logiques](#find-logic-app) que vous souhaitez activer ou désactiver.

1. Pour vérifier si une application logique est activée ou désactivée, dans la page **Logic Apps**, consultez la colonne **État** propre à cette application logique. 

   ![Capture d’écran de la page Logic Apps dans le portail Azure, avec la liste des applications logiques triées selon la colonne État.](./media/manage-logic-apps-with-azure-portal/view-logic-app-status.png)

   Si la colonne **État** n’est pas visible, dans la barre d’outils **Logic Apps**, sélectionnez **Essayer la préversion**.

   ![Capture d’écran de la page Logic Apps dans le portail Azure, avec le bouton Essayer la préversion sélectionné.](./media/manage-logic-apps-with-azure-portal/select-try-preview.png)

1. Dans la colonne des cases à cocher, sélectionnez les applications logiques que vous souhaitez activer ou désactiver. Dans la barre d’outils, sélectionnez **Désactiver** ou **Activer**.

   ![Capture d’écran de la page Logic Apps dans le portail Azure, montrant les boutons Activer et Désactiver pour plusieurs applications logiques.](./media/manage-logic-apps-with-azure-portal/enable-disable-multiple-logic-apps.png)

1. Quand la boîte de confirmation s’affiche, sélectionnez **Oui** pour continuer.

   Le portail Azure affiche une notification dans la barre d’outils Azure principale qui confirme si votre opération a réussi ou échoué.

## <a name="delete-logic-apps"></a>Supprimer des applications logiques

Vous pouvez [supprimer une application logique spécifique](#delete-single-logic-app) ou [supprimer plusieurs applications logiques en même temps](#delete-multiple-logic-apps) dans le portail Azure. Vous pouvez également [supprimer votre application logique dans Visual Studio](manage-logic-apps-with-visual-studio.md#delete-your-logic-app).

La suppression de votre application logique affecte vos instances de workflow des manières suivantes :

* Toutes les exécutions en cours et en attente continuent jusqu’à ce qu’elles soient terminées. En fonction du nombre de ces exécutions, ce processus peut prendre un certain temps.

* Le moteur Logic Apps ne crée pas ou n’exécute pas de nouvelles instances de workflow.

> [!NOTE]
> Si vous supprimez et recréez une application logique enfant, vous devez réenregistrer l’application logique parente. L’application enfant recréée aura des métadonnées différentes.
> Si vous ne réenregistrez pas l’application logique parente après avoir recréé son enfant, vos appels à l’application logique enfant échouent et une erreur de type « non autorisé » s’affiche. Ce comportement s’applique aux applications logiques parent-enfant, par exemple celles qui utilisent des artefacts dans les comptes d’intégration ou qui appellent des fonctions Azure.

<a name="delete-single-logic-app"></a>

### <a name="delete-single-logic-app"></a>Supprimer une application logique spécifique

1. Dans le portail Azure, [recherchez et ouvrez votre application logique](#find-logic-app).

1. Dans le menu de votre application logique, sélectionnez **Vue d’ensemble**. Dans la barre d’outils de votre application logique, sélectionnez **Supprimer**.

   ![Capture d’écran de la barre d’outils de l’application logique, avec le bouton Supprimer sélectionné.](./media/manage-logic-apps-with-azure-portal/delete-single-logic-app.png)

1. Quand la boîte de confirmation s’affiche, entrez le nom de votre application logique, puis sélectionnez **Supprimer**.

   ![Capture d’écran de l’invite Logic Apps pour confirmer la suppression d’une application logique unique.](./media/manage-logic-apps-with-azure-portal/delete-confirmation-single-logic-app.png)

   Le portail Azure affiche une notification dans la barre d’outils Azure principale qui confirme si votre opération a réussi ou échoué.

<a name="delete-multiple-logic-apps"></a>

### <a name="delete-multiple-logic-apps"></a>Supprimer plusieurs applications logiques

1. Dans le portail Azure, [recherchez les applications logiques que vous souhaitez supprimer](#find-logic-app).

1. Dans la colonne des cases à cocher, sélectionnez les applications logiques que vous souhaitez supprimer. Dans la barre d’outils, sélectionnez **Supprimer**.

   ![Capture d’écran de la page Logic Apps, montrant plusieurs applications logiques sélectionnées pour suppression dans une liste.](./media/manage-logic-apps-with-azure-portal/delete-multiple-logic-apps.png)

1. Quand la boîte de confirmation s’affiche, entrez `yes`, puis sélectionnez **Supprimer**.

   ![Capture d’écran de l’invite Logic Apps pour confirmer la suppression de plusieurs applications logiques.](./media/manage-logic-apps-with-azure-portal/delete-confirmation-multiple-logic-apps.png)

   Le portail Azure affiche une notification dans la barre d’outils Azure principale qui confirme si votre opération a réussi ou échoué.

<a name="manage-logic-app-versions"></a>

## <a name="manage-logic-app-versions"></a>Gestion des versions des applications logiques

Il est possible d’utiliser le Portail Azure pour la gestion de version des applications logiques. Vous y trouverez l’historique des versions de votre application logique et pourrez promouvoir les versions précédentes.

<a name="find-version-history"></a>

### <a name="find-and-view-previous-versions"></a>Recherche et affichage des versions précédentes

1. Sur le Portail Azure, [recherchez l’application logique que vous souhaitez gérer](#find-logic-app).

1. Dans le menu de l’application logique, sélectionnez **Versions** sous **Outils de développement**.

   ![Capture d’écran de l’application logique dans le portail Azure, montrant une sélection de la page Versions située sous Outils de développement.](./media/manage-logic-apps-with-azure-portal/logic-apps-menu-versions.png)

1. Sélectionnez la **Version** de votre application logique à gérer dans la liste. Vous pouvez entrer l’identificateur de **Version** dans la barre de recherche pour filtrer la liste.

1. Sur la page **Version de l’historique** apparaissent les détails de la version précédente en mode lecture seule. Vous pouvez choisir entre les modes **Concepteur** et **Code** de Logic Apps.

   ![Capture d’écran de la page Version de l’historique de Logic Apps, montrant les options de mode Code et de mode Création.](./media/manage-logic-apps-with-azure-portal/history-version.png)

<a name="promote-previous-versions"></a>

### <a name="promote-previous-versions"></a>Promotion de versions précédentes

1. Dans l’historique des versions de l’application logique, [recherchez et sélectionnez la version que vous souhaitez promouvoir](#find-version-history).

1. Sur la page **Version de l’historique**, sélectionnez **Promouvoir**.

   ![Capture d’écran de la version de l’historique de l’application logique, avec le bouton pour promouvoir une version précédente.](./media/manage-logic-apps-with-azure-portal/promote-button.png)

1. Sur la page **Concepteur d’applications logiques** qui s’ouvre, modifiez la version que vous promouvez en fonction des besoins. Vous pouvez basculer entre les modes **Concepteur** et **Code**. Il est également possible de mettre à jour **Paramètres**, **Modèles** et **Connecteurs**.

   ![Capture d’écran du Concepteur Logic Apps, avec le bouton pour promouvoir une version précédente d’une application logique.](./media/manage-logic-apps-with-azure-portal/promote-page.png)

1. Pour enregistrer les mises à jour et terminer la promotion de la version précédente, sélectionnez **Enregistrer**. (Pour annuler vos modifications, sélectionnez **Ignorer**.) 

   Quand vous [affichez à nouveau l’historique des versions de votre application logique](#find-version-history), la version promue apparaît en haut de la liste, avec un nouvel identificateur.

## <a name="next-steps"></a>Étapes suivantes

* [Surveiller les applications logiques](monitor-logic-apps.md)
