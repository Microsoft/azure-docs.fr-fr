---
title: Double chiffrement des données au repos
titleSuffix: Azure HDInsight
description: Cet article décrit les deux couches de chiffrement disponibles pour les données au repos sur les clusters Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: hrasheed
ms.service: hdinsight
ms.topic: conceptual
ms.date: 08/10/2020
ms.openlocfilehash: 4e895cdba1bfc16eac0450bd05271f0e41985b7b
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359757"
---
# <a name="azure-hdinsight-double-encryption-for-data-at-rest"></a>Double chiffrement Azure HDInsight pour les données au repos

Cet article décrit les méthodes de chiffrement des données au repos dans les clusters Azure HDInsight. Le chiffrement des données au repos fait référence au chiffrement sur les disques managés (disques de données, disques de système d’exploitation et disques temporaires) attachés aux machines virtuelles du cluster HDInsight. 

Ce document ne traite pas des données stockées dans votre compte Stockage Azure. Vos clusters peuvent avoir un ou plusieurs comptes Stockage Azure attachés où les clés de chiffrement peuvent également être managées par Microsoft ou gérées par le client, mais le service de chiffrement est différent. Pour plus d’informations sur le chiffrement du Stockage Azure, voir [Chiffrement du stockage Azure pour les données au repos](../storage/common/storage-service-encryption.md).

## <a name="introduction"></a>Introduction

Il existe trois principaux rôles de disques managés dans Azure : disque de données, disque de système d’exploitation et disque temporaire. Pour plus d’informations sur les différents types de disques managés, consultez [Présentation des disques managés Azure](../virtual-machines/managed-disks-overview.md). 

HDInsight prend en charge plusieurs types de chiffrement dans deux couches différentes :

- Chiffrement côté serveur (SSE, Server Side Encryption) : il est effectué par le service de stockage. Dans HDInsight, SSE est utilisé pour chiffrer les disques de système d’exploitation et les disques de données. Il est activé par défaut. SSE est un service de chiffrement de couche 1.
- Chiffrement sur l’hôte à l’aide d’une clé gérée par la plateforme : similaire à SSE, ce type de chiffrement est effectué par le service de stockage. Toutefois, il ne concerne que les disques temporaires et n’est pas activé par défaut. Le chiffrement sur l’hôte est également un service de chiffrement de couche 1.
- Chiffrement au repos à l’aide d’une clé gérée par le client : ce type de chiffrement peut être utilisé sur les disques de données et les disques temporaires. Il n’est pas activé par défaut et exige que le client fournisse sa propre clé par le biais du coffre de clés Azure. Le chiffrement au repos est un service de chiffrement de couche 2.

Ces types sont récapitulés dans le tableau ci-dessous.

