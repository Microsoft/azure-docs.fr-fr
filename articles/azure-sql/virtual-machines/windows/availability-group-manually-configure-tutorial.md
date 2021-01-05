---
title: 'Tutoriel : Configurer un groupe de disponibilité SQL Server Always On'
description: Ce tutoriel explique comment créer un groupe de disponibilité AlwaysOn SQL Server sur des machines virtuelles Azure.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.assetid: 08a00342-fee2-4afe-8824-0db1ed4b8fca
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: tutorial
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 08/30/2018
ms.author: mathoma
ms.custom: seo-lt-2019
ms.openlocfilehash: feab48f32396bcc89621433930c9a9f4689d8286
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97355441"
---
# <a name="tutorial-manually-configure-an-availability-group-sql-server-on-azure-vms"></a>Tutoriel : Configurer manuellement un groupe de disponibilité (SQL Server sur des machines virtuelles Azure)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Ce tutoriel montre comment créer un groupe de disponibilité AlwaysOn pour SQL Server sur des machines virtuelles Azure. Il crée un groupe de disponibilité avec un réplica de base de données sur deux serveurs SQL Server.

Cet article configure manuellement l’environnement du groupe de disponibilité. Cependant, il est également possible de le faire avec le [portail Azure](availability-group-azure-portal-configure.md), avec [PowerShell ou Azure CLI](availability-group-az-commandline-configure.md) ou avec des [modèles de démarrage rapide Azure](availability-group-quickstart-template-configure.md). 


**Durée estimée** : Environ 30 minutes une fois les [prérequis](availability-group-manually-configure-prerequisites-tutorial.md) remplis.


## <a name="prerequisites"></a>Prérequis

Ce tutoriel suppose que vous avez des notions de base sur les groupes de disponibilité AlwaysOn SQL Server. Pour plus d’informations, consultez [Vue d’ensemble des groupes de disponibilité AlwaysOn (SQL Server)](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server).

