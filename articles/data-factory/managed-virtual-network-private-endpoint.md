---
title: Réseau virtuel managé et points de terminaison privés managés
description: Découvrez le réseau virtuel managé et les points de terminaison privés managés dans Azure Data Factory.
services: data-factory
ms.author: abnarain
author: nabhishek
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom:
- seo-lt-2019
- references_regions
ms.date: 07/15/2020
ms.openlocfilehash: 81d82bccd6b6bd97b84df5269dd59ffac4903370
ms.sourcegitcommit: f311f112c9ca711d88a096bed43040fcdad24433
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94980357"
---
# <a name="azure-data-factory-managed-virtual-network-preview"></a>Réseau virtuel managé Azure Data Factory (préversion)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article décrit le réseau virtuel managé et les points de terminaison privés managés dans Azure Data Factory.


## <a name="managed-virtual-network"></a>Réseau virtuel managé

Quand vous créez un runtime Azure Integration Runtime au sein d’un réseau virtuel managé Azure Data Factory, ce runtime est provisionné avec le réseau virtuel managé et il tire parti des points de terminaison privés pour se connecter en toute sécurité aux magasins de données pris en charge. 

La création d’un runtime Azure Integration Runtime au sein d’un réseau virtuel managé garantit l’isolation et la sécurisation du processus d’intégration des données. 

Avantages de l’utilisation du réseau virtuel managé :

- Avec un réseau virtuel managé, vous pouvez déplacer la charge liée à la gestion du réseau virtuel vers Azure Data Factory. Vous n’avez pas besoin de créer un sous-réseau pour Azure Integration Runtime qui pourrait finalement utiliser de nombreuses adresses IP privées à partir de votre réseau virtuel et nécessiter une planification préalable de l’infrastructure réseau. 
- Il ne demande pas de connaissances approfondies en réseau Azure pour effectuer des intégrations de données de façon sécurisée. Au contraire, le démarrage avec l’opération ETL sécurisée est très simplifié pour les ingénieurs de données. 
- Le réseau virtuel managé ainsi que les points de terminaison privés managés assurent une protection contre l’exfiltration des données. 

> [!IMPORTANT]
>Actuellement, le réseau virtuel managé est uniquement pris en charge dans la même région qu’Azure Data Factory.
 

![Architecture de réseau virtuel managé ADF](./media/managed-vnet/managed-vnet-architecture-diagram.png)

## <a name="managed-private-endpoints"></a>Points de terminaison privés managés

Les points de terminaison privés managés sont des points de terminaison privés créés sur le réseau virtuel managé Azure Data Factory qui établissent une liaison privée vers des ressources Azure. Azure Data Factory gère ces points de terminaison privés à votre place. 

![Nouveau point de terminaison privé managé](./media/tutorial-copy-data-portal-private/new-managed-private-endpoint.png)

Azure Data Factory prend en charge les liaisons privées. Une liaison privée vous permet d’accéder aux services Azure (PaaS), tels que le Stockage Azure, Azure Cosmos DB, Azure Synapse Analytics (anciennement SQL Data Warehouse).

Quand vous utilisez une liaison privée, le trafic entre vos magasins de données et le réseau virtuel managé transite intégralement par le réseau principal de Microsoft. Une liaison privée assure une protection contre les risques liés à l’exfiltration des données. Vous établissez une liaison privée vers une ressource en créant un point de terminaison privé.

Le point de terminaison privé utilise une adresse IP privée sur le réseau virtuel managé pour y placer de fait le service. Les points de terminaison privés sont mappés à une ressource spécifique dans Azure, et non à l’ensemble du service. Les clients peuvent limiter la connectivité à une ressource spécifique approuvée par leur organisation. Apprenez-en davantage sur [les liaisons privées et les points de terminaison privés](../private-link/index.yml).

> [!NOTE]
> Nous vous recommandons de créer des points de terminaison privés managés pour vous connecter à toutes vos sources de données Azure. 
 
> [!WARNING]
> Si un magasin de données PaaS (Blob, ADLS Gen2, Azure Synapse Analytics) a un point de terminaison privé déjà créé, et même s’il autorise l’accès à partir de tous les réseaux, ADF peut uniquement y accéder à l’aide d’un point de terminaison privé managé. Veillez à créer un point de terminaison privé dans de tels scénarios. 

