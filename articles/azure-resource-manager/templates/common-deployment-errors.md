---
title: Résoudre les erreurs de déploiement courantes
description: Décrit comment résoudre les erreurs courantes lors du déploiement de ressources sur Azure à l’aide d’Azure Resource Manager.
tags: top-support-issue
ms.topic: troubleshooting
ms.date: 09/09/2020
ms.openlocfilehash: 785a74184cae8523b99da7f647ca87fda53c8648
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185825"
---
# <a name="troubleshoot-common-azure-deployment-errors-with-azure-resource-manager"></a>Résolution des erreurs courantes dans des déploiements Azure avec Azure Resource Manager

Cet article décrit certaines erreurs courantes liées au déploiement Azure et fournit des informations pour les résoudre. Si vous ne trouvez pas le code d’erreur correspondant à l’erreur de votre déploiement, consultez [Rechercher un code d’erreur](#find-error-code).

Si vous recherchez des informations sur un code d’erreur et que ces informations ne sont pas fournies dans cet article, faites-le nous savoir. En bas de cette page, vous pouvez laisser des commentaires. Ces commentaires sont suivis avec les problèmes GitHub.

## <a name="error-codes"></a>Codes d’erreur

| Code d'erreur | Limitation des risques | Informations complémentaires |
| ---------- | ---------- | ---------------- |
| AccountNameInvalid | Suivez les restrictions concernant l’attribution de noms pour les comptes de stockage. | [Résoudre les erreurs de nom du compte de stockage](error-storage-account-name.md) |
| AccountPropertyCannotBeSet | Vérifiez les propriétés disponibles du compte de stockage. | [storageAccounts](/azure/templates/microsoft.storage/storageaccounts) |
| AllocationFailed | Le cluster ou la région n’a pas de ressources disponibles ou ne prend pas en charge la taille de machine virtuelle demandée. Renouvelez la demande plus tard ou demandez une taille de machine virtuelle différente. | [Problèmes d’approvisionnement et d’allocation pour Linux](../../virtual-machines/troubleshooting/troubleshoot-deployment-new-vm-linux.md), [Problèmes d’approvisionnement et d’allocation pour Windows](../../virtual-machines/troubleshooting/troubleshoot-deployment-new-vm-windows.md) et [Résoudre des échecs d’allocation](../../virtual-machines/troubleshooting/allocation-failure.md)|
| AnotherOperationInProgress | Attendez que l’opération simultanée soit terminée. | |
| AuthorizationFailed | Votre compte ou principal du service ne dispose pas de droits d’accès suffisants pour terminer le déploiement. Vérifiez le rôle auquel votre compte appartient et son accès dans le cadre du déploiement.<br><br>Cette erreur peut s’afficher quand un fournisseur de ressources requis n’est pas inscrit. | [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/role-assignments-portal.md)<br><br>[Résoudre les erreurs d’inscription](error-register-resource-provider.md) |
| BadRequest | Vous avez envoyé des valeurs de déploiement qui ne correspondent pas aux valeurs attendues par Resource Manager. Vérifiez le message d’état interne pour résoudre plus facilement le problème. | [Référence de modèle](/azure/templates/) et [Emplacements pris en charge](resource-location.md) |
| Conflit | Vous demandez une opération qui n’est pas autorisée dans l’état actuel de la ressource. Par exemple, un redimensionnement de disque est autorisé uniquement durant la création ou la libération d’une machine virtuelle. | |
| DeploymentActiveAndUneditable | Attendez le déploiement simultané sur ce groupe de ressources soit terminé. | |
| DeploymentFailedCleanUp | Lorsque vous déployez en mode complet, toutes les ressources qui ne sont pas dans le modèle sont supprimées. Vous recevez cette erreur lorsque vous n’avez pas les autorisations appropriées pour supprimer toutes les ressources qui ne sont pas dans le modèle. Pour éviter cette erreur, remplacez le mode de déploiement par le mode incrémentiel. | [Modes de déploiement Azure Resource Manager](deployment-modes.md) |
| DeploymentNameInvalidCharacters | Le nom du déploiement ne peut contenir que des lettres, des chiffres, « - », « . » et « _ ». | |
| DeploymentNameLengthLimitExceeded | Les noms de déploiement sont limités à 64 caractères.  | |
| DeploymentFailed | L’erreur DeploymentFailed est une erreur générale qui ne fournit pas les détails dont vous avez besoin pour résoudre l’erreur. Pour en savoir plus, recherchez un code d’erreur dans les détails de l’erreur. | [Rechercher un code d’erreur](#find-error-code) |
| DeploymentQuotaExceeded | Si vous atteignez la limite des 800 déploiements par groupe de ressources, supprimez les déploiements inutiles dans l’historique. | [Résoudre l’erreur de nombre de déploiements supérieur à 800](deployment-quota-exceeded.md) |
| DeploymentSizeExceeded | Simplifiez votre modèle pour réduire la taille. | [Résoudre des erreurs de taille de modèle](error-job-size-exceeded.md) |
| DnsRecordInUse | Le nom de l’enregistrement DNS doit être unique. Entrez un autre nom. | |
| ImageNotFound | Vérifiez les paramètres d’image de machine virtuelle. |  |
| InUseSubnetCannotBeDeleted | Vous pouvez rencontrer cette erreur quand vous tentez de mettre à jour une ressource, et que cette requête est traitée en supprimant et en créant la ressource. Veillez à spécifier toutes les valeurs non modifiées. | [Mettre à jour une ressource](/azure/architecture/building-blocks/extending-templates/update-resource) |
| InvalidAuthenticationTokenTenant | Procurez-vous le jeton d’accès pour le client approprié. Vous pouvez uniquement obtenir le jeton auprès du client auquel appartient votre compte. | |
| InvalidContentLink | Vous avez probablement tenté d’établir une liaison avec un modèle imbriqué qui n’est pas disponible. Vérifiez l’URI que vous avez indiqué pour le modèle imbriqué. Si le modèle existe dans un compte de stockage, assurez-vous que l’URI est accessible. Vous devrez peut-être valider un jeton SAS. Vous ne pouvez pas établir un lien vers un modèle qui se trouve dans un compte de stockage derrière un [pare-feu Stockage Azure](../../storage/common/storage-network-security.md). Envisagez de déplacer votre modèle vers un autre dépôt, par exemple GitHub. | [Modèles liés](linked-templates.md) |
| InvalidDeploymentLocation | Lors du déploiement au niveau de l’abonnement, vous avez fourni un emplacement différent pour un nom de déploiement précédemment utilisé. | [Déploiements de niveau abonnement](deploy-to-subscription.md) |
| InvalidParameter | L’une des valeurs que vous avez fournies pour une ressource ne correspond pas à la valeur attendue. Cette erreur peut être due à de nombreuses conditions différentes. Par exemple, il se peut qu’un mot de passe soit insuffisant ou un nom d’objet blob incorrect. Le message d’erreur doit indiquer la valeur à corriger. | |
| InvalidRequestContent | Les valeurs de déploiement contiennent des valeurs inattendues ou n’incluent pas les valeurs requises. Vérifiez les valeurs pour votre type de ressource. | [Référence de modèle](/azure/templates/) |
| InvalidRequestFormat | Activez l’enregistrement du débogage durant l’exécution du déploiement et vérifiez le contenu de la demande. | [Activer l’enregistrement du débogage](#enable-debug-logging) |
| InvalidResourceNamespace | Vérifiez l’espace de noms de ressources que vous avez spécifié dans la propriété **type**. | [Référence de modèle](/azure/templates/) |
| InvalidResourceReference | La ressource n’existe pas encore ou n’est pas correctement référencée. Vérifiez si vous devez ajouter une dépendance. Vérifiez que votre utilisation de la fonction **reference** inclut les paramètres requis pour votre scénario. | [Résoudre les erreurs de dépendance](error-not-found.md) |
| InvalidResourceType | Vérifiez le type de ressource que vous avez spécifié dans la propriété **type**. | [Référence de modèle](/azure/templates/) |
| InvalidSubscriptionRegistrationState | Inscrivez votre abonnement auprès du fournisseur de ressources. | [Résoudre les erreurs d’inscription](error-register-resource-provider.md) |
| InvalidTemplate | Vérifiez que la syntaxe de votre modèle ne contient pas d’erreurs. | [Résoudre les erreurs de modèle non valide](error-invalid-template.md) |
| InvalidTemplateCircularDependency | Supprimez les dépendances inutiles. | [Résoudre les dépendances circulaires](error-invalid-template.md#circular-dependency) |
| JobSizeExceeded | Simplifiez votre modèle pour réduire la taille. | [Résoudre des erreurs de taille de modèle](error-job-size-exceeded.md) |
| LinkedAuthorizationFailed | Vérifiez si votre compte appartient au même locataire que le groupe de ressources vers lequel vous effectuez le déploiement. | |
| LinkedInvalidPropertyId | L’ID de ressource pour une ressource particulière n’est pas correctement résolu. Assurez-vous de bien fournir toutes les valeurs requises pour l’ID de ressource, notamment l’ID d’abonnement, le nom du groupe de ressources, le type de ressource, le nom de la ressource parente (si nécessaire) et le nom de la ressource. | |
| LocationRequired | Fournissez un emplacement pour la ressource. | [Définir un emplacement](resource-location.md) |
| MismatchingResourceSegments | Assurez-vous que la ressource imbriquée a un nombre correct de segments dans le nom et le type. | [Résoudre les segments de la ressource](error-invalid-template.md#incorrect-segment-lengths)
| MissingRegistrationForLocation | Vérifiez l’état d’inscription du fournisseur de ressources, ainsi que les emplacements pris en charge. | [Résoudre les erreurs d’inscription](error-register-resource-provider.md) |
| MissingSubscriptionRegistration | Inscrivez votre abonnement auprès du fournisseur de ressources. | [Résoudre les erreurs d’inscription](error-register-resource-provider.md) |
| NoRegisteredProviderFound | Vérifier l’état d’inscription du fournisseur de ressources. | [Résoudre les erreurs d’inscription](error-register-resource-provider.md) |
| NotFound | Vous essayez peut-être de déployer une ressource dépendante en parallèle avec une ressource parente. Vérifiez si vous avez besoin d’ajouter une dépendance. | [Résoudre les erreurs de dépendance](error-not-found.md) |
| OperationNotAllowed | Le déploiement tente une opération qui dépasse le quota autorisé pour l’abonnement, le groupe de ressources ou la région. Si possible, modifiez votre déploiement pour respecter les quotas. Dans le cas contraire, vous pouvez demander une modification de vos quotas. | [Résoudre les erreurs de quota](error-resource-quota.md) |
| ParentResourceNotFound | Assurez-vous qu’il existe une ressource parente avant de créer des ressources enfants. | [Résoudre les erreurs de ressource parente](error-parent-resource.md) |
| PasswordTooLong | Vous avez peut-être choisi un mot de passe comportant trop de caractères ou converti la valeur de votre mot de passe en chaîne sécurisée avant de la transmettre en tant que paramètre. Si le modèle inclut un paramètre de **chaîne sécurisée**, vous n’avez pas besoin de convertir la valeur en chaîne sécurisée. Indiquez la valeur de mot de passe sous forme de texte. |  |
| PrivateIPAddressInReservedRange | L’adresse IP spécifiée inclut une plage d’adresses requise par Azure. Modifiez l’adresse IP pour éviter d’utiliser la plage réservée. | [Adresses IP](../../virtual-network/public-ip-addresses.md) |
| PrivateIPAddressNotInSubnet | L’adresse IP spécifiée se trouve en dehors de la plage de sous-réseau. Modifiez l’adresse IP pour qu’elle se trouve dans la plage de sous-réseau. | [Adresses IP](../../virtual-network/public-ip-addresses.md) |
| PropertyChangeNotAllowed | Certaines propriétés ne peuvent pas être changées sur une ressource déployée. Durant la mise à jour d’une ressource, limitez vos modifications aux propriétés autorisées. | [Mettre à jour une ressource](/azure/architecture/building-blocks/extending-templates/update-resource) |
| RequestDisallowedByPolicy | Votre abonnement inclut une stratégie de ressource qui empêche une action que vous tentez d’exécuter au cours du déploiement. Recherchez la stratégie qui bloque l’action. Si possible, modifiez votre déploiement pour respecter les limitations de la stratégie. | [Résoudre les erreurs de stratégie](error-policy-requestdisallowedbypolicy.md) |
| ReservedResourceName | Spécifiez un nom de ressource qui n’inclut pas de nom réservé. | [Noms de ressource réservés](error-reserved-resource-name.md) |
| ResourceGroupBeingDeleted | Attendez que la suppression soit terminée. | |
| ResourceGroupNotFound | Vérifiez le nom du groupe de ressources cible pour le déploiement. Le groupe de ressources cible doit déjà exister dans votre abonnement. Vérifiez le contexte de votre abonnement. | [Azure CLI](/cli/azure/account?#az-account-set) [PowerShell](/powershell/module/Az.Accounts/Set-AzContext) |
| ResourceNotFound | Votre déploiement fait référence à une ressource qui ne peut pas être résolue. Vérifiez que votre utilisation de la fonction **reference** inclut les paramètres requis pour votre scénario. | [Résoudre les erreurs de référence](error-not-found.md) |
| ResourceQuotaExceeded | Le déploiement tente de créer des ressources qui dépassent le quota autorisé pour l’abonnement, le groupe de ressources ou la région. Si possible, modifiez votre infrastructure pour respecter les quotas. Dans le cas contraire, vous pouvez demander une modification de vos quotas. | [Résoudre les erreurs de quota](error-resource-quota.md) |
| SkuNotAvailable | Sélectionnez la référence SKU (par exemple, la taille de la machine virtuelle) disponible pour l’emplacement que vous avez sélectionné. | [Résoudre les erreurs de référence SKU](error-sku-not-available.md) |
| StorageAccountAlreadyExists | Attribuez un nom unique au compte de stockage. | [Résoudre les erreurs de nom du compte de stockage](error-storage-account-name.md)  |
| StorageAccountAlreadyTaken | Attribuez un nom unique au compte de stockage. | [Résoudre les erreurs de nom du compte de stockage](error-storage-account-name.md) |
| StorageAccountNotFound | Vérifiez l’abonnement, le groupe de ressources et le nom du compte de stockage que vous tentez d’utiliser. | |
| SubnetsNotInSameVnet | Une machine virtuelle ne peut avoir qu’un seul réseau virtuel. Si vous déployez plusieurs cartes réseau, assurez-vous qu’elles appartiennent au même réseau virtuel. | [Cartes réseau multiples](../../virtual-machines/windows/multiple-nics.md) |
| SubscriptionNotFound | Impossible d’accéder à un abonnement spécifié pour le déploiement. L’ID d’abonnement est peut-être incorrect, l’utilisateur qui déploie le modèle ne dispose pas des autorisations appropriées pour le déployer sur l’abonnement, ou l’ID d’abonnement n’est pas au bon format. Lorsque vous utilisez des déploiements imbriqués pour [déployer à travers des étendues](./deploy-to-resource-group.md), indiquez le GUID de l’abonnement. | |
| SubscriptionNotRegistered | Lors du déploiement d’une ressource, le fournisseur de ressources doit être inscrit pour votre abonnement. Lorsque vous utilisez un modèle Resource Manager pour le déploiement, le fournisseur de ressources est automatiquement inscrit dans l’abonnement. Parfois, l’inscription automatique ne se termine pas à temps. Pour éviter cette erreur intermittente, inscrivez le fournisseur de ressources avant le déploiement. | [Résoudre les erreurs d’inscription](error-register-resource-provider.md) |
| TemplateResourceCircularDependency | Supprimez les dépendances inutiles. | [Résoudre les dépendances circulaires](error-invalid-template.md#circular-dependency) |
| TooManyTargetResourceGroups | Réduisez le nombre de groupes de ressources pour un déploiement unique. | [Déploiement inter-étendue](./deploy-to-resource-group.md) |

## <a name="find-error-code"></a>Recherche un code d'erreur

Il existe deux types d’erreurs que vous pouvez rencontrer :

* des erreurs de validation
* des erreurs de déploiement

Les erreurs de validation sont liées à des scénarios pouvant être identifiés avant le déploiement. Elles incluent notamment les erreurs de syntaxe dans votre modèle ou le déploiement de ressources entraînant un dépassement des quotas d’abonnement. Les erreurs de déploiement sont liées aux événements se produisant lors du déploiement. Elles incluent la tentative d’accès à une ressource qui est déployée en parallèle.

Les deux types d’erreurs retournent un code d’erreur qui vous permet de résoudre les problèmes liés au déploiement. Les deux types d’erreurs apparaissent dans le [journal d’activité](../management/view-activity-logs.md). Toutefois, les erreurs de validation n’apparaissent pas dans l’historique de votre déploiement, car le déploiement n’a jamais démarré.

### <a name="validation-errors"></a>Erreurs de validation

Lors d’un déploiement via le portail, une erreur de validation s’affiche après l’envoi de vos valeurs.

![afficher l’erreur de validation dans le portail](./media/common-deployment-errors/validation-error.png)

Sélectionnez le message pour obtenir plus d’informations. Dans l’image suivante, vous pouvez voir une erreur **InvalidTemplateDeployment** et un message qui indique qu’une stratégie a bloqué le déploiement.

![afficher les détails de validation](./media/common-deployment-errors/validation-details.png)

### <a name="deployment-errors"></a>Erreurs de déploiement

Quand l’opération réussit la validation, mais qu’elle échoue pendant le déploiement, vous recevez une erreur de déploiement.

Pour voir les codes et les messages d’erreur de déploiement avec PowerShell, utilisez :

```azurepowershell-interactive
(Get-AzResourceGroupDeploymentOperation -DeploymentName exampledeployment -ResourceGroupName examplegroup).Properties.statusMessage
```

Pour voir les codes et les messages d’erreur de déploiement avec Azure CLI, utilisez :

```azurecli-interactive
az deployment operation group list --name exampledeployment -g examplegroup --query "[*].properties.statusMessage"
```

Dans le portail, sélectionnez la notification.

![erreur de notification](./media/common-deployment-errors/notification.png)

Vous voyez plus d’informations sur le déploiement. Sélectionnez l’option pour rechercher plus d’informations sur l’erreur.

![échec du déploiement](./media/common-deployment-errors/deployment-failed.png)

Vous voyez le message et les codes d’erreur. Notez qu’il y a deux codes d’erreur. Le premier code d’erreur (**DeploymentFailed**) est un code d’erreur général qui ne fournit pas les détails dont vous avez besoin pour résoudre l’erreur. Le deuxième code d’erreur (**StorageAccountNotFound**) fournit les détails dont vous avez besoin.

![détails de l’erreur](./media/common-deployment-errors/error-details.png)

## <a name="enable-debug-logging"></a>Activer l’enregistrement du débogage

Vous avez parfois besoin de plus d’informations sur la demande et la réponse pour connaître la cause du problème. Vous pouvez demander la journalisation d’informations supplémentaires lors d’un déploiement.

### <a name="powershell"></a>PowerShell

Dans PowerShell, définissez le paramètre **DeploymentDebugLogLevel** pour All, ResponseContent ou RequestContent.

```powershell
New-AzResourceGroupDeployment `
  -Name exampledeployment `
  -ResourceGroupName examplegroup `
  -TemplateFile c:\Azure\Templates\storage.json `
  -DeploymentDebugLogLevel All
```

Examinez le contenu de la requête avec l’applet de commande suivant :

```powershell
(Get-AzResourceGroupDeploymentOperation `
-DeploymentName exampledeployment `
-ResourceGroupName examplegroup).Properties.request `
| ConvertTo-Json
```

Ou examinez la réponse avec :

```powershell
(Get-AzResourceGroupDeploymentOperation `
-DeploymentName exampledeployment `
-ResourceGroupName examplegroup).Properties.response `
| ConvertTo-Json
```

Ces informations peuvent vous aider à déterminer si une valeur dans le modèle n’est pas définie correctement.

### <a name="azure-cli"></a>Azure CLI

Actuellement, Azure CLI ne prend pas en charge l’activation de la journalisation du débogage, mais vous pouvez récupérer la journalisation du débogage.

Examinez les opérations de déploiement avec la commande suivante :

```azurecli
az deployment operation group list \
  --resource-group examplegroup \
  --name exampledeployment
```

Examinez le contenu de la demande avec la commande suivante :

```azurecli
az deployment operation group list \
  --name exampledeployment \
  -g examplegroup \
  --query [].properties.request
```

Examinez le contenu de la réponse avec la commande suivante :

```azurecli
az deployment operation group list \
  --name exampledeployment \
  -g examplegroup \
  --query [].properties.response
```

### <a name="nested-template"></a>Modèle imbriqué

Pour enregistrer les informations de débogage pour un modèle imbriqué, utilisez l’élément **debugSetting**.

```json
{
  "type": "Microsoft.Resources/deployments",
  "apiVersion": "2016-09-01",
  "name": "nestedTemplate",
  "properties": {
    "mode": "Incremental",
    "templateLink": {
      "uri": "{template-uri}",
      "contentVersion": "1.0.0.0"
    },
    "debugSetting": {
       "detailLevel": "requestContent, responseContent"
    }
  }
}
```

## <a name="create-a-troubleshooting-template"></a>Création d’un modèle de résolution des problèmes

Dans certains cas, le moyen le plus simple pour résoudre les problèmes de votre modèle consiste à tester des parties de celui-ci. Vous pouvez créer un modèle simplifié qui vous permet de vous concentrer sur la partie que vous pensez erronée. Par exemple, supposons que vous recevez une erreur lorsque vous référencez une ressource. Au lieu de traiter un modèle complet, créez un modèle retournant la partie qui peut être à l’origine de votre problème. Il peut vous aider à déterminer si vous transmettez les paramètres appropriés à l’aide de fonctions de modèle et si vous obtenez la ressource que vous attendez.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
  "storageName": {
    "type": "string"
  },
  "storageResourceGroup": {
    "type": "string"
  }
  },
  "variables": {},
  "resources": [],
  "outputs": {
  "exampleOutput": {
    "value": "[reference(resourceId(parameters('storageResourceGroup'), 'Microsoft.Storage/storageAccounts', parameters('storageName')), '2016-05-01')]",
    "type" : "object"
  }
  }
}
```

Si vous obtenez des erreurs de déploiement que vous pensez liées à une mauvaise définition des dépendances, vous pouvez tester votre modèle en le divisant en modèles plus simples. Commencez par créer un modèle déployant une seule ressource (comme un serveur SQL Server). Lorsque vous êtes sûr que cette ressource est correctement définie, ajoutez une ressource qui en dépend (par exemple, une base de données SQL). Une fois ces deux ressources correctement définies, ajoutez les autres ressources dépendantes (telles que les stratégies d’audit). Supprimez le groupe de ressources entre chaque déploiement de test afin de tester correctement les dépendances.

## <a name="next-steps"></a>Étapes suivantes

* Pour étudier la résolution des problèmes, suivez le [Tutoriel : Résoudre les problèmes liés aux déploiements de modèles Resource Manager](template-tutorial-troubleshoot.md)
* Pour en savoir plus sur les actions d’audit, consultez [Opérations d’audit avec Resource Manager](../management/view-activity-logs.md).
* Pour en savoir plus sur les actions visant à déterminer les erreurs au cours du déploiement, consultez [Voir les opérations de déploiement](deployment-history.md).