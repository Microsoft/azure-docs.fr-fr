---
title: Tarifs et modèles de facturation
description: Vue d’ensemble du fonctionnement du modèle de facturation et de tarification pour Azure Logic Apps
services: logic-apps
ms.suite: integration
author: jonfancey
ms.author: jonfan
ms.reviewer: estfan, logicappspm
ms.topic: conceptual
ms.date: 10/29/2020
ms.openlocfilehash: 486930776b4b4b6d852102be723ac1047ebd5e0a
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93098482"
---
# <a name="pricing-model-for-azure-logic-apps"></a>Modèle de tarif pour Azure Logic Apps

[Azure Logic Apps](../logic-apps/logic-apps-overview.md) vous aide à créer et à exécuter des flux de travail d’intégration automatisés et pouvant être mis à l’échelle dans le cloud. Cet article décrit comment fonctionnent la tarification et la facturation pour Azure Logic Apps. Pour connaître la tarification, consultez [Tarification de Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps).

<a name="consumption-pricing"></a>

## <a name="consumption-pricing-model"></a>Modèle de tarification de la consommation

Pour les nouvelles applications logiques qui s’exécutent dans le service Azure Logic Apps multilocataire public ou « global », vous payez uniquement ce que vous utilisez. Ces applications logiques utilisent un modèle tarifaire et un plan basés sur la consommation. Dans votre application logique, chaque étape est une action et Azure Logic Apps mesure toutes les actions qui s’exécutent dans votre application logique.

Ces actions comprennent par exemple :

