---
title: Gestion de l’agent Azure Log Analytics
description: Cet article décrit les différentes tâches de gestion à effectuer en règle générale pendant le cycle de vie de l’agent Log Analytics Windows ou Linux déployé sur une machine.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 06/14/2019
ms.openlocfilehash: 4d0ceacd37748e9761903d02fd7e052d70b10e15
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011878"
---
# <a name="managing-and-maintaining-the-log-analytics-agent-for-windows-and-linux"></a>Gestion et maintenance de l’agent Log Analytics sous Windows et Linux

Après le déploiement initial de l’agent Log Analytics Windows ou Linux dans Azure Monitor, il se peut que vous deviez le reconfigurer, le mettre à niveau ou le supprimer de la machine s’il a atteint la phase de mise hors service dans son cycle de vie. Vous pouvez facilement effectuer ces tâches de maintenance de routine manuellement ou automatiquement ce qui réduit les erreurs opérationnelles et les coûts.

## <a name="upgrading-agent"></a>Mise à niveau de l’agent

L’agent Log Analytics pour Windows et Linux peut être mis à niveau vers la dernière version manuellement ou automatiquement en fonction du scénario de son déploiement et de l’environnement dans lequel la machine virtuelle s’exécute. Les méthodes suivantes peuvent être utilisés pour mettre l’agent à niveau.

| Environnement | Méthode d’installation | Méthode de mise à niveau |
|--------|----------|-------------|
| Azure VM | Extension de machine virtuelle d’agent Log Analytics pour Windows/Linux | L’agent est automatiquement mis à niveau par défaut, sauf si vous avez configuré votre modèle Azure Resource Manager pour la désactiver en définissant la valeur de la propriété *autoUpgradeMinorVersion* sur **false**. |
| Images de machines virtuelles Azure personnalisées | Installation manuelle de l’agent Log Analytics pour Windows ou Linux | La mise à jour des machines virtuelles vers la dernière version de l’agent doit être effectuée à partir de la ligne de commande avec exécution du package d’installation de Windows ou de Linux auto-extractible et du package de script d’installation.|
| Machines virtuelles non Azure | Installation manuelle de l’agent Log Analytics pour Windows ou Linux | La mise à jour des machines virtuelles vers la dernière version de l’agent doit être effectuée à partir de la ligne de commande avec exécution du package d’installation de Windows ou de Linux auto-extractible et du package de script d’installation. |

### <a name="upgrade-windows-agent"></a>Mise à niveau de l’agent Windows 

Pour mettre à jour l’agent vers la dernière version sur une machine virtuelle Windows qui n’a pas été installée à l’aide de l’extension de machine virtuelle Log Analytics, vous l’exécutez à partir de l’invite de commandes, d’un script ou d’une autre solution d’automatisation, ou bien à l’aide de l’Assistant d’installation MMASetup-\<platform\>.msi.  

Vous pouvez télécharger la dernière version de l’agent Windows sur votre espace de travail Log Analytics, en effectuant les opérations suivantes.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Dans le portail Azure, cliquez sur **Tous les services**. Dans la liste de ressources, saisissez **Log Analytics**. Au fur et à mesure de la saisie, la liste est filtrée. Sélectionnez **Espaces de travail Log Analytics**.

3. Dans votre liste d’espaces de travail Log Analytics, sélectionnez l’espace de travail.

4. Dans votre espace de travail Log Analytics, sélectionnez **Paramètres avancés**, puis sélectionnez **Sources connectées** et enfin **Serveurs Windows**.

5. Dans la page **Serveurs Windows**, sélectionnez l’option **Télécharger l’agent Windows** correspondant à la version appropriée en fonction de l’architecture du processeur du système d’exploitation Windows.

