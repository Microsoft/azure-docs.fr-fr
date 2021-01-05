---
title: Configuration de MPIO sur l’hôte Linux StorSimple
description: Apprenez les étapes requises pour configurer la MPIO (gestion multivoie d’E/S) sur votre serveur hôte StorSimple Linux (Centos 6.6).
author: alkohli
ms.assetid: ca289eed-12b7-4e2e-9117-adf7e2034f2f
ms.service: storsimple
ms.topic: how-to
ms.date: 06/12/2019
ms.author: alkohli
ms.openlocfilehash: 6584b2ecc54efd257bb30c479fd0f22150e8d9e1
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96608586"
---
# <a name="configure-mpio-on-a-storsimple-host-running-centos"></a>Configuration de MPIO sur un hôte StorSimple exécutant CentOS
Cet article explique les étapes requises pour configurer la MPIO (gestion multivoie d’E/S) sur votre serveur hôte CentOS 6.6. Le serveur hôte est connecté à votre appareil Microsoft Azure StorSimple pour la haute disponibilité via les initiateurs iSCSI. Il décrit en détail la détection automatique des appareils multivoies et l’installation spécifique uniquement pour les volumes StorSimple.

Cette procédure s’applique à tous les modèles d’appareils StorSimple série 8000.

> [!NOTE]
> Cette procédure ne peut pas être utilisée pour un StorSimple Cloud Appliance. Pour plus d’informations, consultez Configuration des serveurs hôtes de votre appliance cloud.

> [!NOTE]
> Cet article contient des références au terme *liste noire*, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.


## <a name="about-multipathing"></a>À propos de la gestion multivoie
La fonctionnalité de gestion multivoie vous permet de configurer plusieurs chemins d’accès d’E/S entre un serveur hôte et un appareil de stockage. Ces chemins d’accès d’E/S sont des connexions SAN physiques pouvant inclure des câbles distincts, des commutateurs, des interfaces réseau et des contrôleurs. La gestion multivoie agrège les chemins d’accès d’E/S pour configurer un nouvel appareil qui est associé à tous les chemins d’accès agrégés.

L’objectif de la gestion multivoie est double :

* **Haute disponibilité** : elle fournit un autre chemin en cas d’échec d’un élément du chemin d’E/S (p. ex., câble, commutateur, interface réseau ou contrôleur).
* **Équilibrage de la charge** : selon la configuration de votre appareil de stockage, elle peut améliorer les performances en détectant les charges sur les chemins d’E/S et en les rééquilibrant dynamiquement.

### <a name="about-multipathing-components"></a>À propos des composants de la gestion multivoie
Sous Linux, la gestion multivoie comprend des composants du noyau et des composants de l’espace utilisateur comme présenté ci-dessous.

* **Noyau** : le composant principal est le mappeur d’appareils (*device-mapper*) qui redirige les E/S et prend en charge le basculement pour les chemins et les groupes de chemin.

* **Espace utilisateur** : il s’agit des outils multipath (*multipath-tools*) qui permettent de gérer les appareils multichemins (multipath) en indiquant la marche à suivre au module multipath du mappeur d’appareils (device-mapper). Les outils comprennent les éléments suivants :
   
   * **Multipath**: répertorie et configure les appareils à chemins d’accès multiples.
   * **Multipathd**: démon qui exécute la gestion multivoie et surveille les chemins d’accès.
   * **Devmap-name**: fournit un nom d’appareil explicite vers udev pour devmaps.
   * **Kpartx**: mappe les éléments devmaps linéaires aux partitions d’appareils pour effectuer des mappages multivoies configurables en partition.
   * **Multipath.conf**: fichier de configuration pour démon multivoie, utilisé pour remplacer la table de configuration intégrée.

### <a name="about-the-multipathconf-configuration-file"></a>À propos du fichier de configuration multipath.conf
Le fichier de configuration `/etc/multipath.conf` rend un grand nombre de fonctionnalités de gestion multivoie configurables par l’utilisateur. La commande `multipath` et le démon de noyau `multipathd` utilisent les informations figurant dans ce fichier. Le fichier est consulté uniquement lors de la configuration des appareils multivoies. Assurez-vous d’avoir effectué toutes les modifications avant d’exécuter la commande `multipath` . Si vous modifiez le fichier par la suite, vous devrez arrêter, puis redémarrer multipathd pour que les modifications prennent effet.

