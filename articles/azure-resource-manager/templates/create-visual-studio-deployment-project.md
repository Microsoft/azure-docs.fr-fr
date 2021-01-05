---
title: Créer et déployer des projets de groupe de ressources avec Visual Studio
description: Utilisez Visual Studio pour créer un projet de groupe de ressources Azure et déployer les ressources dans Azure.
ms.topic: conceptual
ms.date: 10/16/2019
ms.openlocfilehash: 479dbed1f288148c24fc8464f7895cd3e2b182f9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91372644"
---
# <a name="creating-and-deploying-azure-resource-groups-through-visual-studio"></a>Création et déploiement de groupes de ressources Azure à l’aide de Visual Studio

Avec Visual Studio, vous pouvez créer un projet qui déploie votre infrastructure et votre code sur Azure. Par exemple, vous pouvez déployer l’hôte web, le site web et le code pour le site web. Visual Studio fournit de nombreux modèles de démarrage différents pour déployer des scénarios courants. Dans cet article, vous déployez une application web.

Cet article explique comment utiliser [Visual Studio 2019 ou ultérieur avec les charges de travail Développement Azure et ASP.NET installées](/visualstudio/install/install-visual-studio?view=vs-2019). Si vous utilisez Visual Studio 2017, votre expérience sera en grande partie la même.

## <a name="create-azure-resource-group-project"></a>Créer un projet de groupe de ressources Azure

Dans cette section, vous créez un projet de groupe de ressources Azure avec un modèle **Application web**.

1. Dans Visual Studio, choisissez **Fichier**>**Nouveau**>**Projet**.
1. Sélectionnez le modèle de projet **Groupe de ressources Azure**, puis **Suivant**.

    ![Capture d’écran montrant la fenêtre Créer un projet avec un groupe de ressources Azure et le bouton Suivant mis en surbrillance.](./media/create-visual-studio-deployment-project/create-project.png)

1. Donnez un nom à votre projet. Les autres paramètres par défaut conviennent probablement, mais passez-les en revue pour vérifier qu’ils fonctionnent pour votre environnement. Quand vous avez terminé, sélectionnez **Créer**.

    ![Créer un projet](./media/create-visual-studio-deployment-project/name-project.png)

1. Choisissez le modèle à déployer sur Azure Resource Manager. Notez qu’il existe de nombreuses options différentes selon le type de projet que vous voulez déployer. Pour cet article, choisissez le modèle **Application web**, puis cliquez sur **OK**.

    ![Sélectionner un modèle](./media/create-visual-studio-deployment-project/select-project.png)

    Le modèle que vous choisissez est seulement un point de départ ; vous pouvez ajouter et supprimer des ressources en fonction des besoins de votre scénario.

1. Visual Studio crée un projet de déploiement de groupe de ressources pour l’application web. Pour voir les fichiers de votre projet, examinez le nœud du projet de déploiement.

    ![Afficher les nœuds](./media/create-visual-studio-deployment-project/show-items.png)

    Comme vous avez choisi le modèle Application web, vous voyez les fichiers suivants :

   | Nom de fichier | Description |
   | --- | --- |
   | Deploy-AzureResourceGroup.ps1 |Script PowerShell qui exécute des commandes PowerShell à déployer sur Azure Resource Manager. Visual Studio utilise ce script PowerShell pour déployer votre modèle. |
   | WebSite.json |Le modèle Resource Manager qui définit l’infrastructure que vous voulez déployer sur Azure, et les paramètres que vous pouvez fournir au cours du déploiement. Il définit également les dépendances entre les ressources de manière à ce que Resource Manager les déploie dans le bon ordre. |
   | WebSite.parameters.json |Un fichier de paramètres qui contient les valeurs requises par le modèle. Vous transmettez des valeurs de paramètres pour personnaliser chaque déploiement. |

    Tous les projets de déploiement de groupe de ressources Azure contiennent ces fichiers de base. D'autres projets peuvent contenir des fichiers supplémentaires pour prendre en charge d'autres fonctionnalités.

## <a name="customize-resource-manager-template"></a>Personnaliser le modèle Resource Manager

Vous pouvez personnaliser un projet de déploiement en modifiant le modèle Resource Manager qui décrit les ressources que vous voulez déployer. Pour en savoir plus sur les éléments du modèle Resource Manager, consultez [Création de modèles Azure Resource Manager](template-syntax.md).

1. Pour travailler sur votre modèle, ouvrez **WebSite.json**.

