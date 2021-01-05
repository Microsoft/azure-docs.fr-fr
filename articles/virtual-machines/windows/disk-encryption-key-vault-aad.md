---
title: Création et configuration d’un coffre de clés pour Azure Disk Encryption avec Azure AD (version précédente)
description: Dans cet article, découvrez comment créer et configurer un coffre de clés pour Azure Disk Encryption avec Azure AD.
author: msmbaldwin
ms.service: virtual-machines-windows
ms.subservice: security
ms.topic: how-to
ms.author: mbaldwin
ms.date: 03/15/2019
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: d275a7ca97ac26257a83ad696df780669a00646e
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92737721"
---
# <a name="creating-and-configuring-a-key-vault-for-azure-disk-encryption-with-azure-ad-previous-release"></a>Création et configuration d’un coffre de clés pour Azure Disk Encryption avec Azure AD (version précédente)

**La nouvelle version d’Azure Disk Encryption vous évite de fournir un paramètre d’application Azure AD pour activer le chiffrement de disque de machine virtuelle. Avec la nouvelle version, vous n’êtes plus obligé de fournir les informations d’identification Azure AD lors de l’étape d’activation du chiffrement. Avec la nouvelle version, toutes les nouvelles machines virtuelles doivent être chiffrées sans les paramètres d’application Azure AD. Pour obtenir des instructions sur l’activation du chiffrement de disque de machine virtuelle avec la nouvelle version, consultez [Azure Disk Encryption](disk-encryption-overview.md). Les machines virtuelles déjà chiffrées avec les paramètres d’application Azure AD sont toujours prises en charge et doivent continuer à être gérées avec la syntaxe AAD.**

Azure Disk Encryption utilise Azure Key Vault pour contrôler et gérer les clés et les secrets de chiffrement de disque.  Pour plus d’informations sur les coffres de clés, consultez les articles [Prise en main du coffre de clés Azure](../../key-vault/general/overview.md) et [Sécuriser votre coffre de clés](../../key-vault/general/secure-your-key-vault.md). 

La création et la configuration d’un coffre de clés à utiliser avec Azure Disk Encryption avec Azure AD (version précédente) impliquent trois étapes :

1. Création d’un coffre de clés 
2. Configurez une application Azure AD et le principal de service.
3. Définissez la stratégie d’accès au coffre de clés pour l’application Azure AD.
4. Définissez les stratégies d’accès avancé au coffre de clés.
 
Si vous le souhaitez, vous pouvez également générer ou importer une clé de chiffrement principale (KEK, key encryption key).

