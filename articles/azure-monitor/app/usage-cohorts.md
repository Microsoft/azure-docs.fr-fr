---
title: Cohortes d’utilisation Azure Application Insights | Microsoft Docs
description: Analyser différents jeux d’utilisateurs, sessions, événements ou opérations qui partagent un point commun
ms.topic: conceptual
author: NumberByColors
ms.author: daviste
ms.date: 04/10/2018
ms.reviewer: mbullwin
ms.openlocfilehash: f8edeb452f01af88f0305cd0a3e3a2332ec09e4e
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186947"
---
# <a name="application-insights-cohorts"></a>Cohortes Application Insights

Une cohorte est un ensemble d’utilisateurs, de sessions, d’événements ou d’opérations qui partagent un point commun. Dans Azure Application Insights, les cohortes sont définies par une requête analytique. Si vous analysez un ensemble d’utilisateurs ou d’événements à plusieurs reprises, les cohortes vous offrent davantage de souplesse pour exprimer précisément ce qui vous intéresse.

![Volet des cohortes](./media/usage-cohorts/001.png)

## <a name="cohorts-versus-basic-filters"></a>Cohortes ou filtres de base

L’utilisation des cohortes est similaire à celle des filtres. Toutefois, les définitions des cohortes sont créées à partir de requêtes analytiques personnalisées, pour une plus grande souplesse et une plus grande complexité. Contrairement aux filtres, vous pouvez enregistrer des cohortes afin d’autres membres de votre équipe puissent les réutiliser.

Vous pouvez définir une cohorte pour les utilisateurs qui ont testé une nouvelle fonctionnalité de votre application. Vous pouvez enregistrer cette cohorte dans votre ressource Application Insights. Il vous sera ensuite facile d’analyser ce groupe d’utilisateurs enregistré.

> [!NOTE]
> Une fois créées, les cohortes sont disponibles dans les outils Utilisateurs, Sessions, Événements et Flux d’utilisateurs.

## <a name="example-engaged-users"></a>Exemple : Utilisateurs engagés

Votre équipe définit un utilisateur engagé comme une personne qui utilise votre application au moins cinq fois pendant un mois donné. Dans cette section, vous allez définir une cohorte pour ces utilisateurs engagés.

1. Ouvrez l’outil Cohortes.

2. Sélectionnez l’onglet **Galerie des modèles**. Vous y trouverez des modèles pour différentes cohortes.

3. Sélectionnez **Engaged Users – by Days Used** (Utilisateurs engagés – par jours utilisés).

    Cette cohorte a trois paramètres :
    * **Activities** (Activités) : permet de choisir les événements et les pages consultées qui doivent être comptabilisés dans l’utilisation.
    * **Period** (Période) : définition d’un mois.
    * **UsedAtleastCustom** (Nombre d’utilisations) : nombre de fois qu’un utilisateur doit se servir d’un même élément au cours d’une période pour être considéré comme un utilisateur engagé.

4. Sélectionnez **5+ days** (+ de 5 jours) dans **UsedAtLeastCustom** et conservez la valeur par défaut de 28 jours dans **Period** (Période).

    ![Utilisateurs engagés](./media/usage-cohorts/003.png)

    Maintenant, cette cohorte représente tous les ID d’utilisateur qui ont été envoyés avec un événement personnalisé ou une page consultée sur cinq jours différents, au cours des 28 derniers jours.

5. Sélectionnez **Enregistrer**.

   > [!TIP]
   > Attribuez un nom à votre cohorte, tel que « Utilisateurs engagés (+ de 5 jours) ». Enregistrez-la dans « Mes rapports » ou « Rapports partagés », selon que vous souhaitiez ou non autoriser d’autres personnes qui ont accès à cette ressource Application Insights, à voir cette cohorte.

6. Sélectionnez **Back to Gallery** (Retour à la galerie).

### <a name="what-can-you-do-by-using-this-cohort"></a>Que pouvez-vous faire avec cette cohorte ?

Ouvrez l’outil Utilisateurs. Dans le menu déroulant **Afficher**, choisissez la cohorte que vous avez créée sous **Users who belong to…** (Utilisateurs appartenant à...).

Maintenant, l’outil Utilisateurs est filtré sur cette cohorte d’utilisateurs :

![Volet Utilisateurs filtré sur une cohorte particulière](./media/usage-cohorts/004.png)

Voici quelques points importants à prendre en considération :

* Vous ne pouvez pas créer cet ensemble à l’aide de filtres normaux. La logique de date est plus perfectionnée.
* Vous pouvez filtrer davantage cette cohorte à l’aide des filtres normaux dans l’outil Utilisateurs. Par conséquent, même si la cohorte est définie sur des périodes de 28 jours, vous pouvez régler l’intervalle sur 30, 60 ou 90 jours dans l’outil Utilisateurs.

