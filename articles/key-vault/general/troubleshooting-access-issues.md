---
title: Résolution des problèmes de stratégie d’accès au coffre de clés Azure
description: Résolution des problèmes de stratégie d’accès au coffre de clés Azure
author: sebansal
ms.author: sebansal
ms.date: 08/10/2020
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.openlocfilehash: 78b42a8ad3685d07b61c4faca384c7ee8f5a5f94
ms.sourcegitcommit: 86acfdc2020e44d121d498f0b1013c4c3903d3f3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97616386"
---
# <a name="troubleshooting-azure-key-vault-access-policy-issues"></a>Résolution des problèmes de stratégie d’accès au coffre de clés Azure

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="i-am-not-able-to-list-or-get-secretskeyscertificate-i-am-seeing-something-went-wrong-error"></a>Je ne suis pas en mesure de lister les secrets, les clés ou un certificat, ou d’y accéder. Je vois une erreur de type « un problème est survenu... » .
Si vous rencontrez un problème avec l’affichage, l’obtention, la création ou l’accès à un secret, vérifiez que votre stratégie d’accès vous permet cette opération : [Stratégies d’accès à un coffre de clés](https://docs.microsoft.com/azure/key-vault/general/group-permissions-for-apps)

### <a name="how-can-i-identify-how-and-when-key-vaults-are-accessed"></a>Comment faire pour identifier le mode et le moment d’accès aux coffres de clés ?

Une fois que vous avez créé un ou plusieurs coffres de clés, vous voulez probablement contrôler qui accède à ces derniers, par quel moyen et quand. Pour effectuer cette supervision, vous pouvez activer la journalisation pour Azure Key Vault. Pour plus d’informations sur l’activation de la journalisation, consultez ce [guide pas à pas](./logging.md).

### <a name="how-can-i-monitor-vault-availability-service-latency-periods-or-other-performance-metrics-for-key-vault"></a>Comment faire pour surveiller la disponibilité du coffre, les périodes de latence du service ou d’autres métriques de performances pour le coffre de clés ?

Plus votre service se développera, plus le nombre de demandes envoyées à votre coffre de clés augmentera. Cette demande risque d'augmenter la latence de vos demandes et, dans les cas les plus extrêmes, de les limiter, ce qui aura un impact sur les performances de votre service. Vous pouvez surveiller les métriques de performances du coffre de clés et recevoir des alertes pour des seuils spécifiques. Pour en savoir plus sur la configuration de la surveillance, consultez ce [guide pas à pas](./alert.md).

### <a name="i-am-not-able-to-modify-access-policy-how-can-it-be-enabled"></a>Je ne parviens pas à modifier la stratégie d’accès, comment l’activer ?
L’utilisateur doit disposer d’autorisations AAD suffisantes pour modifier la stratégie d’accès. Dans le cas présent, l’utilisateur doit détenir un rôle Contributeur supérieur.

### <a name="i-am-seeing-unkwown-policy-error-what-does-that-mean"></a>Je vois une erreur « Stratégie inconnue ». Qu’est-ce que cela signifie ?
Il y a deux explications différentes à la présence de la stratégie d’accès dans la section Inconnu :
* Un ancien utilisateur avait un accès et, pour une raison ou une autre, celui-ci n’existe pas.
* Si la stratégie d’accès est ajoutée via PowerShell, et qu’elle est ajoutée pour l’ObjectID de l’application au lieu du principal du service

### <a name="how-can-i-assign-access-control-per-key-vault-object"></a>Comment faire pour attribuer le contrôle d’accès par objet du coffre de clés ? 

La disponibilité de la fonctionnalité de contrôle d’accès par secret/clé/certificat sera notifiée ici. [En savoir plus](https://feedback.azure.com/forums/906355-azure-key-vault/suggestions/32213176-per-secret-key-certificate-access-control)

### <a name="how-can-i-provide-key-vault-authenticate-using-access-control-policy"></a>Comment faire pour fournir une authentification auprès du coffre de clés à l’aide d’une stratégie de contrôle d’accès ?

Le moyen le plus simple d’authentifier une application cloud auprès de Key Vault consiste à utiliser une identité managée. Pour plus d’informations, consultez [Authentification auprès d’Azure Key Vault](authentication.md).
Si vous créez une application locale, si vous effectuez du développement local ou si vous ne parvenez pas à utiliser une identité managée, vous pouvez à la place inscrire un principal de service manuellement et fournir l’accès au coffre de clés à l’aide d’une stratégie de contrôle d’accès. Consultez [Attribution d’une stratégie de contrôle d’accès](assign-access-policy-portal.md).

### <a name="how-can-i-give-the-ad-group-access-to-the-key-vault"></a>Comment faire pour autoriser le groupe AD à accéder au coffre de clés ?

Accordez les autorisations de groupe AD à votre coffre de clés à l’aide de la commande Azure CLI `az keyvault set-policy` ou de la cmdlet Azure PowerShell Set-AzKeyVaultAccessPolicy. Consultez [Attribution d’une stratégie d’accès – CLI](assign-access-policy-cli.md) et [Attribution d’une stratégie d’accès – PowerShell](assign-access-policy-powershell.md).

L’application nécessite également l’affectation d’au moins un rôle IAM (gestion des identités et des accès) au coffre de clés. Sinon, elle ne pourra pas se connecter en raison de droits insuffisants pour accéder à l’abonnement. Les groupes Azure AD avec identités managées peuvent mettre huit heures maximum à actualiser le jeton et à devenir effectifs.

### <a name="how-can-i-redeploy-key-vault-with-arm-template-without-deleting-existing-access-policies"></a>Comment faire pour redéployer Key Vault avec un modèle ARM sans supprimer les stratégies d’accès existantes ?

Actuellement, un redéploiement Key Vault a pour effet de supprimer toutes les stratégies d’accès de Key Vault et de les remplacer par une stratégie d’accès dans un modèle ARM. Il n’existe pas d’option incrémentielle pour les stratégies d’accès du Key Vault. Afin de conserver dans Key Vault les stratégies d’accès existantes, vous devez lire celles-ci dans Key Vault, puis remplir le modèle ARM à l’aide de ces stratégies pour éviter toute interruption d’accès.

Une autre option qui peut être utile pour ce scénario consiste à utiliser le contrôle RBAC Azure et des rôles en guise d’alternative aux stratégies d’accès. Avec le contrôle RBAC Azure, vous pouvez redéployer le coffre de clés sans respécifier la stratégie. Vous trouverez d’autres informations sur cette solution [ici](./rbac-guide.md).

### <a name="recommended-troubleshooting-steps-for-following-error-types"></a>Procédure de résolution des problèmes recommandée pour les types d’erreurs suivants

* HTTP 401 : Requête non authentifiée – [Étapes de résolution des problèmes](rest-error-codes.md#http-401-unauthenticated-request)
* HTTP 403 : Autorisations insuffisantes – [Étapes de résolution des problèmes](rest-error-codes.md#http-403-insufficient-permissions)
* HTTP 429 : Nombre de requêtes trop élevé – [Étapes de résolution des problèmes](rest-error-codes.md#http-429-too-many-requests)
* Vérifiez si vous êtes autorisé à supprimer l’accès au coffre de clés : Consultez [Attribution d’une stratégie d’accès – CLI](assign-access-policy-cli.md), [Attribution d’une stratégie d’accès – PowerShell](assign-access-policy-powershell.md) ou [Attribution d’une stratégie d’accès – Portail](assign-access-policy-portal.md).
* Si vous rencontrez un problème avec l’authentification auprès du coffre de clés dans le code, utilisez le [SDK d’authentification](https://azure.github.io/azure-sdk/posts/2020-02-25/defaultazurecredentials.html)

### <a name="what-are-the-best-practices-i-should-implement-when-key-vault-is-getting-throttled"></a>Quelles sont les meilleures pratiques à implémenter lorsque le coffre de clés est limité ?
Suivez les meilleures pratiques décrites [ici](overview-throttling.md#how-to-throttle-your-app-in-response-to-service-limits).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment résoudre les erreurs d’authentification de coffre de clés : [Guide de résolution des problèmes liés à Key Vault](rest-error-codes.md).
