---
title: Connecter des machines hybrides à Azure à partir du portail Azure
description: Cet article explique comment installer l’agent et connecter des machines à Azure à l’aide d’Azure Arc enabled servers dans le portail Azure.
ms.date: 11/05/2020
ms.topic: conceptual
ms.openlocfilehash: ca3c08acdef1b2a1f7c3774f5755967d472c93ed
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93398026"
---
# <a name="connect-hybrid-machines-to-azure-from-the-azure-portal"></a>Connecter des machines hybrides à Azure à partir du portail Azure

Vous pouvez activer Azure Arc enabled servers pour une seule machine ou un petit nombre de machines Windows ou Linux dans votre environnement en effectuant une série d’étapes manuellement. Vous pouvez également utiliser une méthode automatisée en exécutant un script de modèle que nous fournissons. Ce script automatise le téléchargement et l’installation des deux agents.

Cette méthode nécessite que vous disposiez d’autorisations d’administrateur sur la machine pour installer et configurer l’agent. Sur Linux, vous utilisez le compte root et, sur Windows, vous êtes membre du groupe Administrateurs local.

Avant de commencer, veillez à consulter les [conditions préalables](agent-overview.md#prerequisites) et vérifiez que votre abonnement et vos ressources répondent aux exigences. Pour plus d'informations sur les régions prises en charge et d'autres considérations connexes, consultez [Régions Azure prises en charge](overview.md#supported-regions).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="generate-the-installation-script-from-the-azure-portal"></a>Générer le script d’installation à partir du portail Azure

Le script permettant d’automatiser le téléchargement et l’installation, et d’établir la connexion avec Azure Arc, est disponible sur le portail Azure. Pour effectuer le processus, procédez comme suit :

1. À partir de votre navigateur, accédez au [portail Azure](https://portal.azure.com).

1. Dans la page **Serveurs - Azure Arc**, sélectionnez **Ajouter** en haut à gauche.

1. Dans la page **Sélectionner une méthode**, sélectionnez la vignette **Ajouter des serveurs à l’aide d’un script interactif**, puis sélectionnez **Générer un script**.

1. Dans la page **Générer un script**, sélectionnez l’abonnement et le groupe de ressources où vous souhaitez que la machine soit gérée dans Azure. Sélectionnez une localisation Azure destinée au stockage des métadonnées de la machine. Cet emplacement peut être identique ou différent de l’emplacement du groupe de ressources.

1. Dans la page **Prérequis**, passez en revue les informations, puis sélectionnez **Suivant : Détails des ressources**.

1. Dans la page **Détails des ressources**, spécifiez les informations suivantes :

    1. Dans la liste déroulante **Groupe de ressources**, sélectionnez le groupe de ressources à partir duquel sera gérée la machine.
    1. Dans la liste déroulante **Région**, sélectionnez la région Azure dans laquelle seront stockées les métadonnées des serveurs.
    1. Dans la liste déroulante **Système d’exploitation**, sélectionnez le système d’exploitation sur lequel le script doit s’exécuter.
    1. Si la machine passe par un serveur proxy pour se connecter à Internet, spécifiez l’adresse IP du serveur proxy ou le nom et le numéro de port que la machine doit utiliser pour communiquer avec le serveur proxy. Saisissez la valeur au format `http://<proxyURL>:<proxyport>`.
    1. Sélectionnez **Suivant : Balises**.

1. Dans la page **Balises**, passez en revue les suggestions de **balises d’emplacement physique** par défaut et entrez une valeur, ou spécifiez une ou plusieurs **Balises personnalisées** en fonction de vos standards.

1. Sélectionnez **Suivant : Télécharger et exécuter le script**.

1. Dans la page **Télécharger et exécuter le script**, passez en revue les informations de résumé, puis sélectionnez **Télécharger**. Si vous avez encore besoin d’apporter des modifications, sélectionnez **Précédent**.

## <a name="install-and-validate-the-agent-on-windows"></a>Installer et valider l’agent sur Windows

### <a name="install-manually"></a>Installer manuellement

Vous pouvez installer l’agent Connected Machine manuellement en exécutant le package Windows Installer *AzureConnectedMachineAgent.msi*. Vous pouvez télécharger la dernière version du [package Windows Installer de l’agent pour Windows](https://aka.ms/AzureConnectedMachineAgent) à partir du centre de téléchargement Microsoft.

>[!NOTE]
>* Pour installer ou désinstaller l’agent, vous devez disposer d’autorisations d’*administrateur*.
>* Vous devez tout d’abord télécharger et copier le package Installer dans un dossier sur le serveur cible, ou à partir d’un dossier réseau partagé. Si vous exécutez le package Installer sans aucune option, il démarre un assistant de configuration que vous pouvez suivre pour une installation interactive de l’agent.

Si la machine doit communiquer avec le service via un serveur proxy, après avoir installé l’agent, vous devez exécuter une commande qui est décrite dans les étapes ci-dessous. Cette commande définit la variable d’environnement système `https_proxy` du serveur proxy.

Si vous n’êtes pas familiarisé avec les options de ligne de commande pour les packages Windows Installer, passez en revue [Options de ligne de commande standard Msiexec](/windows/win32/msi/standard-installer-command-line-options) et [Options de ligne de commande Msiexec](/windows/win32/msi/command-line-options).

Par exemple, exécutez le programme d’installation avec le paramètre `/?` pour passer en revue l’option d’aide et de référence rapide.

```dos
msiexec.exe /i AzureConnectedMachineAgent.msi /?
```

1. Pour installer l’agent silencieusement et créer un fichier journal d’installation dans le dossier `C:\Support\Logs` existant, exécutez la commande suivante.

    ```dos
    msiexec.exe /i AzureConnectedMachineAgent.msi /qn /l*v "C:\Support\Logs\Azcmagentsetup.log"
    ```

    Si le démarrage de l’agent échoue une fois l’installation terminée, recherchez des informations détaillées sur l’erreur dans les journaux. Le répertoire des journaux est *%ProgramData%\AzureConnectedMachineAgent\log*.

2. Si la machine doit communiquer via un serveur proxy, exécutez la commande suivante pour définir la variable d’environnement du serveur proxy :

    ```powershell
    [Environment]::SetEnvironmentVariable("https_proxy", "http://{proxy-url}:{proxy-port}", "Machine")
    $env:https_proxy = [System.Environment]::GetEnvironmentVariable("https_proxy","Machine")
    # For the changes to take effect, the agent service needs to be restarted after the proxy environment variable is set.
    Restart-Service -Name himds
    ```

    >[!NOTE]
    >L’agent ne prend pas en charge la configuration de l’authentification proxy dans cette préversion.
    >

3. Après avoir installé l’agent, vous devez le configurer pour qu’il communique avec le service Azure Arc en exécutant la commande suivante :

    ```dos
    "%ProgramFiles%\AzureConnectedMachineAgent\azcmagent.exe" connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID"
    ```

### <a name="install-with-the-scripted-method"></a>Effectuer l’installation à l’aide de la méthode de script

1. Connectez-vous au serveur.

1. Ouvrez une invite de commandes PowerShell avec élévation de privilèges.

    >[!NOTE]
    >Le script ne prend en charge que l’exécution à partir d’une version 64 bits de Windows PowerShell.
    >

1. Accédez au dossier ou au partage sur lequel vous avez copié le script et exécutez-le sur le serveur en exécutant le script `./OnboardingScript.ps1`.

Si le démarrage de l’agent échoue une fois l’installation terminée, recherchez des informations détaillées sur l’erreur dans les journaux. Le répertoire des journaux est *%ProgramData%\AzureConnectedMachineAgent\log*.

## <a name="install-and-validate-the-agent-on-linux"></a>Installer et valider l’agent sur Linux

L’agent Connected Machine pour Linux est fourni dans le format de package par défaut pour la distribution (. RPM ou. DEB) qui est hébergée dans le [dépôt de packages](https://packages.microsoft.com/) de Microsoft. Le [bundle de script shell `Install_linux_azcmagent.sh`](https://aka.ms/azcmagent) effectue les opérations suivantes :

* Configure la machine hôte pour télécharger le package de l’agent à partir de packages.microsoft.com.

* Installe le package du fournisseur de ressources hybrides.

Si vous le souhaitez, vous pouvez configurer l’agent avec vos informations de proxy en incluant le paramètre `--proxy "{proxy-url}:{proxy-port}"`.

En outre, le script contient une logique permettant d’identifier les distributions prises en charge et non prises en charge et il vérifie les autorisations nécessaires pour effectuer l’installation.

L’exemple suivant télécharge l’agent et l’installe :

```bash
# Download the installation package.
wget https://aka.ms/azcmagent -O ~/Install_linux_azcmagent.sh

# Install the connected machine agent.
bash ~/Install_linux_azcmagent.sh
```

1. Pour télécharger et installer l’agent, y compris le paramètre `--proxy` permettant de configurer l’agent afin qu’il communique par le biais de votre serveur proxy, exécutez les commandes suivantes :

    ```bash
    # Download the installation package.
    wget https://aka.ms/azcmagent -O ~/Install_linux_azcmagent.sh

    # Install the connected machine agent.
    bash ~/Install_linux_azcmagent.sh --proxy "{proxy-url}:{proxy-port}"
    ```

2. Après avoir installé l’agent, vous devez le configurer pour qu’il communique avec le service Azure Arc en exécutant la commande suivante :

    ```bash
    azcmagent connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID" --cloud "cloudName"
    if [ $? = 0 ]; then echo "\033[33mTo view your onboarded server(s), navigate to https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.HybridCompute%2Fmachines\033[m"; fi
    ```

### <a name="install-with-the-scripted-method"></a>Effectuer l’installation à l’aide de la méthode de script

1. Connectez-vous au serveur avec un compte disposant d’un accès racine.

1. Accédez au dossier ou au partage sur lequel vous avez copié le script et exécutez-le sur le serveur en exécutant le script `./OnboardingScript.sh`.

Si le démarrage de l’agent échoue une fois l’installation terminée, recherchez des informations détaillées sur l’erreur dans les journaux. Le répertoire des journaux est *var/opt/azcmagent/log*.

## <a name="verify-the-connection-with-azure-arc"></a>Vérifier la connexion avec Azure Arc

Une fois que vous avez installé l’agent et que vous l’avez configuré pour qu’il se connecte à Azure Arc enabled servers, accédez au portail Azure pour vérifier que le serveur s’est correctement connecté. Affichez vos machines dans le [portail Azure](https://aka.ms/hybridmachineportal).

![Connexion au serveur réussie](./media/onboard-portal/arc-for-servers-successful-onboard.png)

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la résolution des problèmes, consultez le [guide Résoudre les problèmes de l’agent Connected Machine](troubleshoot-agent-onboard.md).

* Apprenez à gérer votre machine à l’aide d’[Azure Policy](../../governance/policy/overview.md), par exemple pour la [configuration invité](../../governance/policy/concepts/guest-configuration.md) des machines virtuelles, pour vérifier que l’ordinateur crée des rapports sur l’espace de travail Log Analytics prévu, pour activer l’analyse d’[Azure Monitor sur des machines virtuelles](../../azure-monitor/insights/vminsights-enable-policy.md) et bien plus encore.

* En savoir plus sur [l’agent Log Analytics](../../azure-monitor/platform/log-analytics-agent.md). L’agent Log Analytics pour Windows et Linux est nécessaire quand vous souhaitez collecter des données de supervision du système d’exploitation et de la charge de travail, gérer les ressources à l’aide de runbooks Automation ou de fonctionnalités comme Update Management, ou utiliser d’autres services Azure comme [Azure Security Center](../../security-center/security-center-introduction.md).