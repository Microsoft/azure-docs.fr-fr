---
title: Options de migration de données à partir des appareils de la gamme StorSimple 8000
description: Fournit une vue d’ensemble des options pour migrer des données de la gamme StorSimple 8000.
services: storsimple
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 03/25/2020
ms.author: alkohli
ms.openlocfilehash: 4839f8211e678f5fc2fb3572c7eaa545fbee6c6c
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94961190"
---
# <a name="options-to-migrate-data-from-storsimple-8000-series"></a>Options de migration de données à partir de la gamme StorSimple 8000

> [!IMPORTANT]
> La fin du support de la gamme StorSimple 8000 interviendra le 31 décembre 2022. Nous recommandons aux clients de la gamme StorSimple 8000 de migrer vers l’une des alternatives décrites dans le document.

La [fin du support](https://support.microsoft.com/lifecycle/search?alpha=Azure%20StorSimple%208000%20Series) de la gamme StorSimple 8000 est prévue en décembre 2022. Les clients qui exécutent la gamme StorSimple 8000 peuvent mettre à niveau vers d’autres services hybrides internes Azure. Cet article décrit les options hybrides Azure disponibles pour migrer les données.

## <a name="migration-options"></a>Options de migration

Les clients qui utilisent la gamme StorSimple 8000 disposent d’options Azure ou tierces.

### <a name="azure-options"></a>Options Azure

#### <a name="migrate-to-azure-file-sync"></a>Migrer vers Azure File Sync

Cette toute nouvelle option de migration permet aux clients de stocker les partages de fichiers de leur organisation dans Azure Files. Ces partages de fichiers sont ensuite centralisés pour un accès local à l’aide d’Azure File Sync (AFS). AFS peut être déployé sur un hôte Windows Server. La migration des données réelle est ensuite effectuée sous forme de copie hôte ou à l’aide de l’outil de migration.

Pour plus d’informations sur la migration des données vers Azure File Sync, accédez à [Migration de StorSimple 8100 et 8600 vers Azure File Sync](../storage/files/storage-files-migration-storsimple-8000.md).

### <a name="third-party-options"></a>Options tierces

#### <a name="migrate-to-panzura-freedom-nas"></a>Migrer vers Panzura Freedom NAS

Les clients de la gamme StorSimple 5000-7000 et de la gamme StorSimple 8000 peuvent choisir de migrer vers Panzura Freedom NAS pour conserver leurs données dans Azure. La solution Panzura Freedom fournit une solution NAS qui couvre les centres de données, les bureaux, ainsi que les clouds publics et privés. Elle prend en charge les workflows de données locaux, hybrides et dans le cloud pour les clients NFS, SMB et mobiles.

Cette migration est assurée par Panzura. Les clients peuvent commencer en demandant la prise en charge de la migration sur le [site web Panzura](https://panzura.com/migrate-storsimple-panzura/).

#### <a name="migrate-to-nasuni"></a>Migrer vers Nasuni

Grâce à Nasuni, il est facile de transférer l’ensemble de votre environnement StorSimple vers une plateforme de services de fichiers stable, sécurisée et à haute performance. Nasuni offre la sécurité et les performances du stockage de fichiers localement tout en le combinant avec la scalabilité et la durabilité d’Azure.  En tant qu’éditeur de logiciels indépendant (ISV) Azure, Nasuni intègre tous les outils nécessaires pour déplacer vos données StorSimple vers une plateforme moderne qui vous permet de partager vos fichiers et de collaborer sur plusieurs sites.

Commencez dès aujourd’hui : [site web de Nasuni](https://info.nasuni.com/storsimple8000-webinar).

<!-- 04/09/2020 v-grpr (priestlg) - As per request, commenting out this section because the information that will go into this section is forthcoming
#### Migrate to Cohesity

Cohesity enables you to migrate data from your current StorSimple 5000–7000 to the Cohesity Data Platform on Azure. The Cohesity Data Platform is a software-defined web-scale solution that consolidates files, backups, objects, and VMs onto a single cloud-native solution. After migration to the Data Platform, you can manage, protect, and provision data and apps from cloud to core through a single pane of glass. With Cohesity, start with as few as three nodes. 

Learn more on [migration to the Cohesity Data Platform](https://info.cohesity.com/migrate-from-storsimple-to-cohesity.html).

#### Migrate to Nasuni

Nasuni makes it easy for StorSimple 5000-7000 customers to migrate and keep their data in Azure.  Nasuni is a leading Azure-based NAS storage solution, giving customers the performance and security they expect from on-prem solutions, with cloud economics and scale.  In addition to high performance file storage, Nasuni and Azure handle backup and DR, while allowing you to share and collaborate on your data around the globe with centralized file storage management. 

Nasuni has the experience to make your migration easy – get started today: https://info.nasuni.com/nasuni-storsimple-migration

#### Migrate to Talon FAST

Talon makes it easy for StorSimple 5000-7000 customers to continue to leverage the benefits they valued so much in the StorSimple platform (small on-site footprint backed by unlimited cloud resources) with even greater function.  With the Talon FAST solution, customers can migrate and keep their data in Azure, while now having an even smaller software-only onsite footprint and adding benefits such as global file locking, global namespace, and multi-site collaboration.  Talon is a leading Azure ecosystem solution, working with global customers to migrate their on-premises file server workloads into a consolidated, Azure-based footprint without compromising user workflow or experience.  

Learn more about how to evolve to a cloud-consolidated enterprise at https://www.talonstorage.com/alliances/microsoft-storsimple.
-->

## <a name="migration---frequently-asked-questions"></a>Migration - Forum aux questions

### <a name="q-when-do-the-storsimple-8000-series-devices-reach-end-of-service"></a>Q. Quand la fin du service des appareils de la gamme StorSimple 8000 est-elle prévue ?

R. La [fin du support](https://support.microsoft.com/[lifecycle/search?alpha=Azure%20StorSimple%208000%20Series) de la gamme StorSimple 8000 est prévue en décembre 2022. La fin du support implique que Microsoft ne sera plus en mesure d’assurer le support matériel et logiciel de ces appareils après décembre 2022. Nous vous recommandons vivement de commencer à envisager dès maintenant un plan pour migrer les données de vos appareils.

### <a name="q-what-happens-to-the-data-i-have-stored-in-azure"></a>Q. Que se passe-t-il pour les données que j’ai stockées dans Azure ?  

R. Vous pouvez continuer à utiliser les données dans Azure après avoir migré vers un service plus récent.

### <a name="q-what-happens-to-the-data-i-have-stored-locally-on-my-storsimple-device"></a>Q. Que se passe-t-il pour les données que j’ai stockées localement sur mon appareil StorSimple ?

R. Les données qui se trouvent sur l’appareil local peuvent être copiées vers le service plus récent, comme décrit dans les documents de migration.

### <a name="q-what-happens-if-i-want-to-keep-my-storsimple-8000-series-appliance"></a>Q. Que se passe-t-il si je souhaite conserver mon appliance de la gamme StorSimple 8000 ?

R. Même si les services peuvent continuer à fonctionner, il se peut que Microsoft ne soit plus en mesure de fournir la prise en charge matérielle et logicielle. Une migration est vivement recommandée pour la continuité des activités.

### <a name="q-what-options-are-available-to-migrate-data-from-storsimple-8000-series-devices"></a>Q. Quelles options sont disponibles pour migrer des données à partir d’appareils de la gamme StorSimple 8000 ?

R. En fonction de leur scénario, les utilisateurs de la gamme StorSimple 8000 disposent des options de migration suivantes :

* **Migrer vers Azure File Sync** : utilisez cette option si vous souhaitez basculer vers le format natif d’Azure. Vous pouvez utiliser Azure File Sync pour la gestion centralisée des partages de fichiers.

* **Autres options** : Vous pouvez contacter le Support Microsoft pour discuter des options de migration non répertoriées ici.

### <a name="q-is-migration-to-other-storage-solutions-supported"></a>Q. La migration vers d’autres solutions de stockage est-elle prise en charge ?

R. Oui. La migration vers d’autres solutions de stockage à l’aide de la copie hôte des données est prise en charge.

### <a name="q-is-migration-supported-by-microsoft"></a>Q. La migration est-elle prise en charge par Microsoft ?

R. La migration à partir de la gamme 8000 est une opération entièrement prise en charge. En réalité, Microsoft vous recommande de contacter le support avant de commencer la migration. La migration est actuellement une opération assistée. Si vous envisagez de migrer des données à partir de votre appareil de la gamme StorSimple 8000, [contactez le support StorSimple](mailto:storsimp@microsoft.com).

### <a name="q-what-is-the-pricing-model-for-migration-to-azure-file-sync"></a>Q. Quel est le modèle de tarification pour la migration vers Azure File Sync ?

R. Si vous utilisez Azure File Sync, des frais d’abonnement peuvent s’appliquer pour le service. Les clients devront également s’acquitter des coûts de stockage. Pour obtenir une estimation, reportez-vous à la [tarification d’Azure File Sync]( https://azure.microsoft.com/pricing/details/storage/files/).

### <a name="q-how-long-does-it-take-to-complete-a-migration"></a>Q. Quelle est la durée d’exécution d’une migration ?

R. La durée de migration des données varie en fonction de la quantité de données et de l’option de mise à niveau sélectionnée.

## <a name="next-steps"></a>Étapes suivantes

* [Migrer des données d’un appareil StorSimple 8000 vers Azure File Sync](../storage/files/storage-files-migration-storsimple-8000.md)