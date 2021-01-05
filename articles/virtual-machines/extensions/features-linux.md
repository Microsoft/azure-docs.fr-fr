---
title: Extensions et fonctionnalités des machines virtuelles Azure pour Linux
description: Découvrez les extensions disponibles pour les machines virtuelles sous Linux, regroupées par ce qu’elles fournissent ou améliorent.
services: virtual-machines-linux
documentationcenter: ''
author: axayjo
manager: gwallace
editor: ''
tags: azure-service-management,azure-resource-manager
ms.assetid: 52f5d0ec-8f75-49e7-9e15-88d46b420e63
ms.service: virtual-machines-linux
ms.subservice: extensions
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/30/2018
ms.author: akjosh
ms.openlocfilehash: 129897d3288a900803efbfba8abf86c276077fa8
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966069"
---
# <a name="virtual-machine-extensions-and-features-for-linux"></a>Extensions et fonctionnalités de machine virtuelle pour Linux

Les extensions de machine virtuelle Azure sont de petites applications permettant d’exécuter des tâches de configuration et d’automatisation post-déploiement sur des machines virtuelles Azure. Par exemple, si une machine virtuelle nécessite l’installation d’un logiciel, une protection antivirus ou l’exécution d’un script, vous pouvez utiliser une extension de machine virtuelle. Les extensions de machine virtuelle Azure sont exécutables avec l’interface de ligne de commande Azure, PowerShell, les modèles Azure Resource Manager et le portail Azure. Les extensions peuvent être intégrées à un nouveau déploiement de machine virtuelle ou s’exécuter sur tout système existant.

Cet article offre une vue d’ensemble des extensions de machine virtuelle et des prérequis pour l’utilisation d’extensions de machine virtuelle Azure. Il explique également comment détecter, gérer et supprimer des extensions de machine virtuelle. Cet article fournit des informations générales, car de nombreuses extensions de machine virtuelle sont disponibles, chacune présentant une configuration potentiellement unique. Vous trouverez des informations détaillées sur une extension spécifique dans la documentation consacrée à celle-ci.

## <a name="use-cases-and-samples"></a>Cas d’utilisation et exemples

Plusieurs extensions de machine virtuelle Azure sont disponibles, chacune impliquant un cas d’utilisation spécifique. Voici quelques exemples :

