---
title: Déploiement SGBD de machines virtuelles SAP ASE Azure pour charge de travail SAP | Microsoft Docs
description: Déploiement SGBD de machines virtuelles SAP ASE Azure pour charge de travail SAP
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 04/13/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: a3ef3fc2afa07590ff676d57c22f05ed723539f9
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94957722"
---
# <a name="sap-ase-azure-virtual-machines-dbms-deployment-for-sap-workload"></a>Déploiement SGBD de machines virtuelles SAP ASE Azure pour charge de travail SAP

Ce document décrit les différents domaines à prendre en compte lors du déploiement de SAP ASE dans Azure IaaS. Comme condition préalable à ce document, vous devez avoir lu le document [Considerations for Azure Virtual Machines DBMS deployment for SAP workload](dbms_guide_general.md) (Facteurs à prendre en compte pour le déploiement SGBD des machines virtuelles Azure pour la charge de travail SAP) et d’autres guides de la [documentation sur la charge de travail SAP sur Azure](./get-started.md). Ce document aborde l’exécution de SAP ASE sur les systèmes d’exploitation Linux et Windows. La version minimale prise en charge sur Azure est SAP ASE 16.0.02 (version 16, pack de support 2). Il est recommandé de déployer la dernière version de SAP et le niveau de correctif logiciel le plus récent.  Au minimum, SAP ASE 16.0.03.07 (version 16, pack de support 3, niveau de correctif logiciel 7) est recommandé.  La version la plus récente de SAP est disponible dans [Targeted ASE 16.0 Release Schedule and CR list Information](https://wiki.scn.sap.com/wiki/display/SYBASE/Targeted+ASE+16.0+Release+Schedule+and+CR+list+Information).

Des informations supplémentaires sur la prise en charge des versions avec les applications SAP ou l’emplacement du support d’installation sont disponibles, en plus du tableau de disponibilité des produits SAP aux endroits suivants :

- [Note de support SAP n° 2134316](https://launchpad.support.sap.com/#/notes/2134316)
- [Note de support SAP n° 1941500](https://launchpad.support.sap.com/#/notes/1941500)
- [Note de support SAP n° 1590719](https://launchpad.support.sap.com/#/notes/1590719)
- [Note de support SAP n° 1973241](https://launchpad.support.sap.com/#/notes/1973241)

Remarque : dans toute la documentation à l’intérieur et à l’extérieur de l’environnement SAP, le nom du produit est référencé comme Sybase ASE ou SAP ASE, ou les deux dans certains cas. Afin de rester cohérent, nous utilisons le nom **SAP ASE** dans cette documentation.

## <a name="operating-system-support"></a>Prise en charge du système d’exploitation
Le tableau de disponibilité des produits SAP contient les combinaisons prises en charge du système d’exploitation et du noyau SAP pour chaque application SAP.  Les distributions Linux SUSE 12.x, SUSE 15.x, Red Hat 7.x sont entièrement prises en charge.  Oracle Linux en tant que système d’exploitation pour SAP ASE n’est pas pris en charge.  Nous vous recommandons d’utiliser les versions Linux les plus récentes disponibles. Les clients Windows doivent utiliser les versions Windows Server 2016 ou Windows Server 2019.  Les versions plus anciennes de Windows, telles que Windows 2012, sont techniquement prises en charge, mais la dernière version de Windows est toujours recommandée.


## <a name="specifics-to-sap-ase-on-windows"></a>Caractéristiques de SAP ASE sur Windows
Depuis Microsoft Azure, vous pouvez migrer vos applications SAP ASE existantes vers les machines virtuelles Azure. Dans une machine virtuelle Azure, SAP ASE vous permet de réduire le coût total de possession lié au déploiement, à la gestion et à la maintenance des applications d’entreprise en les migrant facilement vers Microsoft Azure. Lorsque le logiciel SAP ASE est installé dans une machine virtuelle Azure, les administrateurs et développeurs peuvent continuer à utiliser les outils de développement et d’administration disponibles en local.

Microsoft Azure propose de nombreux types de machine virtuelle différents qui vous permettent d’exécuter des petits systèmes et paysages SAP ainsi que des grands systèmes et paysages SAP avec des milliers d’utilisateurs. Dans le cas du dimensionnement SAP, les numéros SAP associés aux références SKU des différentes machines virtuelles certifiées pour SAP sont fournis dans [Note de support SAP n° 1928533](https://launchpad.support.sap.com/#/notes/1928533).

Vous trouverez la documentation pour installer SAP ASE sur Windows dans le [Guide d’installation de SAP ASE pour Windows](https://help.sap.com/viewer/36031975851a4f82b1022a9df877280b/16.0.3.7/en-US/a660d3f1bc2b101487cbdbf10069c3ac.html)

Verrouiller les pages en mémoire est un paramètre qui empêche l’écriture sur le fichier d’échange de la mémoire tampon de la base de données SAP ASE.  Ce paramètre est utile pour les grands systèmes chargés avec beaucoup de mémoire. Pour plus d’informations, contactez BC-DB-SYB. 


## <a name="linux-operating-system-specific-settings"></a>Paramètres spécifiques au système d’exploitation Linux
Sur les machines virtuelles Linux, exécutez `saptune` avec le profil SAP-ASE. Les Huge Pages Linux doivent être activées par défaut et peuvent être vérifiées avec la commande  

`cat /proc/meminfo` 

La taille de la page est généralement de 2 048 Ko. Pour plus d’informations, consultez l’article [Huge Pages sur Linux](https://help.sap.com/viewer/ecbccd52e7024feaa12f4e780b43bc3b/16.0.3.7/en-US/a703d580bc2b10149695f7d838203fad.html) 


## <a name="recommendations-on-vm-and-disk-structure-for-sap-ase-deployments"></a>Recommandations relatives à la structure des machines virtuelles et des disques pour les déploiements SAP ASE

Les applications SAP ASE pour SAP NetWeaver sont prises en charge sur tous les types de machines virtuelles figurant dans [Note de support SAP n° 1928533](https://launchpad.support.sap.com/#/notes/1928533). Les types de machines virtuelles classiques utilisés pour les serveurs de base de données SAP ASE de taille moyenne incluent Esv3.  Les bases de données volumineuses de plusieurs téraoctets peuvent tirer parti des types de machines virtuelles de la série M. Les performances d’écriture sur le disque du journal des transactions SAP ASE peuvent être améliorées en activant l’accélérateur d’écriture de la série M. L’accélérateur d’écriture doit être testé soigneusement avec SAP ASE en raison de la façon dont SAP ASE effectue les écritures de journal.  Passez en revue [Note de support SAP n° 2816580](../../how-to-enable-write-accelerator.md) et envisagez d’exécuter un test de performances.  
L’accélérateur d’écriture est conçu pour le disque du journal des transactions uniquement. Le cache au niveau du disque doit être défini sur AUCUN. Ne soyez pas surpris si l’accélérateur d’écriture Azure n’affiche pas les mêmes améliorations qu’avec un autre système de gestion de base de données (SGBD). En fonction de la façon dont SAP ASE écrit dans le journal des transactions, il se peut que l’accélérateur d’écriture Azure produise peu ou pas du tout d’accélération.
Des disques distincts sont recommandés pour les unités de données et les unités de journaux.  Les bases de données système sybsecurity et `saptools` ne nécessitent pas de disques dédiés et peuvent être placées sur les disques contenant les unités de données et de journaux de la base de données SAP 

![Configuration du stockage pour SAP ASE](./media/dbms-guide-sap-ase/sap-ase-disk-structure.png)

### <a name="file-systems-stripe-size--io-balancing"></a>Systèmes de fichiers, taille de bande et équilibrage des E/S 
SAP ASE écrit des données de façon séquentielle dans les unités de stockage sur disque, sauf en cas de configuration différente. Cela signifie qu’une base de données SAP ASE vide avec quatre unités écrira des données dans la première unité uniquement.  Il sera possible d’écrire des données dans les autres unités de disque uniquement quand la première sera pleine.  Le volume des E/S de lecture et d’écriture sur chaque unité SAP ASE est susceptible d’être différent. Pour équilibrer les E/S de disque sur tous les disques Azure disponibles, vous devez utiliser des espaces de stockage Windows ou Linux LVM2. Sur Linux, il est recommandé d’utiliser le système de fichiers XFS pour formater les disques. La taille de la bande LVM doit être testée à l’aide d’un test de performances. Une taille de bande de 128 Ko est un bon point de départ. Sur Windows, la taille d’unité d’allocation NTFS doit être testée. Une taille de 64 Ko peut être utilisée comme valeur de départ. 

Il est recommandé de configurer l’extension automatique de la base de données comme décrit dans l’article [Configuring Automatic Database Space Expansion in SAP Adaptive Server Enterprise](https://blogs.sap.com/2014/07/09/configuring-automatic-database-space-expansion-in-sap-adaptive-server-enterprise/) et [Note de support SAP n° 1815695](https://launchpad.support.sap.com/#/notes/1815695). 

### <a name="sample-sap-ase-on-azure-virtual-machine-disk-and-file-system-configurations"></a>Exemples de configurations de SAP ASE sur des systèmes de fichiers, des disques et des machines virtuelles Azure 
Les modèles ci-dessous présentent des exemples de configurations pour Linux et Windows. Avant de confirmer la configuration des machines virtuelles et des disques, vérifiez que les quotas de bande passante de stockage et de réseau de la machine virtuelle individuelle sont suffisants pour répondre aux besoins de l’entreprise. Gardez également à l’esprit que différents types de machines virtuelles Azure ont un nombre maximal différent de disques pouvant être attachés à la machine virtuelle. Par exemple, une machine virtuelle E4s_v3 a un débit d’ES de stockage limité à 48 Mo/s. Si le débit de stockage requis par l’activité de sauvegarde de base de données exige plus de 48 Mo/s, un plus grand type de machine virtuelle avec davantage de débit de bande passante de stockage est inévitable. Quand vous configurez le stockage Azure, vous devez également garder à l’esprit que, en particulier avec le [stockage Premium Azure](../../premium-storage-performance.md) le débit et les E/S par seconde par Go de capacité changent. Pour plus d’informations à ce sujet, consultez l’article [Quels sont les types de disque disponibles dans Azure ?](../../disks-types.md). Les quotas pour des types de machines virtuelles Azure spécifiques sont décrits dans l’article [Tailles de machine virtuelle à mémoire optimisée](../../sizes-memory.md) et les articles qui y sont liés. 

> [!NOTE]
>  Si un système SGBD est déplacé d’un site local vers Azure, il est recommandé d’effectuer une supervision de la machine virtuelle et d’évaluer le processeur, la mémoire, les E/S par seconde et le débit de stockage. Comparez les valeurs maximales observées aux limites de quotas de machines virtuelles décrites dans les articles mentionnés ci-dessus

Les exemples ci-dessous sont fournis à titre d’illustration et peuvent être modifiés en fonction des besoins individuels. En raison de la conception de SAP ASE, le nombre d’unités de données n’est pas aussi essentiel qu’avec d’autres bases de données. Le nombre des unités de données décrites dans ce document est donné à titre de référence uniquement. 

Exemple de configuration d’un petit serveur de base de données SAP ASE avec une taille de base de données comprise entre 50 Go et 250 Go, par exemple SAP Solution Manager

| Configuration | Windows | Linux | Commentaires |
| --- | --- | --- | --- |
| Type de machine virtuelle | E4s_v3 (4 processeurs virtuels/32 Go de RAM) | E4s_v3 (4 processeurs virtuels/32 Go de RAM) | --- |
| Mise en réseau accélérée | Activer | Activer | ---|
| Version SAP ASE | 16.0.03.07 ou ultérieure | 16.0.03.07 ou ultérieure | --- |
| Nombre d’unités de données | 4 | 4 | ---|
| Nombre d’unités de journaux | 1 | 1 | --- |
| Nombre d’unités temporaires | 1 | 1 | plus pour la charge de travail SAP BW |
| Système d’exploitation | Windows Server 2019 | SUSE 12 SP4/15 SP1 ou RHEL 7.6 | --- |
| Agrégation de disques | Espaces de stockage | LVM2 | --- |
| Système de fichiers | NTFS | XFS |
| Taille de bloc de format | nécessite des tests de charge de travail | nécessite des tests de charge de travail | --- |
| Nombre et type de disques de données | Stockage Premium : 2 x P10 (RAID0) | Stockage Premium : 2 x P10 (RAID0)| Cache = Lecture seule |
| Nombre et type de disques de données | Stockage Premium : 1 x P20  | Stockage Premium : 1 x P20 | Cache = AUCUN |
| Paramètre ASE MaxMemory | 90 % de RAM physique | 90 % de RAM physique | en supposant une instance unique |
| Nombre d’unités de sauvegarde | 4 | 4| --- |
| Nombre et type de disques de sauvegarde | 1 | 1 | --- |


Exemple de configuration d’un serveur moyen de base de données SAP ASE avec une taille de base de données comprise entre 250 Go et 750 Go, par exemple un plus petit système SAP Business Suite

| Configuration | Windows | Linux | Commentaires |
| --- | --- | --- | --- |
| Type de machine virtuelle | E16s_v3 (16 processeurs virtuels/128 Go de RAM) | E16s_v3 (16 processeurs virtuels/128 Go de RAM) | --- |
| Mise en réseau accélérée | Activer | Activer | ---|
| Version SAP ASE | 16.0.03.07 ou ultérieure | 16.0.03.07 ou ultérieure | --- |
| Nombre d’unités de données | 8 | 8 | ---|
| Nombre d’unités de journaux | 1 | 1 | --- |
| Nombre d’unités temporaires | 1 | 1 | plus pour la charge de travail SAP BW |
| Système d’exploitation | Windows Server 2019 | SUSE 12 SP4/15 SP1 ou RHEL 7.6 | --- |
| Agrégation de disques | Espaces de stockage | LVM2 | --- |
| Système de fichiers | NTFS | XFS |
| Taille de bloc de format | nécessite des tests de charge de travail | nécessite des tests de charge de travail | --- |
| Nombre et type de disques de données | Stockage Premium : 4 x P20 (RAID0) | Stockage Premium : 4 x P20 (RAID0)| Cache = Lecture seule |
| Nombre et type de disques de données | Stockage Premium : 1 x P20  | Stockage Premium : 1 x P20 | Cache = AUCUN |
| Paramètre ASE MaxMemory | 90 % de RAM physique | 90 % de RAM physique | en supposant une instance unique |
| Nombre d’unités de sauvegarde | 4 | 4| --- |
| Nombre et type de disques de sauvegarde | 1 | 1 | --- |

Exemple de configuration d’un petit serveur de base de données SAP ASE avec une taille de base de données comprise entre 750 Go et 2 000 Go, par exemple un plus grand système SAP Business Suite

| Configuration | Windows | Linux | Commentaires |
| --- | --- | --- | --- |
| Type de machine virtuelle | E64s_v3 (64 processeurs virtuels/432 Go de RAM) | E64s_v3 (64 processeurs virtuels/432 Go de RAM) | --- |
| Mise en réseau accélérée | Activer | Activer | ---|
| Version SAP ASE | 16.0.03.07 ou ultérieure | 16.0.03.07 ou ultérieure | --- |
| Nombre d’unités de données | 16 | 16 | ---|
| Nombre d’unités de journaux | 1 | 1 | --- |
| Nombre d’unités temporaires | 1 | 1 | plus pour la charge de travail SAP BW |
| Système d’exploitation | Windows Server 2019 | SUSE 12 SP4/15 SP1 ou RHEL 7.6 | --- |
| Agrégation de disques | Espaces de stockage | LVM2 | --- |
| Système de fichiers | NTFS | XFS |
| Taille de bloc de format | nécessite des tests de charge de travail | nécessite des tests de charge de travail | --- |
| Nombre et type de disques de données | Stockage Premium : 4 x P30 (RAID0) | Stockage Premium : 4 x P30 (RAID0)| Cache = Lecture seule |
| Nombre et type de disques de données | Stockage Premium : 1 x P20  | Stockage Premium : 1 x P20 | Cache = AUCUN |
| Paramètre ASE MaxMemory | 90 % de RAM physique | 90 % de RAM physique | en supposant une instance unique |
| Nombre d’unités de sauvegarde | 4 | 4| --- |
| Nombre et type de disques de sauvegarde | 1 | 1 | --- |


Exemple de configuration d’un petit serveur de base de données SAP ASE avec une taille de base de données de plus de 2 To, par exemple un plus grand système SAP Business Suite utilisé de manière globale

| Configuration | Windows | Linux | Commentaires |
| --- | --- | --- | --- |
| Type de machine virtuelle | Série M (1,0 à 4,0 To de RAM)  | Série M (1,0 à 4,0 To de RAM) | --- |
| Mise en réseau accélérée | Activer | Activer | ---|
| Version SAP ASE | 16.0.03.07 ou ultérieure | 16.0.03.07 ou ultérieure | --- |
| Nombre d’unités de données | 32 | 32 | ---|
| Nombre d’unités de journaux | 1 | 1 | --- |
| Nombre d’unités temporaires | 1 | 1 | plus pour la charge de travail SAP BW |
| Système d’exploitation | Windows Server 2019 | SUSE 12 SP4/15 SP1 ou RHEL 7.6 | --- |
| Agrégation de disques | Espaces de stockage | LVM2 | --- |
| Système de fichiers | NTFS | XFS |
| Taille de bloc de format | nécessite des tests de charge de travail | nécessite des tests de charge de travail | --- |
| Nombre et type de disques de données | Stockage Premium : 4+ x P30 (RAID0) | Stockage Premium : 4+ x P30 (RAID0)| Cache = Lecture seule, envisagez d’utiliser un disque Ultra Azure |
| Nombre et type de disques de journaux | Stockage Premium : 1 x P20  | Stockage Premium : 1 x P20 | Cache = AUCUN, envisagez d’utiliser un disque Ultra Azure |
| Paramètre ASE MaxMemory | 90 % de RAM physique | 90 % de RAM physique | en supposant une instance unique |
| Nombre d’unités de sauvegarde | 16 | 16 | --- |
| Nombre et type de disques de sauvegarde | 4 | 4 | Utilisez LVM2/les espaces de stockage |


### <a name="backup--restore-considerations-for-sap-ase-on-azure"></a>Considérations relatives à la sauvegarde et à la restauration pour SAP ASE sur Azure
L’augmentation du nombre d’unités de données et de sauvegarde améliore les performances de sauvegarde et de restauration. Il est recommandé de distribuer les disques Azure qui hébergent l’unité de sauvegarde SAP ASE comme indiqué dans les tableaux affichés précédemment. Veillez à équilibrer le nombre d’unités de sauvegarde et de disques, et à vous assurer que le débit de sauvegarde ne doit pas dépasser 40 % à 50 % du quota total de débit de machine virtuelle. Il est recommandé d’utiliser la compression de sauvegarde SAP par défaut. Pour plus d’informations, consultez les articles suivants :

- [Note de support SAP n° 1588316](https://launchpad.support.sap.com/#/notes/1588316)
- [Note de support SAP n° 1801984](https://launchpad.support.sap.com/#/notes/1801984)
- [Note de support SAP n° 1585981](https://launchpad.support.sap.com/#/notes/1585981) 

N’utilisez pas le lecteur D:\ ni l’espace /temp comme destination de vidage du journal ou de la base de données.

### <a name="impact-of-database-compression"></a>Impact de la compression de base de données
Dans les configurations pour lesquelles la bande passante d’E/S peut devenir un facteur de limitation, les mesures qui réduisent le nombre d’E/S par seconde peuvent contribuer à étirer la charge de travail exécutable dans un scénario IaaS comme Azure. Par conséquent, il est recommandé de vérifier que la compression SAP ASE est utilisée avant le chargement d’une base de données SAP existante dans Azure.

Nous recommandons d’appliquer la compression avant le chargement sur Azure pour plusieurs raisons :

* La quantité de données à charger dans Azure est moins importante.
* En supposant que l’on peut utiliser en local un matériel plus performant, avec plus d’UC, une bande passante d’E/S supérieure ou une latence d’E/S inférieure, la durée d’exécution de la compression est plus courte.
* Des bases de données plus petites peuvent permettre de diminuer les coûts liés à l’allocation de disque.

La compression des données et des éléments LOB fonctionne au sein d’une machine virtuelle hébergée dans Azure Virtual Machines comme en local. Pour savoir comment vérifier si la compression est déjà utilisée dans une base de données SAP ASE existante, consultez [Note de support SAP n° 1750510](https://launchpad.support.sap.com/#/notes/1750510). Pour plus d’informations sur la compression de base de données SAP ASE, consultez [Note de support SAP n° 2121797](https://launchpad.support.sap.com/#/notes/2121797)

## <a name="high-availability-of-sap-ase-on-azure"></a>Haute disponibilité de SAP ASE sur Azure 
Le Guide de l’utilisateur du HADR décrit l’installation et la configuration d’une solution SAP ASE « Always On » à 2 nœuds.  De plus, un troisième nœud de reprise d’activité est également pris en charge. SAP ASE prend en charge de nombreuses configurations à haute disponibilité, notamment les disques partagés et le clustering de système d’exploitation natif (adresse IP flottante). La seule configuration prise en charge sur Azure est d’utiliser le gestionnaire d’incidents sans adresse IP flottante.  La méthode d’adresse IP flottante ne fonctionnera pas sur Azure.  Le noyau SAP est une application qui prend en charge la haute disponibilité et connaît les serveurs SAP ASE principaux et secondaires. Il n’existe aucune intégration étroite entre SAP ASE et Azure, l’équilibreur de charge interne Azure n’est pas utilisé. Par conséquent, la documentation SAP ASE standard doit être suivie à partir du [Guide de l’utilisateur du HADR SAP ASE](https://help.sap.com/viewer/efe56ad3cad0467d837c8ff1ac6ba75c/16.0.3.7/en-US/a6645e28bc2b1014b54b8815a64b87ba.html) 

> [!NOTE]
> La seule configuration prise en charge sur Azure est d’utiliser le gestionnaire d’incidents sans adresse IP flottante.  La méthode d’adresse IP flottante ne fonctionnera pas sur Azure. 

### <a name="third-node-for-disaster-recovery"></a>Troisième nœud pour la reprise d’activité
Au-delà de l’utilisation de SAP ASE Always On pour la haute disponibilité locale, vous souhaiterez peut-être étendre la configuration à un nœud répliqué de manière asynchrone dans une autre région Azure. Vous trouverez la documentation pour un tel scénario [ici](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/installation-procedure-for-sybase-16-3-patch-level-3-always-on/ba-p/368199).

## <a name="sap-ase-database-encryption--ssl"></a>Chiffrement de la base de données SAP ASE et SSL 
L’outil SAP SWPM (Software Provisioning Manager) donne la possibilité de chiffrer la base de données lors de l’installation.  Si vous souhaitez utiliser le chiffrement, il est recommandé d’utiliser le chiffrement complet de la base de données SAP.  Pour plus d’informations, consultez :

- [Note de support SAP n° 2556658](https://launchpad.support.sap.com/#/notes/2556658)
- [Note de support SAP n° 2224138](https://launchpad.support.sap.com/#/notes/2224138)
- [Note de support SAP n° 2401066](https://launchpad.support.sap.com/#/notes/2401066)
- [Note de support SAP n° 2593925](https://launchpad.support.sap.com/#/notes/2593925) 

> [!NOTE]
> Si une base de données SAP ASE est chiffrée, la compression de vidage de sauvegarde ne fonctionnera pas. Voir aussi [Note de support SAP n° 2680905](https://launchpad.support.sap.com/#/notes/2680905) 

## <a name="sap-ase-on-azure-deployment-checklist"></a>Check-list relative au déploiement de SAP ASE sur Azure
 
- Déployer SAP ASE 16.0.03.07 ou version ultérieure
- Procéder à la mise à jour vers la version et les correctifs les plus récents de FaultManager et SAPHostAgent
- Déployer sur le dernier système d’exploitation certifié disponible, par exemple Windows 2019, Suse 15.1 ou Redhat 7.6, ou version ultérieure
- Utiliser des machines virtuelles certifiées SAP : des références SKU de machine virtuelle Azure mémoire haute comme Es_v3 ou des références SKU de machine virtuelle de série M pour des systèmes x-large sont recommandées
- Associer les E/S par seconde de disque et le quota de débit agrégé total de la machine virtuelle avec la conception du disque.  Déployer un nombre suffisant de disques
- Agréger des disques à l’aide d’espaces de stockage Windows ou de Linux LVM2 avec une taille de bande et un système de fichiers corrects
- Créer un nombre suffisant d’unités pour les données, les journaux, les fichiers temporaires et les sauvegardes
- Envisager d’utiliser UltraDisk pour les systèmes x-large 
- Exécuter `saptune` SAP-ASE sur le système d’exploitation Linux 
- Sécuriser la base de données avec le chiffrement de la base de données, stocker manuellement les clés dans Azure Key Vault 
- Suivre la [check-list relative à SAP sur Azure](./sap-deployment-checklist.md) 
- Configurer la sauvegarde de fichier journal et la sauvegarde complète 
- Tester la haute disponibilité/reprise d’activité, la sauvegarde et la restauration, et exécuter un test de contrainte et de volume 
- Confirmer le fonctionnement de l’extension automatique de la base de données 

## <a name="using-dbacockpit-to-monitor-database-instances"></a>Utilisation de DBACockpit pour superviser les instances de base de données
Pour les systèmes SAP utilisant SAP ASE en tant que plateforme de base de données, la fonction DBACockpit est accessible sous forme de fenêtres de navigateur intégrées dans Transaction DBACockpit ou via Webdynpro. Toutefois, l’ensemble des fonctionnalités de supervision et d’administration de la base de données sont uniquement disponibles dans l’implémentation Webdynpro de DBACockpit.

À l’instar des systèmes locaux, plusieurs étapes sont nécessaires pour activer toutes les fonctionnalités de SAP NetWeaver utilisées par l’implémentation Webdynpro de DBACockpit. Pour activer l’utilisation de Webdynpro et générer les éléments nécessaires, consultez [Note de support SAP n° 1245200](https://launchpad.support.sap.com/#/notes/1245200). Quand vous suivez les instructions fournies dans les notes ci-dessus, vous configurez également Internet Communication Manager (`ICM`), ainsi que les ports à utiliser pour les connexions http et https. Pour http, la configuration par défaut ressemble à :

> icm/server_port_0 = PROT=HTTP,PORT=8000,PROCTIMEOUT=600,TIMEOUT=600
> 
> icm/server_port_1 = PROT=HTTPS,PORT=443$$,PROCTIMEOUT=600,TIMEOUT=600
> 
> 

Quant aux liens générés dans Transaction DBACockpit, ils ressemblent à :

> https:\//\<fullyqualifiedhostname>:44300/sap/bc/webdynpro/sap/dba_cockpit
> 
> http:\//\<fullyqualifiedhostname>:8000/sap/bc/webdynpro/sap/dba_cockpit
> 
> 

En fonction de la connexion de la machine virtuelle Azure hébergeant le système SAP aux instances AD et DNS, vous devez vous assurer qu’ICM utilise un nom d’hôte complet qui peut être résolu sur la machine à partir de laquelle vous essayez d’ouvrir DBACockpit. Consultez [Note de support SAP n° 773830](https://launchpad.support.sap.com/#/notes/773830) pour comprendre comment ICM détermine le nom d’hôte complet en fonction des paramètres de profil, puis définissez le paramètre icm/host_name_full explicitement si nécessaire.

Si vous avez déployé la machine virtuelle dans un scénario de cloud uniquement sans connectivité intersite entre le site local et Azure, vous devez définir une adresse IP publique et une étiquette `domainlabel`. Le format du nom DNS public de la machine virtuelle ressemble à :

> `<custom domainlabel`&gt;.`<azure region`&gt;.cloudapp.azure.com
> 
> 

Vous trouverez plus d’informations sur le nom DNS [ici][virtual-machines-azurerm-versus-azuresm].

Lorsque vous définissez le paramètre de profil SAP icm/host_name_full sur le nom DNS de la machine virtuelle Azure, le lien peut ressembler à ceci :

> https:\//mydomainlabel.westeurope.cloudapp.net:44300/sap/bc/webdynpro/sap/dba_cockpit
> 
> http:\//mydomainlabel.westeurope.cloudapp.net:8000/sap/bc/webdynpro/sap/dba_cockpit

Dans ce cas, vous devez veiller à :

* Ajouter des règles de trafic entrant au groupe de sécurité réseau dans le portail Azure pour les ports TCP/IP utilisés afin de communiquer avec ICM
* ajouter des règles de trafic entrant à la configuration du pare-feu Windows pour les ports TCP/IP utilisés afin de communiquer avec ICM.

Pour effectuer une importation automatisée de toutes les corrections disponibles, il est recommandé d’appliquer périodiquement la note SAP de collection de correction applicable à votre version SAP :

* [Note de support SAP n° 1558958](https://launchpad.support.sap.com/#/notes/1558958)
* [Note de support SAP n° 1619967](https://launchpad.support.sap.com/#/notes/1619967)
* [Note de support SAP n° 1882376](https://launchpad.support.sap.com/#/notes/1882376)

Vous trouverez plus d’informations sur DBA Cockpit pour SAP ASE dans les notes SAP suivantes :

* [Note de support SAP n° 1605680](https://launchpad.support.sap.com/#/notes/1605680)
* [Note de support SAP n° 1757924](https://launchpad.support.sap.com/#/notes/1757924)
* [Note de support SAP n° 1757928](https://launchpad.support.sap.com/#/notes/1757928)
* [Note de support SAP n° 1758182](https://launchpad.support.sap.com/#/notes/1758182)
* [Note de support SAP n° 1758496](https://launchpad.support.sap.com/#/notes/1758496)    
* [Note de support SAP n° 1814258](https://launchpad.support.sap.com/#/notes/1814258)
* [Note de support SAP n° 1922555](https://launchpad.support.sap.com/#/notes/1922555)
* [Note de support SAP n° 1956005](https://launchpad.support.sap.com/#/notes/1956005)


## <a name="useful-links-notes--whitepapers-for-sap-ase"></a>Liens utiles, notes et livres blancs pour SAP ASE
La première page de la [documentation SAP ASE 16.0.03.07](https://help.sap.com/viewer/product/SAP_ASE/16.0.3.7/en-US) fournit des liens vers différents documents, parmi lesquels :

- SAP ASE Learning Journey - Administration & Monitoring
- SAP ASE Learning Journey - Installation & Upgrade

s’avèrent particulièrement utiles. Un autre document utile est [SAP Applications on SAP Adaptive Server Enterprise Best Practices for Migration and Runtime](https://assets.cdn.sap.com/sapcom/docs/2016/06/26450353-767c-0010-82c7-eda71af511fa.pdf).

Voici d’autres notes de support SAP utiles :

- [Note de support SAP n° 2134316](https://launchpad.support.sap.com/#/notes/2134316) 
- [Note de support SAP n° 1748888](https://launchpad.support.sap.com/#/notes/1748888) 
- [Note de support SAP n° 2588660](https://launchpad.support.sap.com/#/notes/2588660) 
- [Note de support SAP n° 1680803](https://launchpad.support.sap.com/#/notes/1680803) 
- [Note de support SAP n° 1724091](https://launchpad.support.sap.com/#/notes/1724091) 
- [Note de support SAP n° 1775764](https://launchpad.support.sap.com/#/notes/1775764) 
- [Note de support SAP n° 2162183](https://launchpad.support.sap.com/#/notes/2162183) 
- [Note de support SAP n° 1928533](https://launchpad.support.sap.com/#/notes/1928533)
- [Note de support SAP n° 2015553](https://launchpad.support.sap.com/#/notes/2015553)
- [Note de support SAP n° 1750510](https://launchpad.support.sap.com/#/notes/1750510) 
- [Note de support SAP n° 1752266](https://launchpad.support.sap.com/#/notes/1752266) 
- [Note de support SAP n° 2162183](https://launchpad.support.sap.com/#/notes/2162183) 
- [Note de support SAP n° 1588316](https://launchpad.support.sap.com/#/notes/158831) 

D’autres informations sont publiées sur 

- [SAP Applications on SAP Adaptive Server Enterprise](https://community.sap.com/topics/applications-on-ase)
- [Centre d'informations SAP ASE](http://infocenter.sybase.com/help/index.jsp) 
- [SAP ASE Always-on avec configuration du nœud DR tiers](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/installation-procedure-for-sybase-16-3-patch-level-3-always-on/ba-p/368199)

Un bulletin d’informations mensuel est publié via [Note de support SAP n° 2381575](https://launchpad.support.sap.com/#/notes/2381575) 


## <a name="next-steps"></a>Étapes suivantes
Consultez l’article [Check-list relative à la planification et au déploiement de la charge de travail SAP sur Azure](./sap-deployment-checklist.md)