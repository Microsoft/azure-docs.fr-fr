---
title: Concepts de base d’Azure Attestation
description: Concepts de base d’Azure Attestation.
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.openlocfilehash: 8ae5bcf103bbb2d2b952fa647ba591e49002f2ff
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96921618"
---
# <a name="basic-concepts"></a>Concepts de base

Voici quelques concepts de base relatifs à Microsoft Azure Attestation.

## <a name="json-web-token-jwt"></a>JSON Web Token (JWT)

[JSON Web Token](https://jwt.io/) (JWT) est une méthode [RFC7519](https://tools.ietf.org/html/rfc7519) de norme ouverte pour la transmission d’informations de manière sécurisée entre des tiers sous la forme d’un objet JSON (JavaScript Object Notation). Ces informations peuvent être vérifiées et approuvées, car elles sont signées numériquement. Les jetons JWT peuvent être signés à l’aide d’un secret ou d’une paire de clés publique/privée.

## <a name="json-web-key-jwk"></a>Clé web JSON (JWK)

[JSON Web Key](https://tools.ietf.org/html/rfc7517) (JWK) est une structure de données JSON qui représente une clé de chiffrement. Cette spécification définit également une structure de données JSON JWK Set qui représente un ensemble de clés JWK.

## <a name="attestation-provider"></a>Fournisseur d’attestations

Le fournisseur d’attestations appartient au fournisseur de ressources Azure nommé Microsoft.Attestation. Le fournisseur de ressources est un point de terminaison de service qui fournit un contrat REST Azure Attestation et qui est déployé à l’aide d’[Azure Resource Manager](../azure-resource-manager/management/overview.md). Chaque fournisseur d’attestations honore une stratégie spécifique et détectable. 

Les fournisseurs d’attestations sont créés avec une stratégie par défaut pour chaque type d’attestation (notez que l’enclave VBS n’a pas de stratégie par défaut). Pour plus d’informations sur la stratégie par défaut pour SGX, consultez [Exemples de stratégie d’attestation](policy-examples.md).

### <a name="regional-default-provider"></a>Fournisseur par défaut régional

Azure Attestation fournit un fournisseur par défaut dans chaque région. Les clients peuvent choisir d’utiliser le fournisseur par défaut pour l’attestation ou créer leurs propres fournisseurs avec des stratégies personnalisées. Les fournisseurs par défaut sont accessibles à tout utilisateur Azure AD et la stratégie associée à un fournisseur par défaut ne peut pas être modifiée.

| Région | URI d’attestation | 
|--|--|
| Sud du Royaume-Uni | `https://shareduks.uks.attest.azure.net` | 
| USA Est 2 | `https://sharedeus2.eus2.attest.azure.net` | 
| USA Centre | `https://sharedcus.cus.attest.azure.net` | 
| USA Est| `https://sharedeus.eus.attest.azure.net` | 
| Centre du Canada | `https://sharedcac.cac.attest.azure.net` | 

## <a name="attestation-request"></a>Demande d’attestation

La demande d’attestation est un objet JSON sérialisé envoyé par une application cliente à un fournisseur d’attestations. L’objet de la demande pour l’enclave SGX a deux propriétés : 
- « Quote » : la valeur de la propriété « Quote » est une chaîne contenant une représentation encodée Base64URL de la déclaration d’attestation
- « EnclaveHeldData » : la valeur de la propriété « EnclaveHeldData » est une chaîne contenant une représentation encodée Base64URL des données détenues par l’enclave.

Azure Attestation valide le « Quote » fourni et vérifie ensuite que le hachage SHA256 des données détenues par l’enclave est exprimé dans les 32 premiers octets du champ reportData du Quote. 

## <a name="attestation-policy"></a>Stratégie d’attestation

La stratégie d’attestation est utilisée pour traiter la preuve d’attestation et peut être configurée par les clients. Au cœur d’Azure Attestation se trouve un moteur de stratégie, qui traite les revendications constituant la preuve. Les stratégies sont utilisées pour déterminer si Azure Attestation doit émettre un jeton d’attestation en fonction de la preuve (ou non) et, ainsi, endosser l’attesteur (ou non). En conséquence, si le traitement par toutes les stratégies n’est pas couronné de succès, aucun jeton JWT n’est émis.

Si la stratégie par défaut dans le fournisseur d’attestations ne répond pas aux besoins, les clients peuvent créer des stratégies personnalisées dans l’une des régions prises en charge par Azure Attestation. La gestion des stratégies est une fonctionnalité clé fournie aux clients par Azure Attestation. Les stratégies sont propres au type d’attestation et peuvent être utilisées pour identifier les enclaves, ajouter ou modifier des revendications dans un jeton de sortie. 

Consultez [Exemples de stratégie d’attestation](policy-examples.md) pour connaître le contenu par défaut des stratégies et obtenir des exemples de stratégie.

## <a name="benefits-of-policy-signing"></a>Avantages de la signature de stratégie

Une stratégie d’attestation est ce qui détermine en dernier lieu si un jeton d’attestation doit être émis par Azure Attestation. La stratégie détermine également les revendications à générer dans le jeton d’attestation. Il est donc primordial que la stratégie évaluée par le service soit de fait la stratégie écrite par l’administrateur et qu’elle n’ait pas été falsifiée ou modifiée par des entités externes. 

Le modèle d’approbation définit le modèle d’autorisation du fournisseur d’attestations pour définir et mettre à jour la stratégie.  Deux modèles sont pris en charge : l’un est basé sur une autorisation Azure AD, tandis que l’autre, appelé « modèle isolé », repose sur la possession de clés de chiffrement gérées par le client.  Le modèle isolé permet à Azure Attestation de s’assurer que la stratégie soumise par le client n’est pas falsifiée.

Dans le modèle isolé, l’administrateur crée un fournisseur d’attestations en spécifiant un ensemble de certificats X.509 de signature approuvés dans un fichier. L’administrateur peut ensuite ajouter une stratégie signée au fournisseur d’attestations. Lors du traitement de la demande d’attestation, Azure Attestation valide la signature de la stratégie à l’aide de la clé publique représentée par le paramètre « jwk » ou « x5c » dans l’en-tête.  Azure Attestation vérifie également si la clé publique dans l’en-tête de la demande figure dans la liste des certificats de signature approuvés associés au fournisseur d’attestations. De cette façon, la partie de confiance (Azure Attestation) peut approuver une stratégie signée à l’aide des certificats X.509 dont elle a connaissance. 

Pour obtenir des exemples, consultez [Exemples de certificat du signataire de stratégie](policy-signer-examples.md).

## <a name="attestation-token"></a>Jeton d’attestation

La réponse d’Azure Attestation est une chaîne JSON dont la valeur contient un jeton JWT. Azure Attestation empaquette les revendications et génère un jeton JWT signé. L’opération de signature est effectuée à l’aide d’un certificat auto-signé dont le nom de l’objet correspond à l’élément AttestUri du fournisseur d’attestations.

L’API de récupération des métadonnées OpenID retourne une réponse de configuration OpenID telle que spécifiée par le [protocole OpenID Connect Discovery](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig). L’API récupère les métadonnées relatives aux certificats de signature utilisés par Azure Attestation.

Exemple de jeton JWT généré pour une enclave SGX :

```
{
  “alg”: “RS256”,
  “jku”: “https://tradewinds.us.attest.azure.net/certs”,
  “kid”: “f1lIjBlb6jUHEUp1/Nh6BNUHc6vwiUyMKKhReZeEpGc=”,
  “typ”: “JWT”
}.{
  “maa-ehd”: <input enclave held data>,
  “exp”: 1568187398,
  “iat”: 1568158598,
  “is-debuggable”: false,
  “iss”: “https://tradewinds.us.attest.azure.net”,
  “nbf”: 1568158598,
  “product-id”: 4639,
  “sgx-mrenclave”: “”,
  “sgx-mrsigner”: “”,
  “svn”: 0,
  “tee”: “sgx”
}.[Signature]
```
Les revendications telles que « exp », « iat », « iss » et « nbf » sont définies par le [document RFC JWT](https://tools.ietf.org/html/rfc7517), tandis que les autres sont générées par Azure Attestation. Pour plus d’informations, consultez les [revendications émises par Azure Attestation](claim-sets.md).

## <a name="encryption-of-data-at-rest"></a>Chiffrement des données au repos

Pour protéger les données des clients, Azure Attestation conserve ses données dans Stockage Azure. Stockage Azure assure un chiffrement des données au repos à mesure qu’elles sont écrites dans les centres de données et les déchiffre pour permettre aux clients d’y accéder. Ce chiffrement s'effectue à l'aide d'une clé de chiffrement gérée par Microsoft. 

En plus de protéger les données dans Stockage Azure, Azure Attestation tire également parti d’Azure Disk Encryption (ADE) pour chiffrer les machines virtuelles du service. Pour Azure Attestation s’exécutant dans une enclave au sein d’environnements informatiques confidentiels Azure, l’extension ADE n’est actuellement pas prise en charge. En pareil cas, pour empêcher que les données soient stockées en mémoire, le fichier d’échange est désactivé. 

Les données des clients ne sont en aucun cas conservées sur les lecteurs de disque dur locaux de l’instance Azure Attestation.


## <a name="next-steps"></a>Étapes suivantes

- [Guide pratique pour créer et signer une stratégie d’attestation](author-sign-policy.md)
- [Configurer Azure Attestation à l’aide de PowerShell](quickstart-powershell.md)
