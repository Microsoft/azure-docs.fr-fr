---
title: Configurer un laboratoire pour gérer et développer avec Azure SQL Database | Azure Lab Services
description: Découvrez comment configurer un laboratoire pour gérer et développer avec Azure SQL Database.
author: emaher
ms.topic: article
ms.date: 06/26/2020
ms.author: enewman
ms.openlocfilehash: 50f71ee1ce59f5809fe8905c58f0399cf484f11a
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659707"
---
# <a name="set-up-a-lab-to-manage-and-develop-with-sql-server"></a>Configurer un laboratoire pour gérer et développer avec SQL Server

Cet article décrit comment configurer un laboratoire pour une classe de base sur le développement et le gestion de SQL Server dans Azure Lab Services.  Les concepts relatifs aux bases de données sont l’un des cours d’introduction enseignés dans la plupart des départements d’informatique à l’université. Le langage SQL est une norme internationale.  SQL est le langage standard pour la gestion des bases de données relationnelles, notamment l’ajout, la récupération et la gestion du contenu dans une base de données.  Il est surtout indiqué pour sa capacité de traitement rapide, sa fiabilité, sa facilité et sa flexibilité d’utilisation.

Dans cet article, nous allons voir comment configurer un modèle de machine virtuelle dans un laboratoire avec [Visual Studio 2019](https://visualstudio.microsoft.com/vs/), [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15) et [Azure Data Studio](https://github.com/microsoft/azuredatastudio).  Nous utiliserons une [base de données SQL Server](../azure-sql/database/sql-database-paas-overview.md) partagée pour l’ensemble du laboratoire. [Azure SQL Database](../azure-sql/database/sql-database-paas-overview.md) est une offre de moteur de base de données PaaS (platform-as-a-service) d’Azure.

## <a name="lab-configuration"></a>Configuration du laboratoire

Pour configurer ce labo, vous avez besoin d’un abonnement Azure et d’un compte Lab pour commencer. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer. Une fois que vous disposez d’un abonnement Azure, vous pouvez créer un compte lab dans Azure Lab Services. Pour plus d’informations sur la création d’un compte de laboratoire, consultez notre [Tutoriel pour configurer un compte Lab](./tutorial-setup-lab-account.md). Vous pouvez également utiliser un compte Lab existant.

### <a name="lab-account-settings"></a>Paramètres du compte Lab

Activez les paramètres décrits dans le tableau ci-dessous pour le compte lab. Pour plus d’informations sur l’activation des images de la Place de marché, consultez [Spécifier les images de la Place de marché accessibles aux créateurs de labo](specify-marketplace-images.md).

| Paramètres du compte lab | Instructions |
| ------------------- | ------------ |
| Image de la Place de marché | Activez l’image « Visual Studio 2019 Community (dernière version) sur Windows 10 Enterprise N (x64) » pour l’utiliser dans votre compte lab. |

### <a name="shared-resource-configuration"></a>Configuration d’une ressource partagée

Pour utiliser une ressource partagée dans Lab Services, vous devez d’abord créer le réseau virtuel et les ressources en question.  Pour créer le réseau virtuel et le connecter au laboratoire, consultez le [guide pratique pour créer un laboratoire avec une ressource partagée dans Azure Lab Services](how-to-create-a-lab-with-shared-resource.md).  N’oubliez pas que les ressources externes à Lab Services sont facturées séparément et ne sont pas incluses dans les estimations des coûts de laboratoire.

>[!WARNING]
>Les ressources partagées d’un laboratoire doivent être configurées avant la création du laboratoire.  Si le réseau virtuel n’est pas [appairé au compte lab](how-to-connect-peer-virtual-network.md) *avant* la création du laboratoire, celui-ci n’aura pas accès à la ressource partagée.

Maintenant que la partie réseau est traitée, créons une base de données SQL Server.  Nous allons créer une [base de données unique](../azure-sql/database/single-database-create-quickstart.md?tabs=azure-portal), car c’est l’option de déploiement la plus rapide pour Azure SQL Database.  Pour les autres options de déploiement, créez un [pool élastique](../azure-sql/database/elastic-pool-overview.md#creating-a-new-sql-database-elastic-pool-using-the-azure-portal), une [instance managée](../azure-sql/managed-instance/instance-create-quickstart.md) ou une [machine virtuelle SQL](../azure-sql/virtual-machines/windows/sql-vm-create-portal-quickstart.md).

1. Dans le menu du portail Azure, choisissez **Créer une ressource**.
2. Choisissez **Base de données SQL**, puis cliquez sur le bouton **Créer**.
3. Sous l’onglet **De base** du formulaire **Créer une base de données SQL**, sélectionnez le groupe de ressources pour la base de données.  Nous allons utiliser *sqldb-rg*.
4. Pour **Nom de la base de données**, entrez *classlabdb*.
5. Sous le paramètre **Serveur**, cliquez sur **Créer** pour créer un serveur qui contiendra la base de données.
6. Dans le menu volant **Nouveau serveur**, entrez le nom du serveur.  Nous allons utiliser *classlabdbserver*.  Le nom du serveur doit être globalement unique.
7. Entrez *azureuser* comme **Connexion administrateur au serveur**.
8. Entrez un mot de passe facile à retenir.  Le mot de passe doit comporter au moins huit caractères et contenir des caractères spéciaux.
9. Choisissez une région comme **emplacement**.  Si possible, entrez le même emplacement que le compte lab et le réseau virtuel appairé pour réduire la latence.
10. Cliquez sur **OK** pour revenir au formulaire **Créer une base de données SQL**.
11. Cliquez sur le lien **Configurer la base de données** sous le paramètre **Calcul + Stockage**.
12. Modifiez les paramètres de base de données en fonction des besoins de la classe.  Vous avez le choix entre les options Provisionné et Serverless.  Dans cet exemple, nous allons utiliser l’option Serverless avec mise à l’échelle automatique et affecter à vCores max. la valeur 4 et à vCores min. la valeur 1. Conservons la valeur minimale pour le paramètre de pause automatique, c’est-à-dire 1 heure. Cliquez sur **Appliquer**.
13. Cliquez sur **Suivant : Réseau**.
14. Sous l’onglet Réseau, choisissez Point de terminaison privé comme **Méthode de connectivité**.
15. Sous la section **Points de terminaison privés**, cliquez sur **Ajouter un point de terminaison privé**.
16. Dans le menu volant **Créer un point de terminaison privé**, choisissez le même groupe de ressources que celui de votre réseau virtuel appairé au compte lab.
17. Pour **Emplacement**, choisissez le même emplacement que celui du réseau virtuel.
18. Pour **Nom**, entrez *labsql-endpt*.
19. Laissez SqlServer comme Sous-ressource cible.
20. Pour **Réseau virtuel**, choisissez le même réseau virtuel que celui appairé au compte Lab.
21. Pour **Sous-réseau**, choisissez le sous-réseau dans lequel vous souhaitez héberger le point de terminaison.  L’adresse IP affectée au point de terminaison provient de la plage affectée à ce sous-réseau.
22. Pour **Intégrer à une zone DNS privée**, indiquez **Non**. Pour plus de simplicité, nous allons utiliser le DNS d’Azure sur sa propre zone DNS privée ou nos propres serveurs DNS.
23. Cliquez sur **OK**.
24. Cliquez sur **Suivant : Paramètres supplémentaires**.
25. Pour le paramètre **Utiliser des données existantes**, choisissez **Exemple**.  Les données de la base de données AdventureWorksLT seront utilisées lors de la création de la base de données.
26. Cliquez sur **Vérifier + créer**.
27. Cliquez sur **Créer**.

Une fois le déploiement de la base de données SQL terminé, nous pouvons créer le laboratoire et installer le logiciel sur la machine modèle du laboratoire.

### <a name="lab-settings"></a>Paramètres du labo

Utilisez les paramètres du tableau ci-dessous lors de la configuration d’un laboratoire de classe. Pour plus d’informations sur la création d’un laboratoire de classe, consultez le [didacticiel Configurer un laboratoire de salle de classe](tutorial-setup-classroom-lab.md).

| Paramètres du labo | Valeur/instructions |
| ------------ | ------------------ |
| Taille de la machine virtuelle | Moyenne. Cette taille est idéale pour les bases de données relationnelles, le caching en mémoire et l’analyse. |
| Image de machine virtuelle | Visual Studio 2019 Community (dernière version) sur Windows 10 Enterprise N (x64) |

Maintenant que notre laboratoire est créé, modifions la machine modèle avec les logiciels dont nous avons besoin.

## <a name="visual-studio"></a>Visual Studio

L’image choisie ci-dessus comprend [Visual Studio 2019 Community](https://visualstudio.microsoft.com/vs/community/).  Toutes les charges de travail et tous les ensembles d’outils sont déjà installés dans l’image.  Utilisez Visual Studio Installer pour [installer des outils facultatifs](/visualstudio/install/modify-visual-studio?view=vs-2019).  [Connectez-vous à Visual Studio](/visualstudio/ide/signing-in-to-visual-studio?view=vs-2019#how-to-sign-in-to-visual-studio) pour déverrouiller l’édition Community.

Visual Studio inclut l’ensemble d’outils **Traitement et stockage des données** qui comprend SQL Server Data Tools (SSDT).  Pour plus d’informations sur les fonctionnalités de SSDT, consultez la [vue d’ensemble de SQL Server Data Tools](/sql/ssdt/sql-server-data-tools?view=sql-server-ver15).  Pour vérifier que la connexion au serveur SQL partagé pour la classe réussira, consultez le [guide pratique pour se connecter à une base de données et parcourir les objets existants](/sql/ssdt/how-to-connect-to-a-database-and-browse-existing-objects?view=sql-server-ver15). Si vous y êtes invité, ajoutez l’adresse IP de la machine modèle à la [liste des ordinateurs autorisés](../azure-sql/database/firewall-configure.md) à se connecter à votre instance SQL Server.

Visual Studio prend en charge plusieurs charges de travail, notamment **Web et cloud** et **Bureau et mobile**.  Ces deux charges de travail prennent en charge SQL Server comme source de données. Pour plus d’informations sur l’utilisation d’ASP.NET Core pour SQL Server, consultez le tutoriel [Créer une application ASP.NET Core et SQL Database dans Azure App Service](../app-service/tutorial-dotnetcore-sqldb-app.md).  Utilisez la bibliothèque [System.Data.SqlClient](/dotnet/api/system.data.sqlclient) pour vous connecter à une base de données SQL à partir d’une application [Xamarin](/xamarin).

## <a name="install-azure-data-studio"></a>Installer Azure Data Studio

[Azure Data Studio](https://github.com/microsoft/azuredatastudio) est un environnement de bureau prenant en charge plusieurs plateformes et bases de données pour les professionnels des données qui utilisent la famille de plateformes de données locales et cloud sur Windows, macOS et Linux.

1. Téléchargez le programme d’installation [*système* Azure Data Studio pour Windows](https://go.microsoft.com/fwlink/?linkid=2127432). Pour rechercher les programmes d’installation pour d’autres systèmes d’exploitation pris en charge, accédez à la page de téléchargement [Azure Data Studio](/sql/azure-data-studio/download).
2. Dans la page **Contrat de licence**, sélectionnez **J’accepte le contrat**. Cliquez sur **Suivant**.
3. Dans la page **Sélectionner l’emplacement de destination**, cliquez sur **Suivant**.
4. Dans la page **Sélectionner le dossier Menu Démarrer**, cliquez sur **Suivant**.
5. Dans la page **Sélectionner les tâches supplémentaires**, cochez **Créer une icône de Bureau** si vous le souhaitez.  Cliquez sur **Suivant**.
6. Dans la page **Prêt pour l’installation**, cliquez sur **Suivant**.
7. Attendez que le programme d’installation s’exécute.  Cliquez sur **Terminer**.

Une fois Azure Data Studio installé, nous allons configurer la connexion à Azure SQL Database.

1. Dans la **page d’accueil** d’Azure Data Studio, cliquez sur le lien **Nouvelle connexion**.
2. Dans la zone **Détails de la connexions**, indiquez les informations nécessaires.
    - Pour **Serveur**, indiquez *classlabdbserver.database.windows.net*.
    - Pour **Utilisateur**, indiquez le nom *azureuser*.
    - Choisissez comme **Mot de passe** celui utilisé pour créer la base de données.
    - Cochez **Mémoriser le mot de passe**.
    - Pour **Base de données**, sélectionnez *classlabdb*.
3. Cliquez sur **Connecter**.

## <a name="install-sql-server-management-studio"></a>Installer SQL Server Management Studio

[SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15) est un environnement intégré pour la gestion des infrastructures SQL.  SSMS est un outil qui permet aux administrateurs de base de données de déployer, de superviser et de mettre à niveau l’infrastructure des données.

1. [Téléchargez SQL Server Management Studio](https://aka.ms/ssmsfullsetup). Une fois le téléchargement terminé, démarrez le programme d’installation.
2. Dans la **page d’accueil**, cliquez sur **Installer**.
3. Dans la page **Installation terminée**, cliquez sur **Fermer**.
4. Exécutez SQL Server Management Studio.  
5. Dans la page **Processus de configuration des dépendances**, cliquez sur **Fermer**.

Notez que si SSMS est installé, vous pouvez [vous connecter à une instance SQL Server et l’interroger](/sql/ssms/tutorials/connect-query-sql-server). Lors de la configuration de la connexion, utilisez les valeurs suivantes :

- Type de serveur : Moteur de base de données
- Nom du serveur : *classlabdbserver.database.windows.net*
- Authentification : l’authentification SQL Server
- Connexion : *azureuser*
- Mot de passe : celui utilisé pour créer la base de données.

## <a name="cost-estimate"></a>Estimation du coût

Nous allons aborder une estimation de coût possible pour cette classe. L’estimation n’inclut pas le coût d’exécution de SQL Server.  Pour plus d’informations, consultez les [prix des bases de données SQL](https://azure.microsoft.com/pricing/details/sql-database).

Nous allons utiliser une classe de 25 élèves. Nous prévoyons 20 heures de temps de classe. En outre, chaque étudiant obtient un quota de 10 heures pour les devoirs ou travaux en dehors des heures de cours planifiées. La taille de machine virtuelle que nous avons choisie était moyenne, soit 42 unités Lab.

Voici un exemple d’estimation de coût possible pour cette classe :

25 élèves \* (20 heures planifiées \+ 10 heures de quota) * 0,42 USD par heure = 315,00 USD

>[!IMPORTANT]
>L’estimation du coût est fournie à titre d’exemple uniquement. Pour en savoir plus sur les tarifs actuels, consultez [Tarification Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).

## <a name="next-steps"></a>Étapes suivantes

Les étapes suivantes sont communes à la configuration de n’importe quel labo.

- [Créer, gérer et publier un modèle](how-to-create-manage-template.md)
- [Ajout d'utilisateurs](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Définir un quota](how-to-configure-student-usage.md#set-quotas-for-users)
- [Définir un calendrier](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab)
- [Envoyer par mail des liens d’inscription aux étudiants](how-to-configure-student-usage.md#send-invitations-to-users)