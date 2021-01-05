---
title: Gestion de l’agent Azure Arc enabled servers
description: Cet article décrit les différentes tâches de gestion à effectuer en règle générale pendant le cycle de vie de l’agent Connected Machine Azure Arc enabled servers.
ms.date: 10/30/2020
ms.topic: conceptual
ms.openlocfilehash: 9e17bf58d1e94b64d1cdc6ff0b57b1b6a81be180
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97107190"
---
# <a name="managing-and-maintaining-the-connected-machine-agent"></a>Gestion et maintenance de l’agent Connected Machine

Après le déploiement initial de l’agent Machine connectée Azure Arc enabled servers pour Windows ou Linux, vous devrez peut-être le reconfigurer, le mettre à niveau ou le supprimer de l’ordinateur. Vous pouvez facilement effectuer ces tâches de maintenance de routine manuellement ou automatiquement ce qui réduit les erreurs opérationnelles et les coûts.

## <a name="before-uninstalling-agent"></a>Avant de désinstaller l’agent

Avant de supprimer l’agent Connected Machine Azure Arc enabled servers, tenez compte des points suivants afin d’éviter des problèmes inattendus ou des coûts supplémentaires sur votre facture Azure :

* Si vous avez déployé des extensions de machine virtuelle Azure sur un serveur avec Arc et que vous supprimez l’agent Connected Machine ou la ressource représentant le serveur avec Arc dans le groupe de ressources, ces extensions continuent de s’exécuter et de fonctionner normalement.

* Si vous supprimez la ressource représentant le serveur avec Arc dans le groupe de ressources, mais que vous ne désinstallez pas les extensions de machine virtuelle, lorsque vous réinscrirez la machine, vous ne pourrez pas gérer les extensions de machine virtuelle installées.

Pour les serveurs ou les machines que vous ne souhaitez plus gérer avec Azure Arc enabled servers, vous devez effectuer les étapes ci-dessous pour arrêter la gestion :

1. Supprimez les extensions de machine virtuelle qui sont installées sur la machine ou le serveur. Les étapes sont détaillées ci-dessous.

