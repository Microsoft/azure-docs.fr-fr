---
title: Basculement et récupération d’urgence pour un appareil de la gamme StorSimple 8000
description: Découvrez comment basculer votre appareil StorSimple vers lui-même, un autre appareil physique ou une appliance cloud.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/03/2017
ms.author: alkohli
ms.openlocfilehash: dffa059b18e159d04b5e3bb8555dabf801ede692
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017115"
---
# <a name="failover-and-disaster-recovery-for-your-storsimple-8000-series-device"></a>Basculement et récupération d’urgence pour votre appareil de la gamme StorSimple 8000

## <a name="overview"></a>Vue d’ensemble

Cet article décrit la fonctionnalité de basculement d’appareil pour les appareils de la gamme StorSimple 8000 et comment cette fonctionnalité peut être utilisée pour récupérer les appareils StorSimple si un incident se produit. StorSimple utilise le basculement d’appareil pour migrer des données d’un appareil source dans le centre de données vers un autre appareil cible. Les instructions de cet article s’appliquent aux appareils physiques de gamme StorSimple 8000 et aux appliances cloud exécutant des versions logicielles Update 3 et versions ultérieures.

StorSimple utilise le panneau **Appareils** pour lancer la fonction de basculement d’appareil en cas d’incident. Ce panneau répertorie tous les appareils StorSimple connectés à votre service StorSimple Device Manager.

![Panneau Appareils](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev1.png)


## <a name="disaster-recovery-dr-and-device-failover"></a>Récupération d’urgence et basculement d’appareil

Dans un scénario de récupération d’urgence, l’appareil principal cesse de fonctionner. StorSimple utilise l’appareil principal comme _source_ et déplace les données cloud associées vers un autre _appareil_ cible. Ce processus est appelé le *basculement*. Le graphique suivant illustre le processus de basculement.

![Que se passe-t-il dans le basculement de l’appareil ?](./media/storsimple-8000-device-failover-disaster-recovery/failover-dr-flow.png)

L’appareil cible pour un basculement peut être un appareil physique ou même une appliance cloud. L’appareil cible peut se trouver dans le même ou un autre emplacement géographique que l’appareil source.

Pendant le basculement, vous pouvez sélectionner des conteneurs de volumes pour la migration. StorSimple modifie alors l’appartenance de ces conteneurs de volumes de l’appareil source vers l’appareil cible. Une fois l’appartenance des conteneurs de volumes modifiée, StorSimple supprime ces conteneurs de l’appareil source. Une fois la suppression effectuée, vous pouvez restaurer automatiquement l’appareil cible. _La restauration automatique_ transfère l’appartenance à l’appareil source d’origine.

### <a name="cloud-snapshot-used-during-device-failover"></a>Capture instantanée cloud utilisée pendant le basculement de l’appareil

