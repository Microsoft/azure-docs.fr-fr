---
title: Gérer un compte Azure Data Lake Storage Gen1 – Azure CLI
description: Utilisez Azure CLI pour créer un compte Data Lake Storage Gen1 et effectuer des opérations de base.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 06/27/2018
ms.author: twooley
ms.openlocfilehash: de98e25cf5703a43282e551a0eda20d7767c6ce8
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92103626"
---
# <a name="get-started-with-azure-data-lake-storage-gen1-using-the-azure-cli"></a>Bien démarrer avec Azure Data Lake Storage Gen1 à l’aide d’Azure CLI

[!INCLUDE [data-lake-storage-gen1-rename-note.md](../../includes/data-lake-storage-gen1-rename-note.md)]

> [!div class="op_single_selector"]
> * [Portail](data-lake-store-get-started-portal.md)
> * [PowerShell](data-lake-store-get-started-powershell.md)
> * [Azure CLI](data-lake-store-get-started-cli-2.0.md)
>
> 

Découvrez comment utiliser Azure CLI pour créer un compte Azure Data Lake Storage Gen1 et effectuer des opérations de base telles que la création de dossiers, le chargement et le téléchargement de fichiers de données, la suppression de votre compte, etc. Pour plus d’informations sur Data Lake Store Gen1, consultez [Vue d’ensemble de Data Lake Storage Gen1](data-lake-store-overview.md).

L’interface de ligne de commande Azure (Azure CLI) est l’expérience de ligne de commande d’Azure pour gérer les ressources Azure. Elle peut être utilisée sur macOS, Linux et Windows. Pour plus d’informations, consultez [Présentation d’Azure CLI](/cli/azure). Pour obtenir la liste complète des commandes et de la syntaxe, consultez la [Référence Azure Data Lake Storage Gen1 CLI](/cli/azure/dls).


## <a name="prerequisites"></a>Prérequis
Avant de commencer cet article, vous devez disposer des éléments suivants :

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).

* **Azure CLI** - Voir [Installer Azure CLI](/cli/azure/install-azure-cli) pour plus d’instructions.

## <a name="authentication"></a>Authentification

Pour l’authentification auprès de Data Lake Storage Gen1, cet article utilise une approche plus simple où vous vous connectez en tant qu’utilisateur final. Le niveau d’accès au compte et au système de fichiers Data Lake Storage Gen1 est alors régi par le niveau d’accès de l’utilisateur connecté. Cependant, il existe d’autres approches pour l’authentification sur Data Lake Storage Gen1, à savoir **l’authentification de l’utilisateur final** ou **l’authentification de service à service**. Pour obtenir des instructions et plus d’informations sur l’authentification, consultez [l’authentification de l’utilisateur final](data-lake-store-end-user-authenticate-using-active-directory.md) ou [l’authentification de service à service](./data-lake-store-service-to-service-authenticate-using-active-directory.md).


## <a name="log-in-to-your-azure-subscription"></a>Connexion à votre abonnement Azure

1. Connectez-vous à votre abonnement Azure.

    ```azurecli
    az login
    ```

    Vous obtenez un code que vous utiliserez à l’étape suivante. Utilisez un navigateur web pour ouvrir la page https://aka.ms/devicelogin et entrez le code pour vous authentifier. Vous êtes invité à vous connecter en utilisant vos informations d’identification.

2. Une fois que vous êtes connecté, la fenêtre répertorie tous les abonnements Azure qui sont associés à votre compte. Exécutez la commande suivante pour utiliser un abonnement spécifique.
   
    ```azurecli
    az account set --subscription <subscription id> 
    ```

## <a name="create-an-azure-data-lake-storage-gen1-account"></a>Créer un compte Azure Data Lake Storage Gen1

1. Créez un groupe de ressources. Dans la commande suivante, définissez des valeurs de paramètre que vous souhaitez utiliser. Si le nom de l'emplacement contient des espaces, entourez-le de guillemets. Par exemple, « USA Est 2 ». 
   
    ```azurecli
    az group create --location "East US 2" --name myresourcegroup
    ```

