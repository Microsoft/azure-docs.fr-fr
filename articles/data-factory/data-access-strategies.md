---
title: Stratégies d’accès aux données
description: Azure Data Factory prend désormais en charge les plages d’adresses IP statiques.
services: data-factory
ms.author: abnarain
author: nabhishek
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 05/28/2020
ms.openlocfilehash: 785381e0a42f2b502e4ea7054753d5f3fb67f385
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92632768"
---
# <a name="data-access-strategies"></a>Stratégies d’accès aux données

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Un objectif de sécurité vital d’une organisation est de protéger ses magasins de données contre tout accès aléatoire à partir d’Internet, qu’il s’agisse de magasins de données SaaS locaux ou cloud. 

En général, un magasin de données cloud contrôle l’accès à l’aide des mécanismes ci-dessous :
* Liaison privée d’un réseau virtuel à des sources de données avec point de terminaison privé
* Règles de pare-feu qui limitent la connectivité par adresse IP.
* Mécanismes d’authentification qui obligent les utilisateurs à prouver leur identité.
* Mécanismes d’autorisation qui restreignent les utilisateurs à certaines actions et données.

> [!TIP]
> Avec l’[introduction de la plage d’adresses IP statiques](./azure-integration-runtime-ip-addresses.md), vous pouvez désormais autoriser en les mettant en liste verte des plages d’adresses IP pour la région du runtime d’intégration Azure, afin de vous assurer que vous n’avez pas à autoriser toutes les adresses IP Azure dans vos magasins de données cloud. De cette façon, vous pouvez limiter les adresses IP qui sont autorisées à accéder aux magasins de données.

> [!NOTE] 
> Les plages d’adresses IP sont bloquées pour Azure Integration Runtime et sont actuellement utilisées uniquement pour le déplacement des données, le pipeline et les activités externes. Les dataflows et Azure Integration Runtime qui activent le réseau virtuel géré n’utilisent plus ces plages d’adresses IP. 

Cela devrait fonctionner dans de nombreux scénarios et nous savons bien qu’une adresse IP statique unique par runtime d’intégration serait souhaitable. Ce ne serait cependant pas actuellement possible à l’aide ’un runtime d’intégration Azure, qui opère sans serveur. Si nécessaire, vous pouvez toujours configurer un runtime d’intégration auto-hébergé et l’utiliser avec votre adresse IP statique. 

## <a name="data-access-strategies-through-azure-data-factory"></a>Stratégies d’accès aux données via Azure Data Factory

