---
title: 'Tutoriel : Créer une application web Java à l’aide d’Azure Cosmos DB et de l’API SQL'
description: 'Tutoriel : Ce didacticiel d’application web Java vous montre comment utiliser le service Azure Cosmos DB et l’API SQL pour stocker les données et y accéder à partir d’une application Java hébergée sur les sites web Azure.'
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: tutorial
ms.date: 05/12/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: b3cb6bf56820da84d17f0b981f461a545bbe5ab6
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96549257"
---
# <a name="tutorial-build-a-java-web-application-using-azure-cosmos-db-and-the-sql-api"></a>Tutoriel : Créer une application web Java à l’aide d’Azure Cosmos DB et de l’API SQL
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET](sql-api-dotnet-application.md)
> * [Java](sql-api-java-application.md)
> * [Node.JS](sql-api-nodejs-application.md)
> * [Python](./create-sql-api-python.md)
> * [Xamarin](mobile-apps-with-xamarin.md)
> 

Ce didacticiel d’application web Java explique comment utiliser le service [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) pour stocker les données et y accéder à partir d’une application Java hébergée sur Azure App Service Web Apps. Cet article portera sur les éléments suivants :

* Créer une application JavaServer Pages (JSP) de base dans Eclipse.
* Utiliser le service Azure Cosmos DB avec le [kit de développement logiciel (SDK) Java d’Azure Cosmos DB](https://github.com/Azure/azure-documentdb-java).

Ce didacticiel d’application Java vous montre comment créer une application de gestion des tâches basée sur le web qui vous permet de créer, récupérer et marquer des tâches comme terminées, comme illustré dans l’image suivante. Chacune des tâches dans la liste ToDo est stockée sous forme de documents JSON dans Azure Cosmos DB.

:::image type="content" source="./media/sql-api-java-application/image1.png" alt-text="Application Java My ToDo List":::

> [!TIP]
> Ce didacticiel de développement d’applications part du principe que vous avez déjà utilisé Java. Si vous débutez avec Java ou avec les [outils requis](#Prerequisites), nous vous recommandons de télécharger l’ensemble du projet [todo](https://github.com/Azure-Samples/documentdb-java-todo-app) sur GitHub et de le générer à l’aide des [instructions fournies à la fin de cet article](#GetProject). Une fois que vous l'avez créé, vous pouvez consulter l'article pour obtenir des informations sur le code dans le contexte du projet.  
>

## <a name="prerequisites-for-this-java-web-application-tutorial"></a><a id="Prerequisites"></a>Conditions préalables à l’exécution de ce didacticiel d’application web Java

Avant de commencer ce didacticiel de développement d’applications, vous devez disposer des éléments suivants :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer. 

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [Kit de développement logiciel Java (JDK) 7+](/java/azure/jdk/?view=azure-java-stable&preserve-view=true)
* [Environnement de développement intégré (IDE) Eclipse pour développeurs Java EE.](https://www.eclipse.org/downloads/packages/release/luna/sr1/eclipse-ide-java-ee-developers)
* [Un site web Azure avec un environnement d’exécution Java (Tomcat ou Jetty, par exemple) activé.](../app-service/quickstart-java.md)

Si vous installez ces outils pour la première fois, coreservlets.com fournit un guide pas à pas de la procédure d’installation dans la section de démarrage rapide de son [Tutoriel : Installation de TomCat7 et son utilisation avec Eclipse](http://www.coreservlets.com/Apache-Tomcat-Tutorial/tomcat-7-with-eclipse.html).

## <a name="create-an-azure-cosmos-db-account"></a><a id="CreateDB"></a>Créer un compte Azure Cosmos DB

Commençons par créer un compte Azure Cosmos DB. Si vous possédez déjà un compte ou si vous utilisez l’émulateur Azure Cosmos DB pour ce didacticiel, vous pouvez passer à l’[étape 2 : Créer l’application JSP Java](#CreateJSP).

[!INCLUDE [create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

[!INCLUDE [keys](../../includes/cosmos-db-keys.md)]

## <a name="create-the-java-jsp-application"></a><a id="CreateJSP"></a>Créer l’application JSP Java

Pour créer l'application JSP :

1. Tout d’abord, nous allons créer un projet Java. Démarrez Eclipse, puis cliquez sur **File** (Fichier), sur **New** (Nouveau), puis sur **Dynamic Web Project** (Projet web dynamique). Si **Dynamic Web Project** (Projet web dynamique) ne s’affiche pas dans les projets disponibles, procédez comme suit : cliquez sur **File** (Fichier), sur **New** (Nouveau) et sur **Project** (Projet), puis développez **Web**, cliquez sur **Dynamic Web Project** (Projet web dynamique) et cliquez sur **Next** (Suivant).
   
    :::image type="content" source="./media/sql-api-java-application/image10.png" alt-text="Développement d’applications Java JSP":::

1. Entrez un nom de projet dans la zone **Project name** (Nom du projet) et sélectionnez **Target Runtime** (Runtime cible) dans le menu déroulant. Vous pouvez aussi sélectionner une valeur (par exemple, Apache Tomcat v7.0), puis cliquer sur **Finish** (Terminer). La sélection d'un runtime cible vous permet d'exécuter votre projet localement via Eclipse.

1. Dans la vue de l'Explorateur de projets Eclipse, développez votre projet. Cliquez avec le bouton droit sur **WebContent**, cliquez sur **New (Nouveau)** , puis sur **JSP File (Fichier JSP)** .

1. Dans la boîte de dialogue **New JSP File** (Nouveau fichier JSP), nommez le fichier **index.jsp**. Gardez le nom du dossier parent **WebContent**, comme l’indique l’illustration ci-dessous, puis cliquez sur **Next** (Suivant).
   
    :::image type="content" source="./media/sql-api-java-application/image11.png" alt-text="Créer un fichier JSP - Tutoriel d’application web Java":::

1. Pour les besoins de ce didacticiel, dans la boîte de dialogue **Select JSP Template** (Sélectionner le modèle JSP), sélectionnez **New JSP File (HTML)** (Nouveau fichier JSP (HTML)) et cliquez sur **Finish** (Terminer).

1. Quand le fichier *index.jsp* s’ouvre dans Eclipse, ajoutez du texte pour afficher **Hello World!** . dans l’élément `<body>` existant. Le contenu `<body>` mis à jour doit se présenter de la façon suivante :

   ```html
   <body>
     <% out.println("Hello World!"); %>
   </body>
   ```

1. Enregistrez le fichier *index.jsp*.

1. Si vous avez défini un runtime cible à l’étape 2, vous pouvez cliquer sur **Project** (Projet), puis sur **Run** (Exécuter) pour exécuter votre application JSP localement :

   :::image type="content" source="./media/sql-api-java-application/image12.png" alt-text="Hello World – Tutoriel sur les applications Java":::

## <a name="install-the-sql-java-sdk"></a><a id="InstallSDK"></a>Installer le kit SDK Java SQL

Le moyen le plus simple d'extraire le Kit SDK Java SQL et ses dépendances est d'utiliser [Apache Maven](https://maven.apache.org/). Pour ce faire, vous devez convertir votre projet en projet Maven en effectuant les étapes suivantes :

1. Cliquez avec le bouton droit sur votre projet dans l’Explorateur de projets, cliquez sur **Configure** (Configurer) et sur **Convert to Maven Project** (Convertir en projet Maven).

1. Dans la fenêtre **Create new POM** (Créer un nouveau POM), acceptez les valeurs par défaut et cliquez sur **Finish** (Terminer).

1. Dans l' **Explorateur de projets**, ouvrez le fichier pom.xml.

1. Sous l’onglet **Dependencies** (Dépendances) du volet **Dependencies** (Dépendances), cliquez sur **Add** (Ajouter).

1. Dans la fenêtre **Select Dependency** (Sélectionner une dépendance), procédez comme suit :
   
   * Dans la zone **Group Id**, entrez `com.azure`.
   * Dans la zone **Artifact Id** (ID d’artefact), entrez `azure-cosmos`.
   * Dans la zone **Version**, entrez `4.0.1-beta.1`.
  
   Sinon, vous pouvez ajouter le contenu XML de dépendance pour Group ID et Artifact ID directement dans le fichier *pom.xml* :

   ```xml
   <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-cosmos</artifactId>
      <version>4.0.1-beta.1</version>
   </dependency>
   ```

1. Cliquez sur **OK**. Maven installe le kit SDK Java SQL ou enregistre le fichier pom.xml.

## <a name="use-the-azure-cosmos-db-service-in-your-java-application"></a><a id="UseService"></a>Utiliser le service Azure Cosmos DB dans votre application Java

Ajoutons à présent les modèles, les vues et les contrôleurs à votre application web.

### <a name="add-a-model"></a>Ajout d'un modèle

Tout d’abord, définissons un modèle dans un nouveau fichier *TodoItem.java*. La classe `TodoItem` définit le schéma d’un élément avec les méthodes getter et setter :

:::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/documentdb/sample/model/TodoItem.java":::

### <a name="add-the-data-access-objectdao-classes"></a>Ajouter les classes d’objets d’accès aux données (DAO)

Créez un objet d’accès aux données (DAO) pour isoler la persistance des éléments ToDo dans Azure Cosmos DB. Pour enregistrer les éléments ToDo dans une collection, le client doit connaître la base de données et la collection de stockage (référencées par les liens réflexifs). En général, il est préférable de mettre en cache la collection et la base de données lorsque cela est possible afin d'éviter les allers-retours supplémentaires vers la base de données.

1. Pour appeler le service Azure Cosmos DB, vous devez instancier un nouvel objet `cosmosClient`. En général, il est préférable de réutiliser l’objet `cosmosClient` plutôt que de construire un nouveau client à chaque demande suivante. Vous pouvez réutiliser le client en le définissant dans la classe `cosmosClientFactory`. Mettez à jour les valeurs HOST et MASTER_KEY que vous avez enregistrées au cours de [étape 1](#CreateDB). Remplacez la variable HOST par votre URI, et celle de MASTER_KEY par votre CLÉ PRIMAIRE. Utilisez le code suivant pour créer la classe `CosmosClientFactory` dans le fichier *CosmosClientFactory.java* :

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/documentdb/sample/dao/CosmosClientFactory.java":::

1. Créez un fichier *TodoDao.java* et ajoutez la classe `TodoDao` pour créer, mettre à jour, lire et supprimer les éléments de tâche :

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/documentdb/sample/dao/TodoDao.java":::

1. Créez un fichier *MockDao.java* et ajoutez la classe `MockDao` ; cette classe implémente la classe `TodoDao` pour effectuer des opérations CRUD sur les éléments :

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/documentdb/sample/dao/MockDao.java":::

1. Créez un fichier *DocDbDao.java* et ajoutez la classe `DocDbDao`. Cette classe définit le code permettant de rendre persistant les TodoItems dans le conteneur ; elle récupère votre base de données et la collection, si elle existe, ou en crée une si elle n’existe pas. Cet exemple utilise [Gson](https://code.google.com/p/google-gson/) pour sérialiser et désérialiser les objets POJO (Plain Old Java Object) TodoItem dans des documents JSON. Pour enregistrer les éléments ToDo dans une collection, le client doit connaître la base de données et la collection de stockage (référencées par les liens réflexifs). Cette classe définit également la fonction d’assistance pour récupérer les documents par un autre attribut (par exemple, « ID ») au lieu d’établir un lien réflexif. Vous pouvez utiliser la méthode d’assistance pour récupérer un document JSON TodoItem par son ID, et le désérialiser ensuite en POJO.

   Vous pouvez également utiliser l’objet client `cosmosClient` pour obtenir une collection ou une liste de TodoItems à l’aide d’une requête SQL. Enfin, vous définissez la méthode de suppression pour supprimer un TodoItem de votre liste. Le code suivant montre le contenu de la classe `DocDbDao` :

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/documentdb/sample/dao/DocDbDao.java":::

1. Créez ensuite un fichier *TodoDaoFactory.java* et ajoutez la classe `TodoDaoFactory` qui crée un nouvel objet DocDbDao :

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/documentdb/sample/dao/TodoDaoFactory.java":::

### <a name="add-a-controller"></a>Ajout d'un contrôleur

Ajoutez le contrôleur *TodoItemController* à votre application. Dans ce projet, vous utilisez [Project Lombok](https://projectlombok.org/) pour générer le constructeur, les accesseurs get, les méthodes setter et un générateur. Vous pouvez également écrire ce code manuellement ou faire en sorte que l’IDE le génère :

:::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/documentdb/sample/controller/TodoItemController.java":::

### <a name="create-a-servlet"></a>Créer un servlet

Ensuite, créez un servlet pour router les requêtes HTTP vers le contrôleur. Créez le fichier *ApiServlet.java* et définissez dessous le code suivant :

:::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/documentdb/sample/ApiServlet.java":::

## <a name="wire-the-rest-of-the-of-java-app-together"></a><a id="Wire"></a>Relier le reste de l’application Java

Après l’agréable, passons à l’utile : la création d’une interface utilisateur rapide à associer à votre DAO.

1. Vous avez besoin d’une interface utilisateur web à afficher à l’utilisateur. Réécrivons le fichier *index.jsp* créé précédemment avec le code suivant :

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/WebContent/index.jsp":::

1. Terminons en écrivant le code JavaScript côté client pour lier l’interface utilisateur web et le servlet :

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/WebContent/assets/todo.js":::

1. Maintenant, il ne nous reste qu'à tester l'application. Exécutez l'application localement et ajoutez des éléments Todo en renseignant le nom et la catégorie de l'élément, puis en cliquant sur **Add Task**(Ajouter une tâche). Après l’apparition de l’élément, vous pouvez le mettre à jour s’il est terminé en activant la case à cocher et en cliquant sur **Mettre à jour les tâches**.

## <a name="deploy-your-java-application-to-azure-web-sites"></a><a id="Deploy"></a>Déployer votre application Java sur des sites web Azure

Les Sites Web Azure permettent de déployer facilement des applications Java en les exportant sous forme de fichiers WAR et en les chargeant par le biais du contrôle de code source (GIT, par exemple) ou par FTP.

1. Pour exporter votre application en tant que fichier WAR, cliquez avec le bouton droit sur votre projet dans **l’Explorateur de projets**, cliquez sur **Export** (Exporter), puis sur **WAR File** (Fichier WAR).

1. Dans la fenêtre **WAR Export** (Exportation WAR), procédez comme suit :
   
   * Dans la boîte de dialogue de projet web, entrez azure-documentdb-java-sample.
   * Dans la boîte de dialogue Destination, choisissez un emplacement d'enregistrement du fichier WAR.
   * Cliquez sur **Terminer**.

1. Maintenant que vous disposez d’un fichier WAR, vous pouvez le charger tout simplement dans votre répertoire **webapps** sur les Sites Web Azure. Vous trouverez des instructions sur le chargement du fichier sur la page [Ajouter une application Java à Azure App Service Web Apps](../app-service/quickstart-java.md). Après le téléchargement du fichier WAR dans le répertoire webapps, l’environnement d’exécution détecte que vous l’avez ajouté et le télécharge automatiquement.

1. Pour consulter le produit terminé, accédez à `http://YOUR\_SITE\_NAME.azurewebsites.net/azure-java-sample/` et ajoutez vos tâches !

## <a name="get-the-project-from-github"></a><a id="GetProject"></a>Obtenir le projet à partir de GitHub

Tous les exemples de ce didacticiel sont inclus dans le projet [todo](https://github.com/Azure-Samples/documentdb-java-todo-app) sur GitHub. Pour importer le projet todo dans Eclipse, vérifiez d'abord que vous disposez des logiciels et ressources indiqués dans la section [Configuration requise](#Prerequisites) , puis procédez comme suit :

1. Installez [Project Lombok](https://projectlombok.org/). Lombok est utilisé pour générer des constructeurs, des méthodes getter et des méthodes setter dans le projet. Une fois que vous avez téléchargé le fichier lombok.jar, double-cliquez dessus pour l'installer ou installez-le à partir de la ligne de commande.

1. Si l'application Eclipse est ouverte, fermez-la et redémarrez-la pour charger Lombok.

1. Dans Eclipse, dans le menu **File** (Fichier), cliquez sur **Import** (Importer).

1. Dans la fenêtre **Import** (Importer), cliquez sur **Git**, sur **Projects from Git** (Projets provenant de Git), puis sur **Next** (Suivant).

1. Dans l’écran **Select Repository Source** (Sélectionner une source de référentiel), cliquez sur **Clone URI** (URI du clone).

1. Dans l’écran **Dépôt Git source**, dans la zone **URI**, entrez https://github.com/Azure-Samples/documentdb-java-todo-app.git, puis cliquez sur **Suivant**.

1. Dans l’écran **Branch Selection** (Sélection d’une branche), vérifiez que **main** (principal) est sélectionné, puis cliquez sur **Next** (Suivant).

1. Dans l’écran **Local Destination** (Destination locale), cliquez sur **Browse** (Parcourir) pour sélectionner un dossier dans lequel copier le référentiel, puis cliquez sur **Next** (Suivant).

1. Dans l’écran **Select a wizard to use for importing projects** (Sélectionner un Assistant à utiliser pour l’importation de projets), vérifiez que **Import existing projects** (Importer des projets existants) est sélectionné, puis cliquez sur **Next** (Suivant).

1. Dans l’écran **Import Projects** (Importer des projets), désélectionnez le projet **DocumentDB**, puis cliquez sur **Finish** (Terminer). Le projet DocumentDB contient le Kit de développement logiciel (SDK) Java Azure Cosmos DB, que nous allons ajouter comme dépendance à la place.

1. Dans **l’Explorateur de projets**, accédez à azure-documentdb-java-sample\src\com.microsoft.azure.documentdb.sample.dao\DocumentClientFactory.java et remplacez les valeurs HOST et MASTER_KEY par l’URI et la CLÉ PRIMAIRE de votre compte Azure Cosmos DB, puis enregistrez le fichier. Pour plus d'informations, consultez l'[Étape 1. Créez un compte de base de données Azure Cosmos](#CreateDB).

1. Dans l’**Explorateur de projets**, cliquez avec le bouton droit sur **azure-documentdb-java-sample**, cliquez sur **Build Path** (Chemin de build), puis sur **Configure Build Path** (Configurer le chemin de build).

1. Dans l’écran **Java Build Path** (Chemin de build Java), dans le volet droit, sélectionnez l’onglet **Libraries** (Bibliothèques), puis cliquez sur **Add External JARs** (Ajouter des JAR externes). Accédez à l’emplacement du fichier lombok.jar et cliquez sur **Open** (Ouvrir), puis sur **OK**.

1. Utilisez l’étape 12 pour rouvrir la fenêtre **Properties** (Propriétés), puis, dans le volet de gauche, cliquez sur **Targeted Runtimes** (Runtime ciblés).

1. Dans l’écran **Targeted Runtimes** (Runtime ciblés), cliquez sur **New** (Nouveau), sélectionnez **Apache Tomcat v7.0**, puis cliquez sur **OK**.

1. Suivez l’étape 12 pour rouvrir la fenêtre **Properties** (Propriétés) et, dans le volet de gauche, cliquez sur **Project Facets** (Facettes du projet).

1. Dans l’écran **Project Facets** (Facettes du projet), sélectionnez **Dynamic Web Module** (Module web dynamique) et **Java**, puis cliquez sur **OK**.

1. Sous l’onglet **Servers** (Serveurs) en bas de l’écran, cliquez sur **Tomcat v7.0 Server at localhost** (Serveur Tomcat v7.0 dans localhost), puis cliquez sur **Add and Remove** (Ajouter et supprimer).

1. Dans la fenêtre **Add and Remove** (Ajouter et supprimer), déplacez **azure-documentdb-java-sample** vers la zone **Configured** (Configuré), puis cliquez sur **Finish** (Terminer).

1. Sous l’onglet **Servers** (Serveurs), cliquez avec le bouton droit sur **Tomcat v7.0 Server at localhost** (Serveur Tomcat v7.0 dans localhost), puis cliquez sur **Restart** (Redémarrer).

1. Dans un navigateur, accédez à `http://localhost:8080/azure-documentdb-java-sample/`, puis commencez à ajouter à votre liste de tâches. Notez que si vous avez modifié les valeurs du port par défaut, définissez 8080 sur la valeur que vous avez sélectionnée.

1. Pour déployer votre projet sur un site web Azure, consultez l'[Étape 6. Déployez votre application sur les Sites Web Azure](#Deploy).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Générer une application node.js avec Azure Cosmos DB](sql-api-nodejs-application.md)