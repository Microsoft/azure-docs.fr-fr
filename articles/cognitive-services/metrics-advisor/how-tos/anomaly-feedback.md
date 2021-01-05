---
title: Envoi de commentaires sur les anomalies au service Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Découvrez comment envoyer des commentaires sur les anomalies détectées par votre instance Metrics Advisor et ajuster les résultats.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 11/24/2020
ms.author: mbullwin
ms.openlocfilehash: 96a6dfebdd0047d5a6f4ff3e295ab96dd8c7226a
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184855"
---
# <a name="provide-anomaly-feedback"></a>Fournir des commentaires sur les anomalies

Les commentaires des utilisateurs sont l’une des principales méthodes de détection de défauts au sein du système de détection des anomalies. Ici, nous fournissons aux utilisateurs un moyen de marquer des résultats de détection incorrects directement sur une série chronologique, et d’appliquer les commentaires immédiatement. Un utilisateur peut ainsi enseigner au système de détection des anomalies comment détecter des anomalies pour une série chronologique spécifique via des interactions actives. 

> [!NOTE]
> Actuellement, les commentaires n’affectent les résultats de la détection des anomalies que par **Détection intelligente**, pas par **Seuil définitif** et **Seuil de modification**.

## <a name="how-to-give-time-series-feedback"></a>Comment fournir des commentaires sur une série chronologique

Vous pouvez fournir des commentaires à partir de la page de détails des métriques de n’importe quelle série. La simple sélection d’un point a pour effet d’afficher la boîte de dialogue de commentaires ci-dessous. Elle présente les dimensions de la série que vous avez choisie. Vous pouvez resélectionner des valeurs de dimension, voire en supprimer certaines pour obtenir un lot de données de série chronologique. Une fois la série chronologique choisie, sélectionnez le bouton **Ajouter** pour ajouter un commentaire. Vous pouvez formuler des commentaires de quatre types. Pour ajouter plusieurs éléments de commentaires, sélectionnez le bouton **Enregistrer** après avoir entré chacune de vos annotations.

:::image type="content" source="../media/feedback/click-on-any-point.png" alt-text="Graphique présentant des données de série chronologique sur une ligne bleue avec différents points rouges. Cadre rouge entourant un point avec le texte : Sélectionnez un point":::

:::image type="content" source="../media/feedback/select-or-remove.png" alt-text="Boîte de dialogue Ajouter des commentaires avec deux dimensions et l’option permettant de sélectionner ou de supprimer des dimensions et d’ajouter des commentaires.":::

### <a name="mark-the-anomaly-point-type"></a>Marquer le type de point d’anomalie

Comme le montre l’image ci-dessous, la boîte de dialogue des commentaires renseigne automatiquement l’horodatage du point choisi, mais vous pouvez modifier cette valeur. Vous indiquez ensuite si vous souhaitez identifier cet élément en tant que `Anomaly`, `NotAnomaly` ou `AutoDetect`.

:::image type="content" source="../media/feedback/anomaly-value.png" alt-text="Menu déroulant avec les options Anomaly, NotAnomaly et AutoDetect":::

La sélection appliquera vos commentaires au traitement de la détection des anomalies ultérieures de la même série. Les points traités ne seront pas recalculés. Cela signifie que, si vous avez marqué une anomalie en tant que NotAnomaly, nous supprimerons les anomalies similaires à l’avenir et que, si vous avez marqué un point `NotAnomaly` comme `Anomaly`, nous aurons tendance à détecter des points similaires en tant que `Anomaly` à l’avenir. Si vous choisissez `AutoDetect`, les commentaires précédents sur le même point seront ignorés à l’avenir.

## <a name="provide-feedback-for-multiple-continuous-points"></a>Formuler des commentaires pour plusieurs points continus 

Si vous souhaitez commenter une anomalie sur plusieurs points continus à la fois, sélectionnez le groupe de points que vous voulez annoter. L’intervalle de temps choisi s’affiche automatiquement lorsque vous formulez un commentaire sur une anomalie.

:::image type="content" source="../media/feedback/continuous-anomaly-feedback.png" alt-text="Menu de commentaires avec une anomalie sélectionnée et un intervalle de temps spécifique":::

Pour voir si un point individuel est affecté par votre commentaire d’anomalie, lorsque vous parcourez une série chronologique, sélectionnez un point. Si son résultat de détection d’anomalie a été modifié par le commentaire, l’info-bulle affiche **Affecté par les commentaires : true**. Si elle affiche **Affecté par les commentaires : false**, cela signifie qu’un calcul de commentaire d’anomalie a été effectué pour ce point, mais que le résultat de la détection d’anomalie ne doit pas être modifié.

:::image type="content" source="../media/feedback/affected-by-feedback.png" alt-text="Affichage d’info-bulle avec les mots : Affecté par les commentaires : true, en surbrillance en rouge":::

Dans certains cas, nous ne suggérons pas de formuler des commentaires :

- L’anomalie est due à un jour de congé. Il est recommandé d’utiliser un événement prédéfini pour résoudre ce genre de fausse alarme, car ce sera plus précis.
- L’anomalie est due à une modification de source de données connue. Par exemple, une modification système en amont s’est produite à ce moment-là. Dans ce cas, il est prévu de déclencher une alerte d’anomalie, car notre système ignore la cause de la modification de valeur et quand d’autres changements similaires se reproduiront. Par conséquent, nous ne suggérons pas d’annoter ce genre de problème en tant que `NotAnomaly`.

## <a name="change-points"></a>Points de changement

Parfois, le changement de tendance des données affecte les résultats de la détection d’anomalies. Quand la décision est prise de considérer un point comme étant ou non une anomalie, la dernière fenêtre de données d’historique est prise en considération. Lorsque votre série chronologique présente un changement de tendance, vous pouvez marquer le point de modification exact. Cela aidera notre détecteur d’anomalies dans le cadre d’analyses ultérieures.

