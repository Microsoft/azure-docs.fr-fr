---
title: Topologies réseau pour les migrations SQL Managed Instance
titleSuffix: Azure Database Migration Service
description: Découvrez les configurations sources et cibles pour les migrations Azure SQL Managed Instance à l’aide du service Azure Database Migration Service.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: reference
ms.date: 01/08/2020
ms.openlocfilehash: ae036b7d893eb268ea55026054bf364dad0b610e
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94961547"
---
# <a name="network-topologies-for-azure-sql-managed-instance-migrations-using-azure-database-migration-service"></a>Topologies de réseau pour les migrations SQL Managed Instance l’aide d'Azure Database Migration Service

Cet article traite de différentes topologies de réseau avec lesquelles Azure Database Migration Service peut fonctionner pour offrir une expérience de migration complète vers Azure SQL Managed Instance à partir d’instances SQL Server.

## <a name="azure-sql-managed-instance-configured-for-hybrid-workloads"></a>Azure SQL Managed Instance configuré pour les charges de travail hybrides 

Utilisez cette topologie si votre instance Azure SQL Managed Instance est connectée à votre réseau local. Cette approche fournit le routage réseau le plus simplifié et génère le débit de données maximale lors de la migration.

![Topologie de réseau pour les charges de travail hybrides](media/resource-network-topologies/hybrid-workloads.png)

**Configuration requise**

- Dans ce scénario, l’instance SQL Managed Instance et l’instance Azure Database Migration Service sont créées sur le même réseau virtuel Microsoft Azure, mais utilisent des sous-réseaux différents.  
- Le réseau virtuel utilisé dans ce scénario est également connecté au réseau local à l’aide d’[ExpressRoute](../expressroute/expressroute-introduction.md) ou de [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md).

## <a name="sql-managed-instance-isolated-from-the-on-premises-network"></a>SQL Managed Instance isolé du réseau local

Utilisez cette topologie de réseau si votre environnement requiert un ou plusieurs des scénarios suivants :

- L’instance SQL Managed Instance est isolée de la connectivité locale, mais votre instance Azure Database Migration Service est connectée au réseau local.
- Si les stratégies de contrôle d’accès en fonction du rôle Azure (Azure RBAC) sont en place et que vous devez limiter l’accès utilisateur à l’abonnement qui héberge SQL Managed Instance.
- Les réseaux virtuels utilisés pour SQL Managed Instance et Azure Database Migration Service se trouvent dans des abonnements différents.

![Topologie de réseau pour l’instance gérée isolée du réseau local](media/resource-network-topologies/mi-isolated-workload.png)

**Configuration requise**

- Le réseau virtuel utilisé par Azure Database Migration Service pour ce scénario doit également être connecté au réseau local à l’aide de https://docs.microsoft.com/azure/expressroute/expressroute-introduction) ou de [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md).
- Configurez [le peering de réseau](../virtual-network/virtual-network-peering-overview.md) entre le réseau virtuel utilisé pour SQL Managed Instance et Azure Database Migration Service.

## <a name="cloud-to-cloud-migrations-shared-virtual-network"></a>Migrations de cloud à cloud : Réseau virtuel partagé

Utilisez cette topologie si le serveur source SQL Server est hébergé dans une machine virtuelle Azure et partage le même réseau virtuel avec SQL Managed Instance et Azure Database Migration Service.

![Topologie de réseau pour les migrations de cloud à cloud avec réseau virtuel partagé](media/resource-network-topologies/cloud-to-cloud.png)

**Configuration requise**

- Aucune exigence supplémentaire.

## <a name="cloud-to-cloud-migrations-isolated-virtual-network"></a>Migrations de cloud à cloud : Réseau virtuel isolé

Utilisez cette topologie de réseau si votre environnement requiert un ou plusieurs des scénarios suivants :