1. L’éditeur Visual Studio fournit des outils pour vous aider à modifier le modèle Resource Manager. La fenêtre **Structure JSON** permet de voir facilement les éléments définis dans votre modèle.

   ![Afficher la structure JSON](./media/create-visual-studio-deployment-project/show-json-outline.png)

1. Sélectionnez un élément dans la structure pour accéder à cette partie du modèle.

   ![Explorer le JSON](./media/create-visual-studio-deployment-project/navigate-json.png)

1. Vous pouvez ajouter une ressource en sélectionnant le bouton **Ajouter une ressource** en haut de la fenêtre Structure JSON, ou en double-cliquant sur **Ressources** et en sélectionnant **Ajouter une nouvelle ressource**.

   ![Capture d’écran montrant la fenêtre Structure JSON avec l’option Ajouter une nouvelle ressource mise en surbrillance.](./media/create-visual-studio-deployment-project/add-resource.png)

1. Sélectionnez **Compte de stockage** et donnez-lui un nom. Choisissez un nom qui ne contient pas plus de 11 caractères et uniquement des chiffres et des lettres minuscules.

   ![Ajouter du stockage](./media/create-visual-studio-deployment-project/add-storage.png)

1. Notez qu’en plus de la ressource, un paramètre pour le compte de stockage de type et une variable pour le nom du compte de stockage ont été ajoutés.

   ![Afficher la structure](./media/create-visual-studio-deployment-project/show-new-items.png)

1. Le paramètre pour le type de compte de stockage est prédéfini avec des types autorisés et un type par défaut. Vous pouvez laisser ces valeurs ou les modifier pour votre scénario. Si vous ne souhaitez autoriser une personne à déployer un compte de stockage **Premium_LRS** via ce modèle, supprimez ce dernier des types autorisés.

   ```json
   "demoaccountType": {
     "type": "string",
     "defaultValue": "Standard_LRS",
     "allowedValues": [
       "Standard_LRS",
       "Standard_ZRS",
       "Standard_GRS",
       "Standard_RAGRS"
     ]
   }
   ```

1. Visual Studio fournit également des fonctionnalités IntelliSense pour vous aider à comprendre les propriétés qui sont disponibles lors de la modification du modèle. Par exemple, pour modifier les propriétés de votre plan App Service, accédez à la ressource **HostingPlan** et ajoutez une valeur pour les **propriétés**. Notez qu’intellisense affiche les valeurs disponibles et fournit une description de cette valeur.

   ![Afficher IntelliSense](./media/create-visual-studio-deployment-project/show-intellisense.png)

   Vous pouvez définir **numberOfWorkers** sur 1 et enregistrer le fichier.

   ```json
   "properties": {
     "name": "[parameters('hostingPlanName')]",
     "numberOfWorkers": 1
   }
   ```

1. Ouvrez le fichier **WebSite.parameters.json**. Vous utilisez le fichier de paramètres pour passer des valeurs lors du déploiement qui personnalisent la ressource en cours de déploiement. Nommez le plan d’hébergement et enregistrez le fichier.

   ```json
   {
     "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
       "hostingPlanName": {
         "value": "demoHostPlan"
       }
     }
   }
   ```

## <a name="deploy-project-to-azure"></a>Déployer le projet sur Azure

Vous êtes maintenant prêt à déployer votre projet sur un groupe de ressources.

Par défaut, le script PowerShell (Deploy-AzureResourceGroup.ps1) du projet utilise le module AzureRM. Si le module AzureRM est encore installé sur votre ordinateur et que vous voulez continuer à l’utiliser, vous pouvez utiliser ce script par défaut. Avec ce script, vous pouvez utiliser l’interface de Visual Studio pour déployer votre solution.