- Appliquer des configurations DSC (Desired State Configuration) PowerShell à une machine virtuelle avec l’extension DSC pour Linux. Pour plus d’informations sur l’extension DSC Azure, consultez [cette page](https://github.com/Azure/azure-linux-extensions/tree/master/DSC) (en anglais).
- Configurer l’analyse d’une machine virtuelle avec l’extension de machine virtuelle Microsoft Monitoring Agent. Pour plus d’informations, voir [How to monitor a Linux VM](../linux/tutorial-monitor.md) (Surveillance d’une machine virtuelle Linux).
- Configurer l’analyse de votre infrastructure Azure à l’aide de l’extension Chef ou Datadog. Pour plus d’informations, consultez les [documents Chef](https://docs.chef.io/azure_portal.html) ou le [blog Datadog](https://www.datadoghq.com/blog/introducing-azure-monitoring-with-one-click-datadog-deployment/).

En plus des extensions propres à des processus, une extension de script personnalisé est disponible pour les machines virtuelles Windows et Linux. L’extension de script personnalisé pour Linux permet d’exécuter n’importe quel script Bash sur une machine virtuelle. Les scripts personnalisés s’avèrent utile pour concevoir des déploiements Azure qui nécessitent une configuration plus avancée que celle fournie par les outils Azure natifs. Pour plus d’informations sur l’extension de script personnalisé pour les machines virtuelles Linux, consultez [cet article](custom-script-linux.md).

## <a name="prerequisites"></a>Prérequis

La gestion de l’extension sur la machine virtuelle nécessite l’installation préalable de l’agent Linux Azure. Certaines extensions spécifiques présentent des prérequis, tels que l’accès à des ressources ou dépendances.

### <a name="azure-vm-agent"></a>Agent de machine virtuelle Azure

L’agent de machine virtuelle Azure gère les interactions entre une machine virtuelle Azure et le contrôleur de structure Microsoft Azure. L’agent de machine virtuelle est responsable de nombreux aspects fonctionnels liés au déploiement et à la gestion des machines virtuelles Azure, notamment l’exécution des extensions de machine virtuelle. L’agent de machine virtuelle Azure est préinstallé sur des images de Place de marché Azure et peut être installé manuellement sur les systèmes d’exploitation pris en charge. L’agent de machine virtuelle Azure pour Linux est connu sous le terme d’agent Linux.

Pour plus d’informations sur les systèmes d’exploitation pris en charge et sur la procédure d’installation, consultez l’article [Agent de machine virtuelle et extensions Azure](agent-linux.md).

#### <a name="supported-agent-versions"></a>Versions d’agent prises en charge

Des versions minimales de l’agent sont disponibles afin d’offrir la meilleure expérience possible. Pour plus d’informations, consultez [cet article](https://support.microsoft.com/en-us/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support).

#### <a name="supported-oses"></a>Systèmes d’exploitation pris en charge

L’agent Linux s’exécute sur plusieurs systèmes d’exploitation ; toutefois, l’infrastructure des extensions présente certaines limites relatives aux systèmes d’exploitation pris en charge par les extensions. Pour plus d’informations, consultez [cet article](https://support.microsoft.com/en-us/help/4078134/azure-extension-supported-operating-systems
).

Certaines extensions ne sont pas prises en charge sur tous les systèmes d’exploitation et peuvent générer le *Code d’erreur 51, « Système d’exploitation non pris en charge »* . Pour plus d’informations sur la capacité de prise en charge, consultez la documentation de l’extension concernée.

#### <a name="network-access"></a>Accès réseau

Les paquets d’extensions sont téléchargés à partir du dépôt d’extensions Stockage Azure, et les chargements d’état d’extension sont publiés dans le service Stockage Azure. Si vous utilisez une version [prise en charge](https://support.microsoft.com/en-us/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support) des agents, vous n’avez pas besoin d’autoriser l’accès au service Stockage Azure dans la région de machine virtuelle, car vous pouvez utiliser l’agent pour rediriger la communication vers le contrôleur de structure Azure pour les communications d’agent. Si vous utilisez une version non prise en charge de l’agent, vous devez autoriser l’accès sortant vers le service Stockage Azure dans cette région à partir de la machine virtuelle.

> [!IMPORTANT]
> Si vous avez bloqué l’accès à l’adresse *168.63.129.16* à l’aide du pare-feu invité, les extensions échouent, que vous utilisiez, ou non, une version prise en charge.

Les agents peuvent être utilisés uniquement pour télécharger les paquets d’extensions et signaler l’état. Par exemple, si une installation d’extension doit télécharger un script à partir de GitHub (script personnalisé) ou accéder au service Stockage Azure (sauvegarde Azure), vous devez ouvrir des ports de pare-feu/de groupe de sécurité réseau (NSG) supplémentaires. Les exigences varient selon les extensions, car ces dernières sont des applications à part entière. Dans le cas des extensions qui exigent un accès au service Stockage Azure, vous pouvez autoriser cet accès à l’aide de balises de service NSG Azure pour le [Stockage](../../virtual-network/network-security-groups-overview.md#service-tags).

Pour rediriger les requêtes de trafic d’agent, l’agent Linux prend en charge les serveurs proxy. Toutefois, cette prise en charge des serveurs proxy ne s’applique pas aux extensions. Vous devez configurer chaque extension individuelle pour qu’elle fonctionne avec un proxy.

## <a name="discover-vm-extensions"></a>Détecter les extensions de machine virtuelle

De nombreuses extensions de machine virtuelle différentes peuvent être utilisées avec les machines virtuelles Azure. Pour en obtenir la liste complète, utilisez la commande [az vm extension image list](/cli/azure/vm/extension/image#az-vm-extension-image-list). L’exemple suivant liste toutes les extensions disponibles à l’emplacement *westus* :

```azurecli
az vm extension image list --location westus --output table
```

## <a name="run-vm-extensions"></a>Exécuter les extensions de machine virtuelle

Les extensions de machine virtuelle Azure s’exécutent sur des machines virtuelles existantes, ce qui se révèle utile quand vous devez apporter des changements de configuration ou restaurer la connectivité sur une machine virtuelle déjà déployée. Les extensions de machines virtuelles peuvent également être intégrées dans des déploiements de modèles Azure Resource Manager. L’utilisation d’extensions avec des modèles Resource Manager vous permet de déployer et de configurer des machines virtuelles Azure sans avoir à intervenir après le déploiement.

Vous pouvez exécuter une extension sur une machine virtuelle existante à l’aide des méthodes ci-après.

### <a name="azure-cli"></a>Azure CLI

Vous pouvez exécuter les extensions de machine virtuelle Azure sur une machine virtuelle existante avec la commande [az vm extension set](/cli/azure/vm/extension#az-vm-extension-set). L’exemple suivant exécute l’extension de script personnalisé sur une machine virtuelle nommée *myVM* dans un groupe de ressources nommé *myResourceGroup*. Remplacez l’exemple de nom de groupe de ressources, le nom de la machine virtuelle et le script à exécuter (https:\//raw.githubusercontent.com/me/project/hello.sh) par vos propres informations. 

```azurecli
az vm extension set `
  --resource-group myResourceGroup `
  --vm-name myVM `
  --name customScript `
  --publisher Microsoft.Azure.Extensions `
  --settings '{"fileUris": ["https://raw.githubusercontent.com/me/project/hello.sh"],"commandToExecute": "./hello.sh"}'
```

Quand l’extension s’exécute correctement, la sortie ressemble à l’exemple suivant :

```bash
info:    Executing command vm extension set
+ Looking up the VM "myVM"
+ Installing extension "CustomScript", VM: "mvVM"
info:    vm extension set command OK
```

### <a name="azure-portal"></a>Portail Azure

Vous pouvez appliquer les extensions de machine virtuelle à une machine virtuelle existante par le biais du portail Azure. Sélectionnez la machine virtuelle dans le portail, choisissez **Extensions**, puis sélectionnez **Ajouter**. Choisissez l’extension souhaitée dans la liste des extensions disponibles, puis suivez les instructions de l’Assistant.

L’image suivante illustre l’installation de l’extension de script personnalisé Linux à partir du portail Azure :

![Installer l’extension de script personnalisé](./media/features-linux/installscriptextensionlinux.png)

### <a name="azure-resource-manager-templates"></a>Modèles Microsoft Azure Resource Manager

Les extensions de machine virtuelle peuvent être ajoutées à un modèle Azure Resource Manager et exécutées avec le déploiement du modèle. Lorsque vous déployez une extension avec un modèle, vous pouvez créer des déploiements Azure entièrement configurés. Par exemple, le code JSON suivant est tiré d’un modèle Resource Manager qui déploie un ensemble de machines virtuelles à charge équilibrée et Azure SQL Database, puis installe une application .NET Core sur chaque machine virtuelle. L’extension de machine virtuelle se charge de l’installation du logiciel.

Pour plus d’informations, consultez le [modèle Resource Manager complet](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-linux).

```json
{
    "apiVersion": "2015-06-15",
    "type": "extensions",
    "name": "config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
    ],
    "tags": {
    "displayName": "config-app"
    },
    "properties": {
    "publisher": "Microsoft.Azure.Extensions",
    "type": "CustomScript",
    "typeHandlerVersion": "2.0",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"
        ]
    },
    "protectedSettings": {
        "commandToExecute": "[concat('sudo sh config-music.sh ',variables('musicStoreSqlName'), ' ', parameters('adminUsername'), ' ', parameters('sqlAdminPassword'))]"
    }
    }
}
```

Pour en savoir plus sur la création de Resource Manager, consultez [Création de modèles Azure Resource Manager](../windows/template-description.md#extensions).

## <a name="secure-vm-extension-data"></a>Sécuriser les données des extensions de machine virtuelle

Quand vous exécutez une extension de machine virtuelle, vous pouvez avoir besoin d’inclure des informations sensibles telles que des informations d’identification, des noms de compte de stockage et des clés d’accès à des comptes de stockage. De nombreuses extensions de machine virtuelle comprennent une configuration protégée qui chiffre les données et les déchiffre uniquement à l’intérieur de la machine virtuelle cible. Chaque extension possède un schéma spécifique de configuration protégée, présenté en détail dans la documentation consacrée à l’extension.

L’exemple suivant illustre une instance de l’extension de script personnalisé pour Linux. La commande à exécuter inclut un ensemble d’informations d’identification. Dans cet exemple, la commande à exécuter n’est pas chiffrée :

```json
{
  "apiVersion": "2015-06-15",
  "type": "extensions",
  "name": "config-app",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
  ],
  "tags": {
    "displayName": "config-app"
  },
  "properties": {
    "publisher": "Microsoft.Azure.Extensions",
    "type": "CustomScript",
    "typeHandlerVersion": "2.0",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"
      ],
      "commandToExecute": "[concat('sudo sh config-music.sh ',variables('musicStoreSqlName'), ' ', parameters('adminUsername'), ' ', parameters('sqlAdminPassword'))]"
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
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
  ],
  "tags": {
    "displayName": "config-app"
  },
  "properties": {
    "publisher": "Microsoft.Azure.Extensions",
    "type": "CustomScript",
    "typeHandlerVersion": "2.0",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"
      ]
    },
    "protectedSettings": {
      "commandToExecute": "[concat('sudo sh config-music.sh ',variables('musicStoreSqlName'), ' ', parameters('adminUsername'), ' ', parameters('sqlAdminPassword'))]"
    }
  }
}
```

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

#### <a name="agent-updates"></a>Mises à jour d’agent

L’agent de machine virtuelle Linux contient dans un seul et même package le *code de l’agent de provisionnement* et le *code de gestion des extensions*, qui ne peuvent pas être séparés. Vous pouvez désactiver l’*agent de provisionnement* quand vous voulez effectuer un provisionnement sur Azure à l’aide de cloud-init. Pour ce faire, consultez [Utilisation de cloud-init](../linux/using-cloud-init.md).

Les versions prises en charge des agents peuvent utiliser des mises à jour automatiques. Le seul code pouvant être mis à jour est le *code de gestion des extensions*, pas le code de provisionnement. Le *code de l’agent de provisionnement* est un code à exécution unique.

Le *code de gestion des extensions* assure la communication avec la structure Azure, ainsi que la gestion des opérations des extensions de machine virtuelle, telles que les installations, le signalement d’état, la mise à jour des différentes extensions et la suppression de ces dernières. Les mises à jour comprennent les correctifs de sécurité, les corrections de bogues et les améliorations du *code de gestion des extensions*.

Quand l’agent est installé, un démon parent est créé. Ce parent génère ensuite un processus enfant qui est utilisé pour gérer les extensions. Si une mise à jour est disponible pour l’agent, elle est téléchargée, le parent arrête le processus enfant, le met à niveau, puis le redémarre. En cas de problème avec la mise à jour, le processus parent restaure la version enfant précédente.

Le processus parent ne peut pas être mis à jour automatiquement. Le parent peut uniquement être mis à jour par une mise à jour du package de distribution.

Pour vérifier quelle version vous exécutez, vérifiez la commande `waagent` comme suit :

```bash
waagent --version
```

Le résultat ressemble à l’exemple suivant :

```bash
WALinuxAgent-2.2.17 running on ubuntu 16.04
Python: 3.5.2
Goal state agent: 2.2.18
```

Dans la sortie de l’exemple précédent, le parent ou la version de package déployé est *WALinuxAgent-2.2.17*

La valeur « Goal state agent » est la version de la mise à jour automatique.

Nous vous recommandons vivement que la mise à jour automatique soit toujours activée pour l’agent, [AutoUpdate.Enabled=y](./update-linux-agent.md). Dans le cas contraire, vous devez conserver manuellement la mise à jour de l’agent, et ne pas obtenir les correctifs de sécurité et les corrections de bogues.

#### <a name="extension-updates"></a>Mises à jour des extensions

Quand une mise à jour d’extension est disponible, l’agent invité Linux la télécharge et met à niveau l’extension. Les mises à jour d’extension automatiques sont soit de type *Minor* (mise à jour mineure), soit de type *Hotfix* (correctif logiciel). Vous pouvez accepter ou refuser les mises à jour d’extension *Minor* quand vous provisionnez l’extension. L’exemple ci-après indique comment mettre à niveau automatiquement les versions mineures dans un modèle Resource Manager avec la commande *"autoUpgradeMinorVersion": true,* :

```json
    "publisher": "Microsoft.Azure.Extensions",
    "type": "CustomScript",
    "typeHandlerVersion": "2.0",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"
        ]
    },
