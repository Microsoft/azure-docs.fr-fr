---
title: Apache Hadoop et stockage avec transfert sécurisé - Azure HDInsight
description: Découvrez comment créer des clusters HDInsight à l’aide de comptes de stockage Azure doté du transfert sécurisé.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 02/18/2020
ms.openlocfilehash: 69844a392aa03dc962f8af456996ab51f1adf561
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92547467"
---
# <a name="apache-hadoop-clusters-with-secure-transfer-storage-accounts-in-azure-hdinsight"></a>Clusters Apache Hadoop avec des comptes de stockage avec transfert sécurisé dans Azure HDInsight

La fonctionnalité [Transfert sécurisé requis](../storage/common/storage-require-secure-transfer.md) améliore la sécurité de votre compte de stockage Azure en appliquant toutes les demandes à votre compte via une connexion sécurisée. Cette fonctionnalité et le schéma wasbs sont uniquement pris en charge par les clusters HDInsight 3.6 ou version ultérieure.

> [!IMPORTANT]
> L’activation du transfert de stockage sécurisé après la création d’un cluster peut entraîner des erreurs lors de l’utilisation de votre compte de stockage, et n’est donc pas recommandée. Il est préférable de créer un cluster à l’aide d’un compte de stockage sur lequel le transfert sécurisé est déjà activé.

## <a name="storage-accounts"></a>Comptes de stockage

### <a name="azure-portal"></a>Portail Azure

Par défaut, la propriété Transfert sécurisé requis est activée quand vous créez un compte de stockage dans le Portail Azure.

Pour mettre à jour un compte de stockage existant avec le Portail Azure, consultez [Exiger un transfert sécurisé avec le Portail Azure](../storage/common/storage-require-secure-transfer.md#require-secure-transfer-for-an-existing-storage-account).

### <a name="powershell"></a>PowerShell

Pour la cmdlet PowerShell [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount), vérifiez que le paramètre `-EnableHttpsTrafficOnly` est défini sur `1`.

Pour mettre à jour un compte de stockage existant avec PowerShell, consultez [Exiger un transfert sécurisé avec PowerShell](../storage/common/storage-require-secure-transfer.md#require-secure-transfer-with-powershell).

### <a name="azure-cli"></a>Azure CLI

Pour la commande Azure CLI [az storage account create](/cli/azure/storage/account#az-storage-account-create), vérifiez que le paramètre `--https-only` est défini sur `true`.

Pour mettre à jour un compte de stockage existant avec Azure CLI, consultez [Exiger un transfert sécurisé avec Azure CLI](../storage/common/storage-require-secure-transfer.md#require-secure-transfer-with-azure-cli).

## <a name="add-additional-storage-accounts"></a>Ajouter des comptes de stockage

Plusieurs options vous permettent d’ajouter des comptes de stockage dotés du transfert sécurisé :

* Modifiez le modèle Azure Resource Manager dans la dernière section.
* Créez un cluster à l’aide du [portail Azure](https://portal.azure.com) et indiquez le compte de stockage lié.
* Utilisez l’action de script pour ajouter des comptes de stockage doté du transfert sécurisé à un cluster HDInsight existant. Pour plus d’informations, consultez [Ajouter des comptes de stockage supplémentaires à HDInsight](hdinsight-hadoop-add-storage.md).

## <a name="next-steps"></a>Étapes suivantes

* l'utilisation du stockage Azure (WASB) au lieu de [Apache Hadoop HDFS](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html), en tant que stockage de données par défaut
* Pour plus d’informations sur la façon dont HDInsight utilise Stockage Azure, consultez la page [Utilisation de Stockage Azure avec HDInsight](hdinsight-hadoop-use-blob-storage.md).
* Pour plus d’informations sur le téléchargement de données dans HDInsight, consultez la page [Téléchargement de données dans HDInsight](hdinsight-upload-data.md).