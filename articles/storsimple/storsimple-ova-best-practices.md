---
title: Bonnes pratiques pour StorSimple Virtual Array | Microsoft Docs
description: Décrit les bonnes pratiques de déploiement et de gestion de la baie virtuelle StorSimple Virtual Array.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 57ac6eeb-c47c-442d-a5f4-b360d81a76a6
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 07/25/2019
ms.author: alkohli
ms.openlocfilehash: 71b018da6b54ebf2b45a261378ea521a397159e5
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94964981"
---
# <a name="storsimple-virtual-array-best-practices"></a>Bonnes pratiques liées à StorSimple Virtual Array

## <a name="overview"></a>Vue d’ensemble

[!INCLUDE [storsimple-virtual-array-eol-banner](../../includes/storsimple-virtual-array-eol-banner.md)]

Microsoft Azure StorSimple Virtual Array est une solution de stockage intégrée qui gère les tâches de stockage entre un appareil virtuel local exécuté dans un hyperviseur et le stockage cloud Microsoft Azure. StorSimple Virtual Array représente une alternative efficace et économique à la baie de stockage physique de la gamme 8000. La baie virtuelle peut s’exécuter sur votre infrastructure de l’hyperviseur, prend en charge les protocoles SMB et iSCSI, et est adaptée aux scénarios de bureaux distants. Pour plus d’informations sur les solutions StorSimple, consultez [Vue d’ensemble de Microsoft Azure StorSimple](https://www.microsoft.com/en-us/server-cloud/products/storsimple/overview.aspx).

Cet article aborde les bonnes pratiques à implémenter au cours de l’installation initiale, du déploiement et de la gestion de la baie virtuelle StorSimple Virtual Array. Ces bonnes pratiques fournissent des instructions validées pour l’installation et la gestion de votre baie virtuelle. Cet article est destiné aux administrateurs informatiques qui déploient et gèrent les baies virtuelles dans leurs centres de données.

Nous vous recommandons une révision périodique des bonnes pratiques afin de garantir que votre périphérique est toujours en conformité lorsque des modifications sont apportées au flux des opérations ou au flux d’installation. Si vous rencontrez des problèmes lors de l’implémentation de ces bonnes pratiques sur votre baie virtuelle, [contactez le support Microsoft](storsimple-virtual-array-log-support-ticket.md) pour obtenir une assistance.

## <a name="configuration-best-practices"></a>Bonnes pratiques de configuration
Ces bonnes pratiques couvrent les instructions à suivre lors de la configuration initiale et du déploiement des baies virtuelles. Elles incluent les bonnes pratiques relatives à l’approvisionnement de la machine virtuelle, aux paramètres de stratégie de groupe, au dimensionnement, au paramétrage de la mise en réseau, à la configuration des comptes de stockage et à la création de partages et de volumes pour la baie virtuelle. 

### <a name="provisioning"></a>Approvisionnement
StorSimple Virtual Array est une machine virtuelle mise en service sur l’hyperviseur (Hyper-V ou VMware) de votre serveur hôte. Lors de l’approvisionnement de la machine virtuelle, assurez-vous que votre hôte est en mesure de dédier des ressources suffisantes. Pour plus d’informations, consultez la page [ressources minimales requises](storsimple-virtual-array-deploy2-provision-hyperv.md#step-1-ensure-that-the-host-system-meets-minimum-virtual-array-requirements) pour mettre en service une baie.

Implémentez les bonnes pratiques suivantes lors de l’approvisionnement de la baie virtuelle :

|  | Hyper-V | VMware |
| --- | --- | --- |
| **Type de machine virtuelle** |**génération 2** à utiliser avec Windows Server 2012 ou version ultérieure et une image *.vhdx* . <br></br> **génération 1** à utiliser avec un serveur Windows Server 2008 ou version ultérieure et une image *.vhd* . |Utilisez la machine virtuelle version 8 si vous utilisez une image *.vmdk*. |
| **Type de mémoire** |Configurez-la en tant que **mémoire statique**. <br></br> N’utilisez pas l’option de **mémoire dynamique** . | |
| **Type de disque de données** |Sélectionnez **Taille dynamique**.<br></br> **Taille fixe** prend beaucoup de temps. <br></br> N’utilisez pas l’option de **différenciation** . |Utilisez l’option **Allocation dynamique** . |
| **Modification du disque de données** |L’expansion et la réduction ne sont pas autorisées. Une tentative dans ce sens entraîne la perte de toutes les données locales sur l’appareil. |L’expansion et la réduction ne sont pas autorisées. Une tentative dans ce sens entraîne la perte de toutes les données locales sur l’appareil. |

### <a name="sizing"></a>Dimensionnement
Lorsque vous dimensionnez votre baie virtuelle StorSimple Virtual Array, tenez compte des facteurs suivants :

* Réservation locale pour des volumes ou des partages. Environ 12 % de l’espace est réservé au niveau local pour chaque volume hiérarchisé et chaque partage à plusieurs niveaux alloué. Environ 10 % de l’espace est également réservé pour un volume épinglé localement pour le système de fichiers.
* Charge des instantanés. Environ 15 % de l’espace au niveau local est réservé pour les instantanés.
* Besoins liés aux restaurations. Si la restauration s’effectue en tant que nouvelle opération, le dimensionnement doit prendre en compte l’espace nécessaire pour la restauration. La restauration s’effectue sur un partage ou un volume de même taille.
* Une certaine mémoire tampon doit être allouée pour toute croissance inattendue.

En fonction des facteurs précédents, les exigences de dimensionnement peuvent être représentées par l’équation suivante :

`Total usable local disk size = (Total provisioned locally pinned volume/share size including space for file system) + (Max (local reservation for a volume/share) for all tiered volumes/share) + (Local reservation for all tiered volumes/shares)`

`Data disk size = Total usable local disk size + Snapshot overhead + buffer for unexpected growth or new share or volume`

Les exemples suivants illustrent la façon de dimensionner une baie virtuelle selon vos besoins.

#### <a name="example-1"></a>Exemple 1 :
Sur votre baie virtuelle, vous souhaitez être en mesure de

* configurer un partage ou un volume hiérarchisé de 2 To ;
* configurer un partage ou un volume hiérarchisé de 1 To ;
* configurer un partage ou un volume épinglé localement de 300 Go.

Pour les partages ou volumes précédents, calculons l’espace nécessaire au niveau local.

Tout d’abord, pour chaque partage/volume hiérarchisé, la réservation locale serait égale à 12 % de la taille du partage/volume. Pour le volume/partage épinglé localement, la réservation locale représente 10 % de la taille du volume/partage épinglé localement (en plus de la taille approvisionnée). Dans cet exemple, vous avez besoin

* d’une réservation locale de 240 Go (pour un partage/volume hiérarchisé de 2 To) ;
* d’une réservation locale de 120 Go (pour un partage/volume hiérarchisé de 1 To) ;
* de 330 Go pour le volume ou le partage épinglé localement (en ajoutant 10 % de réservation locale à la taille approvisionnée de 300 Go) ;

À ce stade, l’espace total requis au niveau local est de : 240 Go + 120 Go + 330 Go = 690 Go.

Deuxièmement, nous avons besoin d’au moins autant d’espace au niveau local que la réservation unique la plus grande. Ce montant supplémentaire est utilisé au cas où il serait nécessaire d’effectuer une restauration à partir d’un instantané cloud. Dans cet exemple, la plus grande réservation locale étant de 330 Go (avec la réservation pour le système de fichiers), vous devez l’ajouter aux 690 Go : 690 Go + 330 Go = 1 020 Go.
Si nous avons effectué des restaurations supplémentaires ultérieures, nous pouvons toujours libérer l’espace utilisé pour l’opération de restauration précédente.

Troisièmement, nous avons besoin de 15 % de votre espace local total jusqu’ici pour stocker les instantanés locaux, si bien que 85 % de celui-ci est disponible. Dans cet exemple, cela représente environ 1 020 Go = 0,85&ast;To du disque de données configuré. Par conséquent, le disque de données configuré aurait une taille de (1 020&ast;(1/0,85)) = 1 200 Go = 1,20 To ~ 1,25 To (en arrondissant au quartile le plus proche)

En tenant compte de la croissance inattendue et de nouvelles restaurations, vous devez configurer un disque local d’environ 1,25 - 1,5 To.

> [!NOTE]
> Nous vous recommandons également de configurer dynamiquement le disque local. Cette recommandation est due au fait que l’espace de restauration est nécessaire uniquement lorsque vous souhaitez restaurer des données datant de plus de cinq jours. La récupération au niveau de l’élément vous permet de restaurer les données des cinq derniers jours sans nécessiter d’espace supplémentaire pour la restauration.


#### <a name="example-2"></a>Exemple 2 :
Sur votre baie virtuelle, vous souhaitez être en mesure de

* configurer un volume hiérarchisé de 2 To ;
* configurer un volume épinglé localement de 300 Go.

En tenant compte des 12 % de réservation de l’espace local pour les partages/volumes hiérarchisés et des 10 % pour les partages/volumes épinglés localement, nous avons besoin

* d’une réservation locale de 240 Go (pour un partage/volume hiérarchisé de 2 To) ;
* de 330 Go pour le volume ou le partage épinglé localement (en ajoutant 10 % de réservation locale à l’espace approvisionné de 300 Go)

L’espace total requis au niveau local est de : 240 Go + 330 Go = 570 Go

L’espace local minimal requis pour la restauration est de 330 Go.

15 % de votre disque total sont utilisés pour stocker les instantanés, si bien que seuls 85 % sont disponibles. Ainsi, la taille du disque est de (900&ast;(1/0,85)) = 1,06 To ~ 1,25 To (en arrondissant au quartile le plus proche)

En tenant compte d’une éventuelle croissance inattendue, vous pouvez configurer un disque local de 1,25 - 1,5 To.

### <a name="group-policy"></a>Stratégie de groupe
La stratégie de groupe est une infrastructure qui vous permet d’implémenter des configurations spécifiques pour les utilisateurs et les ordinateurs. Les paramètres de stratégie de groupe sont contenus dans des objets de stratégie de groupe (GPO), qui sont liés aux conteneurs des services de domaine Active Directory (AD DS) suivants : sites, domaines ou unités d’organisation (UO). 

Si votre baie virtuelle est jointe à un domaine, des objets GPO peuvent lui être appliqués. Ces objets de stratégie de groupe peuvent installer des applications, telles qu’un logiciel antivirus, susceptibles d’affecter négativement le fonctionnement de la baie virtuelle StorSimple Virtual Array.

Par conséquent, nous vous recommandons de :

* vous assurer que votre baie virtuelle figure dans sa propre unité d’organisation (UO) pour Active Directory ;
* vous assurer qu’aucun objet de stratégie de groupe (GPO) n’est appliqué à votre baie virtuelle. Vous pouvez bloquer l’héritage pour vous assurer que la baie virtuelle (nœud enfant) n’hérite pas automatiquement des objets de stratégie de groupe du parent. Pour plus d'informations, consultez la page [Bloquer l’héritage](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731076(v=ws.11)).

### <a name="networking"></a>Mise en réseau
La configuration réseau de votre baie virtuelle s’effectue via l’interface utilisateur web locale. Une interface réseau virtuelle est activée via l’hyperviseur dans lequel la baie virtuelle est mise en service. Utilisez la page [Paramètres réseau](storsimple-virtual-array-deploy3-fs-setup.md) pour configurer l’adresse IP, le sous-réseau et la passerelle de l’interface réseau virtuelle.  Vous pouvez également configurer les serveurs DNS principal et secondaire, les paramètres de temps et les paramètres proxy facultatifs de votre appareil. La plus grande partie de la configuration réseau correspond à une installation unique. Examinez la [configuration requise de mise en réseau pour StorSimple](storsimple-ova-system-requirements.md#networking-requirements) avant de déployer la baie virtuelle.

Lorsque vous déployez une baie virtuelle, nous vous recommandons d’appliquer les bonnes pratiques suivantes :

* Assurez-vous que le réseau dans lequel la baie virtuelle est déployée a toujours la capacité de dédier une bande passante Internet de 5 Mbits/s (ou plus).
  
  * La bande passante Internet requise varie selon les caractéristiques de votre charge de travail et le taux de modification des données.
  * La modification des données qui peut être gérée est directement proportionnelle à la bande passante Internet. Par exemple, lorsque vous effectuez une sauvegarde, une bande passante de 5 Mbits/s peut prendre en charge une modification des données d’environ 18 Go en 8 heures. Avec une bande passante quatre fois supérieure (20 Mbits/s), vous pouvez gérer une modification des données quatre fois supérieure (72 Go).
* Vérifiez que la connectivité Internet est disponible en permanence. Des connexions Internet sporadiques ou non fiables pour les appareils peuvent entraîner une perte d’accès aux données dans le cloud et peuvent aboutir à une configuration non prise en charge.
* Si vous envisagez de déployer votre appareil en tant que serveur iSCSI :
  
  * Nous vous recommandons de désactiver l’option **Obtenir automatiquement une adresse IP** (DHCP).
  * Configurez des adresses IP statiques. Vous devez configurer un serveur DNS principal et un serveur DNS secondaire.
  * Si vous définissez plusieurs interfaces réseau sur votre baie virtuelle, seule la première interface réseau (par défaut, il s’agit d’ **Ethernet**) peut atteindre le cloud. Pour contrôler le type de trafic, vous pouvez créer plusieurs interfaces réseau virtuelles sur votre baie virtuelle (configurée comme un serveur iSCSI) et les connecter aux différents sous-réseaux.
* Pour limiter uniquement la bande passante cloud (utilisée par la baie virtuelle), configurez la limitation sur le routeur ou le pare-feu. Si vous définissez la limitation dans votre hyperviseur, il limite tous les protocoles, y compris iSCSI et SMB, et non pas simplement la bande passante cloud.
* Assurez-vous que la synchronisation date/heure des hyperviseurs est activée. Si vous utilisez Hyper-V, sélectionnez votre baie virtuelle dans le Gestionnaire Hyper-V, accédez à **Paramètres &gt; Services d’intégration** et assurez-vous que l’option **Synchronisation date/heure** est activée.

### <a name="storage-accounts"></a>Comptes de stockage
Une baie virtuelle StorSimple Virtual Array peut être associée à un compte de stockage unique. Ce compte de stockage peut être un compte de stockage généré automatiquement, un compte figurant dans le même abonnement que le service, ou un compte de stockage associé à un autre abonnement. Pour plus d'informations, découvrez comment [gérer des comptes de stockage pour votre baie virtuelle](storsimple-virtual-array-manage-storage-accounts.md).

Utilisez les recommandations suivantes pour les comptes de stockage associés à votre baie virtuelle.

* Lorsque vous liez plusieurs baies virtuelles à un compte de stockage unique, tenez compte de la capacité maximale (64 To) d’une baie virtuelle et de la taille maximale (500 To) d’un compte de stockage. Cela limite le nombre de baies virtuelles pleinement dimensionnées pouvant être associées au compte de stockage à environ 7.
* Lors de la création d’un compte de stockage
  
  * Nous vous recommandons de le créer dans la région la plus proche du bureau distant où votre baie virtuelle StorSimple Virtual Array est déployée, pour réduire au maximum la latence.
  * N’oubliez pas que vous ne pouvez pas déplacer un compte de stockage d’une région à une autre. De plus, vous ne pouvez pas déplacer un service d’un abonnement à un autre.
  * Utilisez un compte de stockage qui met en œuvre la redondance entre les centres de données. Le stockage géoredondant (GRS), le stockage redondant dans une zone (ZRS) et le stockage localement redondant (LRS) sont tous pris en charge avec votre baie virtuelle. Pour plus d’informations sur les différents types de comptes de stockage, consultez la page [Réplication du stockage Azure](../storage/common/storage-redundancy.md).

### <a name="shares-and-volumes"></a>Partages et volumes
Pour votre baie virtuelle StorSimple Virtual Array, vous pouvez mettre en service des partages lorsqu’elle est configurée comme un serveur de fichiers et des volumes lorsqu’elle est configurée comme un serveur iSCSI. Les bonnes pratiques permettant la création de partages et de volumes sont associées à la taille et au type configuré.

#### <a name="volumeshare-size"></a>Taille des volumes/partages
Sur votre baie virtuelle, vous pouvez mettre en service des partages lorsqu’elle est configurée comme un serveur de fichiers et des volumes lorsqu’elle est configurée comme un serveur iSCSI. Les bonnes pratiques permettant la création de partages et de volumes sont associées à la taille et au type configuré. 

Gardez à l’esprit les bonnes pratiques suivantes lors de l’approvisionnement des partages ou des volumes sur votre appareil virtuel.

* La taille des fichiers par rapport à la taille configurée d’un partage hiérarchisé peut affecter les performances de hiérarchisation. L'utilisation de fichiers volumineux peut entraîner une montée en charge lente. Lorsque vous utilisez des fichiers volumineux, vérifiez que la taille du plus grand fichier est inférieure à 3 % de la taille du partage.
* Au plus, 16 volumes/partages peuvent être créés sur la baie virtuelle. Pour les limites de taille des volumes/partages attachés localement et à plusieurs niveaux, reportez-vous toujours aux [limites de StorSimple Virtual Array](storsimple-ova-limits.md).
* Lorsque vous créez un volume, tenez compte de la consommation des données attendue, ainsi que de la croissance future. Il ne sera pas possible d’augmenter la taille du volume ou du partage de fichiers par la suite.
* Une fois que le volume/partage a été créé, vous ne pouvez pas réduire la taille du volume/partage sur StorSimple.
* Lors de l’écriture sur un volume hiérarchisé sur StorSimple, lorsque les données du volume atteignent un certain seuil (par rapport à l’espace local réservé pour le volume), les E/S sont limitées. Continuer à écrire sur ce volume ralentit considérablement les E/S. Bien que vous puissiez écrire sur un volume hiérarchisé au-delà de sa capacité déployée (nous n’empêchons pas activement l’utilisateur d’écrire au-delà de la capacité déployée), une notification d’alerte apparaît lorsque vous effectuez une demande trop importante. Quand cette alerte apparaît, il est impératif de prendre des mesures correctives, en supprimant par exemple les données du volume (l’expansion des volumes n’est pas prise en charge actuellement).
* Pour les cas d’utilisation liés à la récupération d’urgence, comme le nombre de partages/volumes autorisés est de 16 et que le nombre maximal de partages/volumes pouvant être traités en parallèle est également de 16, le nombre de partages/volumes n’a pas d’incidence sur vos RPO et RTO.

#### <a name="volumeshare-type"></a>Type de volume/partage
StorSimple prend en charge deux types de volume/partage en fonction de l’utilisation : épinglés localement et hiérarchisés. Les volumes/partages épinglés localement sont alloués statiquement tandis que les volumes/partages hiérarchisés sont alloués dynamiquement. Une fois crée, il est impossible de convertir un volume/partage épinglé localement en un volume/partage hiérarchisé et *inversement*.

Nous vous recommandons d’implémenter les bonnes pratiques suivantes lors de la configuration de volumes/partages StorSimple :

* Identifiez le type de volume en fonction des charges de travail que vous envisagez de déployer avant de créer un volume. Utilisez des volumes épinglés localement pour les charges de travail qui requièrent des garanties locales de données (même pendant une panne du cloud) et qui nécessitent de faibles temps de latence du cloud. Une fois que vous avez créé un volume sur votre baie virtuelle, vous ne pouvez pas modifier le type de volume en remplaçant le type épinglé localement par le type hiérarchisé, ou *inversement*. Par exemple, créez des volumes épinglés localement lors du déploiement de charges de travail SQL ou de charges de travail hébergeant des machines virtuelles ; utilisez des volumes hiérarchisés pour des charges de travail de partage de fichiers.


#### <a name="volume-format"></a>Format des volumes
Après avoir créé des volumes StorSimple sur votre serveur iSCSI, vous devez initialiser, monter et formater ces volumes. Cette opération est effectuée sur l’hôte connecté à votre appareil StorSimple. Les bonnes pratiques suivantes sont recommandées lors du montage et du formatage des volumes sur l’hôte StorSimple.

* Effectuez un formatage rapide sur tous les volumes StorSimple.
* Lorsque vous formatez un volume StorSimple, utilisez une taille d’unité d’allocation de 64 Ko (la valeur par défaut est de 4 Ko). La taille d’unité d’allocation de 64 Ko est basée sur les tests effectués en interne pour les charges de travail StorSimple courantes et d’autres charges de travail.
* Lorsque vous utilisez la baie virtuelle StorSimple Virtual Array configurée comme un serveur iSCSI, n’utilisez pas de volumes fractionnés ni de disques dynamiques, car ceux-ci ne sont pas pris en charge par StorSimple.

#### <a name="share-access"></a>Accès aux partages
Lorsque vous créez des partages sur le serveur de fichiers de votre baie virtuelle, suivez ces instructions :

* Lorsque vous créez un partage, affectez un groupe d’utilisateurs en tant qu’administrateur de partage à la place d’un utilisateur individuel.
* Vous pouvez gérer les autorisations NTFS après la création du partage en modifiant les partages via l’Explorateur Windows.

#### <a name="volume-access"></a>Accès aux volumes
Lorsque vous configurez les volumes iSCSI sur votre baie virtuelle StorSimple Virtual Array, il est important de contrôler l’accès chaque fois que nécessaire. Afin de déterminer quels serveurs hôtes peuvent accéder aux volumes, créez et associez des enregistrements de contrôle d’accès (ACR) aux volumes StorSimple.

Utilisez les bonnes pratiques suivantes lors de la configuration d’ACR pour les volumes StorSimple :

* Associez toujours au moins un ACR à un volume.

* Lorsque vous attribuez plusieurs ACR à un volume, veillez à ce que ce volume ne soit pas exposé de façon à ce que plusieurs hôtes non-cluster puissent y accéder simultanément. Si vous avez affecté plusieurs ACR à un volume, un message d’avertissement s’affiche pour vous inviter à vérifier votre configuration.

### <a name="data-security-and-encryption"></a>Chiffrement et sécurité des données
Votre baie virtuelle StorSimple Virtual Array a des fonctionnalités de chiffrement et de sécurité des données qui garantissent la confidentialité et l’intégrité de vos données. Lorsque vous utilisez ces fonctionnalités, nous vous recommandons de suivre ces bonnes pratiques : 

* Définissez une clé de chiffrement de stockage cloud pour générer un chiffrement AES-256 avant l’envoi des données de votre baie virtuelle vers le cloud. Cette clé n’est pas requise si vos données sont chiffrées dès le départ. La clé peut être générée et protégée à l’aide d’un système de gestion de clés tel qu’ [Azure Key Vault](../key-vault/general/overview.md).
* Lorsque vous configurez le compte de stockage via le service StorSimple Manager, veillez à activer le mode TLS afin de créer un canal sécurisé pour la communication réseau entre votre appareil StorSimple et le cloud.
* Régénérez les clés de vos comptes de stockage (en accédant au service Stockage Azure) régulièrement pour prendre en compte toutes les modifications de l’accès en fonction de la liste modifiée des administrateurs.
* Les données figurant sur votre baie virtuelle sont compressées et dédupliquées avant leur envoi vers Azure. Nous ne recommandons pas l’utilisation du service de rôle Déduplication des données sur votre hôte Windows Server.

## <a name="operational-best-practices"></a>Bonnes pratiques opérationnelles
Les bonnes pratiques opérationnelles sont des instructions à suivre lors de la gestion ou de l’exploitation quotidienne de la baie virtuelle. Elles couvrent des tâches de gestion spécifiques telles que la réalisation de sauvegardes, la restauration à partir d’un jeu de sauvegarde, la réalisation d’un basculement, la désactivation et la suppression de la baie, l’analyse de l’intégrité et de l’utilisation du système, et l’exécution d’analyses antivirus sur votre baie virtuelle.

### <a name="backups"></a>Sauvegardes
Les données figurant sur votre baie virtuelle sont sauvegardées dans le cloud de deux manières, via une sauvegarde quotidienne par défaut automatisée de l’ensemble de l’appareil commençant à 22h30 et via une sauvegarde manuelle à la demande. Par défaut, l’appareil crée automatiquement des instantanés cloud quotidiens de toutes les données résidant sur ce dernier. Pour plus d’informations, consultez la page [Sauvegarder votre StorSimple Virtual Array](storsimple-virtual-array-backup.md).

Il est impossible de modifier la fréquence et la rétention associées aux sauvegardes par défaut, mais vous pouvez configurer l’heure à laquelle les sauvegardes quotidiennes sont lancées chaque jour. Lorsque vous configurez l’heure de début des sauvegardes automatisées, nous vous recommandons de procéder comme suit :

* Planifiez vos sauvegardes pendant les heures creuses. L’heure de début de la sauvegarde ne doit pas coïncider avec un grand nombre d’E/S d’hôte.
* Lancez une sauvegarde manuelle à la demande lorsque vous envisagez d’effectuer un basculement de l’appareil ou avant la fenêtre de maintenance, pour protéger les données figurant sur votre baie virtuelle.

### <a name="restore"></a>Restaurer
Vous pouvez restaurer à partir d’un jeu de sauvegarde de deux manières : restaurez vers un autre volume ou partage, ou effectuez une récupération au niveau de l’élément (disponible uniquement sur une baie virtuelle configurée en tant que serveur de fichiers). La récupération au niveau de l’élément vous permet d'effectuer une récupération granulaire de fichiers et de dossiers à partir d'une sauvegarde cloud contenant tous les partages sur l'appareil StorSimple. Pour plus d’informations, consultez la page [Restaurer à partir d’une sauvegarde](storsimple-virtual-array-clone.md).

Lorsque vous effectuez une restauration, gardez à l’esprit les instructions suivantes :

* Votre baie virtuelle StorSimple Virtual Array ne prend pas en charge la restauration sur place. Celle-ci peut néanmoins être réalisée facilement via un processus en deux étapes : libérez de l’espace sur la baie virtuelle, puis restaurez vers un autre volume/partage.
* Lors de la restauration à partir d’un volume local, n’oubliez pas que la restauration est une opération longue. Le volume peut être mis en ligne rapidement, mais l’alimentation des données continue en arrière-plan.
* Le type de volume reste le même pendant le processus de restauration. Un volume hiérarchisé est restauré vers un autre volume hiérarchisé, et un volume épinglé localement vers un autre volume épinglé localement.
* Lorsque vous tentez de restaurer un volume ou un partage à partir d’un jeu de sauvegarde, si le travail de restauration échoue, un volume ou partage cible peut toujours être créé dans le portail. Il est important de supprimer ce volume ou partage cible inutilisé dans le portail pour réduire les problèmes futurs découlant de cet élément.

### <a name="failover-and-disaster-recovery"></a>Basculement et récupération d'urgence
Un basculement de l’appareil vous permet de migrer vos données à partir d’un appareil *source* dans le centre de données vers un autre appareil *cible* situé à un emplacement géographique identique ou différent. Le basculement de l'appareil s'applique à l'ensemble de l'appareil. Lors du basculement, les données de cloud pour l'appareil source deviennent la propriété de l'appareil cible.

Pour votre baie virtuelle StorSimple Virtual Array, vous pouvez basculer uniquement vers une autre baie virtuelle gérée par le même service StorSimple Manager. Un basculement vers un appareil de gamme 8000 ou vers une baie gérée par un autre service StorSimple Manager (que celui de l’appareil source) n’est pas autorisé. Pour en savoir plus sur les considérations relatives au basculement, consultez la page [Configuration requise pour le basculement d’appareil](storsimple-virtual-array-failover-dr.md).

Lorsque vous effectuez un basculement pour votre baie virtuelle, gardez à l’esprit les éléments suivants :

* Pour un basculement planifié, une bonne pratique consiste à déconnecter tous les volumes/partages avant de lancer le basculement. Suivez les instructions spécifiques au système d’exploitation pour déconnecter au préalable les volumes/partages sur l’hôte, puis déconnectez ces éléments sur votre appareil virtuel.
* Pour la récupération d'urgence du serveur de fichiers, nous vous recommandons de joindre l'appareil cible au même domaine que celui de la source, afin que les autorisations de partage soient résolues automatiquement. Cette version prend en charge uniquement le basculement vers un appareil cible dans le même domaine.
* Une fois la récupération d’urgence terminée, l’appareil source est supprimé automatiquement. Même si l'appareil n'est plus disponible, la machine virtuelle que vous avez configurée sur le système hôte continue de consommer des ressources. Nous vous recommandons de supprimer cette machine virtuelle de votre système hôte pour empêcher l’apparition de frais.
* Notez que même si le basculement échoue, **les données sont toujours sûres dans le cloud**. Considérez les trois scénarios suivants, dans lesquels le basculement n’a pas réussi :
  
  * Une erreur s’est produite dans les étapes initiales du basculement, par exemple lors de la réalisation des vérifications préalables de récupération d’urgence. Dans ce cas, votre appareil cible est toujours utilisable. Vous pouvez réessayer le basculement sur le même appareil cible.
  * Une erreur s’est produite pendant le processus de basculement lui-même. Dans ce cas, l’appareil cible est marqué comme inutilisable. Vous devez mettre en service et configurer une autre baie virtuelle cible, et l’utiliser pour le basculement.
  * Le basculement a été effectué, après quoi l’appareil source a été supprimé, mais l’appareil cible a des problèmes et vous ne pouvez pas accéder aux données. Les données sont toujours sûres dans le cloud et il est aisé de les récupérer en créant une autre baie virtuelle, puis en l’utilisant comme appareil cible pour la récupération d’urgence.

### <a name="deactivate"></a>Désactivation
Quand vous désactivez un StorSimple Virtual Array, vous interrompez la connexion entre l’appareil et le service StorSimple Manager correspondant. La désactivation est une opération **définitive** et ne peut pas être annulée. Un appareil désactivé ne peut pas être de nouveau enregistré auprès du service StorSimple Manager. Pour plus d’informations, consultez la page [Désactiver et supprimer votre baie virtuelle StorSimple Virtual Array](storsimple-virtual-array-deactivate-and-delete-device.md).

Tenez compte des bonnes pratiques suivantes lors de la désactivation de votre baie virtuelle :

* Prenez un instantané cloud de toutes les données avant de désactiver un appareil virtuel. Lorsque vous désactivez une baie virtuelle, toutes les données locales de l’appareil sont perdues. La prise d’un instantané cloud vous permettra de récupérer les données à un stade ultérieur.
* Avant de désactiver une baie virtuelle StorSimple Virtual Array, veillez à arrêter ou supprimer les clients et les hôtes qui dépendent de cet appareil.
* Supprimez un appareil désactivé si vous ne l’utilisez plus, afin qu’il n’engendre pas de frais.

### <a name="monitoring"></a>Surveillance
Pour vous assurer que votre baie virtuelle StorSimple Virtual Array est en permanence dans un état sain, vous devez surveiller la baie et vérifier que vous recevez des informations à partir du système, y compris les alertes. Pour surveiller l’intégrité globale de la baie virtuelle, mettez en œuvre les bonnes pratiques suivantes :

* Configurez la surveillance pour effectuer le suivi de l’utilisation du disque de données de votre baie virtuelle, ainsi que du disque du système d’exploitation. Si vous exécutez Hyper-V, vous pouvez utiliser une combinaison de System Center Virtual Machine Manager (SCVMM) et de System Center Operations Manager pour surveiller vos hôtes de virtualisation.
* Configurez les notifications par courrier électronique sur votre baie virtuelle pour envoyer des alertes à certains niveaux d’utilisation.                                                                                                                                                                                                

### <a name="index-search-and-virus-scan-applications"></a>Applications d’analyse antivirus et de recherche d’index
Une baie virtuelle StorSimple Virtual Array peut faire passer automatiquement les données du niveau local au cloud Microsoft Azure. Lorsqu’une application telle qu’une recherche d’index ou une analyse antivirus est utilisée pour analyser les données stockées sur StorSimple, vous devez faire en sorte que les données cloud ne soient pas accessibles ni ramenées au niveau local.

Nous vous recommandons de mettre en œuvre les bonnes pratiques suivantes lors de la configuration de la recherche d’index ou de l’analyse antivirus sur votre baie virtuelle :

* Désactivez toutes les opérations d’analyse complète configurées automatiquement.
* Pour les volumes hiérarchisés, configurez l’application de recherche d’index ou d’analyse antivirus pour qu’elle effectue une analyse incrémentielle. Cela permet d’analyser uniquement les nouvelles données résidant éventuellement au niveau local. Les données qui sont transférées dans le cloud ne sont pas accessibles pendant une opération incrémentielle.
* Veillez à ce que les paramètres et les filtres de recherche corrects soient configurés de sorte que seuls les types prévus de fichiers soient analysés. Par exemple, les fichiers image (JPEG, GIF et TIFF) et les dessins techniques ne doivent pas être analysés au cours de la reconstruction d’index complet ou incrémentiel.

Si vous utilisez le processus d’indexation de Windows, suivez ces instructions :

* N’utilisez pas l’indexeur Windows pour les volumes hiérarchisés, car cela entraînera le rappel de grandes quantités de données (plusieurs To) à partir du cloud si l’index doit être reconstruit fréquemment. La reconstruction de l’index entraîne la récupération de tous les types de fichiers pour l’indexation de leur contenu.
* Utilisez le processus d’indexation de Windows pour les volumes épinglés localement, car il n’accède qu’aux données figurant sur les couches locales pour créer l’index (il n’accède pas aux données cloud).

### <a name="byte-range-locking"></a>Verrouillage de plage d’octets
Les applications peuvent verrouiller une plage d’octets donnée au sein des fichiers. Si le verrouillage de plage d’octets est activé sur les applications qui écrivent sur votre appareil StorSimple, la hiérarchisation ne fonctionne pas sur votre baie virtuelle. Pour que la hiérarchisation fonctionne, toutes les zones des fichiers auxquels vous accédez doivent être déverrouillées. Le verrouillage de plage d’octets n’est pas pris en charge avec les volumes hiérarchisés sur votre baie virtuelle.

Les mesures recommandées pour éviter le verrouillage de plage d’octets incluent :

* Désactivez le verrouillage de plage d'octets dans la logique de votre application.
* Utilisez des volumes épinglés localement (à la place de volumes hiérarchisés) pour les données associées à cette application. Les volumes épinglés localement ne hiérarchisent pas les données dans le cloud.
* Lorsque vous utilisez des volumes épinglés localement avec le verrouillage de plage d’octets activé, un volume peut être mis en ligne avant la fin de la restauration. Dans ce cas, vous devez attendre la fin de la restauration.

## <a name="multiple-arrays"></a>Baies multiples
Il peut être nécessaire de déployer plusieurs baies virtuelles pour prendre en compte un ensemble de données de travail croissant, susceptible de déborder dans le cloud et d’affecter ainsi les performances de l’appareil. Dans ce cas, l’idéal est de dimensionner les appareils à mesure que l’ensemble de données de travail augmente. Cette opération nécessite l’ajout d’un ou de plusieurs appareils dans le centre de données local. Lors de l’ajout des appareils, vous pouvez procéder comme suit :

* Fractionnez le jeu de données actuel.
* Déployez de nouvelles charges de travail sur un ou plusieurs nouveaux périphériques.
* Si vous déployez plusieurs baies virtuelles, nous vous recommandons, du point de vue de l’équilibrage de charge, de distribuer une baie entre différents hôtes de l’hyperviseur.
* Plusieurs baies virtuelles (lorsqu’elles sont configurées en tant que serveur de fichiers ou serveur iSCSI) peuvent être déployées dans un espace de noms de système de fichiers distribué. Pour des instructions détaillées, consultez la page [Guide de déploiement d’une solution d’espace de noms de système de fichiers distribué avec stockage cloud hybride](https://www.microsoft.com/download/details.aspx?id=45507). La réplication du système de fichiers distribué n’est actuellement pas recommandée avec la baie virtuelle. 

## <a name="see-also"></a>Voir aussi
Découvrez la [gestion de votre baie virtuelle StorSimple Virtual Array](storsimple-virtual-array-manager-service-administration.md) par le biais du service StorSimple Manager.