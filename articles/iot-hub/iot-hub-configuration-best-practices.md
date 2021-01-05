---
title: Bonnes pratiques en matière de configuration d’appareil pour Azure IoT Hub | Microsoft Docs
description: Découvrez les meilleures pratiques pour l’utilisation de la gestion automatique des périphériques afin de réduire les tâches répétitives et complexes impliquées dans la gestion des appareils IoT à l’échelle.
author: robinsh
ms.author: robinsh
ms.date: 06/28/2019
ms.topic: conceptual
ms.service: iot-hub
services: iot-hub
ms.openlocfilehash: 8a39c2b06ca8a0f852891acb60ba199fc2c6db5c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96024127"
---
# <a name="best-practices-for-device-configuration-within-an-iot-solution"></a>Bonnes pratiques en matière de configuration d’appareil dans une solution IoT

La gestion automatique des appareils dans Azure IoT Hub automatise une grande partie des tâches répétitives et complexes liées à la gestion de grandes flottes d’appareils pendant tout leur cycle de vie. Cet article présente un grand nombre des bonnes pratiques pour les différents rôles impliqués dans le développement et l’exploitation d’une solution IoT.

* **Fabricant/intégrateur de matériel IoT** : Il s’agit de fabricants de matériel IoT, d’intégrateurs qui assemblent des matériels provenant de différents fabricants, ou de fournisseurs de matériels dédiés à un déploiement IoT réalisé ou intégré par d’autres fournisseurs. Ils sont impliqués dans le développement et l’intégration de microprogrammes, de systèmes d’exploitation incorporés et de logiciels incorporés.

* **Développeur de solutions IoT** : le développement d’une solution IoT est généralement assuré par un développeur de solutions. Ce développeur peut faire partie d’une équipe interne ou être un intégrateur système spécialisé dans cette activité. Le développeur de solutions IoT peut développer plusieurs composants de la solution IoT en partant de zéro, intégrer de nombreux composants standard ou open source, ou encore personnaliser un [accélérateur de solution IoT](../iot-accelerators/index.yml).

* **Opérateur de solutions IoT** : une fois déployée, la solution IoT nécessite une exploitation, une surveillance, des mises à niveau et une maintenance sur le long terme. Ces tâches peuvent être assurées par une équipe interne comprenant des spécialistes en technologies de l’information, des équipes d’exploitation et de maintenance du matériel et des spécialistes du domaine qui contrôlent le comportement de l’infrastructure IoT globale.

## <a name="understand-automatic-device-management-for-configuring-iot-devices-at-scale"></a>Comprendre la gestion automatique des appareils pour la configuration des appareils IoT à grande échelle

La gestion automatique des appareils inclut les nombreux avantages des [jumeaux d’appareil](iot-hub-devguide-device-twins.md) et des [jumeaux de module](iot-hub-devguide-module-twins.md) pour synchroniser les états souhaités et signalés entre le cloud et les appareils. Les [configurations automatiques des appareils](./iot-hub-automatic-device-management.md) mettent automatiquement à jour de grands ensembles de jumeaux, et synthétisent la progression et la conformité. La procédure générale suivante décrit la façon dont la gestion automatique des appareils est développée et utilisée :

* Le **fabricant ou intégrateur de matériel IoT** implémente les fonctionnalités de gestion des appareils dans une application incorporée à l’aide de [jumeaux d’appareil](iot-hub-devguide-device-twins.md). Ces fonctionnalités peuvent inclure des mises à jour de microprogramme, l’installation et la mise à jour de logiciels ainsi que la gestion des paramètres.

* Le **développeur de solutions IoT** implémente la couche de gestion des opérations de gestion des appareils à l’aide de [jumeaux d’appareil](iot-hub-devguide-device-twins.md) et de [configurations automatiques des appareils](./iot-hub-automatic-device-management.md). La solution doit inclure la définition d’une interface d’opérateur pour effectuer les tâches de gestion des appareils.

* **L’opérateur de solutions IoT** utilise la solution IoT pour effectuer les tâches de gestion des appareils, en particulier pour regrouper les appareils, lancer des modifications de configuration telles que les mises à jour de microprogramme, surveiller la progression et résoudre les problèmes qui se posent.