```

Si vous souhaitez obtenir les dernières corrections de bogues des versions mineures, nous vous recommandons vivement de toujours sélectionner l’option de mise à jour automatique dans vos déploiements d’extensions. Vous ne pouvez pas refuser les mises à jour de type correctif logiciel qui comportent des correctifs de sécurité ou des corrections de bogues clés.

### <a name="how-to-identify-extension-updates"></a>Comment identifier les mises à jour d’extension

#### <a name="identifying-if-the-extension-is-set-with-autoupgrademinorversion-on-a-vm"></a>Vérification si l’extension est définie avec la propriété autoUpgradeMinorVersion sur une machine virtuelle

Le modèle de machine virtuelle vous permet de déterminer si l’extension a été provisionnée avec la propriété "autoUpgradeMinorVersion". Pour vérifier ce point, utilisez la commande [az vm show](/cli/azure/vm#az-vm-show), et indiquez le groupe de ressources et le nom de machine virtuelle comme suit :

```azurecli
az vm show --resource-group myResourceGroup --name myVM
```

La sortie de l’exemple suivant indique que la propriété *autoUpgradeMinorVersion* est définie sur la valeur *true* :

```json
  "resources": [
    {
      "autoUpgradeMinorVersion": true,
      "forceUpdateTag": null,
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM/extensions/CustomScriptExtension",
```

#### <a name="identifying-when-an-autoupgrademinorversion-occurred"></a>Identification du moment d’exécution d’une mise à jour de type autoUpgradeMinorVersion

Pour déterminer le moment où une extension a fait l’objet d’une mise à jour, consultez les journaux d’activité de l’agent sur la machine virtuelle à l’emplacement */var/log/waagent.log*.

Dans l’exemple ci-dessous, *Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9025* était installé sur la machine virtuelle. Un correctif logiciel est disponible pour *Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027* :

```bash
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Expected handler state: enabled
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Decide which version to use
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Use version: 2.3.9027
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Current handler state is: NotInstalled
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Download extension package
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Unpack extension package
INFO Event: name=Microsoft.OSTCExtensions.LinuxDiagnostic, op=Download, message=Download succeeded
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Initialize extension directory
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Update settings file: 0.settings
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9025] Disable extension.
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9025] Launch command:diagnostic.py -disable
...
INFO Event: name=Microsoft.OSTCExtensions.LinuxDiagnostic, op=Disable, message=Launch command succeeded: diagnostic.py -disable
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Update extension.
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Launch command:diagnostic.py -update
2017/08/14 20:21:57 LinuxAzureDiagnostic started to handle.
```

## <a name="agent-permissions"></a>Autorisations d’agent

Pour exécuter ses tâches, l’agent doit s’exécuter comme *racine*.

## <a name="troubleshoot-vm-extensions"></a>Résoudre les problèmes liés aux extensions de machine virtuelle

Chaque extension de machine virtuelle peut présenter une procédure de résolution des problèmes spécifique. Par exemple, quand vous utilisez l’extension de script personnalisé, les détails de l’exécution du script sont accessibles localement sur la machine virtuelle utilisée pour l’exécution de l’extension. La procédure de résolution des problèmes spécifique d’une extension est présentée en détail dans la documentation de cette dernière.

La procédure de résolution des problèmes ci-après s’applique à toutes les extensions de machine virtuelle.

1. Pour consulter le journal de l’agent Linux, examinez l’activité au moment où votre extension a été provisionnée dans */var/log/waagent.log*

2. Pour plus de détails, vérifiez les journaux d’activité d’extension proprement dits, à l’emplacement */var/log/azure/\<extensionName>*

3. Consultez les sections de résolution des problèmes dans la documentation de l’extension concernée pour vous renseigner sur les codes d’erreur, les problèmes connus, etc.

3. Examinez les journaux d’activité système. Vérifiez si d’autres opérations sont susceptibles d’avoir interféré avec l’extension, par exemple une longue installation d’une autre application nécessitant un accès exclusif au gestionnaire de package.

### <a name="common-reasons-for-extension-failures"></a>Motifs courants des échecs d’extension

1. Délai d’exécution des extensions limité à 20 minutes (étendu à 90 minutes pour les extensions CustomScript, Chef et DSC). Si votre déploiement excède ce délai, il est signalé comme dépassement de délai d’expiration. Ce dépassement peut être dû à une insuffisance de ressources sur les machines virtuelles, découlant du fait que d’autres configurations/tâches de démarrage de machine virtuelle consomment de grandes quantités de ressources alors qu’une tentative de provisionnement d’une extension est en cours.

2. Prérequis minimaux non respectés. Certaines extensions présentent des dépendances vis-à-vis des références SKU de machine virtuelle, telles que les images HPC. Les extensions peuvent imposer certaines exigences d’accès réseau, comme la communication avec le service Stockage Azure ou les services publics. D’autres exemples de prérequis concernent l’accès aux référentiels de packages, l’espace disque nécessaire ou les restrictions de sécurité.

3. Accès exclusif au gestionnaire de package. Dans certains cas, une configuration de machine virtuelle longue peut entrer en conflit avec l’installation d’une extension, quand ces deux tâches exigent un accès exclusif au gestionnaire de package.

### <a name="view-extension-status"></a>Afficher l’état de l’extension

Une fois qu’une extension de machine virtuelle a été exécutée sur une machine virtuelle, utilisez la commande [az vm get-instance-view](/cli/azure/vm#az-vm-get-instance-view) pour retourner l’état de l’extension comme suit :

```azurecli
az vm get-instance-view \
    --resource-group rgName \
    --name myVM \
    --query "instanceView.extensions"
