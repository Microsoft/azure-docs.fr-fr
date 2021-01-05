---
title: Gérer des ressources locales à distance à l'aide des fonctions PowerShell
description: Découvrez comment configurer les connexions hybrides dans Azure Relay pour connecter une application de fonction PowerShell à des ressources locales, ce qui peut ensuite être utilisé pour gérer à distance la ressource locale.
author: eamono
ms.topic: conceptual
ms.date: 04/26/2020
ms.author: eamono
ms.openlocfilehash: 6034d1327d263eda49881af5eedf94ae06495128
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83122138"
---
# <a name="managing-hybrid-environments-with-powershell-in-azure-functions-and-app-service-hybrid-connections"></a>Gestion des environnements hybrides avec PowerShell dans Azure Functions et Connections hybrides App Service

La fonctionnalité Connexions hybrides d'Azure App Service permet d'accéder aux ressources d'autres réseaux. Pour en savoir plus sur cette fonctionnalité, consultez la documentation [Connexions hybrides](../app-service/app-service-hybrid-connections.md). Cet article décrit l'utilisation de cette fonctionnalité qui permet d'exécuter des fonctions PowerShell ciblant un serveur local. Ce serveur peut ensuite être utilisé pour gérer toutes les ressources de l'environnement local à partir d'une fonction Azure PowerShell.


## <a name="configure-an-on-premises-server-for-powershell-remoting"></a>Configurer un serveur local pour la communication à distance PowerShell

Le script suivant active la communication à distance PowerShell, et crée une règle de pare-feu et un écouteur https WinRM. À des fins de test, un certificat auto-signé est utilisé. Dans un environnement de production, nous vous recommandons d'utiliser un certificat signé.

```powershell
# For configuration of WinRM, see
# https://docs.microsoft.com/windows/win32/winrm/installation-and-configuration-for-windows-remote-management.

# Enable PowerShell remoting.
Enable-PSRemoting -Force

# Create firewall rule for WinRM. The default HTTPS port is 5986.
New-NetFirewallRule -Name "WinRM HTTPS" `
                    -DisplayName "WinRM HTTPS" `
                    -Enabled True `
                    -Profile "Any" `
                    -Action "Allow" `
                    -Direction "Inbound" `
                    -LocalPort 5986 `
                    -Protocol "TCP"

# Create new self-signed-certificate to be used by WinRM.
$Thumbprint = (New-SelfSignedCertificate -DnsName $env:COMPUTERNAME  -CertStoreLocation Cert:\LocalMachine\My).Thumbprint

