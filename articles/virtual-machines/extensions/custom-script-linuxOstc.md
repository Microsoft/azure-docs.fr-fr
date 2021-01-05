---
title: Exécuter une extension de script personnalisée version 1 (déconseillée) sur des machines virtuelles Linux dans Azure
description: Automatiser les tâches de configuration de machine virtuelle Linux à l’aide de l’extension de script personnalisé v1
services: virtual-machines-linux
documentationcenter: ''
author: danielsollondon
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.subservice: extensions
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/14/2018
ms.author: danis
ms.openlocfilehash: afce76c8f5330f7d73ff58cc9b9307af2b8832f4
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94962125"
---
# <a name="use-the-azure-custom-script-extension-version-1-with-linux-virtual-machines"></a>Utiliser l’extension de script personnalisé Azure Version 1 avec des machines virtuelles Linux

[!INCLUDE [virtual-machines-extensions-deprecation-statement](../../../includes/virtual-machines-extensions-deprecation-statement.md)]

L’extension de script personnalisé Version 1 télécharge et exécute des scripts sur des machines virtuelles Azure. Elle est utile pour la configuration de post-déploiement, l’installation de logiciels ou toute autre tâche de configuration/gestion. Il est possible de télécharger des scripts à partir du Stockage Azure ou de tout autre emplacement Internet accessible, ou de les fournir au runtime de l’extension.

L’extension de script personnalisé est compatible avec les modèles Azure Resource Manager. Elle est également exécutable avec Azure CLI, PowerShell, le Portail Azure et l’API REST Machines virtuelles Azure.

Cet article explique en détail comment utiliser l’extension de script personnalisé avec Azure CLI et l’exécuter à l’aide d’un modèle Azure Resource Manager. Il indique également la procédure de résolution des problèmes pour les systèmes Linux.

Il existe deux extensions de script personnalisé Linux :

* Version 1 - Microsoft.OSTCExtensions.CustomScriptForLinux

* Version 2 : Microsoft.Azure.Extensions.CustomScript

Basculez vers des déploiements nouveaux et existants pour utiliser la nouvelle version ([Microsoft.Azure.Extensions.CustomScript](custom-script-linux.md)) à la place. Cette nouvelle version se veut être une solution de remplacement instantané. Ainsi, pour procéder à la migration, il vous suffit de changer le nom et la version ; il est inutile de modifier la configuration de l’extension.

### <a name="operating-system"></a>Système d’exploitation

Distributions de Linux prises en charge :

* CentOS 6.5 et versions ultérieures
* Debian 8 et versions ultérieures
  * Debian 8.7 n’intègre pas Python2 dans les images plus récentes, ce qui interrompt CustomScriptForLinux.
* FreeBSD
* OpenSUSE 13.1 et versions ultérieures
* Oracle Linux 6.4 et versions ultérieures
* SUSE Linux Enterprise Server 11 SP3 et versions ultérieures
* Ubuntu 12.04 et versions ultérieures

### <a name="script-location"></a>Emplacement du script

L’extension vous permet d’utiliser vos informations d’identification de stockage d’objets blob Azure pour accéder au stockage d’objets blob Azure. Par ailleurs, le script peut être placé n’importe où, tant que la machine virtuelle peut effectuer le routage vers ce point de terminaison, par exemple GitHub, un serveur de fichiers interne, etc.

### <a name="internet-connectivity"></a>Connectivité Internet

