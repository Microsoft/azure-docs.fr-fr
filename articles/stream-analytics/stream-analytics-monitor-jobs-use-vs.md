---
title: Monitorage et gestion d’Azure Stream Analytics avec Visual Studio
description: Cet article décrit comment utiliser Visual Studio pour surveiller et gérer des tâches Azure Stream Analytics.
author: su-jie
ms.author: sujie
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 12/07/2018
ms.custom: seodec18
ms.openlocfilehash: a151d4ce5a2f9860c3b7919496a2edd25cc58ede
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93123623"
---
# <a name="monitor-and-manage-stream-analytics-jobs-with-visual-studio"></a>Surveiller et gérer des tâches Stream Analytics avec Visual Studio

Cet article montre comment surveiller vos tâches Stream Analytics dans Visual Studio. Le service Azure Stream Analytics Tools pour Visual Studio fournit une expérience de surveillance similaire à celle du portail Azure sans qu’il soit nécessaire de quitter l’environnement de développement intégré. Vous pouvez commencer à surveiller une tâche dès que vous l’ **avez envoyée à Azure** à partir de votre **Script.asaql** , ou surveiller des tâches existantes, quelle que soit la façon dont celles-ci ont été créées. 

## <a name="job-summary"></a>Résumé de la tâche

Le **Résumé de la tâche** et les **Métriques de la tâche** donnent un aperçu rapide de votre travail. Vous pouvez déterminer en un clin de œil, l’état et les informations d’événement d’une tâche.]

<img src="./media/stream-analytics-monitor-jobs-use-vs/stream-analytics-job-summary-metrics.png" alt="Stream Analytics job summary and job metrics" width="300px"/> 


## <a name="job-metrics"></a>Mesures de travail

Vous pouvez réduire le **Résumé de la tâche** et cliquer sur l’onglet **Métriques de la tâche** pour afficher un graphique représentant des métriques importantes. Activez et désactivez des types de métriques pour les ajouter au graphique ou les en retirer.

![Métriques Azure Stream Analytics dans Visual Studio](./media/stream-analytics-monitor-jobs-use-vs/stream-analytics-vs-metrics.png)


## <a name="error-monitoring"></a>Surveillance des erreurs

Vous pouvez également surveiller les erreurs en cliquant sur l’onglet **Erreurs**.

![Erreurs Azure Stream Analytics dans Visual Studio](./media/stream-analytics-monitor-jobs-use-vs/stream-analytics-vs-errors.png)


## <a name="get-support"></a>Obtenir de l’aide
Pour obtenir de l’aide supplémentaire, consultez notre [page de questions Microsoft Q&A pour Azure Stream Analytics](/answers/topics/azure-stream-analytics.html). 

## <a name="next-steps"></a>Étapes suivantes
* [Présentation d’Azure Stream Analytics](stream-analytics-introduction.md)
* [Créer une tâche Azure Stream Analytics avec Visual Studio](stream-analytics-quick-create-vs.md)
* [Installer Azure Stream Analytics Tools pour Visual Studio](stream-analytics-tools-for-visual-studio-install.md)