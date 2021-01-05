---
title: Blocage des connexions pour certains connecteurs d’API
description: Limitez la création et l’utilisation des connexions d’API dans Azure Logic Apps.
services: logic-apps
ms.suite: integration
ms.reviewer: deli, logicappspm
ms.topic: conceptual
ms.date: 07/23/2020
ms.openlocfilehash: 02d9852f6615c3926a02294e0e7eca50f2fbe9a5
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92310037"
---
# <a name="block-connections-created-by-connectors-in-azure-logic-apps"></a>Blocage des connexions créées par les connecteurs dans Azure Logic Apps

Si votre organisation n’autorise pas la connexion à des ressources restreintes ou non approuvées à l’aide de leurs connecteurs dans Azure Logic Apps, vous pouvez bloquer la possibilité de créer et d’utiliser ces connexions dans les flux de travail d’application logique. [Azure Policy](../governance/policy/overview.md) permet de définir et d’appliquer des [stratégies](../governance/policy/overview.md#policy-definition) qui empêchent la création ou l’utilisation de connexions pour les connecteurs à bloquer. Il peut par exemple être utile, pour des raisons de sécurité, de bloquer les connexions à certaines plateformes de réseaux sociaux ou à d’autres services et systèmes.

Cette rubrique montre comment configurer une stratégie qui bloque certaines connexions à l’aide du Portail Azure. Cependant, il existe d’autres moyens de créer des définitions de stratégie, par exemple, par le biais de l’API REST Azure, d’Azure PowerShell, d’Azure CLI et des modèles Azure Resource Manager. Pour plus d’informations, consultez [Didacticiel : Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, créez un [compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer.

* ID de référence du connecteur à bloquer. Pour plus d’informations, consultez [Recherche de l’ID de référence du connecteur](#connector-reference-ID).

<a name="connector-reference-ID"></a>

## <a name="find-connector-reference-id"></a>Recherche de l’ID de référence du connecteur

Si vous disposez déjà d’une application logique avec la connexion que vous souhaitez bloquer, suivez la [procédure pour le Portail Azure](#connector-ID-portal). Dans le cas contraire, procédez ainsi :

1. Consultez la [Liste des connecteurs Logic Apps](/connectors/connector-reference/connector-reference-logicapps-connectors).

1. Recherchez la page de référence du connecteur à bloquer.

   Par exemple, si vous souhaitez bloquer le connecteur Instagram, qui est déprécié, accédez à cette page :

   `https://docs.microsoft.com/connectors/instagram/`

1. À partir de l’URL de la page, copiez et enregistrez l’ID de référence du connecteur à la fin sans la barre oblique (`/`), par exemple, `instagram`.

   Plus tard, lorsque vous créerez votre définition de stratégie, vous utiliserez cet ID dans l’instruction de condition de la définition, par exemple :

   `"like": "*managedApis/instagram"`

<a name="connector-ID-portal"></a>

### <a name="azure-portal"></a>Portail Azure

1. Sur le [Portail Azure](https://portal.azure.com) , recherchez et ouvrez votre application logique.

1. Dans le menu de l’application logique, sélectionnez **Mode Code de l’application logique** pour pouvoir afficher la définition JSON de votre application logique.

   ![Ouverture du « mode Code de l’application logique » pour trouver l’ID du connecteur](./media/block-connections-connectors/code-view-connector-id.png)

1. Recherchez l’objet `parameters` contenant l’objet `$connections`, qui comprend un objet `{connection-name}` par connexion dans votre application logique et spécifie des informations sur cette connexion :

   ```json
   {
      "parameters": {
         "$connections": {
            "value" : {
               "{connection-name}": {
                  "connectionId": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group-name}/providers/Microsoft.Web/connections/{connection-name}",
                  "connectionName": "{connection-name}",
                  "id": "/subscriptions/{Azure-subscription-ID}/providers/Microsoft.Web/locations/{Azure-region}/managedApis/{connection-name}"
               }
            }
         }
      }
   }
   ```

   Par exemple, pour le connecteur Instagram, recherchez l’objet `instagram`, qui identifie une connexion Instagram :

   ```json
   {
      "parameters": {
         "$connections": {
            "value" : {
               "instagram": {
                  "connectionId": "/subscriptions/xxxxxXXXXXxxxxxXXXXXxxxxxXXXXX/resourceGroups/MyLogicApp-RG/providers/Microsoft.Web/connections/instagram",
                  "connectionName": "instagram",
                  "id": "/subscriptions/xxxxxXXXXXxxxxxXXXXXxxxxxXXXXX/providers/Microsoft.Web/locations/westus/managedApis/instagram"
               }
            }
         }
      }
   }
   ```

1. Pour la connexion à bloquer, recherchez la propriété et la valeur `id`, qui suit ce format : 

   `"id": "/subscriptions/{Azure-subscription-ID}/providers/Microsoft.Web/locations/{Azure-region}/managedApis/{connection-name}"`

   Par exemple, voici la propriété et la valeur `id` pour une connexion Instagram :

   `"id": "/subscriptions/xxxxxXXXXXxxxxxXXXXXxxxxxXXXXX/providers/Microsoft.Web/locations/westus/managedApis/instagram"`

1. À partir de la valeur de propriété `id`, copiez et enregistrez l’ID de référence du connecteur à la fin, par exemple, `instagram`.

   Plus tard, lorsque vous créerez votre définition de stratégie, vous utiliserez cet ID dans l’instruction de condition de la définition, par exemple :

   `"like": "*managedApis/instagram"`

<a name="create-policy-connections"></a>

## <a name="create-policy-to-block-creating-connections"></a>Création d’une stratégie pour bloquer la création de connexions

Pour bloquer complètement la création d’une connexion dans une application logique, procédez ainsi :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Dans la zone de recherche du Portail, entrez `policy`, puis sélectionnez **Stratégie**.

   ![Sur le Portail Azure, recherche et sélection de « Stratégie »](./media/block-connections-connectors/find-select-azure-policy.png)

1. Dans le menu **Stratégie**, sous **Création**, sélectionnez **Définitions** >  **+ Définition de stratégie**.

   ![Sélection de « Définitions » > « + Définition de stratégie »](./media/block-connections-connectors/add-new-policy-definition.png)

1. Sous **Définition de stratégie**, indiquez les informations relatives à votre définition de stratégie, en fonction des propriétés décrites dans l’exemple :

   ![Capture d’écran montrant les propriétés de « Définition de stratégie ».](./media/block-connections-connectors/policy-definition-create-connections-1.png)

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Emplacement de la définition** | Oui | <*Azure-subscription-name*> | Abonnement Azure à utiliser pour la définition de stratégie <p><p>1. Pour trouver votre abonnement, sélectionnez le bouton points de suspension ( **…** ). <br>2. Dans la liste **Abonnement**, recherchez et sélectionnez votre abonnement. <br>3. Lorsque vous avez terminé, sélectionnez **Sélectionner**. |
   | **Nom** | Oui | <*nom-définition-stratégie*> | Nom à utiliser pour la définition de stratégie |
   | **Description** | Non | <*nom-définition-stratégie*> | Description de la définition de stratégie |
   | **Catégorie** | Oui | **Logic Apps** | Nom d’une catégorie existante ou d’une nouvelle catégorie pour la définition de stratégie |
   | **Application de stratégies** | Oui | **Activé** | Ce paramètre spécifie si la définition de stratégie doit être activée ou désactivée lorsque vous enregistrez votre travail. |
   ||||

1. Sous **RÈGLE DE STRATÉGIE**, la zone d’édition JSON est préremplie avec un modèle de définition de stratégie. Remplacez ce modèle par votre [définition de stratégie](../governance/policy/concepts/definition-structure.md) en fonction des propriétés décrites dans le tableau ci-dessous et en suivant cette syntaxe :

   ```json
   {
      "mode": "All",
      "policyRule": {
         "if": {
            "field": "Microsoft.Web/connections/api.id",
            "like": "*managedApis/{connector-name}"
         },
         "then": {
            "effect": "deny"
         }
      },
      "parameters": {}
    }
    ```

   | Propriété | Valeur | Description |
   |----------|-------|-------------|
   | `mode` | `All` | Mode déterminant les types de ressources évalués par la stratégie <p><p>Ce scénario affecte à `mode` la valeur `All`, qui applique la stratégie aux groupes de ressources Azure, aux abonnements et à tous les types de ressources. <p><p>Pour plus d’informations, consultez [Structure de définition de stratégie – Mode](../governance/policy/concepts/definition-structure.md#mode). |
   | `if` | `{condition-to-evaluate}` | Condition déterminant quand appliquer la règle de stratégie <p><p>Dans ce scénario, `{condition-to-evaluate}` détermine si la valeur `api.id` de `Microsoft.Web/connections/api.id` correspond à `*managedApis/{connector-name}`, qui spécifie une valeur générique (*). <p><p>Pour plus d’informations, consultez [Structure de définition de stratégie – Règle de stratégie](../governance/policy/concepts/definition-structure.md#policy-rule). |
   | `field` | `Microsoft.Web/connections/api.id` | Valeur `field` à comparer à la condition <p><p>Dans ce scénario, `field` utilise [*l’alias*](../governance/policy/concepts/definition-structure.md#aliases), `Microsoft.Web/connections/api.id`, pour accéder à la valeur dans la propriété du connecteur, `api.id`. |
   | `like` | `*managedApis/{connector-name}` | Opérateur logique et valeur à utiliser pour comparer la valeur `field` <p><p>Dans ce scénario, l’opérateur `like` et le caractère générique (*) font en sorte que la règle fonctionne indépendamment de la région ; la chaîne, `*managedApis/{connector-name}`, représente à la valeur de correspondance, où `{connector-name}` est l’ID du connecteur à bloquer. <p><p>Supposons, par exemple, que vous souhaitiez bloquer la création de connexions à des plateformes de réseaux sociaux ou à des bases de données : <p><p>- Twitter : `twitter` <br>- Instagram : `instagram` <br>- Facebook : `facebook` <br>- Pinterest : `pinterest` <br>- SQL Server ou Azure SQL : `sql` <p><p>Pour rechercher ces ID de connecteur, consultez [Recherche de l’ID de référence du connecteur](#connector-reference-ID) plus haut dans cette rubrique. |
   | `then` | `{effect-to-apply}` | Effet à appliquer lorsque la condition `if` est remplie <p><p>Dans ce scénario, `{effect-to-apply}` consiste à bloquer et à faire échouer une demande ou une opération non conforme à la stratégie. <p><p>Pour plus d’informations, consultez [Structure de définition de stratégie – Règle de stratégie](../governance/policy/concepts/definition-structure.md#policy-rule). |
   | `effect` | `deny` | `effect` consiste à bloquer la demande, à savoir créer la connexion spécifiée <p><p>Pour plus d’informations, consultez [Présentation des effets Azure Policy – Refuser](../governance/policy/concepts/effects.md#deny). |
   ||||

   Supposons, par exemple, que vous souhaitiez bloquer la création de connexions avec le connecteur Instagram. Vous pouvez utiliser cette définition de stratégie :

   ```json
   {
      "mode": "All",
      "policyRule": {
         "if": {
            "field": "Microsoft.Web/connections/api.id",
            "like": "*managedApis/instagram"
         },
         "then": {
            "effect": "deny"
         }
      },
      "parameters": {}
   }
   ```

   La zone **RÈGLE DE STRATÉGIE** s’affiche de la façon suivante :

   ![Capture d’écran montrant la zone « RÈGLE DE STRATÉGIE » avec un exemple de règle de stratégie.](./media/block-connections-connectors/policy-definition-create-connections-2.png)

   Pour plusieurs connecteurs, vous pouvez ajouter des conditions supplémentaires, par exemple :

   ```json
   {
      "mode": "All",
      "policyRule": {
         "if": {
            "anyOf": [
               {
                  "field": "Microsoft.Web/connections/api.id",
                  "like": "*managedApis/instagram"
               },
               {
                  "field": "Microsoft.Web/connections/api.id",
                  "like": "*managedApis/twitter"
               },
               {
                  "field": "Microsoft.Web/connections/api.id",
                  "like": "*managedApis/facebook"
               },
               {
                  "field": "Microsoft.Web/connections/api.id",
                  "like": "*managedApis/pinterest"
               }
            ]
         },
         "then": {
            "effect": "deny"
         }
      },
      "parameters": {}
    }
    ```

1. Quand vous avez terminé, sélectionnez **Enregistrer**. Une fois la définition de stratégie enregistrée, Azure Policy génère et ajoute d’autres valeurs de propriété à la définition de stratégie.

1. Ensuite, pour affecter la définition de stratégie là où vous souhaitez appliquer la stratégie, [créez une affectation de stratégie](#create-policy-assignment).

Pour plus d’informations sur les définitions de stratégies Azure, consultez ces rubriques :

* [Définition d’une structure de stratégie](../governance/policy/concepts/definition-structure.md)
* [Tutoriel : Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)
* [Définitions de stratégies intégrées Azure Policy pour Azure Logic Apps](./policy-reference.md)

<a name="create-policy-connector-usage"></a>

## <a name="create-policy-to-block-using-connections"></a>Création d’une stratégie pour bloquer l’utilisation de connexions

Une connexion créée à l’intérieur d’une application logique existe en tant que ressource Azure distincte. Si vous supprimez uniquement l’application logique, la connexion n’est pas supprimée automatiquement et subsiste jusqu’à ce qu’elle soit supprimée. Il existe des cas de figure dans lesquels la connexion existe déjà et d’autres dans lesquels vous devez créer la connexion pour l’utiliser en dehors d’une application logique. Vous pouvez toujours empêcher la possibilité d’utiliser une connexion existante dans une application logique en créant une stratégie qui empêche l’enregistrement d’applications logiques dont la connexion est restreinte ou non approuvée.

1. Connectez-vous au [portail Azure](https://portal.azure.com). Dans la zone de recherche du Portail, entrez `policy`, puis sélectionnez **Stratégie**.

   ![Sur le Portail Azure, recherche et sélection de « Stratégie »](./media/block-connections-connectors/find-select-azure-policy.png)

1. Dans le menu **Stratégie**, sous **Création**, sélectionnez **Définitions** >  **+ Définition de stratégie**.

   ![Sélection de « Définitions » > « + Définition de stratégie »](./media/block-connections-connectors/add-new-policy-definition.png)

1. Sous **Définition de stratégie**, indiquez les informations relatives à votre définition de stratégie, en fonction des propriétés décrites dans l’exemple d’Instagram :

   ![Propriétés de la définition de stratégie](./media/block-connections-connectors/policy-definition-using-connections-1.png)

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Emplacement de la définition** | Oui | <*Azure-subscription-name*> | Abonnement Azure à utiliser pour la définition de stratégie <p><p>1. Pour trouver votre abonnement, sélectionnez le bouton points de suspension ( **…** ). <br>2. Dans la liste **Abonnement**, recherchez et sélectionnez votre abonnement. <br>3. Lorsque vous avez terminé, sélectionnez **Sélectionner**. |
   | **Nom** | Oui | <*nom-définition-stratégie*> | Nom à utiliser pour la définition de stratégie |
   | **Description** | Non | <*nom-définition-stratégie*> | Description de la définition de stratégie |
   | **Catégorie** | Oui | **Logic Apps** | Nom d’une catégorie existante ou d’une nouvelle catégorie pour la définition de stratégie |
   | **Application de stratégies** | Oui | **Activé** | Ce paramètre spécifie si la définition de stratégie doit être activée ou désactivée lorsque vous enregistrez votre travail. |
   ||||

1. Sous **RÈGLE DE STRATÉGIE**, la zone d’édition JSON est préremplie avec un modèle de définition de stratégie. Remplacez ce modèle par votre [définition de stratégie](../governance/policy/concepts/definition-structure.md) en fonction des propriétés décrites dans le tableau ci-dessous et en suivant cette syntaxe :

   ```json
   {
      "mode": "All",
      "policyRule": {
         "if": {
            "value": "[string(field('Microsoft.Logic/workflows/parameters'))]",
            "contains": "{connector-name}"
         },
         "then": {
            "effect": "deny"
         }
      },
      "parameters": {}
    }
    ```

   | Propriété | Valeur | Description |
   |----------|-------|-------------|
   | `mode` | `All` | Mode déterminant les types de ressources évalués par la stratégie <p><p>Ce scénario affecte à `mode` la valeur `All`, qui applique la stratégie aux groupes de ressources Azure, aux abonnements et à tous les types de ressources. <p><p>Pour plus d’informations, consultez [Structure de définition de stratégie – Mode](../governance/policy/concepts/definition-structure.md#mode). |
   | `if` | `{condition-to-evaluate}` | Condition déterminant quand appliquer la règle de stratégie <p><p>Dans ce scénario, `{condition-to-evaluate}` détermine si la sortie de chaîne de `[string(field('Microsoft.Logic/workflows/parameters'))]` contient la chaîne `{connector-name}`. <p><p>Pour plus d’informations, consultez [Structure de définition de stratégie – Règle de stratégie](../governance/policy/concepts/definition-structure.md#policy-rule). |
   | `value` | `[string(field('Microsoft.Logic/workflows/parameters'))]` | Valeur à comparer à la condition <p><p>Dans ce scénario, `value` est la sortie de chaîne de `[string(field('Microsoft.Logic/workflows/parameters'))]`, qui convertit en chaîne l’objet `$connectors` de l’objet `Microsoft.Logic/workflows/parameters`. |
   | `contains` | `{connector-name}` | Opérateur logique et valeur à utiliser pour la comparaison avec la propriété `value` <p><p>Dans ce scénario, l’opérateur `contains` fait en sorte que la règle fonctionne indépendamment de l’endroit où `{connector-name}` apparaît ; la chaîne `{connector-name}` est l’ID du connecteur à restreindre ou bloquer. <p><p>Supposons, par exemple, que vous souhaitiez bloquer l’utilisation de connexions à des plateformes de réseaux sociaux ou à des bases de données : <p><p>- Twitter : `twitter` <br>- Instagram : `instagram` <br>- Facebook : `facebook` <br>- Pinterest : `pinterest` <br>- SQL Server ou Azure SQL : `sql` <p><p>Pour rechercher ces ID de connecteur, consultez [Recherche de l’ID de référence du connecteur](#connector-reference-ID) plus haut dans cette rubrique. |
   | `then` | `{effect-to-apply}` | Effet à appliquer lorsque la condition `if` est remplie <p><p>Dans ce scénario, `{effect-to-apply}` consiste à bloquer et à faire échouer une demande ou une opération non conforme à la stratégie. <p><p>Pour plus d’informations, consultez [Structure de définition de stratégie – Règle de stratégie](../governance/policy/concepts/definition-structure.md#policy-rule). |
   | `effect` | `deny` | `effect` consiste à `deny` ou à bloquer la demande d’enregistrement d’une application logique qui utilise la connexion spécifiée <p><p>Pour plus d’informations, consultez [Présentation des effets Azure Policy – Refuser](../governance/policy/concepts/effects.md#deny). |
   ||||

   Supposons, par exemple, que vous souhaitiez bloquer l’enregistrement d’applications logiques qui utilisent des connexions Instagram. Vous pouvez utiliser cette définition de stratégie :

   ```json
   {
      "mode": "All",
      "policyRule": {
         "if": {
            "value": "[string(field('Microsoft.Logic/workflows/parameters'))]",
            "contains": "instagram"
         },
         "then": {
            "effect": "deny"
         }
      },
      "parameters": {}
    }
    ```

   La zone **RÈGLE DE STRATÉGIE** s’affiche de la façon suivante :

   ![Règle de la définition de stratégie](./media/block-connections-connectors/policy-definition-using-connections-2.png)

1. Quand vous avez terminé, sélectionnez **Enregistrer**. Une fois la définition de stratégie enregistrée, Azure Policy génère et ajoute d’autres valeurs de propriété à la définition de stratégie.

1. Ensuite, pour affecter la définition de stratégie là où vous souhaitez appliquer la stratégie, [créez une affectation de stratégie](#create-policy-assignment).

Pour plus d’informations sur les définitions de stratégies Azure, consultez ces rubriques :

* [Définition d’une structure de stratégie](../governance/policy/concepts/definition-structure.md)
* [Tutoriel : Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)
* [Définitions de stratégies intégrées Azure Policy pour Azure Logic Apps](./policy-reference.md)

<a name="create-policy-assignment"></a>

## <a name="create-policy-assignment"></a>Création d’affectations de stratégies

Ensuite, vous devez affecter la définition de stratégie là où vous souhaitez appliquer la stratégie, par exemple, à un groupe de ressources unique, à plusieurs groupes de ressources, à un locataire Azure Active Directory (Azure AD) ou à un abonnement Azure. Dans le cadre de cette tâche, suivez cette procédure pour créer une affectation de stratégie :

1. Si vous vous êtes déconnecté, reconnectez-vous au [Portail Azure](https://portal.azure.com). Dans la zone de recherche du Portail, entrez `policy`, puis sélectionnez **Stratégie**.

   ![Sur le Portail Azure, recherche et sélection de « Stratégie »](./media/block-connections-connectors/find-select-azure-policy.png)

1. Dans le menu **Stratégie**, sous **Création**, sélectionnez **Affectations** > **Affecter une stratégie**.

   ![Sélection de « Affectations » > « Affecter »](./media/block-connections-connectors/add-new-policy-assignment.png)

1. Sous **Informations de base**, indiquez ces informations pour l’affectation de stratégie :

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Portée** | Oui | Ressources dans lesquelles vous souhaitez appliquer l’affectation de stratégie <p><p>1. À côté de la zone **Étendue**, sélectionnez le bouton points de suspension ( **…** ). <br>2. Dans la liste **Abonnement**, sélectionnez l’abonnement Azure. <br>3. Dans la liste **Groupe de ressources**, sélectionnez le groupe de ressources (facultatif). <br>4. Lorsque vous avez terminé, sélectionnez **Sélectionner**. |
   | **Exclusions** | Non | Ressources Azure à exclure de l’affectation de stratégie <p><p>1. À côté de la zone **Exclusions**, sélectionnez le bouton points de suspension ( **…** ). <br>2. Dans la liste **Ressource**, sélectionnez la ressource > **Ajouter à l’étendue sélectionnée**. <br>3. Quand vous avez terminé, sélectionnez **Enregistrer**. |
   | **Définition de stratégie** | Oui | Nom de la définition de stratégie que vous souhaitez affecter et appliquer. Cet exemple reprend l’exemple de stratégie Instagram, « Bloquer les connexions Instagram ». <p><p>1. À côté de la zone **Définition de stratégie**, sélectionnez le bouton points de suspension ( **…** ). <br>2. Recherchez et sélectionnez la définition de stratégie à l’aide du filtre **Type** ou de la zone **Rechercher**. <br>3. Lorsque vous avez terminé, sélectionnez **Sélectionner**. |
   | **Nom de l’affectation** | Oui | Nom à utiliser pour l’affectation de stratégie, s’il est différent de la définition de stratégie |
   | **ID de l’affectation** | Oui | ID généré automatiquement pour l’affectation de stratégie |
   | **Description** | Non | Description de l’affectation de stratégie |
   | **Application de stratégies** | Oui | Paramètre qui active ou désactive l’affectation de stratégie |
   | **Affecté par** | Non | Nom de la personne qui a créé et appliqué l’affectation de stratégie |
   ||||

   Par exemple, pour affecter la stratégie à un groupe de ressources Azure dans l’exemple Instagram :

   ![Propriétés d’affectation de stratégie](./media/block-connections-connectors/policy-assignment-basics.png)

1. Quand vous avez terminé, sélectionnez **Vérifier + créer**.

   Après avoir créé une stratégie, vous devrez peut-être attendre jusqu’à 15 minutes pour qu’elle soit effective. Les modifications peuvent également avoir des effets différés similaires.

1. Une fois la stratégie appliquée, vous pouvez [tester votre stratégie](#test-policy).

Pour plus d’informations, consultez [Démarrage rapide : Création d’une affectation de stratégie pour identifier les ressources non conformes](../governance/policy/assign-policy-portal.md).

<a name="test-policy"></a>

## <a name="test-the-policy"></a>Tester la stratégie

Pour tester votre stratégie, commencez à créer une connexion à l’aide du connecteur maintenant restreint dans le concepteur d’applications logiques. Reprenons l’exemple Instagram. Lorsque vous vous connectez à Instagram, vous recevez cette erreur indiquant que votre application logique n’a pas pu créer la connexion :

![Échec de la connexion en raison de la stratégie appliquée](./media/block-connections-connectors/connection-failure-message.png)

Le message comporte les informations suivantes :

| Description | Contenu |
|-------------|---------|
| Raison de l’échec | `"Resource 'instagram' was disallowed by policy."` |
| Nom de l’attribution | `"Block Instagram connections"` |
| ID de l’affectation | `"/subscriptions/xxxxxXXXXXxxxxxXXXXXxxxxxXXXXX/resourceGroups/MyLogicApp-RG/providers/Microsoft.Authorization/policyAssignments/4231890fc3bd4352acb0b673"` |
| ID de définition de stratégie | `"/subscriptions/xxxxxXXXXXxxxxxXXXXXxxxxxXXXXX/providers/Microsoft.Authorization/policyDefinitions/b5ddcfec-1b24-4cac-a353-360846a59f24"` |
|||

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [Azure Policy](../governance/policy/overview.md)