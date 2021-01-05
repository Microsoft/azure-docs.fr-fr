---
title: Bien démarrer avec Azure CLI pour Batch
description: Obtenez une brève introduction aux commandes Batch dans l’interface de ligne de commande Azure pour la gestion des ressources du service Azure Batch.
ms.topic: how-to
ms.date: 07/24/2018
ms.custom: H1Hack27Feb2017, devx-track-azurecli
ms.openlocfilehash: bee25d9b8985f1627a5cfc05bfb336b83be60f74
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92144751"
---
# <a name="manage-batch-resources-with-azure-cli"></a>Gérer les ressources Batch avec Azure CLI

L’interface de ligne de commande Azure (Azure CLI) est l’expérience de ligne de commande d’Azure pour gérer les ressources Azure. Elle peut être utilisée sur macOS, Linux et Windows. Azure CLI est optimisé pour gérer et administrer les ressources Azure à partir de la ligne de commande. Vous pouvez utiliser cette interface pour gérer vos comptes Azure Batch et des ressources telles que les pools, les travaux et les tâches. Avec l’interface de ligne de commande Azure, vous pouvez écrire le script de nombre des tâches que vous réalisez avec les API Batch, le portail Azure et les applets de commande PowerShell pour Batch.

Cet article fournit une vue d’ensemble de l’utilisation de [Azure CLI version 2.0](/cli/azure) avec Batch. Consultez [Bien démarrer avec Azure CLI](/cli/azure/get-started-with-azure-cli) pour une vue d’ensemble de l’utilisation de l’interface de ligne de commande avec Azure.

## <a name="set-up-the-azure-cli"></a>Configuration de l’interface de ligne de commande Azure

Vous pouvez également exécuter l’interface Azure CLI la plus récente dans [Azure Cloud Shell](../cloud-shell/overview.md). Pour installer Azure CLI localement, suivez les instructions fournies dans la section [Installer Azure CLI](/cli/azure/install-azure-cli).

> [!TIP]
> Nous vous recommandons de mettre à jour fréquemment votre installation d’interface de ligne de commande Azure pour tirer parti des améliorations et des mises à jour de service.
> 
> 

## <a name="command-help"></a>Aide relative aux commandes

Vous pouvez afficher le texte d’aide de toutes les commandes de l’interface de ligne de commande Azure en ajoutant `-h` à la commande. Ignorez les autres options. Par exemple :

* Pour obtenir l’aide de la commande `az`, entrez : `az -h`
* Pour obtenir la liste de toutes les commandes Batch dans l’interface de ligne de commande, utilisez : `az batch -h`
* Pour obtenir de l’aide sur la création d’un compte Batch, entrez : `az batch account create -h`

En cas de doute, utilisez l’option de ligne de commande `-h` pour obtenir de l’aide sur toute commande de l’interface de ligne de commande Azure.



Consultez également la documentation de référence Azure CLI pour plus d’informations sur les [commandes Azure CLI pour Batch](/cli/azure/batch). 

## <a name="log-in-and-authenticate"></a>Connexion et authentification

Pour utiliser Azure CLI avec Batch, vous devez vous connecter et vous authentifier. Vous devez suivre deux étapes simples :

1. **Connectez-vous à Azure.** La connexion à Azure vous donne accès aux commandes Azure Resource Manager, notamment aux commandes du [service Batch Management](batch-management-dotnet.md).  
2. **Connectez-vous à votre compte Batch.** La connexion à votre compte Batch vous donne accès aux commandes du service Batch.   

### <a name="log-in-to-azure"></a>Connexion à Azure

Il existe différentes façons de se connecter à Azure, décrites en détail dans [Se connecter avec Azure CLI](/cli/azure/authenticate-azure-cli) :

1. [Connexion interactive](/cli/azure/authenticate-azure-cli). Connectez-vous de manière interactive lorsque vous exécutez des commandes Azure CLI vous-même à partir de la ligne de commande.
2. [Connexion avec un principal de service](/cli/azure/authenticate-azure-cli). Connectez-vous avec un principal de service lorsque vous exécutez des commandes Azure CLI à partir d’un script ou d’une application.

