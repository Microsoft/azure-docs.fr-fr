---
title: Gestion automatique des appareils à grande échelle Azure IoT Hub (CLI) | Microsoft Docs
description: Utiliser des configurations automatiques Azure IoT Hub pour gérer plusieurs Modules ou appareils IoT
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 12/13/2019
ms.author: robinsh
ms.openlocfilehash: 0b8b499613f8234f449e6d72f6ed6ec1f2f21287
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92545410"
---
# <a name="automatic-iot-device-and-module-management-using-the-azure-cli"></a>Gestion automatique des Modules et des appareils IoT avec Azure CLI

[!INCLUDE [iot-edge-how-to-deploy-monitor-selector](../../includes/iot-hub-auto-device-config-selector.md)]

La gestion automatique des appareils dans Azure IoT Hub automatise une grande partie des tâches répétitives et complexes liées à la gestion de grandes flottes d’appareils. Avec la gestion automatique des appareils, vous pouvez cibler un ensemble d’appareils en fonction de leurs propriétés, définir la configuration souhaitée et laisser IoT Hub mettre à jour les appareils quand ils se trouvent dans l’étendue. Cette opération est effectuée à l’aide d’une _configuration d’appareil automatique_ ou d’une _configuration de Module automatique_ , qui vous permet de récapituler les données d’achèvement et de conformité, de gérer la fusion et les conflits, et de déployer les configurations de façon progressive.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

La gestion automatique des appareils fonctionne en mettant à jour un ensemble de jumeaux d’appareil ou des jumeaux de Module avec les propriétés souhaitées et en communiquant un rapport récapitulatif basé sur les propriétés signalées du jumeau.  Cette approche introduit un nouveau document de classe JSON appelé *Configuration* qui comprend trois parties :

* La **condition cible** définit l’étendue des jumeaux d’appareil ou jumeaux de Module à mettre à jour. La condition cible est spécifiée en tant que requête sur les balises de jumeaux d’appareil et/ou sur les propriétés signalées.

* Le **contenu cible** définit les propriétés souhaitées à ajouter ou à mettre à jour dans les jumeaux d’appareil ou de Module ciblés. Le contenu inclut un chemin de la section des propriétés souhaitées à changer.

* Les **métriques** définissent les nombres récapitulatifs des différents états de configuration tels que **Réussite** , **En cours** et **Erreur** . Les indicateurs de performance personnalisés sont spécifiés en tant que requêtes sur les propriétés signalées du jumeau.  Les indicateurs de performance système sont des indicateurs de performance par défaut qui mesurent l’état de la mise à jour des jumeaux, comme le nombre de jumeaux qui sont ciblés et le nombre de jumeaux qui ont été correctement mis à jour.

Les configurations automatiques s’exécutent pour la première fois peu de temps après la création de la configuration, puis à cinq minutes d’intervalle. Les requêtes relatives aux métriques s’exécutent à chaque exécution de la configuration automatique.

## <a name="cli-prerequisites"></a>Prérequis pour l’interface CLI

* Un [hub IoT](../iot-hub/iot-hub-create-using-cli.md) dans votre abonnement Azure. 

* [Azure CLI](/cli/azure/install-azure-cli) dans votre environnement. Vous devez utiliser Azure CLI version 2.0.70 ou ultérieure. Utilisez `az –-version` pour valider. Cette version prend en charge les commandes d’extension az et introduit l’infrastructure de la commande Knack. 

