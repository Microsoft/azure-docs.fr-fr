---
title: Activer Azure Automation State Configuration
description: Cet article explique comment configurer des machines pour la gestion avec Azure Automation State Configuration.
services: automation
ms.service: automation
ms.subservice: dsc
author: mgoedtel
ms.author: magoedte
ms.topic: conceptual
ms.date: 12/10/2019
manager: carmonm
ms.openlocfilehash: ae268534a18a921cca012881fa172261c7ba1063
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86186399"
---
# <a name="enable-azure-automation-state-configuration"></a>Activer Azure Automation State Configuration

Cet rubrique décrit comment vous pouvez configurer vos machines pour la gestion avec Azure Automation State Configuration. Pour plus d’informations sur ce service, consultez [Vue d’ensemble d’Azure Automation State Configuration](automation-dsc-overview.md).

## <a name="enable-azure-vms"></a>Activer les machines virtuelles Azure

Azure Automation State Configuration vous permet d’activer facilement des machines virtuelles Azure pour une gestion de configuration par le biais du portail Azure, des modèles Azure Resource Manager ou de PowerShell. En arrière-plan, et sans qu’aucun administrateur n’ait à contrôler une machine virtuelle à distance, l’extension Azure VM Desired State Configuration inscrit la machine virtuelle auprès d’Azure Automation State Configuration. Étant donné que l’extension Azure s’exécute de façon asynchrone, les étapes pour suivre sa progression sont décrites dans [Vérifier l’état de configuration de machine virtuelle](#check-status-of-vm-setup).

> [!NOTE]
>Le déploiement de DSC sur un nœud Linux utilise le dossier **/tmp**. Les modules tels que `nxautomation` sont téléchargés temporairement pour vérification avant d’être installés à leur emplacement approprié. Pour garantir que les modules seront installés correctement, l’agent Log Analytics pour Linux a besoin d’autorisations en lecture/écriture pour le dossier **/tmp**.<br><br>
>L’agent Log Analytics pour Linux est exécuté avec le compte de l’utilisateur `omsagent`. Pour accorder l’autorisation >write à l’utilisateur `omsagent`, exécutez la commande `setfacl -m u:omsagent:rwx /tmp`.

### <a name="enable-a-vm-using-azure-portal"></a>Activer une machine virtuelle à l’aide du portail Azure

Pour activer une machine virtuelle Azure pour State Configuration via le [portail Azure](https://portal.azure.com/) :

1. Accédez au compte Azure Automation dans lequel activer des machines virtuelles. 

2. Dans la page State Configuration, sélectionnez l’onglet **Nœuds**, puis cliquez sur **Ajouter**.

3. Choisissez une machine virtuelle à activer.

4. Si l’extension d’état PowerShell souhaitée n’est pas installée sur la machine virtuelle et que l’état d’alimentation est en cours d’exécution, cliquez sur **Se connecter**.

5. Sous **Inscription**, entrez les [valeurs du gestionnaire de configuration locale de PowerShell DSC](/powershell/scripting/dsc/managing-nodes/metaConfig) requises pour votre cas d’utilisation. Éventuellement, vous pouvez entrer une configuration de nœud à attribuer à la machine virtuelle.

![activation de machine virtuelle](./media/automation-dsc-onboarding/DSC_Onboarding_6.png)

### <a name="enable-a-vm-using-azure-resource-manager-templates"></a>Activer une machine virtuelle à l’aide de modèles Azure Resource Manager

Vous pouvez installer et activer une machine virtuelle pour State Configuration à l’aide de modèles Azure Resource Manager. Pour un exemple de modèle activant une machine virtuelle existante pour State Configuration, consultez [Serveur géré par le service Desired State Configuration](https://azure.microsoft.com/resources/templates/101-automation-configuration/). Si vous gérez un groupe de machines virtuelles identiques, consultez l’exemple de modèle dans [Configuration des groupes de machines virtuelles identiques gérés par Azure Automation](https://azure.microsoft.com/resources/templates/201-vmss-automation-dsc/).

### <a name="enable-machines-using-powershell"></a>Activer des machines à l’aide de PowerShell

Vous pouvez utiliser la cmdlet [Register-AzAutomationDscNode](/powershell/module/az.automation/register-azautomationdscnode) dans PowerShell pour activer des machines virtuelles pour State Configuration. 

> [!NOTE]
>Le cmdlet `Register-AzAutomationDscNode` est actuellement implémenté uniquement pour les ordinateurs exécutant Windows, car il déclenche uniquement l’extension Windows.

### <a name="register-vms-across-azure-subscriptions"></a>Inscrire des machines virtuelles dans des abonnements Azure

La meilleure façon d’enregistrer des machines virtuelles à partir d’autres abonnements Azure consiste à utiliser l’extension DSC dans un modèle de déploiement Azure Resource Manager. Vous trouverez des exemples dans la section relative à [l’extension de Desired State Configuration avec des modèles Azure Resource Manager](../virtual-machines/extensions/dsc-template.md).

Pour trouver la clé et l’URL d’inscription à utiliser comme paramètres dans ce modèle, consultez [Activer des machines en toute sécurité à l’aide d’une inscription](#enable-machines-securely-using-registration).

## <a name="enable-physicalvirtual-windows-machines"></a>Activer des machines physiques/virtuelles Windows

Vous pouvez activer des serveurs Windows exécutés localement ou dans d’autres environnements cloud, notamment des instances AWS EC2, pour Azure Automation State Configuration. Les serveurs doivent disposer d’un [accès sortant à Azure](automation-dsc-overview.md#network-planning).

1. Vérifiez que la dernière version de [WMF 5](https://aka.ms/wmf5latest) est installée sur les machines à activer pour State Configuration. WMF 5 doit également être installé sur l’ordinateur que vous utilisez pour l’activation des machines.
1. Pour créer un dossier contenant les métaconfigurations DSC requises, suivez les instructions fournies dans [Générer des métaconfigurations DSC](#generate-dsc-metaconfigurations). 
1. Utilisez la cmdlet suivante pour appliquer les métaconfigurations DSC PowerShell à distance aux machines à activer. 

   ```powershell
   Set-DscLocalConfigurationManager -Path C:\Users\joe\Desktop\DscMetaConfigs -ComputerName MyServer1, MyServer2
   ```

1. Si vous ne pouvez pas appliquer les métaconfigurations DSC PowerShell à distance, copiez le dossier **metaconfigurations** vers les machines à activer. Ajoutez ensuite du code pour appeler [Set-DscLocalConfigurationManager](/powershell/module/psdesiredstateconfiguration/set-dsclocalconfigurationmanager?view=powershell-5.1) localement sur les machines.
1. À l’aide du portail Azure ou de cmdlets, vérifiez que les machines apparaissent bien comme des nœuds State Configuration inscrits dans votre compte Azure Automation.

## <a name="enable-physicalvirtual-linux-machines"></a>Activer des machines physiques/virtuelles Linux

Vous pouvez activer des serveurs Linux s’exécutant localement ou dans d’autres environnements cloud pour State Configuration. Les serveurs doivent disposer d’un [accès sortant à Azure](automation-dsc-overview.md#network-planning).

1. Vérifiez que la dernière version de [PowerShell Desired State Configuration pour Linux](https://github.com/Microsoft/PowerShell-DSC-for-Linux) est installée sur les machines à activer pour State Configuration.
2. Si les [valeurs par défaut du gestionnaire de configuration local DSC PowerShell](/powershell/scripting/dsc/managing-nodes/metaConfig4) correspondent à votre cas d’utilisation et que vous voulez activer des machines afin qu’elles échangent avec State Configuration :

   - Sur chaque machine Linux à activer, utilisez `Register.py` pour activer la machine avec les valeurs par défaut du gestionnaire de configuration local DSC PowerShell.

     `/opt/microsoft/dsc/Scripts/Register.py <Automation account registration key> <Automation account registration URL>`

   - Pour trouver la clé et l’URL d’inscription de votre compte Automation, consultez [Activer des machines en toute sécurité à l’aide d’une inscription](#enable-machines-securely-using-registration).

3. Si les valeurs par défaut du gestionnaire de configuration local DSC PowerShell ne correspondent pas à votre cas d’utilisation et que vous voulez activer des machines qui ne font que rendre compte à Azure Automation State Configuration, suivez les étapes 4 à 7. Sinon, passez directement à l’étape 7.

4. Pour produire un dossier contenant les méconfigurations DSC requises, suivez les instructions fournies dans [Générer des métaconfigurations DSC](#generate-dsc-metaconfigurations).

5. Assurez-vous que la dernière version de [WMF 5](https://aka.ms/wmf5latest) est installée sur l’ordinateur utilisé pour activer vos machines pour State Configuration.

6. Ajoutez le code suivant pour appliquer les métaconfigurations DSC PowerShell à distance aux machines à activer.

    ```powershell
    $SecurePass = ConvertTo-SecureString -String '<root password>' -AsPlainText -Force
    $Cred = New-Object System.Management.Automation.PSCredential 'root', $SecurePass
    $Opt = New-CimSessionOption -UseSsl -SkipCACheck -SkipCNCheck -SkipRevocationCheck

    # need a CimSession for each Linux machine to onboard
    $Session = New-CimSession -Credential $Cred -ComputerName <your Linux machine> -Port 5986 -Authentication basic -SessionOption $Opt

    Set-DscLocalConfigurationManager -CimSession $Session -Path C:\Users\joe\Desktop\DscMetaConfigs
    ```

7. Si vous ne pouvez pas appliquer à distance les métaconfigurations PowerShell DSC, copiez les métaconfigurations correspondant aux ordinateurs distants à partir du dossier décrit dans l’étape 4 sur les ordinateurs Linux.

8. Ajoutez du code pour appeler `Set-DscLocalConfigurationManager.py` localement sur chaque machine Linux à activer pour State Configuration.

   `/opt/microsoft/dsc/Scripts/SetDscLocalConfigurationManager.py -configurationmof <path to metaconfiguration file>`

9. À l’aide du portail Azure ou de cmdlets, vérifiez que les machines à activer apparaissent bien en tant que nœuds DSC inscrits dans votre compte Azure Automation.

## <a name="generate-dsc-metaconfigurations"></a>Générer des métaconfigurations DSC

Pour activer une machine pour State Configuration, vous pouvez générer une [métaconfiguration DSC](/powershell/scripting/dsc/managing-nodes/metaConfig). Cette configuration indique à l’agent DSC d’effectuer une extraction et/ou d’envoyer des rapports à Azure Automation State Configuration. Vous pouvez générer une métaconfiguration DSC pour Azure Automation State Configuration à l’aide d’une configuration PowerShell DSC ou des cmdlets PowerShell Azure Automation.

> [!NOTE]
> Les métaconfigurations DSC contiennent les clés secrètes nécessaires à l’activation d’une machine dans un compte Automation à des fins de gestion. Veillez à protéger convenablement les métaconfigurations DSC que vous créez ou supprimez-les après utilisation.

La prise en charge de proxy pour les métaconfigurations est contrôlée par le [Gestionnaire de configuration local](/powershell/scripting/dsc/managing-nodes/metaconfig?view=powershell-7), qui est le moteur DSC Windows PowerShell. Le gestionnaire de configuration local s’exécute sur tous les nœuds cibles et il est chargé d’appeler les ressources de configuration contenues dans un script de métaconfiguration DSC. Vous pouvez inclure la prise en charge du proxy dans une métaconfiguration en incluant les définitions des propriétés `ProxyURL` et `ProxyCredential` nécessaires dans les blocs `ConfigurationRepositoryWeb`, `ResourceRepositoryWeb` et `ReportServerWeb`. `ProxyURL = "http://172.16.3.6:3128";` est un exemple de paramètre d’URL. La propriété `ProxyCredential` est définie sur un objet `PSCredential`, comme décrit dans [Gérer les informations d’identification dans Azure Automation](shared-resources/credentials.md). 

### <a name="generate-dsc-metaconfigurations-using-a-dsc-configuration"></a>Générer des métaconfigurations DSC à l’aide d’une configuration DSC

1. Ouvrez VSCode (ou votre éditeur favori) en tant qu’administrateur sur une machine de votre environnement local. Cette machine doit disposer de la dernière version de [WMF 5](https://aka.ms/wmf5latest) .
1. Copiez le script suivant localement. Ce script contient une configuration DSC PowerShell pour créer des métaconfigurations, ainsi qu’une commande pour lancer la création de métaconfigurations.

    > [!NOTE]
    > Les noms de configuration de nœud State Configuration respectent la casse dans le Portail Azure. S’ils ne correspondent pas, le nœud ne s’affiche pas sous l’onglet **Nœuds**.

   ```powershell
   # The DSC configuration that will generate metaconfigurations
   [DscLocalConfigurationManager()]
   Configuration DscMetaConfigs
   {
        param
        (
            [Parameter(Mandatory=$True)]
            [String]$RegistrationUrl,

            [Parameter(Mandatory=$True)]
            [String]$RegistrationKey,

            [Parameter(Mandatory=$True)]
            [String[]]$ComputerName,

            [Int]$RefreshFrequencyMins = 30,

            [Int]$ConfigurationModeFrequencyMins = 15,

            [String]$ConfigurationMode = 'ApplyAndMonitor',

            [String]$NodeConfigurationName,

            [Boolean]$RebootNodeIfNeeded= $False,

            [String]$ActionAfterReboot = 'ContinueConfiguration',

            [Boolean]$AllowModuleOverwrite = $False,

            [Boolean]$ReportOnly
        )

        if(!$NodeConfigurationName -or $NodeConfigurationName -eq '')
        {
            $ConfigurationNames = $null
        }
        else
        {
            $ConfigurationNames = @($NodeConfigurationName)
        }

        if($ReportOnly)
        {
            $RefreshMode = 'PUSH'
        }
        else
        {
            $RefreshMode = 'PULL'
        }

        Node $ComputerName
        {
            Settings
            {
                RefreshFrequencyMins           = $RefreshFrequencyMins
                RefreshMode                    = $RefreshMode
                ConfigurationMode              = $ConfigurationMode
                AllowModuleOverwrite           = $AllowModuleOverwrite
                RebootNodeIfNeeded             = $RebootNodeIfNeeded
                ActionAfterReboot              = $ActionAfterReboot
                ConfigurationModeFrequencyMins = $ConfigurationModeFrequencyMins
            }

            if(!$ReportOnly)
            {
            ConfigurationRepositoryWeb AzureAutomationStateConfiguration
                {
                    ServerUrl          = $RegistrationUrl
                    RegistrationKey    = $RegistrationKey
                    ConfigurationNames = $ConfigurationNames
                }

                ResourceRepositoryWeb AzureAutomationStateConfiguration
                {
                    ServerUrl       = $RegistrationUrl
                    RegistrationKey = $RegistrationKey
                }
            }

            ReportServerWeb AzureAutomationStateConfiguration
            {
                ServerUrl       = $RegistrationUrl
                RegistrationKey = $RegistrationKey
            }
        }
   }

    # Create the metaconfigurations
    # NOTE: DSC Node Configuration names are case sensitive in the portal.
    # TODO: edit the below as needed for your use case
   $Params = @{
        RegistrationUrl = '<fill me in>';
        RegistrationKey = '<fill me in>';
        ComputerName = @('<some VM to onboard>', '<some other VM to onboard>');
        NodeConfigurationName = 'SimpleConfig.webserver';
        RefreshFrequencyMins = 30;
        ConfigurationModeFrequencyMins = 15;
        RebootNodeIfNeeded = $False;
        AllowModuleOverwrite = $False;
        ConfigurationMode = 'ApplyAndMonitor';
        ActionAfterReboot = 'ContinueConfiguration';
        ReportOnly = $False;  # Set to $True to have machines only report to AA DSC but not pull from it
   }

   # Use PowerShell splatting to pass parameters to the DSC configuration being invoked
   # For more info about splatting, run: Get-Help -Name about_Splatting
   DscMetaConfigs @Params
   ```

1. Entrez la clé et l’URL d’inscription de votre compte Automation, ainsi que les noms des machines à activer. Tous les autres paramètres sont facultatifs. Pour trouver la clé et l’URL d’inscription de votre compte Automation, consultez [Activer des machines en toute sécurité à l’aide d’une inscription](#enable-machines-securely-using-registration).

1. Si vous voulez que les machines adressent les informations d’état DSC à Azure Automation State Configuration sans toutefois extraire la configuration ou des modules PowerShell, affectez au paramètre `ReportOnly` la valeur true.

1. Si `ReportOnly` n’est pas défini, les machines signalent les informations d’état DSC à Azure Automation State Configuration et à la configuration de l’extraction ou aux modules PowerShell. Définissez les paramètres en conséquence dans les blocs `ConfigurationRepositoryWeb`, `ResourceRepositoryWeb`et `ReportServerWeb`.

1. Exécutez le script. Vous devez à présent avoir un dossier de répertoire de travail appelé **DscMetaConfigs**, contenant les métaconfigurations DSC PowerShell pour les machines à activer (en tant qu’administrateur).

    ```powershell
    Set-DscLocalConfigurationManager -Path ./DscMetaConfigs
    ```

### <a name="generate-dsc-metaconfigurations-using-azure-automation-cmdlets"></a>Générer des métaconfigurations DSC à l’aide des cmdlets Azure Automation

Si les valeurs par défaut du gestionnaire de configuration local PowerShell DSC correspondent à votre cas d’utilisation et que vous souhaitez activer des machines pour qu’elles échangent avec Azure Automation State Configuration, vous pouvez générer les métaconfigurations DSC nécessaires plus simplement en utilisant les cmdlets Azure Automation.

1. Ouvrez la console PowerShell ou VSCode en tant qu’administrateur sur une machine de votre environnement local.
2. Connectez-vous à Azure Resource Manager en utilisant [Connect-AzAccount](/powershell/module/Az.Accounts/Connect-AzAccount?view=azps-3.7.0).
3. Téléchargez les métaconfigurations DSC PowerShell pour les machines à activer à partir du compte Automation dans lequel vous configurez des nœuds.

   ```powershell
   # Define the parameters for Get-AzAutomationDscOnboardingMetaconfig using PowerShell Splatting
   $Params = @{
       ResourceGroupName = 'ContosoResources'; # The name of the Resource Group that contains your Azure Automation account
       AutomationAccountName = 'ContosoAutomation'; # The name of the Azure Automation account where you want a node on-boarded to
       ComputerName = @('web01', 'web02', 'sql01'); # The names of the computers that the metaconfiguration will be generated for
       OutputFolder = "$env:UserProfile\Desktop\";
   }
   # Use PowerShell splatting to pass parameters to the Azure Automation cmdlet being invoked
   # For more info about splatting, run: Get-Help -Name about_Splatting
   Get-AzAutomationDscOnboardingMetaconfig @Params
   ```

1. Vous devez à présent avoir un dossier **DscMetaConfigs** contenant les métaconfigurations DSC PowerShell pour les machines à activer (en tant qu’administrateur).

    ```powershell
    Set-DscLocalConfigurationManager -Path $env:UserProfile\Desktop\DscMetaConfigs
    ```

## <a name="enable-machines-securely-using-registration"></a>Activer des machines en toute sécurité à l’aide d’une inscription

Vous pouvez activer des machines en toute sécurité pour un compte Azure Automation par le biais du protocole d’inscription DSC WMF 5. Ce protocole permet à un nœud DSC de s’authentifier auprès d’un serveur de rapports ou d’extraction DSC PowerShell, y compris Azure Automation State Configuration. Le nœud s’enregistre auprès du serveur au niveau de l’URL d’inscription et s’authentifie à l’aide d’une clé d’inscription. Pendant l’enregistrement, le nœud DSC et le serveur d’extraction/rapports DSC négocient un certificat unique pour le nœud qui devra être utilisé pour l’authentification au serveur après l’enregistrement. Ce processus empêche les nœuds activés d’emprunter l’identité d’un autre nœud, par exemple, si un nœud est compromis et se comporte de façon malveillante. Après l’enregistrement, la clé d’enregistrement n’est plus utilisée pour l’authentification et est supprimée du nœud.

Pour obtenir les informations requises pour le protocole d’inscription State Configuration, accédez à **Clés** sous **Paramètres du compte** dans le portail Azure. 

![Clés et URL d’Azure Automation](./media/automation-dsc-onboarding/DSC_Onboarding_4.png)

- L’URL d’enregistrement correspond à la valeur du champ URL dans la page Clés.
- La clé d’inscription est la valeur du champ **Clé d’accès principale** ou du champ **Clé d’accès secondaire** de la page Clés. Vous pouvez utiliser l’une de ces deux clés.

Pour renforcer la sécurité, vous pouvez générer à nouveau les clés d’accès primaire et secondaire d’un compte Automation à tout moment dans la page Clés. La régénération de clé empêche les inscriptions de nœuds futures d’utiliser les clés précédentes.

## <a name="re-register-a-node"></a>Réinscrire un nœud

Après avoir inscrit une machine en tant que nœud DSC dans Azure Automation State Configuration, vous êtes susceptible de réinscrire ce nœud à l’avenir pour plusieurs raisons.

- **Renouvellement du certificat.** Pour les versions de Windows Server antérieures à Windows Server 2019, chaque nœud négocie automatiquement un certificat unique d’authentification qui expire après un an. Si un certificat expire sans renouvellement, le nœud ne peut pas communiquer avec Azure Automation et est marqué comme `Unresponsive`. À ce stade, le protocole d’inscription DSC PowerShell ne peut pas renouveler automatiquement les certificats lorsqu’ils sont sur le point d’expirer. Vous devez réinscrire les nœuds après un an. Avant la réinscription, assurez-vous que chaque nœud exécute WMF5 RTM. 

    La réinscription effectuée dans un délai de 90 jours ou moins à partir de la date d’expiration du certificat, ou à tout moment après cette date, entraîne la génération et l’utilisation d’un nouveau certificat. Une solution à ce problème est incluse dans Windows Server 2019 et ultérieur.

- **Modifications apportées aux valeurs du gestionnaire de configuration local DSC.** Vous devrez peut-être modifier les [valeurs du gestionnaire de configuration local PowerShell DSC](/powershell/scripting/dsc/managing-nodes/metaConfig4) définies lors de l’inscription initiale du nœud, par exemple, `ConfigurationMode`. Actuellement, vous ne pouvez modifier ces valeurs d’agent DSC que par le biais d’une réinscription. La seule exception est la valeur de configuration de nœud attribuée au nœud. Vous pouvez modifier cela dans Azure Automation DSC directement.

Vous pouvez réinscrire un nœud en suivant la procédure d’inscription initiale, en utilisant l’une des méthodes décrites dans ce document. Il est inutile de désinscrire un nœud dans Azure Automation State Configuration avant de le réinscrire.

## <a name="check-status-of-vm-setup"></a>Vérifier l’état de configuration de machine virtuelle

State Configuration vous permet d’activer facilement des machines virtuelles Windows Azure pour la gestion de la configuration. En arrière-plan, l’extension Azure VM Desired State Configuration est utilisée pour enregistrer la machine virtuelle auprès d’Azure Automation State Configuration. Étant donné que cette extension s’exécute de façon asynchrone, il peut être important d’en suivre la progression et de résoudre ses éventuels problèmes d’exécution.

> [!NOTE]
> Pour toute méthode d’activation de machines virtuelles Windows Azure pour State Configuration qui utilise l’extension Azure VM Desired State Configuration, il peut s’écouler jusqu’à une heure avant qu’Azure Automation affiche les machines virtuelles telles qu’elles sont inscrites. Ce délai est dû à l’installation de WMF 5 sur la machine virtuelle par l’extension Azure VM Desired State Configuration, requise pour activer les machines virtuelles pour State Configuration.

Pour afficher l’état de l’extension Azure VM Desired State Configuration :

1. Dans le portail Azure, accédez à la machine virtuelle en cours d’activation.
2. Cliquez sur **Extensions** sous **Paramètres**. 
3. Sélectionnez à présent **DSC** ou **DSCForLinux** selon votre système d’exploitation. 
4. Pour plus de détails, vous pouvez cliquer sur **Afficher l’état détaillé**.

## <a name="next-steps"></a>Étapes suivantes

- Pour commencer, consultez [Bien démarrer avec Azure Automation State Configuration](automation-dsc-getting-started.md).
- Pour découvrir comment compiler des configurations DSC pour pouvoir les affecter à des nœuds cibles, consultez [Compiler des configurations DSC dans Azure Automation State Configuration](automation-dsc-compile.md).
- Pour obtenir des informations de référence sur les applets de commande PowerShell, consultez [Az.Automation](/powershell/module/az.automation/?view=azps-3.7.0#automation).
- Pour obtenir des informations sur les prix, consultez [Tarification d’Azure Automation State Configuration](https://azure.microsoft.com/pricing/details/automation/).
- Pour obtenir un exemple d’utilisation d’Azure Automation State Configuration dans un pipeline de déploiement continu, consultez [Configurer un déploiement continu avec Chocolatey](automation-dsc-cd-chocolatey.md).
- Pour plus d’informations sur la résolution des problèmes, consultez [Résoudre des problèmes liés à Azure Automation State Configuration](./troubleshoot/desired-state-configuration.md).
