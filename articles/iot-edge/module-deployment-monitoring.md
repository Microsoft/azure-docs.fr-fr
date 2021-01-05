---
title: Déploiement automatique pour les groupes d’appareils - Azure IoT Edge | Microsoft Docs
description: Utiliser des déploiements automatiques dans Azure IoT Edge pour gérer des groupes d’appareils en fonction d’étiquettes partagées
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 01/30/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 81db9c7e729aa0be67a807d9d77a3cccb8f41604
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85194788"
---
# <a name="understand-iot-edge-automatic-deployments-for-single-devices-or-at-scale"></a>Comprendre les déploiements automatiques IoT Edge pour un seul ou de nombreux appareils

Les déploiements automatiques et les déploiements en couches facilitent la gestion et la configuration de nombreux modules sur les appareils IoT Edge.

Azure IoT Edge propose deux méthodes de configuration des modules à exécuter sur des appareils IoT Edge. La première consiste à déployer les modules sur la base de chaque appareil. Vous créez un manifeste de déploiement, puis vous l’appliquez à un appareil donné par nom. La seconde méthode consiste à déployer automatiquement les modules sur tous les appareils inscrits répondant à un ensemble de conditions définies. Vous créez un manifeste de déploiement, puis définissez les appareils auxquels il s’applique en fonction des [indicateurs](../iot-edge/how-to-deploy-at-scale.md#identify-devices-using-tags) dans le jumeau d'appareil.

Cet article se concentre sur la configuration et la supervision de flottes d’appareils, collectivement appelées *déploiements automatiques IoT Edge*. Les étapes de déploiement de base sont les suivantes :

1. Un opérateur définit un déploiement décrivant un ensemble de modules, ainsi que les appareils cibles. Chaque déploiement possède un manifeste de déploiement qui reflète ces informations.
2. Le service de IoT Hub communique avec tous les appareils ciblés pour les configurer avec les modules souhaités.
3. Le service IoT Hub récupère l’état des appareils IoT Edge et les met à la disposition de l’opérateur.  Par exemple, un opérateur voit quand un appareil Edge n’est pas correctement configuré ou si un module échoue en cours de runtime.
4. À tout moment, les nouveaux appareils IoT Edge qui remplissent les conditions de ciblage sont configurés pour le déploiement.

Cet article décrit chaque composant impliqué dans la configuration et la surveillance d’un déploiement. Pour connaître la procédure de création et de mise à jour d’un déploiement, consultez [Déployer et surveiller des modules IoT Edge à l’échelle](how-to-deploy-at-scale.md).

## <a name="deployment"></a>Déploiement

Un déploiement automatique IoT Edge assigne des images de module IoT Edge à exécuter en tant qu’instances sur un ensemble ciblé d’appareils IoT Edge. Il fonctionne en configurant un manifeste de déploiement IoT Edge pour inclure une liste de modules comprenant les paramètres d’initialisation correspondants. Un déploiement peut être affecté à un appareil unique (basé sur l’ID de l’appareil) ou à un groupe d’appareils (basé sur des balises). Lorsqu’un appareil IoT Edge reçoit un manifeste de déploiement, il télécharge et installe les images conteneurs dans les référentiels conteneurs respectifs, puis les configure en conséquence. Une fois qu’un déploiement est créé, un opérateur peut surveiller l’état de déploiement pour voir si les appareils ciblés sont configurés correctement.

Seuls les appareils IoT Edge peuvent être configurés avec un déploiement. L’appareil destiné à recevoir le déploiement doit satisfaire aux prérequis suivants :

* Le système d’exploitation de base
* Un système de gestion de conteneur, comme Moby ou Docker
* L’approvisionnement du runtime IoT Edge

### <a name="deployment-manifest"></a>Manifeste de déploiement

Un manifeste de déploiement est un document JSON qui décrit les modules devant être configurés sur les appareils IoT Edge ciblés. Il contient les métadonnées de configuration pour tous les modules, y compris les modules système requis (en particulier l’agent de IoT Edge et l’hub IoT Edge).  

Les métadonnées de configuration pour chaque module incluent :

* Version
* Type
* État (par exemple, en cours d’exécution ou arrêté)
* Stratégie de redémarrage
* Registre d’images et de conteneurs
* Itinéraires pour les données en entrée et en sortie

Si l’image du module est stockée dans un registre de conteneurs privé, l’agent IoT Edge conserve les informations d’identification du registre.

### <a name="target-condition"></a>Condition cible

La condition cible est évaluée en permanence sur toute la durée de vie du déploiement. Les nouveaux appareils qui répondent aux exigences sont inclus ; tous les appareils existants qui n’y satisfont plus sont supprimés. Le déploiement est réactivé si le service détecte une modification de la condition cible.

Prenons l’exemple d’un déploiement comportant une condition cible tags.environment = ’prod’. Au lancement du déploiement, il y a 10 appareils de production. Les modules sont correctement installés sur ces 10 appareils. L’état de l’agent IoT Edge affiche 10 appareils au total, 10 réponses correctes, 0 réponse erronée et 0 réponse en attente. On ajoute maintenant cinq appareils avec tags.environment = 'prod'. Le service détecte la modification, et l’état de l’agent IoT Edge devient : 15 appareils au total, 10 réponses correctes, 0 réponse erronée et 5 réponses en attente quand il déploie sur cinq nouveaux appareils.

Utilisez une condition booléenne sur des balises de jumeaux d’appareils, propriétés signalées de jumeaux d'appareils ou deviceId pour sélectionner les appareils cibles. Si vous souhaitez utiliser une condition avec des balises, vous devez ajouter les propriétés "tags":{} dans le jumeau d’appareil au même niveau. [En savoir plus sur les balises dans le jumeau d’appareil](../iot-hub/iot-hub-devguide-device-twins.md)

Exemples de conditions cibles :

* deviceId =’linuxprod1’
* tags.environment =’prod’
* tags.environment = ’prod’ AND tags.location = ’westus’
* tags.environment = ’prod’ OR tags.location = ’westus’
* tags.operator = 'John' AND tags.environment = 'prod' AND NOT deviceId = 'linuxprod1'
* properties.reported.devicemodel = '4000x'

Prenez en compte les contraintes suivantes lorsque vous créez une condition cible :

* Dans le jumeau d’appareil, seuls les balises, propriétés signalées et deviceId permettent de créer une condition cible.
* Les guillemets doubles ne sont autorisés nulle part dans la condition cible. Utilisez des guillemets simples.
* Les guillemets simples représentent les valeurs de la condition cible. Par conséquent, vous devez échapper le guillemet simple avec un autre guillemet simple s’il fait partie du nom de l’appareil. Par exemple, pour cibler un appareil nommé `operator'sDevice`, écrivez `deviceId='operator''sDevice'`.
* Les nombres, les lettres et les caractères suivants sont autorisés dans les valeurs de la condition cible : `-:.+%_#*?!(),=@;$`.

### <a name="priority"></a>Priority

Une priorité définit si un déploiement doit être appliqué à un appareil ciblé par rapport à d’autres déploiements. Une priorité de déploiement est un entier positif. Plus le nombre est élevé, plus la priorité est supérieure. Si un périphérique IoT Edge est ciblé par plusieurs déploiements, le déploiement avec la priorité la plus élevée s’applique.  Les déploiements avec une priorité plus faible ne sont pas appliqués, ni fusionnés.  Si un périphérique est ciblé par au moins deux déploiements de priorité égale, c’est le déploiement créé le plus récemment (déterminé par l’horodatage de création) qui s’applique.

### <a name="labels"></a>Étiquettes

Les étiquettes sont des paires clé-valeur de type chaîne que vous pouvez utiliser pour filtrer et grouper les déploiements. Un déploiement peut avoir plusieurs étiquettes. Les étiquettes sont facultatives et n’ont aucun impact sur la configuration des appareils IoT Edge.

### <a name="metrics"></a>Mesures

Par défaut, tous les déploiements rapportent quatre métriques :

* **Ciblé** affiche les périphériques IoT Edge qui correspondent à la condition de ciblage du déploiement.
* **Appliqué** affiche les appareils IoT Edge ciblés qui ne sont pas ciblés par un autre déploiement de priorité plus élevée.
* **Signalement d'une réussite** affiche les appareils IoT Edge ayant signalé le bon déploiement des modules.
* **Signalement d’un échec** affiche les appareils IoT Edge ayant signalé l'échec de déploiement d'un ou plusieurs modules. Pour examiner l’erreur plus en détail, connectez-vous à distance à ces appareils et consultez les fichiers journaux.

En outre, vous pouvez définir vos propres mesures personnalisées pour faciliter la surveillance et la gestion du déploiement.

Les métriques fournissent des nombres récapitulatifs des différents états qu’un appareil peut signaler après l’application d’une configuration de déploiement. Les métriques peuvent interroger des [propriétés signalées du jumeau de module edgeHub](module-edgeagent-edgehub.md#edgehub-reported-properties), telles que *lastDesiredStatus* ou *lastConnectTime*. Par exemple :

```sql
SELECT deviceId FROM devices
  WHERE properties.reported.lastDesiredStatus.code = 200
```

L'ajout de vos propres métriques est facultatif et n’a aucun impact sur la configuration des appareils IoT Edge.

## <a name="layered-deployment"></a>Déploiement en couches

Les déploiements en couches sont des déploiements automatiques qui peuvent être combinés afin de réduire le nombre de déploiements uniques à créer. Les déploiements en couches s'avèrent utiles dans les scénarios où les mêmes modules sont réutilisés selon différentes combinaisons dans de nombreux déploiements automatiques.

Les déploiements en couches possèdent les mêmes composants de base que n’importe quel déploiement automatique. Ils ciblent les appareils en fonction des balises présentes dans les jumeaux d'appareil et fournissent les mêmes fonctionnalités en termes d'étiquettes, de métriques et de rapports d’état. Les déploiements en couches sont également associés à des priorités et celles-ci sont utilisées pour déterminer le classement de plusieurs déploiements sur un appareil, et non le déploiement à lui appliquer. Par exemple, si deux déploiements en couches présentent un module ou un itinéraire portant le même nom, le déploiement en couches doté de la priorité la plus élevée est appliqué et remplace la priorité la plus faible.

Les modules d’exécution du système, edgeAgent et edgeHub, ne sont pas configurés dans le cadre d’un déploiement en couches. Tout appareil IoT Edge ciblé par un déploiement en couches doit d'abord faire l'objet d'un déploiement automatique standard. Ce déploiement automatique fait office de base sur laquelle ajouter les déploiements en couches.

Un appareil IoT Edge ne peut appliquer qu'un seul déploiement automatique standard, mais peut appliquer plusieurs déploiements automatiques en couches. Les déploiements en couches ciblant un appareil doivent avoir une priorité plus élevée que le déploiement automatique pour cet appareil.

Prenons pour exemple une entreprise gérant des bâtiments. Cette dernière a développé des modules IoT Edge pour collecter des données à partir de caméras de sécurité, de capteurs de mouvement et d’ascenseurs. Cela étant, tous ses bâtiments ne peuvent pas utiliser les trois modules. Avec les déploiements automatiques standard, l’entreprise doit créer des déploiements individuels pour toutes les combinaisons de modules dont ses bâtiments ont besoin.

![Les déploiements automatiques standard doivent s’adapter à chaque combinaison de modules.](./media/module-deployment-monitoring/standard-deployment.png)

Toutefois, lorsque l'entreprise bascule vers des déploiements automatiques en couches, elle peut créer les mêmes combinaisons de modules pour ses bâtiments, tout en réduisant ses déploiements à gérer. Chaque module possède son propre déploiement en couches, et les balises des appareils identifient les modules ajoutés à chaque bâtiment.

![Le déploiement automatique en couches simplifie les scénarios dans lesquels les mêmes modules sont combinés de différentes façons](./media/module-deployment-monitoring/layered-deployment.png)

### <a name="module-twin-configuration"></a>Configuration de jumeau de module

Lorsque vous utilisez les déploiements en couches, vous pouvez, intentionnellement ou non, avoir deux déploiements avec le même module ciblant un appareil. Dans ces cas, vous pouvez décider si le déploiement doté d’une priorité plus élevée doit remplacer le jumeau de module ou s’y ajouter. Par exemple, vous pouvez avoir un déploiement qui applique le même module à 100 appareils différents. Pour autant, dix de ces appareils se trouvent dans des installations sécurisées et nécessitent une configuration supplémentaire pour pouvoir communiquer via les serveurs proxy. Vous pouvez utiliser un déploiement en couches pour ajouter des propriétés de jumeau de module permettant à ces dix appareils de communiquer en toute sécurité, sans remplacer les informations de jumeau de module existantes du déploiement de base.

Vous pouvez ajouter les propriétés de jumeau de module souhaitées dans le manifeste de déploiement. Alors que dans un déploiement standard, vous ajoutez des propriétés à la section **properties.desired** du jumeau de module, dans un déploiement en couches, vous pouvez déclarer un nouveau sous-ensemble de propriétés souhaitées.

Par exemple, dans un déploiement standard, vous pouvez ajouter le module de capteur de température simulé avec les propriétés souhaitées suivantes pour lui indiquer d’envoyer les données à intervalles de 5 secondes :

```json
"SimulatedTemperatureSensor": {
  "properties.desired": {
    "SendData": true,
    "SendInterval": 5
  }
}
```

Dans un déploiement en couches ciblant tout ou partie des appareils identiques, vous pouvez ajouter une propriété indiquant au capteur simulé d’envoyer 1 000 messages, puis de s’arrêter. Pour ne pas remplacer les propriétés existantes, vous créez une section dans les propriétés souhaitées appelée `layeredProperties`, contenant la nouvelle propriété :

```json
"SimulatedTemperatureSensor": {
  "properties.desired.layeredProperties": {
    "StopAfterCount": 1000
  }
}
```

Un appareil auquel les deux déploiements sont appliqués indique les propriétés suivantes dans le jumeau de module pour le capteur de température simulé :

```json
"properties": {
  "desired": {
    "SendData": true,
    "SendInterval": 5,
    "layeredProperties": {
      "StopAfterCount": 1000
    }
  }
}
```

Si vous définissez le champ `properties.desired` du jumeau de module dans un déploiement en couches, les propriétés souhaitées pour ce module sont remplacées dans les déploiements de priorité inférieure.

## <a name="phased-rollout"></a>Déploiement progressif

Un déploiement progressif est un processus global par lequel un opérateur déploie les modifications sur un ensemble étendu d’appareils IoT Edge. L’objectif est d’apporter des modifications progressivement afin de réduire le risque d’étendre les modifications avec rupture à une plus grande échelle. Les déploiements automatiques facilitent la gestion des déploiements progressifs sur une flotte d'appareils IoT Edge.

Un déploiement progressif est exécuté dans les phases et étapes suivantes :

1. Établissez un environnement de tests d’appareils IoT Edge en les approvisionnant et en paramétrant une balise de jumeau d’appareil comme `tag.environment='test'`. L’environnement de test doit refléter l’environnement de production que le déploiement va cibler.
2. Créez un déploiement comprenant les modules et les configurations souhaités. La condition de ciblage doit cibler l’environnement de test des appareils IoT Edge.
3. Validez la nouvelle configuration du module dans l’environnement de test.
4. Mettez à jour le déploiement pour inclure un sous-ensemble d’appareils de production IoT Edge en ajoutant une nouvelle balise à la condition de ciblage. En outre, assurez-vous que la priorité pour le déploiement est supérieure aux autres déploiements actuellement ciblés sur ces appareils.
5. Vérifiez que le déploiement a réussi sur les appareils IoT ciblés en consultant l’état du déploiement.
6. Mettez à jour le déploiement afin de cibler tous les appareils de production IoT Edge restants.

## <a name="rollback"></a>Restauration

Les déploiements peuvent être restaurés en cas d’erreur ou de mauvaise configuration. Étant donné qu’un déploiement définit la configuration de module absolue pour un appareil IoT Edge, un déploiement supplémentaire doit cibler le même appareil à une priorité inférieure, même si l’objectif est de supprimer tous les modules.  

La suppression d’un déploiement ne supprime pas les modules des appareils ciblés. Un autre déploiement doit définir une nouvelle configuration pour les appareils, même s’il s’agit d’un déploiement vide.

Effectuez des restaurations dans l’ordre suivant :

1. Confirmez qu’un deuxième déploiement cible également le même ensemble d’appareils. Si l’objectif de la restauration est de supprimer tous les modules, le deuxième déploiement ne doit pas contenir de module.
2. Modifiez ou supprimez l’expression de la condition cible du déploiement que vous souhaitez restaurer de façon à ce que les appareils ne répondent pas à la condition de ciblage.
3. Vérifiez que la restauration a réussi en consultant l’état du déploiement.
   * Le déploiement restauré ne doit plus afficher d’état pour les appareils qui ont été restaurés.
   * Le deuxième déploiement doit désormais inclure l’état de déploiement pour les appareils qui ont été restaurés.

## <a name="next-steps"></a>Étapes suivantes

* Suivez les étapes pour créer, mettre à jour ou supprimer un déploiement dans [Déployer et surveiller les modules IoT Edge à grande échelle](how-to-deploy-at-scale.md).
* En savoir plus sur d’autres concepts IoT Edge tels que le [runtime IoT Edge](iot-edge-runtime.md) et les [modules IoT Edge](iot-edge-modules.md).