Le fichier multipath.conf comporte cinq sections :

- **Valeurs par défaut au niveau du système** *(valeurs par défaut)*  : Vous pouvez remplacer les valeurs par défaut au niveau du système.
- **Appareils sur liste rouge** *(blacklist)*  : vous pouvez spécifier la liste des appareils qui ne doivent pas être contrôlés par le mappeur d’appareils.
- **Exceptions de la liste rouge** *(blacklist_exceptions)*  : vous pouvez identifier les appareils à traiter en tant qu’appareils multichemins (multipath), même s’ils figurent dans la liste rouge.
- **Paramètres concernant le contrôleur de stockage** *(devices)*  : vous pouvez spécifier les paramètres de configuration qui seront appliqués aux appareils contenant des informations de produit et de fournisseur.
- **Paramètres propres aux appareils** *(multipaths)*  : vous pouvez utiliser cette section pour optimiser les paramètres de configuration des numéros d’unité logique.

## <a name="configure-multipathing-on-storsimple-connected-to-linux-host"></a>Configuration de la gestion multivoie sur StorSimple connecté à l’hôte Linux
Un appareil StorSimple connecté à un hôte Linux peut être configuré pour la haute disponibilité et l’équilibrage de charge. Par exemple, si l’hôte Linux possède deux interfaces connectées au réseau SAN et que l’appareil possède deux interfaces connectées au réseau SAN de telle sorte qu’elles sont sur le même sous-réseau, 4 chemins d’accès sont disponibles. Cependant, si les interfaces DATA sur l’appareil et l’interface de l’hôte sont sur un autre sous-réseau IP (non routable), alors seuls 2 chemins d’accès sont disponibles. Vous pouvez configurer la gestion multivoie pour détecter automatiquement tous les chemins d’accès disponibles, choisir un algorithme d’équilibrage de charge pour ces chemins d’accès, appliquer des paramètres de configuration spécifiques pour les volumes StorSimple uniquement, puis activer et vérifier la gestion multivoie.

La procédure suivante décrit comment configurer la gestion multivoie lorsqu’un appareil StorSimple possédant deux interfaces réseau est connecté à un hôte avec deux interfaces réseau.

## <a name="prerequisites"></a>Prérequis
Cette section détaille la configuration requise pour le serveur CentOS et votre appareil StorSimple.

### <a name="on-centos-host"></a>Sur l’hôte CentOS
1. Assurez-vous que votre hôte CentOS possède 2 interfaces réseau activées. Tapez :
   
    `ifconfig`
   
    L’exemple suivant montre la sortie lorsque deux interfaces réseau (`eth0` et `eth1`) sont présentes sur l’hôte.
   
    ```output
    [root@centosSS ~]# ifconfig
    eth0  Link encap:Ethernet  HWaddr 00:15:5D:A2:33:41  
        inet addr:10.126.162.65  Bcast:10.126.163.255  Mask:255.255.252.0
        inet6 addr: 2001:4898:4010:3012:215:5dff:fea2:3341/64 Scope:Global
        inet6 addr: fe80::215:5dff:fea2:3341/64 Scope:Link
        UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
        RX packets:36536 errors:0 dropped:0 overruns:0 frame:0
        TX packets:6312 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:1000
        RX bytes:13994127 (13.3 MiB)  TX bytes:645654 (630.5 KiB)

    eth1  Link encap:Ethernet  HWaddr 00:15:5D:A2:33:42  
        inet addr:10.126.162.66  Bcast:10.126.163.255  Mask:255.255.252.0
        inet6 addr: 2001:4898:4010:3012:215:5dff:fea2:3342/64 Scope:Global
        inet6 addr: fe80::215:5dff:fea2:3342/64 Scope:Link
        UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
        RX packets:25962 errors:0 dropped:0 overruns:0 frame:0
        TX packets:11 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:1000
        RX bytes:2597350 (2.4 MiB)  TX bytes:754 (754.0 b)

    loLink encap:Local Loopback  
        inet addr:127.0.0.1  Mask:255.0.0.0
        inet6 addr: ::1/128 Scope:Host
        UP LOOPBACK RUNNING  MTU:65536  Metric:1
        RX packets:12 errors:0 dropped:0 overruns:0 frame:0
        TX packets:12 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:0
        RX bytes:720 (720.0 b)  TX bytes:720 (720.0 b)
    ```