# Create WinRM HTTPS listener.
$Cmd = "winrm create winrm/config/Listener?Address=*+Transport=HTTPS @{Hostname=""$env:COMPUTERNAME ""; CertificateThumbprint=""$Thumbprint""}"
cmd.exe /C $Cmd
```

## <a name="create-a-powershell-function-app-in-the-portal"></a>Créer une application de fonction PowerShell dans le portail

La fonctionnalité Connexions hybrides d'App Service est uniquement disponible avec les plans tarifaires De base, Standard et Isolé. Lors de la création de l'application de fonction avec PowerShell, créez ou sélectionnez l'un de ces plans.

1. Dans le menu du portail Azure ou dans la page **Accueil**, sélectionnez **Créer une ressource**.

1. Dans la page **Nouveau**, sélectionnez **Calcul** > **Application de fonction**.

1. Dans la page **De base**, utilisez les paramètres d’application de fonction comme indiqué dans le tableau ci-dessous.

    | Paramètre      | Valeur suggérée  | Description |
    | ------------ | ---------------- | ----------- |
    | **Abonnement** | Votre abonnement | Abonnement sous lequel est créée cette nouvelle application de fonction. |
    | **[Groupe de ressources](../azure-resource-manager/management/overview.md)** |  *myResourceGroup* | Nom du nouveau groupe de ressources dans lequel créer votre Function App. |
    | **Nom de l’application de fonction** | Nom globalement unique | Nom qui identifie votre nouvelle Function App. Les caractères valides sont `a-z` (insensible à la casse), `0-9`et `-`.  |
    |**Publier**| Code | Option permettant de publier des fichiers de code ou un conteneur Docker. |
    | **Pile d’exécution** | Langage préféré | Choisissez PowerShell Core. |
    |**Version**| Numéro de version | Choisissez la version de votre runtime installé.  |
    |**Région**| Région recommandée | Choisissez une [région](https://azure.microsoft.com/regions/) près de chez vous ou près d’autres services auxquels ont accès vos fonctions. |

    :::image type="content" source="./media/functions-hybrid-powershell/function-app-create-basics.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

1. Sélectionnez **Suivant : Hébergement**. Dans la page **Hébergement**, entrez les paramètres suivants.

    | Paramètre      | Valeur suggérée  | Description |
    | ------------ | ---------------- | ----------- |
    | **[Compte de stockage](../storage/common/storage-account-create.md)** |  Nom globalement unique |  Créez un compte de stockage utilisé par votre application de fonction. Les noms des comptes de stockage doivent comporter entre 3 et 24 caractères, uniquement des lettres minuscules et des chiffres. Vous pouvez également utiliser un compte existant qui doit répondre aux [exigences relatives aux comptes de stockage](../azure-functions/functions-scale.md#storage-account-requirements). |
    |**Système d’exploitation**| Système d’exploitation préféré | Un système d’exploitation est présélectionné pour vous en fonction de la sélection de votre pile d’exécution, mais vous pouvez modifier le paramètre si nécessaire. |
    | **[Type de plan](../azure-functions/functions-scale.md)** | **Plan App service** | Sélectionnez **App Service plan** (Plan App Service). Si vous exécutez dans un plan App Service, vous devez gérer la [mise à l’échelle de votre application de fonction](../azure-functions/functions-scale.md).  |

    :::image type="content" source="./media/functions-hybrid-powershell/function-app-create-hosting.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

1. Sélectionnez **Suivant : Supervision**. Dans la page **Supervision**, entrez les paramètres suivants.

    | Paramètre      | Valeur suggérée  | Description |
    | ------------ | ---------------- | ----------- |
    | **[Application Insights](../azure-functions/functions-monitoring.md)** | Default | Crée une ressource Application Insights avec le même *nom de l’application* dans la région prise en charge la plus proche. En développant ce paramètre ou en sélectionnant **Créer nouveau**, vous pouvez modifier le nom Application Insights ou choisir une autre région dans une [zone géographique Azure](https://azure.microsoft.com/global-infrastructure/geographies/) où vous souhaitez stocker vos données. |

    :::image type="content" source="./media/functions-hybrid-powershell/function-app-create-monitoring.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

1. Sélectionnez **Vérifier + créer** pour passer en revue les sélections de configuration d’application.

1. Dans la page **Vérifier + créer**, vérifiez vos paramètres, puis sélectionnez **Créer** pour provisionner et déployer l’application de fonction.

1. Cliquez sur l’icône **Notifications** en haut à droite du portail pour voir le message **Le déploiement a été effectué**.

1. Sélectionnez **Accéder à la ressource** pour afficher votre nouvelle application de fonction. Vous pouvez également sélectionner **Épingler au tableau de bord**. L’épinglage permet de revenir plus facilement à cette ressource d’application de fonction à partir de votre tableau de bord.

## <a name="create-a-hybrid-connection-for-the-function-app"></a>Créer une connexion hybride pour l’application de fonction

Les connexions hybrides sont configurées à partir de la section de mise en réseau de l'application de fonction :

1. Sous **Settings** (Paramètres) dans l’application de fonction que vous venez de créer, sélectionnez **Networking** (Réseau). 
1. Sélectionnez **Configurer vos points de terminaison de connexion hybride**.
   
    :::image type="content" source="./media/functions-hybrid-powershell/configure-hybrid-connection-endpoint.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

1. Sélectionnez **Ajouter une connexion hybride**.
   
    :::image type="content" source="./media/functions-hybrid-powershell/hybrid-connection-overview.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

1. Entrez les informations relatives à la connexion hybride, comme indiqué juste après la capture d'écran suivante. Vous avez la possibilité de faire correspondre le paramètre **Hôte du point de terminaison** au nom d'hôte du serveur local afin de faciliter la mémorisation du nom du serveur pour la suite, lors de l'exécution de commandes à distance. Le port correspond au port par défaut du service de gestion à distance Windows qui a précédemment été défini sur le serveur.
  
      :::image type="content" source="./media/functions-hybrid-powershell/add-hybrid-connection.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

    | Paramètre      | Valeur suggérée  |
    | ------------ | ---------------- |
    | **Nom de la connexion hybride** | ContosoHybridOnPremisesServer |
    | **Hôte du point de terminaison** | finance1 |
    | **Port du point de terminaison** | 5986 |
    | **Espace de noms Service Bus** | Création |
    | **Lieu** | Sélectionnez un emplacement disponible. |
    | **Nom** | contosopowershellhybrid | 

1. Sélectionnez **OK** pour créer la connexion hybride.

## <a name="download-and-install-the-hybrid-connection"></a>Télécharger et installer la connexion hybride

1. Sélectionnez **Télécharger le gestionnaire de connexions** pour enregistrer le fichier *.msi* localement sur votre ordinateur.

    :::image type="content" source="./media/functions-hybrid-powershell/download-hybrid-connection-installer.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

1. Copiez le fichier *.msi* de votre ordinateur local vers le serveur local.
1. Exécutez le programme d'installation d'Hybrid Connection Manager pour installer le service sur le serveur local.

    :::image type="content" source="./media/functions-hybrid-powershell/hybrid-installation.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

1. À partir du portail, ouvrez la connexion hybride, puis copiez la chaîne de connexion de la passerelle dans le Presse-papiers.

    :::image type="content" source="./media/functions-hybrid-powershell/copy-hybrid-connection.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

1. Ouvrez l’interface utilisateur de Hybrid Connection Manager sur le serveur local.

    :::image type="content" source="./media/functions-hybrid-powershell/hybrid-connection-ui.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

1. Sélectionnez **Entrer manuellement** et collez la chaîne de connexion à partir du Presse-papiers.

    :::image type="content" source="./media/functions-hybrid-powershell/enter-manual-connection.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

1. Redémarrez Hybrid Connection Manager à partir de PowerShell s'il ne s'affiche pas comme connecté.
    ```powershell
    Restart-Service HybridConnectionManager
    ```

## <a name="create-an-app-setting-for-the-password-of-an-administrator-account"></a>Créer un paramètre d’application pour le mot de passe d’un compte d’administrateur

1. Sous **Settings** (Paramètres) dans votre application de fonction, sélectionnez **Configuration**. 
1. Sélectionnez **+ New application setting** (+ Nouveau paramètre d’application).

    :::image type="content" source="./media/functions-hybrid-powershell/select-configuration.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

1. Nommez le paramètre **ContosoUserPassword** et entrez le mot de passe. Sélectionnez **OK**.
1. Sélectionnez **Enregistrer** pour stocker le mot de passe dans l'application de fonction.

    :::image type="content" source="./media/functions-hybrid-powershell/save-administrator-password.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

## <a name="create-a-function-http-trigger"></a>Créer un déclencheur HTTP de fonction

1. Dans votre application de fonction, sélectionnez **Functions** (Fonctions), puis **+ Add** (+ Ajouter).

    :::image type="content" source="./media/functions-hybrid-powershell/create-http-trigger-function.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

1. Sélectionnez le modèle **Déclencheur HTTP** .

    :::image type="content" source="./media/functions-hybrid-powershell/select-http-trigger-template.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

1. Nommez la nouvelle fonction, puis sélectionnez **Fonction**.

    :::image type="content" source="./media/functions-hybrid-powershell/create-new-http-function.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

## <a name="test-the-function"></a>Tester la fonction

1. Dans la nouvelle fonction, sélectionnez **Code + Test** (Coder et tester). Remplacez le code PowerShell du modèle par le code suivant :

    ```powershell
    # Input bindings are passed in via param block.
    param($Request, $TriggerMetadata)
    
    # Write to the Azure Functions log stream.
    Write-Output "PowerShell HTTP trigger function processed a request."
    
    # Note that ContosoUserPassword is a function app setting, so I can access it as $env:ContosoUserPassword.
    $UserName = "ContosoUser"
    $securedPassword = ConvertTo-SecureString  $Env:ContosoUserPassword -AsPlainText -Force
    $Credential = [System.management.automation.pscredential]::new($UserName, $SecuredPassword)
    
    # This is the name of the hybrid connection Endpoint.
    $HybridEndpoint = "finance1"
    
    $Script = {
        Param(
            [Parameter(Mandatory=$True)]
            [String] $Service
        )
        Get-Service $Service
    }
    
    Write-Output "Scenario 1: Running command via Invoke-Command"
    Invoke-Command -ComputerName $HybridEndpoint `
                   -Credential $Credential `
                   -Port 5986 `
                   -UseSSL `
                   -ScriptBlock $Script `
                   -ArgumentList "*" `
                   -SessionOption (New-PSSessionOption -SkipCACheck)
    ```

1. Sélectionnez **Enregistrer**.

    :::image type="content" source="./media/functions-hybrid-powershell/save-http-function.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

 1. Sélectionnez **Test**, puis **Run** (Exécuter) pour tester la fonction. Examinez les journaux pour vérifier que le test a réussi.

     :::image type="content" source="./media/functions-hybrid-powershell/test-function-hybrid.png" alt-text="Créer une application de fonction : concepts de base" border="true":::

## <a name="managing-other-systems-on-premises"></a>Gestion d’autres systèmes en local

Vous pouvez utiliser le serveur local connecté pour vous connecter à d’autres serveurs et systèmes de gestion dans l’environnement local. Cela vous permet de gérer vos opérations de centre de données à partir d'Azure en utilisant vos fonctions PowerShell. Le script suivant inscrit une session de configuration PowerShell qui s'exécute sous les informations d'identification fournies. Ces informations d'identification doivent être destinées à un administrateur sur les serveurs distants. Vous pouvez ensuite utiliser cette configuration pour accéder à d'autres points de terminaison du serveur local ou du centre de données.

```powershell
# Input bindings are passed in via param block.
param($Request, $TriggerMetadata)

