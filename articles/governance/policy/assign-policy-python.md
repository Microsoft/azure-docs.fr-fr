---
title: "Démarrage rapide : Nouvelle attribution de stratégie à l'aide de Python"
description: Dans ce guide de démarrage rapide, vous allez utiliser Python pour créer une attribution Azure Policy afin d'identifier les ressources non conformes.
ms.date: 10/14/2020
ms.topic: quickstart
ms.custom: devx-track-python, devx-track-azurecli
ms.openlocfilehash: aa67b3a2bae2beb62daa6e64871411ee760f802b
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94832194"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-using-python"></a>Démarrage rapide : Créer une attribution de stratégie pour identifier les ressources non conformes à l'aide de Python

La première étape pour comprendre la conformité dans Azure consiste à identifier l’état de vos ressources. Dans ce démarrage rapide, vous créerez une affectation de stratégie pour identifier les machines virtuelles qui n’utilisent pas de disques managés. Lorsque vous aurez terminé, vous identifierez les machines virtuelles qui ne sont _pas conformes_.

La bibliothèque Python permet de gérer les ressources Azure à partir de la ligne de commande ou à l'aide de scripts. Ce guide explique comment utiliser une bibliothèque Python pour créer une attribution de stratégie.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-policy-library"></a>Ajouter la bibliothèque Azure Policy

Pour permettre à Python de fonctionner avec Azure Policy, la bibliothèque doit être ajoutée. Cette bibliothèque fonctionne partout où Python peut être utilisé, notamment avec [bash sur Windows 10](/windows/wsl/install-win10) ou via une installation locale.

