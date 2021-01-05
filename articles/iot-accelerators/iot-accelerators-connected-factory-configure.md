---
title: Configurer la topologie d’usine connectée - Azure | Microsoft Docs
description: Cet article explique comment configurer l’accélérateur de solution Usine connectée, y compris sa topologie.
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 12/12/2017
ms.author: dobett
ms.openlocfilehash: e015c6761b920ef37af2bbfd67ced5fc3218d532
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002236"
---
# <a name="configure-the-connected-factory-solution-accelerator"></a>Configurer l’accélérateur de solution Usine connectée

> [!IMPORTANT]
> Pendant la mise à jour de cet article, consultez [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/) pour obtenir le contenu le plus récent.

L’accélérateur de solution Usine connectée simule le tableau de bord d’une société fictive nommée Contoso. Cette société possède de nombreuses usines implantées dans le monde entier.

Cet article utilise Contoso comme exemple pour décrire comment configurer la topologie d’une solution Usine connectée.

## <a name="simulated-factories-configuration"></a>Simulation de configuration d’usines

Chaque usine de Contoso possède des lignes de production se composant de trois postes. Chaque station est un serveur OPC UA réel ayant un rôle spécifique :

* Poste d’assemblage
* Poste de test
* Poste d’emballage

Ces serveurs OPC UA comprennent des nœuds OPC UA. [OPC Publisher](overview-opc-publisher.md) envoie les valeurs de ces nœuds à Usine connectée. notamment :

* L’état opérationnel actuel, tel que la consommation d’énergie actuelle.
* Des informations relatives à la production, telles que le nombre de produits fabriqués.

Vous pouvez utiliser le tableau de bord pour explorer la topologie d’usine connectée de Contoso à partir d’une vue globale vers une vue détaillée de chaque poste. Le tableau de bord d’Usine connectée permet d’effectuer les actions suivantes :

* Visualiser les chiffres d’OEE et de KPI pour chaque couche de la topologie.
* Visualiser les valeurs actuelles des nœuds OPC UA des postes.
* Agréger les chiffres d’OEE et de KPI à partir du niveau de poste jusqu’au niveau global.
* Visualiser les alertes et les actions à effectuer si les valeurs atteignent des seuils spécifiques.

## <a name="connected-factory-topology"></a>Topologie d’Usine connectée

La topologie des usines, des lignes de production et des postes est hiérarchique :

* Le niveau global comporte des nœuds d’usine enfants.
* Les usines comportent des nœuds de ligne de production enfants.
* Les lignes de production comportent des nœuds de poste enfants.
* Les postes (serveurs OPC UA) comportent des nœuds OPC UA enfants.

Chaque nœud de la topologie possède un ensemble commun de propriétés qui définissent :

* Un identificateur unique pour le nœud de topologie.
* Un nom.
* Une description.
* Une image.
* Les enfants du nœud de topologie.
* Des valeurs minimales, maximales et cibles pour les chiffres d’OEE et de KPI et les actions d’alerte à exécuter.

## <a name="topology-configuration-file"></a>Fichier de configuration de la topologie

