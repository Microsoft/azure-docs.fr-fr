---
title: Haute disponibilité et récupération d’urgence Azure IoT Hub | Microsoft Docs
description: Décrit les options Azure et IoT Hub qui vous aident à créer des solutions Azure IoT à haute disponibilité dotées de fonctionnalités de récupération d’urgence.
author: jlian
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 03/17/2020
ms.author: philmea
ms.openlocfilehash: c665e30ed9b284f7c93cf8588b710c9f22457a0a
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92151675"
---
# <a name="iot-hub-high-availability-and-disaster-recovery"></a>Haute disponibilité et récupération d’urgence IoT Hub :

La première étape de l’implémentation d’une solution IoT résiliente consiste, pour les architectes, développeurs et chefs d’entreprise, à définir les objectifs de temps de fonctionnement pour les solutions qu’ils développent. Ces objectifs peuvent être principalement définis en fonction des objectifs stratégiques applicables à chaque scénario. Dans ce contexte, l’article [Conception d’applications résilientes pour Azure](/azure/architecture/resiliency/) décrit un cadre général qui vous aide à réfléchir aux questions de continuité d'activité et de reprise d'activité. Le document [Récupération d’urgence et haute disponibilité pour les applications Azure](/azure/architecture/reliability/disaster-recovery) contient des recommandations d’architecture concernant les stratégies permettant de mettre en place la haute disponibilité et la récupération d’urgence dans les applications Azure.

Cet article décrit les fonctionnalités de haute disponibilité et de récupération d’urgence offertes par le service IoT Hub. Il aborde essentiellement les aspects suivants :

- Haute disponibilité intra-région
- Récupération d’urgence inter-région
- Haute disponibilité inter-région

Selon les objectifs de temps d’activité que vous définissez pour vos solutions IoT, vous devez identifier parmi les options décrites ci-dessous celle qui correspond le mieux à vos objectifs stratégiques. L’intégration de ces alternatives de haute disponibilité/récupération d’urgence dans votre solution IoT suppose d’évaluer minutieusement les compromis entre :

- le niveau de résilience dont vous avez besoin ; 
- la complexité d’implémentation et de maintenance ;
- l’impact du coût des marchandises vendues.

## <a name="intra-region-ha"></a>Haute disponibilité intra-région

