---
title: Extraire les paramètres d’App Configuration avec Azure Pipelines
description: Découvrir comment utiliser Azure Pipelines pour extraire des paires clé-valeur d’un magasin App Configuration
services: azure-app-configuration
author: drewbatgit
ms.service: azure-app-configuration
ms.topic: how-to
ms.date: 11/17/2020
ms.author: drewbat
ms.openlocfilehash: 1c28b4e9821f31f927ef4f640aa664d330cf8792
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96570992"
---
# <a name="pull-settings-to-app-configuration-with-azure-pipelines"></a>Extraire des paramètres d’App Configuration à l’aide d’Azure Pipelines

La tâche [Azure App Configuration](https://marketplace.visualstudio.com/items?itemName=AzureAppConfiguration.azure-app-configuration-task) extrait des paires clé-valeur de votre magasin App Configuration et les définit en tant que variables de pipeline Azure, lesquelles peuvent être consommées par les tâches à venir. Cette tâche complète la tâche [Azure App Configuration Push](https://marketplace.visualstudio.com/items?itemName=AzureAppConfiguration.azure-app-configuration-task-push) (Push Azure App Configuration), qui envoie les paires clé-valeur d’un fichier config vers votre magasin App Configuration. Pour plus d’informations, consultez [Envoyer des paramètres vers App Configuration avec Azure Pipelines](push-kv-devops-pipeline.md).

## <a name="prerequisites"></a>Prérequis

- Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/)
- Magasin App Configuration : créez-en un gratuitement dans le [portail Azure](https://portal.azure.com).
- Projet Azure DevOps : [créez-en un gratuitement](https://go.microsoft.com/fwlink/?LinkId=2014881)
- Tâche Azure App Configuration : téléchargez-la gratuitement à partir de [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=AzureAppConfiguration.azure-app-configuration-task#:~:text=Navigate%20to%20the%20Tasks%20tab,the%20Azure%20App%20Configuration%20instance.).  

## <a name="create-a-service-connection"></a>Créer une connexion de service

Une [connexion de service](/azure/devops/pipelines/library/service-endpoints) vous permet d’accéder aux ressources de votre abonnement Azure à partir de votre projet Azure DevOps.

1. Dans Azure DevOps, accédez au projet contenant votre pipeline cible et ouvrez les **Paramètres du projet** en bas à gauche.
1. Sous **Pipelines**, sélectionnez **Connexions de service**.
1. Si vous n’avez aucune connexion de service, cliquez sur le bouton **Créer une connexion de service** au milieu de l’écran. Sinon, cliquez sur **Nouvelle connexion de service** en haut à droite de la page.
1. Sélectionnez **Azure Resource Manager**.
1. Sélectionnez **Principal de service (automatique)** .
1. Renseignez votre abonnement et votre ressource. Donnez un nom à votre connexion de service.

Maintenant que votre connexion de service est créée, recherchez le nom du principal de service qui lui est assigné. Vous ajouterez une nouvelle attribution de rôle à ce principal de service à l’étape suivante.

1. Accédez à **Paramètres du projet** > **Connexions de service**.
1. Sélectionnez la connexion de service que vous avez créée dans la section précédente.
1. Sélectionnez **Gérer le principal du service**.
1. Notez le **Nom complet** listé.

## <a name="add-role-assignment"></a>Ajouter une attribution de rôle

Attribuez le rôle App Configuration approprié à la connexion de service utilisée dans la tâche pour que celle-ci puisse accéder au magasin App Configuration.

1. Accédez à votre magasin App Configuration cible. Pour accéder à une procédure pas à pas de configuration d’un magasin App Configuration, consultez [Créer un magasin App Configuration](/azure/azure-app-configuration/quickstart-dotnet-core-app#create-an-app-configuration-store) dans l’un des guides de démarrage rapide consacrés à Azure App Configuration.
1. Sur la gauche, sélectionnez **Contrôle d’accès (IAM)** .
1. En haut, sélectionnez **+ Ajouter** et choisissez **Ajouter une attribution de rôle**.
1. Sous **Rôle**, sélectionnez **Lecteur de données de l’App Configuration**. Ce rôle permet à la tâche de lire le contenu du magasin App Configuration. 
1. Sélectionnez le principal de service associé à la connexion de service que vous avez créée dans la section précédente.

> [!NOTE]
> Pour résoudre les références Azure Key Vault dans App Configuration, la connexion de service doit également avoir l’autorisation de lire les secrets des coffres de clés Azure Key Vault référencés.
  
## <a name="use-in-builds"></a>Utiliser dans les builds

Cette section explique comment utiliser la tâche Azure App Configuration dans un pipeline de build Azure DevOps.

1. Accédez à la page de pipeline de build en cliquant sur **Pipelines** > **Pipelines**. Pour accéder à la documentation sur les pipelines de build, consultez [Créer votre premier pipeline](/azure/devops/pipelines/create-first-pipeline?view=azure-devops&tabs=net%2Ctfs-2018-2%2Cbrowser).
      - Si vous créez un pipeline de build, cliquez sur **Nouveau pipeline**, puis sélectionnez le dépôt de votre pipeline. Sélectionnez **Afficher l’Assistant** sur le côté droit du pipeline, puis recherchez la tâche **Azure App Configuration**.
      - Si vous utilisez un pipeline de build existant, sélectionnez **Modifier** pour modifier le pipeline. Sous l’onglet **Tâches**, recherchez la tâche **Azure App Configuration**.
1. Configurez les paramètres nécessaires à la tâche pour extraire les paires clé-valeur du magasin App Configuration. Vous trouverez les descriptions des paramètres dans la section **Paramètres** ci-dessous ainsi que dans les info-bulles en regard de chaque paramètre.
      - Affectez au paramètre **Abonnement Azure** le nom de la connexion de service que vous avez créée à une étape précédente.
      - Affectez au paramètre **App Configuration name** (Nom App Configuration) le nom de ressource de votre magasin App Configuration.
      - Acceptez les valeurs par défaut des paramètres restants.
1. Enregistrez et mettez en file d’attente une build. Le journal de génération affiche les erreurs qui se sont produites pendant l’exécution de la tâche.

## <a name="use-in-releases"></a>Utiliser dans les mises en production

Cette section explique comment utiliser la tâche Azure App Configuration dans un pipeline de mise en production Azure DevOps.

1. Accédez à la page de pipeline de mise en production en sélectionnant **Pipelines** > **Mises en production**. Pour accéder à la documentation sur les pipelines de mise en production, consultez [Pipelines de mise en production](/azure/devops/pipelines/release?view=azure-devops).
1. Choisissez un pipeline de mise en production existant. Si vous n’en avez pas, cliquez sur **Nouveau pipeline** pour en créer un.
1. Sélectionnez le bouton **Modifier** dans le coin supérieur droit pour modifier le pipeline de mise en production.
1. Choisissez la **phase** à laquelle ajouter la tâche. Pour plus d’informations sur les phases, consultez la section [Ajouter des phases, des dépendances et des conditions](/azure/devops/pipelines/release/environments?view=azure-devops).
1. Cliquez sur **+** pour « Exécuter sur l’agent », puis ajoutez la tâche **Azure App Configuration** sous l’onglet **Ajouter des tâches**.
1. Configurez les paramètres nécessaires dans la tâche pour extraire vos paires clé-valeur du magasin App Configuration. Vous trouverez les descriptions des paramètres dans la section **Paramètres** ci-dessous ainsi que dans les info-bulles en regard de chaque paramètre.
      - Affectez au paramètre **Abonnement Azure** le nom de la connexion de service que vous avez créée à une étape précédente.
      - Affectez au paramètre **App Configuration name** (Nom App Configuration) le nom de ressource de votre magasin App Configuration.
      - Acceptez les valeurs par défaut des paramètres restants.
1. Enregistrez et mettez en file d’attente une mise en production. Le journal de mise en production affiche les erreurs rencontrées pendant l’exécution de la tâche.

## <a name="parameters"></a>Paramètres

Les paramètres suivants sont utilisés par la tâche Azure App Configuration :

- **Azure subscription** : liste déroulante contenant vos connexions de service Azure disponibles. Pour mettre à jour et actualiser votre liste de connexions de service Azure disponibles, appuyez sur le bouton **Refresh Azure subscription** (Actualiser l’abonnement Azure) à droite de la zone de texte.
- **App Configuration Name** : liste déroulante qui charge vos magasins de configuration disponibles sous l’abonnement sélectionné. Pour mettre à jour et actualiser votre liste de magasins de configuration disponibles, appuyez sur le bouton **Refresh App Configuration Name** (Actualiser le nom App Configuration) à droite de la zone de texte.
- **Filtre de clé** : Le filtre permet de sélectionner les paires clé-valeur demandées à partir d’Azure App Configuration. La valeur * permet de sélectionner toutes les paires clé-valeur. Pour plus d’informations, consultez [Interroger des paires clé-valeur](concept-key-value.md#query-key-values).
- **Étiquette** : Spécifie l’étiquette à utiliser au moment de la sélection des paires clé-valeur dans le magasin App Configuration. Si aucune étiquette n’est fournie, les paires clé-valeur sans étiquette sont récupérées. Les caractères suivants ne sont pas autorisés : « , » « * ».
- **Préfixes de clés à découper** : Spécifie un ou plusieurs préfixes de clés App Configuration à découper avant de définir ces clés en tant que variables. Vous pouvez séparer plusieurs préfixes par un caractère de nouvelle ligne.

## <a name="use-key-values-in-subsequent-tasks"></a>Utiliser des paires clé-valeur dans les tâches à venir

Les paires clé-valeur extraites d’App Configuration sont définies en tant que variables de pipeline, lesquelles sont accessibles en tant que variables d’environnement. La clé de la variable d’environnement est la clé de la paire clé-valeur récupérée à partir d’App Configuration après le découpage du préfixe, le cas échéant.

Par exemple, si une tâche à venir exécute un script PowerShell, elle peut consommer une paire clé-valeur avec la clé « myBuildSetting », comme ceci :
```powershell
echo "$env:myBuildSetting"
```
La valeur s’affiche ensuite sur la console.

## <a name="troubleshooting"></a>Dépannage

Si une erreur inattendue se produit, les journaux de débogage peuvent être activés en affectant la valeur `true` à la variable de pipeline `system.debug`.

## <a name="faq"></a>Questions fréquentes (FAQ)

**Comment composer ma configuration à partir de plusieurs clés et étiquettes ?**

Il arrive que la configuration soit impérativement composée à partir de plusieurs étiquettes, par exemple default et dev. Vous pouvez utiliser plusieurs tâches App Configuration dans un pipeline pour implémenter ce scénario. Les paires clé-valeur extraites par une tâche au cours d’une étape à venir remplacent toutes les valeurs des étapes précédentes. Dans l’exemple ci-dessus, une tâche permet de sélectionner des paires clé-valeur avec l’étiquette default, et une deuxième tâche permet de sélectionner des paires clé-valeur avec l’étiquette dev. Les clés avec l’étiquette dev remplacent les mêmes clés avec l’étiquette default.
