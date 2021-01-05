---
title: Déployer des modules à grande échelle à partir du portail Azure – Azure IoT Edge
description: Utilisez le portail Azure afin de créer des déploiements automatiques pour les groupes d’appareils IoT Edge
keywords: ''
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 10/13/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 9d03b6f4a512c22564480405ec0f0e0c0e62a958
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92048421"
---
# <a name="deploy-iot-edge-modules-at-scale-using-the-azure-portal"></a>Déployer des modules IoT Edge à grande échelle à l’aide du portail Azure

Créez un **déploiement automatique IoT Edge** dans le portail Azure pour gérer les déploiements en cours sur plusieurs appareils à la fois. Les déploiements automatiques IoT Edge font partie de la fonctionnalité [Gestion automatique des appareils](../iot-hub/iot-hub-automatic-device-management.md) d’IoT Hub. Les déploiements sont des processus dynamiques qui vous permettent de déployer plusieurs modules sur plusieurs appareils, de suivre l’état et l’intégrité des modules, et d’apporter des modifications si nécessaire.

Pour plus d’informations, consultez [Comprendre les déploiements automatiques IoT Edge pour un seul ou de nombreux appareils](module-deployment-monitoring.md).

## <a name="identify-devices-using-tags"></a>Identifier les appareils à l’aide de balises

Avant de pouvoir créer un déploiement, vous devez être en mesure de spécifier les appareils concernés. Azure IoT Edge identifie les appareils à l’aide de **balises** dans le jumeau d’appareil. Chaque appareil peut avoir plusieurs balises que vous pouvez définir comme bon vous semble pour votre solution.

Par exemple, si vous gérez un campus de bâtiments intelligents, vous pouvez ajouter des étiquettes de localisation, de type de pièce et d’environnement à un appareil :

```json
"tags":{
  "location":{
    "building": "20",
    "floor": "2"
  },
  "roomtype": "conference",
  "environment": "prod"
}
```

Pour plus d’informations sur les étiquettes et les jumeaux d’appareils, consultez [Comprendre et utiliser les jumeaux d’appareil IoT Hub](../iot-hub/iot-hub-devguide-device-twins.md).

## <a name="create-a-deployment"></a>Créer un déploiement

IoT Edge propose deux types de déploiement automatique différents dont vous pouvez vous servir pour personnaliser votre scénario. Vous pouvez créer un *déploiement* standard, qui comprend des modules du runtime système ainsi que des modules et des routes supplémentaires. Chaque appareil ne peut appliquer qu’un seul déploiement. Vous pouvez aussi créer un *déploiement en couches*, qui comprend uniquement des modules et des routes personnalisés, et non le runtime système. Il est possible de combiner divers déploiements en couches sur un appareil, au-dessus d’un déploiement standard. Pour savoir comment les deux types de déploiement automatique fonctionnent ensemble, consultez [Comprendre les déploiements automatiques IoT Edge pour des appareils uniques ou à l’échelle](module-deployment-monitoring.md).

Les étapes nécessaires à la création d’un déploiement et d’un déploiement en couches sont très similaires. Les différences éventuelles sont indiquées dans les étapes suivantes.

