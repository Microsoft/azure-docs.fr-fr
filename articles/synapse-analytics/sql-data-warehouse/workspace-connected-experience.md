---
title: Activation des fonctionnalités de l’espace de travail Synapse sur un pool SQL dédié (anciennement SQL DW)
description: Ce document décrit comment un client peut accéder à son instance autonome SQL DW existante et l’utiliser dans l’espace de travail.
services: synapse-analytics
author: antvgski
manager: igorstan
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/23/2020
ms.author: anvang
ms.reviewer: jrasnick
ms.openlocfilehash: 5efb1df378df323585bc0ca1094451cdb095fe4e
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499779"
---
# <a name="enabling-synapse-workspace-features-on-an-existing-dedicated-sql-pool-formerly-sql-dw"></a>Activation des fonctionnalités de l’espace de travail Synapse sur un pool SQL dédié existant (anciennement SQL DW)

Tous les clients de l’entrepôt de données SQL peuvent désormais accéder à une instance de pool SQL dédié (anciennement SQL DW) et l’utiliser à l’aide de Synapse Studio et de l’espace de travail, sans que cela n’ait d’impact sur l’automatisation, les connexions ou les outils. Cet article explique comment un client Azure Synapse Analytics existant peut se baser sur sa solution Analytics existante et la développer en tirant parti des nouvelles fonctionnalités riches désormais disponibles via l’espace de travail Synapse et Studio.   

## <a name="experience"></a>Expérience
 
Maintenant que l’espace de travail Synapse est en disponibilité générale, une nouvelle fonctionnalité est disponible dans la section Vue d’ensemble du portail DW qui vous permet de créer un espace de travail Synapse pour vos instances de pool SQL (anciennement SQL DW) dédiées existantes. Cette nouvelle fonctionnalité vous permet de connecter le serveur logique qui héberge vos instances d’entrepôt de données existantes à un nouvel espace de travail Synapse. La connexion garantit que tous les entrepôts de données hébergés sur ce serveur sont rendus accessibles à partir de l’espace de travail et de Studio et peuvent être utilisés conjointement avec les services partenaires Synapse (pool SQL serverless, pool Apache Spark et ADF). Vous pouvez commencer à accéder à vos ressources et à les utiliser dès que les étapes de provisionnement sont terminées et que la connexion a été établie avec l’espace de travail nouvellement créé.  
:::image type="content" source="media/workspace-connected-overview/workspace-connected-dw-portal-overview-pre-create.png" alt-text="Espace de travail Synapse connecté":::

## <a name="using-synapse-workspace-and-studio-features-to-access-and-use-a-dedicated-sql-pool-formerly-sql-dw"></a>Utilisation d’un espace de travail Synapse et de fonctionnalités Studio pour accéder à un pool SQL dédié (anciennement SQL DW) et l’utiliser
 
Les informations suivantes s’appliquent lors de l’utilisation d’un DW SQL dédié (anciennement SQL DW) avec les fonctionnalités de l’espace de travail Synapse activées : 
- **Fonctionnalités SQL** Toutes les fonctionnalités SQL sont conservées sur le serveur SQL logique après l’activation de la fonctionnalité de l’espace de travail synapse. L’accès au serveur via le fournisseur de ressources SQL sera toujours possible une fois l’espace de travail activé. Toutes les fonctions de gestion peuvent être lancées par le biais de l’espace de travail et l’opération est effectuée sur le serveur SQL Server logique hébergeant vos pools SQL. L’automatisation, les outils ou les connexions existant(e)s ne sont pas interrompus lorsqu’un espace de travail est activé.  
- **Déplacement de ressources**  Le lancement d’un déplacement de ressources sur un serveur avec la fonctionnalité d’espace de travail Synapse activée entraîne la rupture du lien entre le serveur et l’espace de travail et vous ne pourrez plus accéder à vos instances de pool SQL dédié (anciennement SQL DW) existantes à partir de l’espace de travail. Pour garantir que la connexion est maintenue, il est recommandé de conserver les deux ressources dans le même abonnement et le même groupe de ressources. 
- **Supervision** Les requêtes SQL soumises par le biais de Synapse Studio dans un pool SQL dédié (anciennement SQL DW) activé sur l’espace de travail peuvent être consultées dans le hub Monitor. Pour toutes les autres activités de supervision, vous pouvez accéder à la supervision des pool SQL dédiés (anciennement SQL DW) sur le portail Azure. 
- **Sécurité** et **Contrôle d’accès** Comme indiqué ci-dessus, toutes les fonctions de gestion de votre serveur SQL Server et les instances de pools SQL dédiés (anciennement SQL DW) continuent de résider sur le serveur SQL logique. Ces fonctions incluent la gestion des règles de pare-feu, la configuration de l’administrateur Azure AD du serveur et le contrôle de l’accès aux données de votre pool SQL dédié (anciennement SQL DW). Les étapes suivantes doivent être effectuées pour s’assurer que votre pool SQL dédié (anciennement SQL DW) est accessible et peut être utilisé par le biais de l’espace de travail Synapse. Les appartenances au rôle d’espace de travail ne donnent pas aux utilisateurs des autorisations d’accès aux données dans les instances de pool SQL dédié (anciennement SQL DW). Pour garantir que les utilisateurs peuvent accéder aux instances de pool SQL dédié (anciennement SQL DW) sur le serveur logique, suivez vos stratégies [d’authentification SQL](sql-data-warehouse-authentication.md) habituelles. 

    ```sql
    CREATE USER [<workspace managed identity] FROM EXTERNAL PROVIDER 
    GRANT CONTROL ON DATABASE:: <dedicated SQL pool name> TO [<workspace managed identity>
    ```

    > [!NOTE] 
    > L’espace de travail connecté de Synapse Studio affiche les noms des pools dédiés en fonction des autorisations dont dispose l’utilisateur dans Azure. Les objets dans les pools ne sont pas accessibles si l’utilisateur n’a pas d’autorisations d’accès aux pools SQL. 

- **Sécurité réseau** Si l’espace de travail Synapse que vous avez activé sur votre pool SQL dédié existant (anciennement SQL DW) est activé pour la protection contre l’infiltration de données. Créez une connexion de point de terminaison privée managée à partir de l’espace de travail vers le serveur SQL logique. Approuvez la demande de connexion de point de terminaison privée pour autoriser les communications entre le serveur et l’espace de travail.
- Pools SQL **Studio** dans le **hub Données** Un pool SQL dédié (anciennement SQL DW) activé sur l’espace de travail peut être identifié par le biais de l’info-bulle dans le hub de données. 
- **Création d’un pool SQL dédié (anciennement SQL DW)** Il est possible de créer de nouveaux pools SQL dédiés via l’espace de travail Synapse et Studio une fois que la fonctionnalité d’espace de travail a été activée et le provisionnement d’un nouveau pool aura lieu sur le serveur SQL logique. Les nouvelles ressources s’affichent dans le portail et Studio lorsque le provisionnement est terminé.      

## <a name="next-steps"></a>Étapes suivantes
Activer des [fonctionnalités d’espace de travail Synapse](workspace-connected-create.md) sur votre pool SQL dédié (anciennement SQL DW) existant
