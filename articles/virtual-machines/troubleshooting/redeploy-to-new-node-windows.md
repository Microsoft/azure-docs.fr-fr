---
title: Redéployer des machines virtuelles Windows dans Azure | Microsoft Docs
description: Comment redéployer des machines virtuelles Windows dans Azure pour atténuer les problèmes de connexion RDP.
services: virtual-machines-windows
documentationcenter: virtual-machines
author: genlin
manager: dcscontentpm
tags: azure-resource-manager,top-support-issue
ms.assetid: 0ee456ee-4595-4a14-8916-72c9110fc8bd
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: afbea39a080e1dd768a14d6e0eacda1bad23c5a4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87074418"
---
# <a name="redeploy-windows-virtual-machine-to-new-azure-node"></a>Redéployer des machines virtuelles Windows dans un nouveau nœud Azure
Si vous avez été confronté à des difficultés pour la résolution des problèmes de connexion de Bureau à distance ou l’accès des applications à une machine virtuelle Azure basée sur Windows, le redéploiement de la machine virtuelle peut vous aider. Lorsque vous redéployez une machine virtuelle, Azure ferme la machine virtuelle, la déplace vers un nouveau nœud au sein de l’infrastructure Azure, puis la remet sous tension, en conservant toutes vos options de configuration et ressources associées. Cet article vous montre comment redéployer une machine virtuelle à l’aide d’Azure PowerShell ou du portail Azure.

> [!NOTE]
> Une fois que vous redéployez une machine virtuelle, le disque temporaire est perdu et les adresses IP dynamiques associées à l’interface réseau virtuelle sont mises à jour. 


## <a name="using-azure-powershell"></a>Utilisation de Microsoft Azure PowerShell
Assurez-vous que vous avez installé la dernière version d’Azure PowerShell 1.x. Pour plus d’informations, consultez [Installer et configurer Azure PowerShell](/powershell/azure/).

L’exemple suivant déploie la machine virtuelle nommée `myVM` dans le groupe de ressources nommé `myResourceGroup` :

```powershell
Set-AzVM -Redeploy -ResourceGroupName "myResourceGroup" -Name "myVM"
```

[!INCLUDE [virtual-machines-common-redeploy-to-new-node](../../../includes/virtual-machines-common-redeploy-to-new-node.md)]

## <a name="next-steps"></a>Étapes suivantes
Si vous rencontrez des problèmes de connexion à votre machine virtuelle, vous pouvez rechercher une aide spécifique sur la [résolution des problèmes de connexion Bureau à distance](troubleshoot-rdp-connection.md) ou sur les [étapes de dépannage détaillées des connexions Bureau à distance](detailed-troubleshoot-rdp.md). Vous pouvez également lire des informations sur les [problèmes de dépannage des applications](./troubleshoot-app-connection.md) si vous ne pouvez pas accéder à une application exécutée sur votre machine virtuelle.
