---
title: 'Tutoriel : Sécuriser un serveur web Windows à l’aide de certificats TLS/SSL dans Azure'
description: Avec ce tutoriel, vous allez apprendre à utiliser Azure PowerShell pour sécuriser une machine virtuelle Windows qui exécute le serveur web IIS avec les certificats TLS/SSL stockés dans Azure Key Vault.
author: cynthn
ms.service: virtual-machines-windows
ms.subservice: security
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 02/09/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 3b65d04b383fdc0a409e23ab6b6649604be502c6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86525584"
---
# <a name="tutorial-secure-a-web-server-on-a-windows-virtual-machine-in-azure-with-tlsssl-certificates-stored-in-key-vault"></a>Tutoriel : Sécuriser un serveur web sur une machine virtuelle Windows dans Azure avec les certificats TLS/SSL stockés dans Key Vault

> [!NOTE]
> Cette documentation s’applique uniquement aux images généralisées. Si vous tentez d’effectuer ce tutoriel avec un disque spécialisé, vous recevrez une erreur. 

Pour sécuriser les serveurs web, vous pouvez utiliser un certificat TLS (Transport Layer Security), anciennement SSL (Secure Sockets Layer), et chiffrer ainsi le trafic web. Ces certificats TLS/SSL peuvent être stockés dans Azure Key Vault et autoriser les déploiements sécurisés de certificats sur les machines virtuelles Windows dans Azure. Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer un Azure Key Vault
> * Générer ou télécharger un certificat dans Key Vault
> * Créer une machine virtuelle et installer le serveur web IIS
> * Injecter le certificat dans la machine virtuelle et configurer IIS à l’aide d’une liaison TLS


## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. 

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.


## <a name="overview"></a>Vue d’ensemble
Azure Key Vault protège les clés de chiffrement et les secrets, tels que les certificats ou les mots de passe. Key Vault rationalise le processus de gestion de certificats et vous permet de garder le contrôle sur les clés d’accès à ces certificats. Vous pouvez créer un certificat auto-signé à l’intérieur de Key Vault ou charger un certificat approuvé existant que vous avez déjà.

Au lieu d’utiliser une image de machine virtuelle personnalisée qui inclut des certificats intégrés, vous injectez des certificats dans une machine virtuelle en cours d’exécution. Ce processus garantit que les certificats les plus récents sont installés sur un serveur web pendant le déploiement. Si vous renouvelez ou remplacez un certificat, vous n’êtes pas non plus obligé de créer une image de machine virtuelle personnalisée. Les certificats les plus récents sont automatiquement injectés à la création des machines virtuelles supplémentaires. Pendant tout le processus, les certificats ne quittent jamais la plateforme Azure, ni ne sont exposés dans un script, un historique de ligne de commande ou un modèle.


## <a name="create-an-azure-key-vault"></a>Créer un Azure Key Vault
Avant de créer un coffre de clés et des certificats, créez un groupe de ressources avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). L’exemple suivant crée un groupe de ressources nommé *myResourceGroupSecureWeb* à l’emplacement *USA Est* :

```azurepowershell-interactive
$resourceGroup = "myResourceGroupSecureWeb"
$location = "East US"
New-AzResourceGroup -ResourceGroupName $resourceGroup -Location $location
```

Ensuite, créez un coffre de clés avec [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault). Chaque Key Vault requiert un nom unique en minuscules. Remplacez `mykeyvault` dans l’exemple suivant par le nom unique de votre propre Key Vault :

```azurepowershell-interactive
$keyvaultName="mykeyvault"
New-AzKeyVault -VaultName $keyvaultName `
    -ResourceGroup $resourceGroup `
    -Location $location `
    -EnabledForDeployment
```

## <a name="generate-a-certificate-and-store-in-key-vault"></a>Générer un certificat et le stocker dans Key Vault
Dans un environnement de production, vous devez importer un certificat valide, signé par un fournisseur approuvé, à l’aide de la commande [Import-AzKeyVaultCertificate](/powershell/module/az.keyvault/import-azkeyvaultcertificate). Pour ce didacticiel, l’exemple suivant vous montre comment générer un certificat auto-signé avec la commande [Add-AzKeyVaultCertificate](/powershell/module/az.keyvault/add-azkeyvaultcertificate) qui utilise la stratégie de certificat par défaut à partir de [New-AzKeyVaultCertificatePolicy](/powershell/module/az.keyvault/new-azkeyvaultcertificatepolicy). 

```azurepowershell-interactive
$policy = New-AzKeyVaultCertificatePolicy `
    -SubjectName "CN=www.contoso.com" `
    -SecretContentType "application/x-pkcs12" `
    -IssuerName Self `
    -ValidityInMonths 12

Add-AzKeyVaultCertificate `
    -VaultName $keyvaultName `
    -Name "mycert" `
    -CertificatePolicy $policy 
```


## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle
Définissez un nom d’utilisateur administrateur et un mot de passe pour la machine virtuelle avec [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1) :

