---
title: Installation de SAP HANA sur SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Comment installer SAP HANA sur SAP HANA sur Azure (grandes instances).
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
ms.date: 01/16/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: a1430b32c0e74be7a0e50fa4c5c183018b2b55e0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006300"
---
# <a name="how-to-install-and-configure-sap-hana-large-instances-on-azure"></a>Guide pratique d’installation et de configuration de SAP HANA (grandes instances) sur Azure

Avant de lire cet article, familiarisez-vous avec les [termes courants concernant les grandes Instances HANA](hana-know-terms.md) et les [références SKU des grandes instances HANA](hana-available-skus.md).

L’installation de SAP HANA est de votre responsabilité. Vous pouvez démarrer l’installation d’un nouveau système SAP HANA sur le serveur Azure (grandes instances) une fois que vous avez établi la connectivité entre vos réseaux virtuels Azure et les unités de grandes instances HANA. 

> [!Note]
> Conformément à la stratégie de SAP, l’installation de SAP HANA doit être effectuée par une personne qui a passé la certification Certified SAP Technology Associate – SAP HANA Installation, ou par un intégrateur système certifié SAP.

Avant d'installer HANA 2.0, consultez la [note de support #2235581 relative à SAP HANA : systèmes d'exploitation pris en charge](https://launchpad.support.sap.com/#/notes/2235581/E) afin de vous assurer que le système d'exploitation utilisé est pris en charge par la version de SAP HANA que vous installez. Le nombre de systèmes d’exploitation pris en charge par HANA 2.0 est plus limité que celui des systèmes pris en charge par HANA 1.0. Vous devez également vérifier si la version du système d’exploitation qui vous intéresse est listée comme étant prise en charge pour l’unité HLI qui est indiquée dans [cette liste](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) publiée. Cliquez sur l’unité pour accéder à tous les détails, y compris à la liste des systèmes d’exploitation pris en charge par cette unité. 

Avant de commencer l’installation de HANA, validez les éléments suivants :
- [Unité(s) HLI](#validate-the-hana-large-instance-units)
- [Configuration du système d’exploitation](#operating-system)
- [Configuration réseau](#networking)
- [Configuration du stockage](#storage)


## <a name="validate-the-hana-large-instance-units"></a>Valider les unités de grandes instances HANA

Après avoir reçu l’unité de grandes instances HANA de Microsoft, validez les paramètres suivants, puis effectuez les ajustements nécessaires.

La **première étape** après réception de la Grande instance HANA et l’établissement de l’accès et de la connectivité aux instances consiste à vérifier sur le portail Azure si la ou les instances s’affichent avec les bonnes références SKU et le bon système d’exploitation. Consultez [Contrôle des grandes instances Azure HANA à l’aide du portail Azure](./hana-li-portal.md) pour connaître les étapes nécessaires à l’exécution des vérifications.

Après avoir reçu l’unité de grandes instances HANA, puis établi l’accès et la connectivité aux instances, la **deuxième étape** consiste à enregistrer le système d’exploitation de l’instance auprès de votre fournisseur de système d’exploitation. Cela inclut l’inscription de votre système d’exploitation SUSE Linux dans une instance de SUSE SMT déployée sur une machine virtuelle dans Azure. 

L’unité de grandes instances HANA peut se connecter à cette instance SMT. (Pour plus d’informations, consultez la [configuration du serveur SMT pour SUSE Linux](hana-setup-smt.md).) Sinon, l’enregistrement de votre système d’exploitation Red Hat doit se faire avec le gestionnaire d’abonnements Red Hat auquel vous devez vous connecter. Pour plus d’informations, consultez les remarques de la rubrique [Qu’est-ce que SAP HANA sur Azure (grandes instances) ?](./hana-overview-architecture.md?toc=/azure/virtual-machines/linux/toc.json). 

Cette étape est nécessaire pour l’application de correctifs au système d’exploitation, qui incombe au client. Pour SUSE, vous trouverez la documentation sur l’installation et la configuration SMT sur cette page relative à l’[installation de SMT](https://www.suse.com/documentation/sles-12/book_smt/data/smt_installation.html).

La **troisième étape** consiste à rechercher les nouveaux correctifs applicables à la version spécifique du système d’exploitation. Vérifiez que le niveau de correctif de grande instance HANA est à jour. Il peut arriver que les derniers correctifs ne soient pas inclus. Après réception d’une unité de grandes Instances HANA, il est obligatoire de vérifier si des correctifs doivent être appliqués.

La **quatrième étape** consiste à consulter les remarques SAP pertinentes relatives à l’installation et à la configuration de SAP HANA sur la version spécifique du système d’exploitation. En raison des recommandations variables, des modifications apportées aux remarques SAP ou des configurations qui dépendent de scénarios d’installation uniques, Microsoft ne sera pas toujours en mesure de fournir une unité de grandes instances HANA configurée à la perfection. 

Par conséquent, vous êtes tenu, en tant que client, de lire les remarques SAP liées à SAP HANA sur votre version de Linux. Vérifiez également les configurations de la version de système d’exploitation et appliquez les paramètres de configuration, si ce n’est déjà fait.

Vérifiez en particulier les paramètres ci-dessous et effectuez les ajustements éventuellement requis :

- net.core.rmem_max = 16777216
- net.core.wmem_max = 16777216
- net.core.rmem_default = 16777216
- net.core.wmem_default = 16777216
- net.core.optmem_max = 16777216
- net.ipv4.tcp_rmem = 65536 16777216 16777216
- net.ipv4.tcp_wmem = 65536 16777216 16777216

Sous SLES12 SP1 et RHEL 7.2, ces paramètres doivent être définis dans un fichier de configuration dans le répertoire /etc/sysctl.d. Il faut par exemple créer un fichier de configuration nommé 91-NetApp-HANA.conf. Pour les versions plus anciennes de SLES et RHEL, ces paramètres doivent être définis dans /etc/sysctl.conf.

Pour toutes les versions RHEL à partir de RHEL 6.3, gardez à l’esprit les points suivants : 
- Le paramètre sunrpc.tcp_slot_table_entries = 128 doit être défini dans /etc/modprobe.d/sunrpc-local.conf. Si le fichier n’existe pas, vous devez tout d’abord le créer en ajoutant l’entrée suivante : 
    - options sunrpc tcp_max_slot_table_entries=128

La **cinquième étape** consiste à vérifier l’heure système de votre unité de grandes instances HANA. Les instances sont déployées avec un fuseau horaire système. Ce fuseau horaire représente l’emplacement de la région Azure dans lequel se trouve l’horodatage de grande Instance HANA. Vous pouvez modifier l’heure système ou le fuseau horaire des instances que vous possédez. 

Si vous commandez d’autres instances dans votre locataire, vous devez adapter le fuseau horaire des nouvelles instances. Microsoft n’a aucune visibilité sur le fuseau horaire système défini pour vos instances après leur réception. C’est pourquoi les instances nouvellement déployées peuvent ne pas être définies sur le même fuseau horaire que celui choisi précédemment. Il vous incombe, en tant que client, d’adapter le fuseau horaire des instances remises, si nécessaire. 

La **sixième étape** consiste à vérifier le fichier etc/hosts. À la réception des unités, des adresses IP spécifiques sont affectées à différentes fonctions. Vérifiez le fichiers etc/hosts. Lorsque vous ajoutez des unités à un locataire existant, le fichier etc/hosts des systèmes nouvellement déployés ne contient pas les adresses IP des systèmes reçus précédemment. Il vous incombe donc, en tant que client, de faire en sorte que l’instance nouvellement déployée puisse interagir et résoudre les noms des unités précédemment déployées dans votre locataire. 


## <a name="operating-system"></a>Système d’exploitation

La taille de l'espace d'échange de l'image du système d'exploitation livré est définie sur 2 Go, conformément à la [note de support #1999997 relative à la FAQ : mémoire SAP HANA](https://launchpad.support.sap.com/#/notes/1999997/E). En tant que client, si vous souhaitez un autre paramètre, vous devez le définir vous-même.

[SUSE Linux Enterprise Server 12 SP1 for SAP Applications](https://www.suse.com/products/sles-for-sap/download/) est la distribution de Linux installée pour SAP HANA sur Azure (grandes instances). Cette distribution spéciale fournit des fonctionnalités propres à SAP « prêtes à l’emploi » (y compris des paramètres prédéfinis pour une exécution efficace de SAP sur SLES).

Consultez [la bibliothèque de ressources/les livres blancs](https://www.suse.com/products/sles-for-sap/resource-library#white-papers) sur le site web SUSE et [SAP sur SUSE](https://wiki.scn.sap.com/wiki/display/ATopics/SAP+on+SUSE) sur SCN (SAP Community Network) pour bénéficier de ressources utiles relatives au déploiement de SAP HANA sur SLES (notamment sur la configuration de la haute disponibilité, le renforcement de la sécurité propre aux opérations SAP, etc.).

Voici d’autres liens utiles concernant SAP sur SUSE :

- [SAP HANA sur le site SUSE Linux](https://wiki.scn.sap.com/wiki/display/ATopics/SAP+on+SUSE)
- [Meilleure pratique pour SAP : réplication de file d'attente - SAP NetWeaver sur SUSE Linux Enterprise 12](https://www.suse.com/media/guide/SLES4SAP-NetWeaver-ha-guide-EnqRepl-12_color_en.pdf)
- [ClamSAP – SLES Virus Protection for SAP](https://scn.sap.com/community/linux/blog/2014/04/14/clamsap--suse-linux-enterprise-server-integrates-virus-protection-for-sap) (ClamSAP : protection antivirus SLES pour SAP) (y compris SLES 12 for SAP Applications)

Les notes de support SAP suivantes concernent l’implémentation de SAP HANA sur SLES 12 :

- [Note de support SAP #1944799 relative aux instructions SAP HANA pour l’installation du système d’exploitation SLES](http://service.sap.com/sap/support/notes/1944799)
- [Note de support SAP #2205917 relative aux paramètres de système d’exploitation SAP HANA DB recommandés pour SLES 12, ainsi que les applications SAP](https://launchpad.support.sap.com/#/notes/2205917/E)
- [Note de support SAP #1984787 relative à SUSE Linux Enterprise Server 12 : notes d’installation](https://launchpad.support.sap.com/#/notes/1984787)
- [Note de support SAP #171356 - Logiciels SAP sur Linux :  Informations générales](https://launchpad.support.sap.com/#/notes/1984787)
- [Note de support SAP #1391070 relative aux solutions Linux UUID](https://launchpad.support.sap.com/#/notes/1391070)

[Red Hat Enterprise Linux pour SAP HANA](https://www.redhat.com/en/resources/red-hat-enterprise-linux-sap-hana) est une autre offre permettant l’exécution de SAP HANA sur les grandes instances HANA. Les versions 7.2 et 7.3 de RHEL sont disponibles et prises en charge. 

Voici d’autres liens SAP utiles relatifs à Red Hat :
- [Site de SAP HANA sur Red Hat Linux](https://wiki.scn.sap.com/wiki/display/ATopics/SAP+on+Red+Hat).

Les notes de support SAP suivantes concernent l’implémentation de SAP HANA sur Red Hat :

- [Note de support SAP #2009879 relative aux instructions SAP HANA pour les systèmes d’exploitation Red Hat Enterprise Linux (RHEL)](https://launchpad.support.sap.com/#/notes/2009879/E)
- [Note de support SAP #2292690 - SAP HANA DB : Paramètres de système d’exploitation recommandés pour RHEL 7](https://launchpad.support.sap.com/#/notes/2292690)
- [Note de support SAP #1391070 relative aux solutions Linux UUID](https://launchpad.support.sap.com/#/notes/1391070)
- [Note de support SAP #2228351 - Linux : SAP HANA Database SPS 110 révision 11 (ou plus) sur RHEL 6 ou SLES 11](https://launchpad.support.sap.com/#/notes/2228351)
- [Note de support SAP #2397039 - FAQ : SAP sur RHEL](https://launchpad.support.sap.com/#/notes/2397039)
- [Note de support SAP #2002167 - Red Hat Enterprise Linux 7.x : installation et mise à niveau](https://launchpad.support.sap.com/#/notes/2002167)

### <a name="time-synchronization"></a>Synchronisation temporelle

Les applications SAP basées sur l’architecture de SAP NetWeaver sont sensibles aux différences d’heure des divers composants qui constituent le système SAP. Les vidages courts SAP ABAP avec le titre d’erreur ZDATE\_LARGE\_TIME\_DIFF vous sont probablement familiers. C’est parce que ces vidages courts apparaissent lorsque l’heure système des différents serveurs ou machines virtuelles se décale trop.

Pour SAP HANA sur Azure (grandes instances), la synchronisation de l’heure effectuée dans Azure ne s’applique pas aux unités Compute dans les horodatages de grandes instances. Cette synchronisation ne concerne pas les applications SAP qui s’exécutent sur des machines virtuelles Azure natives, car Azure garantit la bonne synchronisation de l’heure du système. 

Par conséquent, vous devez configurer un serveur temporel distinct qui sera utilisé par les serveurs d’applications SAP s’exécutant sur des machines virtuelles Azure et les instances de base de données SAP HANA s’exécutant sur des grandes instances HANA. L’heure de l’infrastructure de stockage dans les horodatages de grandes instances est synchronisée avec les serveurs NTP.


## <a name="networking"></a>Mise en réseau
Nous partons du principe que vous avez suivi les recommandations données dans les documents ci-après pour concevoir vos réseaux virtuels Azure et connecter ceux-ci aux grandes instances HANA :

- [Vue d’ensemble et architecture de SAP HANA (grandes instances) sur Azure](./hana-overview-architecture.md)
- [Infrastructure et connectivité à SAP HANA (grandes instances) sur Azure](hana-overview-infrastructure-connectivity.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

Certains détails concernant la mise en réseau d’unités individuelles valent la peine d’être soulignés. Chaque unité de grandes instances HANA est fournie avec deux ou trois adresses IP affectées à deux ou trois de ses ports de carte réseau. Trois adresses IP sont utilisées dans les configurations scale-out HANA et dans le scénario de réplication système HANA. Une des adresses IP affectées à la carte réseau de l’unité se situe hors du pool d’adresses IP du serveur décrit à la rubrique [Vue d’ensemble et architecture de SAP HANA (grandes instances) sur Azure](./hana-overview-architecture.md).

Pour plus d’informations sur les détails d’Ethernet pour votre architecture, consultez les [scénarios HLI pris en charge](hana-supported-scenario.md).

## <a name="storage"></a>Stockage

La disposition de stockage pour SAP HANA sur Azure (grandes instances) est configurée par SAP HANA sur Azure `service management` via les recommandations de SAP. Ces instructions sont documentées dans le livre blanc [Exigences de stockage SAP HANA](https://go.sap.com/documents/2015/03/74cdb554-5a7c-0010-82c7-eda71af511fa.html). 

La taille approximative des divers volumes avec les différentes références SKU de grandes instances HANA est documentée dans [Vue d’ensemble et architecture de SAP HANA (grandes instances) sur Azure](hana-overview-architecture.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

Les conventions d’affectation de noms des volumes de stockage sont répertoriées dans le tableau suivant :

| Utilisation du stockage | Nom du montage | Nom du volume | 
| --- | --- | ---|
| Données HANA | /hana/data/SID/mnt0000\<m> | Adresse IP du stockage:/hana_data_SID_mnt00001_tenant_vol |
| Journal HANA | /Hana/log/SID/mnt0000\<m> | Adresse IP du stockage:/hana_log_SID_mnt00001_tenant_vol |
| Sauvegarde de fichier journal HANA | HANA/log/backups | Adresse IP du stockage:/hana_log_backups_SID_mnt00001_tenant_vol |
| HANA partagé | /Hana/Shared/SID | Adresse IP du stockage:/hana_shared_SID_mnt00001_tenant_vol/shared |
| /usr/sap | /usr/SAP/SID | Adresse IP du stockage:/hana_shared_SID_mnt00001_tenant_vol/usr_sap |

*SID* est l’ID système de l’instance HANA. 

*Tenant* est une énumération interne des opérations lors du déploiement d’un locataire.

HANA usr/sap partagent le même volume. La nomenclature des points de montage inclut l’ID système des instances HANA, ainsi que le nombre de montages. Dans les déploiements scale-up, il n’existe qu’un seul montage, par exemple mnt00001. Dans un déploiement scale-out, vous devriez voir un nombre de montages équivalent au nombre de nœuds worker et maîtres. 

Pour les environnements scale-out, les données, les journaux et les volumes de sauvegarde de fichier journal sont partagés et associés à chaque nœud dans la configuration scale-out. Pour les configurations composées de plusieurs instances SAP, un autre jeu de volumes est créé et attaché à l’unité de grandes instances HANA. Consultez [Scénarios HLI pris en charge](hana-supported-scenario.md) pour connaître les détails de la disposition du stockage de votre scénario.

En regardant une unité de grandes instances HANA de plus près, vous pouvez vous rendre compte de la générosité du volume disque attribué à HANA/data et remarquer la présence d’un volume HANA/log/backup. Si HANA/data est aussi volumineux, c’est parce que les captures instantanées du stockage que nous vous proposons l’exigent. Plus vous prenez d’instantanés de stockage, plus les instantanés occupent d’espace dans les volumes de stockage qui vous sont assignés. 

Le volume HANA/log/backup n’est pas conçu pour être le volume sur lequel placer des sauvegardes de base de données. Il est dimensionné pour être utilisé en tant que volume de sauvegarde pour les sauvegardes de fichier journal HANA. Pour plus de détails, consultez [Haute disponibilité et récupération d’urgence SAP HANA (grandes instances) sur Azure](hana-overview-high-availability-disaster-recovery.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). 

En plus de la capacité de stockage fournie, les clients peuvent acheter de l’espace de stockage supplémentaire par incréments de 1 To. Cet espace supplémentaire peut être ajouté en tant que nouveaux volumes à une grande instance HANA.

Pendant l’intégration à SAP HANA sur Azure `service management`, le client spécifie un identifiant d’utilisateur (UID) et un identifiant de groupe (GID) pour l’utilisateur sidadm et le groupe sapsys (par exemple : 1000,500). Vous devez utiliser ces valeurs lors de l’installation du système SAP HANA. Comme vous souhaitez déployer plusieurs instances HANA sur une unité, vous obtenez plusieurs ensembles de volumes (un ensemble pour chaque instance). Par conséquent, au moment du déploiement, vous devez définir :

- Le SID des différentes instances HANA (sidadm en est dérivé).
- Les tailles de mémoire des différentes instances HANA. En effet, la taille de mémoire par instance définit la taille des volumes dans chaque jeu de volumes.

Selon les recommandations du fournisseur de stockage, les options de montage suivantes sont configurées pour tous les volumes montés (exclut le LUN de démarrage) :

- nfs    rw, vers=4, hard, timeo=600, rsize=1048576, wsize=1048576, intr, noatime, lock 0 0

Ces points de montage sont configurés dans/etc/fstab, comme indiqué dans les graphiques suivants :

![fstab des volumes montés dans l’unité de grande instance HANA](./media/hana-installation/image1_fstab.PNG)

La sortie de la commande df -h sur une unité de grande instance HANA S72m ressemblerait à ceci :

![Capture d’écran montrant la sortie de la commande pour une unité de grande instance HANA.](./media/hana-installation/image2_df_output.PNG)


Le contrôleur de stockage et les nœuds dans les tampons de grandes instances sont synchronisés avec les serveurs NTP. Lorsque vous synchronisez les unités SAP HANA sur Azure (grandes instances) et les machines virtuelles Azure avec un serveur NTP, vous ne devriez pas rencontrer d’écart de temps entre l’infrastructure et les unités Compute dans Azure ou les horodatages de grandes instances.

Pour optimiser SAP HANA pour le stockage sous-jacent, définissez les paramètres de configuration SAP HANA suivants :

- max_parallel_io_requests 128
- async_read_submit on
- async_write_submit_active on
- async_write_submit_blocks all
 
Pour les versions de SAP HANA 1.0 jusqu’à SPS12, ces paramètres peuvent être définis au cours de l’installation de la base de données SAP HANA, comme décrit dans la [note de support SAP #2267798 relative à la configuration de la base de données SAP HANA](https://launchpad.support.sap.com/#/notes/2267798).

Vous pouvez également configurer les paramètres après l’installation de la base de données SAP HANA via le framework hdbparam. 

Le stockage utilisé dans les grandes Instances HANA limite la taille des fichiers. La [limite de taille est de 16 To](https://docs.netapp.com/ontap-9/index.jsp?topic=%2Fcom.netapp.doc.dot-cm-vsmg%2FGUID-AA1419CF-50AB-41FF-A73C-C401741C847C.html) par fichier. Contrairement aux limitations de taille de fichier dans les systèmes de fichiers EXT3, HANA ne reconnaît pas implicitement la limite de stockage appliquée par le stockage sur de grandes Instances HANA. Par conséquent, HANA ne crée pas automatiquement un nouveau fichier de données lorsque la limite de taille de 16 To est atteinte. Quand HANA tente d’étendre le fichier au-delà de 16 To, il signale des erreurs et le serveur d’index se bloque à la fin.

> [!IMPORTANT]
> Afin d’empêcher HANA d’essayer d’augmenter la taille des fichiers de données au-delà de la limite de 16 To imposée par le stockage sur une grande Instance HANA, vous devez définir les paramètres suivants dans le fichier de configuration global.ini de SAP HANA.
> 
> - datavolume_striping=true
> - datavolume_striping_size_gb = 15000
> - Voir aussi la note SAP [#2400005](https://launchpad.support.sap.com/#/notes/2400005)
> - N’oubliez pas la note SAP [#2631285](https://launchpad.support.sap.com/#/notes/2631285)


L’utilisation du framework hdbparam est déconseillée avec SAP HANA 2.0. Il faut donc définir les paramètres à l’aide de commandes SQL. Pour plus d'informations, consultez la [note SAP #2399079 : Élimination de hdbparam dans HANA 2](https://launchpad.support.sap.com/#/notes/2399079).

Consultez les [scénarios HLI pris en charge](hana-supported-scenario.md) pour connaître la disposition du stockage de votre architecture.


**Étapes suivantes**

- Voir [Installation de HANA sur HLI](hana-example-installation.md)










































 







 
