---
title: Erreur du gestionnaire de démarrage Windows - État 0xC0000428 - Hachage d’image non valide
titlesuffix: Azure Virtual Machines
description: Cet article décrit les étapes à suivre pour résoudre les problèmes liés à l’utilisation d’une image en préversion alors que la période d’évaluation a expiré, ce qui empêche le démarrage d’une machine virtuelle Azure.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 26c411f9-6c22-4f45-a445-c5781e547828
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 09/03/2020
ms.author: v-miegge
ms.openlocfilehash: e58e349d7b7385ec913986c39462c17deadcb61d
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91969600"
---
# <a name="windows-boot-manager-error---0xc0000428-status-invalid-image-hash"></a>Erreur du gestionnaire de démarrage Windows - État 0xC0000428 - Hachage d’image non valide

Cet article décrit les étapes à suivre pour résoudre les problèmes liés à l’utilisation d’une image en préversion alors que la période d’évaluation a expiré, ce qui empêche le démarrage d’une machine virtuelle Azure.

## <a name="symptom"></a>Symptôme

Quand vous utilisez [Diagnostics de démarrage](./boot-diagnostics.md) pour visualiser la capture d’écran de la machine virtuelle, vous constatez que cette capture d’écran affiche le Gestionnaire de démarrage Windows avec le message :

  `File: \windows\system32\boot\winload.exe`

  `Status: 0xc0000428`

  `Info: Windows cannot verify the digital signature for this file.`

  ![La figure 1 montre le Gestionnaire de démarrage Windows avec l’état « Ox0000428 » et les informations « Windows ne peut pas vérifier la signature numérique de ce fichier ».](./media/windows-boot-error-invalid-image-hash/1-cannot-verify-signature.png)

ou le message :

  `File: \Windows\system32\winload.exe`

  `Status: 0xc0000428`

  `Info: The digital signature for this file couldn't be verified.`
    
  ![La figure 2 montre le Gestionnaire de démarrage Windows avec l’état « Ox0000428 » et les informations « La signature numérique de ce fichier n’a pas pu être vérifiée ».](./media/windows-boot-error-invalid-image-hash/2-digital-signature-not-verified.png)

## <a name="cause"></a>Cause

L’image qui a été utilisée pour créer la machine virtuelle était une image en préversion avec une date d’expiration, au lieu d’une image RTM (en version finalisée). 

Les images en préversion ont un cycle de vie spécifique et la capture d’écran que vous voyez est affichée quand vous avez dépassé la date d’expiration, ce qui signifie que la période d’évaluation de l’image est terminée.

### <a name="example-of-preview-images"></a>Exemple d’images en préversion

`a699494373c04fc0bc8f2bb1389d6106__Windows-Server-Technical-Preview-201505.01-en.us-127GB.vhd`

Vous n’êtes pas en mesure d’étendre la date d’expiration d’une image d’aperçu. Une fois que la préversion a expiré, la machine virtuelle ne peut plus démarrer.

- En fonction du produit, la période d’évaluation peut varier. Par exemple, les images en préversion de Windows ont une période d’évaluation de 180 jours.

- Dans Azure, toutes les images pour Windows qui sont des préversions incluent une note dans leur description indiquant qu’elles ne sont pas destinées à la production et qu’elles peuvent être utilisées seulement pour une période d’évaluation donnée ou comme « version préliminaire ».

## <a name="solution"></a>Solution

Si votre image est une image en préversion, il n’existe aucun moyen d’étendre la date d’expiration de l’image utilisée : vous devez [déployer une nouvelle machine virtuelle](../windows/quick-create-portal.md) en utilisant une image qui n’est pas en préversion. Les étapes ci-dessous vous aideront à identifier si vous avez utilisé une image en préversion ; elles fournissent aussi des ressources pour vous aider à transférer des données à partir depuis cette machine virtuelle vers une nouvelle machine virtuelle. Si vous avez identifié que l’image était bien une image en préversion, cette image n’est pas récupérable, car elle a maintenant expiré.

Selon vos préférences, vous pouvez utiliser Azure PowerShell ou Azure CLI pour interroger votre image afin de déterminer s’il s’agit d’une image en préversion. Vous pouvez utiliser ces commandes pour vérifier si l’image est une image en préversion.

### <a name="query-using-azure-powershell"></a>Interroger en utilisant Azure PowerShell

1. Ouvrez l’application Windows PowerShell.
1. Exécutez les commandes suivantes :

   ```powershell
   $locName = "<LOCATION>" 
   $pubName = "<PUBLISHER NAME>" 
   $offerName = "<OFFER NAME>" 
   $skuName= "<YEAR WHEN THIS IMAGE WAS RELEASED>"
   Get-AzVMImagePublisher -Location $locName | Select $pubName
   Get-AzVMImageOffer -Location $locName -PublisherName $pubName | Select Offer
   Get-AzVMImageSku -Location $locName -PublisherName $pubName -Offer $offerName | Select Skus
   Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Skus $skuName | Select Version
   ```

- Dans les commandes précédentes, remplacez `<LOCATION>`, `<PUBLISHER NAME>`, `<OFFER NAME>` et `<YEAR WHEN THIS IMAGE WAS RELEASED>` par les informations indiquées. Supprimez également les symboles « < » et « > ».

  Voir l’exemple suivant :

  ```powershell
  $locName = "West US" 
  $pubName = "MicrosoftWindowsServer" 
  $offerName = "WindowsServer" 
  $skuName= "2016-Datacenter"
  Get-AzVMImagePublisher -Location $locName | Select $pubName
  Get-AzVMImageOffer -Location $locName -PublisherName $pubName | Select Offer
  Get-AzVMImageSku -Location $locName -PublisherName $pubName -Offer $offerName | Select Skus
  Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Skus $skuName | Select Version
  ```

### <a name="query-using-the-azure-cli"></a>Interroger en utilisant Azure CLI

1. Si vous ne l’avez pas déjà installé, vous devez [installer Azure CLI](/cli/azure/install-azure-cli).
1. Une fois le téléchargement effectué, utilisez l’invite de commandes ou PowerShell pour entrer la commande `az login`, puis connectez-vous avec vos informations d’identification de compte.
1. Une fois connecté, entrez les commandes suivantes :

   ```powershell
   az vm image list-publishers --location "<LOCATION>"
   az vm image list-offers --location "West US" --publisher "<PUBLISHER NAME>"
   az vm image list-skus --location "West US" --publisher "<PUBLISHER NAME>" --offer "<OFFER NAME>"
   az vm image list  --location "West US" --publisher "<PUBLISHER NAME>" --offer "<OFFER NAME>" --sku "<YEAR WHEN THIS IMAGE WAS RELEASED>"
   ```

- Dans les commandes précédentes, remplacez `<LOCATION>`, `<PUBLISHER NAME>`, `<OFFER NAME>` et `<YEAR WHEN THIS IMAGE WAS RELEASED>` par les informations indiquées. Supprimez également les symboles « < » et « > ».

  Voir l’exemple suivant :

  ```powershell
  az vm image list-publishers --location "West US"
  az vm image list-offers --location "West US" --publisher "MicrosoftWindowsServer"
  az vm image list-skus --location "West US" --publisher "MicrosoftWindowsServer" --offer "WindowsServer"
  az vm image list  --location "West US" --publisher "MicrosoftWindowsServer" --offer "WindowsServer" --sku "2016-Datacenter"
  ```