1. Installez *iSCSI-initiator-utils* sur votre serveur CentOS. Effectuez les opérations suivantes pour installer *iSCSI-initiator-utils*.
   
   1. Connectez-vous en tant que `root` à votre hôte CentOS.
   1. Installez *iSCSI-initiator-utils*. Tapez :
      
       `yum install iscsi-initiator-utils`
   1. Après avoir correctement installé *iSCSI-Initiator-utils* , démarrez le service iSCSI. Tapez :
      
       `service iscsid start`
      
       Parfois, `iscsid` peut ne pas démarrer et l’option `--force` peut être nécessaire.
   1. Pour vous assurer que votre initiateur iSCSI est activé à l’heure de démarrage, utilisez la commande `chkconfig` pour activer le service.
      
       `chkconfig iscsi on`
   1. Pour vérifier que l’installation s’est correctement effectuée, exécutez la commande :
      
       `chkconfig --list | grep iscsi`
      
       Voici un exemple de sortie obtenue.
      
        ```output
        iscsi   0:off   1:off   2:on3:on4:on5:on6:off
        iscsid  0:off   1:off   2:on3:on4:on5:on6:off
        ```
      
       Dans l’exemple ci-dessus, vous pouvez voir que votre environnement iSCSI s’exécutera à l’heure de démarrage sur les niveaux d’exécution 2, 3, 4 et 5.
1. Installez *device-mapper-multipath*. Tapez :
   
    `yum install device-mapper-multipath`
   
    L’installation démarre. Tapez **Y** pour continuer lorsque vous êtes invité à confirmer l’opération.

### <a name="on-storsimple-device"></a>Sur l’appareil StorSimple
Votre appareil StorSimple doit disposer des éléments suivants :

* Deux interfaces activées pour iSCSI minimum. Pour vérifier que deux interfaces sont compatibles iSCSI sur votre appareil StorSimple, effectuez les opérations suivantes dans le portail Azure Classic de votre appareil StorSimple :
  
  1. Connectez-vous au portail Classic de votre appareil StorSimple.
  1. Sélectionnez votre service StorSimple Manager, cliquez sur **Appareils** et choisissez l’appareil StorSimple concerné. Cliquez sur **Configurer** et vérifiez les paramètres d’interface réseau. Une capture d’écran avec deux interfaces réseau compatibles iSCSI est affichée ci-dessous. Ici, pour DATA 2 et DATA 3, les deux interfaces 10 Gigabit Ethernet sont activées pour iSCSI.
     
      ![Configuration de MPIO StorSimple DATA 2](./media/storsimple-configure-mpio-on-linux/IC761347.png)
     
      ![Configuration de MPIO StorSimple DATA 3](./media/storsimple-configure-mpio-on-linux/IC761348.png)
     
      Sur la page **Configurer**
     
     1. Assurez-vous que les deux interfaces réseau sont compatibles iSCSI. Le champ **Compatible iSCSI** doit être défini sur **Oui**.
     1. Vérifiez que les interfaces réseau ont la même vitesse, les deux devant être définies sur 1 Gigabit Ethernet ou 10 Gigabit Ethernet.
     1. Notez les adresses IPv4 des interfaces compatibles iSCSI et conservez-les pour une utilisation ultérieure sur l’hôte.
* Les interfaces iSCSI sur votre appareil StorSimple doivent être accessibles à partir du serveur CentOS.
      Pour vérifier ceci, vous devez fournir les adresses IP de vos interfaces réseau compatibles iSCSI StorSimple sur votre serveur hôte. Les commandes utilisées et la sortie correspondante avec DATA2 (10.126.162.25) et DATA3 (10.126.162.26) sont présentées ci-dessous :
  
    ```console
    [root@centosSS ~]# iscsiadm -m discovery -t sendtargets -p 10.126.162.25:3260
    10.126.162.25:3260,1 iqn.1991-05.com.microsoft:storsimple8100-shx0991003g44mt-target
    10.126.162.26:3260,1 iqn.1991-05.com.microsoft:storsimple8100-shx0991003g44mt-target
    ```

