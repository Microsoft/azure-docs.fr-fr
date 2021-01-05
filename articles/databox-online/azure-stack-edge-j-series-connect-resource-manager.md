---
title: Se connecter à Azure Resource Manager sur votre appareil Azure Stack Edge Pro avec GPU
description: Décrit comment se connecter à l’infrastructure Azure Resource Manager s’exécutant sur votre Azure Stack Edge Pro avec GPU à l’aide d’Azure PowerShell.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/28/2020
ms.author: alkohli
ms.openlocfilehash: 6bf0da8716233178889d47ec3d57e9b29bc2658f
ms.sourcegitcommit: 799f0f187f96b45ae561923d002abad40e1eebd6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/24/2020
ms.locfileid: "97763215"
---
# <a name="connect-to-azure-resource-manager-on-your-azure-stack-edge-pro-device"></a>Se connecter à Azure Resource Manager sur votre appareil Azure Stack Edge Pro

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

Azure Resource Manager fournit une couche de gestion qui vous permet de créer, mettre à jour et supprimer des ressources dans votre abonnement Azure. L’appareil Azure Stack Edge Pro prend en charge les mêmes API Azure Resource Manager pour créer, mettre à jour et supprimer des machines virtuelles dans un abonnement local. Cette prise en charge vous permet de gérer l’appareil d’une manière cohérente avec le cloud. 

Ce tutoriel explique comment se connecter aux API locales sur votre appareil Azure Stack Edge Pro via Azure Resource Manager à l’aide d’Azure PowerShell.

## <a name="about-azure-resource-manager"></a>À propos d’Azure Resource Manager

Azure Resource Manager fournit une couche de gestion cohérente pour appeler l’API d’appareil Azure Stack Edge Pro et effectuer des opérations telles que la création, la mise à jour et la suppression de machines virtuelles. L’architecture d’ure Resource Manager est détaillée dans le diagramme suivant.

![Diagramme pour Azure Resource Manager](media/azure-stack-edge-j-series-connect-resource-manager/edge-device-flow.svg)


## <a name="endpoints-on-azure-stack-edge-pro-device"></a>Point de terminaison sur l’appareil Azure Stack Edge Pro

Le tableau suivant récapitule les différents points de terminaison exposés sur votre appareil, les protocoles pris en charge et les ports pour accéder à ces points de terminaison. Tout au long de l’article, vous trouverez des références à ces points de terminaison.

| # | Point de terminaison | Protocoles pris en charge | Port utilisé | Utilisé pour |
| --- | --- | --- | --- | --- |
| 1. | Azure Resource Manager | https | 443 | Pour se connecter à Azure Resource Manager pour l’automatisation |
| 2. | Service d’émission de jeton de sécurité | https | 443 | Pour s’authentifier via des jetons d’accès et d’actualisation |
| 3. | Objet blob | https | 443 | Pour se connecter au service Stockage Blob via REST |


## <a name="connecting-to-azure-resource-manager-workflow"></a>Connexion au flux de travail Azure Resource Manager

Le processus de connexion aux API locales de l’appareil à l’aide d’Azure Resource Manager nécessite les étapes suivantes :

