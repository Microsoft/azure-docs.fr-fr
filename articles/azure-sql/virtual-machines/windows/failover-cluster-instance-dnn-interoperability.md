---
title: Interopérabilité des fonctionnalités avec l’instance FCI de SQL Server et DNN
description: 'En savoir plus sur les considérations supplémentaires à prendre en compte lors de l’utilisation de certaines fonctionnalités de SQL Server et d’une ressource de nom de réseau distribué (DNN) avec une instance de cluster de basculement sur SQL Server sur des machines virtuelles Azure. '
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
ms.date: 06/02/2020
ms.author: mathoma
ms.openlocfilehash: 3c92aa3b35240831fad14919dc73609d803c610a
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358212"
---
# <a name="feature-interoperability-with-sql-server-fci--dnn"></a>Interopérabilité des fonctionnalités avec l’instance FCI de SQL Server et DNN
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Certaines fonctionnalités de SQL Server reposent sur un nom de réseau virtuel (VNN) codé en dur. Par conséquent, lors de l’utilisation de la ressource de nom de réseau distribué (DNN) avec votre instance de cluster de basculement et SQL Server sur des machines virtuelles Azure, certaines considérations supplémentaires doivent être prises en compte. 

Dans cet article, vous apprendrez à configurer l’alias réseau lors de l’utilisation de la ressource DNN, ainsi que les fonctionnalités de SQL Server qui nécessitent un examen supplémentaire.

## <a name="create-network-alias-fci"></a>Créer un alias réseau (FCI)

Certains composants côté serveur s’appuient sur une valeur VNN codée en dur et nécessitent un alias réseau qui mappe le VNN au nom DNS DNN pour fonctionner correctement. Suivez les étapes décrites dans [Créer un alias de serveur](/sql/database-engine/configure-windows/create-or-delete-a-server-alias-for-use-by-a-client) pour créer un alias qui mappe VNN au nom DNS DNN. 

Pour une instance par défaut, vous pouvez mapper le VNN directement au nom DNS DNN , de telle sorte que VNN = nom DNS DNN.
Par exemple, si le nom VNN est `FCI1`, le nom de l’instance est `MSSQLSERVER` et le DNN est `FCI1DNN` (les clients se connectaient précédemment à `FCI`, et à présent ils se connectent à `FCI1DNN`), puis mappent le VNN `FCI1` au DNN `FCI1DNN`. 

Pour une instance nommée, le mappage d’alias réseau doit être effectué pour l’instance complète, de sorte que `VNN\Instance` = `DNN\Instance`. Par exemple, si le nom VNN est `FCI1`, le nom de l’instance est `instA` et le DNN est `FCI1DNN` (les clients se connectaient précédemment à `FCI1\instA`, et à présent ils se connectent à `FCI1DNN\instaA`), puis mappent le VNN `FCI1\instaA` au DNN `FCI1DNN\instaA`. 



## <a name="client-drivers"></a>Pilotes clients

Pour les pilotes ODBC, OLEDB, ADO.NET, JDBC, PHP et Node.js, les utilisateurs doivent spécifier explicitement le nom DNS DNN comme nom de serveur dans la chaîne de connexion. Pour garantir une connectivité rapide lors du basculement, ajoutez `MultiSubnetFailover=True` à la chaîne de connexion si le client SQL le prend en charge. 

## <a name="tools"></a>Outils

Les utilisateurs de [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms), [sqlcmd](/sql/tools/sqlcmd-utility), [Azure Data Studio](/sql/azure-data-studio/what-is) et [SQL Server Data Tools](/sql/ssdt/sql-server-data-tools) doivent spécifier explicitement le nom DNS DNN comme nom de serveur dans la chaîne de connexion. 

## <a name="availability-groups-and-fci"></a>Groupes de disponibilité et instances FCI

Vous pouvez configurer un groupe de disponibilité Always On avec une instance de cluster de basculement en tant qu’un des réplicas. Dans cette configuration, l’URL du point de terminaison de mise en miroir pour le réplica FCI doit utiliser le DNN de l’instance FCI. De même, si l’instance FCI est utilisée en tant que réplica en lecture seule, le routage en lecture seule vers le réplica FCI doit utiliser le DNN de l’instance FCI. 

