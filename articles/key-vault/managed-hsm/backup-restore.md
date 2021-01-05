---
title: Sauvegarde/restauration complète et restauration sélective pour Azure Managed HSM
description: Ce document décrit la sauvegarde/restauration complète et la restauration sélective
services: key-vault
author: amitbapat
tags: azure-key-vault
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 09/15/2020
ms.author: ambapat
ms.openlocfilehash: e051a36b3c91fadc0c3b602cb4ba8e3dbcff1294
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94367129"
---
# <a name="full-backup-and-restore"></a>Sauvegarde et restauration complètes

> [!NOTE]
> Cette fonctionnalité est disponible uniquement pour le HSM managé de type ressource.

Managed HSM prend en charge la création d’une sauvegarde complète de l’ensemble du contenu du HSM, notamment l’ensemble des clés, versions, attributs, étiquettes et attributions de rôles. La sauvegarde est chiffrée à l’aide de clés de chiffrement associées au domaine de sécurité du HSM.

La sauvegarde est une opération de plan de données. L’appelant qui lance l’opération de sauvegarde doit avoir l’autorisation d’exécuter dataAction **Microsoft.KeyVault/managedHsm/backup/start/action**.

Seuls les rôles intégrés suivants sont autorisés à effectuer une sauvegarde complète :
- Administrateur du HSM managé
- Sauvegarde du HSM managé

Vous devez fournir les informations suivantes pour exécuter une sauvegarde complète :
- URL ou nom du HSM
- Nom du compte de stockage
- Conteneur de stockage d’objets blob dans un compte de stockage
- Jeton SAS du conteneur de stockage avec autorisations `crdw`

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="full-backup"></a>Sauvegarde complète

La sauvegarde est une opération de longue durée, mais qui retourne immédiatement un ID de travail. Vous pouvez vérifier l’état du processus de sauvegarde à l’aide de cet ID de travail. Le processus de sauvegarde crée un dossier à l’intérieur du conteneur désigné avec le modèle de nommage suivant **`mhsm-{HSM_NAME}-{YYYY}{MM}{DD}{HH}{mm}{SS}`** , où HSM_NAME est le nom du HSM managé qui est sauvegardé et YYYY, MM, DD, HH, MM, mm, SS représentent l’année, le mois, la date, l’heure, les minutes et les secondes de date/d’heure au format UTC de la réception de la commande de sauvegarde.

Pendant que la sauvegarde est en cours, le HSM peut ne pas fonctionner à un débit complet, car certaines partitions HSM sont occupées à effectuer l’opération de sauvegarde.

```azurecli-interactive
# time for 30 minutes later for SAS token expiry

end=$(date -u -d "30 minutes" '+%Y-%m-%dT%H:%MZ')

# Get storage account key

skey=$(az storage account keys list --query '[0].value' -o tsv --account-name mhsmdemobackup --subscription a1ba9aaa-b7f6-4a33-b038-6e64553a6c7b)

# Create a container

az storage container create --account-name  mhsmdemobackup --name mhsmdemobackupcontainer  --account-key $skey

# Generate a container sas token

sas=$(az storage container generate-sas -n mhsmdemobackupcontainer --account-name mhsmdemobackup --permissions crdw --expiry $end --account-key $skey -o tsv --subscription a1ba9aaa-b7f6-4a33-b038-6e64553a6c7b)

# Backup HSM

az keyvault backup start --hsm-name mhsmdemo2 --storage-account-name mhsmdemobackup --blob-container-name mhsmdemobackupcontainer --storage-container-SAS-token $sas --subscription 361da5d4-a47a-4c79-afdd-d66f684f4070
```

## <a name="full-restore"></a>Restauration complète

La restauration complète vous permet de restaurer complètement le contenu du HSM à l’aide d’une sauvegarde précédente, notamment l’ensemble des clés, versions, attributs, étiquettes et attributions de rôles. Tout ce qui est actuellement stocké dans le HSM sera effacé et il retournera à l’état dans lequel il se trouvait lors de la création de la sauvegarde source.

> [!IMPORTANT]
> La restauration complète est une opération très destructrice et perturbatrice. Par conséquent, il est obligatoire d’avoir effectué une sauvegarde complète au cours des 30 dernières minutes avant de pouvoir exécuter une opération `restore`.

La restauration est une opération de plan de données. L’appelant qui démarre l’opération de restauration doit avoir l’autorisation d’exécuter dataAction **Microsoft.KeyVault/managedHsm/restore/start/action**. Le HSM source où la sauvegarde a été créée et le HSM de destination où la restauration sera effectuée **doivent** avoir le même domaine de sécurité. Apprenez-en davantage sur le [domaine de sécurité du HSM managé](security-domain.md).

Vous devez fournir les informations suivantes pour exécuter une restauration complète :
- URL ou nom du HSM
- Nom du compte de stockage
- Conteneur d’objets blob dans un compte de stockage
- Jeton SAS du conteneur de stockage avec autorisations `rl`
- Nom du dossier du conteneur de stockage dans lequel la sauvegarde source est stockée

La restauration est une opération de longue durée, mais qui retourne immédiatement un ID de travail. Vous pouvez vérifier l’état du processus de restauration à l’aide de cet ID de travail. Lorsque le processus de restauration est en cours, le HSM passe en mode de restauration et toutes les commandes du plan de données (à l’exception de la vérification de l’état de restauration) sont désactivées.

```azurecli-interactive
#### time for 30 minutes later for SAS token expiry

end=$(date -u -d "30 minutes" '+%Y-%m-%dT%H:%MZ')

# Get storage account key

skey=$(az storage account keys list --query '[0].value' -o tsv --account-name mhsmdemobackup --subscription a1ba9aaa-b7f6-4a33-b038-6e64553a6c7b)

# Generate a container sas token

sas=$(az storage container generate-sas -n mhsmdemobackupcontainer --account-name mhsmdemobackup --permissions rl --expiry $end --account-key $skey -o tsv --subscription a1ba9aaa-b7f6-4a33-b038-6e64553a6c7b)
```

## <a name="backup-hsm"></a>Sauvegarder HSM

```
az keyvault restore start --hsm-name mhsmdemo2 --storage-account-name mhsmdemobackup --blob-container-name mhsmdemobackupcontainer --storage-container-SAS-token $sas --backup-folder mhsm-mhsmdemo-2020083120161860
```

## <a name="next-steps"></a>Étapes suivantes
- Consulter [Gérer un HSM managé à l’aide de l’interface Azure CLI](key-management.md)
- En savoir plus sur le [domaine de sécurité du HSM managé](security-domain.md)