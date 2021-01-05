---
title: 'Azure AD Connect : Comment récupérer suite à une erreur liée à la limite de 10 Go de base de données locale | Microsoft Docs'
description: Cette rubrique décrit comment récupérer le service de synchronisation Azure AD Connect lorsque celui-ci rencontre un problème lié à la limite de 10 Go de base de données locale.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 41d081af-ed89-4e17-be34-14f7e80ae358
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 07/17/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4b45decd2f2cf9c99cffb0e08d4d6a5c5cfafc67
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96858397"
---
# <a name="azure-ad-connect-how-to-recover-from-localdb-10-gb-limit"></a>Azure AD Connect : Comment récupérer depuis la limite de 10 Go de base de données locale
Azure AD Connect nécessite une base de données SQL Server pour stocker les données d’identité. Vous pouvez utiliser la base de données locale par défaut de SQL Server 2012 installée avec Azure AD Connect ou utiliser votre propre base de données SQL complète. SQL Server Express impose une limite de taille de 10 Go. Lorsque vous utilisez la base de données locale et que cette limite est atteinte, le service de synchronisation Azure AD Connect ne peut plus démarrer ou se synchroniser correctement. Cet article indique les étapes de récupération.

## <a name="symptoms"></a>Symptômes
Il existe deux symptômes courants :

* Le service de synchronisation Azure AD Connect **est en cours d’exécution** mais ne parvient pas à se synchroniser avec l’erreur *stopped-database-disk-full*.

* Le service de synchronisation Azure AD Connect **ne peut pas démarrer**. Lorsque vous tentez de démarrer le service, celui-ci échoue avec l’événement 6323 et le message d’erreur *Le serveur a rencontré une erreur en raison d’un manque d’espace disque pour SQL Server*.

