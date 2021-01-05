---
title: Extensions et fonctionnalités de machine virtuelle Azure pour Windows
description: Découvrez les extensions disponibles pour les machines virtuelles, regroupées par ce qu’ils fournissent ou améliorent.
services: virtual-machines-windows
documentationcenter: ''
author: axayjo
manager: gwallace
editor: ''
tags: azure-service-management,azure-resource-manager
ms.assetid: 999d63ee-890e-432e-9391-25b3fc6cde28
ms.service: virtual-machines-windows
ms.subservice: extensions
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 03/30/2018
ms.author: akjosh
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 565f98126cea8cc03874bb4f83ecdc2c65f8d5fb
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016452"
---
# <a name="virtual-machine-extensions-and-features-for-windows"></a>Extensions et fonctionnalités de machine virtuelle pour Windows

Les extensions de machine virtuelle Azure sont de petites applications permettant d’exécuter des tâches de configuration et d’automatisation post-déploiement sur des machines virtuelles Azure. Par exemple, si une machine virtuelle nécessite l’installation d’un logiciel, une protection antivirus ou l’exécution d’un script, vous pouvez utiliser une extension de machine virtuelle. Les extensions de machine virtuelle Azure sont exécutables avec l’interface de ligne de commande Azure, PowerShell, les modèles Azure Resource Manager et le portail Azure. Les extensions peuvent être intégrées à un nouveau déploiement de machine virtuelle ou s’exécuter sur tout système existant.

Cet article offre une vue d’ensemble des extensions de machine virtuelle et des prérequis pour l’utilisation d’extensions de machine virtuelle Azure. Il explique également comment détecter, gérer et supprimer des extensions de machine virtuelle. Cet article fournit des informations générales, car de nombreuses extensions de machine virtuelle sont disponibles, chacune présentant une configuration potentiellement unique. Vous trouverez des informations détaillées sur une extension spécifique dans la documentation consacrée à celle-ci.

 

## <a name="use-cases-and-samples"></a>Cas d’utilisation et exemples

Plusieurs extensions de machine virtuelle Azure sont disponibles, chacune impliquant un cas d’utilisation spécifique. Voici quelques exemples :

