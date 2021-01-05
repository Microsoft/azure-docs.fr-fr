---
title: 'Démarrage rapide : Fivetran et pool SQL dédié (anciennement SQL DW)'
description: Démarrez avec Fivetran et un pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics.
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 10/12/2018
ms.author: martinle
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: f332c3b0bd53d80d4a8471f53c56ecab611787c1
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96456369"
---
# <a name="quickstart-fivetran-with-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Démarrage rapide : Fivetran avec un pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics 

Ce guide de démarrage rapide décrit comment configurer un nouvel utilisateur Fivetran pour travailler avec un pool SQL dédié (anciennement SQL DW). Cet article suppose que vous disposez d’un pool SQL dédié (anciennement SQL DW) existant.

## <a name="set-up-a-connection"></a>Configurer une connexion

1. Recherchez le nom complet du serveur et le nom de la base de données utilisés pour vous connecter à votre pool SQL dédié (anciennement SQL DW).
    
    Si vous avez besoin d’aide pour trouver ces informations, consultez [Vous connecter à votre pool SQL dédié (anciennement SQL DW)](sql-data-warehouse-connection-strings.md).

2. Dans l’Assistant Installation, indiquez si vous souhaitez vous connecter directement à votre base de données ou en utilisant un tunnel SSH.

   Si vous choisissez de vous connecter directement à votre base de données, vous devez créer une règle de pare-feu pour autoriser l’accès. Cette méthode est la plus simple et la plus sécurisée.

   Si vous choisissez de vous connecter en utilisant un tunnel SSH, Fivetran se connecte à un serveur distinct sur votre réseau. Le serveur fournit un tunnel SSH vers votre base de données. Vous devez utiliser cette méthode si votre base de données se trouve dans un sous-réseau inaccessible d’un réseau virtuel.

3. Ajoutez l’adresse IP **52.0.2.4** à votre pare-feu au niveau du serveur pour autoriser les connexions entrantes à votre instance du pool SQL dédié (anciennement SQL DW) à partir de Fivetran.

   Pour plus d’informations, consultez [Créer une règle de pare-feu au niveau du serveur](create-data-warehouse-portal.md#create-a-server-level-firewall-rule).

## <a name="set-up-user-credentials"></a>Configurer les informations d’identification de l’utilisateur

1. Connectez-vous à votre pool SQL dédié (anciennement SQL DW) à l’aide de SQL Server Management Studio (SSMS) ou de l’outil que vous préférez. Connectez-vous en tant qu’un utilisateur administrateur du serveur. Ensuite, exécutez les commandes SQL suivantes afin de créer un utilisateur pour Fivetran :

    - Dans la base de données master : 
    
      ```sql
      CREATE LOGIN fivetran WITH PASSWORD = '<password>'; 
      ```

    - Dans la base de données du pool SQL dédié (anciennement SQL DW) :

      ```sql
      CREATE USER fivetran_user_without_login without login;
      CREATE USER fivetran FOR LOGIN fivetran;
      GRANT IMPERSONATE on USER::fivetran_user_without_login to fivetran;
      ```

2. Accordez à l’utilisateur Fivetran les autorisations suivantes sur votre pool SQL dédié (anciennement SQL DW) :

    ```sql
    GRANT CONTROL to fivetran;
    ```

    L’autorisation CONTROL est requise pour créer des informations d’identification incluses dans l’étendue de la base de données qui sont utilisées lorsqu’un utilisateur charge des fichiers à partir du Stockage Blob Azure à l’aide de PolyBase.

3. Ajoutez une classe de ressources appropriée à l’utilisateur Fivetran. La classe de ressources que vous utilisez dépend de la mémoire qui est requise pour créer un index columnstore. Par exemple, les intégrations avec des produits telles que Marketo et Salesforce nécessitent une classe de ressources supérieure en raison du grand nombre de colonnes et du plus grand volume de données utilisées par les produits. Une classe de ressources supérieure nécessite davantage de mémoire pour créer des index columnstore.

    Nous vous recommandons d’utiliser les classes de ressources statiques. Vous pouvez démarrer avec la classe de ressources `staticrc20`. La classe de ressources `staticrc20` alloue 200 Mo alloue pour chaque utilisateur, quel que soit le niveau de performance que vous utilisez. Si l’indexation columnstore échoue au niveau de classe de ressources actuelle, augmentez la classe de ressources.

    ```sql
    EXEC sp_addrolemember '<resource_class_name>', 'fivetran';
    ```

    Pour plus d’informations, consultez les documents relatifs aux [limites de mémoire et de concurrence](memory-concurrency-limits.md) et aux [classes de ressources](sql-data-warehouse-memory-optimizations-for-columnstore-compression.md#ways-to-allocate-more-memory).


## <a name="connect-from-fivetran"></a>Se connecter à partir de Fivetran

Pour vous connecter à votre pool SQL dédié (anciennement SQL DW) à partir de votre compte Fivetran, entrez les informations d’identification que vous utilisez pour accéder à votre pool SQL dédié (anciennement SQL DW) : 

* Hôte (nom de votre serveur).
* Port.
* Base de données.
* Utilisateur (le nom d’utilisateur doit être **fivetran\@_nom_serveur_** où *nom_serveur* fait partie de votre URI d’hôte Azure : **_nom\_serveur_.database.windows.net**).
* Password.
