---
title: Diagnostics de démarrage pour les machines virtuelles dans Azure | Microsoft Docs
description: Vue d’ensemble des deux fonctionnalités de débogage pour les machines virtuelles dans Azure
services: virtual-machines
author: Deland-Han
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines
ms.topic: troubleshooting
ms.date: 10/31/2018
ms.author: delhan
ms.openlocfilehash: fbf8c1987a66cad6d9d9bc9dab829af4b42a224e
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91971555"
---
# <a name="how-to-use-boot-diagnostics-to-troubleshoot-virtual-machines-in-azure"></a>Comment utiliser les diagnostics de démarrage pour résoudre les problèmes des machines virtuelles dans Azure

Plusieurs raisons peuvent expliquer pourquoi une machine virtuelle entre dans un état non démarrable. Concernant les machines virtuelles créées à l’aide du modèle de déploiement Resource Manager, vous pouvez résoudre les problèmes en utilisant les fonctionnalités de débogage suivantes : Prise en charge des sorties de console et des captures d’écran pour les machines virtuelles Azure. 

Dans le cas des machines virtuelles Linux, vous pouvez facilement voir la sortie de votre journal de console dans le portail. Pour les machines virtuelles Windows et Linux, Azure vous permet également de voir une capture d’écran de la machine virtuelle dans l’hyperviseur. Ces deux fonctionnalités sont prises en charge par les machines virtuelles Azure dans toutes les régions. Notez que des captures d’écran ainsi que des sorties peuvent prendre jusqu’à 10 minutes pour apparaître dans votre compte de stockage.

Vous pouvez sélectionner l’option **Diagnostics de démarrage** pour afficher le journal et la capture d’écran.

![Gestionnaire de ressources](./media/virtual-machines-common-boot-diagnostics/screenshot1.png)

## <a name="common-boot-errors"></a>Erreurs de démarrage courantes

- [0xC000000E](https://support.microsoft.com/help/4010129)
- [0xC000000F](https://support.microsoft.com/help/4010130)
- [0xC0000011](https://support.microsoft.com/help/4010134)
- [0xC0000034](https://support.microsoft.com/help/4010140)
- [0xC0000098](https://support.microsoft.com/help/4010137)
- [0xC00000BA](https://support.microsoft.com/help/4010136)
- [0xC000014C](https://support.microsoft.com/help/4010141)
- [0xC0000221](https://support.microsoft.com/help/4010132)
- [0xC0000225](https://support.microsoft.com/help/4010138)
- [0xC0000359](https://support.microsoft.com/help/4010135)
- [0xC0000605](https://support.microsoft.com/help/4010131)
- [Système d’exploitation introuvable.](https://support.microsoft.com/help/4010142)
- [Échec de démarrage ou INACCESSIBLE_BOOT_DEVICE.](https://support.microsoft.com/help/4010143)

## <a name="enable-diagnostics-on-a-virtual-machine-created-using-the-azure-portal"></a>Activer les diagnostics sur une machine virtuelle créée à l’aide du portail Azure

La procédure suivante s’applique aux machines virtuelles créées à l’aide du modèle de déploiement Resource Manager.

Sous l’onglet **Gestion**, dans la section **Supervision**, vérifiez que l’option **Diagnostics de démarrage** est activée. Dans la liste déroulante **Compte de stockage des diagnostics**, sélectionnez un compte de stockage où placer les fichiers de diagnostic.
 
![Créer une machine virtuelle](./media/virtual-machines-common-boot-diagnostics/enable-boot-diagnostics-vm.png)

> [!NOTE]
> La fonctionnalité Diagnostics de démarrage ne prend pas en charge les types de comptes Stockage Premium ou Stockage redondant interzone. Si vous utilisez le compte de stockage Premium pour la fonctionnalité Diagnostics de démarrage, vous pouvez recevoir l’erreur StorageAccountTypeNotSupported lorsque vous démarrez la machine virtuelle.
>

### <a name="deploying-from-an-azure-resource-manager-template"></a>Déploiement à l’aide d’un modèle Azure Resource Manager

Si vous déployez à partir d’un modèle Azure Resource Manager, accédez à votre ressource de machine virtuelle et ajoutez la section de profil des diagnostics. Définissez l’en-tête de version d’API sur « 2015-06-15 » ou version ultérieure. La version la plus récente est « 2018-10-01 ».

```json
{
  "apiVersion": "2018-10-01",
  "type": "Microsoft.Compute/virtualMachines",
  … 
```

Le profil de diagnostics vous permet de sélectionner le compte de stockage dans lequel vous souhaitez placer ces journaux d’activité.

```json
    "diagnosticsProfile": {
    "bootDiagnostics": {
    "enabled": true,
    "storageUri": "[concat('https://', parameters('newStorageAccountName'), '.blob.core.windows.net')]"
    }
    }
    }
}
```

Pour plus d’informations sur le déploiement des ressources à l’aide de modèles, consultez [Démarrage rapide : Créer et déployer des modèles Azure Resource Manager à l’aide du portail Azure](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).

## <a name="enable-boot-diagnostics-on-existing-virtual-machine"></a>Activer les diagnostics de démarrage sur une machine virtuelle existante 

Pour activer la fonctionnalité Diagnostics de démarrage sur une machine virtuelle existante, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis sélectionnez la machine virtuelle.
2. Dans la section **Support + dépannage**, sélectionnez **Diagnostics de démarrage**, puis sélectionnez l’onglet **Paramètres**.
3. Dans les paramètres **Diagnostics de démarrage**, remplacez l’état par **Activé** et, dans la liste déroulante **Compte de stockage**, sélectionnez un compte de stockage. 
4. Enregistrez la modification.

    ![Mettre à jour une machine virtuelle existante](./media/virtual-machines-common-boot-diagnostics/enable-for-existing-vm.png)

### <a name="enable-boot-diagnostics-using-the-azure-cli"></a>Activer les diagnostics de démarrage à l’aide d’Azure CLI

Vous pouvez utiliser Azure CLI pour activer les diagnostics de démarrage sur une machine virtuelle Azure existante. Pour plus d’informations, consultez [az vm boot-diagnostics](/cli/azure/vm/boot-diagnostics?view=azure-cli-latest).
