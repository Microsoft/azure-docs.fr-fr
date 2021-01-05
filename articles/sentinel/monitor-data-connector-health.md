---
title: Surveiller l’intégrité de vos connecteurs de données avec ce classeur Azure Sentinel | Microsoft Docs
description: Utilisez le classeur Analyse du fonctionnement pour suivre la connectivité et les performances de vos connecteurs de données.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/09/2020
ms.author: yelevin
ms.openlocfilehash: 7d52b76601a617f62ae5b10fa38841ef2608bf49
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94656988"
---
# <a name="monitor-the-health-of-your-data-connectors-with-this-azure-sentinel-workbook"></a>Surveiller l’intégrité de vos connecteurs de données avec ce classeur Azure Sentinel

Le **classeur de surveillance de l’intégrité des connecteurs de données** vous permet de suivre l’intégrité, la connectivité et les performances de vos connecteurs de données à partir d’Azure Sentinel. Le classeur fournit des analyses supplémentaires, détecte les anomalies et donne un aperçu de l’état d’ingestion des données de l’espace de travail. Vous pouvez utiliser la logique du classeur pour surveiller l’intégrité générale des données ingérées, et créer des vues personnalisées et autres alertes basées sur des règles.

## <a name="use-the-health-monitoring-workbook"></a>Utiliser le classeur d’analyse du fonctionnement

1. Dans le portail Azure Sentinel, dans le menu **Gestion des menaces**, sélectionnez **Classeurs**.

1. Dans la galerie **Classeurs**, dans la barre de recherche, entrez *intégrité*, puis sélectionnez **Surveillance de l’intégrité de la collecte de données** dans les résultats.

1. Sélectionnez **Afficher le modèle** pour utiliser le classeur tel quel, ou **Enregistrer** pour créer une copie modifiable du classeur. Une fois la copie créée, sélectionnez **Afficher le classeur enregistré**.

1. Une fois dans le classeur, commencez par sélectionner l’**abonnement** et l’**espace de travail** que vous souhaitez afficher, puis définissez la valeur **TimeRange** pour filtrer les données en fonction de vos besoins. Utilisez le bouton bascule **Afficher l’aide** pour afficher une explication à l’emplacement du classeur.

    :::image type="content" source="media/monitor-data-connector-health/data-health-workbook-1.png" alt-text="Page d’arrivée du classeur de surveillance de l’intégrité du connecteur de données" lightbox="media/monitor-data-connector-health/data-health-workbook-1.png":::

Ce classeur comporte trois sections dotées d’onglets :

1. L’onglet **Vue d’ensemble** affiche l’état général de l’ingestion de données dans l’espace de travail sélectionné : mesures du volume, taux d’EPS et heure de réception du dernier journal.

1. L’onglet **Anomalies liées à la collecte de données** vous aide à détecter des anomalies dans le processus de collecte des données, par table et source de données. Chaque onglet présente des anomalies pour une table particulière (l'onglet **Général** contient une collection de tables). Les anomalies sont calculées à l’aide de la fonction **series_decompose_anomalies()** qui retourne un **score d’anomalie**. [Découvrez plus d’informations sur cette fonction](/azure/data-explorer/kusto/query/series-decompose-anomaliesfunction?WT.mc_id=Portal-fx). Définissez les paramètres suivants pour que la fonction les évalue :

    - **AnomaliesTimeRange** : Ce sélecteur d’heure s’applique uniquement à la vue des anomalies liées à la collecte de données.
    - **SampleInterval** : intervalle de temps pendant lequel les données sont échantillonnées dans l’intervalle de temps donné. Le score d’anomalie est calculé uniquement sur les données du dernier intervalle.
    - **PositiveAlertThreshold** : cette valeur définit le seuil du score d’anomalie positif. Elle accepte les valeurs décimales.
    - **NegativeAlertThreshold** : cette valeur définit le seuil du score d’anomalie négatif. Elle accepte les valeurs décimales.

        :::image type="content" source="media/monitor-data-connector-health/data-health-workbook-2.png" alt-text="Page des anomalies du classeur de surveillance de l’intégrité du connecteur de données" lightbox="media/monitor-data-connector-health/data-health-workbook-2.png":::

1. L’onglet **Informations de l’agent** affiche des informations sur l’intégrité des agents Log Analytics installés sur vos différents ordinateurs, qu’il s’agisse de machines virtuelles Azure, cloud, ou locales, ou d’un ordinateur physique. Vous pouvez surveiller les aspects suivants :

   - Emplacement système

   - État et latence de pulsation

   - Mémoire et espace disque disponibles

   - Opérations de l’agent

    Dans cette section, vous devez sélectionner l’onglet qui décrit l’environnement de vos ordinateurs. Choisissez l’onglet **Machines gérées par Azure** si vous souhaitez afficher uniquement les machines gérées par Azure Arc, ou l’onglet **Toutes les machines** pour afficher les machines tant gérées que non gérées pas Azure sur lesquelles l’agent Log Analytics est installé.

    :::image type="content" source="media/monitor-data-connector-health/data-health-workbook-3.png" alt-text="Page d’informations de l’agent du classeur d’analyse du fonctionnement du connecteur de données" lightbox="media/monitor-data-connector-health/data-health-workbook-3.png":::

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [intégrer vos données à Azure Sentinel](quickstart-onboard.md), [connecter des sources de données](connect-data-sources.md) et [obtenir une visibilité de vos données et des menaces potentielles](quickstart-get-visibility.md).