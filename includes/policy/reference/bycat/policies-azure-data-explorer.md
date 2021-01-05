---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 11/20/2020
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 9024d12259a45dcf4e8321866f5c50d01c888934
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "96007583"
---
|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Le chiffrement au repos Azure Data Explorer doit utiliser une clé gérée par le client](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F81e74cea-30fd-40d5-802f-d72103c2aaaa) |Le fait d’activer le chiffrement au repos à l’aide d’une clé gérée par le client dans votre cluster Azure Data Explorer vous offre un contrôle supplémentaire sur cette clé. Cette fonctionnalité est souvent utilisée par les clients qui ont des exigences particulières au niveau de la conformité. Par ailleurs, elle nécessite un coffre de clés pour la gestion des clés. |Audit, Refuser, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Azure%20Data%20Explorer/ADX_CMK.json) |
|[Le chiffrement de disque doit être activé dans Azure Data Explorer](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff4b53539-8df9-40e4-86c6-6b607703bd4e) |Le fait d’activer le chiffrement de disque vous aide à protéger et à préserver vos données de façon à répondre aux engagements de votre entreprise concernant la sécurité et la conformité. |Audit, Refuser, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Azure%20Data%20Explorer/ADX_disk_encrypted.json) |
|[Le chiffrement double doit être activé dans Azure Data Explorer](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fec068d99-e9c7-401f-8cef-5bdde4e6ccf1) |Le fait d’activer le chiffrement double vous aide à protéger et à préserver vos données de façon à répondre aux engagements de votre entreprise concernant la sécurité et la conformité. Lorsque le chiffrement double est activé, les données d’un compte de stockage sont chiffrées deux fois : une fois au niveau du service et une fois au niveau de l’infrastructure, avec deux algorithmes de chiffrement différents et deux clés différentes. |Audit, Refuser, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Azure%20Data%20Explorer/ADX_doubleEncryption.json) |
|[L’injection de réseau virtuel doit être activée dans Azure Data Explorer](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9ad2fd1f-b25f-47a2-aa01-1a5a779e6413) |Sécurisez votre périmètre réseau à l’aide de l’injection de réseau virtuel qui vous permet d’appliquer des règles de groupe de sécurité réseau, de vous connecter localement et de sécuriser vos sources de connexion de données avec des points de terminaison de service. |Audit, Refuser, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Azure%20Data%20Explorer/ADX_VNET_configured.json) |
