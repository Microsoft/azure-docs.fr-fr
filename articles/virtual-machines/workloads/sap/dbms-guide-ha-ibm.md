---
title: Configurer la haute disponibilité et reprise d’activité IBM Db2 sur les machines virtuelles Azure | Microsoft Docs
description: Créez une haute disponibilité d’IBM Db2 LUW sur les machines virtuelles Azure.
author: msjuergent
ms.service: virtual-machines
ms.subservice: workloads
ms.topic: article
ms.date: 10/16/2020
ms.author: juergent
ms.reviewer: cynthn
ms.openlocfilehash: 54bde8c9dd47e88ffdc831ccb9f7833720583238
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96621380"
---
# <a name="high-availability-of-ibm-db2-luw-on-azure-vms-on-suse-linux-enterprise-server-with-pacemaker"></a>Haute disponibilité d’IBM Db2 LUW sur les machines virtuelles Azure sur SUSE Linux Enterprise Server avec Pacemaker

IBM Db2 pour Linux, UNIX et Windows (LUW) dans une [configuration de haute disponibilité et reprise d’activité (HADR)](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_10.5.0/com.ibm.db2.luw.admin.ha.doc/doc/c0011267.html) est constitué d’un nœud qui exécute une instance de base de données primaire et d’au moins un nœud qui exécute une instance de base de données secondaire. Les modifications apportées à l’instance de base de données primaire sont répliquées vers une instance de base de données secondaire de façon synchrone ou asynchrone, selon votre configuration. 

> [!NOTE]
> Cet article contient des références aux termes *maître* et *esclave*, termes que Microsoft n’utilise plus. Lorsque ces termes seront supprimés du logiciel, nous les supprimerons de cet article.
   
Cet article décrit comment déployer et configurer les machines virtuelles Azure, installer le framework de cluster, et installer IBM Db2 LUW avec la configuration HADR. 

L’article n’explique pas comment installer et configurer IBM Db2 LUW avec l’installation des logiciels SAP ou HADR. Pour vous aider à accomplir ces tâches, nous fournissons des références aux manuels d’installation SAP et IBM. Cet article se concentre sur des composants qui sont spécifiques à l’environnement Azure. 

Les versions IBM Db2 prises en charge sont 10.5 et ultérieures, comme décrit dans la note SAP [1928533].

Avant de commencer une installation, consultez les notes SAP suivantes et la documentation :

| Note SAP | Description |
| --- | --- |
| [1928533] | Applications SAP sur Azure : Produits et types de machines virtuelles pris en charge |
| [2015553] | SAP sur Azure : Prérequis pour la prise en charge |
| [2178632] | Métriques de supervision clés pour SAP sur Azure |
| [2191498] | SAP sur Linux avec Azure : Surveillance améliorée |
| [2243692] | Linux sur une machine virtuelle Azure (IaaS) : problèmes de licence SAP |
| [1984787] | SUSE LINUX Enterprise Server 12 Notes d'installation |
| [1999351] | Résolution des problèmes de la surveillance Azure améliorée pour SAP |
| [2233094] | DB6 : applications SAP sur Azure qui utilisent IBM Db2 pour Linux, UNIX et Windows - informations supplémentaires |
| [1612105] | DB6 : forum aux questions sur Db2 avec HADR |


| Documentation | 
| --- |
| [Wiki de la communauté SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) : contient toutes les notes SAP requises pour Linux |
| Guide [Planification et implémentation de machines virtuelles Azure pour SAP sur Linux][planning-guide] |
| [Déploiement de machines virtuelles Azure pour SAP sur Linux][deployment-guide] (le présent article) |
| Guide [Déploiement du système de gestion de base de données (SGBD) des machines virtuelles Azure pour SAP sur Linux][dbms-guide] |
| [Check-list relative à la planification et au déploiement de la charge de travail SAP sur Azure][azr-sap-plancheck] |
| [Guides des meilleures pratiques de SUSE Linux Enterprise Server pour applications SAP 12 SP4][sles-for-sap-bp] |
| [Extension haute disponibilité SUSE Linux Enterprise 12 SP4][sles-ha-guide] |
| [Déploiement SGBD de machines virtuelles Azure IBM Db2 pour charge de travail SAP][dbms-db2] |
| [IBM Db2 HADR 11.1][db2-hadr-11.1] |
| [IBM Db2 HADR R 10.5][db2-hadr-10.5] |

## <a name="overview"></a>Vue d’ensemble
Pour obtenir une haute disponibilité, IBM Db2 LUW avec HADR est installé sur au moins deux machines virtuelles Azure, qui sont déployées sur un [groupe à haute disponibilité Azure](../../windows/tutorial-availability-sets.md) ou des [zones de disponibilité Azure](./sap-ha-availability-zones.md). 

Les graphiques suivants illustrent une configuration de deux machines virtuelles Azure de serveur de base de données. Les deux machines virtuelles Azure de serveur de base de données sont associées à leur propre système de stockage et sont opérationnelles. Dans HADR, une instance de base de données dans l’une des machines virtuelles Azure a le rôle de l’instance primaire. Tous les clients sont connectés à cette instance primaire. Toutes les modifications dans les transactions de base de données sont conservées localement dans le journal des transactions Db2. Comme les enregistrements du journal des transactions sont conservés localement, les enregistrements sont transférés via TCP/IP à l’instance de base de données sur le second serveur de base de données, le serveur de secours ou une instance de secours. L’instance de secours met à jour la base de données locale en restaurant par progression les enregistrements du journal des transactions transférés. De cette façon, le serveur de secours est synchronisé avec le serveur principal.

HADR est uniquement une fonctionnalité de réplication. Elle n’assure pas la détection des défaillances et n’a aucune fonction de prise de contrôle ou basculement automatique. Une prise de contrôle ou un transfert vers le serveur de secours doit être lancé manuellement par un administrateur de base de données. Pour obtenir une prise de contrôle automatique et une détection des défaillances, vous pouvez utiliser la fonctionnalité de clustering Linux Pacemaker. Pacemaker supervise les deux instances de serveur de base de données. Lorsque l’instance de serveur de base de données primaire plante, Pacemaker lance une prise de contrôle HADR *automatique* par le serveur de secours. Pacemaker garantit également que l’adresse IP virtuelle est affectée au nouveau serveur principal.

![Vue d’ensemble de la haute disponibilité d’IBM Db2](./media/dbms-guide-ha-ibm/ha-db2-hadr-lb.png)

