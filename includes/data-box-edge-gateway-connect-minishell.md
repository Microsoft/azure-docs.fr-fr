---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 03/06/2019
ms.author: alkohli
ms.openlocfilehash: 6e603442bdecf9a627172e24390b06cef573f475
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96027683"
---
Selon le système d’exploitation du client, les procédures permettant de se connecter à distance à l’appareil sont différentes.

### <a name="remotely-connect-from-a-windows-client"></a>Connexion à distance depuis un client Windows

Avant de commencer, assurez-vous que votre client Windows exécute Windows PowerShell 5.0 ou une version ultérieure.

Suivez cette procédure pour effectuer une connexion distante depuis un client Windows.

1. Démarrez une session Windows PowerShell en tant qu’administrateur.
2. Assurez-vous que le service Windows Remote Management est en cours d’exécution sur votre client. À l’invite de commandes, tapez :

    `winrm quickconfig`

3. Attribuez une variable à l’adresse IP de l’appareil.

    $ip = "<ip_appareil>"

    Remplacez `<device_ip>` par l’adresse IP de votre appareil.

4. Tapez la commande suivante pour ajouter l’adresse IP de l’appareil à la liste des hôtes approuvés du client :

    `Set-Item WSMan:\localhost\Client\TrustedHosts $ip -Concatenate -Force`

5. Démarrez une session Windows PowerShell sur l’appareil :

    `Enter-PSSession -ComputerName $ip -Credential $ip\EdgeUser -ConfigurationName Minishell`

6. Indiquez le mot de passe lorsque vous y êtes invité. Utilisez le mot de passe vous permettant de vous connecter à l’interface utilisateur web locale. Le mot de passe par défaut de cette interface est *Password1*. Lorsque vous êtes connecté à l’appareil à l’aide de PowerShell à distance, vous pouvez voir l’exemple de sortie suivant :  

    ```
    Windows PowerShell
    Copyright (C) Microsoft Corporation. All rights reserved.
    
    PS C:\WINDOWS\system32> winrm quickconfig
    WinRM service is already running on this machine.
    PS C:\WINDOWS\system32> $ip = "10.100.10.10"
    PS C:\WINDOWS\system32> Set-Item WSMan:\localhost\Client\TrustedHosts $ip -Concatenate -Force
    PS C:\WINDOWS\system32> Enter-PSSession -ComputerName $ip -Credential $ip\EdgeUser -ConfigurationName Minishell

    WARNING: The Windows PowerShell interface of your device is intended to be used only for the initial network configuration. Please engage Microsoft Support if you need to access this interface to troubleshoot any potential issues you may be experiencing. Changes made through this interface without involving Microsoft Support could result in an unsupported configuration.
    [10.100.10.10]: PS>
    ```

### <a name="remotely-connect-from-a-linux-client"></a>Connexion à distance depuis un client Linux

Sur le client Linux que vous utiliserez pour vous connecter :

- [Installez la plus récente de PowerShell Core pour Linux](/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-6), à partir de GitHub, afin d’obtenir la fonctionnalité d’accès distant SSH. 
- [Installez uniquement le package `gss-ntlmssp` à partir du module NTLM](https://github.com/Microsoft/omi/blob/master/Unix/doc/setup-ntlm-omi.md). Pour les clients Ubuntu, exécutez la commande ci-dessous :
    - `sudo apt-get install gss-ntlmssp`

Pour plus d’informations, voir [Accès distant à PowerShell via SSH](/powershell/scripting/learn/remoting/ssh-remoting-in-powershell-core?view=powershell-6).

Suivez cette procédure pour effectuer une connexion distante depuis un client NFS.

1. Pour ouvrir la session PowerShell, saisissez :

    `sudo pwsh`
 
2. Pour vous connecter à l’aide du client distant, saisissez :

    `Enter-PSSession -ComputerName $ip -Authentication Negotiate -ConfigurationName Minishell -Credential ~\EdgeUser`

    Lorsque vous y êtes invité, indiquez le mot de passe utilisé pour la connexion à votre appareil.
 
> [!NOTE]
> Cette procédure ne fonctionne pas sur Mac OS.