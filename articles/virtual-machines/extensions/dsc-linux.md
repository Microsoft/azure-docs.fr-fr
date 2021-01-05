---
title: Extension Azure DSC pour Linux
description: Installation des packages OMI et DSC afin de configurer une machine virtuelle Azure Linux à l’aide de Desired State Configuration (DSC).
services: virtual-machines-linux
documentationcenter: ''
author: bobbytreed
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.subservice: extensions
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 06/12/2018
ms.author: robreed
ms.openlocfilehash: 1d1a5cf67a10a83a227f240fc31d25abfe9c7dd0
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94955937"
---
# <a name="dsc-extension-for-linux-microsoftostcextensionsdscforlinux"></a>Extension DSC pour Linux (Microsoft.OSTCExtensions.DSCForLinux)

Desired State Configuration (DSC) est une plateforme de gestion que vous pouvez utiliser pour gérer votre infrastructure informatique et de développement avec la configuration en tant que code.

> [!NOTE]
> L’extension DSC pour Linux et l’[extension de machine virtuelle Azure Monitor pour Linux](./oms-linux.md) sont actuellement en conflit et ne sont pas prises en charge dans une configuration côte à côte. N’utilisez pas les deux solutions sur la même machine virtuelle.

L’extension DSCForLinux est publiée et prise en charge par Microsoft. L’extension installe l’agent DSC et OMI sur des machines virtuelles Azure. L’extension DSC peut également effectuer les actions suivantes :

- Enregistrer la machine virtuelle Linux sur un compte Azure Automation pour extraire des configurations depuis le service Azure Automation (Register ExtensionAction).
- Envoyer (push) des configurations MOF vers la machine virtuelle Linux (Push ExtensionAction).
- Appliquer la configuration MOF méta à la machine virtuelle Linux pour configurer le serveur Pull et ainsi extraire la configuration de nœud (Pull ExtensionAction).
- Installer des modules DSC personnalisés sur la machine virtuelle Linux (Install ExtensionAction).
- Supprimer des modules DSC personnalisés de la machine virtuelle Linux (Remove ExtensionAction).

## <a name="prerequisites"></a>Prérequis

### <a name="operating-system"></a>Système d’exploitation

Pour les nœuds exécutant Linux, l’extension Linux DSC prend en charge toutes les distributions Linux listées dans la [documentation de DSC PowerShell](/powershell/scripting/dsc/getting-started/lnxgettingstarted).

### <a name="internet-connectivity"></a>Connectivité Internet

La machine virtuelle cible doit être connectée à Internet pour pouvoir utiliser l’extension DSCForLinux. Par exemple, l’extension Register nécessite une connectivité au service Automation.
D’autres actions telles que Push, Pull et Install nécessitent une connectivité à Stockage Azure et GitHub. Cela dépend des paramètres indiqués par le client.

## <a name="extension-schema"></a>Schéma d’extensions

### <a name="public-configuration"></a>Configuration publique

Voici tous les paramètres de configuration publique pris en charge :

* `FileUri` : (facultatif, chaîne) URI du fichier MOF, du fichier MOF méta ou du fichier zip de ressources personnalisées.
* `ResourceName` : (facultatif, chaîne) nom du module de ressources personnalisées.
* `ExtensionAction` : (facultatif, chaîne) spécifie le rôle d’une extension. Les valeurs valides sont Register, Push, Pull, Install et Remove. Si rien n’est spécifié, l’action par défaut est Push.
* `NodeConfigurationName` : (facultatif, chaîne) nom d’une configuration de nœud à appliquer.
* `RefreshFrequencyMins` : (facultatif, entier) spécifie la fréquence (en minutes) à laquelle DSC tente d’obtenir la configuration depuis le serveur Pull.
       Si la configuration du serveur Pull est différente de la configuration actuelle sur le nœud cible, elle est copiée dans le magasin en attente, puis appliquée.
