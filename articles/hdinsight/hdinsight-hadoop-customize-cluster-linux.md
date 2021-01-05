---
title: Personnaliser des clusters Azure HDInsight à l’aide d’actions de script
description: Ajoutez des composants personnalisés aux clusters HDInsight à l’aide d’actions de script. Les actions de script sont des scripts Bash qui peuvent être utilisés pour personnaliser la configuration du cluster. Ou ajoutez des services et des utilitaires supplémentaires tels que Hue, Solr ou R
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020, devx-track-azurecli
ms.date: 09/02/2020
ms.openlocfilehash: 35c3901e9a48523a10c1a6aacbc52e6c165e278f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009787"
---
# <a name="customize-azure-hdinsight-clusters-by-using-script-actions"></a>Personnaliser des clusters Azure HDInsight à l’aide d’actions de script

Azure HDInsight fournit une méthode de configuration appelée **actions de script**, lesquelles appellent des scripts personnalisés pour personnaliser le cluster. Ces scripts sont utilisés pour installer des composants supplémentaires et modifier des paramètres de configuration. Les actions de script peuvent être utilisées pendant ou après la création du cluster.

Des actions de script peuvent également être publiées dans la Place de marché Azure en tant qu’application HDInsight. Pour plus d’informations sur les applications HDInsight, consultez [Publier une application HDInsight sur la Place de marché Azure](hdinsight-apps-publish-applications.md).

## <a name="permissions"></a>Autorisations

Quand vous utilisez des actions de script avec un cluster HDInsight joint à un domaine, deux autorisations Apache Ambari sont requises :

* **AMBARI.RUN\_CUSTOM\_COMMAND**. Le rôle Administrateur d’Ambari dispose de cette autorisation par défaut.
* **CLUSTER.RUN\_CUSTOM\_COMMAND**. L’administrateur de cluster HDInsight et l’administrateur Ambari disposent de cette autorisation par défaut.

Pour plus d’informations sur l’utilisation des autorisations avec un cluster HDInsight joint à un domaine, consultez [Gérer des clusters HDInsight avec le pack Sécurité Entreprise](./domain-joined/apache-domain-joined-manage.md).

## <a name="access-control"></a>Contrôle d’accès

Si vous n’êtes pas l’administrateur ou le propriétaire de votre abonnement Azure, votre compte doit avoir au minimum un accès Contributeur au groupe de ressources qui contient le cluster HDInsight.

