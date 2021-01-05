---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 11/20/2020
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: b413a5ad82fa464fa0fe698812dd36a7375cfead
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96007696"
---
|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les registres de conteneurs doivent être chiffrés avec une clé gérée par le client (CMK)](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5b9159ae-1701-4a6f-9a7a-aa9c8ddd0580) |Auditez ou refusez les registres de conteneurs pour lesquels le chiffrement n’est pas activé avec des clés gérées par le client (CMK). Azure chiffre automatiquement le contenu du registre au repos avec des clés gérées par le service. Vous pouvez compléter le chiffrement par défaut avec une couche de chiffrement supplémentaire à l’aide d’une clé que vous créez et gérez dans Azure Key Vault. Pour plus d’informations sur le chiffrement CMK, consultez : [https://aka.ms/acr/CMK](https://aka.ms/acr/CMK). |Audit, Refuser, Désactivé |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Container%20Registry/ACR_CMKEncryptionEnabled_Audit.json) |
|[Les registres de conteneurs ne doivent pas autoriser un accès réseau non restreint](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd0793b48-0edc-4296-a390-4c75d1bdfd71) |Auditez les registres de conteneurs qui n’ont pas de règles (IP) de réseau ou de pare-feu configurées et autorisent par défaut l’accès à tout le réseau. La restriction de l’accès réseau protège les registres de conteneurs contre les menaces potentielles. Les registres de conteneurs avec au moins une règle IP / de pare-feu ou un réseau virtuel configuré sont considérés comme conformes. Pour plus d’informations sur les règles réseau de Container Registry, consultez : [https://aka.ms/acr/portal/public-network](https://aka.ms/acr/portal/public-network) et [https://aka.ms/acr/vnet](https://aka.ms/acr/vnet). |Audit, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Container%20Registry/ACR_NetworkRulesExist_Audit.json) |
|[Les registres de conteneurs doivent utiliser des liaisons privées](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe8eef0a8-67cf-4eb4-9386-14b0e78733d4) |Auditez les registres de conteneurs qui n’ont pas au moins une connexion de point de terminaison privé approuvée. Les clients d’un réseau virtuel peuvent accéder de façon sécurisée aux ressources qui ont des connexions de point de terminaison privés via des liens privés. L’accès public peut ensuite être désactivé pour garantir que seuls des liens privés peuvent être utilisés pour se connecter au registre. Pour plus d’informations, consultez [https://aka.ms/acr/private-link](https://aka.ms/acr/private-link). |Audit, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Container%20Registry/ACR_PrivateEndpointEnabled_Audit.json) |
