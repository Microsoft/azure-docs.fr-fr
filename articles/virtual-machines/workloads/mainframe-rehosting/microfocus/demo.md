---
title: Configurer Micro Focus CICS BankDemo pour Micro Focus Enterprise Developer 4.0 sur Machines virtuelles Azure
description: Exécutez l’application Micro Focus BankDemo sur Machines virtuelles Azure pour découvrir comment utiliser Micro Focus Enterprise Server et Enterprise Developer.
author: sread
ms.author: sread
ms.date: 03/30/2020
ms.topic: article
ms.service: multiple
ms.openlocfilehash: 7fb72b9a7d0d655f99d1e5cf194f7c6f26976a37
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95976191"
---
# <a name="set-up-micro-focus-cics-bankdemo-for-micro-focus-enterprise-developer-40-on-azure"></a>Configurer Micro Focus CICS BankDemo pour Micro Focus Enterprise Developer 4.0 sur Azure

Quand vous configurez Micro Focus Enterprise Server 4.0 et Enterprise Developer 4.0 sur Azure, vous pouvez tester les déploiements de charges de travail IBM z/OS. Cet article explique comment configurer CICS BankDemo, un exemple d’application qui est fourni avec Enterprise Developer.

CICS est l’acronyme de Customer Information Control System, la plateforme transactionnelle utilisée par de nombreuses applications mainframe en ligne. L’application BankDemo est idéale pour découvrir comment Enterprise Server et code Enterprise Developer fonctionnent, et comment gérer et déployer une application réelle avec terminaux à écran vert.

