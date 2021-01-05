---
title: Configurer la réplication sur Azure SQL Edge
description: Découvrez la configuration de la réplication sur Azure SQL Edge.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: 3b424ece9207328d87068160f78ebc78a3bd1a8d
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93395221"
---
# <a name="configure-replication-to-azure-sql-edge"></a>Configurer la réplication sur Azure SQL Edge 

Vous pouvez configurer une instance d’Azure SQL Edge en tant que titulaire d’un abonnement par émission de données pour la réplication transactionnelle unidirectionnelle ou la réplication de capture instantanée. Cette instance ne peut pas faire office de serveur de publication ou serveur de distribution pour une configuration de réplication transactionnelle. Notez qu’Azure SQL Edge ne prend pas en charge la réplication de fusion, la réplication d’égal à égal ou la publication Oracle.

## <a name="supported-configurations"></a>Configurations prises en charge
  
- L’instance d’Azure SQL Edge doit être titulaire d’un abonnement par émission de données pour un serveur de publication.
- Le serveur de publication et le serveur de distribution peuvent être :
   - Une instance de SQL Server exécutée localement ou une instance de SQL Server exécutée sur une machine virtuelle Azure. Pour plus d’informations, consultez [Présentation de SQL Server sur les machines virtuelles Azure](../azure-sql/virtual-machines/index.yml). Les instances de SQL Server doivent utiliser une version ultérieure à SQL Server 2016.
   - Une instance d’Azure SQL Managed Instance. SQL Managed Instance peut héberger des bases de données de serveur de publication, de serveur de distribution et d’abonné. Pour plus d’informations, consultez [Réplication avec SQL Database Managed Instance](/azure/sql-database/replication-with-sql-database-managed-instance/).

- La base de données de distribution et les agents de réplication ne peuvent pas être placés sur une instance d’Azure SQL Edge.  

> [!NOTE]
> Si vous tentez de configurer la réplication à l’aide d’une version non prise en charge, vous risquez de recevoir les deux erreurs suivantes : MSSQL_REPL20084 (« Le processus n’a pas pu se connecter à l’abonné. ») et MSSQL_REPL40532 (« Impossible d’ouvrir le serveur \<name> demandé par la connexion. La connexion a échoué. »).  

## <a name="remarks"></a>Notes

La configuration requise et les bonnes pratiques suivantes sont importantes à connaître lors de la configuration de la réplication :

- Vous pouvez configurer la réplication à l’aide de [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms). Pour ce faire, vous pouvez également exécuter des instructions Transact-SQL sur le serveur de publication, à l’aide de SQL Server Management Studio ou d’[Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio).
- Pour effectuer une réplication vers une instance d’Azure SQL Edge, vous devez utiliser l’authentification SQL Server afin de vous connecter.
- Les tables répliquées doivent avoir une clé primaire.
- Une même publication sur SQL Server peut à la fois prendre en charge les abonnés d'Azure SQL Edge et de SQL Server (localement et sur une machine virtuelle Azure).  
- La gestion, la supervision et le dépannage de la réplication doivent être effectués à partir de l’instance de SQL Server.  
- Seuls les abonnements par émission de données à Azure SQL Edge sont pris en charge.  
- Seul `@subscriber_type = 0` est pris en charge dans la procédure stockée `sp_addsubscription` pour Azure SQL Edge.  
- Azure SQL Edge ne prend pas en charge les réplications bidirectionnelles, immédiates, actualisables et de pair à pair.
- Azure SQL Edge ne prend en charge qu’un sous-ensemble des fonctionnalités disponibles dans SQL Server ou SQL Managed Instance. Si vous tentez de répliquer une base de données (ou des objets dans la base de données) qui contient une ou plusieurs fonctionnalités non prises en charge, la tentative échoue. Par exemple, si vous tentez de répliquer une base de données qui contient des objets avec des types de données spatiales, vous recevez une erreur. Pour plus d’informations, consultez [Fonctionnalités prises en charge par Azure SQL Edge](features.md).

## <a name="initialize-reference-data-on-an-instance-of-azure-sql-edge"></a>Initialiser des données de référence sur une instance d’Azure SQL Edge

Vous pouvez initialiser votre instance avec des données de référence qui changent au fil du temps. Par exemple, vous souhaiterez peut-être mettre à jour les modèles de machine learning sur votre instance d’Azure SQL Edge, une fois qu’ils ont été entraînés sur une instance de SQL Server. Voici comment initialiser votre instance dans un tel scénario :

1. Créez une publication de réplication transactionnelle sur une base de données SQL Server.  
2. Sur l’instance de SQL Server, utilisez l’ **Assistant Nouvel abonnement** ou des instructions Transact-SQL pour créer une émission de données sur l’abonnement à Azure SQL Edge.  
3. Vous pouvez initialiser la base de données répliquée sur Azure SQL Edge à l’aide d’une capture instantanée générée par l’agent d’instantané et distribuée par l’agent de distribution. Vous pouvez également effectuer l’initialisation à l’aide d’une sauvegarde de la base de données à partir du serveur de publication. Gardez à l’esprit que si la sauvegarde de la base de données contient des objets ou fonctionnalités non pris en charge par Azure SQL Edge, l’opération de restauration échoue.

## <a name="limitations"></a>Limites

Les options suivantes ne sont pas prises en charge par les abonnements Azure SQL Edge :

- Copier les associations de groupes de fichiers  
- Copier les schémas de partition de table  
- Copier les schémas de partition d’index  
- Copier les statistiques définies par l’utilisateur  
- Copier les liaisons par défaut  
- Copier les liaisons de règle  
- Copier les index de recherche en texte intégral  
- Copier le schéma XSD XML  
- Copier les index XML  
- Copier les autorisations  
- Copier les index spatiaux  
- Copier les index filtrés  
- Copier l’attribut de compression de données  
- Copier l’attribut de colonne éparse  
- Copier les types suivants : filestream, `hierarchyid` ou données spatiales
- Convertir le type `hierarchyid` en types de données MAX  
- Convertir le type spatial en types de données MAX  
- Copier les propriétés étendues  
- Copier les autorisations  

## <a name="examples"></a>Exemples

Créez une publication et un abonnement par émission de données. Pour plus d'informations, consultez les pages suivantes :
  
- [Créer une publication](/sql/relational-databases/replication/publish/create-a-publication)
- [Créez un abonnement par émission de données](/sql/relational-databases/replication/create-a-push-subscription/) en utilisant le nom et l’adresse IP du serveur Azure SQL Edge comme abonné (par exemple, **myEdgeinstance, 1433** ) et un nom de base de données de l’instance d’Azure SQL Edge comme base de données de destination (par exemple, **AdventureWorks** ).  

## <a name="next-steps"></a>Étapes suivantes  

- [Créer une publication](/sql/relational-databases/replication/publish/create-a-publication)
- [Créer un abonnement par émission (push)](/sql/relational-databases/replication/create-a-push-subscription/)
- [Types de réplication](/sql/relational-databases/replication/types-of-replication)
- [Supervision (réplication)](/sql/relational-databases/replication/monitor/monitoring-replication)
- [Initialiser un abonnement](/sql/relational-databases/replication/initialize-a-subscription)