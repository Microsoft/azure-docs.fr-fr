---
title: Résolution des problèmes de l’agent invité Windows Azure
description: Résoudre les problèmes de non-fonctionnement de l’agent invité Windows Azure
services: virtual-machines-windows
ms.service: virtual-machines-windows
author: dkdiksha
manager: dcscontentpm
editor: ''
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 09/15/2020
ms.author: genli
ms.openlocfilehash: 738c2a240ad6c88186357e69b02d33b40d366d7f
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91977063"
---
# <a name="troubleshooting-windows-azure-guest-agent"></a>Résolution des problèmes de l’agent invité Windows Azure

L’agent invité Windows Azure est un agent de machine virtuelle. Il permet à la machine virtuelle de communiquer avec le contrôleur de structure (serveur physique sous-jacent sur lequel la machine virtuelle est hébergée) via l’adresse IP 168.63.129.16. Il s’agit d’une adresse IP publique virtuelle qui facilite la communication. Pour plus d’informations, consultez [Qu’est-ce que l’adresse IP 168.63.129.16 ?](../../virtual-network/what-is-ip-address-168-63-129-16.md)

 L’agent invité Windows Azure n’est pas installé sur la machine virtuelle migrée vers Azure à partir d’un emplacement local ou créée à l’aide d’une image personnalisée. Dans ces scénarios, vous devez installer manuellement l’agent de machine virtuelle. Pour plus d’informations sur l’agent de machine virtuelle, consultez [Vue d’ensemble d’agent de machine virtuelle Azure](../extensions/agent-windows.md).

Une fois l’agent invité Windows Azure correctement installé, vous pouvez voir les services suivants répertoriés dans la fichier services.msc sur la machine virtuelle :
 
- service de l’agent invité Microsoft Azure
- Service de télémétrie
- service de l’agent RD

**Service de l’agent invité Microsoft Azure** : Ce service est responsable de l’ensemble de la journalisation dans WAppAgent.log. Il est responsable de la configuration de plusieurs extensions et communications entre Invité et Hôte. 

**Service de télémétrie** : ce service est responsable de l’envoi des données de télémétrie de la machine virtuelle au serveur principal.

**service de l’agent RD** : ce service est responsable de l’installation de l’agent invité. Le programme d’installation transparent est également un composant de l’agent RD, qui permet de mettre à niveau d’autres composants et services de l’agent invité. Le service RDAgent est aussi responsable de l’envoi de pulsations de la machine virtuelle invitée à l’agent hôte sur le serveur physique.

> [!NOTE]
> À partir de la version 2.7.41491.971 de l’agent invité de machine virtuelle, le composant de télémétrie est inclus dans le service RDAgent. Par conséquent, vous ne verrez peut-être pas ce service de télémétrie répertorié parmi les machines virtuelles nouvellement créées.

## <a name="checking-agent-status-and-version"></a>Vérification de l’état et de la version de l’agent

Accédez à la page des propriétés de la machine virtuelle dans le portail Azure, puis vérifiez l’**État de l’agent**. Si l’Agent invité Windows Azure fonctionne correctement, l’état est **Prêt**. Si l’état de l’agent de machine virtuelle est **Non prêt**, les extensions et la **commande Exécuter** sur le portail Azure ne fonctionnent pas.

## <a name="troubleshooting-vm-agent-that-is-in-not-ready-status"></a>Résolution des problèmes de l’agent de machine virtuelle à l’état Prêt

### <a name="step-1-check-whether-the-windows-azure-guest-agent-service-is-installed"></a>Étape 1 : vérifier si le service de l’agent invité Windows Azure est installé

- Rechercher le package

    Localisez le dossier C:\WindowsAzure. Si vous voyez le dossier GuestAgent qui affiche le numéro de version, cela signifie que l’agent invité Windows Azure a été installé sur la machine virtuelle. Vous pouvez également rechercher le package installé.  Si l’agent invité Windows Azure est installé sur la machine virtuelle, le package est enregistré à l’emplacement suivant : `C:\windows\OEM\GuestAgent\VMAgentPackage.zip`.
    
    Vous pouvez exécuter la commande PowerShell suivante pour vérifier si l’agent de machine virtuelle a été déployé sur la machine virtuelle :
    
    `Get-AzVM -ResourceGroupName "RGNAME" -Name "VMNAME" -DisplayHint expand`
    
    Dans la sortie, localisez la propriété **ProvisionVMAgent** et vérifiez si la valeur est définie sur **True**. Si c’est le cas, cela signifie que l’agent est installé sur la machine virtuelle.
    
