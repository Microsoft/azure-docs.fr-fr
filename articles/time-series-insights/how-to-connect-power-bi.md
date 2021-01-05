---
title: Connecter votre environnement à Power BI – Azure Time Series Insights | Microsoft Docs
description: Découvrez comment connecter Azure Time Series Insights à Power BI pour partager, organiser sous la forme de graphique et afficher des données au sein de votre organisation.
author: deepakpalled
ms.author: dpalled
manager: diviso
services: time-series-insights
ms.service: time-series-insights
ms.topic: conceptual
ms.date: 10/01/2020
ms.openlocfilehash: 680b3c5a9548fa06d0139bd441b5583c27427a77
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95020774"
---
# <a name="visualize-data-from-azure-time-series-insights-in-power-bi"></a>Visualiser des données à partir d’Azure Time Series Insights dans Power BI

Azure Time Series Insights est une plateforme permettant de stocker, de gérer, d’interroger et de visualiser des données de série chronologique dans le cloud. [Power BI](https://powerbi.microsoft.com) est un outil d’analytique commercial avec des fonctionnalités de visualisation riches qui vous permet de partager des informations et des résultats au sein de votre organisation. Les deux services peuvent maintenant être intégrés pour obtenir le meilleur des deux fonctionnalités de visualisation inhérentes d’Azure Time Series Insights et Power BI.

Vous découvrirez comment effectuer les actions suivantes :

* Connecter Azure Time Series Insights à Power BI à l’aide du connecteur cloud
* Créer des visuels avec vos données dans Power BI
* Publier le rapport dans Power BI et le partager avec le reste de votre organisation

À la fin, vous apprendrez à visualiser des données de série chronologique par le biais d’Azure Time Series Insights et à les améliorer grâce à la visualisation robuste des données et aux capacités de partage facile de Power BI.

Veillez à vous inscrire pour obtenir un [abonnement Azure gratuit](https://azure.microsoft.com/free/) si vous n’en avez pas déjà un.

## <a name="prerequisites"></a>Prérequis

* Télécharger et installer la dernière version de [Power BI Desktop](https://powerbi.microsoft.com/downloads/)
* Avoir ou créer un environnement [Azure Time Series Insights Gen2](./how-to-provision-manage.md)

> [!IMPORTANT]
>
> * Le connecteur est actuellement pris en charge dans les environnements Azure Time Series Insights Gen2 configurés avec le **magasin Warm** uniquement.
> * Si vous disposez d’un accès invité à l’environnement Azure Time Series Insights Gen2 à partir d’un autre locataire Azure AD, vous ne pourrez pas accéder au connecteur. Apprenez-en davantage sur les [stratégies d’accès de l’environnement](./concepts-access-policies.md).

## <a name="connect-data-from-azure-time-series-insights-to-power-bi"></a>Connecter des données d’Azure Time Series Insights à Power BI

Pour connecter votre environnement Azure Time Series Insights à Power BI, effectuez les étapes suivantes :

1. Ouvrez l’Explorateur Azure Time Series Insights
1. Exporter des données sous forme de requête ou de données brutes
1. Ouvrir Power BI Desktop
1. Charger à partir d’une requête personnalisée

### <a name="export-data-into-power-bi-desktop"></a>Exporter des données dans Power BI Desktop

Pour commencer :

1. Ouvrez l’Explorateur Azure Time Series Insights et organisez vos données.
1. Une fois que vous avez créé un affichage dont vous êtes satisfait, accédez au menu déroulant **Plus d’actions**, puis sélectionnez **Se connecter à Power BI**.

    [![Exportation de l’Explorateur Azure Time Series Insights](media/how-to-connect-power-bi/time-series-insights-export-option.png)](media/how-to-connect-power-bi/time-series-insights-export-option.png#lightbox)

1. Définissez vos paramètres à l’intérieur de cet onglet :

   1. Spécifiez une période relative à afficher. Si vous êtes satisfait de votre affichage existant, laissez-le en tant que **Période existante**.

   1. Choisissez entre **Agrégées** et **Événements bruts**.

       > [!NOTE]
       > Vous pouvez toujours agréger vos données ultérieurement dans Power BI, mais vous ne pouvez pas rétablir les données brutes après l’agrégation.

       > [!NOTE]
       > Il existe une limite de nombre d’événements de 250 000 pour les données de niveau Événement brut.

       [![Connecter](media/how-to-connect-power-bi/connect-to-power-bi.png)](media/how-to-connect-power-bi/connect-to-power-bi.png#lightbox)

   1. Si vous n’avez pas configuré votre environnement Azure Time Series Insights avec le **magasin Warm**, vous recevrez un avertissement.

       [![Avertissement du magasin Warm](media/how-to-connect-power-bi/connect-to-power-bi-warning.png)](media/how-to-connect-power-bi/connect-to-power-bi-warning.png#lightbox)

       > [!TIP]
       > Vous pouvez configurer votre instance existante pour le **magasin Warm** dans le portail Azure.

1. Sélectionnez **Copier la requête dans le Presse-papiers**.
1. À présent, lancez Power BI Desktop.
1. Dans Power BI Desktop, sous l’onglet **Accueil**, sélectionnez **Obtenir les données** dans le coin supérieur droit, puis **Plus**.

    [![Liste déroulante Accueil](media/how-to-connect-power-bi/power-bi-home-drop-down.png)](media/how-to-connect-power-bi/power-bi-home-drop-down.png#lightbox)

1. Recherchez **Azure Time Series Insights**, sélectionnez **Azure Time Series Insights (bêta)** , puis **Se connecter**.

    [![Connecter Power BI à Azure Time Series Insights](media/how-to-connect-power-bi/connect-to-time-series-insights.png)](media/how-to-connect-power-bi/connect-to-time-series-insights.png#lightbox)

    Vous pouvez également accéder à l’onglet **Azure** et sélectionner **Azure Time Series Insights (bêta)** , puis **Connecter**.

1. Une boîte de dialogue de message s’affiche et vous demande l’autorisation de se connecter à des ressources tierces. Sélectionnez **Continuer**.

    [![Choisir ou créer une requête personnalisée](media/how-to-connect-power-bi/confirm-the-connection.png)](media/how-to-connect-power-bi/confirm-the-connection.png#lightbox)

1. Dans le menu déroulant, sous **Source de données**, choisissez **Créer une requête personnalisée**. Collez à partir du Presse-papiers dans le champ facultatif **Requête personnalisée (facultatif)** ci-dessous, puis appuyez sur **OK**.

    [![Transmettre la requête personnalisée et sélectionner OK](media/how-to-connect-power-bi/custom-query-load.png)](media/how-to-connect-power-bi/custom-query-load.png#lightbox)  

1. La table de données sera chargée. Appuyez sur **Charger** pour charger dans Power BI.

    [![Passer en revue les données chargées dans la table et sélectionner Charger](media/how-to-connect-power-bi/review-the-loaded-data-table.png)](media/how-to-connect-power-bi/review-the-loaded-data-table.png#lightbox)  

Si vous avez effectué ces étapes, passez directement à la section suivante.

## <a name="create-a-report-with-visuals"></a>Créer un rapport avec des visuels

Maintenant que vous avez importé les données dans Power BI, il est temps de créer un rapport avec des visuels.

1. Sur le côté gauche de la fenêtre, vérifiez que vous avez sélectionné l’affichage **Rapport**.

    [![Capture d’écran montrant l’icône d la vue Rapport.](media/how-to-connect-power-bi/select-the-report-view.png)](media/how-to-connect-power-bi/select-the-report-view.png#lightbox)

1. Dans la colonne **Visualisations**, sélectionnez le visuel de votre choix. Par exemple, sélectionnez **Graphique en courbes**. Cela permet d’ajouter un graphique en courbes vide à votre canevas.

1. Dans la liste **Champs**, sélectionnez **_Timestamp**, puis faites-le glisser vers le champ **Axe** pour afficher les éléments le long de l’axe X. Veillez à basculer sur **_Timestamp** pour la valeur de l’**axe** (la valeur par défaut est **Hiérarchie de date**).

    [![Capture d’écran montrant le menu Horodateur avec _Horodateur sélectionné.](media/how-to-connect-power-bi/select-timestamp.png)](media/how-to-connect-power-bi/select-timestamp.png#lightbox)

1. À nouveau, dans la liste **Champs**, sélectionnez **TimeSeriesId**, puis faites-le glisser vers le champ **Valeurs** pour afficher les éléments le long de l’axe Y.

    [![Créer un graphique en courbes](media/how-to-connect-power-bi/power-bi-line-chart.png)](media/how-to-connect-power-bi/power-bi-line-chart.png#lightbox)

1. Pour ajouter un autre graphique à votre canevas, sélectionnez n’importe où sur le canevas à l’extérieur du graphique en courbes, puis répétez ce processus.

    [![Créer des graphiques supplémentaires à partager](media/how-to-connect-power-bi/power-bi-additional-charts.png)](media/how-to-connect-power-bi/power-bi-additional-charts.png#lightbox)

Une fois que vous avez créé votre rapport, vous pouvez le publier sur Power BI Reporting Services.

## <a name="advanced-editing"></a>Modification avancée

Si vous avez déjà chargé un jeu de données dans Power BI, mais que vous souhaitez modifier la requête (par exemple, la date/heure ou les paramètres d’ID d’environnement), vous pouvez le faire via la fonctionnalité Éditeur avancé de Power BI. Pour en savoir plus, reportez-vous à la [documentation Power BI](/power-bi/desktop-query-overview).

En guise de vue d’ensemble :

1. Dans Power BI Desktop, sélectionnez **Modifier les requêtes**.
1. Appuyez sur **Éditeur avancé**.

    [![Modifier les requêtes dans l’Éditeur avancé](media/how-to-connect-power-bi/power-bi-advanced-query-editing.png)](media/how-to-connect-power-bi/power-bi-advanced-query-editing.png#lightbox)

1. Modifiez la charge utile JSON comme voulu.
1. Sélectionnez **Terminé** puis **Fermer et appliquer** dans la **Fenêtre Éditeur Power Query**.

L’interface reflète désormais les modifications que vous avez apportées.  

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [concepts de connecteur Power BI](/power-bi/desktop-query-overview) pour Azure Time Series Insights.

* En savoir plus sur [Power BI Desktop](/power-bi/desktop-query-overview).