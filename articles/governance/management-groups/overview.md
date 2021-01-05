---
title: Organiser vos ressources avec des groupes d’administration - Azure Governance
description: Découvrez les groupes d’administration, le fonctionnement des autorisations et leur utilisation.
ms.date: 11/17/2020
ms.topic: overview
ms.custom: contperf-fy21q1
ms.openlocfilehash: 9f23a279733169f17f0f82cb80aa08bfafcd45d0
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97030669"
---
# <a name="what-are-azure-management-groups"></a>Présentation des groupes d’administration Azure

Si votre organisation dispose de plusieurs abonnements, vous pouvez avoir besoin d’un moyen de gérer efficacement l’accès, les stratégies et la conformité de ces abonnements. Les groupes d’administration Azure fournissent un niveau d’étendue au-delà des abonnements. Vous organisez les abonnements en conteneurs appelés « groupes d’administration » et vous appliquez vos conditions de gouvernance aux groupes d’administration. Tous les abonnements d’un groupe d’administration héritent automatiquement des conditions appliquées à ce groupe d’administration. Les groupes d’administration vous permettent une gestion de qualité professionnelle à grande échelle, quel que soit le type de vos abonnements.
Tous les abonnements au sein d’un même groupe d’administration doivent approuver le même locataire Azure Active Directory.

Par exemple, vous pouvez appliquer des stratégies à un groupe d’administration afin de limiter les régions disponibles pour la création de machines virtuelles. Une telle stratégie s’appliquerait alors à tous les groupes d’administration, abonnements et ressources sous ce groupe d’administration en autorisant uniquement la création de machines virtuelles dans une région donnée.

## <a name="hierarchy-of-management-groups-and-subscriptions"></a>Hiérarchie des groupes d’administration et des abonnements

Vous pouvez créer une structure flexible de groupes d’administration et d’abonnements pour organiser vos ressources dans une hiérarchie à des fins de stratégie unifiée et de gestion de l’accès. Le diagramme suivant montre un exemple de création d’une hiérarchie pour la gouvernance à l’aide des groupes d’administration.

:::image type="complex" source="./media/tree.png" alt-text="Schéma d’un exemple de hiérarchie de groupes d’administration." border="false":::
   Schéma d’un groupe d’administration racine contenant des groupes d’administration et des abonnements. Certains groupes d’administration enfants comportent des groupes d’administration, d’autres des abonnements et d’autres les deux. L’un des exemples dans l’exemple de hiérarchie correspond à quatre niveaux de groupes d’administration, le niveau enfant étant tous les abonnements.
:::image-end:::

Vous pouvez créer une hiérarchie qui applique une stratégie, par exemple, qui limite les emplacements de machines virtuelles à la région USA Ouest dans le groupe appelé « Production ». Cette stratégie est héritée par tous les abonnements Contrat Entreprise descendants de ce groupe d’administration et s’applique à toutes les machines virtuelles dans ces abonnements. Cette stratégie de sécurité ne peut pas être modifiée par le propriétaire de ressources ou d’abonnement permettant une gouvernance améliorée.

Un autre scénario où vous pouvez utiliser les groupes d’administration consiste à fournir un accès utilisateur à plusieurs abonnements. En déplaçant plusieurs abonnements sous ce groupe d’administration, vous pouvez créer une [attribution de rôle Azure](../../role-based-access-control/overview.md) sur le groupe d’administration, qui héritera de cet accès à tous les abonnements. Une affectation sur le groupe d’administration peut autoriser les utilisateurs à accéder à tout ce dont ils ont besoin, au lieu de créer un script Azure RBAC sur différents abonnements.

### <a name="important-facts-about-management-groups"></a>Faits importants sur les groupes d’administration

- 10 000 groupes d’administration peuvent être pris en charge dans un seul annuaire.
- Une arborescence de groupes d’administration peut prendre en charge jusqu’à six niveaux de profondeur.
  - Cette limite n’inclut ni le niveau racine ni le niveau d’abonnement.
