---
title: Comprendre les quotas Azure IoT Hub et la limitation | Microsoft Docs
description: Guide du développeur - description des quotas qui s’appliquent à IoT Hub et comportement de limitation attendu.
author: robinsh
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/08/2019
ms.custom:
- 'Role: Cloud Development'
- 'Role: Operations'
- 'Role: Technical Support'
ms.openlocfilehash: 5a5b20efbf804c2ea1097f905da1cfd62727ff15
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94410689"
---
# <a name="reference---iot-hub-quotas-and-throttling"></a>Référence - Quotas et limitation IoT Hub

Cet article explique les quotas d’un IoT Hub et fournit des informations pour vous aider à comprendre le fonctionnement de la limitation.

## <a name="quotas-and-throttling"></a>Quotas et limitation

Chaque abonnement Azure peut avoir au maximum 50 IoT Hubs et au maximum 1 hub gratuit.

Chaque hub IoT est approvisionné avec un certain nombre d’unités dans un niveau spécifique. Le niveau et le nombre d’unités déterminent le quota quotidien maximal de messages que vous pouvez envoyer. La taille de message utilisée pour calculer le quota quotidien est de 0,5 Ko pour un hub de niveau gratuit et de 4 Ko pour tous les autres niveaux. Pour plus d’informations, consultez [Tarification d’Azure IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub/).

Le niveau détermine également les limites qu’IoT Hub applique sur toutes les opérations.

## <a name="iot-plug-and-play"></a>IoT Plug-and-Play

Les appareils IoT Plug-and-Play envoient au moins un message de télémétrie par interface, racine comprise, ce qui peut augmenter le nombre de messages comptabilisés dans le quota de messages.

## <a name="operation-throttles"></a>Limitations d’opérations

