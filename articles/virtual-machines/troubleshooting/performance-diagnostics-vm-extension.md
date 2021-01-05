---
title: Extension de diagnostic de performance des machines virtuelles Azure pour Windows | Microsoft Docs
description: Présente l’extension de diagnostic de performance des machines virtuelles Azure pour Windows.
services: virtual-machines-windows
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: na
tags: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: 9edba575b35613abb8bc3081964a37b838bb358b
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97656593"
---
# <a name="azure-performance-diagnostics-vm-extension-for-windows"></a>Extension de diagnostic de performance des machines virtuelles Azure pour Windows

L’extension de machine virtuelle Diagnostics des performances Azure permet de collecter des données de diagnostic des performances sur des machines virtuelles Windows. Elle effectue une analyse et fournit un rapport de résultats et de recommandations permettant d’identifier et de résoudre les problèmes de performances sur la machine virtuelle. Cette extension installe un outil de résolution des problèmes appelé [PerfInsights](./how-to-use-perfinsights.md).

> [!NOTE]
> Si vous souhaitez exécuter des diagnostics sur votre machine virtuelle à partir du portail Azure pour les machines virtuelles non classiques, il est recommandé d’utiliser la nouvelle expérience. Pour plus d’informations, consultez [Diagnostics de performances pour les machines virtuelles Azure](performance-diagnostics.md) 

## <a name="prerequisites"></a>Prérequis

Cette extension peut être installée sur
* Windows Server 2019
* Windows Server 2016
* Windows Server 2012 R2
* Windows Server 2012
* Windows Server 2008 R2
* Windows 10
* Windows 8.1
* Windows 8

## <a name="extension-schema"></a>Schéma d’extensions
Le code JSON suivant montre le schéma de l’extension de machine virtuelle Diagnostics des performances Azure. Elle a besoin du nom et de la clé d’un compte de stockage pour stocker la sortie et le rapport de diagnostic. Ces valeurs sont sensibles. La clé de compte de stockage doit être stockée dans une configuration de paramètres protégés. Les données des paramètres protégés de l’extension de machine virtuelle Azure sont chiffrées, et ne sont déchiffrées que sur la machine virtuelle cible. Notez que **storageAccountName** et **storageAccountKey** respectent la casse. Les autres paramètres obligatoires sont indiqués dans la section ci-dessous.

```JSON
    {
      "name": "[concat(parameters('vmName'),'/AzurePerformanceDiagnostics')]",
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "location": "[parameters('location')]",
      "apiVersion": "2015-06-15",
      "properties": {
        "publisher": "Microsoft.Azure.Performance.Diagnostics",
        "type": "AzurePerformanceDiagnostics",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "storageAccountName": "[parameters('storageAccountName')]",
          "performanceScenario": "[parameters('performanceScenario')]",
          "traceDurationInSeconds": "[parameter('traceDurationInSeconds')]",
          "perfCounterTrace": "[parameters('perfCounterTrace')]",
          "networkTrace": "[parameters('networkTrace')]",
          "xperfTrace": "[parameters('xperfTrace')]",
          "storPortTrace": "[parameters('storPortTrace')]",
          "srNumber": "[parameters('srNumber')]",
          "requestTimeUtc":  "[parameters('requestTimeUtc')]",
          "resourceId": "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
        },
        "protectedSettings": {
            "storageAccountKey": "[parameters('storageAccountKey')]"        
        }
      }
    }
```

### <a name="property-values"></a>Valeurs de propriétés

| Nom | Valeur/Exemple | Description |
|--|--|--|
| apiVersion | 2015-06-15 | Version de l’API. |
| publisher | Microsoft.Azure.Performance.Diagnostics | Espace de noms du serveur de publication de l’extension. |
| type | AzurePerformanceDiagnostics | Type de l’extension de machine virtuelle. |
| typeHandlerVersion | 1.0 | Version du gestionnaire d’extensions. |
| performanceScenario | basic | Scénario de performances dont les données seront capturées. Les valeurs valides sont : **basic**, **vmslow**, **azurefiles** et **custom**. |
| traceDurationInSeconds | 300 | Durée des traces si l’une des options de trace est sélectionnée. |
| perfCounterTrace | p | Option permettant d’activer la trace du compteur de performances. Les valeurs valides sont **p** ou une valeur vide. Si vous ne souhaitez pas capturer cette trace, laissez la valeur vide. |
| networkTrace | n | Option permettant d’activer Network Trace. Valeurs valides : **n** ou une valeur vide. Si vous ne souhaitez pas capturer cette trace, laissez la valeur vide. |
| xperfTrace | x | Option permettant d’activer la trace XPerf. Les valeurs valides sont **x** ou une valeur vide. Si vous ne souhaitez pas capturer cette trace, laissez la valeur vide. |
| storPortTrace | s | Option permettant d’activer la trace StorPort. Valeurs valides : **s** ou une valeur vide. Si vous ne souhaitez pas capturer cette trace, laissez la valeur vide. |
| srNumber | 123452016365929 | Numéro du ticket de support, s’il est disponible. Laissez la valeur vide si vous ne l’avez pas. |
| requestTimeUtc | 2017-09-28T22:08:53.736Z | Date et heure actuelles UTC. Vous n’avez pas besoin de fournir cette valeur si vous utilisez le portail pour installer cette extension. |
| resourceId | /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/{resourceType}/{resourceName} | Identificateur unique d’une machine virtuelle. |
| storageAccountName | mystorageaccount | Nom du compte de stockage pour stocker les journaux d’activité et les résultats de diagnostic. |
| storageAccountKey | lDuVvxuZB28NNP…hAiRF3voADxLBTcc== | Clé du compte de stockage. |