Le format du point de terminaison de mise en miroir est le suivant : `ENDPOINT_URL = 'TCP://<DNN DNS name>:<mirroring endpoint port>'`. 

Par exemple, si votre nom DNS DNN est `dnnlsnr`et que `5022` est le port du point de terminaison de mise en miroir de l’instance FCI, l’extrait de code Transact-SQL (T-SQL) pour créer l’URL du point de terminaison ressemble à ceci : 

```sql
ENDPOINT_URL = 'TCP://dnnlsnr:5022'
```

De même, le format de l’URL de routage en lecture seule est le suivant : `TCP://<DNN DNS name>:<SQL Server instance port>`. 

Par exemple, si votre nom DNS DNN est `dnnlsnr`et que `1444` est le port utilisé par l’instance FCI cible en lecture seule de SQL Server, l’extrait de code T-SQL pour créer l’URL de routage en lecture seule ressemble à ceci : 

```sql
READ_ONLY_ROUTING_URL = 'TCP://dnnlsnr:1444'
```

Vous pouvez omettre le port dans l’URL s’il s’agit du port 1433 par défaut. Pour une instance nommée, configurez un port statique pour l’instance nommée et spécifiez-le dans l’URL de routage en lecture seule.  

## <a name="replication"></a>Réplication

La réplication comprend trois composants : le serveur de publication, le serveur de distribution et l’abonné. Chacun de ces composants peut être une instance de cluster de basculement. Étant donné que le VNN de l’instance FCI est fortement utilisé dans la configuration de la réplication, à la fois explicitement et implicitement, un alias réseau qui mappe le VNN au DNN peut être nécessaire pour que la réplication fonctionne. 

Continuez d’utiliser le nom VNN comme nom de l’instance FCI dans la réplication, mais créez un alias réseau dans les situations distantes suivantes *avant de configurer la réplication* :

| **Composant de réplication (instance FCI avec DNN)** | **Composant distant** | **Mappage d’alias réseau** | **Serveur avec mappage réseau**| 
|---------|---------|---------|-------- | 
|Serveur de publication | Serveur de distribution | VNN du serveur de publication vers DNN du serveur de publication| Serveur de distribution| 
|Serveur de distribution|Abonné |VNN du serveur de distribution vers DNN du serveur de distribution| Abonné | 
|Serveur de distribution|Serveur de publication | VNN du serveur de distribution vers DNN du serveur de distribution | Serveur de publication| 
|Abonné| Serveur de distribution| VNN de l’abonné vers DNN de l’abonné | Serveur de distribution| 

Par exemple, supposons que vous disposez d’un serveur de publication configuré en tant qu’instance FCI à l’aide du DNN dans une topologie de réplication et que le serveur de distribution est distant. Dans ce cas, créez un alias réseau sur le serveur de distribution pour mapper le VNN du serveur de publication au DNN du serveur de publication : 

:::image type="content" source="media/failover-cluster-instance-dnn-interoperability/alias-in-configuration-manager.png" alt-text="Configurez le nom DNS DNN comme alias réseau à l’aide du gestionnaire de configuration SQL Server." :::

Utilisez le nom complet de l’instance pour une instance nommée, comme dans l’exemple d’image suivant : 

:::image type="content" source="media/failover-cluster-instance-dnn-interoperability/alias-named-instance-configuration-manager.png" alt-text="Utilisez le nom complet de l’instance lors de la configuration d’un alias réseau pour une instance nommée." :::

## <a name="database-mirroring"></a>Mise en miroir de bases de données

Vous pouvez configurer la mise en miroir de bases de données avec une instance FCI en tant que partenaire de mise en miroir de bases de données. Configurez-le à l’aide de [Transact-SQL (T-SQL)](/sql/database-engine/database-mirroring/example-setting-up-database-mirroring-using-windows-authentication-transact-sql) au lieu de l’interface graphique utilisateur SQL Server Management Studio. L’utilisation de T-SQL permet de s’assurer que le point de terminaison de mise en miroir de bases de données est créé à l’aide du DNN au lieu du VNN. 