Les limitations d’opérations sont des limites de taux qui sont appliquées par plages de minutes et sont destinées à éviter les abus. Elles sont également soumises à [la régulation de flux](#traffic-shaping).

Le tableau suivant présente les limitations appliquées. Les valeurs font référence à un hub individuel.

| Limitation | Gratuit, B1 et S1 | B2 et S2 | B3 et S3 | 
| -------- | ------- | ------- | ------- |
| [Opérations de registre des identités](#identity-registry-operations-throttle) (création, récupération, création de listes, mise à jour, suppression) | 1,67/s/unité (100/min/unité) | 1,67/s/unité (100/min/unité) | 83,33/s/unité (5 000/min/unité) |
| _Nouvelles connexions d’appareil_ (cette limite s’applique à la fréquence des [nouvelles connexions](#device-connections-throttle) et non au nombre total de connexions) | Plus de 100/s ou 12/s/unité <br/> Par exemple, deux unités S1 sont 2\*12 = 24 nouvelles connexions/s, mais vous avez au moins 100 nouvelles connexions/s dans toutes vos unités. Avec neuf unités S1, vous avez 108 nouvelles connexion/s (9\*12) dans toutes vos unités. | 120 nouvelles connexions/s/unité | 6 000 nouvelles connexions/s/unité |
| Envois appareil-à-cloud | Plus de 100 opérations d’envoi/s ou 12 opérations d’envoi/s/unité <br/> Par exemple, deux unités S1 équivalent à 2\*12 = 24/s, mais vous obtenez au moins 100 opérations d’envoi/s sur vos unités. Avec neuf unités S1, vous avez 108 opérations d’envoi/s (9\*12) sur vos unités. | 120 opérations d’envoi/s/unité | 6 000 opérations d’envoi/s/unité |
| Envois de cloud-à-appareil<sup>1</sup> | 1,67 opération d’envoi/s/unité (100 messages/min/unité) | 1,67 opération d’envoi/s/unité (100 opérations d’envoi/min/unité) | 83,33 opérations d’envoi/s/unité (5 000 opérations d’envoi/min/unité) |
| Réceptions de cloud-à-appareil<sup>1</sup> <br/> (uniquement quand l’appareil utilise HTTPS)| 16,67 opérations de réception/s/unité (1 000 opérations de réception/min/unité) | 16,67 opérations de réception/s/unité (1 000 opérations de réception/min/unité) | 833,33 opérations de réception/s/unité (50 000 opérations de réception/min/unité) |
| Chargement de fichiers | 1,67 initiations de chargement de fichier/s/unité (100/min/unité) | 1,67 initiations de chargement de fichier/s/unité (100/min/unité) | 83,33 initiations de chargement de fichier/s/unité (5 000/min/unité) |
| Méthodes directes<sup>1</sup> | 160 Ko/s/unité<sup>2</sup> | 480 Ko/s/unité<sup>2</sup> | 24 Mo/s/unité<sup>2</sup> | 
| Requêtes | 20/min/unité | 20/min/unité | 1 000/min/unité |
| Lectures de jumeaux (appareil et module)<sup>1</sup> | 100/s | Plus de 100/s ou 10/s/unité | 500/s/unité |
| Mises à jour de jumeaux (appareil et module)<sup>1</sup> | 50/s | Plus de 50/s ou 5/s/unité | 250/s/unité |
| Opérations de travaux<sup>1</sup> <br/> (créer, mettre à jour, répertorier, supprimer) | 1,67/s/unité (100/min/unité) | 1,67/s/unité (100/min/unité) | 83,33/s/unité (5 000/min/unité) |
| Opérations de l’appareil de travaux<sup>1</sup> <br/> (mettre à jour le jumeau, appeler la méthode directe) | 10/s | Plus de 10/s ou 1/s/unité | 50/s/unité |
| Configurations et déploiements de périphérie<sup>1</sup> <br/> (créer, mettre à jour, répertorier, supprimer) | 0,33/s/unité (20/min/unité) | 0,33/s/unité (20/min/unité) | 0,33/s/unité (20/min/unité) |
| Fréquence d’émission de flux d’appareil<sup>1</sup> | 5 nouveaux flux/s | 5 nouveaux flux/s | 5 nouveaux flux/s |
| Nombre maximal de flux d’appareils connectés simultanément<sup>1</sup> | 50 | 50 | 50 |
| Transfert de données de flux d’appareil maximal<sup>1</sup> (volume agrégé par jour) | 300 Mo | 300 Mo | 300 Mo |

<sup>1</sup>Cette fonctionnalité n’est pas disponible dans le niveau de base d’IoT Hub. Pour plus d’informations, consultez [Comment choisir le bon IoT Hub](iot-hub-scaling.md). <br/><sup>2</sup>La taille du compteur de limitation est de 4 Ko. La limitation de requêtes est basée uniquement sur la taille de charge utile de la requête.

### <a name="throttling-details"></a>Détails de la limitation

* La taille du compteur détermine à quel incrément votre limitation est consommée. Si la charge utile de votre appel direct est comprise entre 0 et 4 Ko, elle est comptabilisée comme étant de 4 Ko. Vous pouvez effectuer jusqu’à 40 appels par seconde par unité avant d’atteindre la limite de 160 Ko/s/unité.

   De même, si votre charge utile est comprise entre 4 Ko et 8 Ko, chaque appel est comptabilisé pour 8 Ko et vous pouvez effectuer jusqu’à 20 appels par seconde par unité avant d’atteindre la limite maximale.

   Enfin, si la taille de votre charge utile est comprise entre 156 Ko et 160 Ko, vous ne pouvez effectuer qu’un seul appel par seconde par unité dans votre hub avant d’atteindre la limite de 160 Ko/s/unité.

*  Pour les *opérations d’appareil de travaux (mise à jour de la représentation, appel de la méthode directe)* pour le niveau S3, 50/s/unité s’applique uniquement lorsque vous appelez des méthodes à l’aide de travaux. Si vous appelez directement des méthodes directes, la limitation d’origine de 24 Mo/s/unité (pour S3) s’applique.

*  Le **quota** est le nombre total de messages que vous pouvez envoyer dans votre hub *par jour*. Vous trouverez la limite de quota de votre hub sous la colonne **Nombre total de messages/jour** sur la [page de tarification IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub/).

*  Vos limites cloud-à-appareil et appareil-à-cloud déterminent la *vitesse* maximale à laquelle vous pouvez envoyer des messages, c’est-à-dire le nombre de messages, quels que soient les blocs de 4 Ko. Chaque message peut atteindre une taille de 256 Ko, soit la [taille de message maximale](iot-hub-devguide-quotas-throttling.md#other-limits).

*  Il est recommandé de limiter vos appels afin de ne pas atteindre/dépasser les limitations. Si vous atteignez la limite, IoT Hub répond avec le code d’erreur 429 et le client doit temporiser et effectuer une nouvelle tentative. Ces limites sont définies par hub (ou, dans certains cas, par hub/unité). Pour plus d’informations, consultez la rubrique [Gérer la connectivité et la messagerie fiable/Modèles de nouvelle tentative](iot-hub-reliability-features-in-sdks.md#retry-patterns).

### <a name="traffic-shaping"></a>Régulation de flux

Pour prendre en charge le trafic en rafale, IoT Hub accepte les requêtes dépassant la limitation sur une durée limitée. Les quelques premières requêtes sont traitées immédiatement. Toutefois, si le nombre de requêtes continue à enfreindre la limitation, IoT Hub commence à placer les requêtes dans une file d’attente. Elles sont alors traitées selon le taux limite. Cet effet est appelé la *régulation de flux*. En outre, la taille de cette file d’attente est limitée. Si la violation de limitation continue, la file d’attente finit par se remplir, et IoT Hub commence à rejeter les requêtes avec `429 ThrottlingException`.

Par exemple, vous utilisez un appareil simulé pour envoyer 200 messages appareil-à-cloud par seconde à votre IoT Hub S1 (dont les envois appareil-à-cloud sont limités à 100/s). Pendant la première ou les deux premières minutes, les messages sont traités immédiatement. Toutefois, étant donné que l’appareil continue à envoyer plus de messages que ne l’autorise la limitation, IoT Hub commence à traiter uniquement 100 messages par seconde et place le reste dans une file d’attente. Vous commencez alors à remarquer une latence plus élevée. Finalement, la file d’attente se remplit et vous obtenez une exception `429 ThrottlingException`, et la valeur [« number of throttle errors » (nombre d’erreurs de limitation) dans les métriques IoT Hub](monitor-iot-hub-reference.md#device-telemetry-metrics) commence à augmenter. Pour savoir comment créer des alertes et des graphiques basés sur des métriques, consultez [Surveiller IoT Hub](monitor-iot-hub.md).

### <a name="identity-registry-operations-throttle"></a>Limitation des opérations du registre des identités

Les opérations du registre des identités d’appareil sont prévues pour une utilisation au moment de l’exécution dans les scénarios de gestion et d’approvisionnement des appareils. La lecture ou la mise à jour d’un grand nombre d’identités d’appareils est prise en charge par le biais des [travaux d’importation et d’exportation](iot-hub-devguide-identity-registry.md#import-and-export-device-identities).

### <a name="device-connections-throttle"></a>Limitation des connexions d’appareils

La limitation des *connexions d’appareil* régit la fréquence à laquelle de nouvelles connexions d’appareil peuvent être établies avec un hub IoT. La limitation des *connexions d’appareils* ne régit pas le nombre maximal d’appareils connectés simultanément. Le taux de limitation des *connexions d’appareil* dépend du nombre d’unités provisionnées pour le hub IoT.

Par exemple, si vous achetez une seule unité S1, vous obtenez une limitation de 100 connexions par seconde. Par conséquent, pour connecter 100 000 appareils, au moins 1 000 secondes (soit environ 16 minutes) sont nécessaires. Toutefois, vous pouvez avoir autant d’appareils connectés simultanément que d’appareils enregistrés dans le registre des identités.

## <a name="other-limits"></a>Autres limites

IoT Hub impose d’autres limites opérationnelles :

| Opération | Limite |
| --------- | ----- |
| Appareils | Le nombre total d’appareils plus les modules qui peuvent être inscrits auprès d’un même hub IoT est limité à 1 million. Vous pouvez augmenter cette limite uniquement en contactant le [Support Microsoft](https://azure.microsoft.com/support/options/).|
| Chargements de fichiers | 10 chargements de fichiers simultanés par appareil. |
| Travaux<sup>1</sup> | Le nombre maximal de travaux simultanés est 1 (pour les niveaux Gratuit et S1), 5 (pour S2) et 10 (pour S3). Toutefois, le nombre maximal de [travaux d’importation/exportation d’appareils](iot-hub-bulk-identity-mgmt.md) simultanés est 1 pour tous les niveaux. <br/>L’historique des travaux est conservé pendant 30 jours maximum. |
| Points de terminaison supplémentaires | Les hubs avec SKU payants peuvent avoir 10 points de terminaison supplémentaires. Les hubs avec SKU gratuits peuvent avoir un point de terminaison supplémentaire. |
| Requêtes de routage de messages | Les hubs avec références SKU payantes peuvent avoir 100 requêtes de routage. Les hubs avec références SKU gratuites peuvent avoir cinq requêtes de routage. |
| Enrichissements de message | Les hubs avec références SKU payantes peuvent avoir jusqu’à 10 enrichissements de message. Les hubs avec références SKU gratuites peuvent avoir jusqu’à 2 enrichissements de message.|
| Messages d’appareil-à-cloud | Taille maximale des messages 256 Ko |
| Messages de cloud-à-appareil<sup>1</sup> | Taille maximale des messages 64 Ko. Le nombre maximal de messages en attente est de 50 par appareil. |
| Méthode directe<sup>1</sup> | La taille maximale de charge utile de la méthode directe est de 128 Ko. |
| Configurations automatiques d’appareils et de modules<sup>1</sup> | 100 configurations par hub de référence SKU payant. 20 configurations par hub de référence SKU gratuit. |
| Déploiement automatique IoT Edge<sup>1</sup> | 50 modules par déploiement. 100 déploiements (déploiements en couches inclus) par hub de référence SKU payant. 10 déploiements par hub de référence SKU gratuit. |
| Jumeaux<sup>1</sup> | La taille maximale des sections de propriétés souhaitées et de propriétés signalées est de 32 Ko chacune. La taille maximale de la section des étiquettes est de 8 Ko. |
| Stratégies d’accès partagé | Le nombre maximum de stratégies d’accès partagé est de 16. |
| Certificats d’autorité de certification x509 | Le nombre maximal de certificats d’autorité de certification x509 pouvant être inscrits sur IoT Hub est de 25. |

<sup>1</sup>Cette fonctionnalité n’est pas disponible dans le niveau de base d’IoT Hub. Pour plus d’informations, consultez [Comment choisir le bon IoT Hub](iot-hub-scaling.md).

## <a name="increasing-the-quota-or-throttle-limit"></a>Augmentation du quota ou de la limitation

À tout moment, vous pouvez augmenter les quotas ou les limitations en [augmentant le nombre d’unités approvisionnées dans un hub IoT](iot-hub-upgrade.md).

## <a name="latency"></a>Latence

IoT Hub s’efforce de fournir une faible latence pour toutes les opérations. Toutefois, en raison des conditions réseau et d’autres facteurs imprévisibles, il ne peut pas garantir une latence spécifique. Lorsque vous concevez votre solution, vous devez :

* Éviter de faire d’hypothèses concernant la latence maximale de toute opération IoT Hub.
* Configurer votre hub IoT dans la région Azure le plus proche de vos appareils.
* Envisager d’utiliser Azure IoT Edge pour effectuer des opérations sensibles à la latence sur l’appareil ou sur une passerelle proche de celui-ci.

Plusieurs unités IoT Hub affectent la limitation comme décrit précédemment, mais ne fournissent pas d’avantages ni de garanties supplémentaires en termes de latence.

Si vous constatez des augmentations inattendues de la latence des opérations, contactez le [Support Microsoft](https://azure.microsoft.com/support/options/).

## <a name="next-steps"></a>Étapes suivantes

Le billet de blog [IoT Hub throttling and you](https://azure.microsoft.com/blog/iot-hub-throttling-and-you/) fournit une présentation détaillée du comportement de limitation d’IoT Hub.

Les autres rubriques de référence de ce Guide du développeur IoT Hub comprennent :

* [Points de terminaison IoT Hub](iot-hub-devguide-endpoints.md)
* [Superviser avec IoT Hub](monitor-iot-hub.md)
