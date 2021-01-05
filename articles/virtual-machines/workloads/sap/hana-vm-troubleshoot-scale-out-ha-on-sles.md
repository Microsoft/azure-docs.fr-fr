---
title: Résolution de problèmes de HSR-Pacemaker pour SAP HANA Scale-Out avec SLES sur des machines virtuelles Azure | Microsoft Docs
description: Guide permettant de vérifier et corriger une configuration complexe SAP HANA scale-out à haute disponibilité, à partir de SAP HANA System Replication (HSR) et Pacemaker sur SLES 12 SP3 s’exécutant sur des machines virtuelles Azure
services: virtual-machines-linux
documentationcenter: ''
author: hermanndms
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/24/2018
ms.author: hermannd
ms.openlocfilehash: 87758100299eb170a7950a1a7a2c6bd2029b27fb
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96621550"
---
# <a name="verify-and-troubleshoot-sap-hana-scale-out-high-availability-setup-on-sles-12-sp3"></a>Vérifier et corriger la configuration SAP HANA scale-out à haute disponibilité sur SLES 12 SP3 

[sles-pacemaker-ha-guide]:high-availability-guide-suse-pacemaker.md
[sles-hana-scale-out-ha-paper]:https://www.suse.com/documentation/suse-best-practices/singlehtml/SLES4SAP-hana-scaleOut-PerfOpt-12/SLES4SAP-hana-scaleOut-PerfOpt-12.html
[sap-hana-iaas-list]:https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html
[suse-pacemaker-support-log-files]:https://www.suse.com/support/kb/doc/?id=7022702
[azure-linux-multiple-nics]:https://docs.microsoft.com/azure/virtual-machines/linux/multiple-nics
[suse-cloud-netconfig]:https://www.suse.com/c/multi-nic-cloud-netconfig-ec2-azure/
[sap-list-port-numbers]:https://help.sap.com/viewer/ports
[sles-12-ha-paper]:https://www.suse.com/documentation/sle-ha-12/pdfdoc/book_sleha/book_sleha.pdf
[sles-zero-downtime-paper]:https://www.suse.com/media/presentation/TUT90846_towards_zero_downtime%20_how_to_maintain_sap_hana_system_replication_clusters.pdf
[sap-nw-ha-guide-sles]:high-availability-guide-suse.md
[sles-12-for-sap]:https://www.suse.com/media/white-paper/suse_linux_enterprise_server_for_sap_applications_12_sp1.pdf


Cet article vous aide à vérifier la configuration du cluster Pacemaker pour SAP HANA scale-out s’exécutant sur des machines virtuelles Azure. La configuration de cluster a été effectuée en association avec SAP HANA System Replication (HSR) et le package SUSE RPM SAPHanaSR-ScaleOut. Tous les tests ont été effectués sur SUSE SLES 12 SP3 uniquement. Les sections de l’article couvrent différents domaines et présentent des exemples de commandes ainsi que des extraits de fichiers config. Ces exemples sont recommandés en guise de méthode pour vérifier l’ensemble de la configuration du cluster.



## <a name="important-notes"></a>Remarques importantes

Tous les tests pour SAP HANA scale-out en association avec SAP HANA System Replication et Pacemaker ont été effectués avec SAP HANA 2.0 uniquement. La version de système d’exploitation est SUSE Linux Enterprise Server 12 SP3 pour les applications SAP. Le dernier package RPM, SAPHanaSR-ScaleOut de SUSE, a été utilisé pour configurer le cluster Pacemaker.
SUSE a publié une [description détaillée de cette configuration optimisée pour les performances][sles-hana-scale-out-ha-paper].

Pour connaître les types de machine virtuelle pris en charge pour SAP HANA scale-out, consultez [l’annuaire des plateformes IaaS certifiées SAP HANA][sap-hana-iaas-list].

> [!NOTE]
> Cet article contient des références aux termes *maître* et *esclave*, termes que Microsoft n’utilise plus. Lorsque ces termes seront supprimés du logiciel, nous les supprimerons de cet article.

Nous avons rencontré un problème technique en associant SAP HANA scale-out à plusieurs sous-réseaux et cartes réseau virtuelles, et pendant la configuration de HSR. Vous devez obligatoirement utiliser les derniers correctifs de SAP HANA 2.0 qui corrigent ce problème. Les versions suivantes de SAP HANA sont prises en charge : 

* rev2.00.024.04 ou version ultérieure 
* rev2.00.032 ou version ultérieure

Si vous avez besoin de la prise en charge de SUSE, suivez ce [guide][suse-pacemaker-support-log-files]. Collectez toutes les informations sur le cluster SAP HANA à haute disponibilité, comme décrit dans l’article. Le support de SUSE a besoin de ces informations pour approfondir l’analyse.

