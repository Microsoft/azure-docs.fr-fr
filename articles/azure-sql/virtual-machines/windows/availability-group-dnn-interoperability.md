---
title: Interopérabilité des fonctionnalités avec les groupes de disponibilité et l’écouteur de DNN
description: 'Découvrez des aspects supplémentaires à prendre en considération lors de l’utilisation de certaines fonctionnalités de SQL Server et d’un écouteur de nom de réseau distribué (distributed network name, DNN) avec un groupe de disponibilité AlwaysOn sur SQL Server sur des machines virtuelles Azure. '
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 10/08/2020
ms.author: mathoma
ms.openlocfilehash: 19b4b7407468b19419e2f85193b1f8fb6ace39c3
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359402"
---
# <a name="feature-interoperability-with-ag-and-dnn-listener"></a>Interopérabilité des fonctionnalités avec un groupe de disponibilité (AG) et un écouteur de DNN 
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Certaines fonctionnalités de SQL Server reposent sur un nom de réseau virtuel (VNN) codé en dur. Par conséquent, lors de l’utilisation de l’écouteur de DNN avec votre groupe de disponibilité AlwaysOn et SQL Server sur des machines virtuelles Azure, il peut y avoir des aspects supplémentaires à prendre en considération. 

Cet article décrit les fonctionnalités et l’interopérabilité de SQL Server avec l’écouteur de DNN du groupe de disponibilité. 


## <a name="client-drivers"></a>Pilotes clients

Pour les pilotes ODBC, OLEDB, ADO.NET, JDBC, PHP et Node.js, les utilisateurs doivent spécifier explicitement le nom et le port de l’écouteur de DNN en tant que nom de serveur dans la chaîne de connexion. Pour garantir une connectivité rapide lors du basculement, ajoutez `MultiSubnetFailover=True` à la chaîne de connexion si le client SQL le prend en charge. 

## <a name="tools"></a>Outils

Les utilisateurs de [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms), [sqlcmd](/sql/tools/sqlcmd-utility), [Azure Data Studio](/sql/azure-data-studio/what-is) et [SQL Server Data Tools](/sql/ssdt/sql-server-data-tools) doivent spécifier explicitement le nom et le port de l’écouteur de DNN en tant que nom de serveur dans la chaîne de connexion pour se connecter à l’écouteur. 

La création de l’écouteur de DNN via l’interface graphique utilisateur de SQL Server Management Studio (SSMS) n’est pas prise en charge actuellement. 


## <a name="availability-groups-and-fci"></a>Groupes de disponibilité et instances FCI

Vous pouvez configurer un groupe de disponibilité AlwaysOn en utilisant une instance de cluster de basculement (failover cluster instance, FCI) en tant qu’un des réplicas. Pour que cette configuration fonctionne avec l’écouteur de DNN, l’[instance de cluster de basculement doit également utiliser le DNN](failover-cluster-instance-distributed-network-name-dnn-configure.md), car il n’existe aucun moyen de placer l’adresse IP virtuelle de celle-ci dans la liste d’adresses IP du DNN d’AG. 

Dans cette configuration, l’URL du point de terminaison de mise en miroir pour le réplica FCI doit utiliser le DNN de l’instance FCI. De même, si l’instance FCI est utilisée en tant que réplica en lecture seule, le routage en lecture seule vers le réplica FCI doit utiliser le DNN de l’instance FCI. 

Le format du point de terminaison de mise en miroir est le suivant : `ENDPOINT_URL = 'TCP://<FCI DNN DNS name>:<mirroring endpoint port>'`. 

Par exemple, si le nom DNS de votre DNN de FCI est `dnnlsnr` et que `5022` est le port du point de terminaison de mise en miroir de la FCI, l’extrait de code Transact-SQL (T-SQL) pour créer l’URL du point de terminaison ressemble à ceci : 

```sql
ENDPOINT_URL = 'TCP://dnnlsnr:5022'
```

De même, le format de l’URL de routage en lecture seule est le suivant : `TCP://<FCI DNN DNS name>:<SQL Server instance port>`. 

Par exemple, si votre nom DNS DNN est `dnnlsnr`et que `1444` est le port utilisé par l’instance FCI cible en lecture seule de SQL Server, l’extrait de code T-SQL pour créer l’URL de routage en lecture seule ressemble à ceci : 

```sql
READ_ONLY_ROUTING_URL = 'TCP://dnnlsnr:1444'
```