Avant de commencer ce tutoriel, lisez [Prérequis de la création de groupes de disponibilité AlwaysOn sur SQL Server sur machines virtuelles Azure](availability-group-manually-configure-prerequisites-tutorial.md). Si ces conditions préalables sont déjà remplies, vous pouvez passer à l’étape [Création d’un cluster](#CreateCluster).

Le tableau suivant répertorie les conditions requises que vous devez remplir avant de commencer ce didacticiel :

| Condition requise |Description |
|----- |----- |----- |
|:::image type="icon" source="./media/availability-group-manually-configure-tutorial/square.png" border="false":::   **Deux instances** | - Dans un groupe à haute disponibilité Azure <br/> - Dans un domaine <br/> - Avec la fonctionnalité de Clustering de basculement installée |
|:::image type="icon" source="./media/availability-group-manually-configure-tutorial/square.png" border="false":::   **Windows Server** | Partage de fichiers pour le témoin de cluster |  
|:::image type="icon" source="./media/availability-group-manually-configure-tutorial/square.png" border="false":::   **Compte du service SQL Server** | Compte du domaine |
|:::image type="icon" source="./media/availability-group-manually-configure-tutorial/square.png" border="false":::   **Compte du service SQL Server Agent** | Compte du domaine |  
|:::image type="icon" source="./media/availability-group-manually-configure-tutorial/square.png" border="false":::   **Ports du pare-feu ouverts** | - SQL Server : **1433** pour l’instance par défaut <br/> - Point de terminaison de mise en miroir de bases de données : **5022** ou n’importe quel port disponible <br/> - Sonde d’intégrité d’adresse IP d’équilibreur de charge de groupe de disponibilité : **59999** ou n’importe quel port disponible <br/> - Sonde d’intégrité d’adresse IP d’équilibreur de charge principal du cluster : **58888** ou n’importe quel port disponible |
|:::image type="icon" source="./media/availability-group-manually-configure-tutorial/square.png" border="false":::   **Ajout de la fonctionnalité de Clustering de basculement** | Fonctionnalité requise sur les deux instances |
|:::image type="icon" source="./media/availability-group-manually-configure-tutorial/square.png" border="false":::   **Compte du domaine d’installation** | - Administrateur local sur chaque serveur SQL Server <br/> - Membres du rôle de serveur fixe sysadmin pour chaque instance de SQL Server  |

>[!NOTE]
> De nombreuses étapes fournies dans ce tutoriel peuvent maintenant être automatisées avec le [portail Azure](availability-group-azure-portal-configure.md), [PowerShell et Azure CLI](./availability-group-az-commandline-configure.md) ou les [modèles de démarrage rapide Azure](availability-group-quickstart-template-configure.md).


<!--**Procedure**: *This is the first "step". Make titles H2's and short and clear – H2's appear in the right pane on the web page and are important for navigation.*-->

<a name="CreateCluster"></a>

## <a name="create-the-cluster"></a>Créer le cluster

Une fois les conditions préalables remplies, la première étape consiste à créer un cluster de basculement Windows Server comprenant deux serveurs SQL Server et un serveur témoin.

1. Utilisez protocole RDP (Remote Desktop Protocol) pour vous connecter au premier SQL Server. Utilisez un compte de domaine administrateur sur le SQL Server et sur le serveur témoin.

   >[!TIP]
   >Si vous avez suivi le [document sur les conditions requises](availability-group-manually-configure-prerequisites-tutorial.md), vous avez créé un compte appelé **CORP\Install**. Utilisez ce compte.

2. Dans le tableau de bord **Gestionnaire de serveur**, sélectionnez **Outils**, puis sélectionnez **Gestionnaire du cluster de basculement**.
3. Dans le volet gauche, cliquez-droit sur **Gestionnaire du cluster de basculement**, puis sélectionnez sur **Créer un cluster**.

   ![Créer un cluster](./media/availability-group-manually-configure-tutorial/40-createcluster.png)

4. Dans l’Assistant Création d’un cluster, créez un cluster à un nœud en complétant les pages avec les paramètres figurant dans le tableau ci-dessous :

   | Page | Paramètres |
   | --- | --- |
   | Avant de commencer |Utilisation des valeurs par défaut |
   | Sélection des serveurs |Tapez le nom du premier SQL Server dans **Entrer le nom du serveur**, puis sélectionnez **Ajouter**. |
   | Avertissement de validation |Cliquez sur **Non. Je n’ai pas besoin de l’assistance de Microsoft pour ce cluster, et par conséquent, je ne souhaite pas exécuter les tests de validation. Lorsque je clique sur Suivant, la création du cluster** continue. |
   | Point d'accès pour l'administration du cluster |Tapez un nom de cluster, par exemple **SQLAGCluster1** dans **Nom du cluster**.|
   | Confirmation |Utilisez les valeurs par défaut, sauf si vous utilisez des espaces de stockage. Consultez la remarque qui suit ce tableau. |

### <a name="set-the-windows-server-failover-cluster-ip-address"></a>Définir l’adresse IP du cluster de basculement de Windows Server

  > [!NOTE]
  > Sur Windows Server 2019, le cluster crée un **nom de serveur distribué** à la place du **nom réseau du cluster**. Si vous utilisez Windows Server 2019, ignorez les étapes qui font référence au nom de base du cluster dans ce tutoriel. Vous pouvez créer un nom réseau de cluster à l’aide de [PowerShell](failover-cluster-instance-storage-spaces-direct-manually-configure.md#create-failover-cluster). Passez en revue le blog [Failover Cluster: Cluster Network Object](https://blogs.windows.com/windowsexperience/2018/08/14/announcing-windows-server-2019-insider-preview-build-17733/#W0YAxO8BfwBRbkzG.97) pour plus d’informations. 

1. Dans **Gestionnaire du cluster de basculement**, accédez à **Principales ressources du cluster** et développez les détails du cluster. Les ressources **Nom** et **Adresse IP** doivent toutes deux être à l’état **Échec**. La ressource à adresse IP ne peut pas être mise en ligne, car le cluster a la même adresse IP que la machine elle-même. Cette adresse est donc dupliquée.

2. Cliquez-droit sur la ressource **Adresse IP** en échec, puis sélectionnez **Propriétés**.

   ![Propriétés du cluster](./media/availability-group-manually-configure-tutorial/42_IPProperties.png)

3. Sélectionnez **Adresse IP statique** et spécifiez une adresse disponible dans le même sous-réseau que vos machines virtuelles.

4. Dans la section **Principales ressources du cluster**, cliquez-droit sur le nom du cluster et sélectionnez **Mettre en ligne**. Attendez que les deux ressources soient en ligne. Lorsque la ressource du nom du cluster apparaît en ligne, elle met à jour le contrôleur de domaine avec un nouveau compte d’ordinateur Active Directory (AD). Utilisez ce compte AD pour exécuter le service en cluster du groupe de disponibilité ultérieurement.

### <a name="add-the-other-sql-server-to-cluster"></a><a name="addNode"></a>Ajouter l’autre serveur SQL Server au cluster

Ajoutez l’autre serveur SQL Server au cluster.

1. Dans l’arborescence du navigateur, cliquez-droit sur le cluster, puis sélectionnez **Ajouter un nœud**.

    ![Ajout d’un nœud au cluster](./media/availability-group-manually-configure-tutorial/44-addnode.png)

1. Dans **l’Assistant d’ajout de nœud**, sélectionnez **Suivant**. Dans la page **Sélectionner les serveurs**, ajouter le second SQL Server. Tapez le nom du serveur dans **Entrer le nom du serveur**, puis sélectionnez **Ajouter**. Lorsque vous avez terminé, sélectionnez **Suivant**.

1. Sur la page **Avertissement de validation**, sélectionnez **Non** (dans un scénario de production, vous devez effectuer les tests de validation). Ensuite, sélectionnez **Suivant**.

8. Dans la page **Confirmation**, si vous utilisez des espaces de stockage, vous devez désactiver la case à cocher **Ajouter la totalité du stockage disponible au cluster**.

   ![Confirmation de l’ajout du nœud](./media/availability-group-manually-configure-tutorial/46-addnodeconfirmation.png)

   >[!WARNING]
   >Si vous utilisez des espaces de stockage et que vous ne désactivez pas la case à cocher **Ajouter la totalité du stockage disponible au cluster**, Windows détache les disques virtuels pendant le processus de mise en cluster. Par conséquent, ils n’apparaissent pas dans le Gestionnaire de Azure Data Box Disk ou dans l’Explorateur, jusqu’à ce que les espaces de stockage soient supprimés du cluster et rattachés à l’aide de PowerShell. Les espaces de stockage regroupent plusieurs disques dans des pools de stockage. Pour plus d’informations, consultez la page [Espaces de stockage](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831739(v=ws.11)).
   >

1. Sélectionnez **Suivant**.

1. Sélectionnez **Terminer**.

   Le Gestionnaire du cluster de basculement indique que votre cluster a un nouveau nœud et l’affiche dans le conteneur **Nœuds**.

10. Déconnectez-vous de la session Bureau à distance.

### <a name="add-a-cluster-quorum-file-share"></a>Ajouter un partage de fichiers de quorum de cluster

Dans cet exemple, le cluster Windows utilise un partage de fichiers pour créer un quorum de cluster. Ce didacticiel utilise un quorum Nœud et partage de fichiers majoritaires. Pour plus d’informations, consultez [Présentation des configurations de quorum dans un cluster de basculement](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731739(v=ws.11)).

1. Connectez-vous au serveur de membre de témoin de partage de fichiers avec une session Bureau à distance.

1. Dans **Gestionnaire de serveur**, sélectionnez **Outils**. Ouvrez **Gestion de l’ordinateur**.

1. Sélectionnez **Dossiers partagés**.

1. Cliquez-droit sur **Partages**, puis cliquez sur **Nouveau partage...** .

   ![Cliquez avec le bouton droit sur Partages, puis sélectionnez Nouveau partage.](./media/availability-group-manually-configure-tutorial/48-newshare.png)

   Utilisez **Assistant Création d’un dossier partagé** pour créer un partage.

1. Dans **Chemin d’accès du dossier**, sélectionnez **Parcourir** et recherchez ou créez le chemin d’accès du dossier partagé. Sélectionnez **Suivant**.

1. Dans **Nom, Description et Paramètres**, vérifiez le nom et le chemin d’accès du partage. Sélectionnez **Suivant**.

1. Dans **Autorisations du dossier partagé**, sélectionnez **Personnaliser les autorisations**. Sélectionnez **Personnalisé...** .

1. Dans **Personnaliser les autorisations**, sélectionnez **Ajouter...** .

1. Vérifiez que le compte utilisé pour créer le cluster possède le contrôle total.

   ![Vérifiez que le compte utilisé pour créer le cluster dispose du contrôle total.](./media/availability-group-manually-configure-tutorial/50-filesharepermissions.png)

1. Sélectionnez **OK**.

1. Dans **Autorisations du dossier partagé**, sélectionnez **Terminer**. Sélectionnez **Terminer**.  

1. Déconnectez-vous du serveur.

### <a name="configure-the-cluster-quorum"></a>Configurer le quorum du cluster

Ensuite, configurez le quorum du cluster.

1. Connectez-vous au premier nœud du cluster avec le Bureau à distance.

1. Dans **Gestionnaire du cluster de basculement**, cliquez-droit sur le cluster, pointez sur **Autres Actions** et sélectionnez **Configurer les paramètres du Quorum du cluster...** .

   ![Sélectionnez Configurer les paramètres du quorum du cluster.](./media/availability-group-manually-configure-tutorial/52-configurequorum.png)

1. Dans l’**Assistant Configuration de Quorum du cluster**, sélectionnez **Suivant**.

1. Dans **Sélectionner l’option de configuration du Quorum**, choisissez **Sélectionner le témoin de quorum**, puis cliquez sur **Suivant**.

1. Dans **Sélectionner le témoin de Quorum**, sélectionnez **Configurer un témoin de partage de fichiers**.

   >[!TIP]
   >Windows Server 2016 prend en charge un témoin cloud. Si vous choisissez ce type de témoin, un témoin de partage de fichiers est inutile. Pour plus d’informations, consultez [Déployer un témoin cloud pour un cluster de basculement](/windows-server/failover-clustering/deploy-cloud-witness). Ce didacticiel utilise un témoin de partage de fichiers, que les systèmes d’exploitation antérieurs prennent en charge.
   >

1. Dans **Configurer le témoin de partage de fichiers**, tapez le chemin d’accès du partage que vous avez créé. Sélectionnez **Suivant**.

1. Vérifiez les paramètres dans **Confirmation**. Sélectionnez **Suivant**.

1. Sélectionnez **Terminer**.

Les principales ressources de cluster sont configurées avec un témoin de partage de fichiers.

## <a name="enable-availability-groups"></a>Activer des groupes de disponibilité

Ensuite, activez la fonctionnalité **Groupes de disponibilité AlwaysOn**. Effectuez ces étapes pour les deux serveurs SQL Server.

1. À partir de l’écran **d’accueil**, lancez le **Gestionnaire de configuration SQL Server**.
2. Dans l’arborescence du navigateur, cliquez sur **Services SQL Server**, cliquez-droit sur le service **SQL Server (MSSQLSERVER)** , puis sélectionnez **Propriétés**.
3. Sélectionnez l’onglet **Haute disponibilité AlwaysOn**, puis sélectionnez **Activer les groupes de disponibilité AlwaysOn**, comme indiqué ci-dessous :

    ![Activer les groupes de disponibilité AlwaysOn](./media/availability-group-manually-configure-tutorial/54-enableAlwaysOn.png)

4. Sélectionnez **Appliquer**. Sélectionnez **OK** dans la boîte de dialogue de la fenêtre contextuelle.

5. Redémarrez le service SQL Server.

Répétez ces étapes sur l’autre serveur SQL Server.

<!-----------------
## <a name="endpoint-firewall"></a>Open firewall for the database mirroring endpoint

Each instance of SQL Server that participates in an availability group requires a database mirroring endpoint. This endpoint is a TCP port for the instance of SQL Server that is used to synchronize the database replicas in the availability groups on that instance.

On both SQL Servers, open the firewall for the TCP port for the database mirroring endpoint.

1. On the first SQL Server **Start** screen, launch **Windows Firewall with Advanced Security**.
2. In the left pane, select **Inbound Rules**. On the right pane, select **New Rule**.
3. For **Rule Type**, choose **Port**.
1. For the port, specify TCP and choose an unused TCP port number. For example, type *5022* and select **Next**.

   >[!NOTE]
   >For this example, we're using TCP port 5022. You can use any available port.

5. In the **Action** page, keep **Allow the connection** selected and select **Next**.
6. In the **Profile** page, accept the default settings and select **Next**.
7. In the **Name** page, specify a rule name, such as **Default Instance Mirroring Endpoint** in the **Name** text box, then select **Finish**.

Repeat these steps on the second SQL Server.
-------------------------->

## <a name="create-a-database-on-the-first-sql-server"></a>Créer une base de données sur le premier serveur SQL Serveur

1. Lancez le fichier RDP pour le premier serveur SQL Server avec un compte de domaine qui est membre du rôle serveur fixe sysadmin.
1. Ouvrez SQL Server Management Studio et connectez-vous au serveur SQL Server.
7. Dans l’**Explorateur d’objets**, cliquez-droit sur **Bases de données**, puis cliquez sur **Nouvelle base de données**.
8. Dans **Nom de base de données**, saisissez **MyDB1**, puis sélectionnez **OK**.

### <a name="create-a-backup-share"></a><a name="backupshare"></a> Créer un partage de sauvegarde

1. Sur le premier serveur SQL Server dans **Gestionnaire de serveur**, sélectionnez **outils**. Ouvrez **Gestion de l’ordinateur**.

1. Sélectionnez **Dossiers partagés**.

1. Cliquez-droit sur **Partages**, puis cliquez sur **Nouveau partage...** .

   ![Sélectionnez Nouveau partage.](./media/availability-group-manually-configure-tutorial/48-newshare.png)

   Utilisez **Assistant Création d’un dossier partagé** pour créer un partage.

1. Dans **Chemin d’accès du dossier**, sélectionnez **Parcourir** et recherchez ou créez le chemin d’accès du dossier partagé de la sauvegarde de la base de données. Sélectionnez **Suivant**.

1. Dans **Nom, Description et Paramètres**, vérifiez le nom et le chemin d’accès du partage. Sélectionnez **Suivant**.

1. Dans **Autorisations du dossier partagé**, sélectionnez **Personnaliser les autorisations**. Sélectionnez **Personnalisé...** .

1. Dans **Personnaliser les autorisations**, sélectionnez **Ajouter...** .

1. Vérifiez que les comptes de service SQL Server et SQL Server Agent pour les deux serveurs ont un contrôle total.

   ![Vérifiez que les comptes de service SQL Server et SQL Server Agent pour les deux serveurs ont un contrôle total.](./media/availability-group-manually-configure-tutorial/68-backupsharepermission.png)

1. Sélectionnez **OK**.

1. Dans **Autorisations du dossier partagé**, sélectionnez **Terminer**. Sélectionnez **Terminer**.  

### <a name="take-a-full-backup-of-the-database"></a>Créer une sauvegarde complète de la base de données

Vous devez sauvegarder la nouvelle base de données pour initialiser la chaîne du journal. Si vous ne sauvegardez pas la nouvelle base de données, vous ne pourrez pas l’inclure dans un groupe de disponibilité.

1. Dans l’**Explorateur d’objets**, cliquez-droit sur la base de données, pointez sur **Tâches...** et sélectionnez **Sauvegarder**.

1. Sélectionnez **OK** pour effectuer une sauvegarde complète à l’emplacement de sauvegarde par défaut.

## <a name="create-the-availability-group"></a>Créez le groupe de disponibilité

Vous pouvez maintenant configurer le groupe de disponibilité en effectuant les étapes suivantes :

* Création d’une base de données sur le premier serveur SQL Server
* Sauvegarde complète et sauvegarde du journal des transactions de la base de données.
* Restaurez des sauvegardes complète et journalisée sur le deuxième serveur SQL Server avec l’option **NORECOVERY**.
* Création du groupe de disponibilité (**AG1**) avec validation synchrone, basculement automatique et réplicas secondaires lisibles.

### <a name="create-the-availability-group"></a>Création du groupe de disponibilité :

1. Dans une session Bureau à distance pour le premier serveur SQL Server. Dans l’**Explorateur d’objets** de SSMS, cliquez avec le bouton droit sur **Haute disponibilité AlwaysOn**, puis sélectionnez **Assistant Nouveau groupe de disponibilité**.

    ![Lancer l’Assistant Nouveau groupe de disponibilité](./media/availability-group-manually-configure-tutorial/56-newagwiz.png)

2. Dans la page **Introduction**, sélectionnez **Suivant**. Dans la page **Spécifier le nom du groupe de disponibilité**, tapez le nom du groupe de disponibilité dans **Nom du groupe de disponibilité**. Par exemple, **AG1**. Sélectionnez **Suivant**.

    ![Assistant Nouveau groupe de disponibilité - Spécifier le nom du groupe de disponibilité](./media/availability-group-manually-configure-tutorial/58-newagname.png)

3. Dans la page **Sélectionner des bases de données**, sélectionnez votre base de données puis cliquez sur **Suivant**.

   >[!NOTE]
   >Cette base de données répond à la configuration requise pour un groupe de disponibilité, car vous avez effectué au moins une sauvegarde complète sur le réplica principal prévu.
   >

   ![Assistant Nouveau groupe de disponibilité - Sélectionner les bases de données](./media/availability-group-manually-configure-tutorial/60-newagselectdatabase.png)

4. Sur la page **Spécifier les réplicas**, sélectionnez **Ajouter un réplica**.

   ![Assistant Nouveau groupe de disponibilité - Spécifier des réplicas](./media/availability-group-manually-configure-tutorial/62-newagaddreplica.png)

5. La boîte de dialogue **Se connecter au serveur** apparaît. Tapez le nom du deuxième serveur dans **Nom du serveur**. Sélectionnez **Connecter**.

   Dans la page **Spécifier les réplicas**, vous devez maintenant voir le deuxième serveur dans **Réplicas de disponibilité**. Configurez les réplicas comme indiqué ci-dessous.

   ![Assistant Nouveau groupe de disponibilité - Spécifier des réplicas (Terminé)](./media/availability-group-manually-configure-tutorial/64-newagreplica.png)

6. Sélectionnez **Points de terminaison** afin d’afficher le point de terminaison de mise en miroir de bases de données pour ce groupe de disponibilité. Utilisez le même port que vous avez utilisé lorsque vous avez défini la [règle de pare-feu pour les points de terminaison de mise en miroir de base de données](availability-group-manually-configure-prerequisites-tutorial.md#endpoint-firewall).

    ![Assistant Nouveau groupe de disponibilité - Sélectionner la synchronisation de données initiale](./media/availability-group-manually-configure-tutorial/66-endpoint.png)

8. Dans la page **Sélectionner la synchronisation de données initiale**, sélectionnez **Complète** et spécifiez un emplacement réseau partagé. Pour l’emplacement, utilisez le [partage de sauvegarde que vous avez créé](#backupshare). Dans l’exemple, il s’agissait de **\\\\<Premier serveur SQL Server\>\Backup\\** . Sélectionnez **Suivant**.

   >[!NOTE]
   >La synchronisation complète effectue une sauvegarde complète de la base de données sur la première instance de SQL Server et la restaure sur la deuxième instance. Pour les bases de données volumineuses, une synchronisation complète n’est pas recommandée, car elle peut prendre longtemps. Vous pouvez réduire ce temps en effectuant manuellement une sauvegarde de la base de données et en la restaurant avec `NO RECOVERY`. Si la base de données a été restaurée avec `NO RECOVERY` sur le deuxième serveur SQL avant la configuration du groupe de disponibilité, choisissez **Joindre uniquement**. Si vous souhaitez effectuer la sauvegarde après avoir configuré le groupe de disponibilité, choisissez **Ignorer la synchronisation de données initiale**.
   >

   ![Choisissez Ignorer la synchronisation de données initiale.](./media/availability-group-manually-configure-tutorial/70-datasynchronization.png)

9. Dans la page **Validation**, sélectionnez **Suivant**. Cette page doit ressembler à l’image suivante :

    ![Assistant Nouveau groupe de disponibilité - Validation](./media/availability-group-manually-configure-tutorial/72-validation.png)

    >[!NOTE]
    >Un avertissement concernant la configuration de l’écouteur s’affiche, car aucun écouteur du groupe de disponibilité n’est configuré. Vous pouvez ignorer cet avertissement car, sur les machines virtuelles, vous créez l’écouteur après Azure Load Balancer.

10. Dans la page **Résumé**, sélectionnez **Terminer**, puis patientez le temps que l’Assistant configure le nouveau groupe de disponibilité. Dans la page **Progression**, vous pouvez sélectionner **Plus de détails** pour afficher la progression détaillée. À la fin de l'Assistant, examinez la page **Résultats** pour vérifier que le groupe de disponibilité a été créé.

     ![Assistant Nouveau groupe de disponibilité - Résultats](./media/availability-group-manually-configure-tutorial/74-results.png)

11. Pour quitter l’Assistant, sélectionnez **Fermer**.

### <a name="check-the-availability-group"></a>Vérifier le groupe de disponibilité

1. Dans l’**Explorateur d’objets**, développez **Haute disponibilité Always On**, puis **Groupes de disponibilité**. Vous devez maintenant voir le nouveau groupe de disponibilité dans ce conteneur. Cliquez avec le bouton droit sur le groupe de disponibilité et sélectionnez **Afficher le tableau de bord**.

   ![Afficher le tableau de bord du groupe de disponibilité](./media/availability-group-manually-configure-tutorial/76-showdashboard.png)

   Votre **Tableau de bord Always On** doit ressembler à celui illustré dans la capture d’écran suivante :

   ![Tableau de bord du groupe de disponibilité](./media/availability-group-manually-configure-tutorial/78-agdashboard.png)

   Vous pouvez voir les réplicas, le mode de basculement de chaque réplica et l'état de synchronisation.

2. Dans le **Gestionnaire du cluster de basculement**, sélectionnez votre cluster. Sélectionnez **Rôles**. Le nom de groupe de disponibilité que vous avez utilisé est un rôle du cluster. Ce groupe de disponibilité n’a pas d’adresse IP pour les connexions clientes, car vous n’avez pas configuré d’écouteur. Vous allez configurer l’écouteur après avoir créé un équilibrage de charge Azure.

   ![Groupe de disponibilité dans le Gestionnaire du cluster de basculement](./media/availability-group-manually-configure-tutorial/80-clustermanager.png)

   > [!WARNING]
   > N'essayez pas de basculer le groupe de disponibilité à partir du Gestionnaire du Cluster de basculement. Vous devez effectuer toutes les opérations de basculement à partir du **tableau de bord AlwaysOn** dans SSMS. Pour plus d’informations, consultez [Restrictions d’utilisation du Gestionnaire du cluster de basculement avec des groupes de disponibilité](/sql/database-engine/availability-groups/windows/failover-clustering-and-always-on-availability-groups-sql-server).
    >

À ce stade, vous avez un groupe de disponibilité avec des réplicas sur les deux instances de SQL Server. Vous pouvez déplacer le groupe de disponibilité entre les instances. Vous ne pouvez pas encore vous connecter au groupe de disponibilité, car vous n’avez pas d’écouteur. Dans les machines virtuelles Azure, l’écouteur requiert un équilibrage de charge. L’étape suivante consiste à créer l’équilibrage de charge dans Azure.

<a name="configure-internal-load-balancer"></a>

## <a name="create-an-azure-load-balancer"></a>Crée un équilibrage de charge Azure

[!INCLUDE [sql-ag-use-dnn-listener](../../includes/sql-ag-use-dnn-listener.md)]

Sur les machines virtuelles Azure, un groupe de disponibilité SQL Serveur nécessite un équilibreur de charge. Cet équilibreur de charge stocke les adresses IP des écouteurs du groupe de disponibilité et du cluster de basculement Windows Server. Cette section indique comment créer l’équilibrage de charge dans le portail Azure.

Un équilibreur de charge dans Azure peut être Standard Load Balancer ou Load Balancer de base. Une équilibreur de charge Standard offre davantage de fonctionnalités qu’un équilibreur de charge De base. Pour un groupe de disponibilité, un équilibreur de charge Standard est requis si vous utilisez une zone de disponibilité (au lieu d’un groupe à haute disponibilité). Pour plus d’informations sur la différence entre les références (SKU) d’équilibreurs de charge, voir [Comparaison des références SKU de Load Balancer](../../../load-balancer/skus.md).

1. Dans le portail Azure, accédez au groupe de ressources où se trouvent vos serveurs SQL Server et sélectionnez **+ Ajouter**.
1. Recherchez **Équilibrage de charge**. Choisissez l’équilibrage de charge publié par Microsoft.

   ![Choisissez l’équilibreur de charge publié par Microsoft.](./media/availability-group-manually-configure-tutorial/82-azureloadbalancer.png)

1. Sélectionnez **Create** (Créer).
1. Configurez les paramètres suivants pour l’équilibreur de charge.

   | Paramètre | Champ |
   | --- | --- |
   | **Nom** |Utilisez un nom pour l’équilibrage de charge, par exemple **sqlLB**. |
   | **Type** |Interne |
   | **Réseau virtuel** |Utilisez le nom de votre réseau virtuel Azure. |
   | **Sous-réseau** |Utilisez le nom du sous-réseau auquel appartient la machine virtuelle.  |
   | **Affectation d’adresses IP** |statique |
   | **Adresse IP** |Utilisez une adresse disponible du sous-réseau. Utilisez cette adresse pour votre écouteur de groupe de disponibilité. Notez que celle-ci diffère de l’adresse IP de votre cluster.  |
   | **Abonnement** |Utilisez le même abonnement que la machine virtuelle. |
   | **Lieu** |Utilisez le même emplacement que la machine virtuelle. |

   Le panneau du portail Azure doit ressembler à ceci :

   ![Créer un équilibreur de charge](./media/availability-group-manually-configure-tutorial/84-createloadbalancer.png)

1. Sélectionnez **Créer** pour générer l’équilibreur de charge.

Pour configurer l’équilibrage de charge, vous devez créer un pool principal ainsi qu’une sonde et définir les règles d’équilibrage de charge. Effectuez ces opérations dans le portail Azure.

### <a name="add-a-backend-pool-for-the-availability-group-listener"></a>Ajouter un pool principal pour l’écouteur de groupe de disponibilité

1. Dans le portail Azure, accédez à votre groupe de disponibilité. Vous devrez peut-être actualiser l’affichage pour voir l’équilibrage de charge tout juste créé.

   ![Rechercher l’équilibrage de charge dans le groupe de ressources](./media/availability-group-manually-configure-tutorial/86-findloadbalancer.png)

1. Sélectionnez l’équilibreur de charge, sélectionnez les **Pools principaux**, puis cliquez sur **Ajouter**.

1. Entez un nom pour le pool principal.

1. Associez le pool principal au groupe à haute disponibilité contenant les machines virtuelles.

1. Sous **Configurations IP du réseau cible**, cochez **MACHINE VIRTUELLE** et choisissez les deux machines virtuelles qui hébergeront les réplicas du groupe de disponibilité. N’incluez pas le serveur témoin de partage de fichiers.

   >[!NOTE]
   >Si les deux machines virtuelles ne sont pas spécifiées, la connexion sera établie avec le réplica principal uniquement.

1. Sélectionnez **OK** pour créer le pool principal.

### <a name="set-the-probe"></a>Configurer la sonde

1. Sélectionnez l’équilibreur de charge, sélectionnez les **Probes d’intégrité**, puis cliquez sur **+ Ajouter**.

1. Définissez la sonde d’intégrité de l’écouteur comme suit :

   | Paramètre | Description | Exemple
   | --- | --- |---
   | **Nom** | Texte | SQLAlwaysOnEndPointProbe |
   | **Protocole** | Choisissez TCP. | TCP |
   | **Port** | Tout port inutilisé. | 59999 |
   | **Intervalle**  | Durée entre les tentatives de la sonde, en secondes. |5 |
   | **Seuil de défaillance sur le plan de l’intégrité** | Nombre d’échecs de sonde consécutifs qui doivent se produire pour qu’une machine virtuelle soit considérée comme défectueuse.  | 2 |

1. Sélectionnez **OK** pour définir la probe d’intégrité.

### <a name="set-the-load-balancing-rules"></a>Configurer les règles d’équilibrage de charge

1. Sélectionnez l’équilibreur de charge, sélectionnez **Règles d’équilibreur de charge**, puis cliquez sur **+Ajouter**.

1. Configurez les règles d’équilibrage de charge de l’écouteur comme suit.

   | Paramètre | Description | Exemple
   | --- | --- |---
   | **Nom** | Texte | SQLAlwaysOnEndPointListener |
   | **Frontend IP address (Adresse IP frontale)** | Choisissez une adresse. |Utilisez l’adresse que vous avez créée lorsque vous avez créé l’équilibrage de charge. |
   | **Protocole** | Choisissez TCP. |TCP |
   | **Port** | Utiliser le port pour l'écouteur de groupe de disponibilité | 1433 |
   | **Port principal** | Ce champ est inutilisé lorsque l’option IP flottante est définie pour Retour au serveur direct. | 1433 |
   | **Sonde** |Nom que vous avez spécifié pour la sonde. | SQLAlwaysOnEndPointProbe |
   | **Persistance de session** | Liste déroulante | **Aucun** |
   | **Délai d’inactivité** | Délai en minutes de maintien d’une connexion TCP ouverte | 4 |
   | **Adresse IP flottante (retour direct du serveur)** | |activé |

   > [!WARNING]
   > Le retour au serveur direct est configuré lors de la création. Cette valeur n’est pas modifiable.
   >

1. Sélectionnez **OK** pour définir les règles d’équilibrage de charge de l’écouteur.

### <a name="add-the-cluster-core-ip-address-for-the-windows-server-failover-cluster-wsfc"></a>Ajouter l’adresse IP du cluster principal pour le cluster de basculement Windows Server (WSFC)

L’adresse IP du cluster WSFC doit également se trouver sur l’équilibreur de charge.

1. Dans le Portail Azure, accédez au même équilibreur de charge Azure. Sélectionnez **Configuration d’adresse IP frontale**, puis sélectionnez **+Ajouter**. Utilisez l’adresse IP que vous avez configurée pour le cluster WSFC dans les ressources principales du cluster. Définissez l’adresse IP comme statique.

1. Sur l’équilibreur de charge, sélectionnez les **Probes d’intégrité**, puis cliquez sur **+Ajouter**.

1. Définissez la sonde d’intégrité de l’adresse IP du cluster principal WSFC comme suit :

   | Paramètre | Description | Exemple
   | --- | --- |---
   | **Nom** | Texte | WSFCEndPointProbe |
   | **Protocole** | Choisissez TCP. | TCP |
   | **Port** | Tout port inutilisé. | 58888 |
   | **Intervalle**  | Durée entre les tentatives de la sonde, en secondes. |5 |
   | **Seuil de défaillance sur le plan de l’intégrité** | Nombre d’échecs de sonde consécutifs qui doivent se produire pour qu’une machine virtuelle soit considérée comme défectueuse.  | 2 |

1. Sélectionnez **OK** pour définir la probe d’intégrité.

1. Configurez les règles d’équilibrage de charge. Sélectionnez **Règles d’équilibrage de charge**, puis sélectionnez **+Ajouter**.

1. Définissez les règles d’équilibrage de charge de l’adresse IP du cluster principal comme suit.

   | Paramètre | Description | Exemple
   | --- | --- |---
   | **Nom** | Texte | WSFCEndPoint |
   | **Frontend IP address (Adresse IP frontale)** | Choisissez une adresse. |Utilisez l’adresse que vous avez créée quand vous avez configuré l’adresse IP du cluster WSFC. Celle-ci diffère de l’adresse IP de l’écouteur |
   | **Protocole** | Choisissez TCP. |TCP |
   | **Port** | Utilisez le port de l’adresse IP du cluster. Il s’agit d’un port disponible qui n’est pas utilisé pour le port de la sonde de l’écouteur. | 58888 |
   | **Port principal** | Ce champ est inutilisé lorsque l’option IP flottante est définie pour Retour au serveur direct. | 58888 |
   | **Sonde** |Nom que vous avez spécifié pour la sonde. | WSFCEndPointProbe |
   | **Persistance de session** | Liste déroulante | **Aucun** |
   | **Délai d’inactivité** | Délai en minutes de maintien d’une connexion TCP ouverte | 4 |
   | **Adresse IP flottante (retour direct du serveur)** | |activé |

   > [!WARNING]
   > Le retour au serveur direct est configuré lors de la création. Cette valeur n’est pas modifiable.
   >

1. Cliquez sur **OK** pour définir les règles d’équilibrage de charge.

## <a name="configure-the-listener"></a><a name="configure-listener"></a> Configurer l’écouteur

La prochaine étape consiste à configurer un écouteur de groupe de disponibilité sur le cluster de basculement.

> [!NOTE]
> Ce didacticiel montre comment créer un seul écouteur avec une adresse IP d’ILB. Pour créer un ou plusieurs écouteurs en utilisant une ou plusieurs adresses IP, consultez [Créer un écouteur de groupe de disponibilité et un équilibreur de charge | Azure](availability-group-listener-powershell-configure.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
>

[!INCLUDE [ag-listener-configure](../../../../includes/virtual-machines-ag-listener-configure.md)]

## <a name="set-listener-port"></a>Configurer le port d’écoute

Dans SQL Server Management Studio, configurez le port d’écoute.

1. Lancez SQL Server Management Studio et connectez-vous au réplica principal.

1. Accédez à **Haute disponibilité AlwaysOn** > **Groupes de disponibilité** > **Écouteurs de groupe de disponibilité**.

1. Vous devez maintenant voir le nom de l'écouteur que vous avez créé dans le Gestionnaire du cluster de basculement. Cliquez avec le bouton droit sur le nom de l’écouteur, puis sélectionnez **Propriétés**.

1. Dans la zone **Port**, spécifiez le numéro de port de l’écouteur de groupe de disponibilité. La valeur par défaut est 1433. Sélectionnez **OK**.

Vous avez maintenant un groupe de disponibilité SQL Server dans des machines virtuelles Azure exécutées en mode Resource Manager.

## <a name="test-connection-to-listener"></a>Tester la connexion à l’écouteur

Pour tester la connexion :

1. Utilisez une requête RDP pour vous connecter à un serveur SQL Server qui se trouve dans le même réseau virtuel, mais qui ne possède pas le réplica. Vous pouvez utiliser l’autre serveur SQL Server du cluster.

1. Utilisez l’utilitaire **sqlcmd** pour tester la connexion. Par exemple, le script suivant établit une connexion **sqlcmd** avec le réplica principal au moyen de l’écouteur avec une authentification Windows :

   ```cmd
   sqlcmd -S <listenerName> -E
   ```

   Si l’écouteur utilise un port autre que le port par défaut (1433), spécifiez le port dans la chaîne de connexion. Par exemple, la commande `sqlcmd` suivante se connecte à un écouteur sur le port 1435 :

   ```cmd
   sqlcmd -S <listenerName>,1435 -E
   ```

La connexion SQLCMD se connecte automatiquement à l’instance SQL Server hébergeant le réplica principal.

> [!TIP]
> Vérifiez que le port spécifié est ouvert sur le pare-feu des deux serveurs SQL. Les deux serveurs requièrent une règle de trafic entrant sur le port TCP utilisé. Pour plus d’informations, consultez [Ajouter ou modifier une règle de pare-feu](/previous-versions/orphan-topics/ws.11/cc753558(v=ws.11)).
>

## <a name="next-steps"></a>Étapes suivantes

- [Add an IP address to an existing load balancer with PowerShell (Ajouter une adresse IP à un équilibrage de charge pour un deuxième groupe de disponibilité)](availability-group-listener-powershell-configure.md#Add-IP).