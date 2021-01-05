---
title: Installer l’agent Connected Machine à l’aide de DSC Windows PowerShell
description: Dans cet article, vous allez apprendre à connecter des machines à Azure à l’aide d’Azure Arc enabled servers en utilisant DSC Windows PowerShell.
ms.date: 09/24/2020
ms.topic: conceptual
ms.openlocfilehash: b351aa8e989bd36c135271d3adafca6eb64ccf39
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92107553"
---
# <a name="how-to-install-the-connected-machine-agent-using-windows-powershell-dsc"></a>Guide pratique pour installer l’agent Connected Machine à l’aide de DSC Windows PowerShell

Avec [Desired State Configuration (DSC) Windows PowerShell](/powershell/scripting/dsc/getting-started/winGettingStarted), vous pouvez automatiser l’installation et la configuration de logiciels pour un ordinateur Windows. Cet article explique comment utiliser DSC pour installer l’agent Connected Machine Azure Arc enabled servers sur des machines Windows hybrides.

## <a name="requirements"></a>Spécifications

- Windows PowerShell version 4.0 ou supérieure

- Module DSC [AzureConnectedMachineDsc](https://www.powershellgallery.com/packages/AzureConnectedMachineDsc)

- Un principal de service pour connecter de manière non interactive les machines à Azure Arc enabled servers. Suivez les étapes sous la section [Créer un principal de service pour une intégration à grande échelle](onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale) si vous n’avez pas déjà créé de principal de service pour Azure Arc enabled servers.

## <a name="install-the-connectedmachine-dsc-module"></a>Installer le module DSC ConnectedMachine

1. Pour installer manuellement le module, téléchargez le code source et décompressez le contenu du répertoire du projet dans `$env:ProgramFiles\WindowsPowerShell\Modules folder`. Ou bien, exécutez la commande suivante pour installer à partir de PowerShell Gallery avec PowerShellGet (dans PowerShell 5.0) :

    ```powershell
    Find-Module -Name AzureConnectedMachineDsc -Repository PSGallery | Install-Module
    ```

2. Pour confirmer l’installation, exécutez la commande suivante et vérifiez que vous voyez les ressources DSC d’Azure Connected Machine disponibles.

    ```powershell
    Get-DscResource -Module AzureConnectedMachineDsc
    ```

   Dans la sortie, vous devez obtenir un résultat similaire à ceci :

   ![Confirmation de l’exemple d’installation du module DSC de Connected Machine](./media/onboard-dsc/confirm-module-installation.png)

## <a name="install-the-agent-and-connect-to-azure"></a>Installer l’agent et le connecter à Azure

Les ressources de ce module sont conçues pour gérer la configuration de l’agent Azure Connected Machine. Également disponible un script PowerShell `AzureConnectedMachineAgent.ps1`, dans le dossier `AzureConnectedMachineDsc\examples`. Il utilise les ressources de la communauté pour automatiser le téléchargement et l’installation, établir aussi une connexion avec Azure Arc. Ce script effectue des étapes similaires à celles décrites dans l’article [Connecter des machines hybrides à Azure à partir du portail Azure](onboard-portal.md).

Si la machine doit communiquer avec le service via un serveur proxy, après avoir installé l’agent, vous devez exécuter une commande décrite [ici](manage-agent.md#update-or-remove-proxy-settings). Cette dernière définit la variable d’environnement système du serveur proxy `https_proxy`. Au lieu d’exécuter la commande manuellement, vous pouvez effectuer cette étape avec DSC en utilisant le module [ComputeManagementDsc](https://www.powershellgallery.com/packages/ComputerManagementDsc).

>[!NOTE]
>Pour permettre l’exécution de DSC, Windows doit être configuré pour recevoir des commandes PowerShell à distance, même lorsque vous exécutez une configuration localhost. Pour configurer facilement et correctement votre environnement, exécutez simplement `Set-WsManQuickConfig -Force` dans un terminal PowerShell avec élévation de privilèges.
>

La configuration de documents (fichiers MOF) peut être appliquée à la machine au moyen de l’applet de commande `Start-DscConfiguration`.

Voici les paramètres que vous transmettez au script PowerShell à utiliser.

- `TenantId`: Identificateur unique (GUID) qui représente votre instance dédiée d’Azure AD.

- `SubscriptionId`: ID d’abonnement (GUID) de votre abonnement Azure dans lequel vous souhaitez placer les machines.

- `ResourceGroup`: Nom du groupe de ressources auquel vous souhaitez que vos ordinateurs connectés appartiennent.

- `Location`: Voir les [régions Azure prises en charge](overview.md#supported-regions). Cet emplacement peut être identique ou différent de l’emplacement du groupe de ressources.

- `Tags`: Tableau de chaînes contenant les balises qui doivent être appliquées à la ressource de la machine connectée.

- `Credential`: Un objet d’informations d’identification PowerShell avec les éléments **ApplicationId** et **password** utilisés pour inscrire des machines à grande échelle à l’aide d’un [principal de service](onboard-service-principal.md).

1. Dans une console PowerShell, accédez au dossier où vous avez enregistré le fichier `.ps1`.

2. Exécutez les commandes PowerShell suivantes pour compiler le document MOF. Pour plus d’informations sur la compilation des configurations DSC, consultez [Configurations DSC](/powershell/scripting/dsc/configurations/configurations) :

    ```powershell
    .\`AzureConnectedMachineAgent.ps1 -TenantId <TenantId GUID> -SubscriptionId <SubscriptionId GUID> -ResourceGroup '<ResourceGroupName>' -Location '<LocationName>' -Tags '<Tag>' -Credential <psCredential>
    ```

3. Cette opération crée un `localhost.mof file` dans un nouveau dossier nommé `C:\dsc`.

Une fois que vous avez installé l’agent et que vous l’avez configuré pour qu’il se connecte à Azure Arc enabled servers, accédez au portail Azure pour vérifier que le serveur a été correctement connecté. Affichez vos machines dans le [portail Azure](https://aka.ms/hybridmachineportal).

## <a name="adding-to-existing-configurations"></a>Ajout aux configurations existantes

Cette ressource peut être ajoutée à des configurations DSC existantes, afin de représenter une configuration de bout en bout pour une machine. Par exemple, vous pouvez souhaiter ajouter cette ressource à une configuration qui définit des paramètres de système d’exploitation sécurisés.

Le module [CompositeResource](https://www.powershellgallery.com/packages/compositeresource) depuis PowerShell Gallery peut être utilisé pour créer une [ressource composite](/powershell/scripting/dsc/resources/authoringResourceComposite) de l’exemple de configuration, afin de simplifier encore davantage la combinaison de configurations.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la résolution des problèmes, consultez le [guide Résoudre les problèmes de l’agent Connected Machine](troubleshoot-agent-onboard.md).

* Apprenez à gérer votre machine à l’aide d’[Azure Policy](../../governance/policy/overview.md), par exemple pour la [configuration invité](../../governance/policy/concepts/guest-configuration.md) des machines virtuelles, pour vérifier que l’ordinateur crée des rapports sur l’espace de travail Log Analytics prévu, pour activer l’analyse d’[Azure Monitor sur des machines virtuelles](../../azure-monitor/insights/vminsights-enable-policy.md) et bien plus encore.

* En savoir plus sur [l’agent Log Analytics](../../azure-monitor/platform/log-analytics-agent.md). L’agent Log Analytics pour Windows et Linux est nécessaire quand vous souhaitez superviser de manière proactive le système d’exploitation et les charges de travail en cours d’exécution sur la machine, gérer le système d’exploitation à l’aide de runbooks Automation ou de solutions comme Update Management ou utiliser d’autres services Azure tels qu’[Azure Security Center](../../security-center/security-center-introduction.md).