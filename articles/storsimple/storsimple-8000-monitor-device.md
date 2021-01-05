---
title: Superviser votre appareil StorSimple 8000 Series
description: Décrit comment utiliser le service StorSimple Device Manager pour surveiller l’utilisation, les performances E/S et l’utilisation de la capacité de l’appareil.
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 10/17/2017
ms.author: alkohli
ms.openlocfilehash: de4cae90f2cc1223271672d9fc5ba5b42e4409e4
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94962431"
---
# <a name="use-the-storsimple-device-manager-service-to-monitor-your-storsimple-device"></a>Utilisation du service StorSimple Device Manager pour surveiller votre appareil StorSimple

## <a name="overview"></a>Vue d’ensemble
Vous pouvez utiliser le service StorSimple Device Manager pour surveiller des appareils spécifiques dans votre solution StorSimple. Vous pouvez créer des graphiques personnalisés basés sur les performances E/S, l’utilisation de la capacité, le débit du réseau et les mesures de performances de l’appareil, et les épingler au tableau de bord. Pour plus d’informations, consultez la section décrivant comment [personnaliser votre tableau de bord sur le portail](../azure-portal/azure-portal-dashboards.md).

Pour afficher les informations de surveillance d’un appareil donné, sélectionnez le service StorSimple Device Manager dans le portail Azure. Sélectionnez votre appareil dans la listes des appareils et accédez à la zone **Moniteur**. Vous pouvez alors voir les graphiques de **capacité**, d’**utilisation** et de **performances** de l’appareil sélectionné.

## <a name="capacity"></a>Capacité
La section **Capacité** effectue le suivi de l’espace approvisionné et de l’espace restant sur l’appareil. La capacité restante s’affiche en dessous, selon deux types : épinglé localement ou hiérarchisé.

La capacité approvisionnée et la capacité restante sont ensuite classées par volumes épinglés localement et hiérarchisés. La capacité approvisionnée et la capacité restante sur l’appareil s’affichent pour chaque volume.

![Capacité d’E/S](./media/storsimple-8000-monitor-device/device-capacity.png)



## <a name="usage"></a>Usage
La section **Utilisation** effectue le suivi des mesures relatives à la quantité d’espace de stockage des données qui est utilisé par les volumes, les conteneurs de volumes ou l’appareil. Vous pouvez créer des rapports basés sur l’utilisation de la capacité du stockage principal, du stockage cloud ou du stockage de l’appareil. L’utilisation de la capacité peut être mesurée sur un volume spécifique, un conteneur de volumes spécifique ou tous les conteneurs de volumes.
Par défaut, l’utilisation des dernières 24 heures est indiquée. Vous pouvez modifier le graphique et sélectionner la durée pendant laquelle l’utilisation est enregistrée parmi les options suivantes :
* Dernières 24 heures
* 7 derniers jours
* 30 derniers jours
* 90 derniers jours
* Année dernière

La croissance et la plage des deux clés mesure sont rapportées dans les graphiques d’utilisation. La plage représente la valeur maximum et les valeurs minimums d’utilisation rapportées durant la période choisie (pour les 7 derniers jours, par exemple).

La croissance représente l’augmentation d’utilisation du premier au dernier jour de la période choisie. 

La croissance et la plage peuvent aussi être représentées par les équations suivantes :

```
Range = {Usage(minimum), Usage(maximum)}

Growth = Usage(Last day) - Usage(first day)

Growth (%) = [{Usage(last day) - Usage(first day)} X 100]/Usage(first day)
```

Le stockage principal, cloud et local utilisé peut être décrit comme suit :

### <a name="primary-storage-usage"></a>Utilisation du stockage principal
Ces graphiques affichent la quantité de données écrites sur des volumes StorSimple avant que les données soient dédupliquées et compressées. Vous pouvez afficher le stockage principal utilisé par tous les volumes d’un conteneur de volumes, ou par un seul volume. Le stockage principal utilisé est ensuite classé en deux catégories : stockage principal hiérarchisé utilisé et stockage principal épinglé localement utilisé.

Les graphiques suivants montrent le stockage principal utilisé sur un appareil StorSimple avant et après un instantané cloud. Étant donné qu’il s’agit uniquement des données de volumes, un instantané cloud ne doit pas modifier le stockage principal. Comme vous pouvez le voir, le graphique n’affiche aucune différence dans le stockage principal hiérarchisé ou épinglé localement utilisé suite à un instantané cloud. L’instantané cloud a démarré à environ 23h50 sur l’appareil.

![Utilisation de la capacité principale après l’instantané cloud](./media/storsimple-8000-monitor-device/device-primary-storage-after-cloudsnapshot.png)

