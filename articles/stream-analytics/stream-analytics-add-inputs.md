---
title: Comprendre les entrées d’Azure Stream Analytics
description: Cet article décrit le concept des entrées d’un travail Azure Stream Analytics, en comparant les entrées de diffusion en continu aux entrées des données de référence.
author: jseb225
ms.author: krishmam
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 10/29/2020
ms.openlocfilehash: b344e9e24d15189b805f586227c7253395e8448e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96022062"
---
# <a name="understand-inputs-for-azure-stream-analytics"></a>Comprendre les entrées d’Azure Stream Analytics

Les travaux Azure Stream Analytics se connectent à une ou plusieurs entrées de données. Chaque entrée définit une connexion à une source de données existante. Stream Analytics accepte des données provenant de divers types de sources d’événements, notamment Event Hubs, IoT Hub et le stockage d’objets blob. Les entrées sont référencées à l’aide du nom de la requête SQL de diffusion en continu que vous écrivez pour chaque travail. Dans la requête, vous pouvez joindre plusieurs entrées pour fusionner des données ou pour comparer des données de diffusion en continu à une recherche pour référencer des données, puis transmettre les résultats aux sorties. 

Stream Analytics possède une intégration de première classe avec quatre types de ressources en tant qu’entrées :
- [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/)
- [Azure IoT Hub](https://azure.microsoft.com/services/iot-hub/) 
- [stockage d’objets blob Azure](https://azure.microsoft.com/services/storage/blobs/) 
- [Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md) 

Ces ressources d’entrée peuvent résider dans le même abonnement Azure que votre travail Stream Analytics, ou provenir d’un autre abonnement.

Vous pouvez utiliser le [portail Azure](stream-analytics-quick-create-portal.md#configure-job-input), [Azure PowerShell](/powershell/module/az.streamanalytics/New-azStreamAnalyticsInput), [API .NET](/dotnet/api/microsoft.azure.management.streamanalytics.inputsoperationsextensions), [API REST](/rest/api/streamanalytics/2016-03-01/inputs) et [Visual Studio](stream-analytics-tools-for-visual-studio-install.md) pour créer, modifier et tester les entrées du travail Stream Analytics.

## <a name="stream-and-reference-inputs"></a>Entrées de flux de données et entrées de données de référence
Lorsque les données sont transmises à une source de données, elles sont utilisées par le travail Stream Analytics et traitées en temps réel. Les entrées sont réparties en deux types : les entrées de flux de données et les entrées de données de référence.

### <a name="data-stream-input"></a>Entrée de flux de données
Un flux de données est une séquence illimitée d’événements au fil du temps. Les travaux Stream Analytics doivent contenir au moins une entrée de flux de données. Event Hubs, IoT Hub, Azure Data Lake Storage Gen2 et Stockage Blob sont pris en charge en tant que sources d’entrée de flux de données. Les concentrateurs Event Hubs permettent de collecter des flux d’événements à partir de plusieurs appareils et services. Il peut s’agir de flux d’activités de réseaux sociaux, d’informations boursières ou de données provenant de capteurs. Les conteneurs IoT Hub sont optimisés pour collecter des données à partir d’appareils connectés dans les scénarios Internet des objets (IoT).  Le stockage d’objets blob peut être utilisé comme source d’entrée pour la réception de données en bloc en tant que flux, par exemple des fichiers journaux.  

Pour plus d’informations sur les entrées de données de diffusion en continu, consultez [Connexion de données : en savoir plus sur les entrées de flux de données pour Stream Analytics](stream-analytics-define-inputs.md).

### <a name="reference-data-input"></a>Entrée de données de référence
Stream Analytics prend également en charge des entrées appelées *données de référence*. Les données de référence sont complètement statiques ou elles subissent de lentes modifications. Elles sont généralement utilisées pour la corrélation et les recherches. Par exemple, vous pouvez joindre des données de l’entrée de flux de données à des données de référence, comme vous effectueriez une jointure SQL pour rechercher des valeurs statiques. Stockage Blob Azure, Azure Data Lake Storage Gen2, et Azure SQL Database sont actuellement pris en charge en tant que sources d’entrée pour les données de référence. Les objets blob des sources de données de référence ont une limite de taille maximale de 300 Mo, selon la complexité de la requête et des unités de streaming allouées (consultez la section [Limite de taille](stream-analytics-use-reference-data.md#size-limitation) dans la documentation relative aux données de référence pour plus d’informations).

Pour plus d’informations sur les entrées de données de référence, consultez [Utiliser des données de référence pour effectuer des recherches dans Stream Analytics](stream-analytics-use-reference-data.md).

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Démarrage rapide : Créer un travail Stream Analytics à l’aide du portail Azure](stream-analytics-quick-create-portal.md)