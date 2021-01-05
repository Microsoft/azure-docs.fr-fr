---
title: Données personnalisées et machines virtuelles Azure
description: Détails sur l’utilisation de données personnalisées et de Cloud-init sur des machines virtuelles Azure
services: virtual-machines
author: mimckitt
ms.service: virtual-machines
ms.topic: how-to
ms.date: 03/06/2020
ms.author: mimckitt
ms.openlocfilehash: 2924caaac5fb8c512100d9e897f7f153af9a3b3e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87284912"
---
# <a name="custom-data-and-cloud-init-on-azure-virtual-machines"></a>Données personnalisées et Cloud-init sur les machines virtuelles Azure

Vous devrez peut-être injecter un script ou d’autres métadonnées dans une machine virtuelle Microsoft Azure au moment du provisionnement.  Dans d’autres clouds, ce concept est souvent appelé données utilisateur.  Dans Microsoft Azure, nous disposons d’une fonctionnalité similaire appelée données personnalisées. 

Les données personnalisées sont uniquement accessibles à la machine virtuelle lors du premier démarrage ou de l’installation initiale ; nous appelons cela la « configuration ». La configuration est le processus au cours duquel les paramètres de création de la machine virtuelle (par exemple, le nom d’hôte, le nom d’utilisateur, le mot de passe, les certificats, les données personnalisées, les clés, etc.) sont mis à la disposition de la machine virtuelle et un agent de configuration les traite, par exemple l’agent [Linux](./extensions/agent-linux.md) ou [Cloud-init](./linux/using-cloud-init.md#troubleshooting-cloud-init). 


## <a name="passing-custom-data-to-the-vm"></a>Transmission de données personnalisées à la machine virtuelle
Pour utiliser des données personnalisées, vous devez d’abord encoder le contenu en base64 avant de le passer à l’API, sauf si vous utilisez un outil CLI qui effectue la conversion pour vous, par exemple AZ CLI. La taille ne peut pas dépasser 64 ko.

Dans l’interface CLI, vous pouvez passer vos données personnalisées en tant que fichier et elles seront converties en base64.
```bash
az vm create \
  --resource-group myResourceGroup \
  --name centos74 \
  --image OpenLogic:CentOS-CI:7-CI:latest \
  --custom-data cloud-init.txt \
  --generate-ssh-keys
```

Dans Azure Resource Manager (ARM), il existe une fonction [base64](../azure-resource-manager/templates/template-functions-string.md#base64).

```json
"name": "[parameters('virtualMachineName')]",
"type": "Microsoft.Compute/virtualMachines",
"apiVersion": "2019-07-01",
"location": "[parameters('location')]",
"dependsOn": [
..],
"variables": {
        "customDataBase64": "[base64(parameters('stringData'))]"
    },
"properties": {
..
    "osProfile": {
        "computerName": "[parameters('virtualMachineName')]",
        "adminUsername": "[parameters('adminUsername')]",
        "adminPassword": "[parameters('adminPassword')]",
        "customData": "[variables('customDataBase64')]"
        },
```

## <a name="processing-custom-data"></a>Traitement des données personnalisées
Les agents de configuration installés sur les machines virtuelles gèrent l’interfaçage avec la plateforme et le placent dans le système de fichiers. 

### <a name="windows"></a>Windows
Les données personnalisées sont placées dans *%SYSTEMDRIVE%\AzureData\CustomData.bin* sous la forme d’un fichier binaire, mais elles ne sont pas traitées. Si vous souhaitez traiter ce fichier, vous devez générer une image personnalisée et écrire du code pour traiter le fichier CustomData.bin.

### <a name="linux"></a>Linux  
Sur les systèmes d’exploitation Linux, les données personnalisées sont transmises à la machine virtuelle via le fichier OVF-env.xml, qui est copié dans le répertoire */var/lib/waagent* lors de la configuration.  Les versions récentes de l’agent Microsoft Azure Linux copient également les données encodées en base64 dans */var/lib/waagent/CustomData* pour des raisons pratiques.

Azure prend actuellement en charge deux agents de configuration :
* Agent Linux : par défaut, l’agent ne traitera pas les données personnalisées ; vous devrez créer une image personnalisée avec cette option activée. Les paramètres pertinents, conformément à la [documentation](https://github.com/Azure/WALinuxAgent#configuration) sont les suivants :
    * Provisioning.DecodeCustomData
    * Provisioning.ExecuteCustomData

Lorsque vous activez des données personnalisées et que vous exécutez un script, cela retardera la création de rapports de la machine virtuelle pour indiquer qu’elle est prête ou que la configuration a réussi jusqu’à la fin du script. Si le script dépasse la durée totale de configuration de la machine virtuelle de 40 minutes, la création de la machine virtuelle échoue. Notez qu’en cas d’échec de l’exécution du script ou d’erreurs lors de son exécution, cela n’est pas considéré comme un échec de configuration irrécupérable. Vous devrez créer un chemin de notification pour vous avertir de l’état d’achèvement du script.

Pour résoudre les problèmes d’exécution de données personnalisées, consultez */var/log/waagent.log*

* Cloud-ini : traite les données personnalisées par défaut. Cloud-init accepte [plusieurs formats](https://cloudinit.readthedocs.io/en/latest/topics/format.html) de données personnalisées, dont la configuration Cloud-init, les scripts, etc. Similaire à l’agent Linux, lorsque Cloud-init traite les données personnalisées. Si des erreurs se produisent pendant l’exécution du traitement de la configuration ou des scripts, cela n’est pas considéré comme un échec d’approvisionnement irrécupérable et vous devrez créer un chemin de notification pour vous avertir de l’état d’achèvement du script. Toutefois, dans le cas de l’agent Linux, Cloud-init n’attend pas la fin des configurations de données personnalisées de l’utilisateur avant de signaler à la plateforme que la machine virtuelle est prête. Pour plus d’informations sur Cloud-init sur Azure, consultez la [documentation](./linux/using-cloud-init.md).


Pour résoudre les problèmes liés à l’exécution de données personnalisées, consultez la [documentation](./linux/using-cloud-init.md#troubleshooting-cloud-init) relative à la résolution des problèmes.


## <a name="faq"></a>Questions fréquentes (FAQ)
### <a name="can-i-update-custom-data-after-the-vm-has-been-created"></a>Puis-je mettre à jour les données personnalisées après la création de la machine virtuelle ?
Pour les machines virtuelles uniques, les données personnalisées dans le modèle de machine virtuelle ne peuvent pas être mises à jour, mais pour les VMSS, vous pouvez mettre à jour les données personnalisées des VMSS via l’[REST API](/rest/api/compute/virtualmachinescalesets/update) (non applicable pour les clients PS ou AZ CLI). Quand vous mettez à jour des données personnalisées dans le modèle de VMSS :
* Les instances existantes dans le VMSS n’obtiendront pas les données personnalisées mises à jour, uniquement jusqu’à ce qu’elles soient réinitialisées.
* Les instances existantes dans les VMSS qui sont mises à niveau n’obtiendront pas les données personnalisées mises à jour.
* Les nouvelles instances recevront les nouvelles données personnalisées.

### <a name="can-i-place-sensitive-values-in-custom-data"></a>Puis-je placer des valeurs sensibles dans les données personnalisées ?
Nous vous conseillons de ne **pas** stocker des données sensibles dans les données personnalisées. Pour plus d’informations, consultez les [Meilleures pratiques pour la sécurité et le chiffrement Azure](../security/fundamentals/data-encryption-best-practices.md).


### <a name="is-custom-data-made-available-in-imds"></a>Les données personnalisées sont-elles rendues disponibles dans IMDS ?
Non, cette fonctionnalité n'est pas disponible pour l'instant.