1. Sur le [portail Azure](https://portal.azure.com), accédez à votre hub IoT.
1. Dans le menu du volet gauche, sélectionnez **IoT Edge** sous **Gestion automatique des appareils**.
1. Dans la barre supérieure, sélectionnez **Créer un déploiement** ou **Créer un déploiement en couches**.

La création d’un déploiement nécessite cinq étapes. Les sections suivantes les décrivent en détail.

>[!NOTE]
>La procédure décrite dans cet article reflète la dernière version de schéma du hub et de l’agent IoT Edge. La version de schéma 1.1 a été publiée avec IoT Edge version 1.0.10 ; elle fournit les fonctionnalités d’ordre de démarrage des modules et de hiérarchisation des routes.
>
>Si vous déployez sur un appareil exécutant la version 1.0.9 ou une version antérieure, modifiez les **paramètres du runtime** à l’étape **Modules** de l’Assistant pour utiliser la version 1.0 du schéma.

### <a name="step-1-name-and-label"></a>Étape 1 : Nom et étiquette

1. Donnez à votre déploiement un nom unique comportant au plus 128 lettres minuscules. Évitez les espaces et les caractères non valides suivants : `& ^ [ ] { } \ | " < > /`.
1. Vous pouvez ajouter des étiquettes sous forme de paires clé-valeur pour vous aider à suivre vos déploiements. Par exemple, **HostPlatform** et **Linux**, ou **Version** et **3.0.1**.
1. Sélectionnez **Suivant : Modules** pour passer à l’étape deux.

### <a name="step-2-modules"></a>Étape 2 : Modules

Vous pouvez ajouter jusqu’à 50 modules dans un déploiement. Si vous créez un déploiement sans module, il supprime tous les modules actuels des appareils cibles.

Dans les déploiements, vous pouvez gérer les paramètres de l’agent IoT Edge et des modules du hub IoT Edge. Sélectionnez **Paramètres du runtime** pour configurer les deux modules de runtime. Dans un déploiement en couches, les modules de runtime ne sont pas inclus et ne peuvent donc pas être configurés.

Pour ajouter du code personnalisé en tant que module, ou pour ajouter manuellement un module de service Azure, effectuez les étapes suivantes :

1. Dans la section **Paramètres de Container Registry** de la page, fournissez les informations d’identification permettant d’accéder à n’importe quels registres de conteneurs privés qui contiennent des images de module.
1. Dans la section **Modules IoT Edge** de la page, sélectionnez **Ajouter**.
1. Choisissez l’un des trois types de modules dans le menu déroulant :

   * **Module IoT Edge** : vous fournissez le nom du module et l’URI de l’image conteneur. Par exemple, l’URI de l’image de l’exemple de module SimulatedTemperatureSensor est `mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0`. Si l’image de module est stockée dans un registre de conteneurs privé, ajoutez les informations d’identification sur cette page pour accéder à l’image.
   * **Module Marketplace** : modules hébergés dans la Place de marché Azure. Certains modules de la Place de marché requièrent une configuration supplémentaire. passez donc en revue les détails du module dans la liste [Modules IoT Edge de la Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/category/internet-of-things?page=1&subcategories=iot-edge-modules).
   * **Module Azure Stream Analytics** : les modules générés à partir d’une charge de travail Azure Stream Analytics.

1. Si nécessaire, répétez les étapes 2 et 3 pour ajouter d’autres modules à votre déploiement.

Après avoir ajouté un module à un déploiement, vous pouvez sélectionner son nom pour ouvrir la page **Mettre à jour le module IoT Edge**. Dans cette page, vous pouvez modifier les paramètres du module, les variables d’environnement, les options de création, l’ordre de démarrage et le jumeau du module. Si vous avez ajouté un module à partir de la Place de marché, certains de ces paramètres sont peut-être déjà renseignés. Pour en savoir plus sur les paramètres de modules disponibles, consultez l’article [Configuration et gestion des modules](module-composition.md#module-configuration-and-management).

Si vous créez un déploiement en couches, il se peut que vous configuriez un module qui existe dans d’autres déploiements ciblant les mêmes appareils. Pour mettre à jour le jumeau de module sans remplacer d’autres versions, ouvrez l’onglet **Paramètres de jumeau de module**. Créez une **Propriété de jumeau de module** avec un nom unique pour une sous-section des propriétés souhaitées du jumeau de module, par exemple `properties.desired.settings`. Si vous définissez des propriétés uniquement dans le champ `properties.desired`, les propriétés souhaitées pour le module défini dans des déploiements de moindre priorité seront remplacées.

![Définir la propriété de jumeau de module pour un déploiement en couches](./media/how-to-deploy-monitor/module-twin-property.png)

Pour plus d’informations sur la configuration de jumeau de module dans les déploiements en couches, consultez [Déploiement en couches.](module-deployment-monitoring.md#layered-deployment)

Une fois que vous avez configuré tous les modules pour un déploiement, sélectionnez **Suivant : Routes** pour passer à l’étape trois.

### <a name="step-3-routes"></a>Étape 3 : Itinéraires

Sous l’onglet **Routes**, vous définissez la manière dont les messages sont transmis entre les modules et le hub IoT. Les messages sont construits à l’aide de paires nom/valeur.

Par exemple, une route qui a le nom **route** et la valeur **FROM /messages/\* INTO $upstream** prendrait tous les messages générés par les modules et les enverrait à votre hub IoT.  

Les paramètres **Priorité** et **Durée de vie** sont des paramètres facultatifs que vous pouvez inclure dans une définition de route. Le paramètre Priorité vous permet de choisir les routes dont les messages doivent être traités en premier ou au contraire, les routes à traiter en dernier. La priorité est déterminée en définissant un nombre entre 0 et 9, 0 étant la priorité la plus haute. Le paramètre Durée de vie vous permet de déclarer la durée de conservation des messages dans cette route avant qu’ils soient traités ou supprimés de la file d’attente.

Pour plus d’informations sur la création de routes, consultez [Déclarer des routes](module-composition.md#declare-routes).

Sélectionnez **Suivant : Métriques**.

### <a name="step-4-metrics"></a>Étape 4 : Mesures

Les métriques fournissent des nombres récapitulatifs des différents états qu’un appareil peut signaler après l’application d’un contenu de configuration.

1. Entrez un nom pour **Nom de métrique**.

1. Entrez une requête pour **Critères de la métrique**. La requête est basée sur les [propriétés signalées](module-edgeagent-edgehub.md#edgehub-reported-properties) du jumeau de module du hub IoT Edge. La métrique représente le nombre de lignes retournées par la requête.

   Par exemple :

   ```sql
   SELECT deviceId FROM devices
     WHERE properties.reported.lastDesiredStatus.code = 200
   ```

Sélectionnez **Suivant : Appareils cibles**.

### <a name="step-5-target-devices"></a>Étape 5 : Appareils cibles

Utilisez la propriété tags à partir de vos appareils pour cibler des appareils spécifiques qui doivent recevoir ce déploiement.

Étant donné que plusieurs déploiements peuvent cibler le même appareil, vous devez donner à chaque déploiement un numéro de priorité. En cas de conflit, le déploiement avec la priorité la plus élevée prévaut (plus la valeur est grande, plus la priorité est élevée). Si deux déploiements ont le même numéro de priorité, celui qui a été créé le plus récemment prévaut.

Si plusieurs déploiements ciblent un même appareil, seul celui qui a la priorité la plus élevée est appliqué. Si plusieurs déploiements en couches ciblent un même appareil, ils sont tous appliqués. En revanche, s’il existe des propriétés en double avec par exemple la présence de deux routes de même nom, celle du déploiement en couches de priorité supérieure remplace le reste.

Pour s’appliquer, un déploiement en couches ciblant un appareil doit avoir une priorité plus élevée que celle du déploiement de base.

1. Entrez un entier positif pour la **Priorité** du déploiement.
1. Entrez une **Condition cible** pour déterminer quels sont les appareils ciblés par ce déploiement.  La condition est basée sur les balises de jumeau d’appareil ou sur les propriétés signalées du jumeau d’appareil et doit correspondre au format de l’expression.  Par exemple, `tags.environment='test'` ou `properties.reported.devicemodel='4000x'`.

Sélectionnez **Suivant : Vérifier + créer** pour passer à l’étape finale.

### <a name="step-6-review-and-create"></a>Étape 6 : Examiner et créer

Vérifiez les informations de votre déploiement, puis sélectionnez **Créer**.

Pour superviser votre déploiement, consultez [Superviser les déploiements IoT Edge](how-to-monitor-iot-edge-deployments.md).

## <a name="modify-a-deployment"></a>Modifier un déploiement

Quand vous modifiez un déploiement, les modifications sont répliquées immédiatement sur tous les appareils ciblés. Vous pouvez modifier les paramètres et fonctionnalités suivants pour un déploiement existant :

* Conditions cibles
* Mesures personnalisées
* Étiquettes
* Balises
* Propriétés souhaitées

### <a name="modify-target-conditions-custom-metrics-and-labels"></a>Modifier les conditions cibles, les métriques personnalisées et les étiquettes

1. Dans votre hub IoT, sélectionnez **IoT Edge** dans le menu du volet gauche.
1. Sélectionnez l’onglet **Déploiements IoT Edge**, puis sélectionnez le déploiement que vous souhaitez configurer.
1. Sélectionnez l’onglet **Condition cible**. Changez la **Condition cible** pour cibler les appareils prévus. Vous pouvez également ajuster la **Priorité**.  Sélectionnez **Enregistrer**.

    Si vous mettez à jour la condition cible, les mises à jour suivantes se produisent :

    * Si un appareil ne remplissait pas l’ancienne condition cible, mais qu’il remplit la nouvelle condition cible et que ce déploiement a la priorité la plus élevée pour cet appareil, ce déploiement est appliqué à l’appareil.
    * Si un appareil exécutant actuellement ce déploiement ne remplit plus la condition cible, il désinstalle ce déploiement et prend le déploiement suivant dans l’ordre de priorité.
    * Si un appareil exécutant actuellement ce déploiement ne remplit plus la condition cible et ne remplit la condition cible d’aucun autre déploiement, aucun changement ne se produit sur l’appareil. L’appareil continue à exécuter ses modules actuels dans leur état actuel, mais il n’est plus géré dans le cadre de ce déploiement. Dès qu’il remplit la condition cible d’un autre déploiement, il désinstalle ce déploiement et prend le nouveau.

1. Sélectionnez l’onglet **Métriques**, puis cliquez sur le bouton **Modifier les métriques**. Ajoutez ou modifiez des métriques personnalisées, en utilisant l’exemple de syntaxe comme guide. Sélectionnez **Enregistrer**.

    ![Modifier des métriques personnalisées dans un déploiement](./media/how-to-deploy-monitor/metric-list.png)

1. Sélectionnez l’onglet **Étiquettes** et effectuez les modifications souhaitées, puis sélectionnez **Enregistrer**.

## <a name="delete-a-deployment"></a>Supprimer un déploiement

Quand vous supprimez un déploiement, les appareils déployés adoptent leur déploiement suivant dans l’ordre de priorité. Si vos appareils ne remplissent la condition cible d’aucun autre déploiement, les modules ne sont pas supprimés lors de la suppression du déploiement.

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre hub IoT.
1. Sélectionnez **IoT Edge**.
1. Sélectionnez l’onglet **Déploiements IoT Edge**.

   ![Afficher les déploiements IoT Edge](./media/how-to-deploy-monitor/iot-edge-deployments.png)

1. Utilisez la case à cocher pour sélectionner le déploiement à supprimer.
1. Sélectionnez **Supprimer**.
1. Un message vous informe que cette action va supprimer ce déploiement et restaurer tous les appareils à leur état précédent.  Un déploiement avec une priorité inférieure sera utilisé.  Si aucun autre déploiement n’est ciblé, aucun module n’est supprimé. Si vous souhaitez supprimer tous les modules de votre appareil, créez un déploiement sans aucun module et déployez-le sur l’appareil en question.  Cliquez sur **Oui** pour continuer.

## <a name="next-steps"></a>Étapes suivantes

Découvrez le [déploiement des modules sur des appareils IoT Edge](module-deployment-monitoring.md).