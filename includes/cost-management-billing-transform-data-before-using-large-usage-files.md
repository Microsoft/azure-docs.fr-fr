---
title: Fichier include
description: Fichier include
author: bandersmsft
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: include
ms.date: 10/09/2020
ms.author: banders
ms.reviwer: ''
ms.custom: include file
ms.openlocfilehash: fb34b4ef1cf209a15c7e6ac9a59cc11dd4b0056e
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92026325"
---
## <a name="transform-data-before-using-large-usage-files"></a>Transformer les données avant d’utiliser des grands fichiers d’utilisation

Parfois, votre fichier d’utilisation ou de réconciliation est trop volumineux pour être ouvert. Ou bien, vous pouvez avoir besoin d’une partie seulement de ces informations pour résoudre un problème. Par exemple, vous voulez peut-être uniquement les informations relatives à une ressource particulière ou uniquement la consommation de quelques services ou groupes de ressources. Vous pouvez transformer les données pour les synthétiser avant de créer des tableaux croisés dynamiques.

1. Ouvrez un classeur vide dans Excel.
1. Dans le menu supérieur, sélectionnez **Données** > **À partir d’un fichier texte/CSV**, sélectionnez votre fichier d’utilisation, puis sélectionnez **Importer**.
1. En bas de la fenêtre, sélectionnez **Transformer les données**. Une nouvelle fenêtre affiche un résumé des données.  
    :::image type="content" source="./media/cost-management-billing-transform-data-before-using-large-usage-files/summarized-data.png" alt-text="Exemple montrant des données résumées" lightbox="./media/cost-management-billing-transform-data-before-using-large-usage-files/summarized-data.png" :::
1. Si vous avez un contrat client Microsoft, ignorez cette étape et passez à la suivante, car les fichiers d’utilisation MCA ont généralement les titres des colonnes dans les premières lignes. Préparez les données en créant la table. Supprimez les lignes du haut, en laissant uniquement les titres. Sélectionnez **Supprimer des lignes** > **Supprimer les lignes du haut**.  
     :::image type="content" source="./media/cost-management-billing-transform-data-before-using-large-usage-files/remove-top-rows.png" alt-text="Exemple montrant des données résumées" :::
1. Dans la fenêtre Supprimer les lignes du haut, entrez le nombre de lignes à supprimer en haut. Pour EA, généralement 2, pour CSP, généralement 1. Sélectionnez **OK**.
1. Sélectionnez **Utiliser la première ligne pour les en-têtes**.  
    :::image type="content" source="./media/cost-management-billing-transform-data-before-using-large-usage-files/use-first-row-as-header.png" alt-text="Exemple montrant des données résumées" :::
    
    La vue table affiche les titres des colonnes en haut.
1. Ensuite, ajoutez un filtre. Utilisez les flèches de sélection à droite de chaque titre de colonne à filtrer. Les filtres suggérés sont ID d’abonnement, Nom du service (catégorie du compteur), ID d’instance et Groupe de ressources. Vous pouvez utiliser plusieurs filtres dans le même document. Nous vous recommandons d’appliquer tous les filtres possibles pour réduire la taille du document et faciliter le travail ultérieur.
1. Une fois que vous avez appliqué vos filtres, sélectionnez **Fermer et charger**.  
    :::image type="content" source="./media/cost-management-billing-transform-data-before-using-large-usage-files/close-and-load.png" alt-text="Exemple montrant des données résumées" :::

Le fichier se charge et montre une table avec les données d’utilisation filtrées. Vous pouvez maintenant créer un tableau croisé dynamique pour résoudre les problèmes d’utilisation.