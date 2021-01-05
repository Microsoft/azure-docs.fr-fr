---
title: Activer et désactiver la console série Azure | Microsoft Docs
description: Comment activer et désactiver le service Console série Azure
services: virtual-machines
documentationcenter: ''
author: asinn826
manager: borisb
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm
ms.workload: infrastructure-services
ms.date: 8/20/2019
ms.author: alsin
ms.openlocfilehash: e09e08f8ba36cf576bc27551254225adee3bb0fd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75451305"
---
# <a name="enable-and-disable-the-azure-serial-console"></a>Activer et désactiver la console série Azure

Comme n’importe quelle autre ressource, la console série Azure peut être activée et désactivée. La console série est activée par défaut pour tous les abonnements dans Azure global. Actuellement, la désactivation de la console série désactive le service pour l’ensemble de votre abonnement. La désactivation ou la réactivation de la console série pour un abonnement requiert un accès de niveau contributeur ou supérieur dans l’abonnement.

Vous pouvez également désactiver la console série pour une machine virtuelle ou une instance de groupe de machines virtuelles identiques individuelle en désactivant les diagnostics de démarrage. Vous aurez besoin d’un accès de niveau collaborateur ou supérieur sur la machine virtuelle/le groupe de machines virtuelles identiques et votre compte de stockage des diagnostics de démarrage.

## <a name="vm-level-disable"></a>Désactiver au niveau de la machine virtuelle
La console série peut être désactivée pour une machine virtuelle ou un groupe de machines virtuelles identiques spécifique en désactivant le paramètre des diagnostics de démarrage. Désactivez les diagnostics de démarrage à partir du portail Azure afin de désactiver la console série pour la machine virtuelle ou le groupe de machines virtuelles identiques. Si vous utilisez la console série sur un groupe de machines virtuelles identiques, veillez à mettre à niveau vos instances de groupe de machines virtuelles identiques vers les modèles les plus récents.


## <a name="subscription-level-enabledisable"></a>Activer/désactiver au niveau de l’abonnement

> [!NOTE]
> Vérifiez que vous êtes dans le cloud approprié (cloud public Azure, cloud Azure US Government) avant d’exécuter cette commande. Vous pouvez vérifier en consultant `az cloud list` et définir votre cloud avec `az cloud set -n <Name of cloud>`.

### <a name="azure-cli"></a>Azure CLI

La console série peut être désactivée et réactivée pour l’intégralité d’un abonnement à l’aide des commandes suivantes dans l’interface de ligne de commande Azure (vous pouvez utiliser le bouton « Essayer » pour lancer une instance d’Azure Cloud Shell dans laquelle vous pouvez exécuter les commandes) :

Pour désactiver la console série pour un abonnement, utilisez les commandes suivantes :
```azurecli-interactive
subscriptionId=$(az account show --output=json | jq -r .id)

az resource invoke-action --action disableConsole --ids "/subscriptions/$subscriptionId/providers/Microsoft.SerialConsole/consoleServices/default" --api-version="2018-05-01"
```

Pour activer la console série pour un abonnement, utilisez les commandes suivantes :
```azurecli-interactive
subscriptionId=$(az account show --output=json | jq -r .id)

az resource invoke-action --action enableConsole --ids "/subscriptions/$subscriptionId/providers/Microsoft.SerialConsole/consoleServices/default" --api-version="2018-05-01"
```

Pour obtenir l’état activé/désactivé actuel de la console série pour un abonnement, utilisez les commandes suivantes :
```azurecli-interactive
subscriptionId=$(az account show --output=json | jq -r .id)

az resource show --ids "/subscriptions/$subscriptionId/providers/Microsoft.SerialConsole/consoleServices/default" --output=json --api-version="2018-05-01" | jq .properties
```

### <a name="powershell"></a>PowerShell

La console série peut aussi être activée et désactivée à l’aide de PowerShell.

Pour désactiver la console série pour un abonnement, utilisez les commandes suivantes :
```azurepowershell-interactive
$subscription=(Get-AzContext).Subscription.Id

Invoke-AzResourceAction -Action disableConsole -ResourceId /subscriptions/$subscription/providers/Microsoft.SerialConsole/consoleServices/default -ApiVersion 2018-05-01
```

Pour activer la console série pour un abonnement, utilisez les commandes suivantes :
```azurepowershell-interactive
$subscription=(Get-AzContext).Subscription.Id

Invoke-AzResourceAction -Action enableConsole -ResourceId /subscriptions/$subscription/providers/Microsoft.SerialConsole/consoleServices/default -ApiVersion 2018-05-01
```

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur la [console série Azure pour les machines virtuelles Linux](./serial-console-linux.md)
* En savoir plus sur la [console série Azure pour les machines virtuelles Windows](./serial-console-windows.md)
* En savoir plus sur les [options de gestion de l’alimentation dans la console série Azure](./serial-console-power-options.md)