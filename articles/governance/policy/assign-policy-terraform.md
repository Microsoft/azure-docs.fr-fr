---
title: 'Démarrage rapide : Nouvelle attribution de stratégie à l’aide de Terraform'
description: Dans ce guide de démarrage rapide, vous allez utiliser Terraform et la syntaxe HCL pour créer une attribution de stratégie dans le but d’identifier les ressources non conformes.
ms.date: 10/27/2020
ms.topic: quickstart
ms.openlocfilehash: dc4dae2dc6e43e7532117bf64af3ce97ddc7c496
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104907"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-using-terraform"></a>Démarrage rapide : Créer une attribution de stratégie pour identifier les ressources non conformes à l'aide de Terraform

La première étape pour comprendre la conformité dans Azure consiste à identifier l’état de vos ressources.
Ce démarrage rapide vous guide pas à pas dans le processus de création d’une attribution de stratégie pour identifier les machines virtuelles qui n’utilisent pas de disques managés.

À la fin de ce processus, vous aurez identifié correctement les machines virtuelles qui n’utilisent pas de disques managés. Elles sont _non conformes_ à l’attribution de stratégie.

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.
- [Terraform](https://www.terraform.io/) version 0.12.0 ou supérieure configurée dans votre environnement.
  Pour obtenir des instructions, consultez [Configurer Terraform avec Azure Cloud Shell](/azure/developer/terraform/get-started-cloud-shell).
- Pour suivre ce guide de démarrage rapide, vous devez utiliser Azure CLI version 2.13.0 ou ultérieure. Pour connaître la version de l’interface, exécutez `az --version`. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli).

## <a name="create-the-terraform-configuration-variable-and-output-file"></a>Créer les fichiers de configuration, de variables et de sortie Terraform

Dans ce démarrage rapide, vous allez créer une attribution de stratégie et attribuer la définition de stratégie **Auditer les machines virtuelles n’utilisant aucun disque managé.** (`06a78e20-9358-41c9-923c-fb736d382a4d`). Cette stratégie identifie les ressources qui ne sont pas conformes aux conditions définies dans sa définition.

