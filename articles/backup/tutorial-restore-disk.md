---
title: Tutoriel - Restaurer une machine virtuelle avec Azure CLI
description: Découvrez comment restaurer un disque et créer une machine virtuelle récupérée dans Azure avec le service Sauvegarde et Recovery Services.
ms.topic: tutorial
ms.date: 01/31/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 999682c9bf4a4d70d886f0e85cede99f215aa046
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97694710"
---
# <a name="restore-a-vm-with-azure-cli"></a>Restaurer une machine virtuelle avec Azure CLI

Azure Backup crée des points de récupération stockés dans des coffres de récupération géoredondants. Quand vous effectuez une restauration à partir d’un point de récupération, vous pouvez restaurer la machine virtuelle entière ou des fichiers individuels. Cet article explique comment restaurer une machine virtuelle entière à l’aide de l’interface de ligne de commande. Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Répertorier et sélectionner des points de récupération
> * Restaurer un disque à partir d’un point de récupération
> * Créer une machine virtuelle à partir du disque restauré

Pour plus d’informations sur l’utilisation de PowerShell pour restaurer un disque et créer une machine virtuelle récupérée, consultez [Restauration d’une machine virtuelle Azure](backup-azure-vms-automation.md#restore-an-azure-vm).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Ce tutoriel nécessite la version 2.0.18 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

 - Ce didacticiel nécessite une machine virtuelle Linux qui a été protégée avec Sauvegarde Azure. Pour simuler une suppression accidentelle de machine virtuelle et le processus de récupération, vous créez une machine virtuelle à partir d’un disque à un point de récupération. Si vous avez besoin d’une machine virtuelle Linux qui a été protégée avec Sauvegarde Azure, consultez [Sauvegarder une machine virtuelle dans Azure avec l’interface CLI](quick-backup-vm-cli.md).

## <a name="backup-overview"></a>Présentation de la sauvegarde

Quand Azure lance une sauvegarde, l’extension de sauvegarde sur la machine virtuelle capture un instantané à un point dans le temps. L’extension de sauvegarde est installée sur la machine virtuelle quand la première sauvegarde est demandée. Sauvegarde Azure peut également capturer un instantané du stockage sous-jacent si la machine virtuelle n’est pas en cours d’exécution durant la sauvegarde.

Par défaut, Sauvegarde Azure effectue une sauvegarde cohérente du système de fichiers. Une fois que Sauvegarde Azure a pris l’instantané, les données sont transférées vers le coffre Recovery Services. Pour optimiser l’efficacité, Sauvegarde Azure identifie et transfère uniquement les blocs de données qui ont été modifiés depuis la sauvegarde précédente.

Une fois le transfert de données terminé, l’instantané est supprimé et un point de récupération est créé.

## <a name="list-available-recovery-points"></a>Obtenir la liste des points de récupération disponibles

Pour restaurer un disque, vous sélectionnez un point de récupération en tant que source pour les données de récupération. Étant donné que la stratégie par défaut crée un point de récupération chaque jour et conserve ces derniers pendant 30 jours, vous pouvez conserver un ensemble de points de récupération qui vous permet de sélectionner un point particulier dans le temps pour la récupération.

Pour afficher la liste des points de récupération disponibles, utilisez [az backup recoverypoint list](/cli/azure/backup/recoverypoint#az-backup-recoverypoint-list). Le point de récupération **name** est utilisé pour récupérer des disques. Dans ce didacticiel, nous voulons le point de récupération le plus récent disponible. Le paramètre `--query [0].name` sélectionne le nom du point de récupération le plus récent comme suit :

```azurecli-interactive
az backup recoverypoint list \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --backup-management-type AzureIaasVM \
    --container-name myVM \
    --item-name myVM \
    --query [0].name \
    --output tsv
```

## <a name="restore-a-vm-disk"></a>Restaurer un disque de machine virtuelle

> [!IMPORTANT]
> Nous vous recommandons vivement d’utiliser AZ CLI version 2.0.74 ou ultérieure pour bénéficier de tous les avantages d’une restauration rapide, notamment la restauration de disque managé. Il est préférable de toujours utiliser la version la plus récente.

### <a name="managed-disk-restore"></a>Restauration de disque managé

Si la machine virtuelle sauvegardée contient des disques managés et si l’objectif est de restaurer des disques managés à partir du point de récupération, vous devez d’abord spécifier un compte Stockage Azure. Ce compte sert à stocker la configuration de la machine virtuelle et le modèle de déploiement qui peut par la suite être utilisé pour déployer la machine virtuelle à partir des disques restaurés. Ensuite, vous spécifiez également un groupe de ressources cible dans lequel restaurer les disques managés.

1. Pour créer un compte de stockage, utilisez la commande [az storage account create](/cli/azure/storage/account#az-storage-account-create). Le nom du compte de stockage doit être écrit entièrement en minuscules et être unique. Remplacez *mystorageaccount* par un nom unique :

    ```azurecli-interactive
    az storage account create \
        --resource-group myResourceGroup \
        --name mystorageaccount \
        --sku Standard_LRS
    ```

2. Restaurez le disque à partir de votre point de récupération avec [az backup restore restore-disks](/cli/azure/backup/restore#az-backup-restore-restore-disks). Remplacez *mystorageaccount* par le nom du compte de stockage que vous avez créé à l’aide de la commande précédente. Remplacez *myRecoveryPointName* par le nom du point de récupération que vous avez obtenu dans la sortie de la commande [az backup recoverypoint list](/cli/azure/backup/recoverypoint#az-backup-recoverypoint-list) précédente. ***Spécifiez également un groupe de ressources cible dans lequel restaurer les disques managés** _.

    ```azurecli-interactive
    az backup restore restore-disks \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --storage-account mystorageaccount \
        --rp-name myRecoveryPointName \
        --target-resource-group targetRG
    ```

    > [!WARNING]
    > Si _ *target-resource-group** n’est pas spécifié, les disques managés sont restaurés en tant que disques non managés dans le compte de stockage donné. Cela aura des conséquences importantes sur la durée de restauration, car le temps nécessaire à la restauration des disques dépend entièrement du compte de stockage donné. Vous bénéficiez de la restauration instantanée seulement si le paramètre target-resource-group est donné. Si l’objectif est de restaurer des disques managés en disques non managés, ne fournissez pas le paramètre **target-resource-group** mais plutôt le paramètre **restore-as-unmanaged-disk**, comme indiqué ci-dessous. Ce paramètre est disponible à partir de az 3.4.0.

    ```azurecli-interactive
    az backup restore restore-disks \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --container-name myVM \
    --item-name myVM \
    --storage-account mystorageaccount \
    --rp-name myRecoveryPointName \
    --restore-as-unmanaged-disk
    ```

Cette opération restaure les disques managés en disques non managés dans le compte de stockage donné, et ne bénéficie pas de la fonctionnalité de restauration « instantanée ». Dans les versions ultérieures de l’interface CLI, la fourniture du paramètre **target-resource-group** ou du paramètre **restore-as-unmanaged-disk** sera obligatoire.

### <a name="unmanaged-disks-restore"></a>Restauration de disques non managés

Si la machine virtuelle sauvegardée contient des disques non managés et si l’objectif est de restaurer des disques à partir du point de récupération, vous devez d’abord spécifier un compte Stockage Azure. Ce compte sert à stocker la configuration de la machine virtuelle et le modèle de déploiement qui peut par la suite être utilisé pour déployer la machine virtuelle à partir des disques restaurés. Par défaut, les disques non managés sont restaurés dans leur compte de stockage d’origine. Si vous souhaitez restaurer tous les disques non managés à un seul emplacement, le compte de stockage donné peut également être utilisé comme emplacement intermédiaire pour ces disques.

À travers des étapes supplémentaires, le disque restauré est utilisé pour créer une machine virtuelle.

1. Pour créer un compte de stockage, utilisez la commande [az storage account create](/cli/azure/storage/account#az-storage-account-create). Le nom du compte de stockage doit être écrit entièrement en minuscules et être unique. Remplacez *mystorageaccount* par un nom unique :

    ```azurecli-interactive
    az storage account create \
        --resource-group myResourceGroup \
        --name mystorageaccount \
        --sku Standard_LRS
    ```

2. Restaurez le disque à partir de votre point de récupération avec [az backup restore restore-disks](/cli/azure/backup/restore#az-backup-restore-restore-disks). Remplacez *mystorageaccount* par le nom du compte de stockage que vous avez créé à l’aide de la commande précédente. Remplacez *myRecoveryPointName* par le nom du point de récupération que vous avez obtenu dans la sortie de la commande [az backup recoverypoint list](/cli/azure/backup/recoverypoint#az-backup-recoverypoint-list) précédente :

    ```azurecli-interactive
    az backup restore restore-disks \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --storage-account mystorageaccount \
        --rp-name myRecoveryPointName
    ```

Comme mentionné plus haut, les disques non managés sont restaurés dans leur compte de stockage d’origine. Cela offre les meilleures performances de restauration. Toutefois, si tous les disques non managés doivent être restaurés vers un compte de stockage donné, utilisez l’indicateur approprié comme indiqué ci-dessous.

```azurecli-interactive
    az backup restore restore-disks \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --storage-account mystorageaccount \
        --rp-name myRecoveryPointName \
        --restore-to-staging-storage-account
    ```

## Monitor the restore job

To monitor the status of restore job, use [az backup job list](/cli/azure/backup/job#az-backup-job-list):

```azurecli-interactive
az backup job list \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --output table
```

La sortie est similaire à l’exemple suivant, qui montre que la tâche de restauration est *en cours* :

```output
Name      Operation        Status      Item Name    Start Time UTC       Duration
--------  ---------------  ----------  -----------  -------------------  --------------
7f2ad916  Restore          InProgress  myvm         2017-09-19T19:39:52  0:00:34.520850
a0a8e5e6  Backup           Completed   myvm         2017-09-19T03:09:21  0:15:26.155212
fe5d0414  ConfigureBackup  Completed   myvm         2017-09-19T03:03:57  0:00:31.191807
```

Quand l’*État* du travail de restauration indique *Terminé*, les informations nécessaires (configuration de la machine virtuelle et modèle de déploiement) ont été restaurées dans le compte de stockage.

## <a name="create-a-vm-from-the-restored-disk"></a>Créer une machine virtuelle à partir du disque restauré

La dernière étape consiste à créer une machine virtuelle à partir des disques restaurés. Vous pouvez utiliser le modèle de déploiement téléchargé dans le compte de stockage donné pour créer la machine virtuelle.

### <a name="fetch-the-job-details"></a>Récupérer les détails du travail

Les détails du travail obtenu donnent l’URI du modèle qui peut être interrogé et déployé. Utilisez la commande Afficher du travail pour obtenir plus de détails sur le travail de restauration déclenché.

```azurecli-interactive
az backup job show \
    -v myRecoveryServicesVault \
    -g myResourceGroup \
    -n 1fc2d55d-f0dc-4ca6-ad48-aca0fe5d0414
```

La sortie de cette requête donne tous les détails, mais nous nous intéressons uniquement au contenu du compte de stockage. Nous pouvons utiliser la [fonctionnalité de requête](/cli/azure/query-azure-cli) d’Azure CLI pour récupérer les détails pertinents.

```azurecli-interactive
az backup job show \
    -v myRecoveryServicesVault \
    -g myResourceGroup \
    -n 1fc2d55d-f0dc-4ca6-ad48-aca0fe5d0414 \
    --query properties.extendedInfo.propertyBag

{
  "Config Blob Container Name": "myVM-daa1931199fd4a22ae601f46d8812276",
  "Config Blob Name": "config-myVM-1fc2d55d-f0dc-4ca6-ad48-aca0fe5d0414.json",
  "Config Blob Uri": "https://mystorageaccount.blob.core.windows.net/myVM-daa1931199fd4a22ae601f46d8812276/config-appvm8-1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json",
  "Job Type": "Recover disks",
  "Recovery point time ": "12/25/2019 10:07:11 PM",
  "Target Storage Account Name": "mystorageaccount",
  "Target resource group": "mystorageaccountRG",
  "Template Blob Uri": "https://mystorageaccount.blob.core.windows.net/myVM-daa1931199fd4a22ae601f46d8812276/azuredeploy1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json"
}
```

### <a name="fetch-the-deployment-template"></a>Récupérer le modèle de déploiement

Le modèle n’est pas directement accessible, car il se trouve sous le compte de stockage d’un client et le conteneur donné. Nous avons besoin de l’URL complète (avec un jeton SAS temporaire) pour accéder à ce modèle.

Tout d’abord, extrayez l’URI de l’objet blob de modèle à partir des détails du travail.

```azurecli-interactive
az backup job show \
    -v myRecoveryServicesVault \
    -g myResourceGroup \
    -n 1fc2d55d-f0dc-4ca6-ad48-aca0fe5d0414 \
    --query properties.extendedInfo.propertyBag."""Template Blob Uri"""

"https://mystorageaccount.blob.core.windows.net/myVM-daa1931199fd4a22ae601f46d8812276/azuredeploy1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json"
```

L’URI de l’objet blob de modèle aura ce format et extraira le nom du modèle.

```https
https://<storageAccountName.blob.core.windows.net>/<containerName>/<templateName>
```

Ainsi, le nom du modèle de l’exemple ci-dessus sera ```azuredeploy1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json``` et le nom du conteneur sera ```myVM-daa1931199fd4a22ae601f46d8812276```.

À présent, récupérez le jeton SAS pour ce conteneur et ce modèle comme détaillé [ici](../azure-resource-manager/templates/secure-template-with-sas-token.md?tabs=azure-cli#provide-sas-token-during-deployment).

```azurecli-interactive
expiretime=$(date -u -d '30 minutes' +%Y-%m-%dT%H:%MZ)
connection=$(az storage account show-connection-string \
    --resource-group mystorageaccountRG \
    --name mystorageaccount \
    --query connectionString)
token=$(az storage blob generate-sas \
    --container-name myVM-daa1931199fd4a22ae601f46d8812276 \
    --name azuredeploy1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json \
    --expiry $expiretime \
    --permissions r \
    --output tsv \
    --connection-string $connection)
url=$(az storage blob url \
   --container-name myVM-daa1931199fd4a22ae601f46d8812276 \
    --name azuredeploy1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json \
    --output tsv \
    --connection-string $connection)
```

### <a name="deploy-the-template-to-create-the-vm"></a>Déployer le modèle pour créer la machine virtuelle

Maintenant, déployez le modèle pour créer la machine virtuelle comme expliqué [ici](../azure-resource-manager/templates/deploy-cli.md).

```azurecli-interactive
az deployment group create \
  --resource-group ExampleGroup \
  --template-uri $url?$token
```

Pour confirmer que votre machine virtuelle a été créée à partir du disque récupéré, répertoriez les machines virtuelles dans votre groupe de ressources avec [az vm list](/cli/azure/vm#az-vm-list) comme suit :

```azurecli-interactive
az vm list --resource-group myResourceGroup --output table
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez restauré un disque à partir d’un point de récupération, puis créé une machine virtuelle à partir du disque. Vous avez appris à :

> [!div class="checklist"]
>
> * Répertorier et sélectionner des points de récupération
> * Restaurer un disque à partir d’un point de récupération
> * Créer une machine virtuelle à partir du disque restauré

Passez au didacticiel suivant pour en savoir plus sur la restauration de fichiers individuels à partir d’un point de récupération.

> [!div class="nextstepaction"]
> [Restaurer des fichiers sur une machine virtuelle dans Azure](tutorial-restore-files.md)
