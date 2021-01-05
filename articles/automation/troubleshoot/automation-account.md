---
title: Résoudre des problèmes liés aux comptes Azure Automation
description: Découvrez comment résoudre les problèmes liés aux comptes Azure.
services: automation
author: mgoedtel
ms.author: magoedte
ms.date: 03/24/2020
ms.topic: conceptual
ms.service: automation
manager: carmonm
ms.openlocfilehash: edd57d3d77432f3bb37872ee26f414b56398cae0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86187283"
---
# <a name="troubleshoot-azure-automation-account-issues"></a>Résoudre des problèmes liés aux comptes Azure Automation

Cet article traite des solutions aux problèmes que vous pourriez rencontrer en utilisant un compte Azure Automation. Pour obtenir des informations générales sur les comptes Automation, consultez [Vue d’ensemble de l’authentification de compte Azure Automation](../automation-security-overview.md).

## <a name="scenario-unable-to-register-automation-resource-provider-for-subscriptions"></a><a name="rp-register"></a>Scénario : Impossible d’inscrire le fournisseur de ressources Automation pour les abonnements

### <a name="issue"></a>Problème

Lorsque vous utilisez des fonctionnalités de gestion comme Update Management dans votre compte Automation, vous rencontrez l’erreur suivante :

```error
Error details: Unable to register Automation Resource Provider for subscriptions:
```

### <a name="cause"></a>Cause

Le fournisseur de ressources Automation n’est pas inscrit dans l’abonnement.

### <a name="resolution"></a>Résolution

Pour inscrire le fournisseur de ressources Automation, effectuez les opérations suivantes dans le portail Azure :

1. À partir de votre navigateur, accédez au [portail Azure](https://portal.azure.com).

2. Accédez à **Abonnements**, puis sélectionnez votre abonnement.   

3. Sous **Paramètres**, sélectionnez **Fournisseurs de ressources**.

4. Dans la liste des fournisseurs de ressources, vérifiez que le fournisseur de ressources **Microsoft.Automation** est inscrit.

5. S’il ne l’est pas, inscrivez-le conformément à la description que vous trouverez dans [Résoudre les erreurs d’inscription du fournisseur de ressources](../../azure-resource-manager/templates/error-register-resource-provider.md).

## <a name="next-steps"></a>Étapes suivantes

Si cet article ne vous permet pas de résoudre votre problème, utilisez l’un des canaux suivants pour obtenir une aide supplémentaire :

* Obtenez des réponses de la part d’experts Azure via les [Forums Azure](https://azure.microsoft.com/support/forums/).
* Contactez [@AzureSupport](https://twitter.com/azuresupport). Il s’agit du compte Microsoft Azure officiel qui permet à la communauté Azure d’accéder aux ressources dont elle a besoin : réponses, support et experts.
* Signaler un incident au support Azure Accédez au [site du support Azure](https://azure.microsoft.com/support/options/), puis sélectionnez **Obtenir de l’aide**.
