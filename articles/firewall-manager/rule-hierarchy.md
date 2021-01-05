---
title: Utiliser la stratégie de pare-feu Azure pour définir une hiérarchie de règles
description: Apprenez à utiliser la stratégie de pare-feu Azure pour définir une hiérarchie de règles et garantir la conformité.
services: firewall-manager
author: vhorne
ms.service: firewall-manager
ms.topic: how-to
ms.date: 08/26/2020
ms.author: victorh
ms.openlocfilehash: 1ba683e3d616f52854f1055dab9b9fe2d389116a
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92331734"
---
# <a name="use-azure-firewall-policy-to-define-a-rule-hierarchy"></a>Utiliser la stratégie de pare-feu Azure pour définir une hiérarchie de règles

Les administrateurs de la sécurité doivent gérer les pare-feu et garantir la conformité des déploiements localement et dans le cloud. Pour ce faire, ils doivent être en mesure de fournir aux équipes d'applications la flexibilité nécessaire à l'implémentation de pipelines CI/CD afin de créer des règles de pare-feu de manière automatisée.

La stratégie de pare-feu Azure vous permet de définir une hiérarchie de règles et de garantir la conformité :

- Elle fournit une structure hiérarchique permettant de superposer une stratégie de base centrale sur une stratégie d'équipe d'applications enfant. La stratégie de base est prioritaire et s'exécute avant la stratégie enfant.
- Utilisez une définition d’un rôle personnalisé Azure pour empêcher toute suppression involontaire de la stratégie de base et fournir un accès sélectif aux regroupements de règles au sein d'un abonnement ou d'un groupe de ressources. 

## <a name="solution-overview"></a>Vue d’ensemble de la solution

Dans le cadre de cet exemple, les principales étapes sont les suivantes :

1. Créez une stratégie de pare-feu de base dans le groupe de ressources de l'équipe de sécurité. 
3. Définissez des règles spécifiques à la sécurité informatique dans la stratégie de base. Un ensemble de règles communes est ainsi ajouté pour autoriser/refuser le trafic.
4. Créez des stratégies d'équipe d'applications qui héritent de la stratégie de base. 
5. Définissez des règles spécifiques à l'équipe d'applications dans la stratégie. Vous pouvez également migrer des règles à partir de pare-feu préexistants.
6. Créez des rôles Azure Active Directory personnalisés pour fournir un accès affiné au regroupement de règles et ajoutez des rôles dans une étendue Stratégie de pare-feu. Dans l'exemple suivant, les membres de l'équipe Ventes peuvent modifier les regroupements de règles correspondant à la stratégie de pare-feu des équipes Ventes. Il en va de même pour les équipes Base de données et Ingénierie.
7. Associez la stratégie au pare-feu correspondant. Une seule stratégie peut être attribuée à un pare-feu Azure. Chaque équipe d'applications doit donc disposer de son propre pare-feu.



:::image type="content" source="media/rule-hierarchy/contoso-teams.png" alt-text="Équipes et exigences" border="false":::

### <a name="create-the-firewall-policies"></a>Créer les stratégies de pare-feu

- Une stratégie de pare-feu de base.

Créez des stratégies pour chacune des équipes d'applications :

- Une stratégie de pare-feu Ventes. La stratégie de pare-feu Ventes hérite de la stratégie de pare-feu de base.
- Une stratégie de pare-feu Base de données. La stratégie de pare-feu Base de données hérite de la stratégie de pare-feu de base.
- Une stratégie de pare-feu Ingénierie. La stratégie de pare-feu Ingénierie hérite également de la stratégie de pare-feu de base.

:::image type="content" source="media/rule-hierarchy/policy-hierarchy.png" alt-text="Équipes et exigences" border="false":::

### <a name="create-custom-roles-to-access-the-rule-collection-groups"></a>Créer des rôles personnalisés pour accéder aux regroupements de règles 

Des rôles personnalisés sont définis pour chaque équipe d'applications. Le rôle définit les opérations et l'étendue. Les équipes d'applications sont autorisées à modifier les regroupements de règles pour leurs applications respectives.

Procédez comme suit pour définir des rôles personnalisés :

1. Obtenez l'abonnement :

   `Select-AzSubscription -SubscriptionId xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxxx`
2. Exécutez la commande suivante :

   `Get-AzProviderOperation "Microsoft.Support/*" | FT Operation, Description -AutoSize`
3. Utilisez la commande Get-AzRoleDefinition pour afficher le rôle Lecteur au format JSON. 

   `Get-AzRoleDefinition -Name "Reader" | ConvertTo-Json | Out-File C:\CustomRoles\ReaderSupportRole.json`
4. Ouvrez le fichier ReaderSupportRole.json dans un éditeur.

   Voici la sortie JSON. Pour plus d'informations sur les différentes propriétés, consultez  [Rôles personnalisés Azure](../role-based-access-control/custom-roles.md).