```azurepowershell-interactive
$cred = Get-Credential
```

Vous pouvez maintenant créer la machine virtuelle avec [New-AzVM](/powershell/module/az.compute/new-azvm). L’exemple suivant permet de créer une machine virtuelle nommée *myVM* dans l’emplacement *EastUS*. Si elles n’existent pas déjà, les ressources réseau requises sont créées. Pour autoriser le trafic web sécurisé, le cmdlet ouvre également le port *443*.

```azurepowershell-interactive
# Create a VM
New-AzVm `
    -ResourceGroupName $resourceGroup `
    -Name "myVM" `
    -Location $location `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -Credential $cred `
    -OpenPorts 443

# Use the Custom Script Extension to install IIS
Set-AzVMExtension -ResourceGroupName $resourceGroup `
    -ExtensionName "IIS" `
    -VMName "myVM" `
    -Location $location `
    -Publisher "Microsoft.Compute" `
    -ExtensionType "CustomScriptExtension" `
    -TypeHandlerVersion 1.8 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server -IncludeManagementTools"}'
```

Il faut quelques minutes pour que la machine virtuelle soit créée. La dernière étape utilise l’extension de script personnalisé Azure pour installer le serveur web IIS avec [Set-AzVmExtension](/powershell/module/az.compute/set-azvmextension).


## <a name="add-a-certificate-to-vm-from-key-vault"></a>Ajouter un certificat à la machine virtuelle à partir de Key Vault
Pour ajouter le certificat à partir de Key Vault sur une machine virtuelle, obtenez l’ID du certificat avec [Get-AzKeyVaultSecret](/powershell/module/az.keyvault/get-azkeyvaultsecret). Ajoutez le certificat à la machine virtuelle avec [Add-AzVMSecret](/powershell/module/az.compute/add-azvmsecret) :

```azurepowershell-interactive
$certURL=(Get-AzKeyVaultSecret -VaultName $keyvaultName -Name "mycert").id

$vm=Get-AzVM -ResourceGroupName $resourceGroup -Name "myVM"
$vaultId=(Get-AzKeyVault -ResourceGroupName $resourceGroup -VaultName $keyVaultName).ResourceId
$vm = Add-AzVMSecret -VM $vm -SourceVaultId $vaultId -CertificateStore "My" -CertificateUrl $certURL

Update-AzVM -ResourceGroupName $resourceGroup -VM $vm
```


## <a name="configure-iis-to-use-the-certificate"></a>Configurer IIS pour utiliser le certificat
Réutilisez l’extension de script personnalisé avec [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension) pour mettre à jour la configuration d’IIS. Cette mise à jour applique le certificat injecté à partir de Key Vault dans IIS et configure la liaison web :

```azurepowershell-interactive
$PublicSettings = '{
    "fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/secure-iis.ps1"],
    "commandToExecute":"powershell -ExecutionPolicy Unrestricted -File secure-iis.ps1"
}'

Set-AzVMExtension -ResourceGroupName $resourceGroup `
    -ExtensionName "IIS" `
    -VMName "myVM" `
    -Location $location `
    -Publisher "Microsoft.Compute" `
    -ExtensionType "CustomScriptExtension" `
    -TypeHandlerVersion 1.8 `
    -SettingString $publicSettings
```


### <a name="test-the-secure-web-app"></a>Tester l’application web sécurisée
Obtenez l’adresse IP publique de votre machine virtuelle avec [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress). L’exemple suivant obtient l’adresse IP de `myPublicIP` créée précédemment :

```azurepowershell-interactive
Get-AzPublicIPAddress -ResourceGroupName $resourceGroup -Name "myPublicIPAddress" | select "IpAddress"
```

Vous pouvez maintenant ouvrir un navigateur web et entrer `https://<myPublicIP>` dans la barre d’adresse. Pour accepter l’avertissement de sécurité si vous avez utilisé un certificat auto-signé, sélectionnez **Détails**, puis **Atteindre la page web** :

![Accepter l’avertissement de sécurité du navigateur web](./media/tutorial-secure-web-server/browser-warning.png)

Votre site IIS sécurisé apparaît maintenant comme dans l’exemple suivant :

![Afficher le site IIS sécurisé en cours d’exécution](./media/tutorial-secure-web-server/secured-iis.png)


## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez sécurisé un serveur web IIS à l’aide d’un certificat TLS/SSL stocké dans Azure Key Vault. Vous avez appris à :

> [!div class="checklist"]
> * Créer un Azure Key Vault
> * Générer ou télécharger un certificat dans Key Vault
> * Créer une machine virtuelle et installer le serveur web IIS
> * Injecter le certificat dans la machine virtuelle et configurer IIS à l’aide d’une liaison TLS

Suivez ce lien pour consulter des exemples de scripts de machine virtuelle prédéfinis.

> [!div class="nextstepaction"]
> [Exemples de scripts de machine virtuelle Windows](./powershell-samples.md)
