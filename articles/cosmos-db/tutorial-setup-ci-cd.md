---
title: Configurer le pipeline CI/CD avec la tâche de génération d’émulateur Azure Cosmos DB
description: Didacticiel sur la configuration des workflows de création et de mise en production dans Azure DevOps à l’aide de la tâche de génération de l’émulateur Cosmos DB
author: deborahc
ms.service: cosmos-db
ms.topic: how-to
ms.date: 01/28/2020
ms.author: dech
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: a5b8842718aa2d9f90ac06283abc5fe2fdd925cb
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95996999"
---
# <a name="set-up-a-cicd-pipeline-with-the-azure-cosmos-db-emulator-build-task-in-azure-devops"></a>Configurer le pipeline CI/CD avec la tâche de génération d’émulateur Azure Cosmos DB dans Azure DevOps
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

L’émulateur Azure Cosmos DB fournit un environnement local qui émule le service Azure Cosmos DB à des fins de développement. L’émulateur vous permet de développer et tester votre application localement, sans créer d’abonnement Azure et sans frais. 

La tâche de génération de l’émulateur Azure Cosmos DB pour Azure DevOps permet de faire de même dans un environnement d’intégration continue. Avec la tâche de génération, vous pouvez exécuter des tests sur l’émulateur dans le cadre de vos flux de création et de mise en production. La tâche active un conteneur Docker avec l’émulateur déjà en cours d’exécution et fournit un point de terminaison qui peut être utilisé par le reste de la définition de build. Vous pouvez créer et démarrer autant d’instances de l’émulateur que vous le souhaitez, chacune étant exécutée dans un conteneur distinct. 

Cet article montre comment configurer un pipeline d’intégration continue dans Azure DevOps pour une application ASP.NET qui utilise la tâche de génération d’émulateur Cosmos DB pour exécuter des tests. Vous pouvez utiliser une approche similaire pour configurer un pipeline CI destiné à une application Node.js ou Python. 

## <a name="install-the-emulator-build-task"></a>Installer la tâche de génération d’émulateur