Vous pouvez omettre le port dans l’URL s’il s’agit du port 1433 par défaut. Pour une instance nommée, configurez un port statique pour l’instance nommée et spécifiez-le dans l’URL de routage en lecture seule.  

## <a name="distributed-availability-group"></a>Groupe de disponibilité distribué

Les groupes de disponibilité distribués ne sont actuellement pas pris en charge avec l’écouteur de DNN. 

## <a name="replication"></a>Réplication

Les réplications transactionnelles, de fusion et de capture instantanée prennent toutes en charge le remplacement de l’écouteur de VNN par l’écouteur de DNN et le port dans les objets de réplication qui se connectent à l’écouteur. 

Pour plus d’informations sur l’utilisation de la réplication avec des groupes de disponibilité, consultez [Éditeur et AG](/sql/database-engine/availability-groups/windows/configure-replication-for-always-on-availability-groups-sql-server), [Abonné et AG](/sql/database-engine/availability-groups/windows/replication-subscribers-and-always-on-availability-groups-sql-server), et [Distributeur and AG](/sql/relational-databases/replication/configure-distribution-availability-group).

## <a name="msdtc"></a>MSDTC

Les MSDTC tant local qu’en cluster sont pris en charge, mais le MSDTC utilise un port dynamique qui requiert un service Azure Load Balancer standard pour configurer le port de haute disponibilité (HA). Par conséquent, la machine virtuelle doit utiliser une réservation IP indispensable pour l’exposer à Internet. 

Définissez deux règles, l’une pour le port 135 du Mappeur de point de terminaison RPC et l’autre pour le port MSDTC réel. Après le basculement, modifiez la règle d’équilibreur de charge sur le nouveau port MSDTC après son changement sur le nouveau nœud. 

Si le MSDTC est local, veillez à autoriser les communications sortantes. 

## <a name="distributed-query"></a>Requête distribuée 

La requête distribuée s’appuie sur un serveur lié, qui peut être configuré à l’aide de l’écouteur de DNN d’AG et du port. Si le port n’est pas 1433, choisissez l’option **Utiliser une autre source de données** dans SQL Server Management Studio (SSMS) lors de la configuration de votre serveur lié. 

## <a name="filestream"></a>FileStream

La fonctionnalité FileStream est prise en charge, mais pas pour les scénarios où les utilisateurs accèdent au partage de fichiers étendu à l’aide de l’API de fichier Windows. 

## <a name="filetable"></a>Filetable

La fonctionnalité FileTable est prise en charge, mais pas pour les scénarios où les utilisateurs accèdent au partage de fichiers étendu à l’aide de l’API de fichier Windows. 

## <a name="linked-servers"></a>Serveurs liés

Configurez le serveur lié à l’aide du nom et du port d’écouteur de DNN d’AG. Si le port n’est pas 1433, choisissez l’option **Utiliser une autre source de données** dans SQL Server Management Studio (SSMS) lors de la configuration de votre serveur lié. 


## <a name="frequently-asked-questions"></a>Forum aux questions


- Quelle version de SQL Server prend en charge l’écouteur de DNN d’AG ? 

   SQL Server 2019 CU8 et versions ultérieures.

- Quelle est le temps de basculement prévu en cas d’utilisation de l’écouteur de DNN ?

   Pour l’écouteur de DNN, le temps de basculement est juste le temps de basculement d’AG, sans temps supplémentaire (tel le temps de sonde lorsque vous utilisez le service Azure Load Balancer).

- Existe-t-il des versions requises pour que les clients SQL prennent en charge le DNN avec OLEDB et ODBC ?

   Nous recommandons la prise en charge de la chaîne de connexion `MultiSubnetFailover=True` pour l’écouteur de DNN. Elle est disponible à partir de SQL Server 2012 (11.x).

- Des modifications de configuration de SQL Server sont-elles nécessaires pour pouvoir utiliser l’écouteur de DNN ? 

   Aucune modification de configuration de SQL Server n’est nécessaire pour utiliser le DNN, mais certaines fonctionnalités de SQL Server peuvent devoir être prises en compte. 

- Le DNN prend-il en charge les clusters qui ont plusieurs sous-réseaux ?

   Oui. Le cluster lie le DNN dans le DNS aux adresses IP physiques de tous les réplicas disponibles, quel que soit le sous-réseau. Le client SQL essaie toutes les adresses IP du nom DNS, quel que soit le sous-réseau. 



## <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations, consultez les pages suivantes : 

- [Technologies de cluster Windows](/windows-server/failover-clustering/failover-clustering-overview)   
- [Groupe de disponibilité AlwaysOn](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server)

