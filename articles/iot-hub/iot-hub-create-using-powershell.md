---
title: Créer un IoT Hub Azure à l’aide d’une cmdlet PowerShell | Microsoft Docs
description: Découvrez comment utiliser les cmdlets PowerShell pour créer un groupe de ressources, puis un hub IoT dans le groupe de ressources. Découvrez également comment supprimer le hub.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/29/2018
ms.author: robinsh
ms.openlocfilehash: da021e3ba0fd93a182ea76a1ba4b7042b325aacc
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92142368"
---
# <a name="create-an-iot-hub-using-the-new-aziothub-cmdlet"></a>Créez un IoT Hub à l’aide de la cmdlet New-AzIotHub

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

## <a name="introduction"></a>Introduction

Vous pouvez utiliser les cmdlets Azure PowerShell pour créer et gérer des IoT Hubs Azure. Ce didacticiel vous montre comment créer un IoT Hub à l’aide de PowerShell.

Pour suivre ce guide pratique, vous devez disposer d’un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="connect-to-your-azure-subscription"></a>Connectez-vous à un abonnement Azure

Si vous utilisez Cloud Shell, vous êtes déjà connecté à votre abonnement. Par contre, si vous exécutez PowerShell localement, entrez la commande suivante pour vous connecter à votre abonnement Azure :

```powershell
# Log into Azure account.
Login-AzAccount
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Vous avez besoin d’un groupe de ressources pour déployer un IoT Hub. Vous pouvez utiliser un groupe de ressources existant ou en créer un.

Pour créer un groupe de ressources pour votre hub IoT, utilisez la commande [New-AzResourceGroup](/powershell/module/az.Resources/New-azResourceGroup). Cet exemple crée un groupe de ressources appelé **MyIoTRG1** dans la région **USA Est** :

```azurepowershell-interactive
New-AzResourceGroup -Name MyIoTRG1 -Location "East US"
```

## <a name="create-an-iot-hub"></a>Créer un hub IoT

Pour créer un hub IoT dans le groupe de ressources que vous avez créé à l’étape précédente, utilisez la commande [New-AzIotHub](/powershell/module/az.IotHub/New-azIotHub). Cet exemple crée un hub **S1** appelé **MyTestIoTHub** dans la région **USA Est** :

```azurepowershell-interactive
New-AzIotHub `
    -ResourceGroupName MyIoTRG1 `
    -Name MyTestIoTHub `
    -SkuName S1 -Units 1 `
    -Location "East US"
```

Le nom du hub IoT doit être globalement unique.

[!INCLUDE [iot-hub-pii-note-naming-hub](../../includes/iot-hub-pii-note-naming-hub.md)]

Vous pouvez afficher tous les hubs IoT de votre abonnement à l’aide de la commande [Get-AzIotHub](/powershell/module/az.IotHub/Get-azIotHub) :

```azurepowershell-interactive
Get-AzIotHub
```

Cet exemple montre le hub IoT Standard S1 que vous avez créé à l’étape précédente.

Vous pouvez supprimer le hub IoT à l’aide de la commande [Remove-AzIotHub](/powershell/module/az.iothub/remove-aziothub) :

```azurepowershell-interactive
Remove-AzIotHub `
    -ResourceGroupName MyIoTRG1 `
    -Name MyTestIoTHub
```

Vous pouvez également supprimer un groupe de ressources et toutes les ressources qu’il contient à l’aide de la commande [Remove-AzResourceGroup](/powershell/module/az.Resources/Remove-azResourceGroup) :

```azurepowershell-interactive
Remove-AzResourceGroup -Name MyIoTRG1
```

## <a name="next-steps"></a>Étapes suivantes

Ayant déployé un hub IoT à l’aide d’une applet de commande PowerShell, vous pouvez aller encore plus loin en consultant les articles suivants :

* [Applets de commande PowerShell pour utiliser votre hub IoT](/powershell/module/az.iothub/).

* [API REST de fournisseur de ressources IoT Hub](/rest/api/iothub/iothubresource).

Pour en savoir plus sur le développement pour IoT Hub, consultez les articles suivants :

* [Présentation du SDK C](iot-hub-device-sdk-c-intro.md)

* [Kits de développement logiciel (SDK) Azure IoT](iot-hub-devguide-sdks.md)

Pour explorer davantage les capacités de IoT Hub, consultez :

* [Déploiement d’une IA sur des appareils de périmètre avec Azure IoT Edge](../iot-edge/quickstart-linux.md)