---
title: Concevoir et implémenter une base de données Oracle sur Azure | Microsoft Docs
description: Concevez et implémentez une base de données Oracle dans votre environnement Azure.
author: dbakevlar
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.date: 08/02/2018
ms.author: kegorman
ms.reviewer: cynthn
ms.openlocfilehash: 5e9ddecd694a9051e746d07cbc1bee4d98bf5829
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96484428"
---
# <a name="design-and-implement-an-oracle-database-in-azure"></a>Concevoir et implémenter une base de données Oracle dans Azure

## <a name="assumptions"></a>Hypothèses

- Vous prévoyez de migrer une base de données Oracle locale vers Azure.
- Vous avez le [Pack Diagnostics](https://docs.oracle.com/cd/E11857_01/license.111/e11987/database_management.htm) pour la base de données Oracle que vous cherchez à migrer
- Vous comprenez les différentes métriques contenues dans les rapports Oracle AWR.
- Vous avez une connaissance élémentaire des performances d’application et de l’utilisation de la plateforme.

## <a name="goals"></a>Objectifs

- Comprendre comment optimiser votre déploiement Oracle sur Azure
- Explorer les options de réglage des performances d’une base de données Oracle dans un environnement Azure

## <a name="the-differences-between-an-on-premises-and-azure-implementation"></a>Différences entre une implémentation locale et une implémentation Azure 

Voici quelques éléments importants à prendre en compte lorsque vous migrez des applications locales vers Azure. 

L’une des principales différences est que, dans une implémentation Azure, les ressources, telles que les machines virtuelles, les disques et les réseaux virtuels, sont partagées avec les autres clients. Les ressources peuvent aussi être limitées en fonction des critères définis. Au lieu de se concentrer sur ce qui permet d’éviter un échec (temps moyen entre les défaillances), Azure est plus axé sur la survie à un échec (temps moyen de récupération).

Le tableau suivant liste certaines des différences qui existent entre une implémentation locale et une implémentation Azure d’une base de données Oracle.


|  | Implémentation locale | Implémentation d’Azure |
| --- | --- | --- |
| **Mise en réseau** |LAN/WAN  |SDN (Software-Defined Networking)|
| **Groupe de sécurité** |Outils de restriction d’adresse IP/de port |[Groupe de sécurité réseau](https://azure.microsoft.com/blog/network-security-groups) |
| **Résilience** |MTBF (temps moyen entre les défaillances) |MTTR (temps moyen de récupération)|
| **Maintenance planifiée** |Correctifs/mises à niveau|[Groupes à haute disponibilité](/previous-versions/azure/virtual-machines/windows/infrastructure-example) (correctifs/mises à niveau gérés par Azure) |
| **Ressource** |Dédié  |Partagée avec d’autres clients|
| **Régions** |Centres de données |[Paires de régions](../../regions.md#region-pairs)|
| **Stockage** |SAN/disques physiques |[Stockage géré par Azure](https://azure.microsoft.com/pricing/details/managed-disks/?v=17.23h)|
| **Mettre à l'échelle** |Mise à l’échelle verticale |Mise à l’échelle horizontale|


### <a name="requirements"></a>Spécifications

- Déterminez le taux de croissance et la taille de la base de données.
- Déterminez les E/S par seconde requises, que vous pouvez estimer en vous basant sur les rapports AWR Oracle ou d’autres outils de surveillance réseau.

## <a name="configuration-options"></a>Options de configuration

Quatre zones potentielles que vous pouvez paramétrer pour améliorer les performances dans un environnement Azure :

- Taille de la machine virtuelle
- Débit du réseau
- Types de disque et configurations
- Paramètres de cache des disques

### <a name="generate-an-awr-report"></a>Générer un rapport AWR

Si vous disposez d’une base de données Oracle que vous envisagez de migrer vers Azure, vous avez plusieurs options. Si vous avez le [Pack Diagnostics](https://www.oracle.com/technetwork/oem/pdf/511880.pdf) pour vos instances Oracle, vous pouvez exécuter le rapport AWR Oracle pour obtenir les métriques (E/S par seconde, Mbits/s, Gibits/s, etc.). Ensuite, choisissez la machine virtuelle en fonction des métriques que vous avez collectées. Vous pouvez également contacter votre équipe d’infrastructure pour obtenir des informations similaires.

Pour comparer, vous pouvez exécuter votre rapport AWR lors de charges de travail normales et maximales. En fonction de ces rapports, vous pouvez dimensionner les machines virtuelles selon la charge de travail moyenne ou maximale.

Voici un exemple montrant comment générer un rapport AWR (générez vos rapports AWR à l’aide d’Oracle Enterprise Manager, si votre installation actuelle en dispose) :

```bash
$ sqlplus / as sysdba
SQL> EXEC DBMS_WORKLOAD_REPOSITORY.CREATE_SNAPSHOT;
SQL> @?/rdbms/admin/awrrpt.sql
```

### <a name="key-metrics"></a>Mesures clés

Voici les métriques que vous pouvez obtenir avec le rapport AWR :

- Nombre total de cœurs
- Vitesse d’horloge du processeur
- Quantité totale de mémoire en Go
- Utilisation du processeur
- Taux maximal de transfert de données
- Taux de modification d’E/S (lecture/écriture)
- Taux d’enregistrement de la restauration par progression (Mbits/s)
- Débit du réseau
- Taux de latence réseau (élevée/basse)
- Taille de la base de données en Go
- Octets reçus par le biais de SQL*Net en provenance/à destination du client

### <a name="virtual-machine-size"></a>Taille de la machine virtuelle

#### <a name="1-estimate-vm-size-based-on-cpu-memory-and-io-usage-from-the-awr-report"></a>1. Estimer la taille de la machine virtuelle en fonction des données d’utilisation du processeur, de la mémoire ou des E/S du rapport AWR

Vous pouvez examiner les cinq principaux événements de premier plan expirés, qui indiquent l’emplacement des goulots d’étranglement.

Par exemple, dans le diagramme suivant, la synchronisation des fichiers journaux se trouve en haut. Cela indique le nombre d’attentes nécessaires pour que le LGWR écrive le tampon journal dans le fichier journal Redo. Ces résultats indiquent qu’un stockage ou des disques plus performants sont nécessaires. Le diagramme montre aussi le nombre de processeurs (cœurs) et la quantité de mémoire.

![Capture d’écran montrant la synchronisation des fichiers journaux en haut du tableau.](./media/oracle-design/cpu_memory_info.png)

Le diagramme suivant montre le nombre total d’E/S de lecture et d’écriture. 59 Go ont été lus et 247,3 Go ont été écrits au moment de la création du rapport.

![Capture d’écran montrant le nombre total d’E/S de lecture et d’écriture.](./media/oracle-design/io_info.png)

#### <a name="2-choose-a-vm"></a>2. Choisir une machine virtuelle

À partir des informations collectées dans le rapport AWR, l’étape suivante consiste à choisir une machine virtuelle de taille similaire qui répond à vos besoins. Vous trouverez la liste des machines virtuelles disponibles dans l’article [Mémoire optimisée](../../sizes-memory.md).

#### <a name="3-fine-tune-the-vm-sizing-with-a-similar-vm-series-based-on-the-acu"></a>3. Ajuster la taille de la machine virtuelle aux séries de machines virtuelles similaires en fonction de l’ACU

Une fois que vous avez choisi la machine virtuelle, faites attention à sa valeur ACU. Vous choisirez peut-être une autre machine virtuelle en fonction de la valeur ACU, qui répondra mieux à vos besoins. Pour plus d’informations, consultez [Unité de calcul Azure (ACU)](../../acu.md).

![Capture d’écran de la page Unités ACU](./media/oracle-design/acu_units.png)

### <a name="network-throughput"></a>Débit du réseau

Le diagramme suivant montre la relation entre le débit et le nombre d’E/S par seconde :

![Capture d’écran du débit](./media/oracle-design/throughput.png)

Le débit réseau total est estimé selon les informations suivantes  :
- Trafic SQL*Net
- Mbits/s x nombre de serveurs (flux sortant tel qu’Oracle Data Guard)
- Autres facteurs, comme la réplication de l’application

![Capture d’écran du débit SQL*Net](./media/oracle-design/sqlnet_info.png)

Selon vos besoins en bande passante réseau, vous pouvez choisir différents types de passerelles, notamment une passerelle de base, VpnGw ou Azure ExpressRoute. Pour plus d’informations, consultez [Tarification Passerelle VPN](https://azure.microsoft.com/pricing/details/vpn-gateway/?v=17.23h).

**Recommandations**

- La latence réseau est supérieure par rapport à un déploiement local. La diminution des allers-retours réseau peut considérablement améliorer les performances.
- Pour réduire le nombre d’allers-retours, regroupez les applications avec un nombre élevé de transactions et les applications « bavardes » sur une même machine virtuelle.
- Utilisez des machines virtuelles avec la [mise en réseau accélérée](../../../virtual-network/create-vm-accelerated-networking-cli.md) pour améliorer les performances réseau.
- Pour certaines distributions Linux, pensez à activer la [prise en charge de TRIM/UNMAP](/previous-versions/azure/virtual-machines/linux/configure-lvm#trimunmap-support).
- Installez [Oracle Enterprise Manager](https://www.oracle.com/technetwork/oem/enterprise-manager/overview/index.html) sur une machine virtuelle distincte.
- Les Huge Pages ne sont pas activées par défaut sur Linux. Envisagez d’activer les Huge Pages et définissez `use_large_pages = ONLY` sur Oracle DB. Cela peut aider à améliorer les performances. Des informations supplémentaires sont disponibles [ici](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/refrn/USE_LARGE_PAGES.html#GUID-1B0F4D27-8222-439E-A01D-E50758C88390).

### <a name="disk-types-and-configurations"></a>Types de disque et configurations

- *Disques de système d'exploitation par défaut* : ces types de disques permettent la persistance et la mise en cache des données. Ils sont optimisés pour un accès au système d’exploitation au moment du démarrage, mais ils ne sont pas conçus pour les charges de travail transactionnelles ou d’entrepôt de données (analytiques).

- *Disques non managés* : ces types de disques permettent de gérer les comptes de stockage qui stockent les fichiers de disque dur virtuel (VHD) correspondant à vos disques de machine virtuelle. Les fichiers VHD sont stockés en tant qu’objets blob de pages dans les comptes de stockage Azure.

- *Disques managés* : Azure gère les comptes de stockage que vous utilisez pour vos disques de machine virtuelle. Vous spécifiez le type de disque (Premium ou Standard) et la taille de disque dont vous avez besoin. Azure crée et gère le disque pour vous.

- *Disques de stockage Premium* : ces types de disques conviennent essentiellement aux charges de travail de production. Le stockage Premium prend en charge les disques de machines virtuelles pouvant être associés à des machines virtuelles de taille spécifique, telles que les machines de séries DS, DSv2, GS, et F. Le disque Premium est fourni dans différentes tailles, allant de 32 Go à 4 096 Go. Chaque taille de disque a ses propres spécifications en matière de performances. Selon les besoins de votre application, vous pouvez associer un ou plusieurs disques à votre machine virtuelle.

Lorsque vous créez un disque managé dans le portail, vous pouvez choisir le **Type de compte** associé au type de disque que vous souhaitez utiliser. N’oubliez pas que les disques disponibles ne s’affichent pas tous dans le menu déroulant. Une fois que vous avez choisi une taille de machine virtuelle, le menu affiche uniquement les références SKU de stockage Premium disponibles qui sont associées à cette taille de machine virtuelle.

![Capture d’écran de la page de disque managé](./media/oracle-design/premium_disk01.png)

Une fois que vous avez configuré votre stockage sur une machine virtuelle, vous pouvez tester la charge des disques avant de créer une base de données. Le fait de connaître la latence et le débit du taux d’E/S peut vous aider à déterminer si les machines virtuelles prennent en charge le débit attendu avec les cibles de latence.

Divers outils sont dédiés au test de charge d’application, comme Oracle Orion, Sysbench et Fio.

Réexécutez le test de charge après avoir déployé une base de données Oracle. Démarrez vos charges de travail normales et maximales pour voir la base de référence de votre environnement.

Il peut être plus important de dimensionner le stockage en fonction du taux d’E/S par seconde que le stockage lui-même. Par exemple, si le taux d’E/S par seconde nécessaire est de 5 000, alors que vous n’avez besoin que de 200 Go, vous pouvez quand même obtenir le disque Premium de classe P30, même s’il est fourni avec plus de 200 Go de stockage.

Le taux d’E/S par seconde se trouve dans le rapport AWR. Il est déterminé par le journal Redo, les lectures physiques et le taux d’écritures.

![Capture d’écran de la page Rapport AWR](./media/oracle-design/awr_report.png)

Par exemple : la restauration par progression fait 12 200 000 octets par seconde, ce qui est égal à 11,63 Mbits/s.
Le nombre d’E/S par seconde est égal à 12 200 000/2 358 = 5 174.

Une fois que vous avez une vision claire de vos besoins en E/S, vous pouvez choisir une combinaison de disques mieux adaptée à ces besoins.

**Recommandations**

- Pour l’espace disque logique de données, répartissez la charge de travail d’E/S sur plusieurs disques à l’aide du stockage managé ou d’Oracle ASM.
- Lorsque la taille du bloc d’E/S augmente pour les opérations de lecture et d’écriture intensives, ajoutez des disques de données.
- Augmentez la taille du bloc pour les processus séquentiels volumineux.
- Utilisez la compression de données pour réduire les E/S (données et index).
- Placez les flux de transport Redo, System, Temp et Undo sur des disques de données distincts.
- Ne placez pas de fichiers d’application sur les disques du système d’exploitation par défaut (/dev/sda). Ces disques sont optimisés pour les démarrages rapides de machine virtuelle et risquent de ne pas fournir de performances optimales pour votre application.
- Lorsque vous utilisez des machines virtuelles de série M sur le stockage Premium, activez [l’Accélérateur d'écriture](../../how-to-enable-write-accelerator.md) sur le disque des journaux d’activité de rétablissement.

### <a name="disk-cache-settings"></a>Paramètres de cache des disques

Il existe trois solutions pour la mise en cache de l’hôte :

- *ReadOnly* : toutes les demandes sont mises en cache pour les lectures ultérieures. Toutes les écritures sont conservées directement dans le Stockage Blob Azure.

- *ReadWrite* : il s'agit d'un algorithme de type « lecture anticipée ». Les lectures et les écritures sont mises en cache pour les lectures ultérieures. Les écritures qui ne sont pas de type double écriture sont d’abord conservées dans le cache local. Il fournit également la latence de disque la plus faible pour les charges de travail légères. L’utilisation d’un cache en lecture/écriture avec une application qui ne gère pas la persistance des données requises peut entraîner des pertes de données en cas de panne de la machine virtuelle.

- *Aucun* (désactivé) : cette option vous permet de contourner le cache. Toutes les données sont transférées sur le disque et conservées dans Stockage Azure. Cette méthode vous donne le taux d’E/S le plus élevé pour les charges de travail intensives d’E/S. Vous devez également tenir compte du coût de transaction.

**Recommandations**

Pour optimiser le débit, il est recommandé de commencer par **Aucun** pour la mise en cache de l’hôte. Pour le stockage Premium, vous devez désactiver les barrières lorsque vous montez le système de fichiers conformément aux options **Lecture seule** ou **Aucun**. Mettez à jour le fichier/etc/fstab avec l’UUID sur les disques.

![Capture d’écran de la page disque managé qui affiche les options ReadOnly et None.](./media/oracle-design/premium_disk02.png)

- Pour les disques du système d’exploitation, utilisez la mise en cache par défaut **Lecture/Écriture**.
- Pour SYSTEM, TEMP et UNDO, utilisez **Aucun** pour la mise en cache.
- Pour DATA, utilisez **Aucun** pour la mise en cache. Toutefois, si votre base de données est en lecture seule ou en lecture intensive, utilisez la mise en cache **Lecture seule**.

Une fois que votre configuration de disque de données est enregistrée, vous ne pouvez pas modifier le paramètre de mise en cache de l’hôte, sauf si vous démontez le disque au niveau du système d’exploitation, puis le remontez après modification.

## <a name="security"></a>Sécurité

Une fois votre environnement Azure configuré, l’étape suivante consiste à sécuriser votre réseau. Voici quelques recommandations :

- *Stratégie du groupe de sécurité réseau* : le groupe de sécurité réseau peut être défini par un sous-réseau ou une carte réseau. Il est plus simple de contrôler la sécurité de l’accès au niveau du sous-réseau et de forcer le routage pour les éléments tels que les pare-feu d’application.

- *Serveur de rebond (jumpbox)*  : pour un accès plus sécurisé, les administrateurs ne doivent pas se connecter directement au service d'application ou à la base de données. Une jumpbox sert de média entre la machine de l’administrateur et les ressources Azure.
![Capture d’écran de la page relative à la topologie Jumpbox](./media/oracle-design/jumpbox.png)

    L’accès à la machine de l’administrateur doit être limité à l’adresse IP de la jumpbox. La jumpbox doit avoir accès à l’application et à la base de données.

- *Réseau privé* (sous-réseaux) : il est recommandé de placer le service d'application et la base de données sur des sous-réseaux distincts afin de mieux contrôler la stratégie de groupe de sécurité réseau.


## <a name="additional-reading"></a>Documentation supplémentaire

- [Configurer Oracle ASM](configure-oracle-asm.md)
- [Implémenter Oracle Data Guard sur une machine virtuelle Linux Azure](configure-oracle-dataguard.md)
- [Implémenter Oracle Golden Gate sur une machine virtuelle Linux Azure](configure-oracle-golden-gate.md)
- [Sauvegarde et récupération Oracle](./oracle-overview.md)

## <a name="next-steps"></a>Étapes suivantes

- [Tutoriel : Créer des machines virtuelles hautement disponibles](../../linux/create-cli-complete.md)
- [Explorer des exemples Azure CLI de déploiement de machines virtuelles](../../linux/cli-samples.md)