```

Le résultat obtenu ressemble à l’exemple de sortie suivant :

```bash
  {
    "name": "customScript",
    "statuses": [
      {
        "code": "ProvisioningState/failed/0",
        "displayStatus": "Provisioning failed",
        "level": "Error",
        "message": "Enable failed: failed to execute command: command terminated with exit status=127\n[stdout]\n\n[stderr]\n/bin/sh: 1: ech: not found\n",
        "time": null
      }
    ],
    "substatuses": null,
    "type": "Microsoft.Azure.Extensions.customScript",
    "typeHandlerVersion": "2.0.6"
  }
```

L’état d’exécution de l’extension est également visible dans le portail Azure. Pour visualiser l’état d’une extension, sélectionnez la machine virtuelle, choisissez **Extensions**, puis sélectionnez l’extension souhaitée.

### <a name="rerun-a-vm-extension"></a>Réexécuter une extension de machine virtuelle

Dans certains cas, vous pouvez avoir besoin de réexécuter une extension de machine virtuelle. Pour ce faire, supprimez-la, puis réexécutez-la avec la méthode d’exécution de votre choix. Pour supprimer une extension, utilisez [az vm extension delete](/cli/azure/vm/extension#az-vm-extension-delete) comme suit :

```azurecli
az vm extension delete \
    --resource-group myResourceGroup \
    --vm-name myVM \
    --name customScript