### <a name="hardware-configuration"></a>Configuration matérielle
Nous vous recommandons de connecter les deux interfaces réseau iSCSI sur des chemins d’accès distincts pour la redondance. La figure ci-dessous illustre la configuration matérielle recommandée pour la gestion multivoie haute disponibilité et équilibrage de charge pour votre serveur CentOS et votre appareil StorSimple.

![Configuration matérielle de MPIO pour StorSimple sur l’hôte Linux](./media/storsimple-configure-mpio-on-linux/MPIOHardwareConfigurationStorSimpleToLinuxHost2M.png)

Comme indiqué dans la figure précédente :

* Votre appareil StorSimple est une configuration active/passive avec deux contrôleurs.
* Deux commutateurs SAN sont connectés à vos contrôleurs d’appareils.
* Deux initiateurs iSCSI sont activés sur votre appareil StorSimple.
* Deux interfaces réseau sont activées sur votre hôte CentOS.

La configuration ci-dessus produira 4 chemins d’accès distincts entre votre appareil et l’hôte si les interfaces de l’hôte et des données sont routables.

> [!IMPORTANT]
> * Nous vous recommandons de ne pas mélanger les interfaces réseau 1 Gigabit Ethernet et 10 Gigabit Ethernet pour la gestion multivoie. Lorsque vous utilisez deux interfaces réseau, elles doivent être du même type.
> * Sur votre appareil StorSimple, DATA0, DATA1, DATA4 et DATA5 sont des interfaces 1 Gigabit Ethernet, alors que DATA2 et DATA3 sont des interfaces réseau 10 Gigabit Ethernet.|
> 
> 

## <a name="configuration-steps"></a>Configuration
Les étapes de configuration pour la gestion multivoie impliquent la configuration des chemins d’accès disponibles pour la détection automatique, la spécification de l’algorithme d’équilibrage de charge à utiliser, l’activation de la gestion multivoie et enfin la vérification de la configuration. Chacune de ces étapes est abordée en détail dans les sections suivantes.

### <a name="step-1-configure-multipathing-for-automatic-discovery"></a>Étape 1 : Configurer le multipathing pour la découverte automatique
Les appareils pris en charge par la gestion multivoie peuvent être automatiquement détectés et configurés.

1. Initialisez le fichier `/etc/multipath.conf` . Tapez :
   
     `mpathconf --enable`
   
    La commande ci-dessus crée un fichier `sample/etc/multipath.conf` .
1. Démarrez le service de gestion multivoie. Tapez :
   
    `service multipathd start`
   
    La sortie suivante s’affiche :
   
    `Starting multipathd daemon:`
1. Activez la détection automatique des chemins d’accès multiples. Tapez :
   
    `mpathconf --find_multipaths y`
   
    Cela modifiera la section defaults de votre `multipath.conf` comme indiqué ci-dessous :
   
    ```config
    defaults {
    find_multipaths yes
    user_friendly_names yes
    path_grouping_policy multibus
    }
    ```

### <a name="step-2-configure-multipathing-for-storsimple-volumes"></a>Étape 2 : Configurer le multipathing pour les volumes StorSimple
Par défaut, tous les appareils sont sur liste rouge dans le fichier multipath.conf et seront contournés. Vous devrez créer des exceptions de la liste rouge pour autoriser la gestion multivoie pour les volumes à partir des appareils StorSimple.

1. Modifiez le fichier `/etc/mulitpath.conf` . Tapez :
   
    `vi /etc/multipath.conf`
1. Recherchez la section blacklist_exceptions dans le fichier multipath.conf. Votre appareil StorSimple doit être répertorié comme une exception de la liste rouge
dans cette section. Vous pouvez supprimer les marques de commentaire des lignes appropriées dans ce fichier pour le modifier comme indiqué ci-dessous (utilisez uniquement le modèle spécifique de l’appareil que vous utilisez) :
   
    ```config
    blacklist_exceptions {
        device {
                    vendor  "MSFT"
                    product "STORSIMPLE 8100*"
        }
        device {
                    vendor  "MSFT"
                    product "STORSIMPLE 8600*"
        }
    }
    ```