## <a name="iot-hardware-manufacturerintegrator"></a>Fabricant/intégrateur de matériel IoT

Voici les bonnes pratiques pour les fabricants de matériel et les intégrateurs qui traitent du développement de logiciels incorporés :

* **Implémenter des [jumeaux d’appareils](iot-hub-devguide-device-twins.md) :** Les jumeaux d’appareil permettent la synchronisation de la configuration souhaitée à partir du cloud, ainsi que le signalement des propriétés de l’appareil et de la configuration actuelle. La meilleure façon d’implémenter des jumeaux d’appareil dans les applications incorporées consiste à utiliser les [SDK Azure IoT](https://github.com/Azure/azure-iot-sdks). Les jumeaux d’appareil sont mieux adaptés à la configuration, car ils :

    * Prennent en charge la communication bidirectionnelle
    * Autorisent à la fois les états d’appareils connectés et déconnectés
    * Suivent le principe de cohérence éventuelle
    * Peuvent entièrement être interrogés dans le cloud

* **Structurer le jumeau d’appareil pour la gestion des appareils :** Le jumeau d’appareil doit être structuré de façon à ce que les propriétés de gestion des appareils soient regroupées logiquement en sections. En procédant ainsi, les modifications de configuration sont isolées sans affecter les autres sections du jumeau. Par exemple, créez une section avec les propriétés souhaitées pour le microprogramme, une autre section pour les logiciels et une troisième pour les paramètres réseau. 

* **Signaler les attributs d’appareil qui sont utiles pour la gestion des appareils :** Les attributs comme la marque et le modèle de l’appareil physique, le microprogramme, le système d’exploitation, le numéro de série et d’autres identificateurs sont utiles pour les rapports et en tant que paramètres pour le ciblage des modifications de configuration.

* **Définir les principaux états pour les rapports sur l’état et la progression :** Les états de plus haut niveau doivent être énumérés afin qu’ils puissent être signalés à l’opérateur. Par exemple, une mise à jour de microprogramme signale un état comme Actuel, Téléchargement, Application, En cours et Erreur. Définissez des champs supplémentaires pour plus d’informations sur chaque état.

## <a name="iot-solution-developer"></a>Développeur de solutions IoT

Voici les bonnes pratiques pour les développeurs de solutions IoT qui génèrent des systèmes basés sur Azure :

* **Implémenter des [jumeaux d’appareils](iot-hub-devguide-device-twins.md) :** Les jumeaux d’appareil permettent la synchronisation de la configuration souhaitée à partir du cloud, ainsi que le signalement des propriétés de l’appareil et de la configuration actuelle. La meilleure façon d’implémenter des jumeaux d’appareil dans les applications de solutions cloud consiste à utiliser les [kits de développement logiciel (SDK) Azure IoT](https://github.com/Azure/azure-iot-sdks). Les jumeaux d’appareil sont mieux adaptés à la configuration, car ils :

    * Prennent en charge la communication bidirectionnelle
    * Autorisent à la fois les états d’appareils connectés et déconnectés
    * Suivent le principe de cohérence éventuelle
    * Peuvent entièrement être interrogés dans le cloud

* **Organiser les appareils en utilisant des étiquettes de jumeau d’appareil :** La solution doit permettre à l’opérateur de définir des anneaux de qualité ou d’autres ensembles d’appareils en fonction de différentes stratégies de déploiement, comme le contrôle de validité. L’organisation des appareils peut être implémentée dans votre solution à l’aide de balises de jumeau d’appareil et de [requêtes](iot-hub-devguide-query-language.md). Elle est nécessaire pour permettre des déploiements de configuration avec précision et en toute sécurité.

* **Implémenter des [configurations automatiques des appareils](./iot-hub-automatic-device-management.md) :** Les configurations automatiques des appareils déploient et supervisent les changements de configuration apportés à grands ensembles d’appareils IoT via des jumeaux d’appareil.

   Les configurations automatiques des appareils ciblent des ensembles de jumeaux d’appareil via la **condition cible**, qui est une requête sur les balises ou propriétés signalées du jumeau d’appareil. Le **contenu cible** est l’ensemble des propriétés voulues qui seront définies dans les jumeaux d’appareil ciblés. Le contenu cible doit s’aligner sur la structure du jumeau d’appareil définie par le fabricant/l’intégrateur de matériel IoT. Les **métriques** sont des requêtes sur les propriétés signalées du jumeau d’appareil et doivent également être alignées sur la structure du jumeau d’appareil définie par le fabricant/l’intégrateur de matériel IoT.

   Les configurations automatiques d’appareils s’exécutent pour la première fois peu de temps après la création de la configuration, puis à cinq minutes d’intervalle. Elles bénéficient également d’IoT Hub qui effectue des opérations de jumeau d’appareil à une fréquence qui ne dépasse jamais les [seuils de limitation](iot-hub-devguide-quotas-throttling.md) pour les lectures et mises à jour de jumeaux d’appareil.

* **Utiliser le [service Device Provisioning](../iot-dps/how-to-manage-enrollments.md) :** Les développeurs de solutions doivent utiliser le service Device Provisioning pour affecter des étiquettes de jumeau d’appareil aux nouveaux appareils, de façon à ce qu’ils soient automatiquement configurés par des **configurations automatiques des appareils** ciblées au niveau des jumeaux avec cette étiquette. 

## <a name="iot-solution-operator"></a>Opérateur de solutions IoT

Voici les bonnes pratiques pour les opérateurs de solutions IoT qui utilisent une solution IoT reposant sur Azure :

* **Organiser les appareils pour la gestion :** La solution IoT doit définir ou permettre la création d’anneaux de qualité ou d’autres ensembles d’appareils en fonction de différentes stratégies de déploiement, comme le contrôle de validité. Les ensembles d’appareils seront utilisés pour déployer les modifications de configuration et effectuer d’autres opérations de gestion des appareils à grande échelle.

* **Effectuer des changements de configuration en utilisant un déploiement par phases :**  Un déploiement par phases est un processus global par lequel un opérateur déploie les modifications sur un ensemble d’appareils IoT qui va en s’élargissant. L’objectif est d’apporter des modifications progressivement afin de réduire le risque d’étendre les modifications avec rupture à une plus grande échelle.    L’opérateur doit utiliser l’interface de la solution pour créer une [configuration automatique de l’appareil](./iot-hub-automatic-device-management.md), et la condition de ciblage doit cibler un ensemble initial d’appareils (par exemple, un groupe de contrôle de validité). L’opérateur doit ensuite valider la modification de configuration dans l’ensemble initial d’appareils.

   Une fois la validation terminée, l’opérateur met à jour la configuration automatique de l’appareil pour inclure un plus grand ensemble d’appareils. L’opérateur doit également définir une priorité pour la configuration qui soit supérieure à d’autres configurations actuellement ciblées sur ces appareils. Le déploiement peut être surveillé à l’aide des métriques signalées par la configuration automatique de l’appareil.

* **Effectuer des restaurations en cas d’erreurs ou de problèmes de configuration :**  Une configuration automatique de l’appareil qui provoque des erreurs ou des problèmes de configuration peut être restaurée en modifiant la **condition de ciblage**, afin que les appareils ne répondent plus à cette condition. Vérifiez qu’une autre configuration automatique de l’appareil de priorité inférieure est toujours ciblée sur ces appareils. Vérifiez que la restauration a réussi en consultant les métriques suivantes : La configuration restaurée ne doit plus afficher d’état pour les appareils non ciblés, et les métriques de la deuxième configuration doivent maintenant inclure des comptages pour les appareils qui sont toujours ciblés.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez l’implémentation des jumeaux d’appareil dans [Comprendre et utiliser les jumeaux d’appareil IoT Hub](iot-hub-devguide-device-twins.md).

* Suivez les étapes pour créer, mettre à jour ou supprimer une configuration automatique de l’appareil dans [Configurer et surveiller des appareils IoT à grande échelle](./iot-hub-automatic-device-management.md).

* Implémentez un modèle de mise à jour de microprogramme à l’aide de jumeaux d’appareil et de configurations automatiques des appareils dans [Tutoriel : Implémenter un processus de mise à jour de microprogramme d’appareil](tutorial-firmware-update.md).