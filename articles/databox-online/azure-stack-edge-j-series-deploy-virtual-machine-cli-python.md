---
title: Déployer des machines virtuelles sur votre appareil GPU Azure Stack Edge Pro par le biais d’Azure CLI et Python
description: Décrit comment créer et gérer des machines virtuelles sur un appareil GPU Azure Stack Edge Pro à l’aide d’Azure CLI et de Python.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 09/07/2020
ms.author: alkohli
ms.openlocfilehash: 8ea0c27fdd64bae1e6fe9443df76c86e0eb89a75
ms.sourcegitcommit: 799f0f187f96b45ae561923d002abad40e1eebd6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/24/2020
ms.locfileid: "97762914"
---
# <a name="deploy-vms-on-your-azure-stack-edge-pro-gpu-device-using-azure-cli-and-python"></a>Déployer des machines virtuelles sur votre appareil GPU Azure Stack Edge Pro à l’aide d’Azure CLI et Python

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

[!INCLUDE [azure-stack-edge-gateway-deploy-virtual-machine-overview](../../includes/azure-stack-edge-gateway-deploy-virtual-machine-overview.md)]

Ce tutoriel explique comment créer et gérer une machine virtuelle sur votre appareil Azure Stack Edge Pro à l’aide de l’interface de ligne de commande (CLI) Azure et de Python.

## <a name="vm-deployment-workflow"></a>Workflow du déploiement de machine virtuelle

Le workflow de déploiement est illustré dans le diagramme suivant.

![Workflow du déploiement de machine virtuelle](media/azure-stack-edge-gpu-deploy-virtual-machine-powershell/vm-workflow-r.svg)

Voici une synthèse globale du workflow de déploiement :

1. Se connecter à Azure Resource Manager
2. Créer un groupe de ressources
3. Créez un compte de stockage.
4. Ajouter l’URI d’objet blob au fichier hosts
5. Installer des certificats
6. Télécharger un disque dur virtuel
7. Créer des disques managés à partir du disque dur virtuel
8. Créer une image de machine virtuelle à partir de l’image de disque managé
9. Créer une machine virtuelle avec des ressources créées précédemment
10. Créer un réseau virtuel
11. Créer une carte réseau virtuelle à l’aide de l’ID de sous-réseau VNet

Pour obtenir une explication détaillée du diagramme de workflow, consultez [Déployer des machines virtuelles sur votre appareil Azure Stack Edge Pro à l’aide d’Azure PowerShell](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md). Pour plus d’informations sur la façon de se connecter à Azure Resource Manager, consultez [Se connecter à Azure Resource Manager à l’aide d’Azure PowerShell](azure-stack-edge-j-series-connect-resource-manager.md).

## <a name="prerequisites"></a>Prérequis

Avant de commencer à créer et à gérer une machine virtuelle sur votre appareil Azure Stack Edge Pro à l’aide d’Azure CLI et de Python, assurez-vous d’avoir effectué les étapes suivantes :

