---
title: Déployer les applications de manière prévisible avec ARM
description: Découvrez comment déployer plusieurs applications Azure App Service sous forme d’unité unique et de manière prévisible en utilisant des modèles Azure Resource Manager et des scripts PowerShell.
ms.assetid: bb51e565-e462-4c60-929a-2ff90121f41d
ms.topic: article
ms.date: 01/06/2016
ms.custom: seodec18
ms.openlocfilehash: 6c45d2da8658740b5e5e7e3dceb7478ea28d712c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88962024"
---
# <a name="provision-and-deploy-microservices-predictably-in-azure"></a>Mise en service et déploiement de microservices de manière prévisible dans Azure
Ce didacticiel explique comment mettre en service et déployer une application composée de [microservices](https://en.wikipedia.org/wiki/Microservices) dans [Azure App Service](https://azure.microsoft.com/services/app-service/) sous la forme d'une unité unique et de manière prévisible, à l'aide de modèles de groupe de ressources JSON et des scripts PowerShell. 

Lors de la mise en service et du déploiement d'applications à grande échelle qui sont composées de microservices hautement découplés, la répétition et la prévisibilité sont essentielles à la réussite. [Azure App Service](https://azure.microsoft.com/services/app-service/) vous permet de créer des microservices qui incluent des applications web, des backends mobiles et des applications API. [Azure Resource Manager](../azure-resource-manager/management/overview.md) vous permet de gérer tous les microservices en tant qu'unité, avec les dépendances de ressources, comme les paramètres de contrôle de la source et de base de données. À présent, vous pouvez également déployer une application de ce type à l’aide de modèles JSON et de scripts PowerShell simples. 

## <a name="what-you-will-do"></a>Procédure à suivre
Dans ce didacticiel, vous allez déployer une application incluant les éléments suivants :

* Deux applications App Service (par exemple, deux microservices)
* Une base de données SQL principale
* Paramètres d’application, chaînes de connexion et contrôle de code source
* Paramètres de mise à l’échelle automatique, alertes et Application Insights

## <a name="tools-you-will-use"></a>Outils à utiliser
Dans ce didacticiel, vous allez utiliser les outils suivants. Dans la mesure où il ne s’agit pas d’une discussion complète sur les outils, je vais opter pour le scénario de bout en bout : je vais vous présenter chacun d’eux et vous indiquer à quel emplacement vous pouvez trouver davantage d’informations le concernant. 

### <a name="azure-resource-manager-templates-json"></a>Modèles Microsoft Azure Resource Manager (JSON)
Chaque fois que vous créez une application dans Microsoft Azure App Service, par exemple, Microsoft Azure Resource Manager utilise un modèle JSON pour créer le groupe de ressources dans son ensemble, avec les ressources des composants. Un modèle complexe sur la [Place de marché Azure](../marketplace/index.yml) peut inclure la base de données, les comptes de stockage, le plan App Service, l’application elle-même, les règles relatives aux alertes, les paramètres d’application et de mise à l’échelle automatique, etc. Tous ces modèles sont à votre disposition, par le biais de PowerShell. Pour en savoir plus sur les modèles Microsoft Azure Resource Manager, voir [Création de modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md)

### <a name="azure-sdk-26-for-visual-studio"></a>Kit de développement logiciel (SDK) Microsoft Azure version 2.6 pour Visual Studio
Le Kit de développement logiciel (SDK) le plus récent contient des améliorations portant sur la prise en charge des modèles Resource Manager dans l’éditeur JSON. Vous pouvez utiliser ce Kit pour créer rapidement un modèle de groupe de ressources à partir de zéro, ou ouvrir un modèle JSON existant (comme un modèle de galerie téléchargé) à des fins de modification, pour remplir le fichier de paramètres, voire pour déployer le groupe de ressources directement à partir d’une solution de groupe de ressources Microsoft Azure.

Pour en savoir plus, voir [Kit de développement logiciel (SDK) Microsoft Azure version 2.6 pour Visual Studio](https://azure.microsoft.com/blog/2015/04/29/announcing-the-azure-sdk-2-6-for-net/).

### <a name="azure-powershell-080-or-later"></a>Microsoft Azure PowerShell 0.8.0 ou plus
Depuis la version 0.8.0, l’installation de Microsoft Azure PowerShell inclut le module Microsoft Azure Resource Manager en plus du module Microsoft Azure. Ce nouveau module vous permet de créer un script pour le déploiement des groupes de ressources.

Pour en savoir plus, voir [Utilisation d’Azure PowerShell avec le Gestionnaire de ressources Azure](../azure-resource-manager/management/manage-resources-powershell.md)

### <a name="azure-resource-explorer"></a>Azure Resource Explorer
Cela [outil Aperçu](https://resources.azure.com) vous permet d’explorer les définitions JSON de tous les groupes de ressources dans votre abonnement et les ressources individuelles. Dans cet outil, vous pouvez modifier les définitions  JSON d’une ressource, supprimer une hiérarchie entière de ressources et créer des ressources.  Les informations proposées dans cet outil sont très utiles pour créer des modèles. En effet, elles vous indiquent quelles propriétés vous devez définir pour un type de ressource spécifique, ainsi que les valeurs correctes, etc. Vous pouvez même créer le groupe de ressources dans le [portail Azure](https://portal.azure.com/), puis inspecter ses définitions JSON dans l’outil d’exploration, afin de pouvoir créer des modèles pour le groupe de ressources.

### <a name="deploy-to-azure-button"></a>Bouton Déployer dans Azure
Si vous utilisez GitHub pour contrôler le code source, vous pouvez placer un [bouton Déployer dans Azure](https://azure.microsoft.com/blog/2014/11/13/deploy-to-azure-button-for-azure-websites-2/) dans votre fichier README.MD, ce qui permet un déploiement clé en main de l’interface utilisateur pour Microsoft Azure. Vous pouvez effectuer cette action pour toute application simple, mais il est possible de l’étendre afin de permettre le déploiement d’un groupe entier de ressources, en plaçant un fichier azuredeploy.json dans la racine du référentiel. Ce fichier JSON, qui contient le modèle de groupe de ressources, sera utilisé par le bouton Déployer dans Azure pour créer le groupe de ressources. Pour vous aider, vous pouvez consulter l’exemple [ToDoApp](https://github.com/azure-appservice-samples/ToDoApp) , que vous utiliserez dans ce didacticiel.

## <a name="get-the-sample-resource-group-template"></a>Obtenir l’exemple de modèle de groupe de ressources
À présent, mettons-nous à l’œuvre.

1. Accédez à l’exemple App Service appelé [ToDoApp](https://github.com/azure-appservice-samples/ToDoApp) .
2. Dans le fichier readme.md, cliquez sur **Déployer dans Azure**.
3. Vous êtes redirigé vers le site [deploy-to-azure](https://deploy.azure.com) et invité à saisir des paramètres de déploiement. Vous pouvez constater que la plupart des champs sont remplis pour vous, avec le nom du référentiel et certaines chaînes aléatoires. Vous pouvez modifier tous les champs que vous souhaitez, mais les seuls éléments que vous devez saisir sont l’identifiant de connexion et le mot de passe d’administration SQL Server. Ensuite, cliquez sur **Suivant**.
   
   ![Montre les paramètres de déploiement d’entrée sur le site deploy-to-azure.](./media/app-service-deploy-complex-application-predictably/gettemplate-1-deploybuttonui.png)
4. Ensuite, cliquez sur **Déployer** pour démarrer le processus de déploiement. Une fois le processus terminé, cliquez sur le lien http://todoapp*XXXX*.azurewebsites.net pour accéder à l’application déployée. 
   
   ![Montre le processus de déploiement de votre application.](./media/app-service-deploy-complex-application-predictably/gettemplate-2-deployprogress.png)
   
   Il se peut que l’interface utilisateur soit un peu lente lorsque vous y accédez pour la première fois, car les applications sont en plein démarrage. Cependant, n’ayez aucune crainte : il s’agit d’une application entièrement opérationnelle.
5. Dans la page Déployer, cliquez sur le lien **Gérer** pour afficher la nouvelle application dans le portail Azure.
6. Dans la liste déroulante **Essentials** , cliquez sur le lien du groupe de ressources. Vous pouvez également voir que l’application est déjà connectée au référentiel GitHub, sous **Projet externe**. 
   
   ![Montre le lien du groupe de ressources dans la section de la liste déroulante Essentials.](./media/app-service-deploy-complex-application-predictably/gettemplate-3-portalresourcegroup.png)
7. Dans le volet du groupe de ressources, notez que ce dernier inclut déjà deux applications et une base de données SQL.
   
   ![Montre les ressources disponibles dans votre groupe de ressources.](./media/app-service-deploy-complex-application-predictably/gettemplate-4-portalresourcegroupclicked.png)

Comme vous avez pu le constater, en quelques minutes, deux applications de microservices ont été entièrement déployées, avec l'ensemble de leurs composants, dépendances, paramètres et bases de données, ainsi que la publication en continue, configurés par l'orchestration automatisée de Microsoft Azure Resource Manager. Tout cela a été possible grâce à deux éléments :

* le bouton Déployer dans Azure ;
* la présence du fichier azuredeploy.json dans la racine du référentiel.

Vous pouvez déployer cette application des dizaines, des centaines, voire des milliers de fois, en appliquant la même configuration chaque fois. La répétabilité et la prévisibilité de cette approche vous permettent de déployer des applications à grande échelle, en toute simplicité et en toute confiance.

## <a name="examine-or-edit-azuredeployjson"></a>Examiner ou modifier le fichier AZUREDEPLOY.JSON
Examinons à présent la configuration du référentiel GitHub. Vous allez à présent utiliser l’éditeur JSON dans le Kit de développement logiciel (SDK) Azure .NET. Vous devez installer le [Kit de développement logiciel (SDK) Microsoft Azure.NET version 2.6](https://azure.microsoft.com/downloads/), le cas échéant.

1. Clonez le référentiel [ToDoApp](https://github.com/azure-appservice-samples/ToDoApp) via votre outil git favori. Dans la capture d’écran ci-dessous, vous pouvez voir que j’utilise l’outil Team Explorer de Visual Studio 2013.
   
   ![Montre comment utiliser un outil git pour cloner le dépôt ToDoApp.](./media/app-service-deploy-complex-application-predictably/examinejson-1-vsclone.png)
2. À partir de la racine du référentiel, ouvrez le fichier azuredeploy.json dans Visual Studio. Si vous ne voyez pas apparaître le volet du plan JSON, vous devez installer le Kit de développement logiciel (SDK) Microsoft Azure .NET.
   
   ![Montre le volet Structure JSON de Visual Studio.](./media/app-service-deploy-complex-application-predictably/examinejson-2-vsjsoneditor.png)

Je ne vais pas décrire tous les détails du format JSON ; cependant, la section [Autres ressources](#resources) présente des liens permettant d’apprendre le langage de modèle du groupe de ressources. Ici, je vais juste vous montrer des fonctionnalités intéressantes, qui peuvent vous aider à vous lancer dans la création de votre propre modèle personnalisé pour le déploiement de l’application.

### <a name="parameters"></a>Paramètres
Accédez à la section des paramètres : vous pouvez voir que la plupart de ces paramètres correspondent aux informations que le bouton **Déployer dans Azure** vous invite à saisir. Le site derrière le bouton **Déployer dans Azure** remplit l’interface utilisateur en entrée, au moyen des paramètres définis dans le fichier azuredeploy.json. Ces paramètres sont utilisés dans les différentes définitions de ressources (noms de ressources, valeurs de propriétés, etc.).

### <a name="resources"></a>Ressources
Dans le nœud de ressources, vous pouvez voir que 4 ressources de niveau supérieur sont définies, ce qui inclut une instance SQL Server, un plan App Service et deux applications. 

#### <a name="app-service-plan"></a>Plan App Service
Commençons par une simple ressource de niveau racine dans le JSON. Dans le plan JSON, cliquez sur le plan App Service appelé **[hostingPlanName]** pour mettre en surbrillance le code JSON correspondant. 

![Montre la section [hostingPlanName] du code JSON.](./media/app-service-deploy-complex-application-predictably/examinejson-3-appserviceplan.png)

Notez que l’élément `type` spécifie la chaîne pour un plan App Service (appelé « ferme de serveurs » dans les temps anciens) ; d’autres éléments et propriétés sont remplis au moyen des paramètres définis dans le fichier JSON, et cette ressource n’inclut aucune ressource imbriquée.

> [!NOTE]
> Notez également la valeur de l’élément `apiVersion`, qui indique à Microsoft Azure la version de l’API REST avec laquelle utiliser la définition de ressource ; cela peut affecter le mode de format à appliquer à la ressource entre les `{}`. 
> 
> 

#### <a name="sql-server"></a>SQL Server
Ensuite, cliquez sur la ressource SQL Server appelée **SQLServer** dans le plan JSON.

![Montre la ressource SQL Server appelée SQLServer dans Structure JSON.](./media/app-service-deploy-complex-application-predictably/examinejson-4-sqlserver.png)

Notez les éléments suivants concernant le code JSON en surbrillance :

* L’utilisation de paramètres garantit que les ressources créées sont nommées et configurées de façon à être cohérentes les unes avec les autres.
* La ressource SQL Server inclut deux ressources imbriquées, qui présentent chacune une valeur différente pour l’élément `type`.
* Les ressources imbriquées dans l’élément `“resources”: […]`, dans lequel la base de données et les règles de pare-feu sont définies, incluent un élément `dependsOn` qui spécifie l’ID de la ressource SQL Server de niveau racine. Ce code indique au logiciel Microsoft Azure Resource Manager que l’autre ressource doit déjà exister avant qu’il ne crée cette ressource ; si cette autre ressource est définie dans le modèle, il doit la créer en premier.
  
  > [!NOTE]
  > Pour en savoir plus sur l’utilisation de la fonction `resourceId()`, voir [Azure Resource Manager Template Functions](../azure-resource-manager/templates/template-functions-resource.md#resourceid).
  > 
  > 
* Grâce à l’élément `dependsOn` , Azure Resource Manager peut savoir quelles ressources peuvent être créées en parallèle et quelles autres doivent être créées de façon séquentielle. 

#### <a name="app-service-app"></a>application App Service
À présent, passons aux applications elles-mêmes, qui sont plus complexes. Cliquez sur l’application [variables(‘nomSiteApi’)] dans le plan JSON pour mettre en évidence son code JSON. Vous pouvez constater que cela devient nettement plus intéressant. Dans ce contexte, j’aborderai les fonctionnalités une par une.

##### <a name="root-resource"></a>Ressource racine
L’application dépend de deux ressources différentes. Cela signifie que le logiciel Microsoft Azure Resource Manager génère l’application uniquement après la création du plan App Service et de l’instance SQL Server.

![Montre les dépendances de l’application dans le plan App Service et l’instance SQL Server.](./media/app-service-deploy-complex-application-predictably/examinejson-5-webapproot.png)

##### <a name="app-settings"></a>Paramètres de l’application
Les paramètres d’application sont également définis en tant que ressource imbriquée.

![Montre les paramètres de l’application définis sous forme de ressource imbriquée dans le code JSON.](./media/app-service-deploy-complex-application-predictably/examinejson-6-webappsettings.png)

Dans l’élément `properties` de `config/appsettings`, vous disposez de deux paramètres d’application au format `"<name>" : "<value>"`.

* `PROJECT` est un [paramètre KUDU](https://github.com/projectkudu/kudu/wiki/Customizing-deployments) qui indique au processus de déploiement de Microsoft Azure quel projet utiliser dans une solution Visual Studio à plusieurs projets. Plus tard, je vous montrerai comment le contrôle de code source est configuré. Toutefois, comme le code ToDoApp figure dans une solution Visual Studio à plusieurs projets, nous avons besoin de ce paramètre.
* `clientUrl` est simplement un paramètre d’application utilisé par le code d’application.

##### <a name="connection-strings"></a>Chaînes de connexion
Les chaînes de connexion sont également définies en tant que ressource imbriquée.

![Montre comment les chaînes de connexion sont définies en tant que ressource imbriquée dans le code JSON.](./media/app-service-deploy-complex-application-predictably/examinejson-7-webappconnstr.png)

Dans l’élément `properties` relatif à `config/connectionstrings`, chaque chaîne de connexion est définie comme une paire nom:valeur, avec le format spécifique de l’élément `"<name>" : {"value": "…", "type": "…"}`. Pour l’élément `type`, les valeurs possibles sont les suivantes : `MySql`, `SQLServer`, `SQLAzure` et `Custom`.

> [!TIP]
> Pour obtenir la liste définitive des types de chaîne de connexion, exécutez la commande suivante dans Azure PowerShell : \[Enum]::GetNames("Microsoft.WindowsAzure.Commands.Utilities.Websites.Services.WebEntities.DatabaseType")
> 
> 

##### <a name="source-control"></a>Contrôle de code source
Les paramètres de contrôle du code source sont également définis en tant que ressource imbriquée. Azure Resource Manager utilise cette ressource pour configurer la publication en continu (nous verrons les inconvénients liés à `IsManualIntegration` plus tard) et pour lancer le déploiement automatique du code d’application lors du traitement du fichier JSON.

![Montre comment les paramètres de contrôle de code source sont définis en tant que ressource imbriquée dans le code JSON.](./media/app-service-deploy-complex-application-predictably/examinejson-8-webappsourcecontrol.png)

Les éléments `RepoUrl` et `branch` doivent être assez intuitifs et doivent pointer vers le référentiel Git et le nom de la branche à partir de laquelle effectuer la publication. Là encore, ces éléments sont définis par les paramètres d’entrée. 

Remarque : dans l’élément `dependsOn`, en plus de la ressource d’application elle-même, `sourcecontrols/web` dépend également des éléments `config/appsettings` et `config/connectionstrings`. En effet, une fois que l’élément `sourcecontrols/web` est configuré, le processus de déploiement de Microsoft Azure tente automatiquement de déployer, de créer et de démarrer le code d’application. Par conséquent, l’insertion de cette dépendance permet de s’assurer que l’application a accès aux paramètres d’application et chaînes de connexion requis avant l’exécution du code d’application. 

> [!NOTE]
> Vous pouvez également voir que le paramètre `IsManualIntegration` est défini sur `true`. Cette propriété est nécessaire dans ce didacticiel parce que vous n’avez pas de dépôt GitHub et, par conséquent, vous ne pouvez pas accorder d’autorisations à Azure pour configurer la publication continue à partir de [ToDoApp](https://github.com/azure-appservice-samples/ToDoApp) (par exemple, envoyer (push) des mises à jour automatiques de dépôt à Azure). Vous pouvez utiliser la valeur par défaut `false` pour le référentiel spécifié uniquement si vous avez préalablement configuré les informations d’identification GitHub du propriétaire dans le [portail Azure](https://portal.azure.com/) . En d’autres termes, si vous avez configuré au préalable le contrôle de code source dans GitHub ou BitBucket pour toute application dans le [portail Azure](https://portal.azure.com/) , en utilisant vos informations d’identification, Azure se rappelle les informations d’identification et les utilise chaque fois que vous déployez une application à partir de GitHub ou BitBucket. Toutefois, si cette opération n’est pas déjà effectuée, le déploiement du modèle JSON échoue lorsque le logiciel Microsoft Azure Resource Manager tente de configurer les paramètres de contrôle du code source de l’application, car il ne peut pas se connecter à GitHub ou BitBucket en utilisant les informations d’identification du propriétaire du référentiel.
> 
> 

## <a name="compare-the-json-template-with-deployed-resource-group"></a>Comparer le modèle JSON avec un groupe de ressources déployées
Ici, vous pouvez faire défiler tous les panneaux de l’application dans le [portail Azure](https://portal.azure.com/), mais il existe un autre outil tout aussi utile, sinon plus. Accédez à l’outil de prévisualisation [Azure Resource Explorer](https://resources.azure.com) , qui permet une représentation JSON de tous les groupes de ressources associés à vos abonnements, tels qu’ils existent sur le système Azure principal. Vous pouvez également voir en quoi la hiérarchie JSON du groupe de ressources dans Microsoft Azure correspond à celle du fichier du modèle utilisé pour la créer.

Par exemple, lorsque j’accède à l’outil [Azure Resource Explorer](https://resources.azure.com) et développe les nœuds dans l’explorateur, je peux afficher le groupe de ressources et les ressources de niveau racine qui sont collectés sous leur type de ressource respectif.

![Montre le groupe de ressources et les ressources de niveau racine dans l’outil développé Azure Resources Explorer.](./media/app-service-deploy-complex-application-predictably/ARM-1-treeview.png)

Si vous explorez une application, vous devez voir apparaître les informations sur la configuration de l’application, de manière similaire à ce qu’indique la capture d’écran ci-dessous :

![Descendez dans la hiérarchie pour afficher les détails de configuration de l’application.](./media/app-service-deploy-complex-application-predictably/ARM-2-jsonview.png)

Là encore, les ressources imbriquées doivent présenter une hiérarchie très similaire à celles du modèle de fichier JSON, et vous devez voir apparaître les paramètres de l’application, les chaînes de connexion, etc. reflétés de manière correcte dans le volet JSON. L’absence de paramètres à ce niveau peut signaler un problème touchant votre fichier JSON et peut vous aider à dépanner votre fichier de modèle JSON.

## <a name="deploy-the-resource-group-template-yourself"></a>Déployer un modèle de groupe de ressources soi-même
Le bouton **Déployer dans Azure** est très utile, mais il vous permet uniquement de déployer le modèle de groupe de ressources dans azuredeploy.json si vous avez déjà envoyé ce dernier à GitHub. Le Kit de développement logiciel (SDK) Microsoft Azure .NET fournit également des outils vous permettant de déployer un fichier de modèle JSON directement à partir de votre ordinateur local. Pour ce faire, procédez comme suit :

1. Dans Visual Studio, cliquez sur **Fichier** > **Nouveau** > **Projet**.
2. Cliquez sur **Visual C#**  > **Cloud** > **Groupe de ressources Azure**, puis cliquez sur **OK**.
   
   ![Créez un projet en tant que groupe de ressources Azure dans le SDK .NET Azure.](./media/app-service-deploy-complex-application-predictably/deploy-1-vsproject.png)
3. Dans **Sélectionner un modèle Azure**, choisissez **Modèle vide** et cliquez sur **OK**.
4. Faites glisser le fichier azuredeploy.json vers le dossier **Modèle** de votre nouveau projet.
   
   ![Montre le résultat du glissement du fichier azuredeploy.json dans le dossier de modèles de votre projet.](./media/app-service-deploy-complex-application-predictably/deploy-2-copyjson.png)
5. Dans l’Explorateur de solutions, ouvrez le fichier azuredeploy.json copié.
6. Pour les besoins de notre démonstration, ajoutons quelques ressources Application Insights standard dans notre fichier JSON, en cliquant sur **Ajouter une ressource**. Si vous êtes uniquement intéressé par le déploiement du fichier JSON, passez directement aux étapes de déploiement.
   
   ![Montre le bouton Ajouter une ressource qui vous permet d’ajouter des ressources Application Insight standard à votre fichier JSON.](./media/app-service-deploy-complex-application-predictably/deploy-3-newresource.png)
7. Sélectionnez **Application Insights pour les applications web**, puis assurez-vous qu’un plan App Service existant et une application sont sélectionnés, puis cliquez sur **Ajouter**.
   
   ![Montre la sélection de Application Insights pour les applications web, Nom, Plan App Service et Application web.](./media/app-service-deploy-complex-application-predictably/deploy-4-newappinsight.png)
   
   À présent, vous pouvez afficher plusieurs nouvelles ressources qui, selon la ressource et son action, présentent des dépendances avec le plan App Service ou l’application. Ces ressources ne sont pas activées par leur définition existante. Vous allez changer cela.
   
   ![Montre les nouvelles ressources qui ont des dépendances dans le plan App Service ou l’application.](./media/app-service-deploy-complex-application-predictably/deploy-5-appinsightresources.png)
8. Dans le plan JSON, cliquez sur **appInsights AutoScale** pour mettre en évidence son code JSON. Il s’agit du paramètre de mise à l’échelle de votre plan App Service.
9. Dans le code JSON mis en surbrillance, localisez les propriétés `location` et `enabled`, et définissez-les comme indiqué ci-dessous.
   
   ![Montre l’emplacement et les propriétés activées dans le code JSON appInsights AutoScale et les valeurs que vous devez leur attribuer.](./media/app-service-deploy-complex-application-predictably/deploy-6-autoscalesettings.png)
10. Dans le plan JSON, cliquez sur **CPUHigh appInsights** pour mettre en évidence son code JSON. Il s’agit d’une alerte.
11. Localisez les propriétés `location` et `isEnabled`, et définissez-les comme indiqué ci-dessous. Faites de même pour les autres trois alertes (ampoules violettes).
    
    ![Montre l’emplacement et les propriétés isEnabled dans le code JSON CPUHigh appInsights et les valeurs que vous devez leur attribuer.](./media/app-service-deploy-complex-application-predictably/deploy-7-alerts.png)
12. À présent, vous êtes prêt à effectuer le déploiement. Cliquez avec le bouton droit de la souris sur le projet et sélectionnez **Déployer** > **New Déployerment**.
    
    ![Montre comment déployer votre nouveau projet.](./media/app-service-deploy-complex-application-predictably/deploy-8-newdeployment.png)
13. Le cas échéant, connectez-vous à votre compte Microsoft Azure.
14. Sélectionnez un groupe de ressources dans votre abonnement ou créez-en un nouveau, sélectionnez **azuredeploy.json**, puis cliquez sur **Modifier les paramètres**.
    
    ![Montre comment modifier les paramètres dans le fichier azuredeploy.json.](./media/app-service-deploy-complex-application-predictably/deploy-9-deployconfig.png)
    
    Vous pouvez maintenant modifier tous les paramètres définis dans le fichier de modèle dans une table complète. Les paramètres qui définissent des valeurs par défaut présentent déjà des valeurs par défaut, et ceux qui définissent une liste de valeurs autorisées s’affichent en tant que listes déroulantes.
    
    ![Montre les paramètres qui définissent une liste de valeurs autorisées sous forme de listes déroulantes.](./media/app-service-deploy-complex-application-predictably/deploy-10-parametereditor.png)
15. Renseignez tous les paramètres vides et utilisez l’ [adresse du référentiel GitHub pour ToDoApp](https://github.com/azure-appservice-samples/ToDoApp.git) dans **repoUrl**. Ensuite, cliquez sur **Enregistrer**.
    
    ![Montre les paramètres nouvellement complétés pour le fichier azuredeploy.json.](./media/app-service-deploy-complex-application-predictably/deploy-11-parametereditorfilled.png)
    
    > [!NOTE]
    > La fonction de mise à l’échelle automatique est proposée au niveau **Standard** ou plus, et les alertes au niveau du plan sont proposées au niveau **Basic** ou plus. Vous devrez définir le paramètre **sku** sur **Standard** ou **Premium** afin de mettre en surbrillance toutes vos nouvelles ressources App Insights.
    > 
    > 
16. Cliquez sur **Déployer**. Si vous avez sélectionné l’option **Enregistrer les mots de passe**, le mot de passe est enregistré dans le fichier de paramètres, **en texte brut**. Dans le cas contraire, vous êtes invité à saisir le mot de passe de la base de données lors du processus de déploiement.

Et voilà ! Maintenant, il vous suffit d’accéder au [portail Azure](https://portal.azure.com/) et à l’outil [Explorateur de ressources Azure](https://resources.azure.com) pour afficher les nouvelles alertes et les paramètres de mise à l’échelle automatique ajoutés à votre application JSON déployée.

Les étapes de cette section ont permis d’accomplir les tâches principales suivantes :

1. Préparation du fichier de modèle
2. Création d’un fichier de paramètres pour accompagner le fichier de modèle
3. Déploiement du fichier de modèle avec le fichier de paramètres

Cette dernière étape est facile à accomplir via l’applet de commande PowerShell. Pour voir les actions effectuées par Visual Studio lors du déploiement de votre application, accédez à l’élément Scripts\Deploy-AzureResourceGroup.ps1. Il y a beaucoup de code, mais je vais mettre en évidence celui que vous devez écrire pour déployer le fichier de modèle avec le fichier de paramètres.

![Montre le code pertinent dans le script que vous devez utiliser pour déployer le fichier de modèle avec le fichier de paramètres.](./media/app-service-deploy-complex-application-predictably/deploy-12-powershellsnippet.png)

La dernière applet de commande, `New-AzureResourceGroup`, est celle qui effectue réellement l’action. Tout cela doit indiquer que, grâce aux outils, il est relativement simple de déployer votre application Cloud de manière prévisible. Chaque fois que vous exécutez l’applet de commande sur le même modèle, avec le même fichier de paramètres, vous obtenez le même résultat.

## <a name="summary"></a>Résumé
Dans DevOps, la répétition et la prévisibilité jouent un rôle majeur pour assurer la réussite du déploiement à grande échelle d'une application composée de microservices. Dans ce didacticiel, vous avez déployé deux applications de microservices sur Microsoft Azure en tant que groupe de ressources unique, à l'aide du modèle Azure Resource Manager. J'espère qu'il vous a permis d'acquérir les connaissances requises pour convertir une application en modèle, dans Microsoft Azure, et de la mettre en service et la déployer de manière prévisible sur cette plate-forme. 

<a name="resources"></a>

## <a name="more-resources"></a>Plus de ressources
* [Langue du modèle Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md)
* [Création de modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md)
* [Fonctions des modèles Azure Resource Manager](../azure-resource-manager/templates/template-functions.md)
* [Déploiement d’une application avec un modèle Azure Resource Manager](../azure-resource-manager/templates/deploy-powershell.md)
* [Utilisation d'Azure PowerShell avec Azure Resource Manager](../azure-resource-manager/management/manage-resources-powershell.md)
* [Résolution des problèmes liés aux déploiements de groupes de ressources dans Azure](../azure-resource-manager/templates/common-deployment-errors.md)

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la syntaxe et les propriétés JSON pour les types de ressources déployés dans cet article, voir :

* [Microsoft.Sql/servers](/azure/templates/microsoft.sql/servers)
* [Microsoft.Sql/servers/databases](/azure/templates/microsoft.sql/servers/databases)
* [Microsoft.Sql/servers/firewallRules](/azure/templates/microsoft.sql/servers/firewallrules)
* [Microsoft.Web/serverfarms](/azure/templates/microsoft.web/serverfarms)
* [Microsoft.Web/sites](/azure/templates/microsoft.web/sites)
* [Microsoft.Web/sites/slots](/azure/templates/microsoft.web/sites/slots)
* [Microsoft.Insights/autoscalesettings](/azure/templates/microsoft.insights/autoscalesettings)