* `ConfigurationMode` : (facultatif, chaîne) indique la façon dont DSC applique la configuration. Les valeurs valides sont ApplyOnly, ApplyAndMonitor et ApplyAndAutoCorrect.
* `ConfigurationModeFrequencyMins` : (facultatif, entier) spécifie la fréquence (en minutes) à laquelle DSC vérifie que la configuration est conforme à l’état souhaité.

> [!NOTE]
> Si vous utilisez une version antérieure à la version 2.3, le paramètre de mode est le même qu’ExtensionAction. Mode semble être un terme surchargé. Pour éviter toute confusion, ExtensionAction est utilisé à partir de la version 2.3. Pour la compatibilité descendante, l’extension prend en charge à la fois mode et ExtensionAction.
>

### <a name="protected-configuration"></a>Configuration protégée

Voici tous les paramètres de configuration protégée pris en charge :

* `StorageAccountName` : (facultatif, chaîne) nom du compte de stockage qui contient le fichier
* `StorageAccountKey` : (facultatif, chaîne) clé du compte de stockage qui contient le fichier
* `RegistrationUrl` : (facultatif, chaîne) URL du compte Azure Automation
* `RegistrationKey` : (facultatif, chaîne) clé d’accès du compte Azure Automation


## <a name="scenarios"></a>Scénarios

### <a name="register-an-azure-automation-account"></a>Inscrire un compte Azure Automation
protected.json
```json
{
  "RegistrationUrl": "<azure-automation-account-url>",
  "RegistrationKey": "<azure-automation-account-key>"
}
```
public.json
```json
{
  "ExtensionAction" : "Register",
  "NodeConfigurationName" : "<node-configuration-name>",
  "RefreshFrequencyMins" : "<value>",
  "ConfigurationMode" : "<ApplyAndMonitor | ApplyAndAutoCorrect | ApplyOnly>",
  "ConfigurationModeFrequencyMins" : "<value>"
}
```

Format PowerShell
```powershell
$privateConfig = '{
  "RegistrationUrl": "<azure-automation-account-url>",
  "RegistrationKey": "<azure-automation-account-key>"
}'

$publicConfig = '{
  "ExtensionAction" : "Register",
  "NodeConfigurationName": "<node-configuration-name>",
  "RefreshFrequencyMins": "<value>",
  "ConfigurationMode": "<ApplyAndMonitor | ApplyAndAutoCorrect | ApplyOnly>",
  "ConfigurationModeFrequencyMins": "<value>"
}'
```

### <a name="apply-an-mof-configuration-file-in-an-azure-storage-account-to-the-vm"></a>Appliquer un fichier config MOF (dans le compte de stockage Azure) sur la machine virtuelle

protected.json
```json
{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}
```

public.json
```json
{
  "FileUri": "<mof-file-uri>",
  "ExtensionAction": "Push"
}
```

Format PowerShell
```powershell
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'

$publicConfig = '{
  "FileUri": "<mof-file-uri>",
  "ExtensionAction": "Push"
}'
```


### <a name="apply-an-mof-configuration-file-in-public-storage-to-the-vm"></a>Appliquer un fichier config MOF (dans le stockage public) sur la machine virtuelle

public.json
```json
{
  "FileUri": "<mof-file-uri>"
}
```

Format PowerShell
```powershell
$publicConfig = '{
  "FileUri": "<mof-file-uri>"
}'
```

### <a name="apply-a-meta-mof-configuration-file-in-an-azure-storage-account-to-the-vm"></a>Appliquer un fichier config MOF méta (dans un compte de stockage Azure) sur la machine virtuelle

protected.json
```json
{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}
```

public.json
```json
{
  "ExtensionAction": "Pull",
  "FileUri": "<meta-mof-file-uri>"
}
```

Format PowerShell
```powershell
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'

$publicConfig = '{
  "ExtensionAction": "Pull",
  "FileUri": "<meta-mof-file-uri>"
}'
```

