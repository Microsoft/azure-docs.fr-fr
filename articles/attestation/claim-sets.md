---
title: Ensembles de revendications d’Azure Attestation
description: Ensembles de revendications d’Azure Attestation
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.openlocfilehash: e17002534d35f477467f0c35833560a0267dd596
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92909775"
---
# <a name="claim-sets"></a>Ensembles de revendications

Les revendications générées dans le processus d’attestation d’enclaves à l’aide de Microsoft Azure Attestation peuvent être réparties dans les catégories ci-dessous :

- **Revendications entrantes**  : revendications générées par Microsoft Azure Attestation après l’analyse de la preuve d’attestation.

- **Revendications sortantes**  : revendications créées en tant que sortie par Azure Attestation. Cette catégorie contient toutes les revendications qui doivent se retrouver dans le jeton d’attestation.

- **revendications de propriété**  : revendications créées en tant que sortie par Azure Attestation. Cette catégorie contient toutes les revendications qui représentent les propriétés du jeton d’attestation, telles que l’encodage du rapport, la durée de validité du rapport, etc.

Les revendications ci-après sont définies par le document RFC JWT et utilisées par Azure Attestation dans l’objet de réponse :

- **Revendication « iss » (émetteur)**  : La revendication « ISS » (émetteur) identifie le principal qui a émis le jeton JWT. Le traitement de cette revendication est généralement spécifique à l’application. La valeur « ISS » est une chaîne qui respecte la casse et qui contient une valeur StringOrURI.
- **Revendication « iat » (émission à)**  : la revendication « iat » (émission à) identifie l’heure à laquelle le jeton JWT a été émis. Cette revendication peut être utilisée pour déterminer l’ancienneté du jeton JWT. Sa valeur DOIT être un nombre contenant une valeur NumericDate.
- **Revendication « exp » (délai d’expiration)**  : La revendication « exp » (délai d'expiration) indique le délai d'expiration à partir duquel le JWT ne doit PAS être accepté pour être traité. Le traitement de la revendication « exp » nécessite que la date/heure actuelle SOIT antérieure à la date/heure d’expiration indiquée dans la revendication « exp ».

  Remarque : Une marge de 5 minutes est ajoutée à l’heure d’émission (iat) pour tenir compte du décalage de l’horloge.
- **Revendication « nbf » (pas avant)**  : la revendication « nbf » (pas avant) indique le délai devant s’écouler avant que le JWT soit accepté pour être traité. Le traitement de la revendication « nbf » nécessite que la date/heure actuelle SOIT postérieure ou égale à la date/heure « not-before » (pas avant) indiquée dans la revendication « nbf ».
  Remarque : Une marge de 5 minutes est ajoutée à l’heure d’émission (iat) pour tenir compte du décalage de l’horloge.

## <a name="claims-issued-by-azure-attestation-in-sgx-enclaves"></a>Revendications émises par Azure Attestation dans les enclaves SGX

### <a name="incoming-claims"></a>Revendications entrantes 

- **$is-debuggable**  : valeur booléenne qui indique si le débogage est activé ou non pour l’enclave.
- **$sgx-mrsigner**  : valeur encodée hexadécimale du champ « mrsigner » de la déclaration
- **$sgx-mrenclave**  : valeur encodée hexadécimale du champ « mrenclave » de la déclaration
- **$product-id**
- **$svn**  : numéro de version de sécurité encodé dans la déclaration 
- **$tee**  : type d’enclave 

### <a name="outgoing-claims"></a>Revendications sortantes

- **is-debuggable**  : valeur booléenne qui indique si le débogage est activé ou non pour l’enclave.
- **sgx-mrsigner**  : valeur encodée hexadécimale du champ « mrsigner » de la déclaration
- **sgx-mrenclave**  : valeur encodée hexadécimale du champ « mrenclave » de la déclaration
- **product-id**
- **svn**  : numéro de version de sécurité encodé dans la déclaration 
- **tee**  : type d’enclave 
- **maa-ehd**  :  version encodée Base64Url des « données détenues par l’enclave » spécifiées dans la demande d’attestation 
- **aas-ehd**  :  version encodée Base64Url des « données détenues par l’enclave » spécifiées dans la demande d’attestation 

## <a name="claims-issued-by-azure-attestation-in-vbs-enclaves"></a>Revendications émises par Azure Attestation dans les enclaves VBS

### <a name="incoming-claims-can-also-be-used-as-outgoing-claims"></a>Revendications entrantes (également utilisables comme revendications sortantes)

- **aikValidated**  :  valeur booléenne contenant des informations si le certificat de la clé d’attestation d’identité (AIK) a été validé ou non.
- **aikPubHash**  :  chaîne contenant la valeur base64(SHA256(clé publique AIK au format DER)).
- **tpmVersion**  :   valeur entière contenant la version principale du module de plateforme sécurisée (TPM).
- **secureBootEnabled**  : valeur booléenne indiquant si le démarrage sécurisé est activé.
- **iommuEnabled**  :  valeur booléenne indiquant si l’unité de gestion de la mémoire d’entrée-sortie (Iommu) est activée.
- **bootDebuggingDisabled**  : valeur booléenne pour indiquer si le débogage de démarrage est désactivé.
- **notSafeMode**  :  valeur booléenne indiquant si Windows ne s’exécute pas en mode sans échec.
- **notWinPE**  :  valeur booléenne indiquant si Windows ne s’exécute pas en mode WinPE.
- **vbsEnabled**  :  valeur booléenne indiquant si VBS est activé.
- **vbsReportPresent**  :  valeur booléenne indiquant si le rapport de l’enclave VBS est disponible.
- **enclaveAuthorId**  :  valeur de chaîne contenant la valeur encodée Base64Url de l’ID de l’auteur de l’enclave. Identificateur de l’auteur du module principal de l’enclave.
- **enclaveImageId**  :  valeur de chaîne contenant la valeur encodée Base64Url de l’ID de l’image de l’enclave. Identificateur de l’image du module principal de l’enclave.
- **enclaveOwnerId**  :  valeur de chaîne contenant la valeur encodée Base64Url de l’ID du propriétaire de l’enclave. Identificateur du propriétaire de l’enclave.
- **enclaveFamilyId**  :  valeur de chaîne contenant la valeur encodée Base64Url de l’ID de famille de l’enclave. Identificateur de la famille du module principal de l’enclave.
- **enclaveSvn**  :  valeur entière contenant le numéro de la version de sécurité du module principal de l’enclave.
- **enclavePlatformSvn**  :  valeur entière contenant le numéro de la version de sécurité de la plateforme qui héberge l’enclave.
- **enclaveFlags**  :  la revendication enclaveFlags est une valeur entière qui contient des indicateurs décrivant la stratégie d’exécution pour l’enclave.
  
### <a name="outgoing-claims"></a>Revendications sortantes

- **policy_hash**  :  valeur de chaîne contenant le hachage SHA256 du texte de la stratégie calculé par BASE64URL(SHA256(BASE64URL(UTF8(texte de la stratégie)))).
- **policy_signer**  :  Contient un objet JWK avec la clé publique ou la chaîne de certificats présente dans l’en-tête de stratégie signé.
- **ver (Version)**  :  valeur de chaîne contenant la version du rapport. Il s’agit de la version 1.0.
- **Revendication cnf (confirmation)**  :  La revendication « cnf » est utilisée pour identifier la clé de preuve de possession. La revendication de confirmation telle que définie dans le document RFC 7800 contient la partie publique de la clé de l’enclave attestée représentée sous la forme d’un objet JWK (JSON Web Key) (RFC 7517).
- **rp_data (données de la partie de confiance)**  :  données de la partie de confiance, le cas échéant, spécifiées dans la demande, utilisées par la partie de confiance comme nonce pour garantir le caractère actuel du rapport.
- **Revendication « jti »(ID JWT)**  : La revendication « JTI » (ID JWT) fournit un identificateur unique pour le jeton JWT. La valeur de l’identificateur est attribuée de manière à garantir qu’il y a une probabilité négligeable que la même valeur soit affectée par accident à un objet de données différent.

### <a name="property-claims"></a>Revendications de propriété

- **report_validity_in_minutes**  : revendication sous forme d’entier indiquant la durée de validité du jeton.
  - **Valeur par défaut (durée)**  : un jour en minutes.
  - **Valeur maximale (durée)**  : un an en minutes.
- **omit_x5c**  : revendication sous forme booléenne indiquant si Azure Attestation doit omettre le certificat utilisé pour fournir la preuve de l’authenticité du service. Si la valeur est true, x5t est ajouté au jeton d’attestation. Si la valeur est false (valeur par défaut), x5c est ajouté au jeton d’attestation.

## <a name="next-steps"></a>Étapes suivantes
- [Guide pratique pour créer et signer une stratégie d’attestation](author-sign-policy.md)
- [Configurer Azure Attestation à l’aide de PowerShell](quickstart-powershell.md)
