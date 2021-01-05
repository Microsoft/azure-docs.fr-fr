---
title: Gérer Azure Data Lake Analytics à l’aide d’Azure CLI
description: Cet article décrit l’utilisation d’Azure CLI pour gérer des travaux, des sources de données et des utilisateurs Azure Data Lake Analytics.
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 01/29/2018
ms.openlocfilehash: 19b471d85a52fe38b72ad55847d022fb56b3c4f0
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92220922"
---
# <a name="manage-azure-data-lake-analytics-using-the-azure-command-line-interface-cli"></a>Gérer Azure Data Lake Analytics à l’aide de l’interface de ligne de commande Azure (CLI)

[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

Découvrez comment gérer des comptes Azure Data Lake Analytics, des sources de données, des utilisateurs et des travaux à l’aide de l’interface de ligne de commande Azure. Pour afficher les rubriques de gestion à l’aide d’autres outils, cliquez sur l’onglet de sélection ci-dessus.

## <a name="prerequisites"></a>Prérequis

Pour pouvoir commencer ce didacticiel, vous devez disposer des ressources suivantes :

- Un abonnement Azure. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).

- Azure CLI. Consultez [Installation et configuration de l’interface de ligne de commande Azure](/cli/azure/install-azure-cli).

  - Téléchargez et installez la **préversion** d’ [outils d’interface de ligne de commande Azure](https://github.com/MicrosoftBigData/AzureDataLake/releases) pour effectuer cette démonstration.

- Authentifiez-vous à l’aide de la commande `az login` et sélectionnez l’abonnement que vous souhaitez utiliser. Pour plus d’informations sur l’authentification à l’aide d’un compte professionnel ou scolaire, consultez la section [Se connecter à un abonnement Azure à partir de l’interface de ligne de commande Azure](/cli/azure/authenticate-azure-cli).

   ```azurecli
   az login
   az account set --subscription <subscription id>
   ```

   Vous avez maintenant accès aux commandes Data Lake Analytics et Data Lake Store. Exécutez la commande suivante pour lister les commandes Data Lake Store et Data Lake Analytics :

   ```azurecli
   az dls -h
   az dla -h
   ```

## <a name="manage-accounts"></a>Gérer les comptes

Avant d'exécuter des travaux Data Lake Analytics, vous devez avoir un compte Data Lake Analytics. Contrairement à Azure HDInsight, vous ne payez pas pour un compte Analytics lorsque celui-ci n'exécute aucun travail. Vous payez uniquement lorsqu'il exécute un travail.  Pour plus d'informations, consultez [Présentation d'Azure Data Lake Analytics](data-lake-analytics-overview.md).

### <a name="create-accounts"></a>Création de comptes

Exécutez la commande suivante pour créer un compte Data Lake :

   ```azurecli
   az dla account create --account "<Data Lake Analytics account name>" --location "<Location Name>" --resource-group "<Resource Group Name>" --default-data-lake-store "<Data Lake Store account name>"
   ```

### <a name="update-accounts"></a>Mise à jour de comptes

La commande suivante met à jour les propriétés d'un compte Data Lake Analytics existant

   ```azurecli
   az dla account update --account "<Data Lake Analytics Account Name>" --firewall-state "Enabled" --query-store-retention 7
   ```

### <a name="list-accounts"></a>Énumérer les comptes

Répertorier les comptes Data Lake Analytics dans un groupe de ressources spécifique

   ```azurecli
   az dla account list "<Resource group name>"
   ```

## <a name="get-details-of-an-account"></a>Obtenir les détails relatifs à un compte

   ```azurecli
   az dla account show --account "<Data Lake Analytics account name>" --resource-group "<Resource group name>"
   ```

### <a name="delete-an-account"></a>Supprimer un compte

   ```azurecli
   az dla account delete --account "<Data Lake Analytics account name>" --resource-group "<Resource group name>"
   ```

## <a name="manage-data-sources"></a>Gérer les sources de données

Data Lake Analytics prend actuellement en charge les deux sources de données suivantes :

- [Azure Data Lake Store](../data-lake-store/data-lake-store-overview.md)
- [Stockage Azure](../storage/common/storage-introduction.md)

Lorsque vous créez un compte Analytics, vous devez désigner un compte Azure Data Lake Storage comme compte de stockage par défaut. Le compte de stockage Data Lake par défaut est utilisé pour stocker les métadonnées et les journaux d’audit des tâches. Après la création d'un compte Analytics, vous pouvez ajouter des comptes Data Lake Storage et/ou des comptes Azure Storage supplémentaires.

### <a name="find-the-default-data-lake-store-account"></a>Rechercher le compte Data Lake Store par défaut

Vous pouvez afficher le compte Data Lake Store utilisé par défaut en exécutant la commande `az dla account show`. Le nom de compte par défaut apparaît sous la propriété defaultDataLakeStoreAccount.

   ```azurecli
   az dla account show --account "<Data Lake Analytics account name>"
   ```

### <a name="add-additional-blob-storage-accounts"></a>Ajouter des comptes de stockage Blob supplémentaires

   ```azurecli
   az dla account blob-storage add --access-key "<Azure Storage Account Key>" --account "<Data Lake Analytics account name>" --storage-account-name "<Storage account name>"
   ```

> [!NOTE]
> Seuls les noms courts Blob Storage sont pris en charge. N'utilisez pas de nom de domaine complet, comme « myblob.blob.core.windows.net ».
>

### <a name="add-additional-data-lake-store-accounts"></a>Ajouter des comptes Data Lake Store supplémentaires

La commande suivante met à jour le compte Data Lake Analytics spécifié en ajoutant un compte Data Lake Store supplémentaire :

   ```azurecli
   az dla account data-lake-store add --account "<Data Lake Analytics account name>" --data-lake-store-account-name "<Data Lake Store account name>"
   ```

### <a name="update-existing-data-source"></a>Mettre à jour la source de données existante

Pour mettre à jour une clé de compte Blob Storage existante :

   ```azurecli
   az dla account blob-storage update --access-key "<New Blob Storage Account Key>" --account "<Data Lake Analytics account name>" --storage-account-name "<Data Lake Store account name>"
   ```

### <a name="list-data-sources"></a>Lister les sources de données

Pour lister les comptes Data Lake Store :

   ```azurecli
   az dla account data-lake-store list --account "<Data Lake Analytics account name>"
   ```

Pour lister le compte de stockage Blob :

   ```azurecli
   az dla account blob-storage list --account "<Data Lake Analytics account name>"
   ```

![Capture d’écran montrant Azure CLI avec les informations « dataLakeStoreAccounts » en surbrillance](./media/data-lake-analytics-manage-use-cli/data-lake-analytics-list-data-source.png)

### <a name="delete-data-sources"></a>Supprimer des sources de données

Pour supprimer un compte de magasin Data Lake :

   ```azurecli
   az dla account data-lake-store delete --account "<Data Lake Analytics account name>" --data-lake-store-account-name "<Azure Data Lake Store account name>"
   ```

Pour supprimer un compte de stockage Blob :

   ```azurecli
   az dla account blob-storage delete --account "<Data Lake Analytics account name>" --storage-account-name "<Data Lake Store account name>"
   ```

## <a name="manage-jobs"></a>Gestion des travaux

Vous devez disposer d'un compte Data Lake Analytics avant de pouvoir créer un travail.  Pour plus d'informations, consultez [Gestion des comptes Data Lake Analytics](#manage-accounts).

### <a name="list-jobs"></a>Répertorier les travaux

   ```azurecli
   az dla job list --account "<Data Lake Analytics account name>"
   ```

   ![Data Lake Analytics : énumérer les sources de données](./media/data-lake-analytics-manage-use-cli/data-lake-analytics-list-jobs.png)

### <a name="get-job-details"></a>Obtenir les détails du travail

   ```azurecli
   az dla job show --account "<Data Lake Analytics account name>" --job-identity "<Job Id>"
   ```

### <a name="submit-jobs"></a>Soumettre les travaux

> [!NOTE]
> La priorité par défaut d'un travail est 1 000 et le degré de parallélisme par défaut d'un travail est 1.
>
>    ```azurecli
>    az dla job submit --account "<Data Lake Analytics account name>" --job-name "<Name of your job>" --script "<Script to submit>"
>    ```

### <a name="cancel-jobs"></a>Annuler les travaux

Utilisez la commande list pour trouver l’ID de travail, puis utilisez la commande cancel pour annuler le travail.

   ```azurecli
   az dla job cancel --account "<Data Lake Analytics account name>" --job-identity "<Job Id>"
   ```

## <a name="pipelines-and-recurrences"></a>Pipelines et récurrences

### <a name="get-information-about-pipelines-and-recurrences"></a>Obtenir des informations sur les pipelines et les périodicités

Utilisez les commandes `az dla job pipeline` pour afficher les tâches déjà soumises sur les informations de pipeline.

```azurecli
az dla job pipeline list --account "<Data Lake Analytics Account Name>"

az dla job pipeline show --account "<Data Lake Analytics Account Name>" --pipeline-identity "<Pipeline ID>"
```

Utilisez les commandes `az dla job recurrence` pour afficher les tâches déjà soumises sur les informations de récurrences.

```azurecli
az dla job recurrence list --account "<Data Lake Analytics Account Name>"

az dla job recurrence show --account "<Data Lake Analytics Account Name>" --recurrence-identity "<Recurrence ID>"
```

## <a name="next-steps"></a>Étapes suivantes

- [Vue d'ensemble de Microsoft Azure Data Lake Analytics](data-lake-analytics-overview.md)
- [Prise en main de Data Lake Analytics à l’aide du portail Azure](data-lake-analytics-get-started-portal.md)
- [Gestion d’Azure Data Lake Analytics à l’aide du portail Azure](data-lake-analytics-manage-use-portal.md)
- [Surveiller et résoudre les problèmes des tâches Azure Data Lake Analytics à l’aide du portail Azure](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)
