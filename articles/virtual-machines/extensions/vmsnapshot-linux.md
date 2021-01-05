---
title: Extension Linux de capture instantanée de machine virtuelle pour la Sauvegarde Azure
description: À partir de Sauvegarde Azure, effectuez une sauvegarde cohérente des applications de la machine virtuelle à l’aide de l’extension Linux de capture instantanée de machine virtuelle.
services: backup, virtual-machines-linux
documentationcenter: ''
author: trinadhkotturu
manager: gwallace
ms.service: virtual-machines-linux
ms.subservice: extensions
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.date: 12/17/2018
ms.author: trinadhk
ms.openlocfilehash: 2bf653bb42351c9ab4811fde695c8f821fd42d7c
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966766"
---
# <a name="vm-snapshot-linux-extension-for-azure-backup"></a>Extension Linux de capture instantanée de machine virtuelle pour la Sauvegarde Azure



La Sauvegarde Azure fournit une prise en charge permettant de sauvegarder les charges de travail depuis l’environnement local vers le cloud, et les ressources cloud dans le coffre Recovery Services. La Sauvegarde Azure se sert d’une extension de capture instantanée de machine virtuelle pour récupérer une sauvegarde cohérente des applications de la machine virtuelle Azure, sans nécessiter l’arrêt de la machine virtuelle. L’extension Linux de capture instantanée de machine virtuelle est publiée et prise en charge par Microsoft dans le cadre du service de Sauvegarde Azure. La Sauvegarde Azure installe l’extension lors de la première sauvegarde planifiée qui est déclenchée après l’activation du service de sauvegarde. Ce document présente les plateformes, configurations et options de déploiement qui sont prises en charge pour l’extension de capture instantanée de machine virtuelle.

L'extension VMSnapshot n'apparaît sur le portail Azure que pour les machines virtuelles non gérées.

## <a name="prerequisites"></a>Prérequis