### <a name="step-3-configure-round-robin-multipathing"></a>Étape 3 : Configurer le multipathing de tourniquet (round robin)
Cet algorithme d’équilibrage de charge utilise tous les chemins d’accès multiples disponibles pour le contrôleur actif de manière équilibrée, en tourniquet (round robin).

1. Modifiez le fichier `/etc/multipath.conf` . Tapez :
   
    `vi /etc/multipath.conf`
1. Sous la section `defaults`, définissez `path_grouping_policy` sur `multibus`. `path_grouping_policy` spécifie la stratégie de regroupement de chemins d’accès par défaut à appliquer aux chemins d’accès multiples non spécifiés. La section defaults ressemblera à ce qui est présenté ci-dessous.
   
    ```config
    defaults {
            user_friendly_names yes
            path_grouping_policy multibus
    }
    ```

> [!NOTE]
> Les valeurs les plus courantes de `path_grouping_policy` incluent :
> 
> * basculement = 1 chemin d’accès par groupe de priorité
> * multibus = tous les chemins d’accès valides dans 1 groupe de priorité
> 
> 

### <a name="step-4-enable-multipathing"></a>Étape 4 : Activer le multipathing
1. Redémarrez le démon `multipathd` . Tapez :
   
    `service multipathd restart`
1. La sortie sera comme indiqué ci-dessous :
   
    ```output
    [root@centosSS ~]# service multipathd start
    Starting multipathd daemon:  [OK]
    ```

