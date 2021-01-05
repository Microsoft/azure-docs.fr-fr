---
title: Commandes Azure CLI courantes
description: Découvrez certaines des commandes Azure CLI courantes pour commencer à gérer vos machines virtuelles en mode Azure Resource Manager
author: RicksterCDN
ms.service: virtual-machines-linux
ms.topic: conceptual
ms.date: 05/12/2017
ms.author: rclaus
ms.openlocfilehash: 5a9dd8aaeed0642461e4244a72a3dab5c96a77b6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87372244"
---
# <a name="common-azure-cli-commands-for-managing-azure-resources"></a>Commandes Azure CLI courantes pour gérer les ressources Azure

Azure CLI vous permet de créer et de gérer vos ressources Azure sur Mac OS, Linux et Windows. Cet article décrit certaines des commandes les plus courantes pour créer et gérer des machines virtuelles.

Cet article requiert Azure CLI version 2.0.4 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez mettre à niveau, consultez [Installation d’Azure CLI](/cli/azure/install-azure-cli). Vous pouvez également utiliser [Cloud Shell](../../cloud-shell/quickstart.md) à partir de votre navigateur.

## <a name="basic-azure-resource-manager-commands-in-azure-cli"></a>Commandes de base Basic Azure Resource Manager de l’interface de ligne de commande Azure
Pour plus d’informations sur les commutateurs et options de ligne de commande spécifiques, vous pouvez utiliser les options et l’aide en ligne des commandes en tapant `az <command> <subcommand> --help`.

### <a name="create-vms"></a>Créer des machines virtuelles
| Tâche | Commandes d’interface de ligne de commande Azure |
| --- | --- |
| Créer un groupe de ressources | `az group create --name myResourceGroup --location eastus` |
| Créer une machine virtuelle Linux | `az vm create --resource-group myResourceGroup --name myVM --image ubuntults` |
| Créer une machine virtuelle Windows | `az vm create --resource-group myResourceGroup --name myVM --image win2016datacenter` |

### <a name="manage-vm-state"></a>Gérer l’état d’une machine virtuelle
| Tâche | Commandes d’interface de ligne de commande Azure |
| --- | --- |
| Démarrer une machine virtuelle | `az vm start --resource-group myResourceGroup --name myVM` |
| Arrêter une machine virtuelle | `az vm stop --resource-group myResourceGroup --name myVM` |
| Désallouer une machine virtuelle | `az vm deallocate --resource-group myResourceGroup --name myVM` |
| Redémarrer une machine virtuelle | `az vm restart --resource-group myResourceGroup --name myVM` |
| Redéploiement d’une machine virtuelle | `az vm redeploy --resource-group myResourceGroup --name myVM` |
| Supprimer une machine virtuelle | `az vm delete --resource-group myResourceGroup --name myVM` |

### <a name="get-vm-info"></a>Obtenir des informations sur les machines virtuelles
| Tâche | Commandes d’interface de ligne de commande Azure |
| --- | --- |
| Énumérer les machines virtuelles | `az vm list` |
| Obtenir des informations sur une machine virtuelle | `az vm show --resource-group myResourceGroup --name myVM` |
| Obtenir l'utilisation des ressources de la machine virtuelle | `az vm list-usage --location eastus` |
| Obtenir toutes les tailles de machines virtuelles disponibles | `az vm list-sizes --location eastus` |

## <a name="disks-and-images"></a>Disques et images
| Tâche | Commandes d’interface de ligne de commande Azure |
| --- | --- |
| Ajouter un disque de données à une machine virtuelle | `az vm disk attach --resource-group myResourceGroup --vm-name myVM --disk myDataDisk --size-gb 128 --new` |
| Supprimer un disque de données à partir d'une machine virtuelle | `az vm disk detach --resource-group myResourceGroup --vm-name myVM --disk myDataDisk` |
| Redimensionner un disque | `az disk update --resource-group myResourceGroup --name myDataDisk --size-gb 256` |
| Effectuer la capture instantanée d’un disque | `az snapshot create --resource-group myResourceGroup --name mySnapshot --source myDataDisk` |
| Créer une image de machine virtuelle | `az image create --resource-group myResourceGroup --source myVM --name myImage` |
| Créer une machine virtuelle à partir d’une image | `az vm create --resource-group myResourceGroup --name myNewVM --image myImage` |


## <a name="next-steps"></a>Étapes suivantes
Pour visualiser des exemples supplémentaires des commandes CLI, consultez le didacticiel [Créer et gérer des machines virtuelles Linux avec l’interface Azure CLI](tutorial-manage-vm.md).
