---
title: Résolution des problèmes de l’agent invité Linux Azure
description: Résoudre les problèmes de fonctionnement de l’agent invité Linux Azure
services: virtual-machines-linux
ms.service: virtual-machines-linux
author: axelg
manager: dcscontentpm
editor: ''
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 11/17/2020
ms.author: axelg
ms.openlocfilehash: fc609b60c9d5d4d4734c3d73cbda87935b533caf
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500255"
---
# <a name="troubleshooting-azure-linux-guest-agent"></a>Résolution des problèmes de l’agent invité Linux Azure

[L’agent invité Linux Azure](../extensions/agent-linux.md) est un agent de machine virtuelle. Il permet à la machine virtuelle de communiquer avec le contrôleur de structure (serveur physique sous-jacent sur lequel la machine virtuelle est hébergée) via l’adresse IP 168.63.129.16. Cette adresse IP est une adresse IP publique virtuelle qui facilite la communication. Pour plus d’informations, consultez [Qu’est-ce que l’adresse IP 168.63.129.16 ?](../../virtual-network/what-is-ip-address-168-63-129-16.md)

## <a name="checking-agent-status-and-version"></a>Vérification de l’état et de la version de l’agent

Voir https://github.com/Azure/WALinuxAgent/wiki/FAQ#what-does-goal-state-agent-mean-in-waagent---version-output

## <a name="troubleshooting-vm-agent-that-is-in-not-ready-status"></a>Résolution des problèmes de l’agent de machine virtuelle à l’état Prêt

### <a name="step-1-check-whether-the-azure-linux-guest-agent-service-is-running"></a>Étape 1 : Vérifier que le service de l’agent invité Linux Azure s’exécute

Selon la distribution, le nom du service peut être walinuxagent ou waagent :

```
root@nam-u18:/home/nam# service walinuxagent status
● walinuxagent.service - Azure Linux Agent
   Loaded: loaded (/lib/systemd/system/walinuxagent.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2020-10-08 17:10:29 UTC; 3min 9s ago
 Main PID: 1036 (python3)
    Tasks: 4 (limit: 4915)
   CGroup: /system.slice/walinuxagent.service
           ├─1036 /usr/bin/python3 -u /usr/sbin/waagent -daemon
           └─1156 python3 -u bin/WALinuxAgent-2.2.51-py2.7.egg -run-exthandlers

Oct 08 17:10:33 nam-u18 python3[1036]: 2020-10-08T17:10:33.129375Z INFO ExtHandler ExtHandler Started tracking cgroup: Microsoft.OSTCExtensions.VMAccessForLinux-1.5.10, path: /sys/fs/cgroup/memory/sys
Oct 08 17:10:35 nam-u18 python3[1036]: 2020-10-08T17:10:35.189020Z INFO ExtHandler [Microsoft.CPlat.Core.RunCommandLinux-1.0.1] Target handler state: enabled [incarnation 2]
Oct 08 17:10:35 nam-u18 python3[1036]: 2020-10-08T17:10:35.197932Z INFO ExtHandler [Microsoft.CPlat.Core.RunCommandLinux-1.0.1] [Enable] current handler state is: enabled
Oct 08 17:10:35 nam-u18 python3[1036]: 2020-10-08T17:10:35.212316Z INFO ExtHandler [Microsoft.CPlat.Core.RunCommandLinux-1.0.1] Update settings file: 0.settings
Oct 08 17:10:35 nam-u18 python3[1036]: 2020-10-08T17:10:35.224062Z INFO ExtHandler [Microsoft.CPlat.Core.RunCommandLinux-1.0.1] Enable extension [bin/run-command-shim enable]
Oct 08 17:10:35 nam-u18 python3[1036]: 2020-10-08T17:10:35.236993Z INFO ExtHandler ExtHandler Started extension in unit 'Microsoft.CPlat.Core.RunCommandLinux_1.0.1_db014406-294a-49ed-b112-c7912a86ae9e
Oct 08 17:10:35 nam-u18 python3[1036]: 2020-10-08T17:10:35.263572Z INFO ExtHandler ExtHandler Started tracking cgroup: Microsoft.CPlat.Core.RunCommandLinux-1.0.1, path: /sys/fs/cgroup/cpu,cpuacct/syst
Oct 08 17:10:35 nam-u18 python3[1036]: 2020-10-08T17:10:35.280691Z INFO ExtHandler ExtHandler Started tracking cgroup: Microsoft.CPlat.Core.RunCommandLinux-1.0.1, path: /sys/fs/cgroup/memory/system.sl
Oct 08 17:10:37 nam-u18 python3[1036]: 2020-10-08T17:10:37.349090Z INFO ExtHandler ExtHandler ProcessGoalState completed [incarnation 2; 4496 ms]
Oct 08 17:10:37 nam-u18 python3[1036]: 2020-10-08T17:10:37.365590Z INFO ExtHandler ExtHandler [HEARTBEAT] Agent WALinuxAgent-2.2.51 is running as the goal state agent [DEBUG HeartbeatCounter: 1;Heartb
root@nam-u18:/home/nam#
```