Si vous exécutez maintenant des E/S sur l’hôte connecté à votre appareil StorSimple, vous verrez une augmentation du stockage principal hiérarchisé ou localement épinglé utilisé en fonction des volumes (hiérarchisés ou épinglés localement) sur lesquels les données sont écrites. Voici les graphiques sur l’utilisation du stockage principal pour un appareil StorSimple. Sur cet appareil, l’hôte StorSimple a commencé à envoyer les écritures à environ 14h30 sur un volume hiérarchisé de l’appareil. Vous pouvez voir le pic dans les octets en écriture par seconde correspondant aux E/S s’exécutant sur l’hôte.

![Performances avec E/S exécutées sur des volumes hiérarchisés](./media/storsimple-8000-monitor-device/device-io-from-initiator.png)

Observons le stockage principal hiérarchisé utilisé. Il a augmenté alors que l’utilisation du stockage principal épinglé localement n’a pas changé : aucune opération d’écriture n’est envoyée au niveau des volumes épinglés localement sur l’appareil.

![Utilisation de la capacité principale lorsque les E/S s’exécutent sur les volumes hiérarchisés](./media/storsimple-8000-monitor-device/device-primary-storage-io-from-initiator.png)

Si vous exécutez l’Update 3 ou une version ultérieure, vous pouvez afficher l’utilisation de la capacité de stockage principale par volume individuel, pour tous les volumes, pour tous les volumes hiérarchisés et pour tous les volumes épinglés localement, comme indiqué ci-dessous. Le fait de diviser la capacité de stockage entre tous les volumes épinglés localement vous permet de déterminer rapidement le niveau d’utilisation d’une couche locale.

![Utilisation de la capacité principale pour tous les volumes hiérarchisés](./media/storsimple-8000-monitor-device/monitor-usage3.png)

![Utilisation de la capacité principale pour tous les volumes épinglés localement](./media/storsimple-8000-monitor-device/monitor-usage4.png)

Vous pouvez cliquer sur chacun des volumes de la liste et consulter l’utilisation correspondante.

![Utilisation de la capacité principale pour tous les volumes épinglés localement 2](./media/storsimple-8000-monitor-device/device-primary-storage-usage-by-volume.png)

### <a name="cloud-storage-usage"></a>Utilisation du stockage cloud
Ces graphiques montrent la quantité de stockage cloud utilisé. Les données sont dédupliquées et compressées. Cette quantité inclut les instantanés cloud qui peuvent contenir des données qui ne sont pas reflétées dans un volume principal et qui sont conservées à des fins de rétention obligatoire ou héritée. Vous pouvez comparer les chiffres de la consommation du stockage principal et du stockage cloud pour obtenir une idée du taux de réduction des données, même si le chiffre ne sera pas exact.

Les graphiques suivants montrent l’utilisation du stockage cloud d’un appareil StorSimple suite à un instantané cloud.

* L’instantané cloud a démarré à environ 11h50 sur cet appareil et vous pouvez constater qu’avant cet instantané, aucun stockage cloud n’était utilisé. 
* Une fois l’instantané cloud pris, l’utilisation du stockage cloud est passée à 0,89 Go. 
* Pendant la prise de l’instantané cloud, vous pouvez également observer un pic d’E/S de l’appareil au cloud.

    ![Utilisation du stockage cloud avant l’instantané cloud](./media/storsimple-8000-monitor-device/device-cloud-storage-before-cloudsnapshot.png)

    ![Utilisation du stockage cloud après l’instantané cloud](./media/storsimple-8000-monitor-device/device-cloud-storage-after-cloudsnapshot.png)

    ![E/s de l’appareil au cloud pendant un instantané cloud](./media/storsimple-8000-monitor-device/device-io-to-cloud-during-cloudsnapshot.png)


### <a name="local-storage-usage"></a>Utilisation du stockage local
Ces graphiques affichent l’utilisation totale pour l’appareil. Celle-ci est supérieure à l’utilisation du stockage principal, car elle inclut le niveau linéaire SSD. Ce niveau contient une quantité de données qui existe également sur les autres niveaux de l’appareil. La capacité du niveau linéaire SSD est cyclique, de sorte que lors de la réception de nouvelles données, les anciennes données sont déplacées vers le niveau HDD (elles sont alors dédupliquées et compressées).

Au fil du temps, le stockage principal et le stockage local utilisés augmenteront en parallèle jusqu’à ce que les données commencent à être transférées vers le cloud. À ce stade, le stockage principal utilisé commencera sans doute à stagner, mais le stockage principal utilisé augmentera à mesure que davantage de données sont écrites.

Les graphiques suivants montrent le stockage principal utilisé pour un appareil StorSimple suite à un instantané cloud. L’instantané cloud a démarré à 11h50 et le stockage local a commencé à diminuer à ce moment-là. Le stockage local utilisé est passé de 1,445 Go à 1,09 Go. Cela indique très probablement que des données non compressées au niveau linéaire SSD ont été dédupliquées, compressées et déplacées vers le niveau de disque dur. Notez que si l’appareil possède déjà une grande quantité de données dans les niveaux de SSD et de disque dur, vous ne verrez pas cette diminution. Dans cet exemple, l’appareil a une petite quantité de données.