# Write to the Azure Functions log stream.
Write-Host "PowerShell HTTP trigger function processed a request."

# Note that ContosoUserPassword is a function app setting, so I can access it as $env:ContosoUserPassword.
$UserName = "ContosoUser"
$SecuredPassword = ConvertTo-SecureString  $Env:ContosoUserPassword -AsPlainText -Force
$Credential = [System.management.automation.pscredential]::new($UserName, $SecuredPassword)

# This is the name of the hybrid connection Endpoint.
$HybridEndpoint = "finance1"

# The remote server that will be connected to run remote PowerShell commands on
$RemoteServer = "finance2".

Write-Output "Use hybrid connection server as a jump box to connect to a remote machine"

# We are registering an endpoint that runs under credentials ($Credential) that has access to the remote server.
$SessionName = "HybridSession"
$ScriptCommand = {
    param (
        [Parameter(Mandatory=$True)]
        $SessionName)

    if (-not (Get-PSSessionConfiguration -Name $SessionName -ErrorAction SilentlyContinue))
    {
        Register-PSSessionConfiguration -Name $SessionName -RunAsCredential $Using:Credential
    }
}

Write-Output "Registering session on hybrid connection jumpbox"
Invoke-Command -ComputerName $HybridEndpoint `
               -Credential $Credential `
               -Port 5986 `
               -UseSSL `
               -ScriptBlock $ScriptCommand `
               -ArgumentList $SessionName `
               -SessionOption (New-PSSessionOption -SkipCACheck)