1. Vous avez appliqué les paramètres réseau sur votre appareil Azure Stack Edge Pro comme décrit dans [Étape 1 : Configurer l’appareil Azure Stack Edge Pro](azure-stack-edge-j-series-connect-resource-manager.md#step-1-configure-azure-stack-edge-pro-device).

2. Vous avez activé une interface réseau pour le calcul. Cette adresse IP d’interface réseau sert à créer un commutateur virtuel pour le déploiement de la machine virtuelle. Les étapes suivantes expliquent comment procéder :

    1. Accédez à **Calcul**. Sélectionnez l’interface réseau que vous utiliserez pour créer un commutateur virtuel.

        > [!IMPORTANT] 
        > Vous ne pouvez configurer qu’un seul port pour le calcul.

    2. Activez le calcul sur l’interface réseau. Azure Stack Edge Pro crée et gère un commutateur virtuel correspondant à cette interface réseau.

    <!--If you decide to use another network interface for compute, make sure that you:

    - Delete all the VMs that you have deployed using Azure Resource Manager.

    - Delete all virtual network interfaces and the virtual network associated with this network interface.

    - You can now enable another network interface for compute.-->

3. Vous avez créé et installé tous les certificats sur votre appareil Azure Stack Edge Pro et dans le magasin approuvé de votre client. Appliquez la procédure décrite dans [Étape 2 : Créer et installer des certificats](azure-stack-edge-j-series-connect-resource-manager.md#step-2-create-and-install-certificates).

4. Vous avez créé un certificat *.cer* encodé en base 64 pour votre appareil Azure Stack Edge Pro. Celui-ci est déjà chargé en tant que chaîne de signature sur l’appareil et installé dans le magasin racine approuvé sur votre client. Ce certificat est également requis au format *pem* pour que Python fonctionne sur ce client.

    Convertissez ce certificat au format pem à l’aide de la commande `certutil`. Vous devez exécuter cette commande dans le répertoire qui contient votre certificat.

    ```powershell
    certutil.exe <SourceCertificateName.cer> <DestinationCertificateName.pem>
    ```
    Voici un exemple illustrant l’utilisation de la commande :

    ```powershell
    PS C:\Certificates> certutil.exe -encode aze-root.cer aze-root.pem
    Input Length = 2150
    Output Length = 3014
    CertUtil: -encode command completed successfully.
    PS C:\Certificates>
    ```    
    Vous ajouterez également ce certificat pem à la boutique Python ultérieurement.

5. Vous avez affecté l’adresse IP de l’appareil dans votre page **Réseau** dans l’interface utilisateur web locale de l’appareil. Vous devez ajouter cette adresse IP :

    - Au fichier hosts sur le client, ou
    - À la configuration du serveur DNS.
    
    > [!IMPORTANT]
    > Nous vous recommandons de modifier la configuration du serveur DNS pour la résolution des noms de points de terminaison.

    1. Démarrez le **Bloc-notes** en tant qu’administrateur (des privilèges d’administrateur sont nécessaires pour enregistrer le fichier), puis ouvrez le fichier **hosts** qui se trouve dans `C:\Windows\System32\Drivers\etc`.
    
        ![Fichier hosts dans l’Explorateur Windows](media/azure-stack-edge-j-series-connect-resource-manager/hosts-file.png)
    
    2. Ajoutez les entrées suivantes à votre fichier **hosts**, en les remplaçant par les valeurs appropriées pour votre appareil :
    
        ```
        <Device IP> login.<appliance name>.<DNS domain>
        <Device IP> management.<appliance name>.<DNS domain>
        <Device IP> <storage name>.blob.<appliance name>.<DNS domain>
        ```
    3. Utilisez l’image suivante pour référence. Enregistrez le fichier **hosts**.

        ![Fichier hosts dans le Bloc-notes](media/azure-stack-edge-j-series-deploy-virtual-machine-cli-python/hosts-screenshot-boxed.png)

6. [Téléchargez le script Python](https://aka.ms/ase-vm-python) utilisé dans cette procédure.

## <a name="step-1-set-up-azure-clipython-on-the-client"></a>Étape 1 : Configurer Azure CLI/Python sur le client

### <a name="verify-profile-and-install-azure-cli"></a>Vérifier le profil et installer Azure CLI

<!--1. Verify the API profile of the client and identify which version of the modules and libraries to include on your client. In this example, the client system will be running Azure Stack 1904 or later. For more information, see [Azure Resource Manager API profiles](/azure-stack/user/azure-stack-version-profiles?view=azs-1908#azure-resource-manager-api-profiles).-->

1. Installez Azure CLI sur votre client. Dans cet exemple, Azure CLI 2.0.80 a été installé. Pour vérifier la version d’Azure CLI, exécutez la commande `az --version` .

    Voici un exemple de sortie de la commande ci-dessus :

    ```powershell
    PS C:\windows\system32> az --version
    azure-cli                         2.0.80
    
    command-modules-nspkg              2.0.3
    core                              2.0.80
    nspkg                              3.0.4
    telemetry                          1.0.4
    Extensions:
    azure-cli-iot-ext                  0.7.1
    
    Python location 'C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\python.exe'
    Extensions directory 'C:\.azure\cliextensions'
    
    Python (Windows) 3.6.6 (v3.6.6:4cf1f54eb7, Jun 27 2018, 02:47:15) [MSC v.1900 32 bit (Intel)]
    
    Legal docs and information: aka.ms/AzureCliLegal
    
    Your CLI is up-to-date.
    
    Please let us know how we are doing: https://aka.ms/clihats
    PS C:\windows\system32>
    ```

    Si vous n’avez pas Azure CLI, téléchargez et [installez Azure CLI sur Windows](/cli/azure/install-azure-cli-windows?view=azure-cli-latest). Vous pouvez exécuter Azure CLI à l’aide de l’invite de commandes Windows ou de Windows PowerShell.

2. Prenez note de l’emplacement Python de l’interface CLI. Vous en aurez besoin pour déterminer l’emplacement du magasin de certificats racines de confiance pour Azure CLI.

3. Pour exécuter l’exemple de script utilisé dans cet article, vous aurez besoin des versions de bibliothèques Python suivantes :

    ```powershell
    azure-common==1.1.23
    azure-mgmt-resource==2.1.0
    azure-mgmt-network==2.7.0
    azure-mgmt-compute==5.0.0
    azure-mgmt-storage==1.5.0
    azure-storage-blob==1.2.0rc1
    haikunator
    msrestazure==0.6.2
    ```
    Pour installer les versions, exécutez la commande suivante :

    ```powershell
    .\python.exe -m pip install haikunator
    ```

    L’exemple de sortie suivant montre l’installation de Haikunator :

    ```powershell
    PS C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2> .\python.exe -m pip install haikunator

    Collecting haikunator
      Downloading https://files.pythonhosted.org/packages/43/fa/130968f1a1bb1461c287b9ff35c630460801783243acda2cbf3a4c5964a5/haikunator-2.1.0-py2.py3-none-any.whl
    
    Installing collected packages: haikunator
    Successfully installed haikunator-2.1.0
    You are using pip version 10.0.1, however version 20.0.1 is available.
    You should consider upgrading using the 'python -m pip install --upgrade pip' command.
    
    PS C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2> 
    ```

    L’exemple de sortie suivant montre l’installation de pip pour `msrestazure` : 
    
    ```powershell
    PS C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2> .\python.exe -m pip install msrestazure==0.6.2
    Requirement already satisfied: msrestazure==0.6.2 in c:\program files (x86)\microsoft sdks\azure\cli2\lib\site-packages (0.6.2)
    Requirement already satisfied: msrest<2.0.0,>=0.6.0 in c:\program files (x86)\microsoft sdks\azure\cli2\lib\site-packages (from msrestazure==0.6.2) (0.6.10)
    === CUT ===========================  CUT ==================================
    Requirement already satisfied: cffi!=1.11.3,>=1.8 in c:\program files (x86)\microsoft sdks\azure\cli2\lib\site-packages (from cryptography>=1.1.0->adal<2.0.0,>=0.6.0->msrestazure==0.6.2) (1.13.2)
    Requirement already satisfied: pycparser in c:\program files (x86)\microsoft sdks\azure\cli2\lib\site-packages (from cffi!=1.11.3,>=1.8->cryptography>=1.1.0->adal<2.0.0,>=0.6.0->msrestazure==0.6.2) (2.18)
    You are using pip version 10.0.1, however version 20.0.1 is available.
    You should consider upgrading using the 'python -m pip install --upgrade pip' command.
    PS C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2>
    ```

### <a name="trust-the-azure-stack-edge-pro-ca-root-certificate"></a>Approuver le certificat racine d’autorité de certification Azure Stack Edge Pro

1. Trouvez l’emplacement du certificat sur votre machine. Cet emplacement peut varier en fonction de l’endroit où vous avez installé `az cli`. Exécutez Windows PowerShell en tant qu'administrateur. Basculez vers le chemin où `az cli` a installé Python : `C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\python.exe`.

    Pour obtenir l’emplacement du certificat, tapez la commande suivante :

    ```powershell
    .\python -c "import certifi; print(certifi.where())"
    ```
    
    L’applet de commande retourne l’emplacement du certificat, comme indiqué ci-dessous :  
        
    ```powershell
    PS C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2> .\python -c "import certifi; print(certifi.where())"
    C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\lib\site-packages\certifi\cacert.pem
    PS C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2>
    ```
      
    Prenez note de cet emplacement, car vous l’utiliserez plus tard - `C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\lib\site-packages\certifi\cacert.pem`

2. Pour approuver le certificat racine d’autorité de certification Azure Stack Edge Pro, ajoutez-le au certificat Python existant. Vous fournirez le chemin où vous avez enregistré le certificat PEM plus tôt.

    ```powershell
    $pemFile = "<Path to the pem format certificate>"
    ```
    Exemple de chemin : « C:\VM-scripts\rootteam3device.pem »
    
    Ensuite, tapez la série de commandes suivante dans Windows PowerShell :

    ```powershell
    $root = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
        $root.Import($pemFile)
        
    Write-Host "Extracting required information from the cert file"
    $md5Hash    = (Get-FileHash -Path $pemFile -Algorithm MD5).Hash.ToLower()
    $sha1Hash   = (Get-FileHash -Path $pemFile -Algorithm SHA1).Hash.ToLower()
    $sha256Hash = (Get-FileHash -Path $pemFile -Algorithm SHA256).Hash.ToLower()

    $issuerEntry  = [string]::Format("# Issuer: {0}", $root.Issuer)
    $subjectEntry = [string]::Format("# Subject: {0}", $root.Subject)
    $labelEntry   = [string]::Format("# Label: {0}", $root.Subject.Split('=')[-1])
    $serialEntry  = [string]::Format("# Serial: {0}", $root.GetSerialNumberString().ToLower())
    $md5Entry = [string]::Format("# MD5 Fingerprint: {0}", $md5Hash)
    $sha1Entry= [string]::Format("# SHA1 Fingerprint: {0}", $sha1Hash)
    $sha256Entry  = [string]::Format("# SHA256 Fingerprint: {0}", $sha256Hash)
    $certText = (Get-Content -Path $pemFile -Raw).ToString().Replace("`r`n","`n")
    
    $rootCertEntry = "`n" + $issuerEntry + "`n" + $subjectEntry + "`n" + $labelEntry + "`n" + `
    $serialEntry + "`n" + $md5Entry + "`n" + $sha1Entry + "`n" + $sha256Entry + "`n" + $certText
    
    Write-Host "Adding the certificate content to Python Cert store"
    Add-Content "${env:ProgramFiles(x86)}\Microsoft SDKs\Azure\CLI2\Lib\site-packages\certifi\cacert.pem" $rootCertEntry
    
    Write-Host "Python Cert store was updated to allow the Azure Stack Edge Pro CA root certificate"
    ```
    
### <a name="connect-to-azure-stack-edge-pro"></a>Se connecter à Azure Stack Edge Pro

1. Inscrivez votre environnement Azure Stack Edge Pro en exécutant la commande `az cloud register`.

    Dans certains scénarios, la connectivité Internet sortante directe est acheminée par l'intermédiaire d'un proxy ou d'un pare-feu, qui assure l'interception SSL. La commande az cloud register peut alors échouer avec une erreur telle que \"Impossible d’obtenir les points de terminaison du cloud\". Pour contourner cette erreur, définissez les variables d’environnement suivantes dans Windows PowerShell :

    ```powershell
    $ENV:AZURE_CLI_DISABLE_CONNECTION_VERIFICATION = 1 
    $ENV:ADAL_PYTHON_SSL_NO_VERIFY = 1
    ```

2. Définissez les variables d’environnement pour le script pour le point de terminaison Azure Resource Manager, l’emplacement où les ressources sont créées et le chemin du disque dur virtuel source. L’emplacement des ressources est fixe sur tous les appareils Azure Stack Edge Pro, et est défini sur `dbelocal`. Vous devez également spécifier les préfixes d’adresse et l’adresse IP privée. Toutes les variables d’environnement suivantes sont des valeurs basées sur vos valeurs, à l’exception de `AZURE_RESOURCE_LOCATION`, qui doit être codée en dur sur `"dbelocal"`.

    ```powershell
    $ENV:ARM_ENDPOINT = "https://management.team3device.teatraining1.com"
    $ENV:AZURE_RESOURCE_LOCATION = "dbelocal"
    $ENV:VHD_FILE_PATH = "C:\Downloads\Ubuntu1604\Ubuntu13.vhd"
    $ENV:ADDRESS_PREFIXES = "5.5.0.0/16"
    $ENV:PRIVATE_IP_ADDRESS = "5.5.174.126"
    ```

3. Inscrivez votre environnement. Utilisez les paramètres suivants lors de l’exécution de az cloud register :

    | Valeur | Description | Exemple |
    | --- | --- | --- |
    | Nom de l’environnement | Nom de l’environnement auquel vous essayez de vous connecter. | Indiquez un nom, par exemple `aze-environ`. |
    | Point de terminaison Resource Manager | Cette URL est `https://Management.<appliancename><dnsdomain>`. <br> Pour obtenir cette URL, accédez à la page **Appareils** de l’interface utilisateur web locale de votre appareil. |Par exemple : `https://management.team3device.teatraining1.com`.  |
    
    ```powershell
    az cloud register -n <environmentname> --endpoint-resource-manager "https://management.<appliance name>.<DNS domain>"
    ```
    L’exemple suivant illustre l’utilisation de la commande ci-dessus :
    
    ```powershell
    PS C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2> az cloud register -n az-new-env --endpoint-resource-manager "https://management.team3device.teatraining1.com"
    ```
    
    
4. Définissez l’environnement actif à l’aide des commandes suivantes :

    ```powershell
    az cloud set -n <EnvironmentName>
    ```
    L’exemple suivant illustre l’utilisation de la commande ci-dessus :

    ```powershell
    PS C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2> az cloud set -n az-new-env
    Switched active cloud to 'az-new-env'.
    Use 'az login' to log in to this cloud.
    Use 'az account set' to set the active subscription.
    PS C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2>
    ```

4. Connectez-vous à votre environnement Azure Stack Edge Pro avec la commande `az login`. Vous pouvez vous connecter à l’environnement Azure Stack Edge Pro en tant qu’utilisateur ou que [principal de service](../active-directory/develop/app-objects-and-service-principals.md).

   Pour vous connecter en tant qu’*utilisateur*, effectuez les étapes suivantes :

   Vous pouvez spécifier directement le nom d’utilisateur et le mot de passe dans la commande `az login`, ou vous authentifier avec un navigateur. Vous devez choisir cette dernière solution si l’authentification multifacteur est activée sur votre compte.

   L’exemple suivant illustre l’utilisation de `az login` :
    
    ```powershell
    PS C:\Certificates> az login -u EdgeARMuser
    ```
   Après l’utilisation de la commande login, vous êtes invité à entrer un mot de passe. Fournissez le mot de passe Azure Resource Manager.

   Voici un exemple de sortie pour une connexion réussie après avoir fourni le mot de passe :  
   
   ```powershell
   PS C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2> az login -u EdgeARMuser
   Password:
   [
        {
            "cloudName": "az-new-env",
            "id": "A4257FDE-B946-4E01-ADE7-674760B8D1A3",
            "isDefault": true,
            "name": "Default Provider Subscription",
            "state": "Enabled",
            "tenantId": "c0257de7-538f-415c-993a-1b87a031879d",
            "user": {
                "name": "EdgeArmUser@localhost",
                "type": "user"
            }
        }
   ]
   PS C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2>
   ```
   Notez les valeurs `id` et `tenantId`, car elles correspondent à votre ID d’abonnement Azure Resource Manager et à l’ID de locataire Azure Resource Manager, respectivement, et elles seront utilisées lors de l’étape suivante.
       
   Les variables d’environnement suivantes doivent être définies pour fonctionner en tant que *principal de service* :

   ```
   $ENV:ARM_TENANT_ID = "c0257de7-538f-415c-993a-1b87a031879d"
   $ENV:ARM_CLIENT_ID = "cbd868c5-7207-431f-8d16-1cb144b50971"
   $ENV:ARM_CLIENT_SECRET - "<Your Azure Resource Manager password>"
   $ENV:ARM_SUBSCRIPTION_ID = "A4257FDE-B946-4E01-ADE7-674760B8D1A3"
   ```

   Votre ID de client Azure Resource Manager est codé en dur. Votre ID de locataire Azure Resource Manager et votre ID d’abonnement Azure Resource Manager sont présents dans la sortie de la commande `az login` que vous avez exécutée précédemment. La clé secrète client Azure Resource Manager est le mot de passe Azure Resource Manager que vous avez défini.

   Pour plus d’informations, consultez [Mot de passe Azure Resource Manager](azure-stack-edge-j-series-set-azure-resource-manager-password.md).

5. Remplacez le profil par la version 2019-03-01-hybrid. Pour modifier la version du profil, exécutez la commande suivante :

    ```powershell
    az cloud update --profile 2019-03-01-hybrid
    ```

    L’exemple suivant illustre l’utilisation de `az cloud update` :

    ```powershell
    PS C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2> az cloud update --profile 2019-03-01-hybrid
    PS C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2>
    ```

## <a name="step-2-create-a-vm"></a>Étape 2 : Créer une machine virtuelle

Un script Python est fourni afin que vous puissiez créer une machine virtuelle. Selon que vous êtes connecté en tant qu’utilisateur ou défini en tant que principal de service, le script prend l’entrée en conséquence et crée une machine virtuelle.

1. Exécutez le script Python à partir du répertoire où Python est installé.

    `.\python.exe example_dbe_arguments_name_https.py cli`

2. Lorsque le script s’exécute, le chargement du disque dur virtuel prend de 20 à 30 minutes. Pour afficher la progression de l’opération de chargement, vous pouvez utiliser l’Explorateur Stockage Azure ou AzCopy.

    Voici un exemple de sortie d’une exécution réussie du script. Le script crée toutes les ressources au sein d’un groupe de ressources, utilise ces ressources pour créer une machine virtuelle, puis supprime le groupe de ressources, notamment toutes les ressources qu’il a créées.

    
    ```powershell
    PS C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2> .\python.exe example_dbe_arguments_name_https.py cli
    
    Create Resource Group
    Create a storage account
    Uploading to Azure Stack Storage as blob:
            ubuntu13.vhd
    
    Listing blobs...
            ubuntu13.vhd
    
    VM image resource id:
                /subscriptions/a4257fde-b946-4e01-ade7-674760b8d1a3/resourceGroups/azure-sample-group-virtual-machines118/providers/Microsoft.Compute/images/UbuntuImage
    
    Create Vnet
    Create Subnet
    Create NIC
    Creating Linux Virtual Machine
    Tag Virtual Machine
    Create (empty) managed Data Disk
    Get Virtual Machine by Name
    Attach Data Disk
    Detach Data Disk
    Deallocating the VM (to prepare for a disk resize)
    Update OS disk size
    Start VM
    Restart VM
    Stop VM
    
    List VMs in subscription
            VM: VmName118
    
    List VMs in resource group
            VM: VmName118
    
    Delete VM
    All example operations completed successfully!
    
    Delete Resource Group
    Deleted: azure-sample-group-virtual-machines118
    PS C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2>
    ``` 


## <a name="next-steps"></a>Étapes suivantes

[Commandes CLI az courantes pour les machines virtuelles Linux](../virtual-machines/linux/cli-manage.md)