## <a name="install-the-extension"></a>Installer l’extension

Suivez ces instructions pour installer l’extension sur des machines virtuelles Windows :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez la machine virtuelle sur laquelle vous voulez installer cette extension.

    ![Capture du Portail Azure, avec les machines virtuelles mises en surbrillance](media/performance-diagnostics-vm-extension/select-the-virtual-machine.png)
3. Sélectionnez le panneau **Extensions**, puis sélectionnez **Ajouter**.

    ![Capture d’écran du panneau Extensions, avec l’option Ajouter mise en surbrillance](media/performance-diagnostics-vm-extension/select-extensions.png)
4. Sélectionnez **Diagnostics des performances Azure**, lisez les conditions générales, puis sélectionnez **Créer**.

    ![Capture de l’écran Nouvelle ressources, avec l’option Diagnostics des performances Azure mise en surbrillance](media/performance-diagnostics-vm-extension/create-azure-performance-diagnostics-extension.png)
5. Indiquez les valeurs des paramètres d’installation, puis sélectionnez **OK** pour installer l’extension. Pour plus d’informations sur les scénarios pris en charge, consultez la page [Guide pratique de PerfInsights](how-to-use-perfinsights.md#supported-troubleshooting-scenarios). 

    ![Capture d’écran de la boîte de dialogue Installer l’extension](media/performance-diagnostics-vm-extension/install-the-extension.png)
6. Un message indique que l’installation a réussi.

    ![Capture d’écran du message Approvisionnement réussi](media/performance-diagnostics-vm-extension/provisioning-succeeded-message.png)

    > [!NOTE]
    > L’extension s’exécute dès la fin de l’approvisionnement. Cela prend deux minutes maximum pour le scénario de base. Pour les autres scénarios, la durée d’exécution est spécifiée pendant l’installation.

## <a name="remove-the-extension"></a>Supprimer l’extension

Pour supprimer l’extension d’une machine virtuelle, effectuez ces étapes :

1. Connectez-vous au [Portail Azure](https://portal.azure.com), sélectionnez la machine virtuelle sur laquelle vous voulez supprimer cette extension, puis sélectionnez le panneau **Extensions**. 
2. Sélectionnez ( **...** ) à côté de l’entrée Extension Diagnostics des performances dans la liste, puis sélectionnez **Désinstaller**.

    ![Capture d’écran du panneau Extensions, avec l’option Désinstaller mise en surbrillance](media/performance-diagnostics-vm-extension/uninstall-the-extension.png)

    > [!NOTE]
    > Vous pouvez également sélectionner l’entrée de l’extension et sélectionner l’option **Désinstaller**.

## <a name="template-deployment"></a>Déploiement de modèle

Les extensions de machines virtuelles Azure peuvent être déployées avec des modèles Azure Resource Manager. Le schéma JSON détaillé dans la section précédente est utilisable dans un modèle Azure Resource Manager. Il exécute l’extension de machine virtuelle Diagnostics des performances Azure pendant un déploiement de modèle Azure Resource Manager. Voici un exemple de modèle :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "yourVMName"
    },
    "location": {
      "type": "string",
      "defaultValue": "southcentralus"
    },
    "storageAccountName": {
      "type": "securestring",
      "defaultValue": "yourStorageAccount"
    },
    "storageAccountKey": {
      "type": "securestring",
      "defaultValue": "yourStorageAccountKey"
    },
    "performanceScenario": {
      "type": "string",
      "defaultValue": "basic"
    },
    "srNumber": {
      "type": "string",
      "defaultValue": ""
    },
  "traceDurationInSeconds": {
    "type": "int",
    "defaultValue": 300
  },
    "perfCounterTrace": {
      "type": "string",
      "defaultValue": "p"
    },
    "networkTrace": {
      "type": "string",
      "defaultValue": ""
    },
    "xperfTrace": {
      "type": "string",
      "defaultValue": ""
    },
    "storPortTrace": {
      "type": "string",
      "defaultValue": ""
    },
    "requestTimeUtc": {
      "type": "string",
      "defaultValue": "10/2/2017 11:06:00 PM"
    }       
  },
  "resources": [
    {
      "name": "[concat(parameters('vmName'),'/AzurePerformanceDiagnostics')]",
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "location": "[parameters('location')]",
      "apiVersion": "2015-06-15",
      "properties": {
        "publisher": "Microsoft.Azure.Performance.Diagnostics",
        "type": "AzurePerformanceDiagnostics",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "storageAccountName": "[parameters('storageAccountName')]",
          "performanceScenario": "[parameters('performanceScenario')]",
          "traceDurationInSeconds": "[parameters('traceDurationInSeconds')]",
          "perfCounterTrace": "[parameters('perfCounterTrace')]",
          "networkTrace": "[parameters('networkTrace')]",
          "xperfTrace": "[parameters('xperfTrace')]",
          "storPortTrace": "[parameters('storPortTrace')]",
          "srNumber": "[parameters('srNumber')]",
          "requestTimeUtc":  "[parameters('requestTimeUtc')]",
          "resourceId": "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
        },
        "protectedSettings": {
            "storageAccountKey": "[parameters('storageAccountKey')]"
        }
      }
    }
  ]
}
```

## <a name="powershell-deployment"></a>Déploiement PowerShell

Vous pouvez utiliser la commande `Set-AzVMExtension` pour déployer l’extension de machine virtuelle Diagnostics des performances Azure sur une machine virtuelle existante.

PowerShell

```
$PublicSettings = @{ "storageAccountName"="mystorageaccount";"performanceScenario"="basic";"traceDurationInSeconds"=300;"perfCounterTrace"="p";"networkTrace"="";"xperfTrace"="";"storPortTrace"="";"srNumber"="";"requestTimeUtc"="2017-09-28T22:08:53.736Z";"resourceId"="VMResourceId" }
$ProtectedSettings = @{"storageAccountKey"="mystoragekey" }