Après une récupération d’urgence, la sauvegarde cloud la plus récente est utilisée pour restaurer les données sur l’appareil cible. Pour plus d’informations sur les captures instantanées cloud, consultez [Take a manual backup](storsimple-8000-manage-backup-policies-u2.md#take-a-manual-backup) (Effectuer une sauvegarde manuelle).

Sur un appareil de la gamme StorSimple 8000, les stratégies de sauvegarde sont associées à des sauvegardes. S’il existe plusieurs stratégies de sauvegarde pour le même volume, StorSimple sélectionne la stratégie de sauvegarde avec le plus grand nombre de volumes. StorSimple utilise alors la sauvegarde la plus récente à partir de la stratégie de sauvegarde sélectionnée pour restaurer les données sur l’appareil cible.

Supposons qu’il existe deux stratégies de sauvegarde, *defaultPol* et *customPol* :

* *defaultPol* : un seul volume, *vol1*, s’exécute tous les jours à 22h30.
* *customPol* : quatre volumes, *vol1*, *vol2*, *vol3* et *vol4*, s’exécutent tous les jours à 22h00.

Dans ce cas, StorSimple donne la priorité pour des raisons de cohérence en cas d’incident et utilise *customPol*, car elle a plus de volumes. La sauvegarde la plus récente de cette stratégie est utilisée pour restaurer les données. Pour plus d’informations sur la façon de créer et gérer des stratégies de sauvegarde, accédez à [Use the StorSimple Device Manager service in Azure portal to manage backup policies](storsimple-8000-manage-backup-policies-u2.md) (Utiliser le service StorSimple Device Manager dans le portail Azure pour gérer les stratégies de sauvegarde).

## <a name="common-considerations-for-device-failover"></a>Considérations courantes relatives au basculement d’appareil

Avant de faire basculer un appareil, lisez les informations suivantes :

* Avant le début du basculement d’un appareil, tous les volumes dans les conteneurs de volumes doivent être hors connexion. Dans un basculement non planifié, les volumes StorSimple seront automatiquement mis hors connexion. Toutefois, si vous effectuez un basculement planifié (pour tester la récupération d’urgence), vous devrez mettre tous les volumes hors connexion.
* Seuls les conteneurs de volumes ayant une capture instantanée cloud associée sont répertoriés pour la récupération d’urgence. Il doit y avoir au moins un conteneur de volumes avec une capture instantanée cloud associée pour récupérer des données.
* S’il existe des captures instantanées cloud qui s’étendent sur plusieurs conteneurs de volumes, StorSimple bascule ces conteneurs de volumes comme un ensemble. Dans quelques cas, s’il existe des captures instantanées locales qui s’étendent sur plusieurs conteneurs de volumes, mais que les captures instantanées cloud associées ne le font pas, StorSimple bascule les captures instantanées locales, et les données locales sont perdues après la récupération d’urgence.
* Les appareils cibles disponibles pour la récupération d’urgence possèdent un espace suffisant pour accueillir les conteneurs de volume sélectionnés. Tous les appareils ne disposant pas de suffisamment d’espace ne sont pas répertoriés en tant qu’appareils cibles.
* Après une récupération d’urgence (pour une durée limitée), les performances d’accès aux données peuvent être affectées de manière significative, car l’appareil doit accéder aux données à partir du cloud et les stocker localement.

#### <a name="device-failover-across-software-versions"></a>Basculement de l'appareil entre les versions du logiciel

Un service StorSimple Device Manager dans un déploiement peut avoir plusieurs appareils physiques et cloud, exécutant tous des versions différentes du logiciel.

Utilisez le tableau suivant pour déterminer si vous pouvez basculer ou effectuer une restauration automatique vers un autre appareil exécutant une version logicielle différente, ainsi que le comportement des types de volumes pendant la récupération d’urgence.

#### <a name="failover-and-failback-across-software-versions"></a>Basculement et restauration automatique entre les versions logicielles

| Basculement/restauration automatique depuis | Appareil physique | Appliance cloud |
| --- | --- | --- |
| Update 3 à Update 4 |Les volumes hiérarchisés sont basculés en tant qu’éléments hiérarchisés. <br></br>Les volumes épinglés localement sont basculés en tant qu’éléments épinglés localement. <br></br> Après un basculement, lorsque vous prenez une capture instantanée sur l’appareil Update 4, le [suivi basé sur une carte thermique](storsimple-update4-release-notes.md#whats-new-in-update-4) intervient. |Les volumes épinglés localement sont basculés en tant qu’éléments hiérarchisés. |
| Update 4 à Update 3 |Les volumes hiérarchisés sont basculés en tant qu’éléments hiérarchisés. <br></br>Les volumes épinglés localement sont basculés en tant qu’éléments épinglés localement. <br></br> Sauvegardes utilisées pour restaurer les métadonnées de carte thermique conservées. <br></br>Le suivi basé sur une carte thermique n’est pas disponible dans Update 3 après une restauration automatique. |Les volumes épinglés localement sont basculés en tant qu’éléments hiérarchisés. |


## <a name="device-failover-scenarios"></a>Scénario de basculement d’appareil

En cas d’incident, vous pouvez choisir de basculer votre appareil StorSimple :

* [vers un appareil physique](storsimple-8000-device-failover-physical-device.md) ;
* [vers lui-même](storsimple-8000-device-failover-same-device.md) ;
* [vers une appliance cloud](storsimple-8000-device-failover-cloud-appliance.md).

Les articles ci-dessus fournissent des procédures détaillées pour chacun des cas de basculement précédents.


## <a name="failback"></a>Restauration automatique

Pour Update 3 et les versions ultérieures, StorSimple prend également en charge la restauration automatique. La restauration automatique est simplement l’inverse du basculement, la cible devient la source et l’appareil source d’origine pendant le basculement devient maintenant l’appareil cible. 

Lors de la restauration automatique, StorSimple resynchronise les données vers l’emplacement principal, arrête les E/S et l’activité de l’application et effectue la transition vers l’emplacement d’origine.

Une fois un basculement terminé, StorSimple effectue les actions suivantes :

* Il nettoie les conteneurs de volumes basculés de l’appareil source.
* Il démarre un travail en arrière-plan par conteneur de volumes (basculé) sur l’appareil source. Si vous tentez d’effectuer une restauration automatique lorsque le travail est en cours, vous recevez une notification à cet effet. Attendez la fin du travail pour démarrer la restauration automatique.
* Le temps nécessaire à la suppression des conteneurs de volumes dépend de divers facteurs tels que la quantité de données, l’ancienneté des données, le nombre de sauvegardes et la bande passante réseau disponible pour l’opération.

Si vous envisagez de tester les basculements ou restaurations automatiques, nous vous recommandons de tester des conteneurs de volumes comprenant moins de données (Go). Généralement, vous pouvez démarrer la restauration automatique 24 heures après la fin du basculement.

## <a name="frequently-asked-questions"></a>Forum aux questions

Q. **Que se passe-t-il si la récupération d’urgence échoue ou réussit partiellement ?**

R. Si la récupération d’urgence échoue, nous vous recommandons d’essayer à nouveau. Le travail de basculement du deuxième appareil tient compte de la progression du premier travail et commence à partir de là.

Q. **Puis-je supprimer un appareil pendant son basculement ?**

R. Vous ne pouvez pas supprimer un appareil lorsqu’une récupération d’urgence est en cours. Vous ne pourrez le faire qu’après la récupération d’urgence. Vous pouvez surveiller la progression du travail de basculement de l’appareil dans le panneau **Travaux**.

Q. **Quand commence le nettoyage de la mémoire (garbage collection) sur l’appareil source pour en supprimer les données locales ?**

R. Le nettoyage de la mémoire n’est activé sur l’appareil source qu’une fois l’appareil entièrement nettoyé. Le nettoyage inclut le nettoyage des objets qui ont échoué sur l’appareil source, notamment les volumes, objets de sauvegarde (et non les données), conteneurs de volumes et stratégies.

Q. **Que se passe-t-il en cas d’échec du travail de suppression associé aux conteneurs de volumes dans l’appareil source ?**

R.  Si le travail de suppression échoue, vous pouvez supprimer manuellement les conteneurs de volumes. Dans le panneau **Appareils**, sélectionnez votre appareil source, puis cliquez sur **Conteneurs de volumes**. Sélectionnez les conteneurs de volumes que vous avez basculés et, en bas du panneau, cliquez sur **Supprimer**. Une fois que vous avez supprimé tous les conteneurs de volumes basculés sur l’appareil source, vous pouvez démarrer la restauration automatique. Pour plus d’informations, accédez à [Delete a volume container](storsimple-8000-manage-volume-containers.md#delete-a-volume-container) (Supprimer un conteneur de volumes).

## <a name="business-continuity-disaster-recovery-bcdr"></a>Continuité d’activité et récupération d’urgence (Business Continuity Disaster Recovery - BCDR)

Un scénario de continuité d’activité et récupération d’urgence (BCDR) se produit lorsque l’ensemble du centre de données Azure cesse de fonctionner. Ce scénario peut affecter votre service StorSimple Device Manager et les appareils StorSimple associés.

Si un appareil StorSimple était inscrit juste avant un incident, cet appareil devra peut-être subir une réinitialisation des paramètres. Après l’incident, l’appareil StorSimple s’affiche dans le portail Azure comme étant hors connexion. Cet appareil doit être supprimé à partir du portail. Réinitialisez l’appareil aux réglages d’usine par défaut et inscrivez-le à nouveau auprès du service.

## <a name="next-steps"></a>Étapes suivantes

Si vous êtes prêt à effectuer un basculement d’appareil, choisissez un des scénarios suivants pour obtenir des instructions détaillées :

- [Basculement vers un autre appareil physique](storsimple-8000-device-failover-physical-device.md)
- [Basculer vers le même appareil](storsimple-8000-device-failover-same-device.md)
- [Basculer vers StorSimple Cloud Appliance](storsimple-8000-device-failover-cloud-appliance.md)

Si vous avez basculé votre appareil, choisissez l’une des options suivantes :

* [Deactivate and delete a StorSimple device](storsimple-8000-deactivate-and-delete-device.md) (Désactiver et supprimer un appareil StorSimple).
* [Use the StorSimple Device Manager service to administer your StorSimple device](storsimple-8000-manager-service-administration.md) (Utiliser le service StorSimple Device Manager pour gérer votre appareil StorSimple)

