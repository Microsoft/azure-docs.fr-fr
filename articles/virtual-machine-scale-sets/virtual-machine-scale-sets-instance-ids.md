---
title: Présentation des ID d’instance des machines virtuelles d’un groupe identique de machines virtuelles Azure
description: Comprendre les ID d’instance pour les machines virtuelles de groupes identiques et les différentes façons dont elles s’exposent.
author: mimckitt
ms.author: mimckitt
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 02/22/2018
ms.reviewer: jushiman
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: a32a5a04c5c71cc06d60f3d2f21946f5361a2afd
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94843245"
---
# <a name="understand-instance-ids-for-azure-vm-scale-set-vms"></a>Présentation des ID d’instance des machines virtuelles d’un groupe identique de machines virtuelles Azure
Cet article décrit les ID d’instance des groupes identiques ainsi que les différents modes de surface.

## <a name="scale-set-instance-ids"></a>ID d’instance des groupes identiques

Chaque machine virtuelle d’un groupe identique obtient un ID d’instance qui l’identifie de façon unique. Cet ID d’instance est utilisé dans les API de groupes identiques pour effectuer des opérations sur une machine virtuelle particulière du groupe identique. Par exemple, vous pouvez spécifier un ID d’instance spécifique pour procéder à une réinitialisation lorsque vous utilisez l’API de réinitialisation :

Une API REST : `POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/virtualmachines/{instanceId}/reimage?api-version={apiVersion}` (pour plus d’informations, consultez la [documentation de l’API REST](/rest/api/compute/virtualmachinescalesetvms/reimage))

PowerShell : `Set-AzVmssVM -ResourceGroupName {resourceGroupName} -VMScaleSetName {vmScaleSetName} -InstanceId {instanceId} -Reimage` (pour plus d’informations, consultez la [documentation PowerShell](/powershell/module/az.compute/set-azvmssvm))

L’interface CLI : `az vmss reimage -g {resourceGroupName} -n {vmScaleSetName} --instance-id {instanceId}` (pour plus d’informations, consultez la [documentation relative à l’interface CLI](/cli/azure/vmss?view=azure-cli-latest))

Vous pouvez obtenir la liste des ID d’instance en répertoriant toutes les instances dans un groupe identique :

Une API REST : `GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/virtualMachines?api-version={apiVersion}` (pour plus d’informations, consultez la [documentation de l’API REST](/rest/api/compute/virtualmachinescalesetvms/list))

PowerShell : `Get-AzVmssVM -ResourceGroupName {resourceGroupName} -VMScaleSetName {vmScaleSetName}` (pour plus d’informations, consultez la [documentation PowerShell](/powershell/module/az.compute/get-azvmssvm))

L’interface CLI : `az vmss list-instances -g {resourceGroupName} -n {vmScaleSetName}` (pour plus d’informations, consultez la [documentation relative à l’interface CLI](/cli/azure/vmss?view=azure-cli-latest))

Vous pouvez également utiliser [resources.azure.com](https://resources.azure.com) ou les [SDK Azure](https://azure.microsoft.com/downloads/) pour répertorier les machines virtuelles d’un groupe identique.

La sortie exacte varie selon les options que vous fournissez à la commande. Voici toutefois un exemple de sortie pour l’interface CLI :

```azurecli
az vmss show -g {resourceGroupName} -n {vmScaleSetName}
```

```output
[
  {
    "instanceId": "85",
    "latestModelApplied": true,
    "location": "westus",
    "name": "nsgvmss_85",
    .
    .
    .
```

Comme vous pouvez le voir, la propriété « instanceId » est simplement un nombre décimal. Les ID d’instance peuvent être réutilisés pour de nouvelles instances une fois que les anciennes instances sont supprimées.

>[!NOTE]
> Il n’existe **aucune garantie** sur la façon dont les ID d’instance sont attribués aux machines virtuelles du groupe identique. De temps en temps, ils peuvent sembler augmenter séquentiellement, mais ce n’est pas toujours le cas. N’établissez pas de dépendance sur la façon spécifique dont les ID d’instance sont attribués aux machines virtuelles.

## <a name="scale-set-vm-names"></a>Noms des machines virtuelles d’un groupe identique

L’exemple de sortie ci-dessus comprend également une propriété « name » pour la machine virtuelle. Le format du nom est le suivant : « {scale-set-name}_{instance-id} ». Ce nom est celui que vous voyez dans le portail Azure lorsque vous répertoriez les instances dans un groupe identique :

![Capture d’écran montrant une liste d’instances dans un groupe de machines virtuelles identiques dans le portail Azure.](./media/virtual-machine-scale-sets-instance-ids/vmssInstances.png)

La partie {instance-id} du nom est le même nombre décimal que la propriété « instanceId » décrite précédemment.

## <a name="instance-metadata-vm-name"></a>Nom de machine virtuelle Métadonnées de l’instance

Si vous interrogez les [métadonnées de l’instance](../virtual-machines/windows/instance-metadata-service.md) dans une machine virtuelle de groupe identique, vous pouvez observer la propriété « name » dans la sortie :

```output
{
  "compute": {
    "location": "westus",
    "name": "nsgvmss_85",
    .
    .
    .
```

Ce nom est identique à celui indiqué précédemment.

## <a name="scale-set-vm-computer-name"></a>Nom de l’ordinateur de machine virtuelle d’un groupe identique

Chaque machine virtuelle d’un groupe identique obtient également un nom d’ordinateur qui lui est attribué. Ce nom d’ordinateur est le nom d’hôte de la machine virtuelle dans la [résolution de noms DNS fournie par Azure dans le réseau virtuel](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md). Ce nom d’ordinateur est au format « {computer-name-prefix}{base-36-instance-id} ».

La partie {base-36-instance-id} est en [base 36](https://en.wikipedia.org/wiki/Base36) et comprend toujours six chiffres. Si la représentation en base 36 du nombre comprend moins de six chiffres, la partie {base-36-instance-id} est remplie avec des zéros pour qu’elle comporte 6 chiffres. Par exemple, une instance avec la partie {computer-name-prefix} « nsgvmss » et l’ID d’instance 85 aura « nsgvmss00002D » comme nom d’ordinateur.

>[!NOTE]
> Le préfixe du nom d’ordinateur est une propriété du modèle de groupe identique que vous pouvez définir. Il peut donc être différent du nom du groupe identique proprement dit.
