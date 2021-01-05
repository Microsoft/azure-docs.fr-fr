---
title: Limites et configuration
description: Limites de service, telles que la durée, le débit et la capacité, ainsi que valeurs de configuration, telles que des adresses IP à autoriser, pour Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: jonfan, logicappspm
ms.topic: article
ms.date: 12/07/2020
ms.openlocfilehash: ee314708f0d564bf1af639a3d864ea19472425cf
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96937625"
---
# <a name="limits-and-configuration-information-for-azure-logic-apps"></a>Limites et informations de configuration pour Azure Logic Apps

Cet article décrit les limites et les détails de configuration liés à la création et à l’exécution de workflows automatisés avec Azure Logic Apps. Pour Power Automate, consultez [Limites et configuration dans Power Automate](/flow/limits-and-config).

<a name="definition-limits"></a>

## <a name="logic-app-definition-limits"></a>Limites de la définition d’application logique

Les limites pour la définition d’une application logique sont les suivantes :

| Nom | Limite | Notes |
| ---- | ----- | ----- |
| Actions par flux de travail | 500 | Pour étendre cette limite, vous pouvez au besoin ajouter des workflows imbriqués. |
| Niveaux d’imbrication d’actions autorisés | 8 | Pour étendre cette limite, vous pouvez au besoin ajouter des workflows imbriqués. |
| Flux de travail par région et par abonnement | 1 000 | |
| Déclencheurs par flux de travail | 10 | Lors de l’utilisation du mode Code, et non du concepteur |
| Limite de cas de basculement d’étendue | 25 | |
| Variables par workflow | 250 | |
| Nom pour `action` ou `trigger` | 80 caractères | |
| Caractères par expression | 8 192 | |
| Longueur de `description` | 256 caractères | |
| Nombre maximal de `parameters` | 50 | |
| Nombre maximal de `outputs` | 10 | |
| Taille maximale de `trackedProperties` | 16 000 caractères |
| Action de code inclus : nombre maximal de caractères de code | 1 024 caractères | Pour étendre cette limite à 100 000 caractères, créez vos applications logiques à l’aide du type de ressource **Application logique (préversion)** , [en utilisant le portail Azure](create-stateful-stateless-workflows-azure-portal.md) ou [en utilisant Visual Studio Code et l’extension **Azure Logic Apps (préversion)** ](create-stateful-stateless-workflows-visual-studio-code.md). |
| Action de code inline – Durée maximale d’exécution de code | 5 secondes | Pour étendre cette limite à 15 secondes, créez vos applications logiques à l’aide du type de ressource **Application logique (préversion)** , [en utilisant le portail Azure](create-stateful-stateless-workflows-azure-portal.md) ou [en utilisant Visual Studio Code et l’extension **Azure Logic Apps (préversion)** ](create-stateful-stateless-workflows-visual-studio-code.md). |
||||

<a name="run-duration-retention-limits"></a>

## <a name="run-duration-and-retention-history-limits"></a>Durée d’exécution et limites de conservation de l’historique

Les limites pour l’exécution d’une application logique sont les suivantes :

