---
title: Se connecter à SQL Database à l’aide de C et C++
description: Utilisez l’exemple de code de ce guide de démarrage rapide pour créer une application moderne utilisant C++ et reposant sur une base de données relationnelle puissante dans le cloud avec Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=1
ms.devlang: cpp
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 12/12/2018
ms.openlocfilehash: e891c5797c9ce93e6cab7a07d2f68de1a9157249
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92674765"
---
# <a name="connect-to-sql-database-using-c-and-c"></a>Se connecter à SQL Database à l’aide de C et C++
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Cette publication est destinée aux développeurs C et C++ qui essaient de se connecter à Azure SQL Database. Elle est divisée en sections afin de vous permettre d’accéder directement à celle qui vous intéresse.

## <a name="prerequisites-for-the-cc-tutorial"></a>Configuration requise pour le didacticiel C/C++

Vérifiez que vous disposez des éléments suivants :

* Un compte Azure actif. Si vous n’en avez pas, vous pouvez vous inscrire pour bénéficier d’un [essai gratuit des services Azure](https://azure.microsoft.com/pricing/free-trial/)dès aujourd’hui.
* [Visual Studio](https://www.visualstudio.com/downloads/). Vous devez installer les composants du langage C++ pour générer et exécuter cet exemple.
* [Développement Visual Studio Linux](/cpp/linux/?view=vs-2019). Si vous développez sur Linux, vous devez également installer l’extension Visual Studio Linux.

## <a name="azure-sql-database-and-sql-server-on-virtual-machines"></a><a id="AzureSQL"></a>Azure SQL Database et SQL Server sur les machines virtuelles

Reposant sur Microsoft SQL Server, Azure SQL Database est conçu pour fournir un service à haute disponibilité, performant et évolutif. L’utilisation d’Azure SQL en lieu et place de votre base de données propriétaire s’exécutant en local présente de nombreux avantages. Avec Azure SQL, vous ne devez pas installer, configurer, mettre à jour ou gérer votre base de données, mais uniquement son contenu et sa structure. Les capacités de base de données importantes, telles que la tolérance de panne et la redondance, sont intégrées.

Azure propose actuellement deux options d’hébergement pour les charges de travail SQL Server : Azure SQL Database, base de données en tant que service, et SQL Server sur machines virtuelles. Nous ne détaillerons pas les différences entre ces deux options, mais sachez que si vous développez de nouvelles applications cloud, Azure SQL Database vous permettra de tirer parti des économies et de l’optimisation des performances assurées par les services cloud. Si vous envisagez de migrer ou d’étendre vos applications locales vers le cloud, SQL Server sur une machine virtuelle Azure vous offrira sans doute de meilleurs résultats. Pour simplifier les choses dans le cadre de cet article, nous allons créer une instance Azure SQL Database.

## <a name="data-access-technologies-odbc-and-ole-db"></a><a id="ODBC"></a>Technologies d’accès aux données : ODBC et OLE DB

Se connecter à Azure SQL Database n’est en rien différent et il existe actuellement deux façons de se connecter à des bases de données : ODBC (Open Database Connectivity) et OLE DB (Object Linking and Embedding Database). Ces dernières années, Microsoft s’est aligné sur [ODBC pour l’accès aux données relationnelles natives](/archive/blogs/sqlnativeclient/microsoft-is-aligning-with-odbc-for-native-relational-data-access). ODBC est relativement simple, mais aussi beaucoup plus rapide qu’OLE DB. Le seul inconvénient ici est qu’ODBC utilise une API de style C ancienne.

## <a name="step-1--creating-your-azure-sql-database"></a><a id="Create"></a>Étape 1 :  Création de votre base de données Azure SQL

Consultez la [page de prise en main](single-database-create-quickstart.md) pour apprendre à créer un exemple de base de données.  Sinon, vous pouvez suivre cette [vidéo de deux minutes](https://azure.microsoft.com/documentation/videos/azure-sql-database-create-dbs-in-seconds/) pour créer une instance Azure SQL Database à l’aide du portail Azure.

## <a name="step-2--get-connection-string"></a><a id="ConnectionString"></a>Étape 2 :  Obtention de la chaîne de connexion

Une fois votre instance Azure SQL Database approvisionnée, vous devez exécuter les étapes suivantes pour déterminer les informations de connexion et ajouter l’adresse IP de votre client pour l’accès au pare-feu.

Dans le [portail Azure](https://portal.azure.com/), accédez à la chaîne de connexion ODBC de votre instance Azure SQL Database en cliquant sur **Afficher les chaînes de connexion de la base de données** dans la vue d’ensemble de votre base de données :

![Chaîne de connexion ODBC](./media/develop-cplusplus-simple/azureportal.png)

![Propriétés de la chaîne de connexion ODBC](./media/develop-cplusplus-simple/dbconnection.png)

Copiez le contenu de la chaîne **ODBC (Inclut Node.js) [Authentification SQL]** . Nous utiliserons cette chaîne ultérieurement pour la connexion à partir de l’interpréteur de ligne de commande ODBC C++. Cette chaîne fournit des détails tels que le pilote, le serveur et d’autres paramètres de connexion de base de données.

## <a name="step-3--add-your-ip-to-the-firewall"></a><a id="Firewall"></a>Étape 3 :  Ajout de votre adresse IP au pare-feu

Accédez à la section du pare-feu de votre serveur et ajoutez [l’adresse IP de votre client au pare-feu à l’aide de ces étapes](firewall-configure.md) pour permettre l’établissement d’une connexion :

![Fenêtre d’ajout de l’adresse IP](./media/develop-cplusplus-simple/ip.png)

À ce stade, vous avez configuré votre instance Azure SQL Database et êtes prêt à vous connecter à partir de votre code C++.

## <a name="step-4-connecting-from-a-windows-cc-application"></a><a id="Windows"></a>Étape 4 : Connexion à partir d’une application C/C++ Windows

Vous pouvez facilement vous connecter à votre [instance Azure SQL Database à l’aide d’ODBC sur Windows en utilisant cet exemple](https://github.com/Microsoft/VCSamples/tree/master/VC2015Samples/ODBC%20database%20sample%20%28windows%29), généré avec Visual Studio. L’exemple implémente un interpréteur de ligne de commande ODBC qui peut être utilisé pour se connecter à l’instance Azure SQL Database. Cet exemple prend comme argument de ligne de commande un fichier de nom de source de base de données (DSN) ou la chaîne de connexion détaillée copiée précédemment à partir du portail Azure. Affichez la page de propriétés de ce projet et collez la chaîne de connexion en tant qu’argument de commande, comme illustré ici :

![Fichier de propriétés DSN](./media/develop-cplusplus-simple/props.png)

Prenez soin d’inclure les informations d’authentification correctes pour votre base de données dans la chaîne de connexion de cette base de données.

Lancez l’application pour la générer. La fenêtre de confirmation de la connexion suivante doit s’afficher. Vous pouvez même exécuter des commandes SQL de base telles que **create table** pour valider la connectivité de votre base de données :

![Commandes SQL](./media/develop-cplusplus-simple/sqlcommands.png)

Vous pouvez aussi créer un fichier DSN à l’aide de l’assistant qui est lancé lorsqu’aucun argument de commande n’est fourni. Nous vous recommandons d’essayer également cette option. Vous pouvez utiliser ce fichier DSN à des fins d’automatisation et pour protéger vos paramètres d’authentification :

![Création d’un fichier DSN](./media/develop-cplusplus-simple/datasource.png)

Félicitations ! Vous avez réussi à vous connecter à Azure SQL à l’aide de C++ et ODBC sur Windows. Vous pouvez poursuivre la lecture pour faire de même sur une plateforme Linux.

## <a name="step-5-connecting-from-a-linux-cc-application"></a><a id="Linux"></a>Étape 5 : Connexion à partir d’une application C/C++ Linux

Si vous ne le saviez pas déjà, Visual Studio permet maintenant de développer également des applications C++ Linux. Vous trouverez plus d’informations sur ce nouveau scénario sur le blog consacré à [Visual C++ pour le développement Linux](https://blogs.msdn.microsoft.com/vcblog/20../../visual-c-for-linux-development/). Pour générer des applications pour Linux, vous avez besoin d’un ordinateur distant sur lequel s’exécute votre distribution Linux. Si vous n’en avez pas, vous pouvez en configurer un rapidement à l’aide de [machines virtuelles Azure Linux](../../virtual-machines/linux/quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

Pour ce didacticiel, nous partirons du principe que vous disposez d’une distribution Linux Ubuntu 16.04. Les étapes ci-dessous s’appliquent également à Ubuntu 15.10, Red Hat 6 et Red Hat 7.

Les étapes suivantes permettent d’installer les bibliothèques nécessaires à SQL et ODBC pour votre distribution :

```console
    sudo su
    sh -c 'echo "deb [arch=amd64] https://apt-mo.trafficmanager.net/repos/mssql-ubuntu-test/ xenial main" > /etc/apt/sources.list.d/mssqlpreview.list'
    sudo apt-key adv --keyserver apt-mo.trafficmanager.net --recv-keys 417A0893
    apt-get update
    apt-get install msodbcsql
    apt-get install unixodbc-dev-utf16 #this step is optional but recommended*
```

Lancez Visual Studio. Sous Outils -> Options -> Multiplateforme -> Gestionnaire de connexion, ajoutez une connexion à votre boîte Linux :

![Outils > Options](./media/develop-cplusplus-simple/tools.png)

Une fois la connexion via SSH établie, créez un modèle de projet vide (Linux) :

![Nouveau modèle de projet](./media/develop-cplusplus-simple/template.png)

Vous pouvez ensuite ajouter un [nouveau fichier source C et le remplacer par ce contenu](https://github.com/Microsoft/VCSamples/blob/master/VC2015Samples/ODBC%20database%20sample%20%28linux%29/odbcconnector/odbcconnector.c). À l’aide des API ODBC SQLAllocHandle, SQLSetConnectAttr et SQLDriverConnect, vous devriez pouvoir initialiser et établir une connexion à votre base de données.
Comme pour l’exemple ODBC Windows, vous devez remplacer l’appel de SQLDriverConnect avec les détails des paramètres de la chaîne de connexion à votre base de données copiés à partir du portail Azure précédemment.

```c
     retcode = SQLDriverConnect(
        hdbc, NULL, "Driver=ODBC Driver 13 for SQL"
                    "Server;Server=<yourserver>;Uid=<yourusername>;Pwd=<"
                    "yourpassword>;database=<yourdatabase>",
        SQL_NTS, outstr, sizeof(outstr), &outstrlen, SQL_DRIVER_NOPROMPT);
```

La dernière chose à faire avant de procéder à la compilation consiste à ajouter **odbc** en tant que dépendance de bibliothèque :

![Ajout d’ODBC en tant que bibliothèque d’entrée](./media/develop-cplusplus-simple/lib.png)

Pour lancer votre application, affichez la Console Linux à partir du menu **Déboguer**  :

![Console Linux](./media/develop-cplusplus-simple/linuxconsole.png)

Si votre connexion a réussi, le nom de la base de données actuelle doit être indiqué dans la Console Linux :

![Sortie de la fenêtre de la Console Linux](./media/develop-cplusplus-simple/linuxconsolewindow.png)

Félicitations ! Vous avez terminé le didacticiel et pouvez maintenant vous connecter à votre instance Azure SQL Database à partir de C++ sur les plateformes Windows et Linux.

## <a name="get-the-complete-cc-tutorial-solution"></a><a id="GetSolution"></a>Obtenir la solution complète du didacticiel C/C++

Vous trouverez la solution GetStarted qui contient tous les exemples de cet article sur GitHub :

* [Exemple ODBC C++ Windows](https://github.com/Microsoft/VCSamples/tree/master/VC2015Samples/ODBC%20database%20sample%20%28windows%29) : téléchargez l’exemple ODBC C++ Windows pour la connexion à Azure SQL
* [Exemple ODBC C++ Linux](https://github.com/Microsoft/VCSamples/tree/master/VC2015Samples/ODBC%20database%20sample%20%28linux%29) : téléchargez l’exemple ODBC C++ Linux pour la connexion à Azure SQL

## <a name="next-steps"></a>Étapes suivantes

* Consultez la [Vue d’ensemble du développement de base de données SQL](develop-overview.md)
* Pour plus d’informations sur les API ODBC, consultez ces [informations de référence](/sql/odbc/reference/syntax/odbc-api-reference/)

## <a name="additional-resources"></a>Ressources supplémentaires

* [Modèles de conception pour les applications SaaS multilocataires avec Azure SQL Database](saas-tenancy-app-design-patterns.md)
* Explorez toutes les [fonctionnalités de la base de données SQL](https://azure.microsoft.com/services/sql-database/)