### <a name="apply-a-meta-mof-configuration-file-in-public-storage-to-the-vm"></a>Appliquer un fichier de configuration MOF méta (dans le stockage public) sur la machine virtuelle
public.json
```json
{
  "FileUri": "<meta-mof-file-uri>",
  "ExtensionAction": "Pull"
}
```
Format PowerShell
```powershell
$publicConfig = '{
  "FileUri": "<meta-mof-file-uri>",
  "ExtensionAction": "Pull"
}'
```

### <a name="install-a-custom-resource-module-a-zip-file-in-an-azure-storage-account-to-the-vm"></a>Installer un module de ressources personnalisées (fichier zip dans un compte de stockage Azure) sur la machine virtuelle
protected.json
```json
{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}
```
public.json
```json
{
  "ExtensionAction": "Install",
  "FileUri": "<resource-zip-file-uri>"
}
```

Format PowerShell
```powershell
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'

$publicConfig = '{
  "ExtensionAction": "Install",
  "FileUri": "<resource-zip-file-uri>"
}'
```

### <a name="install-a-custom-resource-module-a-zip-file-in-public-storage-to-the-vm"></a>Installer un module de ressources personnalisées (fichier zip dans le stockage public) sur la machine virtuelle
public.json
```json
{
  "ExtensionAction": "Install",
  "FileUri": "<resource-zip-file-uri>"
}
```
Format PowerShell
```powershell
$publicConfig = '{
  "ExtensionAction": "Install",
  "FileUri": "<resource-zip-file-uri>"
}'
```

### <a name="remove-a-custom-resource-module-from-the-vm"></a>Supprimer un module de ressources personnalisées de la machine virtuelle
public.json
```json
{
  "ResourceName": "<resource-name>",
  "ExtensionAction": "Remove"
}
```
Format PowerShell
```powershell
$publicConfig = '{
  "ResourceName": "<resource-name>",
  "ExtensionAction": "Remove"
}'
```

## <a name="template-deployment"></a>Déploiement de modèle

Les extensions de machines virtuelles Azure peuvent être déployées avec des modèles Azure Resource Manager. Les modèles sont particulièrement adaptés lorsque vous déployez une ou plusieurs machines virtuelles nécessitant une configuration post-déploiement, comme l’intégration à Azure Automation.