* [Extension IoT pour Azure CLI](https://github.com/Azure/azure-cli).

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="implement-twins"></a>Implémenter des jumeaux

Les configurations d’appareil automatiques nécessitent l’utilisation de jumeaux d’appareil pour synchroniser l’état entre les appareils et le cloud.  Pour en savoir plus, consultez [Comprendre et utiliser les jumeaux d’appareil IoT Hub](iot-hub-devguide-device-twins.md).

Les configurations de Module automatiques nécessitent l’utilisation de jumeaux de Module pour synchroniser l’état entre les Modules et le cloud. Pour en savoir plus, consultez [Comprendre et utiliser les jumeaux de Module IoT Hub](iot-hub-devguide-module-twins.md).

## <a name="use-tags-to-target-twins"></a>Utiliser des balises pour cibler des jumeaux

Avant de pouvoir créer une configuration, vous devez spécifier les appareils ou les Modules concernés. Azure IoT Hub identifie des appareils et utilise des balises dans le jumeau de Module et identifie les Modules à l’aide de balises dans le jumeau de Module. Chaque appareil ou Module peut avoir plusieurs balises, et vous pouvez les définir comme bon vous semble pour votre solution. Par exemple, si vous gérez des appareils à différents emplacements, ajoutez les étiquettes suivantes à un jumeau d’appareil :

```json
"tags": {
    "location": {
        "state": "Washington",
        "city": "Tacoma"
    }
},
```

## <a name="define-the-target-content-and-metrics"></a>Définir le contenu cible et les métriques

Le contenu cible et les requêtes de métriques sont spécifiés sous forme de documents JSON qui décrivent les propriétés souhaitées du jumeau d’appareil ou de Module à définir et les propriétés indiquées à mesurer.  Pour créer une configuration automatique avec Azure CLI, enregistrez le contenu cible et les métriques localement dans des fichiers .txt. Vous utiliserez les chemins des fichiers dans une section ultérieure, au moment d’exécuter la commande pour appliquer la configuration à votre appareil.

Voici un exemple de contenu cible de base pour une configuration d’appareil automatique :

```json
{
  "content": {
    "deviceContent": {
      "properties.desired.chillerWaterSettings": {
        "temperature": 38,
        "pressure": 78
      }
    }
}
```

Les configurations automatiques de Module se comportent de façon similaire, mais vous ciblez `moduleContent` au lieu de `deviceContent`.

```json
{
  "content": {
    "moduleContent": {
      "properties.desired.chillerWaterSettings": {
        "temperature": 38,
        "pressure": 78
      }
    }
}
```

Voici des exemples de requêtes de métrique :

```json
{
  "queries": {
    "Compliant": "select deviceId from devices where configurations.[[chillerdevicesettingswashington]].status = 'Applied' AND properties.reported.chillerWaterSettings.status='current'",
    "Error": "select deviceId from devices where configurations.[[chillerdevicesettingswashington]].status = 'Applied' AND properties.reported.chillerWaterSettings.status='error'",
    "Pending": "select deviceId from devices where configurations.[[chillerdevicesettingswashington]].status = 'Applied' AND properties.reported.chillerWaterSettings.status='pending'"
  }
}
```

Les requêtes de métriques pour les Modules sont également similaires aux requêtes pour les appareils, mais vous sélectionnez pour `moduleId` dans `devices.modules`. Par exemple : 

```json
{
  "queries": {
    "Compliant": "select deviceId, moduleId from devices.module where configurations.[[chillermodulesettingswashington]].status = 'Applied' AND properties.reported.chillerWaterSettings.status='current'"
  }
}
```

## <a name="create-a-configuration"></a>Créer une configuration

Vous configurez des appareils cibles en créant une configuration composée du contenu cible et des métriques. 

Utilisez la commande suivante pour créer une configuration :

```azurecli
   az iot hub configuration create --config-id [configuration id] \
     --labels [labels] --content [file path] --hub-name [hub name] \
     --target-condition [target query] --priority [int] \
     --metrics [metric queries]
```

* --**config-id** : nom de la configuration à créer dans le hub IoT. Donnez à votre configuration un nom unique comportant au plus 128 lettres minuscules. Évitez les espaces et les caractères non valides suivants : `& ^ [ ] { } \ | " < > /`.

* --**labels** : ajoutez des étiquettes pour faciliter le suivi de votre configuration. Les étiquettes sont des paires Nom, Valeur qui décrivent votre déploiement. Par exemple, `HostPlatform, Linux` ou `Version, 3.0.1`.

* --**content** : JSON inline ou chemin du contenu cible à définir en tant que propriétés de jumeau souhaitées. 

* --**hub-name** : nom du hub IoT dans lequel la configuration sera créée. Le hub doit être dans l’abonnement actuel. Basculez vers l’abonnement souhaité avec la commande `az account set -s [subscription name]`.

* --**target-condition** : entrez une condition cible pour déterminer quels sont les appareils ou Modules ciblés par cette configuration.  Pour une configuration d’appareil automatique, la condition est basée sur les balises de jumeau d’appareil ou sur les propriétés souhaitées du jumeau d’appareil et doit correspondre au format de l’expression.  Par exemple, `tags.environment='test'` ou `properties.desired.devicemodel='4000x'`.  Pour la configuration de Module automatique, la condition est basée sur les balises ou les propriétés souhaitées d’un jumeau de Module. Par exemple, `from devices.modules where tags.environment='test'` ou `from devices.modules where properties.reported.chillerProperties.model='4000x'`.

* --**priority** : entier positif. Si deux ou plusieurs configurations sont ciblées sur le même appareil ou Module, la configuration ayant la valeur numérique la plus élevée pour Priority s’applique.

* --**metrics** : chemin des requêtes de métriques. Les métriques fournissent des nombres récapitulatifs des différents états qu’un appareil ou Module peut signaler après l’application d’un contenu de configuration. Par exemple, vous pouvez créer une métrique pour les modifications de paramètres en attente, une métrique pour les erreurs et une métrique pour les modifications de paramètres réussies. 

## <a name="monitor-a-configuration"></a>Surveiller une configuration

Utilisez la commande suivante pour afficher le contenu d’une configuration :

```azurecli
az iot hub configuration show --config-id [configuration id] \
  --hub-name [hub name]
```

* --**config-id** : nom de la configuration existante dans le hub IoT.

* --**hub-name** : nom du hub IoT dans lequel la configuration existe. Le hub doit être dans l’abonnement actuel. Basculez vers l’abonnement souhaité avec la commande `az account set -s [subscription name]`.

Inspectez la configuration dans la fenêtre de commande.  La propriété **metrics** répertorie un nombre pour chaque métrique évaluée par chaque hub :

* **targetedCount** : métrique système qui spécifie le nombre de jumeaux d’appareil ou de Module dans le hub IoT qui correspondent à la condition de ciblage.

* **appliedCount** : métrique système qui spécifie le nombre d’appareils ou de Modules pour lesquels le contenu cible a été appliqué.

* **Votre métrique personnalisée**  : les métriques que vous avez définies sont des métriques utilisateur.

Vous pouvez afficher une liste d’ID d’appareil, d’ID de Module ou d’objets pour chacune des métriques à l’aide de la commande suivante :

```azurecli
az iot hub configuration show-metric --config-id [configuration id] \
   --metric-id [metric id] --hub-name [hub name] --metric-type [type] 
```

* --**config-id** : nom du déploiement qui existe dans le hub IoT.

* --**metric-id** : nom de la métrique pour laquelle vous souhaitez voir la liste des ID d’appareil ou de Module, par exemple `appliedCount`.

* --**hub-name** : nom du hub IoT dans lequel le déploiement existe. Le hub doit être dans l’abonnement actuel. Basculez vers l’abonnement souhaité avec la commande `az account set -s [subscription name]`.

* --**metric-type** : le type de métrique peut être `system` ou `user`.  Les métriques système sont `targetedCount` et `appliedCount`. Toutes les autres métriques sont des métriques utilisateur.

## <a name="modify-a-configuration"></a>Modifier une configuration

Quand vous modifiez une configuration, les modifications sont répliquées immédiatement sur tous les appareils ciblés. 

Si vous mettez à jour la condition cible, les mises à jour suivantes se produisent :

* Si un jumeau ne remplissait pas l’ancienne condition cible, mais qu’il remplit la nouvelle condition cible et que cette configuration a la priorité la plus élevée pour ce jumeau, cette configuration est appliquée. 

* Si un jumeau exécutant cette configuration ne remplit plus la condition cible, les paramètres de la configuration sont supprimés et le jumeau est Modifié par la configuration ayant la priorité la plus élevée suivante. 

* Si un jumeau exécutant cette configuration ne remplit plus la condition cible et ne remplit pas la condition cible des autres configurations, les paramètres de la configuration sont supprimés et aucune autre Modification n’est effectuée sur le jumeau. 

Utilisez la commande suivante pour mettre à jour une configuration :

```azurecli
az iot hub configuration update --config-id [configuration id] \
   --hub-name [hub name] --set [property1.property2='value']
```

* --**config-id** : nom de la configuration existante dans le hub IoT.

* --**hub-name** : nom du hub IoT dans lequel la configuration existe. Le hub doit être dans l’abonnement actuel. Basculez vers l’abonnement souhaité avec la commande `az account set -s [subscription name]`.

* --**set** : met à jour une propriété dans la configuration. Vous pouvez mettre à jour les propriétés suivantes :

    * targetCondition : par exemple, `targetCondition=tags.location.state='Oregon'`

    * étiquettes 

    * priority

## <a name="delete-a-configuration"></a>Supprimer une configuration

Quand vous supprimez une configuration, tous les jumeaux d’appareil ou de Module prennent leur configuration suivante dans l’ordre de priorité. Si des jumeaux ne remplissent la condition cible d’aucune autre configuration, aucun autre paramètre n’est appliqué. 

Utilisez la commande suivante pour supprimer une configuration :

```azurecli
az iot hub configuration delete --config-id [configuration id] \
   --hub-name [hub name] 
```

* --**config-id** : nom de la configuration existante dans le hub IoT.

* --**hub-name** : nom du hub IoT dans lequel la configuration existe. Le hub doit être dans l’abonnement actuel. Basculez vers l’abonnement souhaité avec la commande `az account set -s [subscription name]`.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert comment configurer et superviser des appareils IoT à grande échelle. Suivez ces liens pour en savoir plus sur la gestion de Azure IoT Hub :

* [Gestion de vos identités d’appareil IoT Hub en bloc](iot-hub-bulk-identity-mgmt.md)
* [Surveiller votre IoT Hub](monitor-iot-hub.md)

Pour explorer davantage les capacités de IoT Hub, consultez :

* [Guide du développeur d’IoT Hub](iot-hub-devguide.md)
* [Déploiement d’une IA sur des appareils de périmètre avec Azure IoT Edge](../iot-edge/quickstart-linux.md)

Pour savoir comment utiliser le service d’approvisionnement des appareils IoT Hub afin d’activer l’approvisionnement sans contact et juste-à-temps, consultez : 

* [Service Azure IoT Hub Device Provisioning](../iot-dps/index.yml)