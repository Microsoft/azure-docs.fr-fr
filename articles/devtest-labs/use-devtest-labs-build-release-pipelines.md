---
title: Utiliser DevTest Labs dans les pipelines de build et de mise en production Azure Pipelines
description: Découvrez comment utiliser Azure DevTest Labs dans les pipelines de build et de mise en production Azure.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: d04ed5dd7bebac0c8f24deb9145c3d2e4b77122e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88080332"
---
# <a name="use-devtest-labs-in-azure-pipelines-build-and-release-pipelines"></a>Utiliser DevTest Labs dans les pipelines de build et de mise en production Azure Pipelines
Cet article fournit des informations sur la façon dont DevTest Labs peut être utilisé dans les pipelines de build et de mise en production Azure. 

## <a name="overall-flow"></a>Flux général
Le flux de base consiste à disposer d’un **pipeline de build** qui effectue les tâches suivantes :

1. Générez le code de l’application.
1. Créez l’environnement de base dans DevTest Labs.
1. Mettez à jour l’environnement avec des informations personnalisées.
1. Déployer l’application dans l’environnement DevTest Labs
1. Testez le code. 

Une fois la build terminée, le **pipeline de mise en production** utilisera les artefacts de build pour déployer la mise en lots ou la production. 

L’un des principes nécessaires est que toutes les informations nécessaires à la recréation de l’écosystème testé sont disponibles dans les artefacts de build, y compris la configuration des ressources Azure. Comme les ressources Azure entraînent un coût en cas d’utilisation, les entreprises souhaitent contrôler ou suivre l’utilisation de ces ressources. Dans certains cas, les modèles Azure Resource Manager utilisés pour créer et configurer les ressources peuvent être gérés par un autre service que le département informatique. Et, ces modèles peuvent être stockés dans un référentiel différent. Cela résulte en une situation intéressante dans laquelle une build est créée et testée, et le code et la configuration doivent tous deux être stockés dans les artefacts de build pour recréer correctement le système en production. 

En utilisant DevTest Labs pendant la phase de génération et de test, vous pouvez ajouter des modèles Azure Resource Manager et des fichiers de prise en charge aux sources de génération afin que, pendant la phase de mise en production, la configuration exacte utilisée pour le test soit déployée en production. La tâche **Créer un environnement Azure DevTest Labs** avec la configuration appropriée permet d’enregistrer les modèles Resource Manager dans les artefacts de build. Pour cet exemple, vous allez utiliser le code du [Didacticiel : Générez une application Web .NET Core et SQL Database dans Azure App Service](../app-service/tutorial-dotnetcore-sqldb-app.md), pour déployer et tester l’application Web dans Azure.

![Flux général](./media/use-devtest-labs-build-release-pipelines/overall-flow.png)

## <a name="set-up-azure-resources"></a>Configurer les ressources Azure
Quelques éléments doivent être créés au préalable :

- Deux référentiels. Le premier avec le code du didacticiel et un modèle Resource Manager avec deux machines virtuelles supplémentaires. Le deuxième contient le modèle Azure Resource Manager de base (configuration existante).
- Groupe de ressources pour le déploiement du code et de la configuration de production.
- Un laboratoire doit être configuré avec une [connexion au référentiel de configuration](devtest-lab-create-environment-from-arm.md) pour le pipeline de build. Le modèle Resource Manager doit être archivé dans le référentiel de configuration en tant que fichier azuredeploy.json avec le metadata.json pour permettre à DevTest Labs de reconnaître et de déployer le modèle.

Le pipeline de build crée un environnement DevTest Labs et déploie le code à des fins de test.

## <a name="set-up-a-build-pipeline"></a>Configurer un pipeline de build
Dans Azure Pipelines, créez un pipeline de build à l’aide du code du [didacticiel : Créer une application web .NET Core et SQL Database dans Azure App Service](../app-service/tutorial-dotnetcore-sqldb-app.md). Utilisez le modèle **ASP.NET Core**, qui exécutera la tâche nécessaire pour générer, tester et publier le code.

![Sélectionner le modèle ASP.NET](./media/use-devtest-labs-build-release-pipelines/select-asp-net.png)

