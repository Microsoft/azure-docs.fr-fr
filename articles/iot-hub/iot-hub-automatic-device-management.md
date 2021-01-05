---
title: Gestion automatique des appareils à grande échelle Azure IoT Hub | Microsoft Docs
description: Utiliser des configurations automatiques Azure IoT Hub pour gérer plusieurs Modules et appareils IoT
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 12/13/2019
ms.author: robinsh
ms.custom:
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: 0e017f4df413d6db528bb99756646859d9a74aea
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92545393"
---
# <a name="automatic-iot-device-and-module-management-using-the-azure-portal"></a>Gestion automatique des Modules et des appareils IoT avec le portail Azure

[!INCLUDE [iot-edge-how-to-deploy-monitor-selector](../../includes/iot-hub-auto-device-config-selector.md)]

La gestion automatique des appareils dans Azure IoT Hub automatise une grande partie des tâches répétitives et complexes liées à la gestion de grandes flottes d’appareils. Avec la gestion automatique des appareils, vous pouvez cibler un ensemble d’appareils en fonction de leurs propriétés, définir la configuration souhaitée et laisser IoT Hub mettre à jour les appareils quand ils se trouvent dans l’étendue. Cette opération est effectuée à l’aide d’une _configuration d’appareil automatique_ ou d’une _configuration de Module automatique_ , qui vous permet de récapituler les données d’achèvement et de conformité, de gérer la fusion et les conflits, et de déployer les configurations de façon progressive.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

La gestion automatique des appareils fonctionne en mettant à jour un ensemble de jumeaux d’appareil ou des jumeaux de Module avec les propriétés souhaitées et en communiquant un rapport récapitulatif basé sur les propriétés signalées du jumeau.  Cette approche introduit un nouveau document de classe JSON appelé *Configuration* qui comprend trois parties :

* La **condition cible** définit l’étendue des jumeaux d’appareil ou jumeaux de Module à mettre à jour. La condition cible est spécifiée en tant que requête sur les balises de jumeaux et/ou sur les propriétés signalées.

* Le **contenu cible** définit les propriétés souhaitées à ajouter ou à mettre à jour dans les jumeaux d’appareil ou de Module ciblés. Le contenu inclut un chemin de la section des propriétés souhaitées à changer.

* Les **métriques** définissent les nombres récapitulatifs des différents états de configuration tels que **Réussite** , **En cours** et **Erreur**. Les indicateurs de performance personnalisés sont spécifiés en tant que requêtes sur les propriétés signalées du jumeau.  Les indicateurs de performance système sont des indicateurs de performance par défaut qui mesurent l’état de la mise à jour des jumeaux, comme le nombre de jumeaux qui sont ciblés et le nombre de jumeaux qui ont été correctement mis à jour.

Les configurations automatiques s’exécutent pour la première fois peu de temps après la création de la configuration, puis à cinq minutes d’intervalle. Les requêtes relatives aux indicateurs de performance s’exécutent à chaque exécution de la configuration automatique.

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

## <a name="create-a-configuration"></a>Créer une configuration

