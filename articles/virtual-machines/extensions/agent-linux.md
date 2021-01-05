---
title: Vue d’ensemble de l’agent de machine virtuelle Linux Azure
description: Apprenez à installer et à configurer l'agent Linux (waagent) pour gérer l'interaction de votre machine virtuelle avec le contrôleur de structure Azure.
author: axayjo
ms.service: virtual-machines-linux
ms.subservice: extensions
ms.workload: infrastructure-services
ms.topic: article
ms.date: 10/17/2016
ms.author: akjosh
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: a4a391e52f80ec171d48718a18764f0f88f34133
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016486"
---
# <a name="understanding-and-using-the-azure-linux-agent"></a>Présentation et utilisation de l’agent Linux Azure

L’agent Microsoft Azure Linux (waagent) gère l’approvisionnement de Linux et FreeBSD, ainsi que l’interaction des machines virtuelles avec le contrôleur de structure Azure. Outre l’agent Linux qui apporte la fonctionnalité de provisionnement, Azure fournit également la possibilité d’utiliser cloud-init pour certains systèmes d’exploitation Linux. L’agent Linux offre les fonctionnalités suivantes pour les déploiements IaaS FreeBSD et Linux :

> [!NOTE]
> Pour plus d’informations, consultez [LISEZ-MOI](https://github.com/Azure/WALinuxAgent/blob/master/README.md).
> 
> 

* **Approvisionnement d’image**
  
  * Création d'un compte d'utilisateur
  * Configuration des types d'authentification SSH
  * Déploiement des clés publiques et des paires de clés SSH
  * Définition du nom d'hôte
  * Publication du nom d'hôte sur la plateforme DNS
  * Génération de rapports sur l'empreinte digitale de la clé d'hôte SSH destinés à la plateforme
  * Gestion du disque de ressources
  * Formatage et montage du disque de ressources
  * Configuration de l'espace d'échange
* **Mise en réseau**
  
  * Gestion des itinéraires afin d'améliorer la compatibilité avec les serveurs DHCP de plateforme
  * Garantie de la stabilité du nom de l'interface réseau
* **Noyau**
  
  * Configuration de l’architecture NUMA virtuelle (désactivée pour le noyau <`2.6.37`)
  * Consommation de l'entropie Hyper-V pour /dev/random
  * Configuration des délais d'expiration SCSI de l'appareil racine (qui peut être distant)
* **Diagnostics**
  
  * Redirection de la console vers le port série
* **Déploiements SCVMM**
  
  * Détection et amorçage de l'agent VMM pour Linux lors de l'exécution dans un environnement System Center Virtual Machine Manager 2012 R2
* **Extension de machine virtuelle**
  
  * Injection de composants créés par Microsoft et ses partenaires dans la machine virtuelle Linux pour activer les logiciels et l’automatisation de la configuration
  * Implémentation de référence d’extension de machine virtuelle sur [https://github.com/Azure/azure-linux-extensions](https://github.com/Azure/azure-linux-extensions)

## <a name="communication"></a>Communication
Le flux d'informations de la plateforme à l'agent se produit via deux canaux :

* Un DVD attaché au moment du démarrage pour les déploiements IaaS. Ce DVD comprend un fichier de configuration compatible OVF qui inclut toutes les informations d'approvisionnement différentes des paires de clés SSH réelles.
* Un point de terminaison TCP qui expose une API REST utilisée pour obtenir la configuration du déploiement et de la topologie.

## <a name="requirements"></a>Spécifications
Les systèmes suivants ont été testés et fonctionnent avec l’agent Linux Azure :

> [!NOTE]
> Cette liste peut être différente de la liste officielle des [distributions prises en charge](../linux/endorsed-distros.md).
> 
> 

* CoreOS
* CentOS 6.3+
* Red Hat Enterprise Linux 6.7+
* Debian 7.0+
* Ubuntu 12.04+
* openSUSE 12.3+
* SLES 11 SP3+
* Oracle Linux 6.4+

Autres systèmes pris en charge :

* FreeBSD 10+ (agent Azure Linux v2.0.10+)

L’agent Linux repose sur certains packages système pour fonctionner correctement :

* Python 2.6+
* OpenSSL 1.0+
* OpenSSH 5.3+
* Utilitaires de système de fichiers : sfdisk, fdisk, mkfs, séparés
* Outils de mot de passe : chpasswd, sudo
* Outils de traitement de texte : sed, grep
* Outils réseau : ip-route
* Prise en charge du noyau pour le montage de systèmes de fichiers UDF.

Vérifiez que votre machine virtuelle a accès à l’adresse IP 168.63.129.16. Pour plus d’informations, consultez [Qu’est-ce que l’adresse IP 168.63.129.16 ?](../../virtual-network/what-is-ip-address-168-63-129-16.md)


## <a name="installation"></a>Installation
L’installation à l’aide d’un package RPM ou DEB à partir de votre référentiel de packages de distribution est la méthode privilégiée pour installer et mettre à niveau l’agent Microsoft Linux Azure. Tous les [fournisseurs de distribution approuvés](../linux/endorsed-distros.md) intègrent l’agent Azure Linux dans leurs images et référentiels.

Consultez la documentation dans le [référentiel de l’agent Linux Azure sur GitHub](https://github.com/Azure/WALinuxAgent) pour connaître les options d’installation avancées, telles que les préfixes et l’installation à partir d’une source ou dans des emplacements personnalisés.

## <a name="command-line-options"></a>Options de ligne de commande
### <a name="flags"></a>Indicateurs
* verbose : accroît le niveau de détail de la commande spécifiée.
* force : ignore la confirmation interactive de certaines commandes.

### <a name="commands"></a>Commandes
* help : liste les commandes et les indicateurs pris en charge.
* deprovision : essaie de nettoyer le système et de le préparer pour un nouveau provisionnement. L’opération suivante supprime :
  
  * toutes les clés de l'hôte SSH (si Provisioning.RegenerateSshHostKeyPair a la valeur « y » dans le fichier de configuration) ;
  * la configuration de Nameserver dans /etc/resolv.conf ;
  * le mot de passe racine de /etc/shadow (si Provisioning.DeleteRootPassword a la valeur « y » dans le fichier de configuration) ;
  * les baux du client DHCP mis en cache.
  * Réinitialise le nom d’hôte sur localhost.localdomain.

> [!WARNING]
> L’annulation de l’approvisionnement ne garantit pas que l’image est exempte de toute information sensible et qu’elle convient à la redistribution.
> 
> 

* deprovision+user : effectue tout ce qui est décrit dans -deprovision (ci-dessus) et supprime aussi le dernier compte d’utilisateur provisionné (obtenu à partir de /var/lib/waagent) et les données associées. Ce paramètre est utilisé pour déprovisionner une image qui a été précédemment provisionnée sur Azure, afin qu’elle soit capturée et réutilisée.
* version : affiche la version de waagent.
* serialconsole : configure GRUB pour marquer ttyS0 (premier port série) en tant que console de démarrage. Les journaux d’activité de démarrage du noyau sont ainsi envoyés au port série et sont prêts à être débogués.
* daemon : exécute waagent en tant que démon afin de gérer l’interaction avec la plateforme. Cet argument est spécifié à waagent dans le script waagent init.
* start : exécute waagent en arrière-plan.

## <a name="configuration"></a>Configuration
Un fichier de configuration (/etc/waagent.conf) contrôle les actions de waagent. Voici un exemple de fichier de configuration :

```config
Provisioning.Enabled=y
Provisioning.DeleteRootPassword=n
Provisioning.RegenerateSshHostKeyPair=y
Provisioning.SshHostKeyPairType=rsa
Provisioning.MonitorHostName=y
Provisioning.DecodeCustomData=n
Provisioning.ExecuteCustomData=n
Provisioning.AllowResetSysUser=n
Provisioning.PasswordCryptId=6
Provisioning.PasswordCryptSaltLength=10
ResourceDisk.Format=y
ResourceDisk.Filesystem=ext4
ResourceDisk.MountPoint=/mnt/resource
ResourceDisk.MountOptions=None
ResourceDisk.EnableSwap=n
ResourceDisk.SwapSizeMB=0
LBProbeResponder=y
Logs.Verbose=n
OS.RootDeviceScsiTimeout=300
OS.OpensslPath=None
HttpProxy.Host=None
HttpProxy.Port=None
AutoUpdate.Enabled=y
```

Les différentes options de configuration suivantes sont décrites. Elles sont de trois types : Boolean (Booléen), String (Chaîne) ou Integer (Entier). Les options de configuration Boolean peuvent être spécifiées à l'aide de « y » (oui) ou « n » (non). Le mot clé « None » (Aucun) peut être utilisé dans le cas de certaines entrées de type chaîne, comme décrit ici :

**Provisioning.Enabled :**  
```txt
Type: Boolean  
Default: y
```
L'utilisateur peut activer ou désactiver la fonctionnalité d'approvisionnement dans l'agent. Les valeurs valides sont « y » ou « n ». Si l'approvisionnement est désactivé, les clés d'utilisateur et d'hôte SSH dans l'image sont conservées et toute configuration spécifiée dans l'API d'approvisionnement Azure est ignorée.

> [!NOTE]
> La valeur par défaut du paramètre `Provisioning.Enabled` est « n » dans les images cloud Ubuntu qui utilisent cloud-init pour l’approvisionnement.
> 
> 

**Provisioning.DeleteRootPassword :**  
```txt
Type: Boolean  
Default: n
```
Si elle est définie, le mot de passe racine dans le fichier /etc/shadow est effacé au cours du processus d'approvisionnement.

**Provisioning.RegenerateSshHostKeyPair :**  
```txt
Type: Boolean  
Default: y
```
Si ce paramètre est défini, toutes les paires de clés d’hôte SSH (ecdsa, dsa et rsa) sont supprimées de /etc/ssh/ au cours du processus de provisionnement. Une nouvelle paire de clés unique est générée.

L'entrée Provisioning.SshHostKeyPairType peut configurer le type de chiffrement pour la nouvelle paire de clés. Certaines distributions recréent les paires de clés SSH pour tout type de chiffrement manquant au redémarrage du démon SSH (par exemple, à un redémarrage système).

**Provisioning.SshHostKeyPairType :**  
```txt
Type: String  
Default: rsa
```
Un type d'algorithme de chiffrement qui est pris en charge par le démon SSH sur la machine virtuelle peut être défini. Les valeurs généralement prises en charge sont « rsa », « dsa » et « ecdsa ». « putty.exe » sur Windows ne prend pas en charge « ecdsa ». Par conséquent, si vous envisagez d’utiliser putty.exe sur Windows pour établir une connexion à un déploiement Linux, utilisez « rsa » ou « dsa ».

**Provisioning.MonitorHostName :**  
```txt
Type: Boolean  
Default: y
```
Si ce paramètre est défini, waagent surveille la machine virtuelle Linux afin de détecter des modifications de nom d’hôte (comme retourné par la commande « hostname »), et met automatiquement à jour la configuration de mise en réseau dans l’image pour refléter la modification. Afin de transmettre la modification de nom aux serveurs DNS, la mise en réseau est redémarrée sur la machine virtuelle. La connexion Internet est alors brièvement interrompue.

**Provisioning.DecodeCustomData**  
```txt
Type: Boolean  
Default: n
```
Si ce paramètre est défini, waagent décode CustomData en Base64.

**Provisioning.ExecuteCustomData**  
```txt
Type: Boolean  
Default: n
```
Si ce paramètre est défini, waagent exécute CustomData après le provisionnement.

**Provisioning.AllowResetSysUser**
```txt
Type: Boolean
Default: n
```
Cette option permet de réinitialiser le mot de passe de l’utilisateur sys. Par défaut, elle est désactivée.

**Provisioning.PasswordCryptId**  
```txt
Type: String  
Default: 6
```
Algorithme utilisé par crypt lors de la génération du hachage de mot de passe.  
 1 - MD5  
 2 a - Blowfish  
 5 - SHA-256  
 6 - SHA-512  

**Provisioning.PasswordCryptSaltLength**  
```txt
Type: String  
Default: 10
```
Longueur de la chaîne salt aléatoire utilisée lors de la génération du hachage de mot de passe.

**ResourceDisk.Format :**  
```txt
Type: Boolean  
Default: y
```
Lorsque ce paramètre est défini, le disque de ressources fourni par la plateforme est formaté et monté par waagent si le type de système de fichiers demandé par l’utilisateur dans « ResourceDisk.Filesystem » est différent de « ntfs ». Une partition unique de type Linux (83) est mise à disposition sur le disque. Cette partition n’est pas formatée si elle peut être correctement montée.

**ResourceDisk.Filesystem :**  
```txt
Type: String  
Default: ext4
```
Cette commande spécifie le type de système de fichiers pour le disque de ressources. Les valeurs prises en charge diffèrent selon la distribution Linux. Si la chaîne est X, mkfs.X doit être présent sur l'image Linux. Les images SLES 11 doivent généralement utiliser « ext3 ». Les images FreeBSD doivent utiliser « ufs2 » ici.

**ResourceDisk.MountPoint :**  
```txt
Type: String  
Default: /mnt/resource 
```
Cette commande spécifie le chemin où le disque de ressources est monté. Le disque de ressources est un disque *temporaire* et il peut être vidé lors du déprovisionnement de la machine virtuelle.

**ResourceDisk.MountOptions**  
```txt
Type: String  
Default: None
```
Spécifie les options de montage de disque à transmettre à la commande mount -o. Les valeurs de cette liste sont séparées par des virgules, par exemple. 'nodev,nosuid'. Pour plus d’informations, consultez mount(8).

**ResourceDisk.EnableSwap :**  
```txt
Type: Boolean  
Default: n
```
Si elle est définie, un fichier d'échange (/swapfile) est créé sur le disque de ressources et est ajouté à l'espace d'échange système.

**ResourceDisk.SwapSizeMB :**  
```txt
Type: Integer  
Default: 0
```
Taille du fichier d'échange en mégaoctets.

**Logs.Verbose :**  
```txt
Type: Boolean  
Default: n
```
Si elle est définie, le niveau de détail du journal est optimisé. Waagent enregistre dans /var/log/waagent.log et utilise la fonctionnalité logrotate du système pour faire tourner les journaux d’activité.

**OS.EnableRDMA**  
```txt
Type: Boolean  
Default: n
```
Si ce paramètre est défini, l’agent tente de s’installer, puis charge un pilote de noyau RDMA qui correspond à la version du microprogramme sur le matériel sous-jacent.

**OS.RootDeviceScsiTimeout :**  
```txt
Type: Integer  
Default: 300
```
Ce paramètre configure le délai d’expiration SCSI en secondes sur le disque du système d’exploitation et les lecteurs de données. Si elle n'est pas définie, les valeurs par défaut du système sont utilisées.

**OS.OpensslPath :**  
```txt
Type: String  
Default: None
```
Ce paramètre sert à spécifier un autre chemin pour les données binaires openssl à utiliser pour les opérations de chiffrement.

**HttpProxy.Host, HttpProxy.Port**  
```txt
Type: String  
Default: None
```
Si ce paramètre est défini, l’agent utilise ce serveur proxy pour accéder à internet. 

**AutoUpdate.Enabled**
```txt
Type: Boolean
Default: y
```
Activer ou désactiver la mise à jour automatique du traitement de l’état de l’objectif. Par défaut : activé.



## <a name="ubuntu-cloud-images"></a>Images cloud Ubuntu
Les images cloud Ubuntu utilisent [cloud-init](https://launchpad.net/ubuntu/+source/cloud-init) pour exécuter de nombreuses tâches de configuration qui devraient normalement être gérées par l’agent Linux Azure. Les différences suivantes s’appliquent :

* **Provisioning.Enabled** est défini par défaut sur n sur les images cloud Ubuntu utilisant Cloud-init pour exécuter les tâches d’approvisionnement.
* Les paramètres de configuration suivants n’ont aucun effet sur les images cloud Ubuntu utilisant Cloud-init pour gérer le disque de ressources et l’espace d’échange :
  
  * **ResourceDisk.Format**
  * **ResourceDisk.Filesystem**
  * **ResourceDisk.MountPoint**
  * **ResourceDisk.EnableSwap**
  * **ResourceDisk.SwapSizeMB**

* Pour plus d’informations, consultez les ressources suivantes pour configurer le point de montage du disque de ressources et l’espace d’échange sur les images cloud Ubuntu durant le provisionnement :
  
  * [Wiki Ubuntu : configurer les partitions d’échange](https://go.microsoft.com/fwlink/?LinkID=532955&clcid=0x409)
  * [Injection de données personnalisées dans une machine virtuelle Azure](../windows/tutorial-automate-vm-deployment.md)