>[!NOTE]
>Pendant la mise à niveau de l’agent Log Analytics pour Windows, il n’est pas possible de configurer ni de reconfigurer l’espace de travail auquel il doit faire référence. Pour configurer l’agent, vous devez suivre une des méthodes prises en charge et répertoriés sous [Ajout ou suppression d’un espace de travail](#adding-or-removing-a-workspace).
>

#### <a name="to-upgrade-using-the-setup-wizard"></a>Pour effectuer la mise à niveau à l’aide de l’Assistant d’installation

1. Connectez-vous à la machine avec un compte disposant des droits d’administration.

2. Exécutez **MMASetup-\<platform\>.exe** pour démarrer l’Assistant Installation.

3. Sur la première page de l’Assistant d’installation, cliquez sur **Suivant**.

4. Dans la boîte de dialogue **Installation de Microsoft Monitoring Agent**, cliquez sur **J’accepte** pour accepter le contrat de licence.

5. Dans la boîte de dialogue **Installation de Microsoft Monitoring Agent**, cliquez sur **Mettre à niveau**. La page d’état affiche la progression de la mise à niveau.

6. Lorsque la page **Configuration de Microsoft Monitoring Agent terminée** s’affiche, cliquez sur **Terminer**.

#### <a name="to-upgrade-from-the-command-line"></a>Pour effectuer la mise à niveau à l’aide de la ligne de commande

1. Connectez-vous à la machine avec un compte disposant des droits d’administration.

2. Pour extraire les fichiers d’installation de l’agent, à partir d’une invite de commandes avec élévation de privilèges, exécutez `MMASetup-<platform>.exe /c` et indiquez l’emplacement où extraire les fichiers. L’autre possibilité consiste à spécifier le chemin d’accès à l’aide des arguments `MMASetup-<platform>.exe /c /t:<Full Path>`.

3. Exécutez la commande suivante, où D:\ est l’emplacement du fichier journal de mise à niveau.

    ```dos
    setup.exe /qn /l*v D:\logs\AgentUpgrade.log AcceptEndUserLicenseAgreement=1
    ```

### <a name="upgrade-linux-agent"></a>Mise à niveau de l’agent Linux 

Il est possible d’effectuer la mise à niveau à partir des versions antérieures (> 1.0.0-47). Effectuer l’installation avec la commande `--upgrade` met à niveau tous les composants de l’agent vers la dernière version.

Pour mettre à niveau l’agent, exécutez la commande suivante.

`sudo sh ./omsagent-*.universal.x64.sh --upgrade`

## <a name="adding-or-removing-a-workspace"></a>Ajout ou suppression d’un espace de travail

### <a name="windows-agent"></a>Agent Windows
Les étapes décrites dans cette section sont nécessaires lorsque vous souhaitez non seulement reconfigurer l’agent Windows pour qu’il utilise un autre espace de travail ou supprimer un espace de travail de sa configuration, mais également lorsque vous souhaitez configurer l’agent pour qu’il utilise plus d’un espace de travail (généralement appelée hébergement multiple). La configuration de l’agent Windows pour qu’il utilise plusieurs espaces de travail ne peut être effectuée après l’installation initiale de l’agent et à l’aide des méthodes décrites ci-dessous.    

#### <a name="update-settings-from-control-panel"></a>Mettre à jour les paramètres dans le Panneau de configuration

1. Connectez-vous à la machine avec un compte disposant des droits d’administration.

2. Ouvrez le **Panneau de configuration**.

3. Sélectionnez **Microsoft Monitoring Agent**, puis cliquez sur l’onglet **Azure Log Analytics**.

4. Pour supprimer un espace de travail, sélectionnez-le puis cliquez sur **Supprimer**. Répétez cette étape pour le ou les autres espaces de travail auxquels l’agent ne doit plus communiquer d’informations.

5. Pour ajouter un espace de travail, cliquez sur **Ajouter**, puis dans la boîte de dialogue **Ajouter un espace de travail Log Analytics**, collez l’ID et la clé de l’espace de travail (clé primaire). Si la machine doit communiquer avec un espace de travail Log Analytics dans le cloud Azure Government, sélectionnez Azure - Gouvernement des États-Unis dans la liste déroulante Cloud Azure.

6. Cliquez sur **OK** pour enregistrer vos modifications.

#### <a name="remove-a-workspace-using-powershell"></a>Supprimer un espace de travail à l’aide de PowerShell

```powershell
$workspaceId = "<Your workspace Id>"
$mma = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'
$mma.RemoveCloudWorkspace($workspaceId)
$mma.ReloadConfiguration()
```

#### <a name="add-a-workspace-in-azure-commercial-using-powershell"></a>Ajouter un espace de travail dans Azure commercial à l’aide de PowerShell

```powershell
$workspaceId = "<Your workspace Id>"
$workspaceKey = "<Your workspace Key>"
$mma = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'
$mma.AddCloudWorkspace($workspaceId, $workspaceKey)
$mma.ReloadConfiguration()
```

#### <a name="add-a-workspace-in-azure-for-us-government-using-powershell"></a>Ajouter un espace de travail dans Azure pour US Government à l’aide de PowerShell

```powershell
$workspaceId = "<Your workspace Id>"
$workspaceKey = "<Your workspace Key>"
$mma = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'
$mma.AddCloudWorkspace($workspaceId, $workspaceKey, 1)
$mma.ReloadConfiguration()
```

>[!NOTE]
>Si vous avez utilisé la ligne de commande ou un script pour installer ou configurer l’agent, `EnableAzureOperationalInsights` a été remplacé par `AddCloudWorkspace` et `RemoveCloudWorkspace`.
>

### <a name="linux-agent"></a>Agent Linux
Les étapes suivantes montrent comment reconfigurer l’agent Linux si vous décidez de l’enregistrer avec un autre espace de travail ou si vous souhaitez supprimer un espace de travail à partir de sa configuration.

1. Pour vérifier qu’il est enregistré dans un espace de travail, exécutez la commande suivante :

    `/opt/microsoft/omsagent/bin/omsadmin.sh -l`

    Cela renvoie un état similaire à l’exemple suivant :

    `Primary Workspace: <workspaceId>   Status: Onboarded(OMSAgent Running)`

    Il est important que l’état indique également que l’agent est en cours d’exécution, sinon les étapes suivantes visant à reconfigurer l’agent ne s’achèvent pas correctement.

2. S’il est déjà inscrit auprès d’un espace de travail, supprimez l’espace de travail enregistré en exécutant la commande suivante. Sinon, s’il n’est pas inscrit, passez à l’étape suivante.

    `/opt/microsoft/omsagent/bin/omsadmin.sh -X`

3. Pour inscrire un autre espace de travail, exécutez la commande suivante :

    `/opt/microsoft/omsagent/bin/omsadmin.sh -w <workspace id> -s <shared key> [-d <top level domain>]`
    
4. Pour vérifier que vos modifications sont effectives, exécutez la commande suivante :

    `/opt/microsoft/omsagent/bin/omsadmin.sh -l`

    Cela renvoie un état similaire à l’exemple suivant :

    `Primary Workspace: <workspaceId>   Status: Onboarded(OMSAgent Running)`

Le service de l’agent n’a pas besoin être redémarré pour que les modifications prennent effet.

## <a name="update-proxy-settings"></a>Mettre à jour les paramètres de proxy
Pour permettre à l’agent de communiquer avec le service via un serveur proxy ou une [passerelle Log Analytics](gateway.md) après le déploiement, utilisez l’une des méthodes suivantes.

### <a name="windows-agent"></a>Agent Windows

#### <a name="update-settings-using-control-panel"></a>Mettre à jour les paramètres dans le Panneau de configuration

1. Connectez-vous à la machine avec un compte disposant des droits d’administration.

2. Ouvrez le **Panneau de configuration**.

3. Sélectionnez **Microsoft Monitoring Agent** puis cliquez sur l’onglet **Paramètres proxy**.

4. Cliquez sur **Utiliser un serveur proxy** et indiquez l’URL et le numéro de port du serveur proxy ou de la passerelle. Si votre serveur proxy ou passerelle Log Analytics requiert une authentification, tapez le nom d’utilisateur et un mot de passe pour vous authentifier, puis cliquez sur **OK**.

#### <a name="update-settings-using-powershell"></a>Mettre à jour les paramètres à l’aide de PowerShell

Copiez l’exemple de code PowerShell suivant, mettez-le à jour avec les informations propres à votre environnement et enregistrez-le avec une extension de fichier PS1. Exécutez le script sur chaque machine qui se connecte directement à l’espace de travail Log Analytics dans Azure Monitor.

```powershell
param($ProxyDomainName="https://proxy.contoso.com:30443", $cred=(Get-Credential))

# First we get the Health Service configuration object. We need to determine if we
#have the right update rollup with the API we need. If not, no need to run the rest of the script.
$healthServiceSettings = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'

$proxyMethod = $healthServiceSettings | Get-Member -Name 'SetProxyInfo'

if (!$proxyMethod)
{
    Write-Output 'Health Service proxy API not present, will not update settings.'
    return
}

Write-Output "Clearing proxy settings."
$healthServiceSettings.SetProxyInfo('', '', '')

$ProxyUserName = $cred.username

Write-Output "Setting proxy to $ProxyDomainName with proxy username $ProxyUserName."
$healthServiceSettings.SetProxyInfo($ProxyDomainName, $ProxyUserName, $cred.GetNetworkCredential().password)
```

### <a name="linux-agent"></a>Agent Linux
Effectuez les opérations suivantes si vos ordinateurs Linux doivent communiquer via un serveur proxy ou une passerelle Log Analytics. La valeur de configuration de proxy a la syntaxe suivante `[protocol://][user:password@]proxyhost[:port]`. La propriété *proxyhost* accepte un nom de domaine complet ou l’adresse IP du serveur proxy.

1. Modifiez le fichier `/etc/opt/microsoft/omsagent/proxy.conf` en exécutant les commandes suivantes et modifiez les valeurs en vous basant sur vos paramètres spécifiques.

    ```
    proxyconf="https://proxyuser:proxypassword@proxyserver01:30443"
    sudo echo $proxyconf >>/etc/opt/microsoft/omsagent/proxy.conf
    sudo chown omsagent:omiusers /etc/opt/microsoft/omsagent/proxy.conf
    ```

2. Redémarrez l’agent en exécutant la commande suivante :

    ```
    sudo /opt/microsoft/omsagent/bin/service_control restart [<workspace id>]
    ```

## <a name="uninstall-agent"></a>Désinstaller l’agent
Utilisez une des procédures suivantes pour désinstaller l’agent Windows ou Linux à l’aide de la ligne de commande ou de l’assistant d’installation.

### <a name="windows-agent"></a>Agent Windows

#### <a name="uninstall-from-control-panel"></a>Désinstaller à partir du Panneau de configuration
1. Connectez-vous à la machine avec un compte disposant des droits d’administration.

2. Dans le **Panneau de configuration**, cliquez sur **Programmes et fonctionnalités**.

3. Dans **Programmes et fonctionnalités**, sélectionnez **Microsoft Monitoring Agent**, cliquez sur **Désinstaller** puis sur **Oui**.

>[!NOTE]
>Pour exécuter l’Assistant d’installation de l’agent, vous pouvez aussi double-cliquer sur le fichier **MMASetup-\<platform\>.exe**, disponible en téléchargement à partir d’un espace de travail dans le portail Azure.

#### <a name="uninstall-from-the-command-line"></a>Désinstaller à partir de la ligne de commande
Le fichier téléchargé de l’agent est un package d’installation autonome créé avec IExpress. Le programme d’installation de l’agent et les fichiers de prise en charge sont contenus dans le package et doivent être extraits pour effectuer une désinstallation correcte à l’aide de la ligne de commande indiquée dans l’exemple suivant.

1. Connectez-vous à la machine avec un compte disposant des droits d’administration.

2. Pour extraire les fichiers d’installation de l’agent, à partir d’une invite de commandes avec élévation de privilèges, exécutez `extract MMASetup-<platform>.exe` et indiquez l’emplacement où extraire les fichiers. L’autre possibilité consiste à spécifier le chemin d’accès à l’aide des arguments `extract MMASetup-<platform>.exe /c:<Path> /t:<Path>`. Pour plus d’informations sur les commutateurs de ligne de commande pris en charge par IExpress, consultez [Commutateurs de ligne de commande pour IExpress](https://support.microsoft.com/help/197147/command-line-switches-for-iexpress-software-update-packages) puis mettez à jour l’exemple en fonction de vos besoins.

3. À l’invite, tapez `%WinDir%\System32\msiexec.exe /x <Path>:\MOMAgent.msi /qb`.

### <a name="linux-agent"></a>Agent Linux
Pour supprimer l’agent, exécutez la commande suivante sur l’ordinateur Linux. L’argument *--purge* supprime complètement l’agent et sa configuration.

   `wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh --purge`

## <a name="configure-agent-to-report-to-an-operations-manager-management-group"></a>Configurer l’agent pour qu’il communique avec un groupe d’administration Operations Manager

### <a name="windows-agent"></a>Agent Windows
Procédez comme suit pour que l’agent Log Analytics pour Windows communique avec un groupe d’administration System Center Operations Manager.

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]

1. Connectez-vous à la machine avec un compte disposant des droits d’administration.

2. Ouvrez le **Panneau de configuration**.

3. Cliquez sur **Microsoft Monitoring Agent** puis sur l’onglet **Operations Manager**.

4. Si vos serveurs Operations Manager sont intégrés à Active Directory, cliquez sur **Met à jour automatiquement les attributions du groupe d’administration à partir d’AD DS**.

5. Cliquez sur **Ajouter** pour ouvrir la boîte de dialogue **Ajouter un groupe d’administration**.

6. Dans la zone **Nom du groupe d’administration**, entrez le nom de votre groupe d’administration.

7. Dans le champ **Serveur d’administration principal**, entrez le nom d’ordinateur du serveur d’administration principal.

8. Dans le champ **Port du serveur d’administration**, entrez le numéro du port TCP.

9. Sous **Compte d’action d’agent**, choisissez le compte système local ou un compte de domaine local.

10. Cliquez sur **OK** pour fermer la boîte de dialogue **Ajouter un groupe d’administration**, puis cliquez sur **OK** pour fermer la boîte de dialogue **Propriétés de l’agent Microsoft Monitoring Agent**.

### <a name="linux-agent"></a>Agent Linux
Procédez comme suit pour que l’agent Log Analytics pour Linux communique avec un groupe d’administration System Center Operations Manager.

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]

1. Modifiez le fichier `/etc/opt/omi/conf/omiserver.conf`

2. Vérifiez que la ligne commençant par `httpsport=` spécifie le port 1270. Par exemple : `httpsport=1270`

3. Redémarrez le serveur OMI : `sudo /opt/omi/bin/service_control restart`

## <a name="next-steps"></a>Étapes suivantes

- Consultez [Résolution des problèmes de l’agent Linux](agent-linux-troubleshoot.md) si vous rencontrez des problèmes lors de l’installation ou de gestion de l’agent Linux.

- Consultez [Résolution des problèmes de l’agent Windows](agent-windows-troubleshoot.md) si vous rencontrez des problèmes lors de l’installation ou de gestion de l’agent Windows.
