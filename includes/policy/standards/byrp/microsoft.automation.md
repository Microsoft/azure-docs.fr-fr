---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 12/01/2020
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: c2385d07ccb81041bd340a8bec0412a8f14cef56
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96477366"
---
## <a name="azure-security-benchmark"></a>Benchmark de sécurité Azure

Le [benchmark de sécurité Azure](../../../../articles/security/benchmarks/overview.md) fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure. Pour voir comment ce service est entièrement mappé au benchmark de sécurité Azure, consultez les [fichiers de mappage du benchmark de sécurité Azure](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

Pour passer en revue la façon dont les composants intégrés Azure Policy disponibles pour tous les services Azure sont mappés à ce standard de conformité, consultez [Conformité réglementaire Azure Policy - Benchmark de sécurité Azure](../../../../articles/governance/policy/samples/azure-security-benchmark.md).

|Domain |ID du contrôle |Titre du contrôle |Policy<br /><sub>(Portail Azure)</sub> |Version de la stratégie<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Protection des données |4.8 |Chiffrer des informations sensibles au repos |[Les variables de compte Automation doivent être chiffrées](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3657f5a0-770e-44a3-b44e-9431ba1e9735) |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Automation/Automation_AuditUnencryptedVars_Audit.json) |

> [!NOTE]
> Quand vous créez une variable de compte Automation, vous pouvez spécifier son mode de chiffrement et de stockage par Azure Automation sous la forme d’une ressource sécurisée. Après avoir créé la variable, vous ne pouvez pas modifier son état de chiffrement sans recréer la variable. Si vous avez des variables de compte Automation stockant des données sensibles qui ne sont pas déjà chiffrées, vous devez les supprimer et les recréer en tant que variables chiffrées. Une suggestion Azure Security Center consiste à chiffrer toutes les variables Azure Automation, comme décrit dans [les variables de compte Automation doivent être chiffrées](../../../../articles/security-center/recommendations-reference.md#recs-computeapp). Si vous avez des variables non chiffrées que vous voulez exclure de cette recommandation de sécurité, consultez [Exempter une ressource des recommandations et du degré de sécurisation](../../../../articles/security-center/exempt-resource.md) pour créer une règle d’exemption.