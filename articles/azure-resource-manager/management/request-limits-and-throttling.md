---
title: Limites des demandes et limitation
description: Décrit comment utiliser la limitation avec des requêtes Azure Resource Manager lorsque les limites d’abonnement ont été atteintes.
ms.topic: conceptual
ms.date: 12/15/2020
ms.custom: seodec18
ms.openlocfilehash: 181ed1a3059d86f78e40a9949448af77a551efbc
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97563124"
---
# <a name="throttling-resource-manager-requests"></a>Limitation des requêtes de Resource Manager

Cet article explique comment Azure Resource Manager limite les requêtes. Il vous montre comment suivre le nombre de demandes restantes avant d’atteindre la limite et comment réagir quand vous avez atteint la limite.

La limitation se produit à deux niveaux. Azure Resource Manager limite les demandes pour l’abonnement et le locataire. Si la demande se trouve sous les limites de limitation pour l’abonnement et le locataire, le Gestionnaire des ressources achemine la requête vers le fournisseur de ressources. Le fournisseur de ressources applique des limites de limitation adaptées à ses opérations. L’illustration suivante montre la façon dont la limitation est appliquée lorsqu’une demande passe de l’utilisateur à Azure Resource Manager et au fournisseur de ressources.

![Limitation des tentatives d’accès](./media/request-limits-and-throttling/request-throttling.svg)

## <a name="subscription-and-tenant-limits"></a>Limites de locataire et d’abonnement

Chaque opération au niveau de l’abonnement et du client est soumise à des limites de limitation. Les requêtes appliquées à l’abonnement sont celles qui impliquent la transmission de votre ID d’abonnement, par exemple pour récupérer les groupes de ressources dans votre abonnement. Les requêtes appliquées au locataire n’incluent pas votre ID d’abonnement, notamment pour la récupération des emplacements Azure valides.

Les limites de limitation par heure par défaut sont indiquées dans le tableau suivant.

| Étendue | Opérations | Limite |
| ----- | ---------- | ------- |
| Subscription | lectures | 12 000 |
| Subscription | suppressions | 15000 |
| Subscription | écritures | 1200 |
| Locataire | lectures | 12 000 |
| Locataire | écritures | 1200 |

Ces limites sont définies d’après le principal de sécurité (utilisateur ou application) qui effectue les requêtes et l’ID d’abonnement ou l’ID du locataire. Si vos requêtes proviennent de plusieurs principaux de sécurité, votre limite dans l’abonnement ou le locataire est supérieure à 12 000 et 1200 par heure.

Ces limites s’appliquent à chaque instance Azure Resource Manager. Chaque région Azure comporte plusieurs instances, et Azure Resource Manager est déployé dans toutes les régions Azure.  Par conséquent, dans la pratique, les limites sont supérieures à ces limites. Les demandes d’un utilisateur sont généralement gérées par des instances différentes d’Azure Resource Manager.

## <a name="resource-provider-limits"></a>Limites de fournisseur de ressources

Les fournisseurs de ressources appliquent leurs propres limites de limitation. Étant donné que le Gestionnaire des ressources est limité par l’ID principal et par l’instance du Gestionnaire des ressources, le fournisseur de ressources peut recevoir plus de demandes que les limites par défaut de la section précédente.

Cette section décrit les limites de limitation de certains fournisseurs de ressources couramment utilisés.

### <a name="storage-throttling"></a>Limitation du stockage

[!INCLUDE [azure-storage-limits-azure-resource-manager](../../../includes/azure-storage-limits-azure-resource-manager.md)]

### <a name="network-throttling"></a>Limitation du réseau

Le fournisseur de ressources Microsoft. Network applique les limites de limitation suivantes :

| Opération | Limite |
| --------- | ----- |
| écriture / suppression (PUT) | 1000 toutes les 5 minutes |
| lecture (GET) | 10000 toutes les 5 minutes |

### <a name="compute-throttling"></a>Calculer la limitation

Pour plus d’informations sur les limites de limitation des opérations de calcul, consultez [Résolution des erreurs de limitation d’API - Calcul](../../virtual-machines/troubleshooting/troubleshooting-throttling-errors.md).

Pour vérifier les instances de machine virtuelle dans un groupe de machines virtuelles identiques, utilisez les [opérations Virtual Machine Scale Sets](/rest/api/compute/virtualmachinescalesetvms). Par exemple, utilisez la [requête List sur les VM d’un groupe de machines virtuelles identiques](/rest/api/compute/virtualmachinescalesetvms/list) avec des paramètres pour vérifier l’état d’alimentation des instances de la machine virtuelle. Cette API réduit le nombre de demandes.

### <a name="azure-resource-graph-throttling"></a>Limitation Azure Resource Graph

