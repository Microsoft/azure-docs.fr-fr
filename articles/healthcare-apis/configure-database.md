---
title: Configurer les paramètres de base de données dans l’API Azure pour FHIR
description: Cet article explique comment configurer les paramètres de base de données dans l’API Azure pour FHIR
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 11/15/2019
ms.author: matjazl
ms.openlocfilehash: 3fe4118a8ecf4479732ba4073b342d3ec9f941b8
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93398196"
---
# <a name="configure-database-settings"></a>Configuration des paramètres de la base de données 

L’API Azure pour FHIR utilise une base de données pour stocker ses données. Le niveau de performance de la base de données sous-jacente dépend du nombre de RU (unités de requête) sélectionné durant le provisionnement du service ou dans les paramètres de base de données une fois le service provisionné.

L’API Azure pour FHIR emprunte le concept de RU à Cosmos DB (consultez [Unités de requête dans Azure Cosmos DB](../cosmos-db/request-units.md)) au moment de la définition du niveau de performance de la base de données sous-jacente. 

Le débit doit être provisionné afin qu’il y ait suffisamment de ressources système disponibles pour votre base de données à tout moment. Le nombre de RU nécessaires à votre application dépend des opérations que vous effectuez. Les opérations peuvent aller de simples lectures et écritures à des requêtes plus complexes. 

> [!NOTE]
> Dans la mesure où différentes opérations consomment un nombre distinct de RU, nous retournons le nombre réel de RU consommées dans chaque appel d’API au sein de l’en-tête de réponse. Ainsi, vous pouvez profiler le nombre de RU consommées par votre application.

## <a name="update-throughput"></a>Mettre à jour le débit

Si vous souhaitez changer ce paramètre dans le portail Azure, accédez à votre API Azure pour FHIR, puis ouvrez le panneau Base de données. Remplacez ensuite le débit provisionné par la valeur souhaitée en fonction de vos besoins de performances. Vous pouvez changer la valeur du nombre d’unités de requête en veillant à ne pas dépasser la limite maximale de 10 000 RU/s. Si vous avez besoin d’une valeur supérieure, contactez le support Azure.

Si le débit de la base de données est supérieur à 10 000 RU/s ou si les données stockées dans la base de données sont supérieures à 50 Go, votre application cliente doit être en capacité de gérer les jetons de continuation. Une nouvelle partition est créée dans la base de données pour chaque augmentation du débit de 10 000 RU/s ou si la quantité de données stockées est supérieure à 50 Go. Plusieurs partitions créent une réponse de plusieurs pages dans laquelle la pagination est implémentée à l’aide de jetons de continuation.

> [!NOTE] 
> Une valeur plus élevée signifie un débit plus élevé pour l’API Azure pour FHIR mais également un coût plus élevé du service.

![Configurer Cosmos DB](media/database/database-settings.png)

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à mettre à jour vos RU dans le cadre de l’API Azure pour FHIR. Pour en savoir plus sur la configuration des clés gérées par le client en tant que paramètre de base de données :

>[!div class="nextstepaction"]
>[Configurer les clés gérées par le client](customer-managed-key.md)

Vous pouvez également déployer une API Azure pour FHIR complètement managée :
 
>[!div class="nextstepaction"]
>[Déployer l’API Azure pour FHIR](fhir-paas-portal-quickstart.md)