2. Créez le compte Data Lake Storage Gen1.
   
    ```azurecli
    az dls account create --account mydatalakestoragegen1 --resource-group myresourcegroup
    ```

## <a name="create-folders-in-a-data-lake-storage-gen1-account"></a>Créer des dossiers dans un compte Data Lake Storage Gen1

Vous pouvez créer des dossiers dans votre compte Azure Data Lake Storage Gen1 pour gérer et stocker des données. Utilisez la commande suivante pour créer un dossier nommé **mynewfolder** à la racine du compte Data Lake Storage Gen1.

```azurecli
az dls fs create --account mydatalakestoragegen1 --path /mynewfolder --folder
```

> [!NOTE]
> Le paramètre `--folder` permet de s’assurer que la commande crée un dossier. Si ce paramètre n’est pas présent, la commande crée un fichier vide nommé mynewfolder à la racine du compte Data Lake Storage Gen1.
> 
>

## <a name="upload-data-to-a-data-lake-storage-gen1-account"></a>Charger des données sur un compte Data Lake Storage Gen1

Vous pouvez charger des données dans Data Lake Storage Gen1 directement à la racine ou dans un dossier que vous avez créé dans le compte. Les extraits de code ci-dessous montrent comment charger des exemples de données dans le dossier (**mynewfolder**) que vous avez créé dans la section précédente.

