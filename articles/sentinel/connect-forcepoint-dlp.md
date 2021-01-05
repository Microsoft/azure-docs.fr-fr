---
title: Connexion de Forcepoint DLP à Azure Sentinel | Microsoft Docs
description: Découvrez comment connecter Forcepoint DLP à Azure Sentinel.
services: sentinel
author: yelevin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/20/2020
ms.author: yelevin
ms.openlocfilehash: 3bdb9d2b23ce05929ba5612e0c6a03fe1aab05de
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77588244"
---
# <a name="connect-your-forcepoint-dlp-to-azure-sentinel"></a>Connexion de Forcepoint DLP à Azure Sentinel

> [!IMPORTANT]
> Le connecteur de données de protection contre la perte de données (DLP, Data Loss Prevention) Forcepoint dans Azure Sentinel est actuellement en préversion publique. Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).



Le connecteur Forcepoint DLP permet d’exporter automatiquement des données d’incident de protection contre la perte de données dans Azure Sentinel. Vous pouvez l’utiliser pour bénéficier d’une visibilité sur les activités des utilisateurs et les incidents de perte de données. Il permet également d’établir des corrélations avec les données issues des charges de travail Azure et d’autres flux, et améliore la capacité de monitoring des classeurs dans Azure Sentinel.

> [!NOTE]
> Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

## <a name="configure-and-connect-forcepoint-dlp"></a>Configuration et connexion de Forcepoint DLP

Configurez Forcepoint DLP de façon à transférer les données d’incident au format JSON dans votre espace de travail Azure via l’API REST, comme l’explique le [Guide d’intégration Forcepoint DLP](https://frcpnt.com/dlp-sentinel).


## <a name="find-your-data"></a>Rechercher vos données

Une fois le connecteur Forcepoint DLP configuré, les données s’affichent dans Log Analytics sous CustomLogs **ForcepointDLPEvents_CL**.


Pour utiliser le schéma approprié dans Log Analytics pour Forcepoint DLP, recherchez **ForcepointDLPEvents_CL**.


## <a name="validate-connectivity"></a>Valider la connectivité

Plus de 20 minutes peuvent être nécessaires avant que vos journaux ne commencent à apparaître dans Log Analytics.

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à connecter Forcepoint DLP à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :

- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.