---
title: Détecter des problèmes d’extension de machine virtuelle Azure dans la récupération d’urgence de machines virtuelles VMware avec Azure Site Recovery
description: Détecter des problèmes d’extension de machine virtuelle Azure dans la récupération d’urgence de machines virtuelles VMware avec Azure Site Recovery
author: sideeksh
manager: rochakm
ms.topic: troubleshooting
ms.date: 11/27/2018
ms.openlocfilehash: c1915d108bf9465d3e5b8d6a55053b583ee4f580
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009702"
---
# <a name="troubleshoot-azure-vm-extension-issues"></a>Résoudre les problèmes d’extension de machine virtuelle Azure

Cet article indique les étapes à suivre pour résoudre les erreurs d’Azure Site Recovery liées à l’agent et à l’extension de machine virtuelle.

## <a name="low-system-resources"></a>Ressources système faibles

Ce problème se produit lorsque la mémoire système est insuffisante et qu’elle n’est pas en mesure d’allouer de la mémoire pour l’installation du service Mobilité. Veillez à libérer suffisamment de mémoire afin que l’installation s’effectue correctement.

## <a name="azure-site-recovery-extension-time-out"></a>Expiration de l’extension d’Azure Site Recovery  

Message d’erreur : « L’exécution de la tâche a expiré pendant le suivi du démarrage de l’opération d’extension. »<br>
Code d’erreur : « 151076 »

 Azure Site Recovery a installé une extension sur la machine virtuelle dans le cadre d’un travail d’activation de la protection. Il est possible que l’une des conditions suivantes empêche le déclenchement de la protection et entraîne l’échec du travail. Suivez les étapes de dépannage ci-dessous, puis réessayez l’opération :