Cependant, si vous avez migré vers le nouveau [module Az](/powershell/azure/new-azureps-module-az), vous devez ajouter un nouveau script à votre projet. Pour ajouter un script qui utilise le module Az, copiez le script [Deploy-AzTemplate.ps1](https://github.com/Azure/azure-quickstart-templates/blob/master/Deploy-AzTemplate.ps1) et ajoutez-le à votre projet. Pour utiliser ce script pour le déploiement, vous devez l’exécuter à partir d’une console PowerShell, au lieu d’utiliser l’interface de déploiement de Visual Studio.

Les deux approches sont présentées dans cet article. Cet article désigne le script par défaut sous le nom de « script du module AzureRM » et le nouveau script sous le nom de « script du module Az ».

### <a name="az-module-script"></a>Script du module Az

Pour le script du module Az, ouvrez une console PowerShell et exécutez :

```powershell
.\Deploy-AzTemplate.ps1 -ArtifactStagingDirectory . -Location centralus -TemplateFile WebSite.json -TemplateParametersFile WebSite.parameters.json
```

### <a name="azurerm-module-script"></a>Script du module AzureRM

Pour le script du module AzureRM, utilisez Visual Studio :

1. Dans le menu contextuel du nœud du projet de déploiement, choisissez **Déployer** > **Nouveau**.

    ![Élément de menu Nouveau déploiement](./media/create-visual-studio-deployment-project/deploy.png)

1. La boîte de dialogue **Déployer vers le groupe de ressources** s’affiche. Dans la zone de liste déroulante **Groupe de ressources**, sélectionnez un groupe de ressources existant ou créez-en un. Sélectionnez **Déployer**.

    ![Boîte de dialogue Déployer vers le groupe de ressources](./media/create-visual-studio-deployment-project/show-deployment.png)

1. La fenêtre **Sortie** indique l’état du déploiement. Lorsque le déploiement est terminé, le dernier message indique que le déploiement a été réalisé avec succès :

   ```output
   18:00:58 - Successfully deployed template 'website.json' to resource group 'ExampleAppDeploy'.
   ```

## <a name="view-deployed-resources"></a>Voir les ressources déployées

Vérifions les résultats.

1. Dans un navigateur, ouvrez le [portail Azure](https://portal.azure.com/) et connectez-vous à votre compte. Pour afficher le groupe de ressources, sélectionnez **Groupes de ressources** et le groupe de ressources vers lequel vous avez effectué le déploiement.

1. Vous verrez toutes les ressources déployées. Notez que le nom du compte de stockage n’est pas exactement celui spécifié lors de l’ajout de cette ressource. Le compte de stockage doit être unique. Le modèle ajoute automatiquement une chaîne de caractères au nom que vous avez spécifié pour créer un nom unique.

    ![Afficher les ressources](./media/create-visual-studio-deployment-project/show-deployed-resources.png)

## <a name="add-code-to-project"></a>Ajouter du code au projet

À ce stade, vous avez déployé l’infrastructure de votre application, mais aucun code réel n’est déployé avec le projet.

1. Ajoutez un projet à votre solution Visual Studio. Cliquez avec le bouton droit sur la solution et sélectionnez **Ajouter** > **Nouveau projet**.

    ![Ajouter un projet](./media/create-visual-studio-deployment-project/add-project.png)

1. Ajoutez une **Application web ASP.NET Core**.

    ![Ajouter une application web](./media/create-visual-studio-deployment-project/add-app.png)

1. Donnez un nom à votre application web, puis sélectionnez **Créer**.

    ![Nommer l’application web](./media/create-visual-studio-deployment-project/name-web-app.png)

1. Sélectionnez **Application web**, puis **Créer**.

    ![Sélectionner une application web](./media/create-visual-studio-deployment-project/select-project-type.png)

1. Une fois votre application web créée par Visual Studio, vous voyez les deux projets dans la solution.

    ![Afficher les projets](./media/create-visual-studio-deployment-project/show-projects.png)

1. À présent, vous devez vous assurer que votre projet de groupe de ressources est informé du nouveau projet. Revenez à votre projet de groupe de ressources (ExampleAppDeploy). Cliquez avec le bouton droit sur **Références** et sélectionnez **Ajouter une référence**.

    ![Capture d’écran montrant le menu ExampleAppDeploy avec l’option Ajouter une référence mise en surbrillance.](./media/create-visual-studio-deployment-project/add-new-reference.png)

1. Sélectionnez le projet d’application web que vous avez créé.

   ![Ajouter la référence](./media/create-visual-studio-deployment-project/add-reference.png)

   En ajoutant une référence, vous liez le projet d’application web au projet de groupe de ressources, et vous définissez automatiquement trois propriétés. Vous voyez ces propriétés dans la fenêtre **Propriétés** de la référence. **Inclure le chemin d’accès au fichier** contient le chemin d’accès à l’emplacement de création du package. Notez le dossier (ExampleApp) et le fichier (package.zip). Vous devez connaître ces valeurs, car elles doivent être fournies en tant que paramètres lors du déploiement de l’application.

   ![Voir une référence](./media/create-visual-studio-deployment-project/see-reference.png)

1. Revenez à votre modèle (WebSite.json) et ajoutez une ressource au modèle.

    ![Ajouter une ressource](./media/create-visual-studio-deployment-project/add-resource-2.png)

1. Cette fois-ci, sélectionnez **Web Deploy pour les applications Web**.

    ![Ajouter un déploiement web](./media/create-visual-studio-deployment-project/add-web-deploy.png)

   Enregistrez votre modèle.

1. Il y a de nouveaux paramètres dans votre modèle. Ils ont été ajoutés à l’étape précédente. Vous n’avez pas besoin de fournir des valeurs pour **_artifactsLocation** ou **_artifactsLocationSasToken**, car ces valeurs sont générées automatiquement. Cependant, vous devez définir le dossier et le nom du fichier sur le chemin qui contient le package de déploiement. Les noms de ces paramètres se terminent par **PackageFolder** et **PackageFileName**. La première partie du nom est le nom de la ressource Web Deploy que vous avez ajoutée. Dans cet article, ils sont nommés **ExampleAppPackageFolder** et **ExampleAppPackageFileName**.

   Ouvrez **Website.parameters.json** et définissez ces paramètres sur les valeurs que vous avez vues dans les propriétés de référence. Définissez **ExampleAppPackageFolder** sur le nom du dossier. Définissez **ExampleAppPackageFileName** sur le nom du fichier zip.

   ```json
   {
     "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
       "hostingPlanName": {
         "value": "demoHostPlan"
       },
       "ExampleAppPackageFolder": {
         "value": "ExampleApp"
       },
       "ExampleAppPackageFileName": {
         "value": "package.zip"
       }
     }
   }
   ```

## <a name="deploy-code-with-infrastructure"></a>Déployer le code avec l’infrastructure

Comme vous avez ajouté du code au projet, votre déploiement est cette fois un peu différent. Au cours du déploiement, vous placez en zone intermédiaire les artefacts de votre projet à un emplacement accessible par Resource Manager. Les artefacts sont placés en zone intermédiaire dans un compte de stockage.

### <a name="az-module-script"></a>Script du module Az

Vous devez apporter une petite modification à votre modèle si vous utilisez le script du module Az. Ce script ajoute une barre oblique à l’emplacement des artefacts, mais votre modèle n’attend pas cette barre oblique. Ouvrez WebSite.json et recherchez les propriétés de l’extension MSDeploy. Elle a une propriété nommée **packageUri**. Supprimez la barre oblique entre l’emplacement des artefacts et le dossier du package.

Il doit se présenter comme ceci :

```json
"packageUri": "[concat(parameters('_artifactsLocation'), parameters('ExampleAppPackageFolder'), '/', parameters('ExampleAppPackageFileName'), parameters('_artifactsLocationSasToken'))]",
```

Notez que dans l’exemple précédent, il n’existe pas de `'/',` entre **parameters('_artifactsLocation')** et **parameters('ExampleAppPackageFolder')** .

Regénérez le projet. Générer le projet permet de garantir que les fichiers que vous devez déployer sont ajoutés au dossier intermédiaire.

Ouvrez maintenant une console PowerShell et exécutez :

```powershell
.\Deploy-AzTemplate.ps1 -ArtifactStagingDirectory .\bin\Debug\staging\ExampleAppDeploy -Location centralus -TemplateFile WebSite.json -TemplateParametersFile WebSite.parameters.json -UploadArtifacts -StorageAccountName <storage-account-name>
```

### <a name="azurerm-module-script"></a>Script du module AzureRM

Pour le script du module AzureRM, utilisez Visual Studio :

1. Pour redéployer, choisissez **Déployer** et le groupe de ressources que vous avez déployé précédemment.

    ![Redéployer le projet](./media/create-visual-studio-deployment-project/redeploy.png)

1. Sélectionnez le compte de stockage que vous avez déployé avec ce groupe de ressources pour le **Compte de stockage des artefacts**.

   ![Redéployer un déploiement web](./media/create-visual-studio-deployment-project/redeploy-web-app.png)

## <a name="view-web-app"></a>Voir l’application web

1. Une fois le déploiement terminé, sélectionnez votre application web dans le portail. Sélectionnez l’URL pour accéder au site.

   ![Parcourir le site](./media/create-visual-studio-deployment-project/browse-site.png)

1. Notez que vous avez déployé avec succès l’application ASP.NET par défaut.

   ![Afficher l’application déployée](./media/create-visual-studio-deployment-project/show-deployed-app.png)

## <a name="add-operations-dashboard"></a>Ajouter le tableau de bord des opérations

Vous n’êtes pas limité aux seules ressources qui sont disponibles via l’interface de Visual Studio. Vous pouvez personnaliser votre déploiement en ajoutant une ressource personnalisée à votre modèle. Pour afficher l’ajout d’une ressource, vous ajoutez un tableau de bord opérationnel pour gérer la ressource déployée.

1. Ouvrez le fichier WebSite.json et ajoutez le code JSON suivant après la ressource de compte de stockage, mais avant le `]` de fermeture de la section des ressources.

   ```json
    ,{
      "properties": {
        "lenses": {
          "0": {
            "order": 0,
            "parts": {
              "0": {
                "position": {
                  "x": 0,
                  "y": 0,
                  "colSpan": 4,
                  "rowSpan": 6
                },
                "metadata": {
                  "inputs": [
                    {
                      "name": "resourceGroup",
                      "isOptional": true
                    },
                    {
                      "name": "id",
                      "value": "[resourceGroup().id]",
                      "isOptional": true
                    }
                  ],
                  "type": "Extension/HubsExtension/PartType/ResourceGroupMapPinnedPart"
                }
              },
              "1": {
                "position": {
                  "x": 4,
                  "y": 0,
                  "rowSpan": 3,
                  "colSpan": 4
                },
                "metadata": {
                  "inputs": [],
                  "type": "Extension[azure]/HubsExtension/PartType/MarkdownPart",
                  "settings": {
                    "content": {
                      "settings": {
                        "content": "__Customizations__\n\nUse this dashboard to create and share the operational views of services critical to the application performing. To customize simply pin components to the dashboard and then publish when you're done. Others will see your changes when you publish and share the dashboard.\n\nYou can customize this text too. It supports plain text, __Markdown__, and even limited HTML like images <img width='10' src='https://portal.azure.com/favicon.ico'/> and <a href='https://azure.microsoft.com' target='_blank'>links</a> that open in a new tab.\n",
                        "title": "Operations",
                        "subtitle": "[resourceGroup().name]"
                      }
                    }
                  }
                }
              }
            }
          }
        },
        "metadata": {
          "model": {
            "timeRange": {
              "value": {
                "relative": {
                  "duration": 24,
                  "timeUnit": 1
                }
              },
              "type": "MsPortalFx.Composition.Configuration.ValueTypes.TimeRange"
            }
          }
        }
      },
      "type": "Microsoft.Portal/dashboards",
      "apiVersion": "2015-08-01-preview",
      "name": "[concat('ARM-',resourceGroup().name)]",
      "location": "[resourceGroup().location]",
      "tags": {
        "hidden-title": "[concat('OPS-',resourceGroup().name)]"
      }
    }
   ```

1. Redéployez votre projet.

1. Une fois le déploiement terminé, regardez votre tableau de bord dans le portail. Sélectionnez **Tableau de bord** et choisissez celui que vous avez déployé.

   ![Capture d’écran montrant la page Tableau de bord avec un exemple de tableau de bord personnalisé mis en surbrillance.](./media/create-visual-studio-deployment-project/view-custom-dashboards.png)

1. Vous voyez le tableau de bord personnalisé.

   ![Tableau de bord personnalisé](./media/create-visual-studio-deployment-project/Ops-DemoSiteGroup-dashboard.png)

Vous pouvez gérer l’accès au tableau de bord à l’aide du contrôle d’accès en fonction du rôle Azure (RBAC Azure). Vous pouvez également personnaliser l’apparence du tableau de bord après son déploiement. Toutefois, si vous redéployez le groupe de ressources, le tableau de bord conserve son état par défaut dans votre modèle. Pour plus d’informations sur la création de tableaux de bord, consultez [Créer par programmation des tableaux de bord Azure](../../azure-portal/azure-portal-dashboards-create-programmatically.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, nettoyez les ressources Azure que vous avez déployées en supprimant le groupe de ressources.

1. Dans le portail Azure, sélectionnez **Groupes de ressources** dans le menu de gauche.

1. Sélectionnez le nom du groupe de ressources.

1. Sélectionnez **Supprimer le groupe de ressources** dans le menu supérieur.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert comment créer et déployer des modèles avec Visual Studio. Pour en savoir plus sur le développement de modèles, consultez notre nouvelle série de tutoriels pour les débutants :

> [!div class="nextstepaction"]
> [Tutoriels pour les débutants](./template-tutorial-create-first-template.md)