![Utilisation du stockage local après l’instantané cloud](./media/storsimple-8000-monitor-device/device-local-storage-after-cloudsnapshot.png)

## <a name="performance"></a>Performances
La section **Performances** effectue le suivi des mesures relatives au nombre d’opérations de lecture et d’écriture entre les interfaces de l’initiateur iSCSI sur le serveur hôte et l’appareil ou entre l’appareil et le cloud. Ces performances peuvent être mesurées pour un volume spécifique, un conteneur de volumes spécifique ou tous les conteneurs de volumes. Elles incluent également l’utilisation du processeur et le débit du réseau pour les diverses interfaces réseau de votre appareil.

### <a name="io-performance-for-initiator-to-device"></a>Performances d’E/S entre l’initiateur et l’appareil
Le tableau ci-dessous montre les E/S pour l’initiateur vers votre appareil pour tous les volumes d’un appareil de production. Les mesures tracées correspondent aux octets en lecture et en écriture par seconde. Vous pouvez également tracer les E/S de lecture, d’écriture et en attente, ou les latences de lecture et d’écriture.

![Performances d’E/S entre l’initiateur et l’appareil](./media/storsimple-8000-monitor-device/device-io-from-initiator.png)

### <a name="io-performance-for-device-to-cloud"></a>Performances d’E/S entre l’appareil et le cloud
Pour le même appareil, les opérations d’E/S sont tracées pour les données de l’appareil vers le cloud pour tous les conteneurs de volumes. Sur cet appareil, les données sont uniquement dans la couche linéaire et aucune n’est dispersée dans le cloud. Aucune opération en lecture seule n’émane de l’appareil vers le cloud. Par conséquent, les pics du graphique sont séparés par un intervalle de cinq minutes, ce qui correspond à la fréquence à laquelle la pulsation est vérifiée entre l’appareil et le service.

Pour le même appareil, un instantané cloud a été pris pour les données de volume, avec une heure de début à 11h50. Cela a entraîné un flux de données de l’appareil vers le cloud. Des opérations d’écriture ont été prises en charge pour le cloud dans cette durée. Le graphique d’E/S affiche un pic dans les octets en écriture par seconde, qui correspond à l’heure où l’instantané a été pris.

![E/s de l’appareil au cloud pendant un instantané cloud](./media/storsimple-8000-monitor-device/device-io-to-cloud-during-cloudsnapshot.png)

### <a name="network-throughput-for-device-network-interfaces"></a>Débit du réseau pour les interfaces réseau de l’appareil
**Débit du réseau** : cette option assure le suivi des mesures relatives à la quantité de données transférées entre les interfaces réseau de l’initiateur iSCSI sur le serveur hôte et l’appareil, et entre l’appareil et le cloud. Vous pouvez surveiller cette mesure pour chacune des interfaces réseau iSCSI sur votre appareil.

Les graphiques suivants montrent le débit du réseau Data 0, 1 1 GbE sur votre appareil, à la fois activé pour le cloud (par défaut) et compatible iSCSI. Sur cet appareil, le 14 juin à environ 21h, les données ont été hiérarchisées dans le cloud (aucun instantané cloud n’a été effectué à ce moment-là, ce qui montre que la hiérarchisation est le mécanisme utilisé pour déplacer les données dans le cloud), ce qui a entraîné le traitement des E/S sur le cloud. Un pic équivalent apparaît dans le graphique du débit de réseau à la même heure et la plus grande partie du trafic réseau correspond à du trafic sortant vers le cloud.

![Débit du réseau pour Data 0](./media/storsimple-8000-monitor-device/device-network-throughput-data0.png)

Si nous examinons le graphique de débit de l’interface Data 1, une autre interface réseau 1 GbE uniquement compatible iSCSI, nous remarquons un trafic réseau quasi nul pendant cette durée.

![Débit du réseau pour Data 1](./media/storsimple-8000-monitor-device/device-network-throughput-data1.png)


## <a name="cpu-utilization-for-device"></a>Utilisation du processeur pour l’appareil
La section **Utilisation de l’UC** effectue le suivi des mesures relatives à l’utilisation du processeur sur votre appareil. Le tableau suivant affiche les statistiques d’utilisation du processeur pour un appareil en production.

![Utilisation du processeur pour l’appareil](./media/storsimple-8000-monitor-device/device-cpu-utilization.png)



## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [utiliser le tableau de bord de l’appareil du service StorSimple Device Manager](./storsimple-8000-device-dashboard.md).
* Découvrez comment [utiliser le service StorSimple Device Manager pour gérer votre appareil StorSimple](./storsimple-8000-manager-service-administration.md).