* **[Azure Private Link](../private-link/private-link-overview.md)**  : vous pouvez créer un runtime d’intégration Azure au sein d’un réseau virtuel géré Azure Data Factory et il tirera parti des points de terminaison privés pour se connecter en toute sécurité aux magasins de données pris en charge. Le trafic entre le réseau virtuel géré et les sources de données transite par le réseau principal de Microsoft et n’est pas exposé au réseau public.
* **[Service approuvé](../storage/common/storage-network-security.md#exceptions)**  : le Stockage Azure (BLOB, ADLS Gen2) prend en charge une configuration de pare-feu qui permet à certains services de plateforme Azure approuvés d’accéder au compte de stockage en toute sécurité . Les services approuvés appliquent une authentification d’identité gérée, qui garantit qu’aucune autre fabrique de données ne peut se connecter à ce stockage, sauf si elle est autorisée à le faire à l’aide de son identité gérée. Pour plus d’informations, lisez **[ce blog](https://techcommunity.microsoft.com/t5/azure-data-factory/data-factory-is-now-a-trusted-service-in-azure-storage-and-azure/ba-p/964993)** . Cette solution est donc extrêmement sécurisée et recommandée. 
* **Adresse IP statique unique**  : vous devez configurer un runtime d’intégration auto-hébergé pour obtenir une adresse IP statique pour les connecteurs Data Factory. Ce mécanisme garantit que vous pouvez bloquer l’accès à partir de toutes les autres adresses IP. 
* **[Plage d’adresses IP statiques](./azure-integration-runtime-ip-addresses.md)**  : vous pouvez utiliser les adresses IP d’Azure Integration Runtime pour les autoriser par mise en liste verte dans votre stockage (par exemple, S3, Salesforce, etc.). Cela restreint certainement les adresses IP qui peuvent se connecter aux magasins de données, mais dépend également des règles d’authentification/autorisation.
* **[Balise de service](../virtual-network/service-tags-overview.md)**  : une balise de service représente un groupe de préfixes d’adresses IP d’un service Azure donné (comme Azure Data Factory). Microsoft gère les préfixes d’adresses englobés par l’étiquette de service et met à jour automatiquement l’étiquette de service quand les adresses changent, ce qui réduit la complexité des mises à jour fréquentes relatives aux règles de sécurité réseau. Cette solution est utile lors du filtrage de l’accès aux données sur des magasins de données hébergés par IaaS dans un réseau virtuel.
* **Autoriser les services Azure**  : certains services vous permettent d’autoriser tous les services Azure à s’y connecter si vous choisissez cette option. 

Pour plus d’informations sur les mécanismes de sécurité réseau pris en charge sur les banques de données dans Azure Integration Runtime et le runtime d’intégration auto-hébergé, voir les deux tableaux ci-dessous.  
* **Azure Integration Runtime**

    | Magasins de données                  | Mécanisme de sécurité réseau pris en charge sur les banques de données | Private Link     | Service approuvé     | Plage d’adresses IP statiques | Étiquettes de service | Autoriser les services Azure |
    |------------------------------|-------------------------------------------------------------|---------------------|-----------------|--------------|----------------------|-----------------|
    | Magasins de données PaaS Azure       | Azure Cosmos DB                                     | Oui              | -                   | Oui             | -            | Oui                  |
    |                              | Explorateur de données Azure                                 | -                | -                   | Oui*            | Oui*         | -                    |
    |                              | Azure Data Lake Gen1                                | -                | -                   | Oui             | -            | Oui                  |
    |                              | Azure Database for MariaDB, MySQL et PostgreSQL       | -                | -                   | Oui             | -            | Oui                  |
    |                              | Stockage Fichier Azure                                  | Oui              | -                   | Oui             | -            | .                    |
    |                              | Stockage Azure (Blob, ADLS Gen2)                     | Oui              | Oui (authentification MSI uniquement) | Oui             | -            | .                    |
    |                              | Azure SQL DB, Azure Synapse Analytics, SQL Ml  | Oui (uniquement Azure SQL DB/DW)        | -                   | Oui             | -            | Oui                  |
    |                              | Azure Key Vault (pour l’extraction de secrets/chaîne de connexion) | Oui      | Oui                 | Oui             | -            | -                    |
    | Autres magasins de données PaaS/SaaS | AWS S3, SalesForce, Google Cloud Storage, etc.    | -                | -                   | Oui             | -            | -                    |
    | IaaS Azure                   | SQL Server, Oracle, etc.                          | -                | -                   | Oui             | Oui          | -                    |
    | IaaS locale              | SQL Server, Oracle, etc.                          | -                | -                   | Oui             | -            | -                    |
    
    **S’applique uniquement quand Data Explorer est injecté par un réseau virtuel et que la plage d’adresses IP peut être appliquée à un groupe de sécurité réseau/pare-feu.* 

* **Runtime d’intégration auto-hébergé (dans un réseau virtuel/en local)**
    
    | Magasins de données                  | Mécanisme de sécurité réseau pris en charge sur les banques de données         | Adresse IP statique | Services approuvés  |
    |--------------------------------|---------------------------------------------------------------|-----------|---------------------|
    | Magasins de données PaaS Azure       | Azure Cosmos DB                                               | Oui       | -                   |
    |                                | Explorateur de données Azure                                           | -         | -                   |
    |                                | Azure Data Lake Gen1                                          | Oui       | -                   |
    |                                | Azure Database for MariaDB, MySQL et PostgreSQL               | Oui       | -                   |
    |                                | Stockage Fichier Azure                                            | Oui       | -                   |
    |                                | Stockage Azure (Blog, ADLS Gen2)                             | Oui       | Oui (authentification MSI uniquement) |
    |                                | Azure SQL DB, Azure Synapse Analytics, SQL Ml          | Oui       | -                   |
    |                                | Azure Key Vault (pour l’extraction de secrets/chaîne de connexion) | Oui       | Oui                 |
    | Autres magasins de données PaaS/SaaS | AWS S3, SalesForce, Google Cloud Storage, etc.              | Oui       | -                   |
    | IaaS Azure                     | SQL Server, Oracle, etc.                                  | Oui       | -                   |
    | IaaS locale              | SQL Server, Oracle, etc.                                  | Oui       | -                   |    

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles connexes suivants :
* [Magasins de données pris en charge](./copy-activity-overview.md#supported-data-stores-and-formats)
* [Services approuvés par Azure Key Vault](../key-vault/general/overview-vnet-service-endpoints.md#trusted-services)
* [Services Microsoft approuvés par Stockage Azure](../storage/common/storage-network-security.md#trusted-microsoft-services)
* [Identité managée pour Data Factory](./data-factory-service-identity.md)