> [!NOTE]
> Bientôt disponible : Instructions pour la configuration de [Micro Focus Enterprise Server 5.0](https://techcommunity.microsoft.com/t5/azurecat/micro-focus-enterprise-server-5-0-quick-start-template-on-azure/ba-p/1160110) sur des machines virtuelles Azure.

## <a name="prerequisites"></a>Prérequis

- Une machine virtuelle avec [Enterprise Developer](set-up-micro-focus-azure.md). Gardez à l’esprit qu’Enterprise Developer dispose d’une instance complète d’Enterprise Server utilisable pour le développement et les tests. Cette instance est l’instance d’Enterprise Developer utilisée pour la démonstration.

- [SQL Server 2017 Express Edition](https://www.microsoft.com/sql-server/sql-server-editions-express). Téléchargez-le et installez-le sur la machine virtuelle Enterprise Developer. Enterprise Developer nécessite une base de données pour la gestion des régions CICS ; l’application BankDemo utilise également une base de données SQL Server appelée BANKDEMO. Cette démonstration suppose que vous utilisez SQL Server Express pour les deux bases de données. Lors de l’installation, sélectionnez l’installation de base.

- [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-2017) (SSMS). SSMS est utilisé pour la gestion des bases de données et l’exécution d’un script T-SQL. Téléchargez-le et installez-le sur la machine virtuelle Enterprise Developer.

- [Visual Studio 2019](https://azure.microsoft.com/downloads/) avec le dernier Service Pack ou [Visual Studio Community](https://visualstudio.microsoft.com/vs/community/), que vous pouvez télécharger gratuitement.

- Rumba Desktop ou un autre émulateur 3270.

## <a name="configure-the-windows-environment"></a>Configurer l’environnement Windows

Après avoir installé Enterprise Developer 4.0 sur la machine virtuelle, vous devez configurer l’instance d’Enterprise Server qui l’accompagne. Pour cela, vous devez installer quelques fonctionnalités Windows supplémentaires, comme suit.

1. Utilisez RDP pour vous connecter à la machine virtuelle Enterprise Server 4.0 que vous avez créée.

2. Cliquez sur l’icône **Rechercher** en regard du bouton **Démarrer** et tapez **fonctionnalités Windows**. L’Assistant Ajout de rôles et de fonctionnalités de Server Manager s’ouvre.

3. Sélectionnez **Rôle de serveur web (IIS)** , puis cochez les options suivantes :

    - Outils d’administration Web
    - Compatibilité d’administration IIS - IIS 6 (sélectionnez toutes les fonctionnalités disponibles)
    - Console de gestion d’IIS
    - Scripts et outils de gestion d’IIS
    - Service de gestion d’IIS

4. Sélectionnez **Services World Wide Web** et cochez les options suivantes :

     Fonctionnalités de développement d’applications :
    - Extensibilité .NET
    - ASP.NET
    - Fonctionnalités HTTP communes : Ajoutez toutes les fonctionnalités disponibles
    - Intégrité et diagnostics : Ajoutez toutes les fonctionnalités disponibles
    - Sécurité :
        - Authentification de base
        - Authentification Windows

5. Sélectionnez **Service d’activation des processus Windows** et tous ses enfants.

6. Pour **Fonctionnalités**, cochez **Microsoft .NET Framework 3.5.1** et cochez les options suivantes :

    - Activation HTTP de Windows Communication Foundation
    - Activation non-HTTP de Windows Communication Foundation

7. Pour **Fonctionnalités**, cochez **Microsoft .NET Framework 4.6** et cochez les options suivantes :

   - Activation de canaux nommés
   - Activation de TCP
   - Partage de port TCP

     ![Assistant Ajout de rôles et de fonctionnalités : Services de rôle](media/01-demo-roles.png)

8. Quand vous avez sélectionné toutes les options, cliquez sur **Suivant** pour installer.

9. Une fois les fonctionnalités Windows installées, accédez à **Panneau de configuration \> Système et sécurité \> Outils d’administration**, puis sélectionnez **Services**. Faites défiler vers le bas et vérifiez que les services suivants sont en cours d’exécution et définis sur **Automatique** :

    - **NetTcpPortSharing**
    - **Adaptateur d’écouteur Net.Pipe**
    - **Adaptateur d’écouteur Net.Tcp**

10. Pour configurer la prise en charge d’IIS et de WAS, dans le menu, recherchez **Micro Focus Enterprise Developer Command Prompt (64 bit)** (Invite de commandes Micro Focus Enterprise Developer (64 bits)) et exécutez en tant que **Administrateur**.

11. Tapez **wassetup –i** et appuyez sur **Entrée**.

12. Après l’exécution du script, vous pouvez fermer la fenêtre.

## <a name="configure-the-local-system-account-for-sql-server"></a>Configurer le compte système local pour SQL Server

Certains processus Enterprise Server doivent être en mesure de se connecter à SQL Server, et de créer des bases de données et d’autres objets. Ces processus utilisent le compte système local : vous devez donc donner une autorité sysadmin à ce compte.

1. Lancez **SSMS** et cliquez sur **Se connecter** pour vous connecter au serveur SQLEXPRESS local avec l’authentification Windows. Il doit être disponible dans la liste **Nom du serveur**.

2. Sur la gauche, développez le dossier **Sécurité** et sélectionnez **Connexions**.

3. Sélectionnez **AUTORITÉ NT\\SYSTÈME**, puis sélectionnez **Propriétés**.

4. Sélectionnez **Rôles de serveur** et cochez **sysadmin**.

     ![Fenêtre de l’Explorateur d’objets SSMS : Propriétés de connexion](media/02-demo-explorer.png)

## <a name="create-the-bankdemo-database-and-all-its-objects"></a>Créer la base de données BankDemo et tous ses objets

1. Ouvrez l’**Explorateur Windows** et accédez à **C:\\Users\\Public\\Documents\\Micro Focus\\Enterprise Developer\\ Samples\\Mainframe\\CICS\\DotNet\\BankDemo\\SQL**.

2. Copiez le contenu du fichier **BankDemoCreateAll.SQL** dans le Presse-papiers.

3. Ouvrez **SSMS**. Sur la droite, cliquez sur **Serveur** et sélectionnez **Nouvelle requête**.

4. Collez le contenu du Presse-papiers dans la zone **Nouvelle requête**.

5. Exécutez le code SQL en cliquant sur **Exécuter** à partir de l’onglet **Commande** au-dessus de la requête.

La requête doit normalement s’exécuter sans erreurs. Une fois qu’elle est terminée, vous disposez de l’exemple de base de données pour l’application BankDemo.

![Sortie de SQLQuery1.sql](media/03-demo-query.png)

## <a name="verify-that-the-database-tables-and-objects-have-been-created"></a>Vérifiez que les tables et les objets de la base de données ont été créés.

1. Cliquez sur la base de données **BANKDEMO** et sélectionnez **Actualiser**.

2. Développez la **Base de données** et sélectionnez **Tables**. Vous voyez normalement quelque chose de similaire à ceci.

     ![Table de BANKDEMO développée dans l’Explorateur d’objets](media/04-demo-explorer.png)

## <a name="build-the-application-in-enterprise-developer"></a>Générer l’application dans Enterprise Developer

1. Ouvrez Visual Studio et connectez-vous.

2. Sous l’option de menu **Fichier**, sélectionnez **Ouvrir un projet/une solution**, accédez à **C:\\Users\\Public\\Documents\\Micro Focus\\Enterprise Developer\\Samples\\Mainframe\\CICS\\DotNet\\BankDemo**, puis sélectionnez le fichier **sln**.

3. Prenez un peu de temps pour examiner les objets. Les programmes COBOL sont affichées dans l’Explorateur de solutions avec l’extension CBL ainsi que CopyBooks (CPY) et JCL.

4. Cliquez avec le bouton droit sur le projet **BankDemo2**, puis sélectionnez **Définir comme projet de démarrage**.

    > [!NOTE]
    > Le projet BankDemo utilise HCOSS (Host Compatibility Option for SQL Server), qui n’est pas utilisé pour cette démonstration.

5. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet **BankDemo2** et sélectionnez **Générer**.

    > [!NOTE]
    > La génération au niveau de la solution produit des erreurs, car HCOSS n’a pas été configuré.

6. Une fois le projet généré, examinez la fenêtre **Sortie**. Elle doit se présenter comme l’image ci-dessous.

     ![Fenêtre Sortie montrant la build réussie](media/05-demo-output.png)

## <a name="deploy-the-bankdemo-application-into-the-region-database"></a>Déployer l’application BankDemo dans la base de données Region

1. Ouvrez une invite de commandes Enterprise Developer (64 bits) en tant qu’administrateur.

2. Accédez à **%PUBLIC%\\Documents\\Micro Focus\\Enterprise Developer\\samples\\Mainframe\\CICS\\DotNet\\BankDemo**.

3. À l’invite de commandes, exécutez **bankdemodbdeploy** et incluez le paramètre pour la base de données à déployer, par exemple :

    ```
    bankdemodbdeploy (local)/sqlexpress
    ```

> [!NOTE]
> Veillez à utiliser une barre oblique (/) et non pas une barre oblique inverse (\\). Ce script s’exécute pendant un certain temps.

![Administration : Fenêtre d’invite de commandes Enterprise Developer](media/06-demo-cmd.png)

## <a name="create-the-bankdemo-region-in-enterprise-administrator-for-net"></a>Créer la région BankDemo dans Enterprise Administrator pour .NET

1. Ouvrez l’interface utilisateur de **Enterprise Server for .NET Administration**.

2. Pour démarrer le composant logiciel enfichable MMC, dans le menu **Démarrer** de Windows, choisissez **Micro Focus Enterprise Developer \> Configuration \> Enterprise Server for .NET Admin**. (Pour Windows Server, choisissez **Micro Focus Enterprise Developer \> Enterprise Server for .NET Admin**).

3. Développez le conteneur **Regions** dans le volet gauche, puis cliquez avec le bouton droit sur **CICS**.

4. Sélectionnez **Define Region** (Définir une région) pour créer une nouvelle région CICS nommée **BANKDEMO**, hébergée dans la base de données (locale).

5. Spécifiez l’instance du serveur de base de données, cliquez sur **Next** (Suivant), puis entrez le nom de la région **BANKDEMO**.

     ![Boîte de dialogue Define Region (Définir une région)](media/07-demo-cics.png)

6. Pour sélectionner le fichier de définition de région pour la base de données inter-régions, accédez à **region\_bankdemo\_db.config** dans **C:\\Users\\Public\\Documents\\Micro Focus\\Enterprise Developer\\Samples\\Mainframe\\CICS\\DotNet\\BankDemo**.

     ![Define Region - Region name (Définir la région - Nom de la région) : BANKDEMO](media/08-demo-cics.png)

7. Cliquez sur **Terminer**.

## <a name="create-xa-resource-definitions"></a>Créer des définitions de ressources XA

1. Dans le volet gauche de l’interface utilisateur de **Enterprise Server for .NET Administration**, développez **System** (Système), puis **XA Resource Definitions** (Définitions de ressources XA). Ce paramètre définit la façon dont la région interagit avec Enterprise Server et avec les bases de données de l’application.

2. Cliquez avec le bouton droit sur **XA Resource Definitions** (Définitions de ressources XA) et sélectionnez **Add Server Instance** (Ajouter une instance de serveur).

3. Dans la zone de liste déroulante, sélectionnez **Database Service Instance** (Instance de service de base de données). Il s’agit de la machine locale SQLEXPRESS.

4. Sélectionnez l’instance sous le conteneur **XA Resource Definitions (nom_machine\\sqlexpress)** , puis cliquez sur **Add** (Ajouter).

5. Sélectionnez **Database XA Resource Definition** (Définition de ressource de base de données XA), puis tapez **BANKDEMO** pour **Name** (Nom) et pour **Region** (Région).

     ![Écran de définition d’une ressource de base de données XA](media/09-demo-xa.png)

6. Cliquez sur les points de suspension ( **...** ) pour ouvrir l’Assistant Connection String (Chaîne de connexion). Pour **Server Name** (Nom du serveur), tapez **(local)\\SQLEXPRESS**. Pour **Logon** (Ouverture de session), sélectionnez **Windows Authentication** (Authentification Windows). Pour le nom de la base de données, tapez **BANKDEMO**

     ![Écran de modification de la chaîne de connexion](media/10-demo-string.png)

7. Testez la connexion.

## <a name="start-the-bankdemo-region"></a>Démarrer la région BANKDEMO

> [!NOTE]
> La première étape est importante : Vous devez définir la région pour qu’elle utilise la définition de ressource XA que vous venez de créer.

1. Accédez à la **Région CICS BANKDEMO** sous **Regions Container** (Conteneur de régions), puis sélectionnez **Edit Region Startup File** (Modifier le fichier de démarrage de la région) dans le volet **Actions**. Faites défiler vers le bas jusqu’aux propriétés de SQL et entrez **bankdemo** pour **XA resource name** (Nom de la ressource XA), ou utilisez les points de suspension pour le sélectionner.

2. Cliquez sur le bouton **Save** (Enregistrer) pour enregistrer vos modifications.

3. Cliquez avec le bouton droit sur **BANKDEMO CICS Region** dans le volet **Console** et sélectionnez **Start/Stop Region** (Démarrer/arrêter la région).

4. Dans le bas de la zone **Start/Stop Region** (Démarrer/arrêter la région) qui apparaît dans le volet central, sélectionnez **Start** (Démarrer). Après quelques secondes, la région démarre.

     ![Zone Start/Stop (Démarrer/arrêter) de SQL](media/11-demo-sql.png)

     ![Écran CICS Region BANKDEMO - Started (Région CICS BANKDEMO - Démarrée)](media/12-demo-cics.png)

## <a name="create-a-listener"></a>Créer un écouteur

Créez un écouteur pour les sessions TN3270 qui accèdent à l’application BankDemo.

1. Dans le volet gauche, développez **Configuration Editors** (Éditeurs de configuration) et sélectionnez **Listener** (Écouteur).

2. Cliquez sur l’icône **Ouvrir un fichier** et sélectionnez le fichier **seelistener.exe.config**. Ce fichier est modifié et chargé chaque fois qu’Enterprise Server démarre.

3. Notez les deux régions précédemment définies (ESDEMO et JCLDEMO).

4. Pour créer une nouvelle région pour BANKDEMO, cliquez avec le bouton droit sur **Regions** (Régions), puis sélectionnez **Add Region** (Ajouter une région).

5. Sélectionnez **BANKDEMO Region**.

6. Ajoutez un canal TN3270 en double-cliquant sur **BANKDEMO Region** et en sélectionnant **Add Channel** (Ajouter un canal).

7. Pour **Name** (Name), entrez **TN3270**. Pour **Port**, entrez **9024**. L’application ESDEMO utilise le port 9230 : vous devez donc utiliser un autre port.

8. Pour enregistrer le fichier, cliquez sur l’icône **Enregistrer** ou choisissez **File** (Fichier) \> **Save** (Enregistrer).

9. Pour démarrer l’écouteur, cliquez sur l’icône **Démarrer l’écouteur** ou choisissez **Options** \> **Start Listener** (Démarrer l’écouteur).

     ![Fenêtres de l’éditeur de configuration d’écouteur](media/13-demo-listener.png)


## <a name="configure-rumba-to-access-the-bankdemo-application"></a>Configurer Rumba pour accéder à l’application BankDemo

La dernière chose que vous devez faire est de configurer une session 3270 en utilisant Rumba, un émulateur 3270. Cette étape vous permet d’accéder à l’application BankDemo via l’écouteur que vous avez créé.

1. Dans le menu **Démarrer** de Windows, lancez Rumba Desktop.

2. Sous l’élément de menu **Connections** (Connexions), sélectionnez **TN3270**.

3. Cliquez sur **Insert** (Insérer) et tapez **127.0.0.1** pour l’adresse IP et **9024** pour le port défini par l’utilisateur.

4. Dans le bas de la boîte de dialogue, cliquez sur **Connect** (Se connecter). Un écran CICS noir apparaît.

5. Type **bank** pour afficher l’écran 3270 initial pour l’application BankDemo.

6. Pour l’ID d’utilisateur, tapez **B0001** et pour le mot de passe, tapez ce que vous voulez. Le premier écran BANK20 s’ouvre.

![Affichage du mainframe - Écran d’accueil](media/14-demo.png)
![Affichage du mainframe - Rumba - Écran de démonstration du sous-système](media/15-demo.png)

Félicitations ! Vous exécutez à présent une application CICS dans Azure en utilisant Micro Focus Enterprise Server.

## <a name="next-steps"></a>Étapes suivantes

- [Exécuter Enterprise Server dans des conteneurs Docker sur Azure](run-enterprise-server-container.md)
- [Migration de mainframe - Portail](/archive/blogs/azurecat/mainframe-migration-to-azure-portal)
- [Machines virtuelles](../../../linux/overview.md)
- [Dépannage](../../../troubleshooting/index.yml)
- [Démystifier la migration d’un mainframe vers Azure](https://azure.microsoft.com/resources/demystifying-mainframe-to-azure-migration/en-us/)