1. Accédez à votre IoT Hub dans le [Portail Azure](https://portal.azure.com). 

2. Sélectionnez **Configuration de l’appareil IoT**.

3. Sélectionnez **Ajouter une configuration d’appareil** ou **Ajouter une configuration de Module**.

   ![Ajouter une configuration d’appareil ou Ajouter une configuration de Module](./media/iot-hub-automatic-device-management/create-automatic-configuration.png)

La création d’une configuration nécessite cinq étapes. Les sections suivantes les décrivent en détail. 

### <a name="name-and-label"></a>Nom et étiquette

1. Donnez à votre configuration un nom unique comportant au plus 128 lettres minuscules. Évitez les espaces et les caractères non valides suivants : `& ^ [ ] { } \ | " < > /`.

2. Ajoutez des étiquettes pour faciliter le suivi de vos configurations. Les étiquettes sont des paires **Nom** , **Valeur** qui décrivent votre configuration. Par exemple, `HostPlatform, Linux` ou `Version, 3.0.1`.

3. Cliquez sur **Suivant** pour passer à l’étape suivante. 

### <a name="specify-settings"></a>Spécifier les paramètres

Cette section définit le contenu à définir dans les jumeaux d’appareil ou de Module ciblés. Il existe deux entrées pour chaque ensemble de paramètres. La première est le chemin du jumeau, qui est le chemin de la section JSON dans les propriétés souhaitées du jumeau qui seront définies.  La seconde est le contenu JSON à insérer dans cette section. 

Par exemple, vous pouvez définir le chemin du jumeau sur `properties.desired.chiller-water`, puis fournir le contenu JSON suivant : 

```json
{
  "temperature": 66,
  "pressure": 28
}
```

![Définir le chemin du jumeau et le contenu](./media/iot-hub-automatic-device-management/module-config-twin-settings.png)


Vous pouvez également définir des paramètres individuels en spécifiant le chemin du jumeau complet et la valeur sans crochets. Par exemple, avec le chemin du jumeau `properties.desired.chiller-water.temperature`, définissez le contenu sur `66`. Créez ensuite un paramètre de jumeau pour la propriété de pression. 

Si deux ou plusieurs configurations ciblent le même chemin de jumeau, le contenu de la configuration ayant la priorité la plus élevée s’applique (la priorité est définie à l’étape 4).

Si vous souhaitez supprimer une propriété existante, définissez la valeur de la propriété sur `null`.

Vous pouvez ajouter des paramètres supplémentaires en sélectionnant **Ajouter un paramètre de jumeau d’appareil** ou **Ajouter un paramètre de jumeau de Module**.

### <a name="specify-metrics-optional"></a>Spécifier les métriques (facultatif)

Les métriques fournissent des nombres récapitulatifs des différents états qu’un appareil ou Module peut signaler après l’application d’un contenu de configuration. Par exemple, vous pouvez créer une métrique pour les modifications de paramètres en attente, une métrique pour les erreurs et une métrique pour les modifications de paramètres réussies.

Chaque configuration peut comprendre jusqu’à cinq indicateurs de performance personnalisés. 

1. Entrez un nom pour **Nom de métrique**.

2. Entrez une requête pour **Critères de la métrique**.  La requête est basée sur les propriétés signalées du jumeau d’appareil.  La métrique représente le nombre de lignes retournées par la requête.

Par exemple :

```sql
SELECT deviceId FROM devices 
  WHERE properties.reported.chillerWaterSettings.status='pending'
```

Vous pouvez inclure une clause spécifiant que la configuration a été appliquée, par exemple : 

```sql
/* Include the double brackets. */
SELECT deviceId FROM devices 
  WHERE configurations.[[yourconfigname]].status='Applied'
```

Si vous créez un indicateur de performance pour créer un rapport sur les Modules configurés, sélectionnez `moduleId` dans `devices.modules`. Par exemple :

```sql
SELECT deviceId, moduleId FROM devices.modules
  WHERE properties.reported.lastDesiredStatus.code = 200
```

### <a name="target-devices"></a>Appareils cibles

Utilisez la propriété tags de vos jumeaux pour cibler des appareils ou Modules spécifiques qui doivent recevoir cette configuration. Vous pouvez également cibler des propriétés signalées pour le jumeau.

Les configurations d’appareil automatiques peuvent uniquement cibler des balises de jumeau d’appareil, et les configurations de Module automatiques peuvent uniquement cibler des balises de jumeau de Module. 

Étant donné que plusieurs configurations peuvent cibler le même appareil ou Module, chaque configuration doit avoir un numéro de priorité. En cas de conflit, la configuration avec la priorité la plus élevée prévaut. 

1. Entrez un entier positif pour la **Priorité** de la configuration. La valeur numérique la plus élevée est considérée comme la priorité la plus élevée. Si deux configurations ont le même numéro de priorité, celle qui a été créée le plus récemment prévaut. 

2. Entrez une **Condition cible** pour déterminer quels sont les appareils ou Modules ciblés par cette configuration. La condition est basée sur les balises de jumeau ou sur les propriétés signalées du jumeau et doit correspondre au format de l’expression. 

   Pour une configuration d’appareil automatique, vous pouvez spécifier uniquement la balise ou la propriété signalée à cibler. Par exemple, `tags.environment='test'` ou `properties.reported.chillerProperties.model='4000x'`. Vous pouvez spécifier `*` pour cibler tous les appareils. 
   
   Pour une configuration de Module automatique, utilisez une requête pour spécifier des balises ou des propriétés signalées à partir des Modules inscrits auprès d’IoT Hub. Par exemple, `from devices.modules where tags.environment='test'` ou `from devices.modules where properties.reported.chillerProperties.model='4000x'`. Le caractère générique ne peut pas être utilisé pour cibler tous les Modules. 

3. Sélectionnez **Suivant** pour passer à l’étape finale.

### <a name="review-configuration"></a>Passer en revue la configuration

Passez en revue les informations de votre configuration, puis sélectionnez **Envoyer**.

## <a name="monitor-a-configuration"></a>Surveiller une configuration

Pour afficher les détails d’une configuration et surveiller les appareils qui l’exécutent, effectuez les étapes suivantes :

1. Accédez à votre IoT Hub dans le [Portail Azure](https://portal.azure.com). 

2. Sélectionnez **Configuration de l’appareil IoT**.

3. Inspectez la liste des configurations. Pour chaque configuration, vous pouvez voir les détails suivants :

   * **ID** : nom de la configuration.

   * **Condition cible** : requête utilisée pour définir les appareils ou Modules ciblés.

   * **Priorité** : numéro de priorité de la configuration.

   * **Heure de création** : horodatage de création de la configuration. Cet horodatage sert à départager deux configurations ayant la même priorité. 

   * **Métriques système** : métriques qui sont calculées par IoT Hub et qui ne peuvent pas être personnalisées par les développeurs. « Ciblé » spécifie le nombre de jumeaux d’appareil remplissant la condition cible. « Appliqué » spécifie le nombre de jumeaux d’appareil qui ont été modifiés par la configuration ; cela peut inclure des modifications partielles dans le cas où une configuration distincte de priorité plus élevée a également apporté des modifications. 

   * **Indicateurs de performance personnalisés** : indicateurs de performance qui ont été spécifiés par le développeur sous forme de requêtes portant sur les propriétés signalées du jumeau.  Vous pouvez définir jusqu’à cinq métriques personnalisées par configuration. 
   
4. Sélectionnez la configuration que vous souhaitez surveiller.  

5. Inspectez les détails de la configuration. Vous pouvez utiliser les onglets pour afficher des détails spécifiques sur les appareils qui ont reçu la configuration.

   * **Condition cible** : appareils ou Modules qui remplissent la condition cible. 

   * **Métriques** : liste des métriques système et des métriques personnalisées.  Vous pouvez voir la liste des appareils ou Modules pris en compte pour chaque indicateur de performance en sélectionnant l’indicateur de performance dans la liste déroulante, puis en sélectionnant **Afficher les appareils** ou **Afficher les Modules**.

   * **Paramètres de jumeau d’appareil** ou **Paramètres de jumeau de Module** : paramètres de jumeau qui sont définis par la configuration. 

   * **Étiquettes de configuration** : paires clé-valeur utilisées pour décrire une configuration.  Les étiquettes n’ont aucun impact sur les fonctionnalités. 

## <a name="modify-a-configuration"></a>Modifier une configuration

Lorsque vous Modifiez une configuration, les Modifications sont répliquées immédiatement sur tous les appareils ou Modules ciblés. 

Si vous mettez à jour la condition cible, les mises à jour suivantes se produisent :

* Si un jumeau ne remplissait pas l’ancienne condition cible, mais qu’il remplit la nouvelle condition cible et que cette configuration a la priorité la plus élevée pour ce jumeau, cette configuration est appliquée. 

* Si un jumeau exécutant cette configuration ne remplit plus la condition cible, les paramètres de la configuration sont supprimés et le jumeau est Modifié par la configuration ayant la priorité la plus élevée suivante. 

* Si un jumeau exécutant cette configuration ne remplit plus la condition cible et ne remplit pas la condition cible des autres configurations, les paramètres de la configuration sont supprimés et aucune autre Modification n’est effectuée sur le jumeau. 

Pour modifier une configuration, effectuez les étapes suivantes : 

1. Accédez à votre IoT Hub dans le [Portail Azure](https://portal.azure.com). 

2. Sélectionnez **Configuration de l’appareil IoT**. 

3. Sélectionnez la configuration que vous souhaitez modifier. 

4. Mettez à jour les champs suivants : 

   * Condition cible 
   * Étiquettes 
   * Priority 
   * Mesures

4. Sélectionnez **Enregistrer**.

5. Suivez les étapes fournies dans [Surveiller une configuration](#monitor-a-configuration) pour observer le déploiement des modifications. 

## <a name="delete-a-configuration"></a>Supprimer une configuration

Quand vous supprimez une configuration, tous les jumeaux d’appareil prennent leur configuration suivante dans l’ordre de priorité. Si des jumeaux d’appareil ne remplissent la condition cible d’aucune autre configuration, aucun autre paramètre n’est appliqué. 

1. Accédez à votre IoT Hub dans le [Portail Azure](https://portal.azure.com). 

2. Sélectionnez **Configuration de l’appareil IoT**. 

3. Utilisez la case à cocher pour sélectionner la configuration à supprimer. 

4. Sélectionnez **Supprimer**.

5. Une invite vous demande de confirmer.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert comment configurer et superviser des appareils IoT à grande échelle. Suivez ces liens pour en savoir plus sur la gestion de Azure IoT Hub :

* [Gestion de vos identités d’appareil IoT Hub en bloc](iot-hub-bulk-identity-mgmt.md)
* [Surveiller votre hub IoT](monitor-iot-hub.md)

Pour explorer davantage les capacités de IoT Hub, consultez :

* [Guide du développeur d’IoT Hub](iot-hub-devguide.md)
* [Déploiement d’une IA sur des appareils de périmètre avec Azure IoT Edge](../iot-edge/quickstart-linux.md)

Pour savoir comment utiliser le service d’approvisionnement des appareils IoT Hub afin d’activer l’approvisionnement sans contact et juste-à-temps, consultez : 

* [Service Azure IoT Hub Device Provisioning](../iot-dps/index.yml)