### <a name="step-5-verify-multipathing"></a>Étape 5 : Vérifier le multipathing
1. Tout d’abord, assurez-vous que la connexion iSCSI est établie avec l’appareil StorSimple comme suit :
   
   a. Détectez votre appareil StorSimple. Tapez :
      
    `iscsiadm -m discovery -t sendtargets -p  <IP address of network interface on the device>:<iSCSI port on StorSimple device>`
    
    La sortie lorsque l’adresse IP pour DATA0 est 10.126.162.25 et que le port 3260 est ouvert sur l’appareil StorSimple pour le trafic iSCSI sortant est comme indiqué ci-dessous :
    
    ```output
    10.126.162.25:3260,1 iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target
    10.126.162.26:3260,1 iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target
    ```

    Copiez le nom qualifié de votre appareil StorSimple, `iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target`, à partir de la sortie précédente.

   b. Connectez-vous à l’appareil à l’aide du nom qualifié cible. L’appareil StorSimple est la cible iSCSI ici. Tapez :

      `iscsiadm -m node --login -T <IQN of iSCSI target>`

    L’exemple suivant illustre la sortie avec un nom qualifié cible de `iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target`. La sortie indique que vous vous êtes correctement connecté aux deux interfaces réseau compatibles iSCSI sur votre appareil.

    ```output
    Logging in to [iface: eth0, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.25,3260] (multiple)
    Logging in to [iface: eth1, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.25,3260] (multiple)
    Logging in to [iface: eth0, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.26,3260] (multiple)
    Logging in to [iface: eth1, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.26,3260] (multiple)
    Login to [iface: eth0, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.25,3260] successful.
    Login to [iface: eth1, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.25,3260] successful.
    Login to [iface: eth0, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.26,3260] successful.
    Login to [iface: eth1, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.26,3260] successful.
    ```

    Si vous ne voyez qu’une seule interface hôte et deux chemins d’accès ici, vous devez activer les deux interfaces sur l’hôte pour iSCSI. Vous pouvez suivre les [instructions détaillées dans la documentation Linux](https://access.redhat.com/documentation/Red_Hat_Enterprise_Linux/5/html/Online_Storage_Reconfiguration_Guide/iscsioffloadmain.html).

1. Un volume est exposé au serveur CentOS à partir de l’appareil StorSimple. Pour plus d’informations, consultez [Étape 6 : Créer un volume](storsimple-8000-deployment-walkthrough-u2.md#step-6-create-a-volume) via le portail Azure sur votre appareil StorSimple.

1. Vérifiez les chemins d’accès disponibles. Tapez :

    `multipath -l`

      L’exemple suivant montre la sortie de deux interfaces réseau sur un appareil StorSimple connecté à une interface réseau de l’hôte unique avec deux chemins d’accès disponibles.

    ```output
    mpathb (36486fd20cc081f8dcd3fccb992d45a68) dm-3 MSFT,STORSIMPLE 8100
    size=100G features='0' hwhandler='0' wp=rw
    `-+- policy='round-robin 0' prio=0 status=active
    |- 7:0:0:1 sdc 8:32 active undef running
    `- 6:0:0:1 sdd 8:48 active undef running
    ```

    L’exemple suivant montre la sortie de deux interfaces réseau sur un appareil StorSimple connecté à deux interfaces réseau de l’hôte avec quatre chemins d’accès disponibles.

    ```output
    mpathb (36486fd27a23feba1b096226f11420f6b) dm-2 MSFT,STORSIMPLE 8100
    size=100G features='0' hwhandler='0' wp=rw
    `-+- policy='round-robin 0' prio=0 status=active
    |- 17:0:0:0 sdb 8:16 active undef running
    |- 15:0:0:0 sdd 8:48 active undef running
    |- 14:0:0:0 sdc 8:32 active undef running
    `- 16:0:0:0 sde 8:64 active undef running
    ```

    Après avoir configuré les chemins d’accès, reportez-vous aux instructions spécifiques de votre système d’exploitation hôte (CentOS 6.6) pour monter et formater ce volume.

## <a name="troubleshoot-multipathing"></a>Résolution des problèmes de la gestion multivoie
Cette section fournit des conseils utiles si vous rencontrez des problèmes lors de la configuration de la gestion multivoie.

Q. Je ne vois pas les modifications apportées au fichier `multipath.conf` s’appliquer.

R. Si vous avez apporté des modifications au fichier `multipath.conf` , vous devez redémarrer le service de gestion multivoie. Tapez la commande suivante :

`service multipathd restart`

Q. J’ai activé deux interfaces réseau sur l’appareil StorSimple et deux interfaces réseau sur l’hôte. Lorsque je répertorie les chemins d’accès disponibles, je ne vois que deux chemins d’accès. Je m’attendais à voir quatre chemins d’accès disponibles.

R. Vérifiez que les deux chemins d’accès sont sur le même sous-réseau et routables. Si les interfaces réseau se trouvent sur des réseaux locaux virtuels différents et non routables, vous ne verrez que deux chemins d’accès. L’un des moyens de vérifier cela consiste à vous assurer que vous pouvez atteindre les deux interfaces hôte à partir d’une interface réseau sur l’appareil StorSimple. Vous devrez [contacter le support technique Microsoft](storsimple-8000-contact-microsoft-support.md) car cette vérification ne peut être effectuée qu’au moyen d’une session de support.

Q. Lorsque je répertorie les chemins d’accès disponibles, je ne vois aucune sortie.

R. En règle générale, le non-affichage de chemins d’accès multiples suggère un problème avec le démon de gestion multivoie et il est très probable que le problème éventuel réside dans le fichier `multipath.conf`.

Il convient également de vérifier que vous pouvez voir certains disques après la connexion à la cible, car l’absence de réponse des listes multivoies peut signifier que vous n’avez aucun disque.

* Pour relancer l’analyse du bus SCSI, utilisez la commande suivante :
  
    `$ rescan-scsi-bus.sh` (dans le package sg3_utils)
* Tapez les commandes suivantes :
  
    `$ dmesg | grep sd*`
     
     ou
  
    `$ fdisk -l`
  
    Celles-ci renverront les détails des disques récemment ajoutés.
* Pour déterminer s’il s’agit d’un disque StorSimple, utilisez les commandes suivantes :
  
    `cat /sys/block/<DISK>/device/model`
  
    Cela renvoie une chaîne qui déterminera s’il s’agit d’un disque StorSimple.

Cause moins probable mais possible, la présence d’un PID iSCSI obsolète. Utilisez la commande suivante pour vous déconnecter des sessions iSCSI :

`iscsiadm -m node --logout -p <Target_IP>`

Répétez cette commande pour toutes les interfaces réseau connectées sur la cible iSCSI, qui est votre appareil StorSimple. Une fois que vous vous êtes déconnecté de toutes les sessions iSCSI, utilisez le nom qualifié cible iSCSI pour rétablir la session iSCSI. Tapez la commande suivante :

`iscsiadm -m node --login -T <TARGET_IQN>`


Q. Je ne sais pas si mon appareil est autorisé.

R. Pour vérifier si votre appareil est autorisé, utilisez cette commande interactive de résolution des problèmes :

```console
multipathd -k
multipathd> show devices
available block devices:
ram0 devnode blacklisted, unmonitored
ram1 devnode blacklisted, unmonitored
ram2 devnode blacklisted, unmonitored
ram3 devnode blacklisted, unmonitored
ram4 devnode blacklisted, unmonitored
ram5 devnode blacklisted, unmonitored
ram6 devnode blacklisted, unmonitored
ram7 devnode blacklisted, unmonitored
ram8 devnode blacklisted, unmonitored
ram9 devnode blacklisted, unmonitored
ram10 devnode blacklisted, unmonitored
ram11 devnode blacklisted, unmonitored
ram12 devnode blacklisted, unmonitored
ram13 devnode blacklisted, unmonitored
ram14 devnode blacklisted, unmonitored
ram15 devnode blacklisted, unmonitored
loop0 devnode blacklisted, unmonitored
loop1 devnode blacklisted, unmonitored
loop2 devnode blacklisted, unmonitored
loop3 devnode blacklisted, unmonitored
loop4 devnode blacklisted, unmonitored
loop5 devnode blacklisted, unmonitored
loop6 devnode blacklisted, unmonitored
loop7 devnode blacklisted, unmonitored
sr0 devnode blacklisted, unmonitored
sda devnode whitelisted, monitored
dm-0 devnode blacklisted, unmonitored
dm-1 devnode blacklisted, unmonitored
dm-2 devnode blacklisted, unmonitored
sdb devnode whitelisted, monitored
sdc devnode whitelisted, monitored
dm-3 devnode blacklisted, unmonitored
```


Pour plus d’informations, accédez à [Résolution des problèmes de multipathing](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/dm_multipath/mpio_admin-troubleshoot).

## <a name="list-of-useful-commands"></a>Liste des commandes utiles
| Type | Commande | Description |
| --- | --- | --- |
| **iSCSI** |`service iscsid start` |Démarrer le service iSCSI |
| &nbsp; |`service iscsid stop` |Arrêter le service iSCSI |
| &nbsp; |`service iscsid restart` |Redémarrer le service iSCSI |
| &nbsp; |`iscsiadm -m discovery -t sendtargets -p <TARGET_IP>` |Détecter les cibles disponibles à l'adresse spécifiée |
| &nbsp; |`iscsiadm -m node --login -T <TARGET_IQN>` |Se connecter à la cible iSCSI |
| &nbsp; |`iscsiadm -m node --logout -p <Target_IP>` |Se déconnecter de la cible iSCSI |
| &nbsp; |`cat /etc/iscsi/initiatorname.iscsi` |Imprimer le nom de l'initiateur iSCSI |
| &nbsp; |`iscsiadm -m session -s <sessionid> -P 3` |Vérifier l'état de la session iSCSI et le volume détecté sur l’hôte |
| &nbsp; |`iscsi -m session` |Affiche toutes les sessions iSCSI établies entre l'hôte et l'appareil StorSimple |
|  | | |
| **Gestion multivoie** |`service multipathd start` |Démarrer le démon multivoie |
| &nbsp; |`service multipathd stop` |Arrêter le démon multivoie |
| &nbsp; |`service multipathd restart` |Redémarrer le démon multivoie |
| &nbsp; |`chkconfig multipathd on` </br> OR </br> `mpathconf -with_chkconfig y` |Activer le lancement du démon multivoie au démarrage |
| &nbsp; |`multipathd -k` |Démarrer la console interactive pour le dépannage |
| &nbsp; |`multipath -l` |Lister les connexions et les périphériques multivoie |
| &nbsp; |`mpathconf --enable` |Créer un exemple de fichier mulitpath.conf dans `/etc/mulitpath.conf` |
|  | | |

## <a name="next-steps"></a>Étapes suivantes
Étant donné que vous configurez MPIO sur un hôte Linux, vous pouvez avoir besoin de consulter les documents CentOS 6.6 suivants :

* [Configuration de MPIO sur CentOS](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/dm_multipath/index)
* [Guide de formation Linux](http://linux-training.be/linuxsys.pdf)