| Nom | Limite multilocataire | Limite d’environnement de service d’intégration | Notes |
|------|--------------------|---------------------------------------|-------|
| Durée d’exécution | 90 jours | 366 jours | La durée d’exécution est calculée à l’aide de l’heure de début d’une exécution et de la limite spécifiée par le paramètre de workflow, [**Conservation de l’historique des exécutions en jours**](#change-duration) à l’heure de début. <p><p>Pour modifier la limite par défaut, consultez [Modifier la durée d’exécution et la conservation de l’historique dans le stockage](#change-duration). |
| Conservation de l’historique des exécutions dans le stockage | 90 jours | 366 jours | Si la durée d’une exécution dépasse la limite de conservation actuelle, l’exécution est supprimée de l’historique des exécutions dans le stockage. Que l’exécution se termine ou expire, la rétention de l’historique des exécutions est toujours calculée en utilisant l’heure de début de l’exécution et la limite actuelle spécifiée dans le paramètre du workflow, [**Conservation de l’historique des exécutions en jours**](#change-retention). Quelle que soit la limite précédente, la limite actuelle est toujours utilisée pour le calcul de la rétention. <p><p>Pour modifier la limite par défaut et pour plus d’informations, consultez [Modifier l’exécution et la conservation de l’historique des exécutions dans le stockage](#change-retention). Pour augmenter la limite maximale, [contactez l’équipe Logic Apps](mailto://logicappsemail@microsoft.com) pour qu’elle réponde à vos besoins. |
| Intervalle de périodicité minimal | 1 seconde | 1 seconde ||
| Intervalle de périodicité maximal | 500 jours | 500 jours ||
|||||

<a name="change-duration"></a>
<a name="change-retention"></a>

### <a name="change-run-duration-and-history-retention-in-storage"></a>Modifier la durée d’exécution et la conservation de l’historique dans le stockage

Le même paramètre contrôle le nombre maximal de jours pendant lesquels un workflow peut s’exécuter, ainsi que la conservation de l’historique des exécutions dans le stockage. Pour modifier la limite par défaut ou actuelle de ces propriétés, procédez comme suit.

* Pour les applications logiques dans Azure multilocataire, la limite par défaut de 90 jours est identique à la limite maximale. Vous pouvez uniquement réduire cette valeur.

* Pour les applications logiques dans un environnement de service d’intégration, vous pouvez réduire ou augmenter la limite par défaut de 90 jours.

Par exemple, supposons que vous réduisiez la limite de conservation de 90 à 30 jours. Une exécution datant de 60 jours est supprimée de l’historique des exécutions. Si vous augmentez la période de conservation de 30 à 60 jours, une exécution datant de 20 jours reste dans l’historique des exécutions pendant 40 jours supplémentaires.

> [!IMPORTANT]
> Si la durée d’une exécution dépasse la limite de conservation actuelle, l’exécution est supprimée de l’historique des exécutions dans le stockage. Pour éviter de perdre l’historique des exécutions, assurez-vous que la limite de conservation est *toujours* plus que la durée la plus longue de l’exécution.

1. Dans la zone de recherche du [portail Azure](https://portal.azure.com), recherchez et sélectionnez **Logic Apps**.

1. Recherchez et sélectionnez votre application logique. Ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Dans le menu de l’application logique, sélectionnez **Paramètres de flux de travail**.

1. Sous **Options de runtime**, dans la liste **Conservation de l’historique des exécutions en jours**, choisissez **Personnalisée**.

1. Faites glisser le curseur pour modifier le nombre de jours souhaité.

1. Lorsque vous avez terminé, dans la barre d’outils **Paramètres du workflow**, sélectionnez **Enregistrer**.

Si vous générez un modèle de Azure Resource Manager pour votre application logique, ce paramètre apparaît en tant que propriété dans la définition de ressource de votre workflow, qui est décrite dans la [référence du modèle de workflow Microsoft. Logic](/azure/templates/microsoft.logic/workflows) :

```json
{
   "name": "{logic-app-name}",
   "type": "Microsoft.Logic/workflows",
   "location": "{Azure-region}",
   "apiVersion": "2019-05-01",
   "properties": {
      "definition": {},
      "parameters": {},
      "runtimeConfiguration": {
         "lifetime": {
            "unit": "day",
            "count": {number-of-days}
         }
      }
   }
}
```

<a name="looping-debatching-limits"></a>

## <a name="concurrency-looping-and-debatching-limits"></a>Limites de simultanéité, de bouclage et de décomposition

Les limites pour l’exécution d’une application logique sont les suivantes :

### <a name="loops"></a>Boucles

| Nom | Limite | Notes |
| ---- | ----- | ----- |
| Éléments du tableau Foreach | 100 000 | Cette limite décrit le nombre maximal d’éléments de tableau qu’une boucle « for each » peut traiter. <p><p>Pour filtrer des tables plus grandes, vous pouvez utiliser l’[action de requête](logic-apps-perform-data-operations.md#filter-array-action). |
| Accès concurrentiel Foreach | Avec la concurrence désactivée : 20 <p><p>Avec la concurrence activée : <p><p>- Par défaut : 20 <br>- Min : 1 <br>- Max : 50 | Cette limite indique le nombre maximal d’itérations de boucles « for each » qui peuvent s’exécuter simultanément ou en parallèle. <p><p>Pour changer cette limite, consultez [Changer la limite de concurrence « for each »](../logic-apps/logic-apps-workflow-actions-triggers.md#change-for-each-concurrency) ou [Exécuter des boucles « for each » séquentiellement](../logic-apps/logic-apps-workflow-actions-triggers.md#sequential-for-each). |
| Itérations Until | - Par défaut : 60 <br>- Min : 1 <br>- Max : 5 000 | Nombre le plus élevé de cycles qu’une boucle « Until » peut avoir pendant l’exécution d’une application logique. <p><p>Pour modifier cette limite, dans la forme de la boucle « Until », sélectionnez **Modifier les limites**, puis spécifiez la valeur de la propriété **Count**. |
| Délai d’expiration d’Until | - Par défaut : PT1H (1 heure) | La durée la plus longue d’exécution de la boucle « Until » avant qu’elle ne se termine, spécifiée au [format ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). La valeur du délai d’attente est évaluée pour chaque cycle de boucle. Si une action dans la boucle dure plus longtemps que la limite de délai d’attente, le cycle actuel ne s’arrête pas. Toutefois, le cycle suivant ne démarre pas, car la condition de limite n’est pas remplie. <p><p>Pour modifier cette limite, dans la forme de la boucle « Until », sélectionnez **Modifier les limites**, puis spécifiez la valeur de la propriété **Timeout**. |
||||

### <a name="concurrency-and-debatching"></a>Accès concurrentiel et décomposition

| Nom | Limite | Notes |
| ---- | ----- | ----- |
| Déclencheur simultané | Avec la concurrence désactivée : Illimité <p><p>Avec la concurrence activée, ce que vous ne pouvez pas faire après l’activation : <p><p>- Par défaut : 25 <br>- Min : 1 <br>- Max : 50 | Cette limite est le nombre maximal d’instances d’application logique pouvant être exécutées simultanément ou en parallèle. <p><p>**Remarque** : Lorsque la concurrence est activée, la limite SplitOn est réduite à 100 éléments pour la [décomposition des tableaux](../logic-apps/logic-apps-workflow-actions-triggers.md#split-on-debatch). <p><p>Pour modifier cette limite, consultez [Modifier la limite de concurrence du déclencheur](../logic-apps/logic-apps-workflow-actions-triggers.md#change-trigger-concurrency) ou [Déclencher des instances séquentiellement](../logic-apps/logic-apps-workflow-actions-triggers.md#sequential-trigger). |
| Exécutions en attente maximale | Avec la concurrence désactivée : <p><p>- Min : 1 <br>- Max : 50 <p><p>Avec la concurrence activée : <p><p>- Min : 10, plus le nombre d’exécutions simultanées (déclencher la concurrence) <br>- Max : 100 | Cette limite est le nombre maximal d’instances d’application logique pouvant attendre de s’exécuter quand votre application logique exécute déjà le nombre maximal d’instances simultanées. <p><p>Pour changer cette limite, consultez [Changer la limite d’exécutions en attente](../logic-apps/logic-apps-workflow-actions-triggers.md#change-waiting-runs). |
| Éléments SplitOn | Avec la concurrence désactivée : 100 000 <p><p>Avec la concurrence activée : 100 | Pour les déclencheurs qui retournent un tableau, vous pouvez spécifier une expression utilisant une propriété « SplitOn » qui [fractionne ou dégroupe des éléments de tableau en plusieurs instances de workflows](../logic-apps/logic-apps-workflow-actions-triggers.md#split-on-debatch) à des fins de traitement, au lieu d’utiliser une boucle « for each ». Cette expression fait référence au tableau à utiliser pour la création et l’exécution d’une instance de workflow pour chaque élément du tableau. <p><p>**Remarque** : Lorsque la concurrence est activée, la limite SplitOn est réduite à 100 éléments. |
||||

<a name="throughput-limits"></a>

## <a name="throughput-limits"></a>Limites de débit

Les limites pour la définition d’une application logique sont les suivantes :

### <a name="multi-tenant-logic-apps-service"></a>Service Logic Apps multilocataire

| Nom | Limite | Notes |
| ---- | ----- | ----- |
| Action : Exécutions par tranche de 5 minutes | 100 000 est la limite par défaut, mais 300 000 est la limite maximale. | Pour augmenter la limite par défaut à la valeur maximale de votre application logique, consultez [Exécuter en mode de débit élevé](#run-high-throughput-mode), disponible dans la préversion. Vous pouvez [répartir la charge de travail entre plusieurs applications logiques](../logic-apps/handle-throttling-problems-429-errors.md#logic-app-throttling) si nécessaire. |
| Action : Appels sortants simultanés | ~2,500 | Vous pouvez diminuer le nombre de requêtes simultanées ou réduire la durée si nécessaire. |
| Point de terminaison du runtime : Appels entrants simultanés | ~1,000 | Vous pouvez diminuer le nombre de requêtes simultanées ou réduire la durée si nécessaire. |
| Point de terminaison du runtime : appels de lecture toutes les cinq minutes  | 60 000 | Cette limite s’applique aux appels qui obtiennent les entrées et sorties brutes à partir de l’historique des exécutions d’une application logique. Vous pouvez répartir la charge de travail entre plusieurs applications si nécessaire. |
| Point de terminaison du runtime : appels d’invocation toutes les cinq minutes | 45,000 | Vous pouvez répartir une charge de travail entre plusieurs applications si nécessaire. |
| Débit de contenu par tranche de 5 minutes | 600 Mo | Vous pouvez répartir une charge de travail entre plusieurs applications si nécessaire. |
||||

<a name="run-high-throughput-mode"></a>

#### <a name="run-in-high-throughput-mode"></a>Exécuter en mode de débit élevé

Pour une définition d’application logique unique, le nombre d’actions qui s’exécutent toutes les cinq minutes présente une [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#throughput-limits). Pour augmenter la limite par défaut à la valeur maximale de votre application logique, vous pouvez exécuter en mode de débit élevé, disponible dans la préversion. Vous pouvez [répartir la charge de travail entre plusieurs applications logiques](../logic-apps/handle-throttling-problems-429-errors.md#logic-app-throttling) si nécessaire.

1. Dans le portail Azure, dans le menu de votre application logique, sous **Paramètres**, sélectionnez **Paramètres de workflow**.

1. Sous **Options d’exécution** > **Débit élevé**, **activez** le paramètre.

   ![Capture d’écran montrant le menu d’application logique dans le portail Azure avec les options « Paramètres de workflow » et « Débit élevé » activées.](./media/logic-apps-limits-and-config/run-high-throughput-mode.png)

Pour activer ce paramètre dans un modèle ARM pour le déploiement de votre application logique, dans l’objet `properties` de la définition de ressource de votre application logique, ajoutez l’objet `runtimeConfiguration` avec la propriété `operationOptions` définie sur `OptimizedForHighThroughput` :

```json
{
   <template-properties>
   "resources": [
      // Start logic app resource definition
      {
         "properties": {
            <logic-app-resource-definition-properties>,
            <logic-app-workflow-definition>,
            <more-logic-app-resource-definition-properties>,
            "runtimeConfiguration": {
               "operationOptions": "OptimizedForHighThroughput"
            }
         },
         "name": "[parameters('LogicAppName')]",
         "type": "Microsoft.Logic/workflows",
         "location": "[parameters('LogicAppLocation')]",
         "tags": {},
         "apiVersion": "2016-06-01",
         "dependsOn": [
         ]
      }
      // End logic app resource definition
   ],
   "outputs": {}
}
```

Pour plus d’informations sur la définition de ressource de votre application logique, consultez [Vue d’ensemble : Automatiser le déploiement pour le service Azure Logic Apps à l’aide de modèles Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md#logic-app-resource-definition).

### <a name="integration-service-environment-ise"></a>Environnement de service d’intégration (ISE)

Les limites de débit pour la [référence SKU ISE Premium](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level) sont les suivantes :

| Nom | Limite | Notes |
|------|-------|-------|
| Limite d’exécution de l’unité de base | Limitation du système quand la capacité de l’infrastructure atteint 80 % | Assure environ 4 000 exécutions d’action par minute, ce qui représente environ 160 millions d’exécutions d’action par mois | |
| Limite d’exécution de l’unité d’échelle | Limitation du système quand la capacité de l’infrastructure atteint 80 % | Chaque unité d’échelle peut assurer environ 2 000 exécutions d’action supplémentaires par minute, ce qui représente environ 80 millions d’exécutions d’action supplémentaires par mois | |
| Nombre maximum d’unités d’échelle que vous pouvez ajouter | 10 | |
||||

Pour dépasser ces limites dans le cadre d’un traitement normal ou exécuter des tests de charge susceptibles d’entraîner leur dépassement, [contactez l’équipe Logic Apps](mailto://logicappsemail@microsoft.com) afin d’obtenir de l’aide.

> [!NOTE]
> La [référence SKU ISE Développeur](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level) n’a pas de limites publiées, aucune possibilité de montée en puissance et aucun contrat SLA. Utilisez cette référence SKU uniquement à des fins d’expérimentation, de développement et de test, pas pour la production ou les tests de performances.

<a name="gateway-limits"></a>

## <a name="gateway-limits"></a>Limites de la passerelle

Azure Logic Apps prend en charge les opérations d’écriture, notamment les insertions et les mises à jour, par le biais de la passerelle. Toutefois, ces opérations ont des [limites quant à la taille de leur charge utile](/data-integration/gateway/service-gateway-onprem#considerations).

<a name="http-limits"></a>

## <a name="http-limits"></a>Limites HTTP

Voici les limites d’un seul appel entrant ou sortant :

<a name="http-timeout-limits"></a>

#### <a name="timeout-duration"></a>Durée du délai d’expiration

Comme certaines opérations de connecteur effectuent des appels asynchrones ou écoutent les demandes de webhook, le délai d’expiration pour ces opérations peut dépasser ces limites. Pour plus d’informations, consultez les détails techniques pour le connecteur spécifique et aussi [Actions et déclencheurs de workflow](../logic-apps/logic-apps-workflow-actions-triggers.md#http-action).

| Nom | Logic Apps (multilocataire) | Logic Apps (préversion) | Environnement de service d’intégration (ISE) | Notes |
|------|---------------------------|----------------------|---------------------------------|-------|
| Requête sortante | 120 secondes <br>(2 minutes) | 230 secondes <br>(3,9 minutes) | 240 secondes <br>(4 minutes) | Les appels effectués par l’action ou le déclencheur HTTP sont des exemples de demandes sortantes. Pour plus d’informations sur la préversion, consultez [Préversion d’Azure Logic Apps](logic-apps-overview-preview.md). <p><p>**Conseil** : Pour les opérations en cours d’exécution plus longues, utilisez un [modèle d’interrogation asynchrone](../logic-apps/logic-apps-create-api-app.md#async-pattern) ou une [boucle Until](../logic-apps/logic-apps-workflow-actions-triggers.md#until-action). Pour contourner les limites de délai d’attente lorsque vous appelez une autre application logique qui a un [point de terminaison appelable](logic-apps-http-endpoint.md), vous pouvez utiliser l’action Azure Logic Apps intégrée à la place, que vous pouvez trouver dans le sélecteur de connecteur sous **Élément intégré**. |
| Requête entrante | 120 secondes <br>(2 minutes) | 230 secondes <br>(3,9 minutes) | 240 secondes <br>(4 minutes) | Les appels reçus par le déclencheur de requête, le déclencheur Webhook HTTP et l’action Webhook HTTP sont des exemples de demandes entrantes. Pour plus d’informations sur la préversion, consultez [Préversion d’Azure Logic Apps](logic-apps-overview-preview.md). <p><p>**Remarque** : Pour que l’appelant d’origine obtienne la réponse, toutes les étapes de la réponse doivent être terminées avant la limite, sauf si vous appelez une autre application logique en tant que workflow imbriqué. Pour plus d’informations, consultez [Appeler, déclencher ou imbriquer des applications logiques](../logic-apps/logic-apps-http-endpoint.md). |
||||||

<a name="message-size-limits"></a>

#### <a name="message-size"></a>Taille des messages

| Nom | Limite multilocataire | Limite d’environnement de service d’intégration | Notes |
|------|--------------------|---------------------------------------|-------|
| Taille des messages | 100 Mo | 200 Mo | Pour contourner cette limite, consultez [Gérer les messages volumineux avec la segmentation](../logic-apps/logic-apps-handle-large-messages.md). Toutefois, certains connecteurs et API peuvent ne pas prendre en charge la segmentation ou même la limite par défaut. <p><p>- Les connecteurs tels que AS2, X12 et EDIFACT ont leur propre [limites de messages B2B](#b2b-protocol-limits). <br>- Les connecteurs ISE utilisent la limite ISE, et non les limites de leurs connecteurs non-ISE. |
| Taille des messages avec segmentation | 1 Go | 5 Go | Cette limite s’applique aux actions qui prennent en charge la segmentation en mode natif ou vous permettent d’activer la segmentation dans la configuration de leur runtime. <p><p>Si vous utilisez un ISE, le moteur Logic Apps prend en charge cette limite, mais les connecteurs ont leurs propres limites de segmentation jusqu’à la limite du moteur. Pour un exemple, consultez les [Informations de référence sur l’API du connecteur Stockage Blob Azure](/connectors/azureblob/). Pour plus d’informations sur la segmentation, consultez [Gérer les messages volumineux avec la segmentation](../logic-apps/logic-apps-handle-large-messages.md). |
|||||

#### <a name="character-limits"></a>Limites de caractères

| Nom | Notes |
|------|-------|
| Limite d’évaluation des expressions | 131 072 caractères | Les expressions `@concat()`, `@base64()`, `@string()` ne peuvent pas contenir plus de caractères. |
| Limite de caractères de l’URL de la requête | 16 384 caractères |
|||

<a name="retry-policy-limits"></a>

#### <a name="retry-policy"></a>Stratégie de nouvelle tentative

| Nom | Limite | Notes |
| ---- | ----- | ----- |
| Nouvelles tentatives | 90 | Valeur par défaut : 4. Pour modifier la valeur par défaut, utilisez le [paramètre de stratégie de nouvelles tentatives](../logic-apps/logic-apps-workflow-actions-triggers.md). |
| Délai maximal avant nouvelle tentative | 1 jour | Pour modifier la valeur par défaut, utilisez le [paramètre de stratégie de nouvelles tentatives](../logic-apps/logic-apps-workflow-actions-triggers.md). |
| Délai minimal avant nouvelle tentative | 5 secondes | Pour modifier la valeur par défaut, utilisez le [paramètre de stratégie de nouvelles tentatives](../logic-apps/logic-apps-workflow-actions-triggers.md). |
||||

<a name="authentication-limits"></a>

### <a name="authentication-limits"></a>Limites d’authentification

Voici les limites d’une application logique qui démarre avec un déclencheur de requête et active [Azure Active Directory Open Authentication](../active-directory/develop/index.yml) (Azure AD OAuth) pour autoriser les appels entrants vers le déclencheur de requête :

| Nom | Limite | Notes |
| ---- | ----- | ----- |
| Stratégies d’autorisation Azure AD | 5 | |
| Demandes par stratégie d’autorisation | 10 | |
| Valeur de revendication – Nombre maximal de caractères | 150 |
||||

<a name="custom-connector-limits"></a>

## <a name="custom-connector-limits"></a>Limites des connecteurs personnalisés

Les limites pour les connecteurs personnalisés qu’il est possible de créer à partir d’API web sont les suivantes.

| Nom | Limite multilocataire | Limite d’environnement de service d’intégration | Notes |
|------|--------------------|---------------------------------------|-------|
| Nombre de connecteurs personnalisés | 1 000 par abonnement Azure | 1 000 par abonnement Azure ||
| Nombre de demandes par minute pour un connecteur personnalisé | 500 demandes par minute par connexion | 2 000 demandes par minute par *connecteur personnalisé* ||
|||

<a name="managed-identity"></a>

## <a name="managed-identities"></a>Identités managées

| Nom | Limite |
|------|-------|
| Identités gérées par application logique | Identité affectée par le système ou identité affectée par 1 utilisateur |
| Nombre d’applications logiques qui ont une identité managée dans un abonnement Azure par région | 1 000 |
|||

<a name="integration-account-limits"></a>

## <a name="integration-account-limits"></a>Limites du compte d’intégration

Chaque abonnement Azure a les limites de compte d’intégration suivantes :

* Un compte d’intégration par région Azure [niveau Gratuit](../logic-apps/logic-apps-pricing.md#integration-accounts). Ce niveau est disponible uniquement pour les régions publiques dans Azure (par exemple, USA Ouest et Asie Sud-Est), mais pas pour [Azure China 21Vianet](/azure/china/overview-operations) ni [Azure Government](../azure-government/documentation-government-welcome.md).

* 1 000 comptes d’intégration au total, notamment des comptes d’intégration dans des [environnements de service d’intégration](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) sur les [références SKU Développeur et Premium](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level).

* Chaque environnement de service d’intégration, [Développeur ou Premium](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level), est limité à un nombre total de comptes d’intégration, mais [vous pouvez augmenter cette limite pour un coût supplémentaire](logic-apps-pricing.md#fixed-pricing) :

  | Référence (SKU) de l’environnement de service d’intégration | Limites du compte d’intégration |
  |---------|----------------------------|
  | **Premium** | 20 comptes [Standard](../logic-apps/logic-apps-pricing.md#integration-accounts) au total, y compris un compte standard gratuit. Vous pouvez avoir [plus de comptes d’intégration contre un supplément](logic-apps-pricing.md#fixed-pricing). Aucun compte Gratuit ou De base n’est autorisé. |
  | **Développeur** | 20 comptes [Gratuits](../logic-apps/logic-apps-pricing.md#integration-accounts) au total (limité à 1 compte) et [Standard](../logic-apps/logic-apps-pricing.md#integration-accounts) combinés, ou tous les comptes standard. Vous pouvez avoir [plus de comptes d’intégration contre un supplément](logic-apps-pricing.md#fixed-pricing). Aucun compte De base n’est autorisé. Utilisez la [référence SKU du développeur](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level) à des fins d’expérimentation, de développement et de test, mais pas pour la production ou les tests de performances. |
  |||

Pour plus d’informations sur la tarification et la facturation des environnements de service d’intégration, consultez le [modèle de tarif pour Logic Apps](../logic-apps/logic-apps-pricing.md#fixed-pricing). Pour connaître la tarification, consultez [Tarification Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/).

<a name="artifact-number-limits"></a>

### <a name="artifact-limits-per-integration-account"></a>Limites du nombre d’artefacts par compte d’intégration

Les limites du nombre d’artefacts pour chaque niveau de compte d’intégration sont les suivantes.
Pour connaître la tarification, consultez [Tarification Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/). Pour plus d’informations sur la tarification et la facturation des comptes d’intégration, consultez [Modèle de tarification Logic apps](../logic-apps/logic-apps-pricing.md#integration-accounts).

> [!NOTE]
> Utilisez le niveau gratuit uniquement pour les scénarios exploratoires, pas pour les scénarios de production. Ce niveau limite le débit et l’utilisation et n’inclut aucun contrat de niveau de service (SLA).

| Artefact | Gratuit | De base | Standard |
|----------|------|-------|----------|
| Contrats commerciaux EDI | 10 | 1 | 1 000 |
| Partenaires commerciaux EDI | 25 | 2 | 1 000 |
| Cartes | 25 | 500 | 1 000 |
| Schémas | 25 | 500 | 1 000 |
| Assemblys | 10 | 25 | 1 000 |
| Certificats | 25 | 2 | 1 000 |
| Configurations par lots | 5 | 1 | 50 |
||||

<a name="artifact-capacity-limits"></a>

### <a name="artifact-capacity-limits"></a>Limites de capacité des artefacts

| Artefact | Limite | Notes |
| -------- | ----- | ----- |
| Assembly | 8 Mo | Pour charger des fichiers d’une taille supérieure à 2 Mo, utilisez un [compte de stockage Azure et un conteneur d’objets blob](../logic-apps/logic-apps-enterprise-integration-schemas.md). |
| Mappage (fichier XSLT) | 8 Mo | Pour charger des fichiers d’une taille supérieure à 2 Mo, utilisez l’[API REST Azure Logic Apps – Maps](/rest/api/logic/maps/createorupdate). <p><p>**Remarque** : La quantité de données ou d’enregistrements qu’un mappage peut traiter avec succès est basée sur les limites de taille de message et de délai d’expiration d’action dans Azure Logic Apps. Par exemple, si vous utilisez une action HTTP, en fonction des [limites de taille et de délai d’attente des messages HTTP](#http-limits), un mappage peut traiter les données jusqu’à la limite de taille de message HTTP si l’opération se termine dans le délai imparti par le protocole HTTP. |
| schéma | 8 Mo | Pour charger des fichiers d’une taille supérieure à 2 Mo, utilisez un [compte de stockage Azure et un conteneur d’objets blob](../logic-apps/logic-apps-enterprise-integration-schemas.md). |
||||

<a name="integration-account-throughput-limits"></a>

### <a name="throughput-limits"></a>Limites de débit

| Point de terminaison du runtime | Gratuit | De base | Standard | Notes |
|------------------|------|-------|----------|-------|
| appels de lecture toutes les cinq minutes | 3 000 | 30,000 | 60 000 | Cette limite s’applique aux appels qui obtiennent les entrées et sorties brutes à partir de l’historique des exécutions d’une application logique. Vous pouvez répartir la charge de travail entre plusieurs comptes si nécessaire. |
| appels d’invocation toutes les cinq minutes | 3 000 | 30,000 | 45,000 | Vous pouvez répartir la charge de travail entre plusieurs comptes si nécessaire. |
| appels de suivi toutes les cinq minutes | 3 000 | 30,000 | 45,000 | Vous pouvez répartir la charge de travail entre plusieurs comptes si nécessaire. |
| appels simultanés de blocage | ~1,000 | ~1,000 | ~1,000 | Identique pour toutes les références (SKU). Vous pouvez diminuer le nombre de requêtes simultanées ou réduire la durée si nécessaire. |
||||

<a name="b2b-protocol-limits"></a>

### <a name="b2b-protocol-as2-x12-edifact-message-size"></a>Taille des messages de protocole B2B (AS2, X12, EDIFACT)

Les limites de taille des messages qui s’appliquent aux protocoles B2B sont les suivantes :

| Nom | Limite multilocataire | Limite d’environnement de service d’intégration | Notes |
|------|--------------------|---------------------------------------|-------|
| AS2 | v2 - 100 Mo<br>v1 - 25 Mo | v2 - 200 Mo <br>v1 - 25 Mo | S’applique au décodage et à l’encodage. |
| X 12 | 50 Mo | 50 Mo | S’applique au décodage et à l’encodage. |
| EDIFACT | 50 Mo | 50 Mo | S’applique au décodage et à l’encodage. |
||||

<a name="disable-delete"></a>

## <a name="disabling-or-deleting-logic-apps"></a>Désactivation ou suppression des applications logiques

Lorsque vous désactivez une application logique, aucune nouvelle exécution n’est instanciée. Toutes les exécutions en cours et en attente continuent jusqu’à ce qu’elles soient terminées, ce qui peut prendre du temps.

Lorsque vous supprimez une application logique, aucune nouvelle exécution n’est instanciée. Toutes les exécutions en cours et en attente sont annulées. Si vous avez des milliers d’exécutions, l’annulation peut prendre beaucoup de temps.

<a name="configuration"></a>

## <a name="firewall-configuration-ip-addresses-and-service-tags"></a>Configuration du pare-feu : Adresses IP et balises de service

Les adresses IP qu’Azure Logic Apps utilise pour les appels entrants et sortants dépendent de la région où se trouve votre application logique. *Toutes* les applications logiques qui se trouvent dans une même région utilisent les mêmes plages d’adresses IP. Certains appels de [Power Automate](/power-automate/getting-started), tels que les requêtes **HTTP** et **HTTP + OpenAPI**, passent directement par le service Azure Logic Apps et proviennent des adresses IP listées ici. Pour plus d’informations sur les adresses IP utilisées par Power Automate, consultez [Limites et configuration dans Power Automate](/flow/limits-and-config#ip-address-configuration).

> [!TIP]
> Pour réduire la complexité de la création des règles de sécurité, vous pouvez éventuellement utiliser les [balises de service](../virtual-network/service-tags-overview.md), au lieu de spécifier les adresses IP Logic Apps pour chaque région, comme décrit plus loin dans cette section.
> Ces balises fonctionnent dans les régions où le service Logic Apps est disponible :
>
> * **LogicAppsManagement** : représente les préfixes d’adresses IP entrantes pour le service Logic Apps.
> * **LogicApps** : représente les préfixes d’adresse IP sortantes pour le service Logic Apps.

* Pour [Azure Chine 21Vianet](/azure/china/), les adresses IP fixes ou réservées ne sont pas disponibles pour les [connecteurs personnalisés](../logic-apps/custom-connector-overview.md) et les [connecteurs managés](../connectors/apis-list.md#managed-api-connectors), par exemple Stockage Azure, SQL Server, Office 365 Outlook, etc.

* Pour prendre en charge les appels que les applications logiques passent directement à [HTTP](../connectors/connectors-native-http.md), [HTTP + Swagger](../connectors/connectors-native-http-swagger.md) et à d’autres requêtes HTTP, configurez votre pare-feu avec toutes les adresses IP [entrantes](#inbound) *et* [sortantes](#outbound) qui sont utilisées par le service Logic Apps, en fonction de l’emplacement de vos applications logiques. Ces adresses s’affichent sous les en-têtes **Entrantes** et **Sortantes** dans cette section, et sont triées par région.

* Pour prendre en charge les appels que passent les [connecteurs managés](../connectors/apis-list.md#managed-api-connectors), configurez votre pare-feu avec *toutes* les adresses IP [sortantes](#outbound) utilisées par ces connecteurs, en fonction de l’emplacement de vos applications logiques. Ces adresses s’affichent sous l’en-tête **Sortantes** dans cette section, et sont triées par région.

* Pour permettre la communication des applications logiques qui s’exécutent dans un environnement de service d’intégration (ISE), veillez à [ouvrir ces ports](../logic-apps/connect-virtual-network-vnet-isolated-environment.md#network-ports-for-ise).

* Si vos applications logiques rencontrent des problèmes d’accès aux comptes de stockage Azure qui utilisent des [pare-feu et des règles de pare-feu](../storage/common/storage-network-security.md), vous disposez de [différentes options pour activer l’accès](../connectors/connectors-create-api-azureblobstorage.md#access-storage-accounts-behind-firewalls).

  Par exemple, les applications logiques n’ont pas directement accès aux comptes de stockage qui utilisent des règles de pare-feu et se trouvent dans la même région. Toutefois, si vous autorisez les [adresses IP sortantes pour les connecteurs managés dans votre région](../logic-apps/logic-apps-limits-and-config.md#outbound), vos applications logiques peuvent accéder à des comptes de stockage dans une autre région, sauf lorsque vous utilisez le connecteur de Stockage Table Azure ou des connecteurs de Stockage File d’attente Azure. Pour accéder à votre Stockage Table ou Stockage File d’attente, vous pouvez toujours utiliser le déclencheur et les actions HTTP à la place. Pour d’autres options, voir [Accéder à des comptes de stockage derrière des pare-feu](../connectors/connectors-create-api-azureblobstorage.md#access-storage-accounts-behind-firewalls).

<a name="inbound"></a>

### <a name="inbound-ip-addresses"></a>Adresses IP entrantes

Cette section répertorie les adresses IP entrantes pour le service Azure Logic Apps uniquement. Si vous avez Azure Government, consultez [Azure Government - Adresses IP entrantes](#azure-government-inbound).

> [!TIP]
> Pour réduire la complexité de la création des règles de sécurité, vous pouvez éventuellement utiliser la [balise de service](../virtual-network/service-tags-overview.md), **LogicAppsManagement**, au lieu de spécifier les préfixes d’adresses IP Logic Apps entrants pour chaque région.
> Cette balise fonctionne dans les régions où le service Logic Apps est disponible.

<a name="multi-tenant-inbound"></a>

#### <a name="multi-tenant-azure---inbound-ip-addresses"></a>Azure multilocataires - Adresses IP entrantes

| Région multilocataire | IP |
|---------------------|----|
| Australie Est | 13.75.153.66, 104.210.89.222, 104.210.89.244, 52.187.231.161 |
| Sud-Australie Est | 13.73.115.153, 40.115.78.70, 40.115.78.237, 52.189.216.28 |
| Brésil Sud | 191.235.86.199, 191.235.95.229, 191.235.94.220, 191.234.166.198 |
| Centre du Canada | 13.88.249.209, 52.233.30.218, 52.233.29.79, 40.85.241.105 |
| Est du Canada | 52.232.129.143, 52.229.125.57, 52.232.133.109, 40.86.202.42 |
| Inde centrale | 52.172.157.194, 52.172.184.192, 52.172.191.194, 104.211.73.195 |
| USA Centre | 13.67.236.76, 40.77.111.254, 40.77.31.87, 104.43.243.39 |
| Asie Est | 168.63.200.173, 13.75.89.159, 23.97.68.172, 40.83.98.194 |
| USA Est | 137.135.106.54, 40.117.99.79, 40.117.100.228, 137.116.126.165 |
| USA Est 2 | 40.84.25.234, 40.79.44.7, 40.84.59.136, 40.70.27.253 |
| France Centre | 52.143.162.83, 20.188.33.169, 52.143.156.55, 52.143.158.203 |
| France Sud | 52.136.131.145, 52.136.129.121, 52.136.130.89, 52.136.131.4 |
| Allemagne Nord | 51.116.211.29, 51.116.208.132, 51.116.208.37, 51.116.208.64 |
| Allemagne Centre-Ouest | 51.116.168.222, 51.116.171.209, 51.116.233.40, 51.116.175.0 |
| Japon Est | 13.71.146.140, 13.78.84.187, 13.78.62.130, 13.78.43.164 |
| OuJapon Est | 40.74.140.173, 40.74.81.13, 40.74.85.215, 40.74.68.85 |
| Centre de la Corée | 52.231.14.182, 52.231.103.142, 52.231.39.29, 52.231.14.42 |
| Corée du Sud | 52.231.166.168, 52.231.163.55, 52.231.163.150, 52.231.192.64 |
| Centre-Nord des États-Unis | 168.62.249.81, 157.56.12.202, 65.52.211.164, 65.52.9.64 |
| Europe Nord | 13.79.173.49, 52.169.218.253, 52.169.220.174, 40.112.90.39 |
| Afrique du Sud Nord | 102.133.228.4, 102.133.224.125, 102.133.226.199, 102.133.228.9 |
| Afrique du Sud Ouest | 102.133.72.190, 102.133.72.145, 102.133.72.184, 102.133.72.173 |
| États-Unis - partie centrale méridionale | 13.65.98.39, 13.84.41.46, 13.84.43.45, 40.84.138.132 |
| Inde Sud | 52.172.9.47, 52.172.49.43, 52.172.51.140, 104.211.225.152 |
| Asie Sud-Est | 52.163.93.214, 52.187.65.81, 52.187.65.155, 104.215.181.6 |
| Suisse Nord | 51.103.128.52, 51.103.132.236, 51.103.134.138, 51.103.136.209 |
| Émirats arabes unis Centre | 20.45.75.193, 20.45.64.29, 20.45.64.87, 20.45.71.213 |
| Sud du Royaume-Uni | 51.140.79.109, 51.140.78.71, 51.140.84.39, 51.140.155.81 |
| Ouest du Royaume-Uni | 51.141.48.98, 51.141.51.145, 51.141.53.164, 51.141.119.150 |
| Centre-USA Ouest | 52.161.26.172, 52.161.8.128, 52.161.19.82, 13.78.137.247 |
| Europe Ouest | 13.95.155.53, 52.174.54.218, 52.174.49.6 |
| Inde Ouest | 104.211.164.112, 104.211.165.81, 104.211.164.25, 104.211.157.237 |
| USA Ouest | 52.160.90.237, 138.91.188.137, 13.91.252.184, 157.56.160.212 |
| USA Ouest 2 | 13.66.224.169, 52.183.30.10, 52.183.39.67, 13.66.128.68 |
|||

<a name="azure-government-inbound"></a>

#### <a name="azure-government---inbound-ip-addresses"></a>Azure Government - Adresses IP entrantes

| Région Azure Government | IP |
|-------------------------|----|
| Gouvernement des États-Unis – Arizona | 52.244.67.164, 52.244.67.64, 52.244.66.82 |
| Gouvernement des États-Unis – Texas | 52.238.119.104, 52.238.112.96, 52.238.119.145 |
| Gouvernement américain - Virginie | 52.227.159.157, 52.227.152.90, 23.97.4.36 |
| Centre des États-Unis – US DoD | 52.182.49.204, 52.182.52.106 |
|||

<a name="outbound"></a>

### <a name="outbound-ip-addresses"></a>Adresses IP sortantes

Cette section répertorie les adresses IP sortantes pour le service Azure Logic Apps et les connecteurs managés. Si vous avez Azure Government, consultez [Azure Government - Adresses IP sortantes](#azure-government-outbound).

> [!TIP]
> Pour réduire la complexité de la création des règles de sécurité, vous pouvez éventuellement utiliser la [balise de service](../virtual-network/service-tags-overview.md), **LogicApps**, au lieu de spécifier les préfixes d’adresses IP Logic Apps sortants pour chaque région.
> Pour les connecteurs gérés, vous pouvez éventuellement utiliser la balise de service **AzureConnectors**, plutôt que de spécifier des préfixes d’adresse IP sortants de connecteur géré pour chaque région. Ces balises fonctionnent dans les régions où le service Logic Apps est disponible. 

<a name="multi-tenant-outbound"></a>

#### <a name="multi-tenant-azure---outbound-ip-addresses"></a>Azure multilocataires - Adresses IP sortantes

| Région multilocataire | Adresse IP Logic Apps | Adresse IP des connecteurs managés |
|---------------------|---------------|-----------------------|
| Australie Est | 13.75.149.4, 104.210.91.55, 104.210.90.241, 52.187.227.245, 52.187.226.96, 52.187.231.184, 52.187.229.130, 52.187.226.139 | 52.237.214.72, 13.72.243.10, 13.70.72.192 - 13.70.72.207, 13.70.78.224 - 13.70.78.255 |
| Sud-Australie Est | 13.73.114.207, 13.77.3.139, 13.70.159.205, 52.189.222.77, 13.77.56.167, 13.77.58.136, 52.189.214.42, 52.189.220.75 | 52.255.48.202, 13.70.136.174, 13.77.50.240 - 13.77.50.255, 13.77.55.160 - 13.77.55.191 |
| Brésil Sud | 191.235.82.221, 191.235.91.7, 191.234.182.26, 191.237.255.116, 191.234.161.168, 191.234.162.178, 191.234.161.28, 191.234.162.131 | 191.232.191.157, 104.41.59.51, 191.233.203.192 - 191.233.203.207, 191.233.207.160 - 191.233.207.191 |
| Centre du Canada | 52.233.29.92, 52.228.39.244, 40.85.250.135, 40.85.250.212, 13.71.186.1, 40.85.252.47, 13.71.184.150 | 52.237.32.212, 52.237.24.126, 13.71.170.208 - 13.71.170.223, 13.71.175.160 - 13.71.175.191 |
| Est du Canada | 52.232.128.155, 52.229.120.45, 52.229.126.25, 40.86.203.228, 40.86.228.93, 40.86.216.241, 40.86.226.149, 40.86.217.241 | 52.242.30.112, 52.242.35.152, 40.69.106.240 - 40.69.106.255, 40.69.111.0 - 40.69.111.31 |
| Inde centrale | 52.172.154.168, 52.172.186.159, 52.172.185.79, 104.211.101.108, 104.211.102.62, 104.211.90.169, 104.211.90.162, 104.211.74.145 | 52.172.212.129, 52.172.211.12, 20.43.123.0 - 20.43.123.31, 104.211.81.192 - 104.211.81.207 |
| USA Centre | 13.67.236.125, 104.208.25.27, 40.122.170.198, 40.113.218.230, 23.100.86.139, 23.100.87.24, 23.100.87.56, 23.100.82.16 | 52.173.241.27, 52.173.245.164, 13.89.171.80 - 13.89.171.95, 13.89.178.64 - 13.89.178.95 |
| Asie Est | 13.75.94.173, 40.83.127.19, 52.175.33.254, 40.83.73.39, 65.52.175.34, 40.83.77.208, 40.83.100.69, 40.83.75.165 | 13.75.110.131, 52.175.23.169, 13.75.36.64 - 13.75.36.79, 104.214.164.0 - 104.214.164.31 |
| USA Est | 13.92.98.111, 40.121.91.41, 40.114.82.191, 23.101.139.153, 23.100.29.190, 23.101.136.201, 104.45.153.81, 23.101.132.208 | 40.71.249.139, 40.71.249.205, 40.114.40.132, 40.71.11.80 - 40.71.11.95, 40.71.15.160 - 40.71.15.191 |
| USA Est 2 | 40.84.30.147, 104.208.155.200, 104.208.158.174, 104.208.140.40, 40.70.131.151, 40.70.29.214, 40.70.26.154, 40.70.27.236 | 52.225.129.144, 52.232.188.154, 104.209.247.23, 40.70.146.208 - 40.70.146.223, 40.70.151.96 - 40.70.151.127 |
| France Centre | 52.143.164.80, 52.143.164.15, 40.89.186.30, 20.188.39.105, 40.89.191.161, 40.89.188.169, 40.89.186.28, 40.89.190.104 | 40.89.186.239, 40.89.135.2, 40.79.130.208 - 40.79.130.223, 40.79.148.96 - 40.79.148.127 |
| France Sud | 52.136.132.40, 52.136.129.89, 52.136.131.155, 52.136.133.62, 52.136.139.225, 52.136.130.144, 52.136.140.226, 52.136.129.51 | 52.136.142.154, 52.136.133.184, 40.79.178.240 - 40.79.178.255, 40.79.180.224 - 40.79.180.255 |
| Allemagne Nord | 51.116.211.168, 51.116.208.165, 51.116.208.175, 51.116.208.192, 51.116.208.200, 51.116.208.222, 51.116.208.217, 51.116.208.51 | 51.116.60.192, 51.116.211.212, 51.116.59.16 - 51.116.59.31, 51.116.60.192 - 51.116.60.223 |
| Allemagne Centre-Ouest | 51.116.233.35, 51.116.171.49, 51.116.233.33, 51.116.233.22, 51.116.168.104, 51.116.175.17, 51.116.233.87, 51.116.175.51 | 51.116.158.97, 51.116.236.78, 51.116.155.80 - 51.116.155.95, 51.116.158.96 - 51.116.158.127 |
| Japon Est | 13.71.158.3, 13.73.4.207, 13.71.158.120, 13.78.18.168, 13.78.35.229, 13.78.42.223, 13.78.21.155, 13.78.20.232 | 13.73.21.230, 13.71.153.19, 13.78.108.0 - 13.78.108.15, 40.79.189.64 - 40.79.189.95 |
| OuJapon Est | 40.74.140.4, 104.214.137.243, 138.91.26.45, 40.74.64.207, 40.74.76.213, 40.74.77.205, 40.74.74.21, 40.74.68.85 | 104.215.27.24, 104.215.61.248, 40.74.100.224 - 40.74.100.239, 40.80.180.64 - 40.80.180.95 |
| Centre de la Corée | 52.231.14.11, 52.231.14.219, 52.231.15.6, 52.231.10.111, 52.231.14.223, 52.231.77.107, 52.231.8.175, 52.231.9.39 | 52.141.1.104, 52.141.36.214, 20.44.29.64 - 20.44.29.95, 52.231.18.208 - 52.231.18.223 |
| Corée du Sud | 52.231.204.74, 52.231.188.115, 52.231.189.221, 52.231.203.118, 52.231.166.28, 52.231.153.89, 52.231.155.206, 52.231.164.23 | 52.231.201.173, 52.231.163.10, 52.231.147.0 - 52.231.147.15, 52.231.148.224 - 52.231.148.255 |
| Centre-Nord des États-Unis | 168.62.248.37, 157.55.210.61, 157.55.212.238, 52.162.208.216, 52.162.213.231, 65.52.10.183, 65.52.9.96, 65.52.8.225 | 52.162.126.4, 52.162.242.161, 52.162.107.160 - 52.162.107.175, 52.162.111.192 - 52.162.111.223 |
| Europe Nord | 40.113.12.95, 52.178.165.215, 52.178.166.21, 40.112.92.104, 40.112.95.216, 40.113.4.18, 40.113.3.202, 40.113.1.181 | 52.169.28.181, 52.178.150.68, 94.245.91.93, 13.69.227.208 - 13.69.227.223, 13.69.231.192 - 13.69.231.223 |
| Afrique du Sud Nord | 102.133.231.188, 102.133.231.117, 102.133.230.4, 102.133.227.103, 102.133.228.6, 102.133.230.82, 102.133.231.9, 102.133.231.51 | 102.133.168.167, 40.127.2.94, 102.133.155.0 - 102.133.155.15, 102.133.253.0 - 102.133.253.31 |
| Afrique du Sud Ouest | 102.133.72.98, 102.133.72.113, 102.133.75.169, 102.133.72.179, 102.133.72.37, 102.133.72.183, 102.133.72.132, 102.133.75.191 | 102.133.72.85, 102.133.75.194, 102.37.64.0 - 102.37.64.31, 102.133.27.0 - 102.133.27.15 |
| États-Unis - partie centrale méridionale | 104.210.144.48, 13.65.82.17, 13.66.52.232, 23.100.124.84, 70.37.54.122, 70.37.50.6, 23.100.127.172, 23.101.183.225 | 52.171.130.92, 13.65.86.57, 13.73.244.224 - 13.73.244.255, 104.214.19.48 - 104.214.19.63 |
| Inde Sud | 52.172.50.24, 52.172.55.231, 52.172.52.0, 104.211.229.115, 104.211.230.129, 104.211.230.126, 104.211.231.39, 104.211.227.229 | 13.71.127.26, 13.71.125.22, 20.192.184.32 - 20.192.184.63, 40.78.194.240 - 40.78.194.255 |
| Asie Sud-Est | 13.76.133.155, 52.163.228.93, 52.163.230.166, 13.76.4.194, 13.67.110.109, 13.67.91.135, 13.76.5.96, 13.67.107.128 | 52.187.115.69, 52.187.68.19, 13.67.8.240 - 13.67.8.255, 13.67.15.32 - 13.67.15.63 |
| Suisse Nord | 51.103.137.79, 51.103.135.51, 51.103.139.122, 51.103.134.69, 51.103.138.96, 51.103.138.28, 51.103.136.37, 51.103.136.210 | 51.103.142.22, 51.107.86.217, 51.107.59.16 - 51.107.59.31, 51.107.60.224 - 51.107.60.255 |
| Émirats arabes unis Centre | 20.45.75.200, 20.45.72.72, 20.45.75.236, 20.45.79.239, 20.45.67.170, 20.45.72.54, 20.45.67.134, 20.45.67.135 | 20.45.67.45, 20.45.67.28, 20.37.74.192 - 20.37.74.207, 40.120.8.0 - 40.120.8.31 |
| Sud du Royaume-Uni | 51.140.74.14, 51.140.73.85, 51.140.78.44, 51.140.137.190, 51.140.153.135, 51.140.28.225, 51.140.142.28, 51.140.158.24 | 51.140.74.150, 51.140.80.51, 51.140.61.124, 51.105.77.96 - 51.105.77.127, 51.140.148.0 - 51.140.148.15 |
| Ouest du Royaume-Uni | 51.141.54.185, 51.141.45.238, 51.141.47.136, 51.141.114.77, 51.141.112.112, 51.141.113.36, 51.141.118.119, 51.141.119.63 | 51.141.52.185, 51.141.47.105, 51.141.124.13, 51.140.211.0 - 51.140.211.15, 51.140.212.224 - 51.140.212.255 |
| Centre-USA Ouest | 52.161.27.190, 52.161.18.218, 52.161.9.108, 13.78.151.161, 13.78.137.179, 13.78.148.140, 13.78.129.20, 13.78.141.75 | 52.161.101.204, 52.161.102.22, 13.78.132.82, 13.71.195.32 - 13.71.195.47, 13.71.199.192 - 13.71.199.223 |
| Europe Ouest | 40.68.222.65, 40.68.209.23, 13.95.147.65, 23.97.218.130, 51.144.182.201, 23.97.211.179, 104.45.9.52, 23.97.210.126 | 52.166.78.89, 52.174.88.118, 40.91.208.65, 13.69.64.208 - 13.69.64.223, 13.69.71.192 - 13.69.71.223 |
| Inde Ouest | 104.211.164.80, 104.211.162.205, 104.211.164.136, 104.211.158.127, 104.211.156.153, 104.211.158.123, 104.211.154.59, 104.211.154.7 | 104.211.189.124, 104.211.189.218, 20.38.128.224 - 20.38.128.255, 104.211.146.224 - 104.211.146.239 |
| USA Ouest | 52.160.92.112, 40.118.244.241, 40.118.241.243, 157.56.162.53, 157.56.167.147, 104.42.49.145, 40.83.164.80, 104.42.38.32 | 13.93.148.62, 104.42.122.49, 40.112.195.87, 13.86.223.32 - 13.86.223.63, 40.112.243.160 - 40.112.243.175 |
| USA Ouest 2 | 13.66.210.167, 52.183.30.169, 52.183.29.132, 13.66.210.167, 13.66.201.169, 13.77.149.159, 52.175.198.132, 13.66.246.219 | 52.191.164.250, 52.183.78.157, 13.66.140.128 - 13.66.140.143, 13.66.145.96 - 13.66.145.127 |
||||

<a name="azure-government-outbound"></a>

#### <a name="azure-government---outbound-ip-addresses"></a>Azure Government - Adresses IP sortantes

| Région | Adresse IP Logic Apps | Adresse IP des connecteurs managés |
|--------|---------------|-----------------------|
| Centre des États-Unis – US DoD | 52.182.48.215, 52.182.92.143 | 52.127.58.160 - 52.127.58.175, 52.182.54.8, 52.182.48.136, 52.127.61.192 - 52.127.61.223 |
| Gouvernement des États-Unis – Arizona | 52.244.67.143, 52.244.65.66, 52.244.65.190 | 52.127.2.160 - 52.127.2.175, 52.244.69.0, 52.244.64.91, 52.127.5.224 - 52.127.5.255 |
| Gouvernement des États-Unis – Texas | 52.238.114.217, 52.238.115.245, 52.238.117.119 | 52.127.34.160 - 52.127.34.175, 40.112.40.25, 52.238.161.225, 20.140.137.128 - 20.140.137.159 |
| Gouvernement américain - Virginie | 13.72.54.205, 52.227.138.30, 52.227.152.44 | 52.127.42.128 - 52.127.42.143, 52.227.143.61, 52.227.162.91 |
||||

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md)
* En savoir plus sur [les exemples et les scénarios courants](../logic-apps/logic-apps-examples-and-scenarios.md)