Pour que les serveurs d’applications SAP se connectent à la base de données primaire, vous avez besoin d’un nom d’hôte virtuel et d’une adresse IP virtuelle. En cas de basculement, les serveurs d’applications SAP se connecteront à la nouvelle instance de base de données primaire. Dans un environnement Azure, un [équilibreur de charge Azure](https://microsoft.sharepoint.com/teams/WAG/AzureNetworking/Wiki/Load%20Balancing.aspx) est nécessaire pour utiliser une adresse IP virtuelle de la façon requise pour HADR d’IBM Db2. 

Pour vous aider à mieux comprendre comment IBM Db2 LUW avec HADR et Pacemaker s’adapte à une configuration de système SAP hautement disponible, l’image suivante présente une vue d’ensemble d’une configuration hautement disponible d’un système SAP basé sur une base de données IBM Db2. Cet article couvre uniquement IBM Db2, mais il fournit des références à d’autres articles sur la façon de configurer d’autres composants d’un système SAP.

![Vue d’ensemble de l’environnement complet de la haute disponibilité d’IBM Db2](.//media/dbms-guide-ha-ibm/end-2-end-ha.png)


### <a name="high-level-overview-of-the-required-steps"></a>Vue d’ensemble globale des étapes à suivre
Pour déployer une configuration IBM Db2, vous devez procéder comme suit :

  + Planifiez votre environnement.
  + Déployez les machines virtuelles.
  + Mettez à jour SUSE Linux et configurez des systèmes de fichiers.
  + Installez et configurez Pacemaker.
  + Installez un [système NFS à haute disponibilité][nfs-ha].
  + Installez [ASCS/ERS sur un cluster séparé][ascs-ha].
  + Installez la base de données IBM Db2 avec l’option Distributed/High Availability (Distribué/Haute disponibilité) (SWPM).
  + Installez et créez une instance et un nœud de base de données secondaire, et configurez HADR.
  + Vérifiez que HADR fonctionne.
  + Appliquez la configuration Pacemaker pour contrôler IBM Db2.
  + Configurez Azure Load Balancer.
  + Installez des serveurs d’applications principal et de boîte de dialogue.
  + Vérifiez et adaptez la configuration des serveurs d’applications SAP.
  + Effectuez des tests de basculement et de prise de contrôle.



## <a name="plan-azure-infrastructure-for-hosting-ibm-db2-luw-with-hadr"></a>Planifier l’infrastructure Azure pour l’hébergement d’IBM Db2 LUW avec HADR

Terminez le processus de planification avant d’exécuter le déploiement. La planification génère la base pour le déploiement d’une configuration de Db2 avec HADR dans Azure. Les éléments clés qui doivent faire partie de la planification d’IMB Db2 LUW (partie base de données de l’environnement SAP) sont listés dans le tableau suivant :

| Rubrique | Description courte |
| --- | --- |
| Définir des groupes de ressources Azure | Groupes de ressources où vous déployez la machine virtuelle, le réseau virtuel, Azure Load Balancer et d’autres ressources. Ils peuvent être nouveaux ou existants. |
| Définition de sous-réseau/réseau virtuel | Endroit où des machines virtuelles pour IBM Db2 et Azure Load Balancer sont déployées. Elles peuvent être existantes ou récemment créées. |
| Machines virtuelles hébergeant IBM Db2 LUW | Taille de machine virtuelle, stockage, mise en réseau, adresse IP. |
| Nom d’hôte virtuel et adresse IP virtuelle pour la base de données IBM Db2| Adresse IP virtuelle ou nom d’hôte utilisé pour la connexion des serveurs d’applications SAP. **db-virt-hostname**, **db-virt-ip**. |
| Délimitation Azure | Délimitation Azure ou isolation SBD (hautement recommandée). Méthode permettant d’éviter les situations de Split-Brain. |
| Machine virtuelle SBD | Taille de la machine virtuelle SBD, stockage, réseau. |
| Azure Load Balancer | Utilisation du niveau De base ou Standard (recommandé), port de la sonde pour la base de données Db2 (notre recommandation est 62500) **probe-port**. |
| Résolution de noms| Fonctionnement de la résolution de noms dans l’environnement. Le service DNS est fortement recommandé. Le fichier d’hôte local peut être utilisé. |
    
Pour plus d’informations sur Linux Pacemaker dans Azure, consultez [Configurer Pacemaker sur SUSE Linux Enterprise Server dans Azure](./high-availability-guide-suse-pacemaker.md).

## <a name="deployment-on-suse-linux"></a>Déploiement sur SUSE Linux

L’agent de ressource pour IBM Db2 LUW est inclus dans SUSE Linux Enterprise Server pour applications SAP. Pour la configuration qui est décrite dans ce document, vous devez utiliser SUSE Linux Server pour applications SAP. La Place de marché Azure contient une image de SUSE Enterprise Server pour applications SAP 12 que vous pouvez utiliser pour déployer de nouvelles machines virtuelles Azure. Soyez conscient des différents modèles de prise en charge ou service proposés par SUSE via la Place de marché Azure quand vous choisissez une image de machine virtuelle dans la Place de marché de machines virtuelles Azure. 

### <a name="hosts-dns-updates"></a>Hôtes : Mises à jour de DNS
Dressez la liste de tous les noms d’hôte, y compris les noms des hôtes virtuels, et mettez à jour vos serveurs DNS afin d’activer une adresse IP appropriée pour la résolution de nom d’hôte. S’il n’existe aucun serveur DNS ou que vous ne pouvez pas mettre à jour et créer des entrées DNS, vous devez utiliser les fichiers d’hôte local des machines virtuelles individuelles qui font partie de ce scénario. Si vous utilisez des entrées des fichiers d’hôte, assurez-vous que les entrées sont appliquées à toutes les machines virtuelles dans l’environnement du système SAP. Toutefois, nous vous recommandons d’utiliser votre serveur DNS qui, dans l’idéal, s’étend à Azure


### <a name="manual-deployment"></a>Déploiement manuel

Vérifiez que le système d’exploitation sélectionné est pris en charge par IBM/SAP pour IBM Db2 LUW. La liste des versions de système d’exploitation prises en charge pour les machines virtuelles Azure et les versions Db2 est disponible dans la note SAP [1928533]. La liste des versions de système d’exploitation par version Db2 individuelle est disponible dans le tableau de disponibilité des produits SAP. Nous recommandons vivement SLES 12 SP4 au minimum en raison d’améliorations des performances liées à Azure dans cette version ou une version SUSE Linux ultérieure.

1. Créez ou sélectionnez un groupe de ressources.
1. Créez ou sélectionnez un réseau virtuel et un sous-réseau.
1. Créez un groupe à haute disponibilité Azure ou déployez une zone de disponibilité.
    + Pour le groupe à haute disponibilité, affectez 2 au nombre maximal de domaines de mise à jour.
1. Créez la machine virtuelle 1.
    + Utilisez l’image SLES pour SAP dans la Place de marché Azure.
    + Sélectionnez le groupe à haute disponibilité Azure créé à l’étape 3, ou une zone de disponibilité.
1.  Créez la machine virtuelle 2.
    + Utilisez l’image SLES pour SAP dans la Place de marché Azure.
    + Sélectionnez le groupe à haute disponibilité Azure créé à l’étape 3, ou une zone de disponibilité (différente de celle de l’étape 3).
1. Ajoutez des disques de données aux machines virtuelles et vérifiez ensuite la suggestion sur la configuration d’un système de fichiers dans l’article [Déploiement SGBD de machines virtuelles Azure IBM Db2 pour charge de travail SAP][dbms-db2].

## <a name="create-the-pacemaker-cluster"></a>Créez le cluster Pacemaker
    
Pour créer un cluster Pacemaker de base pour ce serveur IBM Db2, consultez [Configurer Pacemaker sur SUSE Linux Enterprise Server dans Azure][sles-pacemaker]. 

## <a name="install-the-ibm-db2-luw-and-sap-environment"></a>Installer l’environnement SAP et IBM Db2 LUW

Avant de commencer l’installation d’un environnement SAP basé sur IBM Db2 LUW, passez en revue la documentation suivante :

+ Documentation Azure
+ Documentation SAP
+ Documentation IBM

Des liens vers cette documentation sont fournis dans la section d’introduction de cet article.

Consultez les guides d’installation SAP sur l’installation d’applications basées sur NetWeaver sur IBM Db2 LUW.

Vous pouvez trouver les guides dans le portail d’aide SAP à l’aide de [SAP Installation Guide Finder][sap-instfind].

Vous pouvez réduire le nombre de guides affichés dans le portail en définissant les filtres suivants :

- Je souhaite : « Installer un nouveau système »
- Ma base de données : « IBM Db2 pour Linux, UNIX et Windows »
- Filtres supplémentaires pour les versions SAP NetWeaver, la configuration de pile ou le système d’exploitation

### <a name="installation-hints-for-setting-up-ibm-db2-luw-with-hadr"></a>Conseils d’installation pour la configuration d’IBM Db2 LUW avec HADR

Pour configurer l’instance de base de données IBM Db2 LUW primaire :

- Utilisez l’option Distributed ou High Availability (Distribué ou Haute disponibilité).
- Installez SAP ASCS/ERS et l’instance de base de données.
- Effectuez une sauvegarde de la base de données nouvellement installée.


> [!IMPORTANT] 
> Notez le « port de communication de la base de données » qui est défini lors de l’installation. Ce doit être le même numéro de port pour les deux instances de base de données

Pour configurer le serveur de base de données de secours à l’aide de la procédure de copie de système homogène SAP, exécutez ces étapes :

1. Sélectionnez l’option **System copy** > **Target systems** > **Distributed** > **Database instance** (Copie du système > Systèmes cibles > Distribué > Instance de base de données.
1. Comme méthode de copie, sélectionnez **Homogeneous System** (Système homogène) afin que vous puissiez utiliser la copie de sauvegarde pour restaurer une sauvegarde sur l’instance de serveur de secours.
1. Lorsque vous atteignez la dernière étape pour restaurer la base de données pour la copie de système homogène, quittez le programme d’installation. Restaurez la base de données à partir d’une sauvegarde de l’hôte principal. Toutes les phases d’installation suivantes ont déjà été exécutées sur le serveur de base de données principal.
1. Configurez HADR pour IBM Db2.

   > [!NOTE]
   > Pour une installation et une configuration spécifiques à Azure et Pacemaker : Au cours de la procédure d’installation via SAP Software Provisioning Manager, une question explicite se pose sur la haute disponibilité pour IBM Db2 LUW :
   >+ Ne sélectionnez pas **IBM Db2 pureScale**.
   >+ Ne sélectionnez pas **Install IBM Tivoli System Automation for Multiplatforms** (Installer IBM Tivoli System Automation pour plusieurs plateformes).
   >+ Ne sélectionnez pas **Generate cluster configuration files** (Générer des fichiers de configuration de cluster).

   Lorsque vous utilisez un appareil SBD pour Linux Pacemaker, définissez les paramètres HADR Db2 suivants :
   + HADR peer window duration (seconds) (HADR_PEER_WINDOW) = 300  
   + HADR timeout value (HADR_TIMEOUT) = 60

   Lorsque vous utilisez un agent de délimitation Azure Pacemaker, définissez les paramètres suivants :
   + HADR peer window duration (seconds) (HADR_PEER_WINDOW) = 900  
   + HADR timeout value (HADR_TIMEOUT) = 60

Nous vous recommandons les paramètres précédents d’après les tests de basculement/prise de contrôle initiaux. Il est obligatoire que vous testiez le fonctionnement correct du basculement et de la prise de contrôle avec ces paramètres. Étant donné que les configurations individuelles peuvent varier, les paramètres peuvent nécessiter un ajustement. 

> [!IMPORTANT]
> Propre à IBM Db2 avec une configuration HADR et un démarrage normal : l’instance de base de données secondaire ou de secours doit être opérationnelle avant de pouvoir démarrer l’instance de base de données primaire.

À des fins de démonstration et pour les procédures décrites dans cet article, l’ID de sécurité de la base de données est **PTR**.

#### <a name="ibm-db2-hadr-check"></a>Vérification HADR IBM Db2
Une fois que vous avez configuré HADR et que l’état est PEER et CONNECTED sur les nœuds principal et de secours, effectuez la vérification suivante :

<pre><code>
Execute command as db2&lt;sid&gt; db2pd -hadr -db &lt;SID&gt;

#Primary output:
# Database Member 0 -- Database PTR -- Active -- Up 1 days 01:51:38 -- Date 2019-02-06-15.35.28.505451
# 
#                             <b>HADR_ROLE = PRIMARY
#                           REPLAY_TYPE = PHYSICAL
#                         HADR_SYNCMODE = NEARSYNC
#                            STANDBY_ID = 1
#                         LOG_STREAM_ID = 0
#                            HADR_STATE = PEER
#                            HADR_FLAGS = TCP_PROTOCOL
#                   PRIMARY_MEMBER_HOST = azibmdb02
#                      PRIMARY_INSTANCE = db2ptr
#                        PRIMARY_MEMBER = 0
#                   STANDBY_MEMBER_HOST = azibmdb01
#                      STANDBY_INSTANCE = db2ptr
#                        STANDBY_MEMBER = 0
#                   HADR_CONNECT_STATUS = CONNECTED</b>
#              HADR_CONNECT_STATUS_TIME = 02/05/2019 13:51:47.170561 (1549374707)
#           HEARTBEAT_INTERVAL(seconds) = 15
#                      HEARTBEAT_MISSED = 0
#                    HEARTBEAT_EXPECTED = 6137
#                 HADR_TIMEOUT(seconds) = 60
#         TIME_SINCE_LAST_RECV(seconds) = 13
#              PEER_WAIT_LIMIT(seconds) = 0
#            LOG_HADR_WAIT_CUR(seconds) = 0.000
#     LOG_HADR_WAIT_RECENT_AVG(seconds) = 0.000025
#    LOG_HADR_WAIT_ACCUMULATED(seconds) = 434.595
#                   LOG_HADR_WAIT_COUNT = 223713
# SOCK_SEND_BUF_REQUESTED,ACTUAL(bytes) = 0, 46080
# SOCK_RECV_BUF_REQUESTED,ACTUAL(bytes) = 0, 374400
#             PRIMARY_LOG_FILE,PAGE,POS = S0000280.LOG, 15571, 27902548040
#             STANDBY_LOG_FILE,PAGE,POS = S0000280.LOG, 15571, 27902548040
#                   HADR_LOG_GAP(bytes) = 0
#      STANDBY_REPLAY_LOG_FILE,PAGE,POS = S0000280.LOG, 15571, 27902548040
#        STANDBY_RECV_REPLAY_GAP(bytes) = 0
#                      PRIMARY_LOG_TIME = 02/06/2019 15:34:39.000000 (1549467279)
#                      STANDBY_LOG_TIME = 02/06/2019 15:34:39.000000 (1549467279)
#               STANDBY_REPLAY_LOG_TIME = 02/06/2019 15:34:39.000000 (1549467279)
#          STANDBY_RECV_BUF_SIZE(pages) = 2048
#              STANDBY_RECV_BUF_PERCENT = 0
#            STANDBY_SPOOL_LIMIT(pages) = 0
#                 STANDBY_SPOOL_PERCENT = NULL
#                    STANDBY_ERROR_TIME = NULL
#                  PEER_WINDOW(seconds) = 300
#                       PEER_WINDOW_END = 02/06/2019 15:40:25.000000 (1549467625)
#              READS_ON_STANDBY_ENABLED = N

#Secondary output:
# Database Member 0 -- Database PTR -- Standby -- Up 1 days 01:46:43 -- Date 2019-02-06-15.38.25.644168
# 
#                             <b>HADR_ROLE = STANDBY
#                           REPLAY_TYPE = PHYSICAL
#                         HADR_SYNCMODE = NEARSYNC
#                            STANDBY_ID = 0
#                         LOG_STREAM_ID = 0
#                            HADR_STATE = PEER
#                            HADR_FLAGS = TCP_PROTOCOL
#                   PRIMARY_MEMBER_HOST = azibmdb02
#                      PRIMARY_INSTANCE = db2ptr
#                        PRIMARY_MEMBER = 0
#                   STANDBY_MEMBER_HOST = azibmdb01
#                      STANDBY_INSTANCE = db2ptr
#                        STANDBY_MEMBER = 0
#                   HADR_CONNECT_STATUS = CONNECTED</b>
#              HADR_CONNECT_STATUS_TIME = 02/05/2019 13:51:47.205067 (1549374707)
#           HEARTBEAT_INTERVAL(seconds) = 15
#                      HEARTBEAT_MISSED = 0
#                    HEARTBEAT_EXPECTED = 6186
#                 HADR_TIMEOUT(seconds) = 60
#         TIME_SINCE_LAST_RECV(seconds) = 5
#              PEER_WAIT_LIMIT(seconds) = 0
#            LOG_HADR_WAIT_CUR(seconds) = 0.000
#     LOG_HADR_WAIT_RECENT_AVG(seconds) = 0.000023
#    LOG_HADR_WAIT_ACCUMULATED(seconds) = 434.595
#                   LOG_HADR_WAIT_COUNT = 223725
# SOCK_SEND_BUF_REQUESTED,ACTUAL(bytes) = 0, 46080
# SOCK_RECV_BUF_REQUESTED,ACTUAL(bytes) = 0, 372480
#             PRIMARY_LOG_FILE,PAGE,POS = S0000280.LOG, 15574, 27902562173
#             STANDBY_LOG_FILE,PAGE,POS = S0000280.LOG, 15574, 27902562173
#                   HADR_LOG_GAP(bytes) = 0
#      STANDBY_REPLAY_LOG_FILE,PAGE,POS = S0000280.LOG, 15574, 27902562173
#        STANDBY_RECV_REPLAY_GAP(bytes) = 155
#                      PRIMARY_LOG_TIME = 02/06/2019 15:37:34.000000 (1549467454)
#                      STANDBY_LOG_TIME = 02/06/2019 15:37:34.000000 (1549467454)
#               STANDBY_REPLAY_LOG_TIME = 02/06/2019 15:37:34.000000 (1549467454)
#          STANDBY_RECV_BUF_SIZE(pages) = 2048
#              STANDBY_RECV_BUF_PERCENT = 0
#            STANDBY_SPOOL_LIMIT(pages) = 0
#                 STANDBY_SPOOL_PERCENT = NULL
#                    STANDBY_ERROR_TIME = NULL
#                  PEER_WINDOW(seconds) = 300
#                       PEER_WINDOW_END = 02/06/2019 15:43:19.000000 (1549467799)
#              READS_ON_STANDBY_ENABLED = N
</code></pre>



## <a name="db2-pacemaker-configuration"></a>Configuration Pacemaker de Db2

Lorsque vous utilisez Pacemaker pour le basculement automatique en cas de défaillance d’un nœud, vous devez configurer vos instances Db2 et Pacemaker en conséquence. Cette section décrit ce type de configuration.

Les éléments suivants sont précédés de :

- **[A]**  : applicable à tous les nœuds
- **[1]**  : applicable uniquement au nœud 1 
- **[2]**  : applicable uniquement au nœud 2

**[A]** Prérequis pour la configuration Pacemaker :
1. Arrêtez les deux serveurs de base de données avec l’utilisateur db2\<sid> avec db2stop.
1. Remplacez l’environnement d’interpréteur de commandes pour l’utilisateur db2\<sid> par */bin/ksh*. Nous vous recommandons d’utiliser l’outil Yast. 


### <a name="pacemaker-configuration"></a>Configuration Pacemaker

> [!IMPORTANT]
> Des tests récents ont révélé des cas où netcat cessait de répondre aux demandes en raison du backlog et de sa capacité à ne gérer qu’une seule connexion. La ressource netcat cesse d’écouter les demandes d’Azure Load Balancer et l’adresse IP flottante devient indisponible.  
> Pour les clusters Pacemaker existants, nous vous recommandons de remplacer netcat par socat. Actuellement, nous vous recommandons d'utiliser l'agent de ressources azure-lb, qui fait partie du package resource-agents, avec la configuration requise suivante pour la version du package :
> - Pour SLES 12 SP4/SP5, la version minimum est resource-agents-4.3.018.a7fb5035-3.30.1.  
> - Pour SLES 15/15 SP1, la version minimum est resource-agents-4.3.0184.6ee15eb2-4.13.1.  
>
> Notez que la modification nécessitera un bref temps d’arrêt.  
> Pour les clusters Pacemaker existants, si la configuration a déjà été modifiée pour utiliser socat comme décrit à la page [Azure Load-Balancer Detection Hardening](https://www.suse.com/support/kb/doc/?id=7024128), il n’est pas nécessaire de passer immédiatement à l’agent de ressources azure-lb.

**[1]** Configuration Pacemaker spécifique à HADR IBM Db2 :
<pre><code># Put Pacemaker into maintenance mode
sudo crm configure property maintenance-mode=true
</code></pre>

**[1]** Créez des ressources IBM Db2 :
<pre><code># Replace **bold strings** with your instance name db2sid, database SID, and virtual IP address/Azure Load Balancer.

sudo crm configure primitive rsc_Db2_db2ptr_<b>PTR</b> db2 \
        params instance="<b>db2ptr</b>" dblist="<b>PTR</b>" \
        op start interval="0" timeout="130" \
        op stop interval="0" timeout="120" \
        op promote interval="0" timeout="120" \
        op demote interval="0" timeout="120" \
        op monitor interval="30" timeout="60" \
        op monitor interval="31" role="Master" timeout="60"

# Configure virtual IP - same as Azure Load Balancer IP
sudo crm configure primitive rsc_ip_db2ptr_<b>PTR</b> IPaddr2 \
        op monitor interval="10s" timeout="20s" \
        params ip="<b>10.100.0.10</b>"

# Configure probe port for Azure load Balancer
sudo crm configure primitive rsc_nc_db2ptr_<b>PTR</b> azure-lb port=<b>62500</b>

sudo crm configure group g_ip_db2ptr_<b>PTR</b> rsc_ip_db2ptr_<b>PTR</b> rsc_nc_db2ptr_<b>PTR</b>

sudo crm configure ms msl_Db2_db2ptr_<b>PTR</b> rsc_Db2_db2ptr_<b>PTR</b> \
        meta target-role="Started" notify="true"

sudo crm configure colocation col_db2_db2ptr_<b>PTR</b> inf: g_ip_db2ptr_<b>PTR</b>:Started msl_Db2_db2ptr_<b>PTR</b>:Master

sudo crm configure order ord_db2_ip_db2ptr_<b>PTR</b> inf: msl_Db2_db2ptr_<b>PTR</b>:promote g_ip_db2ptr_<b>PTR</b>:start

sudo crm configure rsc_defaults resource-stickiness=1000
sudo crm configure rsc_defaults migration-threshold=5000
</code></pre>

**[1]** Démarrez les ressources IBM Db2 :
* Sortez Pacemaker du mode de maintenance.
<pre><code># Put Pacemaker out of maintenance-mode - that start IBM Db2
sudo crm configure property maintenance-mode=false</pre></code>

**[1]** Vérifiez que l’état du cluster est OK et que toutes les ressources sont démarrées. Le nœud sur lequel les ressources s’exécutent n’a aucune importance.
<pre><code>sudo crm status</code>

# <a name="2-nodes-configured"></a>2 nœuds configurés
# <a name="5-resources-configured"></a>5 ressources configurées

# <a name="online--azibmdb01-azibmdb02-"></a>En ligne : [ azibmdb01 azibmdb02 ]

# <a name="full-list-of-resources"></a>Liste complète des ressources :

#  <a name="stonith-sbd----stonithexternalsbd-started-azibmdb02"></a>stonith-sbd    (stonith:external/sbd): Started azibmdb02
#  <a name="resource-group-g_ip_db2ptr_ptr"></a>Groupe de ressources : g_ip_db2ptr_PTR
#      <a name="rsc_ip_db2ptr_ptr--ocfheartbeatipaddr2-------started-azibmdb02"></a>rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
#      <a name="rsc_nc_db2ptr_ptr--ocfheartbeatazure-lb------started-azibmdb02"></a>rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
#  <a name="masterslave-set-msl_db2_db2ptr_ptr-rsc_db2_db2ptr_ptr"></a>Ensemble maître/esclave : msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
#      <a name="masters--azibmdb02-"></a>Maîtres : [ azibmdb02 ]
#      <a name="slaves--azibmdb01-"></a>Esclaves : [ azibmdb01 ]
</pre>

> [!IMPORTANT]
> Vous devez gérer l’instance Db2 en cluster Pacemaker à l’aide des outils Pacemaker. Si vous utilisez des commandes db2, telles que db2stop, Pacemaker détecte l’action comme un échec de ressource. Si vous effectuez une maintenance, vous pouvez placer les nœuds ou les ressources en mode de maintenance. Pacemaker suspend la supervision des ressources et vous pouvez alors utiliser les commandes d’administration db2 normales.


### <a name="configure-azure-load-balancer"></a>Configurer Azure Load Balancer
Pour configurer Azure Load Balancer, nous vous recommandons d’utiliser la [référence SKU standard Azure Load Balancer](../../../load-balancer/load-balancer-overview.md), puis de procéder comme suit :

> [!NOTE]
> La référence SKU Standard Load Balancer a des restrictions quant à l’accès aux adresses IP publiques à partir des nœuds situés sous le Load Balancer. L’article [Connectivité des points de terminaison publics pour les machines virtuelles avec Azure Standard Load Balancer dans les scénarios de haute disponibilité SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md) décrit comment activer ces nœuds pour accéder aux adresses IP publiques.

> [!IMPORTANT]
> L'adresse IP flottante n'est pas prise en charge sur une configuration d'adresse IP secondaire de carte réseau dans les scénarios d'équilibrage de charge. Pour plus d'informations, consultez [Limitations relatives à Azure Load Balancer](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Si vous avez besoin d'une adresse IP supplémentaire pour la machine virtuelle, déployez une deuxième carte réseau.  

1. Créez un pool d’adresses IP front-end :

   a. Dans le portail Azure, ouvrez Azure Load Balancer, sélectionnez **Pool d’adresses IP front-end**, puis **Ajouter**.

   b. Entrez le nom du nouveau pool d’adresses IP front-end (par exemple, **Db2-connection**).

   c. Définissez l’**affectation** sur **Statique** et entrez l’adresse IP **Virtual-IP** définie au début.

   d. Sélectionnez **OK**.

   e. Une fois le pool d’adresses IP frontal créé, notez son adresse IP.

1. Créez un pool back-end :

   a. Dans le portail Azure, ouvrez Azure Load Balancer, sélectionnez **backend pools** (Pools back-end), puis **Ajouter**.

   b. Entrez le nom du nouveau pool back-end (par exemple, **Db2-backend**).

   c. Cliquez sur **Ajouter une machine virtuelle**.

   d. Sélectionnez le groupe à haute disponibilité ou les machines virtuelles hébergeant la base de données IBM Db2 créée à l’étape précédente.

   e. Sélectionnez les machines virtuelles du cluster IBM Db2.

   f. Sélectionnez **OK**.

1. Créez une sonde d’intégrité :

   a. Dans le portail Azure, ouvrez Azure Load Balancer, sélectionnez **Sondes d’intégrité**, puis **Ajouter**.

   b. Entrez le nom de la nouvelle sonde d’intégrité (par exemple, **Db2-hp**).

   c. Sélectionnez **TCP** comme protocole et le port **62500**. Conservez la valeur **5** pour **Intervalle** et la valeur **2** pour **Seuil de défaillance sur le plan de l’intégrité**.

   d. Sélectionnez **OK**.

1. Créez les règles d’équilibrage de charge :

   a. Dans le portail Azure, ouvrez Azure Load Balancer, sélectionnez **Règles d’équilibrage de charge**, puis **Ajouter**.

   b. Entrez le nom de la nouvelle règle d’équilibrage de charge (par exemple, **Db2-SID**).

   c. Sélectionnez l’adresse IP front-end, le pool back-end et la sonde d’intégrité que vous avez créée précédemment (par exemple, **Db2-frontend**).

   d. Conservez **TCP** pour **Protocole**, puis entrez le *port de communication de la base de données*.

   e. Augmentez le **délai d’inactivité** à 30 minutes.

   f. Veillez à **activer l’IP flottante** .

   g. Sélectionnez **OK**.


### <a name="make-changes-to-sap-profiles-to-use-virtual-ip-for-connection"></a>Apporter des modifications aux profils SAP afin d’utiliser l’adresse IP virtuelle pour la connexion
Pour se connecter à l’instance primaire de la configuration HADR, la couche d’application SAP doit utiliser l’adresse IP virtuelle que vous avez définie et configurée pour Azure Load Balancer. Les modifications suivantes sont requises :

/sapmnt/\<SID>/profile/DEFAULT.PFL
<pre><code>SAPDBHOST = db-virt-hostname
j2ee/dbhost = db-virt-hostname
</code></pre>

/sapmnt/\<SID>/global/db6/db2cli.ini
<pre><code>Hostname=db-virt-hostname
</code></pre>



## <a name="install-primary-and-dialog-application-servers"></a>Installer des serveurs d’applications principal et de boîte de dialogue

Lorsque vous installez des serveurs d’applications principal et de boîte de dialogue avec une configuration HADR Db2, utilisez le nom d’hôte virtuel que vous avez choisi pour la configuration. 

Si vous avez effectué l’installation avant de créer la configuration HADR Db2, apportez les modifications comme décrit dans la section précédente et comme suit pour les piles SAP Java.

### <a name="abapjava-or-java-stack-systems-jdbc-url-check"></a>Vérification de l’URL JDBC des systèmes de piles Java ou ABAP+Java

Utilisez l’outil de configuration J2EE pour vérifier ou mettre à jour l’URL JDBC. Étant donné que l’outil de configuration J2EE est un outil graphique, le serveur X doit être installé :
 
1. Connectez-vous au serveur d’applications principal de l’instance J2EE et exécutez :   `sudo /usr/sap/*SID*/*Instance*/j2ee/configtool/configtool.sh`
1. Dans le cadre de gauche, choisissez **Security store** (Magasin de sécurité).
1. Dans le cadre de droite, choisissez la clé jdbc/pool/\<SAPSID>/url.
1. Remplacez le nom d’hôte dans l’URL JDBC par le nom d’hôte virtuel.
     `jdbc:db2://db-virt-hostname:5912/TSP:deferPrepares=0`
1. Sélectionnez **Ajouter**.
1. Pour enregistrer vos modifications, sélectionnez l’icône en forme de disque en haut à gauche.
1. Fermez l’outil de configuration.
1. Redémarrez l’instance Java.

## <a name="configure-log-archiving-for-hadr-setup"></a>Configurer l’archivage des journaux pour la configuration HADR
Pour configurer l’archivage des journaux Db2 pour la configuration HADR, nous vous recommandons de configurer à la fois les bases de données primaire et de secours pour activer la fonctionnalité de récupération des journaux automatique à partir de tous les emplacements des archives des journaux. Les bases de données primaire et de secours doivent être en mesure de récupérer les fichiers d’archive des journaux à partir de tous les emplacements auxquels l’une ou l’autre des instances de base de données peut archiver des fichiers journaux. 

L’archivage des journaux est effectué uniquement par la base de données primaire. Si vous modifiez les rôles HADR des serveurs de base de données ou si une défaillance se produit, la nouvelle base de données primaire est responsable de l’archivage des journaux. Si vous avez configuré plusieurs emplacements pour les archives des journaux, il est possible que vos journaux soient archivés à deux reprises. En cas d’une mise à jour locale ou distante, vous devrez peut-être également copier manuellement les journaux archivés à partir de l’ancien serveur principal à l’emplacement actif des journaux du nouveau serveur principal.

Nous vous recommandons de configurer un partage NFS commun dans lequel les journaux sont écrits à partir des deux nœuds. Le partage NFS doit être hautement disponible. 

Vous pouvez utiliser des partages NFS hautement disponibles existants pour les transports ou un répertoire de profil. Pour plus d'informations, consultez les pages suivantes :

- [Haute disponibilité pour NFS sur les machines virtuelles Azure sur SUSE Linux Enterprise Server][nfs-ha] 
- [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server avec Azure NetApp Files pour les applications SAP](./high-availability-guide-suse-netapp-files.md)
- [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) (pour créer des partages NFS)


## <a name="test-the-cluster-setup"></a>Tester la configuration du cluster

Cette section explique comment vous pouvez tester votre configuration HADR Db2. *Chaque test suppose que vous êtes connecté en tant qu’utilisateur racine* et que l’instance primaire IBM Db2 est exécutée sur la machine virtuelle *azibmdb01*.

L’état initial de tous les cas de test est expliqué ici : (crm_mon -r ou crm status)

- **crm status** est un instantané de l’état de Pacemaker au moment de l’exécution 
- **crm_mon -r** est la sortie continue de l’état de Pacemaker

<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Stopped
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Stopped
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     rsc_Db2_db2ptr_PTR      (ocf::heartbeat:db2):   Promoting azibmdb01
     Slaves: [ azibmdb02 ]
</code></pre>

L’état d’origine dans un système SAP est documenté dans Transaction DBACOCKPIT > Configuration > Vue d’ensemble, comme illustré dans l’image suivante :

![DBACockpit - Prémigration](./media/dbms-guide-ha-ibm/hadr-sap-mgr-org.png)




### <a name="test-takeover-of-ibm-db2"></a>Tester la prise de contrôle d’IBM Db2


> [!IMPORTANT] 
> Avant de commencer le test, vérifiez les points suivants :
> * Pacemaker ne comporte pas d’action ayant échoué (crm status).
> * Il n’existe aucune contrainte d’emplacement (reliquats d’un test de migration)
> * La synchronisation HADR IBM Db2 fonctionne. Vérifiez avec l’utilisateur db2\<sid> <pre><code>db2pd -hadr -db \<DBSID></code></pre>


Migrez le nœud qui exécute la base de données Db2 primaire en exécutant la commande suivante :
<pre><code>crm resource migrate msl_<b>Db2_db2ptr_PTR</b> azibmdb02</code></pre>

Une fois la migration terminée, la sortie crm status ressemble à ce qui suit :
<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb02 ]
     Slaves: [ azibmdb01 ]
</code></pre>

L’état d’origine dans un système SAP est documenté dans Transaction DBACOCKPIT > Configuration > Vue d’ensemble, comme illustré dans l’image suivante :

![DBACockpit - Post-migration](./media/dbms-guide-ha-ibm/hadr-sap-mgr-post.png)

La migration de ressources avec « crm resource migrate » crée des contraintes d’emplacement. Les contraintes d’emplacement doivent être supprimées. Si les contraintes d’emplacement ne sont pas supprimées, la ressource ne peut pas effectuer une restauration automatique, ou vous pouvez subir des prises de contrôle indésirables. 

Ramenez la ressource vers *azibmdb01* et effacez les contraintes d’emplacement
<pre><code>crm resource migrate msl_<b>Db2_db2ptr_PTR</b> azibmdb01
crm resource clear msl_<b>Db2_db2ptr_PTR</b>
</code></pre>

- **crm resource migrate \<res_name> \<host> :** crée des contraintes d’emplacement et peut entraîner des problèmes de prise de contrôle
- **crm resource clear \<res_name>**  : efface les contraintes d’emplacement
- **crm resource cleanup \<res_name>**  : efface toutes les erreurs de la ressource

### <a name="test-the-fencing-agent"></a>Tester l’agent de délimitation

Dans ce cas, nous testons l’isolation SBD, qui est recommandée lorsque vous utilisez SUSE Linux.

<pre><code>
azibmdb01:~ # ps -ef|grep sbd
root       2374      1  0 Feb05 ?        00:00:17 sbd: inquisitor
root       2378   2374  0 Feb05 ?        00:00:40 sbd: watcher: /dev/disk/by-id/scsi-36001405fbbaab35ee77412dacb77ae36 - slot: 0 - uuid: 27cad13a-0bce-4115-891f-43b22cfabe65
root       2379   2374  0 Feb05 ?        00:01:51 sbd: watcher: Pacemaker
root       2380   2374  0 Feb05 ?        00:00:18 sbd: watcher: Cluster

azibmdb01:~ # kill -9 2374
</code></pre>

Le nœud de cluster *azibmdb01* doit être redémarré. Le rôle HADR principal IBM Db2 va être déplacé vers *azibmdb02*. Quand *azibmdb01* est de nouveau en ligne, l’instance Db2 prend le rôle d’une instance de base de données secondaire. 

Si le service Pacemaker ne démarre pas automatiquement sur l’ancien nœud primaire redémarré, veillez à le démarrer manuellement avec :

<code><pre>sudo service pacemaker start</code></pre>

### <a name="test-a-manual-takeover"></a>Tester une prise de contrôle manuelle

Vous pouvez tester une prise de contrôle manuelle en arrêtant le service Pacemaker sur le nœud *azibmdb01* :
<pre><code>service pacemaker stop</code></pre>

status on *azibmdb02*
<pre><code>
2 nodes configured
5 resources configured

Online: [ azibmdb02 ]
OFFLINE: [ azibmdb01 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb02 ]
     Stopped: [ azibmdb01 ]
</code></pre>

Après le basculement, vous pouvez redémarrer le service sur *azibmdb01*.
<pre><code>service pacemaker start</code></pre>


### <a name="kill-the-db2-process-on-the-node-that-runs-the-hadr-primary-database"></a>Arrêter le processus Db2 sur le nœud qui exécute la base de données primaire HADR

<pre><code>#Kill main db2 process - db2sysc
azibmdb01:~ # ps -ef|grep db2s
db2ptr    34598  34596  8 14:21 ?        00:00:07 db2sysc 0

azibmdb01:~ # kill -9 34598
</code></pre>

L’instance Db2 va subir une défaillance et Pacemaker signalera l’état suivant :

<pre><code>
2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Stopped
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Stopped
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Slaves: [ azibmdb02 ]
     Stopped: [ azibmdb01 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_demote_0 on azibmdb01 'unknown error' (1): call=157, status=complete, exitreason='',
    last-rc-change='Tue Feb 12 14:28:19 2019', queued=40ms, exec=223ms

</code></pre>

Pacemaker va redémarrer l’instance de base de données primaire Db2 sur le même nœud, ou basculer vers le nœud qui exécute l’instance de base de données secondaire et une erreur est signalée.

<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb01 ]
     Slaves: [ azibmdb02 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_demote_0 on azibmdb01 'unknown error' (1): call=157, status=complete, exitreason='',
    last-rc-change='Tue Feb 12 14:28:19 2019', queued=40ms, exec=223ms
</code></pre>


### <a name="kill-the-db2-process-on-the-node-that-runs-the-secondary-database-instance"></a>Arrêter le processus Db2 sur le nœud qui exécute l’instance de base de données secondaire

<pre><code>azibmdb02:~ # ps -ef|grep db2s
db2ptr    65250  65248  0 Feb11 ?        00:09:27 db2sysc 0

azibmdb02:~ # kill -9</code></pre>

Le nœud passe en état d’échec et une erreur est signalée
<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     rsc_Db2_db2ptr_PTR      (ocf::heartbeat:db2):   FAILED azibmdb02
     Masters: [ azibmdb01 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_monitor_30000 on azibmdb02 'not running' (7): call=144, status=complete, exitreason='',
last-rc-change='Tue Feb 12 14:36:59 2019', queued=0ms, exec=0ms</code></pre>

L’instance Db2 est redémarrée dans le rôle secondaire qu’elle avait auparavant.

<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb01 ]
     Slaves: [ azibmdb02 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_monitor_30000 on azibmdb02 'not running' (7): call=144, status=complete, exitreason='',
    last-rc-change='Tue Feb 12 14:36:59 2019', queued=0ms, exec=0ms</code></pre>



### <a name="stop-db-via-db2stop-force-on-the-node-that-runs-the-hadr-primary-database-instance"></a>Arrêter Db2 par le biais de db2stop force sur le nœud qui exécute l’instance de base de données primaire HADR

<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb01 ]
     Slaves: [ azibmdb02 ]</code></pre>

En tant qu’utilisateur db2\<sid>, exécutez la commande db2stop force :
<pre><code>azibmdb01:~ # su - db2ptr
azibmdb01:db2ptr> db2stop force</code></pre>

Échec détecté
<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Stopped
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Stopped
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     rsc_Db2_db2ptr_PTR      (ocf::heartbeat:db2):   FAILED azibmdb01
     Slaves: [ azibmdb02 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_demote_0 on azibmdb01 'unknown error' (1): call=201, status=complete, exitreason='',
    last-rc-change='Tue Feb 12 14:45:25 2019', queued=1ms, exec=150ms</code></pre>

L’instance de base de données secondaire HADR Db2 a été promue au rôle principal
<pre><code> nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb02 ]
     Stopped: [ azibmdb01 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_start_0 on azibmdb01 'unknown error' (1): call=205, stat
us=complete, exitreason='',
    last-rc-change='Tue Feb 12 14:45:27 2019', queued=0ms, exec=865ms</pre></code>


### <a name="crash-vm-with-restart-on-the-node-that-runs-the-hadr-primary-database-instance"></a>Planter la machine virtuelle avec restart sur le nœud qui exécute l’instance de base de données primaire HADR

<pre><code>#Linux kernel panic - with OS restart
azibmdb01:~ # echo b > /proc/sysrq-trigger</code></pre>

Pacemaker va promouvoir l’instance secondaire au rôle d’instance primaire. L’ancienne instance primaire va prendre le rôle secondaire une fois la machine virtuelle et tous les services entièrement restaurés après le redémarrage de la machine virtuelle :

<pre><code> nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb01 ]
     Slaves: [ azibmdb02 ]</code></pre>



### <a name="crash-the-vm-that-runs-the-hadr-primary-database-instance-with-halt"></a>Planter la machine virtuelle qui exécute l’instance de base de données primaire HADR avec « halt »

<pre><code>#Linux kernel panic - halts OS
azibmdb01:~ # echo b > /proc/sysrq-trigger</code></pre>

Dans ce cas, Pacemaker détecte que le nœud qui exécute l’instance de base de données primaire ne répond pas.

<pre><code>2 nodes configured
5 resources configured

Node azibmdb01: UNCLEAN (online)
Online: [ azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb01 ]
     Slaves: [ azibmdb02 ]</code></pre>

L’étape suivante consiste à rechercher une situation de *Split-Brain*. Une fois que le nœud restant a déterminé que le nœud qui a exécuté en dernier l’instance de base de données primaire est en panne, un basculement des ressources est exécuté.
<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb02 ]
OFFLINE: [ azibmdb01 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb02 ]
     Stopped: [ azibmdb01 ] </code></pre>


En cas d’un « arrêt » du nœud, le nœud défaillant doit être redémarré à l’aide des outils de gestion Azure (dans le portail Azure, PowerShell ou l’interface de ligne de commande Azure). Une fois que le nœud défaillant est de nouveau en ligne, il démarre l’instance Db2 dans le rôle secondaire.

<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb02 ]
     Slaves: [ azibmdb01 ]</code></pre>

## <a name="next-steps"></a>Étapes suivantes
- [Scénarios et architecture de haute disponibilité pour SAP NetWeaver](./sap-high-availability-architecture-scenarios.md)
- [Configurer Pacemaker sur SUSE Linux Enterprise Server dans Azure](./high-availability-guide-suse-pacemaker.md)

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2233094]:https://launchpad.support.sap.com/#/notes/2233094
[1612105]:https://launchpad.support.sap.com/#/notes/1612105

[sles-for-sap-bp]:https://www.suse.com/documentation/sles-for-sap-12/
[db2-hadr-11.1]:https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.admin.ha.doc/doc/c0011267.html
[db2-hadr-10.5]:https://www.ibm.com/support/knowledgecenter/en/SSEPGG_10.5.0/com.ibm.db2.luw.admin.ha.doc/doc/c0011267.html
[dbms-db2]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_ibm
[sles-pacemaker]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-suse-pacemaker
[sap-instfind]:https://help.sap.com/viewer/9e41ead9f54e44c1ae1a1094b0f80712/ALL/en-US/576f5c1808de4d1abecbd6e503c9ba42.html
[nfs-ha]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-suse-nfs
[sles-ha-guide]:https://www.suse.com/releasenotes/x86_64/SLE-HA/12-SP4/
[ascs-ha]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-suse

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md
[azr-sap-plancheck]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-deployment-checklist