Pendant les tests internes, la configuration du cluster a été perturbée par un arrêt approprié normal des machines virtuelles via le portail Azure. Par conséquent, nous vous recommandons de tester d’autres méthodes de basculement de cluster. Utilisez des méthodes qui consistent, par exemple, à forcer une alerte sur le noyau, arrêter les réseaux ou migrer la ressource **msl**. Pour en savoir plus, consultez les sections suivantes. Nous partons du principe qu’un arrêt standard se produit intentionnellement. Le meilleur exemple d’un arrêt intentionnel est celui utilisé pour la maintenance. Pour plus d’informations, consultez [Maintenance planifiée](#planned-maintenance).

En outre, pendant les tests internes, la configuration du cluster a été perturbée à la suite d’une prise de contrôle manuelle de SAP HANA pendant que le cluster était en mode maintenance. Nous vous recommandons de le basculer à nouveau manuellement avant d’arrêter le mode de maintenance de cluster. Une autre option consiste à déclencher un basculement avant de placer le cluster en mode de maintenance. Plus d’informations, consultez [Maintenance planifiée](#planned-maintenance). La documentation de SUSE décrit comment réinitialiser le cluster à cet effet à l’aide de la commande **crm**. Toutefois, l’approche mentionnée auparavant s’est avérée plutôt robuste pendant les tests internes et n’a jamais entraîné d’effets secondaires inattendus.

Lorsque vous utilisez la commande **crm migrate**, veillez à nettoyer la configuration du cluster. Elle ajoute des contraintes d’emplacement dont vous pouvez ne pas avoir connaissance. Ces contraintes affectent le comportement d’un cluster. Pour plus d’informations, consultez [Maintenance planifiée](#planned-maintenance).



## <a name="test-system-description"></a>Description du système de test

 Pour la vérification et la certification de SAP HANA scale-out à haute disponibilité, un programme d’installation a été utilisé. Il était composé de deux systèmes avec trois nœuds SAP HANA chacun : un maître et deux workers. Le tableau suivant répertorie les noms des machines virtuelles et les adresses IP internes. Tous les exemples de vérification indiqués plus loin ont été effectués sur ces machines virtuelles. En utilisant ces noms de machine virtuelle et ces adresses IP dans les exemples de commande, vous pouvez mieux comprendre les commandes et leurs résultats :


| Type de nœud | nom de la machine virtuelle | Adresse IP |
| --- | --- | --- |
| Nœud master sur le site 1 | hso-hana-vm-s1-0 | 10.0.0.30 |
| Nœud worker 1 sur le site 1 | hso-hana-vm-s1-1 | 10.0.0.31 |
| Nœud worker 2 sur le site 1 | hso-hana-vm-s1-2 | 10.0.0.32 |
| | | |
| Nœud master sur le site 2 | hso-hana-vm-s2-0 | 10.0.0.40 |
| Nœud worker 1 sur le site 2 | hso-hana-vm-s2-1 | 10.0.0.41 |
| Nœud worker 2 sur le site 2 | hso-hana-vm-s2-2  | 10.0.0.42 |
| | | |
| Nœud majority maker | hso-hana-dm | 10.0.0.13 |
| Serveur d’appareil SBD | hso-hana-sbd | 10.0.0.19 |
| | | |
| Serveur NFS 1 | hso-nfs-vm-0 | 10.0.0.15 |
| Serveur NFS 2 | hso-nfs-vm-1 | 10.0.0.14 |



## <a name="multiple-subnets-and-vnics"></a>Plusieurs sous-réseaux et cartes réseau virtuelles

Suivant les recommandations de réseau SAP HANA, nous avons créé trois sous-réseaux dans un même réseau virtuel Azure. SAP HANA scale-out sur Azure doit être installé en mode non partagé. Cela signifie que chaque nœud utilise les volumes de disque locaux pour **/hana/data** et **/hana/log**. Étant donné que les nœuds utilisent seulement des volumes de disque locaux, nous n’avons pas besoin de définir un sous-réseau distinct pour le stockage :

- 10.0.2.0/24 pour la communication SAP HANA entre nœuds
- 10.0.1.0/24 pour la réplication système SAP HANA (HSR)
- 10.0.0.0/24 pour tout le reste

Pour plus d’informations sur la configuration SAP HANA en cas d’utilisation de plusieurs réseaux, consultez [SAP HANA global.ini](#sap-hana-globalini).

Chaque machine virtuelle du cluster a trois cartes réseau virtuelles qui correspondent au nombre de sous-réseaux. [Guide de création d’une machine virtuelle Linux dans Azure avec plusieurs cartes d’interface réseau][azure-linux-multiple-nics] décrit un problème de routage potentiel sur Azure lors du déploiement d’une machine virtuelle Linux. Cet article de routage spécifique s’applique uniquement à l’utilisation de plusieurs cartes réseau virtuelles. Le problème est résolu par défaut par SUSE dans SLES 12 SP3. Pour plus d’informations, consultez [Plusieurs cartes réseau avec cloud-netconfig dans EC2 et Azure][suse-cloud-netconfig].


Pour vérifier que SAP HANA est correctement configuré pour utiliser plusieurs réseaux, exécutez les commandes suivantes. Commencez par vérifier au niveau du système d’exploitation que les trois adresses IP internes des trois sous-réseaux sont actives. Si vous avez défini les sous-réseaux avec différentes plages d’adresses IP, vous devez adapter les commandes :

<pre><code>
ifconfig | grep "inet addr:10\."
</code></pre>

Voici un exemple de sortie du deuxième nœud worker sur le site 2. Vous pouvez voir trois différentes adresses IP internes eth0, eth1 eth2 :

<pre><code>
inet addr:10.0.0.42  Bcast:10.0.0.255  Mask:255.255.255.0
inet addr:10.0.1.42  Bcast:10.0.1.255  Mask:255.255.255.0
inet addr:10.0.2.42  Bcast:10.0.2.255  Mask:255.255.255.0
</code></pre>


Ensuite, vérifiez les ports SAP HANA pour le serveur de noms et HSR. SAP HANA doit écouter sur les sous-réseaux correspondants. Selon le nombre d’instances SAP HANA, vous devez adapter les commandes. Pour le système de test, le nombre d’instances est **00**. Plusieurs méthodes vous permettent de déterminer les ports utilisés. 

L’instruction SQL suivante retourne l’ID d’instance, le nombre d’instances et d’autres informations :

<pre><code>
select * from "SYS"."M_SYSTEM_OVERVIEW"
</code></pre>

Pour rechercher les numéros de port appropriés, vous pouvez effectuer la recherche, par exemple, dans HANA Studio sous **Configuration** ou à l’aide d’une instruction SQL :

<pre><code>
select * from M_INIFILE_CONTENTS WHERE KEY LIKE 'listen%'
</code></pre>

Pour trouver tous les ports qui sont utilisés dans la pile logicielle SAP, y compris SAP HANA, recherchez les [ports TCP/IP de tous les produits SAP][sap-list-port-numbers].

Comme le nombre d’instances est **00** dans le système de test SAP HANA 2.0, le numéro de port du serveur de noms est **30001**. Le numéro de port pour la communication des métadonnées HSR est **40002**. Vous pouvez vous connecter à un nœud worker et vérifiez les services du nœud master. Pour cet article, nous avons vérifié le nœud worker 2 sur le site 2 en essayant de nous connecter au nœud master sur le site 2.

Vérifiez le port du serveur de noms :

<pre><code>
nc -vz 10.0.0.40 30001
nc -vz 10.0.1.40 30001
nc -vz 10.0.2.40 30001
</code></pre>

Le résultat doit ressembler à l’exemple de sortie ci-dessous pour prouver que la communication entre nœuds utilise le sous-réseau **10.0.2.0/24**.
Seule la connexion via le sous-réseau **10.0.2.0/24** doit réussir :

<pre><code>
nc: connect to 10.0.0.40 port 30001 (tcp) failed: Connection refused
nc: connect to 10.0.1.40 port 30001 (tcp) failed: Connection refused
Connection to 10.0.2.40 30001 port [tcp/pago-services1] succeeded!
</code></pre>

Vérifiez maintenant pour le port HSR **40002** :

<pre><code>
nc -vz 10.0.0.40 40002
nc -vz 10.0.1.40 40002
nc -vz 10.0.2.40 40002
</code></pre>

Le résultat doit ressembler à l’exemple de sortie ci-dessous pour prouver que la communication HSR utilise le sous-réseau **10.0.1.0/24**.
Seule la connexion via le sous-réseau **10.0.1.0/24** doit réussir :

<pre><code>
nc: connect to 10.0.0.40 port 40002 (tcp) failed: Connection refused
Connection to 10.0.1.40 40002 port [tcp/*] succeeded!
nc: connect to 10.0.2.40 port 40002 (tcp) failed: Connection refused
</code></pre>



## <a name="corosync"></a>Corosync


Le fichier de configuration **corosync** doit être correct sur chaque nœud du cluster, y compris le nœud majority maker (créateur de majorité). Si la jonction de cluster d’un nœud ne fonctionne pas comme prévu, créez ou copiez **/etc/corosync/corosync.conf** manuellement sur tous les nœuds et redémarrez le service. 

Le contenu de **corosync.conf** du système de test est un exemple.

La première section est **totem** comme décrit dans [Installation du cluster](./high-availability-guide-suse-pacemaker.md#cluster-installation) (étape 11). Vous pouvez ignorer la valeur de **mcastaddr**. Gardez simplement l’entrée existante. Les entrées pour **token** et **consensus** doivent être définies en fonction de la [documentation de Microsoft Azure SAP HANA][sles-pacemaker-ha-guide].

<pre><code>
totem {
    version:    2
    secauth:    on
    crypto_hash:    sha1
    crypto_cipher:  aes256
    cluster_name:   hacluster
    clear_node_high_bit: yes

    token:      30000
    token_retransmits_before_loss_const: 10
    join:       60
    consensus:  36000
    max_messages:   20

    interface {
        ringnumber:     0
        bindnetaddr: 10.0.0.0
        mcastaddr:  239.170.19.232
        mcastport:  5405

        ttl:        1
    }
    transport:      udpu

}
</code></pre>

La deuxième section **logging** n’a pas été changée et conserve les valeurs par défaut :

<pre><code>
logging {
    fileline:   off
    to_stderr:  no
    to_logfile:     no
    logfile:    /var/log/cluster/corosync.log
    to_syslog:  yes
    debug:      off
    timestamp:  on
    logger_subsys {
        subsys:     QUORUM
        debug:  off
    }
}
</code></pre>

La troisième section montre la **nodelist** (liste de nœuds). Tous les nœuds du cluster doivent s’afficher avec leur **ID de nœud** :

<pre><code>
nodelist {
  node {
   ring0_addr:hso-hana-vm-s1-0
   nodeid: 1
   }
  node {
   ring0_addr:hso-hana-vm-s1-1
   nodeid: 2
   }
  node {
   ring0_addr:hso-hana-vm-s1-2
   nodeid: 3
   }
  node {
   ring0_addr:hso-hana-vm-s2-0
   nodeid: 4
   }
  node {
   ring0_addr:hso-hana-vm-s2-1
   nodeid: 5
   }
  node {
   ring0_addr:hso-hana-vm-s2-2
   nodeid: 6
   }
  node {
   ring0_addr:hso-hana-dm
   nodeid: 7
   }
}
</code></pre>

Dans la dernière section **quorum**, définissez correctement la valeur de **expected_votes**. Il doit s’agir du nombre de nœuds, y compris le nœud majority maker. La valeur de **two_node** doit être **0**. Ne supprimez pas complètement l’entrée. Définissez simplement la valeur sur **0**.

<pre><code>
quorum {
    # Enable and configure quorum subsystem (default: off)
    # see also corosync.conf.5 and votequorum.5
    provider: corosync_votequorum
    expected_votes: 7
    two_node: 0
}
</code></pre>


Redémarrez le service via **systemctl** :

<pre><code>
systemctl restart corosync
</code></pre>




## <a name="sbd-device"></a>Appareil SBD

La configuration d’un appareil SBD sur une machine virtuelle Azure est décrite dans [Isolation SBD](./high-availability-guide-suse-pacemaker.md#sbd-fencing).

Commencez par vérifier sur la machine virtuelle du serveur SBD s’il existe des entrées ACL pour chaque nœud du cluster. Exécutez la commande suivante sur la machine virtuelle du serveur SBD :


<pre><code>
targetcli ls
</code></pre>


Sur le système de test, la sortie de la commande ressemble à l’exemple ci-dessous. Les noms ACL comme **iqn.2006-04.hso-db-0.local:hso-db-0** doivent être entrés comme nom de l’initiateur correspondant sur les machines virtuelles. Chaque machine virtuelle doit en avoir un différent.

<pre><code>
 | | o- sbddbhso ................................................................... [/sbd/sbddbhso (50.0MiB) write-thru activated]
  | |   o- alua ................................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ....................................................................... [ALUA state: Active/optimized]
  | o- pscsi .................................................................................................. [Storage Objects: 0]
  | o- ramdisk ................................................................................................ [Storage Objects: 0]
  o- iscsi ............................................................................................................ [Targets: 1]
  | o- iqn.2006-04.dbhso.local:dbhso ..................................................................................... [TPGs: 1]
  |   o- tpg1 ............................................................................................... [no-gen-acls, no-auth]
  |     o- acls .......................................................................................................... [ACLs: 7]
  |     | o- iqn.2006-04.hso-db-0.local:hso-db-0 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-1.local:hso-db-1 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-2.local:hso-db-2 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-3.local:hso-db-3 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-4.local:hso-db-4 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-5.local:hso-db-5 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-6.local:hso-db-6 .................................................................. [Mapped LUNs: 1]
</code></pre>

Vérifiez ensuite que les noms d’initiateur sur toutes les machines virtuelles sont différents et correspondent aux entrées affichées précédemment. Voici un exemple du nœud worker 1 sur le site 1 :

<pre><code>
cat /etc/iscsi/initiatorname.iscsi
</code></pre>

La sortie ressemble à l’exemple suivant :

<pre><code>
##
## /etc/iscsi/iscsi.initiatorname
##
## Default iSCSI Initiatorname.
##
## DO NOT EDIT OR REMOVE THIS FILE!
## If you remove this file, the iSCSI daemon will not start.
## If you change the InitiatorName, existing access control lists
## may reject this initiator.  The InitiatorName must be unique
## for each iSCSI initiator.  Do NOT duplicate iSCSI InitiatorNames.
InitiatorName=iqn.2006-04.hso-db-1.local:hso-db-1
</code></pre>

Ensuite, vérifiez que la **découverte** fonctionne correctement. Exécutez la commande suivante sur chaque nœud de cluster à l’aide de l’adresse IP de la machine virtuelle du serveur SBD :

<pre><code>
iscsiadm -m discovery --type=st --portal=10.0.0.19:3260
</code></pre>

La sortie doit se présenter comme suit :

<pre><code>
10.0.0.19:3260,1 iqn.2006-04.dbhso.local:dbhso
</code></pre>

Le point de vérification suivant consiste à vérifier que le nœud voit l’appareil SDB. Vérifiez-le sur chaque nœud, y compris le nœud majority maker :

<pre><code>
lsscsi | grep dbhso
</code></pre>

La sortie doit se présenter comme suit. Toutefois, les noms peuvent différer. Le nom de l’appareil peut également changer après le redémarrage de la machine virtuelle :

<pre><code>
[6:0:0:0]    disk    LIO-ORG  sbddbhso         4.0   /dev/sdm
</code></pre>

En fonction de l’état du système, il est parfois utile de redémarrer les services iSCSI pour résoudre les problèmes. Exécutez ensuite les commande suivantes :

<pre><code>
systemctl restart iscsi
systemctl restart iscsid
</code></pre>


Vous pouvez vérifier si tous les nœuds sont **nettoyés** à partir de n’importe quel nœud. Faites attention à utiliser le nom d’appareil approprié sur un nœud spécifique :

<pre><code>
sbd -d /dev/sdm list
</code></pre>

La sortie doit indiquer **clear** pour chaque nœud du cluster :

<pre><code>
0       hso-hana-vm-s1-0        clear
1       hso-hana-vm-s2-2        clear
2       hso-hana-vm-s2-1        clear
3       hso-hana-dm     clear
4       hso-hana-vm-s1-1        clear
5       hso-hana-vm-s2-0        clear
6       hso-hana-vm-s1-2        clear
</code></pre>


Autre vérification SBD : l’option **dump** de la commande **sbd**. Dans cet exemple de commande et de sortie du nœud majority maker, le nom d’appareil était **sdd**, pas **sdm** :

<pre><code>
sbd -d /dev/sdd dump
</code></pre>

La sortie (hormis le nom d’appareil) doit être identique sur tous les nœuds :

<pre><code>
==Dumping header on disk /dev/sdd
Header version     : 2.1
UUID               : 9fa6cc49-c294-4f1e-9527-c973f4d5a5b0
Number of slots    : 255
Sector size        : 512
Timeout (watchdog) : 60
Timeout (allocate) : 2
Timeout (loop)     : 1
Timeout (msgwait)  : 120
==Header on disk /dev/sdd is dumped
</code></pre>

Vous pouvez aussi vérifier pour SBD la possibilité d’envoyer un message à un autre nœud. Pour envoyer un message au nœud worker 2 sur le site 2, exécutez la commande suivante sur le nœud worker 1 sur le site 2 :

<pre><code>
sbd -d /dev/sdm message hso-hana-vm-s2-2 test
</code></pre>

Du côté de la machine virtuelle cible, **hso-hana-vm-s2-2** dans cet exemple, vous trouvez l’entrée suivante dans **/var/log/messages** :

<pre><code>
/dev/disk/by-id/scsi-36001405e614138d4ec64da09e91aea68:   notice: servant: Received command test from hso-hana-vm-s2-1 on disk /dev/disk/by-id/scsi-36001405e614138d4ec64da09e91aea68
</code></pre>

Vérifiez que les entrées dans **/etc/sysconfig/sbd** correspondent à la description sous [Configuration de Pacemaker sur SUSE Linux Enterprise Server dans Azure](./high-availability-guide-suse-pacemaker.md#sbd-fencing). Vérifiez que le paramètre de démarrage **/etc/iscsi/iscsid.conf** est défini sur automatique.

Les entrées suivantes sont importantes dans **/etc/sysconfig/sbd**. Adaptez la valeur **id** si nécessaire :

<pre><code>
SBD_DEVICE="/dev/disk/by-id/scsi-36001405e614138d4ec64da09e91aea68;"
SBD_PACEMAKER=yes
SBD_STARTMODE=always
SBD_WATCHDOG=yes
</code></pre>


Vérifiez le paramètre de démarrage dans **/etc/iscsi/iscsid.conf**. Le paramètre nécessaire doit avoir été produit par la commande **iscsiadm** suivante, décrite dans la documentation. Vérifiez-le et adaptez-le manuellement avec **vi** s’il est différent.

Cette commande définit le comportement au démarrage :

<pre><code>
iscsiadm -m node --op=update --name=node.startup --value=automatic
</code></pre>

Effectuez cette entrée dans **/etc/iscsi/iscsid.conf** :

<pre><code>
node.startup = automatic
</code></pre>

Pendant les tests et la vérification, après le redémarrage d’une machine virtuelle, l’appareil SBD n’était plus visible dans certains cas. C’est le résultat d’une différence entre le paramètre de démarrage et la sortie de YaST2. Pour vérifier les paramètres, procédez comme suit :

1. Démarrez YaST2.
2. Sélectionnez **Network Services** (Services réseau) à gauche.
3. Faites défiler vers le bas à droite jusqu’à **iSCSI Initiator** (Initiateur iSCSI) et sélectionnez-le.
4. Dans l’écran suivant, sous l’onglet **Service**, vous devez voir le nom d’initiateur unique pour le nœud.
5. Au-dessus du nom d’initiateur, vérifiez que la valeur de **Service Start** (Démarrage du service) est définie sur **When Booting** (Au démarrage).
6. Si ce n’est pas le cas, définissez-la sur **When Booting** au lieu de **Manually** (Manuellement).
7. Basculez ensuite l’onglet supérieur sur **Connected Targets** (Cibles connectées).
8. Dans l’écran **Connected Targets**, vous devez voir une entrée pour l’appareil SBD, comme dans cet exemple : **10.0.0.19:3260 iqn.2006-04.dbhso.local:dbhso**.
9. Vérifiez si la **valeur de démarrage** est définie sur **on boot**.
10. Sinon, choisissez **Modifier** et modifiez-la.
11. Enregistrez les changements et quittez YaST2.



## <a name="pacemaker"></a>Pacemaker

Une fois que tout est correctement configuré, vous pouvez exécuter la commande suivante sur chaque nœud pour vérifier l’état du service Pacemaker :

<pre><code>
systemctl status pacemaker
</code></pre>

La partie supérieure de la sortie doit se présenter comme suit. Il est important que l’état après **Active** soit affiché comme **loaded** et **active (running)** . L’état après **Loaded** doit apparaître comme **enabled**.

<pre><code>
  pacemaker.service - Pacemaker High Availability Cluster Manager
   Loaded: loaded (/usr/lib/systemd/system/pacemaker.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2018-09-07 05:56:27 UTC; 4 days ago
     Docs: man:pacemakerd
           http://clusterlabs.org/doc/en-US/Pacemaker/1.1-pcs/html/Pacemaker_Explained/index.html
 Main PID: 4496 (pacemakerd)
    Tasks: 7 (limit: 4915)
   CGroup: /system.slice/pacemaker.service
           ├─4496 /usr/sbin/pacemakerd -f
           ├─4499 /usr/lib/pacemaker/cib
           ├─4500 /usr/lib/pacemaker/stonithd
           ├─4501 /usr/lib/pacemaker/lrmd
           ├─4502 /usr/lib/pacemaker/attrd
           ├─4503 /usr/lib/pacemaker/pengine
           └─4504 /usr/lib/pacemaker/crmd
</code></pre>

Si le paramètre est toujours défini sur **disabled**, exécutez la commande suivante :

<pre><code>
systemctl enable pacemaker
</code></pre>

Pour voir toutes les ressources configurées dans Pacemaker, exécutez la commande suivante :

<pre><code>
crm status
</code></pre>

La sortie doit se présenter comme suit. Il est normal que les ressources **cln** et **msl** soient affichées comme étant arrêtés sur la machine virtuelle majority maker (**hso-hana-dm**). Il n’y a aucune installation de SAP HANA sur le nœud majority maker. Par conséquent, les ressources **cln** et **msl** apparaissent comme étant arrêtées. Vous devez retrouver le nombre total correct de machines virtuelles (**7**). Toutes les machines virtuelles qui font partie du cluster doivent être listées avec l’état **Online**. Le nœud master principal actuel doit être reconnu correctement. Dans cet exemple, il s’agit de **hso-hana-vm-s1-0** :

<pre><code>
Stack: corosync
Current DC: hso-hana-dm (version 1.1.16-4.8-77ea74d) - partition with quorum
Last updated: Tue Sep 11 15:56:40 2018
Last change: Tue Sep 11 15:56:23 2018 by root via crm_attribute on hso-hana-vm-s1-0

7 nodes configured
17 resources configured

Online: [ hso-hana-dm hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started hso-hana-dm
 Clone Set: cln_SAPHanaTop_HSO_HDB00 [rsc_SAPHanaTop_HSO_HDB00]
     Started: [ hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]
     Stopped: [ hso-hana-dm ]
 Master/Slave Set: msl_SAPHanaCon_HSO_HDB00 [rsc_SAPHanaCon_HSO_HDB00]
     Masters: [ hso-hana-vm-s1-0 ]
     Slaves: [ hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]
     Stopped: [ hso-hana-dm ]
 Resource Group: g_ip_HSO_HDB00
     rsc_ip_HSO_HDB00   (ocf::heartbeat:IPaddr2):       Started hso-hana-vm-s1-0
     rsc_nc_HSO_HDB00   (ocf::heartbeat:anything):      Started hso-hana-vm-s1-0
</code></pre>

Une fonctionnalité importante de Pacemaker est le mode de maintenance. Dans ce mode, vous pouvez apporter des modifications sans déclencher une action immédiate du cluster. Un redémarrage de machine virtuelle constitue un exemple. Un cas d’utilisation classique serait la maintenance planifiée de l’infrastructure Azure ou d’un système d’exploitation. Consultez [Maintenance planifiée](#planned-maintenance). Utilisez la commande suivante pour placer Pacemaker en mode de maintenance :

<pre><code>
crm configure property maintenance-mode=true
</code></pre>

Quand vous effectuez une vérification avec **crm status**, vous constatez dans la sortie que toutes les ressources sont marquées comme étant **unmanaged** (non managées). Dans cet état, le cluster ne réagit pas aux changements, comme le démarrage ou l’arrêt de SAP HANA.
Voici un exemple de sortie de la commande **crm status** quand le cluster est en mode de maintenance :

<pre><code>
Stack: corosync
Current DC: hso-hana-dm (version 1.1.16-4.8-77ea74d) - partition with quorum
Last updated: Wed Sep 12 07:48:10 2018
Last change: Wed Sep 12 07:46:54 2018 by root via cibadmin on hso-hana-vm-s2-1

7 nodes configured
17 resources configured

              **_ Resource management is DISABLED _*_
  The cluster will not attempt to start, stop or recover services

Online: [ hso-hana-dm hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started hso-hana-dm (unmanaged)
 Clone Set: cln_SAPHanaTop_HSO_HDB00 [rsc_SAPHanaTop_HSO_HDB00] (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s1-1 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s1-0 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s1-2 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s2-1 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s2-2 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s2-0 (unmanaged)
     Stopped: [ hso-hana-dm ]
 Master/Slave Set: msl_SAPHanaCon_HSO_HDB00 [rsc_SAPHanaCon_HSO_HDB00] (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Slave hso-hana-vm-s1-1 (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Slave hso-hana-vm-s1-2 (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Slave hso-hana-vm-s2-1 (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Slave hso-hana-vm-s2-2 (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Master hso-hana-vm-s2-0 (unmanaged)
     Stopped: [ hso-hana-dm hso-hana-vm-s1-0 ]
 Resource Group: g_ip_HSO_HDB00
     rsc_ip_HSO_HDB00   (ocf::heartbeat:IPaddr2):       Started hso-hana-vm-s2-0 (unmanaged)
     rsc_nc_HSO_HDB00   (ocf::heartbeat:anything):      Started hso-hana-vm-s2-0 (unmanaged)
</code></pre>


Dans l’exemple de commande ci-dessous, vous voyez comment terminer le mode de maintenance du cluster :

<pre><code>
crm configure property maintenance-mode=false
</code></pre>


Une autre commande _ *crm** obtient la configuration complète du cluster dans un éditeur, afin que vous puissiez la modifier. Après avoir enregistré les changements, le cluster démarre les actions appropriées :

<pre><code>
crm configure edit
</code></pre>

Pour consulter la configuration de cluster complète, utilisez l’option **crm show** :

<pre><code>
crm configure show
</code></pre>



En cas d’échec des ressources de cluster, la commande **crm status** affiche une liste des **Failed Actions** (Actions ayant échoué). L’exemple de sortie suivante s’affiche :


<pre><code>
Stack: corosync
Current DC: hso-hana-dm (version 1.1.16-4.8-77ea74d) - partition with quorum
Last updated: Thu Sep 13 07:30:44 2018
Last change: Thu Sep 13 07:30:20 2018 by root via crm_attribute on hso-hana-vm-s1-0

7 nodes configured
17 resources configured

Online: [ hso-hana-dm hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started hso-hana-dm
 Clone Set: cln_SAPHanaTop_HSO_HDB00 [rsc_SAPHanaTop_HSO_HDB00]
     Started: [ hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]
     Stopped: [ hso-hana-dm ]
 Master/Slave Set: msl_SAPHanaCon_HSO_HDB00 [rsc_SAPHanaCon_HSO_HDB00]
     Masters: [ hso-hana-vm-s1-0 ]
     Slaves: [ hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]
     Stopped: [ hso-hana-dm hso-hana-vm-s2-0 ]
 Resource Group: g_ip_HSO_HDB00
     rsc_ip_HSO_HDB00   (ocf::heartbeat:IPaddr2):       Started hso-hana-vm-s1-0
     rsc_nc_HSO_HDB00   (ocf::heartbeat:anything):      Started hso-hana-vm-s1-0

Failed Actions:
* rsc_SAPHanaCon_HSO_HDB00_monitor_60000 on hso-hana-vm-s2-0 'unknown error' (1): call=86, status=complete, exitreason='none',
    last-rc-change='Wed Sep 12 17:01:28 2018', queued=0ms, exec=277663ms
</code></pre>

Vous devez nettoyer le cluster après des incidents. Utilisez à nouveau la commande **crm** avec l’option de commande **cleanup** pour effacer les entrées des actions ayant échoué. Nommez la ressource de cluster correspondante comme suit :

<pre><code>
crm resource cleanup rsc_SAPHanaCon_HSO_HDB00
</code></pre>

La sortie de la commande doit ressembler à l’exemple suivant :

<pre><code>
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-dm
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s1-0
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s1-1
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s1-2
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s2-0
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s2-1
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s2-2
Waiting for 7 replies from the CRMd....... OK
</code></pre>



## <a name="failover-or-takeover"></a>Basculement ou prise de contrôle

Comme mentionné précédemment dans les [Remarques importantes](#important-notes), vous ne devez pas utiliser d’arrêt approprié standard pour tester le basculement de cluster ou la prise de contrôle de SAP HANA HSR. Nous vous recommandons plutôt de déclencher une alerte de noyau, de forcer la migration d’une ressource ou d’arrêter tous les réseaux au niveau du système d’exploitation sur une machine virtuelle. Vous pouvez aussi utiliser la commande **crm \<node\> standby**. Consultez la [documentation SUSE][sles-12-ha-paper]. 

Ci-dessous, vous voyez trois exemples de commandes qui permettent de forcer le basculement de cluster :

<pre><code>
echo c &gt /proc/sysrq-trigger

crm resource migrate msl_SAPHanaCon_HSO_HDB00 hso-hana-vm-s2-0 force

wicked ifdown eth0
wicked ifdown eth1
wicked ifdown eth2
......
wicked ifdown eth&ltn&gt
</code></pre>

Comme décrit dans [Maintenance planifiée](#planned-maintenance), un bon moyen de superviser les activités de cluster est d’exécuter **SAPHanaSR-showAttr** avec la commande **watch** :

<pre><code>
watch SAPHanaSR-showAttr
</code></pre>

Par ailleurs, vous pouvez voir l’état du paysage SAP HANA à partir d’un script Python SAP. Cette valeur d’état est celle que recherche la configuration du cluster. C’est assez évident en cas d’échec d’un nœud worker. Si un nœud worker est en panne, SAP HANA ne retourne pas immédiatement une erreur d’intégrité pour l’ensemble du système scale-out. 

Un certain nombre de nouvelles tentatives sont effectuées pour éviter les basculements inutiles. C’est seulement si l’état passe de **Ok** (valeur de retour **4**) à **Erreur** (valeur de retour **1**) que le cluster réagit. Par conséquent, c’est correct si la sortie **SAPHanaSR-showAttr** montre une machine virtuelle avec l’état **offline** (hors connexion). Mais il n’existe encore aucune activité pour basculer entre principal et secondaire. Aucune activité de cluster n’est déclenchée tant que SAP HANA ne retourne pas d’erreur.

Vous pouvez superviser l’état d’intégrité du paysage SAP HANA en tant qu’utilisateur **\<HANA SID\>adm** en appelant le script Python SAP comme suit. Vous devrez peut-être adapter le chemin d’accès :

<pre><code>
watch python /hana/shared/HSO/exe/linuxx86_64/HDB_2.00.032.00.1533114046_eeaf4723ec52ed3935ae0dc9769c9411ed73fec5/python_support/landscapeHostConfiguration.py
</code></pre>

La sortie de cette commande doit ressembler à l’exemple ci-dessous. La colonne **Host Status** (État de l’hôte) et la colonne **overall host status** (état général de l’hôte) sont toutes deux importantes. La sortie réelle contient plus de colonnes.
Nous avons supprimé la plupart des colonnes à droite pour améliorer la lisibilité du tableau dans ce document :

<pre><code>
| Host             | Host   | Host   | Failover | Remove | 
|                  | Active | Status | Status   | Status | 
|                  |        |        |          |        | 
| ---------------- | ------ | ------ | -------- | ------ |    .......
| hso-hana-vm-s2-0 | yes    | ok     |          |        |        
| hso-hana-vm-s2-1 | yes    | ok     |          |        |         
| hso-hana-vm-s2-2 | yes    | ok     |          |        |        

overall host status: ok
</code></pre>


Une autre commande permet de vérifier les activités actuelles du cluster. L’exemple ci-dessous montre la commande et la fin de la sortie après que le nœud master du site principal a été tué. Vous pouvez voir la liste des actions de transition comme la **promotion** de l’ancien nœud master secondaire (**hso-hana-vm-s2-0**) en nouveau master principal. Si tout est correct et que toutes les activités sont terminées, cette liste **Transition Summary** (Récapitulatif des transactions) doit être vide.

<pre><code>
 crm_simulate -Ls

...........

Transition Summary:
 * Fence hso-hana-vm-s1-0
 * Stop    rsc_SAPHanaTop_HSO_HDB00:1   (hso-hana-vm-s1-0)
 * Demote  rsc_SAPHanaCon_HSO_HDB00:1   (Master -> Stopped hso-hana-vm-s1-0)
 * Promote rsc_SAPHanaCon_HSO_HDB00:5   (Slave -> Master hso-hana-vm-s2-0)
 * Move    rsc_ip_HSO_HDB00     (Started hso-hana-vm-s1-0 -> hso-hana-vm-s2-0)
 * Move    rsc_nc_HSO_HDB00     (Started hso-hana-vm-s1-0 -> hso-hana-vm-s2-0)
</code></pre>



## <a name="planned-maintenance"></a>Maintenance planifiée 

Différents cas d’utilisation doivent être examinés dans le cadre de la maintenance planifiée. Demandez-vous s’il s’agit simplement d’une maintenance d’infrastructure impliquant des changements au niveau du système d’exploitation et de la configuration de disque, ou s’il s’agit d’une mise à niveau de HANA.
Vous trouverez des informations supplémentaires dans les documents SUSE comme [Towards Zero Downtime][sles-zero-downtime-paper] ou [SAP HANA SR Performance Optimized Scenario][sles-12-for-sap]. Ces documents comprennent également des exemples de migration manuelle d’un site principal.

Nous avons effectué des tests internes intensifs pour vérifier le cas d’usage d’une maintenance d’infrastructure. Pour éviter des problèmes liés à la migration du site principal, nous avons décidé de toujours migrer un site principal avant de mettre un cluster en mode de maintenance. De cette façon, le cluster n’a plus besoin d’oublier l’ancienne situation : quel était le site principal et quel autre était le secondaire.

Deux situations sont possibles :

- **Maintenance planifiée sur le site secondaire actuel**. Dans ce cas, vous pouvez simplement mettre le cluster en mode de maintenance et effectuer le travail sur le site secondaire sans affecter le cluster.

- **Maintenance planifiée sur le site principal actuel**. Afin que les utilisateurs puissent continuer à travailler pendant la maintenance, vous devez forcer un basculement. Avec cette approche, vous devez déclencher le basculement de cluster avec Pacemaker et pas seulement au niveau de SAP HANA HSR. La configuration de Pacemaker déclenche automatiquement la prise de contrôle de SAP HANA. Vous devez aussi effectuer le basculement avant de placer le cluster en mode de maintenance.

La procédure de maintenance sur le site secondaire actuel est la suivante :

1. Mettez le cluster en mode de maintenance.
2. Effectuez le travail sur le site secondaire. 
3. Terminez le mode de maintenance du cluster.

La procédure de maintenance sur le site principal actuel est plus complexe :

1. Déclencher manuellement un basculement ou une prise de contrôle de SAP HANA avec une migration de ressources Pacemaker. Consultez les détails qui suivent.
2. SAP HANA sur l’ancien site principal est arrêté par la configuration de cluster.
3. Mettez le cluster en mode de maintenance.
4. Une fois le travail de maintenance terminé, inscrivez l’ancien site principal comme nouveau site secondaire.
5. Nettoyez la configuration du cluster. Consultez les détails qui suivent.
6. Terminez le mode de maintenance du cluster.


La migration d’une ressource ajoute une entrée à la configuration du cluster. Par exemple, un basculement forcé. Vous devez nettoyer ces entrées avant de terminer le mode de maintenance. Consultez l’exemple suivant.

La première étape est de forcer un basculement de cluster en migrant la ressource **msl** vers le nœud master secondaire actuel. Cette commande retourne un avertissement indiquant qu’une **contrainte de déplacement** a été créée :

<pre><code>
crm resource migrate msl_SAPHanaCon_HSO_HDB00 force

INFO: Move constraint created for msl_SAPHanaCon_HSO_HDB00
</code></pre>


Vérifiez le processus de basculement avec la commande **SAPHanaSR-showAttr**. Pour superviser l’état du cluster, ouvrez une fenêtre d’interpréteur de commandes dédiée et démarrez la commande avec **watch** :

<pre><code>
watch SAPHanaSR-showAttr
</code></pre>

La sortie doit refléter le basculement manuel. L’ancien nœud master secondaire a été **promu**, dans cet exemple, **hso-hana-vm-s2-0**. L’ancien site principal a été arrêté, valeur **lss** de **1** pour l’ancien nœud master principal **hso-hana-vm-s1-0** : 

<pre><code>
Global cib-time                 prim  sec srHook sync_state
------------------------------------------------------------
global Wed Sep 12 07:40:02 2018 HSOS2 -   SFAIL  SFAIL


Sites lpt        lss mns              srr
------------------------------------------
HSOS1 10         1   hso-hana-vm-s1-0 P
HSOS2 1536738002 4   hso-hana-vm-s2-0 P


Hosts            clone_state node_state roles                        score  site
----------------------------------------------------------------------------------
hso-hana-dm                  online
hso-hana-vm-s1-0 UNDEFINED   online     master1::worker:             150    HSOS1
hso-hana-vm-s1-1 DEMOTED     online     slave::worker:               -10000 HSOS1
hso-hana-vm-s1-2 DEMOTED     online     slave::worker:               -10000 HSOS1
hso-hana-vm-s2-0 PROMOTED    online     master1:master:worker:master 150    HSOS2
hso-hana-vm-s2-1 DEMOTED     online     slave:slave:worker:slave     -10000 HSOS2
hso-hana-vm-s2-2 DEMOTED     online     slave:slave:worker:slave     -10000 HSOS2
</code></pre>

Après le basculement de cluster et la prise de contrôle de SAP HANA, mettez le cluster en mode de maintenance, comme décrit dans [Pacemaker](#pacemaker).

Les commandes **SAPHanaSR-showAttr** et **crm status** n’indiquent rien sur les contraintes créées par la migration de ressources. Pour que ces contraintes soient visibles, affichez la configuration de ressources de cluster complète avec la commande suivante :

<pre><code>
crm configure show
</code></pre>

Dans la configuration de cluster, vous avez une nouvelle contrainte d’emplacement liée à l’ancienne migration de ressources manuelle. Cet exemple d’entrée commence par **location cli-** :

<pre><code>
location cli-ban-msl_SAPHanaCon_HSO_HDB00-on-hso-hana-vm-s1-0 msl_SAPHanaCon_HSO_HDB00 role=Started -inf: hso-hana-vm-s1-0
</code></pre>

Malheureusement ces contraintes peuvent avoir un impact sur le comportement général du cluster. Par conséquent, vous devez obligatoirement les supprimer à nouveau avant de remettre en service l’ensemble du système. Avec la commande **unmigrate**, vous pouvez nettoyer les contraintes d’emplacement créées précédemment. Le terme peut être un peu déroutant. La commande ne tente pas de migrer à nouveau la ressource vers sa machine virtuelle d’origine. Elle supprime simplement les contraintes d’emplacement et retourne les informations correspondantes lorsque vous exécutez la commande :


<pre><code>
crm resource unmigrate msl_SAPHanaCon_HSO_HDB00

INFO: Removed migration constraints for msl_SAPHanaCon_HSO_HDB00
</code></pre>

À la fin du travail de maintenance, vous arrêtez le mode maintenance du cluster comme indiqué dans [Pacemaker](#pacemaker).



## <a name="hb_report-to-collect-log-files"></a>hb_report pour collecter les fichiers journaux

Pour analyser les problèmes du cluster Pacemaker, le support de SUSE demande d’exécuter l’utilitaire **hb_report**. Il collecte tous les fichiers journaux importants dont vous avez besoin pour analyser ce qui est arrivé. Cet exemple d’appel utilise une heure de début et de fin où un incident spécifique s’est produit. Consultez aussi [Remarques importantes](#important-notes) :

<pre><code>
hb_report -f "2018/09/13 07:36" -t "2018/09/13 08:00" /tmp/hb_report_log
</code></pre>

La commande vous indique où elle a placé les fichiers journaux compressés :

<pre><code>
The report is saved in /tmp/hb_report_log.tar.bz2
Report timespan: 09/13/18 07:36:00 - 09/13/18 08:00:00
</code></pre>

Vous pouvez ensuite extraire les fichiers individuels avec la commande standard **tar** :

<pre><code>
tar -xvf hb_report_log.tar.bz2
</code></pre>

Les fichiers extraits comprennent tous les fichiers journaux. La plupart d’entre eux ont été placés dans des répertoires distincts pour chaque nœud du cluster :

<pre><code>
-rw-r--r-- 1 root root  13655 Sep 13 09:01 analysis.txt
-rw-r--r-- 1 root root  14422 Sep 13 09:01 description.txt
-rw-r--r-- 1 root root      0 Sep 13 09:01 events.txt
-rw-r--r-- 1 root root 275560 Sep 13 09:00 ha-log.txt
-rw-r--r-- 1 root root     26 Sep 13 09:00 ha-log.txt.info
drwxr-xr-x 4 root root   4096 Sep 13 09:01 hso-hana-dm
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s1-0
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s1-1
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s1-2
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s2-0
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s2-1
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s2-2
-rw-r--r-- 1 root root 264726 Sep 13 09:00 journal.log
</code></pre>


Dans l’intervalle de temps spécifié, le nœud master actuel **hso-hana-vm-s1-0** a été tué. Dans **journal.log** se trouvent les entrées relatives à cet événement :

<pre><code>
2018-09-13T07:38:01+0000 hso-hana-vm-s2-1 su[93494]: (to hsoadm) root on none
2018-09-13T07:38:01+0000 hso-hana-vm-s2-1 su[93494]: pam_unix(su-l:session): session opened for user hsoadm by (uid=0)
2018-09-13T07:38:01+0000 hso-hana-vm-s2-1 systemd[1]: Started Session c44290 of user hsoadm.
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 corosync[28302]:   [TOTEM ] A new membership (10.0.0.13:120996) was formed. Members left: 1
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 corosync[28302]:   [TOTEM ] Failed to receive the leave message. failed: 1
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 attrd[28313]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 attrd[28313]:   notice: Removing all hso-hana-vm-s1-0 attributes for peer loss
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 attrd[28313]:   notice: Purged 1 peer with id=1 and/or uname=hso-hana-vm-s1-0 from the membership cache
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 stonith-ng[28311]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 stonith-ng[28311]:   notice: Purged 1 peer with id=1 and/or uname=hso-hana-vm-s1-0 from the membership cache
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 cib[28310]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 corosync[28302]:   [QUORUM] Members[6]: 7 2 3 4 5 6
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 corosync[28302]:   [MAIN  ] Completed service synchronization, ready to provide service.
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 crmd[28315]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 pacemakerd[28308]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 cib[28310]:   notice: Purged 1 peer with id=1 and/or uname=hso-hana-vm-s1-0 from the membership cache
2018-09-13T07:38:03+0000 hso-hana-vm-s2-1 su[93494]: pam_unix(su-l:session): session closed for user hsoadm
</code></pre>

Autre exemple : le fichier journal Pacemaker sur le nœud master secondaire (devenu le nouveau nœud master principal). Cet extrait montre que l’état du nœud master principal tué a été défini sur **offline** :

<pre><code>
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 3 still member of group stonith-ng (peer=hso-hana-vm-s1-2, counter=5.1)
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 4 still member of group stonith-ng (peer=hso-hana-vm-s2-0, counter=5.2)
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 5 still member of group stonith-ng (peer=hso-hana-vm-s2-1, counter=5.3)
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 6 still member of group stonith-ng (peer=hso-hana-vm-s2-2, counter=5.4)
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 7 still member of group stonith-ng (peer=hso-hana-dm, counter=5.5)
Sep 13 07:38:02 [4184] hso-hana-vm-s2-0       crmd:     info: pcmk_cpg_membership:      Node 1 left group crmd (peer=hso-hana-vm-s1-0, counter=5.0)
Sep 13 07:38:02 [4184] hso-hana-vm-s2-0       crmd:     info: crm_update_peer_proc:     pcmk_cpg_membership: Node hso-hana-vm-s1-0[1] - corosync-cpg is now offline
Sep 13 07:38:02 [4184] hso-hana-vm-s2-0       crmd:     info: peer_update_callback:     Client hso-hana-vm-s1-0/peer now has status [offline] (DC=hso-hana-dm, changed=4000000)
Sep 13 07:38:02 [4184] hso-hana-vm-s2-0       crmd:     info: pcmk_cpg_membership:      Node 2 still member of group crmd (peer=hso-hana-vm-s1-1, counter=5.0)
</code></pre>





## <a name="sap-hana-globalini"></a>SAP HANA global.ini


Les extraits suivants proviennent du fichier SAP HANA **global.ini** sur le site de cluster 2. Cet exemple montre les entrées de résolution du nom d’hôte pour l’utilisation de différents réseaux pour la communication entre les nœuds de SAP HANA et HSR :

<pre><code>
[communication]
tcp_keepalive_interval = 20
internal_network = 10.0.2/24
listeninterface = .internal
</code></pre>


<pre><code>
[internal_hostname_resolution]
10.0.2.40 = hso-hana-vm-s2-0
10.0.2.42 = hso-hana-vm-s2-2
10.0.2.41 = hso-hana-vm-s2-1
</code></pre>

<pre><code>
[ha_dr_provider_SAPHanaSR]
provider = SAPHanaSR
path = /hana/shared/myHooks
execution_order = 1
</code></pre>

<pre><code>
[system_replication_communication]
listeninterface = .internal

[system_replication_hostname_resolution]
10.0.1.30 = hso-hana-vm-s1-0
10.0.1.31 = hso-hana-vm-s1-1
10.0.1.32 = hso-hana-vm-s1-2
10.0.1.40 = hso-hana-vm-s2-0
10.0.1.41 = hso-hana-vm-s2-1
10.0.1.42 = hso-hana-vm-s2-2
</code></pre>



## <a name="hawk"></a>Hawk

La solution de cluster fournit une interface de navigateur avec une interface graphique utilisateur pour les personnes qui préfèrent des menus et des graphiques à la place de toutes les commandes de l’interpréteur de commandes.
Pour utiliser l’interface de navigateur, remplacez **\<node\>** par un nœud SAP HANA réel dans l’URL suivante. Puis entrez les informations d’identification du cluster (utilisateur **cluster**) :

<pre><code>
https://&ltnode&gt:7630
</code></pre>

Cette capture d’écran montre le tableau de bord du cluster :


![Tableau de bord du cluster dans Hawk](media/hana-vm-scale-out-HA-troubleshooting/hawk-1.png)


Cet exemple illustre les contraintes d’emplacement causées par une migration de ressources de cluster, comme expliqué dans [Maintenance planifiée](#planned-maintenance) :


![Contraintes de liste Hawk](media/hana-vm-scale-out-HA-troubleshooting/hawk-2.png)


Vous pouvez également charger la sortie **hb_report** dans Hawk sous **Historique**, illustrée comme suit. Consultez hb_report pour collecter les fichiers journaux : 

![Sortie hb_report chargée dans Hawk](media/hana-vm-scale-out-HA-troubleshooting/hawk-3.png)

Avec **History Explorer** (l’Explorateur d’historique), vous pouvez ensuite parcourir toutes les transitions de cluster incluses dans la sortie **hb_report** :

![Transitions Hawk dans la sortie hb_report](media/hana-vm-scale-out-HA-troubleshooting/hawk-4.png)

Cette capture d’écran finale montre la section **Détails** d’une transition unique. Le cluster a réagi sur une panne de nœud master principal, nœud **hso-hana-vm-s1-0**. Il promeut maintenant le nœud secondaire comme nouveau master, **hso-hana-vm-s2-0** :

![Transition unique Hawk](media/hana-vm-scale-out-HA-troubleshooting/hawk-5.png)


## <a name="next-steps"></a>Étapes suivantes

Ce guide de dépannage décrit la haute disponibilité de SAP HANA dans une configuration scale-out. En plus de la base de données, la pile SAP NetWeaver est un autre composant important du paysage SAP. Découvrez la [haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure qui utilisent SUSE Enterprise Linux Server][sap-nw-ha-guide-sles].
