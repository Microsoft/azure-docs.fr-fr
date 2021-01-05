---
title: Comprendre vos clients dans Azure Application Insights | Microsoft Docs
description: Didacticiel sur l’utilisation d’Azure Application Insights pour comprendre comment les clients utilisent votre application.
ms.subservice: application-insights
ms.topic: tutorial
author: mrbullwinkle
ms.author: mbullwin
ms.date: 09/20/2017
ms.custom: mvc
ms.openlocfilehash: ce9bf6fe6d130cf8511db2b2351c0de01b753d81
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87318400"
---
# <a name="use-azure-application-insights-to-understand-how-customers-are-using-your-application"></a>Utiliser Azure Application Insights pour comprendre comment les clients utilisent votre application

Azure Application Insights collecte des informations d’utilisation pour vous aider à comprendre comment vos utilisateurs interagissent avec votre application.  Ce didacticiel vous présente les différentes ressources disponibles pour analyser ces informations.  Vous apprendrez à :

> [!div class="checklist"]
> * Analyser les détails sur les utilisateurs qui accèdent à votre application
> * Utiliser les Informations de session pour analyser la façon dont les clients utilisent votre application
> * Définir des entonnoirs qui vous permettent de comparer l’activité idéale d’un utilisateur et son activité réelle 
> * Créer un classeur pour consolider les visualisations et les requêtes dans un même document
> * Regrouper les utilisateurs similaires pour les analyser ensemble
> * Identifier les utilisateurs qui retournent dans votre application
> * Examiner la façon dont les utilisateurs naviguent dans votre application


## <a name="prerequisites"></a>Conditions préalables requises

Pour suivre ce tutoriel :

