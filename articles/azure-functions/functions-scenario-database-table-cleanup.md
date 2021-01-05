---
title: Utiliser Azure Functions pour effectuer une tâche de nettoyage de base de données
description: Utilisez Azure Functions pour planifier une tâche qui se connecte à Azure SQL Database pour nettoyer des lignes régulièrement.
ms.assetid: 076f5f95-f8d2-42c7-b7fd-6798856ba0bb
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 10/02/2019
ms.openlocfilehash: 0b5e255d7d108eb063ece4e5489a8762261a0bed
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88207253"
---
# <a name="use-azure-functions-to-connect-to-an-azure-sql-database"></a>Utiliser Azure Functions pour se connecter à une base de données Azure SQL Database

Cet article vous montre comment utiliser Azure Functions pour créer un travail planifié qui se connecte à une base de données ou une instance managée Azure SQL. Le code de fonction nettoie les lignes d’une table de la base de données. La fonction C# est créée selon un modèle de déclencheur de minuteur prédéfini dans Visual Studio 2019. Pour prendre en charge ce scénario, vous devez également définir une chaîne de connexion de base de données comme paramètre d’application dans l’application de fonction. Pour une instance managée Azure SQL, vous devez [activer le point de terminaison public](../azure-sql/managed-instance/public-endpoint-configure.md) pour pouvoir vous connecter à partir d’Azure Functions. Ce scénario utilise une opération en bloc sur la base de données. 

Si c’est la première fois que vous utilisez Azure Functions pour C#, vous devez consulter les [Informations de référence pour les développeurs C# sur Azure Functions](functions-dotnet-class-library.md).

## <a name="prerequisites"></a>Prérequis

+ Suivez les étapes de l’article [Créer votre première fonction à l’aide de Visual Studio](functions-create-your-first-function-visual-studio.md) pour créer une application de fonction locale qui cible la version 2.x ou une version ultérieure du runtime. Vous devez également avoir publié votre projet sur une application de fonction dans Azure.

+ Cette article montre une commande Transact-SQL qui exécute une opération de nettoyage en bloc dans la table nommée **SalesOrderHeader** de l’exemple de base de données AdventureWorksLT. Pour créer l’exemple de base de données AdventureWorksLT, effectuez les étapes de l’article [Créer une base de données dans Azure SQL Database à l’aide du portail Azure](../azure-sql/database/single-database-create-quickstart.md).

+ Vous devez ajouter une [règle de pare-feu au niveau du serveur](../azure-sql/database/firewall-create-server-level-portal-quickstart.md) pour l’adresse IP publique de l’ordinateur que vous utilisez pour ce guide de démarrage rapide. Cette règle vous permet d’accéder à l’instance SQL Database à partir de votre ordinateur local.  

## <a name="get-connection-information"></a>Obtenir des informations de connexion

Vous devez obtenir la chaîne de connexion pour la base de données que vous avez créée quand vous avez effectué les étapes décrites dans [Créer une base de données dans Azure SQL Database à l’aide du portail Azure](../azure-sql/database/single-database-create-quickstart.md).

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Sélectionnez **Bases de données SQL** dans le menu de gauche, puis sélectionnez votre base de données dans la page **Bases de données SQL**.

1. Sélectionnez **Chaînes de connexion** sous **Paramètres** et copiez la chaîne de connexion **ADO.NET** complète. Pour une instance managée Azure SQL, copiez la chaîne de connexion pour le point de terminaison public.

    ![Copiez la chaîne de connexion ADO.NET.](./media/functions-scenario-database-table-cleanup/adonet-connection-string.png)

## <a name="set-the-connection-string"></a>Définir la chaîne de connexion

Une Function App héberge l’exécution de vos fonctions dans Azure. En guise de bonne pratique, stockez les chaînes de connexion et autres secrets dans vos paramètres de conteneur de fonctions. L’utilisation de paramètres d’application empêche la divulgation accidentelle de la chaîne de connexion avec votre code. Vous pouvez accéder aux paramètres de votre application de fonction directement à partir de Visual Studio.

