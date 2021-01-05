---
title: Déplacer des ressources vers un nouvel abonnement ou un nouveau groupe de ressources
description: Utilisez Azure Resource Manager ou une API REST pour déplacer des ressources vers un nouveau groupe de ressources ou abonnement.
ms.topic: conceptual
ms.date: 09/15/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: cd05fe045532ee1b1f1fb88e502d786daabf9365
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91319552"
---
# <a name="move-resources-to-a-new-resource-group-or-subscription"></a>Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement

Cet article vous montre comment déplacer des ressources Azure vers un autre abonnement Azure ou un autre groupe de ressources sous le même abonnement. Vous pouvez utiliser le portail Azure, Azure PowerShell, Azure CLI ou l’API REST pour déplacer des ressources.

Le groupe source et le groupe cible sont verrouillés pendant l’opération de déplacement. Les opérations d’écriture et de suppression sont bloquées sur les groupes de ressources tant que le déplacement n’est pas terminé. Ce verrou signifie que vous ne pouvez pas ajouter, mettre à jour ou supprimer des ressources dans les groupes de ressources. Cela ne signifie pas que les ressources sont figées. Par exemple, si vous déplacez un serveur SQL Server et sa base de données vers un nouveau groupe de ressources, une application qui utilise la base de données ne rencontre aucune interruption de service. Elle peut toujours lire et écrire dans la base de données. Le verrou peut durer jusqu’à quatre heures, mais la plupart des déplacements se terminent en moins de temps.

Déplacer une ressource la déplace seulement vers un nouveau groupe de ressources ou un nouvel abonnement. Cette opération ne change pas l’emplacement de la ressource.

## <a name="checklist-before-moving-resources"></a>Liste de contrôle avant le déplacement de ressources

Plusieurs étapes importantes doivent être effectuées avant de déplacer une ressource. Vérifiez ces conditions pour prévenir d'éventuelles erreurs.

1. Les ressources que vous voulez déplacer doivent prendre en charge l’opération de déplacement. Pour obtenir la liste des ressources prenant en charge l’opération de déplacement, consultez [Prise en charge de l’opération de déplacement pour les ressources](move-support-resources.md).

1. Certains services ont des limitations ou des exigences spécifiques lors du déplacement des ressources. Si vous déplacez l’un des services suivants, consultez d’abord ces conseils.

   * Si vous utilisez Azure Stack Hub, vous ne pouvez pas déplacer de ressources entre des groupes.
   * [Conseils pour le déplacement d’App Services](./move-limitations/app-service-move-limitations.md)
   * [Conseils pour le déplacement de DevOps Services](/azure/devops/organizations/billing/change-azure-subscription?toc=/azure/azure-resource-manager/toc.json)
   * [Conseils pour le déplacement de modèles de déploiement classique](./move-limitations/classic-model-move-limitations.md) : Calcul classique, Stockage classique, Réseaux virtuels classiques et services Cloud
   * [Aide pour le déplacement du réseau](./move-limitations/networking-move-limitations.md)
   * [Conseils pour le déplacement de Recovery Services](../../backup/backup-azure-move-recovery-services-vault.md?toc=/azure/azure-resource-manager/toc.json)
   * [Conseils pour le déplacement de machines virtuelles](./move-limitations/virtual-machines-move-limitations.md)