Tout d’abord, configurez les fichiers de configuration, de variables et de sortie Terraform. Les ressources Terraform pour Azure Policy utilisent le [fournisseur Azure](https://www.terraform.io/docs/providers/azurerm/index.html).

1. Créez un dossier nommé `policy-assignment` et accédez à ses répertoires.

1. Créez `main.tf` avec le code suivant :

   ```hcl
   provider "azurerm" {
       version = "~>2.0"
       features {}
   }
   
   resource "azurerm_policy_assignment" "auditvms" {
       name = "audit-vm-manageddisks"
       scope = var.cust_scope
       policy_definition_id = "/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d"
       description = "Shows all virtual machines not using managed disks"
       display_name = "Audit VMs without managed disks Assignment"
   }
   ```
1. Créez `variables.tf` avec le code suivant :

   ```hcl
   variable "cust_scope" {
       default = "{scope}"
   }
   ```

   Une étendue détermine les ressources ou le regroupement de ressources sur lequel la stratégie est appliquée. Cette étendue peut aller d’un groupe d’administration à une ressource individuelle. Veillez à remplacer `{scope}` par l’un des modèles suivants :

   - Abonnement : `/subscriptions/{subscriptionId}`
   - Groupe de ressources : `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`
   - Ressource : `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/[{parentResourcePath}/]`

1. Créez `output.tf` avec le code suivant :

   ```hcl
   output "assignment_id" {
       value = azurerm_policy_assignment.auditvms.id
   }
   ```

## <a name="initialize-terraform-and-create-plan"></a>Initialiser Terraform et créer un plan

Initialisez ensuite Terraform pour télécharger les fournisseurs nécessaires, puis créez un plan.

1. Exécutez la commande [terraform init](https://www.terraform.io/docs/commands/init.html). Cette commande télécharge les modules Azure nécessaires pour créer les ressources Azure dans la configuration Terraform.

   ```bash
   terraform init
   ```

   :::image type="content" source="./media/assign-policy-terraform/terraform-initialize.png" alt-text="Capture d’écran de l’exécution de la commande terraform init qui montre le téléchargement du module azurerm et un message de réussite.":::

1. Authentifiez-vous auprès de Terraform à l’aide d’[Azure CLI](/cli/azure/). Pour plus d’informations, consultez [Fournisseur Azure : Authentification à l’aide d’Azure CLI](https://www.terraform.io/docs/providers/azurerm/guides/azure_cli.html).

   ```bash
   az login
   ```

1. Créez le plan d’exécution avec la commande [terraform plan](https://www.terraform.io/docs/commands/plan.html) et le paramètre **out**.

   ```bash
   terraform plan -out assignment.tfplan
   ```

   :::image type="content" source="./media/assign-policy-terraform/terraform-plan-out.png" alt-text="Capture d’écran de l’exécution de la commande terraform plan avec le paramètre out qui montre la ressource Azure créée.":::

   > [!NOTE]
   > Pour plus d’informations sur la conservation des plans d’exécution et sur la sécurité, consultez [Plan Terraform : Avertissement de sécurité](https://www.terraform.io/docs/commands/plan.html#security-warning).

## <a name="apply-the-terraform-execution-plan"></a>Appliquer le plan d’exécution Terraform

Enfin, appliquez le plan d’exécution.

Exécutez la commande [terraform apply](https://www.terraform.io/docs/commands/apply.html) en spécifiant le plan `assignment.tfplan` créé précédemment.

```bash
terraform apply assignment.tfplan
```

:::image type="content" source="./media/assign-policy-terraform/terraform-apply.png" alt-text="Capture d’écran de l’exécution de la commande terraform apply et de la création de la ressource résultante.":::

Quand le message « Apply complete! Ressources : 1 added, 0 changed, 0 destroyed. » s’affiche, cela indique que l’attribution de stratégie a été créée. Étant donné que nous avons défini le fichier `outputs.tf`, la valeur _assignment\_id_ est également retournée.

## <a name="identify-non-compliant-resources"></a>Identifier des ressources non conformes

Pour voir les ressources qui ne sont pas compatibles avec cette nouvelle attribution, utilisez la valeur _assignment\_id_ retournée par `terraform apply`. Avec cette valeur, exécutez la commande suivante pour obtenir les ID des ressources non conformes, dans un fichier de sortie JSON :

```console
armclient post "/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$filter=IsCompliant eq false and PolicyAssignmentId eq '<policyAssignmentID>'&$apply=groupby((ResourceId))" > <json file to direct the output with the resource IDs into>
```

Vos résultats doivent ressembler à l’exemple suivant :

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest",
    "@odata.count": 3,
    "value": [{
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachineId>"
        },
        {
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine2Id>"
        },
        {
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionName>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine3ID>"
        }

    ]
}
```

Les résultats sont comparables à ce que vous devriez généralement voir sous **Ressources non conformes** dans la vue du portail Azure.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer l’attribution créée, utilisez Azure CLI ou annulez le plan d’exécution Terraform avec `terraform destroy`.

- Azure CLI

  ```azurecli-interactive
  az policy assignment delete --name 'audit-vm-manageddisks' --scope '/subscriptions/<subscriptionID>/<resourceGroupName>'
  ```

- Terraform

  ```bash
  terraform destroy assignment.tfplan
  ```

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez affecté une définition de stratégie pour identifier les ressources non conformes de votre environnement Azure.

Pour en savoir plus sur l’affectation de stratégies visant à vérifier que les nouvelles ressources sont conformes, suivez le tutoriel :

> [!div class="nextstepaction"]
> [Création et gestion des stratégies](./tutorials/create-and-manage.md)