[Azure Resource Graph](../../governance/resource-graph/overview.md) limite le nombre de demandes à ses opérations. Les étapes décrites dans cet article pour déterminer les requêtes restantes et comment réagir si la limite est atteinte s’appliquent également à Resource Graph. Toutefois, Resource Graph définit ses propres limites et son propre taux de réinitialisation. Pour plus d’informations, voir [En-têtes de limitation de Resource Graph](../../governance/resource-graph/concepts/guidance-for-throttled-requests.md#understand-throttling-headers).

### <a name="other-resource-providers"></a>Autres fournisseurs de ressources

Pour plus d’informations sur la limitation dans d’autres fournisseurs de ressources, consultez :

* [Aide sur la limitation de requêtes Azure Key Vault](../../key-vault/general/overview-throttling.md)
* [Résolution des problèmes liés à AKS](../../aks/troubleshooting.md#im-receiving-429---too-many-requests-errors)

## <a name="error-code"></a>Code d'erreur

Lorsque vous atteignez la limite, vous recevez le code d’état HTTP **429 Trop de requêtes**. La réponse inclut la valeur **Retry-After** qui spécifie le nombre de secondes pendant lesquelles votre application doit attendre (ou rester en veille) avant d’envoyer la requête suivante. Si vous envoyez une requête avant l’écoulement du temps d’attente avant une nouvelle tentative, votre requête n’est pas traitée et une autre valeur de nouvelle tentative est renvoyée.

Après avoir attendu l’heure spécifiée, vous pouvez également fermer puis rouvrir votre connexion à Azure. En réinitialisant la connexion, vous pouvez vous connecter à une autre instance d’Azure Resource Manager.

Si vous utilisez un kit de développement logiciel (SDK) Azure, celui-ci peut inclure une configuration de nouvelle tentative automatique. Pour plus d’informations, consultez [Guide du mécanisme de nouvelle tentative relatif aux services Azure](/azure/architecture/best-practices/retry-service-specific).

Certains fournisseurs de ressources retournent l’erreur 429 pour signaler un problème temporaire. Le problème peut être une condition de surcharge qui n’est pas directement due à votre demande. Il peut s’agir d’une erreur temporaire concernant l’état de la ressource cible ou de la ressource dépendante. Par exemple, le fournisseur de ressources réseau renvoie le message 429 avec le code d’erreur **RetryableErrorDueToAnotherOperation** lorsque la ressource cible est verrouillée par une autre opération. Pour déterminer si l’erreur provient d’une limitation ou d’une condition temporaire, affichez les détails de l’erreur dans la réponse.

## <a name="remaining-requests"></a>Requêtes restantes

Vous pouvez déterminer le nombre de requêtes restantes en examinant les en-têtes de réponse. Les requêtes de lecture retournent une valeur dans l’en-tête pour le nombre de requêtes de lecture restantes. Les requêtes d’écriture incluent une valeur pour le nombre de requêtes d’écriture restantes. Le tableau suivant décrit les en-têtes de réponse que vous pouvez examiner pour ces valeurs :

| En-tête de réponse | Description |
| --- | --- |
| x-ms-ratelimit-remaining-subscription-reads |Requêtes de lecture restantes étendues à l’abonnement. Cette valeur est renvoyée pour les opérations de lecture. |
| x-ms-ratelimit-remaining-subscription-writes |Requêtes d’écriture restantes étendues à l’abonnement. Cette valeur est renvoyée pour les opérations d’écriture. |
| x-ms-ratelimit-remaining-tenant-reads |Requêtes de lecture restantes étendues au locataire |
| x-ms-ratelimit-remaining-tenant-writes |Requêtes d’écriture restantes étendues au locataire |
| x-ms-ratelimit-remaining-subscription-resource-requests |Requêtes de type de ressource restantes étendues à l’abonnement.<br /><br />Cette valeur d’en-tête est retournée uniquement si un service a remplacé la limite par défaut. Resource Manager ajoute cette valeur au lieu des requêtes de lecture ou d’écriture de l’abonnement. |
| x-ms-ratelimit-remaining-subscription-resource-entities-read |Requêtes de collecte de type de ressource restantes étendues à l’abonnement.<br /><br />Cette valeur d’en-tête est retournée uniquement si un service a remplacé la limite par défaut. Cette valeur fournit le nombre de requêtes de collecte restantes (répertorier les ressources). |
| x-ms-ratelimit-remaining-tenant-resource-requests |Requêtes de type de ressource restantes étendues au locataire.<br /><br />Cet en-tête est ajouté uniquement pour les requêtes au niveau du locataire, et uniquement si un service a remplacé la limite par défaut. Resource Manager ajoute cette valeur au lieu des requêtes de lecture ou d’écriture du locataire. |
| x-ms-ratelimit-remaining-tenant-resource-entities-read |Requêtes de collecte de type de ressource restantes étendues au locataire.<br /><br />Cet en-tête est ajouté uniquement pour les requêtes au niveau du locataire, et uniquement si un service a remplacé la limite par défaut. |

Le fournisseur de ressources peut également retourner des en-têtes de réponse avec des informations sur les demandes restantes. Pour plus d’informations sur les en-têtes de réponse retournés par le fournisseur de ressources de calcul, consultez [En-têtes de réponse d’information de débit d’appels](../../virtual-machines/troubleshooting/troubleshooting-throttling-errors.md#call-rate-informational-response-headers).

## <a name="retrieving-the-header-values"></a>Récupération des valeurs d’en-tête

La récupération de ces valeurs d’en-tête dans votre code ou script est similaire à la récupération de n’importe quelle valeur d’en-tête. 

Par exemple, en **C#**, vous récupérez la valeur d’en-tête d’un objet **HttpWebResponse** nommé **response** avec le code suivant :

```cs
response.Headers.GetValues("x-ms-ratelimit-remaining-subscription-reads").GetValue(0)
```

Dans **PowerShell**, vous récupérez la valeur de l’en-tête d’une opération Invoke-WebRequest.

```powershell
$r = Invoke-WebRequest -Uri https://management.azure.com/subscriptions/{guid}/resourcegroups?api-version=2016-09-01 -Method GET -Headers $authHeaders
$r.Headers["x-ms-ratelimit-remaining-subscription-reads"]
```

Pour obtenir un exemple PowerShell complet, consultez [Vérifier les limites de Resource Manager pour un abonnement](https://github.com/Microsoft/csa-misc-utils/tree/master/psh-GetArmLimitsViaAPI).

Si voulez voir les requêtes restantes pour le débogage, vous pouvez définir le paramètre **-Debug** sur votre cmdlet **PowerShell**.

```powershell
Get-AzResourceGroup -Debug
```

Retourne un grand nombre de valeurs, notamment la valeur de réponse suivante :

```output
DEBUG: ============================ HTTP RESPONSE ============================

Status Code:
OK

Headers:
Pragma                        : no-cache
x-ms-ratelimit-remaining-subscription-reads: 11999
```

Pour obtenir des limites d’écriture, utilisez une opération d’écriture : 

```powershell
New-AzResourceGroup -Name myresourcegroup -Location westus -Debug
```

Renvoie un grand nombre de valeurs, notamment celles-ci :

```output
DEBUG: ============================ HTTP RESPONSE ============================

Status Code:
Created

Headers:
Pragma                        : no-cache
x-ms-ratelimit-remaining-subscription-writes: 1199
```

Dans **l’interface de ligne de commande**, vous récupérez la valeur d’en-tête à l’aide de l’option plus détaillée.

```azurecli
az group list --verbose --debug
```

Renvoie un grand nombre de valeurs, notamment celles-ci :

```output
msrest.http_logger : Response status: 200
msrest.http_logger : Response headers:
msrest.http_logger :     'Cache-Control': 'no-cache'
msrest.http_logger :     'Pragma': 'no-cache'
msrest.http_logger :     'Content-Type': 'application/json; charset=utf-8'
msrest.http_logger :     'Content-Encoding': 'gzip'
msrest.http_logger :     'Expires': '-1'
msrest.http_logger :     'Vary': 'Accept-Encoding'
msrest.http_logger :     'x-ms-ratelimit-remaining-subscription-reads': '11998'
```

Pour obtenir des limites d’écriture, utilisez une opération d’écriture : 

```azurecli
az group create -n myresourcegroup --location westus --verbose --debug
```

Renvoie un grand nombre de valeurs, notamment celles-ci :

```output
msrest.http_logger : Response status: 201
msrest.http_logger : Response headers:
msrest.http_logger :     'Cache-Control': 'no-cache'
msrest.http_logger :     'Pragma': 'no-cache'
msrest.http_logger :     'Content-Length': '163'
msrest.http_logger :     'Content-Type': 'application/json; charset=utf-8'
msrest.http_logger :     'Expires': '-1'
msrest.http_logger :     'x-ms-ratelimit-remaining-subscription-writes': '1199'
```

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir un exemple PowerShell complet, consultez [Vérifier les limites de Resource Manager pour un abonnement](https://github.com/Microsoft/csa-misc-utils/tree/master/psh-GetArmLimitsViaAPI).
* Pour plus d’informations sur les limites et les quotas, consultez [Abonnement Azure et limites, quotas et contraintes du service](../../azure-resource-manager/management/azure-subscription-service-limits.md).
* Pour en savoir plus sur la gestion des demandes REST asynchrones, consultez [Suivi des opérations asynchrones Azure](async-operations.md).