```

Vous pouvez également supprimer une extension dans le portail Azure en procédant comme suit :

1. Sélectionnez une machine virtuelle.
2. Choisissez **Extensions**.
3. Sélectionnez l’extension souhaitée.
4. Choisissez **Désinstaller**.

## <a name="common-vm-extension-reference"></a>Informations de référence sur les extensions de machine virtuelle courantes

| Nom de l’extension | Description | Informations complémentaires |
| --- | --- | --- |
| Extension de script personnalisé pour Linux |Exécuter des scripts sur une machine virtuelle Azure |[Extension de script personnalisé pour Linux](custom-script-linux.md) |
| Extension d’accès aux machines virtuelles |Récupérer l’accès à une machine virtuelle Azure |[Extension d’accès aux machines virtuelles](https://github.com/Azure/azure-linux-extensions/tree/master/VMAccess) |
| Extension Diagnostics Azure |Gérer les diagnostics Azure |[Extension Diagnostics Azure](https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/) |
| Extension d’accès aux machines virtuelles Azure |Gérer les utilisateurs et les informations d’identification |[Extension d’accès aux machines virtuelles pour Linux](https://azure.microsoft.com/blog/using-vmaccess-extension-to-reset-login-credentials-for-linux-vm/) |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les extensions de machine virtuelle, consultez [Vue d’ensemble des extensions et des fonctionnalités des machines virtuelles Azure](overview.md).