1. Si vous déplacez une ressource à laquelle un rôle Azure est affecté directement (ou est affecté à une ressource enfant de cette ressource), l’attribution de rôle n’est pas déplacée et devient orpheline. Après le déplacement, vous devez recréer l’attribution de rôle. Finalement, l’attribution de rôle orpheline sera automatiquement supprimée, mais il est recommandé de supprimer l’attribution de rôle avant de déplacer la ressource.

    Pour plus d’informations sur la gestion des attributions de rôles, consultez [Répertorier les attributions de rôle](../../role-based-access-control/role-assignments-list-portal.md#list-role-assignments-at-a-scope) et [Ajouter ou supprimer des attributions de rôle](../../role-based-access-control/role-assignments-portal.md).

1. Les abonnements source et de destination doivent être actifs. Si vous rencontrez des problèmes lors de l’activation d’un compte qui a été désactivé, [créez une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). Sélectionnez **Gestion des abonnements** comme type de problème.

1. Les abonnements source et de destination doivent exister dans le même [client Azure Active Directory](../../active-directory/develop/quickstart-create-new-tenant.md). Pour vérifier que les deux abonnements ont le même ID client, utilisez Azure PowerShell ou Azure CLI.

   Pour Azure PowerShell, utilisez :

   ```azurepowershell-interactive
   (Get-AzSubscription -SubscriptionName <your-source-subscription>).TenantId
   (Get-AzSubscription -SubscriptionName <your-destination-subscription>).TenantId
   ```

   Pour l’interface de ligne de commande Azure, consultez :

   ```azurecli-interactive
   az account show --subscription <your-source-subscription> --query tenantId
   az account show --subscription <your-destination-subscription> --query tenantId
   ```

   Si les ID client pour les abonnements source et de destination ne sont pas identiques, utilisez les méthodes suivantes pour rapprocher les ID client :

   * [Transfert de la propriété d’un abonnement Azure à un autre compte](../../cost-management-billing/manage/billing-subscription-transfer.md)
   * [Associer ou ajouter un abonnement Azure à Azure Active Directory](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md)

1. L’abonnement de destination doit être inscrit pour le fournisseur de la ressource déplacée. Sinon, vous recevez une erreur indiquant que **l’abonnement n’est pas inscrit pour un type de ressource**. Vous pouvez rencontrer cette erreur lors du déplacement d’une ressource vers un nouvel abonnement qui n’a jamais été utilisé avec ce type de ressource.

   Pour PowerShell, utilisez les commandes suivantes pour obtenir l’état de l’inscription :

   ```azurepowershell-interactive
   Set-AzContext -Subscription <destination-subscription-name-or-id>
   Get-AzResourceProvider -ListAvailable | Select-Object ProviderNamespace, RegistrationState
   ```

   Pour inscrire un fournisseur de ressources, utilisez :

   ```azurepowershell-interactive
   Register-AzResourceProvider -ProviderNamespace Microsoft.Batch
   ```

   Pour l’interface CLI d’Azure, utilisez les commandes suivantes pour obtenir l’état de l’inscription :

   ```azurecli-interactive
   az account set -s <destination-subscription-name-or-id>
   az provider list --query "[].{Provider:namespace, Status:registrationState}" --out table
   ```

   Pour inscrire un fournisseur de ressources, utilisez :

   ```azurecli-interactive
   az provider register --namespace Microsoft.Batch
   ```

1. Le compte déplaçant les ressources doit avoir au moins les autorisations suivantes :

   * **Microsoft.Resources/subscriptions/resourceGroups/moveResources/action** sur le groupe de ressources source.
   * **Microsoft.Resources/subscriptions/resourceGroups/write** sur le groupe de ressources de destination.

1. Avant de déplacer les ressources, vérifiez les quotas d’abonnement pour l’abonnement vers lequel vous souhaitez déplacer les ressources. Si le déplacement des ressources signifie que l’abonnement dépassera ses limites, vous devez vérifier si vous pouvez demander une augmentation du quota. Pour connaître la liste des limites et savoir comment demander une augmentation, consultez [Abonnement Azure et limites, quotas et contraintes du service](../../azure-resource-manager/management/azure-subscription-service-limits.md).

1. **Pour un déplacement entre des abonnements, la ressource et ses ressources dépendantes doivent se trouver dans le même groupe de ressources et être déplacées ensemble.** Par exemple, une machine virtuelle avec des disques managés nécessite le déplacement simultané de la machine virtuelle, des disques managés ainsi que des autres ressources dépendantes.

   Si vous déplacez une ressource vers un nouvel abonnement, vérifiez si la ressource a des ressources dépendantes et si elles se trouvent dans le même groupe de ressources. Si les ressources ne se trouvent pas dans le même groupe de ressources, vérifiez si elles peuvent y être combinées. Dans ce cas, mettez toutes ces ressources dans le même groupe de ressources à l’aide d’une opération de déplacement entre les groupes de ressources.

   Pour plus d’informations, consultez [Scénario de déplacement entre des abonnements](#scenario-for-move-across-subscriptions).

## <a name="scenario-for-move-across-subscriptions"></a>Scénario de déplacement entre différents abonnements

Le déplacement de ressources d’un abonnement vers un autre est un processus en trois étapes :

![scénario de déplacement entre des abonnements](./media/move-resource-group-and-subscription/cross-subscription-move-scenario.png)

Pour illustrer cela, nous avons une seule ressource dépendante.

* Étape 1 : Si les ressources dépendantes sont réparties entre différents groupes de ressources, commencez par les déplacer vers un même groupe de ressources.
* Étape 2 : Déplacez la ressource et les ressources dépendantes ensemble de l’abonnement source vers l’abonnement cible.
* Étape 3 : Si vous le souhaitez, redistribuez les ressources dépendantes dans différents groupes de ressources dans l’abonnement cible.

## <a name="validate-move"></a>Valider le déplacement

[L’opération de validation du déplacement](/rest/api/resources/resources/validatemoveresources) vous permet de tester votre scénario de déplacement sans réellement déplacer les ressources. Utilisez cette opération pour vérifier si le déplacement va réussir. La validation est appelée automatiquement quand vous envoyez une demande de déplacement. Utilisez cette opération seulement quand vous devez prédéterminer les résultats. Pour exécuter cette opération, vous avez besoin des éléments suivants :

* Nom du groupe de ressources source
* ID de ressource du groupe de ressources cible
* ID de ressource de chaque ressource à déplacer
* [Jeton d’accès](/rest/api/azure/#acquire-an-access-token) pour votre compte

Envoyez la requête suivante :

```HTTP
POST https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<source-group>/validateMoveResources?api-version=2019-05-10
Authorization: Bearer <access-token>
Content-type: application/json
```

Avec le corps de la demande :

```json
{
 "resources": ["<resource-id-1>", "<resource-id-2>"],
 "targetResourceGroup": "/subscriptions/<subscription-id>/resourceGroups/<target-group>"
}
```

Si la demande est correctement mise en forme, l’opération retourne :

```HTTP
Response Code: 202
cache-control: no-cache
pragma: no-cache
expires: -1
location: https://management.azure.com/subscriptions/<subscription-id>/operationresults/<operation-id>?api-version=2018-02-01
retry-after: 15
...
```

Le code d’état 202 indique que la demande de validation a été acceptée, mais la réussite de l’opération de déplacement n’est pas encore déterminée. La valeur de `location` contient une URL que vous utilisez pour vérifier l’état de l’opération longue.  

Pour vérifier l’état, envoyez la demande suivante :

```HTTP
GET <location-url>
Authorization: Bearer <access-token>
```

Pendant l’exécution de l’opération, vous continuez à recevoir le code d’état 202. Attendez le nombre de secondes indiqué dans la valeur `retry-after` avant de réessayer. Si l’opération de déplacement est validée, vous recevez le code d’état 204. Si la validation de déplacement échoue, vous recevez un message d’erreur, par exemple :

```json
{"error":{"code":"ResourceMoveProviderValidationFailed","message":"<message>"...}}
```

## <a name="use-the-portal"></a>Utiliser le portail

Pour déplacer des ressources, sélectionnez le groupe de ressources qui contient ces ressources.

Lorsque vous affichez le groupe de ressources, l’option Déplacer est désactivée.

:::image type="content" source="./media/move-resource-group-and-subscription/move-first-view.png" alt-text="option Déplacer désactivée":::

Pour activer l’option Déplacer, sélectionnez les ressources que vous souhaitez déplacer. Pour sélectionner toutes les ressources, activez la case à cocher en haut de la liste. Ou sélectionnez les ressources individuellement. Après avoir sélectionné les ressources, l’option Déplacer est activée.

:::image type="content" source="./media/move-resource-group-and-subscription/select-resources.png" alt-text="option Déplacer désactivée":::

Sélectionnez le bouton **Déplacer**.

:::image type="content" source="./media/move-resource-group-and-subscription/move-options.png" alt-text="option Déplacer désactivée":::

Ce bouton vous donne trois options :

* Déplacer vers un nouveau groupe de ressources.
* Accéder à un nouvel abonnement.
* Déplacez dans une nouvelle région. Pour modifier des régions, consultez [Déplacer des ressources entre les régions (à partir du groupe de ressources)](../../resource-mover/move-region-within-resource-group.md?toc=/azure/azure-resource-manager/management/toc.json).

Indiquez si vous déplacez les ressources vers un nouveau groupe de ressources ou vers un nouvel abonnement.

Sélectionnez le groupe de ressources de destination. Confirmez que vous devez mettre à jour les scripts de ces ressources et sélectionnez **OK**. Si vous avez choisi de passer à un nouvel abonnement, vous devez également sélectionner l’abonnement de destination.

:::image type="content" source="./media/move-resource-group-and-subscription/move-destination.png" alt-text="option Déplacer désactivée":::

Après avoir vérifié que les ressources peuvent être déplacées, vous voyez une notification indiquant que l’opération de déplacement est en cours d’exécution.

:::image type="content" source="./media/move-resource-group-and-subscription/move-notification.png" alt-text="option Déplacer désactivée":::

Lorsque l’opération est terminée, vous êtes informé du résultat.

Si vous recevez une erreur, consultez [Résoudre les problèmes liés au déplacement de ressources vers un nouveau groupe de ressource ou un nouvel abonnement](troubleshoot-move.md).

## <a name="use-azure-powershell"></a>Utilisation d'Azure PowerShell

Pour déplacer des ressources existantes vers un autre groupe de ressources ou un autre abonnement, utilisez la commande [Move-AzResource](/powershell/module/az.resources/move-azresource). L’exemple suivant vous indique comment déplacer plusieurs ressources vers un nouveau groupe de ressources.

```azurepowershell-interactive
$webapp = Get-AzResource -ResourceGroupName OldRG -ResourceName ExampleSite
$plan = Get-AzResource -ResourceGroupName OldRG -ResourceName ExamplePlan
Move-AzResource -DestinationResourceGroupName NewRG -ResourceId $webapp.ResourceId, $plan.ResourceId
```

Pour déplacer des ressources vers un nouvel abonnement, renseignez une valeur pour le paramètre `DestinationSubscriptionId`.

Si vous recevez une erreur, consultez [Résoudre les problèmes liés au déplacement de ressources vers un nouveau groupe de ressource ou un nouvel abonnement](troubleshoot-move.md).

## <a name="use-azure-cli"></a>Utiliser l’interface de ligne de commande Microsoft Azure

Pour déplacer des ressources existantes vers un autre groupe de ressources ou un autre abonnement, exécutez la commande [az resource move](/cli/azure/resource#az-resource-move) . Fournissez les ID des ressources à déplacer. L’exemple suivant vous indique comment déplacer plusieurs ressources vers un nouveau groupe de ressources. Dans le paramètre `--ids`, spécifiez une liste séparée par des espaces des ID des ressources à déplacer.

```azurecli
webapp=$(az resource show -g OldRG -n ExampleSite --resource-type "Microsoft.Web/sites" --query id --output tsv)
plan=$(az resource show -g OldRG -n ExamplePlan --resource-type "Microsoft.Web/serverfarms" --query id --output tsv)
az resource move --destination-group newgroup --ids $webapp $plan
```

Pour déplacer des ressources vers un nouvel abonnement, spécifiez le paramètre `--destination-subscription-id`.

Si vous recevez une erreur, consultez [Résoudre les problèmes liés au déplacement de ressources vers un nouveau groupe de ressource ou un nouvel abonnement](troubleshoot-move.md).

## <a name="use-rest-api"></a>Avec l’API REST

Pour déplacer des ressources existantes vers un autre groupe de ressources ou un autre abonnement, utilisez l’opération [Déplacer des ressources](/rest/api/resources/Resources/MoveResources).

```HTTP
POST https://management.azure.com/subscriptions/{source-subscription-id}/resourcegroups/{source-resource-group-name}/moveResources?api-version={api-version}
```

Dans le corps de la requête, vous indiquez le groupe de ressources cible et les ressources à déplacer.

```json
{
 "resources": ["<resource-id-1>", "<resource-id-2>"],
 "targetResourceGroup": "/subscriptions/<subscription-id>/resourceGroups/<target-group>"
}
```

Si vous recevez une erreur, consultez [Résoudre les problèmes liés au déplacement de ressources vers un nouveau groupe de ressource ou un nouvel abonnement](troubleshoot-move.md).

## <a name="frequently-asked-questions"></a>Forum aux questions

**Question : Mon opération de déplacement de ressources, qui prend généralement quelques minutes, s’exécute depuis presque une heure. Y a-t-il un problème ?**

Le déplacement d’une ressource est une opération complexe qui a des phases différentes. Elle peut impliquer davantage que le fournisseur de ressources de la ressource que vous essayez de déplacer. En raison des dépendances entre les fournisseurs de ressources, Azure Resource Manager accorde à l’opération 4 heures pour se terminer. Ce laps de temps donne aux fournisseurs de ressources la possibilité de récupérer des problèmes temporaires. Si votre demande de déplacement se situe dans le délai de quatre heures, l’opération continue de s’exécuter et peut encore aboutir. Les groupes de ressources sources et de destination sont verrouillés pendant ce temps afin d’éviter les problèmes de cohérence.

**Question : Pourquoi mon groupe de ressources est-il verrouillé pendant quatre heures pendant le déplacement des ressources ?**

Une demande de déplacement est autorisée à s’exécuter un maximum de quatre heures. Pour empêcher les ressources déplacées d’être modifiées, les groupes de ressources sources et de destination sont verrouillés pendant la durée du déplacement des ressources.

Une demande de déplacement comporte deux phases. Dans la première phase, la ressource est déplacée. Dans la deuxième phase, des notifications sont envoyées à d’autres fournisseurs de ressources qui dépendent de la ressource déplacée. Un groupe de ressources peut être verrouillé durant l’intégralité du délai de quatre heures lorsqu’un fournisseur de ressources échoue dans l’une ou l’autre phase. Pendant le temps imparti, Resource Manager retente l’étape qui a échoué.

Si une ressource ne peut pas être déplacée dans le délai de quatre heures, Resource Manager déverrouille les deux groupes de ressources. Les ressources qui ont été déplacées se trouvent dans le groupe de ressources de destination. Les ressources qui n’ont pas pu être déplacées sont conservées dans le groupe de ressources source.

**Question : Quelles sont les implications des groupes de ressources sources et de destination verrouillés pendant le déplacement des ressources ?**

Le verrou vous empêche de supprimer un groupe de ressources, de créer une nouvelle ressource dans un groupe de ressources ou de supprimer les ressources impliquées dans le déplacement.

L’illustration suivante montre un message d’erreur du portail Azure lorsqu’un utilisateur tente de supprimer un groupe de ressources qui fait partie d’un déplacement en cours.

![Message d’erreur lors du déplacement, tentative de suppression](./media/move-resource-group-and-subscription/move-error-delete.png)

**Question : Que signifie le code d’erreur « MissingMoveDependentResources » ?**

Lors du déplacement d’une ressource, ses ressources dépendantes doivent exister dans l’abonnement ou le groupe de ressources de destination, ou elle doivent être incluses dans la demande de déplacement. Vous recevez le code d’erreur MissingMoveDependentResources quand une ressource dépendante n’est pas conforme à cette exigence. Le message d’erreur contient des détails sur la ressource dépendante qui doit être incluse dans la demande de déplacement.

Par exemple, le déplacement d’une machine virtuelle peut nécessiter le déplacement de sept types de ressources avec trois fournisseurs de ressources différents. Ces types et fournisseurs de ressources sont les suivants :

* Microsoft.Compute

  * virtualMachines
  * disks
* Microsoft.Network
  * networkInterfaces
  * publicIPAddresses
  * networkSecurityGroups
  * virtualNetworks
* Microsoft.Storage
  * storageAccounts

Un autre exemple courant implique le déplacement d’un réseau virtuel. Vous devrez peut-être déplacer plusieurs autres ressources associées à ce réseau virtuel. La demande de déplacement peut nécessiter le déplacement d’adresses IP publiques, de tables de routage, de passerelles de réseau virtuel, de groupes de sécurité réseau, etc.

**Question : Pourquoi ne puis-je pas déplacer certaines ressources dans Azure ?**

Actuellement, toutes les ressources dans Azure prennent en charge le déplacement. Pour obtenir la liste des ressources prenant en charge l’opération de déplacement, consultez [Prise en charge de l’opération de déplacement pour les ressources](move-support-resources.md).

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir la liste des ressources prenant en charge l’opération de déplacement, consultez [Prise en charge de l’opération de déplacement pour les ressources](move-support-resources.md).