|Type de cluster |Disque de système d’exploitation (disque managé) |Disque de données (disque managé) |Disque de données temporaire (SSD local) |
|---|---|---|---|
|Kafka, HBase avec écritures accélérées|Couche 1 : [Chiffrement SSE](../virtual-machines/managed-disks-overview.md#encryption) par défaut|Couche 1 : [Chiffrement SSE](../virtual-machines/managed-disks-overview.md#encryption) par défaut, Couche 2 : Chiffrement au repos facultatif à l’aide d’une clé CMK|Couche 1 : Chiffrement facultatif sur l’hôte à l’aide d’une clé PMK, Couche 2 : Chiffrement au repos facultatif à l’aide d’une clé CMK|
|Tous les autres clusters (Spark, interactive, Hadoop, HBase sans écritures accélérées)|Couche 1 : [Chiffrement SSE](../virtual-machines/managed-disks-overview.md#encryption) par défaut|N/A|Couche 1 : Chiffrement facultatif sur l’hôte à l’aide d’une clé PMK, Couche 2 : Chiffrement au repos facultatif à l’aide d’une clé CMK|

## <a name="encryption-at-rest-using-customer-managed-keys"></a>Chiffrement au repos à l’aide de clés gérées par le client

Le chiffrement à l’aide de clés gérées par le client est un processus en une étape géré pendant la création d’un cluster sans coût supplémentaire. Il vous suffit d’autoriser une identité managée auprès d’Azure Key Vault et d’ajouter la clé de chiffrement lors de la création du cluster.

Les disques de données et les disques temporaires sur chaque nœud du cluster sont chiffrés avec une clé de chiffrement de données (DEK, Data Encryption Key) symétrique. La clé DEK est protégée avec la clé de chiffrement principale (KEK) de votre coffre de clés. Les processus de chiffrement et de déchiffrement sont entièrement gérés par Azure HDInsight.

Pour les disques de système d’exploitation attachés aux machines virtuelles du cluster, une seule couche de chiffrement (PMK) est disponible. Nous recommandons aux clients d’éviter de copier des données sensibles sur des disques de système d’exploitation si un chiffrement CMK est requis pour leurs scénarios.

Si le pare-feu de coffre de clés est activé sur le coffre de clés dans lequel la clé de chiffrement de disque est stockée, les adresses IP du fournisseur de ressources régionales HDInsight pour la région où le cluster sera déployé doivent être ajoutées à la configuration du pare-feu de coffre de clés. Cela est nécessaire, car HDInsight n’est pas un service de coffre de clés Azure approuvé.

Vous pouvez utiliser le portail Azure ou Azure CLI pour faire alterner les clés du coffre de clés en toute sécurité. Quand une clé permute, le cluster HDInsight démarre en quelques minutes en utilisant la nouvelle clé. Activez les fonctionnalités de protection de clés [Suppression réversible](../key-vault/general/soft-delete-overview.md) pour vous protéger contre les scénarios de rançongiciel et de suppression accidentelle. Les coffres de clés sans cette fonctionnalité de protection ne sont pas pris en charge.

### <a name="get-started-with-customer-managed-keys"></a>Prise en main des clés gérées par le client

Pour créer un cluster HDInsight avec clé gérée par le client, nous allons passer en revue les étapes suivantes :

1. Création d’identités managées pour les ressources Azure
1. Créer un Azure Key Vault
1. Créer une clé
1. Créer une stratégie d’accès
1. Création d’un cluster HDInsight avec clé gérée par le client
1. Rotation de la clé de chiffrement

Chaque étape est expliquée en détail dans l’une des sections suivantes.

### <a name="create-managed-identities-for-azure-resources"></a>Création d’identités managées pour les ressources Azure

Créez une identité gérée affectée par l’utilisateur pour l’authentification auprès de Key Vault.

Pour les étapes spécifiques, voir [Créer une identité managée attribuée par l’utilisateur](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md). Pour plus d’informations sur le fonctionnement des identités managées dans Azure HDInsight, consultez [Identités managées dans Azure HDInsight](hdinsight-managed-identities.md). Veillez à enregistrer l’ID de ressource d’identité managée, vous en aurez besoin pour l’ajouter à la stratégie d’accès Key Vault.

### <a name="create-azure-key-vault"></a>Créer un Azure Key Vault

Création d’un coffre de clés Pour les étapes spécifiques, voir [Créer un Azure Key Vault](../key-vault/general/quick-create-portal.md).

HDInsight prend uniquement en charge Azure Key Vault. Si vous disposez de votre propre coffre de clés, vous pouvez importer vos clés dans Azure Key Vault. N’oubliez pas que la fonctionnalité **Suppression réversible** doit être activée pour le coffre de clés. Pour plus d’informations sur l’importation de clés existantes, consultez [Présentation des clés, des secrets et des certificats](../key-vault/general/about-keys-secrets-certificates.md).

### <a name="create-key"></a>Créer une clé

1. À partir de votre nouveau coffre de clés, accédez à **Paramètres** > **Clés** >  **+ Générer/importer**.

    ![Générer une nouvelle clé dans Azure Key Vault](./media/disk-encryption/create-new-key.png "Générer une nouvelle clé dans Azure Key Vault")

1. Fournissez un nom, puis sélectionnez **Créer**. Conservez le **Type de clé** par défaut **RSA**.

    ![génère un nom de clé](./media/disk-encryption/create-key.png "Générer le nom de la clé")

1. Lorsque vous revenez à la page **Clés**, sélectionnez la clé que vous avez créée.

    ![liste de clés de coffre de clés](./media/disk-encryption/key-vault-key-list.png)

1. Sélectionnez la version pour ouvrir la page **Version de la clé**. Quand vous utilisez votre propre clé pour le chiffrement du cluster HDInsight, vous devez spécifier l’URI de la clé. Copiez l’**identificateur de clé** et enregistrez-le quelque part jusqu’à la création du cluster.

    ![obtenir l’identificateur de clé](./media/disk-encryption/get-key-identifier.png)

### <a name="create-access-policy"></a>Créer une stratégie d’accès

1. À partir de votre nouveau coffre de clés, accédez à **Paramètres** > **Stratégies d’accès** >  **+ Ajouter une stratégie d’accès.**

    ![Créer une stratégie d’accès Azure Key Vault](./media/disk-encryption/key-vault-access-policy.png)

1. Dans la page **Ajouter une stratégie d’accès**, fournissez les informations suivantes :

    |Propriété |Description|
    |---|---|
    |Autorisations de clé|Sélectionnez **Obtenir**, **Ne pas inclure la clé** et **Inclure la clé**.|
    |Autorisations de secret|Sélectionnez **Obtenir**, **Définir**, puis **Supprimer**.|
    |Sélectionner le principal|Choisissez l’identité managée affectée par l’utilisateur que vous avez créée précédemment.|

    ![Définir Sélectionner le principal pour la stratégie d’accès Azure Key Vault](./media/disk-encryption/azure-portal-add-access-policy.png)

1. Sélectionnez **Ajouter**.

1. Sélectionnez **Enregistrer**.

    ![Enregistrer une stratégie d’accès Azure Key Vault](./media/disk-encryption/add-key-vault-access-policy-save.png)

### <a name="create-cluster-with-customer-managed-key-disk-encryption"></a>Créer un cluster avec chiffrement de disque par clé gérée par le client

Vous êtes maintenant prêt à créer un cluster HDInsight. Les clés gérées par le client ne peuvent être appliquées qu’aux nouveaux clusters, pendant leur création. Vous ne pouvez pas supprimer le chiffrement des clusters à clés gérées par le client, ni ajouter de clés gérées par le client aux clusters existants.

À compter de la [version de novembre 2020](hdinsight-release-notes.md#release-date-11182020), HDInsight prend en charge la création de clusters à l’aide d’URI de clé avec et sans version. Si vous créez le cluster avec un URI de clé sans version, le cluster HDInsight essaiera d’effectuer la rotation automatique des clés lorsque la clé est mise à jour dans votre coffre de clés Azure. Si vous créez le cluster avec un URI de clé avec version, vous devez effectuer une rotation manuelle de clé, comme indiqué dans [Rotation de la clé de chiffrement](#rotating-the-encryption-key).

Pour les clusters créés avant la version de novembre 2020, vous devez effectuer une rotation de clé manuelle à l’aide de l’URI de clé avec version.

#### <a name="using-the-azure-portal"></a>Utilisation du portail Azure

Lors de la création du cluster, vous pouvez utiliser une clé avec version ou une clé sans version de la façon suivante :

- **Avec version** – Pendant la création du cluster, fournissez l’**identificateur de clé** complet, y compris la version de la clé. Par exemple : `https://contoso-kv.vault.azure.net/keys/myClusterKey/46ab702136bc4b229f8b10e8c2997fa4`.
- **Sans version** – Pendant la création du cluster, fournissez uniquement l’**identificateur de clé**. Par exemple : `https://contoso-kv.vault.azure.net/keys/myClusterKey`.

Vous devez aussi affecter l’identité managée au cluster.

![Créer un cluster](./media/disk-encryption/create-cluster-portal.png)

#### <a name="using-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure

L’exemple suivant montre comment utiliser Azure CLI pour créer un nouveau cluster Apache Spark avec le chiffrement de disque activé. Pour plus d’informations, voir [Azure CLI az hdinsight create](/cli/azure/hdinsight#az-hdinsight-create). Le paramètre `encryption-key-version` est facultatif.

```azurecli
az hdinsight create -t spark -g MyResourceGroup -n MyCluster \
-p "HttpPassword1234!" --workernode-data-disks-per-node 2 \
--storage-account MyStorageAccount \
--encryption-key-name SparkClusterKey \
--encryption-key-version 00000000000000000000000000000000 \
--encryption-vault-uri https://MyKeyVault.vault.azure.net \
--assign-identity MyMSI
```

#### <a name="using-azure-resource-manager-templates"></a>Utilisation de modèles Azure Resource Manager

L’exemple suivant montre comment utiliser un modèle Azure Resource Manager pour créer un nouveau cluster Apache Spark avec le chiffrement de disque activé. Pour plus d’informations, consultez [Que sont les modèles ARM ?](../azure-resource-manager/templates/overview.md) La propriété de modèle de gestionnaire des ressources `diskEncryptionKeyVersion` est facultative.

Cet exemple utilise PowerShell pour appeler le modèle.

```powershell
$templateFile = "azuredeploy.json"
$ResourceGroupName = "MyResourceGroup"
$clusterName = "MyCluster"
$password = ConvertTo-SecureString 'HttpPassword1234!' -AsPlainText -Force
$diskEncryptionVaultUri = "https://MyKeyVault.vault.azure.net"
$diskEncryptionKeyName = "SparkClusterKey"
$diskEncryptionKeyVersion = "00000000000000000000000000000000"
$managedIdentityName = "MyMSI"

New-AzResourceGroupDeployment `
  -Name mySpark `
  -TemplateFile $templateFile `
  -ResourceGroupName $ResourceGroupName `
  -clusterName $clusterName `
  -clusterLoginPassword $password `
` -sshPassword $password `
  -diskEncryptionVaultUri $diskEncryptionVaultUri `
  -diskEncryptionKeyName $diskEncryptionKeyName `
  -diskEncryptionKeyVersion $diskEncryptionKeyVersion `
  -managedIdentityName $managedIdentityName
```

Le contenu du modèle de gestion des ressources, `azuredeploy.json` :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "0.9.0.0",
  "parameters": {
    "clusterName": {
      "type": "string",
      "metadata": {
        "description": "The name of the HDInsight cluster to create."
      }
    },
    "clusterLoginUserName": {
      "type": "string",
      "defaultValue": "admin",
      "metadata": {
        "description": "These credentials can be used to submit jobs to the cluster and to log into cluster dashboards."
      }
    },
    "clusterLoginPassword": {
      "type": "securestring",
      "metadata": {
        "description": "The password must be at least 10 characters in length and must contain at least one digit, one non-alphanumeric character, and one upper or lower case letter."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "The location where all azure resources will be deployed."
      }
    },
    "sshUserName": {
      "type": "string",
      "defaultValue": "sshuser",
      "metadata": {
        "description": "These credentials can be used to remotely access the cluster."
      }
    },
    "sshPassword": {
      "type": "securestring",
      "metadata": {
        "description": "The password must be at least 10 characters in length and must contain at least one digit, one non-alphanumeric character, and one upper or lower case letter."
      }
    },
    "headNodeSize": {
      "type": "string",
      "defaultValue": "Standard_D12_v2",
      "metadata": {
        "description": "The VM size of the head nodes."
      }
    },
    "workerNodeSize": {
      "type": "string",
      "defaultValue": "Standard_D13_v2",
      "metadata": {
        "description": "The VM size of the worker nodes."
      }
    },
    "diskEncryptionVaultUri": {
      "type": "string",
      "metadata": {
        "description": "The Key Vault DNSname."
      }
    },
    "diskEncryptionKeyName": {
      "type": "string",
      "metadata": {
        "description": "The Key Vault key name."
      }
    },
    "diskEncryptionKeyVersion": {
      "type": "string",
      "metadata": {
        "description": "The Key Vault key version for the selected key."
      }
    },
    "managedIdentityName": {
      "type": "string",
      "metadata": {
        "description": "The user-assigned managed identity."
      }
    }
  },
  "variables": {
    "defaultStorageAccount": {
      "name": "[uniqueString(resourceGroup().id)]",
      "type": "Standard_LRS"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('defaultStorageAccount').name]",
      "location": "[parameters('location')]",
      "apiVersion": "2019-06-01",
      "sku": {
        "name": "[variables('defaultStorageAccount').type]"
      },
      "kind": "Storage",
      "properties": {}
    },
    {
      "apiVersion": "2018-06-01-preview",
      "name": "[parameters('clusterName')]",
      "type": "Microsoft.HDInsight/clusters",
      "location": "[parameters('location')]",
      "properties": {
        "clusterVersion": "3.6",
        "osType": "Linux",
        "tier": "standard",
        "clusterDefinition": {
          "kind": "spark",
          "componentVersion": {
            "Spark": "2.3"
          },
          "configurations": {
            "gateway": {
              "restAuthCredential.isEnabled": true,
              "restAuthCredential.username": "[parameters('clusterLoginUserName')]",
              "restAuthCredential.password": "[parameters('clusterLoginPassword')]"
            }
          }
        },
        "storageProfile": {
          "storageaccounts": [
            {
              "name": "[replace(replace(reference(resourceId('Microsoft.Storage/storageAccounts', variables('defaultStorageAccount').name), '2019-06-01').primaryEndpoints.blob,'https://',''),'/','')]",
              "isDefault": true,
              "container": "[parameters('clusterName')]",
              "key": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('defaultStorageAccount').name), '2019-06-01').keys[0].value]"
            }
          ]
        },
        "computeProfile": {
          "roles": [
            {
              "name": "headnode",
              "minInstanceCount": 1,
              "targetInstanceCount": 2,
              "hardwareProfile": {
                "vmSize": "[parameters('headNodeSize')]"
              },
              "osProfile": {
                "linuxOperatingSystemProfile": {
                  "username": "[parameters('sshUserName')]",
                  "password": "[parameters('sshPassword')]"
                },
              },
            },
            {
              "name": "workernode",
              "targetInstanceCount": 1,
              "hardwareProfile": {
                "vmSize": "[parameters('workerNodeSize')]"
              },
              "osProfile": {
                "linuxOperatingSystemProfile": {
                  "username": "[parameters('sshUserName')]",
                  "password": "[parameters('sshPassword')]"
                },
              },
            }
          ]
        },
        "minSupportedTlsVersion": "1.2",
        "diskEncryptionProperties": {
          "vaultUri": "[parameters('diskEncryptionVaultUri')]",
          "keyName": "[parameters('diskEncryptionKeyName')]",
          "keyVersion": "[parameters('diskEncryptionKeyVersion')]",
          "msiResourceId": "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('managedIdentityName'))]"
        }
      },
      "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
          "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('managedIdentityName'))]": {}
        }
      }
    }
  ]
}
```

### <a name="rotating-the-encryption-key"></a>Rotation de la clé de chiffrement

Vous pouvez modifier les clés de chiffrement utilisées sur votre cluster en cours d’exécution, à l’aide du portail Azure ou d’Azure CLI. Pour cette opération, le cluster doit avoir accès à la clé actuelle et à la nouvelle clé prévue. Sinon, l’opération de rotation des clés échouera. Pour les clusters créés après la version de novembre 2020, vous pouvez choisir si vous souhaitez que votre nouvelle clé ait une version ou non. Pour les clusters créés avant la version de novembre 2020, vous devez utiliser une clé avec version lors de la rotation de la clé de chiffrement.

#### <a name="using-the-azure-portal"></a>Utilisation du portail Azure

Pour assurer la rotation de la clé, vous avez besoin de l’URI de base du coffre de clés. Une fois que vous avez effectué cette opération, accédez à la section Propriétés du cluster HDInsight dans le portail, puis cliquez sur **Changer de clé** sous **URL de la clé de chiffrement de disque**. Entrez la nouvelle URL de clé et envoyez-la pour effectuer la rotation.

![permutation de la clé de chiffrement de disque](./media/disk-encryption/change-key.png)

#### <a name="using-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure

L’exemple suivant montre comment faire permuter la clé de chiffrement de disque d’un cluster HDInsight existant. Pour plus d’informations, voir [Azure CLI az hdinsight rotate-disk-encryption-key](/cli/azure/hdinsight#az-hdinsight-rotate-disk-encryption-key).

```azurecli
az hdinsight rotate-disk-encryption-key \
--encryption-key-name SparkClusterKey \
--encryption-key-version 00000000000000000000000000000000 \
--encryption-vault-uri https://MyKeyVault.vault.azure.net \
--name MyCluster \
--resource-group MyResourceGroup
```

## <a name="faq-for-customer-managed-key-encryption"></a>FAQ sur le chiffrement avec clé gérée par le client

**Comment le cluster HDInsight accède-t-il à mon coffre de clés ?**

HDInsight accède à votre instance Azure Key Vault à l’aide de l’identité managée que vous associez au cluster HDInsight. Cette identité managée peut être créée avant ou pendant la création du cluster. Vous devez aussi accorder à l’identité managée un accès au coffre de clés dans lequel la clé est stockée.

**Cette fonctionnalité est-elle accessible à tous les clusters sur HDInsight ?**

Le chiffrement avec clé gérée par le client est disponible pour tous les types de cluster sauf Spark 2.1 et 2.2.

**Puis-je utiliser plusieurs clés pour chiffrer différents disques ou dossiers ?**

Non, tous les disques managés et les disques de ressources sont chiffrés à l’aide de la même clé.

**Que se passe-t-il si le cluster ne peut plus accéder au coffre de clés ou à la clé ?**

Si le cluster ne peut plus accéder à la clé, des avertissements s’affichent dans le portail Apache Ambari. Dans ce cas, l’opération **Changer la clé** échouera. Une fois l’accès à la clé rétabli, les avertissements Ambari disparaissent et les opérations telles que la rotation des clés peuvent de nouveau être effectuées.

![alerte Ambari concernant l’accès à la clé](./media/disk-encryption/ambari-alert.png)

**Comment puis-je récupérer le cluster si les clés sont supprimées ?**

Sachant que seules sont prises en charge les clés pour lesquelles la fonctionnalité « Suppression réversible » est activée, si les clés sont récupérées dans le coffre de clés, le cluster doit retrouver l’accès aux clés. Pour récupérer une clé Azure Key Vault, consultez [Undo-AzKeyVaultKeyRemoval](/powershell/module/az.keyvault/Undo-AzKeyVaultKeyRemoval) ou [az-keyvault-key-recover](/cli/azure/keyvault/key#az-keyvault-key-recover).


**Si un cluster subit un scale-up, les nouveaux nœuds prendront-ils en charge les clés gérées par le client sans interruption ?**

Oui. Le cluster doit accéder à la clé dans le coffre de clés pendant la montée en puissance. La même clé est utilisée pour chiffrer les disques managés et les disques de ressources dans le cluster.

**Les clés gérées par le client sont-elles disponibles à mon emplacement ?**

Les clés gérées par le client HDInsight sont disponibles dans tous les clouds publics et nationaux.

## <a name="encryption-at-host-using-platform-managed-keys"></a>Chiffrement sur l’hôte à l’aide de clés gérées par la plateforme

### <a name="enable-in-the-azure-portal"></a>Activer dans le portail Azure

Le chiffrement sur l’hôte peut être activé lors de la création du cluster dans le portail Azure.

> [!Note]
> Lorsque le chiffrement sur l’hôte est activé, vous ne pouvez pas ajouter d’applications à votre cluster HDInsight à partir de la Place de marché Azure.

:::image type="content" source="media/disk-encryption/encryption-at-host.png" alt-text="Activez le chiffrement sur l’hôte.":::

Cette option active le [chiffrement sur l’hôte](../virtual-machines/disks-enable-host-based-encryption-portal.md) pour les disques de données temporaires des machines virtuelles HDInsight à l’aide de la clé PMK. Le chiffrement sur l’hôte est uniquement [pris en charge sur certaines références SKU de machine virtuelle dans des régions limitées](../virtual-machines/disks-enable-host-based-encryption-portal.md), et HDInsight prend en charge les [configurations de nœuds et références SKU mentionnées dans cet article](./hdinsight-supported-node-configuration.md).

Pour déterminer la taille de machine virtuelle adaptée à votre cluster HDInsight, consultez [Sélection de la taille de machine virtuelle adaptée à votre cluster Azure HDInsight](hdinsight-selecting-vm-size.md). La référence SKU de machine virtuelle par défaut pour le nœud Zookeeper lorsque le chiffrement sur l’hôte est activé est DS2V2.

### <a name="enable-using-powershell"></a>Activer à l’aide de PowerShell

L’extrait de code suivant montre comment créer un cluster Azure HDInsight pour lequel le chiffrement sur l’hôte est activé à l’aide de PowerShell. Il utilise le paramètre `-EncryptionAtHost $true` pour activer la fonctionnalité.

```powershell
$storageAccountResourceGroupName = "Group"
$storageAccountName = "yourstorageacct001"
$storageAccountKey = Get-AzStorageAccountKey `
    -ResourceGroupName $storageAccountResourceGroupName `
    -Name $storageAccountName | %{ $_.Key1 }
$storageContainer = "container002"
# Cluster configuration info
$location = "East US 2"
$clusterResourceGroupName = "Group"
$clusterName = "your-hadoop-002"
$clusterCreds = Get-Credential
# If the cluster's resource group doesn't exist yet, run:
# New-AzResourceGroup -Name $clusterResourceGroupName -Location $location
# Create the cluster
New-AzHDInsightCluster `
    -ClusterType Hadoop `
    -ClusterSizeInNodes 4 `
    -ResourceGroupName $clusterResourceGroupName `
    -ClusterName $clusterName `
    -HttpCredential $clusterCreds `
    -Location $location `
    -DefaultStorageAccountName "$storageAccountName.blob.core.contoso.net" `
    -DefaultStorageAccountKey $storageAccountKey `
    -DefaultStorageContainer $storageContainer `
    -SshCredential $clusterCreds `
    -EncryptionAtHost $true `
```

### <a name="enable-using-azure-cli"></a>Activer à l’aide d’Azure CLI

L’extrait de code suivant montre comment créer un cluster Azure HDInsight pour lequel le chiffrement sur l’hôte est activé à l’aide d’Azure CLI. Il utilise le paramètre `--encryption-at-host true` pour activer la fonctionnalité.

```azurecli
az hdinsight create -t spark -g MyResourceGroup -n MyCluster \\
-p "yourpass" \\
--storage-account MyStorageAccount --encryption-at-host true
```

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Azure Key Vault, voir [Qu’est-ce qu’Azure Key Vault](../key-vault/general/overview.md).
* [Vue d’ensemble de la sécurité d’entreprise dans Azure HDInsight](./domain-joined/hdinsight-security-overview.md).
