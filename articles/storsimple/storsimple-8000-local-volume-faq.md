---
title: FAQ sur les volumes StorSimple épinglés localement | Microsoft Docs
description: Fournit des réponses aux questions fréquemment posées sur les volumes StorSimple épinglés localement.
services: storsimple
documentationcenter: NA
author: manuaery
manager: syadav
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/26/2017
ms.author: manuaery
ms.openlocfilehash: 483fa81b409e1bd740af85b431a86b6c814831e6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002712"
---
# <a name="storsimple-locally-pinned-volumes-frequently-asked-questions-faq"></a>Volumes StorSimple épinglés localement : forum aux questions (FAQ)
## <a name="overview"></a>Vue d’ensemble
Voici quelques questions/réponses qui pourraient vous être utiles lorsque vous créez un volume StorSimple épinglé localement, convertissez un volume à plusieurs niveaux en volume épinglé localement (et inversement), ou sauvegardez et restaurez un volume épinglé localement.

Les questions/réponses sont organisées en plusieurs catégories :

* Création d’un volume épinglé localement
* Sauvegarde d’un volume épinglé localement
* Conversion d’un volume à plusieurs niveaux en volume épinglé localement
* Restauration d’un volume épinglé localement
* Basculement d’un volume épinglé localement

## <a name="questions-about-creating-a-locally-pinned-volume"></a>Questions sur la création d’un volume épinglé localement
**Q.** Quelle est la taille maximale des volumes épinglés localement que je peux créer sur les appareils de la gamme 8000 ?

**R.** Sur les appareils exécutant la gamme StorSimple 8000 Update 3.0, vous pouvez approvisionner des volumes épinglés localement jusqu'à 8,5 To ou des volumes hiérarchisés jusqu'à 200 To sur l’appareil 8100. Sur l’appareil 8600, qui a une plus grande capacité, vous pouvez configurer des volumes épinglés localement d’une taille maximale de 22,5 To ou des volumes à plusieurs niveaux d’une taille maximale de 500 To.

**Q.** J’ai récemment mis à niveau mon appareil 8100 vers Update 3.0 et, lorsque j’essaie de créer un volume épinglé localement, la taille maximale disponible est de seulement 6 To, au lieu de 8,5 To. Pourquoi ne puis-je pas créer un volume de 8,5 To ?

**R.** Si votre appareil exécute la version Update 3.0, vous pouvez configurer des volumes épinglés localement d’une taille maximale de 8,5 To OU des volumes à plusieurs niveaux d’une taille maximale de 200 To sur l’appareil 8100. Si votre appareil dispose déjà de volumes à plusieurs niveaux, l’espace disponible pour la création d’un volume épinglé localement sera proportionnellement inférieur à cette limite maximale. Par exemple, si environ 106 To de volumes à plusieurs niveaux ont déjà été configurés sur votre appareil 8100 (ce qui représente la moitié de la capacité à plusieurs niveaux), la taille maximale du volume local que vous pouvez créer sur l’appareil 8100 sera ramenée à 4 To (soit à peu près la moitié de la valeur maximale de la capacité pour les volumes épinglés localement).

Dans la mesure où une partie de l’espace local de l’appareil est utilisée pour héberger la plage de travail de volumes hiérarchisés, l’espace disponible pour la création d’un volume épinglé localement est réduit si l’appareil dispose de volumes hiérarchisés. À l’inverse, la création d’un volume épinglé localement réduit proportionnellement l’espace disponible pour les volumes hiérarchisés. Les tableaux suivants résument la capacité disponible pour les volumes à plusieurs niveaux sur les appareils 8100 et 8600 lorsque des volumes épinglés localement ont été créés.

#### <a name="update-30"></a>Update 3.0 

| Capacité configurée pour les volumes épinglés localement | Capacité disponible pour la configuration pour des volumes à plusieurs niveaux - 8100 | Capacité disponible pour la configuration pour des volumes à plusieurs niveaux - 8600 |
| --- | --- | --- |
| 0 |200 To |500 To |
| 1 To |176,5 To |477,8 To |
| 4 To |105,9 To |411,1 To |
| 8,5 To |0 To |311,1 To |
| 10 To |N/D |277,8 To |
| 15 To |N/D |166,7 To |
| 22,5 To |N/D |0 To |