Dans le cadre de cet article, nous montrons comment se connecter à Azure de manière interactive. Entrez [az login](/cli/azure/reference-index#az-login) dans la ligne de commande :

```azurecli
# Log in to Azure and authenticate interactively.
az login
```

La commande `az login` renvoie un jeton pouvant vous servir lors d’authentifications, comme vous pouvez le constater ici. Suivez les instructions fournies pour ouvrir une page web et envoyer le jeton à Azure :

![Connexion à Azure](./media/batch-cli-get-started/az-login.png)

Les exemples listés dans la section Exemples de scripts de l’interpréteur de commandes montrent également comment démarrer votre session Azure CLI en vous connectant à Azure de manière interactive. Une fois que vous êtes connecté, vous pouvez appeler des commandes pour travailler avec des ressources Batch Management, y compris les comptes Batch, les clés, les packages d’applications et les quotas.  

### <a name="log-in-to-your-batch-account"></a>Connexion à votre compte Batch

Pour utiliser Azure CLI pour gérer les ressources Batch, telles que les pools, les travaux et les tâches, vous devez vous connecter à votre compte Batch et vous authentifier. Pour vous connecter au service Batch, utilisez la commande [az batch account login](/cli/azure/batch/account#az-batch-account-login). 

Deux options s’offrent à vous pour l’authentification sur votre compte Batch :

- **Via l’authentification Azure Active Directory (Azure AD)** 

    L’authentification avec Azure AD est l’option par défaut lorsque vous utilisez Azure CLI avec Batch. Elle est recommandée pour la plupart des scénarios. 
    
    Lorsque vous vous connectez à Azure de manière interactive, comme décrit dans la section précédente, vos informations d’identification sont mises en cache, afin qu’Azure CLI puisse vous connecter à votre compte Batch à l’aide de ces mêmes informations d’identification. Si vous vous connectez à Azure à l’aide d’un principal de service, ces informations d’identification sont également utilisées pour vous connecter à votre compte Batch.

    L’avantage d’Azure AD est que le contrôle d’accès se fait en fonction du rôle Azure (Azure RBAC). Grâce à Azure RBAC, l’accès dont bénéficie un utilisateur dépend de son rôle, et non du fait qu’il possède ou non les clés du compte. Au lieu de gérer les clés de compte, vous pouvez gérer les rôles Azure et permettre à Azure AD de gérer l’accès et l’authentification.  

     Pour vous connecter à votre compte Batch à l’aide d’Azure AD, appelez la commande [az batch account login](/cli/azure/batch/account#az-batch-account-login) : 

    ```azurecli
    az batch account login -g myresource group -n mybatchaccount
    ```

- **Via l’authentification par clé partagée**

    [L’authentification par clé partagée](/rest/api/batchservice/authenticate-requests-to-the-azure-batch-service#authentication-via-shared-key) utilise des clés d’accès à votre compte pour authentifier les commandes Azure CLI pour le service Batch.

    Si vous créez des scripts Azure CLI pour automatiser l’appel des commandes Batch, vous pouvez utiliser soit l’authentification par clé partagée, soit un principal de service Azure AD. Dans certains scénarios, le recours à l’authentification par clé partagée peut être plus simple que la création d’un principal de service.  

    Pour vous connecter en utilisant l’authentification par clé partagée, incluez l’option `--shared-key-auth` sur la ligne de commande :

    ```azurecli
    az batch account login -g myresourcegroup -n mybatchaccount --shared-key-auth
    ```

Les exemples listés dans la section Exemples de scripts de l’interpréteur de commandes montrent comment se connecter à votre compte Batch avec Azure CLI, à la fois via Azure AD et par clé partagée.

## <a name="use-azure-batch-cli-extension-commands"></a>Utilisez les commandes de l’extension CLI Azure Batch

En installant l’extension CLI Azure Batch, vous pouvez utiliser l’interface Azure CLI pour exécuter des programmes de traitement par lots de bout en bout sans écrire de code. Les commandes Batch prises en charge par l’extension permettent d’utiliser des modèles JSON pour créer des pools, travaux et des tâches avec Azure CLI. Vous pouvez également utiliser les commandes CLI d’extension pour charger les fichiers d’entrée de travail sur le compte de Stockage Azure associé au compte Batch. À partir de ce compte, vous pouvez également télécharger les fichiers de sortie de travail. Pour en savoir plus, consultez [Utiliser des modèles d’interface CLI Azure Batch et le transfert de fichiers (préversion)](batch-cli-templates.md).

## <a name="script-examples"></a>Exemples de scripts

Consultez les [exemples de scripts de l’interface de ligne de commande](./scripts/batch-cli-sample-create-account.md) pour Batch pour accomplir des tâches courantes. Ces exemples couvrent la plupart des commandes disponibles dans Azure CLI pour Batch pour créer et gérer des comptes, des pools, des travaux et des tâches.

## <a name="json-files-for-resource-creation"></a>Fichiers JSON pour la création de ressources

Lorsque vous créez des ressources Batch comme des pools et des travaux, vous pouvez spécifier un fichier JSON contenant la configuration de la nouvelle ressource au lieu de transmettre ses paramètres en tant qu’options de ligne de commande. Par exemple :

```azurecli
az batch pool create my_batch_pool.json
```

Si vous pouvez créer la plupart des ressources Batch en utilisant uniquement des options de ligne de commande, certaines fonctionnalités requièrent de spécifier un fichier au format JSON contenant les détails des ressources. Par exemple, vous devez utiliser un fichier JSON si vous souhaitez spécifier des fichiers de ressources pour une tâche de démarrage.

Pour afficher la syntaxe JSON requise pour créer une ressource, consultez la documentation [Informations de référence de l’API REST Batch][rest_api]. Chaque rubrique « Ajouter *type de ressource*  » dans la référence de l’API REST contient des exemples de scripts JSON pour la création de la ressource en question. Vous pouvez utiliser ces exemples de scripts JSON comme modèles pour les fichiers JSON à utiliser avec Azure CLI. Par exemple, pour afficher la syntaxe JSON pour la création d’un pool, reportez-vous à [Ajouter un pool à un compte][rest_add_pool].

Pour un exemple de script qui spécifie un fichier JSON, consultez [Exécuter un travail et des tâches avec Batch](./scripts/batch-cli-sample-run-job.md).

> [!NOTE]
> Si vous spécifiez un fichier JSON lorsque vous créez une ressource, tous les autres paramètres que vous spécifiez sur la ligne de commande de cette ressource sont ignorés.
> 
> 

## <a name="efficient-queries-for-batch-resources"></a>Requêtes efficaces pour les ressources Batch

Chaque type de ressource Batch prend en charge une commande `list` qui interroge votre compte Batch et répertorie les ressources de ce type. Par exemple, vous pouvez répertorier les pools de votre compte et les tâches d’un travail :

```azurecli
az batch pool list
az batch task list --job-id job001
```

Lorsque vous interrogez le service Batch avec une opération `list`, vous pouvez spécifier une clause OData pour limiter la quantité de données renvoyées. Comme tout le filtrage se produit côté serveur, seules les données que vous demandez sont transmises. Utilisez ces clauses pour économiser la bande passante (et par conséquent gagner du temps) lorsque vous effectuez des opérations de liste.

Le tableau suivant décrit les clauses OData prises en charge par le service Batch :

| Clause | Description |
|---|---|
| `--select-clause [select-clause]` | Retourne un sous-ensemble de propriétés pour chaque entité. |
| `--filter-clause [filter-clause]` | Retourne uniquement les entités qui correspondent à l’expression OData spécifiée. |
| `--expand-clause [expand-clause]` | Obtient les informations d’entité dans un seul appel REST sous-jacent. Pour le moment, la clause expand ne prend en charge que la propriété `stats`. |

Pour obtenir un exemple de script qui montre comment utiliser une clause OData, consultez [Exécuter un travail et des tâches avec Batch](./scripts/batch-cli-sample-run-job.md).

Pour plus d’informations sur l’exécution de requêtes de liste efficaces avec des clauses OData, consultez [Interroger efficacement le service Azure Batch](batch-efficient-list-queries.md).

## <a name="troubleshooting-tips"></a>Conseils de dépannage

Les conseils suivants peuvent vous aider lors de la résolution de problèmes sur Azure CLI :

* Utilisez `-h` pour obtenir le **texte d’aide** d’une commande d’interface de ligne de commande.
* Utilisez `-v` et `-vv` pour afficher la sortie de la commande **verbose** . Lorsque l’indicateur `-vv` est inclus, Azure CLI affiche les requêtes REST réelles et les réponses correspondantes. Ces commutateurs sont pratiques pour afficher une sortie complète des erreurs.
* L’option `--json` permet d’afficher la **sortie de commande au format JSON** . Par exemple, `az batch pool show pool001 --json` affiche les propriétés de pool001 au format JSON. Vous pouvez ensuite copier et modifier cette sortie pour l’utiliser dans un `--json-file` (voir la section Fichiers JSON plus haut dans cet article).
<!---Loc Comment: Please, check link [JSON files] since it's not redirecting to any location.--->

## <a name="next-steps"></a>Étapes suivantes

* Consultez la [Documentation sur Azure CLI](/cli/azure).
* Apprenez-en davantage sur le [flux de travail et les ressources principales du service Batch](batch-service-workflow-features.md), telles que les pools, les nœuds, les travaux et les tâches.
* Découvrez l’utilisation des modèles Batch afin de créer des pools, des travaux et des tâches sans écrire de code dans la section [Utiliser des modèles d’interface CLI Azure Batch et le transfert de fichiers (préversion)](batch-cli-templates.md).

[github_readme]: https://github.com/Azure/azure-xplat-cli/blob/dev/README.md
[rest_api]: /rest/api/batchservice/
[rest_add_pool]: /rest/api/batchservice/pool/add
