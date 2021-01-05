---
title: Problèmes courants - Azure Database Migration Service
description: Découvrez comment résoudre certains problèmes et erreurs connus courants liés à l’utilisation d’Azure Database Migration Service.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: troubleshooting
ms.date: 02/20/2020
ms.openlocfilehash: f4baca7f261aa7544b54992a5e1ddf620794774f
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94962278"
---
# <a name="troubleshoot-common-azure-database-migration-service-issues-and-errors"></a>Résoudre certains problèmes et erreurs courants liés à l’utilisation d’Azure Database Migration Service

Cet article décrit certains problèmes et erreurs courants que les utilisateurs d’Azure Database Migration Service sont susceptibles de rencontrer. L’article explique également comment résoudre ces problèmes et erreurs.

> [!NOTE]
> Communication sans stéréotype
>
> La diversité et l’inclusion sont au cœur des valeurs de Microsoft. Cet article contient des références au mot _esclave_. Le [guide de style de Microsoft sur la communication sans stéréotype](https://github.com/MicrosoftDocs/microsoft-style-guide/blob/master/styleguide/bias-free-communication.md) le reconnaît comme un mot à exclure. Le mot est utilisé dans cet article pour des raisons de cohérence, car il s’agit du mot qui figure dans le logiciel. Une fois que le mot aura été supprimé du logiciel, cet article sera mis à jour en conséquence.
>

## <a name="migration-activity-in-queued-state"></a>Activité de migration en attente

Quand vous créez des activités dans un projet Azure Database Migration Service, ces activités restent en attente.

| Cause         | Résolution |
| ------------- | ------------- |
| Ce problème se produit quand l’instance Azure Database Migration Service a atteint la capacité maximale de tâches en cours pouvant s’exécuter simultanément. Toute nouvelle activité est mise en file d’attente jusqu’à la libération de la capacité. | Vérifiez si l’instance Data Migration Service a des activités en cours d’exécution sur plusieurs projets. Vous pouvez continuer à créer des activités, lesquelles seront automatiquement ajoutées à la file d’attente avant d’être exécutées. Dès que l’une des activités en cours d’exécution est terminée, l’activité suivante dans la file d’attente commence à s’exécuter et elle passe automatiquement à l’état en cours d’exécution. Aucune action supplémentaire n’est requise de votre part pour démarrer la migration d’une activité en attente.<br><br> |

## <a name="max-number-of-databases-selected-for-migration"></a>Nombre maximal de bases de données sélectionnées pour la migration

L’erreur suivante se produit quand vous créez une activité dans un projet de migration de base de données vers Azure SQL Database ou vers une instance Azure SQL Managed Instance :

* **Erreur** : Erreur de validation des paramètres de migration", "errorDetail":« Plus de '4' objets 'Databases' ont été sélectionnés pour la migration, ce qui dépasse le nombre maximal. »

| Cause         | Résolution |
| ------------- | ------------- |
| Cette erreur s’affiche si vous avez sélectionné plus de quatre bases de données pour une seule activité de migration. Chaque activité de migration est actuellement limitée à quatre bases de données. | Sélectionnez au maximum quatre bases de données par activité de migration. Si vous devez migrer plus de quatre bases de données en parallèle, provisionnez une autre instance Azure Database Migration Service. Actuellement, chaque abonnement peut prendre en charge jusqu’à deux instances Azure Database Migration Service.<br><br> |

## <a name="errors-for-mysql-migration-to-azure-mysql-with-recovery-failures"></a>Erreurs pour la migration de MySQL vers Azure MySQL avec des échecs de récupération

Quand vous effectuez une migration de MySQL vers Azure Database pour MySQL à l’aide d’Azure Database Migration Service, l’activité de migration échoue avec l’erreur suivante :

* **Erreur** : Erreur de migration de base de données - La tâche 'TaskID' a été suspendue après [n] échecs de récupération successifs.

| Cause         | Résolution |
| ------------- | ------------- |
| Cette erreur peut se produire si l’utilisateur qui effectue la migration n’a pas le rôle ReplicationAdmin et/ou les privilèges REPLICATION CLIENT, REPLICATION REPLICA et SUPER (versions antérieures à MySQL 5.6.6).<br><br><br><br><br><br><br><br><br><br><br><br><br> | Vérifiez que les [privilèges nécessaires](./tutorial-mysql-azure-mysql-online.md#prerequisites) pour le compte d’utilisateur sont correctement configurés sur l’instance Azure Database pour MySQL. Par exemple, effectuez les étapes suivantes pour créer un utilisateur nommé 'migrateuser' avec les privilèges nécessaires :<br>1. Créer un utilisateur avec CREATE USER migrateuser@'%' IDENTIFIED BY 'secret' ; <br>2. Accorder tous les privilèges sur db_name.* à 'migrateuser'@'%' identifié par 'secret' ; // répéter cette étape pour accorder l’accès sur d’autres bases de données <br>3. Accorder le privilège de subordonné de réplication sur db_name *.* à 'migrateuser'@'%' identifié par 'secret' ;<br>4. Accorder le privilège de client de réplication sur db_name *.* à 'migrateuser'@'%' identifié par 'secret' ;<br>5. Supprimer (flush) les privilèges. |

## <a name="error-when-attempting-to-stop-azure-database-migration-service"></a>Erreur lors de la tentative d’arrêt d’Azure Database Migration Service

Vous recevez l’erreur suivante quand vous tentez d’arrêter l’instance Azure Database Migration Service :

* **Erreur** : Échec de l’arrêt du service. Erreur : {'error':{'code':'InvalidRequest','message':'Une ou plusieurs activités sont en cours d’exécution. Pour arrêter le service, attendez la fin des activités ou arrêtez-les manuellement, puis réessayez.'}}

| Cause         | Résolution |
| ------------- | ------------- |
| Cette erreur s’affiche quand l’instance de service que vous tentez d’arrêter inclut des activités qui sont encore en cours d’exécution ou qui sont présentes dans des projets de migration. <br><br><br><br><br><br> | Assurez-vous qu’il n’y a plus aucune activité en cours d’exécution dans l’instance Azure Database Migration Service que vous tentez d’arrêter. Vous pouvez également supprimer les activités ou les projets avant de tenter d’arrêter le service. Les étapes suivantes permettent de supprimer des projets pour nettoyer l’instance du service de migration en supprimant toutes les tâches en cours d’exécution :<br>1. Install-Module -Name AzureRM.DataMigration <br>2. Login-AzureRmAccount <br>3. Select-AzureRmSubscription -SubscriptionName "\<subName>" <br> 4. Remove-AzureRmDataMigrationProject -Name \<projectName> -ResourceGroupName \<rgName> -ServiceName \<serviceName> -DeleteRunningTask |

## <a name="error-when-attempting-to-start-azure-database-migration-service"></a>Erreur lors de la tentative de démarrage d’Azure Database Migration Service

Vous recevez l’erreur suivante quand vous tentez de démarrer l’instance Azure Database Migration Service :

* **Erreur** : Échec du démarrage du service. Erreur : {'errorDetail':'Le service n’a pas pu démarrer ; contactez le support Microsoft'}

| Cause         | Résolution |
| ------------- | ------------- |
| Cette erreur s’affiche après un échec interne de l’instance précédente. Cette erreur se produit rarement, mais l’équipe d’ingénierie en a connaissance. <br> | Supprimez l’instance du service qui ne démarre pas et provisionnez une nouvelle instance pour la remplacer. |

## <a name="error-restoring-database-while-migrating-sql-to-azure-sql-db-managed-instance"></a>Erreur lors de la restauration de la base de données pendant la migration de SQL Server vers une instance managée Azure SQL Database

Lorsque vous effectuez une migration en ligne de SQL Server vers Azure SQL Managed Instance, le basculement échoue avec l’erreur suivante :

* **Erreur** : L’opération de restauration a échoué pour l’opération avec l’ID 'operationId'. Code 'AuthorizationFailed', Message 'Le client 'clientId' avec l’ID d’objet 'objectId' n’a pas l’autorisation d’effectuer l’action 'Microsoft.Sql/locations/managedDatabaseRestoreAzureAsyncOperation/read' sur l’étendue '/subscriptions/subscriptionId'.'.

| Cause         | Résolution    |
| ------------- | ------------- |
| Cette erreur indique que le principal d’application utilisé pour la migration en ligne de SQL Server vers Azure SQL Managed Instance n’a pas l’autorisation de contribution sur l’abonnement. Certains appels d’API avec Managed Instance requièrent cette autorisation sur l’abonnement pour l’opération de restauration. <br><br><br><br><br><br><br><br><br><br><br><br><br><br> | Utilisez l’applet de commande PowerShell `Get-AzureADServicePrincipal` en spécifiant la valeur `-ObjectId` indiquée dans le message d’erreur pour obtenir le nom complet de l’ID d’application utilisé.<br><br> Vérifiez les autorisations configurées pour cette application et assurez-vous que cette dernière a le [rôle Contributeur](../role-based-access-control/built-in-roles.md#contributor) au niveau de l’abonnement. <br><br> L’équipe d’ingénierie Azure Database Migration Service travaille sur la restriction de l’accès requis par le rôle contributeur sur l’abonnement. Si vous avez une exigence métier qui n’autorise pas l’utilisation du rôle de contribution, contactez le support Azure pour obtenir de l’aide. |

## <a name="error-when-deleting-nic-associated-with-azure-database-migration-service"></a>Erreur lors de la suppression de la carte d’interface réseau associée à Azure Database Migration Service

Quand vous essayez de supprimer une carte d’interface réseau qui est associée à Azure Database Migration Service, la tentative de suppression échoue avec cette erreur :

* **Erreur** : Impossible de supprimer la carte d’interface réseau associée à Azure Database Migration Service, car le service DMS utilise cette carte

| Cause         | Résolution    |
| ------------- | ------------- |
| Ce problème se produit quand l’instance Azure Database Migration Service est toujours active et qu’elle utilise la carte d’interface réseau. <br><br><br><br><br><br><br><br> | Pour supprimer cette carte d’interface réseau, supprimez l’instance du service DMS, ce qui supprime automatiquement la carte d’interface réseau utilisée par le service.<br><br> **Important !** Assurez-vous que l’instance Azure Database Migration Service à supprimer n’a plus aucune activité en cours d’exécution.<br><br> Une fois que tous les projets et activités associés à l’instance Azure Database Migration Service ont été supprimés, vous pouvez supprimer l’instance du service. La carte d’interface réseau utilisée par l’instance du service est automatiquement nettoyée pendant le processus de suppression du service. |

## <a name="connection-error-when-using-expressroute"></a>Erreur de connexion durant l’utilisation d’ExpressRoute

Quand vous essayez de vous connecter à une source dans l’Assistant Projet Azure Database Migration Service, la connexion échoue après un délai d’attente trop long si la source utilise ExpressRoute pour la connectivité.

| Cause         | Résolution    |
| ------------- | ------------- |
| Quand [ExpressRoute](https://azure.microsoft.com/services/expressroute/) est utilisé, Azure Database Migration Service [requiert](./tutorial-sql-server-azure-sql-online.md) le provisionnement de trois points de terminaison de service sur le sous-réseau du réseau virtuel associé au service :<br> -- Point de terminaison Service Bus<br> -- Point de terminaison de stockage<br> -- Point de terminaison de base de données cible (un point de terminaison SQL ou Cosmos DB, par exemple)<br><br><br><br><br> | [Activez](./tutorial-sql-server-azure-sql-online.md) les points de terminaison de service requis pour la connectivité ExpressRoute entre la source et Azure Database Migration Service. <br><br><br><br><br><br><br><br> |

## <a name="lock-wait-timeout-error-when-migrating-a-mysql-database-to-azure-db-for-mysql"></a>Verrouiller l’erreur de délai d’expiration lors de la migration d’une base de données MySQL vers une base de données Azure pour MySQL

Quand vous migrez une base de données MySQL vers une instance Azure Database pour MySQL en utilisant Azure Database Migration Service, la migration échoue avec l’erreur de délai d’expiration de verrouillage suivante :

* **Erreur** : Erreur de migration de base de données - Échec du chargement du fichier - Échec du démarrage du processus de chargement du fichier 'n' RetCode: SQL_ERROR SqlState : HY000 NativeError: 1205 Message: [MySQL][ODBC Driver][mysqld] Dépassement du délai d’attente des verrous. Essayez de redémarrer la transaction

| Cause         | Résolution    |
| ------------- | ------------- |
| Cette erreur se produit quand la migration échoue en raison du délai d’attente des verrous pendant la migration. | Essayez d’augmenter la valeur du paramètre serveur **'innodb_lock_wait_timeout'** . La plus grande valeur autorisée est 1073741824. |

## <a name="error-connecting-to-source-sql-server-when-using-dynamic-port-or-named-instance"></a>Erreur de connexion au serveur SQL Server source lors de l’utilisation d’un port dynamique ou d’une instance nommée

Quand vous essayez de connecter Azure Database Migration Service à une source SQL Server qui s’exécute sur une instance nommée ou un port dynamique, la connexion échoue avec cette erreur :

* **Erreur** : -1 - La connexion SQL a échoué. Une erreur liée au réseau ou propre à une instance s’est produite lors de l’établissement d’une connexion à SQL Server. Le serveur est introuvable ou inaccessible. Vérifiez que le nom de l’instance est correct et que SQL Server est configuré pour autoriser les connexions à distance. (fournisseur : interfaces réseau SQL, erreur : 26 - Erreur lors de la localisation du serveur/de l’instance spécifiés)

| Cause         | Résolution    |
| ------------- | ------------- |
| Ce problème se produit quand l’instance SQL Server source à laquelle Azure Database Migration Service essaie de se connecter utilise un port dynamique ou une instance nommée. Le service SQL Server Browser écoute le port UDP 1434 pour les connexions entrantes vers une instance nommée, ou si un port dynamique est utilisé. Le port dynamique change potentiellement à chaque redémarrage du service SQL Server. Vérifiez quel port dynamique est affecté à une instance via la configuration réseau définie dans le Gestionnaire de configuration SQL Server.<br><br><br> |Vérifiez qu’Azure Database Migration Service peut se connecter au service SQL Server Browser source sur le port UDP 1434 et à l’instance SQL Server par le port TCP affecté dynamiquement selon le cas. |

## <a name="additional-known-issues"></a>Autres problèmes connus

* [Problèmes connus/limitations de migration dans le cadre des migrations en ligne vers Azure SQL Database](./known-issues-azure-sql-online.md)
* [Problèmes connus/limitations de migration dans le cadre des migrations en ligne vers Azure Database pour MySQL](./known-issues-azure-mysql-online.md)
* [Problèmes connus/limitations de migration dans le cadre des migrations en ligne vers Azure Database pour PostgreSQL](./known-issues-azure-postgresql-online.md)

## <a name="next-steps"></a>Étapes suivantes

* Consultez l’article [PowerShell pour Azure Database Migration Service](/powershell/module/azurerm.datamigration/?view=azurermps-6.13.0#data_migration).
* Consultez l’article [Guide pratique pour configurer des paramètres de serveur dans Azure Database pour MySQL à l’aide du portail Azure](../mysql/howto-server-parameters.md).
* Consultez l’article [Vue d’ensemble des prérequis pour l’utilisation d’Azure Database Migration Service](./pre-reqs.md).
* Consultez les [questions fréquentes (FAQ) sur l’utilisation d’Azure Database Migration Service](./faq.md).