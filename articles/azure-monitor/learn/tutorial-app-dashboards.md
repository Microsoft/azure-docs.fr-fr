---
title: Créer des tableaux de bord personnalisés dans Azure Application Insights | Microsoft Docs
description: Didacticiel concernant la création de tableaux de bord d’indicateur de performance clé à l’aide d’Azure Application Insights.
ms.subservice: application-insights
ms.topic: tutorial
author: lgayhardt
ms.author: lagayhar
ms.date: 09/30/2020
ms.custom: mvc, contperf-fy21q1
ms.openlocfilehash: 701480300101b019830c57d9aa000534fa63bb6a
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97028527"
---
# <a name="create-custom-kpi-dashboards-using-azure-application-insights"></a>Créer des tableaux de bord d’indicateur de performance clé à l’aide d’Azure Application Insights

Vous pouvez créer dans le portail Azure plusieurs tableaux de bord incluant des vignettes présentant des données de plusieurs ressources Azure dans différents groupes de ressources et abonnements.  Vous pouvez épingler différents graphiques et affichages d’Azure Application Insights pour créer des tableaux de bord personnalisés qui donnent une image complète de l’intégrité et des performances de votre application. Ce didacticiel vous guide dans la création d’un tableau de bord personnalisé incluant plusieurs types de données et visualisations d’Azure Application Insights.

 Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer un tableau de bord personnalisé dans Azure
> * Ajouter une vignette à partir de la Galerie de vignettes
> * Ajouter des métriques standard au tableau de bord dans Application Insights
> * Ajouter un graphique de métrique personnalisé Application Insights au tableau de bord
> * Ajouter les résultats d’une requête Logs (Analytics) au tableau de bord

## <a name="prerequisites"></a>Conditions préalables requises

Pour suivre ce tutoriel :

- Déployez une application .NET pour Azure et [activez le Kit SDK Application Insights](../app/asp-net.md).

