---
title: Options de migration de données à partir des appareils de la gamme StorSimple 5000-7000
description: Fournit une vue d’ensemble des options pour migrer des données de la gamme StorSimple 5000-7000.
services: storsimple
documentationcenter: NA
author: alkohli
manager: twooley
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/15/2019
ms.author: alkohli
ms.openlocfilehash: 316b513f761aa422f87c83658e6c50efec25efb4
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94961139"
---
# <a name="options-to-migrate-data-from-storsimple-5000-7000-series"></a>Options pour migrer des données de la gamme StorSimple 5000-7000 

> [!IMPORTANT]
> La fin du support de la série StorSimple 5000/7000 interviendra le 9 juillet 2019. Nous recommandons aux clients de la série StorSimple 5000/7000 de migrer vers l’une des alternatives décrites dans le document.

La [fin de la prise en charge](https://support.microsoft.com/lifecycle/search?alpha=StorSimple%205000%2F7000%20Series) de la gamme StorSimple 5000-7000 est prévue en juillet 2019. Les clients qui exécutent la gamme StorSimple 5000-7000 peuvent mettre à niveau vers d’autres services hybrides internes Azure. Cet article décrit les options hybrides Azure disponibles pour migrer les données. 

## <a name="migration-options"></a>Options de migration

Les clients qui utilisent la gamme StorSimple 5000-7000 disposent d'options Azure ou tierces.

### <a name="azure-options"></a>Options Azure

#### <a name="upgrade-to-storsimple-8000-series"></a>Mettre à niveau vers la gamme StorSimple 8000

Procédez à une mise à niveau vers la gamme StorSimple 8000 et continuez ainsi sur la plateforme StorSimple.  Ce chemin d’accès de mise à niveau nécessite que les clients remplacent leurs appareils de la gamme 5000-7000 par un de la gamme 8000. Les données sont migrées à partir de l’appareil de la gamme 5000-7000 à l’aide de l’outil de migration. Une fois la migration terminée avec succès, les appareils de la gamme StorSimple 8000 continueront de classer les données dans le stockage Blob Azure. 

Pour plus d’informations sur la migration des données à l’aide d’un appareil de la gamme StorSimple 8000, accédez à [Migrer des données d’un appareil StorSimple 5000-7000 vers un appareil StorSimple 8000](storsimple-8000-migrate-from-5000-7000.md).

#### <a name="migrate-to-azure-file-sync"></a>Migrer vers Azure File Sync

Cette toute nouvelle option de migration permet aux clients de stocker les partages de fichiers de leur organisation dans Azure Files. Ces partages de fichiers sont ensuite centralisés pour un accès local à l’aide d’Azure File Sync (AFS). AFS peut être déployé sur un hôte Windows Server. La migration des données réelle est ensuite effectuée sous forme de copie hôte ou à l’aide de l’outil de migration.

Pour plus d’informations sur la migration des données vers Azure File Sync, accédez à [Migrer des données de la gamme StorSimple 5000-7000 vers Azure File Sync](../storage/files/storage-files-migration-storsimple-8000.md).

### <a name="third-party-options"></a>Options tierces

#### <a name="migrate-to-panzura-freedom-nas"></a>Migrer vers Panzura Freedom NAS

Les clients de StorSimple 5000-7000 peuvent choisir de migrer vers Panzura Freedom NAS pour conserver leurs données dans Azure. La solution Panzura Freedom fournit une solution NAS qui couvre les centres de données, les bureaux, ainsi que les clouds publics et privés. Elle prend en charge les workflows de données locaux, hybrides et dans le cloud pour les clients NFS, SMB et mobiles. 

Cette migration est assurée par Panzura. Les clients peuvent commencer en demandant la prise en charge de la migration sur le [site web Panzura](https://panzura.com/storsimple-migration/).

#### <a name="migrate-to-cohesity"></a>Migrer vers Cohesity

Cohesity vous permet de migrer des données de votre solution StorSimple 5000-7000 actuelle vers la plateforme de données Cohesity sur Azure. La plateforme de données Cohesity est une solution à définition logicielle à l’échelle du web qui consolide les fichiers, sauvegardes, objets et machines virtuelles en une seule solution native du cloud. À l’issue de la migration vers la plateforme de données, vous pouvez gérer, protéger et configurer les données et applications du cloud vers le noyau via un seul écran. Avec Cohesity, démarrez avec un minimum de trois nœuds. 

En savoir plus sur la [migration vers la plateforme de données Cohesity](https://info.cohesity.com/migrate-from-storsimple-to-cohesity.html).

#### <a name="migrate-to-nasuni"></a>Migrer vers Nasuni

Nasuni permet aux clients StorSimple 5000-7000 de migrer et de conserver facilement leurs données dans Azure.  Nasuni est une solution de stockage NAS basée sur Azure, offrant aux clients les performances et la sécurité qu’ils attendent d’une solution locale de premier plan, tout en bénéficiant des avantages économiques et de l’échelle du cloud.  Outre le stockage de fichiers hautes performances, Nasuni et Azure prennent en charge la sauvegarde et la récupération d’urgence, tout en vous permettant de partager et de collaborer sur vos données dans le monde entier grâce à la gestion centralisée du stockage de fichiers. 

Nasuni dispose de l’expérience nécessaire pour faciliter votre migration. Franchissez le pas dès aujourd’hui : https://info.nasuni.com/nasuni-storsimple-migration

#### <a name="migrate-to-talon-fast"></a>Migrer vers Talon FAST

Talon permet aux clients StorSimple 5000-7000 de continuer à bénéficier des précieux avantages offerts par la plateforme StorSimple (faible encombrement sur site et ressources cloud illimitées) avec encore plus de fonctions.  Avec la solution Talon FAST, les clients peuvent facilement migrer et conserver leurs données dans Azure. Avec un encombrement sur site encore plus réduit se limitant des logiciels, ils bénéficient de nouveaux avantages comme le verrouillage de fichiers global, l’espace de noms global et la collaboration multisite.  Talon est une solution d’écosystème Azure de premier plan qui permet à des clients du monde entier de migrer leurs charges de travail de serveurs de fichiers locaux pour bénéficier d’une empreinte Azure consolidé sans compromettre le workflow ou l’expérience utilisateur.  

Découvrez plus en détail comment consolider votre entreprise dans le cloud à l’adresse https://www.talonstorage.com/alliances/microsoft-storsimple.


## <a name="migration---frequently-asked-questions"></a>Migration - Forum aux questions

### <a name="q-when-do-the-storsimple-5000-and-7000-series-devices-reach-end-of-service"></a>Q. Quand la fin du service des appareils des gammes StorSimple 5000 et 7000 est-elle prévue ? 

R. La [fin du service](https://support.microsoft.com/lifecycle/search?alpha=StorSimple%205000%2F7000%20Series) de la gamme StorSimple 5000-7000 est prévue en juillet 2019. La fin du service implique que Microsoft ne sera plus en mesure d’assurer le support matériel et logiciel de ces appareils après juillet 2019. Nous vous recommandons vivement de commencer à envisager dès maintenant un plan pour migrer les données de vos appareils.

### <a name="q-what-happens-to-the-data-i-have-stored-in-azure"></a>Q. Que se passe-t-il pour les données que j’ai stockées dans Azure ?  

R. Vous pouvez continuer à utiliser les données dans Azure après avoir migré vers un service plus récent. 


### <a name="q-what-happens-to-the-data-i-have-stored-locally-on-my-storsimple-device"></a>Q. Que se passe-t-il pour les données que j’ai stockées localement sur mon appareil StorSimple ? 

R. Les données qui se trouvent sur l’appareil local peuvent être copiées vers le service plus récent, comme décrit dans les documents de migration.

### <a name="q-what-happens-if-i-want-to-keep-my-storsimple-50007000-series-appliance"></a>Q. Que se passe-t-il si je souhaite conserver mon appliance de la gamme StorSimple 5000/7000 ? 

R. Même si les services peuvent continuer à fonctionner, il se peut que Microsoft ne soit plus en mesure de fournir la prise en charge matérielle et logicielle. Une migration est vivement recommandée pour la continuité des activités.

### <a name="q-what-options-are-available-to-migrate-data-from-storsimple-5000-7000-series-devices"></a>Q. Quelles options sont disponibles pour migrer des données à partir d’appareils de la gamme StorSimple 5000-7000 ? 

R. En fonction de leur scénario, les utilisateurs de la gamme StorSimple 5000-7000 disposent des options de migration suivantes. 

 - **Mettre à niveau vers la gamme 8000** : utilisez cette option si vous souhaitez continuer sur la plateforme StorSimple. 
 - **Migrer vers Azure File Sync** : utilisez cette option si vous souhaitez basculer vers le format natif d’Azure. Vous pouvez utiliser Azure File Sync pour la gestion centralisée des partages de fichiers. 

Vous pouvez contacter le Support Microsoft pour discuter des options de migration non répertoriées ici.

### <a name="q-is-migration-to-other-storage-solutions-supported"></a>Q. La migration vers d’autres solutions de stockage est-elle prise en charge ?

R. Oui. La migration vers d’autres solutions de stockage à l’aide de la copie hôte des données est prise en charge.

### <a name="q-is-migration-supported-by-microsoft"></a>Q. La migration est-elle prise en charge par Microsoft ? 

R. La migration à partir de la gamme 5000 ou 7000 est une opération entièrement prise en charge. En réalité, Microsoft vous recommande de contacter le support avant de commencer la migration. La migration est actuellement une opération assistée. Si vous envisagez de migrer des données à partir de votre appareil de la gamme StorSimple 5000-7000, [ouvrez un ticket de support](storsimple-8000-contact-microsoft-support.md).

### <a name="q-what-is-the-pricing-model-for-both-the-migration-options"></a>Q. Quel est le modèle de tarification pour les deux options de migration ?

R. Le coût de la migration varie selon l’option choisie. Alors que la migration proprement dite est gratuite, si vous décidez de mettre à niveau vers la gamme StorSimple 8000, le coût correspondra à l’appareil matériel. 

De même, si vous utilisez Azure File Sync, des frais d’abonnement peuvent s’appliquer pour le service. Dans chaque cas, les clients devront également s’acquitter de coûts de stockage. Consultez les informations suivantes pour obtenir une estimation : 
- [Tarification de StorSimple](https://azure.microsoft.com/pricing/details/storsimple/)  
- [Tarification d’AFS]( https://azure.microsoft.com/pricing/details/storage/files/)

### <a name="q--how-long-does-it-take-to-complete-a-migration"></a>Q.  Quelle est la durée d’exécution d’une migration ?

R. La durée de migration des données varie en fonction de la quantité de données et de l’option de mise à niveau sélectionnée. 

### <a name="q-what-is-the-end-of-support-date-for-storsimple-8000-series"></a>Q. Quelle est la date de fin de support pour la gamme StorSimple 8000 ?

R. La date de fin de support pour la gamme StorSimple 8000 est publiée [ici](https://support.microsoft.com/lifecycle/search?alpha=Azure%20StorSimple%208000%20Series).


## <a name="next-steps"></a>Étapes suivantes
 - [Migrer des données d’un appareil StorSimple 5000-7000 vers un appareil StorSimple 8000](storsimple-8000-migrate-from-5000-7000.md).
 - [Migrer des données d’un appareil StorSimple 5000-7000 vers Azure File Sync](../storage/files/storage-files-migration-storsimple-8000.md)