- Chaque groupe d’administration et chaque abonnement ne prennent en charge qu’un seul parent.
- Chaque groupe d’administration peut avoir de nombreux enfants.
- Dans chaque annuaire, tous les abonnements et groupes d’administration sont dans une même hiérarchie. Consultez [Faits importants sur le groupe d’administration racine](#important-facts-about-the-root-management-group).

## <a name="root-management-group-for-each-directory"></a>Groupe d’administration racine pour chaque annuaire

Chaque annuaire reçoit un groupe d’administration de niveau supérieur unique appelé groupe d’administration « racine ». Ce groupe d’administration racine est intégré à la hiérarchie et contient tous les groupes d’administration et abonnements. Il permet d’appliquer des stratégies globales et des affectations de rôles Azure au niveau de l’annuaire. L’[administrateur général d’Azure AD doit élever ses privilèges](../../role-based-access-control/elevate-access-global-admin.md) au rôle Administrateur d’accès utilisateur de ce groupe racine au départ. Une fois l’accès obtenu, l’administrateur peut attribuer un rôle Azure à d’autres utilisateurs ou groupes de l’annuaire pour gérer la hiérarchie. En tant qu’administrateur, vous pouvez attribuer votre propre compte comme propriétaire du groupe d’administration racine.

### <a name="important-facts-about-the-root-management-group"></a>Faits importants sur le groupe d’administration racine

- Par défaut, le nom d’affichage du groupe d’administration racine est **Groupe racine de locataire**. L’ID est l’ID Azure Active Directory.
- Pour changer le nom d’affichage, votre compte doit avoir le rôle Propriétaire ou Contributeur sur le groupe d’administration racine. Pour savoir comment mettre à jour le nom du groupe d’administration, consultez [Changer le nom d’un groupe d’administration](manage.md#change-the-name-of-a-management-group).
- Le groupe d’administration racine ne peut pas être déplacé ni supprimé, contrairement aux autres groupes d’administration.  
- Tous les abonnements et groupes d’administration sont contenus dans le groupe d’administration racine de l’annuaire.
  - Toutes les ressources de l’annuaire sont contenues dans le groupe d’administration racine à des fins de gestion globale.
  - Lors de leur création, les nouveaux abonnements sont attribués par défaut au groupe d’administration racine.
- Tous les clients Azure peuvent voir le groupe d’administration racine, mais tous ne peuvent pas le gérer.
  - Toute personne ayant accès à un abonnement peut voir où celui-ci se trouve dans la hiérarchie.  
  - Personne ne reçoit par défaut l’accès au groupe d’administration racine. Les administrateurs généraux Azure AD sont les seuls utilisateurs à pouvoir élever leurs privilèges pour obtenir l’accès. Une fois qu’ils ont accès au groupe d’administration racine, les administrateurs généraux peuvent attribuer un rôle Azure aux autres utilisateurs pour  
    le gérer.
- Dans le SDK, le groupe d’administration racine, ou « racine de locataire », fonctionne comme un groupe d’administration.

> [!IMPORTANT]
> Les attributions d’accès utilisateur et de stratégies effectuées au niveau du groupe d’administration racine **s’appliquent à toutes les ressources de l’annuaire**. Pour cette raison, tous les utilisateurs doivent évaluer la nécessité de définir des ressources dans cette étendue. Les attributions d’accès utilisateur et de stratégies ne doivent être obligatoires que pour cette  
> étendue.

## <a name="initial-setup-of-management-groups"></a>Configuration initiale des groupes d’administration

Lorsqu’un utilisateur commence à utiliser les groupes d’administration, un processus de configuration initiale est lancé. La première étape est la création du groupe d’administration racine dans l’annuaire. Une fois le groupe créé, tous les abonnements existants de l’annuaire deviennent des enfants du groupe d’administration racine.
Ce processus permet de faire en sorte que l’annuaire ne contienne qu’une seule hiérarchie de groupes gestion. Le fait de n’avoir qu’une seule hiérarchie par annuaire permet aux administrateurs d’appliquer un accès global et des stratégies que les autres utilisateurs de l’annuaire ne peuvent pas contourner. Tout élément attribué au niveau racine est appliqué à toute la hiérarchie, qui comprend tous les groupes d’administration, les abonnements, les groupes de ressources et les ressources de ce locataire Azure AD.

## <a name="trouble-seeing-all-subscriptions"></a>Difficultés pour afficher tous les abonnements

Quelques annuaires ayant commencé à utiliser des groupes d’administration de manière anticipée dans la préversion antérieure au 25 juin 2018 ont rencontré un problème où tous les abonnements n’étaient pas dans la hiérarchie. Le processus permettant d’avoir tous les abonnements dans la hiérarchie a été mis en place après l’attribution d’un rôle ou d’une stratégie au groupe d’administration racine de l’annuaire.

### <a name="how-to-resolve-the-issue"></a>Que pouvez-vous faire pour résoudre le problème ?

Pour résoudre ce problème, deux options s’offrent à vous.

- Supprimer toutes les affectations de rôle et de stratégie du groupe d’administration racine
  - En supprimant toutes les affectations de stratégie et de rôle du groupe d’administration racine, le service renvoie tous les abonnements dans la hiérarchie du prochain cycle pendant la nuit. Ce processus permet de garantir qu’aucun accès n’est donné et qu’aucune stratégie n’est affectée accidentellement à tous les abonnements de locataires.
  - La meilleure méthode pour effectuer ce processus sans affecter vos services consiste à appliquer les affectations de rôle ou de stratégie à un niveau en dessous du groupe d’administration racine. Vous pouvez ensuite supprimer toutes les affectations au niveau de la racine.
- Appeler l’API directement afin de démarrer le processus de renvoi
  - Tous les clients de l’annuaire peuvent appeler les API _TenantBackfillStatusRequest_ ou _StartTenantBackfillRequest_. Lorsque l’API StartTenantBackfillRequest est appelée, elle lance le processus de configuration initiale de déplacement de tous les abonnements dans la hiérarchie. Ce processus démarre également l’application de tous les nouveaux abonnements en tant qu’enfants du groupe d’administration racine.
    Ce processus peut être effectué sans changer d’affectation au niveau racine. En appelant l’API, vous dites qu’une affectation de stratégie ou d’accès à la racine peut être appliquée à tous les abonnements.

Si vous avez des questions sur ce processus de renvoi, contactez : `managementgroups@microsoft.com`
  
## <a name="management-group-access"></a>Accès aux groupes d’administration

Les groupes d’administration Azure prennent en charge le [contrôle d’accès en fonction du rôle Azure (RBAC Azure)](../../role-based-access-control/overview.md) pour tous les accès aux ressources et toutes les définitions de rôles.
Les ressources enfants qui existent dans la hiérarchie héritent de ces autorisations. Vous pouvez attribuer n’importe quel rôle Azure à un groupe d’administration, qui héritera ensuite de la hiérarchie des ressources. Par exemple, un contributeur de machine virtuelle avec un rôle Azure peut être affecté à un groupe d’administration. Ce rôle n’a aucun effet sur le groupe d’administration, mais il hérite de toutes les machines virtuelles situées sous ce groupe d’administration.

Le graphique suivant montre la liste des rôles, ainsi que les actions prises en charge par les groupes d’administration.

| Nom du rôle Azure             | Créer | Renommer | Déplacer\*\* | DELETE | Attribuer l’accès | Attribuer la stratégie | Lire  |
|:-------------------------- |:------:|:------:|:--------:|:------:|:-------------:| :------------:|:-----:|
|Propriétaire                       | X      | X      | X        | X      | X             | X             | X     |
|Contributeur                 | X      | X      | X        | X      |               |               | X     |
|Contributeur MG\*            | X      | X      | X        | X      |               |               | X     |
|Lecteur                      |        |        |          |        |               |               | X     |
|Lecteur MG\*                 |        |        |          |        |               |               | X     |
|Contributeur de la stratégie de ressource |        |        |          |        |               | X             |       |
|Administrateur de l'accès utilisateur   |        |        |          |        | X             | X             |       |

\*: Contributeur MG et lecteur MG autorisent uniquement les utilisateurs à effectuer ces actions sur l’étendue du groupe d’administration.  
\*\* : Les attributions de rôles sur le groupe d’administration racine ne sont pas nécessaires pour déplacer un abonnement ou un groupe d’administration. Consultez [Gérer vos ressources avec des groupes d’administration](manage.md) pour des détails sur le déplacement d’éléments dans la hiérarchie.

## <a name="azure-custom-role-definition-and-assignment"></a>Définition et attribution d’un rôle personnalisé Azure

La prise en charge des rôles personnalisés Azure pour les groupes d’administration est en préversion et comprend certaines [limitations](#limitations). Vous pouvez définir l’étendue d’un groupe d’administration dans l’étendue attribuable de la définition de rôle. Ce rôle personnalisé Azure est alors attribuable dans ce groupe d’administration ainsi que tout groupe d’administration, abonnement, groupe de ressources ou ressource dont il est parent. Ce rôle personnalisé hérite ensuite la hiérarchie comme n’importe quel rôle intégré.  

### <a name="example-definition"></a>Exemple de définition

Le processus de [définition et création d’un rôle personnalisé](../../role-based-access-control/custom-roles.md) ne change pas avec l’inclusion de groupes d’administration. Spécifiez le chemin complet pour définir le groupe d’administration **/providers/Microsoft.Management/managementgroups/{groupId}** .

Utilisez l’ID du groupe d’administration et non le nom d’affichage du groupe d’administration. Cette erreur courante est due au fait qu’il s’agit de deux champs personnalisés définis au moment de la création d’un groupe d’administration.

```json
...
{
  "Name": "MG Test Custom Role",
  "Id": "id", 
  "IsCustom": true,
  "Description": "This role provides members understand custom roles.",
  "Actions": [
    "Microsoft.Management/managementgroups/delete",
    "Microsoft.Management/managementgroups/read",
    "Microsoft.Management/managementgroup/write",
    "Microsoft.Management/managementgroup/subscriptions/delete",
    "Microsoft.Management/managementgroup/subscriptions/write",
    "Microsoft.resources/subscriptions/read",
    "Microsoft.Authorization/policyAssignments/*",
    "Microsoft.Authorization/policyDefinitions/*",
    "Microsoft.Authorization/policySetDefinitions/*",
    "Microsoft.PolicyInsights/*",
    "Microsoft.Authorization/roleAssignments/*",
    "Microsoft.Authorization/roledefinitions/*"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
        "/providers/microsoft.management/managementGroups/ContosoCorporate"
  ]
}
...
```

### <a name="issues-with-breaking-the-role-definition-and-assignment-hierarchy-path"></a>Problèmes de chemin rompu entre la définition de rôle et l’attribution de rôle

L’étendue attribuable aux définitions de rôles peut être n’importe où dans la hiérarchie d’un groupe d’administration. Une définition de rôle peut être définie sur un groupe d’administration parent alors que l’attribution de rôle réelle existe sur l’abonnement enfant. Comme ces deux éléments sont liés, une erreur se produit si vous tentez de séparer l’attribution de sa définition.

Par exemple, examinons une petite section d’une hiérarchie pour un visuel.

:::image type="complex" source="./media/subtree.png" alt-text="Diagramme d’un sous-ensemble de l’exemple de hiérarchie de groupes d’administration." border="false":::
   Le diagramme se concentre sur le groupe d’administration racine avec les groupes d’administration enfants Informatique et Marketing. Le groupe d’administration Informatique possède un seul groupe d’administration enfant, nommé production, tandis que le groupe d’administration Marketing dispose de deux abonnements enfants d’essai gratuit.
:::image-end:::

Prenons l’exemple d’un rôle personnalisé défini sur le groupe d’administration Marketing. Ce rôle personnalisé est ensuite attribué dans les deux abonnements d’essai gratuit.  

Si nous tentons de déplacer l’un de ces abonnements pour qu’il devienne enfant du groupe d’administration Production, ce déplacement rompt le chemin entre l’attribution de rôle de l’abonnement et la définition de rôle du groupe d’administration Marketing. Dans ce scénario, vous recevez une erreur indiquant que le déplacement n’est pas autorisé, car il rompt cette relation.  

Il existe plusieurs solutions pour corriger ce scénario :
- Supprimez l’attribution de rôle de l’abonnement avant de déplacer ce dernier vers un autre groupe d’administration parent.
- Ajoutez l’abonnement dans l’étendue attribuable de la définition de rôle.
- Changez l’étendue attribuable dans la définition de rôle. Dans l’exemple ci-dessus, vous pouvez changer les étendues attribuables du groupe d’administration Marketing vers le groupe d’administration racine afin que la définition soit disponible dans les deux branches de la hiérarchie.  
- Créez un rôle personnalisé supplémentaire qui sera défini dans l’autre branche. Pour ce nouveau rôle, vous devrez également changer l’attribution de rôle dans l’abonnement.  

### <a name="limitations"></a>Limites  

Certaines limitations s’appliquent quand vous utilisez des rôles personnalisés dans des groupes d’administration. 

 - Vous pouvez définir un seul groupe d’administration dans les étendues attribuables d’un nouveau rôle. Cette limitation vise à réduire le nombre de situations où la relation entre les définitions de rôles et les attributions de rôles est rompue. Cette situation se produit quand un abonnement ou un groupe d’administration comportant une attribution de rôle est déplacé vers un autre parent dépourvu de la définition de rôle.  
 - Il n’est pas possible de définir les actions du plan de données du fournisseur de ressources dans des rôles personnalisés de groupe d’administration. Cette restriction s’explique par un problème de latence avec la mise à jour des fournisseurs de ressources du plan de données.
   Nous travaillons actuellement sur ce problème de latence ; ces actions seront désactivées de la définition de rôle pour réduire les risques.
 - Azure Resource Manager ne valide pas le groupe d’administration existant dans l’étendue attribuable de la définition de rôle. Même si vous avez fait une faute de frappe ou indiqué un ID de groupe d’administration incorrect, la définition de rôle est créée.

> [!IMPORTANT]
> L’ajout d’un groupe d’administration à `AssignableScopes` est actuellement en préversion. Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production.
> Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="moving-management-groups-and-subscriptions"></a>Déplacement des groupes d’administration et des abonnements 

Pour déplacer un groupe d’administration ou un abonnement de sorte qu’il devienne l’enfant d’un autre groupe d’administration, trois règles doivent être remplies.

Pour effectuer le déplacement, vous devez avoir : 

- Les autorisations en écriture pour le groupe d’administration et l’attribution de rôle dans l’abonnement ou le groupe d’administration enfant.
  - Un rôle intégré, par exemple, **Propriétaire**
- L’accès en écriture au groupe d’administration dans le groupe d’administration parent cible.
  - Un rôle intégré, par exemple, **Propriétaire**, **Contributeur**, **Contributeur du groupe d’administration**
- L’accès en écriture au groupe d’administration dans le groupe d’administration parent existant.
  - Un rôle intégré, par exemple, **Propriétaire**, **Contributeur**, **Contributeur du groupe d’administration**

**Exception** : Si le groupe d'administration parent cible ou existant correspond au groupe d'administration racine, les exigences en matière d'autorisations ne s'appliquent pas. Le groupe d’administration racine correspondant à l'emplacement de destination de tous les nouveaux groupes d’administration et abonnements, vous ne devez pas disposer d'autorisations sur ce dernier pour déplacer un élément.

Si le rôle Propriétaire de l'abonnement est hérité du groupe d’administration actuel, vos cibles de déplacement sont limitées. Vous pouvez uniquement déplacer l’abonnement vers un autre groupe d’administration pour lequel vous détenez le rôle Propriétaire. Vous ne pouvez pas le déplacer vers un groupe d’administration pour lequel vous détenez un rôle Contributeur si vous perdez la propriété de l’abonnement. Si vous vous voyez attribuer directement le rôle Propriétaire de l'abonnement (non hérité du groupe d’administration), vous pouvez le déplacer vers un groupe d’administration au sein duquel vous détenez un rôle Contributeur.

## <a name="audit-management-groups-using-activity-logs"></a>Auditer les groupes d’administration à l’aide des journaux d’activité

Les groupes d’administration sont pris en charge dans le [journal d’activité Azure](../../azure-monitor/platform/platform-logs-overview.md). Vous pouvez rechercher dans tous les événements qui se produisent dans un groupe d’administration au même emplacement central, tout comme d’autres ressources Azure. Par exemple, vous pouvez voir tous les changements d’attributions de rôles ou de stratégie apportés à un groupe d’administration spécifique.

:::image type="content" source="./media/al-mg.png" alt-text="Capture d’écran du journal d’activité et des opérations associées au groupe d’administration sélectionné." border="false":::

Quand vous cherchez à interroger les groupes d’administration en dehors du portail Azure, l’étendue cible pour les groupes d’administration ressemble à **"/providers/Microsoft.Management/managementGroups/{yourMgID}"** .

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les groupes d’administration, consultez :

- [Créer des groupes d’administration pour organiser les ressources Azure](./create-management-group-portal.md)
- [Guide pratique pour modifier, supprimer ou gérer vos groupes d’administration](./manage.md)
- Découvrez les différentes options permettant de [protéger votre hiérarchie de ressources](./how-to/protect-resource-hierarchy.md)