Une connexion de point de terminaison privé est créée dans un état « en attente » quand vous créez un point de terminaison privé managé dans Azure Data Factory. Un workflow d’approbation est lancé. Le propriétaire de la ressource de liaison privée est responsable de l’approbation ou du refus de la connexion.

![Point de terminaison privé managé](./media/tutorial-copy-data-portal-private/manage-private-endpoint.png)

Si le propriétaire approuve la connexion, la liaison privée est établie. S’il la refuse, la liaison privée n’est pas établie. Dans les deux cas, le point de terminaison privé managé est mis à jour avec l’état de la connexion.

![Approuver le point de terminaison privé managé](./media/tutorial-copy-data-portal-private/approve-private-endpoint.png)

Seule une instance de point de terminaison privé managé dans un état approuvé peut envoyer du trafic vers une ressource de liaison privée donnée.

## <a name="interactive-authoring"></a>Création interactive
Les options de création interactive sont utilisées pour des fonctionnalités telles que tester la connexion, parcourir la liste des dossiers et la liste des tables, obtenir un schéma et afficher un aperçu des données. Vous pouvez activer la création interactive lors de la création ou de la modification d’un runtime d’intégration Azure figurant dans un réseau virtuel géré par ADF. Le service back-end pré-allouera le calcul pour les fonctionnalités de création interactive. Sinon, le calcul sera alloué chaque fois qu’une opération interactive sera exécutée, ce qui prendra plus de temps. La durée de vie (TTL) pour la création interactive est de 60 minutes, ce qui signifie qu’elle sera automatiquement désactivée 60 minutes après de la dernière opération de création interactive.

![Création interactive](./media/managed-vnet/interactive-authoring.png)

## <a name="limitations-and-known-issues"></a>Limitations et problèmes connus
### <a name="supported-data-sources"></a>Sources de données prises en charge
Les sources de données suivantes sont prises en charge pour établir une connexion via une liaison privée à partir d’un réseau virtuel managé ADF.
- Stockage Blob Azure
- Stockage Table Azure
- Azure Files
- Azure Data Lake Gen2
- Azure SQL Database (sans Azure SQL Managed Instance)
- Azure Synapse Analytics (anciennement SQL Data Warehouse)
- SQL Azure Cosmos DB
- Azure Key Vault
- Service Azure Private Link
- Recherche Azure
- Azure Database pour MySQL
- Azure Database pour PostgreSQL
- Azure Database for MariaDB

### <a name="azure-data-factory-managed-virtual-network-is-available-in-the-following-azure-regions"></a>Le réseau virtuel managé Azure Data Factory est disponible dans les régions Azure suivantes :
- USA Est
- USA Est 2
- Centre-USA Ouest
- USA Ouest
- USA Ouest 2
- États-Unis - partie centrale méridionale
- USA Centre
- Europe Nord
- Europe Ouest
- Sud du Royaume-Uni
- Asie Sud-Est
- Australie Est
- Sud-Australie Est

### <a name="outbound-communications-through-public-endpoint-from-adf-managed-virtual-network"></a>Communications sortantes via un point de terminaison public à partir du réseau virtuel managé ADF
- Seul le port 443 est ouvert pour les communications sortantes.
- Le Stockage Azure et Azure Data Lake Gen2 ne sont pas pris en charge pour une connexion via un point de terminaison public à partir du réseau virtuel managé ADF.

### <a name="linked-service-creation-of-azure-key-vault"></a>Création d’un service lié à Azure Key Vault 
- Lorsque vous créez un service lié pour Azure Key Vault, il n’existe aucune référence Azure Integration Runtime. Vous ne pouvez donc pas créer de point de terminaison privé pendant la création du service lié d’Azure Key Vault. Toutefois, lorsque vous créez un service lié pour des magasins de données qui fait référence au service lié Azure Key Vault et que ce service lié fait référence à Azure Integration Runtime avec un réseau virtuel managé activé, vous pouvez créer un point de terminaison privé pour le service lié Azure Key Vault lors de la création. 
- L’opération **Tester la connexion** du service lié d’Azure Key Vault valide uniquement le format d’URL, mais n’effectue aucune opération sur le réseau.

## <a name="next-steps"></a>Étapes suivantes

- Tutoriel : [Générer un pipeline de copie à l’aide d’un réseau virtuel managé et de points de terminaison privés](tutorial-copy-data-portal-private.md) 
- Tutoriel : [Générer un pipeline de dataflow de mappage à l’aide d’un réseau virtuel managé et de points de terminaison privés](tutorial-data-flow-private.md)