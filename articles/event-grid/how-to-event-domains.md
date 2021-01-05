---
title: Publier des événements avec des domaines d’événements avec Azure Event Grid
description: Explique comment gérer de grands ensembles de rubriques dans Azure Event Grid et y publier des événements à l’aide de domaines d’événements.
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 277db97211b196c9853470c2d12cc2246a4005b2
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92330075"
---
# <a name="manage-topics-and-publish-events-using-event-domains"></a>Gérer des rubriques et publier des événements à l’aide de domaines de l’événement

Cet article montre comment :

* Créer un domaine Event Grid
* S’abonner aux rubriques de grille d’événement
* List keys (Afficher la liste des clés)
* Publier des événements sur un domaine

Pour en savoir plus sur les domaines d’événements, consultez [Comprendre les domaines d’événements pour gérer les rubriques Event Grid](event-domains.md).

[!INCLUDE [requires-azurerm](../../includes/requires-azurerm.md)]

## <a name="install-preview-feature"></a>Installer la fonctionnalité d'évaluation

[!INCLUDE [event-grid-preview-feature-note.md](../../includes/event-grid-preview-feature-note.md)]

## <a name="create-an-event-domain"></a>Créer un domaine d’événement

Pour gérer de grands ensembles de rubriques, créez un domaine d’événements.

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli)

```azurecli-interactive
# If you haven't already installed the extension, do it now.
# This extension is required for preview features.
az extension add --name eventgrid

az eventgrid domain create \
  -g <my-resource-group> \
  --name <my-domain-name> \
  -l <location>
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)
```azurepowershell-interactive
# If you have not already installed the module, do it now.
# This module is required for preview features.
Install-Module -Name AzureRM.EventGrid -AllowPrerelease -Force -Repository PSGallery

New-AzureRmEventGridDomain `
  -ResourceGroupName <my-resource-group> `
  -Name <my-domain-name> `
  -Location <location>
```
---

Une création réussie retourne les valeurs suivantes :

```json
{
  "endpoint": "https://<my-domain-name>.westus2-1.eventgrid.azure.net/api/events",
  "id": "/subscriptions/<sub-id>/resourceGroups/<my-resource-group>/providers/Microsoft.EventGrid/domains/<my-domain-name>",
  "inputSchema": "EventGridSchema",
  "inputSchemaMapping": null,
  "location": "westus2",
  "name": "<my-domain-name>",
  "provisioningState": "Succeeded",
  "resourceGroup": "<my-resource-group>",
  "tags": null,
  "type": "Microsoft.EventGrid/domains"
}
```

Notez les paramètres `endpoint` et `id`, dont vous avez besoin pour gérer le domaine et publier des événements.

## <a name="manage-access-to-topics"></a>Gérer l’accès aux rubriques

La gestion de l’accès aux rubriques est effectuée par le biais d’une [attribution de rôle](../role-based-access-control/role-assignments-cli.md). L’attribution de rôle utilise le contrôle d’accès en fonction du rôle Azure pour limiter les opérations sur les ressources Azure aux utilisateurs autorisés dans une étendue donnée.

Event Grid a deux rôles intégrés que vous pouvez utiliser pour attribuer à des utilisateurs spécifiques l’accès à différentes rubriques au sein d’un domaine. Ces rôles sont `EventGrid EventSubscription Contributor (Preview)`, qui permet de créer et de supprimer des abonnements, et `EventGrid EventSubscription Reader (Preview)`, qui permet uniquement de répertorier les abonnements aux événements.

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli)
La commande Azure CLI suivante limite `alice@contoso.com` à la création et à la suppression d’abonnements aux événements uniquement sur la rubrique `demotopic1` :

```azurecli-interactive
az role assignment create \
  --assignee alice@contoso.com \
  --role "EventGrid EventSubscription Contributor (Preview)" \
  --scope /subscriptions/<sub-id>/resourceGroups/<my-resource-group>/providers/Microsoft.EventGrid/domains/<my-domain-name>/topics/demotopic1
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)
La commande PowerShell suivante limite `alice@contoso.com` à la création et à la suppression d’abonnements aux événements uniquement sur la rubrique `demotopic1` :

```azurepowershell-interactive
New-AzureRmRoleAssignment `
  -SignInName alice@contoso.com `
  -RoleDefinitionName "EventGrid EventSubscription Contributor (Preview)" `
  -Scope /subscriptions/<sub-id>/resourceGroups/<my-resource-group>/providers/Microsoft.EventGrid/domains/<my-domain-name>/topics/demotopic1
```
---

Pour plus d’informations sur la gestion des accès pour les opérations Event Grid, consultez la page [Sécurité et authentification Azure Event Grid](./security-authentication.md).

## <a name="create-topics-and-subscriptions"></a>Créer des rubriques et des abonnements

