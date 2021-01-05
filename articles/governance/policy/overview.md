---
title: Présentation de la stratégie Azure
description: Azure Policy est un service dans Azure, que vous utilisez pour créer, affecter et gérer les définitions de stratégie dans votre environnement Azure.
ms.date: 10/05/2020
ms.topic: overview
ms.openlocfilehash: 8a32e32afb544588bb033cc64ede5ecbe6e2bac2
ms.sourcegitcommit: 93329b2fcdb9b4091dbd632ee031801f74beb05b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92097386"
---
# <a name="what-is-azure-policy"></a>Présentation d’Azure Policy

Azure Policy aide à appliquer les normes organisationnelles et à évaluer la conformité à l’échelle. Avec son tableau de bord de conformité, il fournit une vue agrégée permettant d’évaluer l’état général de l’environnement, avec la possibilité d’explorer au niveau de chaque ressource et stratégie. Il vous aide également à mettre vos ressources en conformité par le biais de la correction en bloc pour les ressources existantes et de la correction automatique pour les nouvelles ressources.

Les cas d’usage courants pour Azure Policy incluent la mise en œuvre de la gouvernance pour la cohérence des ressources, la conformité réglementaire, la sécurité, le coût et la gestion. Les définitions de stratégie pour ces cas d’usage courants sont déjà disponibles dans votre environnement Azure comme éléments intégrés pour vous aider à démarrer.

La totalité des données et objets Azure Policy sont chiffrés au repos. Pour plus d’informations, consultez [Chiffrement des données Azure au repos](../../security/fundamentals/encryption-atrest.md).

## <a name="overview"></a>Vue d’ensemble

