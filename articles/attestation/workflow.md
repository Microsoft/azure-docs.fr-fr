---
title: Workflow d’Azure Attestation
description: Workflow d’Azure Attestation.
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.openlocfilehash: 09d793f3d8ed544a386a362677f24be6d18673d7
ms.sourcegitcommit: 003ac3b45abcdb05dc4406661aca067ece84389f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96748725"
---
# <a name="workflow"></a>Workflow

Microsoft Azure Attestation reçoit une preuve d’enclaves qu’elle évalue par rapport à la base de référence de la sécurité Azure et aux stratégies configurables. Si la vérification est concluante, Azure Attestation génère un jeton d’attestation pour vérifier la fiabilité des enclaves.

Voici les acteurs impliqués dans un workflow Azure Attestation :

- **Partie de confiance** : composant qui s’appuie sur Azure Attestation pour vérifier la validité de l’enclave. 
- **Client** : composant qui collecte des informations à partir d’une enclave et envoie des demandes à Azure Attestation. 
- **Azure Attestation** : composant qui accepte la preuve de l’enclave du client, la valide et retourne le jeton d’attestation au client.


## <a name="intel-software-guard-extensions-sgx-enclave-validation-work-flow"></a>Workflow de validation d’enclave Intel® SGX (Software Guard Extensions)

Voici les étapes générales d’un workflow d’attestation d’enclave SGX type (en utilisant Azure Attestation) :

1. Le client collecte la preuve auprès d’une enclave. La preuve est constituée d’informations sur l’environnement de l’enclave et la bibliothèque de client qui s’exécute dans l’enclave.
1. Le client a un URI qui fait référence à une instance d’Azure Attestation. Le client envoie la preuve à Azure Attestation. Les informations envoyées au fournisseur dépendent du type d’enclave.
1. Azure Attestation valide les informations soumises et les évalue par rapport à une stratégie configurée. Si la vérification est réussie, Azure Attestation émet un jeton d’attestation et le retourne au client. Si cette étape échoue, Azure Attestation signale une erreur au client. 
1. Le client envoie le jeton d’attestation à la partie de confiance. La partie de confiance appelle le point de terminaison de métadonnées de clé publique d’Azure Attestation pour récupérer les certificats de signature. La partie de confiance vérifie ensuite la signature du jeton d’attestation et garantit la fiabilité de l’enclave. 

![Flux de validation d’enclave SGX](./media/sgx-validation-flow.png)

> [!Note]
> Quand vous envoyez des demandes d’attestation dans la version d’API [2018-09-01-preview](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/attestation/data-plane/Microsoft.Attestation/stable/2018-09-01-preview), le client doit envoyer la preuve à Azure Attestation avec le jeton d’accès Azure AD.

## <a name="next-steps"></a>Étapes suivantes
- [Guide pratique pour créer et signer une stratégie d’attestation](author-sign-policy.md)
- [Configurer Azure Attestation à l’aide de PowerShell](quickstart-powershell.md)
