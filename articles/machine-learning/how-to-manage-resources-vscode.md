---
title: Créer et gérer des ressources avec l’extension VS Code (préversion)
titleSuffix: Azure Machine Learning
description: Découvrez comment créer et gérer des ressources Azure Machine Learning à l’aide de l’extension Visual Studio Code pour Azure Machine Learning.
services: machine-learning
author: luisquintanilla
ms.author: luquinta
ms.reviewer: luquinta
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.date: 11/16/2020
ms.openlocfilehash: f8eb18b190b72381f1a93575eb39b3d19d8d431b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010671"
---
# <a name="manage-azure-machine-learning-resources-with-the-vs-code-extension-preview"></a>Gérer les ressources Azure Machine Learning avec l’extension VS Code (préversion)

Découvrez comment gérer les ressources Azure Machine Learning avec l’extension VS Code.

![Extension VS Code Azure Machine Learning](media/how-to-manage-resources-vscode/azure-machine-learning-vscode-extension.png)

## <a name="prerequisites"></a>Prérequis

- Abonnement Azure. Si vous n’en avez pas, inscrivez-vous ou essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree)
- Visual Studio Code. [Installez-le](https://code.visualstudio.com/docs/setup/setup-overview) si ce n’est déjà fait.
- Extension VS Code Azure Machine Learning. Suivez le [Guide d’installation de l’extension VS Code Azure Machine Learning](tutorial-setup-vscode-extension.md#install-the-extension) pour installer l’extension.

Toutes les procédures ci-dessous partent du principe que vous êtes dans la vue Azure Machine Learning dans Visual Studio Code. Pour lancer l’extension, sélectionnez l’icône **Azure** dans la barre d’activité de VS Code.

## <a name="workspaces"></a>Workspaces

Pour plus d’informations, consultez [Espaces de travail](concept-workspace.md).

### <a name="create-a-workspace"></a>Créer un espace de travail

1. Dans la vue Azure Machine Learning, cliquez avec le bouton droit sur le nœud de votre abonnement et sélectionnez **Créer un espace de travail**.
1. À l’invite :
    1. Spécifiez un nom pour votre espace de travail.
    1. Choisir votre abonnement Azure
    1. Choisissez ou créez un groupe de ressources dans lequel provisionner l’espace de travail.
    1. Sélectionnez l’emplacement où vous souhaitez provisionner l’espace de travail.

Les autres méthodes de création d’un espace de travail sont les suivantes :

- Ouvrez la palette de commandes **Afficher > Palette de commandes** et entrez dans l’invite de texte **Azure ML : Créez un espace de travail**.
- Cliquez sur l’icône `+` en haut de la vue Azure Machine Learning.
- Créez un espace de travail quand vous êtes invité à en sélectionner un lors du provisionnement d’autres ressources.

### <a name="remove-a-workspace"></a>Supprimer un espace de travail

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Cliquez avec le bouton doit sur l’espace de travail que vous souhaitez supprimer.
1. Indiquez si vous souhaitez supprimer :
    - *Uniquement l’espace de travail* : cette option supprime **uniquement** la ressource Azure de l’espace de travail. Le groupe de ressources, les comptes de stockage et toute autre ressource à laquelle l’espace de travail a été attaché sont toujours dans Azure.
    - *Avec les ressources associées* : cette option supprime l’espace de travail **et** toutes les ressources qui lui sont associées.

## <a name="datastores"></a>Magasins de données

L’extension VS Code prend actuellement en charge les magasins de données des types suivants :

- Partage de fichiers Azure
- Stockage Blob Azure

Quand vous créez un espace de travail, un magasin de données est créé pour chacun de ces types.

Pour plus d’informations, consultez [Magasins de données](concept-data.md#datastores).

### <a name="create-a-datastore"></a>Créer une banque de données

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud d’espace de travail sous lequel vous souhaitez créer le magasin de données.
1. Cliquez avec le bouton droit sur le nœud **Magasins de données** et sélectionnez **Inscrire le magasin de données**.
1. À l’invite :
    1. Spécifiez un nom pour votre magasin de données.
    1. Choisissez le type de magasin de données.
    1. Sélectionnez votre ressource de stockage. Vous pouvez choisir une ressource de stockage qui est associée à votre espace de travail ou sélectionner parmi les ressources de stockage valides dans vos abonnements Azure.
    1. Choisissez le conteneur où se trouvent vos données à l’intérieur de la ressource de stockage sélectionnée précédemment.
1. Un fichier de configuration s’affiche dans VS Code. Si vous êtes satisfait de votre fichier de configuration, sélectionnez **Enregistrer et continuer** ou ouvrez la palette de commandes VS Code (**Affichage > Palette de commandes**) et tapez **Azure ML : Enregistrer et continuer**.

### <a name="manage-a-datastore"></a>Gérer un magasin de données

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud de votre espace de travail.
1. Développez le nœud **Magasins de données** à l’intérieur de votre espace de travail.
1. Sélectionnez le magasin de données de votre choix pour les options suivantes :
    - *Définir par défaut*. Chaque fois que vous exécuterez des expériences, c’est ce magasin de données qui sera utilisé.
    - *Inspecter les paramètres en lecture seule*.
    - *Modifier*. Changez le type d’authentification et les informations d’identification. Les types d’authentification pris en charge sont la clé de compte et le jeton SAS.

## <a name="datasets"></a>Groupes de données

L’extension prend actuellement en charge les types de jeux de données suivants :

- *Tabulaire* : vous permet de matérialiser des données dans un DataFrame (Pandas ou PySpark).
- *Fichier* : fichier ou collection de fichiers. Vous permet de télécharger ou de monter des fichiers dans votre capacité de calcul.

Pour plus d’informations, consultez [Jeux de données](concept-data.md#datasets).

### <a name="create-dataset"></a>Créer un jeu de données

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud d’espace de travail sous lequel vous souhaitez créer le magasin de données.
1. Cliquez avec le bouton droit sur le nœud **Jeux de données** et sélectionnez **Créer un jeu de données**.
1. À l’invite :
    1. Choisissez le type de jeu de données.
    1. Précisez si les données se trouvent sur votre PC ou sur le web.
    1. Spécifiez l’emplacement de vos données. Il peut s’agir d’un seul fichier ou d’un répertoire contenant vos fichiers de données.
    1. Choisissez le magasin de données vers lequel vous souhaitez charger vos données.
    1. Fournissez un préfixe qui permet d’identifier votre jeu de données dans le magasin de données.

### <a name="version-datasets"></a>Jeux de données avec version

Lors de la génération de modèles Machine Learning, à mesure que les données changent, vous souhaiterez peut-être affecter une version à votre jeu de données. Pour cela, dans l’extension VS Code, effectuez les étapes suivantes :

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud de votre espace de travail.
1. Développez le nœud **Jeux de données**.
1. Cliquez avec le bouton droit sur le jeu de données dont vous souhaitez gérer les versions, puis sélectionnez **Créer une nouvelle version**.
1. À l’invite :
    1. Sélectionnez le type de jeu de données.
    1. Précisez si les données se trouvent sur votre PC ou sur le web.
    1. Spécifiez l’emplacement de vos données. Il peut s’agir d’un seul fichier ou d’un répertoire contenant vos fichiers de données.
    1. Choisissez le magasin de données vers lequel vous souhaitez charger vos données.
    1. Fournissez un préfixe qui permet d’identifier votre jeu de données dans le magasin de données.

### <a name="view-dataset-properties"></a>Afficher les propriétés d’un jeu de données

Cette option vous permet d’afficher les métadonnées associées à un jeu de données spécifique. Pour cela, dans l’extension VS Code, effectuez les étapes suivantes :

1. Développez le nœud de votre espace de travail.
1. Développez le nœud **Jeux de données**.
1. Cliquez avec le bouton droit sur le jeu de données que vous souhaitez inspecter, puis sélectionnez **Afficher les propriétés du jeu de données**. Un fichier de configuration s’affiche, avec les propriétés de la dernière version du jeu de données.

> [!NOTE]
> Si vous avez plusieurs versions de votre jeu de données, vous pouvez choisir d’afficher uniquement les propriétés d’une version spécifique en développant le nœud du jeu de données et en effectuant les mêmes étapes que celles décrites dans cette section sur la version qui vous intéresse.

### <a name="unregister-datasets"></a>Désinscrire des jeux de données

Pour supprimer un jeu de données et toutes ses versions, annulez son inscription. Pour cela, dans l’extension VS Code, effectuez les étapes suivantes :

1. Développez le nœud de votre espace de travail.
1. Développez le nœud **Jeux de données**.
1. Cliquez avec le bouton droit sur le jeu de données dont vous souhaitez annuler l’inscription, puis sélectionnez **Désinscrire le jeu de données**.

## <a name="environments"></a>Environnements

Pour plus d’informations, consultez [Environnements](concept-environments.md).

### <a name="create-environment"></a>Créer un environnement

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud d’espace de travail sous lequel vous souhaitez créer le magasin de données.
1. Cliquez avec le bouton droit sur le nœud **Environnements** et sélectionnez **Créer un environnement**.
1. À l’invite :
    1. Donnez un nom à votre environnement.
    1. Définissez la configuration de votre environnement :
        - *Environnements organisés* : environnements préconfigurés dans Azure Machine Learning. Vous pouvez personnaliser davantage l’environnement en modifiant la propriété `dependencies` dans le fichier JSON. Apprenez-en davantage sur les [environnements organisés](resource-curated-environments.md).
        - *Fichier de dépendances Conda* : pour les environnements Anaconda, vous pouvez fournir le fichier contenant la définition de votre environnement.
        - *Fichier de spécifications pip* : pour les environnements pip, vous pouvez fournir le fichier contenant la définition de votre environnement.
        - *Environnement Conda existant* : cette option recherche les environnements Conda sur votre PC local et tente de créer un environnement à partir de l’environnement sélectionné.
        - *Personnalisé* : définissez vos propres canaux et dépendances.
    1. Un fichier de configuration s’ouvre dans l’éditeur. Si vous êtes satisfait de votre configuration, sélectionnez **Enregistrer et continuer** ou ouvrez la palette de commandes VS Code (**Affichage > Palette de commandes**) et tapez **Azure ML : Enregistrer et continuer**.

### <a name="view-environment-configurations"></a>Afficher les configurations de l’environnement

Pour afficher les dépendances et les configurations d’un environnement spécifique dans l’extension

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud de votre espace de travail.
1. Développez le nœud **Environnements**.
1. Cliquez avec le bouton droit sur l’environnement que vous souhaitez afficher et sélectionnez **Afficher l’environnement**.

### <a name="edit-environment-configurations"></a>Modifier des configurations d’environnement

Pour modifier les dépendances et les configurations d’un environnement spécifique dans l’extension :

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Environnements** à l’intérieur de votre espace de travail.
1. Cliquez avec le bouton droit sur l’environnement que vous souhaitez afficher, puis sélectionnez **Modifier l’environnement**.
1. Après avoir effectué les modifications, si vous êtes satisfait de votre configuration, sélectionnez **Enregistrer et continuer** ou ouvrez la palette de commandes VS Code (**Affichage > Palette de commandes**) et tapez **Azure ML : Enregistrer et continuer**.

## <a name="experiments"></a>Expériences

Pour plus d’informations, consultez [Expériences](concept-azure-machine-learning-architecture.md#experiments).

### <a name="create-experiment"></a>Créer une expérience

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud de votre espace de travail.
1. Cliquez avec le bouton droit sur le nœud **Expériences** dans votre espace de travail, puis sélectionnez **Créer une expérience**.
1. À l’invite, spécifiez un nom pour votre expérience.

### <a name="run-experiment"></a>Exécuter une expérience

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Expériences** à l’intérieur de votre espace de travail.
1. Cliquez avec le bouton droit sur l’expérience que vous souhaitez exécuter.
1. Sélectionnez l’icône **Exécuter l’expérience** dans la barre d’activité.
1. Indiquez si vous souhaitez exécuter votre expérience localement ou à distance. Pour plus d’informations sur l’exécution et le débogage des expériences localement, consultez le [Guide de débogage](how-to-debug-visual-studio-code.md).
1. Choisissez votre abonnement.
1. Choisissez l’espace de travail ML Azure sous lequel exécuter l’expérience.
1. Choisissez votre expérience.
1. Choisissez ou créez une capacité de calcul sur laquelle exécuter l’expérience.
1. Choisissez ou créez une configuration d’exécution pour votre expérience.

Vous pouvez aussi sélectionner le bouton **Exécuter l’expérience** en haut de l’extension et configurer votre exécution d’expérience à l’invite.

### <a name="view-experiment"></a>Afficher une expérience

Pour afficher votre expérience dans Azure Machine Learning Studio

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Expériences** à l’intérieur de votre espace de travail.
1. Cliquez avec le bouton droit sur l’expérience que vous souhaitez afficher et sélectionnez **Afficher l’expérience**. 
1. Un message s’affiche pour vous inviter à ouvrir l’URL d’expérience dans Azure Machine Learning Studio. Sélectionnez **Ouvrir**.

### <a name="track-run-progress"></a>Suivre la progression d’une exécution

Vous souhaiterez peut-être voir la progression de l’exécution de votre expérience. Pour suivre la progression d’une exécution dans Azure Machine Learning Studio à partir de l’extension :

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Expériences** à l’intérieur de votre espace de travail.
1. Développez le nœud de l’expérience dont vous souhaitez suivre la progression.
1. Cliquez avec le bouton droit sur l’exécution, puis sélectionnez **Afficher l’exécution dans le portail Azure**.
1. Un message s’affiche pour vous inviter à ouvrir l’URL d’exécution dans Azure Machine Learning Studio. Sélectionnez **Ouvrir**.

### <a name="download-run-logs--outputs"></a>Télécharger les journaux et les sorties d’exécution

Une fois l’exécution terminée, vous souhaiterez peut-être télécharger les journaux et des ressources telles que le modèle généré dans le cadre d’une exécution d’expérience.

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Expériences** à l’intérieur de votre espace de travail.
1. Développez le nœud de l’expérience dont vous souhaitez suivre la progression.
1. Cliquez avec le bouton droit sur l’exécution :
    - Pour télécharger les sorties, sélectionnez **Télécharger les sorties**.
    - Pour télécharger les journaux, sélectionnez **Télécharger les journaux**.

### <a name="view-run-metadata"></a>Afficher les métadonnées d’une exécution

Dans l’extension, vous pouvez inspecter des métadonnées telles que la configuration d’exécution utilisée pour l’exécution ainsi que les détails de l’exécution.

## <a name="compute-instances"></a>Instances de calcul

Pour plus d’informations, consultez [Instances de calcul](concept-compute-instance.md).

### <a name="create-compute-instance"></a>Créer une instance de calcul

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud d’espace de travail sous lequel vous souhaitez créer l’instance de calcul.
1. Cliquez avec le bouton droit sur le nœud **Instances de calcul**, puis sélectionnez **Créer une instance de calcul**.
1. À l’invite :
    1. Donnez un nom à votre instance de calcul.
    1. Sélectionnez une taille de machine virtuelle dans la liste.
    1. Indiquez si vous souhaitez activer l’accès SSH.
        1. Si vous activez l’accès SSH, vous devez également fournir la clé SSH publique ou le fichier contenant la clé. Pour plus d’informations, consultez le [guide sur la création et l’utilisation de clés SSH sur Azure](../virtual-machines/linux/mac-create-ssh-keys.md).

### <a name="stop-or-restart-compute-instance"></a>Arrêter ou redémarrer une instance de calcul

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Instance de calcul** à l’intérieur de votre espace de travail.
1. Cliquez avec le bouton droit sur l’instance de calcul que vous souhaitez arrêter ou redémarrer, puis sélectionnez **Arrêter l’instance de calcul** ou **Redémarrer l’instance de calcul**.

### <a name="view-compute-instance-configuration"></a>Afficher la configuration d’instances de calcul

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Instance de calcul** à l’intérieur de votre espace de travail.
1. Cliquez avec le bouton droit sur l’instance de calcul que vous souhaitez inspecter, puis sélectionnez **Afficher les propriétés de l’instance de calcul**.

### <a name="delete-compute-instance"></a>Supprimer une instance de calcul

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Instance de calcul** à l’intérieur de votre espace de travail.
1. Cliquez avec le bouton droit sur l’instance de calcul à supprimer, puis sélectionnez **Supprimer l’instance de calcul**.

## <a name="compute-clusters"></a>Clusters de calcul

L’extension prend en charge les types de capacité de calcul suivants :

- Cluster de calcul Azure Machine Learning
- Azure Kubernetes Service

Pour plus d’informations, consultez [Cibles de calcul](concept-compute-target.md#train).

### <a name="create-compute"></a>Créer une capacité de calcul

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud d’espace de travail sous lequel vous souhaitez créer le cluster de calcul.
1. Cliquez avec le bouton droit sur le nœud **Clusters de calcul**, puis sélectionnez **Créer un calcul**.
1. À l’invite :
    1. Choisissez un type de capacité de calcul.
    1. Choisissez une taille de machine virtuelle. Apprenez-en davantage sur les [tailles de machines virtuelles](../virtual-machines/sizes.md).
    1. Entrez un nom pour votre capacité de calcul.

### <a name="view-compute-configuration"></a>Afficher une configuration de calcul

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Clusters de calcul** à l’intérieur de votre espace de travail.
1. Cliquez avec le bouton droit sur la capacité de calcul que vous souhaitez afficher, puis sélectionnez **Afficher les propriétés du calcul**.

### <a name="edit-compute-scale-settings"></a>Modifier les paramètres d’échelle de calcul

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Clusters de calcul** à l’intérieur de votre espace de travail.
1. Cliquez avec le bouton droit sur la capacité de calcul à modifier, puis sélectionnez **Modifier le calcul**.
1. Un fichier de configuration pour votre capacité de calcul s’ouvre dans l’éditeur. Si vous êtes satisfait de votre configuration, sélectionnez **Enregistrer et continuer** ou ouvrez la palette de commandes VS Code (**Affichage > Palette de commandes**) et tapez **Azure ML : Enregistrer et continuer**.

### <a name="delete-compute"></a>Supprimer une capacité de calcul

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Clusters de calcul** à l’intérieur de votre espace de travail.
1. Cliquez avec le bouton droit sur la capacité de calcul à supprimer, puis sélectionnez **Supprimer le calcul**.

### <a name="create-run-configuration"></a>Créer une configuration d’exécution

Pour créer une configuration d’exécution dans l’extension

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Clusters de calcul** à l’intérieur de votre espace de travail.
1. Cliquez avec le bouton droit sur la cible de calcul sous laquelle vous souhaitez créer la configuration d’exécution, puis sélectionnez **Créer une configuration d’exécution**.
1. À l’invite :
    1. Donnez un nom à votre cible de calcul.
    1. Choisissez ou créez un environnement.
    1. Tapez le nom du script que vous souhaitez exécuter, ou appuyez sur **Entrée** pour naviguer jusqu’au script sur votre ordinateur local.
    1. (Facultatif) Précisez si vous souhaitez créer une référence de données pour votre exécution d’entraînement. Vous serez alors invité à définir un jeu de données dans votre configuration d’exécution.
        1. Sélectionnez l’un de vos jeux de données inscrits afin de le lier à la configuration d’exécution. Un fichier de configuration pour votre jeu de données s’ouvre dans l’éditeur. Si vous êtes satisfait de votre configuration, sélectionnez **Enregistrer et continuer** ou ouvrez la palette de commandes VS Code (**Affichage > Palette de commandes**) et tapez **Azure ML : Enregistrer et continuer**.
    1. Si vous êtes satisfait de votre configuration, sélectionnez **Enregistrer et continuer** ou ouvrez la palette de commandes VS Code (**Affichage > Palette de commandes**) et tapez **Azure ML : Enregistrer et continuer**.

### <a name="edit-run-configuration"></a>Modifier une configuration d’exécution

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez votre nœud de cluster de calcul dans le nœud **Clusters de calcul** de votre espace de travail.
1. Cliquez avec le bouton droit sur la configuration d’exécution que vous souhaitez modifier, puis sélectionnez **Modifier la configuration d’exécution**.
1. Un fichier de configuration pour votre configuration d’exécution s’ouvre dans l’éditeur. Si vous êtes satisfait de votre configuration, sélectionnez **Enregistrer et continuer** ou ouvrez la palette de commandes VS Code (**Affichage > Palette de commandes**) et tapez **Azure ML : Enregistrer et continuer**.

### <a name="delete-run-configuration"></a>Supprimer une configuration d’exécution

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud de votre espace de travail.
1. Développez le nœud de cluster de calcul qui vous intéresse dans le nœud **Clusters de calcul**.
1. Cliquez avec le bouton droit sur la configuration d’exécution que vous souhaitez modifier, puis sélectionnez **Supprimer la configuration d’exécution**.

## <a name="models"></a>Modèles

Pour plus d’informations, consultez [Modèles](concept-azure-machine-learning-architecture.md#models).

### <a name="register-model"></a>Inscrire le modèle

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud de votre espace de travail.
1. Cliquez avec le bouton droit sur le nœud **Modèles** et sélectionnez **Inscrire un modèle**.
1. À l’invite :
    1. Donnez un nom à votre modèle.
    1. Choisissez si votre modèle est un fichier ou un dossier.
    1. Recherchez le modèle sur votre PC local.
    1. Un fichier de configuration pour votre modèle s’ouvre dans l’éditeur. Si vous êtes satisfait de votre configuration, sélectionnez **Enregistrer et continuer** ou ouvrez la palette de commandes VS Code (**Affichage > Palette de commandes**) et tapez **Azure ML : Enregistrer et continuer**.

### <a name="view-model-properties"></a>Afficher les propriétés d’un modèle

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Modèles** à l’intérieur de votre espace de travail.
1. Cliquez avec le bouton droit sur le modèle dont vous souhaitez afficher les propriétés, puis sélectionnez **Afficher les propriétés du modèle**. Un fichier contenant les propriétés de votre modèle s’ouvre dans l’éditeur.

### <a name="download-model"></a>Télécharger un modèle

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Modèles** à l’intérieur de votre espace de travail.
1. Cliquez avec le bouton droit sur le modèle que vous souhaitez télécharger, puis sélectionnez **Télécharger le fichier de modèle**.

### <a name="delete-a-model"></a>Supprimer un modèle

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Modèles** à l’intérieur de votre espace de travail.
1. Cliquez avec le bouton droit sur le modèle à supprimer, puis sélectionnez **Supprimer le modèle**.

## <a name="endpoints"></a>Points de terminaison

L’extension VS Code prend en charge les cibles de déploiement suivantes :

- Azure Container Instances
- Azure Kubernetes Service

Pour plus d’informations, consultez [Points de terminaison de service web](concept-azure-machine-learning-architecture.md#web-service-endpoint).

### <a name="create-deployments"></a>Créer des déploiements

> [!NOTE]
> Actuellement, la création de déploiement fonctionne uniquement avec les environnements Conda.

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud de votre espace de travail.
1. Cliquez avec le bouton droit sur le nœud **Points de terminaison** et sélectionnez **Déployer le service**.
1. À l’invite :
    1. Précisez si vous souhaitez utiliser un modèle déjà inscrit ou un fichier de modèle local.
    1. Sélectionnez votre modèle.
    1. Choisissez la cible de déploiement vers laquelle vous souhaitez déployer votre modèle.
    1. Donnez un nom à votre modèle.
    1. Fournissez le script à exécuter lors du scoring du modèle.
    1. Spécifiez un fichier de dépendances Conda.
    1. Un fichier de configuration pour votre déploiement s’affiche dans l’éditeur. Si vous êtes satisfait de votre configuration, sélectionnez **Enregistrer et continuer** ou ouvrez la palette de commandes VS Code (**Affichage > Palette de commandes**) et tapez **Azure ML : Enregistrer et continuer**.

> [!NOTE]
> Vous pouvez également cliquer avec le bouton droit sur un modèle inscrit dans le nœud *Modèles* et sélectionner **Déployer le service à partir du modèle inscrit**.

### <a name="delete-deployments"></a>Supprimer des déploiements

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Points de terminaison** à l’intérieur de votre espace de travail.
1. Cliquez avec le bouton droit sur le déploiement que vous souhaitez supprimer, puis sélectionnez **Supprimer le service**.
1. Un message s’affiche pour vous inviter à confirmer la suppression du service. Sélectionnez **OK**.

### <a name="manage-deployments"></a>Gérer les déploiements

Vous pouvez également afficher et modifier les paramètres associés au déploiement.

1. Développez le nœud d’abonnement qui contient votre espace de travail.
1. Développez le nœud **Points de terminaison** à l’intérieur de votre espace de travail.
1. Cliquez avec le bouton droit sur le déploiement que vous souhaitez gérer :
    - Pour modifier des paramètres, sélectionnez **Modifier le service**.
        - Un fichier de configuration pour votre déploiement s’affiche dans l’éditeur. Si vous êtes satisfait de votre configuration, sélectionnez **Enregistrer et continuer** ou ouvrez la palette de commandes VS Code (**Affichage > Palette de commandes**) et tapez **Azure ML : Enregistrer et continuer**.
    - Pour afficher les paramètres de configuration du déploiement, sélectionnez **Afficher les propriétés du service**.

## <a name="next-steps"></a>Étapes suivantes

[Entraînez un modèle de classification d’image](tutorial-train-deploy-image-classification-model-vscode.md) avec l’extension VS Code.