Le service Event Grid crée et gère automatiquement la rubrique correspondante dans un domaine en fonction de l’appel destiné à créer un abonnement aux événements pour une rubrique de domaine. Il n’existe pas d’autre étape pour créer une rubrique dans un domaine. De même, quand le dernier abonnement aux événements pour une rubrique est supprimé, celle-ci est également supprimée.

L’abonnement à une rubrique dans un domaine est identique à l’abonnement à toute autre ressource Azure. Pour un ID de ressource source, spécifiez l’ID de domaine d’événements retourné au moment de la création du domaine. Pour spécifier la rubrique à laquelle vous voulez vous abonner, ajoutez `/topics/<my-topic>` à la fin de l’ID de ressource de la source. Pour créer un abonnement d’événement de portée de domaine qui reçoit tous les événements dans le domaine, indiquez l’ID de domaine d’événements sans spécifier de rubriques.

Généralement, l’utilisateur auquel vous avez accordé l’accès dans la section précédente créera l’abonnement. Pour plus de simplicité dans cet article, c’est vous qui créez l’abonnement. 

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli)

```azurecli-interactive
az eventgrid event-subscription create \
  --name <event-subscription> \
  --source-resource-id "/subscriptions/<sub-id>/resourceGroups/<my-resource-group>/providers/Microsoft.EventGrid/domains/<my-domain-name>/topics/demotopic1" \
  --endpoint https://contoso.azurewebsites.net/api/updates
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```azurepowershell-interactive
New-AzureRmEventGridSubscription `
  -ResourceId "/subscriptions/<sub-id>/resourceGroups/<my-resource-group>/providers/Microsoft.EventGrid/domains/<my-domain-name>/topics/demotopic1" `
  -EventSubscriptionName <event-subscription> `
  -Endpoint https://contoso.azurewebsites.net/api/updates
```

---

Si vous avez besoin d’un point de terminaison de test auquel abonner vos événements, vous pouvez toujours déployer une [application web prédéfinie](https://github.com/Azure-Samples/azure-event-grid-viewer) qui affiche les événements entrants. Vous pouvez envoyer vos événements à votre site web de test à l’adresse `https://<your-site-name>.azurewebsites.net/api/updates`.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fazure-event-grid-viewer%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"  alt="Button to Deploy to Aquent." /></a>

Les autorisations définies pour une rubrique sont stockées dans Azure Active Directory et doivent être supprimées explicitement. La suppression d’un abonnement aux événements ne révoque pas l’accès des utilisateurs à la création d’abonnements aux événements s’ils ont un accès en écriture à une rubrique.


## <a name="publish-events-to-an-event-grid-domain"></a>Publier des événements sur un domaine Event Grid

La publication d’événements sur un domaine est similaire à la [publication sur une rubrique personnalisée](./post-to-custom-topic.md). Toutefois, au lieu de publier dans la rubrique personnalisée, vous publiez tous les événements sur le point de terminaison de domaine. Dans les données d’événement JSON, vous spécifiez la rubrique dans laquelle placer les événements. Dans le tableau d’événements suivant, l’événement avec `"id": "1111"` est destiné à la rubrique `demotopic1`, alors que l’événement avec `"id": "2222"` est destiné à la rubrique `demotopic2` :

```json
[{
  "topic": "demotopic1",
  "id": "1111",
  "eventType": "maintenanceRequested",
  "subject": "myapp/vehicles/diggers",
  "eventTime": "2018-10-30T21:03:07+00:00",
  "data": {
    "make": "Contoso",
    "model": "Small Digger"
  },
  "dataVersion": "1.0"
},
{
  "topic": "demotopic2",
  "id": "2222",
  "eventType": "maintenanceCompleted",
  "subject": "myapp/vehicles/tractors",
  "eventTime": "2018-10-30T21:04:12+00:00",
  "data": {
    "make": "Contoso",
    "model": "Big Tractor"
  },
  "dataVersion": "1.0"
}]
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli)
Pour obtenir le point de terminaison de domaine avec Azure CLI, utilisez

```azurecli-interactive
az eventgrid domain show \
  -g <my-resource-group> \
  -n <my-domain>
```

Pour obtenir les clés d’un domaine, utilisez :

```azurecli-interactive
az eventgrid domain key list \
  -g <my-resource-group> \
  -n <my-domain>
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)
Pour obtenir le point de terminaison de domaine avec PowerShell, utilisez

```azurepowershell-interactive
Get-AzureRmEventGridDomain `
  -ResourceGroupName <my-resource-group> `
  -Name <my-domain>
```

Pour obtenir les clés d’un domaine, utilisez :

```azurepowershell-interactive
Get-AzureRmEventGridDomainKey `
  -ResourceGroupName <my-resource-group> `
  -Name <my-domain>
```
---

Ensuite, publiez vos événements sur votre domaine Event Grid en effectuant une requête HTTP POST selon la méthode de votre choix.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les concepts généraux des domaines d’événement et sur ce qui fait leur utilité, consultez la [vue d’ensemble conceptuelle des domaines d’événement](event-domains.md).