Vous devez ajouter trois tâches supplémentaires pour créer l’environnement dans DevTest Labs et le déployer dans l’environnement.

![Pipeline avec trois tâches](./media/use-devtest-labs-build-release-pipelines/pipeline-tasks.png)

### <a name="create-environment-task"></a>Tâche Créer un environnement
Dans la tâche Créer un environnement (tâche **Créer un environnement Azure DevTest Labs**), utilisez les listes déroulantes pour sélectionner les valeurs suivantes :

- Abonnement Azure
- nom du labo
- nom du référentiel
- nom du modèle (qui affiche le dossier dans lequel l’environnement est stocké). 

Nous vous recommandons d’utiliser des listes déroulantes sur la page au lieu d’entrer les informations manuellement. Si vous entrez manuellement les informations, entrez des ID de ressource Azure complets. La tâche affiche les noms conviviaux au lieu des ID de ressource. 

Le nom de l’environnement est le nom affiché dans DevTest Labs. Il doit s’agir d’un nom unique pour chaque build. Par exemple : **TestEnv$(Build.BuildId)** . 

Vous pouvez spécifier un fichier de paramètres ou des paramètres pour transmettre des informations dans le modèle Resource Manager. 

Sélectionnez l'option **Créer des variables de sortie en fonction de la sortie du modèle d’environnement** et entrez un nom de référence. Pour cet exemple, entrez **BaseEnv** pour le nom de référence. Vous allez utiliser ce BaseEnv lors de la configuration de la tâche suivante. 

![Tâche Créer un environnement Azure DevTest Labs](./media/use-devtest-labs-build-release-pipelines/create-environment.png)

### <a name="populate-environment-task"></a>Tâche Peupler l’environnement
La deuxième tâche (tâche **Peupler l’environnement Azure DevTest Labs**) consiste à mettre à jour l’environnement DevTest Labs existant. La tâche Créer un environnement génère **BaseEnv.environmentResourceId** qui est utilisé pour configurer le nom de l’environnement pour cette tâche. Le modèle Resource Manager pour cet exemple a deux paramètres : **adminUserName** et **adminPassword**. 

![Tâche Peupler un environnement Azure DevTest Labs](./media/use-devtest-labs-build-release-pipelines/populate-environment.png)

## <a name="app-service-deploy-task"></a>Tâche Déployer App Service
La troisième tâche est **Déployer Azure App Service**. Le type d’application est défini sur **Application Web** et le nom App Service est défini sur **$(WebSite)** .

![Tâche Déployer App Service](./media/use-devtest-labs-build-release-pipelines/app-service-deploy.png)

## <a name="set-up-release-pipeline"></a>Configurer un pipeline de mise en production
Vous créez un pipeline de mise en production avec deux tâches : **Déploiement Azure : Créer ou mettre à jour un groupe de ressources** et **Déployer Azure App Service**. 

Pour la première tâche, spécifiez le nom et l’emplacement du groupe de ressources. L’emplacement du modèle est un artefact lié. Si le modèle Resource Manager comprend des modèles liés, un déploiement de groupe de ressources personnalisé doit être implémenté. Le modèle se trouve dans l’artefact de dépôt publié. Remplacez les paramètres de modèle pour le modèle Resource Manager. Vous pouvez conserver les valeurs par défaut des autres paramètres. 

Pour la deuxième tâche **Déployer Azure App Service**, spécifiez l’abonnement Azure, sélectionnez **Application Web** pour le **Type d’application** et **$(WebSite)** pour le **Nom App Service**. Vous pouvez conserver les valeurs par défaut des autres paramètres. 

## <a name="test-run"></a>Série de tests
Maintenant que les deux pipelines sont configurés, vous pouvez mettre en file d’attente une build manuellement et la voir fonctionner. L’étape suivante consiste à définir le déclencheur approprié pour la build et à connecter la build au pipeline de mise en production.

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- [Intégrer Azure DevTest Labs à votre pipeline de livraison et d’intégration continue d’Azure Pipelines](devtest-lab-integrate-ci-cd.md)
- [Intégrer des environnements à vos pipelines CI/CD Azure Pipelines](integrate-environments-devops-pipeline.md)