Le service IoT Hub fournit une haute disponibilité intra-région en implémentant les redondances à presque tous les niveaux du service. Le [SLA publié par le service IoT Hub](https://azure.microsoft.com/support/legal/sla/iot-hub) est atteint grâce à l’utilisation de ces redondances. Les développeurs d’une solution IoT peuvent tirer parti de ces fonctionnalités de haute disponibilité sans le moindre effort supplémentaire. Bien qu’IoT Hub offre une garantie de disponibilité raisonnablement élevée, des défaillances temporaires peuvent toujours se produire comme avec n’importe quelle plateforme informatique distribuée. Si vous démarrez seulement avec la migration de vos solutions locales vers le cloud, vous devez vous concentrer davantage sur l’optimisation du « temps moyen de récupération » que sur le « délai moyen entre défaillances ». En d’autres termes, les défaillances temporaires sont considérées comme normales lorsque le cloud est dans la balance. Des [stratégies de nouvelle tentative](iot-hub-reliability-features-in-sdks.md) appropriées doivent être intégrées dans les composants qui interagissent avec une application cloud de manière à gérer les défaillances temporaires.

> [!NOTE]
> Certains services Azure fournissent également des couches supplémentaires de disponibilité au sein d’une région grâce à l’intégration des [Zones de disponibilité (AZ)](../availability-zones/az-overview.md). Ces zones de disponibilité ne sont actuellement pas prises en charge par le service IoT Hub.

## <a name="cross-region-dr"></a>Récupération d’urgence inter-région

Il peut arriver à de rares occasions qu’un centre de données connaisse une interruption prolongée en raison de pannes d’alimentation ou d’autres défaillances impliquant des biens physiques. Ces événements se produisent rarement et la fonction de haute disponibilité intra-région décrite ci-dessus ne peut pas toujours s’appliquer dans ces situations. IoT Hub propose plusieurs solutions de récupération face à ce type d’interruption prolongée. 

Les clients disposent de deux options de récupération dans une telle situation : le [basculement initié par Microsoft](#microsoft-initiated-failover) et le [basculement manuel](#manual-failover). La différence fondamentale entre les deux est que, dans la première option, Microsoft intervient, tandis que l’utilisateur déclenche lui-même le basculement dans la deuxième option. Le basculement manuel garantit également un objectif de temps de récupération (RTO) inférieur à celui de l’option de basculement initié par Microsoft. Les RTO spécifiques offerts pour chaque option sont décrits dans les sections ci-dessous. Lorsqu’une de ces options de basculement d’un IoT hub à partir de sa région primaire est utilisée, le hub devient entièrement fonctionnel dans la [région Azure qui lui est associée géographiquement](../best-practices-availability-paired-regions.md).

Ces deux options de basculement offrent les objectifs de point de récupération (RPO) suivants :

| Type de données | Objectifs de point de récupération (RPO) |
| --- | --- |
| Registre des identités |Perte de données entre 0 et 5 minutes |
| Données de jumeau d’appareil |Perte de données entre 0 et 5 minutes |
| Messages cloud-à-appareil<sup>1</sup> |Perte de données entre 0 et 5 minutes |
| Travaux d’appareils et parents<sup>1</sup> |Perte de données entre 0 et 5 minutes |
| Messages appareil-à-cloud |Perte de tous les messages non lus |
| Messages de surveillance des opérations |Perte de tous les messages non lus |
| Messages de rétroaction cloud-à-appareil |Perte de tous les messages non lus |

<sup>1</sup>Les messages cloud-à-appareil et les travaux parents ne sont pas récupérés dans le cadre d’un basculement manuel.

Une fois l’opération de basculement terminée pour l’IoT Hub, toutes les opérations exécutées à partir des applications principales et de l’appareil sont supposées continuer de fonctionner sans intervention manuelle. Cela signifie que vos messages appareil-à-cloud continuent de fonctionner et que l’intégralité du registre de l’appareil est intacte. Les événements émis via Event Grid peuvent être utilisés sur les mêmes abonnements configurés précédemment tant que ces abonnements Event Grid restent disponibles. Aucune manipulation supplémentaire n’est requise pour les points de terminaison personnalisés.

> [!CAUTION]
> - Le nom et le point de terminaison compatibles Event Hub, de même que le point de terminaison Events intégré à IoT Hub, changent après le basculement. À la réception des messages de télémétrie à partir du point de terminaison intégré à l’aide du client Event Hub ou d’un hôte de processeur d’événements, vous devez [utiliser la chaîne de connexion IoT Hub](iot-hub-devguide-messages-read-builtin.md#read-from-the-built-in-endpoint) pour établir la connexion. De cette manière, vos applications principales continueront de fonctionner sans nécessiter d’intervention manuelle après le basculement. Si vous utilisez le nom et le point de terminaison compatibles Event Hub directement dans votre application, vous devrez [extraire le nouveau point de terminaison compatible Event Hub](iot-hub-devguide-messages-read-builtin.md#read-from-the-built-in-endpoint) après le basculement pour pouvoir poursuivre vos opérations. 
>
> - Si vous utilisez Azure Functions ou Azure Stream Analytics pour connecter le point de terminaison Events intégré, vous devrez peut-être effectuer un **redémarrage**. Cela est dû au fait que, pendant le basculement, les décalages précédents ne sont plus valides.
>
> - Pour effectuer un routage vers le stockage, nous vous recommandons de répertorier les objets blob ou les fichiers, puis d’exécuter une itération sur ces derniers, afin de garantir que tous les objets blob ou les fichiers seront lus, sans avoir à faire de suppositions concernant la partition. La plage de la partition peut changer pendant un basculement initié par Microsoft ou pendant un basculement manuel. Vous pouvez utiliser l’[API Lister les blobs](/rest/api/storageservices/list-blobs) pour énumérer la liste des objets blob ou l’[API Lister ADLS Gen2 API](/rest/api/storageservices/datalakestoragegen2/path/list) pour lister les fichiers. Pour en savoir plus, consultez [Stockage Azure en tant que point de terminaison de routage](iot-hub-devguide-messages-d2c.md#azure-storage-as-a-routing-endpoint).

## <a name="microsoft-initiated-failover"></a>Basculement initié par Microsoft

Le basculement initié par Microsoft est déclenché par Microsoft dans de rares situations pour faire basculer tous les hubs IoT entre une région affectée et la région géographiquement associée. Ce processus représente l’option par défaut (aucun moyen pour les utilisateurs d’y renoncer) et ne nécessite aucune intervention de la part de l’utilisateur. Microsoft se réserve le droit de déterminer dans quelles circonstances exercer cette option. Ce mécanisme n’implique pas le consentement de l’utilisateur avant le basculement du hub de l’utilisateur. Le basculement initié par Microsoft est associé à un objectif de temps de récupération (RTO) de 2 à 26 heures. 

Ce long RTO s’explique par le fait que Microsoft doit effectuer l’opération de basculement pour le compte de tous les clients affectés dans cette région. Si vous exécutez une solution IoT moins critique capable de supporter un temps d’arrêt d’environ une journée, vous pouvez très bien prendre une dépendance sur cette option pour satisfaire les objectifs généraux de récupération d’urgence pour votre solution IoT. Le temps total nécessaire pour que les opérations de runtime soient totalement opérationnelles après le déclenchement de ce processus est décrit dans la section « Temps de récupération ».

## <a name="manual-failover"></a>Basculement manuel

Si vos objectifs de disponibilité ne sont pas satisfaits par le RTO associé à l’option de basculement initié par Microsoft, envisagez d’utiliser un basculement manuel pour déclencher vous-même le processus de basculement. Cette option peut vous assurer un RTO allant de 10 minutes à quelques heures. Le RTO dépend actuellement du nombre d’appareils inscrits sur l’instance IoT Hub en cours de basculement. Pour un hub hébergeant environ 100 000 appareils, vous pouvez vous attendre à un RTO de l’ordre de 15 minutes. Le temps total nécessaire pour que les opérations de runtime soient totalement opérationnelles après le déclenchement de ce processus est décrit dans la section « Temps de récupération ».

L’option de basculement manuel est toujours disponible, que la région primaire rencontre ou non des temps d’arrêt. Par conséquent, vous pouvez potentiellement utiliser cette option pour effectuer des basculements planifiés. Les basculements planifiés peuvent être par exemple utilisés dans le cadre d’exercices de basculement périodiques. Il faut cependant savoir qu’une opération de basculement planifié entraîne l’arrêt du hub pendant la période définie par le RTO pour cette option. Elle s’accompagne également d’une perte de données, telle que définie dans le tableau des RPO ci-dessus. Vous pouvez envisager de configurer une instance IoT Hub de test pour exercer l’option de basculement planifié de manière périodique, afin de gagner confiance en votre capacité à maintenir le bon fonctionnement de vos solutions de bout en bout en cas d’incident réel.

La fonctionnalité de basculement manuel est disponible sans coût supplémentaire pour les hubs IoT créés après le 18 mai 2017.

Pour obtenir des instructions pas à pas, consultez le [Tutoriel : Effectuer un basculement manuel pour un hub IoT](tutorial-manual-failover.md)

### <a name="running-test-drills"></a>Exécution d’exercices de test

Les exercices de test ne doivent pas être effectués sur des IoT Hubs utilisés dans vos environnements de production.

### <a name="dont-use-manual-failover-to-migrate-iot-hub-to-a-different-region"></a>Ne pas utiliser le basculement manuel pour migrer un hub IoT vers une autre région

Il n’est *pas* conseillé d’utiliser le basculement manuel comme un mécanisme servant à migrer définitivement votre hub entre les régions Azure associées géographiquement. Cela augmente la latence des opérations exécutées sur le hub IoT à partir d’appareils hébergés dans l’ancienne région primaire.

## <a name="failback"></a>Restauration automatique

La restauration automatique vers l’ancienne région primaire peut être obtenue en déclenchant de nouveau l’action de basculement. Si l’opération de basculement d’origine a été effectuée dans le cadre d’une récupération après une panne prolongée dans la région primaire d’origine, il est recommandé de basculer le hub vers l’emplacement d’origine une fois la situation de panne corrigée.

> [!IMPORTANT]
> - Les utilisateurs sont limités à 2 opérations de basculement réussies et à 2 opérations de restauration automatique réussies par jour.
>
> - Les opérations de basculement/restauration automatique parallèles ne sont pas autorisées. Vous devez respecter un délai d’attente de 1 heure entre ces opérations.

## <a name="time-to-recover"></a>Temps de récupération

Alors que le FQDN (et par conséquent, la chaîne de connexion) de l’instance IoT Hub reste le même après le basculement, l’adresse IP sous-jacente change. Par conséquent, le temps global nécessaire pour que les opérations de runtime exécutées sur votre instance IoT Hub deviennent totalement opérationnelles une fois le processus de basculement déclenché peut être exprimé à l’aide de la fonction suivante.

Temps de récupération = RTO [10 min -2 heures pour un basculement manuel | 2 à 26 heures pour un basculement initié par Microsoft] + délai de propagation DNS + temps nécessaire à l’application cliente pour actualiser l’adresse IP du hub IoT mise en cache.

> [!IMPORTANT]
> Les SDK IoT ne mettent pas en cache l’adresse IP de l’instance IoT Hub. Nous recommandons que le code utilisateur interagissant avec les SDK ne mettent pas en cache l’adresse IP de l’IoT Hub.

## <a name="achieve-cross-region-ha"></a>Haute disponibilité inter-région

Si vos objectifs de temps d’activité ne sont pas satisfaits par le RTO associé à un basculement initié par Microsoft ou par un basculement manuel, vous devez envisager d’implémenter un mécanisme de basculement automatique inter-région pour chaque appareil.
Un traitement complet des topologies de déploiement dans des solutions IoT dépasserait la portée de cet article. L’article traite du modèle de déploiement avec *basculement régional* pour les besoins de haute disponibilité et de récupération d’urgence.

Dans un modèle de basculement régional, le backend de solution s’exécute principalement dans un emplacement de centre de données. Un IoT Hub et un backend secondaire sont déployés dans un autre emplacement de centre de données. Si l’IoT Hub de la région primaire subit une panne ou si la connectivité réseau de l’appareil à la région primaire est interrompue, les appareils utilisent un point de terminaison de service secondaire. Vous pouvez améliorer la disponibilité de la solution en implémentant un modèle de basculement inter-régions au lieu de rester au sein d’une seule région. 

À un niveau supérieur, pour implémenter un modèle de basculement régional avec IoT Hub, vous devez procéder de la manière suivante :

* **Un hub IoT secondaire et une logique de routage des appareils** : En cas d’interruption de service dans votre région principale, les appareils doivent commencer à se connecter à votre région secondaire. Étant donné l’état de la plupart des services impliqués, il est courant pour les administrateurs de solution de déclencher le processus de basculement inter-régions. La meilleure façon de communiquer le nouveau point de terminaison aux appareils tout en conservant le contrôle du processus consiste à consulter régulièrement un service de *conciergerie* pour connaître le point de terminaison actif en cours. Le service de concierge peut être une application web répliquée accessible à l’aide de techniques de redirection DNS (par exemple, l’utilisation [d’Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md)).

   > [!NOTE]
   > Le service IoT Hub n’est pas un type de point de terminaison pris en charge dans Azure Traffic Manager. Il est recommandé d’intégrer le service de concierge proposé avec Azure Traffic Manager pour faire en sorte qu’il implémente l’API de sonde d’intégrité du point de terminaison.

* **Réplication du registre des identités** : Pour être utilisable, le hub IoT secondaire doit contenir toutes les identités des appareils pouvant se connecter à la solution. La solution doit conserver des sauvegardes géo-répliquées d’identités d’appareils et les télécharger dans le hub IoT secondaire avant de basculer le point de terminaison actif des appareils. La fonctionnalité d’exportation des identités d’appareil d’IoT Hub est utile dans ce contexte. Pour plus d’informations, consultez[Guide du développeur IoT Hub - Registre des identités](iot-hub-devguide-identity-registry.md).

* **Logique de fusion** : Quand la région principale redevient disponible, l’ensemble des états et des données qui ont été créés dans le site secondaire doivent être migrés en sens inverse vers la région principale. Ces états et ces données ont essentiellement trait aux identités des appareils et aux métadonnées d’application qui doivent être fusionnées avec l’IoT Hub principal et d’autres magasins propres à l’application dans la région principale. 

Pour simplifier cette étape, vous devez utiliser des opérations idempotentes. Des opérations idempotentes minimisent les effets indésirables de la distribution cohérente éventuelle d’événements, et des doublons ou des livraisons d’événements hors d’usage. Par ailleurs, la logique d’application doit être conçue pour tolérer les éventuelles incohérences ou de légers retards. Cette situation peut se produire en raison du temps additionnel que le système prend pour se réparer en fonction des objectifs de points de récupération (RPO).

## <a name="choose-the-right-hadr-option"></a>Choisir la bonne option de haute disponibilité/récupération d’urgence

Voici un résumé des options de haute disponibilité/récupération d’urgence présentées dans cet article et que vous pouvez utiliser comme cadre de référence pour choisir l’option qui convient le mieux à votre solution.

| Option de haute disponibilité/récupération d’urgence | RTO | RPO | Nécessite une intervention manuelle ? | Complexité de l’implémentation | Coût supplémentaire|
| --- | --- | --- | --- | --- | --- |
| Basculement initié par Microsoft |2 - 26 heures|Voir le tableau des RPO ci-dessus|Non|None|None|
| Basculement manuel |10 min - 2 heures|Voir le tableau des RPO ci-dessus|Oui|Très faible. Vous ne devez déclencher cette opération qu’à partir du portail.|None|
| Haute disponibilité inter-région |< 1 min|Dépend de la fréquence de réplication de votre solution de haute disponibilité personnalisée|Non|Élevé|> 1 fois supérieure au coût de 1 IoT Hub|

## <a name="next-steps"></a>Étapes suivantes

* [Qu’est-ce qu’Azure IoT Hub ?](about-iot-hub.md)
* [Bien démarrer avec les hubs IoT (démarrage rapide)](quickstart-send-telemetry-dotnet.md)
* [Tutoriel : Effectuer un basculement manuel pour un hub IoT](tutorial-manual-failover.md)