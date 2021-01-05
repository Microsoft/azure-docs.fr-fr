---
title: Authentifier avec des identités gérées
description: Accéder aux ressources protégées par Azure Active Directory sans devoir se connecter à l’aide d’informations d’identification ou de secrets, grâce à une identité managée
services: logic-apps
ms.suite: integration
ms.reviewer: jonfan, logicappspm
ms.topic: article
ms.date: 10/27/2020
ms.openlocfilehash: bb146f03000f17d94d3d2ffc93b55c42eea20dac
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92736407"
---
# <a name="authenticate-access-to-azure-resources-by-using-managed-identities-in-azure-logic-apps"></a>Authentifier l’accès aux ressources Azure avec des identités managées dans Azure Logic Apps

Pour accéder facilement à d’autres ressources protégées par Azure Active Directory (Azure AD) et authentifier votre identité sans vous connecter, votre application logique peut utiliser une [identité managée](../active-directory/managed-identities-azure-resources/overview.md) (anciennement appelée MSI ou Managed Service Identity) à la place d’informations d’identification ou de secrets. Azure gère cette identité pour vous et vous aide à sécuriser vos informations d’identification, car vous n’êtes pas obligé de fournir ni de faire pivoter des secrets.

Azure Logic Apps prend en charge les identités managées [*affectées par le système*](../active-directory/managed-identities-azure-resources/overview.md) et [*affectées par l’utilisateur*](../active-directory/managed-identities-azure-resources/overview.md). Votre application logique peut utiliser soit l’identité affectée par le système, soit une *seule* identité affectée par l’utilisateur, que vous pouvez partager dans un groupe d’applications logiques. Seuls des [déclencheurs et actions intégrés spécifiques](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound) prennent en charge les identités managées, et non les connecteurs ou connexions managés, par exemple :

* HTTP
* Azure Functions
* Gestion des API Azure
* Azure App Services

Cet article montre comment configurer les deux genres d’identité managée pour votre application logique. Pour plus d’informations, consultez les rubriques suivantes :

* [Déclencheurs et actions qui prennent en charge les identités managées](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound)
* [Types d’authentification pris en charge sur des appels sortants](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound)
* [Limites sur les identités managées pour les applications logiques](../logic-apps/logic-apps-limits-and-config.md#managed-identity)
* [Services Azure qui prennent en charge l’authentification Azure AD avec des identités managées](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication)

## <a name="prerequisites"></a>Prérequis

* Un compte et un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/). L’identité managée et la ressource Azure cible à laquelle vous souhaitez accéder doivent utiliser le même abonnement Azure.

* Pour accorder à une identité managée l’accès à une ressource Azure, vous devez ajouter un rôle à la ressource cible pour cette identité. Pour ajouter des rôles, vous devez disposer d’[autorisations d’administrateur Azure AD](../active-directory/roles/permissions-reference.md) qui permettent d’attribuer des rôles à des identités dans le locataire Azure AD correspondant.

* Ressource Azure cible à laquelle vous souhaitez accéder. Dans cette ressource, vous allez ajouter un rôle pour l’identité managée, ce qui permet à l’application logique d’authentifier l’accès à la ressource cible.

* Application logique où vous souhaitez utiliser le [déclencheur ou les actions qui prennent en charge les identités managées](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound)

## <a name="enable-managed-identity"></a>Activer une identité managée

Pour configurer l’identité managée à utiliser, suivez le lien de cette identité :