# Script to run on the jump box to run against the second machine.
$RemoteScriptCommand = {
    param (
        [Parameter(Mandatory=$True)]
        $ComputerName)
        # Write out the hostname of the hybrid connection server.
        hostname
        # Write out the hostname of the remote server.
        Invoke-Command -ComputerName $ComputerName -Credential $Using:Credential -ScriptBlock {hostname} `
                        -UseSSL -Port 5986 -SessionOption (New-PSSessionOption -SkipCACheck)
}

Write-Output "Running command against remote machine via jumpbox by connecting to the PowerShell configuration session"
Invoke-Command -ComputerName $HybridEndpoint `
               -Credential $Credential `
               -Port 5986 `
               -UseSSL `
               -ScriptBlock $RemoteScriptCommand `
               -ArgumentList $RemoteServer `
               -SessionOption (New-PSSessionOption -SkipCACheck) `
               -ConfigurationName $SessionName
```

Remplacez les variables suivantes de ce script par les valeurs applicables de votre environnement :
* $HybridEndpoint
* $RemoteServer

Dans les deux scénarios précédents, vous pouvez vous connecter à vos environnements locaux et les gérer à l'aide de PowerShell dans Azure Functions et Connexions hybrides. Nous vous encourageons à en apprendre plus sur les [Connexions hybrides](../app-service/app-service-hybrid-connections.md) et [PowerShell dans les fonctions](./functions-reference-powershell.md).

Vous pouvez également utiliser les [réseaux virtuels](./functions-create-vnet.md) Azure pour vous connecter à votre environnement local via Azure Functions.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"] 
> [En savoir plus sur l’utilisation des fonctions PowerShell](functions-reference-powershell.md)