Set-AzVMExtension -ExtensionName "AzurePerformanceDiagnostics" `
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Publisher "Microsoft.Azure.Performance.Diagnostics" `
    -ExtensionType "AzurePerformanceDiagnostics" `
    -TypeHandlerVersion 1.0 `
    -Settings $PublicSettings `
    -ProtectedSettings $ProtectedSettings `
    -Location WestUS
```

## <a name="information-on-the-data-captured"></a>Informations sur les données capturées
L’outil PerfInsights collecte différents types de journaux d’activité, de configurations et de données de diagnostic, selon le scénario sélectionné. Pour plus d’informations, consultez la [documentation PerfInsights](./how-to-use-perfinsights.md).

## <a name="view-and-share-the-results"></a>Afficher et partager les résultats

La sortie de l’extension peut être obtenue dans le fichier zip qui a été chargé dans le compte de stockage indiqué pendant l’installation et est partagée pendant 30 jours à l’aide de [signatures d’accès partagé (SAP)](../../storage/common/storage-sas-overview.md). Ce fichier zip contient des journaux de diagnostic et un rapport avec les résultats et les recommandations. Un lien SAS vers le fichier zip de sortie est fourni dans le fichier texte nommé *zipfilename* _saslink.txt, qui se trouve dans le dossier **C:\Packages\Plugins\Microsoft.Azure.Performance.Diagnostics.AzurePerformanceDiagnostics\\\<version>** . Toute personne qui dispose de ce lien est en mesure de télécharger le fichier zip.

Pour aider l’ingénieur qui travaille sur le ticket de support, Microsoft a la possibilité d’utiliser ce lien SAP pour télécharger les données de diagnostic.

Pour afficher le rapport, extrayez le fichier zip et ouvrez le fichier **PerfInsights Report.htm**.

Vous devriez également pouvoir télécharger le fichier zip directement sur le portail, en sélectionnant l’extension.

![Capture d’écran de l’état détaillé des diagnostics des performances](media/performance-diagnostics-vm-extension/view-detailed-status.png)

> [!NOTE]
> Le lien SAP affiché sur le portail ne fonctionne pas parfois. Cela peut être dû à une URL malformée pendant les opérations de codage et de décodage. Vous pourrez alors récupérer le lien directement dans le fichier *_saslink.txt de la machine virtuelle.

## <a name="troubleshoot-and-support"></a>Dépannage et support technique

- Il peut arriver que l’état du déploiement de l’extension (dans la zone de notification) indique « Déploiement en cours » même si l’extension est correctement approvisionnée.

    Vous pouvez ignorer ce problème sans risque tant que l’état de l’extension indique que l’extension est correctement approvisionnée.
- Vous pouvez résoudre certains problèmes survenant lors de l’installation en utilisant les journaux d’activité de l’extension. La sortie de l’exécution de l’extension est enregistrée dans les fichiers qui que se trouvent dans le répertoire suivant :

    `C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Performance.Diagnostics.AzurePerformanceDiagnostics\<version>`

Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, vous pouvez contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [Site du support Azure](https://azure.microsoft.com/support/options/), puis cliquez sur **Obtenir un support**. Pour plus d’informations sur le support Azure, lisez la [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).