**Q.** Pourquoi la création d’un volume épinglé localement est-elle une opération longue ?

**A.** Les volumes épinglés localement sont configurés complètement. Pour créer de l’espace sur les niveaux locaux de l’appareil, certaines données des volumes à plusieurs niveaux existants peuvent être envoyées dans le cloud pendant la configuration. Et étant donné que cela dépend de la taille du volume à configurer, des données existantes sur votre appareil et de la bande passante disponible vers le cloud, la création d’un volume local peut nécessiter plusieurs heures.

**Q.** Combien de temps faut-il pour créer un volume épinglé localement ?

**A.** Étant donné que les volumes épinglés localement sont configurés complètement, des données existantes sur les volumes à plusieurs niveaux peuvent être envoyées dans le cloud pendant la configuration. Par conséquent, le temps nécessaire pour créer un volume épinglé localement dépend de plusieurs facteurs, notamment la taille du volume, les données sur votre appareil et la bande passante disponible. Sur un appareil installé récemment et qui ne possède aucun volume, le temps nécessaire à la création d’un volume épinglé localement est d’environ 10 minutes par téraoctet de données. Toutefois, la création de volumes locaux peut prendre plusieurs heures sur un appareil en cours d’utilisation, en fonction des facteurs décrits ci-dessus.

**Q.** Je souhaite créer un volume épinglé localement. Existe-t-il des meilleures pratiques que je dois connaître ?

**A.** Les volumes épinglés localement sont adaptés aux charges de travail qui requièrent des garanties locales sur les données en permanence et sont sensibles à la latence du cloud. Lorsque vous envisagez d’utiliser des volumes locaux pour l’une de vos charges de travail, n’oubliez pas les éléments suivants :

* Les volumes épinglés localement sont approvisionnés de façon traditionnelle et la création de volumes locaux a un impact sur l’espace disponible pour les volumes hiérarchisés. Par conséquent, nous vous conseillons de commencer par des volumes de petite taille et de monter en puissance au fur et à mesure que vos besoins de stockage augmentent.
* La configuration des volumes locaux est une opération longue qui peut impliquer l’envoi de données existantes à partir de volumes à plusieurs niveaux vers le cloud. Par conséquent, les performances sur ces volumes peuvent se trouver amoindries.
* La configuration des volumes locaux est une opération longue. Le temps réellement nécessaire dépend de plusieurs facteurs : la taille du volume en cours de configuration, les données sur votre appareil et la bande passante disponible. Si vous n’avez pas sauvegardé vos volumes existants dans le cloud, la création de volume est plus lente. Nous vous suggérons de prendre des instantanés cloud de vos volumes existants avant de configurer un volume local.
* Vous pouvez convertir des volumes à plusieurs niveaux existants en volumes épinglés localement. Cette conversion implique la configuration d’espace sur l’appareil pour le volume épinglé localement ainsi créé (en plus de mettre hors fonction les données à plusieurs niveaux du cloud, le cas échéant). Là encore, il s’agit d’une opération longue qui dépend des facteurs dont nous avons parlé plus haut. Nous vous suggérons de sauvegarder vos volumes existants avant la conversion, car le processus sera encore plus lent si des volumes existants n’ont pas été sauvegardés. Les performances de votre appareil peuvent également être limitées pendant ce processus.