- Vérifier les services et les processus

   Accédez à la console Services (services.msc) et vérifiez l’état des services suivants : service de l’agent invité Windows Azure, service RDAgent, service de télémétrie Windows Azure et service de l’agent réseau Windows Azure.
 
    Vous pouvez également vérifier si ces services s’exécutent en examinant le Gestionnaire des tâches pour les processus suivants :
       
    - WindowsAzureGuestAgent.exe : service de l’agent invité Microsoft Azure.
    - WaAppAgent.exe : Service RDAgent
    - WindowsAzureNetAgent.exe : service de l’agent réseau Windows Azure
    - WindowsAzureTelemetryService.exe : Service de télémétrie Windows Azure

   Si vous ne parvenez pas à trouver ces processus et services, cela signifie que l’agent invité Windows Azure n’est pas installé.

- Vérifier le programme et la fonctionnalité

    Dans le Panneau de configuration, accédez à **Programmes et fonctionnalités** pour déterminer si le service de l’agent invité Windows Azure est installé.

Si vous ne trouvez pas de packages, de services et de processus en cours d’exécution et ne voyez même pas l’agent invité Windows Azure installé sous Programmes et fonctionnalités, essayez d’[installer le service de l’agent invité Windows Azure](../extensions/agent-windows.md). Si l’agent invité ne s’installe pas correctement, vous pouvez [installer l’agent de machine virtuelle en mode hors connexion](./install-vm-agent-offline.md).

