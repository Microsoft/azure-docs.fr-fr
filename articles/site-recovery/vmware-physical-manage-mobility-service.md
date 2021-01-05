---
title: Gérer l’agent du service Mobilité pour les serveurs physiques et VMware avec Azure Site Recovery
description: Gérez l'agent du service Mobilité pour la récupération d'urgence des serveurs physiques et des machines virtuelles VMware sur Azure à l’aide du service Azure Site Recovery.
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 03/25/2019
ms.author: ramamill
ms.openlocfilehash: d921bddf90c415cb244e2cc9ad98354392a537ee
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90530146"
---
# <a name="manage-the-mobility-agent"></a>Gérer l’agent du service Mobilité 

Quand vous utilisez Azure Site Recovery pour la récupération d'urgence de machines virtuelles VMware et de serveurs physiques sur Azure, vous devez configurer l’agent de mobilité sur votre serveur. L’agent de mobilité coordonne la communication entre votre ordinateur protégé, le serveur de configuration/serveur de traitement scale-out et gère la réplication des données. Cet article récapitule les tâches courantes de gestion de l’agent de mobilité après son déploiement.

>[!TIP]
>Pour télécharger le programme d’installation d’une distribution de Linux ou d’un système d’exploitation spécifique, reportez-vous au guide [ici](vmware-physical-mobility-service-overview.md#locate-installer-files). Pour effectuer une mise à jour automatique à partir du portail, vous n’avez pas besoin de télécharger le programme d’installation. [ASR récupère automatiquement le programme d’installation à partir du serveur de configuration, et met à jour l’agent](#update-mobility-service-from-azure-portal).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="update-mobility-service-from-azure-portal"></a>Mettre à jour le service de mobilité à partir du portail Azure

1. Avant de commencer, veillez à ce que le serveur de configuration, les serveurs de processus de scale-out et les serveurs cibles maîtres qui font partie de votre déploiement soient mis à jour avant de procéder à la mise à jour du service Mobilité sur les machines protégées.
    1. À partir de la version 9.36, pour SUSE Linux Enterprise Server 11 SP3, RHEL 5, CentOS 5, Debian 7, vérifiez que le programme d'installation le plus récent est [disponible sur le serveur de configuration et sur le serveur de processus de scale-out](vmware-physical-mobility-service-overview.md#download-latest-mobility-agent-installer-for-suse-11-sp3-rhel-5-debian-7-server).
1. Sur le portail, ouvrez le coffre > **Éléments répliqués**.
1. Si le serveur de configuration correspond à la dernière version, une notification doit s’afficher et indiquer « Une nouvelle mise à jour de l’agent de réplication Site Recovery est disponible. Cliquez pour installer. »

     ![Fenêtre Éléments répliqués](./media/vmware-azure-install-mobility-service/replicated-item-notif.png)

4. Cliquez sur la notification, puis, dans **Mise à jour de l’agent**, sélectionnez les machines sur lesquelles vous souhaitez mettre à niveau le service Mobilité. Cliquez ensuite sur **OK**.

     ![Éléments répliqués - Liste des machines virtuelles](./media/vmware-azure-install-mobility-service/update-okpng.png)

5. La tâche Mettre à jour le service Mobilité est alors lancée pour chacune des machines sélectionnées. L’agent de mobilité est mis à jour avec la version du serveur de configuration. Par exemple, si le serveur de configuration est sur la version 9.33, l’agent de mobilité sur une machine virtuelle protégée est également mis à jour vers la version 9.33.

## <a name="update-mobility-service-through-powershell-script-on-windows-server"></a>Mettre à jour le service de mobilité via le script powershell sur le serveur Windows

Avant de commencer, veillez à ce que le serveur de configuration, les serveurs de processus de scale-out et les serveurs cibles maîtres qui font partie de votre déploiement soient mis à jour avant de procéder à la mise à jour du service Mobilité sur les machines protégées.

Utiliser le script suivant pour mettre à niveau le service de mobilité sur un serveur via la cmdlet power shell

```azurepowershell
Update-AzRecoveryServicesAsrMobilityService -ReplicationProtectedItem $rpi -Account $fabric.fabricSpecificDetails.RunAsAccounts[0]
```

## <a name="update-mobility-service-manually-on-each-protected-server"></a>Mettre à jour le service de mobilité manuellement sur chaque serveur protégé

1. Avant de commencer, veillez à ce que le serveur de configuration, les serveurs de processus de scale-out et les serveurs cibles maîtres qui font partie de votre déploiement soient mis à jour avant de procéder à la mise à jour du service Mobilité sur les machines protégées.

2. [Recherchez le programme d’installation de l’agent](vmware-physical-mobility-service-overview.md#locate-installer-files) basé sur le système d’exploitation du serveur.

>[!IMPORTANT]
> Si vous répliquez une machine virtuelle IaaS Azure depuis une région Azure vers une autre, n’utilisez pas cette méthode. Pour plus d’informations sur toutes les options disponibles, reportez-vous aux [conseils que nous fournissons](azure-to-azure-autoupdate.md).

3. Copiez le fichier d’installation sur la machine protégée, puis exécutez-le pour mettre à jour l’agent de mobilité.

## <a name="update-account-used-for-push-installation-of-mobility-service"></a>Mettre à jour le compte utilisé pour l’installation Push du service Mobilité

Au moment du déploiement de Site Recovery, pour activer l’installation Push du service Mobilité, vous avez spécifié un compte que le serveur de processus Site Recovery utilise afin d'accéder aux machines et installer le service lorsque de la réplication est activée pour la machine. Pour mettre à jour les informations d’identification de ce compte, suivez [ces instructions](vmware-azure-manage-configuration-server.md#modify-credentials-for-mobility-service-installation).

## <a name="uninstall-mobility-service"></a>Désinstaller le service Mobilité

### <a name="on-a-windows-machine"></a>Sur une machine Windows

Désinstallez le service à partir de l’interface utilisateur ou d’une invite de commandes.

- **À partir de l’interface utilisateur** : dans le Panneau de configuration de la machine, sélectionnez **Programmes**. Sélectionnez **Service Mobilité/Serveur cible maître Microsoft Azure Site Recovery** > **Désinstaller**.
- **À partir d’une invite de commandes** : sur la machine, ouvrez une fenêtre d’invite de commandes en tant qu’administrateur. Exécutez la commande suivante : 
    ```
    MsiExec.exe /qn /x {275197FC-14FD-4560-A5EB-38217F80CBD1} /L+*V "C:\ProgramData\ASRSetupLogs\UnifiedAgentMSIUninstall.log"
    ```

### <a name="on-a-linux-machine"></a>Sur une machine Linux
1. Sur la machine Linux, connectez-vous en tant qu’utilisateur **racine**.
2. Dans un terminal, accédez à /usr/local/ASR.
3. Exécutez la commande suivante :
    ```
    uninstall.sh -Y
   ```
   
## <a name="install-site-recovery-vss-provider-on-source-machine"></a>Installer le fournisseur VSS Site Recovery sur la machine source

Le fournisseur VSS Site Recovery est requis sur la machine source pour générer des points de cohérence d’application. Si l’installation du fournisseur n’a pas réussi par le biais de l’installation push, suivez les instructions ci-dessous pour l’installer manuellement.

1. Ouvrez la fenêtre admin cmd.
2. Accédez à l’emplacement d’installation du service Mobilité. (Par exemple C:\Program Files (x86)\Microsoft Azure Site Recovery\agent)
3. Exécutez le script InMageVSSProvider_Uninstall.cmd. Le service sera alors désinstallé s’il existe déjà.
4. Exécutez le script InMageVSSProvider_Install.cmd pour installer le fournisseur VSS manuellement.

## <a name="next-steps"></a>Étapes suivantes

- [Configurer la reprise d'activité pour les machines virtuelles VMware](vmware-azure-tutorial.md)
- [Configurer la reprise d'activité pour les serveurs physiques](physical-azure-disaster-recovery.md)
