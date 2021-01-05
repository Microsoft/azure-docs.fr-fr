---
title: Protocole de sécurité basée sur la virtualisation (VBS) pour Azure Attestation
description: Protocole d’attestation VBS
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: reference
ms.date: 07/20/2020
ms.author: mbaldwin
ms.openlocfilehash: e5cc3b5fb7ca38df196119de12d346f5d0346b58
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91338815"
---
# <a name="virtualization-based-security-vbs-attestation-protocol"></a>Protocole d’attestation de sécurité basée sur la virtualisation (VBS) 

Pour que Microsoft Azure Attestation puisse fournir des garanties de sécurité fortes que les données signalées sont authentiques, il est nécessaire de créer une chaîne d’approbation, du microprogramme au lancement de l’hyperviseur et du noyau sécurisé. Pour cela, Azure Attestation doit attester de l’état de démarrage de l’ordinateur afin que nous puissions établir une confiance dans l’enclave sécurisée. Le système d’exploitation, l’hyperviseur et les fichiers binaires du noyau sécurisé doivent être signés par les autorités Microsoft officielles appropriées et configurés de façon sécurisée. Une fois que nous avons établi la confiance entre le Module de plateforme sécurisée (TPM) et l’intégrité de l’hyperviseur, nous pouvons approuver l’IDKS VBS fourni dans le journal de démarrage mesuré. Nous pouvons ainsi valider qu’une paire de clés a été générée par l’enclave et émettre un rapport d’attestation qui établit la confiance dans cette clé et contient d’autres revendications, telles que le niveau de sécurité et les propriétés d’attestation de démarrage.

## <a name="protocol-messages"></a>Messages de protocole

### <a name="init-message"></a>Message d’initialisation

#### <a name="direction"></a>Sens

Client -> Azure Attestation

#### <a name="payload"></a>Payload

```
{
  "type": "aikcert"
}
```

« type » (chaîne ASCII) : représente le type d’attestation demandé. Actuellement, seul « aikcert » est pris en charge.

### <a name="challenge-message"></a>Message de vérification

#### <a name="direction"></a>Sens

Azure Attestation -> Client

#### <a name="payload"></a>Payload

```
{

  "challenge": "<BASE64URL(CHALLENGE)>",
  
  "service_context": "<BASE64URL(SERVICECONTEXT)>"
  
}
```

**vérification** (BASE64URL(OCTETS)) : Valeur aléatoire émise par le service.

**service_context** (BASE64URL(OCTETS)) : Contexte chiffré, opaque, créé par le service, qui comprend, entre autres, la vérification et le délai d’expiration pour celle-ci.


### <a name="request-message"></a>Message de requête

#### <a name="direction"></a>Sens

Client -> Azure Attestation 

#### <a name="payload"></a>Payload

```
{

  "request": "<JWS>"
  
}
```

**demande** (JWS) : La demande se compose d’une structure de signature web JSON (JWS). L’en-tête protégé JWS et la charge utile JWS sont présentés ci-dessous. Comme dans toute structure JWS, la valeur finale se compose comme suit :

BASE64URL(UTF8(En-tête protégé JWS)) || ’.’ ||

BASE64URL(Charge utile JWS) || ’.’ ||

BASE64URL(Signature JWS)

##### <a name="jws-protected-header"></a>En-tête protégé JWS

```
{
  "alg": "PS256",
  "typ": "attReq"
  // no "kid" parameter as the key specified by attest_key MUST sign this JWS to prove possession.
}
```

##### <a name="jws-payload"></a>Charge utile JWS

La charge utile JWS peut être de type de base ou VBS. Le type de base est utilisé quand la preuve d’attestation n’inclut pas de données VBS.

Exemple de base

``` 
{
  "att_type": "basic",
  "att_data": {
    "rp_id": "<URL>",
    "rp_data": "<BASE64URL(RPCUSTOMDATA)>",
    "challenge": "<BASE64URL(CHALLENGE)>",
    "tpm_att_data": {
      "srtm_boot_log": "<BASE64URL(SRTMBOOTLOG)>",
      "srtm_resume_log": "<BASE64URL(SRTMRESUMELOG)>",
      "drtm_boot_log": "<BASE64URL(DRTMBOOTLOG)>",
      "drtm_resume_log": "<BASE64URL(DRTMRESUMELOG)>",
      "aik_cert": "<BASE64URL(AIKCERTIFICATE)>",
      // aik_pub is represented as a JSON Web Key (JWK) object (RFC 7517).
      "aik_pub": {
        "kty": "RSA",
        "n": "<Base64urlUInt(MODULUS)>",
        "e": "<Base64urlUInt(EXPONENT)>"
      },
      "current_claim": "<BASE64URL(CURRENTCLAIM)>",
      "boot_claim": "<BASE64URL(BOOTCLAIM)>"
    },
    // attest_key is represented as a JSON Web Key (JWK) object (RFC 7517).
    "attest_key": {
      "kty": "RSA",
      "n": "<Base64urlUInt(MODULUS)>",
      "e": "<Base64urlUInt(EXPONENT)>"
    },
    "custom_claims": [
      {
        "name": "<name>",
        "value": "<value>",
        "value_type": "<value_type>"
      },
      {
        "name": "<name>",
        "value": "<value>",
        "value_type": "<value_type>"
      }
    ],
    "service_context": "<BASE64URL(SERVICECONTEXT)>"
  }
}
```

Exemple VBS