### <a name="operating-system"></a>Système d’exploitation
Pour obtenir la liste des systèmes d’exploitation pris en charge, reportez-vous à [Systèmes d’exploitation pris en charge par la Sauvegarde Azure](../../backup/backup-azure-arm-vms-prepare.md#before-you-start)

## <a name="extension-schema"></a>Schéma d’extensions

L’extrait JSON suivant illustre le schéma de l’extension de capture instantanée de machine virtuelle. L’extension nécessite l’ID de tâche (qui permet d’identifier la tâche de sauvegarde ayant déclenché la capture instantanée sur la machine virtuelle), l’uri de l’objet blob d’état (où est écrit l’état de l’opération de capture instantanée), l’heure planifiée à laquelle débute la capture instantanée, l’uri de l’objet blob des journaux d’activité (où sont écrits les journaux d’activité correspondant à la tâche de capture instantanée) et objstr (qui est la représentation des métadonnées et des disques de la machine virtuelle).  Ces paramètres devant être traités en tant que données sensibles, ils sont stockés dans une configuration de paramètres protégés. Les données du paramètre de protection de l’extension de machine virtuelle Azure sont chiffrées et ne sont déchiffrées que sur la machine virtuelle cible. Notez qu’il est recommandé de ne transmettre ces paramètres à partir du service de Sauvegarde Azure que dans le cadre d’une tâche de sauvegarde.

```json
{
  "type": "extensions",
  "name": "VMSnapshot",
  "location":"<myLocation>",
  "properties": {
    "publisher": "Microsoft.RecoveryServices",
    "type": "VMSnapshot",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "locale":"<location>",
      "taskId":"<taskId used by Azure Backup service to communicate with extension>",
      "commandToExecute": "snapshot",
      "commandStartTimeUTCTicks": "<scheduled start time of the snapshot task>",
      "vmType": "microsoft.compute/virtualmachines"
    },
    "protectedSettings": {
      "objectStr": "<blob SAS uri representation of VM sent by Azure Backup service to extension>",
      "logsBlobUri": "<blob uri where logs of command execution by extension are written to>",
      "statusBlobUri": "<blob uri where status of the command executed by extension is written>"
    }
  }
}
```

### <a name="property-values"></a>Valeurs de propriétés

| Nom | Valeur/Exemple | Type de données |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | Date |
| taskId | e07354cf-041e-4370-929f-25a319ce8933_1 | string |
| commandStartTimeUTCTicks | 6.36458E + 17 | string |
| locale | fr-FR | string |
| objectStr | Encoding of sas uri array- "blobSASUri": ["https:\/\/sopattna5365.blob.core.windows.net\/vhds\/vmubuntu1404ltsc201652903941.vhd?sv=2014-02-14&sr=b&sig=TywkROXL1zvhXcLujtCut8g3jTpgbE6JpSWRLZxAdtA%3D&st=2017-11-09T14%3A23%3A28Z&se=2017-11-09T17%3A38%3A28Z&sp=rw", "https:\/\/sopattna8461.blob.core.windows.net\/vhds\/vmubuntu1404ltsc-20160629-122418.vhd?sv=2014-02-14&sr=b&sig=5S0A6YDWvVwqPAkzWXVy%2BS%2FqMwzFMbamT5upwx05v8Q%3D&st=2017-11-09T14%3A23%3A28Z&se=2017-11-09T17%3A38%3A28Z&sp=rw", "https:\/\/sopattna8461.blob.core.windows.net\/bootdiagnostics-vmubuntu1-deb58392-ed5e-48be-9228-ff681b0cd3ee\/vmubuntu1404ltsc-20160629-122541.vhd?sv=2014-02-14&sr=b&sig=X0Me2djByksBBMVXMGIUrcycvhQSfjYvqKLeRA7nBD4%3D&st=2017-11-09T14%3A23%3A28Z&se=2017-11-09T17%3A38%3A28Z&sp=rw", "https:\/\/sopattna5365.blob.core.windows.net\/vhds\/vmubuntu1404ltsc-20160701-163922.vhd?sv=2014-02-14&sr=b&sig=oXvtK2IXCNqWv7fpjc7TAzFDpc1GoXtT7r%2BC%2BNIAork%3D&st=2017-11-09T14%3A23%3A28Z&se=2017-11-09T17%3A38%3A28Z&sp=rw", "https:\/\/sopattna5365.blob.core.windows.net\/vhds\/vmubuntu1404ltsc-20170705-124311.vhd?sv=2014-02-14&sr=b&sig=ZUM9d28Mvvm%2FfrhJ71TFZh0Ni90m38bBs3zMl%2FQ9rs0%3D&st=2017-11-09T14%3A23%3A28Z&se=2017-11-09T17%3A38%3A28Z&sp=rw"] | string |
| logsBlobUri | https://seapod01coord1exsapk732.blob.core.windows.net/bcdrextensionlogs-d45d8a1c-281e-4bc8-9d30-3b25176f68ea/sopattna-vmubuntu1404ltsc.v2.Logs.txt?sv=2014-02-14&sr=b&sig=DbwYhwfeAC5YJzISgxoKk%2FEWQq2AO1vS1E0rDW%2FlsBw%3D&st=2017-11-09T14%3A33%3A29Z&se=2017-11-09T17%3A38%3A29Z&sp=rw | string |
| statusBlobUri | https://seapod01coord1exsapk732.blob.core.windows.net/bcdrextensionlogs-d45d8a1c-281e-4bc8-9d30-3b25176f68ea/sopattna-vmubuntu1404ltsc.v2.Status.txt?sv=2014-02-14&sr=b&sig=96RZBpTKCjmV7QFeXm5IduB%2FILktwGbLwbWg6Ih96Ao%3D&st=2017-11-09T14%3A33%3A29Z&se=2017-11-09T17%3A38%3A29Z&sp=rw | string |



## <a name="template-deployment"></a>Déploiement de modèle

Les extensions de machines virtuelles Azure peuvent être déployées avec des modèles Azure Resource Manager. Cependant, la méthode recommandée permettant d’ajouter une extension de capture instantanée de machine virtuelle à une machine virtuelle passe par l’activation de la sauvegarde sur la machine virtuelle. Vous pouvez l’obtenir par le biais d’un modèle Resource Manager.  Un exemple de modèle Resource Manager activant la sauvegarde sur une machine virtuelle est à votre disposition dans la [galerie de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-recovery-services-backup-vms/).


## <a name="azure-cli-deployment"></a>Déploiement de l’interface de ligne de commande Azure

Il est possible d’utiliser Azure CLI pour activer la sauvegarde sur une machine virtuelle. Une fois l’activation de la sauvegarde effectuée, la première sauvegarde planifiée qui suit installe l’extension de capture instantanée de machine virtuelle sur la machine virtuelle.

```azurecli
az backup protection enable-for-vm \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --vm myVM \
    --policy-name DefaultPolicy
```

## <a name="troubleshoot-and-support"></a>Dépannage et support technique

### <a name="troubleshoot"></a>Dépanner

Vous pouvez récupérer les données sur l’état des déploiements d’extension à partir du portail Azure et à l’aide de l’interface de ligne de commande Azure. Pour afficher l’état du déploiement des extensions pour une machine virtuelle donnée, exécutez la commande suivante à l’aide de l’interface de ligne de commande Azure.

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

La sortie de l’exécution de l’extension est enregistrée dans le fichier suivant :

```
/var/log/waagent.log
```

### <a name="error-codes-and-their-meanings"></a>Codes d’erreur et signification

Pour obtenir des informations sur la résolution des problèmes, consultez le [guide de résolution des problèmes de la sauvegarde de machines virtuelles Azure](../../backup/backup-azure-vms-troubleshoot.md).

### <a name="support"></a>Support

Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, vous pouvez contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur Obtenir un support. Pour plus d’informations sur l’utilisation du support Azure, lisez le [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).
