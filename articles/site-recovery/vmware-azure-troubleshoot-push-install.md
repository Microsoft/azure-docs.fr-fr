---
title: Résoudre les problèmes d’installation Push du service Mobility avec Azure Site Recovery
description: Résolvez les erreurs d’installation des services Mobilité lors de l’activation de la réplication dans le cadre d’une récupération d’urgence avec Azure Site Recovery.
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.author: ramamill
ms.date: 04/03/2020
ms.openlocfilehash: 8ee6449f357a578b30809bb03723ac1556e4f459
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88816166"
---
# <a name="troubleshoot-mobility-service-push-installation"></a>Résoudre les problèmes d’installation Push du service Mobility

L’installation du service Mobility est une étape clé de l’activation de la réplication. La réussite de cette étape dépend de la satisfaction des conditions préalables et de l’utilisation des configurations prises en charge. Les erreurs les plus courantes que vous rencontrez au cours de l’installation du service Mobility sont dues aux :

* [Erreurs d’informations d’identification/de privilèges](#credentials-check-errorid-95107--95108)
* [Échecs de connexion](#login-failures-errorid-95519-95520-95521-95522)
* [Erreurs de connectivité](#connectivity-failure-errorid-95117--97118)
* [Erreurs de partage de fichiers et d’imprimantes](#file-and-printer-sharing-services-check-errorid-95105--95106)
* [Échecs Windows Management Instrumentation (WMI)](#windows-management-instrumentation-wmi-configuration-check-error-code-95103)
* [Systèmes d’exploitation non pris en charge](#unsupported-operating-systems)
* [Configurations de démarrage non prises en charge](#unsupported-boot-disk-configurations-errorid-95309-95310-95311)
* [Échecs d’installation du service de cliché instantané des volumes (VSS)](#vss-installation-failures)
* [Nom de l’appareil dans la configuration GRUB au lieu de l’UUID d’appareil](#enable-protection-failed-as-device-name-mentioned-in-the-grub-configuration-instead-of-uuid-errorid-95320)
* [Volume LVM (Logical Volume Manager)](#lvm-support-from-920-version)
* [Avertissements de redémarrage](#install-mobility-service-completed-with-warning-to-reboot-errorid-95265--95266)

Lorsque vous activez la réplication, Azure Site Recovery tente d’installer l’agent du service Mobility sur votre machine virtuelle. Dans le cadre de ce processus, le serveur de configuration essaie de se connecter à la machine virtuelle et de copier l’agent. Pour réussir l’installation, suivez les instructions de résolution des problèmes étape par étape.

## <a name="credentials-check-errorid-95107--95108"></a>Vérification des informations d’identification (ErrorID : 95107 & 95108)

Vérifiez si le compte d’utilisateur choisi lors de l’activation de la réplication est valide et correct. Azure Site Recovery requiert le compte **racine** ou le compte d’utilisateur avec des **privilèges Administrateur** pour effectuer l’installation Push. Autrement, l’installation Push sera bloquée sur la machine source.

Pour Windows (**erreur 95107**), vérifiez si le compte d’utilisateur a un accès administratif sur l’ordinateur source, que ce soit avec un comte local ou de domaine. Si vous n’utilisez pas un compte de domaine, vous devez désactiver le contrôle d’accès des utilisateurs distants sur l’ordinateur local.

* Pour ajouter manuellement une clé de Registre qui désactive le contrôle d’accès des utilisateurs distants :

  * `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System`
  * Ajoutez un nouveau `DWORD` : `LocalAccountTokenFilterPolicy`
  * Définissez la valeur sur `1`

* Pour ajouter la clé de Registre, à partir d’une invite de commandes, exécutez la commande suivante :

  `REG ADD HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1`

Pour Linux (**erreur 95108**), vous devez choisir le compte **racine** pour réussir l’installation de l’agent du service Mobility. En outre, les services du protocole FTP SSH (SFTP) doivent être en cours d’exécution. Pour activer l’authentification par mot de passe et le sous-système SFTP dans le fichier _sshd_config_ :

1. Connectez-vous en tant qu’utilisateur **racine**.
1. Accédez au fichier _/etc/ssh/sshd_config_ et recherchez la ligne commençant par `PasswordAuthentication`.
1. Supprimez les marques de commentaire de la ligne et remplacez la valeur actuelle par `yes`.
1. Recherchez la ligne commençant par `Subsystem`, puis supprimez les marques de commentaire de la ligne.
1. Redémarrez le service `sshd` .

Si vous souhaitez modifier les informations d’identification du compte d’utilisateur choisi, suivez [ces instructions](vmware-azure-manage-configuration-server.md#modify-credentials-for-mobility-service-installation).

## <a name="insufficient-privileges-failure-errorid-95517"></a>Échec en raison de privilèges insuffisants (ErrorID : 95517)

Lorsque l’utilisateur choisi pour installer l’agent Mobility n’a pas les privilèges Administrateur, le serveur de configuration/serveur de traitement avec Scale-out ne sera pas autorisé à copier le logiciel agent Mobility sur la machine source. Cette erreur est due à un échec de type Accès refusé. Assurez-vous que le compte d’utilisateur dispose des privilèges d’administrateur.

Si vous souhaitez modifier les informations d’identification du compte d’utilisateur choisi, suivez [ces instructions](vmware-azure-manage-configuration-server.md#modify-credentials-for-mobility-service-installation).

## <a name="insufficient-privileges-failure-errorid-95518"></a>Échec en raison de privilèges insuffisants (ErrorID : 95518)

Si l’établissement d’une relation d’approbation de domaine entre le domaine principal et la station de travail échoue alors que vous essayez de vous connecter à la machine source, l’installation de l’agent Mobility échoue avec l’erreur ID 95518. Assurez-vous que le compte d’utilisateur choisi pour installer l’agent Mobility possède les privilèges Administrateur pour se connecter via le domaine principal de la machine source.

Si vous souhaitez modifier les informations d’identification du compte d’utilisateur choisi, suivez [ces instructions](vmware-azure-manage-configuration-server.md#modify-credentials-for-mobility-service-installation).

## <a name="login-failures-errorid-95519-95520-95521-95522"></a>Échecs de connexion (ErrorID : 95519, 95520, 95521, 95522)

Cette section décrit les messages d’erreur d’identification et de connexion.

### <a name="credentials-of-the-user-account-have-been-disabled-errorid-95519"></a>Les informations d’identification du compte d’utilisateur ont été désactivées (ID d’erreur : 95519)

Le compte d’utilisateur choisi lors de l’activation de la réplication a été désactivé. Pour activer le compte d’utilisateur, reportez-vous à [cet article](/powershell/module/Microsoft.PowerShell.LocalAccounts/Enable-LocalUser) ou exécutez la commande suivante en remplaçant le texte _username_ par le nom d’utilisateur réel.

`net user 'username' /active:yes`

### <a name="credentials-locked-out-because-of-multiple-failed-login-attempts-errorid-95520"></a>Les informations d’identification ont été verrouillées en raison de plusieurs tentatives de connexion infructueuses (ErrorID : 95520)

Plusieurs échecs de nouvelles tentatives d’accès à la machine ont pour effet de verrouiller le compte d’utilisateur. L’échec peut être dû à ce qui suit :

* Les informations d’identification fournies lors du paramétrage de la configuration sont incorrectes.
* Le compte d’utilisateur choisi lors de l’activation de la réplication est incorrect.

Modifiez les informations d’identification choisies en suivant [ces instructions](vmware-azure-manage-configuration-server.md#modify-credentials-for-mobility-service-installation) et recommencez l’opération.

### <a name="logon-servers-arent-available-on-the-source-machine-errorid-95521"></a>Les serveurs d’ouverture de session ne sont pas disponibles sur la machine source (ErrorID : 95521)

Cette erreur se produit lorsque les serveurs d’ouverture de session ne sont pas disponibles sur la machine source. Si les serveurs d’ouverture de session ne sont pas disponibles, les demandes de connexion échouent et l’agent Mobility ne peut pas être installé. Pour vous connecter, vérifiez que les serveurs d’ouverture de session sont disponibles sur la machine source et démarrez le service Netlogon. Pour plus d’informations, consultez [Scénarios d’ouverture de session Windows](/windows-server/security/windows-authentication/windows-logon-scenarios).

### <a name="logon-service-isnt-running-on-the-source-machine-errorid-95522"></a>Le service d’ouverture de session ne s’exécute pas sur l’ordinateur source (ID d’erreur : 95522)

Le service de connexion n’est pas en cours d’exécution sur votre machine source et a provoqué l’échec de la demande de connexion. L’agent Mobility ne peut pas être installé. Pour résoudre l’erreur, utilisez l’une des méthodes suivantes pour démarrer le service `Netlogon` sur la machine source :

* Pour démarrer le service `Netlogon` à partir d’une invite de commandes, exécutez la commande `net start Netlogon`.
* Dans le Gestionnaire des tâches, démarrez le service `Netlogon`.

## <a name="connectivity-failure-errorid-95117--97118"></a>Échec de connectivité (ErrorID : 95117 et 97118)

Le serveur de configuration/serveur de traitement avec Scale-out tente de se connecter à la machine virtuelle source pour installer l’agent Mobility. Cette erreur se produit lorsque la machine source n’est pas accessible en raison de problèmes de connectivité réseau.

Pour résoudre l’erreur :

* Vérifiez que vous pouvez effectuer un test ping sur votre machine source à partir du serveur de configuration. Si vous avez choisi le serveur de traitement avec Scale-out lors de l’activation de la réplication, assurez-vous de pouvoir effectuer un test ping sur votre machine source à partir du serveur de traitement.

* À partir de la ligne de commande de l’ordinateur du serveur source, utilisez `Telnet` pour effectuer un test ping sur le serveur de configuration ou sur le serveur de traitement avec Scale-out sur le port HTTPS 135, comme indiqué dans la commande suivante. Cette commande vérifie s’il existe des problèmes de connectivité réseau ou des problèmes bloquant les ports du pare-feu.

  `telnet <CS/ scale-out PS IP address> <135>`

* En outre, pour une machine virtuelle Linux :
  * Vérifiez que les packages OpenSSH, OpenSSH Server et OpenSSL les plus récents sont installés.
  * Vérifiez que le protocole Secure Shell (SSH) est activé et s’exécute sur le port 22.
  * Les services SFTP doivent être en cours d’exécution. Pour activer l’authentification par mot de passe et le sous-système SFTP dans le fichier _sshd_config_ :

    1. Connectez-vous en tant qu’utilisateur **racine**.
    1. Accédez au fichier _/etc/ssh/sshd_config_ et recherchez la ligne commençant par `PasswordAuthentication`.
    1. Supprimez les marques de commentaire de la ligne et remplacez la valeur actuelle par `yes`.
    1. Recherchez la ligne commençant par `Subsystem`, puis supprimez les marques de commentaire de la ligne.
    1. Redémarrez le service `sshd` .

* Une tentative de connexion peut échouer si aucune réponse correcte n’est fournie après une période déterminée, ou la connexion établie échoue car l’hôte connecté n’a pas répondu.
* Il peut s’agir d’un problème lié au domaine, au réseau ou à la connectivité. Cela peut également être dû à un problème de résolution de nom DNS ou d’insuffisante de ports TCP. Vérifiez s’il existe des problèmes connus dans votre domaine.

## <a name="connectivity-failure-errorid-95523"></a>Échec de connectivité (ErrorID : 95523)

Cette erreur se produit lorsque le réseau dans lequel réside la machine source est introuvable, peut avoir été supprimé ou n’est plus disponible. La seule façon de résoudre l’erreur est de s’assurer que le réseau existe.

## <a name="check-access-for-network-shared-folders-on-source-machine-errorid-9510595523"></a>Vérifiez l’accès aux dossiers partagés du réseau sur la machine source (ErrorID : 95105,95523)

Vérifiez si les dossiers partagés sur la machine source sont accessibles à partir du serveur de processus à distance en utilisant les informations d’identification spécifiées. Pour confirmez l’accès : 

1. Connectez-vous à votre serveur de processus.
2. Ouvrez l’Explorateur de fichiers. Dans la barre d’adresses, tapez `\\<SOURCE-MACHINE-IP>\C$`, puis cliquez sur Entrée.

    ![Ouvrir un dossier dans le serveur de processus](./media/vmware-azure-troubleshoot-push-install/open-folder-process-server.PNG)

3. Dans l’Explorateur de fichiers, vous êtes invité à indiquer des informations d’identification. Entrez le nom d’utilisateur et le mot de passe, puis cliquez sur OK. <br><br/>

    ![Fournir des informations d’identification](./media/vmware-azure-troubleshoot-push-install/provide-credentials.PNG)

    >[!NOTE]
    > Si la machine source est jointe au domaine, indiquez le nom de domaine ainsi que le nom d’utilisateur sous la forme `<domainName>\<username>`. Si la machine source est dans un groupe de travail, fournissez uniquement le nom d’utilisateur.

4. Si la connexion aboutit, les dossiers de machine source sont visibles à distance depuis le serveur de processus.

    ![Dossiers visibles de la machine source](./media/vmware-azure-troubleshoot-push-install/visible-folders-from-source.png)

Si la connexion échoue, vérifiez que tous les prérequis sont bien respectés.

## <a name="file-and-printer-sharing-services-check-errorid-95105--95106"></a>Vérification des services de partage de fichiers et d’imprimantes (ErrorID : 95105 & 95106)

Après une vérification de la connectivité, vérifiez si le service de partage de fichiers et d’imprimantes est activé sur votre machine virtuelle. Ces paramètres sont requis pour copier l’agent Mobility sur la machine source.

Pour **Windows 2008 R2 et les versions antérieures** :

* Pour activer le partage de fichiers et d’impression via le Pare-feu Windows,
  1. Ouvrez **Panneau de configuration** > **Système et sécurité** > **Pare-feu Windows**. Dans le volet gauche, sélectionnez **Paramètres avancés** > **Règles de trafic entrant** dans l’arborescence de la console.
  1. Recherchez les règles de partage de fichiers et d’imprimantes (NB-Session-In) et de partage de fichiers et d’imprimantes (SMB-In).
  1. Pour chaque règle, cliquez avec le bouton droit sur la règle, puis sur **Activer la règle**.

* Pour activer le partage de fichiers avec la stratégie de groupe :
  1. Accédez au menu **Démarrer**, saisissez `gpmc.msc` et lancez la recherche.
  1. Dans le volet de navigation, ouvrez les dossiers suivants : **Stratégie ordinateur local** > **Configuration utilisateur** > **Modèles d’administration** > **Composants Windows** > **Partage réseau**.
  1. Dans le volet d’informations, double-cliquez sur **Prevent users from sharing files within their profile** (Empêcher les utilisateurs de partager des fichiers dans leur profil).

     Pour désactiver le paramètre de stratégie de groupe et activer la capacité d’utilisateur à partager des fichiers, sélectionnez **Désactivé**.

  1. Sélectionnez **OK** pour enregistrer vos modifications.

  Pour en savoir plus, consultez [Activer ou désactiver le partage de fichiers avec la stratégie de groupe](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754359(v=ws.10)).

Afin d’activer le partage de fichiers et d’imprimantes pour les versions ultérieures, suivez les instructions fournies dans [Installer le service Mobility pour la récupération d’urgence de machines virtuelles VMware et de serveurs physiques](vmware-azure-install-mobility-service.md).

## <a name="windows-management-instrumentation-wmi-configuration-check-error-code-95103"></a>Vérification de la configuration de Windows Management Instrumentation (WMI) (code erreur : 95103)

Après avoir vérifié les services de partage de fichiers et d’imprimantes, activez le service WMI pour les profils privés, publics et de domaine via le pare-feu. Ces paramètres sont requis pour terminer l’exécution à distance sur la machine source.

Pour activer WMI :

1. Accédez à **Panneau de configuration** > **Sécurité** et sélectionnez **Pare-feu Windows**.
1. Sélectionnez **Modifier les paramètres**, puis l’onglet **Exceptions**.
1. Dans la fenêtre **Exceptions**, sélectionnez la case à cocher de Windows Management Instrumentation (WMI) pour autoriser le trafic WMI à traverser le pare-feu.

Vous pouvez également autoriser le trafic WMI à traverser le pare-feu à l’aide de la commande suivante :

`netsh advfirewall firewall set rule group="windows management instrumentation (wmi)" new enable=yes`

Vous pouvez trouver d’autres articles concernant la résolution des problèmes WMI dans les articles suivants.

* [Test de base WMI](https://techcommunity.microsoft.com/t5/ask-the-performance-team/bg-p/AskPerf)
* [Résolution des problèmes WMI](/windows/win32/wmisdk/wmi-troubleshooting)
* [Résolution des problèmes liés aux scripts et aux services WMI](/previous-versions/tn-archive/ff406382(v=msdn.10))

## <a name="unsupported-operating-systems"></a>Systèmes d’exploitation non pris en charge

Une autre raison courante de l’échec peut être un système d’exploitation non pris en charge. Utilisez une version de noyau et un système d’exploitation pris en charge pour réussir l’installation du service Mobility. Évitez d’utiliser des correctifs privés.

Pour voir la liste des systèmes d’exploitation et versions du noyau pris en charge par Azure Site Recovery, consultez le [document de la matrice de prise en charge](vmware-physical-azure-support-matrix.md#replicated-machines).

## <a name="unsupported-boot-disk-configurations-errorid-95309-95310-95311"></a>Configurations de disque de démarrage non prises en charge (ID d’erreur : 95309, 95310, 95311)

### <a name="boot-and-system-partitions--volumes-arent-the-same-disk-errorid-95309"></a>Les partitions/volumes de démarrage et système ne correspondent pas au même disque (ErrorID : 95309)

Avant la version 9.20, les volumes/partitions de démarrage et système installés sur des disques différents n’étaient pas pris en charge. À partir de la [version 9.20](https://support.microsoft.com/help/4478871/update-rollup-31-for-azure-site-recovery), cette configuration est prise en charge.

### <a name="the-boot-disk-isnt-available-errorid-95310"></a>Le disque de démarrage n’est pas disponible (ErrorID : 95310)

Une machine virtuelle sans disque de démarrage ne peut pas être protégée. Un disque de démarrage garantit la récupération sans heurts d’une machine virtuelle pendant une opération de basculement. L’absence de disque de démarrage occasionne l’échec du démarrage de la machine après le basculement. Vérifiez que la machine virtuelle contient un disque de démarrage et recommencez l’opération. Par ailleurs, les machines comprenant plusieurs disques de démarrage ne sont pas prises en charge.

### <a name="multiple-boot-disks-present-on-the-source-machine-errorid-95311"></a>Plusieurs disques de démarrage sont présents sur l’ordinateur source (ID d’erreur : 95311)

Une machine virtuelle dotée de plusieurs disques de démarrage ne constitue pas une [configuration prise en charge](vmware-physical-azure-support-matrix.md#linux-file-systemsguest-storage).

## <a name="system-partition-on-multiple-disks-errorid-95313"></a>Partition système sur plusieurs disques (ErrorID : 95313)

Avant la version 9.20, la configuration d’une partition racine ou d’un volume installé sur plusieurs disques n’était pas prise en charge. À partir de la [version 9.20](https://support.microsoft.com/help/4478871/update-rollup-31-for-azure-site-recovery), cette configuration est prise en charge.

## <a name="enable-protection-failed-as-device-name-mentioned-in-the-grub-configuration-instead-of-uuid-errorid-95320"></a>L'activation de la protection a échoué, car le nom de l'appareil est mentionné dans la configuration GRUB au lieu de l'UUID (ID d'erreur : 95320)

### <a name="possible-cause"></a>Cause possible

Les fichiers config Grand Unified Bootloader ( _/boot/grub/menu.lst_, _/boot/grub/grub.cfg_, _/boot/grub2/grub.cfg_ ou _/etc/default/grub_) peuvent contenir la valeur des paramètres **root** et **resume** en tant que noms d’appareils en lieu et place d’un identificateur unique universel (UUID). Site Recovery impose l’approche UUID, car les noms d’appareils peuvent changer au redémarrage de la machine virtuelle. Par exemple, la machine virtuelle peut être mise en ligne avec un nom différent lors du basculement, ce qui entraîne des problèmes.

Par exemple :

- La ligne suivante provient du fichier GRUB _/boot/grub2/grub.cfg_ :

  `linux /boot/vmlinuz-3.12.49-11-default root=/dev/sda2  ${extra_cmdline} resume=/dev/sda1 splash=silent quiet showopts`

- La ligne suivante provient du fichier GRUB _/boot/grub/menu.lst_ :

  `kernel /boot/vmlinuz-3.0.101-63-default root=/dev/sda2 resume=/dev/sda1 splash=silent crashkernel=256M-:128M showopts vga=0x314`

> [!NOTE]
> Les lignes GRUB contiennent des noms d’appareils réels pour les paramètres **root** et **resume** plutôt que l’UUID.

### <a name="how-to-fix"></a>Procédure de résolution

Le nom de chaque appareil doit être remplacé par l'UUID correspondante.

1. Recherchez l’UUID de l’appareil en exécutant la commande `blkid \<device name>`.

   Par exemple :

   ```shell
   blkid /dev/sda1
   /dev/sda1: UUID="6f614b44-433b-431b-9ca1-4dd2f6f74f6b" TYPE="swap"
   blkid /dev/sda2
   /dev/sda2: UUID="62927e85-f7ba-40bc-9993-cc1feeb191e4" TYPE="ext3"
   ```

1. Maintenant, remplacez le nom de l’appareil par son UUID dans un format du type `root=UUID=\<UUID>`. Par exemple, si l’on remplace le nom des appareils par l’UUID pour les paramètres root et resume mentionnés dans les fichiers _/boot/grub2/grub.cfg_, _/boot/grub2/grub.cfg_ ou _/etc/default/grub_, les lignes des fichiers ressemblent à la ligne suivante :

   `kernel /boot/vmlinuz-3.0.101-63-default root=UUID=62927e85-f7ba-40bc-9993-cc1feeb191e4 resume=UUID=6f614b44-433b-431b-9ca1-4dd2f6f74f6b splash=silent crashkernel=256M-:128M showopts vga=0x314`

1. Redémarrez la protection.

## <a name="install-mobility-service-completed-with-warning-to-reboot-errorid-95265--95266"></a>Installation du service Mobility terminée avec avertissement de redémarrage (ErrorID : 95265, 95266)

Le service Mobility de Site Recovery a de nombreux composants, et notamment le pilote de filtre. Le pilote de filtre n’est chargé dans la mémoire système que pendant le redémarrage du système. Les correctifs du pilote de filtre ne peuvent être réalisés que lorsqu’un nouveau pilote de filtre est chargé au moment du redémarrage du système.

> [!IMPORTANT]
> Il s’agit d’un avertissement et la réplication existante fonctionnera toujours après la nouvelle mise à jour de l’agent. Vous pouvez choisir de redémarrer chaque fois que vous souhaitez profiter des avantages du nouveau pilote de filtre ; en l’absence de redémarrage, l’ancien pilote de filtre continue de fonctionner. Par conséquent, après une mise à jour sans redémarrage, à l’exception du pilote de filtre, **les autres améliorations et correctifs du service Mobility offrent tous leurs avantages**. Bien que cela soit recommandé, il n’est pas obligatoire de redémarrer après chaque mise à niveau. Pour savoir quand un redémarrage est obligatoire, consultez la section [Redémarrage après la mise à niveau du service Mobility](service-updates-how-to.md#reboot-after-mobility-service-upgrade) des mises à jour de service dans Azure Site Recovery.

> [!TIP]
>Pour connaître les meilleures pratiques de planification des mises à niveau pendant votre fenêtre de maintenance, consultez la section [Prise en charge des systèmes d’exploitation ou noyaux les plus récents ](service-updates-how-to.md#support-for-latest-operating-systemskernels) des mises à jour de service dans Azure Site Recovery.

## <a name="lvm-support-from-920-version"></a>Prise en charge de LVM à partir de la version 9.20

Avant la version 9.20, Logical Volume Manager (LVM) était pris en charge uniquement pour les disques de données. La partition `/boot` doit se trouver sur une partition de disque et ne doit pas être un volume LVM.

À partir de la [version 9.20](https://support.microsoft.com/help/4478871/update-rollup-31-for-azure-site-recovery), le [disque du système d’exploitation sur LVM](vmware-physical-azure-support-matrix.md#linux-file-systemsguest-storage) est pris en charge.

## <a name="insufficient-space-errorid-95524"></a>Espace insuffisant (ErrorID : 95524)

Lorsque l’agent Mobility est copié sur la machine source, au moins 100 Mo d’espace libre sont nécessaires. Assurez-vous que votre machine source dispose de la quantité d’espace libre requise et recommencez l’opération.

## <a name="low-system-resources"></a>Ressources système faibles

Les ID d’erreur possibles affichés pour ce problème sont 95572 et 95573. Ce problème se produit lorsque la mémoire système est insuffisante et qu’elle n’est pas en mesure d’allouer de la mémoire pour l’installation du service Mobilité. Veillez à libérer suffisamment de mémoire afin que l’installation s’effectue correctement.

## <a name="vss-installation-failures"></a>Échecs d’installation VSS

L’installation de Volume Shadow Copy Service (VSS) fait partie de l’installation de l’agent Mobility. Ce service est utilisé dans le processus pour générer des points de récupération cohérents d’application. Les échecs lors de l’installation VSS peuvent avoir diverses raisons. Pour identifier les erreurs exactes, reportez-vous à _C:\ProgramData\ASRSetupLogs\ASRUnifiedAgentInstaller.log_. Certaines des erreurs courantes et leur procédure de résolution sont mises en évidence dans la section suivante.

### <a name="vss-error--2147023170-0x800706be---exit-code-511"></a>Erreur VSS -2147023170 [0x800706BE] - code de sortie 511

Ce problème se produit le plus souvent lorsqu’un logiciel antivirus bloque le fonctionnement des services Azure Site Recovery.

Pour résoudre ce problème :

1. Consultez la liste des [exclusions de dossiers du programme antivirus](vmware-azure-set-up-source.md#azure-site-recovery-folder-exclusions-from-antivirus-program).
1. Suivez les recommandations publiées par votre fournisseur d’antivirus pour débloquer l’inscription des DLL dans Windows.

### <a name="vss-error-7-0x7---exit-code-511"></a>Erreur VSS 7 [0x7] - code de sortie 511

Cette erreur est une erreur de runtime due à une mémoire insuffisante pour installer VSS. Augmentez l’espace disque pour terminer cette opération.

### <a name="vss-error--2147023824-0x80070430---exit-code-517"></a>Erreur VSS -2147023824 [0x80070430] - code de sortie 517

Cette erreur se produit lorsque le service fournisseur VSS d’Azure Site Recovery est [marqué pour suppression](/previous-versions/ms838153(v=msdn.10)). Essayez d’installer manuellement VSS sur la machine source en exécutant la commande suivante :

`"C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Install.cmd"`

### <a name="vss-error--2147023841-0x8007041f---exit-code-512"></a>Erreur VSS -2147023841 [0x8007041F] - code de sortie 512

Cette erreur se produit lorsque la base de données de service fournisseur VSS d’Azure Site Recovery est [verrouillée](/previous-versions/ms833798(v=msdn.10)). Essayez d’installer manuellement VSS sur la machine source en exécutant la commande suivante à partir d’une invite de commandes :

`"C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Install.cmd"`

En cas d’échec, vérifiez si un programme antivirus ou d’autres services sont bloqués à l’état **Démarrage**. Un processus à l’état **Démarrage** peut conserver le verrou sur les services de base de données. Des échecs lors l'installation du fournisseur VSS se produiront. Assurez-vous qu’aucun service n’est à l’état **Démarrage**, puis recommencez l’opération ci-dessus.

### <a name="vss-exit-code-806"></a>Code de sortie VSS 806

Cette erreur se produit lorsque le compte d’utilisateur utilisé pour l’installation n’a pas d’autorisations pour exécuter la commande `CSScript`. Fournissez les autorisations nécessaires pour le compte d’utilisateur afin d’exécuter le script, et recommencez l’opération.

### <a name="other-vss-errors"></a>Autres erreurs VSS

Essayez d’installer manuellement le service fournisseur VSS sur la machine source en exécutant la commande suivante à partir d’une invite de commandes :

`"C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Install.cmd"`

## <a name="vss-error---0x8004e00f"></a>Erreur VSS - 0x8004E00F

Cette erreur se produit généralement pendant l’installation de l’agent Mobility en raison de problèmes liés à `DCOM`, tels que `DCOM` est dans un état critique.

Utilisez la procédure suivante pour déterminer la cause de l’erreur.

### <a name="examine-the-installation-logs"></a>Examiner les journaux d’installation

1. Ouvrez le journal d’installation situé dans _C:\ProgramData\ASRSetupLogs\ASRUnifiedAgentInstaller.log_.
2. La présence de l’erreur suivante indique ce problème :

    ```Output
    Unregistering the existing application...
    Create the catalogue object
    Get the collection of Applications

    ERROR:

    - Error code: -2147164145 [0x8004E00F]
    - Exit code: 802
    ```

Pour résoudre le problème :

Contactez l’[équipe de la plateforme Microsoft Windows](https://aka.ms/Windows_Support) pour obtenir de l’aide dans la résolution du problème DCOM.

Lorsque le problème DCOM est résolu, réinstallez manuellement le fournisseur VSS d’Azure Site Recovery à l’aide de la commande suivante à partir d’une invite de commandes :

`"C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Install.cmd"`

Si la cohérence des applications n’est pas critique pour vos besoins de récupération d’urgence, vous pouvez contourner l’installation du fournisseur VSS.

Pour contourner l’installation du fournisseur VSS d’Azure Site Recovery et installer manuellement le fournisseur VSS d’Azure Site Recovery après l’installation :

1. Installez le service Mobilité. L’installation échouera à l’étape suivante : **Configurations après installation**.
1. Pour contourner l’installation de VSS :
   1. Ouvrez le répertoire d’installation de Mobility Service d’Azure Site Recovery situé dans :

      _C:\Program Files (x86)\Microsoft Azure Site Recovery\agent_

   1. Modifiez les scripts d’installation du fournisseur VSS d’Azure Site Recovery _InMageVSSProvider_Install_ et _InMageVSSProvider_Uninstall.cmd_ pour qu’ils réussissent toujours en ajoutant les lignes suivantes :

      ```plaintext
      rem @echo off
      setlocal
      exit /B 0
      ```

1. Effectuez une installation manuelle de l’agent Mobility.
1. Lorsque l’installation réussit et passe à l’étape suivante, **Configurer**, supprimez les lignes que vous avez ajoutées.
1. Pour installer le fournisseur VSS, ouvrez une invite de commandes en tant qu’administrateur et exécutez la commande suivante :

   `"C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Install.cmd"`

1. Vérifiez que le fournisseur VSS d’Azure Site Recovery est installé en tant que service dans Services Windows. Ouvrez l’interface MMC Services de composants pour confirmer que le fournisseur VSS est répertorié.
1. Si l’installation du fournisseur VSS continue d’échouer, contactez le support technique pour résoudre les erreurs d’autorisations dans l’interface de programmation d’applications de chiffrement (CAPI2).

## <a name="vss-provider-installation-fails-because-the-cluster-service-being-enabled-on-non-cluster-machine"></a>L’installation du fournisseur VSS échoue en raison de l’activation du service de cluster sur une machine non cluster

Ce problème entraîne l’échec de l’installation de l’agent Mobility d’Azure Site Recovery lors de l’installation du fournisseur VSS d’Azure Site Recovery. L’échec est dû à un problème avec `COM+` qui empêche l’installation du fournisseur VSS.

### <a name="to-identify-the-issue"></a>Pour identifier le problème

Dans le journal situé sur le serveur de configuration dans _C:\ProgramData\ASRSetupLogs\UploadedLogs\<date-time>UA_InstallLogFile.log_, vous trouverez l’exception suivante :

```plaintext
COM+ was unable to talk to the Microsoft Distributed Transaction Coordinator (Exception from HRESULT: 0x8004E00F)
```

Pour résoudre le problème :

1. Vérifiez que cette machine est une machine non cluster et que les composants du cluster ne sont pas utilisés.
1. Si les composants ne sont pas utilisés, supprimez les composants du cluster de la machine.

## <a name="drivers-are-missing-on-the-source-server"></a>Pilotes manquants sur le serveur source

Si l’installation de l’agent Mobility échoue, examinez les journaux situés sous _C:\ProgramData\ASRSetupLogs_ pour déterminer si des pilotes requis sont manquants dans des jeux de contrôles.

Pour résoudre le problème :

1. À l’aide d’un éditeur du Registre tel que `regedit.msc`, ouvrez le Registre.
1. Ouvrez le nœud `HKEY_LOCAL_MACHINE\SYSTEM`.
1. Dans le nœud `SYSTEM`, localisez les jeux de contrôles.
1. Ouvrez chaque jeu de contrôles et vérifiez que les pilotes Windows suivants sont présents :

   * Atapi
   * Vmbus
   * Storflt
   * Storvsc
   * Intelide

1. Réinstallez les pilotes manquants.

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus](vmware-azure-tutorial.md) sur la configuration de la récupération d’urgence de machines virtuelles VMware.