- [L’agent est installé dans la machine virtuelle, mais ne répond pas (machines virtuelles Windows)](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms)
- [L’agent installé dans la machine virtuelle est obsolète (machines virtuelles Linux)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)
- [Impossible de mettre à jour ou de charger l’extension Site Recovery](#the-site-recovery-extension-fails-to-update-or-load)

Message d’erreur : « L’opération d’extension Site Recovery précédente prend plus de temps que prévu. »<br>
Code d’erreur : « 150066 »

- [L’agent est installé dans la machine virtuelle, mais ne répond pas (machines virtuelles Windows)](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms)
- [L’agent installé dans la machine virtuelle est obsolète (machines virtuelles Linux)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)
- [L’état de l’extension Site Recovery est incorrect](#the-site-recovery-extension-fails-to-update-or-load)

## <a name="protection-fails-because-the-vm-agent-is-unresponsive"></a>La protection échoue, car l’agent de machine virtuelle ne répond pas.

Message d’erreur : « L’exécution de la tâche a expiré pendant le suivi du démarrage de l’opération d’extension. »<br>
Code d’erreur : « 151099 »

Cette erreur peut se produire si l’agent invité Azure dans la machine virtuelle n’est pas à l’état prêt.

Vous pouvez vérifier l’état de l’agent invité Azure dans le [portail Azure](https://portal.azure.com/). Accédez à la machine virtuelle que vous essayez de protéger, puis vérifiez l’état dans **Machine virtuelle** > **Paramètres** > **Propriétés** > **État de l’agent**. La plupart du temps, l’état de l’agent est prêt après le redémarrage de la machine virtuelle. Toutefois, si vous ne pouvez pas la redémarrer ou si vous rencontrez toujours le problème, effectuez les étapes de dépannage suivantes :

- [L’agent est installé dans la machine virtuelle, mais ne répond pas (machines virtuelles Windows)](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms)
- [L’agent installé dans la machine virtuelle est obsolète (machines virtuelles Linux)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)


Message d’erreur : « L’exécution de la tâche a expiré pendant le suivi du démarrage de l’opération d’extension. »<br>
Code d’erreur : « 151095 »

Cette erreur se produit quand la version de l’agent sur la machine Linux est obsolète. Effectuez l’étape de dépannage suivante :

- [L’agent installé dans la machine virtuelle est obsolète (machines virtuelles Linux)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)  

## <a name="causes-and-solutions"></a>Causes et solutions

### <a name="the-agent-is-installed-in-the-vm-but-its-unresponsive-for-windows-vms"></a><a name="the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms"></a>L’agent est installé dans la machine virtuelle, mais ne répond pas (machines virtuelles Windows)

#### <a name="solution"></a>Solution
Il se peut que l’agent de machine virtuelle soit endommagé ou que le service ait été arrêté. Réinstallez l’agent de machine virtuelle pour obtenir la dernière version. Cela permet également de redémarrer la communication avec le service.

1. Regardez si le service d’agent invité Microsoft Azure s’exécute dans les services de machine virtuelle (services.msc). Redémarrez le service d’agent invité Microsoft Azure.    
1. Si le service d’agent invité Microsoft Azure n’est pas visible dans les services, ouvrez le Panneau de configuration. Accédez à **Programmes et fonctionnalités** pour voir si le service d’agent invité Windows est installé.
1. Si le service d’agent invité Wondows Azure figure sous **Programmes et fonctionnalités**, désinstallez-le.
1. Téléchargez et installez la [dernière version du MSI de l’agent](https://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409). Vous devez disposer de droits d’administrateur pour effectuer l’installation.
1. Vérifiez que le service d’agent invité Microsoft Azure apparaît dans les services.
1. Redémarrez le travail de protection.

Vérifiez également que [Microsoft .NET 4.5 est installé](/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed) dans la machine virtuelle. Vous avez besoin de .NET 4.5 pour permettre à l’agent de machine virtuelle de communiquer avec le service.

### <a name="the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms"></a>L’agent installé dans la machine virtuelle est obsolète (pour les machines virtuelles Linux)

#### <a name="solution"></a>Solution
La plupart des échecs des machines virtuelles Linux liés aux agents ou aux extensions sont causés par des problèmes qui affectent un agent obsolète de machine virtuelle. Pour résoudre ce problème, suivez ces recommandations générales :

1. Suivez les instructions fournies pour la [mise à jour d’un agent de machine virtuelle Linux](../virtual-machines/extensions/update-linux-agent.md).

   > [!NOTE]
   > Nous *recommandons vivement* la mise à jour de l’agent uniquement par le biais de référentiel de distribution. Nous ne recommandons pas de télécharger le code de l’agent à partir de GitHub directement et d’effectuer la mise à jour. Si la dernière version de l’agent n’est pas disponible pour la distribution, contactez le support de distribution pour savoir comment l’installer. Pour rechercher l’agent le plus récent, accédez à la page relative à l’[agent Windows Azure Linux](https://github.com/Azure/WALinuxAgent/releases) du référentiel GitHub.

1. Vérifiez que l’agent Azure s’exécute sur la machine virtuelle à l’aide de la commande suivante : `ps -e`.

   Si le processus ne s’exécute pas, redémarrez-le à l’aide des commandes suivantes :

   - Pour Ubuntu : `service walinuxagent start`
   - Pour les autres distributions : `service waagent start`

1. [Configurez l’agent de redémarrage automatique](https://github.com/Azure/WALinuxAgent/wiki/Known-Issues#mitigate_agent_crash).
1. Activez la protection de la machine virtuelle.

### <a name="the-site-recovery-extension-fails-to-update-or-load"></a>Impossible de mettre à jour ou de charger l’extension Site Recovery

L’état d’extension est « Empty » (Vide), « NotReady » (Non prêt) ou « Transitioning » (Transition en cours).

#### <a name="solution"></a>Solution

Désinstallez l’extension et redémarrez l’opération.

Pour désinstaller l’extension :

1. Sur le [portail Azure](https://portal.azure.com/), accédez à la machine virtuelle qui rencontre des échecs de sauvegarde.
1. Sélectionnez **Paramètres**.
1. Sélectionnez **Extensions**.
1. Sélectionnez **Extension Site Recovery**.
1. Sélectionner **Désinstaller**.

Pour les machines virtuelles Linux, si l’extension VMSnapshot ne figure pas dans le portail Azure, [mettez à jour l’agent Linux Azure](../virtual-machines/extensions/update-linux-agent.md), puis exécutez la protection.

Cette procédure réinstalle l’extension au cours de la protection.
