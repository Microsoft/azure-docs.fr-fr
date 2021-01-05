---
title: Développer et déployer WebJobs à l’aide de Visual Studio
description: Découvrez comment développer des tâches Azure WebJobs dans Visual Studio et les déployer dans Azure App Service, notamment en créant une tâche planifiée.
author: ggailey777
ms.assetid: a3a9d320-1201-4ac8-9398-b4c9535ba755
ms.topic: conceptual
ms.custom: devx-track-csharp, vs-azure
ms.date: 07/30/2020
ms.author: glenga
ms.reviewer: david.ebbo;suwatch;pbatum;naren.soni
ms.openlocfilehash: de10903be86b52b3415b57a53be81e7fd1661f63
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89226027"
---
# <a name="develop-and-deploy-webjobs-using-visual-studio"></a>Développer et déployer WebJobs à l’aide de Visual Studio

Cet article explique comme utiliser Visual Studio pour déployer un projet d’application console sur une application web dans [Azure App Service](overview.md) en tant que [WebJob Azure](https://go.microsoft.com/fwlink/?LinkId=390226). Pour plus d’informations sur le déploiement de WebJobs à l’aide du [portail Azure](https://portal.azure.com), consultez la section [Exécuter des tâches en arrière-plan avec WebJobs dans Azure App Service](webjobs-create.md).

Vous pouvez choisir de développer un WebJob qui s’exécute soit comme une [application .NET Core](#webjobs-as-net-core-console-apps), soit comme une [application .NET Framework](#webjobs-as-net-framework-console-apps). La version 3.x du [SDK Azure WebJobs](webjobs-sdk-how-to.md) vous permet de développer des tâches web qui s'exécutent en tant qu'applications .NET Core ou .NET Framework, tandis que la version 2.x ne prend en charge que le .NET Framework. La façon dont vous déployez un projet WebJobs est différente pour les projets .NET Core par rapport aux projets .NET Framework.

Vous pouvez publier plusieurs WebJobs dans une application web unique, à condition que chaque WebJob d’une application web ait un nom unique.

## <a name="webjobs-as-net-core-console-apps"></a>Tâches web en tant qu’applications de console .NET Core

Avec la version 3.x du Kit de développement logiciel (SDK) Azure WebJobs, vous pouvez créer et publier des WebJobs en tant qu’applications console .NET Core. Pour obtenir des instructions étape par étape afin de créer et de publier une application console .NET Core pour Azure en tant que WebJob, consultez [Prise en main du Kit de développement logiciel (SDK) Azure WebJobs pour le traitement en arrière-plan basé sur les événements](webjobs-sdk-get-started.md).

> [!NOTE]
> Les WebJobs .NET Core ne peuvent pas être liés à des projets web. Si vous avez besoin de déployer votre WebJob avec une application web, [créez vos WebJobs en tant qu’application console .NET Framework](#webjobs-as-net-framework-console-apps).  

### <a name="deploy-to-azure-app-service"></a>Déployer dans Azure App Service

La publication d’un WebJob .NET Core sur Azure App Service à partir de Visual Studio utilise les mêmes outils que la publication d’une application ASP.NET Core.

[!INCLUDE [webjobs-publish-net-core](../../includes/webjobs-publish-net-core.md)] 

## <a name="webjobs-as-net-framework-console-apps"></a>Tâches web en tant qu’applications de console .NET Framework  

Si vous utilisez Visual Studio pour déployer un projet d’application console .NET Framework pour WebJobs, il copie les fichiers de runtime dans le dossier approprié de l’application web (*App_Data/jobs/continuous* pour les WebJobs continus et *App_Data/jobs/triggered* pour les WebJobs planifiés ou à la demande).

Visual Studio ajoute les éléments suivants à un projet pour WebJobs :

* Le package NuGet [Microsoft.Web.WebJobs.Publish](https://www.nuget.org/packages/Microsoft.Web.WebJobs.Publish/) .
* Un fichier [webjob-publish-settings.json](#publishsettings) qui contient des paramètres de déploiement et de planification. 

![Diagramme montrant ce qui est ajouté à une application console pour activer le déploiement en tant que WebJob](./media/webjobs-dotnet-deploy-vs/convert.png)

Vous pouvez ajouter ces éléments à un projet d’application console existant ou utiliser un modèle pour créer un projet d’application console pour WebJobs. 

Déployez un projet sous forme de WebJob seul ou liez-le à un projet web afin qu’il soit déployé automatiquement lorsque vous déployez le projet web. Pour lier les projets, Visual Studio inclut le nom du projet compatible avec les tâches web dans un fichier [webjobs-list.json](#webjobslist) dans le projet web.

![Diagram showing WebJob project linking to web project](./media/webjobs-dotnet-deploy-vs/link.png)

### <a name="prerequisites"></a>Prérequis

Installez Visual Studio 2017 ou Visual Studio 2019 avec la [charge de travail Développement Azure](/visualstudio/install/install-visual-studio#step-4---choose-workloads).

### <a name="enable-webjobs-deployment-for-an-existing-console-app-project"></a><a id="convert"></a> Activer le déploiement de WebJobs pour un projet d’application console

Deux options s'offrent à vous :

* [Activation d'un déploiement automatique avec un projet web](#convertlink).

  Configurez un projet d’application console existant afin qu’il soit déployé automatiquement sous forme de WebJob lorsque vous déployez un projet web. Utilisez cette option lorsque vous voulez exécuter votre tâche web dans la même application web que celle dans laquelle vous exécutez l’application web liée.

* [Activation d'un déploiement sans projet web](#convertnolink).

  Configurez un projet d’application console existant pour déployer en tant que WebJob seul, sans lien avec un projet web. Utilisez cette option lorsque vous voulez exécuter une tâche web dans une application web seule, sans application web s’exécutant dans cette dernière. Vous pouvez utiliser cette méthode pour mettre à l’échelle vos ressources WebJob indépendamment de vos ressources d’application web.

#### <a name="enable-automatic-webjobs-deployment-with-a-web-project"></a><a id="convertlink"></a> Activer un déploiement automatique de tâches web avec un projet web

1. Cliquez avec le bouton droit sur le projet web dans **Explorateur de solutions**, puis sélectionnez **Ajouter** > **Projet existant en tant que WebJob Azure**.
   
    ![Projet existant sous forme de tâche web Azure](./media/webjobs-dotnet-deploy-vs/eawj.png)
   
    La boîte de dialogue [Ajouter une tâche web Azure](#configure) s'affiche.
2. Dans la liste déroulante **Nom du projet**, sélectionnez le projet d’application console à ajouter sous forme de WebJob.
   
    ![Sélection du projet dans la boîte de dialogue Ajouter un WebJob Azure](./media/webjobs-dotnet-deploy-vs/aaw1.png)
3. Remplissez la boîte de dialogue [Ajouter un WebJob Azure](#configure), puis sélectionnez **OK**. 

#### <a name="enable-webjobs-deployment-without-a-web-project"></a><a id="convertnolink"></a> Activer un déploiement de tâches web sans projet web
1. Cliquez avec le bouton droit sur le projet d’application console dans **Explorateur de solutions**, puis sélectionnez **Publier en tant que WebJob Azure**. 
   
    ![Publier sous forme de tâche web Azure](./media/webjobs-dotnet-deploy-vs/paw.png)
   
    La boîte de dialogue [Ajouter une tâche web Azure](#configure) s'affiche avec le projet sélectionné dans la zone **Nom du projet** .
2. Remplissez la boîte de dialogue [Ajouter un WebJob Azure](#configure), puis sélectionnez **OK**.
   
   L'Assistant **Publier le site Web** s'ouvre. Si vous ne voulez pas publier immédiatement, fermez l'Assistant. Les paramètres que vous avez saisis sont enregistrés au cas où vous souhaiteriez [déployer le projet](#deploy).

### <a name="create-a-new-webjobs-enabled-project"></a><a id="create"></a>Créer un projet compatible avec les tâches web
Pour créer un projet pour WebJobs, utilisez le modèle de projet d’application console et activez le déploiement de WebJobs comme expliqué dans [la section précédente](#convert). Vous pouvez également utiliser le modèle de nouveau projet de tâche web :

* [Utiliser le modèle de nouveau projet WebJobs pour une tâche web indépendante](#createnolink)
  
    Créez un projet et configurez-le pour qu'il se déploie seul sous forme de tâche web, sans lien avec un projet web. Utilisez cette option lorsque vous voulez exécuter une tâche web dans une application web seule, sans application web s’exécutant dans cette dernière. Vous pouvez utiliser cette méthode pour mettre à l’échelle vos ressources WebJob indépendamment de vos ressources d’application web.
* [Utiliser le modèle de nouveau projet WebJobs pour une tâche web liée à un projet web](#createlink)
  
    Créez un projet configuré pour être déployé automatiquement sous forme de WebJob lorsque vous déployez un projet web dans la même solution. Utilisez cette option lorsque vous voulez exécuter votre tâche web dans la même application web que celle dans laquelle vous exécutez l’application web liée.

> [!NOTE]
> Le modèle de nouveau projet WebJobs installe automatiquement les packages NuGet et inclut le code dans *Program.cs* pour le [Kit de développement logiciel (SDK) WebJobs](https://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/getting-started-with-windows-azure-webjobs). Si vous ne souhaitez pas utiliser le Kit de développement logiciel (SDK) WebJobs, supprimez ou modifiez l’instruction `host.RunAndBlock` dans *Program.cs*.
> 
> 

#### <a name="use-the-webjobs-new-project-template-for-an-independent-webjob"></a><a id="createnolink"></a> Utiliser le modèle de nouveau projet WebJobs pour une tâche web indépendante
1. Sélectionnez **Fichier** > **Nouveau** > **Projet**. Dans la boîte de dialogue **Créer un nouveau projet**, recherchez et sélectionnez **WebJob Azure (.NET Framework)** pour C#.
   
2. Suivez les instructions précédentes pour [transformer le projet d’application console en projet WebJobs indépendant](#convertnolink).

#### <a name="use-the-webjobs-new-project-template-for-a-webjob-linked-to-a-web-project"></a><a id="createlink"></a> Utiliser le modèle de nouveau projet WebJobs pour une tâche web liée à un projet web
1. Cliquez avec le bouton droit sur le projet web dans **Explorateur de solutions**, puis sélectionnez **Ajouter** > **Nouveau projet de WebJob Azure**.
   
    ![New Azure WebJob Project menu entry](./media/webjobs-dotnet-deploy-vs/nawj.png)
   
    La boîte de dialogue [Ajouter une tâche web Azure](#configure) s'affiche.
2. Remplissez la boîte de dialogue [Ajouter un WebJob Azure](#configure), puis sélectionnez **OK**.


### <a name="webjob-publish-settingsjson-file"></a><a id="publishsettings"></a>Fichier webjob-publish-settings.json
Lorsque vous configurez une application console pour un déploiement WebJobs, Visual Studio installe le package NuGet [Microsoft.Web.WebJobs.Publish](https://www.nuget.org/packages/Microsoft.Web.WebJobs.Publish/) et stocke les informations de planification dans un fichier *webjob-publish-settings.json* du dossier *Propriétés* du projet WebJobs. Voici un exemple de ce fichier :

```json
{
  "$schema": "http://schemastore.org/schemas/json/webjob-publish-settings.json",
  "webJobName": "WebJob1",
  "startTime": "null",
  "endTime": "null",
  "jobRecurrenceFrequency": "null",
  "interval": null,
  "runMode": "Continuous"
}
```

Vous pouvez modifier ce fichier directement. Visual Studio est doté d'IntelliSense. Le schéma de fichier est stocké à l’adresse [https://schemastore.org](http://schemastore.org/schemas/json/webjob-publish-settings.json) où vous pouvez le consulter.  

### <a name="webjobs-listjson-file"></a><a id="webjobslist"></a>Fichier webjobs-list.json
Lorsque vous liez un projet compatible avec des tâches web à un projet web, Visual Studio stocke le nom du projet de tâches web sous le nom de fichier *webjobs-list.json* dans le dossier *Propriétés* du projet web. La liste peut contenir plusieurs projets WebJobs, comme illustré dans l’exemple suivant :

```json
{
  "$schema": "http://schemastore.org/schemas/json/webjobs-list.json",
  "WebJobs": [
    {
      "filePath": "../ConsoleApplication1/ConsoleApplication1.csproj"
    },
    {
      "filePath": "../WebJob1/WebJob1.csproj"
    }
  ]
}
```

Vous pouvez modifier ce fichier directement dans Visual Studio, grâce à IntelliSense. Le schéma de fichier est stocké à l’adresse [https://schemastore.org](http://schemastore.org/schemas/json/webjobs-list.json).

### <a name="deploy-a-webjobs-project"></a><a id="deploy"></a>Déployer un projet WebJobs
Lorsqu’il est lié à un projet web, un projet WebJobs est déployé automatiquement avec ce dernier. Pour plus d’informations sur le déploiement du projet web, consultez **Guides pratiques** > **Déployer l’application** dans le volet de navigation gauche.

Pour déployer un projet WebJobs seul, cliquez avec le bouton droit sur le projet dans **Explorateur de solutions** et sélectionnez **Publier en tant que WebJob Azure**. 

![Publier sous forme de tâche web Azure](./media/webjobs-dotnet-deploy-vs/paw.png)

Pour une tâche web indépendante, l'Assistant **Publier le site Web** utilisé pour les projets web s'affiche, mais avec moins de paramètres modifiables.

### <a name="add-azure-webjob-dialog-box"></a><a id="configure"></a>Boîte de dialogue Ajouter un WebJob Azure
La boîte de dialogue **Ajouter un WebJob Azure** vous permet d’entrer le nom du WebJob et le paramètre du mode d’exécution de votre WebJob. 

![Boîte de dialogue Ajouter un WebJob Azure](./media/webjobs-dotnet-deploy-vs/aaw2.png)

Certains champs de cette boîte de dialogue correspondent aux champs de la boîte de dialogue **Ajouter un WebJob** du portail Azure. Pour plus d’informations, consultez [Exécuter des tâches en arrière-plan avec WebJobs dans Azure App Service](webjobs-create.md).

Informations sur le déploiement de WebJob :

* Pour plus d’informations sur le déploiement en ligne de commande, consultez la page [Activation de la ligne de commande ou de la livraison continue de tâches web Azure](https://azure.microsoft.com/blog/2014/08/18/enabling-command-line-or-continuous-delivery-of-azure-webjobs/).

* Si vous déployez un WebJob et que vous décidez ensuite d’en modifier le type et de le redéployer, supprimez le fichier *webjobs-publish-settings.json*. Dans ce cas, Visual Studio affiche à nouveau les options de publication, ce qui vous permet de modifier le type de WebJob.

* Si vous déployez une tâche web et que vous modifiez ultérieurement le mode d'exécution continue en non continue ou vice-versa, Visual Studio crée une tâche web dans Azure lorsque vous procédez à un nouveau déploiement. Si vous modifiez d’autres paramètres de planification, mais que vous conservez le même mode d’exécution ou passez du type Planifié au type À la demande, Visual Studio met à jour la tâche existante plutôt que d’en créer une.

## <a name="webjob-types"></a>Types de tâches web

Le type d’un WebJob peut être *déclenché* ou *continu* :

- Déclenché (par défaut) : Un WebJob déclenché démarre en fonction d’un événement contraignant, d’une [planification](#scheduling-a-triggered-webjob) ou lorsque vous le déclenchez manuellement (à la demande). Il s’exécute sur une instance unique où l’application web s’exécute.

- Continu : Un WebJob [continu](#continuous-execution) démarre immédiatement lorsque le WebJob est créé. Il s’exécute sur toutes les instances d’application web mises à l’échelle par défaut, mais peut être configuré pour s’exécuter en tant qu’instance unique via *settings.job*.

[!INCLUDE [webjobs-alwayson-note](../../includes/webjobs-always-on-note.md)]

### <a name="scheduling-a-triggered-webjob"></a>Planification d’une tâche web déclenchée

Lorsque vous publiez une application console sur Azure, Visual Studio définit le type de WebJob sur **Déclenché** par défaut et ajoute un nouveau fichier *settings.job* au projet. Pour les types de WebJob déclenchés, vous pouvez utiliser ce fichier pour définir une planification de l’exécution de votre WebJob.

Utilisez le fichier *settings.job* pour définir une planification de l’exécution de votre tâche web. L’exemple suivant s’exécute toutes les heures entre 9 h et 17 h :

```json
{
    "schedule": "0 0 9-17 * * *"
}
```

Ce fichier se trouve à la racine du dossier WebJobs avec le script de votre WebJob, par exemple `wwwroot\app_data\jobs\triggered\{job name}` ou `wwwroot\app_data\jobs\continuous\{job name}`. Quand vous déployez un WebJob à partir de Visual Studio, marquez les propriétés du fichier *settings.job* comme **Copier si plus récent**.

Si vous [créez un WebJob à partir du portail Azure](webjobs-create.md), le fichier *settings.job* est créé pour vous.

#### <a name="cron-expressions"></a>Expressions CRON

Un projet WebJobs utilise les mêmes expressions CRON pour la planification que le déclencheur de minuteur dans Azure Functions. Pour plus d’informations sur la prise en charge de Cron, consultez [Déclencheur de minuteur pour Azure Functions](../azure-functions/functions-bindings-timer.md#ncrontab-expressions).

[!INCLUDE [webjobs-cron-timezone-note](../../includes/webjobs-cron-timezone-note.md)]

#### <a name="settingsjob-reference"></a>Référence settings.job

Les paramètres suivants sont pris en charge par les tâches web :

| **Paramètre** | **Type**  | **Description** |
| ----------- | --------- | --------------- |
| `is_in_place` | Tous | Autorise le WebJob à s’exécuter sur place sans être d’abord copié vers un dossier temporaire. Pour en savoir plus, consultez la section [Répertoire de travail WebJob](https://github.com/projectkudu/kudu/wiki/WebJobs#webjob-working-directory). |
| `is_singleton` | Continue | Exécutez uniquement le WebJob sur une seule instance lors d’un scale-out. Pour plus d’informations, consultez [Set a continuous job as singleton](https://github.com/projectkudu/kudu/wiki/WebJobs-API#set-a-continuous-job-as-singleton) (Définir une tâche continue comme singleton). |
| `schedule` | Déclenchée | Exécutez la tâche web selon une planification basée sur CRON. Pour plus d’informations, consultez [Expressions NCRONTAB](../azure-functions/functions-bindings-timer.md#ncrontab-expressions). |
| `stopping_wait_time`| Tous | Permet de contrôler le comportement d’arrêt. Pour plus d'informations, consultez [Arrêt correct](https://github.com/projectkudu/kudu/wiki/WebJobs#graceful-shutdown). |

### <a name="continuous-execution"></a>Exécution en continu

Si vous activez **Always On** dans Azure, vous pouvez utiliser Visual Studio pour modifier le WebJob pour qu’il s’exécute en continu :

1. Si ce n’est déjà fait, [publiez le projet sur Azure](#deploy-to-azure-app-service).

1. Dans **l’Explorateur de solutions**, cliquez avec le bouton droit sur le projet, puis sélectionnez **Publier**.

1. Dans l’onglet **Publier**, choisissez **Modifier**. 

1. Dans la boîte de dialogue **Paramètres du profil**, choisissez **Continu** pour **Type de WebJob**, puis choisissez **Enregistrer**.

    ![Boîte de dialogue Paramètres de publication pour un WebJob](./media/webjobs-dotnet-deploy-vs/publish-settings.png)

1. Sélectionnez **Publier** dans l’onglet **Publier** pour republier le WebJob avec les paramètres mis à jour.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur le kit SDK WebJobs](webjobs-sdk-how-to.md)