Pour configurer les propriétés répertoriées dans la section précédente, la solution Usine connectée utilise un fichier de configuration appelé [ContosoTopologyDescription.json](https://github.com/Azure/azure-iot-connected-factory/blob/master/WebApp/Contoso/Topology/ContosoTopologyDescription.json).

Ce fichier se trouve dans le code source de la solution situé dans le dossier `WebApp/Contoso/Topology`.

L’extrait de code suivant montre un plan du fichier de configuration `ContosoTopologyDescription.json` :

```json
{
  <global_configuration>,
  "Factories": [
    <factory_configuration>,
    "ProductionLines": [
      <production_line_configuration>,
      "Stations": [
        <station_configuration>,
        <more station_configurations>
      ],
      <more production_line_configurations>
    ]
    <more factory_configurations>
  ]
}
```

Les propriétés communes de `<global_configuration>`, `<factory_configuration>`, `<production_line_configuration>`, et `<station_configuration>` sont :

* **Name** (de type chaîne)

  Définit un nom descriptif. Il doit s’agir d’un seul mot pour le nœud de topologie à afficher dans le tableau de bord.

* **Description** (de type chaîne)

  Décrit le nœud de topologie de manière plus détaillée.

* **Image** (de type chaîne)

  Chemin d’accès à une image dans la solution de l’application web à afficher lorsque les informations sur le nœud de la topologie s’affichent dans le tableau de bord.

* **OeeOverall**, **OeePerformance**, **OeeAvailability**, **OeeQuality**, **Kpi1**, **Kpi2** (de type `<performance_definition>`)

  Ces propriétés définissent les valeurs minimales, maximales et cibles du chiffre relatif au fonctionnement utilisé pour générer des alertes. Ces propriétés définissent également les actions à exécuter si une alerte est détectée.

Les éléments `<factory_configuration>` et `<production_line_configuration>` possèdent une propriété :

* **Guid** (de type chaîne)

  Identifie le nœud de topologie de manière unique.

`<factory_configuration>` a une propriété :

* **Location** (de type `<location_definition>`)

  Spécifie l’emplacement de l’usine.

`<station_configuration>` possède les propriétés :

* **OpcUri** (de type chaîne)

  Cette propriété doit être définie sur l’URI d’application OPC UA du serveur OPC UA.
  Dans la mesure où elle doit être unique à l’échelle internationale conformément à la spécification relative à OPC UA, cette propriété est utilisée pour identifier le nœud de topologie du poste.

* **OpcNodes**, qui sont un ensemble de nœuds OPC UA (de type `<opc_node_description>`)

`<location_definition>` possède les propriétés :

* **City** (de type chaîne)

  Nom de la ville la plus proche de l’emplacement

* **Country** (de type chaîne)

  Pays de l'emplacement.

* **Latitude** (de type double)

  Latitude de l'emplacement.

* **Longitude** (de type double)

  Longitude de l'emplacement.

`<performance_definition>` possède les propriétés :

* **Minimum** (de type double)

  Seuil inférieur que la valeur peut atteindre. Si la valeur actuelle est inférieure à ce seuil, une alerte est générée.

* **Target** (de type double)

  Valeur cible idéale.

* **Maximum** (de type double)

  Seuil supérieur que la valeur peut atteindre. Si la valeur actuelle est supérieure à ce seuil, une alerte est générée.

* **MinimumAlertActions** (de type `<alert_action>`)

  Définit l’ensemble des actions pouvant être prises en tant que réponse à une alerte minimale.

* **MaximumAlertActions** (de type `<alert_action>`)

  Définit l’ensemble des actions pouvant être prises en tant que réponse à une alerte maximale.

`<alert_action`> possède les propriétés :

* **Type** (de type chaîne)

  Type de l’action d’alerte. Les types suivants sont connus :

  * **AcknowledgeAlert** : l’état de l’alerte doit passer à confirmé.
  * **CloseAlert** : toutes les alertes plus anciennes du même type ne doivent plus s’afficher dans le tableau de bord.
  * **CallOpcMethod** : une méthode OPC UA doit être appelée.
  * **OpenWebPage** : une fenêtre de navigateur doit être ouverte et afficher des informations contextuelles supplémentaires.

* **Description** (de type chaîne)

  Description de l’action affichée dans le tableau de bord.

* **Parameter** (de type chaîne)

  Paramètres requis pour exécuter l’action. La valeur dépend du type d’action.

  * **AcknowledgeAlert** : aucun paramètre requis.
  * **CloseAlert** : aucun paramètre requis.
  * **CallOpcMethod**: informations et paramètres de nœud de la méthode OPC UA à appeler au format « NodeId du nœud parent, NodeId de la méthode à appeler, URI du serveur OPC UA ».
  * **OpenWebPage** : URL à afficher dans la fenêtre du navigateur.

`<opc_node_description>` contient des informations sur les nœuds de OPC UA d’un poste (serveur OPC UA). Les nœuds qui ne représentent aucun nœud OPC UA existant, mais qui sont utilisés comme stockage dans la logique de calcul d’Usine connectée, sont également valides. Il possède les propriétés suivantes :

* **NodeId** (de type chaîne)

  Adresse du nœud OPC UA de l’espace d’adressage du poste (du serveur OPC UA). En ce qui concerne NodeId, la syntaxe doit être spécifiée dans la spécification OPC UA.

* **SymbolicName** (de type chaîne)

  Nom à afficher dans le tableau de bord lorsque la valeur de ce nœud OPC UA est affichée.

* **Relevance** (tableau de type chaîne)

  Indique pour quel calcul de OEE et de KPI la valeur du nœud OPC UA s’applique. Chaque élément du tableau peut être l’une des valeurs suivantes :

  * **OeeAvailability_Running** : la valeur est utile pour le calcul de la disponibilité OEE.
  * **OeeAvailability_Fault** : la valeur est utile pour le calcul de la disponibilité OEE.
  * **OeePerformance_Ideal** : la valeur est utile pour le calcul des performances OEE et est généralement une valeur constante.
  * **OeePerformance_Actual** : la valeur est utile pour le calcul des performances OEE.
  * **OeeQuality_Good** : la valeur est utile pour le calcul de la qualité OEE.
  * **OeeQuality_Bad** : la valeur est utile pour le calcul de la qualité OEE.
  * **Kpi1** : la valeur est utile pour le calcul de KPI1.
  * **Kpi2** : la valeur est utile pour le calcul de KPI2.

* **OpCode** (de type chaîne)

  Indique comment la valeur du nœud OPC UA est traitée dans les requêtes Time Series Insight et les calculs d’OEE/de KPI. Chaque requête Time Series Insight cible un intervalle de temps spécifique, qui est un paramètre de la requête et fournit un résultat. La propriété OpCode contrôle la manière dont le résultat est calculé et peut prendre l’une des valeurs suivantes :

  * **Diff** : différence entre la dernière et la première valeur dans l’intervalle de temps.
  * **Avg** : moyenne de toutes les valeurs dans l’intervalle de temps.
  * **Sum** : somme de toutes les valeurs dans l’intervalle de temps.
  * **Last** : non utilisé actuellement.
  * **Count** : nombre de valeurs à créer dans l’intervalle de temps.
  * **Max**: valeur maximale dans l’intervalle de temps.
  * **Min**: valeur minimale dans l’intervalle de temps.
  * **Const** : le résultat est la valeur spécifiée par la propriété ConstValue.
  * **SubMaxMin** : différence entre la valeur maximale et minimale.
  * **Timespan** : intervalle de temps.

* **Units** (de type chaîne)

  Définit une unité de la valeur pour l’affichage dans le tableau de bord.

* **Visible** (de type booléen)

  Contrôle si la valeur doit être indiquée dans le tableau de bord.

* **ConstValue** (de type double)

  Si **OpCode** est **Const**, cette propriété est la valeur du nœud.

* **Minimum** (de type double)

  Si la valeur actuelle se situe en dessous de cette valeur, une alerte minimale est générée.

* **Maximum** (de type double)

  Si la valeur actuelle se situe au-dessus de cette valeur, une alerte maximale est générée.

* **MinimumAlertActions** (de type `<alert_action>`)

  Définit l’ensemble des actions pouvant être prises en tant que réponse à une alerte minimale.

* **MaximumAlertActions** (de type `<alert_action>`)

  Définit l’ensemble des actions pouvant être prises en tant que réponse à une alerte maximale.

Au niveau du poste, vous voyez également des objets **Simulation**. Ces objets sont uniquement utilisés pour configurer la simulation d’Usine connectée et ne doivent pas servir à configurer une topologie réelle.

## <a name="how-the-configuration-data-is-used-at-runtime"></a>Comment les données de configuration sont utilisées lors de l’exécution

Toutes les propriétés utilisées dans le fichier de configuration peuvent être regroupées en différentes catégories en fonction de la façon dont elles sont utilisées. Ces catégories sont les suivantes :

### <a name="visual-appearance"></a>Apparence visuelle

Les propriétés de cette catégorie définissent l’apparence visuelle du tableau de bord Usine connectée. Voici quelques exemples :

* Nom
* Description
* Image
* Emplacement
* Units
* Visible

### <a name="internal-topology-tree-addressing"></a>Adressage d’arborescence de la topologie interne

L’application web gère un dictionnaire de données interne contenant des informations sur tous les nœuds de la topologie. Les propriétés **Guid** et **OpcUri** sont utilisées comme des clés pour accéder à ce dictionnaire et doivent être uniques.

### <a name="oeekpi-computation"></a>Calcul de l’OEE/ du KPI

Les valeurs de l’OEE/du KPI relatives à la simulation d’Usine connectée sont paramétrées par :

* Les valeurs du nœud OPC UA à inclure dans le calcul.
* Comment la valeur est calculée à partir des valeurs de télémétrie.

L’Usine connectée utilise les formules OEE publiées par [http://www.oeefoundation.org](http://www.oeefoundation.org).

Les objets du nœud OPC UA des postes activent le balisage pour une utilisation dans le calcul de l’OEE/du KPI. La propriété **Relevance** indique pour quelle valeur d’OEE/de KPI la valeur du nœud OPC UA doit être utilisée. La propriété **OpCode** définit la manière dont la valeur est incluse dans le calcul.

### <a name="alert-handling"></a>Gestion des alertes

L’Usine connectée prend en charge un mécanisme de génération d’alertes basé sur un seuil minimal/maximal simple. Il existe un certain nombre d’actions prédéfinies, que vous pouvez configurer en réponse à ces alertes. Les propriétés suivantes contrôlent ce mécanisme :

* Maximale
* Minimum
* MaximumAlertActions
* MinimumAlertActions

## <a name="correlating-to-telemetry-data"></a>Corrélation avec les données de télémétrie

Pour certaines opérations, telles que la visualisation de la dernière valeur ou la création de requêtes Time Series Insight, l’application web a besoin d’un schéma d’adressage pour les données de télémétrie réceptionné. Les données de télémétrie envoyées à l’Usine connectée doivent également être stockées dans des structures de données internes. Les deux propriétés permettant ces opérations se situent au niveau du nœud OPC UA et de poste (serveur OPC UA) :

* **OpcUri**

  Identifie le serveur OPC UA (unique au niveau global) duquel provient les données de télémétrie. Dans les messages ingérés, cette propriété est envoyée en tant que **ApplicationUri**.

* **NodeId**

  Identifie la valeur du nœud dans le serveur OPC UA. Le format de la propriété doit être tel que défini dans la spécification OPC UA. Dans les messages ingérés, cette propriété est envoyée en tant que **NodeId**.

Pour plus d’informations sur la façon dont les données de télémétrie sont intégrées à l’Usine connectée, consultez [Définition d’OPC Publisher](overview-opc-publisher.md).

## <a name="example-how-kpi1-is-calculated"></a>Exemple : mode de calcul de KPI1

La configuration du fichier `ContosoTopologyDescription.json` contrôle la façon dont les valeurs de l’OEE/du KPI sont calculées. L’exemple suivant montre comment les propriétés de ce fichier contrôlent le calcul de KPI1.

Dans l’Usine connectée, KPI1 est utilisée pour mesurer le nombre de produits fabriqués au cours de la dernière heure. Chaque poste (serveur OPC UA) de la simulation d’Usine connectée fournit un nœud OPC UA (`NodeId: "ns=2;i=385"`), qui fournit les données de télémétrie permettant de calculer ce KPI.

La configuration de ce nœud OPC UA ressemble à l’extrait de code suivant :

```json
{
  "NodeId": "ns=2;i=385",
  "SymbolicName": "NumberOfManufacturedProducts",
  "Relevance": [ "Kpi1", "OeeQuality_Good" ],
  "OpCode": "SubMaxMin"
},
```

Cette configuration permet d’interroger les valeurs de télémétrie de ce nœud à l’aide de Time Series Insights. La requête Time Series Insights extrait :

* Le nombre de valeurs.
* La valeur minimale.
* La valeur minimale.
* La moyenne de toutes les valeurs.
* La somme de toutes les valeurs pour toutes les paires **OpcUri** (**ApplicationUri**), **NodeId** uniques d’un intervalle de temps donné.

L’une des caractéristiques de la valeur du nœud **NumberOfManufactureredProducts** est qu’elle ne peut qu’augmenter. Pour calculer le nombre de produits fabriqués dans l’intervalle de temps, l’Usine connectée utilise la propriété **OpCode** **SubMaxMin**. Le calcul récupère la valeur minimale au début de l’intervalle de temps et la valeur maximale au terme de celui-ci.

La propriété **OpCode** de la configuration configure la logique de calcul permettant de calculer le résultat de la différence entre la valeur minimale et la valeur maximale. Ces résultats sont ensuite cumulés de manière à remonter jusqu’au niveau (global) racine et affichés dans le tableau de bord.

## <a name="next-steps"></a>Étapes suivantes

Une étape suivante suggérée vise à savoir comment [personnaliser la solution Usine connectée](iot-accelerators-connected-factory-customize.md).