Les exemples de modèles du Gestionnaire des ressources sont [201-dsc-linux-azure-storage-on-ubuntu](https://github.com/Azure/azure-quickstart-templates/tree/master/201-dsc-linux-azure-storage-on-ubuntu) et [201-dsc-linux-public-storage-on-ubuntu](https://github.com/Azure/azure-quickstart-templates/tree/master/201-dsc-linux-public-storage-on-ubuntu).

Pour en savoir plus sur les modèles Azure Resource Manager, consultez [Création de modèles Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md).


## <a name="azure-cli-deployment"></a>Déploiement de l’interface de ligne de commande Azure

### <a name="use-azure-cliazure-cli"></a>Utiliser [Azure CLI][azure-cli]
Avant de déployer l’extension DSCForLinux, configurez `public.json` et `protected.json` conformément aux scénarios décrits à la section 3.

#### <a name="classic"></a>Classique

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

Le mode de déploiement classique est également appelé mode Azure Service Management. Vous pouvez basculer vers ce mode en exécutant la commande suivante :
```
$ azure config mode asm
```

Vous pouvez déployer l’extension DSCForLinux en exécutant la commande suivante :
```
$ azure vm extension set <vm-name> DSCForLinux Microsoft.OSTCExtensions <version> \
--private-config-path protected.json --public-config-path public.json
```

Pour en savoir sur la dernière version d’extension disponible, exécutez la commande suivante :
```
$ azure vm extension list
```

#### <a name="resource-manager"></a>Gestionnaire de ressources
Vous pouvez basculer vers le mode Azure Resource Manager en exécutant la commande suivante :
```
$ azure config mode arm
```

Vous pouvez déployer l’extension DSCForLinux en exécutant la commande suivante :
```
$ azure vm extension set <resource-group> <vm-name> \
DSCForLinux Microsoft.OSTCExtensions <version> \
--private-config-path protected.json --public-config-path public.json
```
> [!NOTE]
> En mode Azure Resource Manager, `azure vm extension list` n’est pas disponible pour l’instant.
>

### <a name="use-azure-powershellazure-powershell"></a>Utiliser [Azure PowerShell][azure-powershell]

#### <a name="classic"></a>Classique

Vous pouvez vous connecter à votre compte Azure en mode Azure Service Management en exécutant la commande suivante :

```powershell
Add-AzureAccount
```

Et déployez l’extension DSCForLinux en exécutant la commande suivante :

```powershell
$vmname = '<vm-name>'
$vm = Get-AzureVM -ServiceName $vmname -Name $vmname
$extensionName = 'DSCForLinux'
$publisher = 'Microsoft.OSTCExtensions'
$version = '< version>'
```

Modifier le contenu de $privateConfig et de $publicConfig conformément aux scénarios de la section précédente.
```
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'
```

```
$publicConfig = '{
  "ExtensionAction": "Push",
  "FileUri": "<mof-file-uri>"
}'
```

```
Set-AzureVMExtension -ExtensionName $extensionName -VM $vm -Publisher $publisher `
  -Version $version -PrivateConfiguration $privateConfig `
  -PublicConfiguration $publicConfig | Update-AzureVM
```

#### <a name="resource-manager"></a>Gestionnaire de ressources

Vous pouvez vous connecter à votre compte Azure en mode Azure Resource Manager en exécutant la commande suivante :

```powershell
Login-AzAccount
```

Pour en savoir plus sur l’utilisation d’Azure PowerShell avec Azure Resource Manager, consultez [Gérer des ressources Azure à l’aide d’Azure PowerShell](../../azure-resource-manager/management/manage-resources-powershell.md).

Vous pouvez déployer l’extension DSCForLinux en exécutant la commande suivante :

```powershell
$rgName = '<resource-group-name>'
$vmName = '<vm-name>'
$location = '< location>'
$extensionName = 'DSCForLinux'
$publisher = 'Microsoft.OSTCExtensions'
$version = '< version>'
```

Modifier le contenu de $privateConfig et de $publicConfig conformément aux scénarios de la section précédente.
```
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'
```

```
$publicConfig = '{
  "ExtensionAction": "Push",
  "FileUri": "<mof-file-uri>"
}'
```

```
Set-AzVMExtension -ResourceGroupName $rgName -VMName $vmName -Location $location `
  -Name $extensionName -Publisher $publisher -ExtensionType $extensionName `
  -TypeHandlerVersion $version -SettingString $publicConfig -ProtectedSettingString $privateConfig
```

## <a name="troubleshoot-and-support"></a>Dépannage et support technique

### <a name="troubleshoot"></a>Dépanner

Vous pouvez récupérer les données sur l’état des déploiements d’extension à partir du portail Azure et à l’aide de l’interface de ligne de commande Azure. Pour afficher l’état du déploiement des extensions pour une machine virtuelle donnée, exécutez la commande suivante à l’aide de l’interface de ligne de commande Azure.

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

La sortie de l’exécution de l’extension est enregistrée dans le fichier suivant :

```
/var/log/azure/<extension-name>/<version>/extension.log file.
```

Code d’erreur : 51 correspond soit à une distribution non prise en charge, soit à une action de l’extension non prise en charge.
Dans certains cas, l’extension Linux DSC ne parvient pas à installer OMI lorsqu’une version plus récente d’OMI existe déjà sur la machine. [error response: (000003)Downgrade not allowed]



### <a name="support"></a>Support

Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, vous pouvez contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/community/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/), puis sélectionnez **Obtenir de l’aide**. Pour plus d’informations sur l’utilisation du support Azure, lisez la [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les extensions, consultez la page [Extensions et fonctionnalités de machine virtuelle pour Linux](features-linux.md).
