---
title: Créer un IoT Hub Azure à l’aide d’un modèle (PowerShell) | Microsoft Docs
description: Comment utiliser un modèle Azure Resource Manager pour créer un hub IoT avec Azure PowerShell.
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/02/2019
ms.openlocfilehash: 1fedadfa7e5b4ec3d7de30d0ad3ef1b1bfa0e0ec
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92144392"
---
# <a name="create-an-iot-hub-using-azure-resource-manager-template-powershell"></a>Créer un IoT Hub avec un modèle Azure Resource Manager (PowerShell)

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

Découvrez comment utiliser un modèle Azure Resource Manager pour créer un hub IoT et un groupe de consommateurs. Les modèles Azure Resource Manager sont des fichiers JSON qui définissent les ressources nécessaires au déploiement de votre solution. Pour plus d’informations sur le développement de modèles Azure Resource Manager, consultez la [documentation Azure Resource Manager](../azure-resource-manager/index.yml).

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="create-an-iot-hub"></a>Créer un hub IoT

Le modèle Resource Manager utilisé dans ce guide de démarrage rapide vient des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-iothub-with-consumergroup-create/). Voici une copie du modèle :

[!code-json[iothub-creation](~/quickstart-templates/101-iothub-with-consumergroup-create/azuredeploy.json)]

Le modèle crée un hub Azure IoT avec trois points de terminaison (hub d’événements, cloud-à-appareil et messagerie) et un groupe de consommateurs. Pour obtenir d’autres exemples de modèles, consultez [Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Devices&pageNumber=1&sort=Popular). Vous trouverez le schéma des modèles IoT Hub [ici](/azure/templates/microsoft.devices/iothub-allversions).

Il existe plusieurs méthodes pour déployer un modèle.  Dans ce tutoriel, vous utilisez Azure PowerShell.

Pour exécuter le script PowerShell, sélectionnez **Essayer** pour ouvrir Azure Cloud Shell. Pour coller le script, cliquez avec le bouton droit dans le shell, puis sélectionnez Coller :

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$iotHubName = Read-Host -Prompt "Enter the IoT Hub name"

New-AzResourceGroup -Name $resourceGroupName -Location "$location"
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-iothub-with-consumergroup-create/azuredeploy.json" `
    -iotHubName $iotHubName
```

Comme vous pouvez le voir dans le script PowerShell, le modèle utilisé provient des modèles de démarrage rapide Azure. Pour utiliser votre propre modèle, vous devez d’abord charger le fichier de modèle dans Cloud Shell, puis utiliser le commutateur `-TemplateFile` pour spécifier le nom de fichier.  Pour obtenir un exemple, consultez [Déployer le modèle](../azure-resource-manager/templates/quickstart-create-templates-use-visual-studio-code.md?tabs=PowerShell#deploy-the-template).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez déployé un hub IoT en utilisant un modèle Azure Resource Manager, vous pouvez aller plus loin :

* Découvrez les capacités de [l’API REST du fournisseur de ressources IoT Hub][lnk-rest-api].
* Pour plus d’informations sur les capacités d’Azure Resource Manager, consultez [Vue d’ensemble d’Azure Resource Manager][lnk-azure-rm-overview].
* Pour connaître la syntaxe JSON et les propriétés à utiliser dans les modèles, consultez [Types de ressources Microsoft.Devices](/azure/templates/microsoft.devices/iothub-allversions).

Pour en savoir plus sur le développement pour IoT Hub, consultez les articles suivants :

* [Présentation du SDK C][lnk-c-sdk]
* [Kits de développement logiciel (SDK) Azure IoT][lnk-sdks]

Pour explorer davantage les capacités de IoT Hub, consultez :

* [Déploiement d’une IA sur des appareils de périmètre avec Azure IoT Edge][lnk-iotedge]

<!-- Links -->
[lnk-free-trial]: https://azure.microsoft.com/pricing/free-trial/
[lnk-azure-portal]: https://portal.azure.com/
[lnk-status]: https://azure.microsoft.com/status/
[lnk-powershell-install]: /powershell/azure/install-Az-ps
[lnk-rest-api]: /rest/api/iothub/iothubresource
[lnk-azure-rm-overview]: ../azure-resource-manager/management/overview.md
[lnk-powershell-arm]: ../azure-resource-manager/management/manage-resources-powershell.md

[lnk-c-sdk]: iot-hub-device-sdk-c-intro.md
[lnk-sdks]: iot-hub-devguide-sdks.md

[lnk-iotedge]: ../iot-edge/quickstart-linux.md