Pour connaître les étapes à suivre pour [Installer les outils et se connecter à Azure](disk-encryption-key-vault.md#install-tools-and-connect-to-azure), consultez l’article principal [Création et configuration d’un coffre de clés pour Azure Disk Encryption](disk-encryption-key-vault.md).

> [!Note]
> Les étapes de cet article sont automatisées dans le [script d’interface CLI des prérequis Azure Disk Encryption](https://github.com/ejarvi/ade-cli-getting-started) et le [script PowerShell des prérequis Azure Disk Encryption](https://github.com/Azure/azure-powershell/tree/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts).


## <a name="create-a-key-vault"></a>Création d’un coffre de clés 
Azure Disk Encryption est intégré à [Azure Key Vault](https://azure.microsoft.com/documentation/services/key-vault/) pour vous permettre de contrôler et gérer les clés et les secrets de chiffrement de disque dans votre abonnement de coffre de clés. Vous pouvez créer un coffre de clés ou utiliser un coffre existant pour Azure Disk Encryption. Pour plus d’informations sur les coffres de clés, consultez les articles [Prise en main du coffre de clés Azure](../../key-vault/general/overview.md) et [Sécuriser votre coffre de clés](../../key-vault/general/secure-your-key-vault.md). Vous pouvez utiliser un modèle Resource Manager, Azure PowerShell ou Azure CLI pour créer un coffre de clés. 


>[!WARNING]
>Pour garantir que les secrets de chiffrement ne franchissent pas les limites régionales, Azure Disk Encryption exige que le coffre de clés se trouve dans la même région que les machines virtuelles. Créez et utilisez un coffre de clés situé dans la même région que la machine virtuelle à chiffrer. 


### <a name="create-a-key-vault-with-powershell"></a>Créer un coffre de clés avec PowerShell

Vous pouvez créer un coffre de clés avec Azure PowerShell à l’aide de l’applet de commande [New-AzKeyVault](/powershell/module/az.keyvault/New-azKeyVault). Pour découvrir les applets de commande supplémentaires pour Key Vault, consultez l’article [Az.KeyVault](/powershell/module/az.keyvault/). 

1. Le cas échéant, créez un groupe de ressources avec la commande [New-AzResourceGroup](/powershell/module/az.Resources/New-azResourceGroup).  Pour obtenir la liste des emplacements des centres de données, utilisez [Get-AzLocation](/powershell/module/az.resources/get-azlocation). 
     
     ```azurepowershell-interactive
     # Get-AzLocation 
     New-AzResourceGroup –Name 'MyKeyVaultResourceGroup' –Location 'East US'
     ```

1. Créez un coffre de clés avec [New-AzKeyVault](/powershell/module/az.keyvault/New-azKeyVault).
    
      ```azurepowershell-interactive
     New-AzKeyVault -VaultName 'MySecureVault' -ResourceGroupName 'MyKeyVaultResourceGroup' -Location 'East US'
     ```

4. Notez les données **Nom du coffre** , **Nom du groupe de ressources** , **ID de ressource** , **URI du coffre** et **ID d’objet** qui sont renvoyées à des fins d’utilisation ultérieure dans le cadre du chiffrement des données. 


### <a name="create-a-key-vault-with-azure-cli"></a>Créer un coffre de clés avec Azure CLI
Vous pouvez gérer votre coffre de clés avec Azure CLI à l’aide des commandes [az keyvault](/cli/azure/keyvault#commands). Pour créer un coffre de clés, utilisez la commande [az keyvault create](/cli/azure/keyvault#az-keyvault-create).

1. Le cas échéant, créez un groupe de ressources avec la commande [az group create](/cli/azure/group#az-group-create). Pour obtenir la liste des emplacements, utilisez la commande [az account list-locations](/cli/azure/account#az-account-list) 
     
     ```azurecli-interactive
     # To list locations: az account list-locations --output table
     az group create -n "MyKeyVaultResourceGroup" -l "East US"
     ```

3. Créez un coffre de clés avec la commande [az keyvault create](/cli/azure/keyvault#az-keyvault-create).
    
     ```azurecli-interactive
     az keyvault create --name "MySecureVault" --resource-group "MyKeyVaultResourceGroup" --location "East US"
     ```

4. Notez les données **Nom du coffre** , **Nom du groupe de ressources** , **ID de ressource** , **URI du coffre** et **ID d’objet** qui sont renvoyées à des fins d’utilisation ultérieure. 

### <a name="create-a-key-vault-with-a-resource-manager-template"></a>Créer un coffre de clés avec un modèle Resource Manager

Vous pouvez créer un coffre de clés à l’aide du [modèle Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-key-vault-create).

1. Dans le modèle de démarrage rapide Azure, cliquez sur **Déployer sur Azure** .
2. Sélectionnez l’abonnement, le groupe de ressources, l’emplacement du groupe de ressources, le nom du coffre de clés, l’ID d’objet, les conditions juridiques et le contrat, puis cliquez sur **Acheter** . 


## <a name="set-up-an-azure-ad-app-and-service-principal"></a>Configurer une application Azure AD et le principal de service 
Lorsque le chiffrement doit être activé sur une machine virtuelle en cours d’exécution dans Azure, Azure Disk Encryption génère et écrit les clés de chiffrement dans votre coffre de clés. La gestion des clés de chiffrement dans votre coffre de clés nécessite l’authentification Azure AD. Créez une application Azure AD à cet effet. Pour l’authentification, vous pouvez utiliser soit l’authentification par clé secrète client, soit [l’authentification Azure AD par certificat client](../../active-directory/authentication/active-directory-certificate-based-authentication-get-started.md).


### <a name="set-up-an-azure-ad-app-and-service-principal-with-azure-powershell"></a>Configurer une application Azure AD et un principal de service avec Azure PowerShell 
Pour exécuter les commandes suivantes, obtenez et utilisez le [module Azure AD PowerShell](/powershell/azure/active-directory/install-adv2). 

1. Pour créer une application Azure AD, utilisez la cmdlet PowerShell [New-AzADApplication](/powershell/module/az.resources/new-azadapplication). Vous pouvez définir MyApplicationHomePage et MyApplicationUri sur les valeurs de votre choix.

     ```azurepowershell
     $aadClientSecret = "My AAD client secret"
     $aadClientSecretSec = ConvertTo-SecureString -String $aadClientSecret -AsPlainText -Force
     $azureAdApplication = New-AzADApplication -DisplayName "My Application Display Name" -HomePage "https://MyApplicationHomePage" -IdentifierUris "https://MyApplicationUri" -Password $aadClientSecretSec
     $servicePrincipal = New-AzADServicePrincipal –ApplicationId $azureAdApplication.ApplicationId
     ```

3. $azureAdApplication.ApplicationId est l’ID de client Azure AD, et $aadClientSecret est la clé secrète client que vous utiliserez par la suite pour activer Azure Disk Encryption. Prenez soin de bien sauvegarder la clé secrète de client Azure AD. L’exécution de l’élément `$azureAdApplication.ApplicationId` vous présentera l’ID d’application.


### <a name="set-up-an-azure-ad-app-and-service-principal-with-azure-cli"></a>Configurer une application Azure AD et un principal de service avec Azure CLI

Vous pouvez gérer vos principaux de service avec Azure CLI à l’aide des commandes [az ad sp](/cli/azure/ad/sp). Pour plus d’informations, consultez l’article [Créer un principal du service Azure](/cli/azure/create-an-azure-service-principal-azure-cli).

1. Créez un principal de service.
     
     ```azurecli-interactive
     az ad sp create-for-rbac --name "ServicePrincipalName" --password "My-AAD-client-secret" --skip-assignment 
     ```
3.  L’élément appId renvoyé est l’ID de client Azure AD utilisé dans d’autres commandes. Il s’agit également du nom de principal de service (SPN) que vous utiliserez pour la commande az keyvault set-policy. Le mot de passe est la clé secrète client que vous utiliserez par la suite pour activer Azure Disk Encryption. Prenez soin de bien sauvegarder la clé secrète de client Azure AD.
 
### <a name="set-up-an-azure-ad-app-and-service-principal-though-the-azure-portal"></a>Configurer une application Azure AD et un principal de service avec le portail Azure
Pour créer une application Azure AD, exécutez la procédure de l’article [Utiliser le portail pour créer une application et un principal du service Azure Active Directory pouvant accéder aux ressources](../../active-directory/develop/howto-create-service-principal-portal.md). Chaque étape répertoriée ci-après vous permet d’accéder directement à la section de l’article appropriée. 

1. [Vérifiez les autorisations requises](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).
2. [Créez une application Azure Active Directory](../../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal). 
     - Lorsque vous créez l’application, vous pouvez utiliser le nom et l’URL de connexion de votre choix.
3. [Obtenez l’ID d’application et la clé d’authentification](../../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in). 
     - La clé d’authentification correspond à la clé secrète client et est utilisée comme élément AadClientSecret pour Set-AzVMDiskEncryptionExtension. 
        - La clé d’authentification est utilisée par l’application en tant qu’information d’identification pour la connexion à Azure AD. Dans le Portail Azure, ce secret est appelé clés, mais n’a aucun rapport avec les coffres de clés. Sécurisez ce secret de manière appropriée. 
     - L’ID d’application sera utilisé par la suite en tant qu’élément AadClientId pour Set-AzVMDiskEncryptionExtension et en tant qu’élément ServicePrincipalName pour Set-AzKeyVaultAccessPolicy. 

## <a name="set-the-key-vault-access-policy-for-the-azure-ad-app"></a>Définir la stratégie d’accès au coffre de clés pour l’application Azure AD
Pour écrire des secrets de chiffrement dans un coffre de clés spécifié, Azure Disk Encryption a besoin de l’ID client et de la clé secrète client de l’application Azure Active Directory qui dispose des autorisations pour écrire des secrets dans le coffre de clés. 

> [!NOTE]
> Azure Disk Encryption requiert de configurer les stratégies d’accès suivantes sur votre application cliente Azure AD : autorisations _WrapKey_ et _Set_ .

### <a name="set-the-key-vault-access-policy-for-the-azure-ad-app-with-azure-powershell"></a>Définir la stratégie d’accès au coffre de clés pour l’application Azure AD avec Azure PowerShell
Votre application Azure AD a besoin d’autorisations d’accès aux clés ou aux clés secrètes dans le coffre. Exécutez la cmdlet [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) pour accorder des autorisations à l’application, en utilisant l’ID client (qui a été généré quand l’application a été enregistrée) comme valeur du paramètre _–ServicePrincipalName_ . Pour en savoir plus, consultez le billet de blog [Azure Key Vault – Étape par étape](/archive/blogs/kv/azure-key-vault-step-by-step). 

1. Définissez la stratégie d’accès au coffre de clés pour l’application AD avec PowerShell.

     ```azurepowershell
     $keyVaultName = 'MySecureVault'
     $aadClientID = 'MyAadAppClientID'
     $KVRGname = 'MyKeyVaultResourceGroup'
     Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -ServicePrincipalName $aadClientID -PermissionsToKeys 'WrapKey' -PermissionsToSecrets 'Set' -ResourceGroupName $KVRGname
     ```

### <a name="set-the-key-vault-access-policy-for-the-azure-ad-app-with-azure-cli"></a>Définir la stratégie d’accès au coffre de clés pour l’application Azure AD avec Azure CLI
Utilisez la commande [az keyvault set-policy](/cli/azure/keyvault#az-keyvault-set-policy) pour définir la stratégie d’accès. Pour plus d’informations, consultez l’article [Gestion de Key Vault à l’aide de l’interface de ligne de commande (CLI) 2.0](../../key-vault/general/manage-with-cli2.md#authorizing-an-application-to-use-a-key-or-secret).

Fournissez le principal de service que vous avez créé par le biais de l’accès à Azure CLI pour obtenir les secrets et inclure les clés dans un wrapper avec la commande suivante :

```azurecli-interactive
az keyvault set-policy --name "MySecureVault" --spn "<spn created with CLI/the Azure AD ClientID>" --key-permissions wrapKey --secret-permissions set
```

### <a name="set-the-key-vault-access-policy-for-the-azure-ad-app-with-the-portal"></a>Définir la stratégie d’accès au coffre de clés pour l’application Azure AD avec le portail

1. Ouvrez le groupe de ressources avec votre coffre de clés.
2. Sélectionnez votre coffre de clés, accédez à **Stratégies d’accès** , puis cliquez sur **Ajouter nouveau** .
3. Sous **Sélectionner le principal** , recherchez l’application Azure AD que vous avez créée, puis sélectionnez-la. 
4. Pour **Autorisations de clé** , cochez la case en regard de **Inclure la clé** sous **Opérations de chiffrement** .
5. Pour **Autorisations du secret** , cochez la case en regard de **Définir** sous **Opérations de gestion des secrets** .
6. Cliquez sur **OK** pour enregistrer la stratégie d’accès. 

![Opérations de chiffrement Azure Key Vault - Inclure la clé](../media/disk-encryption/keyvault-portal-fig3.png)

![Autorisations du secret Azure Key Vault - Définir](../media/disk-encryption/keyvault-portal-fig3b.png)

## <a name="set-key-vault-advanced-access-policies"></a>Définir des stratégies d’accès avancé au coffre de clés
La plateforme Azure doit avoir accès aux clés et aux clés secrètes de chiffrement dans votre coffre de clés afin de les mettre à disposition de la machine virtuelle pour lancer et déchiffrer les volumes. Activez le chiffrement de disque sur le coffre de clés ; dans le cas contraire, les déploiements échoueront.  

### <a name="set-key-vault-advanced-access-policies-with-azure-powershell"></a>Définir des stratégies d’accès avancé au coffre de clés avec Azure PowerShell
 Utilisez l’applet de commande PowerShell de coffre de clés [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) pour activer le chiffrement de disque pour le coffre de clés.

  - **Activer Key Vault pour le chiffrement de disque :** EnabledForDiskEncryption est requis pour Azure Disk Encryption.
      
     ```azurepowershell-interactive 
     Set-AzKeyVaultAccessPolicy -VaultName 'MySecureVault' -ResourceGroupName 'MyKeyVaultResourceGroup' -EnabledForDiskEncryption
     ```

  - **Activer Key Vault pour le déploiement, si nécessaire :** autorise le fournisseur de ressources Microsoft.Compute à récupérer des secrets à partir de ce coffre de clés lorsque ce dernier est référencé dans le cadre de la création de ressources, par exemple lors de la création d’une machine virtuelle.

     ```azurepowershell-interactive
      Set-AzKeyVaultAccessPolicy -VaultName 'MySecureVault' -ResourceGroupName 'MyKeyVaultResourceGroup' -EnabledForDeployment
     ```

  - **Activer Key Vault pour le déploiement d’un modèle, si nécessaire :** autorise Azure Resource Manager à obtenir des secrets à partir de ce coffre de clés lorsque ce dernier est référencé dans un déploiement de modèle.

     ```azurepowershell-interactive             
     Set-AzKeyVaultAccessPolicy -VaultName 'MySecureVault' -ResourceGroupName 'MyKeyVaultResourceGroup' -EnabledForTemplateDeployment
     ```

### <a name="set-key-vault-advanced-access-policies-using-the-azure-cli"></a>Définir des stratégies d’accès avancé au coffre de clés avec Azure CLI
Utilisez la commande [az keyvault update](/cli/azure/keyvault#az-keyvault-update) afin d’activer le chiffrement de disque pour le coffre de clés. 

 - **Activer Key Vault pour le chiffrement de disque :** Enabled-for-disk-encryption est requis. 

     ```azurecli-interactive
     az keyvault update --name "MySecureVault" --resource-group "MyKeyVaultResourceGroup" --enabled-for-disk-encryption "true"
     ```  

 - **Activer Key Vault pour le déploiement, si nécessaire :** autorise les machines virtuelles à récupérer des certificats stockés en tant que secrets dans le coffre.
     ```azurecli-interactive
     az keyvault update --name "MySecureVault" --resource-group "MyKeyVaultResourceGroup" --enabled-for-deployment "true"
     ``` 

 - **Activer Key Vault pour le déploiement d’un modèle, si nécessaire :** autoriser Resource Manager à récupérer des secrets dans le coffre.
     ```azurecli-interactive  
     az keyvault update --name "MySecureVault" --resource-group "MyKeyVaultResourceGroup" --enabled-for-template-deployment "true"
     ```


### <a name="set-key-vault-advanced-access-policies-through-the-azure-portal"></a>Définir des stratégies d’accès avancé au coffre de clés avec le portail Azure

1. Sélectionnez votre coffre de clés, accédez à **Stratégies d’accès** , puis sélectionnez **Cliquez ici pour afficher les stratégies d’accès avancé** .
2. Cochez la case **Activer l’accès à Azure Disk Encryption pour chiffrer des volumes** .
3. Sélectionnez **Activer l’accès aux machines virtuelles Azure pour le déploiement** et/ou **Activer l’accès à Azure Resource Manager pour le déploiement de modèles** si nécessaire. 
4. Cliquez sur **Enregistrer** .

![Stratégies d’accès avancé au coffre de clés Azure](../media/disk-encryption/keyvault-portal-fig4.png)


## <a name="set-up-a-key-encryption-key-optional"></a>Configurer une clé de chiffrement à clé (facultatif)
Si vous souhaitez utiliser une clé de chiffrement à clé pour renforcer la protection des clés de chiffrement, ajoutez une clé de chiffrement à clé à votre coffre de clés. Utilisez la cmdlet [Add-AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey) pour créer une clé de chiffrement à clé dans le coffre de clés. Vous pouvez également importer une clé de chiffrement à clé à partir de votre module de sécurité matériel de gestion des clés locales. Pour plus d’informations, consultez la [documentation concernant Key Vault](../../key-vault/keys/hsm-protected-keys.md). Quand une clé de chiffrement principale est spécifiée, Azure Disk Encryption utilise cette clé pour wrapper les secrets de chiffrement avant d’écrire dans Key Vault. 

* Lors de la génération de clés, utilisez un type de clé RSA. Azure Disk Encryption ne prend pas encore en charge l’utilisation de clés à courbe elliptique.

* Les URL de clé secrète de coffre de clés et de clé de chiffrement à clé (KEK) doivent être des versions gérées. Azure met en vigueur cette restriction de gestion de version. Voici des exemples d’URL de clé secrète et de clé de chiffrement à clé valides :

  * Exemple d’URL de secret valide : *https://contosovault.vault.azure.net/secrets/EncryptionSecretWithKek/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx*
  * Exemple d’URL KEK valide : *https://contosovault.vault.azure.net/keys/diskencryptionkek/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx*

* Azure Disk Encryption ne prend pas en charge l’intégration de numéros de port aux secrets de coffre de clés et aux URL KEK. Voici des exemples d’URL de coffre de clés valides et non valides :

  * URL de coffre de clés inacceptable : *https://contosovault.vault.azure.net:443/secrets/contososecret/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx*
  * URL de coffre de clés acceptable : *https://contosovault.vault.azure.net/secrets/contososecret/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx*

### <a name="set-up-a-key-encryption-key-with-azure-powershell"></a>Configurer une clé de chiffrement principale avec Azure PowerShell 
Avant d’utiliser le script PowerShell, vous devez connaître les prérequis pour Azure Disk Encryption afin de comprendre les étapes figurant dans le script. L’exemple de script peut nécessiter certaines modifications pour votre environnement. Ce script crée tous les prérequis pour Azure Disk Encryption et chiffre une machine virtuelle IaaS existante en incluant la clé de chiffrement de disque dans un wrapper à l’aide d’une clé de chiffrement à clé. 

 ```powershell
 # Step 1: Create a new resource group and key vault in the same location.
     # Fill in 'MyLocation', 'MyKeyVaultResourceGroup', and 'MySecureVault' with your values.
     # Use Get-AzLocation to get available locations and use the DisplayName.
     # To use an existing resource group, comment out the line for New-AzResourceGroup
     
     $Loc = 'MyLocation';
     $KVRGname = 'MyKeyVaultResourceGroup';
     $KeyVaultName = 'MySecureVault'; 
     New-AzResourceGroup –Name  $KVRGname –Location $Loc;
     New-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname -Location $Loc;
     $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName  $KVRGname;
     $KeyVaultResourceId = (Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName  $KVRGname).ResourceId;
     $diskEncryptionKeyVaultUrl = (Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName  $KVRGname).VaultUri;
     
 # Step 2: Create the AD application and service principal.
     # Fill in 'MyAADClientSecret', "<My Application Display Name>", "<https://MyApplicationHomePage>", and "<https://MyApplicationUri>" with your values.
     # MyApplicationHomePage and the MyApplicationUri can be any values you wish.
     
     $aadClientSecret =  'MyAADClientSecret';
     $aadClientSecretSec = ConvertTo-SecureString -String $aadClientSecret -AsPlainText -Force;
     $azureAdApplication = New-AzADApplication -DisplayName "<My Application Display Name>" -HomePage "<https://MyApplicationHomePage>" -IdentifierUris "<https://MyApplicationUri>" -Password $aadClientSecretSec
     $servicePrincipal = New-AzADServicePrincipal –ApplicationId $azureAdApplication.ApplicationId;
     $aadClientID = $azureAdApplication.ApplicationId;
     
 #Step 3: Enable the vault for disk encryption and set the access policy for the Azure AD application.
     
     Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $KVRGname -EnabledForDiskEncryption;
     Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -ServicePrincipalName $aadClientID -PermissionsToKeys 'WrapKey' -PermissionsToSecrets 'Set' -ResourceGroupName  $KVRGname;
     
 #Step 4: Create a new key in the key vault with the Add-AzKeyVaultKey cmdlet.
     # Fill in 'MyKeyEncryptionKey' with your value.
     
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     Add-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName -Destination 'Software';
     $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;
     
 #Step 5: Encrypt the disks of an existing IaaS VM
     # Fill in 'MySecureVM' and 'MyVirtualMachineResourceGroup' with your values. 
     
     $VMName = 'MySecureVM';
      $VMRGName = 'MyVirtualMachineResourceGroup';
     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;
```

## <a name="certificate-based-authentication-optional"></a>Authentification par certificat (facultatif)
Si vous souhaitez utiliser l’authentification par certificat, vous pouvez charger un certificat dans votre coffre de clés et le déployer sur le client. Avant d’utiliser le script PowerShell, vous devez connaître les prérequis pour Azure Disk Encryption afin de comprendre les étapes figurant dans le script. L’exemple de script peut nécessiter certaines modifications pour votre environnement.

     
 ```powershell

 # Fill in "MyKeyVaultResourceGroup", "MySecureVault", and 'MyLocation' ('My location' only if needed)

   $KVRGname = 'MyKeyVaultResourceGroup'
   $KeyVaultName= 'MySecureVault'

   # Create a key vault and set enabledForDiskEncryption property on it. 
   # Comment out the next three lines if you already have an existing key vault enabled for encryption. No need to set 'My location' in this case.

   $Loc = 'MyLocation'
   New-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname -Location $Loc
   Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $KVRGname -EnabledForDiskEncryption

   #Setting some variables with the key vault information 
   $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname
   $DiskEncryptionKeyVaultUrl = $KeyVault.VaultUri
   $KeyVaultResourceId = $KeyVault.ResourceId

   # Create the Azure AD application and associate the certificate with it. 
   # Fill in "C:\certificates\mycert.pfx", "Password", "<My Application Display Name>", "<https://MyApplicationHomePage>", and "<https://MyApplicationUri>" with your values.
   # MyApplicationHomePage and the MyApplicationUri can be any values you wish

   $CertPath = "C:\certificates\mycert.pfx"
   $CertPassword = "Password"
   $Cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($CertPath, $CertPassword)
   $CertValue = [System.Convert]::ToBase64String($cert.GetRawCertData())

   $AzureAdApplication = New-AzADApplication -DisplayName "<My Application Display Name>" -HomePage "<https://MyApplicationHomePage>" -IdentifierUris "<https://MyApplicationUri>" -CertValue $CertValue 
   $ServicePrincipal = New-AzADServicePrincipal -ApplicationId $AzureAdApplication.ApplicationId

   $AADClientID = $AzureAdApplication.ApplicationId
   $aadClientCertThumbprint= $cert.Thumbprint

   Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -ServicePrincipalName $aadClientID -PermissionsToKeys 'WrapKey' -PermissionsToSecrets 'Set' -ResourceGroupName $KVRGname
   
   # Upload the pfx file to the key vault. 
   # Fill in "MyAADCert".  

   $KeyVaultSecretName = "MyAADCert"
   $FileContentBytes = get-content $CertPath -Encoding Byte
   $FileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)
           $JSONObject = @"
           { 
               "data" : "$filecontentencoded", 
               "dataType" : "pfx", 
               "password" : "$CertPassword" 
           } 
"@

   $JSONObjectBytes = [System.Text.Encoding]::UTF8.GetBytes($jsonObject)
   $JSONEncoded = [System.Convert]::ToBase64String($jsonObjectBytes)

   #Set the secret and set the key vault policy for -EnabledForDeployment

   $Secret = ConvertTo-SecureString -String $JSONEncoded -AsPlainText -Force
   Set-AzKeyVaultSecret -VaultName $KeyVaultName -Name $KeyVaultSecretName -SecretValue $Secret
   Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $KVRGname -EnabledForDeployment

   # Deploy the certificate to the VM
   # Fill in 'MySecureVM' and 'MyVirtualMachineResourceGroup' with your values.

   $VMName = 'MySecureVM'
   $VMRGName = 'MyVirtualMachineResourceGroup'
   $CertUrl = (Get-AzKeyVaultSecret -VaultName $KeyVaultName -Name $KeyVaultSecretName).Id
   $SourceVaultId = (Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGName).ResourceId
   $VM = Get-AzVM -ResourceGroupName $VMRGName -Name $VMName 
   $VM = Add-AzVMSecret -VM $VM -SourceVaultId $SourceVaultId -CertificateStore "My" -CertificateUrl $CertUrl
   Update-AzVM -VM $VM -ResourceGroupName $VMRGName 

   #Enable encryption on the VM using Azure AD client ID and the client certificate thumbprint

   Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $VMName -AadClientID $AADClientID -AadClientCertThumbprint $AADClientCertThumbprint -DiskEncryptionKeyVaultUrl $DiskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId
 ```

## <a name="certificate-based-authentication-and-a-kek-optional"></a>Authentification par certificat et clé KEK (facultatif)

Si vous souhaitez utiliser l’authentification par certificat et inclure la clé de chiffrement dans un wrapper avec une clé de chiffrement à clé, vous pouvez utiliser le script ci-après en guise d’exemple. Avant d’utiliser le script PowerShell, vous devez connaître tous les prérequis précédents pour Azure Disk Encryption afin de comprendre les étapes figurant dans le script. L’exemple de script peut nécessiter certaines modifications pour votre environnement.

     
 ```powershell
# Fill in 'MyKeyVaultResourceGroup', 'MySecureVault', and 'MyLocation' (if needed)

   $KVRGname = 'MyKeyVaultResourceGroup'
   $KeyVaultName= 'MySecureVault'

   # Create a key vault and set enabledForDiskEncryption property on it. 
   # Comment out the next three lines if you already have an existing key vault enabled for encryption.

   $Loc = 'MyLocation'
   New-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname -Location $Loc
   Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $KVRGname -EnabledForDiskEncryption

   # Create the Azure AD application and associate the certificate with it.  
   # Fill in "C:\certificates\mycert.pfx", "Password", "<My Application Display Name>", "<https://MyApplicationHomePage>", and "<https://MyApplicationUri>" with your values.
   # MyApplicationHomePage and the MyApplicationUri can be any values you wish

   $CertPath = "C:\certificates\mycert.pfx"
   $CertPassword = "Password"
   $Cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($CertPath, $CertPassword)
   $CertValue = [System.Convert]::ToBase64String($cert.GetRawCertData())

   $AzureAdApplication = New-AzADApplication -DisplayName "<My Application Display Name>" -HomePage "<https://MyApplicationHomePage>" -IdentifierUris "<https://MyApplicationUri>" -CertValue $CertValue 
   $ServicePrincipal = New-AzADServicePrincipal -ApplicationId $AzureAdApplication.ApplicationId

   $AADClientID = $AzureAdApplication.ApplicationId
   $aadClientCertThumbprint= $cert.Thumbprint

   ## Give access for setting secrets and wraping keys
   Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -ServicePrincipalName $aadClientID -PermissionsToKeys 'WrapKey' -PermissionsToSecrets 'Set' -ResourceGroupName $KVRGname

   # Upload the pfx file to the key vault. 
   # Fill in "MyAADCert". 

   $KeyVaultSecretName = "MyAADCert"
   $FileContentBytes = get-content $CertPath -Encoding Byte
   $FileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)
           $JSONObject = @"
           { 
               "data" : "$filecontentencoded", 
               "dataType" : "pfx", 
               "password" : "$CertPassword" 
           } 
"@

   $JSONObjectBytes = [System.Text.Encoding]::UTF8.GetBytes($jsonObject)
   $JSONEncoded = [System.Convert]::ToBase64String($jsonObjectBytes)

   #Set the secret and set the key vault policy for deployment

   $Secret = ConvertTo-SecureString -String $JSONEncoded -AsPlainText -Force
   Set-AzKeyVaultSecret -VaultName $KeyVaultName -Name $KeyVaultSecretName -SecretValue $Secret
   Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $KVRGname -EnabledForDeployment

   #Setting some variables with the key vault information and generating a KEK 
   # FIll in 'KEKName'
   
   $KEKName ='KEKName'
   $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname
   $DiskEncryptionKeyVaultUrl = $KeyVault.VaultUri
   $KeyVaultResourceId = $KeyVault.ResourceId
   $KEK = Add-AzKeyVaultKey -VaultName $KeyVaultName -Name $KEKName -Destination "Software"
   $KeyEncryptionKeyUrl = $KEK.Key.kid



   # Deploy the certificate to the VM
   # Fill in 'MySecureVM' and 'MyVirtualMachineResourceGroup' with your values.

   $VMName = 'MySecureVM';
   $VMRGName = 'MyVirtualMachineResourceGroup';
   $CertUrl = (Get-AzKeyVaultSecret -VaultName $KeyVaultName -Name $KeyVaultSecretName).Id
   $SourceVaultId = (Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGName).ResourceId
   $VM = Get-AzVM -ResourceGroupName $VMRGName -Name $VMName 
   $VM = Add-AzVMSecret -VM $VM -SourceVaultId $SourceVaultId -CertificateStore "My" -CertificateUrl $CertUrl
   Update-AzVM -VM $VM -ResourceGroupName $VMRGName

   #Enable encryption on the VM using Azure AD client ID and the client certificate thumbprint

   Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $VMName -AadClientID $AADClientID -AadClientCertThumbprint $AADClientCertThumbprint -DiskEncryptionKeyVaultUrl $DiskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId
```

 
## <a name="next-steps"></a>Étapes suivantes

[Activer Azure Disk Encryption avec Azure AD sur des machines virtuelles Windows (version précédente)](disk-encryption-windows-aad.md)
