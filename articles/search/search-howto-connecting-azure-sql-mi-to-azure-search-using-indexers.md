---
title: Connexion Azure SQL Managed Instance pour l’indexation de recherche
titleSuffix: Azure Cognitive Search
description: Activez le point de terminaison public pour autoriser les connexions à SQL Managed Instances à partir d’un indexeur dans Recherche cognitive Azure.
manager: nitinme
author: vl8163264128
ms.author: victliu
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 9e8625724f67caac99ae799674f9db9399e11ad8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89294252"
---
# <a name="configure-a-connection-from-an-azure-cognitive-search-indexer-to-sql-managed-instance"></a>Configurer une connexion entre un indexeur Recherche cognitive Azure et SQL Managed Instance

Comme indiqué dans [Connexion d’Azure SQL Database à Recherche cognitive Azure à l’aide d’indexeurs](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md#faq), la création d’indexeurs sur **SQL Managed Instances** est prise en charge par Recherche cognitive Azure via le point de terminaison public.

## <a name="create-azure-sql-managed-instance-with-public-endpoint"></a>Créer une instance Azure SQL Managed Instance avec un point de terminaison public
Créez une instance SQL Managed Instance avec l’option **Activer le point de terminaison public** sélectionnée.

   ![Activer le point de terminaison public](media/search-howto-connecting-azure-sql-mi-to-azure-search-using-indexers/enable-public-endpoint.png "Activer un point de terminaison public")

## <a name="enable-azure-sql-managed-instance-public-endpoint"></a>Activer un point de terminaison public Azure SQL Managed Instance
Vous pouvez également activer un point de terminaison public sur une instance SQL Managed Instance existante sous **Sécurité** > **Réseau virtuel** > **Point de terminaison public** > **Activer**.

   ![Activer un point de terminaison public au moyen d’une instance managée de VNET](media/search-howto-connecting-azure-sql-mi-to-azure-search-using-indexers/mi-vnet.png "Activer un point de terminaison public")

## <a name="verify-nsg-rules"></a>Vérifier les règles du groupe de sécurité réseau
Vérifiez que le groupe de sécurité réseau contient des **règles de sécurité de trafic entrant** appropriées qui autorisent les connexions à partir des services Azure.

   ![Règle de sécurité de trafic entrant de NSG](media/search-howto-connecting-azure-sql-mi-to-azure-search-using-indexers/nsg-rule.png "Règle de sécurité de trafic entrant de NSG")

> [!NOTE]
> Les indexeurs requièrent toujours que SQL Managed Instance soit configuré avec un point de terminaison public pour pouvoir lire les données.
> Toutefois, vous pouvez choisir de restreindre l’accès entrant de ce point de terminaison public en remplaçant la règle actuelle (`public_endpoint_inbound`) par les deux règles suivantes :
>
> * Autorisation de l’accès entrant à partir de la [balise de service](../virtual-network/service-tags-overview.md#available-service-tags) `AzureCognitiveSearch` ("SOURCE" = `AzureCognitiveSearch`, "NAME" = `cognitive_search_inbound`)
>
> * Autorisation de l’accès entrant à partir de l’adresse IP du service de recherche, qui peut être obtenue en exécutant une commande ping sur son nom de domaine complet (par exemple, `<your-search-service-name>.search.windows.net`). ("SOURCE" = `IP address`, "NAME" = `search_service_inbound`)
>
> Pour chacune de ces deux règles, définissez "PORT" = `3342`, "PROTOCOL" = `TCP`, "DESTINATION" = `Any`, "ACTION" = `Allow`

## <a name="get-public-endpoint-connection-string"></a>Obtenir la chaîne de connexion du point de terminaison public
Veillez à utiliser la chaîne de connexion pour le **point de terminaison public** (le port 3342, et non le port 1433).

   ![Chaîne de connexion du point de terminaison public](media/search-howto-connecting-azure-sql-mi-to-azure-search-using-indexers/mi-connection-string.png "Chaîne de connexion du point de terminaison public")

## <a name="next-steps"></a>Étapes suivantes
Une fois la configuration résolue, vous pouvez spécifier une instance SQL Managed Instance comme source de données pour un indexeur Recherche cognitive Azure à l’aide du portail ou de l’API REST. Pour plus d’informations, consultez [Connexion d’Azure SQL Database à Recherche cognitive Azure à l’aide d’indexeurs](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md).