- L’instance SQL Managed Instance est configurée dans un réseau virtuel isolé.
- Si les stratégies de contrôle d’accès en fonction du rôle Azure (Azure RBAC) sont en place et que vous devez limiter l’accès utilisateur à l’abonnement qui héberge SQL Managed Instance.
- Les réseaux virtuels utilisés pour SQL Managed Instance et Azure Database Migration Service se trouvent dans des abonnements différents.

![Topologie de réseau pour les migrations de cloud à cloud avec un réseau virtuel isolé](media/resource-network-topologies/cloud-to-cloud-isolated.png)

**Configuration requise**

- Configurez [le peering de réseau](../virtual-network/virtual-network-peering-overview.md) entre le réseau virtuel utilisé pour SQL Managed Instance et Azure Database Migration Service.

## <a name="inbound-security-rules"></a>Règles de sécurité de trafic entrant

| **NAME**   | **PORT** | **PROTOCOLE** | **SOURCE** | **DESTINATION** | **ACTION** |
|------------|----------|--------------|------------|-----------------|------------|
| DMS_subnet | Quelconque      | Quelconque          | DMS SUBNET | Quelconque             | Allow      |

## <a name="outbound-security-rules"></a>Règles de sécurité de trafic entrant

| **NAME**                  | **PORT**                                              | **PROTOCOLE** | **SOURCE** | **DESTINATION**           | **ACTION** | **Raison de la règle**                                                                                                                                                                              |
|---------------------------|-------------------------------------------------------|--------------|------------|---------------------------|------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| gestion                | 443,9354                                              | TCP          | Quelconque        | Quelconque                       | Allow      | Communication du plan de gestion via Service Bus et Stockage Blob Azure. <br/>(Si le peering Microsoft est activé, vous n’avez peut-être pas besoin de cette règle.)                                                             |
| Diagnostics               | 12 000                                                 | TCP          | Quelconque        | Quelconque                       | Allow      | DMS utilise cette règle pour collecter des informations de diagnostic à des fins de dépannage.                                                                                                                      |
| Serveur source SQL         | 1433 (ou port TCP IP que SQL Server écoute) | TCP          | Quelconque        | Espace d’adressage local | Allow      | Connectivité source SQL Server à partir de DMS <br/>(Si vous disposez d’une connectivité de site à site, vous n’aurez peut-être pas besoin de cette règle.)                                                                                       |
| Instance nommée SQL Server | 1434                                                  | UDP          | Quelconque        | Espace d’adressage local | Allow      | Connectivité source de l’instance nommée SQL Server à partir de DMS <br/>(Si vous disposez d’une connectivité de site à site, vous n’aurez peut-être pas besoin de cette règle.)                                                                        |
| Partage SMB                 | 445                                                   | TCP          | Quelconque        | Espace d’adressage local | Allow      | Partage réseau SMB pour DMS afin de stocker les fichiers de sauvegarde des bases de données pour les migrations vers Azure SQL Database MI et SQL Server sur machines virtuelles Azure <br/>(Si vous disposez d’une connectivité de site à site, vous n’aurez peut-être pas besoin de cette règle). |
| DMS_subnet                | Quelconque                                                   | Quelconque          | Quelconque        | DMS_subnet                | Allow      |                                                                                                                                                                                                  |

## <a name="see-also"></a>Voir aussi

- [Migrer SQL Server vers une instance gérée SQL](./tutorial-sql-server-to-managed-instance.md)
- [Vue d’ensemble de la configuration requise pour l’utilisation d’Azure Database Migration Service](./pre-reqs.md)
- [Créer un réseau virtuel au moyen du portail Azure](../virtual-network/quick-create-portal.md)

## <a name="next-steps"></a>Étapes suivantes

- Pour une vue d’ensemble d’Azure Database Migration Service, consultez l’article [Présentation d’Azure Database Migration Service](dms-overview.md).
- Pour obtenir des informations actuelles sur la disponibilité régionale d’Azure Database Migration Service, consultez la page [Produits disponibles par région](https://azure.microsoft.com/global-infrastructure/services/?products=database-migration).