---
title: Facteurs à prendre en compte pour le déploiement SGBD des machines virtuelles Azure pour la charge de travail SAP | Microsoft Docs
description: Facteurs à prendre en compte pour le déploiement SGBD des machines virtuelles Azure pour la charge de travail SAP
author: msjuergent
ms.service: virtual-machines
ms.subservice: workloads
ms.topic: article
ms.date: 09/20/2020
ms.author: juergent
ms.reviewer: cynthn
ms.openlocfilehash: 5638b60b60f3952ab40fa000e457082f4748ac35
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96484343"
---
# <a name="considerations-for-azure-virtual-machines-dbms-deployment-for-sap-workload"></a>Facteurs à prendre en compte pour le déploiement SGBD des machines virtuelles Azure pour la charge de travail SAP
[1114181]:https://launchpad.support.sap.com/#/notes/1114181
[1409604]:https://launchpad.support.sap.com/#/notes/1409604
[1597355]:https://launchpad.support.sap.com/#/notes/1597355
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2039619]:https://launchpad.support.sap.com/#/notes/2039619
[2069760]:https://launchpad.support.sap.com/#/notes/2069760
[2171857]:https://launchpad.support.sap.com/#/notes/2171857
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2233094]:https://launchpad.support.sap.com/#/notes/2233094
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[deployment-guide]:deployment-guide.md
[deployment-guide-3]:deployment-guide.md#b3253ee3-d63b-4d74-a49b-185e76c4088e
[planning-guide]:planning-guide.md

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png



Ce guide fait partie de la documentation sur l’implémentation et le déploiement des logiciels SAP sur Microsoft Azure. Avant de lire ce guide, consultez le [Guide de planification et d’implémentation][planning-guide] et les articles proposés par le guide de planification. Ce document décrit les aspects généraux du déploiement de systèmes SGBD de type SAP sur des machines virtuelles Microsoft Azure à l’aide des fonctionnalités Azure IaaS.

Ce document vient compléter la documentation sur l’installation SAP et les notes SAP, qui constituent les principales ressources à consulter pour installer et déployer des logiciels SAP sur des plateformes.

Ce document aborde l’exécution des systèmes SGBD de type SAP sur les machines virtuelles Azure. Il comporte peu de références à des systèmes SGBD spécifiques. Les systèmes SGBD spécifiques sont couverts dans la suite de ce document.

## <a name="definitions"></a>Définitions
Le document emploie les termes suivants :

* **IaaS** : Infrastructure as a Service.
* **PaaS** : Platform as a Service.
* **SaaS** : Software as a Service.
* **Composant SAP** : application SAP individuelle, comme ERP Central Component (ECC), Business Warehouse (BW), Solution Manager ou Enterprise Portal (EP). Les composants SAP peuvent reposer sur des technologies ABAP ou Java traditionnelles ou sur une application non basée sur NetWeaver telle que Business Objects.
* **Environnement SAP** : un ou plusieurs composants SAP regroupés de manière logique pour exécuter une fonction métier, comme le développement, l’assurance qualité, la formation, la reprise d’activité après sinistre ou la production.
* **Paysage SAP** : ce terme fait référence à l’ensemble des ressources SAP dans le paysage informatique d’un client. Le paysage SAP comprend tous les environnements de production et hors production.
* **Système SAP** : combinaison d’une couche SGBD et d’une couche Application d’un système de développement SAP ERP, d’un système de test SAP Business Warehouse ou d’un système de production SAP CRM, par exemple. Dans les déploiements Azure, la séparation de ces deux couches entre les sites locaux et Azure n’est pas prise en charge. Par conséquent, un système SAP donné est déployé soit localement, soit dans Azure. Les différents systèmes d’un paysage SAP peuvent être déployés dans Azure ou en local. Par exemple, vous pouvez déployer les systèmes de développement et de test SAP CRM dans Azure, mais déployer le système de production SAP CRM en local.
* **Entre différents locaux** : décrit un scénario dans lequel les machines virtuelles sont déployées dans un abonnement Azure qui fournit une connectivité de site à site, multisite ou Azure ExpressRoute entre les centres de données locaux et Azure. Dans la documentation Azure courante, ces types de déploiements sont également décrits comme des scénarios intersites. 

    La connexion a pour but d’étendre les domaines locaux, les instances locales d’Active Directory et le serveur DNS local à Azure. Le paysage local est étendu aux ressources Azure de l’abonnement. Grâce à cette extension, les machines virtuelles peuvent faire partie du domaine local. Les utilisateurs du domaine local peuvent accéder aux serveurs et exécuter des services sur ces machines virtuelles, tels que les services SGBD. La communication et la résolution de noms entre les machines virtuelles déployées en local et les machines virtuelles déployées dans Azure est possible. Ce scénario est le scénario le plus courant pour déployer des ressources SAP sur Azure. Pour plus d’informations, consultez [Planification et conception de la passerelle VPN](../../../vpn-gateway/vpn-gateway-about-vpngateways.md).