```json
   { 
     "Name": "Reader", 
     "Id": "acdd72a7-3385-48ef-bd42-f606fba81ae7", 
     "IsCustom": false, 
     "Description": "Lets you view everything, but not make any changes.", 
     "Actions": [ 
      "*/read" 
     ], 
     "NotActions": [], 
     "DataActions": [], 
     "NotDataActions": [], 
     "AssignableScopes": [ 
       "/" 
     ] 
   } 
```
5. Modifiez le fichier JSON pour ajouter 

   `*/read", "Microsoft.Network/*/read", "Microsoft.Network/firewallPolicies/ruleCollectionGroups/write` 

   l'opération à la propriété  **Actions** . Veillez à inclure une virgule après l’opération de lecture. Cette action permet à l'utilisateur de créer et de mettre à jour des regroupements de règles.
6. Dans  **AssignableScopes** , ajoutez votre ID d'abonnement au format suivant : 

   `/subscriptions/xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxxx`

   Vous devez ajouter des ID d’abonnements explicites, sinon vous ne serez pas autorisé à importer le rôle dans votre abonnement.
7. Supprimez la ligne de propriété  **Id**  et remplacez la propriété  **IsCustom**  par true.
8. Remplacez les propriétés  **Nom**  et  **Description**  par *Auteur du regroupement de règles AZFM* et *Les utilisateurs de ce rôle peuvent modifier les regroupements de règles de la stratégie de pare-feu* .

Votre fichier JSON doit être semblable à l'exemple suivant :

```
{ 

    "Name":  "AZFM Rule Collection Group Author", 
    "IsCustom":  true, 
    "Description":  "Users in this role can edit Firewall Policy rule collection groups", 
    "Actions":  [ 
                    "*/read", 
                    "Microsoft.Network/*/read", 
                     "Microsoft.Network/firewallPolicies/ruleCollectionGroups/write" 
                ], 
    "NotActions":  [ 
                   ], 
    "DataActions":  [ 
                    ], 
    "NotDataActions":  [ 
                       ], 
    "AssignableScopes":  [ 
                             "/subscriptions/xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxxx"] 
} 
```
9. Pour créer le rôle personnalisé, utilisez la commande New-AzRoleDefinition et spécifiez le fichier de définition de rôle JSON. 

   `New-AzRoleDefinition -InputFile "C:\CustomRoles\RuleCollectionGroupRole.json`

### <a name="list-custom-roles"></a>Répertorier les rôles personnalisés

Pour dresser la liste de tous les rôles personnalisés, vous pouvez utiliser la commande Get-AzRoleDefinition :

   `Get-AzRoleDefinition | ? {$_.IsCustom -eq $true} | FT Name, IsCustom`

Vous pouvez également voir les rôles personnalisés sur le portail Azure. Accédez à votre abonnement et sélectionnez **Contrôle d'accès (IAM)** , **Rôles** .

:::image type="content" source="media/rule-hierarchy/sales-app-policy.png" alt-text="Équipes et exigences":::

:::image type="content" source="media/rule-hierarchy/sales-app-policy-read.png" alt-text="Équipes et exigences":::

Pour plus d’informations, consultez [Didacticiel : Créer un rôle personnalisé Azure à l’aide d’Azure PowerShell](../role-based-access-control/tutorial-custom-role-powershell.md).

### <a name="add-users-to-the-custom-role"></a>Ajouter des utilisateurs au rôle personnalisé

Sur le portail, vous pouvez attribuer le rôle Auteurs du regroupement de règles AZFM à des utilisateurs et leur fournir un accès aux stratégies de pare-feu.

1. Sur le portail, sélectionnez la stratégie de pare-feu de l'équipe d'applications (par exemple, SalesAppPolicy).
2. Sélectionnez **Contrôle d’accès** .
3. Sélectionnez **Ajouter une attribution de rôle** .
4. Attribuez le rôle à des utilisateurs/groupes d'utilisateurs (par exemple, l'équipe Ventes).

Répétez cette procédure pour les autres stratégies de pare-feu.

### <a name="summary"></a>Résumé

La stratégie de pare-feu avec des rôles personnalisés offre désormais un accès sélectif aux regroupements de règles de la stratégie de pare-feu.

Les utilisateurs ne disposent pas des autorisations nécessaires pour :
- Supprimer le Pare-feu Azure ou la stratégie de pare-feu
- Mettre à jour la hiérarchie des stratégies de pare-feu, les paramètres DNS ou le renseignement sur les menaces
- Mettre à jour la stratégie de pare-feu lorsqu'ils ne sont pas membres du groupe Auteur du regroupement de règles AZFM

Les administrateurs de la sécurité peuvent utiliser la stratégie de base pour mettre en place des garde-fous et bloquer certains types de trafic (par exemple, ICMP) selon les besoins de leur entreprise. 

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la [stratégie de pare-feu Azure](policy-overview.md).