* Des [déclencheurs](#triggers), qui sont des actions spéciales. Toutes les applications logiques nécessitent un déclencheur comme première étape.

* [Des actions « intégrées » ou natives](../connectors/apis-list.md#built-in) telles que HTTP, les appels à Azure Functions et la gestion des API etc.

* Des appels aux [connecteurs managés](../connectors/apis-list.md#managed-connectors) tels qu’Outlook 365, Dropbox etc.

* Des [actions de workflows de contrôle](../connectors/apis-list.md#control-workflow), telles que des boucles, des instructions conditionnelles, etc.

Les [connecteurs standard](../connectors/apis-list.md#managed-connectors) sont facturés au [prix d’un connecteur standard](https://azure.microsoft.com/pricing/details/logic-apps). Les [connecteurs d’entreprise](../connectors/apis-list.md#managed-connectors) généralement disponibles sont facturés au [tarif des connecteurs d’entreprise](https://azure.microsoft.com/pricing/details/logic-apps), tandis que les connecteurs d’entreprise en préversion publique sont facturés au [prix d’un connecteur standard](https://azure.microsoft.com/pricing/details/logic-apps).

En savoir sur le fonctionnement de la facturation au niveau des [déclencheurs](#triggers) et des [actions](#actions). Ou, pour plus d’informations sur les limites, consultez [Limites et configuration pour Azure Logic Apps](logic-apps-limits-and-config.md).

<a name="fixed-pricing"></a>

## <a name="fixed-pricing-model"></a>Modèle de tarif fixe

Un [*environnement de service d’intégration* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) fournit un moyen isolé de créer et d’exécuter des applications logiques pouvant accéder aux ressources dans un réseau virtuel Azure. Les applications logiques qui s’exécutent dans un ISE n’entraînent aucun coût en termes de conservation des données. Lorsque vous créez un environnement ISE, et uniquement lors de la création, vous pouvez choisir un [niveau ISE ou une référence SKU](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level), dont les [tarifs](https://azure.microsoft.com/pricing/details/logic-apps) sont différents :

* ISE **Premium**  : Cette unité de base de référence SKU dispose d’une capacité fixe, mais si vous avez besoin de davantage de débit, vous pouvez [ajouter des unités d’échelle](../logic-apps/ise-manage-integration-service-environment.md#add-capacity), pendant ou après la création de l’environnement ISE. Pour connaître les limites d’ISE, consultez [Limites et configuration pour Azure Logic Apps](logic-apps-limits-and-config.md#integration-service-environment-ise).

* ISE **Developer**  : Cette référence SKU n’offre aucune possibilité de montée en puissance, aucun contrat de niveau de service (SLA) et aucune limite publiée. Utilisez cette référence SKU uniquement à des fins d’expérimentation, de développement et de test, pas pour la production ou les tests de performances.

En ce qui concerne les applications logiques créées et exécutées dans un environnement ISE, vous payez un [prix fixe](https://azure.microsoft.com/pricing/details/logic-apps) (et non un tarif à l’utilisation) pour ces fonctionnalités :

* Déclencheurs et actions [intégrés](../connectors/apis-list.md#built-in)

  Au sein d’un ISE, les déclencheurs et actions intégrés présentent l’étiquette **Core** et s’exécutent dans le même ISE que vos applications logiques.

* Connecteurs [Standard](../connectors/apis-list.md#managed-connectors) et connecteurs [Entreprise](../connectors/apis-list.md#enterprise-connectors), vous permettant d’avoir autant de connexions Enterprise que vous le souhaitez

   Les connecteurs standard et entreprise qui présentent l’étiquette **ISE** s’exécutent dans le même ISE que vos applications logiques. Les connecteurs qui ne portent pas l’étiquette ISE s’exécutent dans le service Logic Apps multilocataire public ou « global ». Le tarif fixe s’applique également aux connecteurs s’exécutant dans le service multilocataire et utilisés avec des applications logiques qui s’exécutent dans un environnement ISE.

* Utilisation du [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) sans coût supplémentaire, en fonction de votre [référence SKU ISE](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level) :

  * Référence (SKU) de l’ISE **Premium**  : Un compte d’intégration unique de [niveau standard](../logic-apps/logic-apps-limits-and-config.md#artifact-number-limits)

  * Référence (SKU) de l’ISE **Developer**  : Un compte d’intégration unique de [niveau Gratuit](../logic-apps/logic-apps-limits-and-config.md#artifact-number-limits)

  Chaque référence SKU ISE est limitée à 5 comptes d’intégration au total. Contre un supplément, vous pouvez avoir plus de comptes d’intégration en fonction de votre référence SKU ISE :

  * Référence (SKU) de l’ISE **Premium**  : jusqu’à quatre comptes standard supplémentaires. Aucun compte gratuit ou de base.

  * Référence (SKU) de l’ISE **Developer**  : jusqu’à 4 comptes standard supplémentaires ou jusqu’à 5 comptes standard au total. Aucun compte de base.

  Pour plus d’informations sur les limites des comptes d’intégration, consultez [Limites et configuration pour Azure Logic Apps](../logic-apps/logic-apps-limits-and-config.md#integration-account-limits). Vous trouverez plus d’informations sur les [niveaux de comptes d’intégration et leur modèle de tarification](#integration-accounts) plus loin dans cette rubrique.

<a name="connectors"></a>

## <a name="connectors"></a>Connecteurs

Les connecteurs Azure Logic Apps aident votre application logique à accéder aux applications, services et systèmes localement ou dans le cloud en fournissant des [déclencheurs](#triggers), des [actions](#actions), ou les deux. Les connecteurs sont classés en tant que connecteurs standard ou entreprise. Pour une vue d’ensemble de ces connecteurs, consultez la [liste des connecteurs Azure Logic Apps](../connectors/apis-list.md). Si aucun connecteur prédéfini n’est disponible pour les API REST que vous voulez utiliser dans vos applications logiques, vous pouvez créer des [connecteurs personnalisés](/connectors/custom-connectors), qui ne sont que des wrappers autour de ces API REST. Les connecteurs personnalisés sont facturés comme des connecteurs standard. Les sections offrent plus d’informations sur la facturation des déclencheurs et des actions.

<a name="triggers"></a>

## <a name="triggers"></a>Déclencheurs

Un déclencheur est toujours la première étape du workflow d’une application logique. Il s’agit d’une action spéciale qui crée et exécute une instance d’application logique lorsque des critères spécifiques sont remplis ou qu’un événement spécifique se produit. Les déclencheurs agissent de différentes façons, ce qui affecte la façon dont l’application logique est mesurée. Voici les différents types de déclencheurs qui existent dans Azure Logic Apps :

* **Déclencheur de périodicité** : Vous pouvez utiliser ce déclencheur générique, qui n’est spécifique à aucun service ou système, pour démarrer un workflow d’application logique et créer une instance d’application logique qui s’exécute en fonction de l’intervalle de périodicité que vous avez configuré dans le déclencheur. Par exemple, vous pouvez configurer un déclencheur de périodicité qui s’exécute tous les trois jours ou d’après un calendrier plus complexe.

* **Déclencheur d’interrogation** : Vous pouvez utiliser ce déclencheur de récurrence plus spécialisé, qui est généralement associé au connecteur géré pour un service ou un système spécifique, pour rechercher des événements ou des messages qui répondent aux critères de création et d’exécution d’une instance d’application logique en fonction de l’intervalle de périodicité que vous avez configuré dans le déclencheur. Même quand aucune instance d’application logique n’est créée, par exemple, lorsque les déclencheurs sont ignorés, le service Logic Apps mesure chaque requête d’interrogation en tant qu’exécution. Pour spécifier l’intervalle d’interrogation, configurez le déclencheur par le biais du concepteur d’application logique.

  [!INCLUDE [logic-apps-polling-trigger-non-standard-metering](../../includes/logic-apps-polling-trigger-non-standard-metering.md)]

* **Déclencheur de webhook** : Au lieu d’utiliser un déclencheur d’interrogation, vous pouvez utiliser un déclencheur de webhook pour attendre que le client envoie une requête à votre application logique à une URL de point de terminaison spécifique. Chaque requête envoyée au point de terminaison du webhook est considérée comme une exécution d’action. Par exemple, les déclencheurs Requête et Webhook HTTP sont tous deux des déclencheurs de webhook génériques. Certains connecteurs pour les services ou les systèmes possèdent également des déclencheurs de webhook.

<a name="actions"></a>

## <a name="actions"></a>Actions

Azure Logic Apps mesure les actions « intégrées » (telles que HTTP) en tant qu’actions natives. Par exemple, les actions intégrées incluent les appels HTTP, les appels à partir d’Azure Functions ou de Gestion des API et les étapes de flux de contrôle telles que les conditions, les boucles et les instructions switch. Chaque action a son propre type. Par exemple, les actions qui appellent des [connecteurs](/connectors) ont le type « ApiConnection ». Ces connecteurs sont classés en connecteurs standard ou d’entreprise, et sont mesurés en fonction de leur [tarification](https://azure.microsoft.com/pricing/details/logic-apps) respective. Les connecteurs d’entreprise en *préversion* sont facturés comme connecteurs standard.

Azure Logic Apps mesure toutes les actions ayant réussi ou non en tant qu’exécutions. Cependant, Logic Apps ne mesure pas les actions suivantes :

* Actions ignorées en raison de conditions non remplies
* Actions qui ne s’exécutent pas, car l’application logique s’est arrêtée avant la fin

Pour les actions qui s’exécutent dans des boucles, Azure Logic Apps prend en compte chaque action par cycle dans la boucle. Par exemple, supposons que vous ayez une boucle « for each » qui traite une liste. Logic Apps mesure une action dans cette boucle en multipliant le nombre d’éléments dans la liste par le nombre d’actions dans la boucle, puis ajoute l’action qui démarre la boucle. Le calcul pour une liste de 10 éléments est donc (10 * 1) + 1, ce qui donne 11 exécutions d’action.

## <a name="disabled-logic-apps"></a>Applications logiques désactivées

Les applications logiques désactivées ne sont pas facturées, car elles ne peuvent pas créer d’instances lorsqu’elles sont désactivées. Une fois que vous avez désactivé une application logique, l’arrêt complet des instances en cours d’exécution peut prendre un certain de temps.

<a name="integration-accounts"></a>

## <a name="integration-accounts"></a>Comptes d’intégration

Un [modèle de tarification fixe](https://azure.microsoft.com/pricing/details/logic-apps) s’applique aux [comptes d’intégration](logic-apps-enterprise-integration-create-integration-account.md) à des fins d’exploration, de développement et de test des fonctionnalités de [traitement XML](logic-apps-enterprise-integration-b2b.md) et [B2B/EDI](logic-apps-enterprise-integration-xml.md) d’Azure Logic Apps sans coût supplémentaire. Chaque abonnement Azure peut avoir une [limite spécifique de comptes d’intégration](../logic-apps/logic-apps-limits-and-config.md#integration-account-limits). Chaque compte d’intégration peut stocker jusqu’à un [limite spécifique d’artefacts](../logic-apps/logic-apps-limits-and-config.md#artifact-number-limits), qui incluent les contrats, partenaires commerciaux, cartes, schémas, assemblys, certificats, configurations de lot etc.

Azure Logic Apps offre des comptes d’intégration gratuit, de base et standard. Les niveaux De base et Standard sont pris en charge par le contrat de niveau de service (SLA) Logic Apps, tandis que le niveau Gratuit n’est pas pris en charge par un contrat SLA et présente des limites de disponibilité régionale, de débit et d’utilisation. À l’exception des comptes d’intégration de niveau Gratuit, vous pouvez avoir plusieurs comptes d’intégration dans chaque région Azure. Pour connaître la tarification, consultez [Tarification Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/).

Si vous avez un [*environnement de service d'intégration* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), [Premium ou Développeur](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level), votre environnement ISE peut avoir 5 comptes d’intégration au total. Pour savoir comment fonctionne le modèle de tarification fixe pour un environnement ISE, consultez la section précédente [Modèle de tarification fixe](#fixed-pricing) dans cette rubrique. Pour connaître la tarification, consultez [Tarification Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps).

Pour choisir un compte d’intégration gratuit, de base ou standard, étudiez ces descriptions de cas d’usage :

* **Gratuit**  : Lorsque vous voulez essayer les scénarios exploratoires, pas les scénarios de production. Ce niveau est disponible uniquement pour les régions publiques dans Azure (par exemple, USA Ouest et Asie Sud-Est), mais pas pour [Azure China 21Vianet](/azure/china/overview-operations) ni [Azure Government](../azure-government/documentation-government-welcome.md).

* **De base**  : quand vous souhaitez utiliser la gestion des messages ou agir en tant que petite entreprise partenaire ayant une relation de partenariat commercial avec une entité professionnelle plus importante.

* **Standard**  : quand vous avez des relations B2B plus complexes et un nombre croissant d’entités que vous devez gérer.

<a name="data-retention"></a>

## <a name="data-retention"></a>Conservation des données

Hormis les applications logiques qui s’exécutent dans un ISE, toutes les entrées et sorties qui sont stockées dans votre historique des exécutions d’applications logiques sont facturées en fonction de la [période de rétention de l’application logique](logic-apps-limits-and-config.md#run-duration-retention-limits). Les applications logiques qui s’exécutent dans un ISE n’entraînent aucun coût en termes de conservation des données. Pour connaître la tarification, consultez [Tarification Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps).

Pour mieux surveiller la consommation de stockage de votre application logique, vous pouvez :

* Affichez le nombre des unités de stockage en Go utilisés mensuellement par votre application logique.

* Affichez les tailles des entrées et sorties d’une action spécifique dans l’historique des exécutions de votre application logique.

<a name="storage-consumption"></a>

### <a name="view-logic-app-storage-consumption"></a>Afficher la consommation de stockage de l’application logique

1. Dans le portail Azure, recherchez et sélectionnez votre application logique.

1. Depuis le menu de l’application logique, sous **Surveillance** , sélectionnez **Métriques**.

1. Dans le volet à droite, sous **Titre du graphique** , depuis la liste **Métrique** , sélectionnez **Utilisation de la facturation pour les exécutions de consommation du stockage**.

   Cette mesure vous donne le nombre d’unités de consommation de stockage en Go par mois qui sont facturées.

   > [!NOTE]
   > Les exécutions qui consomment moins de 500 Mo de stockage peuvent ne pas apparaître dans l’affichage de la supervision, mais elles sont quand même facturées.

<a name="input-output-sizes"></a>

### <a name="view-action-input-and-output-sizes"></a>Afficher les tailles d’entrée et de sortie d’une action

1. Dans le portail Azure, recherchez et sélectionnez votre application logique.

1. Dans le menu de votre application logique, sélectionnez **Vue d’ensemble**.

1. Dans le volet à droite, sous **Historique des exécutions** , sélectionnez l’exécution qui dispose des entrées et des sorties que vous voulez vérifier.

1. Sous **Exécution de l’application logique** , choisissez **Détails de l’exécution**.

1. Dans le volet **Détails de l’exécution de l’application logique** , dans la table des actions qui répertorie la durée et l’état de chaque action, sélectionnez l’action que vous souhaitez afficher.

1. Dans le volet **Action de l’application logique** , recherchez les tailles des entrées et des sorties de cette action. Sous **Lien d’entrées** et **Lien de sorties** , recherchez les liens vers ces entrées et sorties.

   > [!NOTE]
   > Pour les boucles, seules les actions de niveau supérieur affichent des tailles pour leurs entrées et sorties. Pour les actions dans les boucles imbriquées, les entrées et les sorties affichent une taille nulle et aucun lien.

## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur Azure Logic Apps](logic-apps-overview.md)
* [Créez votre première application logique](quickstart-create-first-logic-app-workflow.md)