- Installez [Visual Studio 2019](https://www.visualstudio.com/downloads/) avec les charges de travail suivantes :
    - Développement web et ASP.NET
    - Développement Azure
- Téléchargez et installez le [débogueur d'instantané de Visual Studio](https://aka.ms/snapshotdebugger).
- Déployez une application .NET pour Azure et [activez le Kit SDK Application Insights](../app/asp-net.md). 
- [Envoyez des données de télémétrie à partir de votre application](../app/usage-overview.md#send-telemetry-from-your-app) pour ajouter des affichages d’événement/de page personnalisés.
- Envoyez un [contexte utilisateur](../app/usage-send-user-context.md) pour suivre les actions d’un utilisateur au fil du temps et exploiter pleinement les fonctionnalités d’utilisation.

## <a name="log-in-to-azure"></a>Connexion à Azure
Connectez-vous au portail Azure à l’adresse [https://portal.azure.com](https://portal.azure.com).

## <a name="get-information-about-your-users"></a>Obtenir des informations sur vos utilisateurs
Le panneau **Utilisateurs** vous permet de comprendre les détails importants sur vos utilisateurs de plusieurs façons. Vous pouvez utiliser ce panneau pour comprendre ces informations, par exemple l’emplacement à partir duquel vos utilisateurs se connectent, les détails de leurs clients, et les zones de votre application auxquelles ils accédent. 

1. Sélectionnez **Application Insights** choisissez votre abonnement.
2. Sélectionnez **Utilisateurs** dans le menu.
3. L’affichage par défaut indique le nombre d’utilisateurs uniques qui se sont connectés à votre application au cours des dernières 24 heures.  Vous pouvez modifier la fenêtre de temps et définir divers autres critères pour filtrer ces informations.

    ![Générateur de requêtes](media/tutorial-users/QueryBuilder.png)

6. Cliquez sur la liste déroulante **Pendant** et choisissez une fenêtre de temps de 7 jours.  Cette option augmente les données incluses dans les différents graphiques du panneau.

    ![Modifier l’intervalle de temps](media/tutorial-users/TimeRange.png)

4. Cliquez sur la liste déroulante **Fractionner par** pour ajouter une répartition par propriété d’utilisateur au graphique.  Sélectionnez un **pays ou une région**.  Le graphique inclut les mêmes données mais vous permet d’afficher une répartition du nombre d’utilisateurs pour chaque pays/région.

    ![Graphique Pays ou Région](media/tutorial-users/CountryorRegion.png)

5. Placez le curseur sur les différentes barres du graphique et notez que le nombre pour chaque pays/région correspond uniquement à la fenêtre de temps représentée par cette barre.
6. Examinez la colonne **Insights** à droite, qui analyse les données de vos utilisateurs.  Elle fournit des informations telles que le nombre de sessions uniques sur la période de temps ainsi que les enregistrements avec des propriétés communes qui composent principalement les données utilisateur 

    ![Colonne Insights](media/tutorial-users/insights.png)


## <a name="analyze-user-sessions"></a>Analyser les sessions utilisateur
Le panneau **Sessions** est similaire au panneau **Utilisateurs**.  **Utilisateurs** vous permet d’obtenir des détails sur les utilisateurs qui accèdent à votre application, tandis que **Sessions** vous aide à comprendre comment ces utilisateurs se servent de votre application.  

1. Sélectionnez **Sessions** dans le menu.
2. Examinez le graphique et notez que vous disposez des mêmes options pour filtrer et décomposer les données comme dans le panneau **Utilisateurs**.

    ![Générateur de requêtes de sessions](media/tutorial-users/SessionsBuilder.png)

3. Le volet **Échantillon de ces sessions** à droite répertorie les sessions qui incluent un grand nombre d’événements.  Il s’agit de sessions intéressantes à analyser.

    ![Échantillon de ces sessions](media/tutorial-users/SessionsSample.png)

4. Cliquez sur une des sessions pour afficher sa **Chronologie de la session**, qui affiche chaque action dans les sessions.  Cette chronologie peut vous aider à identifier des informations telles que les sessions contenant un grand nombre d’exceptions.

    ![Chronologie des sessions](media/tutorial-users/SessionsTimeline.png)

## <a name="group-together-similar-users"></a>Regrouper les utilisateurs similaires
Une **cohorte** est un ensemble de groupes d’utilisateurs qui partagent des caractéristiques similaires.  Vous pouvez utiliser des cohortes pour filtrer les données dans d’autres panneaux, ce qui vous permet d’analyser des groupes d’utilisateurs particuliers.  Par exemple, vous pouvez analyser uniquement les utilisateurs ayant effectué un achat.

1.  Sélectionnez **Cohortes** dans le menu.
2.  Cliquez sur **Nouveau** pour créer une cohorte.
3.  Sélectionnez la liste déroulante **Qui ont utilisé** puis choisissez une action.  Seuls les utilisateurs qui ont effectué cette action dans la fenêtre de temps du rapport seront inclus.

    ![Cohorte qui a effectué les actions spécifiées](media/tutorial-users/CohortsDropdown.png)

4.  Sélectionnez **Utilisateurs** dans le menu.
5.  Dans la liste déroulante **Afficher**, sélectionnez la cohorte que vous venez de créer.  Les données du graphique sont limitées à ces utilisateurs.

    ![Cohorte dans l’outil des utilisateurs](media/tutorial-users/UsersCohort.png)


## <a name="compare-desired-activity-to-reality"></a>Comparer l’activité souhaitée à la réalité
Alors que les panneaux précédents se concentrent sur ce que les utilisateurs ont fait avec votre application, les **entonnoirs** se focalisent sur ce que vous souhaitez que les utilisateurs fassent.  Un entonnoir représente un ensemble d’étapes dans votre application, avec le pourcentage d’utilisateurs qui passent d’une étape à l’autre.  Par exemple, vous pouvez créer un entonnoir qui calcule le pourcentage d’utilisateurs qui se connectent à votre application pour rechercher un produit.  Vous pouvez ainsi voir le pourcentage d’utilisateurs qui ajoutent ce produit à leur panier, puis le pourcentage de ceux qui ont effectué un achat.

1. Sélectionnez **Entonnoirs** dans le menu, puis cliquez sur **Nouveau**. 

    ![Capture d’écran montrant comment créer un tunnel.](media/tutorial-users/funnelsnew.png)

2. Tapez un **nom d’entonnoir**.
3. Créez un entonnoir avec au moins deux étapes en sélectionnant une action pour chaque étape.  La liste d’actions est générée à partir des données d’utilisation collectées par Application Insights.

    ![Capture d’écran montrant comment créer des étapes dans un nouveau tunnel.](media/tutorial-users/funnelsedit.png)

4. Cliquez sur **Enregistrer** pour enregistrer l’entonnoir, puis affichez les résultats.  La fenêtre à droite de l’entonnoir montre les événements les plus courants avant la première activité et après la dernière activité pour vous aider à comprendre les tendances des utilisateurs qui caractérisent la séquence particulière.

    ![Capture d’écran montrant les résultats d’événement d’un tunnel récemment créé.](media/tutorial-users/funnelsright.png)


## <a name="learn-which-customers-return"></a>Identifier les clients récurrents
L’option **Rétention** vous aide à identifier les utilisateurs qui retournent dans votre application.  

1. Sélectionnez **Rétention** dans le menu.
2. Par défaut, les informations analysées incluent les utilisateurs qui ont effectué une action, puis qui sont retournés dans l’application pour effectuer à nouveau une action.  Vous pouvez modifier ce filtre, par exemple pour inclure uniquement les utilisateurs qui y sont retournés après avoir effectué un achat.

    ![Capture d’écran montrant comment définir un filtre de rétention.](media/tutorial-users/retentionquery.png)

3. Les utilisateurs récurrents qui répondent aux critères apparaissent sous forme d’un graphique et d’un tableau pour différentes durées.  Le modèle par défaut montre une baisse progressive des utilisateurs récurrents au fil du temps.  Une baisse soudaine d’une période à une autre peut indiquer un problème. 

    ![Capture d’écran montrant un graphe pour les utilisateurs correspondant aux critères définis pour un filtre de rétention.](media/tutorial-users/retentiongraph.png)

## <a name="analyze-user-navigation"></a>Analyser la navigation de l’utilisateur
Un **flux utilisateur** permet de visualiser la façon dont vos utilisateurs naviguent entre les pages et les fonctionnalités de votre application.  Cela vous permet de répondre à certaines questions, par exemple à quel moment les utilisateurs quittent une page particulière, comment ils ferment votre application, et si des actions sont répétées régulièrement.

1.  Sélectionnez **Flux d’utilisateurs** dans le menu.
2.  Cliquez sur **Nouveau** pour créer un flux utilisateur, puis sur **Modifier** pour modifier ses détails.
3.  Augmentez l’**intervalle de temps** à 7 jours, puis sélectionnez un événement initial.  Le flux effectuera le suivi des sessions utilisateur qui commencent par cet événement.

    ![Capture d’écran montrant comment créer un flux d’utilisateur.](media/tutorial-users/flowsedit.png)

4.  Le flux d’utilisateur s’affiche et montre les différents chemins d’accès des utilisateurs et leur nombre de sessions.  Les lignes bleues indiquent une action effectuée par l’utilisateur après l’action en cours.  Une ligne rouge indique la fin de la session de l’utilisateur.

    ![Capture d’écran montrant l’affichage de chemins d’accès d’utilisateur et de nombres de sessions pour un flux d’utilisateur.](media/tutorial-users/flows.png)

5.  Pour supprimer un événement du flux, cliquez sur le signe **x** dans l’angle de l’action, puis sur **Créer un graphique**.  Le graphique est redessiné et toutes les instances de cet événement sont supprimées.  Cliquez sur **Modifier** pour constater que l’événement est désormais ajouté à la liste des **événements exclus**.

    ![Capture d’écran montrant la liste d’événements exclus pour un flux d’utilisateur.](media/tutorial-users/flowsexclude.png)

## <a name="consolidate-usage-data"></a>Consolider les données d’utilisation
Les **classeurs** associent des visualisations de données, des requêtes Analytics et du texte dans des documents interactifs.  Vous pouvez utiliser des classeurs pour regrouper les informations d’utilisation communes, consolider les informations d’un incident spécifique, ou fournir à votre équipe des détails sur l’utilisation de votre application.

1.  Sélectionnez **Classeurs** dans le menu.
2.  Cliquez sur **Nouveau** pour créer un classeur.
3.  Une requête est déjà fournie et inclut toutes les données d’utilisation du dernier jour sous la forme d’un graphique à barres.  Vous pouvez utiliser cette requête, la modifier manuellement, ou cliquer sur **Exemples de requêtes** pour sélectionner d’autres requêtes utiles.

    ![Capture d’écran montrant une liste d’exemples de requêtes que vous pouvez utiliser.](media/tutorial-users/samplequeries.png)

4.  Cliquez sur **Fin de l'édition**.
5.  Cliquez sur **Modifier** dans le volet supérieur pour modifier le texte en haut du classeur.  Le texte est mis en forme à l’aide de markdown.

    ![Capture d’écran montrant comment modifier le texte en haut du classeur.](media/tutorial-users/markdown.png)

6.  Cliquez sur **Ajouter des utilisateurs** pour ajouter un graphique avec des informations sur les utilisateurs.  Modifiez les détails du graphique si vous le souhaitez, puis cliquez sur **Fin de l'édition** pour enregistrer le graphique.


## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez appris à analyser vos utilisateurs, passez à l’étape suivante du didacticiel pour apprendre à créer des tableaux de bord personnalisés qui combinent ces informations avec d’autres données utiles concernant votre application.

> [!div class="nextstepaction"]
> [Créer des tableaux de bord personnalisés](./tutorial-app-dashboards.md)