Si vous devez télécharger un script en externe, par exemple à partir de GitHub ou du stockage Azure, vous devez ouvrir des ports de pare-feu/de groupe de sécurité réseau supplémentaires. Par exemple, si votre script se trouve dans le Stockage Azure, vous pouvez en autoriser l’accès à l’aide de balises de service du groupe de sécurité réseau Azure pour le [Stockage](../../virtual-network/network-security-groups-overview.md#service-tags).

Si votre script se trouve sur un serveur local, vous devrez peut-être encore ouvrir des ports de pare-feu/de groupe de sécurité réseau supplémentaires.

### <a name="tips-and-tricks"></a>Astuces et conseils

* Le taux d’échec le plus élevé de cette extension est dû aux erreurs de syntaxe contenues dans le script. Testez les exécutions de script sans erreur et insérez également une journalisation supplémentaire dans le script pour trouver plus facilement l’emplacement de l’échec.
* Écrivez des scripts idempotents ; s’ils sont exécutés plusieurs fois par erreur, ils n’entraîneront pas de modification du système.
* Vérifiez que l’exécution des scripts ne nécessite pas d’entrée utilisateur.
* L’exécution du script est autorisée pendant 90 minutes. Toute exécution d’une durée supérieure entraîne l’échec du provisionnement de l’extension.
* N’insérez pas de redémarrages dans le script, car cela entraîne des problèmes avec d’autres extensions en cours d’installation : après un redémarrage, l’extension ne poursuit pas son exécution. 
* Si l’un de vos scripts provoque un redémarrage, installez des applications, puis exécutez des scripts, etc. Vous devez planifier le redémarrage à l’aide d’une tâche Cron, d’outils tels que DSC ou d’extensions Chef ou Puppet.
* L’extension n’exécute un script qu’une seule fois. Si vous souhaitez exécuter un script à chaque démarrage, vous pouvez utiliser [une image cloud-init](../linux/using-cloud-init.md) et utiliser un module [Scripts au démarrage](https://cloudinit.readthedocs.io/en/latest/topics/modules.html#scripts-per-boot). Vous pouvez également utiliser le script pour créer une unité de service Systemd.
* Si vous souhaitez planifier le moment de l’exécution d’un script, vous devez utiliser l’extension pour créer une tâche Cron.
* Lors de l’exécution du script, vous voyez seulement l’état de l’extension « transition en cours » dans le portail Azure ou l’interface Azure CLI. Si vous souhaitez que les mises à jour de l’état d’un script en cours d’exécution soient plus fréquentes, vous devez créer votre propre solution.
* L’extension de script personnalisé ne prend pas en charge les serveurs proxy en mode natif, en revanche vous pouvez utiliser un outil de transfert de fichiers qui prend en charge les serveurs proxy dans votre script, par exemple *Curl*.
* Tenez compte des emplacements de répertoire autres que par défaut, et qui sont susceptibles d’être utilisés pour vos scripts ou commandes. Gérez cette situation de façon logique.

## <a name="extension-schema"></a>Schéma d’extensions

La configuration de l’extension de script personnalisé spécifie des éléments tels que l’emplacement du script et la commande à exécuter. Cette configuration peut être stockée dans des fichiers de configuration, ou spécifiée en ligne de commande ou dans un modèle Azure Resource Manager. 

Les données sensibles peuvent être stockées dans une configuration protégée qui n’est chiffrée et déchiffrée qu’à l’intérieur de la machine virtuelle. La configuration protégée est utile lorsque la commande d’exécution comprend des secrets tels qu’un mot de passe.

Ces éléments doivent être traités comme des données sensibles et spécifiés dans la configuration du paramètre de protection des extensions. Les données du paramètre de protection de l’extension de machine virtuelle Azure sont chiffrées et ne sont déchiffrées que sur la machine virtuelle cible.

```json
{
  "name": "config-app",
  "type": "extensions",
  "location": "[resourceGroup().location]",
  "apiVersion": "2015-06-15",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
  ],
  "tags": {
    "displayName": "config-app"
  },
  "properties": {
    "publisher": "Microsoft.OSTCExtensions",
    "type": "CustomScriptForLinux",
    "typeHandlerVersion": "1.5",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": [
        "<url>"
      ],
      "enableInternalDNSCheck": true
    },
    "protectedSettings": {
      "storageAccountName": "<storage-account-name>",
      "storageAccountKey": "<storage-account-key>",
      "commandToExecute": "<command>"
    }
  }
}
```

### <a name="property-values"></a>Valeurs de propriétés

| Nom | Valeur/Exemple | Type de données |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | Date |
| publisher | Microsoft.OSTCExtensions | string |
| type | CustomScriptForLinux | string |
| typeHandlerVersion | 1.5 | int |
| fileUris (p. ex.) | `https://github.com/MyProject/Archive/MyPythonScript.py` | tableau |
| commandToExecute (p. ex.) | python MyPythonScript.py \<my-param1\> | string |
| enableInternalDNSCheck | true | boolean |
| storageAccountName (p. ex.) | examplestorageacct | string |
| storageAccountKey (p. ex.) | TmJK/1N3AbAZ3q/+hOXoi/l73zOqsaxXDhqa9Y83/v5UpXQp2DQIBuv2Tifp60cE/OaHsJZmQZ7teQfczQj8hg== | string |

### <a name="property-value-details"></a>Détails des valeurs de propriété

* `fileUris` : (facultatif, tableau de chaînes) liste d’uri des scripts
* `enableInternalDNSCheck` : (facultatif, booléen) par défaut est True, définir sur False pour désactiver la vérification DNS
* `commandToExecute` : (facultatif, chaîne) script de point d’entrée à exécuter
* `storageAccountName` : (facultatif, chaîne) nom du compte de stockage
* `storageAccountKey` : (facultatif, chaîne) clé d’accès du compte de stockage

Les valeurs suivantes peuvent être définies dans les paramètres publics ou protégés, mais elles ne doivent pas être définies dans les deux à la fois.

* `commandToExecute`

L’utilisation des paramètres publics peut être utile pour le débogage, mais il est vivement recommandé d’utiliser des paramètres protégés.

Les paramètres publics sont envoyés en texte clair à la machine virtuelle sur laquelle le script est exécuté.  Les paramètres protégés sont chiffrés à l’aide d’une clé connue uniquement d’Azure et de la machine virtuelle. Les paramètres sont enregistrés sur la machine virtuelle tels qu’ils ont été envoyés. Autrement dit, si les paramètres ont été chiffrés, ils sont enregistrés chiffrés sur la machine virtuelle. Le certificat utilisé pour déchiffrer les valeurs chiffrées est stocké sur la machine virtuelle et permet de déchiffrer les paramètres (si nécessaire) lors de l’exécution.

## <a name="template-deployment"></a>Déploiement de modèle

Les extensions de machines virtuelles Azure peuvent être déployées avec des modèles Azure Resource Manager. Le schéma JSON détaillé dans la section précédente peut être utilisé dans un modèle Azure Resource Manager pour exécuter l’extension de script personnalisé pendant un déploiement de modèle Azure Resource Manager.

```json
{
  "name": "config-app",
  "type": "extensions",
  "location": "[resourceGroup().location]",
  "apiVersion": "2015-06-15",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
  ],
  "tags": {
    "displayName": "config-app"
  },
  "properties": {
    "publisher": "Microsoft.OSTCExtensions",
    "type": "CustomScriptForLinux",
    "typeHandlerVersion": "1.5",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": ["http://MyAccount.blob.core.windows.net/vhds/MyShellScript.sh"]
    },
    "protectedSettings": {
      "storageAccountName": "MyAccount",
      "storageAccountKey": "<storage-account-key>",
      "commandToExecute": "sh MyShellScript.sh"
    }
  }
}
```

>[!NOTE]
>Ces noms de propriétés respectent la casse. Pour éviter des problèmes de déploiement, utilisez les noms présentés ici.

## <a name="azure-cli"></a>Azure CLI

Si vous utilisez Azure CLI pour exécuter l’extension de script personnalisé, créez un fichier de configuration ou des fichiers. Vous devez indiquer au moins la propriété « commandToExecute ».

```azurecli
az vm extension set -n VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.5 \
  --vm-name MyVm --resource-group MyResourceGroup \
  --protected-settings '{"commandToExecute": "echo hello"}'
```

Vous pouvez éventuellement spécifier les paramètres dans la commande sous forme de chaînes au format JSON. Cela permet de spécifier la configuration lors de l’exécution et sans fichier de configuration distinct.

```azurecli
az vm extension set \
  --resource-group exttest \
  --vm-name exttest \
  --name CustomScriptForLinux \
  --publisher Microsoft.OSTCExtensions \
  --settings '{"fileUris": ["https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"],"commandToExecute": "./config-music.sh"}'
```

### <a name="azure-cli-examples"></a>Exemples d’interface de ligne de commande Azure

#### <a name="public-configuration-with-no-script-file"></a>Configuration publique sans fichier de script

```json
{
  "commandToExecute": "apt-get -y update && apt-get install -y apache2"
}
```

Commande d’interface de ligne de commande Azure :

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name CustomScriptForLinux \
  --publisher Microsoft.OSTCExtensions \
  --settings ./script-config.json
```

#### <a name="public-and-protected-configuration-files"></a>Fichiers de configuration publique et protégée

Un fichier de configuration publique permet de spécifier l’URI du fichier de script. Un fichier de configuration protégée permet de spécifier la commande à exécuter.

Fichier de configuration publique :

```json
{
  "fileUris": ["https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"]
}
```

Fichier de configuration protégée :  

```json
{
  "commandToExecute": "./config-music.sh <param1>"
}
```

Commande d’interface de ligne de commande Azure :

```azurecli
az vm extension set
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name CustomScriptForLinux \
  --publisher Microsoft.OSTCExtensions \
  --settings ./script-config.json \
  --protected-settings ./protected-config.json
```

## <a name="troubleshooting"></a>Dépannage

Lors de l’exécution de l’extension de script personnalisé, le script est créé ou téléchargé dans un répertoire semblable à l’exemple suivant. La sortie de la commande y est également enregistrée, dans les fichiers `stdout` et `stderr`.

```bash
/var/lib/waagent/Microsoft.OSTCExtensions.CustomScriptForLinux-<version>/download/1
```

Pour résoudre les problèmes, recherchez d’abord le journal de l’agent Linux, assurez-vous de l’exécution de l’extension, puis vérifiez :

```bash
/var/log/waagent.log
```

Vous devez rechercher l’exécution de l’extension, qui doit se présenter ainsi :

```output
2018/04/26 15:29:44.835067 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Target handler state: enabled
2018/04/26 15:29:44.867625 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] [Enable] current handler state is: notinstalled
2018/04/26 15:29:44.959605 INFO Event: name=Microsoft.OSTCExtensions.CustomScriptForLinux, op=Download, message=Download succeeded, duration=59
2018/04/26 15:29:44.993269 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Initialize extension directory
2018/04/26 15:29:45.022972 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Update settings file: 0.settings
2018/04/26 15:29:45.051763 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Install extension [customscript.py -install]
2018/04/26 15:29:45 CustomScriptForLinux started to handle.
2018/04/26 15:29:45 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] cwd is /var/lib/waagent/Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2
2018/04/26 15:29:45 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Change log file to /var/log/azure/Microsoft.OSTCExtensions.CustomScriptForLinux/1.5.2.2/extension.log
2018/04/26 15:29:46.088212 INFO Event: name=Microsoft.OSTCExtensions.CustomScriptForLinux, op=Install, message=Launch command succeeded: customscript.py -install, duration=1005
2018/04/26 15:29:46.133367 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Enable extension [customscript.py -enable]
2018/04/26 15:29:46 CustomScriptForLinux started to handle.
..
2018/04/26 15:29:47.178163 INFO Event: name=Microsoft.OSTCExtensions.CustomScriptForLinux, op=Enable, message=Launch command succeeded: customscript.py -enable, duration=1012
```

Points à noter :

1. La commande Enable correspond au début de l’exécution de la commande.
1. La commande Download se rapporte au téléchargement du package d’extension CustomScript à partir d’Azure, et non aux fichiers de script spécifiés dans fileUris.
1. Vous pouvez aussi voir dans quel fichier il est écrit, `/var/log/azure/Microsoft.OSTCExtensions.CustomScriptForLinux/1.5.2.2/extension.log`

L’étape suivante consiste à rechercher le fichier journal, selon le format suivant :

```bash
/var/log/azure/<extension-name>/<version>/extension.log file.
```

Vous devez rechercher l’exécution individuelle, qui doit se présenter ainsi :

```output
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Enable,transitioning,0,Launching the script...
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] sequence number is 0
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] setting file path is/var/lib/waagent/Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2/config/0.settings
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] JSON config: {"runtimeSettings": [{"handlerSettings": {"protectedSettings": "MIIB0AYJKoZIhvcNAQcDoIIBwTCCAb0CAQAxggF+hnEXRtFKTTuKiFC8gTfHKupUSs7qI0zFYRya", "publicSettings": {"fileUris": ["https://dannytesting.blob.core.windows.net/demo/myBash.sh"]}, "protectedSettingsCertThumbprint": "4385AB21617C2452FF6998C0A37F71A0A01C8368"}}]}
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Config decoded correctly.
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Will try to download files, number of retries = 10, wait SECONDS between retrievals = 20s
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Downloading,transitioning,0,Downloading files...
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] No azure storage account and key specified in protected settings. Downloading scripts from external links...
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Converting /var/lib/waagent/Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2/download/0/myBash.sh from DOS to Unix formats: Done
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Removing BOM of /var/lib/waagent/Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2/download/0/myBash.sh: Done
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Succeeded to download files, retry count = 0
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Internal DNS is ready, retry count = 0
2018/04/26 15:29:47 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Command is finished.
2018/04/26 15:29:47 ---stdout---
2018/04/26 15:29:47
2018/04/26 15:29:47 ---errout---
2018/04/26 15:29:47
2018/04/26 15:29:47
2018/04/26 15:29:47 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Daemon,success,0,Command is finished.
2018/04/26 15:29:47 ---stdout---
2018/04/26 15:29:47
2018/04/26 15:29:47 ---errout---
2018/04/26 15:29:47
2018/04/26 15:29:47
```

Là, vous pouvez voir :

* La commande Enable démarrant dans ce journal
* Les paramètres transmis à l’extension
* L’extension téléchargeant des fichiers et le résultat de cette opération
* La commande en cours d’exécution et le résultat

Il est également possible de récupérer l’état d’exécution de l’extension de script personnalisé avec Azure CLI :

```azurecli
az vm extension list -g myResourceGroup --vm-name myVM
```

La sortie ressemble au texte suivant :

```output
Name                  ProvisioningState    Publisher                   Version  AutoUpgradeMinorVersion
--------------------  -------------------  ------------------------  ---------  -------------------------
CustomScriptForLinux  Succeeded            Microsoft.OSTCExtensions        1.5  True
```

## <a name="next-steps"></a>Étapes suivantes

Pour afficher le code, les problèmes et les versions en cours, consultez [Référentiel de l’extension CustomScript](https://github.com/Azure/azure-linux-extensions/tree/master/CustomScript).