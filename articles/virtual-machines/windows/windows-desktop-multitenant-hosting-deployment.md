---
title: Guide pratique pour déployer Windows 10 sur Azure avec des droits d’hébergement multilocataire
description: Découvrez comment optimiser les avantages de votre contrat Software Assurance pour Windows pour mettre des licences locales sur Azure avec des droits d’hébergement multilocataire.
author: xujing
ms.service: virtual-machines-windows
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 1/24/2018
ms.author: xujing
ms.openlocfilehash: 101f2cfe57624502764d145351a6343cfdd2a334
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96572862"
---
# <a name="how-to-deploy-windows-10-on-azure-with-multitenant-hosting-rights"></a>Guide pratique pour déployer Windows 10 sur Azure avec des droits d’hébergement multilocataire 
Pour les clients avec Windows 10 Entreprise E3/E5 par utilisateur ou Windows Virtual Desktop Access par utilisateur (licences d’abonnement utilisateur ou licences d’abonnement utilisateur de composant additionnel), les droits d’hébergement multilocataire pour Windows 10 permettent de placer les licences Windows 10 sur le cloud et d’exécuter des machines virtuelles Windows 10 sur Azure sans acheter de licence supplémentaire. Pour plus d’informations, consultez [Multitenant Hosting for Windows 10](https://www.microsoft.com/en-us/CloudandHosting/licensing_sca.aspx) (Hébergement multilocataire pour Windows 10).

> [!NOTE]
> Cet article montre comment tirer parti des licences pour les images Windows 10 Pro Desktop sur Place de marché.
> - Pour les images Windows 7, 8.1, 10 Entreprise (x64) sur Place de marché pour les abonnements MSDN, reportez-vous au [client Windows dans Azure pour les scénarios de développement/test](client-images.md)
> - Pour les avantages des licences Windows Server, reportez-vous à [Avantages Utilisation hybride d’Azure pour les images Windows Server](hybrid-use-benefit-licensing.md).
>

## <a name="deploying-windows-10-image-from-azure-marketplace"></a>Déploiement de l’image Windows 10 à partir de la Place de marché Azure 
Pour les déploiements PowerShell, CLI et avec un modèle Azure Resource Manager, l’image Windows 10 est accessible avec le nom d’éditeur, l’offre et la référence (SKU) indiqués ci-après.

| Système d''exploitation  |      PublisherName      |  Offre | Sku |
|:----------|:-------------:|:------|:------|
| Windows 10 Pro    | MicrosoftWindowsDesktop | Windows-10  | RS2-Pro   |
| Windows 10 Pro N  | MicrosoftWindowsDesktop | Windows-10  | RS2-ProN  |
| Windows 10 Pro    | MicrosoftWindowsDesktop | Windows-10  | RS3-Pro   |
| Windows 10 Pro N  | MicrosoftWindowsDesktop | Windows-10  | RS3-ProN  |

## <a name="qualify-for-multi-tenant-hosting-rights"></a>Bénéficier de droits d’hébergement multi-locataire 
Pour bénéficier de droits d’hébergement multi-locataire et pour exécuter des images Windows 10 sur Azure, les utilisateurs doivent disposer de l’un des abonnements suivants : 

-   Microsoft 365 E3/E5 
-   Microsoft 365 F3 
-   Microsoft 365 A3/A5 
-   Windows 10 Entreprise E3/E5
-   Windows 10 Éducation A3/A5 
-   Windows VDA E3/E5


## <a name="uploading-windows-10-vhd-to-azure"></a>Chargement d’un VHD Windows 10 sur Azure
Si vous chargez un VHD Windows 10 généralisé, notez que Windows 10 n’a pas de compte Administrateur intégré activé par défaut. Pour activer le compte Administrateur intégré, incluez la commande suivante dans le cadre de l’extension de script personnalisé.

```powershell
Net user <username> /active:yes
```

L’extrait de code PowerShell suivant permet de marquer tous les comptes Administrateur comme étant actifs, y compris le compte Administrateur intégré. Cet exemple est utile si le nom d’utilisateur du compte Administrateur intégré est inconnu.
```powershell
$adminAccount = Get-WmiObject Win32_UserAccount -filter "LocalAccount=True" | ? {$_.SID -Like "S-1-5-21-*-500"}
if($adminAccount.Disabled)
{
    $adminAccount.Disabled = $false
    $adminAccount.Put()
}
```
Pour plus d'informations : 
* [Guide pratique pour charger un VHD sur Azure](upload-generalized-managed.md)
* [Guide pratique pour préparer un VHD Windows à charger sur Azure](prepare-for-upload-vhd-image.md)


## <a name="deploying-windows-10-with-multitenant-hosting-rights"></a>Déploiement de Windows 10 avec des droits d’hébergement multilocataire
Vérifiez que vous avez [installé et configuré la dernière version d’Azure PowerShell](/powershell/azure/). Une fois votre disque dur virtuel préparé, chargez-le dans votre compte Azure Storage en utilisant l’applet de commande `Add-AzVhd` comme suit :

```powershell
Add-AzVhd -ResourceGroupName "myResourceGroup" -LocalFilePath "C:\Path\To\myvhd.vhd" `
    -Destination "https://mystorageaccount.blob.core.windows.net/vhds/myvhd.vhd"
```


**Déploiement à l’aide d’un modèle Azure Resource Manager** Dans vos modèles Resource Manager, vous pouvez spécifier un paramètre supplémentaire pour `licenseType`. Pour en savoir plus sur la création de modèles Azure Resource Manager, [cliquez ici](../../azure-resource-manager/templates/template-syntax.md). Une fois que votre disque dur virtuel téléchargé dans Azure, modifiez votre modèle Resource Manager pour inclure le type de licence dans le fournisseur de calcul et déployez votre modèle normalement :
```json
"properties": {
    "licenseType": "Windows_Client",
    "hardwareProfile": {
        "vmSize": "[variables('vmSize')]"
    }
```

**Déploiement par le biais de PowerShell** Quand vous déployez votre machine virtuelle Windows Server par le biais de PowerShell, vous disposez d’un paramètre supplémentaire pour `-LicenseType`. Une fois votre disque dur virtuel chargé dans Azure, vous créez une machine virtuelle en utilisant `New-AzVM` et spécifiez le type de licence comme suit :
```powershell
New-AzVM -ResourceGroupName "myResourceGroup" -Location "West US" -VM $vm -LicenseType "Windows_Client"
```

## <a name="verify-your-vm-is-utilizing-the-licensing-benefit"></a>Vérifiez que votre machine virtuelle utilise l’avantage de licence
Une fois votre machine virtuelle déployée par le biais de PowerShell ou de Resource Manager, vérifiez le type de licence avec `Get-AzVM` comme suit :
```powershell
Get-AzVM -ResourceGroup "myResourceGroup" -Name "myVM"
```

Le résultat ressemble à l’exemple suivant pour Windows 10 avec le type de licence correct :

```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              : Windows_Client
```

Ce résultat est différent de celui de la machine virtuelle suivante déployée sans la licence Azure Hybrid Use Benefit, comme une machine virtuelle déployée directement depuis la Galerie Azure :

```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              :
```

## <a name="additional-information-about-joining-azure-ad"></a>Informations supplémentaires pour rejoindre Azure AD
>[!NOTE]
>Azure approvisionne toutes les machines virtuelles Windows avec un compte Administrateur intégré, qui ne peut pas être utilisé pour joindre AAD. Par exemple, *Paramètres > Compte > Accès Professionnel ou Scolaire > + Se connecter* ne fonctionne pas. Vous devez créer un deuxième compte Administrateur et ouvrir une session sous celui-ci pour joindre Azure AD manuellement. Vous pouvez également configurer Azure AD à l’aide d’un package de provisionnement. Pour plus d’informations, utilisez le lien approprié dans la section *Étapes suivantes*.
>
>

## <a name="next-steps"></a>Étapes suivantes
- Découvrez plus en détail la [configuration de VDA pour Windows 10](/windows/deployment/vda-subscription-activation).
- Découvrez plus en détail [l’hébergement multilocataire pour Windows 10](https://www.microsoft.com/en-us/CloudandHosting/licensing_sca.aspx).
