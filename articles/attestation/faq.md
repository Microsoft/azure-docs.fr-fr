---
title: Forum aux questions
description: Réponses aux questions fréquemment posées sur Microsoft Azure Attestation
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: reference
ms.date: 07/20/2020
ms.author: mbaldwin
ms.openlocfilehash: 39f628845bdc9d54b48b1c8037f4a506a9d5c00a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89236519"
---
# <a name="frequently-asked-questions-for-microsoft-azure-attestation"></a>Foire aux questions pour Microsoft Azure Attestation

Cet article fournit des réponses à certaines des questions les plus courantes sur [Azure Attestation](overview.md).

Si votre problème Azure n’est pas traité dans cet article, vous pouvez également envoyer une demande de support Azure sur la [page du support Azure](https://azure.microsoft.com/support/options/).

## <a name="what-is-azure-pck-caching-service-and-its-role-in-enclave-attestation"></a>Qu’est-ce que le service caching Azure PCK et son rôle dans l’attestation d’enclave ?

Le service caching Azure PCK définit la ligne de base an matière de sécurité Azure pour les nœuds[Informatique confidentielle Azure (ACC)](../confidential-computing/overview.md) d’Intel et met en cache les données. Les informations mises en cache seront ensuite utilisées par Azure Attestation dans la validation des environnements d’exécution approuvés (TEE).  

Le service caching Azure PCK :
   - Offre une haute disponibilité. 
   - Réduit la dépendance vis-à-vis des services hébergés à l’extérieur et de la connectivité Internet.
   - Récupère les versions les plus récentes des certificats Intel, des listes de révocation de certificats, des informations TCB (Trusted Computing Base) et de l’identité Quoting Enclave des nœuds ACC auprès d’Intel. Le service confirme donc la ligne de base de sécurité Azure à laquelle doit se référer Azure Attestation lors de la validation des TEE, ce qui réduit considérablement les échecs d’attestation dus à l’invalidation ou à la révocation des certificats Intel.  

## <a name="is-sgx-attestation-supported-by-azure-attestation-in-non-azure-environments"></a>L’attestation SGX est-elle prise en charge par Azure Attestation dans des environnements non-Azure ?

Azure Attestation dépend de la ligne de base de sécurité indiquée par le service caching Azure PCK pour valider les TEE. Le service caching Azure PCK est actuellement conçu pour prendre en charge uniquement les nœuds Informatique confidentielle Azure. 

## <a name="what-validations-does-azure-attestation-perform-for-attesting-sgx-enclaves"></a>Quelles sont les validations effectuées par Azure Attestation pour attester des enclaves SGX ?

Azure Attestation est une infrastructure unifiée permettant d’attester à distance de différents types de TEE. Azure Attestation :

   - Valide si la racine approuvée d’un devis d’enclave signé appartient à Intel.
   - Valide si le devis d’enclave est conforme à la ligne de base de sécurité Azure, telle que définie par le service caching Azure PCK.
   - Valide si le hachage SHA256 de la propriété Enclave Held Data (EHD) dans l’objet de demande d’attestation correspond aux 32 premiers octets du champ reportData dans le devis d’enclave.
   - Permet aux clients de créer un fournisseur d’attestations et de configurer une stratégie personnalisée. Outre les validations ci-dessus, Azure Attestation évalue le devis d’enclave par rapport à la stratégie. Les stratégies définissent des règles d’autorisation pour l’enclave et dictent les règles de délivrance pour la génération du jeton d’attestation. Pour confirmer si le logiciel prévu fonctionne dans une enclave, les clients peuvent ajouter des règles d’autorisation afin de vérifier si les champs **mrsigner** et **mrenclave** du devis d’enclave correspondent aux valeurs des fichiers binaires du client.

## <a name="how-can-a-verifier-obtain-the-collateral-for-sgx-attestation-supported-by-azure-attestation"></a>Comment un vérificateur peut-il obtenir la documentation relative à l’attestation SGX prise en charge par Azure Attestation ?

En général, pour les modèles d’attestation ayant Intel comme racine de confiance, le client d’attestation parle aux API d’enclave pour extraire la preuve de l’enclave. Les API d’enclave appellent en interne le service caching Intel PCK pour extraire les certificats Intel du nœud à attester. Les certificats sont utilisés pour signer la preuve de l’enclave, générant ainsi une documentation pouvant être attestée à distance.  

Le même processus peut être implémenté pour Azure Attestation. Toutefois, pour tirer parti des avantages offerts par le service caching Azure PCK, après l’installation de la machine virtuelle ACC, il est recommandé d’installer la [bibliothèque Azure DCAP](https://www.nuget.org/packages/Microsoft.Azure.DCAP). Selon l’accord conclu avec Intel, lors de l’installation de la bibliothèque Azure DCAP, les demandes de génération de preuve d’enclave sont redirigées du service caching Intel PCK vers le service caching Azure PCK. La bibliothèque Azure DCAP est prise en charge dans les environnements Windows et Linux.

## <a name="how-to-shift-to-azure-attestation-from-other-attestation-models"></a>Comment passer à Azure Attestation à partir d’autres modèles d’attestation ?

- Après l’installation de la machine virtuelle Informatique confidentielle Azure, installez la bibliothèque Azure DCAP ([Windows](https://www.nuget.org/packages/Microsoft.Azure.DCAP/)/[Linux](https://packages.microsoft.com/ubuntu/18.04/prod/pool/main/a/az-dcap-client/)) pour tirer parti des avantages offerts par le service caching Azure PCK.
- Un client d’attestation à distance doit être créé pour pouvoir récupérer la preuve de l’enclave et envoyer des demandes à Azure Attestation. Voir des [exemples de code](/samples/browse/?expanded=azure&terms=attestation) pour référence. 
- Les demandes d’attestation peuvent être envoyées au point de terminaison de l’API REST des fournisseurs par défaut ou des fournisseurs d’attestations personnalisés. 
- Les API Azure Attestation sont protégées par l’authentification Azure AD. Par conséquent, le client qui appelle les API d’attestation doit pouvoir obtenir et transmettre un jeton d’accès Azure AD valide dans la demande d’attestation. 

## <a name="how-can-the-relying-party-verify-the-integrity-of-attestation-token"></a>Comment la partie de confiance peut-elle vérifier l’intégrité du jeton d’attestation ?

Le jeton d’attestation généré par Azure Attestation est signé à l’aide d’un certificat auto-signé. Les certificats sont exposés via un [point de terminaison de métadonnées OpenID](/rest/api/attestation/metadataconfiguration/get). La partie de confiance peut récupérer les certificats de signature à partir de ce point de terminaison et vérifier la signature du jeton d’attestation. La durée de validité du jeton d’attestation est de 8 heures. 

## <a name="how-to-identify-the-certificate-to-be-used-for-signature-verification-from-the-openid-metadata-endpoint"></a>Comment identifier le certificat à utiliser pour la vérification de la signature dans le point de terminaison de métadonnées OpenID ?

Plusieurs certificats exposés dans le point de terminaison de métadonnées OpenID correspondent à différents cas d’utilisation (par exemple, l’attestation SGX) pris en charge par Azure Attestation. Conformément aux critères spécifiés par la norme [RFC 7515](https://tools.ietf.org/html/rfc7515), le certificat dont l’ID de clé (kid) correspond au paramètre *Kid* dans l’en-tête du jeton d’attestation doit être utilisé pour la vérification de la signature. Si aucune correspondance **kid** n’est trouvée, vous devez essayer tous les certificats exposés par le point de terminaison de métadonnées OpenID.

## <a name="is-it-possible-for-the-relying-party-to-share-secrets-with-the-validated-trusted-execution-environments-tees"></a>Est-il possible pour la partie de confiance de partager des secrets avec les environnements d’exécution approuvés (TEE) validés ?

La clé publique générée au sein d’une enclave peut être exprimée dans la propriété Enclave Held Data (EHD) de l’objet de demande d’attestation envoyé par le client à Azure Attestation. Après avoir confirmé si le hachage SHA256 d’EHD est exprimé dans le champ reportData du devis, Azure Attestation inclut EHD dans le jeton d’attestation. La partie de confiance peut utiliser la propriété EHD de la réponse d’attestation vérifiée pour chiffrer les secrets et les partager avec l’enclave. Pour plus d’informations, consultez [Concepts de base d’Azure Attestation](basic-concepts.md).