Plus d’informations sur comment [créer un volume épinglé localement](storsimple-8000-manage-volumes-u2.md#add-a-volume)

**Q.** Puis-je créer plusieurs volumes épinglés localement simultanément ?

**A.** Oui, mais tous les travaux de création et d’extension d’un volume épinglé localement sont traités de manière séquentielle.

Les volumes épinglés localement étant configurés complètement, ils nécessitent la création d’espace local sur l’appareil (ce qui peut entraîner l’envoi de données existantes de volumes à plusieurs niveaux vers le cloud pendant la configuration). Par conséquent, si une tâche de configuration est en cours, d’autres tâches de création de volumes locaux seront mises en attente jusqu’à ce que cette tâche soit terminée.

De même, si un volume local existant est étendu ou qu’un volume hiérarchisé est converti en volume épinglé localement, la création d’un nouveau volume épinglé localement est mise en attente jusqu’à ce que le travail précédent soit terminé. L’augmentation de la taille d’un volume épinglé localement implique l’extension de l’espace local existant pour ce volume. La conversion d’un volume à plusieurs niveaux en volume épinglé localement implique également la création d’espace local pour le volume épinglé localement ainsi créé. Dans ces deux opérations, la création ou l’extension de l’espace local est une tâche de longue haleine.

Vous pouvez visualiser ces tâches sur le panneau **Tâches** du service StorSimple Device Manager. Le travail en cours de traitement est mis à jour en continu pour refléter l’avancement de l’approvisionnement de l’espace. Les autres travaux sur les volumes épinglés localement sont marqués comme en cours d’exécution, mais leur progression est bloquée et ils sont pris en charge dans l’ordre où ils ont été mis en file d’attente.

**Q.** J’ai supprimé un volume épinglé localement. Pourquoi l’espace ainsi récupéré n’est-il pas reflété dans l’espace disponible lorsque j’essaie de créer un nouveau volume ?

**A.** Si vous supprimez un volume épinglé localement, l'espace disponible pour les nouveaux volumes ne sera peut-être pas mis à jour immédiatement. Le service StorSimple Device Manager met à jour l’espace local disponible environ toutes les heures. Nous vous recommandons de patienter une heure avant d’essayer de créer le nouveau volume.

**Q.** Les volumes épinglés localement sont-ils pris en charge sur l’appliance cloud ?

**A.** Les volumes épinglés localement ne sont pas pris en charge sur l’appliance cloud (les appareils 8010 et 8020 la nommaient auparavant l’appareil virtuel StorSimple).

**Q.** Puis-je utiliser les applets de commande Azure PowerShell pour créer et gérer des volumes épinglés localement ?

**A.** Non, il est impossible de créer des volumes épinglés localement avec les applets de commande Azure PowerShell (tous les volumes que vous créez à l’aide d’Azure PowerShell sont hiérarchisés). Nous vous recommandons également de ne pas utiliser les applets de commande Azure PowerShell pour modifier les propriétés d’un volume épinglé localement, dans la mesure où cela aurait pour effet indésirable de modifier le type de volume en volume à plusieurs niveaux.

## <a name="questions-about-backing-up-a-locally-pinned-volume"></a>Questions sur la sauvegarde d’un volume épinglé localement
**Q.** Les instantanés locaux des volumes épinglés localement sont-ils pris en charge ?

**A.** Oui, vous pouvez prendre des instantanés locaux de vos volumes épinglés localement. Toutefois, nous vous recommandons vivement de sauvegarder régulièrement vos volumes épinglés localement avec des instantanés cloud pour vous assurer que vos données sont protégées en cas de sinistre.

Notez que les instantanés locaux des volumes épinglés localement peuvent également être élevés vers le cloud et ne restent pas forcément au niveau local de l’appareil.

**Q.** Existe-t-il des instructions spécifiques pour la gestion des instantanés locaux des volumes épinglés localement ?

**A.** La prise fréquente d’instantanés locaux avec un taux élevé d’activité des données dans le volume épinglé localement risque de provoquer une consommation rapide de l’espace local sur l’appareil et l’envoi des données des volumes à plusieurs niveaux vers le cloud. Par conséquent, nous vous suggérons de réduire le nombre d’instantanés locaux.

**Q.** J’ai reçu une alerte indiquant que mes instantanés locaux des volumes épinglés localement peuvent être rendus non valides. Quand cela peut-il se produire ?

**A.** La prise fréquente d’instantanés locaux avec un taux élevé d’activité des données dans le volume épinglé localement risque de provoquer une consommation rapide de l’espace local sur l’appareil. Si les niveaux locaux de l’appareil sont largement utilisés, une indisponibilité durable du cloud peut engendrer la saturation de l’appareil. Dans ce cas, les écritures entrantes sur le volume peuvent entraîner l’invalidation des instantanés (dans la mesure où aucun espace n’est disponible pour mettre à jour les instantanés et les lier aux blocs de données plus anciens qui ont été remplacés). Dans ce cas, les écritures sur le volume continueront à être prises en charge, mais les instantanés locaux seront peut-être non valides. Cela n’a aucun impact sur vos instantanés cloud existants.

L’avertissement sert à vous informer que cette situation peut se produire et à s’assurer que vous la gérez en temps voulu, soit en vérifiant votre planification d’instantanés locaux pour prendre des instantanés locaux moins fréquemment, soit en supprimant les anciens instantanés locaux qui ne sont plus nécessaires.

Si les instantanés locaux sont invalidés, vous recevrez un message d’information vous indiquant que les instantanés locaux pour la stratégie de sauvegarde donnée ont été invalidés, ainsi qu’une liste des horodatages des instantanés locaux qui ont été invalidés. Ces instantanés seront automatiquement supprimés et vous ne pourrez plus les afficher sur le panneau **Catalogues de sauvegarde** du portail Azure.

## <a name="questions-about-converting-a-tiered-volume-to-a-locally-pinned-volume"></a>Questions sur la conversion d’un volume à plusieurs niveaux en volume épinglé localement
**Q.** Je constate une certaine lenteur de l’appareil lors de la conversion d’un volume à plusieurs niveaux en volume épinglé localement. Que se passe-t-il ?

**A.** Le processus de conversion se compose de deux étapes :

1. La configuration de l’espace sur l’appareil pour le volume épinglé localement bientôt converti.
2. Le téléchargement des données à plusieurs niveaux à partir du cloud pour assurer les garanties locales.

Ces deux étapes sont des opérations longues, qui dépendent de la taille du volume en cours de conversion, des données sur l’appareil et de la bande passante disponible. Dans la mesure où certaines données des volumes à plusieurs niveaux existants peuvent déborder sur le cloud lors de la configuration, les performances de votre appareil peuvent être réduites pendant ce temps. En outre, le processus de conversion peut être plus lent si :

* Les volumes existants n’ont pas été sauvegardés dans le cloud ; nous vous suggérons donc de sauvegarder vos volumes avant de lancer une conversion.
* Des stratégies de limitation de la bande passante ont été appliquées, ce qui peut limiter la bande passante disponible dans le cloud ; il est donc recommandé de disposer d’une connexion au cloud dédiée d’au moins 40 Mbit/s.
* La conversion peut durer plusieurs heures en raison des divers facteurs mentionnés ci-dessus ; par conséquent, nous vous suggérons d’effectuer cette opération pendant les heures creuses ou pendant un week-end pour éviter tout impact sur les clients.

Plus d’informations sur comment [convertir un volume à plusieurs niveaux en volume épinglé localement](storsimple-8000-manage-volumes-u2.md#change-the-volume-type)

**Q.** Puis-je annuler l’opération de conversion de volume ?

**A.** Non, vous ne pouvez pas annuler l’opération de conversion une fois qu’elle est lancée. Comme indiqué dans la réponse précédente, vous devez être conscient des éventuels problèmes de performances que vous pouvez rencontrer au cours du processus et suivre les meilleures pratiques ci-dessus lorsque vous planifiez votre conversion.

**Q.** Que devient mon volume si la conversion échoue ?

**A.** La conversion de volume peut échouer en raison de problèmes de connexion au cloud. L’appareil peut également interrompre le processus de conversion après plusieurs tentatives infructueuses de récupération des données à plusieurs niveaux transmises par le cloud. Dans ce scénario, le type de volume restera celui du volume source avant la conversion et :

* Une alerte critique sera déclenchée pour vous avertir de l’échec de la conversion du volume. Plus d’informations sur les [alertes liées aux volumes épinglés localement](storsimple-8000-manage-alerts.md#locally-pinned-volume-alerts)
* Si vous convertissez un volume à plusieurs niveaux en volume épinglé localement, le volume continuera à présenter les propriétés d’un volume à plusieurs niveaux, dans la mesure où des données peuvent encore résider sur le cloud. Nous vous suggérons de résoudre les problèmes de connectivité et de recommencer l’opération de conversion.
* De même, en cas d’échec de la conversion d’un volume épinglé localement en volume à plusieurs niveaux, même si le volume reste marqué comme volume épinglé localement, il fonctionnera comme un volume à plusieurs niveaux (car les données peuvent avoir été dispersées dans le cloud). Toutefois, il continue d’occuper l’espace sur les couches locales de l’appareil. Cet espace ne sera pas disponible pour les autres volumes épinglés localement. Nous vous suggérons de recommencer cette opération pour vous assurer que la conversion du volume est terminée et que l’espace local sur l’appareil peut être récupéré.

## <a name="questions-about-restoring-a-locally-pinned-volume"></a>Questions sur la restauration d’un volume épinglé localement
**Q.** Les volumes épinglés localement sont-ils restaurés instantanément ?

**A.** Oui, les volumes épinglés localement sont restaurés instantanément. Dès que les informations de métadonnées pour le volume sont extraites du cloud dans le cadre de l’opération de restauration, le volume est mis en ligne et accessible pour l’hôte. Toutefois, les garanties locales pour les données du volume ne seront pas présentes tant que toutes les données n’auront pas été téléchargées à partir du cloud. Les performances de ces volumes peuvent également être limitées pendant la durée de la restauration.

**Q.** Combien de temps faut-il pour restaurer un volume épinglé localement ?

**A.** Les volumes épinglés localement sont restaurés instantanément et mis en ligne dès que les informations de métadonnées du volume sont récupérées à partir du cloud, alors que les données du volume continuent à être téléchargées en arrière-plan. Cette dernière partie de l’opération de restauration (la récupération des garanties locales pour les données du volume) est une opération longue : plusieurs heures peuvent s’écouler avant que toutes les données soient de nouveau locales. Le temps nécessaire pour terminer cette opération dépend de plusieurs facteurs, notamment la taille du volume en cours de restauration et la bande passante disponible. Si le volume d’origine du volume en cours de restauration a été supprimé, l’opération prendra plus de temps car il faudra créer l’espace local sur l’appareil dans le cadre de la restauration.

**Q.** J’ai besoin de restaurer mon volume épinglé localement existant vers un instantané plus ancien (effectué lorsque le volume était un volume à plusieurs niveaux). Le volume sera-t-il restauré en tant que volume à plusieurs niveaux ?

**A.** Non, un volume épinglé localement sera restauré en tant que volume épinglé localement. Bien que les instantanés datent du temps où le volume était un volume à plusieurs niveaux, lors de la restauration des volumes existants, StorSimple utilise toujours le type de volume qui existe actuellement sur le disque.

**Q.** J’ai récemment étendu mon volume épinglé localement, mais j’ai maintenant besoin de restaurer les données à une heure à laquelle le volume était de plus petite taille. La restauration redimensionnera-t-elle le volume actuel et devrai-je étendre la taille du volume une fois la restauration terminée ?

**A.** Oui, la restauration redimensionnera le volume et vous devrez étendre la taille du volume une fois la restauration terminée.

**Q.** Puis-je modifier le type d’un volume pendant la restauration ?

**A.** Non, vous ne pouvez pas modifier le type de volume pendant la restauration.

* Les volumes qui ont été supprimés sont restaurés sous le type stocké dans l’instantané.
* Les volumes existants sont restaurés en fonction de leur type actuel, quel que soit le type stocké dans l’instantané (reportez-vous aux deux questions précédentes).

**Q.** Je dois restaurer mon volume épinglé localement, mais j’ai choisi un instantané dans le temps incorrect. Puis-je annuler l’opération de restauration en cours ?

**A.** Oui, vous pouvez annuler une opération de restauration en cours. L’état du volume sera restauré à l’état au début de la restauration. Toutefois, toutes les écritures qui ont été apportées au volume lors de la restauration seront perdues.

**Q.** J’ai commencé une opération de restauration sur un de mes volumes épinglés localement, et maintenant un instantané que je ne me souviens pas avoir créé apparaît dans mon catalogue de file d’attente. À quoi sert-il ?

**A.** Il s’agit de l’instantané temporaire créé avant l’opération de restauration et qui est utilisé pour le rétablissement du volume si la restauration est annulée ou échoue. Ne supprimez pas cet instantané ; il sera automatiquement supprimé une fois la restauration terminée. Ce comportement peut se produire si votre travail de restauration possède des volumes épinglés localement ou une combinaison de volumes épinglés localement et de volumes hiérarchisés. Si le travail de restauration comporte uniquement des volumes hiérarchisés, cette situation ne se produira pas.

**Q.** Puis-je cloner un volume épinglé localement ?

**A.** Oui, vous pouvez. Cependant, le volume épinglé localement sera par défaut cloné en tant que volume à plusieurs niveaux. Plus d’informations sur le [clonage d’un volume épinglé localement](storsimple-8000-clone-volume-u2.md)

## <a name="questions-about-failing-over-a-locally-pinned-volume"></a>Questions sur le basculement d’un volume épinglé localement
**Q.** J’ai besoin de basculer mon appareil vers un autre appareil physique. Mes volumes épinglés localement basculeront-ils sous forme de volumes épinglés localement ou de volumes à plusieurs niveaux ?

**A.** Les volumes épinglés localement sont basculés comme épinglés localement si l’appareil cible exécute une mise à jour 3 ou supérieure de la gamme StorSimple 8000.

Plus d’informations sur [le basculement et la récupération d’urgence de volumes épinglés localement en fonction des versions](storsimple-8000-device-failover-disaster-recovery.md#device-failover-across-software-versions)

**Q.** Les volumes épinglés localement sont-ils instantanément restaurés lors de la récupération d’urgence ?

**A.** Oui, les volumes épinglés localement sont restaurés instantanément lors du basculement. Dès que les informations de métadonnées pour le volume sont extraites du cloud dans le cadre de l’opération de basculement, le volume est mis en ligne sur l’appareil cible et devient accessible pour l’hôte. Pendant ce temps, les données du volume continueront à se télécharger en arrière-plan. Les performances de ces volumes sont susceptibles d’être limitées pendant la durée du basculement.

**Q.** La tâche de basculement est terminée. Comment puis-je suivre la progression de la restauration du volume épinglé localement sur l’appareil cible ?

**A.** Pendant une opération de basculement, la tâche de basculement est marquée comme exécutée une fois que tous les volumes du jeu de basculement ont été instantanément restaurés et mis en ligne sur l’appareil cible. Cela inclut tous les volumes épinglés localement qui pourraient avoir été basculés. Toutefois, les garanties locales des données seront uniquement disponibles lorsque toutes les données du volume auront été téléchargées. Vous pouvez suivre la progression pour chaque volume épinglé localement qui a été basculé en surveillant les tâches de restauration correspondantes qui sont créées dans le cadre du basculement. Ces tâches de restauration individuelles sont créées uniquement pour les volumes épinglés localement.

**Q.** Puis-je modifier le type d’un volume pendant le basculement ?

**A.** Non, vous ne pouvez pas modifier le type de volume pendant le basculement. Si vous basculez vers un autre appareil physique qui exécute la mise à jour 3 de la gamme StorSimple 8000, les volumes basculent en fonction du type de volume stocké dans l’instantané.

**Q.** Puis-je basculer un conteneur de volumes avec les volumes épinglés localement pour l’appliance cloud ?

**A.** Oui, vous pouvez. Les volumes épinglés localement sont basculés sous forme de volumes à plusieurs niveaux. Plus d’informations sur [le basculement et la récupération d’urgence de volumes épinglés localement en fonction des versions](storsimple-8000-device-failover-disaster-recovery.md#common-considerations-for-device-failover)

