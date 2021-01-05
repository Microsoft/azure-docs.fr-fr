---
title: Installer l’agent Log Analytics sur des ordinateurs Windows
description: Cet article décrit la connexion d’ordinateurs Windows hébergés dans d’autres clouds ou localement à Azure Monitor avec l’agent Log Analytics pour Windows.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/03/2020
ms.openlocfilehash: d283c2b2cdbbeb3ef4bc4e25f4288dfd95158552
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89003369"
---
# <a name="install-log-analytics-agent-on-windows-computers"></a>Installer l’agent Log Analytics sur des ordinateurs Windows
Cet article fournit des détails sur l’installation de l’agent Log Analytics sur des ordinateurs Windows à l’aide des méthodes suivantes :

* Installation manuelle à l’aide de l’[Assistant Installation](#install-agent-using-setup-wizard) d’une [ligne de commande](#install-agent-using-command-line).
* [Desired State Configuration (DSC) d’Azure Automation](#install-agent-using-dsc-in-azure-automation). 

>[!IMPORTANT]
> Les méthodes d’installation décrites dans cet article sont généralement utilisées pour des machines virtuelles locales ou dans d’autres clouds. Pour d’autres options efficaces que vous pouvez utiliser pour des machines virtuelles Azure, consultez [Options d’installation](log-analytics-agent.md#installation-options).

> [!NOTE]
> Si vous avez besoin de configurer l’agent pour qu’il rende compte à plusieurs espaces de travail, cette opération ne peut être effectuée pendant l’installation initiale, mais uniquement plus tard en mettant à jour les paramètres dans le panneau de configuration ou PowerShell comme décrit dans [Ajout ou suppression d’un espace de travail](agent-manage.md#adding-or-removing-a-workspace).  

## <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge

Pour obtenir la liste des versions de Windows que l’agent Log Analytics prend en charge, consultez [Vue d’ensemble des agents Azure Monitor](agents-overview.md#supported-operating-systems).

### <a name="sha-2-code-signing-support-requirement"></a>Configuration requise pour la prise en charge de la signature de code SHA-2 
À compter du 17 août 2020, l’agent Windows utilisera exclusivement la signature SHA-2. Ce changement aura un impact sur les clients qui utilisent l’agent Log Analytics sur un système d’exploitation hérité dans le cadre d’un service Azure (Azure Monitor, Azure Automation, Azure Update Management, Azure Change Tracking, Azure Security Center, Azure Sentinel, Windows Defender ATP). Le client n’aura rien à faire à moins que l’agent s’exécute sur une version héritée du système d’exploitation (Windows 7, Windows Server 2008 R2 et Windows Server 2008). Les clients utilisant une version héritée du système d’exploitation sont tenus d’effectuer les actions suivantes sur leurs ordinateurs avant le 17 août 2020. À défaut, leurs agents cesseront d’envoyer les données à leurs espaces de travail Log Analytics :

1. Installer le dernier Service Pack du système d’exploitation. Les versions de Service Pack nécessaires sont les suivantes :
    - Windows 7 SP1
    - Windows Server 2008 SP2
    - Windows Server 2008 R2 SP1

2. Installer les mises à jour Windows de signature SHA-2 pour le système d’exploitation en question, comme décrit dans [Obligation de prise en charge de la signature du code SHA-2 2019 pour Windows et WSUS](https://support.microsoft.com/help/4472027/2019-sha-2-code-signing-support-requirement-for-windows-and-wsus).
3. Effectuer une mise à jour vers la dernière version de l’agent Windows (version 10.20.18029).
4. Configurer l’agent de sorte qu’il [utilise TLS 1.2](agent-windows.md#configure-agent-to-use-tls-12) (recommandé). 

## <a name="network-requirements"></a>Configuration requise pour le réseau
Pour connaître la configuration réseau requise pour l’agent Windows, consultez [Présentation de l’agent Log Analytics](log-analytics-agent.md#network-requirements).


   
## <a name="configure-agent-to-use-tls-12"></a>Configurer l’Agent de façon à utiliser TLS 1.2
Le protocole [TLS 1.2](/windows-server/security/tls/tls-registry-settings#tls-12) garantit la sécurité des données en transit pour la communication entre l’agent Windows et le service Log Analytics. Si vous opérez l’installation sur un [système d’exploitation sans le protocole TLS 1.2 activé par défaut](data-security.md#sending-data-securely-using-tls-12), vous devez configurer le protocole TLS 1.2 en procédant de la manière décrite ci-dessous.

1. Localisez les sous-clés de registre suivantes : **HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols**
2. Créez une sous-clé sous **Protocoles** pour TLS 1.2 : **HKLM\System\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2**.
3. Créez une sous-clé **Client** sous la sous-clé de version du protocole TLS 1.2 que vous venez de créer. Par exemple, **HKLM\System\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client**.
4. Créez les valeurs DWORD suivantes sous **HKLM\System\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client** :

    * **Enabled** [Valeur = 1]
    * **DisabledByDefault** [Valeur = 0]  

Configurez .NET Framework 4.6 (ou version ultérieure) de façon à prendre en charge le chiffrement sécurisé, désactivé par défaut. Le [chiffrement fort](/dotnet/framework/network-programming/tls#schusestrongcrypto) utilise des protocoles réseau sécurisés, comme TLS 1.2, et bloque les autres. 

1. Localisez les sous-clés de registre suivantes : **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\\.NETFramework\v4.0.30319**.  
2. Créez la valeur DWORD **SchUseStrongCrypto** sous cette sous-clé avec la valeur **1**.  
3. Localisez les sous-clés de registre suivantes : **HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\\.NETFramework\v4.0.30319**.  
4. Créez la valeur DWORD **SchUseStrongCrypto** sous cette sous-clé avec la valeur **1**. 
5. Redémarrez le système pour que les paramètres s’appliquent. 

## <a name="install-agent-using-setup-wizard"></a>Installer l’agent à l’aide de l’Assistant Installation
Les étapes suivantes installent et configurent l’agent Log Analytics dans le cloud Azure et Azure Government en utilisant l’Assistant Installation de l’agent sur votre ordinateur. Pour apprendre à configurer l’agent de façon à ce qu’il rende également des comptes à un groupe d’administration System Center Operations Manager, voir [Déployer l’Agent Operations Manager avec l’Assistant Installation de l’agent](/system-center/scom/manage-deploy-windows-agent-manually#to-deploy-the-operations-manager-agent-with-the-agent-setup-wizard).

1. Revenez sur la page **Serveurs Windows** de votre espace de travail Log Analytics et sélectionnez l’option **Télécharger l’agent Windows** correspondant à la version adaptée à l’architecture du processeur du système d’exploitation Windows.   
2. Exécutez le programme d’installation pour installer l’agent sur votre ordinateur.
2. Sur la page d’**accueil**, cliquez sur **Suivant**.
3. Dans la page **Termes du contrat de licence**, lisez les conditions de licence, puis cliquez sur **J’accepte**.
4. Dans la page **Dossier de destination**, modifiez ou conservez le dossier d’installation par défaut, puis cliquez sur **Suivant**.
5. Sur la page **Options d’installation de l’agent**, choisissez de connecter l’agent à Azure Log Analytics, puis cliquez sur **Suivant**.   
6. Dans la page **Azure Log Analytics**, procédez comme suit :
   1. Collez l’**ID de l’espace de travail** et la **Clé d’espace de travail (Clé primaire)** que vous avez copiés précédemment.  Si l’ordinateur doit rendre compte à un espace de travail Log Analytics dans le cloud Azure Government, dans la liste déroulante **Cloud Azure**, sélectionnez **Azure - Gouvernement des États-Unis**.  
   2. Si l’ordinateur a besoin de communiquer via un serveur proxy avec le service Log Analytics, cliquez sur **Avancé**, puis indiquez l’URL et le numéro de port du serveur proxy.  Si votre serveur proxy requiert une authentification, tapez le nom d’utilisateur et un mot de passe pour vous authentifier auprès du serveur proxy, puis cliquez sur **Suivant**.  
7. Cliquez sur **Suivant** après avoir fourni les paramètres de configuration nécessaires.<br><br> ![Coller l’ID et la clé primaire de l’espace de travail](media/agent-windows/log-analytics-mma-setup-laworkspace.png)<br><br>
8. Dans la page **Prêt pour l’installation**, passez en revue vos choix, puis cliquez sur **Installer**.
9. Dans la page **Configuration effectuée**, cliquez sur **Terminer**.

Lorsque vous avez terminé, **Microsoft Monitoring Agent** apparaît dans le **Panneau de configuration**. Pour confirmer qu’il rend compte à Log Analytics, passez en revue [Vérifier la connectivité de l’agent à Log Analytics](#verify-agent-connectivity-to-azure-monitor). 

## <a name="install-agent-using-command-line"></a>Installer le client à l’aide d’une ligne de commande
Le fichier téléchargé pour l’agent est un package d’installation autonome.  Le programme d’installation de l’agent et les fichiers de prise en charge sont contenus dans le package et doivent être extraits pour effectuer une installation correcte à l’aide de la ligne de commande indiquée dans les exemples suivants.    

>[!NOTE]
>Si vous souhaitez mettre à niveau un agent, vous devez utiliser l’API de script Log Analytics. Consultez la rubrique [Gestion et maintenance de l’agent Log Analytics sous Windows et Linux](agent-manage.md) pour plus d’informations.

Le tableau suivant répertorie les paramètres spécifiques pris en charge par le programme d'installation de l'agent, notamment lors d'un déploiement à l'aide d'Automation DSC.

|Options propres à MMA                   |Notes         |
|---------------------------------------|--------------|
| NOAPM=1                               | Paramètre facultatif. Installe l’agent sans analyse des performances des applications .NET.|   
|ADD_OPINSIGHTS_WORKSPACE               | 1 = Configurer l’agent de façon à ce qu’il se connecte à un espace de travail                |
|OPINSIGHTS_WORKSPACE_ID                | ID (GUID) de l’espace de travail à ajouter                    |
|OPINSIGHTS_WORKSPACE_KEY               | Clé d’espace de travail utilisée initialement pour l’authentification auprès de l’espace de travail |
|OPINSIGHTS_WORKSPACE_AZURE_CLOUD_TYPE  | Spécifie l’environnement cloud où se trouve l’espace de travail <br> 0 = cloud commercial Azure (par défaut) <br> 1 = Azure Government |
|OPINSIGHTS_PROXY_URL               | URI du proxy à utiliser |
|OPINSIGHTS_PROXY_USERNAME               | Nom d’utilisateur permettant d’accéder à un proxy authentifié |
|OPINSIGHTS_PROXY_PASSWORD               | Mot de passe permettant d’accéder à un proxy authentifié |

1. Pour extraire les fichiers d’installation de l’agent, à partir d’une invite de commandes avec élévation de privilèges, exécutez `MMASetup-<platform>.exe /c` et indiquez l’emplacement où extraire les fichiers.  L’autre possibilité consiste à spécifier le chemin d’accès à l’aide des arguments `MMASetup-<platform>.exe /c /t:<Full Path>`.  
2. Pour installer l’agent en mode silencieux et le configurer pour qu’il rende compte à un espace de travail dans le cloud Azure Commercial, à partir du dossier dans lequel vous avez extrait les fichiers d’installation, tapez : 
   
     ```dos
    setup.exe /qn NOAPM=1 ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_AZURE_CLOUD_TYPE=0 OPINSIGHTS_WORKSPACE_ID="<your workspace ID>" OPINSIGHTS_WORKSPACE_KEY="<your workspace key>" AcceptEndUserLicenseAgreement=1
    ```

   ou, pour configurer l’agent afin qu’il rende compte au cloud du gouvernement des États-Unis, tapez : 

     ```dos
    setup.exe /qn NOAPM=1 ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_AZURE_CLOUD_TYPE=1 OPINSIGHTS_WORKSPACE_ID="<your workspace ID>" OPINSIGHTS_WORKSPACE_KEY="<your workspace key>" AcceptEndUserLicenseAgreement=1
    ```
    >[!NOTE]
    >Les valeurs de chaîne des paramètres *OPINSIGHTS_WORKSPACE_ID* et *OPINSIGHTS_WORKSPACE_KEY* doivent être encapsulées dans des guillemets doubles pour indiquer à Windows Installer de les interpréter comme des options valides pour le package. 

## <a name="install-agent-using-dsc-in-azure-automation"></a>Installer l’agent à l’aide de DSC dans Azure Automation

Vous pouvez utiliser l’exemple de script suivant pour installer l’agent à l’aide d’Azure Automation DSC.   Si vous n’avez pas de compte Automation, consultez [Prise en main d’Azure Automation](../../automation/index.yml) pour comprendre les exigences et les étapes de création d’un compte Automation à respecter avant d’utiliser Automation DSC.  Si vous n’êtes pas familiarisé avec Automation DSC, consultez [Prise en main d’Azure Automation DSC](../../automation/automation-dsc-getting-started.md).

L’exemple suivant installe l’agent 64 bits, identifié par la valeur `URI`. Vous pouvez également utiliser la version 32 bits en remplaçant la valeur de l’URI. Les URI pour les deux versions sont :

- Agent Windows 64 bits : https://go.microsoft.com/fwlink/?LinkId=828603
- Agent Windows 32 bits : https://go.microsoft.com/fwlink/?LinkId=828604


>[!NOTE]
>Cet exemple de procédure et de script ne prend pas en charge la mise à niveau de l’agent déjà déployé sur un ordinateur Windows.

Les versions 32 bits et 64 bits du package de l’agent ont des codes de produit différents et les nouvelles versions publiées ont également une valeur unique.  Le code de produit est un GUID qui est l’identification principale d’une application ou d’un produit, et est représenté par la propriété **ProductCode** de Windows Installer.  Dans le script **MMAgent.ps1**, la valeur `ProductId` doit correspondre au code de produit dans le package du programme d’installation 32 bits ou 64 bits de l’agent.

Pour récupérer le code de produit du package d’installation de l’agent directement, vous pouvez utiliser Orca.exe à partir des [composants SDK Windows pour les développeurs Windows Installer](/windows/win32/msi/platform-sdk-components-for-windows-installer-developers) (composant du SDK Windows) ou utiliser PowerShell en suivant un [exemple de script](https://www.scconfigmgr.com/2014/08/22/how-to-get-msi-file-information-with-powershell/) écrit par un MVP (Microsoft Valuable Professional).  Pour les deux approches, vous devez tout d’abord extraire le fichier **MOMagent.msi** du package d’installation MMASetup.  Cela est indiqué plus haut dans la première étape, sous la section [Installer l’agent à l’aide de la ligne de commande](#install-agent-using-command-line).  

1. Importer le Module DSC xPSDesiredStateConfiguration à partir de [https://www.powershellgallery.com/packages/xPSDesiredStateConfiguration](https://www.powershellgallery.com/packages/xPSDesiredStateConfiguration) dans Azure Automation.  
2.    Créez des ressources variables Azure Automation pour *OPSINSIGHTS_WS_ID* et *OPSINSIGHTS_WS_KEY*. Affectez votre ID d’espace de travail Log Analytics comme valeur *OPSINSIGHTS_WS_ID* et affectez la clé primaire de votre espace de travail comme valeur *OPSINSIGHTS_WS_KEY*.
3.    Copiez le script et enregistrez-le sous le nom MMAgent.ps1.

```powershell
Configuration MMAgent
{
    $OIPackageLocalPath = "C:\Deploy\MMASetup-AMD64.exe"
    $OPSINSIGHTS_WS_ID = Get-AutomationVariable -Name "OPSINSIGHTS_WS_ID"
    $OPSINSIGHTS_WS_KEY = Get-AutomationVariable -Name "OPSINSIGHTS_WS_KEY"

    Import-DscResource -ModuleName xPSDesiredStateConfiguration
    Import-DscResource -ModuleName PSDesiredStateConfiguration

    Node OMSnode {
        Service OIService
        {
            Name = "HealthService"
            State = "Running"
            DependsOn = "[Package]OI"
        }

        xRemoteFile OIPackage {
            Uri = "https://go.microsoft.com/fwlink/?LinkId=828603"
            DestinationPath = $OIPackageLocalPath
        }

        Package OI {
            Ensure = "Present"
            Path  = $OIPackageLocalPath
            Name = "Microsoft Monitoring Agent"
            ProductId = "8A7F2C51-4C7D-4BFD-9014-91D11F24AAE2"
            Arguments = '/C:"setup.exe /qn NOAPM=1 ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_ID=' + $OPSINSIGHTS_WS_ID + ' OPINSIGHTS_WORKSPACE_KEY=' + $OPSINSIGHTS_WS_KEY + ' AcceptEndUserLicenseAgreement=1"'
            DependsOn = "[xRemoteFile]OIPackage"
        }
    }
}

```

4. Mettez à jour la valeur `ProductId` du script avec le code de produit extrait de la dernière version du package d'installation de l'agent à l'aide des méthodes recommandées précédemment. 
5. [Importez le script de configuration MMAgent.ps1](../../automation/automation-dsc-getting-started.md#import-a-configuration-into-azure-automation) dans votre compte Automation. 
6. [Affectez un nœud ou ordinateur Windows](../../automation/automation-dsc-getting-started.md#enable-an-azure-resource-manager-vm-for-management-with-state-configuration) à la configuration. En moins de 15 minutes, le nœud vérifie sa configuration et l’agent est poussé vers le nœud.

## <a name="verify-agent-connectivity-to-azure-monitor"></a>Vérifier la connectivité de l’agent à Azure Monitor

À l’issue de l’installation de l’agent, vous pouvez vérifier qu’il est bien connecté et qu’il rend compte correctement de deux façons différentes.  

À partir de l’ordinateur, dans le **Panneau de configuration**, recherchez l’élément **Microsoft Monitoring Agent**.  Sélectionnez-le. Sous l'onglet **Azure Log Analytics**, l’agent doit afficher le message suivant : **Microsoft Monitoring Agent s’est connecté au service Microsoft Operations Management Suite**.<br><br> ![État de la connexion MMA à Log Analytics](media/agent-windows/log-analytics-mma-laworkspace-status.png)

Vous pouvez également effectuer une recherche simple dans les journaux dans le portail Azure.  

1. Dans le portail Azure, recherchez et sélectionnez **Monitor**.
1. Sélectionnez **Journaux** dans le menu.
1. Dans le volet **Journaux**, dans le champ de requête, tapez :  

    ```
    Heartbeat 
    | where Category == "Direct Agent" 
    | where TimeGenerated > ago(30m)  
    ```

Dans les résultats de recherche retournés, vous devez voir des enregistrements de pulsation pour l’ordinateur, indiquant qu’il est connecté et qu’il rend compte au service.

## <a name="cache-information"></a>Informations sur le cache

Les données de l’agent de Log Analytics sont mises en cache sur l’ordinateur local à l’emplacement *C:\Program Files\Microsoft Monitoring Agent\Agent\Health service State* avant leur envoi à Azure Monitor. L’agent tente de charger toutes les 20 secondes. En cas d’échec, il met un temps de plus en plus long pour réussir. Il attend 30 secondes avant la deuxième tentative, 60 secondes avant la suivante, puis 120 secondes, et ainsi de suite jusqu’à un maximum de 8,5 heures entre les nouvelles tentatives avant que la reconnexion aboutisse. Ce temps d’attente est rendu légèrement aléatoire pour éviter que tous les agents tentent de se connecter simultanément. Les données les plus anciennes sont abandonnées lorsque la taille maximale de la mémoire tampon est atteinte.

La taille de cache par défaut est de 50 Mo, mais vous pouvez configurer une taille comprise entre 5 Mo et 1,5 Go. Elle est stockée dans la clé de registre *HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\HealthService\Parameters\Persistence Cache Maximum*. La valeur représente le nombre de pages, avec 8 Ko par page.


## <a name="next-steps"></a>Étapes suivantes

- Consultez [Gestion et maintenance de l’agent Log Analytics pour Windows et Linux](agent-manage.md) pour en savoir plus sur la manière de reconfigurer, mettre à niveau ou supprimer l’agent de la machine virtuelle.

- Consultez [Résolution des problèmes de l’agent Windows](agent-windows-troubleshoot.md) si vous rencontrez des problèmes lors de l’installation ou de la gestion de l’agent.
