---
title: Data Sync Agent pour SQL Data Sync
description: Apprenez à installer et exécuter l'Agent de synchronisation des données pour permettre à SQL Data Sync de synchroniser des données avec des bases de données SQL Server.
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 12/20/2018
ms.openlocfilehash: ed8d51adf5a93b470f287383a4d3eeb866b15236
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92791458"
---
# <a name="data-sync-agent-for-sql-data-sync"></a>Data Sync Agent pour SQL Data Sync
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Synchronisez des données avec des bases de données SQL Server en installant et en configurant l'Agent de synchronisation des données pour SQL Data Sync dans Azure. Pour plus d'informations sur SQL Data Sync, consultez [Synchroniser des données entre plusieurs bases de données locales et cloud avec SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md).

> [!IMPORTANT]
> Pour le moment, SQL Data Sync ne prend **pas** en charge Azure SQL Managed Instance.

## <a name="download-and-install"></a>Télécharger et installer

Pour télécharger l’Agent de synchronisation des données, accédez à [Agent de synchronisation des données SQL](https://www.microsoft.com/download/details.aspx?id=27693). Pour mettre à niveau Data Sync Agent, installez l’agent dans le même emplacement que l’ancien agent, ce qui remplacera celui d’origine.

### <a name="install-silently"></a>Installer en mode silencieux

Pour installer l’Agent de synchronisation des données en mode silencieux à partir de l’invite de commandes, entrez une commande semblable à l’exemple suivant. Vérifiez le nom du fichier .msi téléchargé et entrez vos propres valeurs pour les arguments **TARGETDIR** et **SERVICEACCOUNT** .

- Si vous ne fournissez pas de valeur pour l'argument **TARGETDIR** , la valeur par défaut est `C:\Program Files (x86)\Microsoft SQL Data Sync 2.0`.

- Si vous fournissez `LocalSystem` comme valeur de l'argument **SERVICEACCOUNT** , utilisez l'authentification SQL Server lors de la configuration de l'agent pour qu'il se connecte au serveur SQL.

- Si vous fournissez un compte d'utilisateur de domaine ou un compte d'utilisateur local comme valeur de **SERVICEACCOUNT** , vous devez également fournir le mot de passe avec l'argument **SERVICEPASSWORD** . Par exemple : `SERVICEACCOUNT="<domain>\<user>"  SERVICEPASSWORD="<password>"`.

```cmd
msiexec /i "SQLDataSyncAgent-2.0-x86-ENU.msi" TARGETDIR="C:\Program Files (x86)\Microsoft SQL Data Sync 2.0" SERVICEACCOUNT="LocalSystem" /qn
```

## <a name="sync-data-with-a-sql-server-database"></a>Synchroniser les données avec une base de données SQL Server

Pour configurer l'Agent de synchronisation des données afin de synchroniser des données avec une ou plusieurs bases de données SQL Server, consultez l'article [Ajouter une base de données SQL Server](sql-data-sync-sql-server-configure.md#add-on-prem).

## <a name="data-sync-agent-faq"></a><a name="agent-faq"></a>FAQ sur l'Agent de synchronisation des données

### <a name="why-do-i-need-a-client-agent"></a>Pourquoi ai-je besoin d’un agent client ?

Le service SQL Data Sync communique avec les bases de données SQL Server via l’agent client. Cette fonctionnalité de sécurité empêche la communication directe avec les bases de données derrière un pare-feu. Quand le service SQL Data Sync communique avec l’agent, il utilise des connexions chiffrées et un jeton unique ou une *clé de l’agent* . Les bases de données SQL Server authentifient l’agent à l’aide de la chaîne de connexion et de la clé d’agent. Cette méthode assure un haut niveau de sécurité pour vos données.

### <a name="how-many-instances-of-the-local-agent-ui-can-be-run"></a>Combien d’instances de l’interface utilisateur de l’agent local peuvent être exécutées ?

Une seule instance de l’interface utilisateur peut être exécutée.

### <a name="how-can-i-change-my-service-account"></a>Comment faire pour changer mon compte de service ?

Après avoir installé un agent client, la seule façon de modifier le compte de service consiste à le désinstaller et à installer un nouvel agent client avec le nouveau compte de service.

### <a name="how-do-i-change-my-agent-key"></a>Comment faire pour modifier la clé de mon agent ?

Une clé d’agent ne peut être utilisée qu’une seule fois par un agent. Elle ne peut pas être réutilisée lorsque vous supprimez puis réinstallez un nouvel agent, et ne peut pas non plus être utilisée par plusieurs agents. Si vous avez besoin de créer une clé pour un agent existant, vous devez vous assurer que la même clé est enregistrée auprès de l’agent client et du service SQL Data Sync.

### <a name="how-do-i-retire-a-client-agent"></a>Comment mettre hors service un agent client ?

Pour invalider ou mettre hors service immédiatement un agent, régénérez sa clé dans le portail, mais ne l’envoyez pas dans l’interface utilisateur de l’agent. La régénération d’une clé invalide la clé précédente, peu importe que l’agent correspondant soit en ligne ou hors connexion.

### <a name="how-do-i-move-a-client-agent-to-another-computer"></a>Comment déplacer un agent client vers un autre ordinateur ?

Si vous souhaitez exécuter l’agent local à partir d’un autre ordinateur, procédez comme suit :

1. Installez l’agent sur l’ordinateur souhaité.
2. Connectez-vous au portail SQL Data Sync et regénérez une clé de l’agent pour le nouvel agent.
3. Utilisez l’interface utilisateur du nouvel agent pour envoyer la nouvelle clé d’agent.
4. Patientez pendant que l’agent client télécharge la liste des bases de données locales qui ont été enregistrées précédemment.
5. Indiquez les informations d’identification de base de données pour toutes les bases de données apparaissant comme inaccessibles. Ces bases de données doivent être accessibles depuis le nouvel ordinateur sur lequel l’agent est installé.

## <a name="troubleshoot-data-sync-agent-issues"></a><a name="agent-tshoot"></a> Résoudre les problèmes liés à l’Agent de synchronisation des données

- [L’installation, la désinstallation ou la réparation de l’agent client échoue.](#agent-install)

- [L’agent client ne fonctionne pas après l’annulation de la désinstallation.](#agent-uninstall)

- [Ma base de données n’est pas répertoriée dans la liste des agents.](#agent-list)

- [L’agent client ne démarre pas (Erreur 1069).](#agent-start)

- [Je ne parviens pas à envoyer la clé d’un agent.](#agent-key)

- [L’agent client ne peut pas être supprimé du portail si la base de données locale associée est inaccessible.](#agent-delete)

- [L’application locale Agent de synchronisation ne peut pas se connecter au service de synchronisation local.](#agent-connect)

### <a name="the-client-agent-install-uninstall-or-repair-fails"></a><a name="agent-install"></a> L’installation, la désinstallation ou la réparation de l’agent client échoue.

- **Cause** . Cet échec peut avoir de nombreuses causes. Pour déterminer la cause spécifique de cet échec, consultez les journaux d’activité.

- **Résolution** . Pour rechercher la cause spécifique de l’échec rencontré, générez et consultez les journaux d’activité Windows Installer. Vous pouvez activer la journalisation à partir d’une invite de commandes. Par exemple, si le fichier d'installation est `SQLDataSyncAgent-2.0-x86-ENU.msi`, générez et examinez les fichiers journaux à l'aide des lignes de commande suivantes :

  - Pour les installations : `msiexec.exe /i SQLDataSyncAgent-2.0-x86-ENU.msi /l*v LocalAgentSetup.Log`
  - Pour les désinstallations : `msiexec.exe /x SQLDataSyncAgent-2.0-x86-ENU.msi /l*v LocalAgentSetup.Log`

    Vous pouvez également activer la journalisation pour toutes les installations effectuées par Windows Installer. L’article de la Base de connaissances Microsoft [Guide pratique pour activer la journalisation de Windows Installer](https://support.microsoft.com/help/223300/how-to-enable-windows-installer-logging) fournit une solution en un clic pour activer la journalisation pour Windows Installer. Il indique également l’emplacement des journaux d’activité.

### <a name="the-client-agent-doesnt-work-after-i-cancel-the-uninstall"></a><a name="agent-uninstall"></a> Mon agent client ne fonctionne pas après l’annulation de la désinstallation.

L’agent client ne fonctionne pas, même après l’annulation de sa désinstallation.

- **Cause** . Ce problème survient car l’agent client SQL Data Sync ne stocke pas les informations d’identification.

- **Résolution** . Vous pouvez essayer les deux solutions suivantes :

    -   Utilisez services.msc afin de réentrer les informations d’identification pour l’agent client.
    -   Désinstallez cet agent client, puis installez-en un nouveau. Téléchargez et installez l’agent client le plus récent à partir du [Centre de téléchargement](https://www.microsoft.com/download/details.aspx?id=27693).

### <a name="my-database-isnt-listed-in-the-agent-list"></a><a name="agent-list"></a> Ma base de données n’est pas répertoriée dans la liste des agents.

Lorsque vous essayez d’ajouter une base de données SQL Server existante à un groupe de synchronisation, la base de données n’apparaît pas dans la liste des agents.

Ce problème peut avoir les causes suivantes :

- **Cause** . L’agent client et le groupe de synchronisation se trouvent dans des centres de données différents.

- **Résolution** . L’agent client et le groupe de synchronisation doivent se trouver dans le même centre de données. Pour cela, vous disposez de deux options :

    -   Créez un agent dans le centre de données où se trouve le groupe de synchronisation. Puis inscrivez la base de données auprès de cet agent.
    -   Supprimez le groupe de synchronisation actuel. Recréez ensuite le groupe de synchronisation dans le centre de données où se trouve l’agent.

- **Cause** . La liste des bases de données de l’agent client n’est pas à jour.

- **Résolution** . Arrêtez, puis redémarrez le service agent client.

    L’agent local télécharge la liste des bases de données associées uniquement lors du premier envoi de la clé de l’agent. Il ne la télécharge pas lors des envois suivants. Les bases de données inscrites pendant le déplacement d’un agent ne sont pas visibles sur l’instance d’origine de l’agent.

### <a name="client-agent-doesnt-start-error-1069"></a><a name="agent-start"></a> L’agent client ne démarre pas (Erreur 1069).

Vous découvrez que l’agent n’est pas exécuté sur un ordinateur qui héberge SQL Server. Lorsque vous essayez de démarrer manuellement l’agent, une boîte de dialogue affiche le message « Erreur 1069 : L’échec d’une ouverture de session a empêché le démarrage du service ».

![Boîte de dialogue de l’erreur 1069 Data Sync](./media/sql-data-sync-agent-overview/sync-error-1069.png)

- **Cause** . Cette erreur peut être due au fait que le mot de passe du serveur local a changé depuis que vous avez créé l’agent et son mot de passe.

- **Résolution** . Remplacez le mot de passe de l’agent par le mot de passe actuel du serveur :

  1. Recherchez le service de l’agent du client SQL Data Sync.  
    a. Sélectionnez **Démarrer** .  
    b. Dans la zone de recherche, entrez **services.msc** .  
    c. Dans les résultats de la recherche, sélectionnez **Services** .  
    d. Dans la fenêtre **Services** , faites défiler jusqu’à l’entrée **Agent SQL Data Sync** .  
  1. Cliquez avec le bouton droit sur **Agent SQL Data Sync** , puis sélectionnez **Arrêter** .
  1. Cliquez avec le bouton droit sur **Agent SQL Data Sync** , puis sélectionnez **Propriétés** .
  1. Dans **Propriétés de l’agent SQL Data Sync** , sélectionnez l’onglet **Connexion** .
  1. Dans la zone **Mot de passe** , entrez votre mot de passe.
  1. Dans la zone **Confirmer le mot de passe** , entrez de nouveau votre mot de passe.
  1. Sélectionnez **Apply** (Appliquer), puis **OK** .
  1. Dans la fenêtre **Services** , cliquez avec le bouton droit sur le service **Agent SQL Data Sync** , puis cliquez sur **Démarrer** .
  1. Fermez la fenêtre **Services** .

### <a name="i-cant-submit-the-agent-key"></a><a name="agent-key"></a> Je ne parviens pas à envoyer la clé d’un agent.

Une fois que vous avez créé ou recréé la clé d’un agent, vous essayez d’envoyer cette clé par le biais de l’application SqlAzureDataSyncAgent. Mais l’envoi échoue.

![Boîte de dialogue d’erreur de synchronisation - Impossible d’envoyer la clé d’agent](./media/sql-data-sync-agent-overview/sync-error-cant-submit-agent-key.png)

- **Conditions préalables** . Avant de continuer, vérifiez les conditions préalables suivantes :

  - Le service Windows SQL Data Sync est en cours d’exécution.

  - Le compte de service pour le service Windows SQL Data Sync a accès au réseau.

  - Le port 1433 de trafic sortant est ouvert dans votre règle de pare-feu local.

  - L’adresse IP locale est ajoutée au serveur ou à la règle de pare-feu de base de données pour la base de données de métadonnées de synchronisation.

- **Cause** . La clé d’agent identifie de façon unique chaque agent local. La clé doit remplir deux conditions :

  -   La clé de l’agent client sur le serveur SQL Data Sync et l’ordinateur local doivent être identiques.
  -   La clé de l’agent client ne peut être utilisée qu’une seule fois.

- **Résolution** . Si votre agent ne fonctionne pas, cela signifie que l’une de ces conditions (ou les deux) n’est pas remplie. Pour que votre agent fonctionne de nouveau :

  1. Générez une nouvelle clé.
  1. Appliquez la nouvelle clé à l’agent.

  Pour appliquer la nouvelle clé à l’agent :

  1. Dans l’Explorateur de fichiers, accédez au répertoire d’installation de votre agent. Le répertoire d’installation par défaut est C:\\Program Files (x86)\\Microsoft SQL Data Sync.
  1. Double-cliquez sur le sous-répertoire bin.
  1. Ouvrez l’application SqlAzureDataSyncAgent.
  1. Sélectionnez **Envoyer la clé d’agent** .
  1. Collez la clé placée dans votre Presse-papiers à l’espace prévu à cet effet.
  1. Sélectionnez **OK** .
  1. Fermez le programme.

### <a name="the-client-agent-cant-be-deleted-from-the-portal-if-its-associated-on-premises-database-is-unreachable"></a><a name="agent-delete"></a> L’agent client ne peut pas être supprimé du portail si la base de données locale associée est inaccessible.

Si un point de terminaison local (autrement dit, une base de données) inscrite auprès d’un agent client SQL Data Sync devient inaccessible, l’agent client ne peut pas être supprimé.

- **Cause** . L’agent local ne peut pas être supprimé, car la base de données inaccessible est encore inscrite auprès de l’agent. Quand vous tentez de supprimer l’agent, le processus de suppression essaie d’atteindre la base de données, et il échoue.

- **Résolution** . Utilisez « Forcer la suppression » pour supprimer la base de données inaccessible.

> [!NOTE]
> S’il reste des tables de métadonnées de synchronisation après une opération « Forcer la suppression », utilisez `deprovisioningutil.exe` pour les nettoyer.

### <a name="local-sync-agent-app-cant-connect-to-the-local-sync-service"></a><a name="agent-connect"></a> L’application locale Agent de synchronisation ne peut pas se connecter au service de synchronisation local.

- **Résolution** . Essayez les étapes suivantes :

  1. Quittez l’application.  
  1. Ouvrez le panneau Services de composants.  
    a. Dans la zone de recherche de la barre des tâches, entrez **services.msc** .  
    b. Dans les résultats de la recherche, double-cliquez sur **Services** .  
  1. Arrêtez le service **SQL Data Sync** .
  1. Redémarrez le service **SQL Data Sync** .  
  1. Rouvrez l’application.

## <a name="run-the-data-sync-agent-from-the-command-prompt"></a>Exécuter l’Agent de synchronisation des données à partir de l’invite de commandes

Vous pouvez exécuter les commandes suivantes de l’Agent de synchronisation des données à partir de l’invite de commandes :

### <a name="ping-the-service"></a>Effectuer un test ping du service

#### <a name="usage"></a>Usage

```cmd
SqlDataSyncAgentCommand.exe -action pingsyncservice
```

#### <a name="example"></a>Exemple

```cmd
SqlDataSyncAgentCommand.exe -action "pingsyncservice"
```

### <a name="display-registered-databases"></a>Afficher les bases de données inscrites

#### <a name="usage"></a>Usage

```cmd
SqlDataSyncAgentCommand.exe -action displayregistereddatabases
```

#### <a name="example"></a>Exemple

```cmd
SqlDataSyncAgentCommand.exe -action "displayregistereddatabases"
```

### <a name="submit-the-agent-key"></a>Envoyer la clé de l’agent

#### <a name="usage"></a>Usage

```cmd
Usage: SqlDataSyncAgentCommand.exe -action submitagentkey -agentkey [agent key]  -username [user name] -password [password]
```

#### <a name="example"></a>Exemple

```cmd
SqlDataSyncAgentCommand.exe -action submitagentkey -agentkey [agent key generated from portal, PowerShell, or API] -username [user name to sync metadata database] -password [user name to sync metadata database]
```

### <a name="register-a-database"></a>Inscrire une base de données

#### <a name="usage"></a>Usage

```cmd
SqlDataSyncAgentCommand.exe -action registerdatabase -servername [on-premisesdatabase server name] -databasename [on-premisesdatabase name]  -username [domain\\username] -password [password] -authentication [sql or windows] -encryption [true or false]
```

#### <a name="examples"></a>Exemples

```cmd
SqlDataSyncAgentCommand.exe -action "registerdatabase" -serverName localhost -databaseName testdb -authentication sql -username <user name> -password <password> -encryption true

SqlDataSyncAgentCommand.exe -action "registerdatabase" -serverName localhost -databaseName testdb -authentication windows -encryption true

```

### <a name="unregister-a-database"></a>Annuler l’inscription d’une base de données

Lorsque vous utilisez cette commande pour annuler l'inscription d'une base de données, cette dernière est totalement déprovisionnée. Si la base de données participe à d'autres groupes de synchronisation, cette opération rompt les autres groupes de synchronisation.

#### <a name="usage"></a>Usage

```cmd
SqlDataSyncAgentCommand.exe -action unregisterdatabase -servername [on-premisesdatabase server name] -databasename [on-premisesdatabase name]
```

#### <a name="example"></a>Exemple

```cmd
SqlDataSyncAgentCommand.exe -action "unregisterdatabase" -serverName localhost -databaseName testdb
```

### <a name="update-credentials"></a>Mettre à jour les informations d’identification

#### <a name="usage"></a>Usage

```cmd
SqlDataSyncAgentCommand.exe -action updatecredential -servername [on-premisesdatabase server name] -databasename [on-premisesdatabase name]  -username [domain\\username] -password [password] -authentication [sql or windows] -encryption [true or false]
```

#### <a name="examples"></a>Exemples

```cmd
SqlDataSyncAgentCommand.exe -action "updatecredential" -serverName localhost -databaseName testdb -authentication sql -username <user name> -password <password> -encryption true

SqlDataSyncAgentCommand.exe -action "updatecredential" -serverName localhost -databaseName testdb -authentication windows -encryption true
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur SQL Data Sync, consultez les articles suivants :

-   Vue d'ensemble - [Synchroniser des données entre plusieurs bases de données cloud et locales avec SQL Data Sync dans Azure](sql-data-sync-data-sql-server-sql-database.md)
-   Configurer Data Sync
    - Sur le portail - [Tutoriel : Configurer SQL Data Sync pour synchroniser les données entre Azure SQL Database et SQL Server](sql-data-sync-sql-server-configure.md)
    - Avec PowerShell
        -  [Utiliser PowerShell pour synchroniser des données entre plusieurs bases de données dans Azure SQL Database](scripts/sql-data-sync-sync-data-between-sql-databases.md)
        -  [Utiliser PowerShell pour synchroniser des données entre une base de données d'Azure SQL Database et une base de données d'une instance SQL Server](scripts/sql-data-sync-sync-data-between-azure-onprem.md)
-   Bonnes pratiques - [Bonnes pratiques pour Azure SQL Data Sync](sql-data-sync-best-practices.md)
-   Supervision – [Superviser SQL Data Sync avec les journaux d’activité Azure Monitor](./monitor-tune-overview.md)
-   Résolution des problèmes - [Résoudre les problèmes liés à Azure SQL Data Sync]sql-data-sync-troubleshoot.md)
-   Mettre à jour le schéma de synchronisation
    -   Avec Transact-SQL - [Automatiser la réplication des modifications de schéma avec SQL Data Sync dans Azure](sql-data-sync-update-sync-schema.md)
    -   Avec PowerShell - [Utiliser PowerShell pour mettre à jour le schéma de synchronisation dans un groupe de synchronisation existant](scripts/update-sync-schema-in-sync-group.md)