Azure Policy évalue les ressources dans Azure en comparant les propriétés de ces ressources aux règles d’entreprise. Ces règles d’entreprise, décrites au [format JSON](./concepts/definition-structure.md), sont appelées [définitions de stratégie](#policy-definition). Pour simplifier la gestion, plusieurs règles d’entreprise peuvent être regroupées pour former une [initiative de stratégie](#initiative-definition) (parfois appelée _policySet_ ). Une fois que vos règles d’entreprise ont été formées, l’initiative ou la définition de stratégie est [affectée](#assignments) à n’importe quelle étendue de ressources prise en charge par Azure, comme des [groupes d’administration](../management-groups/overview.md), des abonnements, des [groupes de ressources](../../azure-resource-manager/management/overview.md#resource-groups) ou des ressources individuelles. L’affectation s’applique à toutes les ressources au sein de l’[étendue Resource Manager](../../azure-resource-manager/management/overview.md#understand-scope) de cette affectation. Des sous-étendues peuvent être exclues, si nécessaire. Pour plus d’informations, consultez [Étendue d’Azure Policy](./concepts/scope.md).

Azure Policy utilise un [format JSON](./concepts/definition-structure.md) afin de former la logique utilisée par l’évaluation pour déterminer si une ressource est conforme ou non. Les définitions incluent des métadonnées et la règle de stratégie. La règle définie peut utiliser des fonctions, des paramètres, des opérateurs logiques, des conditions et des [alias](./concepts/definition-structure.md#aliases) de propriété afin d’établir une correspondance exacte au scénario de votre choix. La règle de stratégie détermine quelles ressources de l’étendue de l’affectation sont évaluées.

### <a name="understand-evaluation-outcomes"></a>Comprendre les résultats de l’évaluation

Les ressources sont évaluées à des moments spécifiques pendant le cycle de vie des ressources, le cycle de vie d’affectation de stratégie, et en vue d’une évaluation régulière et continue de la conformité. Voici les moments ou les événements qui provoquent l’évaluation d’une ressource :

- Création, mise à jour ou suppression d’une ressource dans une étendue avec une affectation de stratégie
- Affectation d’une nouvelle stratégie ou initiative à une étendue.
- Mise à jour d’une stratégie ou initiative déjà assignée à une étendue.
- Lors du cycle d’évaluation de la conformité standard, qui se produit une fois toutes les 24 heures.

Pour plus d’informations sur le moment et la façon dont l’évaluation de la stratégie se produit, consultez [Déclencheurs d’évaluation](./how-to/get-compliance-data.md#evaluation-triggers).

### <a name="control-the-response-to-an-evaluation"></a>Contrôler la réponse à une évaluation

Les règles d’entreprise pour la gestion des ressources non conformes varient considérablement d’une organisation à l’autre. Voici des exemples de la façon dont une organisation peut souhaiter que la plateforme réponde à une ressource non conforme :

- Refus de la modification de la ressource
- Enregistrement de la modification apportée à la ressource
- Changement de la ressource avant la modification
- Changement de la ressource après la modification
- Déploiement de ressources conformes associées

Azure Policy rend possible chacune de ces réponses métier par le biais de l’application d’[effets](./concepts/effects.md). Les effets sont définis dans la partie **règle de stratégie**  de la [définition de stratégie](./concepts/definition-structure.md).

### <a name="remediate-non-compliant-resources"></a>Remédier aux ressources non conformes

Bien que ces effets affectent principalement une ressource lors de sa création ou de sa mise à jour, Azure Policy prend également en charge la gestion des ressources non conformes existantes sans avoir à modifier ces ressources. Pour plus d’informations sur la façon de rendre les ressources existantes conformes, consultez [Corriger les ressources non conformes avec Azure Policy](./how-to/remediate-resources.md).

### <a name="video-overview"></a>Présentation vidéo

La présentation suivante d’Azure Policy est à partir de la Build 2018. Pour le téléchargement des diapositives ou de la vidéo, accédez à [Govern your Azure environment through Azure Policy](https://channel9.msdn.com/events/Build/2018/THR2030) (Gouvernance de votre environnement Azure à l’aide d’Azure Policy) sur Channel 9.

> [!VIDEO https://www.youtube.com/embed/dxMaYF2GB7o]

## <a name="getting-started"></a>Prise en main

### <a name="azure-policy-and-azure-rbac"></a>Azure Policy et Azure RBAC

Il existe quelques différences importantes entre Azure Policy et le contrôle d’accès en fonction du rôle Azure (Azure RBAC). Azure Policy évalue l’état en examinant les propriétés des ressources qui sont représentées dans Resource Manager et les propriétés de certains fournisseurs de ressources. Azure Policy ne limite pas les actions (également appelées _opérations_ ). Azure Policy garantit que l’état des ressources est conforme à vos règles d’entreprise sans se préoccuper de qui a apporté la modification ou qui a l’autorisation d’apporter une modification.

Azure RBAC est axé sur la gestion des [actions](../../role-based-access-control/resource-provider-operations.md) des utilisateurs dans différentes étendues. Si le contrôle d’une action est nécessaire, Azure RBAC est l’outil qu’il convient d’utiliser. Même si une personne est autorisée à exécuter une action, si le résultat est une ressource non conforme, Azure Policy bloque toujours la création ou la mise à jour.

La combinaison d’Azure RBAC et d’Azure Policy fournit un contrôle d’étendue complète dans Azure.

### <a name="azure-rbac-permissions-in-azure-policy"></a>Autorisations Azure RBAC dans Azure Policy

Azure Policy dispose d’autorisations, aussi appelées opérations, dans deux fournisseurs de ressources :

- [Microsoft.Authorization](../../role-based-access-control/resource-provider-operations.md#microsoftauthorization)
- [Microsoft.PolicyInsights](../../role-based-access-control/resource-provider-operations.md#microsoftpolicyinsights)

Plusieurs rôles intégrés accordent des autorisations aux ressources Azure Policy. Le rôle **Contributeur de stratégie de ressource** inclut la plupart des opérations d’Azure Policy. Quant au rôle **Propriétaire** , il dispose de tous les droits. Les rôles **Contributeur** et **Lecteur** ont accès à toutes les opérations de _lecture_ Azure Policy. Un **Contributeur** peut déclencher la correction d’une ressource, mais ne peut pas _créer_ des définitions ou des affectations. Un **Administrateur de l’accès utilisateur** est nécessaire pour accorder les autorisations nécessaires à l’identité managée sur les affectations **deployIfNotExists** ou **modify** .

Si aucun des rôles intégrés ne dispose d’autorisations, créez un [rôle personnalisé](../../role-based-access-control/custom-roles.md).

> [!NOTE]
> L’identité managée d’une affectation de stratégie **deployIfNotExists** ou **modify** a besoin d’autorisations suffisantes pour créer ou mettre à jour les ressources ciblées. Pour plus d’informations, consultez [Configurer une définition de stratégie pour la correction](./how-to/remediate-resources.md#configure-policy-definition).

### <a name="resources-covered-by-azure-policy"></a>Ressources couvertes par Azure Policy

Azure Policy évalue toutes les ressources dans Azure et toutes les ressources Arc. Pour certains fournisseurs de ressources tels que [Guest Configuration](./concepts/guest-configuration.md), [Azure Kubernetes Service](../../aks/intro-kubernetes.md) et [Azure Key Vault](../../key-vault/general/overview.md), il existe une intégration plus poussée pour la gestion des paramètres et des objets. Pour en savoir plus, consultez [Modes Fournisseur de ressources](./concepts/definition-structure.md).

### <a name="recommendations-for-managing-policies"></a>Recommandations pour la gestion des stratégies

Voici quelques conseils et astuces à garder à l’esprit :

- Commencez avec un effet d’audit plutôt qu’un effet de refus pour suivre l’impact de la définition de votre stratégie sur votre environnement. Si vous avez déjà des scripts en place pour mettre automatiquement à l’échelle vos applications, la définition d’un effet de refus peut entraver ces tâches d’automatisation déjà en place.

- Tenez compte des hiérarchies de l’organisation lors de la création de définitions et d’affectations. Nous vous recommandons de créer des définitions à des niveaux supérieurs, comme au niveau de l’abonnement ou du groupe d’administration. Ensuite, créez l’affectation au niveau enfant suivant. Si vous créez une définition au niveau d’un groupe d’administration, l’affectation peut être limitée à un abonnement ou groupe de ressources au sein de ce groupe d’administration.

- Nous vous recommandons de créer et d’affecter des définitions d’initiative même pour une définition de stratégie unique.
  Par exemple, vous avez la définition de stratégie _policyDefA_ et la créez sous la définition d’initiative _initiativeDefC_ . Si vous créez une autre définition de stratégie ultérieurement pour _policyDefB_ avec des objectifs similaires à _policyDefA_ , vous pouvez l’ajouter sous _initiativeDefC_ et les suivre ensemble.

- Une fois que vous avez créé une affectation d’initiative, les définitions de stratégie ajoutées à l’initiative font également partie des affectations de cette initiative.

- Lors de l’évaluation d’une affectation d’initiative, toutes les stratégies dans l’initiative sont également évaluées.
  Si vous devez évaluer une stratégie individuellement, il est préférable de ne pas l’inclure dans une initiative.

## <a name="azure-policy-objects"></a>Objets Azure Policy

### <a name="policy-definition"></a>Définition de stratégie

Pour créer et implémenter une stratégie dans Azure Policy, il faut commencer par créer une définition de stratégie. Chaque définition de stratégie présente des conditions dans lesquelles elle est appliquée. Si les conditions sont remplies, un effet défini se produit.

Dans Azure Policy, nous proposons plusieurs stratégies intégrées qui sont disponibles par défaut. Par exemple :

- **Références SKU de compte de stockage autorisées** (Refuser) : Détermine si un compte de stockage en cours de déploiement se trouve dans un ensemble de tailles de référence SKU. Son effet consiste à refuser tous les comptes de stockage dont la taille ne fait pas partie de l’ensemble de tailles de référence SKU définies.
- **Type de ressource autorisé** (Refuser) : Définit les types de ressources que vous pouvez déployer. Son effet consiste à refuser toutes les ressources qui ne font pas partie de cette liste définie.
- **Emplacements autorisés** (Refuser) : Restreint les emplacements disponibles pour les nouvelles ressources. Son effet permet d’appliquer vos exigences de conformité géographique.
- **Références SKU de machine virtuelle autorisées** (Refuser) : Spécifie un ensemble de références SKU de machine virtuelle que vous pouvez déployer.
- **Ajouter une étiquette aux ressources** (Modifier) : Applique une balise requise et sa valeur par défaut si elle n’est pas spécifiée par la requête de déploiement.
- **Types de ressources non autorisés** (Refuser) : Empêche une liste de types de ressources d’être déployés.

Pour implémenter ces définitions de stratégie (définitions intégrées et personnalisées), vous devez les affecter. Vous pouvez affecter l’une de ces stratégies par le biais du portail Azure, de PowerShell ou d’Azure CLI.

Une évaluation de la stratégie a lieu avec plusieurs actions différentes comme l’affectation de stratégie ou les mises à jour de stratégie. Pour obtenir la liste complète, consultez [Policy evaluation triggers](./how-to/get-compliance-data.md#evaluation-triggers) (Déclencheurs d’évaluation de stratégie).

Pour plus d’informations sur les structures des définitions de stratégie, consultez [Structure des définitions de stratégie](./concepts/definition-structure.md).

Les paramètres de stratégie permettent de simplifier la gestion des stratégies en réduisant le nombre de définitions de stratégies que vous devez créer. Vous pouvez définir des paramètres lors de la création d’une définition de stratégie, pour la rendre plus générique. Vous pouvez ensuite réutiliser cette définition de stratégie pour différents scénarios. Pour ce faire, transmettez différentes valeurs lors de l’affectation de la définition de stratégie. Par exemple, spécifiez un ensemble d’emplacements pour un abonnement.

Les paramètres sont définis lors de la création d’une définition de stratégie. Quand un paramètre est défini, un nom lui est affecté et éventuellement une valeur. Par exemple, vous pouvez définir un paramètre pour une stratégie intitulée _Emplacement_ . Vous pouvez ensuite lui attribuer différentes valeurs comme _EastUS_ ou _WestUS_ lors de l’affectation d’une stratégie.

Pour plus d’informations sur les paramètres de stratégie, consultez [Definition structure - Parameters](./concepts/definition-structure.md#parameters) (Structure de la définition - Paramètres).

### <a name="initiative-definition"></a>Définition d’initiative

Une définition d’initiative est une collection de définitions de stratégie qui sont spécialement conçues pour atteindre un objectif global particulier. Les définitions d’initiative simplifient la gestion et l’affectation des définitions de stratégie. Elles simplifient ces procédures en regroupant un ensemble de stratégies en un seul élément. Par exemple, vous pouvez créer une initiative intitulée **Activer la surveillance dans Azure Security Center** , avec comme objectif de surveiller toutes les recommandations de sécurité disponibles dans votre centre Azure Security Center.

> [!NOTE]
> Le SDK, et notamment Azure CLI et Azure PowerShell, utilisent des propriétés et des paramètres nommés **PolicySet** pour référencer les initiatives.

Dans cette initiative, vous avez par exemple des définitions de stratégie comme celles-ci :

- **Surveiller les bases de données non chiffrées dans Security Center** : pour surveiller les bases de données et les serveurs SQL Server non chiffrés.
- **Surveiller les vulnérabilités du système d’exploitation dans Security Center** : pour surveiller les serveurs qui ne répondent pas à la ligne de base configurée.
- **Surveiller l’absence d’Endpoint Protection dans Security Center** : pour surveiller les serveurs où un agent Endpoint Protection n’est pas installé.

Comme les paramètres de stratégie, les paramètres d’initiative permettent de simplifier la gestion en réduisant la redondance. Les paramètres d’initiative sont les paramètres utilisés par les définitions de stratégie dans l’initiative.

Par exemple, imaginons un scénario où vous avez une définition d’initiative ( **initiativeC** ) avec les définitions de stratégie **policyA** et **policyB** , et chacune des définitions de stratégie attend un type de paramètre différent :

| Policy | Nom de paramètre |Type de paramètre  |Remarque |
|---|---|---|---|
| policyA | allowedLocations | tableau  |Ce paramètre attend une liste de chaînes pour une valeur, le type de paramètre ayant été défini comme tableau |
| policyB | allowedSingleLocation |string |Ce paramètre attend un mot pour une valeur, le type de paramètre ayant été défini comme chaîne |

Dans ce scénario, quand vous définissez les paramètres d’initiative pour **initiativeC** , vous avec trois options :

- Utilisez les paramètres des définitions de stratégie dans cette initiative : Dans cet exemple, _allowedLocations_ et _allowedSingleLocation_ deviennent des paramètres d’initiative pour **initiativeC** .
- Fournir des valeurs pour les paramètres des définitions de stratégie dans la définition de cette initiative. Dans cet exemple, vous pouvez fournir une liste d’emplacements au paramètre de **policyA** ( **allowedLocations** ) et au paramètre de **policyB** ( **allowedSingleLocation** . Vous pouvez également fournir des valeurs lors de l’affectation de cette initiative.
- Fournir une liste d’options de _valeurs_ qui peuvent être utilisées lors de l’affectation de cette initiative. Lorsque vous affectez cette initiative, les paramètres hérités des définitions de stratégie dans l’initiative peuvent avoir seulement des valeurs provenant de cette liste fournie.

Lorsque vous créez des options de valeur dans une définition d’initiative, vous ne pouvez pas entrer de valeur différente lors de l’affectation d’initiative, car elle ne fait pas partie de la liste.

Pour en savoir plus sur les structures des définitions d’initiative, passez en revue [structure des définitions d’initiative](./concepts/initiative-definition-structure.md).

### <a name="assignments"></a>Attributions

Une affectation est une initiative ou définition de stratégie qui a été affectée avec une étendue spécifique. Cette étendue peut aller d’un [groupe d’administration](../management-groups/overview.md) à une ressource individuelle. Le terme _étendue_ désigne l’ensemble des ressources, groupes de ressources, abonnements ou groupes d’administration auxquels la définition est affectée. Toutes les ressources enfants héritent des affectations. Grâce à cette structure, si une définition est appliquée à un groupe de ressources, elle est également appliquée à toutes les ressources de ce groupe. Toutefois, vous pouvez exclure une sous-étendue de l’affectation.

Par exemple, dans l’étendue de l’abonnement, vous pouvez affecter une définition qui empêche la création de ressources réseau. Vous pouvez exclure un groupe de ressources au sein de cet abonnement qui est destiné à l’infrastructure réseau. Vous accordez ensuite l’accès à ce groupe de ressources réseau aux utilisateurs auxquels vous faites confiance avec la création des ressources réseau.

Dans un autre exemple, vous souhaiterez peut-être affecter une définition de liste d’autorisation de type de ressource au niveau du groupe d’administration. Ensuite, vous affecterez une stratégie plus permissive (autorisant plus de types de ressources) à un groupe d’administration enfant ou même directement aux abonnements. Toutefois, cet exemple ne fonctionnera pas, car Azure Policy est un système de refus explicite. Au lieu de cela, vous devez exclure le groupe d’administration enfant ou l’abonnement de l’affectation au niveau du groupe d’administration. Affectez ensuite la définition plus permissive au niveau du groupe d’administration enfant ou de l’abonnement. Si une affectation se traduit par le refus d’une ressource, alors la seule façon d’autoriser la ressource consiste à modifier l’affectation de refus.

Pour plus d’informations sur les affectations par le biais du portail, consultez [Créer une affectation de stratégie pour identifier les ressources non conformes dans votre environnement Azure](./assign-policy-portal.md). Les étapes pour [PowerShell](./assign-policy-powershell.md) et [Azure CLI](./assign-policy-azurecli.md) sont également disponibles. Pour plus d’informations sur la structure des affectations, consultez [Structure des affectations](./concepts/assignment-structure.md).

## <a name="maximum-count-of-azure-policy-objects"></a>Nombre maximal d’objets Azure Policy

[!INCLUDE [policy-limits](../../../includes/azure-policy-limits.md)]

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez une vue d’ensemble d’Azure Policy et des autres concepts clés, voici les étapes suivantes que nous suggérons :

- [Examiner la structure de définition de stratégie](./concepts/definition-structure.md)
- [Affecter une définition de stratégie à l’aide du portail](./assign-policy-portal.md).