Si vous recherchez des exemples de données à charger, vous pouvez récupérer le dossier **Données Ambulance** dans le [Référentiel Git Azure Data Lake](https://github.com/MicrosoftBigData/usql/tree/master/Examples/Samples/Data/AmbulanceData). Téléchargez le fichier et stockez-le dans un répertoire local sur votre ordinateur, comme C:\sampledata\.

```azurecli
az dls fs upload --account mydatalakestoragegen1 --source-path "C:\SampleData\AmbulanceData\vehicle1_09142014.csv" --destination-path "/mynewfolder/vehicle1_09142014.csv"
```

> [!NOTE]
> Pour la destination, vous devez spécifier le chemin d’accès complet, y compris le nom de fichier.
> 
>


## <a name="list-files-in-a-data-lake-storage-gen1-account"></a>Lister les fichiers dans un compte Data Lake Storage Gen1

Utilisez la commande suivante pour lister les fichiers dans un compte Data Lake Storage Gen1.

```azurecli
az dls fs list --account mydatalakestoragegen1 --path /mynewfolder
```

Le résultat doit ressembler à ceci :

```output
[
    {
        "accessTime": 1491323529542,
        "aclBit": false,
        "blockSize": 268435456,
        "group": "1808bd5f-62af-45f4-89d8-03c5e81bac20",
        "length": 1589881,
        "modificationTime": 1491323531638,
        "msExpirationTime": 0,
        "name": "mynewfolder/vehicle1_09142014.csv",
        "owner": "1808bd5f-62af-45f4-89d8-03c5e81bac20",
        "pathSuffix": "vehicle1_09142014.csv",
        "permission": "770",
        "replication": 1,
        "type": "FILE"
    }
]
```

## <a name="rename-download-and-delete-data-from-a-data-lake-storage-gen1-account"></a>Renommer, télécharger et supprimer des données d’un compte Data Lake Storage Gen1 

* Utilisez la commande suivante **pour renommer un fichier** :
  
    ```azurecli
    az dls fs move --account mydatalakestoragegen1 --source-path /mynewfolder/vehicle1_09142014.csv --destination-path /mynewfolder/vehicle1_09142014_copy.csv
    ```

* Utilisez la commande suivante **pour télécharger un fichier**. Assurez-vous que le chemin de destination que vous spécifiez existe.
  
    ```azurecli     
    az dls fs download --account mydatalakestoragegen1 --source-path /mynewfolder/vehicle1_09142014_copy.csv --destination-path "C:\mysampledata\vehicle1_09142014_copy.csv"
    ```

    > [!NOTE]
    > La commande crée le dossier de destination, si celui-ci n’existe pas.
    > 
    >

* Utilisez la commande suivante **pour supprimer un fichier** :
  
    ```azurecli
    az dls fs delete --account mydatalakestoragegen1 --path /mynewfolder/vehicle1_09142014_copy.csv
    ```

    Si vous souhaitez supprimer simultanément le dossier **mynewfolder** et le fichier **vehicle1_09142014_copy.csv** en une seule commande, utilisez le paramètre --recurse

    ```azurecli
    az dls fs delete --account mydatalakestoragegen1 --path /mynewfolder --recurse
    ```

## <a name="work-with-permissions-and-acls-for-a-data-lake-storage-gen1-account"></a>Fonctionne avec les autorisations et les listes ACL d’un compte Data Lake Storage Gen1

Dans cette section, vous allez apprendre à gérer les listes ACL et les autorisations à l’aide d’Azure CLI. Pour obtenir une présentation détaillée sur la façon dont les listes ACL sont implémentées dans Azure Data Lake Storage Gen1, consultez [Contrôle d’accès dans Azure Data Lake Storage Gen1](data-lake-store-access-control.md).

* **Pour mettre à jour le propriétaire d’un fichier/dossier**, utilisez la commande suivante :

    ```azurecli
    az dls fs access set-owner --account mydatalakestoragegen1 --path /mynewfolder/vehicle1_09142014.csv --group 80a3ed5f-959e-4696-ba3c-d3c8b2db6766 --owner 6361e05d-c381-4275-a932-5535806bb323
    ```

* **Pour mettre à jour les autorisations d’un fichier/dossier**, utilisez la commande suivante :

    ```azurecli
    az dls fs access set-permission --account mydatalakestoragegen1 --path /mynewfolder/vehicle1_09142014.csv --permission 777
    ```
    
* **Pour obtenir les listes ACL pour un chemin d’accès donné**, utilisez la commande suivante :

    ```azurecli
    az dls fs access show --account mydatalakestoragegen1 --path /mynewfolder/vehicle1_09142014.csv
    ```

    La sortie doit ressembler à ce qui suit :

    ```output
    {
        "entries": [
        "user::rwx",
        "group::rwx",
        "other::---"
        ],
        "group": "1808bd5f-62af-45f4-89d8-03c5e81bac20",
        "owner": "1808bd5f-62af-45f4-89d8-03c5e81bac20",
        "permission": "770",
        "stickyBit": false
    }
    ```

* **Pour définir une entrée pour une liste ACL**, utilisez la commande suivante :

    ```azurecli
    az dls fs access set-entry --account mydatalakestoragegen1 --path /mynewfolder --acl-spec user:6360e05d-c381-4275-a932-5535806bb323:-w-
    ```

* **Pour supprimer une entrée d’une liste ACL**, utilisez la commande suivante :

    ```azurecli
    az dls fs access remove-entry --account mydatalakestoragegen1 --path /mynewfolder --acl-spec user:6360e05d-c381-4275-a932-5535806bb323
    ```

* **Pour supprimer dans son intégralité une liste ACL par défaut**, utilisez la commande suivante :

    ```azurecli
    az dls fs access remove-all --account mydatalakestoragegen1 --path /mynewfolder --default-acl
    ```

* **Pour supprimer dans son intégralité une liste ACL autre que la liste ACL par défaut**, utilisez la commande suivante :

    ```azurecli
    az dls fs access remove-all --account mydatalakestoragegen1 --path /mynewfolder
    ```
    
## <a name="delete-a-data-lake-storage-gen1-account"></a>Supprimer un compte Data Lake Storage Gen1
Utilisez la commande suivante pour supprimer un compte Data Lake Storage Gen1.

```azurecli
az dls account delete --account mydatalakestoragegen1
```

Quand vous y êtes invité, entrez **Y** pour supprimer le compte.

## <a name="next-steps"></a>Étapes suivantes
* [Utiliser Azure Data Lake Storage Gen1 pour le Big Data](data-lake-store-data-scenarios.md) 
* [Sécuriser les données dans Data Lake Storage Gen1](data-lake-store-secure-data.md)
* [Utiliser Azure Data Lake Analytics avec Data Lake Storage Gen1](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [Utiliser Azure HDInsight avec Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md)