Ces filtres prennent en charge des questions plus sophistiquées qui sont impossibles à exprimer via le Générateur de requêtes. Par exemple _personnes engagées au cours des 28 derniers jours. Quel a été le comportement de ces personnes au cours des 60 derniers jours ?_

## <a name="example-events-cohort"></a>Exemple : Cohorte d’événements

Vous pouvez également créer des cohortes d’événements. Dans cette section, vous allez définir une cohorte pour ces événements et pour ces pages consultées. Ensuite, vous allez voir comment les utiliser dans d’autres outils. Cette cohorte peut définir un ensemble d’événements que votre équipe considère comme une _utilisation active_, ou un ensemble d’événements liés à une nouvelle fonctionnalité.

1. Ouvrez l’outil Cohortes.

2. Sélectionnez l’onglet **Galerie des modèles**. Vous y trouverez des modèles pour différentes cohortes.

3. Sélectionnez **Events Picker** (Sélecteur d’événements).

    ![Capture d’écran d’Events Picker](./media/usage-cohorts/006.png)

4. Dans le menu déroulant **Activities** (Activités), sélectionnez les événements que vous souhaitez inclure dans la cohorte.

5. Enregistrez la cohorte et attribuez-lui un nom.

## <a name="example-active-users-where-you-modify-a-query"></a>Exemple : utilisateurs actifs dont vous modifiez la requête

Les deux cohortes précédentes ont été définies à l’aide de zones de liste déroulante. Toutefois, pour une flexibilité totale, vous pouvez également définir des cohortes avec des requêtes analytiques. Voyons comment faire en créant une cohorte d’utilisateurs du Royaume-Uni.

![Présentation animée de l’utilisation de l’outil Cohorts](./media/usage-cohorts/cohorts0001.gif)

1. Ouvrez l’outil Cohortes, sélectionnez l’onglet **Galerie des modèles**, puis sélectionnez **Cohorte d’utilisateurs vide**.

    ![Cohorte d’utilisateurs vide](./media/usage-cohorts/001.png)

    Les sections sont au nombre de trois :
   * Une section de texte Markdown, où vous pouvez décrire la cohorte plus en détail pour les membres de votre équipe.

   * Une section de paramètres où vous créez vos propres paramètres, tels que les **activités** et les autres menus déroulants des deux exemples précédents.

   * Une section de requête que vous utilisez pour définir la cohorte à l’aide d’une requête analytique.

     Dans la section de requête, vous [écrivez une requête analytique](/azure/kusto/query). La requête sélectionne les lignes qui décrivent la cohorte que vous souhaitez définir. Ensuite, l’outil Cohorts ajoute implicitement une clause « | summarize by user_Id » à la requête. Ces données s’affichent sous la requête dans un tableau. Ainsi, vous pouvez être sûr que votre requête retourne des résultats.

     > [!NOTE]
     > Si la requête n’apparaît pas, essayez de redimensionner la section pour l’agrandir et afficher la requête. Le fichier GIF animé, au début de cette section, illustre le comportement de redimensionnement.

2. Copiez et collez le texte suivant dans l’éditeur de texte :

    ```KQL
    union customEvents, pageViews
    | where client_CountryOrRegion == "United Kingdom"
    ```

3. Sélectionnez **Run Query** (Exécuter la requête). Si vous ne voyez pas d’ID d’utilisateurs dans le tableau, passez à un pays/une région où votre application a des utilisateurs.

4. Enregistrez la cohorte et attribuez-lui un nom.

## <a name="frequently-asked-questions"></a>Forum aux questions

_J’ai défini une cohorte d’utilisateurs d’un pays/d’une région. Lorsque je la compare dans l’outil Utilisateurs à la définition d’un filtre sur ce pays/cette région, j’obtiens des résultats différents. Pourquoi ?_

Les cohortes et les filtres sont différents. Supposons que vous ayez une cohorte d’utilisateurs du Royaume-Uni (définie comme dans l’exemple précédent) et que vous compariez ses résultats à ceux obtenus avec le filtre « Country or region = United Kingdom » (Pays ou région = Royaume-Uni).

* La version cohorte affiche tous les événements des utilisateurs qui ont envoyé au moins un événement au Royaume-Uni pendant la période. Si vous fractionnez les données par pays ou région, vous verrez probablement de nombreux pays et régions.
* La version filtre n’affiche que les événements du Royaume-Uni. Toutefois, si vous fractionnez les données par pays ou région, vous ne voyez que celles du Royaume-Uni.

## <a name="learn-more"></a>En savoir plus

* [Langage de requête Analytics](../log-query/log-analytics-tutorial.md?toc=%2fazure%2fazure-monitor%2ftoc.json)
* [Utilisateurs, sessions, événements](usage-segmentation.md)
* [Flux d’utilisateurs](usage-flows.md)
* [Vue d’ensemble de l’utilisation](usage-overview.md)