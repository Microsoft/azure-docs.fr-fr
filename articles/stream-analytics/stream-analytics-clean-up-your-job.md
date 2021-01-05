---
title: Supprimer une tâche Azure Stream Analytics
description: Cet article montre différentes méthodes de suppression des tâches Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 06/21/2019
ms.custom: seodec18
ms.openlocfilehash: 2652b97c9392d3016bbc52209d4b2bda81c31706
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96022079"
---
# <a name="stop-or-delete-your-azure-stream-analytics-job"></a>Arrêter ou supprimer votre travail Azure Stream Analytics

Vous pouvez facilement arrêter ou supprimer les travaux Azure Stream Analytics via le portail Azure, Azure PowerShell, le kit SDK Azure pour .NET ou l’API REST. Un travail Stream Analytics ne peut pas être récupéré une fois qu’il a été supprimé.

>[!NOTE] 
>Quand vous arrêtez votre tâche Azure Stream Analytics, les données sont seulement conservées dans le stockage d’entrée et de sortie, comme Event Hubs ou Azure SQL Database. Si vous devez supprimer des données dans Azure, veillez à suivre le processus de suppression des ressources d’entrée et de sortie de votre tâche Azure Stream Analytics.

## <a name="stop-a-job-in-azure-portal"></a>Arrêter une tâche dans le portail Azure

Lorsque vous arrêtez un travail, les ressources sont déprovisionnées et le traitement des événements est stoppé. Les frais associés à ce travail sont également suspendus. Toutefois, l’ensemble de votre configuration est conservée et vous pouvez redémarrer le travail plus tard 

1. Connectez-vous au [portail Azure](https://portal.azure.com). 

2. Localisez votre tâche Stream Analytics en cours d’exécution et sélectionnez-la.

3. Dans la page de la tâche Stream Analytics, sélectionnez **Arrêter** pour arrêter la tâche. 

   ![Arrêter une tâche Azure Stream Analytics](./media/stream-analytics-clean-up-your-job/stop-stream-analytics-job.png)


## <a name="delete-a-job-in-azure-portal"></a>Supprimer une tâche dans le portail Azure

>[!WARNING] 
>Un travail Stream Analytics ne peut pas être récupéré une fois qu’il a été supprimé.

1. Connectez-vous au portail Azure. 

2. Localisez votre tâche Stream Analytics existante et sélectionnez-la.

3. Dans la page de la tâche Stream Analytics, sélectionnez **Supprimer** pour supprimer la tâche. 

   ![Supprimer une tâche Azure Stream Analytics](./media/stream-analytics-clean-up-your-job/delete-stream-analytics-job.png)


## <a name="stop-or-delete-a-job-using-powershell"></a>Arrêter ou supprimer une tâche avec PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Pour arrêter une tâche avec PowerShell, utilisez la cmdlet [Stop-AzStreamAnalyticsJob](/powershell/module/az.streamanalytics/stop-azstreamanalyticsjob). Pour supprimer une tâche avec PowerShell, utilisez la cmdlet [Remove-AzStreamAnalyticsJob](/powershell/module/az.streamanalytics/Remove-azStreamAnalyticsJob).

## <a name="stop-or-delete-a-job-using-azure-sdk-for-net"></a>Arrêter ou supprimer une tâche avec Azure SDK pour .NET

Pour arrêter un travail avec Azure SDK pour .NET, utilisez la méthode [StreamingJobsOperationsExtensions.BeginStop](/dotnet/api/microsoft.azure.management.streamanalytics.streamingjobsoperationsextensions.beginstop). Pour supprimer un travail avec Azure SDK pour .NET, utilisez la méthode [StreamingJobsOperationsExtensions.BeginDelete](/dotnet/api/microsoft.azure.management.streamanalytics.streamingjobsoperationsextensions.begindelete).

## <a name="stop-or-delete-a-job-using-rest-api"></a>Arrêter ou supprimer une tâche avec l’API REST

Pour arrêter une tâche avec l’API REST, référencez la méthode [Stop](/rest/api/streamanalytics/2016-03-01/streamingjobs/stop). Pour supprimer une tâche avec l’API REST, référencez la méthode [Delete](/rest/api/streamanalytics/2016-03-01/streamingjobs/delete).