1. Vérifiez que la version la plus récente de Python est installée (**3.8** au minimum). Si ce n’est pas le cas, téléchargez-la à l’adresse [Python.org](https://www.python.org/downloads/).

1. Vérifiez que la version la plus récente d’Azure CLI est installée (**2.5.1** au minimum). Si ce n’est pas le cas, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

   > [!NOTE]
   > Azure CLI est requis pour permettre à Python d’utiliser **l’authentification basée sur l’interface CLI** dans les exemples suivants. Pour plus d’informations sur les autres options, consultez [S’authentifier à l’aide des bibliothèques de gestion Azure pour Python](/azure/developer/python/azure-sdk-authenticate).

1. S’authentifier par le biais d’Azure CLI.

   ```azurecli
   az login
   ```

1. Dans votre environnement Python, installez les bibliothèques requises pour Azure Resource Graph :

   ```bash
   # Add the Python library for Python
   pip install azure-mgmt-policyinsights

   # Add the Resources library for Python
   pip install azure-mgmt-resource

   # Add the CLI Core library for Python for authentication (development only!)
   pip install azure-cli-core
   ```

   > [!NOTE]
   > Si Python est installé pour tous les utilisateurs, cette commande doit être exécutée à partir d’une console avec élévation de privilèges.

1. Vérifiez que les bibliothèques ont été installées. `azure-mgmt-policyinsights` doit avoir la version **0.5.0** ou supérieure, `azure-mgmt-resource` doit avoir la version **9.0.0** ou supérieure, et `azure-cli-core` doit avoir la version **2.5.0** ou supérieure.

   ```bash
   # Check each installed library
   pip show azure-mgmt-policyinsights azure-mgmt-resource azure-cli-core
   ```

## <a name="create-a-policy-assignment"></a>Créer une affectation de stratégie

Dans ce guide de démarrage rapide, vous allez créer une attribution de stratégie et attribuer la définition **Auditer les machines virtuelles qui n'utilisent aucun disque managé** (`06a78e20-9358-41c9-923c-fb736d382a4d`). Cette stratégie identifie les ressources qui ne sont pas conformes aux conditions définies dans sa définition.

Exécutez le code suivant pour créer une nouvelle attribution de stratégie :

```python
# Import specific methods and models from other libraries
from azure.common.credentials import get_azure_cli_credentials
from azure.common.client_factory import get_client_from_cli_profile
from azure.mgmt.resource.policy import PolicyClient
from azure.mgmt.resource.policy.models import PolicyAssignment

# Get your credentials from Azure CLI (development only!) and get your subscription list
policyClient = get_client_from_cli_profile(PolicyClient)

# Create details for the assignment
policyAssignmentDetails = PolicyAssignment(display_name="Audit VMs without managed disks Assignment", policy_definition_id="/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d", scope="{scope}", description="Shows all virtual machines not using managed disks")

# Create new policy assignment
policyAssignment = policyClient.policy_assignments.create("{scope}", "audit-vm-manageddisks", policyAssignmentDetails)

# Show results
print(policyAssignment)
```

Les commandes précédentes utilisent les informations suivantes :

Détails de l'attribution :
- **display_name** : nom d'affichage de l'attribution de stratégie. Dans ce cas, nous allons utiliser l’affectation _Audit VMs without managed disks_ (Auditer les machines virtuelles sans disques managés).
- **policy_definition_id** : chemin de la définition de stratégie, sur lequel vous vous basez pour créer l'attribution. Dans ce cas, il s’agit de l’ID de la définition de stratégie _Auditer les machines virtuelles qui n’utilisent pas de disques managés_. Dans cet exemple, la définition de stratégie est intégrée et le chemin n'inclut pas d'informations sur le groupe d'administration ou l'abonnement.
- **scope** : une étendue détermine les ressources ou le groupe de ressources sur lesquels l'attribution de stratégie est appliquée. Cette étendue peut aller d'un groupe d'administration à une ressource individuelle. Veillez à remplacer `{scope}` par l'un des modèles suivants :
  - Groupe d'administration : `/providers/Microsoft.Management/managementGroups/{managementGroup}`
  - Abonnement : `/subscriptions/{subscriptionId}`
  - Groupe de ressources : `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`
  - Ressource : `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/[{parentResourcePath}/]`
- **description** : explication plus détaillée de ce que fait la stratégie ou de la raison pour laquelle elle est attribuée à cette étendue.

Création d'attribution :

- Étendue : cette étendue détermine où l'attribution de stratégie est enregistrée. L'étendue définie dans les détails de l'attribution doit se trouver dans cette étendue.
- Name : nom réel de l’attribution. Pour cet exemple, _audit-vm-manageddisks_ a été utilisé.
- Attribution de stratégie : objet Python **PolicyAssignment** créé à l'étape précédente.

Vous êtes maintenant prêt à identifier les ressources non conformes pour comprendre l’état de conformité de votre environnement.

## <a name="identify-non-compliant-resources"></a>Identifier des ressources non conformes

Utilisez les informations suivantes pour identifier les ressources qui ne sont pas conformes à l’attribution de stratégie que vous avez créée. Exécutez le code ci-dessous :

```python
# Import specific methods and models from other libraries
from azure.common.client_factory import get_client_from_cli_profile
from azure.mgmt.policyinsights._policy_insights_client import PolicyInsightsClient
from azure.mgmt.policyinsights.models import QueryOptions

# Get your credentials from Azure CLI (development only!) and get your subscription list
policyInsightsClient = get_client_from_cli_profile(PolicyInsightsClient)

# Set the query options
queryOptions = QueryOptions(filter="IsCompliant eq false and PolicyAssignmentId eq 'audit-vm-manageddisks'",apply="groupby((ResourceId))")

# Fetch 'latest' results for the subscription
results = policyInsightsClient.policy_states.list_query_results_for_subscription(policy_states_resource="latest", subscription_id="{subscriptionId}", query_options=queryOptions)

# Show results
print(results)
```

Remplacez `{subscriptionId}` par l'abonnement dont vous souhaitez voir les résultats de conformité pour cette attribution de stratégie. Pour obtenir la liste des autres étendues et méthodes permettant de synthétiser les données, consultez [Méthodes relatives à l'état de la stratégie](/python/api/azure-mgmt-policyinsights/azure.mgmt.policyinsights.operations.policystatesoperations#methods).

Vos résultats doivent ressembler à l’exemple suivant :

```output
{
    'additional_properties': {
        '@odata.nextLink': None
    },
    'odatacontext': 'https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest',
    'odatacount': 12,
    'value': [{data}]
}
```

Les résultats correspondent à ce que vous voyez sous l’onglet **Conformité des ressources** d’une attribution de stratégie dans la vue du portail Azure.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Utilisez la commande suivante pour supprimer l’affectation créée :

```python
# Import specific methods and models from other libraries
from azure.common.client_factory import get_client_from_cli_profile
from azure.mgmt.resource.policy import PolicyClient

# Get your credentials from Azure CLI (development only!) and get your subscription list
policyClient = get_client_from_cli_profile(PolicyClient)

# Delete the policy assignment
policyAssignment = policyClient.policy_assignments.delete("{scope}", "audit-vm-manageddisks")

# Show results
print(policyAssignment)
```

Remplacez `{scope}` par l'étendue que vous avez utilisée pour créer l'attribution de stratégie.

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez affecté une définition de stratégie pour identifier les ressources non conformes de votre environnement Azure.

Pour en savoir plus sur l'attribution de définitions de stratégie visant à vérifier que les nouvelles ressources sont conformes, suivez le tutoriel :

> [!div class="nextstepaction"]
> [Création et gestion des stratégies](./tutorials/create-and-manage.md)