Si vous pouvez voir les services et qu’ils sont en cours d’exécution, redémarrez le service pour voir si le problème est résolu. Si les services sont arrêtés, démarrez-les et patientez quelques minutes. Vérifiez ensuite si l’**État de l’agent** est **Prêt**. Pour une résolution plus avancée de ces problèmes, contactez le [Support Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

### <a name="step-2-check-whether-auto-update-is-enabled"></a>Étape 2 : Vérifier que la mise à jour automatique est activée

Vérifiez ce paramètre dans /etc/waagent.conf :

```
AutoUpdate.Enabled=y
```

Pour plus d’informations sur la mise à jour de l’agent Linux Azure, consultez https://docs.microsoft.com/azure/virtual-machines/extensions/update-linux-agent 
    

### <a name="step-3-check-whether-the-vm-can-connect-to-the-fabric-controller"></a>Étape 3 : vérifier si la machine virtuelle peut se connecter au contrôleur de structure

À l’aide d’un outil tel que curl, effectuez un test pour déterminer si la machine virtuelle peut accéder à l’adresse IP 168.63.129.16 sur les ports 80, 32526 et 443. Si la machine virtuelle ne se connecte pas comme prévu, vérifiez si la communication sortante sur les ports 80, 443 et 32526 est ouverte dans votre pare-feu local sur la machine virtuelle. Si cette adresse IP est bloquée, l’agent de machine virtuelle peut présenter un comportement inattendu dans divers scénarios.

## <a name="advanced-troubleshooting"></a>Dépannage avancé

Les événements pour le dépannage de l’agent invité Linux Azure sont enregistrés dans les fichiers journaux suivants :

- /var/log/waagent.log


  
### <a name="unable-to-connect-to-wireserver-ip-host-ip"></a>Impossible de se connecter à l’adresse IP WireServer (adresse IP de l’hôte) 

Vous pouvez remarquer les entrées d’erreur suivantes dans /var/log/waagent.log :

```
2020-10-02T18:11:13.148998Z WARNING ExtHandler ExtHandler An error occurred while retrieving the goal state:
```

**Analyse**

La machine virtuelle ne peut pas joindre l’adresse IP WireServer sur le serveur hôte.

**Solution**

1. Le serveur hôte WireServer n’étant pas accessible, connectez-vous à la machine virtuelle au moyen du protocole SSH, puis essayez d’accéder à l’URL suivante à partir de curl : http://168.63.129.16/?comp=versions 
1. Vérifiez si un pare-feu, un proxy ou une autre source pourraient occasionner des problèmes susceptibles de bloquer l’accès à l’adresse IP 168.63.129.16.
1. Vérifiez si Linux IPTables ou un pare-feu tiers bloque l’accès aux ports 80, 443 et 32526. Pour plus d’informations sur la raison pour laquelle cette adresse ne doit pas être bloquée, consultez [Qu’est-ce que l’adresse IP 168.63.129.16](../../virtual-network/what-is-ip-address-168-63-129-16.md).


## <a name="next-steps"></a>Étapes suivantes

Pour résoudre le problème de non-fonctionnement de l’agent invité Windows Azure, [contactez le Support Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).