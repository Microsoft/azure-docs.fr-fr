---
title: Modifier les clés d’accès au compte de stockage
titleSuffix: Azure Machine Learning
description: Découvrez comment modifier les clés d’accès du compte Stockage Azure utilisé par votre espace de travail. Azure Machine Learning utilise un compte Stockage Azure pour stocker les données et les modèles.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: aashishb
author: aashishb
ms.reviewer: larryfr
ms.date: 06/19/2020
ms.openlocfilehash: 78829ae52d74cf6ec58c12779c51bca9a98e0af1
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96450793"
---
# <a name="regenerate-storage-account-access-keys"></a>Régénérer des clés d’accès de compte de stockage


Découvrez comment changer les clés d’accès des comptes Stockage Azure utilisées par Azure Machine Learning. Azure Machine Learning peut utiliser des comptes de stockage pour stocker des données ou des modèles formés.

Pour des raisons de sécurité, vous devrez peut-être modifier les clés d’accès d’un compte Stockage Azure. Lorsque vous régénérez la clé d’accès, Azure Machine Learning doit être mis à jour pour utiliser la nouvelle clé. Azure Machine Learning peut utiliser le compte de stockage à la fois pour le stockage des modèles et comme banque de données.

> [!IMPORTANT]

> Les informations d’identification enregistrées avec les magasins de données sont enregistrées dans votre coffre de clés Azure associé à l’espace de travail. Si la [suppression réversible](../key-vault/general/soft-delete-overview.md) est activée pour votre coffre de clés, cet article fournit des instructions pour la mise à jour des informations d’identification. Si vous désinscrivez le magasin de données et essayez de le réinscrire sous le même nom, l’action échoue. Consultez [Activer la suppression réversible pour un coffre de clés existant]( https://docs.microsoft.com/azure/key-vault/general/soft-delete-change#turn-on-soft-delete-for-an-existing-key-vault) pour savoir comment activer la suppression réversible dans ce scénario.

## <a name="prerequisites"></a>Prérequis

* Un espace de travail Azure Machine Learning. Pour plus d’informations, consultez l’article [Créer un espace de travail](how-to-manage-workspace.md).

* Le [Kit de développement logiciel (SDK) Azure Machine Learning](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py).

* [L’extension CLI Azure Machine Learning](reference-azure-machine-learning-cli.md).

> [!NOTE]
> Les extraits de code de ce document ont été testés avec la version 1.0.83 du Kit de développement logiciel (SDK) Python.

<a id="whattoupdate"></a> 

## <a name="what-needs-to-be-updated"></a>Éléments à mettre à jour

Les comptes de stockage peuvent être utilisés par l’espace de travail Azure Machine Learning (stockage des journaux, modèles, instantanés, etc.) et comme magasin de données. Le processus de mise à jour de l’espace de travail est une commande unique de l’interface de ligne de commande Azure et peut être exécuté après la mise à jour de la clé de stockage. Le processus de mise à jour des banques de données est plus complexe et nécessite de découvrir quelles banques de données utilisent actuellement le compte de stockage, puis de les réinscrire.

> [!IMPORTANT]
> Mettez à jour l’espace de travail à l’aide de l’interface de ligne de commande Azure, et les banques de données à l’aide de Python simultanément. La mise à jour de l’un ou l’autre n’est pas suffisante et peut entraîner des erreurs tant que les deux ne sont pas mis à jour.

Pour découvrir les comptes de stockage qui sont utilisés par vos banques de données, utilisez le code suivant :

```python
import azureml.core
from azureml.core import Workspace, Datastore

ws = Workspace.from_config()

default_ds = ws.get_default_datastore()
print("Default datstore: " + default_ds.name + ", storage account name: " +
      default_ds.account_name + ", container name: " + default_ds.container_name)

datastores = ws.datastores
for name, ds in datastores.items():
    if ds.datastore_type == "AzureBlob":
        print("Blob store - datastore name: " + name + ", storage account name: " +
              ds.account_name + ", container name: " + ds.container_name)
    if ds.datastore_type == "AzureFile":
        print("File share - datastore name: " + name + ", storage account name: " +
              ds.account_name + ", container name: " + ds.container_name)
```

Ce code recherche les banques de données inscrites qui utilisent Stockage Azure et répertorie les informations suivantes :

* Nom de la banque de données : nom de la base de données sous laquelle le compte de stockage est inscrit.
* Nom du compte de stockage : Nom du compte Stockage Azure.
* Conteneur : conteneur dans le compte de stockage qui est utilisé par cette inscription.

Il indique également si le magasin de stockage est destiné à un objet blob Azure ou à un partage de fichiers Azure, car il existe différentes méthodes pour réinscrire chaque type de magasin de stockage.

S’il existe une entrée pour le compte de stockage dont vous prévoyez de régénérer les clés d’accès, enregistrez le nom de la banque de données, le nom du compte de stockage et le nom du conteneur.

## <a name="update-the-access-key"></a>Mettre à jour la clé d’accès

Pour mettre à jour Azure Machine Learning afin d’utiliser la nouvelle clé, effectuez les étapes suivantes :

> [!IMPORTANT]
> Effectuez toutes les étapes, en mettant à jour l’espace de travail à l’aide de l’interface de ligne de commande et les bases de données à l’aide de Python. La mise à jour de l’un ou l’autre peut entraîner des erreurs tant que les deux ne sont pas mis à jour.

1. Générez à nouveau la clé. Pour plus d’informations sur la regénération d’une clé d’accès, consultez [Gérer les clés d’accès d’un compte de stockage](../storage/common/storage-account-keys-manage.md). Enregistrez la nouvelle clé.

1. L’espace de travail Azure Machine Learning synchronise automatiquement la nouvelle clé et commence à l’utiliser après une heure. Pour forcer l’espace de travail à synchroniser la nouvelle clé immédiatement, suivez les étapes suivantes :

    1. Pour vous connecter à l’abonnement Azure qui contient votre espace de travail en utilisant la commande Azure CLI suivante :

        ```azurecli-interactive
        az login
        ```

        [!INCLUDE [select-subscription](../../includes/machine-learning-cli-subscription.md)]

    1. Pour mettre à jour l’espace de travail afin d’utiliser la nouvelle clé, utilisez la commande suivante. Remplacez `myworkspace` par le nom de votre espace de travail Azure Machine Learning et remplacez `myresourcegroup` par le nom du groupe de ressources Azure qui contient l’espace de travail.

        ```azurecli-interactive
        az ml workspace sync-keys -w myworkspace -g myresourcegroup
        ```

        [!INCLUDE [install extension](../../includes/machine-learning-service-install-extension.md)]

        Cette commande synchronise automatiquement les nouvelles clés du compte de stockage Azure utilisé par l’espace de travail.

1. Vous pouvez réenregistrer les magasins de données qui utilisent le compte de stockage via le kit de développement logiciel (SDK) ou [le studio Azure Machine Learning](https://ml.azure.com).
    1. **Pour réinscrire les magasins de données via le kit de développement logiciel (SDK) Python**, utilisez les valeurs de la section [Éléments à mettre à jour](#whattoupdate) et la clé de l’étape 1 avec le code suivant. 
    
        Puisque `overwrite=True` est spécifié, ce code écrase l’inscription existante et la met à jour pour utiliser la nouvelle clé.
    
        ```python
        # Re-register the blob container
        ds_blob = Datastore.register_azure_blob_container(workspace=ws,
                                                  datastore_name='your datastore name',
                                                  container_name='your container name',
                                                  account_name='your storage account name',
                                                  account_key='new storage account key',
                                                  overwrite=True)
        # Re-register file shares
        ds_file = Datastore.register_azure_file_share(workspace=ws,
                                              datastore_name='your datastore name',
                                              file_share_name='your container name',
                                              account_name='your storage account name',
                                              account_key='new storage account key',
                                              overwrite=True)
        
        ```
    
    1. **Pour réenregistrer les magasins de données via le studio**, sélectionnez **Magasins de données** dans le volet gauche du studio. 
        1. Sélectionnez le magasin de données à mettre à jour.
        1. Sélectionnez le bouton **Mettre à jour les informations d’identification** en haut à gauche. 
        1. Utilisez votre nouvelle clé d’accès de l’étape 1 pour remplir le formulaire et cliquez sur **Enregistrer**.
        
            Si vous mettez à jour les informations d’identification de votre **magasin de données par défaut**, effectuez cette étape et répétez l’étape 2b pour resynchroniser votre nouvelle clé avec le magasin de données par défaut de l’espace de travail. 

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’inscription des banques de données, consultez la documentation de référence de la classe [`Datastore`](/python/api/azureml-core/azureml.core.datastore%28class%29?preserve-view=true&view=azure-ml-py).