> [!NOTE]
> Les autorisations requises pour l’utilisation des tableaux de bord sont décrites dans l’article [Présentation du contrôle d’accès relatif aux tableaux de bord](../../azure-portal/azure-portal-dashboard-share-access.md#understanding-access-control-for-dashboards).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="create-a-new-dashboard"></a>Créer un tableau de bord

> [!WARNING]
> Si vous déplacez vos ressources Application Insights vers un autre groupe de ressources ou un autre abonnement, vous devrez mettre à jour manuellement le tableau de bord en supprimant les anciennes vignettes et en épinglant de nouvelles vignettes à partir des mêmes ressources Application Insights au nouvel emplacement.

Un tableau de bord unique peut contenir des ressources de plusieurs applications, groupes de ressources et abonnements.  Démarrez le didacticiel en créant un tableau de bord pour votre application.  

1. Dans la liste déroulante du menu sur la gauche du portail Azure, sélectionnez **Tableau de bord**.

    ![Liste déroulante du menu du portail Azure](media/tutorial-app-dashboards/dashboard-from-menu.png)

2. Dans le volet Tableau de bord, sélectionnez **Nouveau tableau de bord**, puis **Tableau de bord vide**.

   ![Nouveau tableau de bord](media/tutorial-app-dashboards/new-dashboard.png)

3. Tapez un nom pour le tableau de bord.
4. Examinez la **Galerie de vignettes** pour voir l’éventail des vignettes que vous pouvez ajouter à votre tableau de bord.  Outre l’ajout de vignettes de la galerie, vous pouvez épingler directement des graphiques et d’autres affichages d’Application Insights au tableau de bord.
5. Recherchez la vignette **Markdown**, puis faites-la glisser sur votre tableau de bord.  Cette vignette permet d’ajouter du texte mis en forme dans markdown, ce qui est idéal pour l’ajout d’un texte descriptif à votre tableau de bord. Pour plus d’informations, consultez [Utiliser une vignette Markdown sur les tableaux de bord Azure pour afficher un contenu personnalisé](../../azure-portal/azure-portal-markdown-tile.md).
6. Ajoutez du texte au propriétés de la vignette, puis redimensionnez celle-ci sur le canevas du tableau de bord.

    [![Modifier une vignette markdown](media/tutorial-app-dashboards/markdown.png)](media/tutorial-app-dashboards/dashboard-edit-mode.png#lightbox)

7. Sélectionnez **Personnalisation terminée** en haut de l’écran pour quitter le mode de personnalisation des vignettes.

## <a name="add-health-overview"></a>Ajouter une vue d'ensemble de l'intégrité

Un tableau de bord contenant uniquement du texte statique n’est pas très attrayant. Ajoutez donc une vignette d’Application Insights pour présenter les informations relatives à votre application. Vous pouvez ajouter des vignettes d’Application Insights à partir de la Galerie de vignettes, ou les épingler directement à partir des écrans d’Application Insights. Cela vous permet de configurer des graphiques et affichages qui vous sont déjà familiers avant de les épingler à votre tableau de bord.  Commencez par ajouter la vue d'ensemble standard de l'intégrité pour votre application.  Cela ne nécessite aucune configuration et permet une personnalisation minimale du tableau de bord.


1. Sélectionnez votre ressource **Application Insights** dans l’écran d’accueil.
2. Dans le volet **Vue d’ensemble**, sélectionnez l’icône d’épingle ![Icône d’épingle](media/tutorial-app-dashboards/pushpin.png) pour ajouter la vignette à un tableau de bord.
3. Sous l’onglet « Épingler au tableau de bord », sélectionnez le tableau de bord auquel ajouter la vignette ou créez-en un nouveau.
 
3. En haut à droite, une notification s’affiche indiquant que votre vignette a été épinglée à votre tableau de bord.  Sélectionnez **Épinglé au tableau de bord** dans la notification pour revenir à votre tableau de bord ou utiliser le volet du tableau de bord.
4. Cette vignette est désormais ajoutée à votre tableau de bord. Sélectionnez **Modifier** pour modifier le positionnement de la vignette. Sélectionnez la vignette et faites-la glisser à sa position, puis sélectionnez **Personnalisation terminée**. Votre tableau de bord affiche à présent une mosaïque avec des informations utiles.

    [![Tableau de bord en mode Édition](media/tutorial-app-dashboards/dashboard-edit-mode.png)](media/tutorial-app-dashboards/dashboard-edit-mode.png#lightbox)

## <a name="add-custom-metric-chart"></a>Ajouter un graphique de métrique personnalisée

Le panneau **Métriques** vous permet de tracer le graphique d’une métrique collectée par Application Insights au fil du temps avec des filtres et un regroupement facultatifs.  Comme tout autre élément dans Application Insights, vous pouvez ajouter ce graphique au tableau de bord.  Cela nécessite au préalable un peu de personnalisation.

1. Sélectionnez votre ressource **Application Insights** dans l’écran d’accueil.
1. Sélectionnez **Métriques**.  
2. Un graphique vide a déjà été créé, et vous êtes invité à ajouter une métrique.  Ajoutez une métrique au graphique, et éventuellement un filtre et un regroupement.  L’exemple ci-dessous présente le nombre de demandes de serveur regroupées par réussite.  Cela produit une vue dynamique des requêtes ayant réussi ou échoué.

    [![Ajouter une métrique](media/tutorial-app-dashboards/metrics.png)](media/tutorial-app-dashboards/metrics.png#lightbox)

4. Sélectionnez **Épingler au tableau de bord** sur la droite.

3.  En haut à droite, une notification s’affiche indiquant que votre vignette a été épinglée à votre tableau de bord. Sélectionnez **Épinglé au tableau de bord** dans la notification pour revenir à votre tableau de bord ou utiliser l’onglet Tableau de bord.

4. Cette vignette est désormais ajoutée à votre tableau de bord. Sélectionnez **Modifier** pour modifier le positionnement de la vignette. Sélectionnez la vignette et faites-la glisser à sa position, puis sélectionnez **Personnalisation terminée**.

## <a name="add-logs-query"></a>Ajouter une requête de journaux

Les journaux Azure Application Insights utilisent un langage de requête riche permettant d’analyser toutes les données collectées par Application Insights. Tout comme aux graphiques et autres affichages, vous pouvez ajouter à votre tableau de bord la sortie d’une requête Logs.

1. Sélectionnez votre ressource **Application Insights** dans l’écran d’accueil.
2. Sélectionnez **Journaux** à gauche sous « Supervision » pour ouvrir l’onglet Journaux.
3. Tapez la requête suivante qui renvoie les 10 pages les plus demandées et le nombre de demandes :

    ``` Kusto
    requests
    | summarize count() by name
    | sort by count_ desc
    | take 10
    ```

4. Sélectionnez **Exécuter** pour vérifier les résultats de la requête.
5. Sélectionnez l’icône en forme d’épingle ![Icône d’épingle](media/tutorial-app-dashboards/pushpin.png) et sélectionnez le nom de votre tableau de bord.

5. Avant de revenir au tableau de bord, ajoutez une autre requête, mais affichez-la sous forme de graphique pour voir les différentes façons de visualiser une requête de journaux sur un tableau de bord. Commencez avec la requête suivante qui résume les 10 opérations avec le plus grand nombre d’exceptions.

    ``` Kusto
    exceptions
    | summarize count() by operation_Name
    | sort by count_ desc
    | take 10
    ```

6. Sélectionnez **Graphique**, puis choisissez l’option **Anneau** pour visualiser la sortie.

    [![Graphique en anneau avec la requête ci-dessus](media/tutorial-app-dashboards/logs-doughnut.png)](media/tutorial-app-dashboards/logs-doughnut.png#lightbox)

6. Sélectionnez l’icône en forme d’épingle ![Icône d’épingle](media/tutorial-app-dashboards/pushpin.png) en haut à droite pour épingler le graphique à votre tableau de bord, puis revenez à votre tableau de bord.
7. Les résultats des requêtes sont à présent ajoutés à votre tableau de bord au format que vous avez sélectionné. Sélectionnez chaque vignette et faites-la glisser à sa position, puis sélectionnez **Personnalisation terminée**.
8. Sélectionnez l’icône de crayon ![Icône de crayon](media/tutorial-app-dashboards/pencil.png) sur chaque titre afin de leur donner un titre descriptif.

## <a name="share-dashboard"></a>Partager le tableau de bord

1. En haut du tableau de bord, sélectionnez **Partager** pour publier vos modifications.
2. Vous pouvez éventuellement définir des utilisateurs spécifiques pouvant accéder au tableau de bord. Pour plus d’informations, consultez [Partager des tableaux de bord Azure en utilisant le contrôle d’accès en fonction du rôle Azure](../../azure-portal/azure-portal-dashboard-share-access.md).
3. Sélectionnez **Publier**.

## <a name="next-steps"></a>Étapes suivantes

À présent que vous avez appris à créer des tableaux de bord personnalisés, jetez un coup d’œil au reste de la documentation d’Application Insights, notamment à une étude de cas.

> [!div class="nextstepaction"]
> [Diagnostics profonds](../app/devops.md)