- Appliquer des configurations DSC (Desired State Configuration) PowerShell à une machine virtuelle avec l’extension DSC pour Windows. Pour plus d’informations sur l’extension DSC Azure, consultez [cette page](dsc-overview.md) (en anglais).
- Configurez la supervision d’une machine virtuelle avec l’extension de machine virtuelle Log Analytics Agent. Pour plus d’informations, consultez l’article [Connecter des machines virtuelles Azure aux journaux d’acvtivité Azure Monitor](../../azure-monitor/learn/quick-collect-azurevm.md).
- Configurer une machine virtuelle Azure à l’aide de Chef. Pour plus d’informations, consultez l’article [Automatisation du déploiement de machine virtuelle Azure avec Chef](/azure/developer/chef/windows-vm-configure).
- Configurer l’analyse de votre infrastructure Azure à l’aide de l’extension Datadog. Pour plus d’informations, consultez le [blog Datadog](https://www.datadoghq.com/blog/introducing-azure-monitoring-with-one-click-datadog-deployment/).


En plus des extensions propres à des processus, une extension de script personnalisé est disponible pour les machines virtuelles Windows et Linux. L’extension de script personnalisé (CustomScript) pour Windows permet d’exécuter n’importe quel script PowerShell sur une machine virtuelle. Les scripts personnalisés s’avèrent utile pour concevoir des déploiements Azure qui nécessitent une configuration plus avancée que celle fournie par les outils Azure natifs. Pour plus d’informations sur l’extension de script personnalisé pour les machines virtuelles Windows, consultez [cet article](custom-script-windows.md).

## <a name="prerequisites"></a>Prérequis

Pour gérer l’extension de la machine virtuelle, vous devez installer l’agent Windows Azure. Certaines extensions spécifiques présentent des prérequis, tels que l’accès à des ressources ou dépendances.

### <a name="azure-vm-agent"></a>Agent de machine virtuelle Azure

L’agent de machine virtuelle Azure gère les interactions entre une machine virtuelle Azure et le contrôleur de structure Microsoft Azure. L’agent de machine virtuelle est responsable de nombreux aspects fonctionnels liés au déploiement et à la gestion des machines virtuelles Azure, notamment l’exécution des extensions de machine virtuelle. L’agent de machine virtuelle Azure est préinstallé sur des images de Place de marché Azure et peut être installé manuellement sur les systèmes d’exploitation pris en charge. L’agent de machine virtuelle Azure pour Windows est connu sous le terme d’agent invité Windows.

Pour plus d’informations sur les systèmes d’exploitation pris en charge et sur la procédure d’installation, consultez l’article [Agent de machine virtuelle et extensions Azure](agent-windows.md).

#### <a name="supported-agent-versions"></a>Versions d’agent prises en charge

Des versions minimales de l’agent sont disponibles afin d’offrir la meilleure expérience possible. Pour plus d’informations, consultez [cet article](https://support.microsoft.com/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support).

#### <a name="supported-oses"></a>Systèmes d’exploitation pris en charge

L’agent invité Windows s’exécute sur plusieurs systèmes d’exploitation ; toutefois, l’infrastructure d’extensions présente certaines limites relatives aux systèmes d’exploitation pris en charge par les extensions. Pour plus d’informations, consultez [cet article](https://support.microsoft.com/help/4078134/azure-extension-supported-operating-systems
).

Certaines extensions ne sont pas prises en charge sur tous les systèmes d’exploitation et peuvent générer le *Code d’erreur 51, « Système d’exploitation non pris en charge »* . Pour plus d’informations sur la capacité de prise en charge, consultez la documentation de l’extension concernée.

#### <a name="network-access"></a>Accès réseau

Les paquets d’extensions sont téléchargés à partir du dépôt d’extensions Stockage Azure, et les chargements d’état d’extension sont publiés dans le service Stockage Azure. Si vous utilisez une version [prise en charge](https://support.microsoft.com/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support) des agents, vous n’avez pas besoin d’autoriser l’accès au service Stockage Azure dans la région de la machine virtuelle, car vous pouvez utiliser l’agent pour rediriger les communications vers le contrôleur de structure Azure pour les communications d’agent (fonctionnalité HostGAPlugin via le canal privilégié à l’adresse IP privée [168.63.129.16](../../virtual-network/what-is-ip-address-168-63-129-16.md)). Si vous utilisez une version non prise en charge de l’agent, vous devez autoriser l’accès sortant vers le service Stockage Azure dans cette région à partir de la machine virtuelle.

> [!IMPORTANT]
> Si vous avez bloqué l’accès à l’adresse *168.63.129.16* à l’aide du pare-feu invité ou avec un proxy, les extensions échouent, que vous utilisiez ou non une version prise en charge. Les ports 80, 443 et 32526 sont nécessaires.

Les agents peuvent être utilisés uniquement pour télécharger les paquets d’extensions et signaler l’état. Par exemple, si une installation d’extension doit télécharger un script à partir de GitHub (script personnalisé) ou accéder au service Stockage Azure (sauvegarde Azure), vous devez ouvrir des ports de pare-feu/de groupe de sécurité réseau (NSG) supplémentaires. Les exigences varient selon les extensions, car ces dernières sont des applications à part entière. Dans le cas des extensions qui nécessitent un accès au service Stockage Azure ou Azure Active Directory, vous pouvez autoriser l’accès à l’aide d’[étiquettes de service NSG Azure](../../virtual-network/network-security-groups-overview.md#service-tags) pour le Stockage ou AzureActiveDirectory.

L’agent invité Windows ne prend pas en charge le serveur proxy qui vous permettrait de rediriger les demandes de trafic de l’agent, ce qui signifie que l’agent invité Windows passe par votre proxy personnalisé (si vous en avez un) pour accéder aux ressources sur Internet ou sur l’hôte via l’adresse IP 168.63.129.16.

## <a name="discover-vm-extensions"></a>Détecter les extensions de machine virtuelle

De nombreuses extensions de machine virtuelle différentes peuvent être utilisées avec les machines virtuelles Azure. Pour en obtenir la liste complète, utilisez la commande [Get-AzVMExtensionImage](/powershell/module/az.compute/get-azvmextensionimage). L’exemple ci-après répertorie toutes les extensions disponibles à l’emplacement *WestUS* :

```powershell
Get-AzVmImagePublisher -Location "WestUS" | `
Get-AzVMExtensionImageType | `
Get-AzVMExtensionImage | Select Type, Version
```

## <a name="run-vm-extensions"></a>Exécuter les extensions de machine virtuelle

Les extensions de machine virtuelle Azure s’exécutent sur des machines virtuelles existantes, ce qui se révèle utile quand vous devez apporter des changements de configuration ou restaurer la connectivité sur une machine virtuelle déjà déployée. Les extensions de machines virtuelles peuvent également être intégrées dans des déploiements de modèles Azure Resource Manager. L’utilisation d’extensions avec des modèles Resource Manager vous permet de déployer et de configurer des machines virtuelles Azure sans avoir à intervenir après le déploiement.

Vous pouvez exécuter une extension sur une machine virtuelle existante à l’aide des méthodes ci-après.

### <a name="powershell"></a>PowerShell

Il existe plusieurs commandes PowerShell pour l’exécution des extensions. Pour en obtenir la liste, utilisez la commande [Get-Command](/powershell/module/microsoft.powershell.core/get-command) et appliquez aux données le filtre *Extension* :

```powershell
Get-Command Set-Az*Extension* -Module Az.Compute
```

Vous générez ainsi des informations qui ressemblent à ce qui suit :

```powershell
CommandType     Name                                          Version    Source
-----------     ----                                          -------    ------
Cmdlet          Set-AzVMAccessExtension                       4.5.0      Az.Compute
Cmdlet          Set-AzVMADDomainExtension                     4.5.0      Az.Compute
Cmdlet          Set-AzVMAEMExtension                          4.5.0      Az.Compute
Cmdlet          Set-AzVMBackupExtension                       4.5.0      Az.Compute
Cmdlet          Set-AzVMBginfoExtension                       4.5.0      Az.Compute
Cmdlet          Set-AzVMChefExtension                         4.5.0      Az.Compute
Cmdlet          Set-AzVMCustomScriptExtension                 4.5.0      Az.Compute
Cmdlet          Set-AzVMDiagnosticsExtension                  4.5.0      Az.Compute
Cmdlet          Set-AzVMDiskEncryptionExtension               4.5.0      Az.Compute
Cmdlet          Set-AzVMDscExtension                          4.5.0      Az.Compute
Cmdlet          Set-AzVMExtension                             4.5.0      Az.Compute
Cmdlet          Set-AzVMSqlServerExtension                    4.5.0      Az.Compute
Cmdlet          Set-AzVmssDiskEncryptionExtension             4.5.0      Az.Compute
```

L’exemple suivant utilise l’extension de script personnalisé pour télécharger un script sur la machine virtuelle cible depuis un dépôt GitHub, puis pour exécuter le script. Pour plus d’informations sur l’extension de script personnalisé, consultez [Vue d’ensemble de l’extension de script personnalisé](custom-script-windows.md).

```powershell
Set-AzVMCustomScriptExtension -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" -Name "myCustomScript" `
    -FileUri "https://raw.githubusercontent.com/neilpeterson/nepeters-azure-templates/master/windows-custom-script-simple/support-scripts/Create-File.ps1" `
    -Run "Create-File.ps1" -Location "West US"
```

Dans l’exemple ci-après, l’extension d’accès aux machines virtuelles (VMAccess) est utilisée pour redéfinir le mot de passe d’administration d’une machine virtuelle Windows sur un mot de passe temporaire. Pour plus d’informations sur l’extension d’accès aux machines virtuelles, consultez [Réinitialiser le service Bureau à distance pour une machine virtuelle Windows](../troubleshooting/reset-rdp.md). Après avoir exécuté cette extension, vous devrez réinitialiser le mot de passe lors de votre première connexion :

```powershell
$cred=Get-Credential

Set-AzVMAccessExtension -ResourceGroupName "myResourceGroup" -VMName "myVM" -Name "myVMAccess" `
    -Location WestUS -UserName $cred.GetNetworkCredential().Username `
    -Password $cred.GetNetworkCredential().Password -typeHandlerVersion "2.0"
```

Vous pouvez utiliser la commande `Set-AzVMExtension` pour démarrer n’importe quelle extension de machine virtuelle. Pour plus d’informations, consultez [Référence Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension).


### <a name="azure-portal"></a>Portail Azure

Vous pouvez appliquer les extensions de machine virtuelle à une machine virtuelle existante par le biais du portail Azure. Sélectionnez la machine virtuelle dans le portail, choisissez **Extensions**, puis sélectionnez **Ajouter**. Choisissez l’extension souhaitée dans la liste des extensions disponibles, puis suivez les instructions de l’Assistant.

L’exemple ci-après illustre l’installation de l’extension Microsoft Antimalware à partir du Portail Azure :

![Installer l’extension Antimalware](./media/features-windows/installantimalwareextension.png)

### <a name="azure-resource-manager-templates"></a>Modèles Microsoft Azure Resource Manager

Les extensions de machine virtuelle peuvent être ajoutées à un modèle Azure Resource Manager et exécutées avec le déploiement du modèle. Lorsque vous déployez une extension avec un modèle, vous pouvez créer des déploiements Azure entièrement configurés. Par exemple, le code JSON ci-après est tiré d’un modèle Resource Manager qui déploie un ensemble de machines virtuelles à charge équilibrée et une base de données Azure SQL, puis installe une application .NET Core sur chaque machine virtuelle. L’extension de machine virtuelle se charge de l’installation du logiciel.

Pour plus d’informations, consultez le [modèle Resource Manager complet](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-windows).

```json
{
    "apiVersion": "2015-06-15",
    "type": "extensions",
    "name": "config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'),copyindex())]",
    "[variables('musicstoresqlName')]"
    ],
    "tags": {
    "displayName": "config-app"
    },
    "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
        ]
    },
    "protectedSettings": {
        "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 -user ',parameters('adminUsername'),' -password ',parameters('adminPassword'),' -sqlserver ',variables('musicstoresqlName'),'.database.windows.net')]"
    }
    }
}
```

Pour plus d’informations sur la création de modèles Resource Manager, consultez l’article [Création de modèles Azure Resource Manager avec des extensions de machine virtuelle Windows](../windows/template-description.md#extensions).

## <a name="secure-vm-extension-data"></a>Sécuriser les données des extensions de machine virtuelle

Quand vous exécutez une extension de machine virtuelle, vous pouvez avoir besoin d’inclure des informations sensibles telles que des informations d’identification, des noms de compte de stockage et des clés d’accès à des comptes de stockage. De nombreuses extensions de machine virtuelle comprennent une configuration protégée qui chiffre les données et les déchiffre uniquement à l’intérieur de la machine virtuelle cible. Chaque extension possède un schéma spécifique de configuration protégée, présenté en détail dans la documentation consacrée à l’extension.

L’exemple suivant illustre une instance de l’extension de script personnalisé pour Windows. La commande à exécuter inclut un ensemble d’informations d’identification. Dans cet exemple, la commande à exécuter n’est pas chiffrée :

```json
{
    "apiVersion": "2015-06-15",
    "type": "extensions",
    "name": "config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'),copyindex())]",
    "[variables('musicstoresqlName')]"
    ],
    "tags": {
    "displayName": "config-app"
    },
    "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
        ],
        "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 -user ',parameters('adminUsername'),' -password ',parameters('adminPassword'),' -sqlserver ',variables('musicstoresqlName'),'.database.windows.net')]"
    }
    }
}
```

Pour sécuriser la chaîne d’exécution, déplacez la propriété **commandToExecute** vers la configuration **protected**, comme indiqué dans l’exemple suivant :

```json
{
    "apiVersion": "2015-06-15",
    "type": "extensions",
    "name": "config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'),copyindex())]",
    "[variables('musicstoresqlName')]"
    ],
    "tags": {
    "displayName": "config-app"
    },
    "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
        ]
    },
    "protectedSettings": {
        "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 -user ',parameters('adminUsername'),' -password ',parameters('adminPassword'),' -sqlserver ',variables('musicstoresqlName'),'.database.windows.net')]"
    }
    }
}
```

Sur une machine virtuelle Azure IaaS qui utilise des extensions, dans la console des certificats, vous pouvez voir des certificats dont le nom de sujet est **_Windows Azure CRP Certificate Generator_** Sur une machine virtuelle RDFE classique, ces certificats ont comme nom de sujet **_Windows Azure Service Management for Extensions_**.

Ces certificats sécurisent la communication entre la machine virtuelle et son hôte pendant le transfert des paramètres protégés (mot de passe, autres informations d’identification) qui sont utilisés par les extensions. Les certificats sont générés par le contrôleur de structure Azure et sont passés à l’agent de machine virtuelle. Si vous arrêtez et démarrez la machine virtuelle tous les jours, un nouveau certificat peut être créé par le contrôleur de structure. Le certificat est stocké dans le magasin de certificats Personnel de l’ordinateur. Ces certificats peuvent être supprimés. L’agent de machine virtuelle recrée les certificats si nécessaire.

### <a name="how-do-agents-and-extensions-get-updated"></a>Comment les agents et les extensions sont-ils mis à jour ?

Les agents et les extensions partagent le même mécanisme de mise à jour. Certaines mises à jour ne nécessitent pas de règles de pare-feu supplémentaires.

Quand une mise à jour est disponible, elle est installée sur la machine virtuelle uniquement quand un changement est apporté aux extensions, ainsi qu’en cas d’autres changements de modèle de machine virtuelle, tels que :

- Disques de données
- Extensions
- Conteneur de diagnostics de démarrage
- Secrets de système d’exploitation invité
- Taille de la machine virtuelle
- Profil réseau

Les dates de publication de mises à jour par les éditeurs varient selon les régions. Vous pouvez donc disposer de machines virtuelles équipées de versions distinctes dans différentes régions.

#### <a name="listing-extensions-deployed-to-a-vm"></a>Liste des extensions déployées sur une machine virtuelle

```powershell
$vm = Get-AzVM -ResourceGroupName "myResourceGroup" -VMName "myVM"
$vm.Extensions | select Publisher, VirtualMachineExtensionType, TypeHandlerVersion
```

```powershell
Publisher             VirtualMachineExtensionType          TypeHandlerVersion
---------             ---------------------------          ------------------
Microsoft.Compute     CustomScriptExtension                1.9
```

#### <a name="agent-updates"></a>Mises à jour d’agent

L’agent invité Windows contient uniquement le *code de gestion des extensions* ; le *code d’approvisionnement Windows* est fourni séparément. Vous pouvez désinstaller l’agent invité Windows. Vous ne pouvez pas désactiver la mise à jour automatique de l’agent invité Windows.

Le *code de gestion des extensions* assure la communication avec la structure Azure, ainsi que la gestion des opérations des extensions de machine virtuelle, telles que les installations, le signalement d’état, la mise à jour des différentes extensions et la suppression de ces dernières. Les mises à jour comprennent les correctifs de sécurité, les corrections de bogues et les améliorations du *code de gestion des extensions*.

Si vous souhaitez vérifier la version que vous exécutez, consultez la section [Detecting installed Windows Guest Agent](agent-windows.md#detect-the-vm-agent) (Détection de l’agent invité Windows installé).

#### <a name="extension-updates"></a>Mises à jour des extensions

Lorsqu’une mise à jour d’extension devient disponible, l’agent invité Windows la télécharge et met à niveau l’extension. Les mises à jour d’extension automatiques sont soit de type *Minor* (mise à jour mineure), soit de type *Hotfix* (correctif logiciel). Vous pouvez accepter ou refuser les mises à jour d’extension *Minor* quand vous provisionnez l’extension. L’exemple ci-après indique comment mettre à niveau automatiquement les versions mineures dans un modèle Resource Manager avec la commande *"autoUpgradeMinorVersion": true,* :

```json
    "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
        ]
    },
```

Si vous souhaitez obtenir les dernières corrections de bogues des versions mineures, nous vous recommandons vivement de toujours sélectionner l’option de mise à jour automatique dans vos déploiements d’extensions. Vous ne pouvez pas refuser les mises à jour de type correctif logiciel qui comportent des correctifs de sécurité ou des corrections de bogues clés.

### <a name="how-to-identify-extension-updates"></a>Comment identifier les mises à jour d’extension

#### <a name="identifying-if-the-extension-is-set-with-autoupgrademinorversion-on-a-vm"></a>Vérification si l’extension est définie avec la propriété autoUpgradeMinorVersion sur une machine virtuelle

Le modèle de machine virtuelle vous permet de déterminer si l’extension a été provisionnée avec la propriété "autoUpgradeMinorVersion". Pour vérifier ce point, utilisez la commande [Get-AzVm](/powershell/module/az.compute/get-azvm) et indiquez le groupe de ressources et le nom de machine virtuelle comme suit :

```powerShell
 $vm = Get-AzVm -ResourceGroupName "myResourceGroup" -VMName "myVM"
 $vm.Extensions
```

La sortie de l’exemple suivant indique que la propriété *autoUpgradeMinorVersion* est définie sur la valeur *true* :

```powershell
ForceUpdateTag              :
Publisher                   : Microsoft.Compute
VirtualMachineExtensionType : CustomScriptExtension
TypeHandlerVersion          : 1.9
AutoUpgradeMinorVersion     : True
```

#### <a name="identifying-when-an-autoupgrademinorversion-occurred"></a>Identification du moment d’exécution d’une mise à jour de type autoUpgradeMinorVersion

Pour déterminer le moment où une extension a fait l’objet d’une mise à jour, consultez les journaux d’activité d’agent sur la machine virtuelle à l’emplacement *C:\WindowsAzure\Logs\WaAppAgent.log*.

Dans l’exemple ci-après, la machine virtuelle était équipée de la version *Microsoft.Compute.CustomScriptExtension 1.8*. Un correctif logiciel est ensuite devenu disponible sous la forme de la version *1.9* :

```powershell
[INFO]  Getting plugin locations for plugin 'Microsoft.Compute.CustomScriptExtension'. Current Version: '1.8', Requested Version: '1.9'
[INFO]  Auto-Upgrade mode. Highest public version for plugin 'Microsoft.Compute.CustomScriptExtension' with requested version: '1.9', is: '1.9'
```

## <a name="agent-permissions"></a>Autorisations d’agent

Pour exécuter ses tâches, l’agent doit s’exécuter en tant que *Système local*.

## <a name="troubleshoot-vm-extensions"></a>Résoudre les problèmes liés aux extensions de machine virtuelle

Chaque extension de machine virtuelle peut présenter une procédure de résolution des problèmes spécifique. Par exemple, quand vous utilisez l’extension de script personnalisé, les détails de l’exécution du script sont accessibles localement sur la machine virtuelle utilisée pour l’exécution de l’extension. La procédure de résolution des problèmes spécifique d’une extension est présentée en détail dans la documentation de cette dernière.

La procédure de résolution des problèmes ci-après s’applique à toutes les extensions de machine virtuelle.

1. Pour consulter le journal de l’agent invité Windows, examinez l’activité au moment où votre extension a été approvisionnée dans le fichier *C:\WindowsAzure\Logs\WaAppAgent.log*

2. Pour plus de détails, vérifiez les journaux d’activité d’extension proprement dits à l’emplacement *C:\WindowsAzure\Logs\Plugins\\<extensionName>*

3. Consultez les sections de résolution des problèmes dans la documentation de l’extension concernée pour vous renseigner sur les codes d’erreur, les problèmes connus, etc.

4. Examinez les journaux d’activité système. Vérifiez si d’autres opérations sont susceptibles d’avoir interféré avec l’extension, par exemple une longue installation d’une autre application nécessitant un accès exclusif au gestionnaire de package.

### <a name="common-reasons-for-extension-failures"></a>Motifs courants des échecs d’extension

1. Délai d’exécution des extensions limité à 20 minutes (étendu à 90 minutes pour les extensions CustomScript, Chef et DSC). Si votre déploiement excède ce délai, il est signalé comme dépassement de délai d’expiration. Ce dépassement peut être dû à une insuffisance de ressources sur les machines virtuelles, découlant du fait que d’autres configurations/tâches de démarrage de machine virtuelle consomment de grandes quantités de ressources alors qu’une tentative de provisionnement d’une extension est en cours.

2. Prérequis minimaux non respectés. Certaines extensions présentent des dépendances vis-à-vis des références SKU de machine virtuelle, telles que les images HPC. Les extensions peuvent imposer certaines exigences d’accès réseau, comme la communication avec le service Stockage Azure ou les services publics. D’autres exemples de prérequis concernent l’accès aux référentiels de packages, l’espace disque nécessaire ou les restrictions de sécurité.

3. Accès exclusif au gestionnaire de package. Dans certains cas, une configuration de machine virtuelle longue peut entrer en conflit avec l’installation d’une extension, quand ces deux tâches exigent un accès exclusif au gestionnaire de package.

### <a name="view-extension-status"></a>Afficher l’état de l’extension

Une fois qu’une extension de machine virtuelle a été exécutée sur une machine virtuelle, utilisez la commande [Get-AzVM ](/powershell/module/az.compute/get-azvm) pour récupérer l’état de l’extension. L’état *Substatuses[0]* indique que l’approvisionnement de l’extension s’est correctement déroulé, autrement dit que l’extension a été déployée sur la machine virtuelle, mais l’état *Substatuses[1]* signale que l’exécution de l’extension dans la machine virtuelle a échoué.

```powershell
Get-AzVM -ResourceGroupName "myResourceGroup" -VMName "myVM" -Status
```

Le résultat obtenu ressemble à l’exemple de sortie suivant :

```powershell
Extensions[0]           :
  Name                  : CustomScriptExtension
  Type                  : Microsoft.Compute.CustomScriptExtension
  TypeHandlerVersion    : 1.9
  Substatuses[0]        :
    Code                : ComponentStatus/StdOut/succeeded
    Level               : Info
    DisplayStatus       : Provisioning succeeded
    Message             : Windows PowerShell \nCopyright (C) Microsoft Corporation. All rights reserved.\n
  Substatuses[1]        :
    Code                : ComponentStatus/StdErr/succeeded
    Level               : Info
    DisplayStatus       : Provisioning succeeded
    Message             : The argument 'cseTest%20Scriptparam1.ps1' to the -File parameter does not exist. Provide the path to an existing '.ps1' file as an argument to the

-File parameter.
  Statuses[0]           :
    Code                : ProvisioningState/failed/-196608
    Level               : Error
    DisplayStatus       : Provisioning failed
    Message             : Finished executing command
```

L’état d’exécution de l’extension est également visible dans le portail Azure. Pour visualiser l’état d’une extension, sélectionnez la machine virtuelle, choisissez **Extensions**, puis sélectionnez l’extension souhaitée.

### <a name="rerun-vm-extensions"></a>Réexécuter les extensions de machine virtuelle

Dans certains cas, vous pouvez avoir besoin de réexécuter une extension de machine virtuelle. Pour ce faire, supprimez-la, puis réexécutez-la avec la méthode d’exécution de votre choix. Pour supprimer une extension, utilisez la commande [Remove-AzVMExtension](/powershell/module/az.compute/remove-azvmextension) comme suit :

```powershell
Remove-AzVMExtension -ResourceGroupName "myResourceGroup" -VMName "myVM" -Name "myExtensionName"
```

Vous pouvez également supprimer une extension dans le portail Azure en procédant comme suit :

1. Sélectionnez une machine virtuelle.
2. Choisissez **Extensions**.
3. Sélectionnez l’extension souhaitée.
4. Choisissez **Désinstaller**.

## <a name="common-vm-extensions-reference"></a>Informations de référence sur les extensions de machine virtuelle courantes
| Nom de l’extension | Description | Informations complémentaires |
| --- | --- | --- |
| Extension de script personnalisé pour Windows |Exécuter des scripts sur une machine virtuelle Azure |[Extension de script personnalisé pour Windows](custom-script-windows.md) |
| Extension DSC pour Windows |Extension PowerShell DSC (Desired State Configuration, configuration d’état souhaité) |[Extension DSC pour Windows](dsc-overview.md) |
| Extension Diagnostics Azure |Gérer les diagnostics Azure |[Extension Diagnostics Azure](https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/) |
| Extension d’accès aux machines virtuelles Azure |Gérer les utilisateurs et les informations d’identification |[Extension d’accès aux machines virtuelles pour Linux](https://azure.microsoft.com/blog/using-vmaccess-extension-to-reset-login-credentials-for-linux-vm/) |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les extensions de machine virtuelle, consultez [Vue d’ensemble des extensions et des fonctionnalités des machines virtuelles Azure](overview.md).