Comme le montre la figure ci-dessous, vous pouvez sélectionner `ChangePoint` comme type de commentaire, puis `ChangePoint`, `NotChangePoint` ou `AutoDetect` dans la liste déroulante.

:::image type="content" source="../media/feedback/changepoint.png" alt-text="Menu Point de changement avec une liste déroulante contenant les options ChangePoint, NotChangePoint et AutoDetect":::

> [!NOTE]
> Si vos données continuent de changer, vous ne devez marquer qu’un seul point en tant que `ChangePoint`. Par conséquent, si vous avez marqué un `timerange`, nous remplissons automatiquement l’horodatage et l’heure du dernier point. Dans ce cas, votre annotation n’affecte les résultats de détection des anomalies qu’après 12 points.

## <a name="seasonality"></a>Caractère saisonnier

Pour les données saisonnières, lorsque nous procédons à la détection d’anomalies, une étape consiste à estimer la période (caractère saisonnier) de la série chronologique, et à l’appliquer à la phase de détection des anomalies. Parfois, il est difficile d’identifier une période précise, et la période peut changer. Une période mal définie peut avoir des effets collatéraux sur vos résultats de détection d’anomalies. Vous pouvez rechercher la période actuelle à partir d’une info-bulle. Son nom est `Min Period`.

:::image type="content" source="../media/feedback/min-period.png" alt-text="Superposition d’info-bulle avec les mots Min Period et le chiffre sept encadrés en rouge.":::

Vous pouvez fournir un commentaire pour la période afin de corriger ce type d’erreur de détection d’anomalie. Comme le montre la figure, vous pouvez définir une valeur de période. L’unité `interval` signifie une granularité. Zéro intervalle signifie ici que les données ne sont pas saisonnières. Vous pouvez également sélectionner `AutoDetect` si vous souhaitez annuler les commentaires précédents et laisser le pipeline détecter automatiquement la période. 
 
> [!NOTE]
> Lors de la définition de la période, vous n’avez pas besoin d’affecter un horodatage ou une plage horaire. La période affecte les futures détections d’anomalies sur l’ensemble de la chronologie à partir du moment où vous envoyez des commentaires.


:::image type="content" source="../media/feedback/period-feedback.png" alt-text="Menu avec période de détection automatique définie sur 28 et un nombre d’intervalles défini sur zéro, indiquant l’absence de caractère saisonnier.":::

## <a name="provide-comment-feedback"></a>Fournir des commentaires

Vous pouvez également ajouter des commentaires pour annoter et contextualiser vos données. Pour ajouter des commentaires, sélectionnez un intervalle de temps et ajoutez le texte de votre commentaire.

:::image type="content" source="../media/feedback/feedback-comment.png" alt-text="Menu offrant la possibilité de définir un intervalle de temps et comportant une zone pour ajouter un commentaire textuel":::

## <a name="time-series-batch-feedback"></a>Commentaire de lot de séries chronologiques

Comme décrit précédemment, la fenêtre modale de commentaire vous permet de resélectionner ou de supprimer des valeurs de dimension pour obtenir un lot de séries chronologiques défini par un filtre de dimension. Vous pouvez également ouvrir cette fenêtre modale en cliquant sur le bouton « + » pour Commentaires dans le volet gauche, puis sélectionner des dimensions et leurs valeurs.

:::image type="content" source="../media/feedback/feedback-time-series.png" alt-text="Menu avec un signe plus bleu mis en évidence en rouge à côté du mot Commentaires":::

:::image type="content" source="../media/feedback/feedback-dimension.png" alt-text="Menu Ajouter un commentaire avec deux dimensions indiquées par Dim1 et Dim2":::

## <a name="how-to-view-feedback-history"></a>Comment afficher un historique de commentaires

Il existe deux façons d’afficher un historique de commentaires. Vous pouvez sélectionner le bouton de l’historique des commentaires dans le volet gauche pour afficher une fenêtre modale de liste de commentaires. Celle-ci répertorie tous les commentaires que vous avez formulés précédemment, soit pour une série unique, soit pour des filtres de dimension.

:::image type="content" source="../media/feedback/feedback-history-options.png" alt-text="Menu de la liste de commentaires":::

Vous pouvez également consulter l’historique des commentaires d’une série. Plusieurs boutons figurent dans l’angle supérieur droit de chaque série. Sélectionnez le bouton d’affichage des commentaires. La ligne passe de l’affichage des points d’anomalies à l’affichage des entrées de commentaires. L’indicateur vert représente un point de modification, et les points bleus d’autres points de commentaires. Vous pouvez également les sélectionner pour obtenir une fenêtre modale de liste de commentaires répertoriant les détails des commentaires donnés pour ce point.

:::image type="content" source="../media/feedback/feedback-history-graph.png" alt-text="Graphique de l’historique des commentaires":::

:::image type="content" source="../media/feedback/feedback-list.png" alt-text="Menu de liste de commentaires avec deux dimensions":::

> [!NOTE]
> Toute personne ayant accès à la métrique est autorisée à fournir des commentaires, ce qui vous permet de voir les commentaires formulés par d’autres propriétaires de flux de données. Si vous modifiez le même point que quelqu’un d’autre, votre commentaire remplace l’entrée de commentaire précédente.       

## <a name="next-steps"></a>Étapes suivantes
- [Diagnostic d’un incident](diagnose-incident.md)
- [Configuration des métriques et affinage de la configuration de la détection](configure-metrics.md)
- [Configuration des alertes et réception de notifications à l’aide d’un hook](../how-tos/alerts.md)