> [!NOTE]
> Les déploiements entre différents locaux de systèmes SAP s’appliquent quand les machines virtuelles Azure qui exécutent des systèmes SAP font partie d’un domaine local et sont prises en charge par les systèmes SAP de production. Les configurations intersites sont prises en charge pour le déploiement d’éléments ou de l’intégralité des paysages SAP dans Azure. Ces machines virtuelles doivent faire partie d’un domaine et de services Active Directory/LDAP locaux même lorsque l’intégralité du paysage SAP est exécutée dans Azure. 
>
> Dans les versions précédentes de la documentation, les scénarios hybrides étaient mentionnés. Le terme *hybride* tient au fait qu’il existe une connectivité entre locaux entre les sites locaux et Azure. Dans ce cas, « hybride » signifie également que les machines virtuelles dans Azure font partie du service Active Directory local.
>
>

Certaines documentations Microsoft décrivent les scénarios entre différents locaux un peu différemment, en particulier pour les configurations SGBD à haute disponibilité. Dans les documents portant sur SAP, le scénario de déploiement entre différents locaux se résume à l’établissement d’une connectivité [ExpressRoute](https://azure.microsoft.com/services/expressroute/) privée ou de site à site et à la répartition du paysage SAP entre les sites locaux et Azure.

## <a name="resources"></a>Ressources
D’autres articles sont disponibles sur la charge de travail SAP sur Azure. Commencez par lire [Charge de travail SAP sur Azure : bien démarrer](./get-started.md), puis choisissez le domaine qui vous intéresse.

Les notes SAP suivantes concernent SAP sur Azure et s’appliquent au domaine traité dans ce document.

| Numéro de la note | Intitulé |
| --- | --- |
| [1928533] |Applications SAP sur Azure : Produits et types de machines virtuelles pris en charge |
| [2015553] |SAP sur Microsoft Azure : Prérequis pour la prise en charge |
| [1999351] |Résolution des problèmes de la surveillance Azure améliorée pour SAP |
| [2178632] |Métriques de surveillance clés pour SAP sur Microsoft Azure |
| [1409604] |Virtualisation sur Windows : Surveillance améliorée |
| [2191498] |SAP sur Linux avec Azure : Surveillance améliorée |
| [2039619] |Applications SAP sur Microsoft Azure avec la base de données Oracle : Produits et versions pris en charge |
| [2233094] |DB6 : Applications SAP sur Azure avec IBM DB2 pour Linux, UNIX et Windows : Informations supplémentaires |
| [2243692] |Linux sur machine virtuelle Microsoft Azure (IaaS) : problèmes de licence SAP |
| [1984787] |SUSE LINUX Enterprise Server 12 Notes d'installation |
| [2002167] |Red Hat Enterprise Linux 7.x : Installation et mise à niveau |
| [2069760] |Installation et mise à niveau de SAP pour Oracle Linux 7.x |
| [1597355] |Recommandations relatives à l’espace d’échange pour Linux |
| [2171857] |Oracle Database 12c : Prise en charge du système de fichiers sur Linux |
| [1114181] |Oracle Database 11g : Prise en charge du système de fichiers sur Linux |


Pour plus d’informations sur l’ensemble des notes SAP pour Linux, consultez le [Wiki de la communauté SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes).

Vous devez avoir une connaissance pratique de l’architecture Microsoft Azure ainsi que du déploiement et du fonctionnement des machines virtuelles Microsoft Azure. Pour plus d’informations, consultez la [documentation Azure](../../../index.yml).

En règle générale, les processus d’installation et de configuration sur Windows, Linux ou SGBD sont globalement les mêmes que pour une machine virtuelle ou un système nu que vous installez en local. Les décisions relatives à l’implémentation de l’architecture et de la gestion des systèmes diffèrent sur certains points lorsque vous utilisez Azure IaaS. Ce document explique les différences spécifiques de l’architecture et de la gestion des systèmes que vous devez prendre en compte quand vous utilisez Azure IaaS.


## <a name="storage-structure-of-a-vm-for-rdbms-deployments"></a><a name="65fa79d6-a85f-47ee-890b-22e794f51a64"></a>Structure du stockage d’une machine virtuelle pour les déploiements SGBDR
Pour suivre le présent chapitre, lisez d’abord attentivement :

- [Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver](./planning-guide.md)
- [Types de stockage Azure pour une charge de travail SAP](./planning-guide-storage.md)
- [Logiciels SAP pris en charge pour les déploiements Azure](./sap-supported-product-on-azure.md)
- [Charge de travail SAP sur les machines virtuelles Azure - Scénarios pris en charge](./sap-planning-supported-configurations.md) 

Vous devez comprendre et connaître les différentes séries de machines virtuelles ainsi que les différences entre le stockage Standard et Premium. 

Pour Azure Block Storage, l’utilisation de disques managés Azure est fortement recommandée. Pour plus d’informations sur les disques managés Azure, consultez l’article [Introduction aux disques managés pour les machines virtuelles Azure](../../managed-disks-overview.md).

Dans une configuration de base, nous privilégions généralement une structure de déploiement dans laquelle le système d’exploitation, le SGBD et les fichiers binaires SAP éventuels sont séparés des fichiers de base de données. En modifiant les recommandations précédentes, nous vous conseillons de disposer de disques Azure distincts pour les ressources suivantes :

- Système d’exploitation (disque dur virtuel de base ou disque dur virtuel de système d’exploitation)
- Exécutables du système de gestion de base de données
- Exécutables SAP comme /usr/SAP

Une configuration qui sépare ces composants dans trois disques Azure différents peut entraîner une plus grande résilience puisque les écritures du journal ou du fichier de sauvegarde trop nombreuses résultant de l’activité des exécutables du SGBD ou SAP n’interfèrent pas avec les quotas de disque du disque du système d’exploitation. 

Les fichiers de données SGBD et les fichiers journaux de transactions/restaurations sont stockés dans le stockage par blocs pris en charge par Azure ou Azure NetApp Files. Ils sont stockés sur des disques distincts et attachés en tant que disques logiques à la machine virtuelle image du système d’exploitation Azure d’origine. Pour les déploiements Linux, différentes recommandations sont documentées, en particulier pour SAP HANA. Lisez l’article [Types de stockage Azure pour la charge de travail SAP](./planning-guide-storage.md) sur les fonctionnalités et la prise en charge des différents types de stockage pour votre scénario. 

Quand vous planifiez la disposition des disques, essayez de trouver le meilleur compromis entre ces éléments :

* Le nombre de fichiers de données.
* Le nombre de disques qui contiennent les fichiers
* Les quotas d’IOPS pour un même disque ou un même partage NFS.
* Le débit de données par disque ou partage NFS.
* Le nombre de disques supplémentaires possibles par taille de machine virtuelle
* Le débit global de stockage ou de réseau qu’une machine virtuelle peut fournir.
* Le temps de latence entre les différents types de stockage Azure.
* Contrats SLA de machine virtuelle.

Azure applique un quota IOPS pour chaque disque de données ou partage NFS. Ces quotas sont différents pour les disques hébergés sur les solutions ou partages différents de stockage de bloc Azure. La latence d’E/S est également différente entre ces différents types de stockage. 

Pour chaque type de machine virtuelle, vous ne pouvez attacher qu’un nombre limité de disques de données. Une autre restriction est que seuls certains types de machine virtuelle peuvent utiliser le stockage Premium, par exemple. En règle générale, vous choisissez le type de machine virtuelle à utiliser en fonction des exigences en mémoire et processeur. Vous pouvez également prendre en compte les exigences en IOPS, latence et débit de disque, auxquelles permettent généralement de répondre l’augmentation du nombre de disques ou le choix du type de stockage Premium. Le nombre d’IOPS et le débit devant être fourni par chaque disque sont des facteurs déterminants pour le choix de la taille des disques, en particulier avec le stockage Premium.

> [!NOTE]
> Pour les déploiements SGBD, nous recommandons le stockage Premium Azure, les partages NFS basés sur les disques Ultra ou Azure NetApp Files (exclusivement pour SAP HANA) pour les fichiers de données, les journaux de transactions ou les fichiers de restauration par progression. Peu importe que vous vouliez déployer des systèmes dans des environnements de production ou hors production.

> [!NOTE]
> Pour bénéficier du [contrat SLA de machine virtuelle unique](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/) d’Azure, tous les disques attachés doivent être du type de stockage Premium ou Azure Ultra, qui inclut le disque dur virtuel de base (stockage Premium Azure).

> [!NOTE]
> Les fichiers principaux des bases de données SAP (comme les fichiers de données et les fichiers journaux) ne peuvent pas être hébergés sur du matériel de stockage qui se trouve dans des centres de données tiers colocalisés adjacents aux centres de données Azure. Dans ce cas d’usage, le stockage fourni via les appliances logicielles hébergées sur des machines virtuelles Azure n’est pas non plus pris en charge. Pour les charges de travail de SGBD SAP, seul le stockage représenté en tant que service Azure natif est pris en charge pour les fichiers de données et les fichiers journaux de transactions des bases de données SAP en général. Les types de stockage Azure pris en charge peuvent être différents selon les SGBD. Pour plus d’informations, consultez l’article [Types de stockage Azure pour une charge de travail SAP](./planning-guide-storage.md)

L’emplacement des fichiers de base de données, des fichiers journaux et des fichiers de restauration ainsi que le type de stockage Azure utilisé sont dictés par les exigences en IOPS, en latence et en débit. Pour répondre aux exigences en terme d’IOPS dans le cas du stockage Premium Azure, il se peut que vous soyez obligé d’utiliser plusieurs disques ou alors un disque de stockage Premium de plus grande taille. Si vous utilisez plusieurs disques, créez une bande logicielle sur les disques qui contiennent les fichiers de données ou les fichiers journaux et de restauration. Dans ce cas, les contrats SLA relatifs aux IOPS et au débit des disques de stockage Premium sous-jacents ou la limite maximale d’IOPS des disques de stockage Standard se cumulent pour la bande ainsi créée.

Si vos exigences en IOPS dépassent ce qu’un seul disque dur virtuel peut fournir, répartissez le nombre d’IOPS nécessaires pour les fichiers de base de données entre plusieurs disques durs virtuels. Le moyen le plus simple pour distribuer la charge IOPS entre les disques consiste à créer une bande logicielle sur les différents disques. Ensuite, vous placez un certain nombre de fichiers de données du SGBD SAP sur les LUN issus de la bande logicielle. Le nombre de disques dans la bande est dicté par les exigences en IOPS, en débit de disque et en volume.


---
> ![Agrégation de stockage par bandes Windows][Logo_Windows] Windows
>
> Nous vous recommandons d’utiliser Windows Storage Spaces pour créer des agrégats par bandes sur plusieurs disques durs virtuels Azure. Utilisez Windows Server 2012 R2 ou Windows Server 2016 au minimum.
>
> ![Agrégation de stockage par bandes Linux][Logo_Linux] Linux
>
> Seuls MDADM et LVM (Logical Volume Manager) sont pris en charge pour créer un RAID logiciel sur Linux. Pour plus d'informations, consultez les pages suivantes :
>
> - [Configurer un RAID logiciel sur Linux](/previous-versions/azure/virtual-machines/linux/configure-raid) à l’aide de MDADM
> - [Configurer LVM sur une machine virtuelle Linux dans Azure](/previous-versions/azure/virtual-machines/linux/configure-lvm) à l’aide de LVM
>
>

---

Pour les disques Ultra Azure, l’entrelacement n’est pas nécessaire, car vous pouvez définir des IOPS et un débit de disque indépendant de la taille du disque.


> [!NOTE]
> Dans la mesure où le stockage Azure conserve trois images des disques durs virtuels, il est inutile de configurer une redondance si vous utilisez des agrégats par bandes. Vous devez uniquement configurer des agrégats par bandes afin que les E/S soient distribuées entre les différents disques durs virtuels.
>

### <a name="managed-or-nonmanaged-disks"></a>Disques managés ou non managés
Un compte de stockage Azure est une construction administrative qui, par ailleurs, fait également l’objet de limitations. Les limitations ne sont pas les mêmes pour les comptes de stockage Standard et les comptes de stockage Premium. Pour avoir des informations sur les fonctionnalités et les limitations, consultez [Objectifs de performance et de scalabilité du stockage Azure](../../../storage/common/scalability-targets-standard-account.md).

Pour le stockage Standard, n’oubliez pas qu’il y a une limite d’IOPS par compte de stockage. Reportez-vous à la ligne **Taux de requêtes maximal** dans l’article [Objectifs de performance et de scalabilité du stockage Azure](../../../storage/common/scalability-targets-standard-account.md). Il y a également une limite initiale du nombre de comptes de stockage par abonnement Azure. Répartissez les disques durs virtuels de l’environnement SAP le plus grand entre les différents comptes de stockage pour ne pas dépasser les limites de ces comptes. Ce travail peut s’avérer fastidieux quand vous avez des centaines de machines virtuelles avec plus d’un millier de disques durs virtuels.

Il n’est pas recommandé d’utiliser le stockage standard pour les déploiements SGBD conjointement avec une charge de travail SAP. Par conséquent, les références et les recommandations relatives au stockage standard sont limitées à ce bref [article](/archive/blogs/mast/configuring-azure-virtual-machines-for-optimal-storage-performance)

Afin d’éviter le travail d’administration lié à la planification et au déploiement de disques durs virtuels sur plusieurs comptes de stockage Azure, Microsoft a introduit la fonctionnalité [Disques managés Azure](https://azure.microsoft.com/services/managed-disks/) en 2017. Les disques managés sont disponibles pour le stockage Standard et le stockage Premium. Voici les principaux avantages des disques managés par rapport aux disques non managés :

- Avec des disques managés, Azure distribue les différents disques durs virtuels entre tous les comptes de stockage automatiquement, au moment du déploiement. De cette façon, les limites des comptes de stockage pour le volume de données, le débit d’E/S et les IOPS ne sont pas dépassées.
- Grâce aux disques managés, le stockage Azure respecte les concepts des groupes à haute disponibilité Azure. Si la machine virtuelle fait partie d’un groupe à haute disponibilité Azure, le disque dur virtuel de base et un disque attaché d’une machine virtuelle sont déployés dans différents domaines d’erreur et de mise à jour.


> [!IMPORTANT]
> Étant donné les avantages des disques managés Azure, nous vous recommandons vivement d’utiliser Azure Managed Disks pour vos déploiements SGBD et SAP en général.
>

Pour convertir des disques non managés en disques managés, consultez :

- [Convertir les disques non managés d’une machine virtuelle Windows en disques managés](../../windows/convert-unmanaged-to-managed-disks.md).
- [Convertir les disques non managés d’une machine virtuelle Linux en disques managés](../../linux/convert-unmanaged-to-managed-disks.md).


### <a name="caching-for-vms-and-data-disks"></a><a name="c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f"></a>Mise en cache pour les machines virtuelles et les disques de données
Lorsque vous montez des disques sur des machines virtuelles, vous pouvez choisir de mettre ou non en cache le trafic d’E/S entre la machine virtuelle et les disques situés dans le stockage Azure.

Les recommandations qui suivent s’appliquent à ces caractéristiques d’E/S dans les environnements SGBD standard :

- Il s’agit la plupart du temps d’une charge de travail de lecture dans les fichiers de données d’une base de données. Ces lectures sont un facteur critique des performances du système SGBD.
- Les points de contrôle ou un flux constant donnent lieu à des écritures en rafales dans les fichiers de données. Si l’on établit une moyenne quotidienne, les écritures sont moins nombreuses que les lectures. Contrairement aux lectures à partir de fichiers de données, ces écritures sont asynchrones et ne bloquent pas les transactions utilisateur.
- Il n’y a pratiquement pas de lectures à partir de fichiers journaux ou de fichiers de restauration. Les exceptions sont les E/S volumineuses quand vous effectuez des sauvegardes des journaux des transactions.
- Les écritures constituent la charge principale portant sur les fichiers journaux des transactions ou de restauration. Selon la nature de la charge de travail, la taille des E/S peut aller de 4 Ko à 1 Mo, voire plus.
- Toutes les écritures doivent être rendues persistantes sur le disque de manière fiable.

Pour le stockage Standard, les types de mise en cache possibles sont les suivants :

* None
* Lire
* Lecture/écriture

Pour obtenir des performances homogènes et déterministes, définissez la mise en cache dans le stockage Standard pour tous les disques contenant des fichiers de données, des fichiers journaux et des fichiers de restauration du SGBD, et définissez l’espace de table sur **AUCUN**. La valeur par défaut peut être conservée pour la mise en cache du disque dur virtuel de base.

Pour le stockage Premium Azure, les options de mise en cache suivantes sont disponibles :

* None
* Lire
* Lecture/écriture
* Aucun + Accélérateur d’écriture, option uniquement disponible pour les machines virtuelles Azure de série M
* Lecture + Accélérateur d’écriture, option uniquement disponible pour les machines virtuelles Azure de série M

Pour le stockage Premium, nous vous recommandons d’utiliser la **mise en cache de lecture pour les fichiers de données** de la base de données SAP et de choisir l’option avec laquelle **aucune mise en cache n’est effectuée pour les disques des fichiers journaux**.

Dans les déploiements de machines virtuelles de série M, nous vous recommandons d’utiliser l’Accélérateur d’écriture Azure pour votre déploiement SGBD. Pour en savoir plus sur les restrictions et le déploiement de l’Accélérateur d’écriture Azure, consultez [Activer l’Accélérateur d’écriture](../../how-to-enable-write-accelerator.md).

Aucune option de mise en cache n’est proposée pour le disque Ultra et Azure NetApp Files.


### <a name="azure-nonpersistent-disks"></a>Disques Azure non persistants
Les machines virtuelles Azure fournissent des disques non persistants après le déploiement d’une machine virtuelle. Si une machine virtuelle est redémarrée, tout le contenu de ces disques est effacé. Il est évident que les fichiers de données, les fichiers journaux et les fichiers de restauration ne doivent en aucun cas se trouver sur ces disques. Il peut exister des exceptions pour certaines bases de données, où ces disques non persistants sont parfois appropriés pour les espaces de stockage tempdb et temp. N’utilisez pas ces disques pour des machines virtuelles de série A, car ces disques non persistants sont limités en débit avec cette famille de machines virtuelles. 

Pour plus d’informations, consultez [Understand the temporary drive on Windows VMs in Azure](/archive/blogs/mast/understanding-the-temporary-drive-on-windows-azure-virtual-machines), qui décrit le disque temporaire sur les machines virtuelles Windows dans Azure.

---
> ![Disque non persistant Windows][Logo_Windows] Windows
>
> Sur une machine virtuelle Azure, le disque D est un disque non persistant soutenu par des disques locaux présents sur le nœud de calcul Azure. Comme le disque n’est pas persistant, toutes les modifications apportées au contenu du disque D sont perdues au redémarrage de la machine virtuelle. Les modifications incluent les fichiers stockés, les répertoires créés et les applications installées.
>
> ![Disque non persistant Linux][Logo_Linux] Linux
>
> Les machines virtuelles Linux dans Azure montent automatiquement un disque à l’emplacement /mnt/resource. Il s’agit d’un disque non persistant soutenu par des disques locaux présents sur le nœud de calcul Azure. Comme le disque n’est pas persistant, toutes les modifications apportées au contenu de l’emplacement /mnt/resource sont perdues au redémarrage de la machine virtuelle. Les modifications incluent les fichiers stockés, les répertoires créés et les applications installées.
>
>

---



### <a name="microsoft-azure-storage-resiliency"></a><a name="10b041ef-c177-498a-93ed-44b3441ab152"></a>Résilience du Stockage Microsoft Azure
Le stockage Microsoft Azure stocke le disque dur virtuel de base, avec le système d’exploitation et les disques ou objets blob associés, sur au moins trois nœuds de stockage distincts. Ce type de stockage est appelé stockage localement redondant (LRS). Il s’agit du type de stockage par défaut pour tous les stockages dans Azure.

Il existe d’autres méthodes de redondance. Pour plus d’informations, consultez l’article [Réplication de Stockage Azure](../../../storage/common/storage-redundancy.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json).

> [!NOTE]
> Stockage Premium Azure, disque Ultra et Azure NetApp Files (pour SAP HANA exclusivement) sont les types de stockage recommandés pour les machines virtuelles et les disques de SGBD hébergeant les fichiers de base de données et les fichiers journaux/de restauration. Le stockage localement redondant est la seule méthode pour ces types de stockage. Par conséquent, vous devez configurer les méthodes de base de données afin d’activer la réplication des données de base de données dans une autre région ou zone de disponibilité Azure. Les méthodes de base de données incluent SQL Server Always On, Oracle Data Guard et la réplication de système HANA.


> [!NOTE]
> Dans les déploiements SGBD, l’utilisation du stockage géoredondant (GRS) n’est pas recommandée pour le stockage Standard. Le stockage géoredondant diminue les performances et ne respecte pas l’ordre d’écriture sur les différents disques durs virtuels qui sont attachés à une machine virtuelle. Le non-respect de l’ordre d’écriture sur les disques durs virtuels risque potentiellement de créer des bases de données incohérentes du côté de la cible de réplication. Cette situation se produit si les fichiers de base de données ainsi que les fichiers journaux et de restauration sont répartis entre plusieurs disques durs virtuels, comme c’est généralement le cas, du côté de la machine virtuelle source.



## <a name="vm-node-resiliency"></a>Résilience des nœuds de machine virtuelle
Azure propose plusieurs contrats SLA différents pour les machines virtuelles. Pour plus d’informations, consultez la dernière mise à jour du [SLA pour Machines virtuelles](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/). La couche SGBD étant un facteur critique pour la disponibilité dans un système SAP, vous devez bien comprendre les concepts des groupes à haute disponibilité, des Zones de disponibilité et des événements de maintenance. Pour plus d’informations sur ces concepts, consultez [Gérer la disponibilité des machines virtuelles Windows dans Azure](../../manage-availability.md) et [Gérer la disponibilité des machines virtuelles Linux dans Azure](../../manage-availability.md).

Les recommandations minimales pour les scénarios de SGBD de production avec une charge de travail SAP sont les suivantes :

- Déployer deux machines virtuelles dans un groupe à haute disponibilité distinct dans la même région Azure.
- Exécutez ces deux machines virtuelles dans le même réseau virtuel Azure et attacher des cartes réseau issues des mêmes sous-réseaux.
- Utilisez les méthodes de base de données pour conserver un serveur de secours avec la deuxième machine virtuelle. Il peut s’agir des méthodes SQL Server Always On, Oracle Data Guard ou HANA System Replication.

Vous pouvez aussi déployer une troisième machine virtuelle dans une autre région Azure et utiliser les mêmes méthodes de base de données pour fournir un réplica asynchrone dans une autre région Azure.

Pour savoir comment configurer les groupes à haute disponibilité Azure, consultez [ce tutoriel](../../windows/tutorial-availability-sets.md).



## <a name="azure-network-considerations"></a>Considérations relatives au réseau Azure
Dans les déploiements SAP à grande échelle, utilisez le blueprint du [Centre de données virtuel Azure](/azure/architecture/vdc/networking-virtual-datacenter). Utilisez-le pour configurer votre réseau virtuel, et attribuer les autorisations et rôles réseau appropriés aux différentes parties de votre organisation.

Ces bonnes pratiques sont le résultat de centaines de déploiements clients :

- Les réseaux virtuels sur lesquels l’application SAP est déployée n’ont pas accès à Internet.
- Les machines virtuelles de base de données s’exécutent dans le même réseau virtuel que la couche application, séparées dans un sous-réseau différent de la couche application SAP.
- Les machines virtuelles du réseau virtuel ont une allocation statique de l’adresse IP privée. Pour plus d’informations, consultez [Types d’adresses IP et méthodes d’allocation dans Azure](../../../virtual-network/public-ip-addresses.md).
- Les restrictions de routage vers et depuis les machines virtuelles SGBD ne sont *pas* définies avec des pare-feu installés sur les machines virtuelles SGBD locales. À la place, le routage du trafic est défini avec des [groupes de sécurité réseau (NSG)](../../../virtual-network/network-security-groups-overview.md).
- Pour séparer et isoler le trafic à destination des machines virtuelles SGBD, attribuez des cartes réseau distinctes aux machines virtuelles. Chaque carte réseau obtient une adresse IP différente et est attribuée à un sous-réseau virtuel différent. Chaque sous-réseau a ses propres règles NSG. L’isolation ou la séparation du trafic réseau est une méthode de routage. Elle ne permet pas de définir des quotas de débit réseau.

> [!NOTE]
> Quand vous attribuez des adresses IP statiques dans Azure, vous devez les attribuer individuellement aux cartes réseau virtuelles. N’attribuez pas d’adresses IP statiques utilisées au sein du système d’exploitation invité à une carte réseau virtuelle. Certains services Azure, comme Sauvegarde Azure, s’appuient sur le fait qu’au moins la carte réseau virtuelle principale est configurée pour utiliser DHCP à la place d’adresses IP statiques. Pour plus d’informations, consultez [Résoudre les problèmes de sauvegarde des machines virtuelles Azure](../../../backup/backup-azure-vms-troubleshoot.md#networking). Pour attribuer plusieurs adresses IP statiques à une machine virtuelle, attribuez plusieurs cartes réseau virtuelles à une machine virtuelle.
>


> [!WARNING]
> Il n’est pas possible de configurer des [appliances virtuelles réseau](https://azure.microsoft.com/solutions/network-appliances/) dans le chemin de communication entre l’application SAP et la couche SGBD d’un système SAP NetWeaver, Hybris ou S/4HANA basé sur SAP. Cette restriction est implémentée pour des raisons de performances et de fonctionnalités. Le chemin de communication entre la couche Application SAP et la couche SGBD doit être direct. La restriction n’inclut pas les [règles NSG et règles de groupe de sécurité d’application (ASG)](../../../virtual-network/network-security-groups-overview.md) si ces règles ASG et NSG autorisent un chemin de communication direct. 
>
> Voici d’autres scénarios où les appliances réseau virtuelles ne sont pas prises en charge :
>
> * Chemins de communication entre les machines virtuelles Azure qui représentent les nœuds de cluster Linux Pacemaker et les appareils SBD comme décrit dans [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP](./high-availability-guide-suse.md).
> * Chemins de communication entre les machines virtuelles Azure et le serveur de fichiers SOFS Windows Server configurés comme décrit dans [Mettre en cluster une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide du partage de fichiers dans Azure](./sap-high-availability-guide-wsfc-file-share.md). 
>
> Les appliances réseau virtuelles dans les chemins de communication peuvent facilement doubler la latence du réseau entre deux partenaires de communication. Elles peuvent également limiter le débit dans les chemins critiques entre la couche Application SAP et la couche SGBD. Dans certains scénarios clients, les appliances réseau virtuelles sont susceptibles de provoquer des défaillances de clusters Pacemaker Linux. Cela se produit notamment si les communications entre les nœuds de cluster Linux Pacemaker communiquent avec l’appareil SBD associé par le biais d’une appliance réseau virtuelle.
>

> [!IMPORTANT]
> Une autre conception qui n’est *pas* prise en charge est la séparation de la couche Application SAP et de la couche SGBD sur des réseaux virtuels Azure distincts qui ne sont pas [appairés](../../../virtual-network/virtual-network-peering-overview.md) l’un avec l’autre. Nous vous recommandons de séparer la couche Application SAP et la couche SGBD en utilisant des sous-réseaux au sein d’un seul réseau virtuel Azure plutôt que plusieurs réseaux virtuels Azure. 
>
> Si vous choisissez de ne pas suivre cette recommandation et de séparer les deux couches sur des réseaux virtuels distincts, ces deux réseaux virtuels doivent être [appairés](../../../virtual-network/virtual-network-peering-overview.md). 
>
> N’oubliez pas que le trafic réseau entre deux réseaux virtuels Azure [appairés](../../../virtual-network/virtual-network-peering-overview.md) engage des coûts de transfert. Un énorme volume de données de plusieurs téraoctets est échangé entre la couche Application SAP et la couche SGBD. Vous risquez donc de voir vos coûts augmenter de façon significative si la couche Application SAP et la couche SGBD sont séparées sur deux réseaux virtuels Azure appairés.

Utilisez deux machines virtuelles pour le déploiement SGBD de production au sein d’un groupe à haute disponibilité Azure ou entre deux Zones de disponibilité Azure. Utilisez également un routage distinct pour la couche Application SAP et pour le trafic des opérations et de gestion vers les deux machines virtuelles SGBD. Consultez le graphique suivant :

![Diagramme de deux machines virtuelles dans deux sous-réseaux](./media/virtual-machines-shared-sap-deployment-guide/general_two_dbms_two_subnets.PNG)


### <a name="use-azure-load-balancer-to-redirect-traffic"></a>Utiliser Azure Load Balancer pour rediriger le trafic
L’emploi d’adresses IP virtuelles privées utilisées dans des fonctionnalités comme la réplication SQL Server Always On ou la réplication HANA System nécessite la configuration d’un équilibreur de charge Azure. L’équilibreur de charge utilise des ports probe pour déterminer le nœud SGBD actif et router le trafic exclusivement vers ce nœud de base de données actif. 

S’il y a basculement du nœud de base de données, la reconfiguration de l’application SAP est inutile. En fait, les architectures d’application SAP les plus courantes se reconnectent à l’adresse IP virtuelle privée. Quant à l’équilibreur de charge, il répond au basculement du nœud en redirigeant le trafic envoyé à l’adresse IP virtuelle privée vers le second nœud.

Azure fournit deux [références SKU d’équilibreur de charge](../../../load-balancer/load-balancer-overview.md) différentes : une référence SKU de base et une référence SKU standard. En fonction des avantages de l’installation et des fonctionnalités, vous devez utiliser la référence SKU standard d’Azure Load Balancer. L’un des grands avantages de la version standard de l’équilibreur de charge est que celui-ci ne reçoit pas de trafic de données routé.

Un exemple de configuration d’un équilibreur de charge interne figure dans l’article [Tutoriel : Configurer manuellement un groupe de disponibilité SQL Server sur des machines virtuelles Azure](../../../azure-sql/virtual-machines/windows/availability-group-manually-configure-tutorial.md#create-an-azure-load-balancer)

> [!NOTE]
> Il existe des différences de comportement entre la référence SKU de base et la référence SKU standard en ce qui concerne l'accès aux adresses IP publiques. Le contournement des restrictions de la référence SKU standard pour accéder aux adresses IP publiques est décrite dans le document [Connectivité des points de terminaison publics pour les machines virtuelles avec Azure Standard Load Balancer dans les scénarios de haute disponibilité SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).


### <a name="azure-accelerated-networking"></a>Mise en réseau accélérée Azure
Afin de réduire davantage la latence du réseau entre les machines virtuelles Azure, nous vous recommandons de choisir l’option [Mise en réseau accélérée Azure](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/). Utilisez cette fonctionnalité quand vous déployez des machines virtuelles Azure pour une charge de travail SAP, en particulier pour la couche Application SAP et la couche SGBD SAP.

> [!NOTE]
> La mise en réseau accélérée n’est pas prise en charge par tous les types de machines virtuelles. L’article précédent liste les types de machines virtuelles qui la prennent en charge.
>

---
> ![Mise en réseau accélérée Windows][Logo_Windows] Windows
>
> Pour savoir comment déployer des machines virtuelles avec mise en réseau accélérée sur Windows, consultez l’article [Créer une machine virtuelle Windows avec mise en réseau accélérée](../../../virtual-network/create-vm-accelerated-networking-powershell.md).
>
> ![Mise en réseau accélérée Linux][Logo_Linux] Linux
>
> Pour plus d’informations sur la distribution Linux, consultez [Créer une machine virtuelle Linux avec mise en réseau accélérée](../../../virtual-network/create-vm-accelerated-networking-cli.md).
>
>

---

> [!NOTE]
> SUSE, Red Hat et Oracle Linux prennent en charge la mise en réseau accélérée avec les versions récentes. Les versions plus anciennes comme SLES 12 SP2 ou RHEL 7.2 ne prennent pas en charge la mise en réseau accélérée Azure.
>


## <a name="deployment-of-host-monitoring"></a>Déploiement de la supervision d’hôte
Pour une utilisation en production des applications SAP sur des machines virtuelles Azure, SAP doit pouvoir collecter des données de supervision d’hôte auprès des hôtes physiques qui exécutent les machines virtuelles Azure. Un niveau de correctif logiciel SAP HostAgent spécifique est nécessaire pour activer cette fonctionnalité dans SAPOSCOL et SAP HostAgent. Le niveau de correctif logiciel exact est indiqué dans la Note de SAP [1409604].

Pour plus d’informations sur le déploiement de composants qui fournissent des données d’hôte à SAPOSCOL et à SAP Host Agent, et sur la gestion du cycle de vie de ces composants, consultez le [Guide de déploiement][deployment-guide].


## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur un système SGBD particulier, consultez :

- [Déploiement SGBD de machines virtuelles SQL Server Azure pour charge de travail SAP](dbms_guide_sqlserver.md)
- [Déploiement SGBD de machines virtuelles Oracle Azure pour charge de travail SAP](dbms_guide_oracle.md)
- [Déploiement SGBD de machines virtuelles IBM DB2 Azure pour charge de travail SAP](dbms_guide_ibm.md)
- [Déploiement SGBD de machines virtuelles SAP ASE Azure pour charge de travail SAP](dbms_guide_sapase.md)
- [Déploiement Content Server, LiveCache et SAP maxDB sur Azure](dbms_guide_maxdb.md)
- [Guide des opérations SAP HANA sur Azure](hana-vm-operations.md)
- [Haute disponibilité de SAP HANA pour les machines virtuelles Azure](sap-hana-availability-overview.md)
- [Guide de sauvegarde pour SAP HANA sur des machines virtuelles Azure](sap-hana-backup-guide.md)