## <a name="short-term-recovery-steps"></a>Étapes de récupération à court terme
Cette section indique les étapes permettant de récupérer de l’espace de base de données requis pour que le service de synchronisation Azure AD Connect puisse reprendre l’opération. Procédez comme suit :
1. [Déterminer l’état du service de synchronisation](#determine-the-synchronization-service-status)
2. [Réduire la base de données](#shrink-the-database)
3. [Supprimer les données d’historique d’exécution](#delete-run-history-data)
4. [Réduire la période de rétention des données d’historique d’exécution](#shorten-retention-period-for-run-history-data)

### <a name="determine-the-synchronization-service-status"></a>Déterminer l’état du service de synchronisation
Tout d’abord, déterminez si le service de synchronisation est en cours d’exécution ou non :

1. Connectez-vous à votre serveur Azure AD Connect en tant qu’administrateur.

2. Accédez à **Gestionnaire de contrôle des services**.

3. Vérifiez l’état de **Microsoft Azure AD Sync**.


4. Si le service est en cours d’exécution, n’arrêtez pas ou ne redémarrez pas le service. Ignorez l’étape [Réduire la base de données](#shrink-the-database) et passez à l’étape [Supprimer les données d’historique d’exécution](#delete-run-history-data).

5. Si le service n’est pas en cours d’exécution, essayez de le démarrer. Si le service démarre, ignorez l’étape [Réduire la base de données](#shrink-the-database) et passez à l’étape [Supprimer les données d’historique d’exécution](#delete-run-history-data). Dans le cas contraire, passez à l’étape [Réduire la base de données](#shrink-the-database).

### <a name="shrink-the-database"></a>Réduire la base de données
Utilisez l’opération de réduction pour libérer suffisamment d’espace dans la base de données pour démarrer le service de synchronisation. Cette opération libère de l’espace dans la base de données en supprimant les espaces blancs. Il s’agit de l’étape recommandée, mais elle ne constitue pas la garantie que vous pouvez toujours récupérer de l’espace. Pour en savoir plus sur l’opération de réduction, lisez l’article [Réduire une base de données](/sql/relational-databases/databases/shrink-a-database).

> [!IMPORTANT]
> Ignorez cette étape si vous pouvez faire en sorte que le service de synchronisation s’exécute. Il est déconseillé de réduire la base de données SQL, car cela peut entraîner une baisse des performances en raison d’une fragmentation accrue.

Le nom de la base de données créée pour Azure AD Connect est **ADSync**. Pour effectuer une opération de réduction, vous devez vous connecter en tant que sysadmin ou propriétaire de la base de données. Lors de l’installation d’Azure AD Connect, les comptes suivants disposent de droits d’administrateur système :
* Administrateurs locaux
* Compte d’utilisateur qui a été utilisé pour exécuter l’installation d’Azure AD Connect.
* Compte de service de synchronisation qui est utilisé comme contexte d’opération du service de synchronisation Azure AD Connect.
* Groupe local ADSyncAdmins créé pendant l’installation.

1. Sauvegardez la base de données en copiant les fichiers **ADSync.mdf** et **ADSync_log.ldf** situés sous `%ProgramFiles%\Microsoft Azure AD Sync\Data` dans un emplacement sûr.

2. Démarrez une nouvelle session PowerShell.

3. Accédez au dossier `%ProgramFiles%\Microsoft SQL Server\110\Tools\Binn`.

4. Démarrez l’utilitaire **sqlcmd** en exécutant la commande `./SQLCMD.EXE -S "(localdb)\.\ADSync" -U <Username> -P <Password>`, à l’aide des informations d’identification de l’administrateur système ou du propriétaire de la base de données.

5. Pour réduire la base de données, à l’invite sqlcmd (1>), entrez `DBCC Shrinkdatabase(ADSync,1);` suivi par `GO` dans la ligne suivante.

6. Si l’opération réussit, essayez de redémarrer le service de synchronisation. Si vous pouvez démarrer le service de synchronisation, passez à l’étape [Supprimer les données d’historique d’exécution](#delete-run-history-data). Sinon, contactez le support.

### <a name="delete-run-history-data"></a>Supprimer les données d’historique d’exécution
Par défaut, Azure AD Connect conserve jusqu’à sept jours les données d’historique d’exécution. Dans cette étape, vous allez supprimer les données d’historique d’exécution pour récupérer de l’espace dans la base de données afin que le service de synchronisation Azure AD Connect puisse recommencer la synchronisation.

1. Démarrez **Synchronization Service Manager** en accédant au menu DÉMARRER → Service de synchronisation.

2. Accédez à l’onglet **Opérations**.

3. Sous **Actions**, sélectionnez **Clear Runs... (Effacer les exécutions)** .

4. Vous pouvez choisir l’option **Clear all runs (Effacer toutes les exécutions)** ou **Clear runs before… (Effacer les exécutions avant)\<date>** . Il est recommandé de commencer par désactiver les données d’historique d’exécution présentes depuis plus de deux jours. Si vous continuez à rencontrer le problème de taille de base de données, choisissez l’option **Clear all runs (Effacer toutes les exécutions)** .

### <a name="shorten-retention-period-for-run-history-data"></a>Réduire la période de rétention des données d’historique d’exécution
Cette étape consiste à réduire la probabilité de rencontrer le problème de limite de 10 Go après plusieurs cycles de synchronisation.

1. Ouvrez une nouvelle session PowerShell.

2. Exécutez `Get-ADSyncScheduler` et prenez note de la propriété PurgeRunHistoryInterval, qui indique la période de rétention actuelle.

3. Exécutez `Set-ADSyncScheduler -PurgeRunHistoryInterval 2.00:00:00` pour définir une période de rétention de deux jours. Ajustez la période de rétention selon le cas.

## <a name="long-term-solution--migrate-to-full-sql"></a>Solution à long terme : migrer vers une base de données SQL complète
En général, ce problème indique que la taille de base de données de 10 Go n’est plus suffisante pour qu’Azure AD Connect synchronise votre annuaire Active Directory local sur Azure AD. Nous vous recommandons de passer à la version complète de SQL Server. Vous ne pouvez pas remplacer directement la base de données locale d’un déploiement Azure AD Connect existant par la base de données de la version complète de SQL. Au lieu de cela, vous devez déployer un nouveau serveur Azure AD Connect avec la version complète de SQL. Nous vous recommandons d’effectuer une migration de basculement dans laquelle le nouveau serveur Azure AD Connect (avec la base de données SQL) est déployé comme serveur intermédiaire à côté du serveur Azure AD Connect existant (avec la base de données locale). 
* Pour obtenir des instructions sur la configuration à distance de SQL avec Azure AD Connect, consultez l’article [Installation personnalisée d’Azure AD Connect](./how-to-connect-install-custom.md).
* Pour obtenir des instructions sur la migration de basculement pour la mise à niveau d'Azure AD Connect, consultez l'article [Azure AD Connect : effectuer une mise à niveau vers la dernière version](./how-to-upgrade-previous-version.md#swing-migration).

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).
