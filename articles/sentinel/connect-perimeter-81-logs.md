---
title: Connecter des données Perimeter 81 à Azure Sentinel | Microsoft Docs
description: Découvrez comment connecter des données Perimeter 81 à Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/21/2020
ms.author: yelevin
ms.openlocfilehash: 8c9a7a09d3085b1ec67bf29f142c6e7b205561ef
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87021747"
---
# <a name="connect-your-perimeter-81-activity-logs-to-azure-sentinel"></a>Connectez vos journaux d’activité Perimeter 81 à Azure Sentinel

> [!IMPORTANT]
> Le connecteur de données Perimeter 81 dans Azure Sentinel est actuellement en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Cet article explique comment connecter votre appliance de [journaux d’activité Perimeter 81](https://www.perimeter81.com/) à Azure Sentinel. Le connecteur de journaux d’activité Perimeter 81 vous permet de placer facilement vos données Perimeter 81 dans Azure Sentinel, pour pouvoir les consulter dans des classeurs, les utiliser pour créer des alertes personnalisées et les incorporer pour améliorer l’investigation.

> [!NOTE]
> Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

## <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’autorisations en lecture et en écriture dans l’espace de travail Azure Sentinel.

- Vous devez disposer d’autorisations de lecture sur les clés partagées pour accéder à l’espace de travail.

## <a name="configure-and-connect-perimeter-81-activity-logs"></a>Configurer et connecter les journaux d’activité Perimeter 81

Les journaux d’activité Perimeter 81 peuvent intégrer et exporter des journaux directement dans Azure Sentinel.

1. Dans le portail Azure Sentinel, cliquez sur **Connecteurs de données** dans le menu de navigation.

1. Sélectionnez **Journaux d’activité Perimeter 81** dans la galerie, puis cliquez sur le bouton de la **page d’ouverture de connecteur**.

1. Dans la page connecteur des journaux d’activité Perimeter 81, copiez l’**ID d’espace de travail** et la **clé primaire**, puis collez-les dans Perimeter 81, [comme indiqué ici](https://support.perimeter81.com/hc/en-us/articles/360012680780).

1. Une fois la procédure IP terminée, les types de données connectés apparaissent dans la page du connecteur Azure Sentinel.

## <a name="find-your-data"></a>Recherche de données

Une fois la connexion correctement établie, les données apparaissent dans **Journaux** sous **CustomLogs** - **Perimeter81_CL**.

Plus de 20 minutes peuvent être nécessaires avant que vos journaux ne commencent à apparaître.

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à connecter les journaux d’activité Perimeter 81 à Azure Sentinel. Pour tirer pleinement parti des fonctionnalités intégrées à ce connecteur de données, cliquez sur l’onglet **Étapes suivantes** de la page du connecteur de données. Vous y trouverez un classeur prêt à l’emploi et des exemples de requêtes pour commencer à trouver des informations utiles.

Pour en savoir plus sur Azure Sentinel, voir les articles suivants :

- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.
