---
title: Créer un cluster Azure Stream Analytics - Démarrage rapide
description: Découvrez comment créer un cluster Azure Stream Analytics.
author: sidramadoss
ms.author: sidram
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: overview
ms.custom: mvc
ms.date: 09/22/2020
ms.openlocfilehash: a7be204c30d242be991fb9a57d239b69342ace97
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "90944337"
---
# <a name="quickstart-create-a-dedicated-azure-stream-analytics-cluster-using-azure-portal"></a>Démarrage rapide : Créer un cluster dédié Azure Stream Analytics à l’aide du portail Azure

Utilisez le portail Azure pour créer un cluster Azure Stream Analytics. Un [cluster Stream Analytics cluster](cluster-overview.md) permet de déployer un seul locataire pour les cas d’usage de streaming complexes et exigeants. Vous pouvez exécuter plusieurs travaux Stream Analytics sur un même cluster Stream Analytics.

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Exécution du [Démarrage rapide : Créer un travail Stream Analytics à l’aide du portail Azure](stream-analytics-quick-create-portal.md).

## <a name="create-a-stream-analytics-cluster"></a>Créer un cluster Stream Analytics

Dans cette section, vous créez une ressource de cluster Stream Analytics.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez **Créer une ressource**. Dans la zone de recherche *Rechercher sur la Place de marché*, entrez et sélectionnez **Cluster Stream Analytics**. Sélectionnez ensuite **Ajouter**.

   :::image type="content" source="./media/create-cluster/search-result.png" alt-text="Résultat de la recherche de cluster Stream Analytics.":::

1. Sur la page **Créer un cluster Stream Analytics**, entrez les paramètres de base de votre nouveau cluster.

   |Paramètre|Valeur|Description |
   |---|---|---|
   |Abonnement|Nom d’abonnement|Sélectionnez l’abonnement Azure que vous souhaitez utiliser pour ce cluster Stream Analytics. |
   |Groupe de ressources|Nom de groupe ressources|Sélectionnez un groupe de ressources ou sélectionnez **Créer**, puis entrez un nom unique pour le nouveau groupe de ressources. |
   |Nom du cluster|Un nom unique|Entrez un nom pour identifier votre cluster Stream Analytics.|
   |Emplacement|Région la plus proche de vos sources de données et récepteurs|Sélectionnez l’emplacement géographique où héberger votre cluster Stream Analytics. Utilisez l’emplacement le plus proche de vos sources de données et récepteurs à des fins d’analyse à faible latence.|
   |Capacité d'unités de streaming|36 à 216 |Déterminez la taille du cluster en estimant le nombre de travaux Stream Analytics que vous envisagez d’exécuter et le nombre total de SU requis par le travail. Vous pouvez commencer par 36 SU, puis augmenter ou diminuer la taille des unités en fonction de vos besoins.|

   ![Créer un cluster](./media/create-cluster/create-cluster.png)

1. Sélectionnez **Revoir + créer**. Vous pouvez ignorer les sections **Balises**.

1. Passez en revue les paramètres du cluster, puis sélectionnez **Créer**. La création du cluster est une opération de longue durée qui peut prendre environ 60 minutes. Attendez que la page du portail affiche **Votre déploiement est terminé**. En attendant, vous pouvez créer et développer les [travaux Stream Analytics](stream-analytics-quick-create-portal.md#create-a-stream-analytics-job) que vous souhaitez exécuter sur ce cluster, si vous ne l’avez pas déjà fait.

1. Sélectionnez **Accéder à la ressource** pour accéder à la page du cluster Stream Analytics.

## <a name="delete-your-cluster"></a>Supprimer votre cluster

Vous pouvez supprimer votre cluster Stream Analytics si vous n’envisagez pas d’y exécuter des travaux Stream Analytics. Pour supprimer votre cluster, suivez les étapes ci-dessous dans le portail Azure :

1. Accédez à **Travaux Stream Analytics** sous **Paramètres** et arrêtez tous les travaux en cours d’exécution.

1. Accédez à la **Vue d’ensemble** de votre cluster. Sélectionnez **Supprimer** et suivez les instructions pour supprimer votre cluster.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez découvert comment créer un cluster Stream Analytics. Passez à l’article suivant pour apprendre à exécuter un travail Stream Analytics sur votre cluster :

> [!div class="nextstepaction"]
> [Gérer des travaux Stream Analytics dans un cluster Stream Analytics](manage-jobs-cluster.md)