Pour utiliser la tâche de génération, nous devons d’abord l’installer dans notre organisation Azure DevOps. Recherchez l’extension **Azure Cosmos DB Emulator** sur la [Place de marché](https://marketplace.visualstudio.com/items?itemName=azure-cosmosdb.emulator-public-preview) et cliquez sur **Obtenir gratuitement.**

:::image type="content" source="./media/tutorial-setup-ci-cd/addExtension_1.png" alt-text="Rechercher la tâche de génération de l’émulateur Azure Cosmos DB dans la Place de marché Azure DevOps et l’installer":::

Ensuite, choisissez l’organisation dans laquelle installer l’extension. 

> [!NOTE]
> Pour installer une extension dans une organisation Azure DevOps, vous devez être propriétaire de compte ou administrateur de collection de projets. Si vous ne disposez pas des autorisations nécessaires, mais que vous êtes un membre de compte, vous pouvez demander des extensions. [En savoir plus.](/azure/devops/marketplace/faq-extensions?preserve-view=true&view=vsts)

:::image type="content" source="./media/tutorial-setup-ci-cd/addExtension_2.png" alt-text="Choisir une organisation Azure DevOps dans laquelle installer une extension":::

## <a name="create-a-build-definition"></a>Créer une définition de build

Maintenant que l’extension est installée, connectez-vous à votre organisation Azure DevOps et recherchez votre projet dans le tableau de bord de projets. Vous pouvez ajouter un [pipeline de build](/azure/devops/pipelines/get-started-designer?preserve-view=true&tabs=new-nav&view=vsts) à votre projet ou modifier un pipeline de build existant. Si vous disposez déjà d’un pipeline de build, vous pouvez passer directement à la section [Ajouter la tâche de génération d’émulateur à une définition de build](#addEmulatorBuildTaskToBuildDefinition).

1. Pour créer une définition de build, accédez à l’onglet **Builds** dans Azure DevOps. Sélectionnez **+Nouveau.** \> **Nouveau pipeline de build**

   :::image type="content" source="./media/tutorial-setup-ci-cd/CreateNewBuildDef_1.png" alt-text="Créer un nouveau pipeline de build":::

2. Sélectionnez la **source**, le **projet d’équipe**, le **référentiel** et la **branche par défaut des builds manuels et planifiés** souhaités. Après avoir choisi les options requises, sélectionnez **Continuer**.

   :::image type="content" source="./media/tutorial-setup-ci-cd/CreateNewBuildDef_2.png" alt-text="Sélectionner le projet d’équipe, le référentiel et la branche pour le pipeline de build":::

3. Enfin, sélectionnez le modèle souhaité pour le pipeline de build. Nous allons sélectionner le modèle **ASP.NET** dans ce tutoriel. Vous disposez maintenant d’un pipeline de build que vous pouvez configurer pour utiliser la tâche de génération de l’émulateur Azure Cosmos DB. 

> [!NOTE]
> Le pool d’agents à sélectionner pour ce CI doit disposer de Docker pour Windows, sauf si l’installation est effectuée manuellement dans une tâche précédente dans le cadre de ce CI. Consultez l’article [Agents hébergés Microsoft](/azure/devops/pipelines/agents/hosted?preserve-view=true&tabs=yaml&view=azure-devops) pour une sélection de pools d’agents ; nous vous recommandons de commencer par `Hosted VS2017`.

L’émulateur Azure Cosmos DB ne prend actuellement pas en charge le pool d’agents VS2019 hébergé. Toutefois, l’émulateur est déjà équipé de VS2019. Pour l’utiliser, démarrez l’émulateur avec les applets de commande PowerShell suivantes. Si vous rencontrez des problèmes lors de l’utilisation de VS2019, contactez l’équipe [Azure DevOps](https://developercommunity.visualstudio.com/spaces/21/index.html) pour obtenir de l’aide :

```powershell
Import-Module "$env:ProgramFiles\Azure Cosmos DB Emulator\PSModules\Microsoft.Azure.CosmosDB.Emulator"
Start-CosmosDbEmulator
```

## <a name="add-the-task-to-a-build-pipeline"></a><a name="addEmulatorBuildTaskToBuildDefinition"></a>Ajouter la tâche à un pipeline de build

1. Avant d’ajouter une tâche au pipeline de build, vous devez ajouter un travail d’agent. Accédez à votre pipeline de build, sélectionnez **...** et choisissez **Ajouter un travail d’agent**.

1. Sélectionnez ensuite le symbole **+** en regard de la tâche d’agent pour ajouter la tâche de génération d’émulateur. Dans la zone de recherche, tapez **cosmos**, puis sélectionnez **Azure Cosmos DB Emulator** (Émulateur Azure Cosmos DB) et ajoutez-le au travail d’agent. La tâche de génération démarre un conteneur avec une instance de l’émulateur Cosmos DB déjà en cours d’exécution sur ce dernier. La tâche de l’émulateur Azure Cosmos DB doit être placée avant toutes les autres tâches qui attendent que l’émulateur soit en cours d’exécution.

   :::image type="content" source="./media/tutorial-setup-ci-cd/addExtension_3.png" alt-text="Ajouter la tâche de génération d’émulateur à une définition de build":::

Dans ce didacticiel, vous allez ajouter la tâche au début, afin que l’émulateur soit disponible avant d’exécuter nos tests.

### <a name="add-the-task-using-yaml"></a>Ajouter la tâche à l’aide de YAML

Cette étape est facultative et n’est nécessaire que si vous configurez le pipeline CI/CD au moyen d’une tâche YAML. Dans ce cas, vous pouvez définir la tâche YAML comme indiqué dans le code suivant :

```yml
- task: azure-cosmosdb.emulator-public-preview.run-cosmosdbemulatorcontainer.CosmosDbEmulator@2
  displayName: 'Run Azure Cosmos DB Emulator'

- script: yarn test
  displayName: 'Run API tests (Cosmos DB)'
  env:
    HOST: $(CosmosDbEmulator.Endpoint)
    # Hardcoded key for emulator, not a secret
    AUTH_KEY: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
    # The emulator uses a self-signed cert, disable TLS auth errors
    NODE_TLS_REJECT_UNAUTHORIZED: '0'
```

## <a name="configure-tests-to-use-the-emulator"></a>Configurer des tests pour utiliser l’émulateur

Maintenant, nous allons configurer nos tests pour utiliser l’émulateur. La tâche de génération d’émulateur exporte une variable d’environnement (CosmosDbEmulator.Endpoint) vers laquelle toutes les tâches suivantes du pipeline de build peuvent émettre des requêtes. 

Dans ce tutoriel, nous allons utiliser la [tâche Visual Studio Test](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/VsTestV2/README.md) pour exécuter des tests unitaires configurés via un fichier **.runsettings**. Pour en savoir plus sur l’initialisation de test unitaire, consultez la [documentation](/visualstudio/test/configure-unit-tests-by-using-a-dot-runsettings-file?preserve-view=true&view=vs-2017). L’exemple complet de code d’application Todo que vous utilisez dans ce document est disponible sur [GitHub](https://github.com/Azure-Samples/documentdb-dotnet-todo-app).

Voici un exemple de fichier **.runsettings** qui définit les paramètres à transmettre dans les tests unitaires d’une application. Remarquez que la variable `authKey` utilisée est la [clé connue](./local-emulator.md#authenticate-requests) pour l’émulateur. Cette `authKey` est la clé attendue par la tâche de génération d’émulateur Elle doit être définie dans votre fichier **.runsettings**.

```csharp
<RunSettings>
    <TestRunParameters>
    <Parameter name="endpoint" value="https://localhost:8081" />
    <Parameter name="authKey" value="C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==" />
    <Parameter name="database" value="ToDoListTest" />
    <Parameter name="collection" value="ItemsTest" />
  </TestRunParameters>
</RunSettings>
```

Si vous configurez un pipeline CI/CD pour une application qui utilise l’API Azure Cosmos DB pour MongoDB, la chaîne de connexion par défaut inclut le numéro de port 10255. Toutefois, ce port n’est pas ouvert actuellement ; en guise d’alternative, utilisez plutôt le port 10250 pour établir la connexion. La chaîne de connexion de l’API Azure Cosmos DB pour MongoDB reste la même, à l’exception du numéro de port pris en charge qui est 10250 au lieu de 10255.

Ces paramètres `TestRunParameters` sont référencés via une propriété `TestContext` dans le projet de test de l’application. Voici un exemple de test qui s’exécute sur Cosmos DB.

```csharp
namespace todo.Tests
{
    [TestClass]
    public class TodoUnitTests
    {
        public TestContext TestContext { get; set; }

        [TestInitialize()]
        public void Initialize()
        {
            string endpoint = TestContext.Properties["endpoint"].ToString();
            string authKey = TestContext.Properties["authKey"].ToString();
            Console.WriteLine("Using endpoint: ", endpoint);
            DocumentDBRepository<Item>.Initialize(endpoint, authKey);
        }
        [TestMethod]
        public async Task TestCreateItemsAsync()
        {
            var item = new Item
            {
                Id = "1",
                Name = "testName",
                Description = "testDescription",
                Completed = false,
                Category = "testCategory"
            };

            // Create the item
            await DocumentDBRepository<Item>.CreateItemAsync(item);
            // Query for the item
            var returnedItem = await DocumentDBRepository<Item>.GetItemAsync(item.Id, item.Category);
            // Verify the item returned is correct.
            Assert.AreEqual(item.Id, returnedItem.Id);
            Assert.AreEqual(item.Category, returnedItem.Category);
        }

        [TestCleanup()]
        public void Cleanup()
        {
            DocumentDBRepository<Item>.Teardown();
        }
    }
}
```

Accédez aux options d’exécution dans la tâche de test Visual Studio. Dans l’option **Settings file** (Fichier de paramètres), spécifiez que les tests sont configurés à l’aide du fichier **.runsettings**. Dans l’option **Remplacer les paramètres de série de tests**, ajoutez `-endpoint $(CosmosDbEmulator.Endpoint)`. Cela configurera la tâche de test pour faire référence au point de terminaison de la tâche de génération d’émulateur, au lieu de celle définie dans le fichier **.runsettings**.  

:::image type="content" source="./media/tutorial-setup-ci-cd/addExtension_5.png" alt-text="Remplacer la variable de point de terminaison par le point de terminaison de la tâche de génération d’émulateur":::

## <a name="run-the-build"></a>Exécutez la génération

Maintenant, **enregistrez et mettez en file d’attente** le build. 

:::image type="content" source="./media/tutorial-setup-ci-cd/runBuild_1.png" alt-text="Capture d’écran présentant une build avec l’option Enregistrer et ajouter à la file d’attente sélectionnée.":::

Une fois que la génération a démarré, vous noterez que la tâche d’émulateur Cosmos DB a commencé une extraction de l’image Docker avec l’émulateur installé. 

:::image type="content" source="./media/tutorial-setup-ci-cd/runBuild_4.png" alt-text="Capture d’écran montre la tâche d’émulateur Cosmos D B en cours d’extraction.":::

Une fois la génération terminée, vous noterez que vos tests réussissent, et qu’ils s’exécutent tous sur l’émulateur Cosmos DB de la tâche de génération !

:::image type="content" source="./media/tutorial-setup-ci-cd/buildComplete_1.png" alt-text="Capture d’écran affichant la valeur de progression sous l’onglet Résumé.":::

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’utilisation de l’émulateur pour développer en local et tester, consultez [Utilisation de l’émulateur Azure Cosmos DB pour le développement local et le test](./local-emulator.md).

Pour exporter les certificats TLS/SSL de l’émulateur, consultez [Exporter les certificats de l’émulateur Azure Cosmos DB pour une utilisation avec Java, Python et Node.js](./local-emulator-export-ssl-certificates.md)