Par exemple, si votre nom DNS DNN est `dnnlsnr`et que le point de terminaison de mise en miroir de bases de données est 7022, l’extrait de code T-SQL suivant configure le partenaire de mise en miroir de bases de données : 

```sql
ALTER DATABASE AdventureWorks
    SET PARTNER =
    'TCP://dnnlsnr:7022'
GO 
```

Pour l’accès client, la propriété du **partenaire de basculement** peut gérer le basculement de la mise en miroir de bases de données, mais pas le basculement de l’instance FCI. 

## <a name="msdtc"></a>MSDTC

L’instance FCI peut participer à des transactions distribuées coordonnées par l'aide de Microsoft Distributed Transaction Coordinator (MSDTC). Bien que MSDTC en cluster et MSDTC local soient pris en charge avec le DNN de l’instance FCI, dans Azure, un équilibreur de charge est toujours nécessaire pour le MSDTC en cluster. Même lorsque le DNN est défini dans l’instance FCI Azure Load Balancer pour le MSDTC en cluster dans Azure est toujours exigé. 

## <a name="filestream"></a>FileStream

Bien que FileStream soit pris en charge pour une base de données dans une instance FCI, l’accès à FileStream ou à FileTable à l’aide des API du système de fichiers avec DNN n’est pas pris en charge. 

## <a name="linked-servers"></a>Serveurs liés

L’utilisation d’un serveur lié avec un DNN d’instance FCI est prise en charge. Utilisez le DNN directement pour configurer un serveur lié, ou utilisez un alias réseau pour mapper le VNN au DNN. 


Par exemple, pour créer un serveur lié avec le nom DNS DNN `dnnlsnr` pour l’instance nommée `insta1`, utilisez la commande Transact-SQL (T-SQL) suivante :

```sql
USE [master]   
GO   

EXEC master.dbo.sp_addlinkedserver    
    @server = N'dnnlsnr\inst1',    
    @srvproduct=N'SQL Server' ;   
GO 
```

Vous pouvez également créer le serveur lié à l’aide du nom de réseau virtuel (VNN), mais vous devrez alors définir un alias réseau pour mapper le VNN au DNN. 

Par exemple, pour le nom d’instance `insta1`, le nom VNN `vnnname` et le nom DNN `dnnlsnr`, utilisez la commande Transact-SQL (T-SQL) suivante pour créer un serveur lié à l’aide du VNN :

```sql
USE [master]
GO   

EXEC master.dbo.sp_addlinkedserver   
    @server = N'vnnname\inst1',    
    @srvproduct=N'SQL Server' ;   
GO 

```

Créez ensuite un alias réseau pour mapper `vnnname\insta1` à `dnnlsnr\insta1`. 



## <a name="frequently-asked-questions"></a>Forum aux questions


- À partir de quelle version de SQL Server le DNN est-il pris en charge ? 

   SQL Server 2019 CU2 et versions ultérieures.

- Quel est le temps de basculement attendu quand le DNN est utilisé ?

   Pour le DNN, le temps de basculement est juste le temps de basculement de l’instance FCI, sans aucun temps supplémentaire (comme le temps de probe lorsque vous utilisez Azure Load Balancer).

- Existe-t-il des versions requises pour que les clients SQL prennent en charge le DNN avec OLEDB et ODBC ?

   Nous recommandons la prise en charge de la chaîne de connexion `MultiSubnetFailover=True` pour le DNN. Elle est disponible à partir de SQL Server 2012 (11.x).

- Des modifications de la configuration de SQL Server sont-elles nécessaires pour que je puisse utiliser le DNN ? 

   Aucune modification de configuration de SQL Server n’est nécessaire pour utiliser le DNN, mais certaines fonctionnalités de SQL Server peuvent devoir être prises en compte. 

- Le DNN prend-il en charge les clusters qui ont plusieurs sous-réseaux ?

   Oui. Le cluster lie le DNN dans le DNS aux adresses IP physiques de tous les nœuds du cluster, quel que soit le sous-réseau. Le client SQL essaie toutes les adresses IP du nom DNS, quel que soit le sous-réseau. 



## <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations, consultez les pages suivantes : 

- [Technologies de cluster Windows](/windows-server/failover-clustering/failover-clustering-overview)   
- [Instances de cluster de basculement SQL Server](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)