``` 
{
  "att_type": "vbs",
  "att_data": {
    "report_signed": {
      "rp_id": "<URL>",
      "rp_data": "<BASE64URL(RPCUSTOMDATA)>",
      "challenge": "<BASE64URL(CHALLENGE)>",
      "tpm_att_data": {
        "srtm_boot_log": "<BASE64URL(SRTMBOOTLOG)>",
        "srtm_resume_log": "<BASE64URL(SRTMRESUMELOG)>",
        "drtm_boot_log": "<BASE64URL(DRTMBOOTLOG)>",
        "drtm_resume_log": "<BASE64URL(DRTMRESUMELOG)>",
        "aik_cert": "<BASE64URL(AIKCERTIFICATE)>",
        // aik_pub is represented as a JSON Web Key (JWK) object (RFC 7517).
        "aik_pub": {
          "kty": "RSA",
          "n": "<Base64urlUInt(MODULUS)>",
          "e": "<Base64urlUInt(EXPONENT)>"
        },
        "current_claim": "<BASE64URL(CURRENTCLAIM)>",
        "boot_claim": "<BASE64URL(BOOTCLAIM)>"
      },
      // attest_key is represented as a JSON Web Key (JWK) object (RFC 7517).
      "attest_key": {
        "kty": "RSA",
        "n": "<Base64urlUInt(MODULUS)>",
        "e": "<Base64urlUInt(EXPONENT)>"
      },
      "custom_claims": [
        {
          "name": "<name>",
          "value": "<value>",
          "value_type": "<value_type>"
        },
        {
          "name": "<name>",
          "value": "<value>",
          "value_type": "<value_type>"
        }
      ],
      "service_context": "<BASE64URL(SERVICECONTEXT)>"
    },
    "vbs_report": "<BASE64URL(REPORT)>"
  }
}
``` 

**rp_id** (StringOrURI) : identificateur de la partie de confiance. Utilisé par le service dans le calcul de la revendication d’ID d’ordinateur

**rp_data** (BASE64URL(OCTETS)) : données opaques transmises par la partie de confiance. Elles sont normalement utilisées par la partie de confiance en tant que nonce pour garantir l’actualisation du rapport.

**challenge** (BASE64URL(OCTETS)) : valeur aléatoire émise par le service

**tpm_att_data** : données d’attestation associées au module de plateforme sécurisée

- **srtm_boot_log (BASE64URL(OCTETS))**  : journal de démarrage SRTM extrait par la fonction Tbsi_Get_TCG_Log_Ex avec le type de journal = TBS_TCGLOG_SRTM_BOOT

- **srtm_resume_log (BASE64URL(OCTETS))**  : journal de reprise SRTM extrait par la fonction Tbsi_Get_TCG_Log_Ex avec le type de journal = TBS_TCGLOG_SRTM_RESUME

- **drtm_boot_log (BASE64URL(OCTETS))**  : journal de démarrage DRTM extrait par la fonction Tbsi_Get_TCG_Log_Ex avec le type de journal = TBS_TCGLOG_DRTM_BOOT

- **drtm_resume_log (BASE64URL(OCTETS))**  : journal de reprise DRTM extrait par la fonction Tbsi_Get_TCG_Log_Ex avec le type de journal = TBS_TCGLOG_DRTM_RESUME

- **aik_cert (BASE64URL(OCTETS))**  : certificat X. 509 pour l’AIK, tel que retourné par la fonction NCryptGetProperty avec la propriété = NCRYPT_CERTIFICATE_PROPERTY

- **aik_pub** : partie publique de l’AIK représentée sous la forme d’un objet clé web JSON (JWK) (RFC 7517)

- **current_claim (BASE64URL(OCTETS))**  : revendication d’attestation pour l’état PCR actuel, tel que retourné par la fonction NCryptCreateClaim avec dwClaimType = NCRYPT_CLAIM_PLATFORM et le paramètre NCRYPTBUFFER_TPM_PLATFORM_CLAIM_PCR_MASK défini pour inclure tous les PCR. La vérification envoyée par le service doit également être utilisée dans le calcul de cette revendication.

- **boot_claim (BASE64URL(OCTETS))**  : revendication d’attestation pour l’état PCR au démarrage, tel que retourné par la fonction NCryptCreateClaim avec dwClaimType = NCRYPT_CLAIM_PLATFORM et le paramètre NCRYPTBUFFER_TPM_PLATFORM_CLAIM_PCR_MASK défini pour inclure tous les PCR.

**vbs report** (BASE64URL(OCTETS)) : rapport d’attestation d’enclave VBS tel que retourné par la fonction EnclaveGetAttestationReport. Le paramètre EnclaveData doit être le hachage SHA-512 de la valeur report_signed (y compris les accolades ouvrantes et fermantes). L’entrée de la fonction de hachage est UTF8(report_signed)

**attest_key** : partie publique de la clé enclave représentée sous la forme d’un objet clé web JSON (JWK) (RFC 7517)

**custom_claims** : tableau de revendications d’enclaves personnalisées envoyées au service, que la stratégie peut évaluer. La revendication

- **name (String)**  : nom de la revendication. Ce nom sera ajouté à une URL déterminée par le service d’attestation (pour éviter les conflits), et la chaîne concaténée devient le type de la revendication qui peut être utilisée dans la stratégie.

- **value (String)**  : valeur de la revendication.

- **value_type (String)**  : type de données de la valeur de la revendication.

**service_context** (BASE64URL(OCTETS)) : Contexte chiffré, opaque, créé par le service, qui comprend, entre autres, la vérification et le délai d’expiration pour celle-ci.

### <a name="report-message"></a>Message du rapport

#### <a name="direction"></a>Sens

Azure Attestation -> Client

#### <a name="payload"></a>Payload

```
{
  "report": "<JWT>"
}
```

**report** (JWT) : rapport d’attestation au format JSON Web Token (JWT) (RFC 7519).