Vous devez avoir publié votre application sur Azure. Si ce n’est déjà fait, [publiez votre application de fonction sur Azure](functions-develop-vs.md#publish-to-azure).

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet d’application de fonction et choisissez **Publier** > **Modifier les paramètres d’Azure App Service**. Sélectionnez **Ajouter un paramètre**, dans **Nom du nouveau paramètre d’application**, tapez `sqldb_connection`, puis sélectionnez **OK**.

    ![Paramètres de l’application pour l’application de fonction.](./media/functions-scenario-database-table-cleanup/functions-app-service-add-setting.png)

1. Dans le nouveau paramètre **sqldb_connection**, collez la chaîne de connexion que vous avez copiée dans la section précédente dans le champ **Local** et remplacez les espaces réservés `{your_username}` et `{your_password}` par des valeurs réelles. Sélectionnez **Insérer une valeur locale** pour copier la valeur mise à jour dans le champ **À distance**, puis sélectionnez **OK**.

    ![Ajoutez un paramètre de chaîne de connexion SQL.](./media/functions-scenario-database-table-cleanup/functions-app-service-settings-connection-string.png)

    Les chaînes de connexion sont stockées dans Azure (**À distance**). Pour empêcher la fuite des secrets, vous devez exclure le fichier projet local.settings.json (**Local**) du contrôle de code source, par exemple à l’aide d’un fichier .gitignore.

## <a name="add-the-sqlclient-package-to-the-project"></a>Ajouter le package SqlClient au projet

Vous devez ajouter le package NuGet qui contient la bibliothèque SqlClient. Cette bibliothèque d’accès aux données est nécessaire pour la connexion à SQL Database.

1. Ouvrez votre projet d’application de fonction locale dans Visual Studio 2019.

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet d’application de fonction, puis choisissez **Gérer les packages NuGet**.

1. Dans l’onglet **Parcourir**, recherchez l’élément ```System.Data.SqlClient``` et, une fois que vous l’avez trouvé, sélectionnez-le.

1. Dans la page **System.Data.SqlClient**, sélectionnez la version `4.5.1` et cliquez sur **Installer**.

1. Une fois l’installation terminée, passez en revue les modifications, puis cliquez sur **OK** pour fermer la fenêtre **Aperçu**.

1. Si une fenêtre **Acceptation de la licence**, cliquez sur **J’accepte**.

À présent, vous pouvez ajouter le code de fonction C# qui se connecte à votre base de données SQL.

## <a name="add-a-timer-triggered-function"></a>Ajouter une fonction de minuteur déclencheur

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet d’application de fonction, puis choisissez **Ajouter** > **Nouvelle fonction Azure**.

1. Le modèle **Azure Functions** étant sélectionné, attribuez au nouvel élément un nom comme `DatabaseCleanup.cs` et sélectionnez **Ajouter**.

1. Dans la boîte de dialogue **Nouvelle fonction Azure**, sélectionnez **Déclencheur de minuteur**, puis **OK**. Cette boîte de dialogue crée un fichier de code pour la fonction déclenchée par minuteur.

1. Ouvrez le nouveau fichier de code et ajoutez les instructions using suivantes au début du fichier :

    ```cs
    using System.Data.SqlClient;
    using System.Threading.Tasks;
    ```

1. Remplacez la fonction `Run` existante par le code suivant :

    ```cs
    [FunctionName("DatabaseCleanup")]
    public static async Task Run([TimerTrigger("*/15 * * * * *")]TimerInfo myTimer, ILogger log)
    {
        // Get the connection string from app settings and use it to create a connection.
        var str = Environment.GetEnvironmentVariable("sqldb_connection");
        using (SqlConnection conn = new SqlConnection(str))
        {
            conn.Open();
            var text = "UPDATE SalesLT.SalesOrderHeader " +
                    "SET [Status] = 5  WHERE ShipDate < GetDate();";

            using (SqlCommand cmd = new SqlCommand(text, conn))
            {
                // Execute the command and log the # rows affected.
                var rows = await cmd.ExecuteNonQueryAsync();
                log.LogInformation($"{rows} rows were updated");
            }
        }
    }
    ```

    Cette fonction s’exécute toutes les 15 secondes pour mettre à jour la colonne `Status` en fonction de la date d’expédition. Pour en savoir plus sur le déclencheur de minuteur, consultez [Déclencheur de minuteur pour Azure Functions](functions-bindings-timer.md).

1. Appuyez sur **F5** pour démarrer l’application de fonction. La fenêtre d’exécution [Azure Functions Core Tools](functions-develop-local.md) s’ouvre en arrière-plan de Visual Studio.

1. 15 secondes après le démarrage, la fonction s’exécute. Regardez la sortie afin de noter le nombre de lignes mises à jour dans la table **SalesOrderHeader**.

    ![Consultez les journaux d’activité des fonctions.](./media/functions-scenario-database-table-cleanup/function-execution-results-log.png)

    À la première exécution, vous devez mettre à jour 32 lignes de données. Les exécutions suivantes ne mettent à jour aucune ligne de données, sauf si vous apportez aux données de la table SalesOrderHeader des modifications qui entraînent la sélection de lignes supplémentaires par l’instruction `UPDATE`.

Si vous envisagez de [publier cette fonction](functions-develop-vs.md#publish-to-azure), pensez à délaisser l’attribut `TimerTrigger` au profit d’une [planification cron](functions-bindings-timer.md#ncrontab-expressions) plus raisonnable qu’une exécution toutes les 15 secondes.

## <a name="next-steps"></a>Étapes suivantes

À présent, apprenez à utiliser des fonctions avec Logic Apps en vue de les intégrer à d’autres services.

> [!div class="nextstepaction"]
> [Créer une fonction qui s’intègre avec Logic Apps](functions-twitter-email.md)

Pour plus d’informations sur Functions, consultez les articles suivants :

+ [Informations de référence pour les développeurs sur Azure Functions](functions-reference.md)  
  Référence du programmeur pour le codage de fonctions et la définition de déclencheurs et de liaisons.
+ [Test d’Azure Functions](functions-test-a-function.md)  
  décrit plusieurs outils et techniques permettant de tester vos fonctions.  