2. Déconnectez la machine d’Azure Arc en choisissant l’une des méthodes suivantes :

    * Exécutez la commande `azcmagent disconnect` sur la machine ou le serveur.

    * Sur le serveur Arc inscrit que vous avez sélectionné dans le portail Azure, sélectionnez **Supprimer** dans la barre supérieure.

    * Utilisez [Azure CLI](../../azure-resource-manager/management/delete-resource-group.md?tabs=azure-cli#delete-resource) ou [Azure PowerShell](../../azure-resource-manager/management/delete-resource-group.md?tabs=azure-powershell#delete-resource). Pour le paramètre`ResourceType`, utilisez `Microsoft.HybridCompute/machines`.

3. Désinstallez l’agent de la machine ou du serveur. Suivez les étapes ci-dessous.

## <a name="upgrading-agent"></a>Mise à niveau de l’agent

Azure Connected Machine Agent est régulièrement mis à jour pour appliquer des correctifs de bogues, des améliorations de la stabilité et de nouvelles fonctionnalités. [Azure Advisor](../../advisor/advisor-overview.md) identifie les ressources qui n’utilisent pas la dernière version de l’agent de machine et vous recommande d’effectuer une mise à niveau vers la dernière version. Il vous avertit quand vous sélectionnez le serveur Arc en présentant une bannière dans la page **Vue d’ensemble** ou quand vous accédez à Advisor par le biais du portail Azure.

L’agent Azure Connected Machine pour Windows et Linux peut être mis à niveau vers la dernière version de façon manuelle ou automatique selon vos besoins.

Le tableau suivant décrit les méthodes prises en charge pour effectuer la mise à niveau de l’agent.

| Système d’exploitation | Méthode de mise à niveau |
|------------------|----------------|
| Windows | Manuellement<br> Windows Update |
| Ubuntu | [Apt](https://help.ubuntu.com/lts/serverguide/apt.html) |
| SUSE Linux Enterprise Server | [zypper](https://en.opensuse.org/SDB:Zypper_usage_11.3) |
| RedHat Enterprise, Amazon, CentOS Linux | [yum](https://wiki.centos.org/PackageManagement/Yum) |

### <a name="windows-agent"></a>Agent Windows

Le package de mise à jour de l’agent Connected Machine pour Windows est disponible sur :

* Microsoft Update

* [Catalogue Microsoft Update](https://www.catalog.update.microsoft.com/Home.aspx)

* [Package Windows Installer de l’agent pour Windows](https://aka.ms/AzureConnectedMachineAgent) à partir du centre de téléchargement Microsoft.

L’agent peut être mis à jour à l’aide de différentes méthodes pour prendre en charge votre processus de gestion des mises à jour logicielles. À part l’obtention depuis Microsoft Update, vous pouvez également procéder au téléchargement et à l’exécution manuels à partir de l’invite de commandes, d’un script ou d’une autre solution d’automatisation, ou à partir de l’Assistant interface utilisateur en exécutant `AzureConnectedMachine.msi`.

> [!NOTE]
> * Pour mettre à niveau l’agent, vous devez disposer d’autorisations *Administrateur*.
> * Pour mettre à niveau manuellement, vous devez tout d’abord télécharger et copier le package Installer dans un dossier sur le serveur cible, ou à partir d’un dossier réseau partagé. 

Si vous n’êtes pas familiarisé avec les options de ligne de commande pour les packages Windows Installer, passez en revue [Options de ligne de commande standard Msiexec](/windows/win32/msi/standard-installer-command-line-options) et [Options de ligne de commande Msiexec](/windows/win32/msi/command-line-options).

#### <a name="to-upgrade-using-the-setup-wizard"></a>Pour effectuer la mise à niveau à l’aide de l’Assistant d’installation

1. Connectez-vous à la machine avec un compte disposant des droits d’administration.

2. Exécutez **AzureConnectedMachineAgent.msi** pour démarrer l’Assistant Installation.

L’Assistant Installation détecte si une version précédente existe, puis effectue automatiquement une mise à niveau de l’agent. Une fois la mise à niveau terminée, l’Assistant Installation se ferme automatiquement.

#### <a name="to-upgrade-from-the-command-line"></a>Pour effectuer la mise à niveau à l’aide de la ligne de commande

1. Connectez-vous à la machine avec un compte disposant des droits d’administration.

2. Pour mettre à niveau l’agent en mode silencieux et créer un fichier journal d’installation dans le dossier `C:\Support\Logs`, exécutez la commande suivante.

    ```dos
    msiexec.exe /i AzureConnectedMachineAgent.msi /qn /l*v "C:\Support\Logs\Azcmagentupgradesetup.log"
    ```

### <a name="linux-agent"></a>Agent Linux

La mise à jour de l’agent vers la dernière version sur un ordinateur Linux implique deux commandes. Une commande pour mettre à jour l’index de package local avec la liste des derniers packages disponibles dans les référentiels, et une commande pour mettre à niveau le package local.

Vous pouvez télécharger le package d’agent le plus récent à partir du [référentiel de packages](https://packages.microsoft.com/) de Microsoft.

> [!NOTE]
> Pour mettre à niveau l’agent, vous devez disposer d’autorisations d’accès *racine* ou d’un compte avec des droits élevés utilisant Sudo.

#### <a name="upgrade-ubuntu"></a>Mettre à niveau Ubuntu

1. Pour mettre à jour l’index de package local avec les dernières modifications apportées aux référentiels, exécutez la commande suivante :

    ```bash
    apt update
    ```

2. Pour mettre à niveau votre système, exécutez la commande suivante :

    ```bash
    apt upgrade
    ```

Les actions de la commande [apt](https://help.ubuntu.com/lts/serverguide/apt.html), telles que l’installation et la suppression de packages, sont consignées dans le fichier journal `/var/log/dpkg.log`.

#### <a name="upgrade-red-hatcentosamazon-linux"></a>Mettre à niveau Red Hat/CentOS/Amazon Linux

1. Pour mettre à jour l’index de package local avec les dernières modifications apportées aux référentiels, exécutez la commande suivante :

    ```bash
    yum check-update
    ```

2. Pour mettre à niveau votre système, exécutez la commande suivante :

    ```bash
    yum update
    ```

Les actions de la commande [yum](https://access.redhat.com/articles/yum-cheat-sheet), telles que l’installation et la suppression de packages, sont consignées dans le fichier journal `/var/log/yum.log`. 

#### <a name="upgrade-suse-linux-enterprise"></a>Mettre à niveau SUSE Linux Enterprise

1. Pour mettre à jour l’index de package local avec les dernières modifications apportées aux référentiels, exécutez la commande suivante :

    ```bash
    zypper refresh
    ```

2. Pour mettre à niveau votre système, exécutez la commande suivante :

    ```bash
    zypper update
    ```

Les actions de la commande [zypper](https://en.opensuse.org/Portal:Zypper), telles que l’installation et la suppression de packages, sont consignées dans le fichier journal `/var/log/zypper.log`.

## <a name="about-the-azcmagent-tool"></a>À propos de l’outil Azcmagent

L’outil Azcmagent (Azcmagent.exe) sert à configurer l’agent Connected Machine Azure Arc enabled servers pendant l’installation, ou à modifier la configuration initiale de l’agent après installation. Azcmagent.exe fournit des paramètres de ligne de commande pour personnaliser l’agent et afficher son état :

* **Connect** : pour connecter la machine à Azure Arc.

* **Disconnect** : pour déconnecter la machine d’Azure Arc.

* **Show** : afficher l’état et les propriétés de configuration de l’agent (nom du groupe de ressources, ID d’abonnement, version, etc.), qui peuvent être utiles lors de la résolution d’un problème avec l’agent. Incluez le paramètre `-j` pour générer les résultats au format JSON.

* **Logs**  : crée un fichier. zip dans le répertoire actif contenant les journaux pour vous aider lors de la résolution des problèmes.

* **Version** : montre la version de l’agent Connected Machine.

* **-h ou--Help** : afficher les paramètres de ligne de commande disponibles.

    Par exemple, pour obtenir une aide détaillée sur le paramètre **Connect**, tapez `azcmagent connect -h`. 

* **-v ou--verbose** : activer la journalisation détaillée.

Vous pouvez effectuer une opération **Connect** et **Disconnect** manuellement quand vous êtes connecté de manière interactive, l’automatiser en utilisant le principal de service utilisé pour intégrer plusieurs agents ou avec un [jeton d’accès](../../active-directory/develop/access-tokens.md) de plateforme d’identité Microsoft. Si vous n’avez pas utilisé de principal de service pour inscrire la machine auprès d’Azure Arc enabled servers, consultez l’[article](onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale) suivant pour créer un principal de service.

>[!NOTE]
>Vous devez disposer des autorisations d’accès *racine* sur les ordinateurs Linux pour exécuter **azcmagent**.

### <a name="connect"></a>Se connecter

Ce paramètre spécifie une ressource dans Azure Resource Manager représentant la machine créée dans Azure. La ressource se trouve dans l’abonnement et le groupe de ressources spécifiés, et les données relatives à la machine sont stockées dans la région Azure spécifiée par le paramètre `--location`. Le nom de ressource par défaut est le nom d’hôte de la machine s’il n’est pas spécifié.

Un certificat correspondant à l’identité attribuée par le système de cet machine est ensuite téléchargé, puis stocké localement. Après cette étape, l’instance Metadata Service d’Azure Connected Machine et l’agent Guest Configuration commencent la synchronisation avec Azure Arc enabled servers.

Pour vous connecter à un principal de service, exécutez la commande suivante :

`azcmagent connect --service-principal-id <serviceprincipalAppID> --service-principal-secret <serviceprincipalPassword> --tenant-id <tenantID> --subscription-id <subscriptionID> --resource-group <ResourceGroupName> --location <resourceLocation>`

Pour vous connecter à l’aide d’un jeton d’accès, exécutez la commande suivante :

`azcmagent connect --access-token <> --subscription-id <subscriptionID> --resource-group <ResourceGroupName> --location <resourceLocation>`

Pour vous connecter à l’aide de vos informations d’identification d’ouverture de session avec privilèges élevés (interactives), exécutez la commande suivante :

`azcmagent connect --tenant-id <TenantID> --subscription-id <subscriptionID> --resource-group <ResourceGroupName> --location <resourceLocation>`

### <a name="disconnect"></a>Déconnecter

Ce paramètre spécifie une ressource dans Azure Resource Manager, indiquant que la machine est supprimée dans Azure. Il ne supprime pas l’agent de la machine. Cette opération doit être effectuée en tant qu’étape distincte. Une fois la machine déconnectée, si vous souhaitez la réinscrire auprès d’Azure Arc enabled servers, utilisez la commande `azcmagent connect` afin qu’une nouvelle ressource soit créée pour elle dans Azure.

> [!NOTE]
> Si vous avez déployé une ou plusieurs extensions de machine virtuelle Azure sur votre instance Azure Arc enabled servers et que vous désinscrivez la machine dans Azure, les extensions restent installées. Il est important de comprendre que, en fonction de l’extension installée, elle est exécutée activement. Avant de désinscrire dans Azure une machine que vous souhaitez mettre hors service ou ne plus gérer via Arc enabled servers, vous devez supprimer les extensions qui y sont installées.

Pour vous déconnecter à l’aide d’un principal du service, exécutez la commande suivante :

`azcmagent disconnect --service-principal-id <serviceprincipalAppID> --service-principal-secret <serviceprincipalPassword> --tenant-id <tenantID>`

Pour vous déconnecter à l’aide d’un jeton d’accès, exécutez la commande suivante :

`azcmagent disconnect --access-token <accessToken>`

Pour vous déconnecter à l’aide de vos informations d’identification d’ouverture de session avec privilèges élevés (interactives), exécutez la commande suivante :

`azcmagent disconnect`

## <a name="remove-the-agent"></a>Supprimer l’agent

Utilisez l’une des méthodes suivantes pour désinstaller l’agent Connected Machine Windows ou Linux de l’ordinateur. La suppression de l’agent n’entraîne pas la désinscription de la machine auprès d’Azure Arc enabled servers ni la suppression des extensions de machine virtuelle Azure installées. Vous devez effectuer ces étapes séparément quand vous n’avez plus besoin de gérer la machine dans Azure, et préalablement à la désinstallation de l’agent.

### <a name="windows-agent"></a>Agent Windows

Les deux méthodes suivantes suppriment l’agent, mais pas le dossier *C:\Program Files\AzureConnectedMachineAgent* sur l’ordinateur.

#### <a name="uninstall-from-control-panel"></a>Désinstaller à partir du Panneau de configuration

1. Pour désinstaller l’agent Windows de la machine, procédez comme suit :

    a. Connectez-vous à la machine avec un compte disposant des autorisations d’administration.  
    b. Dans **Panneau de configuration**, sélectionnez **Programmes et fonctionnalités**.  
    c. Dans **Programmes et fonctionnalités**, sélectionnez **Agent Azure Connected Machine Agent**, **Désinstaller**, puis **Oui**.  

    >[!NOTE]
    > Vous pouvez également exécuter l’Assistant Installation de l’agent en double-cliquant sur le package d’installation **AzureConnectedMachineAgent.msi**.

#### <a name="uninstall-from-the-command-line"></a>Désinstaller à partir de la ligne de commande

Pour désinstaller l’agent manuellement à partir de l’invite de commandes ou pour utiliser une méthode automatisée, telle qu’un script, vous pouvez utiliser l’exemple suivant. Tout d’abord, vous devez récupérer le code du produit, c’est-à-dire un GUID qui est l’identificateur principal du package d’application, à partir du système d’exploitation. La désinstallation est effectuée à l’aide de la ligne de commande Msiexec.exe - `msiexec /x {Product Code}`.

1. Ouvrez l’Éditeur du Registre.

2. Sous la clé de Registre `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Uninstall`, recherchez et copiez le GUID du code du produit.

3. Vous pouvez alors désinstaller l’agent à l’aide de Msiexec en utilisant les exemples suivants :

   * À partir du type de la ligne de commande :

       ```dos
       msiexec.exe /x {product code GUID} /qn
       ```

   * Vous pouvez effectuer les mêmes étapes en utilisant PowerShell :

       ```powershell
       Get-ChildItem -Path HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall | `
       Get-ItemProperty | `
       Where-Object {$_.DisplayName -eq "Azure Connected Machine Agent"} | `
       ForEach-Object {MsiExec.exe /x "$($_.PsChildName)" /qn}
       ```

### <a name="linux-agent"></a>Agent Linux

> [!NOTE]
> Pour désinstaller l’agent, vous devez disposer d’autorisations d’accès *racine* ou d’un compte avec des droits élevés utilisant Sudo.

Pour désinstaller l’agent Linux, la commande à utiliser dépend du système d’exploitation Linux.

- Pour Ubuntu, exécutez la commande suivante :

    ```bash
    sudo apt purge azcmagent
    ```

- Pour RHEL, CentOS et Amazon Linux, exécutez la commande suivante :

    ```bash
    sudo yum remove azcmagent
    ```

- Pour SLES, exécutez la commande suivante :

    ```bash
    sudo zypper remove azcmagent
    ```

## <a name="unregister-machine"></a>Désinscrire l’ordinateur

Si vous prévoyez d’arrêter la gestion de la machine avec les services de prise en charge dans Azure, effectuez les étapes suivantes pour désinscrire la machine auprès d’Arc enabled servers. Vous pouvez suivre ces étapes avant ou après la suppression de l’agent Connected Machine de la machine.

1. Ouvrez Azure Arc enabled servers à partir du [portail Azure](https://aka.ms/hybridmachineportal).

2. Sélectionnez la machine dans la liste, sélectionnez les points de suspension ( **...** ), puis sélectionnez **Supprimer**.

## <a name="update-or-remove-proxy-settings"></a>Mettre à jour ou supprimer des paramètres de proxy

Pour configurer l’agent afin qu’il communique avec le service via un serveur proxy ou pour supprimer cette configuration après le déploiement, utilisez l’une des méthodes suivantes.

### <a name="windows"></a>Windows

Pour définir la variable d’environnement du serveur proxy, exécutez la commande suivante :

```powershell
# If a proxy server is needed, execute these commands with the proxy URL and port.
[Environment]::SetEnvironmentVariable("https_proxy","http://{proxy-url}:{proxy-port}","Machine")
$env:https_proxy = [System.Environment]::GetEnvironmentVariable("https_proxy","Machine")
# For the changes to take effect, the agent service needs to be restarted after the proxy environment variable is set.
Restart-Service -Name himds
```

Pour configurer l’agent afin qu’il cesse de communiquer via un serveur proxy, exécutez la commande suivante pour supprimer la variable d’environnement du serveur proxy et redémarrer le service de l’agent :

```powershell
[Environment]::SetEnvironmentVariable("https_proxy",$null,"Machine")
$env:https_proxy = [System.Environment]::GetEnvironmentVariable("https_proxy","Machine")
# For the changes to take effect, the agent service needs to be restarted after the proxy environment variable removed.
Restart-Service -Name himds
```

### <a name="linux"></a>Linux

Pour définir le serveur proxy, exécutez la commande suivante à partir du répertoire dans lequel vous avez téléchargé le package d’installation de l’agent :

```bash
# Reconfigure the connected machine agent and set the proxy server.
bash ~/Install_linux_azcmagent.sh --proxy "{proxy-url}:{proxy-port}"
```

Pour configurer l’agent afin qu’il cesse de communiquer via un serveur proxy, exécutez la commande suivante pour supprimer la configuration du proxy :

```bash
sudo azcmagent_proxy remove
```

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la résolution des problèmes, consultez le [guide Résoudre les problèmes de l’agent Connected Machine](troubleshoot-agent-onboard.md).

* Apprenez à gérer votre machine à l’aide d’[Azure Policy](../../governance/policy/overview.md), par exemple pour la [configuration invité](../../governance/policy/concepts/guest-configuration.md) des machines virtuelles, pour vérifier que l’ordinateur crée des rapports sur l’espace de travail Log Analytics prévu, pour activer l’analyse d’[Azure Monitor sur des machines virtuelles](../../azure-monitor/insights/vminsights-enable-policy.md) et bien plus encore.

* En savoir plus sur [l’agent Log Analytics](../../azure-monitor/platform/log-analytics-agent.md). L’agent Log Analytics pour Windows et Linux est nécessaire quand vous souhaitez collecter des données de supervision du système d’exploitation et de la charge de travail, gérer les ressources à l’aide de runbooks Automation ou de fonctionnalités comme Update Management, ou utiliser d’autres services Azure comme [Azure Security Center](../../security-center/security-center-introduction.md).