| N° de l’étape | Vous accomplirez cette tâche... | .. à cet emplacement. |
| --- | --- | --- |
| 1. | [Configurer votre appareil Azure Stack Edge Pro](#step-1-configure-azure-stack-edge-pro-device) | Interface utilisateur web locale |
| 2. | [Créer et installer des certificats](#step-2-create-and-install-certificates) | Client Windows/interface utilisateur web locale |
| 3. | [Examiner les conditions préalables et configurer en conséquence](#step-3-install-powershell-on-the-client) | Client Windows |
| 4. | [Configurer Azure PowerShell sur le client](#step-4-set-up-azure-powershell-on-the-client) | Client Windows |
| 5. | [Modifier le fichier hôte pour la résolution de nom de point de terminaison](#step-5-modify-host-file-for-endpoint-name-resolution) | Client Windows ou serveur DNS |
| 6. | [Vérifier que le nom du point de terminaison est résolu](#step-6-verify-endpoint-name-resolution-on-the-client) | Client Windows |
| 7. | [Utiliser des cmdlets Azure PowerShell pour vérifier la connexion à Azure Resource Manager](#step-7-set-azure-resource-manager-environment) | Client Windows |

Les sections suivantes décrivent chacune des étapes ci-dessus de connexion à Azure Resource Manager.

## <a name="prerequisites"></a>Conditions préalables

Avant de commencer, assurez-vous que le client utilisé pour se connecter à l’appareil via Azure Resource Manager utilise le protocole TLS 1.2. Pour plus d’informations, accédez à [Configurer le protocole TLS 1.2 sur le client Windows accédant à l’appareil Azure Stack Edge Pro](azure-stack-edge-j-series-configure-tls-settings.md).

## <a name="step-1-configure-azure-stack-edge-pro-device"></a>Étape 1 : Configurer l’appareil Azure Stack Edge Pro 

Suivez les étapes suivantes dans l’interface utilisateur web locale de votre appareil Azure Stack Edge Pro.

1. Définissez les paramètres réseau de votre appareil Azure Stack Edge Pro. 

    ![Page « Paramètres réseau » de l’interface utilisateur web locale](./media/azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy/compute-network-2.png)


    Prenez note de l’adresse IP de l’appareil. Vous l’utiliserez ultérieurement.

2. Configurez le nom de l’appareil et le domaine DNS à partir de la page **Appareil**. Notez le nom de l’appareil et le domaine DNS, car vous les utiliserez ultérieurement.

    ![Page « Appareil » de l’interface utilisateur web locale](./media/azure-stack-edge-gpu-deploy-set-up-device-update-time/device-2.png)

    > [!IMPORTANT]
    > Le nom de l’appareil et le domaine DNS seront utilisés pour former les points de terminaison exposés.
    > Utilisez les points de terminaison Azure Resource Manager et Blob de la page **Appareil** de l’interface utilisateur web locale.


## <a name="step-2-create-and-install-certificates"></a>Étape 2 : Créer et installer les certificats

Les certificats garantissent que votre communication est approuvée. Sur votre appareil Azure Stack Edge Pro, l’appliance auto-signée, le blob et les certificats Azure Resource Manager sont générés automatiquement. Si vous le souhaitez, vous pouvez également insérer votre propre blob signé ainsi que vos certificats Azure Resource Manager.

Lorsque vous apportez votre propre certificat signé, vous avez également besoin de la chaîne de signature correspondante du certificat. Pour la chaîne de signature, Azure Resource Manager et les certificats de blob sur l’appareil, vous aurez besoin des certificats correspondants sur l’ordinateur client pour vous authentifier et communiquer avec l’appareil.

Pour vous connecter à Azure Resource Manager, vous devez créer ou obtenir des certificats de chaîne de signature et de point de terminaison, importer ces certificats sur votre client Windows, puis télécharger ces certificats sur l’appareil.

### <a name="create-certificates-optional"></a>Créer des certificats (facultatif)

À des fins de développement ou de test uniquement, vous pouvez utiliser Windows PowerShell pour créer des certificats sur votre système local. Lors de la création des certificats pour le client, suivez les instructions suivantes :

1. Vous commencer par créer un certificat racine pour la chaîne de signature. Pour plus d’informations, consultez les étapes à suivre pour [Créer des certificats de chaîne de signature](azure-stack-edge-j-series-manage-certificates.md#create-signing-chain-certificate).

2. Vous pouvez ensuite créer les certificats de point de terminaison pour le blob et Azure Resource Manager. Vous pouvez accéder à ces points de terminaison à partir de la page **Appareil** dans l’interface utilisateur web locale. Consultez les étapes à suivre pour [Créer des certificats de point de terminaison](azure-stack-edge-j-series-manage-certificates.md#create-signed-endpoint-certificates).

3. Pour tous ces certificats, assurez-vous que le nom de l’objet et l’autre nom de l’objet sont conformes aux recommandations suivantes :

    |Type |Nom de l’objet (SN)  |Autre nom de l’objet (SAN)  |Exemple de nom d’objet |
    |---------|---------|---------|---------|
    |Azure Resource Manager|`management.<Device name>.<Dns Domain>`|`login.<Device name>.<Dns Domain>`<br>`management.<Device name>.<Dns Domain>`|`management.mydevice1.microsoftdatabox.com` |
    |Stockage d'objets blob|`*.blob.<Device name>.<Dns Domain>`|`*.blob.< Device name>.<Dns Domain>`|`*.blob.mydevice1.microsoftdatabox.com` |
    |Certificat unique à plusieurs SAN pour les deux points de terminaison|`<Device name>.<dnsdomain>`|`login.<Device name>.<Dns Domain>`<br>`management.<Device name>.<Dns Domain>`<br>`*.blob.<Device name>.<Dns Domain>`|`mydevice1.microsoftdatabox.com` |

Pour plus d’informations sur les certificats, consultez [Gérer des certificats](azure-stack-edge-j-series-manage-certificates.md).

### <a name="upload-certificates-on-the-device"></a>Charger des certificats sur l’appareil

Les certificats que vous avez créés à l’étape précédente se trouvent dans le magasin personnel sur votre client. Ces certificats doivent être exportés sur votre client dans des fichiers de format appropriés qui peuvent ensuite être chargés sur votre appareil.

1. Le certificat racine doit être exporté sous la forme d’un fichier de format DER avec l’extension *.cer*. Pour obtenir des instructions détaillées, consultez [Exporter des certificats au format .cer](azure-stack-edge-j-series-manage-certificates.md#export-certificates-as-der-format).

2. Les certificats de point de terminaison doivent être exportés en tant que fichiers *.pfx* avec des clés privées. Pour obtenir des instructions détaillées, consultez [Exporter des certificats en tant que fichiers .pfx avec des clés privées](azure-stack-edge-j-series-manage-certificates.md#export-certificates-as-pfx-format-with-private-key).

3. Les certificats racines et de point de terminaison sont ensuite chargés sur l’appareil à l’aide de l’option **+ Ajouter un certificat** dans la page **Certificats** de l’interface utilisateur web locale. Pour charger les certificats, suivez les étapes décrites dans la section [Charger des certificats](azure-stack-edge-j-series-manage-certificates.md#upload-certificates).


### <a name="import-certificates-on-the-client-running-azure-powershell"></a>Importer des certificats sur le client exécutant Azure PowerShell

Le client Windows dans lequel vous allez appeler les API Azure Resource Manager doit établir la confiance avec l’appareil. À cette fin, les certificats que vous avez créés à l’étape précédente doivent être importés sur votre client Windows dans le magasin de certificats approprié.

1. Le certificat racine que vous avez exporté au format DER avec l’extension *.cer* doit maintenant être importé dans les Autorités de certification racine approuvées sur votre système client. Pour obtenir des instructions détaillées, consultez [Importer des certificats dans le magasin Autorités de certification racine approuvées](azure-stack-edge-j-series-manage-certificates.md#import-certificates-as-der-format).

2. Les certificats de point de terminaison que vous avez exportés dans un fichier de format *.pfx* doivent être exportés dans un fichier de format *.cer*. Ce fichier *.cer* est ensuite importé dans le magasin de certificats **Personnel** sur votre système. Pour obtenir des instructions détaillées, consultez [Importer des certificats dans le magasin personnel](azure-stack-edge-j-series-manage-certificates.md#import-certificates-as-der-format).

## <a name="step-3-install-powershell-on-the-client"></a>Étape 3 : Installer PowerShell sur le client 

Votre client Windows doit remplir les conditions préalables suivantes :

1. Exécuter PowerShell Version 5.0. Vous devez disposer de PowerShell version 5.0 ou ultérieure. Pour vérifier la version de PowerShell installée sur votre système, exécutez la cmdlet suivante :

    ```powershell
    $PSVersionTable.PSVersion
    ```

    Comparez la version **Majeure** et assurez-vous qu’il s’agit de la version 5.0 ou d’une version ultérieure.

    Si votre version est obsolète, consultez [Mise à niveau des instances Windows PowerShell existantes](/powershell/scripting/install/installing-windows-powershell?view=powershell-6#upgrading-existing-windows-powershell).

    Si vous n\'avez pas PowerShell 5.0, suivez les instructions d’[Installation de PowerShell](/powershell/scripting/install/installing-windows-powershell?view=powershell-6).

    Voici un exemple de sortie obtenue.

    ```powershell
    Windows PowerShell
    Copyright (C) Microsoft Corporation. All rights reserved. 
    Try the new cross-platform PowerShell https://aka.ms/pscore6
    PS C:\windows\system32> $PSVersionTable.PSVersion
    Major  Minor  Build  Revision
    -----  -----  -----  --------
    5      1      18362  145
    ```
    
2. Vous pouvez accéder à la PowerShell Gallery.

    Démarrez PowerShell en tant qu'administrateur. Vérifiez que la PowerShell Gallery est inscrit en tant que référentiel.

    ```powershell
    Import-Module -Name PowerShellGet -ErrorAction Stop
    Import-Module -Name PackageManagement -ErrorAction Stop
    Get-PSRepository -Name "PSGallery"
    ```
    
    Voici un exemple de sortie obtenue.
    
    ```powershell
    PS C:\windows\system32> Import-Module -Name PowerShellGet -ErrorAction Stop
    PS C:\windows\system32> Import-Module -Name PackageManagement -ErrorAction Stop
    PS C:\windows\system32> Get-PSRepository -Name "PSGallery"
    Name                      InstallationPolicy   SourceLocation
    ----                      ------------------   --------------
    PSGallery                 Trusted              https://www.powershellgallery.com/api/v2
    ```
    
Si votre référentiel n’est pas approuvé ou si vous avez besoin d’informations supplémentaires, consultez [Vérifier l’accessibilité de PowerShell Gallery](/azure-stack/operator/azure-stack-powershell-install?view=azs-1908#2-validate-the-powershell-gallery-accessibility).

## <a name="step-4-set-up-azure-powershell-on-the-client"></a>Étape 4 : Configurer Azure PowerShell sur le client 

<!--1. Verify the API profile of the client and identify which version of the Azure PowerShell modules and libraries to include on your client. In this example, the client system will be running Azure Stack 1904 or later. For more information, see [Azure Resource Manager API profiles](/azure-stack/user/azure-stack-version-profiles?view=azs-1908#azure-resource-manager-api-profiles).-->

1. Vous allez installer sur votre client des modules Azure PowerShell qui fonctionneront avec votre appareil.

    a. Exécutez PowerShell ISE en tant qu’administrateur. Vous devez accéder à la PowerShell Gallery. 


    b. Pour installer les modules Azure PowerShell requis à partir de la PowerShell Gallery, exécutez la commande suivante :

    ```powershell
    # Install the AzureRM.BootStrapper module. Select Yes when prompted to install NuGet.
    
    Install-Module -Name AzureRM.BootStrapper
    
   # Install and import the API Version Profile into the current PowerShell session.
    
    Use-AzureRmProfile -Profile 2019-03-01-hybrid -Force
    
    # Confirm the installation of PowerShell
    Get-Module -Name "Azure*" -ListAvailable
    ```
    
    Assurez-vous que le module Azure-RM version 2.5.0 s’exécute à la fin de l’installation. 
    Si vous disposez d’une version existante du module Azure-RM qui ne correspond pas à la version requise, désinstallez-la à l’aide de la commande suivante :

    `Get-Module -Name Azure* -ListAvailable | Uninstall-Module -Force -Verbose`

    Vous devez à présent réinstaller la version requise.
   

Un exemple de sortie est illustré ci-dessous, indiquant que les modules AzureRM version 2.5.0 ont été installés avec succès.

```powershell
PS C:\windows\system32> Install-Module -Name AzureRM.BootStrapper
PS C:\windows\system32> Use-AzureRmProfile -Profile 2019-03-01-hybrid -Force
Loading Profile 2019-03-01-hybrid
PS C:\windows\system32> Get-Module -Name "Azure*" -ListAvailable
 
    Directory: C:\Program Files\WindowsPowerShell\Modules
 
ModuleType Version    Name                          ExportedCommands
---------- -------    ----                          ----------------
Script     4.5.0      Azure.Storage                       {Get-AzureStorageTable, New-AzureStorageTableSASToken, New...
Script     2.5.0      AzureRM
Script     0.5.0      AzureRM.BootStrapper                {Update-AzureRmProfile, Uninstall-AzureRmProfile, Install-...
Script     4.6.1      AzureRM.Compute                     {Remove-AzureRmAvailabilitySet, Get-AzureRmAvailabilitySet...
Script     3.5.1      AzureRM.Dns                         {Get-AzureRmDnsRecordSet, New-AzureRmDnsRecordConfig, Remo...
Script     5.1.5      AzureRM.Insights                    {Get-AzureRmMetricDefinition, Get-AzureRmMetric, Remove-Az...
Script     4.2.0      AzureRM.KeyVault                    {Add-AzureKeyVaultCertificate, Set-AzureKeyVaultCertificat...
Script     5.0.1      AzureRM.Network                     {Add-AzureRmApplicationGatewayAuthenticationCertificate, G...
Script     5.8.3      AzureRM.profile                     {Disable-AzureRmDataCollection, Disable-AzureRmContextAuto...
Script     6.4.3      AzureRM.Resources                   {Get-AzureRmProviderOperation, Remove-AzureRmRoleAssignmen...
Script     5.0.4      AzureRM.Storage                     {Get-AzureRmStorageAccount, Get-AzureRmStorageAccountKey, ...
Script     4.0.2      AzureRM.Tags                        {Remove-AzureRmTag, Get-AzureRmTag, New-AzureRmTag}
Script     4.0.3      AzureRM.UsageAggregates             Get-UsageAggregates
Script     5.0.1      AzureRM.Websites                    {Get-AzureRmAppServicePlan, Set-AzureRmAppServicePlan, New...

 
    Directory: C:\Program Files (x86)\Microsoft Azure Information Protection\Powershell
 
ModuleType Version    Name                            ExportedCommands
---------- -------    ----                           ----------------
Binary     1.48.204.0 AzureInformationProtection          {Clear-RMSAuthentication, Get-RMSFileStatus, Get-RMSServer...
```


## <a name="step-5-modify-host-file-for-endpoint-name-resolution"></a>Étape 5 : Modifier le fichier hosts pour la résolution de noms de point de terminaison 

Vous allez maintenant ajouter l’adresse IP virtuelle cohérente Azure que vous avez définie dans l’interface utilisateur web locale de l’appareil aux entités suivantes :

- fichier hosts sur le client, ou
- configuration du serveur DNS.

> [!IMPORTANT]
> Nous vous recommandons de modifier la configuration du serveur DNS pour la résolution des noms de point de terminaison.

Sur le client Windows que vous utilisez pour vous connecter à l’appareil, procédez comme suit :

1. Démarrez le **Bloc-notes** en tant qu’administrateur, puis ouvrez le fichier **hosts** qui se trouve dans C:\Windows\System32\Drivers\etc.

    ![Fichier hosts dans l’Explorateur Windows](media/azure-stack-edge-j-series-connect-resource-manager/hosts-file.png)

2. Ajoutez les entrées suivantes à votre fichier **hosts**, en les remplaçant par les valeurs appropriées pour votre appareil : 

    ```
    <Device IP> login.<appliance name>.<DNS domain>
    <Device IP> management.<appliance name>.<DNS domain>
    <Device IP> <storage name>.blob.<appliance name>.<DNS domain>
    ```

    > [!IMPORTANT]
    > L’entrée dans le fichier hosts doit correspondre exactement à celle fournie pour se connecter à Azure Resource Manager à une étape ultérieure. Assurez-vous que l’entrée Domaine DNS ici est entièrement en minuscules.

    Vous avez enregistré l’adresse IP de l’appareil à partir de l’interface utilisateur web locale lors d’une étape précédente.

    L’entrée login.\<appliance name\>.\<DNS domain\> entry est le point de terminaison du service d’émission de jeton de sécurité (STS). STS est responsable de la création, de la validation, du renouvellement et de l’annulation des jetons de sécurité. Le service de jeton de sécurité est utilisé pour créer le jeton d’accès et le jeton d’actualisation utilisés pour la communication continue entre l’appareil et le client.

3. Pour référence, utilisez l’image suivante. Enregistrez le fichier **hosts**.

    ![Fichier hosts dans le Bloc-notes](media/azure-stack-edge-j-series-connect-resource-manager/hosts-file-notepad.png)

## <a name="step-6-verify-endpoint-name-resolution-on-the-client"></a>Étape 6 : Vérifier la résolution de noms de point de terminaison sur le client

Vérifiez si le nom de point de terminaison est résolu sur le client que vous utilisez pour vous connecter à l’adresse IP virtuelle cohérente Azure.

1. Vous pouvez vous servir de l’utilitaire de ligne de commande ping. exe pour vérifier que le nom du point de terminaison est résolu. À partir d’une adresse IP donnée, la commande ping renvoie le nom d’hôte TCP/IP de l’ordinateur que vous suivez.

    Ajoutez le commutateur `-a` à la ligne de commande, comme dans l’exemple ci-dessous. Si le nom d’hôte peut être retourné, la commande le retourne également cette information potentiellement précieuse dans la réponse.

    ![Ping dans l’invite de commandes](media/azure-stack-edge-j-series-connect-resource-manager/ping-command-prompt.png)



## <a name="step-7-set-azure-resource-manager-environment"></a>Étape 7 : Définir l’environnement Azure Resource Manager

Définissez l’environnement Azure Resource Manager et vérifiez que la communication de votre appareil avec le client via Azure Resource Manager fonctionne correctement. Pour effectuer cette vérification, procédez comme suit :


1. Utilisez la cmdlet `Add-AzureRmEnvironment` pour vérifier que la communication via Azure Resource Manager fonctionne correctement et que les appels d’API transitent par le port dédié pour Azure Resource Manager (443).

    La cmdlet `Add-AzureRmEnvironment` ajoute des points de terminaison et des métadonnées pour permettre aux cmdlets d’Azure Resource Manager de se connecter à une nouvelle instance d’Azure Resource Manager. 


    > [!IMPORTANT]
    > L’URL de point de terminaison d’Azure Resource Manager que vous fournissez dans la cmdlet suivante respecte la casse. Assurez-vous que l’URL de point de terminaison est entièrement en minuscules et qu’elle correspond à ce que vous avez fourni dans le fichier hosts. Si la casse ne correspond pas, une erreur s’affiche.

    ```powershell
    Add-AzureRmEnvironment -Name <Environment Name> -ARMEndpoint "https://management.<appliance name>.<DNSDomain>/"
    ```

    Voici un exemple de sortie obtenue :
    
    ```powershell
    PS C:\windows\system32> Add-AzureRmEnvironment -Name AzDBE -ARMEndpoint https://management.dbe-n6hugc2ra.microsoftdatabox.com/
    
    Name  Resource Manager Url                    ActiveDirectory Authority
    ----  --------------------                   -------------------------
    AzDBE https://management.dbe-n6hugc2ra.microsoftdatabox.com https://login.dbe-n6hugc2ra.microsoftdatabox.com/adfs/
    ```

2. Pour l’environnement, définissez Azure Stack Edge Pro, et pour le port à utiliser pour les appels à Azure Resource Manager, définissez le port 443. Vous définissez l’environnement de deux manières :

    - Définissez l’environnement. Tapez la commande suivante :

    ```powershell
    Set-AzureRMEnvironment -Name <Environment Name>
    ```
    
    Pour plus d’informations, accédez à [Set-AzureRMEnvironment](/powershell/module/azurerm.profile/set-azurermenvironment?view=azurermps-6.13.0).

    - Définissez l’environnement inclus pour chaque cmdlet que vous exécutez. Cela permet de s’assurer que tous les appels d’API transitent par l’environnement approprié. Par défaut, les appels transitent par le cloud public Azure, mais vous souhaitez qu’ils transitent par l’environnement que vous avez défini pour l’appareil Azure Stack Edge Pro.

    - Pour plus d’informations sur la manière de changer d’environnement AzureRM, consultez [Changer d’environnement](#switch-environments).

2. Appelez les API de l’appareil local pour authentifier les connexions à Azure Resource Manager. 

    1. Ces informations d’identification sont destinées à un compte d’ordinateur local et sont uniquement utilisées pour l’accès d’API.

    2. Vous pouvez vous connecter via `login-AzureRMAccount` ou la commande `Connect-AzureRMAccount`. 

        1. Pour vous connecter, entrez la commande suivante. L’ID de locataire dans cette instance est codé en dur - c0257de7-538f-415c-993a-1b87a031879d. Utilisez le nom d’utilisateur et le mot de passe suivants.

            - **Nom d’utilisateur** - *EdgeArmUser*

            - **Mot de passe** - [Définissez le mot de passe pour Azure Resource Manager](azure-stack-edge-j-series-set-azure-resource-manager-password.md), puis utiliser ce mot de passe pour vous connecter. 

            ```powershell
            PS C:\windows\system32> $pass = ConvertTo-SecureString "<Your password>" -AsPlainText -Force;
            PS C:\windows\system32> $cred = New-Object System.Management.Automation.PSCredential("EdgeArmUser", $pass)
            PS C:\windows\system32> Connect-AzureRmAccount -EnvironmentName AzDBE -TenantId c0257de7-538f-415c-993a-1b87a031879d -credential $cred
            
            Account       SubscriptionName   TenantId            Environment
            -------       ----------------   --------            -----------
            EdgeArmUser@localhost Default Provider Subscription c0257de7-538f-415c-993a-1b87a031879d AzDBE
            
            PS C:\windows\system32>
            ```
                   
        
            Une autre façon de se connecter consiste à utiliser la cmdlet `login-AzureRmAccount`. 
            
            `login-AzureRMAccount -EnvironmentName <Environment Name>` -TenantId c0257de7-538f-415c-993a-1b87a031879d 

            Voici un exemple de sortie de la commande. 
         
            ```powershell
            PS C:\Users\Administrator> login-AzureRMAccount -EnvironmentName AzDBE -TenantId c0257de7-538f-415c-993a-1b87a031879d
            
            Account         SubscriptionName  TenantId              Environment
            -------         ----------------  --------              -------
            EdgeArmUser@localhost Default Provider Subscription c0257de7-538f-415c-993a-1b87a031879d AzDBE
            PS C:\Users\Administrator>
            ```



> [!IMPORTANT]
> La connexion à Azure Resource Manager expire toutes les 1,5 heure ou si votre appareil Azure Stack Edge Pro redémarre. Dans ce cas, toute cmdlet que vous exécutez retourne un message d’erreur indiquant que vous n’êtes plus connecté à Azure. Vous devez vous reconnecter.

## <a name="switch-environments"></a>Changer d’environnement

Exécutez la commande `Disconnect-AzureRmAccount` pour basculer vers un autre `AzureRmEnvironment`. 

Si vous utilisez `Set-AzureRmEnvironment` et `Login-AzureRmAccount` sans `Disconnect-AzureRmAccount`, l’environnement ne change pas.  

Les exemples suivants montrent comment basculer entre deux environnements, `AzDBE1` et `AzDBE2`.

Tout d’abord, répertoriez tous les environnements existants sur votre client.


```azurepowershell
PS C:\WINDOWS\system32> Get-AzureRmEnvironment
Name    Resource Manager Url     ActiveDirectory Authority
----    --------------------      -------------------------
AzureChinaCloud   https://management.chinacloudapi.cn/                 https://login.chinacloudapi.cn/
AzureCloud        https://management.azure.com/                        https://login.microsoftonline.com/
AzureGermanCloud  https://management.microsoftazure.de/                https://login.microsoftonline.de/
AzDBE1            https://management.HVTG1T2-Test.microsoftdatabox.com https://login.hvtg1t2-test.microsoftdatabox.com/adfs/
AzureUSGovernment https://management.usgovcloudapi.net/                https://login.microsoftonline.us/
AzDBE2            https://management.CVV4PX2-Test.microsoftdatabox.com https://login.cvv4px2-test.microsoftdatabox.com/adfs/
```

Ensuite, accédez à l’environnement auquel vous êtes actuellement connecté via votre Azure Resource Manager.

```azurepowershell
PS C:\WINDOWS\system32> Get-AzureRmContext |fl *

Name               : Default Provider Subscription (A4257FDE-B946-4E01-ADE7-674760B8D1A3) - EdgeArmUser@localhost
Account            : EdgeArmUser@localhost
Environment        : AzDBE2
Subscription       : a4257fde-b946-4e01-ade7-674760b8d1a3
Tenant             : c0257de7-538f-415c-993a-1b87a031879d
TokenCache         : Microsoft.Azure.Commands.Common.Authentication.ProtectedFileTokenCache
VersionProfile     :
ExtendedProperties : {}
```

Vous devez maintenant vous déconnecter de l’environnement actuel avant de basculer vers l’autre environnement.


```azurepowershell
PS C:\WINDOWS\system32> Disconnect-AzureRmAccount

Id                    : EdgeArmUser@localhost
Type                  : User
Tenants               : {c0257de7-538f-415c-993a-1b87a031879d}
AccessToken           :
Credential            :
TenantMap             : {}
CertificateThumbprint :
ExtendedProperties    : {[Subscriptions, A4257FDE-B946-4E01-ADE7-674760B8D1A3], [Tenants, c0257de7-538f-415c-993a-1b87a031879d]}
```

Connectez-vous à l’autre environnement. Voici un exemple de sortie obtenue.

```azurepowershell
PS C:\WINDOWS\system32> Login-AzureRmAccount -Environment "AzDBE1" -TenantId $ArmTenantId

Account     SubscriptionName   TenantId        Environment
-------     ----------------   --------        -----------
EdgeArmUser@localhost Default Provider Subscription c0257de7-538f-415c-993a-1b87a031879d AzDBE1
```

Exécutez cette cmdlet pour vérifier l’environnement auquel vous êtes connecté.

```azurepowershell
PS C:\WINDOWS\system32> Get-AzureRmContext |fl *

Name               : Default Provider Subscription (A4257FDE-B946-4E01-ADE7-674760B8D1A3) - EdgeArmUser@localhost
Account            : EdgeArmUser@localhost
Environment        : AzDBE1
Subscription       : a4257fde-b946-4e01-ade7-674760b8d1a3
Tenant             : c0257de7-538f-415c-993a-1b87a031879d
TokenCache         : Microsoft.Azure.Commands.Common.Authentication.ProtectedFileTokenCache
VersionProfile     :
ExtendedProperties : {}
```
Vous avez maintenant basculé vers l’environnement voulu.

## <a name="next-steps"></a>Étapes suivantes

[Déployer des machines virtuelles sur votre appareil Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md).