Une personne disposant au moins d’un accès Contributeur à l’abonnement Azure doit avoir déjà inscrit le fournisseur. L’inscription du fournisseur se produit quand un utilisateur disposant d’un accès Contributeur à l’abonnement crée une ressource. Pour faire cela sans créer de ressource, consultez [Inscrire un fournisseur à l’aide de REST](/rest/api/resources/providers#Providers_Register).

Obtenez plus d’informations sur le fonctionnement de la gestion des accès :

* [Bien démarrer avec la gestion des accès dans le portail Azure](../role-based-access-control/overview.md)
* [Utiliser les attributions de rôle pour gérer l’accès à vos ressources d’abonnement Azure](../role-based-access-control/role-assignments-portal.md)

## <a name="understand-script-actions"></a>Comprendre les actions de script

Une action de script est un script bash qui s’exécute sur les nœuds dans un cluster HDInsight. Les actions de script présentent les caractéristiques et fonctionnalités suivantes :

* Elles doivent être stockées sur un URI accessible à partir du cluster HDInsight. Voici les emplacements de stockage possibles :

    * Pour les clusters normaux :

      * ADLS Gen1 : Le principal de service utilisé par HDInsight pour accéder à Data Lake Storage doit avoir accès en lecture au script. Le format d’URI pour les scripts stockés dans Data Lake Storage Gen1 est `adl://DATALAKESTOREACCOUNTNAME.azuredatalakestore.net/path_to_file`.

      * Un blob est un compte de stockage Azure utilisé comme compte de stockage principal ou supplémentaire pour le cluster HDInsight. HDInsight peut accéder à ces deux types de comptes de stockage lors de la création du cluster.

        > [!IMPORTANT]  
        > Ne faites pas pivoter la clé de stockage sur ce compte Stockage Azure, car cela entraînera l’échec des actions de script suivantes avec des scripts stockés ici.

      * Un service de partage de fichiers public accessible via les chemins d’accès http://. Exemple : Blob Azure, GitHub, OneDrive. Pour obtenir des exemples d’URI, consultez [Exemples de scripts d’action de script](#example-script-action-scripts).

     * Les URI wasb://, wasbs:// ou http[s]:// sont pris en charge pour les clusters avec ESP.

* Elles peuvent être limitées de manière à s’exécuter uniquement sur certains types de nœuds, par exemple des nœuds principaux ou des nœuds worker.

* Elles peuvent avoir un état persistant ou `ad hoc`.

    Les actions de script persistantes doivent avoir un nom unique. Les scripts persistants servent à personnaliser les nouveaux nœuds worker qui sont ajoutés au cluster lors d’opérations de mise à l’échelle. Un script persistant peut également appliquer des modifications à un autre type de nœud au moment de ces opérations. Le nœud principal en est un exemple.

    Les scripts `Ad hoc` ne sont pas persistants. Les actions de script utilisées lors de la création du cluster sont automatiquement rendues persistantes. Ils ne sont pas appliqués aux nœuds worker ajoutés au cluster après l’exécution du script. Vous pouvez donc promouvoir un script `ad hoc` en script persistant ou rétrograder un script persistant en script `ad hoc`. Les scripts qui échouent ne sont pas rendus persistants, même si vous précisez qu’ils doivent l’être.

* Elles peuvent prendre les paramètres utilisés par le script au moment de l’exécution.

* Elles s’exécutent avec des privilèges racines sur les nœuds du cluster.

* Elles peuvent être utilisées par le biais du Portail Azure, d’Azure PowerShell, de l’interface de ligne de commande Azure (CLI) ou du Kit de développement logiciel (SDK) HDInsight .NET.

* Les actions de script qui suppriment ou modifient des fichiers de service sur la machine virtuelle peuvent avoir un impact sur l’intégrité et la disponibilité du service.

Le cluster conserve un historique de tous les scripts qui ont été exécutés. L’historique facilite la recherche de l’ID d’un script pour promouvoir ou abaisser des opérations.

> [!IMPORTANT]  
> Il n’existe aucune méthode automatique pour annuler les modifications apportées par une action de script. Il faut soit inverser les modifications manuellement, soit fournir un script qui les inverse.

### <a name="script-action-in-the-cluster-creation-process"></a>Action de script dans le processus de création de cluster

Les actions de script utilisées lors de la création d’un cluster sont légèrement différentes de celles exécutées sur un cluster existant :

* Le script est automatiquement rendu persistant.

* Un échec dans le script peut provoquer l’échec du processus de création du cluster.

Le diagramme suivant illustre la phase d’exécution de l’action de script pendant le processus de création :

![Personnalisation du cluster HDInsight et procédure de création d’un cluster][img-hdi-cluster-states]

Le script est exécuté pendant la configuration de HDInsight. Le script s’exécute en parallèle sur tous les nœuds du cluster spécifiés. Il s’exécute avec des privilèges racines sur les nœuds.

Vous pouvez effectuer des opérations comme l’arrêt et le démarrage des services, notamment des services liés à Apache Hadoop. Si vous arrêtez des services, assurez-vous qu’Ambari et les autres services liés à Hadoop sont en cours d’exécution avant la fin du script. Ces services indispensables déterminent l’intégrité et l’état du cluster pendant sa création.

Durant la création du cluster, vous pouvez exécuter de nombreuses actions de script à la fois. Ces scripts sont appelés dans l’ordre dans lequel ils ont été spécifiés.

> [!IMPORTANT]  
> Les actions de script doivent se terminer dans un délai de 60 minutes, faute de quoi elles expirent. Lors de l’approvisionnement du cluster, le script est exécuté en même temps que les autres processus d’installation et de configuration. En raison de cette concurrence sur les ressources, par exemple au niveau du temps processeur ou de la bande passante, l’exécution du script risque de prendre plus de temps que dans votre environnement de développement.
>
> Pour réduire le temps d’exécution du script, évitez les tâches comme le téléchargement et la compilation d’applications à partir de la source. À la place, précompilez les applications et enregistrez le fichier binaire dans le stockage Azure.

### <a name="script-action-on-a-running-cluster"></a>Action de script sur un cluster en cours d’exécution

Un échec de script sur un cluster déjà en cours d’exécution n’entraîne pas automatiquement l’échec du cluster. À la fin d’un script, le cluster doit normalement revenir à un état en cours d’exécution. Toutefois, même si le cluster présente un état en cours d’exécution, l’échec du script peut avoir endommagé l’installation. Par exemple, un script peut avoir supprimé certains fichiers nécessaires au cluster.

Les actions de script s’exécutent avec des privilèges racines. Soyez donc sûr de comprendre ce que fait un script avant de l’appliquer à votre cluster.

Quand vous appliquez un script à un cluster, le cluster passe de l’état **En cours d’exécution** à **Accepté**, puis à **Configuration HDInsight**. Si le script s’est correctement exécuté, le cluster revient finalement à l’état **En cours d’exécution**. L’état du script est enregistré dans l’historique des actions de script. Cette information vous permet de savoir si le script a réussi ou échoué. Par exemple, l’applet de commande PowerShell `Get-AzHDInsightScriptActionHistory` affiche l’état d’un script. Cette commande renvoie des informations semblables au texte suivant :

```output
ScriptExecutionId : 635918532516474303
StartTime         : 8/14/2017 7:40:55 PM
EndTime           : 8/14/2017 7:41:05 PM
Status            : Succeeded
```

> [!IMPORTANT]  
> Si vous changez le mot de passe, l’utilisateur ou l’administrateur du cluster après la création de ce dernier, les actions de script exécutées sur ce cluster risquent d’échouer. Si des actions de script persistantes ciblent des nœuds worker, ces scripts risquent d’échouer au moment d’une mise à l’échelle du cluster.

## <a name="example-script-action-scripts"></a>Exemple de script d’action de script

Des scripts d’action de script peuvent être utilisés par les utilitaires suivants :

* Portail Azure
* Azure PowerShell
* Azure CLI
* Kit HDInsight .NET SDK

HDInsight propose des scripts pour installer les composants suivants sur des clusters HDInsight :

| Nom | Script |
| --- | --- |
| Ajouter un compte de stockage Azure |`https://hdiconfigactions.blob.core.windows.net/linuxaddstorageaccountv01/add-storage-account-v01.sh`. Consultez [Ajouter des comptes de stockage supplémentaires à HDInsight](hdinsight-hadoop-add-storage.md). |
| Installer Hue |`https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv02/install-hue-uber-v02.sh`. Consultez [Installer et utiliser Hue sur des clusters HDInsight Hadoop](hdinsight-hadoop-hue-linux.md). |
| Précharger les bibliothèques Hive |`https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh`. Consultez [Ajouter des bibliothèques Apache Hive personnalisées lors de la création de votre cluster HDInsight](hdinsight-hadoop-add-hive-libraries.md). |

## <a name="script-action-during-cluster-creation"></a>Action de script lors de la création d’un cluster

Cette section explique les différentes façons d’utiliser des actions de script quand vous créez un cluster HDInsight.

### <a name="use-a-script-action-during-cluster-creation-from-the-azure-portal"></a>Utiliser une action de script lors de la création d’un cluster à partir du portail Azure

1. Passez à la création d’un cluster en suivant les étapes décrites dans [Créer des clusters Linux dans HDInsight à l’aide du portail Azure](hdinsight-hadoop-create-linux-clusters-portal.md). Sous l’onglet **Configuration + prix**, sélectionnez **+ Ajouter une action de script**.

    ![Portail Azure - Action de script dans le cluster](./media/hdinsight-hadoop-customize-cluster-linux/azure-portal-cluster-configuration-scriptaction.png)

1. Utilisez l’entrée __Sélectionner un script__ pour sélectionner un script prédéfini. Pour utiliser un script personnalisé, sélectionnez __Personnalisé__. Entrez ensuite le __nom__ et l’__URI de script Bash__ pour votre script.

    ![Ajouter un script sous la forme du script sélectionné](./media/hdinsight-hadoop-customize-cluster-linux/hdinsight-select-script.png)

    La table suivante décrit les éléments sous la forme :

    | Propriété | Valeur |
    | --- | --- |
    | Sélectionner un script | Pour utiliser votre propre script, sélectionnez __Personnalisé__. Sinon, sélectionnez un des scripts fournis. |
    | Nom |Indiquez un nom pour l’action de script. |
    | URI de script bash |Spécifiez l’URI du script. |
    | Head/Worker/ZooKeeper |Indiquez les nœuds sur lesquels le script est exécuté : **Head**, **Worker** ou **ZooKeeper**. |
    | Paramètres |Spécifiez les paramètres, si le script le demande. |

    Utilisez l’option __Conserver cette action de script__ pour vous assurer que le script sera appliqué aux nœuds lors des opérations de mise à l’échelle.

1. Sélectionnez __Créer__ pour enregistrer le script. Vous pouvez ensuite utiliser __+ Soumettre nouveau__ pour ajouter un autre script.

    ![HDInsight - Plusieurs actions de script](./media/hdinsight-hadoop-customize-cluster-linux/multiple-scripts-actions.png)

    Lorsque vous avez terminé d’ajouter des scripts, revenez à l’onglet **Configuration + prix**.

1. Effectuez les étapes de création de cluster restantes comme vous le faites d’habitude.

### <a name="use-a-script-action-from-azure-resource-manager-templates"></a>Utiliser une action de script à partir de modèles Azure Resource Manager

Les actions de script peuvent être utilisées avec des modèles Azure Resource Manager. Pour obtenir un exemple, consultez [Créer un cluster HDInsight Linux et exécuter une action de script](https://azure.microsoft.com/resources/templates/hdinsight-linux-run-script-action/).

Dans cet exemple, l’action de script est ajoutée à l’aide du code suivant :

```json
"scriptActions": [
    {
        "name": "setenvironmentvariable",
        "uri": "[parameters('scriptActionUri')]",
        "parameters": "headnode"
    }
]
```

Obtenez plus d’informations sur le déploiement d’un modèle :

* [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md)

* [Déployer des ressources à l’aide de modèles Resource Manager et de l’interface de ligne de commande Azure](../azure-resource-manager/templates/deploy-cli.md)

### <a name="use-a-script-action-during-cluster-creation-from-azure-powershell"></a>Utiliser une action de script lors de la création d’un cluster à partir d’Azure PowerShell

Dans cette section, vous allez utiliser la cmdlet [Add-AzHDInsightScriptAction](/powershell/module/az.hdinsight/add-azhdinsightscriptaction) pour appeler des scripts afin de personnaliser un cluster. Avant de commencer, assurez-vous que vous avez installé et configuré Azure PowerShell. Pour utiliser ces commandes PowerShell, vous avez besoin du [module AZ](/powershell/azure/).

Le script suivant montre comment appliquer une action de script quand vous créez un cluster à l’aide de PowerShell :

[!code-powershell[main](../../powershell_scripts/hdinsight/use-script-action/use-script-action.ps1?range=5-90)]

La création du cluster peut prendre plusieurs minutes.

### <a name="use-a-script-action-during-cluster-creation-from-the-hdinsight-net-sdk"></a>Utiliser une action de script lors de la création d’un cluster à l’aide du kit HDInsight .NET SDK

Le kit HDInsight .NET SDK fournit des bibliothèques clientes qui facilitent l’utilisation d’HDInsight à partir d’une application .NET. Pour obtenir un exemple de code, consultez [Actions de script](/dotnet/api/overview/azure/hdinsight?view=azure-dotnet&preserve-view=true#script-actions).

## <a name="script-action-to-a-running-cluster"></a>Action de script sur un cluster en cours d’exécution

Cette section explique comment appliquer des actions de script à un cluster en cours d’exécution.

### <a name="apply-a-script-action-to-a-running-cluster-from-the-azure-portal"></a>Appliquer une action de script à un cluster en cours d’exécution à partir du portail Azure

1. Connectez-vous au [Portail Azure](https://portal.azure.com) et recherchez votre cluster.

1. Dans la vue par défaut, sous **Paramètres**, sélectionnez **Actions de script**.

1. En haut de la page **Actions de script**, sélectionnez **+ Envoyer**.

    ![Ajouter un script à un cluster en cours d’exécution](./media/hdinsight-hadoop-customize-cluster-linux/add-script-running-cluster.png)

1. Utilisez l’entrée __Sélectionner un script__ pour sélectionner un script prédéfini. Pour utiliser un script personnalisé, sélectionnez __Personnalisé__. Entrez ensuite le __nom__ et l’__URI de script Bash__ pour votre script.

    ![Ajouter un script sous la forme du script sélectionné](./media/hdinsight-hadoop-customize-cluster-linux/hdinsight-select-script.png)

    La table suivante décrit les éléments sous la forme :

    | Propriété | Valeur |
    | --- | --- |
    | Sélectionner un script | Pour utiliser votre propre script, sélectionnez __Personnalisé__. Sinon, sélectionnez un script fourni. |
    | Nom |Indiquez un nom pour l’action de script. |
    | URI de script bash |Spécifiez l’URI du script. |
    | Head/Worker/Zookeeper |Indiquez les nœuds sur lesquels le script est exécuté : **Head**, **Worker** ou **ZooKeeper**. |
    | Paramètres |Spécifiez les paramètres, si le script le demande. |

    Utilisez l’entrée __Continuer cette action de script__ pour vous assurer que le script est appliqué aux nœuds lors de la mise à l’échelle.

1. Pour finir, sélectionnez le bouton **Créer** afin d’appliquer le script au cluster.

### <a name="apply-a-script-action-to-a-running-cluster-from-azure-powershell"></a>Appliquer une action de script à un cluster en cours d’exécution à partir d’Azure PowerShell

Pour utiliser ces commandes PowerShell, vous avez besoin du [module AZ](/powershell/azure/). L’exemple suivant montre comment appliquer une action de script à un cluster en cours d’exécution :

[!code-powershell[main](../../powershell_scripts/hdinsight/use-script-action/use-script-action.ps1?range=105-117)]

Au terme de l’opération, vous voyez s’afficher des informations similaires au texte ci-dessous :

```output
OperationState  : Succeeded
ErrorMessage    :
Name            : Giraph
Uri             : https://hdiconfigactions.blob.core.windows.net/linuxgiraphconfigactionv01/giraph-installer-v01.sh
Parameters      :
NodeTypes       : {HeadNode, WorkerNode}
```

### <a name="apply-a-script-action-to-a-running-cluster-from-the-azure-cli"></a>Appliquer une action de script à un cluster en cours d’exécution à partir d’Azure CLI

Avant de commencer, assurez-vous que vous avez installé et configuré Azure CLI. Vérifiez que vous avez la version la plus récente. Pour plus d’informations, consultez la rubrique [Installation de l’interface de ligne de commande Azure (CLI)](/cli/azure/install-azure-cli).

1. Authentifiez-vous auprès de votre abonnement Azure :

    ```azurecli
    az login
    ```

1. Appliquez une action de script à un cluster en cours d’exécution :

    ```azurecli
    az hdinsight script-action execute --cluster-name CLUSTERNAME --name SCRIPTNAME --resource-group RESOURCEGROUP --roles ROLES
    ```

    Les rôles valides sont `headnode`, `workernode`, `zookeepernode` et `edgenode`. Si le script doit être appliqué à plusieurs types de nœuds, séparez les rôles par une espace. Par exemple : `--roles headnode workernode`.

    Pour rendre le script persistant, ajoutez le paramètre `--persist-on-success`. Vous pouvez également continuer le script à une date ultérieure à l’aide de `az hdinsight script-action promote`.

### <a name="apply-a-script-action-to-a-running-cluster-by-using-rest-api"></a>Appliquer une action de script à un cluster en cours d’exécution à l’aide de l’API REST

Consultez [API REST de cluster dans Azure HDInsight](/rest/api/hdinsight/hdinsight-cluster).

### <a name="apply-a-script-action-to-a-running-cluster-from-the-hdinsight-net-sdk"></a>Appliquer une action de script à un cluster en cours d’exécution à partir du kit HDInsight .NET SDK

Pour obtenir un exemple d’utilisation du SDK .NET afin d’appliquer des scripts à un cluster, consultez [Appliquer une action de script sur un cluster HDInsight Linux en cours d’exécution](https://github.com/Azure-Samples/hdinsight-dotnet-script-action).

## <a name="view-history-and-promote-and-demote-script-actions"></a>Afficher l’historique et promouvoir ou abaisser des actions de script

### <a name="the-azure-portal"></a>Le portail Azure

1. Connectez-vous au [Portail Azure](https://portal.azure.com) et recherchez votre cluster.

1. Dans la vue par défaut, sous **Paramètres**, sélectionnez **Actions de script**.

1. Un historique des scripts pour ce cluster s’affiche dans la section des actions de script. Ces informations comprennent une liste de scripts persistants. La capture d’écran suivante montre que le script Solr a été exécuté sur ce cluster. La capture d’écran n’affiche aucun script persistant.

    ![Actions de script du portail - Historique des soumissions](./media/hdinsight-hadoop-customize-cluster-linux/script-action-history.png)

1. Sélectionnez un script dans l’historique pour afficher la section **Propriétés** du script. Dans le coin supérieur de l’écran, vous pouvez réexécuter le script ou le promouvoir.

    ![Propriétés des actions de script - Promouvoir](./media/hdinsight-hadoop-customize-cluster-linux/promote-script-actions.png)

1. Vous pouvez également sélectionner le bouton de sélection ( **…** ) à droite des entrées dans la section Actions de script pour effectuer des actions.

    ![Actions de script persistantes - Supprimer](./media/hdinsight-hadoop-customize-cluster-linux/hdi-delete-promoted-sa.png)

### <a name="azure-powershell"></a>Azure PowerShell

| Applet de commande | Fonction |
| --- | --- |
| `Get-AzHDInsightPersistedScriptAction` |Récupérer des informations sur les actions de script persistantes. Cette cmdlet n’annule pas les actions effectuées par un script, elle supprime uniquement l’indicateur persistant.|
| `Get-AzHDInsightScriptActionHistory` |Récupérer l’historique des actions de script appliquées au cluster ou les informations d’un script spécifique. |
| `Set-AzHDInsightPersistedScriptAction` |Promouvoir une action de script `ad hoc` en action de script persistante. |
| `Remove-AzHDInsightPersistedScriptAction` |Rétrograder une action de script persistante en action `ad hoc`. |

L’exemple de script suivant illustre l’utilisation des applets de commande pour promouvoir, puis abaisser un script.

[!code-powershell[main](../../powershell_scripts/hdinsight/use-script-action/use-script-action.ps1?range=123-140)]

### <a name="azure-cli"></a>Azure CLI

| Commande | Description |
| --- | --- |
| [`az hdinsight script-action delete`](/cli/azure/hdinsight/script-action#az-hdinsight-script-action-delete) |Supprime l’action de script persistante spécifiée du cluster. Cette commande n’annule pas les actions effectuées par un script, elle supprime uniquement l’indicateur persistant.|
|[`az hdinsight script-action execute`](/cli/azure/hdinsight/script-action#az-hdinsight-script-action-execute)|Exécute des actions de script sur le cluster HDInsight spécifié.|
| [`az hdinsight script-action list`](/cli/azure/hdinsight/script-action#az-hdinsight-script-action-list) |Répertorie toutes les actions de script persistantes pour le cluster spécifié. |
|[`az hdinsight script-action list-execution-history`](/cli/azure/hdinsight/script-action#az-hdinsight-script-action-list-execution-history)|Répertorie l’historique d’exécution de tous les scripts pour le cluster spécifié.|
|[`az hdinsight script-action promote`](/cli/azure/hdinsight/script-action#az-hdinsight-script-action-promote)|Promeut l’exécution de script ad hoc spécifiée dans un script persistant.|
|[`az hdinsight script-action show-execution-details`](/cli/azure/hdinsight/script-action#az-hdinsight-script-action-show-execution-details)|Obtient le détail de l’exécution du script pour l’ID d’exécution du script donné.|

### <a name="hdinsight-net-sdk"></a>Kit HDInsight .NET SDK

Pour obtenir un exemple d’utilisation du SDK .NET afin de récupérer l’historique des scripts d’un cluster, et de promouvoir ou d’abaisser des scripts, consultez [Appliquer une action de script sur un cluster HDInsight Linux en cours d’exécution](https://github.com/Azure-Samples/hdinsight-dotnet-script-action).

> [!NOTE]  
> Cet exemple montre également comment installer une application HDInsight à l’aide du SDK .NET.

## <a name="next-steps"></a>Étapes suivantes

* [Développer des scripts d’action de script pour HDInsight](hdinsight-hadoop-script-actions-linux.md)
* [Ajouter un stockage supplémentaire à un cluster HDInsight](hdinsight-hadoop-add-storage.md)
* [Résoudre les problèmes liés aux actions de script](troubleshoot-script-action.md)

[img-hdi-cluster-states]: ./media/hdinsight-hadoop-customize-cluster-linux/cluster-provisioning-states.png "Procédure de création d’un cluster"
