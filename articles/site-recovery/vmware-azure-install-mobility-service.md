---
title: Préparer les machines sources à une installation Push de Mobility Service pour la reprise d’activité de machines virtuelles VMware et serveurs physiques sur Azure | Microsoft Docs
description: Découvrez comment préparer votre serveur à une installation Push de l'agent de mobilité pour la reprise d’activité de machines virtuelles VMware et serveurs physiques sur Azure à l’aide du service Azure Site Recovery.
author: Rajeswari-Mamilla
ms.service: site-recovery
ms.topic: conceptual
ms.date: 03/25/2019
ms.author: ramamill
ms.openlocfilehash: 78fddb5b4512883f8e78d6ed53f6e3dbbeba0e4f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90524995"
---
# <a name="prepare-source-machine-for-push-installation-of-mobility-agent"></a>Préparer la machine source pour l’installation Push de l’agent de mobilité

Lorsque vous configurez la récupération d’urgence sur des serveurs physiques et des machines virtuelles VMware avec [Azure Site Recovery](site-recovery-overview.md), vous installez le [service Mobilité de Site Recovery](vmware-physical-mobility-service-overview.md) sur l’ensemble des serveurs physiques et des machines virtuelles VMware locaux.  Le service Mobilité enregistre les écritures de données sur la machine et les transmet au serveur de traitement Site Recovery.

## <a name="install-on-windows-machine"></a>Installer sur un ordinateur Windows

Sur chaque ordinateur Windows que vous souhaitez protéger, procédez ainsi :

1. Vérifiez qu’une connectivité réseau est établie entre l’ordinateur et le serveur de traitement. Si vous n’avez pas configuré de serveur de traitement distinct, il s’exécute par défaut sur le serveur de configuration.
1. Créez un compte pouvant être utilisé par le serveur de traitement pour accéder à l’ordinateur. Le compte doit disposer des droits d’administrateur (local ou de domaine). Utilisez ce compte uniquement pour l’installation Push et pour les mises à jour de l’agent.
2. Si vous n’utilisez pas de compte de domaine, désactivez le contrôle d’accès utilisateur distant sur l’ordinateur local de la façon suivante :
    - Sous la clé de Registre HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System, ajoutez un nouveau DWORD : **LocalAccountTokenFilterPolicy**. Définissez la valeur sur **1**.
    -  Pour exécuter cette action à l’invite de commandes, exécutez la commande suivante :
    
       ```
       REG ADD HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f
       ```

3. Sur le Pare-feu Windows de l’ordinateur à protéger, sélectionnez **Autoriser une application ou une fonctionnalité à franchir le pare-feu**. Activez **Partage de fichiers et d’imprimantes** et **Windows Management Instrumentation (WMI)** . Pour les ordinateurs qui appartiennent à un domaine, vous pouvez configurer les paramètres de pare-feu avec un objet de stratégie de groupe.

   ![Paramètres du pare-feu](./media/vmware-azure-install-mobility-service/mobility1.png)

4. Ajoutez le compte créé dans l’outil CSPSConfigtool. Pour cela, connectez-vous à votre serveur de configuration.
5. Ouvrez le fichier **cspsconfigtool.exe**. Il est disponible sous forme de raccourci sur le bureau et dans le dossier %ProgramData%\home\svsystems\bin.
6. Dans l’onglet **Gérer les comptes**, sélectionnez **Ajouter un compte**.
7. Ajoutez le compte que vous venez de créer.
8. Entrez les informations d’identification utilisées lors de l’activation de la réplication d’un ordinateur.

## <a name="install-on-linux-machine"></a>Installer sur un ordinateur Linux

Sur chaque ordinateur Linux que vous souhaitez protéger, procédez ainsi :

1. Vérifiez qu’une connectivité réseau est établie entre l’ordinateur Linux et le serveur de traitement.
2. Créez un compte pouvant être utilisé par le serveur de traitement pour accéder à l’ordinateur. Le compte est un utilisateur **racine** sur le serveur Linux source. Utilisez ce compte uniquement pour l’installation Push et pour les mises à jour.
3. Vérifiez que le fichier /etc/hosts sur le serveur Linux source contient des entrées qui mappent le nom d’hôte local sur les adresses IP associées à toutes les cartes réseau.
4. Installez les packages openssh, openssh-server et openssl les plus récents sur l’ordinateur à répliquer.
5. Vérifiez que le protocole Secure Shell (SSH) est activé et s’exécute sur le port 22.
4. Activez l’authentification par mot de passe et le sous-système SFTP dans le fichier sshd_config. Pour cela, connectez-vous en tant que **racine**.
5. Dans le fichier **/etc/ssh/sshd_config**, recherchez la ligne commençant par **PasswordAuthentication**.
6. Supprimez les marques de commentaire de la ligne et remplacez la valeur actuelle par **yes**.
7. Recherchez la ligne commençant par **Subsystem**, puis supprimez les marques de commentaire de la ligne.

      ![Linux](./media/vmware-azure-install-mobility-service/mobility2.png)

8. Redémarrez le service **sshd**.
9. Ajoutez le compte créé dans l’outil CSPSConfigtool. Pour cela, connectez-vous à votre serveur de configuration.
10. Ouvrez le fichier **cspsconfigtool.exe**. Il est disponible sous forme de raccourci sur le Bureau et dans le dossier %ProgramData%\home\svsystems\bin.
11. Dans l’onglet **Gérer les comptes**, sélectionnez **Ajouter un compte**.
12. Ajoutez le compte que vous venez de créer.
13. Entrez les informations d’identification utilisées lors de l’activation de la réplication d’un ordinateur.
1. Étape supplémentaire pour la mise à jour ou la protection des machines SUSE Linux Enterprise Server 11 SP3 ou RHEL 5 ou CentOS 5 ou Debian 7. [Vérifiez que la dernière version est disponible dans le serveur de configuration](vmware-physical-mobility-service-overview.md#download-latest-mobility-agent-installer-for-suse-11-sp3-rhel-5-debian-7-server).

## <a name="anti-virus-on-replicated-machines"></a>Protection antivirus sur les machines répliquées

Si les machines que vous souhaitez répliquer ont un logiciel antivirus actif en cours d'exécution, assurez-vous d'exclure le dossier d'installation du service Mobilité des opérations antivirus (*C:\ProgramData\ASR\agent*). Cela garantit le bon fonctionnement de la réplication.

## <a name="next-steps"></a>Étapes suivantes

Une fois le service Mobilité installé, sélectionnez **+ Répliquer** sur le Portail Azure pour protéger ces machines virtuelles. En savoir plus sur l’activation de la réplication pour les [machines virtuelles VMware](vmware-azure-enable-replication.md) et les [serveurs physiques VMware](physical-azure-disaster-recovery.md#enable-replication).