* [Identité affectée par le système](#system-assigned)
* [Identité affectée par l’utilisateur](#user-assigned)

<a name="system-assigned"></a>

### <a name="enable-system-assigned-identity"></a>Activer l’identité attribuée par le système

À la différences des identités attribuées par l’utilisateur, vous n’avez pas besoin de créer manuellement l’identité attribuée par le système. Pour configurer l’identité affectée par le système dans le cadre de votre application logique, voici les options que vous pouvez utiliser :

* [Azure portal](#azure-portal-system-logic-app)
* [Modèles Microsoft Azure Resource Manager](#template-system-logic-app)

<a name="azure-portal-system-logic-app"></a>

#### <a name="enable-system-assigned-identity-in-azure-portal"></a>Activer une identité attribuée par le système sur le portail Azure

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Dans le menu de l’application logique, sous **Paramètres** , sélectionnez **Identité**. Sélectionnez **Affecté(e) par le système** > **Actif** > **Enregistrer**. Quand Azure vous invite à confirmer l’opération, sélectionnez **Oui**.

   ![Activer l’identité attribuée par le système](./media/create-managed-service-identity/enable-system-assigned-identity.png)

   > [!NOTE]
   > Si vous recevez une erreur indiquant que vous ne pouvez avoir qu’une seule identité managée, cela signifie que votre application logique est déjà associée à l’identité affectée par l’utilisateur. Avant de pouvoir ajouter l’identité affectée par le système, vous devez d’abord *supprimer* l’identité affectée par l’utilisateur de votre application logique.

   Votre application logique peut désormais utiliser l’identité attribuée par le système qui est inscrite auprès d’Azure Active Directory et représentée par un ID d’objet.

   ![ID d’objet pour l’identité attribuée par le système](./media/create-managed-service-identity/object-id-system-assigned-identity.png)

   | Propriété | Valeur | Description |
   |----------|-------|-------------|
   | **ID d’objet** | <*identity-resource-ID*> | GUID (identificateur global unique) qui représente l’identité affectée par le système pour votre application logique dans un locataire Azure AD |
   ||||

1. À présent, suivez les [étapes qui donnent à cette identité l’accès à la ressource](#access-other-resources), et que vous trouverez plus loin dans cette rubrique.

<a name="template-system-logic-app"></a>

#### <a name="enable-system-assigned-identity-in-azure-resource-manager-template"></a>Activer l’identité attribuée par le système dans un modèle Resource Manager

Pour automatiser la création et le déploiement de ressources Azure telles que des applications logiques, vous pouvez utiliser des [modèles Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md). Pour activer l’identité managée attribuée par le système pour votre application logique dans le modèle, ajoutez l’objet `identity` et la propriété enfant `type` à la définition de ressource de l’application logique dans le modèle, par exemple :

```json
{
   "apiVersion": "2016-06-01",
   "type": "Microsoft.logic/workflows",
   "name": "[variables('logicappName')]",
   "location": "[resourceGroup().location]",
   "identity": {
      "type": "SystemAssigned"
   },
   "properties": {
      "definition": {
         "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
         "actions": {},
         "parameters": {},
         "triggers": {},
         "contentVersion": "1.0.0.0",
         "outputs": {}
   },
   "parameters": {},
   "dependsOn": []
}
```

Quand Azure crée la définition de ressource de votre application logique, l’objet `identity` reçoit les propriétés supplémentaires suivantes :

```json
"identity": {
   "type": "SystemAssigned",
   "principalId": "<principal-ID>",
   "tenantId": "<Azure-AD-tenant-ID>"
}
```

| Propriété (JSON) | Valeur | Description |
|-----------------|-------|-------------|
| `principalId` | <*principal-ID*> | Identificateur global unique (GUID) de l’objet du principal de service pour l’identité managée qui représente votre application logique dans le locataire Azure AD. Ce GUID apparaît parfois sous la forme d’un « ID d’objet » ou `objectID`. |
| `tenantId` | <*Azure-AD-tenant-ID*> | Identificateur global unique (GUID) représentant le locataire Azure AD où l’application logique est maintenant membre. À l’intérieur du locataire Azure AD, le principal du service a le même nom que l’instance d’application logique. |
||||

<a name="user-assigned"></a>

### <a name="enable-user-assigned-identity"></a>Activer l’identité affectée par l’utilisateur

Pour configurer une identité managée affectée par l’utilisateur dans le cadre de votre application logique, vous devez d’abord créer cette identité en tant que ressource Azure autonome distincte. Voici les options que vous pouvez utiliser :

* [Azure portal](#azure-portal-user-identity)
* [Modèles Microsoft Azure Resource Manager](#template-user-identity)
* Azure PowerShell
  * [Créer une identité affectée par l’utilisateur](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md)
  * [Ajouter une attribution de rôle](../active-directory/managed-identities-azure-resources/howto-assign-access-powershell.md)
* Azure CLI
  * [Créer une identité affectée par l’utilisateur](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md)
  * [Ajouter une attribution de rôle](../active-directory/managed-identities-azure-resources/howto-assign-access-cli.md)
* API REST Azure
  * [Créer une identité affectée par l’utilisateur](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-rest.md)
  * [Ajouter une attribution de rôle](../role-based-access-control/role-assignments-rest.md)

<a name="azure-portal-user-identity"></a>

#### <a name="create-user-assigned-identity-in-the-azure-portal"></a>Créer une identité affectée par l’utilisateur dans le portail Azure

1. Dans le [portail Azure](https://portal.azure.com), dans la zone de recherche de n’importe quelle page, entrez `managed identities`, puis sélectionnez **Identités managées**.

   ![Rechercher et sélectionner « Identités managées »](./media/create-managed-service-identity/find-select-managed-identities.png)

1. Sous **Identités managées** , sélectionnez **Ajouter**.

   ![Ajouter une nouvelle identité managée](./media/create-managed-service-identity/add-user-assigned-identity.png)

1. Fournissez des informations sur votre identité managée, puis sélectionnez **Vérifier + créer** , par exemple :

   ![Créer une identité managée affectée par l’utilisateur](./media/create-managed-service-identity/create-user-assigned-identity.png)

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Abonnement** | Oui | <*Azure-subscription-name*> | Nom de l’abonnement Azure à utiliser. |
   | **Groupe de ressources** | Oui | <*nom-groupe-de-ressources-Azure*> | Nom du groupe de ressources à utiliser. Créez un groupe, ou sélectionnez un groupe existant. Cet exemple crée un groupe nommé `fabrikam-managed-identities-RG`. |
   | **Région** | Oui | <*Azure-region*> | Région Azure où stocker les informations relatives à votre ressource. Cet exemple utilise la région « USA Ouest ». |
   | **Nom** | Oui | <*nom-identité-affectée-par-utilisateur*> | Nom à donner à votre identité affectée par l’utilisateur. Cet exemple utilise `Fabrikam-user-assigned-identity`. |
   |||||

   Une fois ces détails validés, Azure crée votre identité managée. À présent, vous pouvez ajouter l’identité affectée par l’utilisateur à votre application logique. Vous ne pouvez pas ajouter plusieurs identités affectées par l’utilisateur à votre application logique.

1. Dans le portail Azure, recherchez et ouvrez votre application logique dans le Concepteur d’application logique.

1. Dans le menu de l’application logique, sous **Paramètres** , sélectionnez **Identité** , puis **Affecté(e) par l’utilisateur** > **Ajouter**.

   ![Ajouter une identité managée affectée par l’utilisateur](./media/create-managed-service-identity/add-user-assigned-identity-logic-app.png)

1. Dans le volet **Ajouter une identité managée affectée par l’utilisateur** , dans la liste **Abonnement** , sélectionnez votre abonnement Azure, si cela n’est pas déjà fait. Dans la liste qui montre *toutes* les identités managées de cet abonnement, recherchez et sélectionnez l’identité affectée par l’utilisateur qui vous intéresse. Pour filtrer la liste, dans la zone de recherche **Identités managées affectées par l’utilisateur** , entrez le nom de l’identité ou du groupe de ressources. Une fois que vous avez terminé, sélectionnez **Ajouter**.

   ![Sélectionner l’identité affectée par l’utilisateur à employer](./media/create-managed-service-identity/select-user-assigned-identity.png)

   > [!NOTE]
   > Si vous recevez une erreur indiquant que vous ne pouvez avoir qu’une seule identité managée, cela signifie que votre application logique est déjà associée à l’identité affectée par le système. Avant de pouvoir ajouter l’identité affectée par l’utilisateur, vous devez d’abord désactiver l’identité affectée par le système sur votre application logique.

   Votre application logique est désormais associée à l’identité managée affectée par l’utilisateur.

   ![Association à l’identité affectée par l’utilisateur](./media/create-managed-service-identity/added-user-assigned-identity.png)

1. À présent, suivez les [étapes qui donnent à cette identité l’accès à la ressource](#access-other-resources), et que vous trouverez plus loin dans cette rubrique.

<a name="template-user-identity"></a>

#### <a name="create-user-assigned-identity-in-an-azure-resource-manager-template"></a>Créer une identité affectée par l’utilisateur dans un modèle Azure Resource Manager

Pour automatiser la création et le déploiement de ressources Azure telles que les applications logiques, vous pouvez utiliser des [modèles Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md). Ceux-ci prennent en charge les [identités affectées par l’utilisateur pour l’authentification](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-arm.md). Dans la section `resources` de votre modèle, la définition de ressource de votre application logique nécessite les éléments suivants :

* Objet `identity` dont la propriété `type` a la valeur `UserAssigned`

* Objet `userAssignedIdentities` enfant qui spécifie la ressource et le nom affectés par l’utilisateur

Cet exemple montre une définition de ressource d’application logique pour une requête HTTP PUT, et comprend un objet `identity` non paramétré. La réponse à la requête PUT et à l’opération GET qui suit comporte également cet objet `identity` :

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {<template-parameters>},
   "resources": [
      {
         "apiVersion": "2016-06-01",
         "type": "Microsoft.logic/workflows",
         "name": "[variables('logicappName')]",
         "location": "[resourceGroup().location]",
         "identity": {
            "type": "UserAssigned",
            "userAssignedIdentities": {
               "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user-assigned-identity-name>": {}
            }
         },
         "properties": {
            "definition": {<logic-app-workflow-definition>}
         },
         "parameters": {},
         "dependsOn": []
      },
   ],
   "outputs": {}
}
```

Si votre modèle inclut également la définition de ressource de l’identité managée, vous pouvez paramétrer l’objet `identity`. Cet exemple montre comment l’objet `userAssignedIdentities` enfant référence une variable `userAssignedIdentity` que vous définissez dans la section `variables` de votre modèle. Cette variable référence l’ID de ressource de l’identité affectée par l’utilisateur.

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {
      "Template_LogicAppName": {
         "type": "string"
      },
      "Template_UserAssignedIdentityName": {
         "type": "securestring"
      }
   },
   "variables": {
      "logicAppName": "[parameters(`Template_LogicAppName')]",
      "userAssignedIdentityName": "[parameters('Template_UserAssignedIdentityName')]"
   },
   "resources": [
      {
         "apiVersion": "2016-06-01",
         "type": "Microsoft.logic/workflows",
         "name": "[variables('logicAppName')]",
         "location": "[resourceGroup().location]",
         "identity": {
            "type": "UserAssigned",
            "userAssignedIdentities": {
               "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', variables('userAssignedIdentityName'))]": {}
            }
         },
         "properties": {
            "definition": {<logic-app-workflow-definition>}
         },
         "parameters": {},
         "dependsOn": [
            "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', variables('userAssignedIdentityName'))]"
         ]
      },
      {
         "apiVersion": "2018-11-30",
         "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
         "name": "[parameters('Template_UserAssignedIdentityName')]",
         "location": "[resourceGroup().location]",
         "properties": {}
      }
  ]
}
```

<a name="access-other-resources"></a>

## <a name="give-identity-access-to-resources"></a>Accorder à une identité l’accès aux ressources

Pour pouvoir vous servir de l’identité managée de votre application logique dans le cadre de l’authentification, configurez l’accès de cette identité à la ressource Azure où vous prévoyez de l’utiliser. Pour effectuer cette tâche, attribuez le rôle approprié à cette identité sur la ressource Azure cible. Voici les options que vous pouvez utiliser :

* [Azure portal](#azure-portal-assign-access)
* [Modèle Azure Resource Manager](../role-based-access-control/role-assignments-template.md)
* Azure PowerShell ([New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment)) : pour plus d’informations, consultez [Ajouter ou supprimer des attributions de rôles à l’aide du RBAC Azure et d’Azure PowerShell](../role-based-access-control/role-assignments-powershell.md).
* Azure CLI ([az role assignment create](/cli/azure/role/assignment?view=azure-cli-latest&preserve-view=true#az-role-assignment-create)) : pour plus d’informations, consultez [Ajouter ou supprimer des attributions de rôles avec le RBAC Azure et Azure CLI](../role-based-access-control/role-assignments-cli.md).
* [API REST Azure](../role-based-access-control/role-assignments-rest.md)

<a name="azure-portal-assign-access"></a>

### <a name="assign-access-in-the-azure-portal"></a>Accorder l’accès dans le portail Azure

1. Dans le [portail Azure](https://portal.azure.com), accédez à la ressource Azure à laquelle vous souhaitez que votre identité managée ait accès.

1. Dans le menu de la ressource, sélectionnez **Contrôle d’accès (IAM)**  > **Attributions de rôle** pour examiner les attributions de rôles actuelles pour cette ressource. Dans la barre d’outils, sélectionnez **Ajouter** > **Ajouter une attribution de rôle**.

   ![Sélectionner « Ajouter » > « Ajouter une attribution de rôle »](./media/create-managed-service-identity/add-role-to-resource.png)

   > [!TIP]
   > Si l’option **Ajouter une attribution de rôle** est désactivée, vous n’avez probablement pas les autorisations appropriées. Pour plus d’informations sur les autorisations permettant de gérer des rôles pour des ressources, voir [Autorisations des rôles d’administrateur dans Azure Active Directory](../active-directory/roles/permissions-reference.md).

1. Sous **Ajouter une attribution de rôle** , sélectionnez un **rôle** qui donne à votre identité l’accès nécessaire à la ressource cible.

   Pour l’exemple de cette rubrique, votre identité a besoin d’un [rôle pouvant accéder au blob dans un conteneur de stockage Azure](../storage/common/storage-auth-aad.md#assign-azure-roles-for-access-rights). Sélectionnez donc le rôle **Contributeur aux données Blob du stockage** pour l’identité managée.

   ![Sélectionner « Contributeur aux données Blob du stockage »](./media/create-managed-service-identity/select-role-for-identity.png)

1. Suivez les étapes ci-dessous pour votre identité managée :

   * **Identité affectée par le système**

     1. Dans la zone **Attribuer l’accès à** , sélectionnez **Application logique**. Quand la propriété **Abonnement** s’affiche, sélectionnez l’abonnement Azure associé à votre identité.

        ![Sélectionner l’accès pour un identité attribuée par le système](./media/create-managed-service-identity/assign-access-system.png)

     1. Dans la zone **Sélectionner** , sélectionnez votre application logique dans la liste. Si la liste est trop longue, utilisez la zone **Sélectionner** pour filtrer la liste.

        ![Sélectionner une application logique pour l’identité attribuée par le système](./media/create-managed-service-identity/add-permissions-select-logic-app.png)

   * **Identité affectée par l’utilisateur**

     1. Dans la zone **Attribuer l’accès à** , sélectionnez **Identité managée affectée par l’utilisateur**. Quand la propriété **Abonnement** s’affiche, sélectionnez l’abonnement Azure associé à votre identité.

        ![Sélectionner l’accès pour l’identité affectée par l’utilisateur](./media/create-managed-service-identity/assign-access-user.png)

     1. Sous la case **Sélectionner** , sélectionnez votre identité dans la liste. Si la liste est trop longue, utilisez la zone **sélectionner** pour filtrer la liste.

        ![Sélectionner votre identité affectée par l’utilisateur](./media/create-managed-service-identity/add-permissions-select-user-assigned-identity.png)

1. Quand vous avez terminé, sélectionnez **Enregistrer**.

   La liste d’attributions de rôle de la ressource cible affiche maintenant l’identité et le rôle managés sélectionnés. Cet exemple montre comment vous pouvez utiliser l’identité affectée par le système pour une application logique, et une identité affectée par l’utilisateur pour un autre groupe d’applications logiques.

   ![Identités et rôles managés ajoutés à la ressource cible](./media/create-managed-service-identity/added-roles-for-identities.png)

   Pour plus d’informations, consultez [Attribuer à une identité managée un accès à une ressource à l’aide du portail Azure](../active-directory/managed-identities-azure-resources/howto-assign-access-portal.md).

1. À présent, suivez les [étapes pour authentifier l’accès avec l’identité](#authenticate-access-with-identity) dans un déclencheur ou une action prenant en charge les identités managées.

<a name="authenticate-access-with-identity"></a>

## <a name="authenticate-access-with-managed-identity"></a>Authentifier l’accès avec des identités managées

Une fois que vous avez [activé l’identité managée pour votre application logique](#azure-portal-system-logic-app) et [accordé à cette identité l’accès à la ressource ou entité cibles](#access-other-resources), vous pouvez utiliser cette identité dans des [déclencheurs et actions prenant en charge les identités managées](logic-apps-securing-a-logic-app.md#managed-identity-authentication).

> [!IMPORTANT]
> Si vous avez une fonction Azure dans laquelle vous voulez utiliser l’identité attribuée par le système, commencez par [activer l’authentification pour les fonctions Azure](../logic-apps/logic-apps-azure-functions.md#enable-authentication-for-azure-functions).

Ces étapes montrent comment utiliser l’identité managée avec un déclencheur ou une action via le portail Azure. Pour spécifier l’identité managée dans la définition JSON sous-jacente d’un déclencheur ou d’une action, voir [Authentification d’identité managée](../logic-apps/logic-apps-securing-a-logic-app.md#managed-identity-authentication).

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Si ce n’est encore fait, ajoutez [le déclencheur ou l’action prenant en charge les identités managées](logic-apps-securing-a-logic-app.md#managed-identity-authentication).

   Par exemple, le déclencheur ou l’action HTTP peuvent utiliser l’identité attribuée par le système que vous avez activée pour votre application logique. En général, le déclencheur ou l’action HTTP utilisent ces propriétés pour spécifier la ressource ou l’entité auxquelles vous souhaitez accéder :

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Méthode** | Oui | Méthode HTTP utilisée par l’opération que vous souhaitez exécuter |
   | **URI** | Oui | URL de point de terminaison pour accéder à la ressource ou entité Azure cibles. La syntaxe de l’URI comprend généralement l’[ID de ressource](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication) pour la ressource ou le service Azure. |
   | **En-têtes** | Non | Toute valeur d’en-tête que vous devez ou souhaitez inclure dans la demande sortante, telle que le type de contenu |
   | **Requêtes** | Non | Tout paramètre de requête que vous devez ou souhaitez inclure dans la demande, tel que le paramètre pour une opération spécifique ou la version de l’API pour l’opération que vous souhaitez exécuter |
   | **Authentification** | Oui | Type d’authentification à utiliser pour authentifier l’accès à la ressource ou entité cibles |
   ||||

   À titre d’exemple, supposons que vous souhaitez exécuter l’[opération de capture instantanée d’objet blob](/rest/api/storageservices/snapshot-blob) sur un blob dans le compte de Stockage Azure où vous avez précédemment configuré l’accès pour votre identité. Toutefois, le [connecteur de Stockage Blob Azure](/connectors/azureblob/) ne propose pas cette opération actuellement. Au lieu de cela, vous pouvez l’exécuter à l’aide de l’[action HTTP](../logic-apps/logic-apps-workflow-actions-triggers.md#http-action) ou d’une autre [opération de l’API REST du service BLOB](/rest/api/storageservices/operations-on-blobs).

   > [!IMPORTANT]
   > Pour accéder aux comptes de stockage Azure derrière des pare-feu en utilisant des requêtes HTTP et des identités managées, veillez à configurer également votre compte de stockage avec l’[exception qui autorise l’accès de services Microsoft approuvés](../connectors/connectors-create-api-azureblobstorage.md#access-trusted-service).

   Pour exécuter l’[opération de capture instantanée d’objet blob](/rest/api/storageservices/snapshot-blob), l’action HTTP spécifie les propriétés suivantes :

   | Propriété | Obligatoire | Valeur d'exemple | Description |
   |----------|----------|---------------|-------------|
   | **Méthode** | Oui | `PUT`| Méthode HTTP utilisée par l’opération de capture instantanée d’objet blob |
   | **URI** | Oui | `https://{storage-account-name}.blob.core.windows.net/{blob-container-name}/{folder-name-if-any}/{blob-file-name-with-extension}` | ID de ressource d’un fichier de Stockage Blob Azure dans l’environnement global (public) Azure qui utilise cette syntaxe |
   | **En-têtes** | Pour Stockage Azure | `x-ms-blob-type` = `BlockBlob` <p>`x-ms-version` = `2019-02-02` <p>`x-ms-date` = `@{formatDateTime(utcNow(),'r'}` | Les valeurs d’en-tête `x-ms-blob-type`, `x-ms-version` et `x-ms-date` sont requises pour les opérations du service Stockage Azure. <p><p>**Important !** Dans le déclencheur HTTP sortant et les demandes d’action pour Stockage Azure, l’en-tête requiert la propriété `x-ms-version` et la version de l’API pour l’opération que vous souhaitez exécuter. La valeur `x-ms-date` doit être la date actuelle. Dans le cas contraire, votre application logique échoue avec une erreur `403 FORBIDDEN`. Pour obtenir la date actuelle au format requis, vous pouvez utiliser l’expression dans l’exemple de valeur. <p>Pour plus d’informations, consultez les rubriques suivantes : <p><p>- [En-têtes de demande – Capture instantanée d’objet blob](/rest/api/storageservices/snapshot-blob#request) <br>- [Contrôle de version pour les services Stockage Azure](/rest/api/storageservices/versioning-for-the-azure-storage-services#specifying-service-versions-in-requests) |
   | **Requêtes** | Uniquement pour l’opération de capture instantanée de blob | `comp` = `snapshot` | Nom et valeur du paramètre de requête pour l’opération. |
   |||||

   Voici l’exemple d’action HTTP qui affiche toutes ces valeurs de propriété :

   ![Ajouter une action HTTP pour accéder à une ressource Azure](./media/create-managed-service-identity/http-action-example.png)

1. Ajoutez à présent la propriété **Authentification** à l’action HTTP. Dans la liste **Ajouter un nouveau paramètre** , sélectionnez **Authentification**.

   ![Ajouter la propriété « Authentification » à l’action HTTP](./media/create-managed-service-identity/add-authentication-property.png)

   > [!NOTE]
   > Tous les déclencheurs et toutes les actions ne vous permettent pas d’ajouter un type d’authentification. Pour plus d’informations, voir [Ajouter l’authentification aux appels sortants](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound).

1. Dans la liste **Type d’authentification** , sélectionnez **Identité managée**.

   ![Pour « Authentification », sélectionner « Identité managée »](./media/create-managed-service-identity/select-managed-identity.png)

1. Dans la liste des identités managées, sélectionnez les options disponibles en fonction de votre scénario.

   * Si vous configurez l’identité affectée par le système, sélectionnez **Identité managée affectée par le système** , si cela n’est pas déjà fait.

     ![Sélectionner « Identité managée affectée par le système »](./media/create-managed-service-identity/select-system-assigned-identity-for-action.png)

   * Si vous configurez une identité affectée par l’utilisateur, sélectionnez cette identité, si cela n’est pas déjà fait.

     ![Sélectionner l’identité affectée par l’utilisateur](./media/create-managed-service-identity/select-user-assigned-identity-for-action.png)

   Cet exemple se poursuit avec **Identité managée affectée par le système**.

1. Sur certains déclencheurs et actions, la propriété **Audience** apparaît également pour vous permettre de définir l’ID de ressource cible. Affectez à la propriété **Audience** la valeur de l’ [ID de ressource de la ressource ou du service cible](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication). Autrement, par défaut, la propriété **Audience** utilise l’ID de ressource `https://management.azure.com/`, qui est l’ID de ressource pour Azure Resource Manager.

   > [!IMPORTANT]
   > Vérifiez que l’ID de ressource cible *correspond exactement* à la valeur qu’attend Azure Active Directory, y compris les barres obliques de fin obligatoires. Par exemple, l’ID de ressource pour tous les comptes de Stockage Blob Azure requiert une barre oblique finale. Toutefois, l’ID de ressource pour un compte de stockage spécifique ne requiert pas de barre oblique finale. Vérifiez les [ID de ressource des services Azure qui prennent en charge Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

   Cet exemple définit la propriété **Audience** sur `https://storage.azure.com/` afin que les jetons d’accès utilisés pour l’authentification soient valides pour tous les comptes de stockage. Toutefois, vous pouvez également spécifier l’URL du service racine, `https://fabrikamstorageaccount.blob.core.windows.net`, pour un compte de stockage spécifique.

   ![Affecter à la propriété « Audience » l’ID de ressource cible](./media/create-managed-service-identity/specify-audience-url-target-resource.png)

   Pour plus d’informations sur l’autorisation de l’accès avec Azure AD pour Azure Storage, consultez les rubriques suivantes :

   * [Autoriser l’accès aux objets blob et aux files d’attente Azure avec Azure Active Directory](../storage/common/storage-auth-aad.md)
   * [Autoriser l’accès au Stockage Azure avec Azure Active Directory](/rest/api/storageservices/authorize-with-azure-active-directory#use-oauth-access-tokens-for-authentication)

1. Continuez à créer l’application logique comme vous le souhaitez.

<a name="remove-identity"></a>

## <a name="disable-managed-identity"></a>Désactiver l’identité managée

Pour arrêter d’utiliser une identité managée pour votre application logique, vous disposez des options suivantes :

* [Azure portal](#azure-portal-disable)
* [Modèles Microsoft Azure Resource Manager](#template-disable)
* Azure PowerShell
  * [Supprimer une attribution de rôle](../role-based-access-control/role-assignments-powershell.md)
  * [Supprimer une identité affectée par l’utilisateur](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md)
* Azure CLI
  * [Supprimer une attribution de rôle](../role-based-access-control/role-assignments-cli.md)
  * [Supprimer une identité affectée par l’utilisateur](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md)
* API REST Azure
  * [Supprimer une attribution de rôle](../role-based-access-control/role-assignments-rest.md)
  * [Supprimer une identité affectée par l’utilisateur](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-rest.md)

Si vous supprimez votre application logique, Azure supprime automatiquement d’Azure AD l’identité managée.

<a name="azure-portal-disable"></a>

### <a name="disable-managed-identity-in-the-azure-portal"></a>Désactiver l’identité managée dans le portail Azure

Dans le portail Azure, supprimez d’abord l’accès de l’identité à [votre ressource cible](#disable-identity-target-resource). Désactivez ensuite l’identité affectée par le système, ou supprimez l’identité affectée par l’utilisateur de [votre application logique](#disable-identity-logic-app).

<a name="disable-identity-target-resource"></a>

#### <a name="remove-identity-access-from-resources"></a>Supprimer l’accès d’une identité à des ressources

1. Dans le [portail Azure](https://portal.azure.com), accédez à la ressource Azure cible dont vous souhaitez supprimer l’accès à l’identité managée.

1. Dans le menu de la ressource cible, sélectionnez **Contrôle d’accès (IAM)** . Sous la barre d’outils, sélectionnez **Attributions de rôle**.

1. Dans la liste des rôles, sélectionnez les identités managées que vous souhaitez supprimer. Dans la barre d’outils, sélectionnez **Supprimer**.

   > [!TIP]
   > Si l’option **Supprimer** est désactivée, vous n’avez probablement pas les autorisations appropriées. Pour plus d’informations sur les autorisations permettant de gérer des rôles pour des ressources, voir [Autorisations des rôles d’administrateur dans Azure Active Directory](../active-directory/roles/permissions-reference.md).

L’identité managée est maintenant supprimée et n’a plus accès à la ressource cible.

<a name="disable-identity-logic-app"></a>

#### <a name="disable-managed-identity-on-logic-app"></a>Désactiver l’identité managée sur l’application logique

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Dans le menu de l’application logique, sous **Paramètres** , sélectionnez **Identité** , puis suivez les étapes correspondant à votre identité :

   * Sélectionnez **Affecté(e) par le système** > **Actif** > **Enregistrer**. Quand Azure vous invite à confirmer l’opération, sélectionnez **Oui**.

     ![Désactiver l’identité affectée par le système](./media/create-managed-service-identity/disable-system-assigned-identity.png)

   * Sélectionnez **Affecté(e) par l’utilisateur** et l’identité managée, puis sélectionnez **Supprimer**. Quand Azure vous invite à confirmer l’opération, sélectionnez **Oui**.

     ![Supprimer l’identité affectée par l’utilisateur](./media/create-managed-service-identity/remove-user-assigned-identity.png)

L’identité managée est désormais désactivée sur votre application logique.

<a name="template-disable"></a>

### <a name="disable-managed-identity-in-azure-resource-manager-template"></a>Désactiver une identité managée dans un modèle Azure Resource Manager

Si vous avez créé l’identité managée de l’application logique à l’aide d’un modèle Azure Resource Manager, affectez la valeur `None` à la propriété enfant `type` de l’objet `identity`.

```json
"identity": {
   "type": "None"
}
```

## <a name="next-steps"></a>Étapes suivantes

* [Sécuriser l’accès et les données dans Azure Logic Apps](../logic-apps/logic-apps-securing-a-logic-app.md)