Si vous pouvez voir les services et qu’ils sont en cours d’exécution, redémarrez le service pour voir si le problème est résolu. Si les services sont arrêtés, démarrez-les et patientez quelques minutes. Vérifiez ensuite si l’**État de l’agent** est **Prêt**. Si vous constatez que ces services se bloquent, il se peut que certains processus tiers provoquent leur blocage. Pour une résolution plus avancée de ces problèmes, contactez le [Support Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

### <a name="step-2-check-whether-auto-update-is-working"></a>Étape 2 : vérifier si la mise à jour automatique fonctionne

L’agent invité Windows Azure intègre une fonctionnalité de mise à jour automatique. Il recherche automatiquement les nouvelles mises à jour et les installe. Si la fonctionnalité de mise à jour automatique ne fonctionne pas correctement, essayez de désinstaller et réinstaller l’agent invité Windows Azure en procédant comme suit :

1. Si le service de l’agent invité Windows Azure figure sous **Programmes et fonctionnalités**, désinstallez-le.

2. Ouvrez une fenêtre d’invite de commandes avec des privilèges d’administrateur.

3. Arrêtez les services de l’agent invité. Si les services ne s’arrêtent pas, vous devez les configurer pour un **démarrage manuel**, puis redémarrer la machine virtuelle.
    ```
    net stop rdagent
    net stop WindowsAzureGuestAgent
    net stop WindowsAzureTelemetryService
    ```
1. Supprimez les services de l’agent invité :
    ```
    sc delete rdagent
    sc delete WindowsAzureGuestAgent
    sc delete WindowsAzureTelemetryService
    ```
1. Sous `C:\WindowsAzure`, créez un dossier nommé OLD.

1. Déplacez tous les dossiers nommés Packages ou GuestAgent dans le dossier OLD.

1. Téléchargez et installez la dernière version du package d’installation de l’agent à partir d’[ici](https://go.microsoft.com/fwlink/?linkid=394789&clcid=0x409). Des droits d’administrateur sont nécessaires pour effectuer l’installation.

1. Installez l’agent invité à l’aide de la commande suivante :

    ```
    msiexec.exe /i c:\VMAgentMSI\WindowsAzureVmAgent.2.7.<version>.fre.msi /quiet /L*v c:\VMAgentMSI\msiexec.log
    ```
    Vérifiez ensuite si les services de l’agent invité démarrent correctement.
 
    Dans les rares cas où l’agent invité ne s’installe pas correctement, vous pouvez [installer l’agent de machine virtuelle en mode hors connexion](./install-vm-agent-offline.md).
    

### <a name="step-3-check-whether-the-vm-can-connect-to-the-fabric-controller"></a>Étape 3 : vérifier si la machine virtuelle peut se connecter au contrôleur de structure

À l’aide d’un outil tel que PsPing, effectuez un test pour déterminer si la machine virtuelle peut accéder à l’adresse IP 168.63.129.16 sur les ports 80, 32526 et 443. Si la machine virtuelle ne se connecte pas comme prévu, vérifiez si la communication sortante sur les ports 80, 443 et 32526 est ouverte dans votre pare-feu local sur la machine virtuelle. Si cette adresse IP est bloquée, l’agent de machine virtuelle peut présenter un comportement inattendu dans divers scénarios.

## <a name="advanced-troubleshooting"></a>Dépannage avancé

Les événements pour le dépannage de l’agent invité Windows Azure sont enregistrés dans les fichiers journaux suivants :

- C:\WindowsAzure\Logs\WaAppAgent.log
- C:\WindowsAzure\Logs\TransparentInstaller.log

Voici quelques scénarios courants dans lesquels l’agent invité Windows Azure peut passer à l’état **Non prêt** ou cesser de fonctionner comme prévu.

### <a name="agent-stuck-on-starting"></a>Agent bloqué dans l’état « Démarrage »

Dans le journal WaAppAgent, vous pouvez voir que l’agent est bloqué dans l’état Démarrage et qu’il ne peut pas démarrer.

**Enregistrement d’informations**
 
[00000007] [05/28/2019 12:58:50.90] [INFO] WindowsAzureGuestAgent starting. Version 2.7.41491.901

**Analyse**
 
La machine virtuelle exécute toujours l’ancienne version de l’agent invité Windows Azure. Dans le dossier C:\WindowsAzure, vous remarquerez peut-être que plusieurs instances de l’agent invité Windows Azure sont installées, dont plusieurs instances de la même version. Étant donné que plusieurs instances d’agent sont installées, la machine virtuelle ne démarre pas la dernière version de l’agent invité Windows Azure.

**Solution**

Désinstallez manuellement l’agent invité Windows Azure, puis réinstallez-le en procédant comme suit :

1. Ouvrez Panneau de configuration > Programmes et fonctionnalités, puis désinstallez l’agent invité Windows Azure.
1. Ouvrez le Gestionnaire des tâches et arrêtez les services suivants : service de l’agent invité Windows Azure, service RDAgent, service de télémétrie Windows Azure et service de l’agent réseau Windows Azure.
1. Sous C:\WindowsAzure, créez un dossier nommé OLD.
1. Déplacez tous les dossiers nommés Packages ou GuestAgent dans le dossier OLD. Déplacez également les dossiers GuestAgent figurant dans C:\WindowsAzure\logs, qui commencent par GuestAgent_x.x.xxxxx, vers le dossier OLD.
1. Téléchargez et installez la dernière version de l’agent MSI. Pour effectuer l’installation, vous devez disposer de droits d’administrateur.
1. Installez l’agent invité à l’aide de la commande MSI suivante :
    ```
    msiexec.exe /i c:\VMAgentMSI\WindowsAzureVmAgent.2.7.<version>.fre.msi /quiet /L*v c:\VMAgentMSI\msiexec.log`
    ```
1. Vérifiez que RDAgent, l’agent invité Windows Azure et les services de télémétrie de Windows Azure sont en cours d’exécution.
 
1. Vérifiez le fichier WaAppAgent.log pour vous assurer que la dernière version de l’agent invité Windows Azure est en cours d’exécution.
 
1. Supprimez le dossier OLD sous C:\WindowsAzure.
  
### <a name="unable-to-connect-to-wireserver-ip-host-ip"></a>Impossible de se connecter à l’adresse IP WireServer (adresse IP de l’hôte) 

Vous pouvez remarquer les entrées d’erreur suivantes dans les journaux WaAppAgent.log et Telemetry.log :

**Enregistrement d’informations**

```Log sample
[ERROR] GetVersions() failed with exception: Microsoft.ServiceModel.Web.WebProtocolException: Server Error: Service Unavailable (ServiceUnavailable) ---> 
System.Net.WebException: The remote server returned an error: (503) Server Unavailable.
```
```Log sample
[00000011] [12/11/2018 06:27:55.66] [WARN]  (Ignoring) Exception while fetching supported versions from HostGAPlugin: System.Net.WebException: Unable to connect to the remote server 
---> System.Net.Sockets.SocketException: An attempt was made to access a socket in a way forbidden by its access permissions 168.63.129.16:32526
at System.Net.Sockets.Socket.DoConnect(EndPoint endPointSnapshot, SocketAddress socketAddress)
at System.Net.ServicePoint.ConnectSocketInternal(Boolean connectFailure, Socket s4, Socket s6, Socket& socket, IPAddress& address, ConnectSocketState status, IAsyncResult asyncResult, Exception& exception)
--- End of inner exception stack trace ---
at System.Net.WebClient.DownloadDataInternal(Uri address, WebRequest& request)
at System.Net.WebClient.DownloadString(Uri address)
at Microsoft.GuestAgentHostPlugin.Client.GuestInformationServiceClient.GetVersions()
at Microsoft.WindowsAzure.GuestAgent.ContainerStateMachine.HostGAPluginUtility.UpdateHostGAPluginAvailability()`
```
**Analyse**

La machine virtuelle ne peut pas atteindre le serveur hôte wireserver.

**Solution**

1. Le serveur hôte wireserver n’étant pas accessible, connectez-vous à la machine virtuelle à l’aide du Bureau à distance, puis essayez d’accéder à l’URL suivante à partir d’un navigateur Internet : http://168.63.129.16/?comp=versions 
1. Si vous ne parvenez pas à accéder à l’URL de l’étape 1, vérifiez l’interface réseau afin de déterminer si le protocole DHCP est activé et si elle dispose d’un DNS. Pour vérifier l’état du protocole DHCP de l’interface réseau, exécutez la commande suivante : `netsh interface ip show config`.
1. Si le protocole DHCP est désactivé, exécutez la commande suivante en veillant à remplacer la valeur en jaune par le nom de votre interface : `netsh interface ip set address name="Name of the interface" source=dhcp`.
1. Vérifiez si un pare-feu, un proxy ou une autre source pourraient occasionner des problèmes susceptibles de bloquer l’accès à l’adresse IP 168.63.129.16.
1. Vérifiez si le Pare-feu Windows ou un pare-feu tiers bloquent l’accès aux ports 80, 443 et 32526. Pour plus d’informations sur la raison pour laquelle cette adresse ne doit pas être bloquée, consultez [Qu’est-ce que l’adresse IP 168.63.129.16](../../virtual-network/what-is-ip-address-168-63-129-16.md).

### <a name="guest-agent-is-stuck-stopping"></a>Agent invité bloqué dans l’état « Arrêt en cours »  

Vous pouvez remarquer les entrées d’erreur suivantes dans le journal WaAppAgent.log :

**Enregistrement d’informations** 

```Log sample
[00000007] [07/18/2019 14:46:28.87] [WARN] WindowsAzureGuestAgent stopping.
[00000007] [07/18/2019 14:46:28.89] [INFO] Uninitializing StateExecutor with WaitForTerminalStateReachedOnEnd : True
[00000004] [07/18/2019 14:46:28.89] [WARN] WindowsAzureGuestAgent could not be stopped. Exception: System.NullReferenceException: Object reference not set to an instance of an object.
at Microsoft.WindowsAzure.GuestAgent.ContainerStateMachine.GoalStateExecutorBase.WaitForExtensionWorkflowComplete(Boolean WaitForTerminalStateReachedOnEnd)
at Microsoft.WindowsAzure.GuestAgent.ContainerStateMachine.GoalStateExecutorBase.Uninitialize(Boolean WaitForTerminalStateReachedOnEnd)
at Microsoft.WindowsAzure.GuestAgent.ContainerStateMachine.GoalStateExecutorForCloud.Uninitialize(Boolean WaitForTerminalStateReachedOnEnd)
at Microsoft.WindowsAzure.GuestAgent.AgentCore.AgentCore.Stop(Boolean waitForTerminalState)
at Microsoft.WindowsAzure.GuestAgent.AgentCore.AgentService.DoStopService()
at Microsoft.WindowsAzure.GuestAgent.AgentCore.AgentService.<>c__DisplayClass2.<OnStopProcessing>b__1()
```
**Analyse**

L’agent invité Windows Azure est bloqué dans l’état Arrêt en cours.

**Solution**

1. Assurez-vous que le fichier WaAppAgent.exe est en cours d’exécution sur la machine virtuelle. Si ce n’est pas le cas, redémarrez le service rdgagent et patientez cinq minutes. Si le fichier WaAppAgent.exe est en cours d’exécution, terminez le processus WindowsAzureGuest.exe.
2. Si l’étape 1 ne résout pas le problème, supprimez la version actuellement installée et installez la version la plus récente de l’agent manuellement.

### <a name="npcap-loopback-adapter"></a>Carte de bouclage Npcap 

Vous pouvez remarquer les entrées d’erreur suivantes dans le journal WaAppAgent.log :
 
```Log sample
[00000006] [06/20/2019 07:44:28.93] [INFO]  Attempting to discover fabric address on interface Npcap Loopback Adapter.
[00000024] [06/20/2019 07:44:28.93] [WARN]  Empty DHCP option data returned
[00000006] [06/20/2019 07:44:28.93] [ERROR] Did not discover fabric address on interface Npcap Loopback Adapter
```
**Analyse**

La carte de bouclage Npcap est installée sur la machine virtuelle par Wireshark. Wireshark est un outil open source permettant de profiler le trafic réseau et d’analyser les paquets. Ce type d’outil est souvent appelé analyseur réseau, analyseur de protocole réseau ou renifleur.

**Solution**

La carte de bouclage Npcap est probablement installée par WireShark. Essayez de la désactiver, puis vérifiez si le problème est résolu.

## <a name="next-steps"></a>Étapes suivantes

Pour résoudre le problème de non-fonctionnement de l’agent invité Windows Azure, [contactez le Support Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).