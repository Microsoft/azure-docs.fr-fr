---
title: Spécification Bring Your Own Key - Azure Key Vault | Microsoft Docs
description: Ce document décrit la spécification Bring Your Own Key (BYOK).
services: key-vault
author: amitbapat
manager: devtiw
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: keys
ms.topic: conceptual
ms.date: 05/29/2020
ms.author: ambapat
ms.openlocfilehash: feef35ef86a933f32949468366fea85eb87d4866
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91315777"
---
# <a name="bring-your-own-key-specification"></a>Spécification Bring Your Own Key

Ce document décrit les spécifications relatives à l'importation dans Key Vault de clés protégées par HSM depuis les modules HSM locaux des clients.

## <a name="scenario"></a>Scénario

Un client Key Vault souhaite transférer en toute sécurité une clé de son module HSM local (en dehors d'Azure) vers le module HSM qui soutient Azure Key Vault. Le processus d'importation d'une clé générée en dehors de Key Vault est généralement appelé Bring Your Own Key (BYOK).

Les conditions requises sont les suivantes :
* La clé à transférer n'existe jamais en dehors d'un module HSM en clair.
* En dehors d'un module HSM, la clé à transférer est toujours protégée par une clé conservée dans le module HSM Azure Key Vault.

## <a name="terminology"></a>Terminologie

|Nom de la clé|Type de clé|Origine|Description|
|---|---|---|---|
|Key Exchange Key (KEK)|RSA|HSM Azure Key Vault|Paire de clés RSA-HSM générée dans Azure Key Vault
Clé d'enveloppement|AES|HSM du fournisseur|Clé AES [éphémère] générée localement par HSM
Clé cible|RSA, EC, AES|HSM du fournisseur|Clé à transférer au HSM d’Azure Key Vault

**Key Exchange Key (KEK)**  : clé HSM que le client génère dans le coffre de clés où la clé BYOK sera importée. Les propriétés de cette KEK doivent être les suivantes :

* Il s'agit d'une clé RSA-HSM (4096, 3072 ou 2048 bits)
* Elle disposera de key_ops fixes (« import » UNIQUEMENT) qui lui permettront d'être utilisée EXCLUSIVEMENT dans le cadre du processus BYOK
* Elle doit se trouver dans le coffre où la clé cible sera importée

## <a name="user-steps"></a>Procédure à suivre par l'utilisateur

Pour effectuer un transfert de clé, en tant qu'utilisateur, vous devez procéder comme suit :

1. Générez la KEK.
2. Récupérez la clé publique de la KEK.
3. À l'aide de l'outil BYOK du fournisseur du module HSM, importez la KEK dans le module HSM cible et exportez la clé cible protégée par la KEK.
4. Importez la clé cible protégée dans Azure Key Vault.

À l'étape 3, les clients utilisent l'outil BYOK et la documentation du fournisseur du module HSM. Un objet blob de transfert de clé est généré (fichier « .byok »).


## <a name="hsm-constraints"></a>Contraintes HSM

Les modules HSM existants peuvent imposer des contraintes à la clé qu'ils gèrent, notamment :
* Il peut être nécessaire de configurer le module HSM afin qu'il autorise l'exportation basée sur l'enveloppement de la clé.
* Il peut être nécessaire de marquer la clé cible comme CKA_EXTRACTABLE pour que le module HSM autorise une exportation contrôlée.
* Dans certains cas, il peut être nécessaire de marquer la KEK et la clé d'enveloppement comme CKA_TRUSTED. Cela permet de l'utiliser pour envelopper les clés dans le module HSM.

La configuration du module HSM source est généralement en dehors du champ d'application de cette spécification. Microsoft attend du fournisseur du module HSM qu'il produise une documentation pour accompagner son outil BYOK afin d'y inclure ces étapes de configuration.

> [!NOTE]
> Les étapes 1, 2 et 4 décrites ci-dessous peuvent être effectuées à l'aide d'autres interfaces telles qu'Azure PowerShell et le portail Azure. Elles peuvent également être exécutées par programmation à l'aide de fonctions équivalentes du kit de développement logiciel Key Vault SDK.

### <a name="step-1-generate-kek"></a>Étape 1 : Générer la KEK

Pour créer une clé KEK avec les opérations de clé définies sur import, utilisez la commande **az keyvault key create**. Notez l’identificateur de clé « kid » retourné à partir de la commande ci-dessous.

```azurecli
az keyvault key create --kty RSA-HSM --size 4096 --name KEKforBYOK --ops import --vault-name ContosoKeyVaultHSM
```

### <a name="step-2-retrieve-the-public-key-of-the-kek"></a>Étape 2 : Récupérer la clé publique de la KEK

Téléchargez la partie clé publique de la KEK et stockez-la dans un fichier PEM.

```azurecli
az keyvault key download --name KEKforBYOK --vault-name ContosoKeyVaultHSM --file KEKforBYOK.publickey.pem
```

### <a name="steps-3-generate-key-transfer-blob-using-hsm-vendor-provided-byok-tool"></a>Étape 3 : Générer un objet blob de transfert de clé à l'aide de l'outil BYOK du fournisseur du module HSM

Le client utilisera l'outil BYOK proposé par le fournisseur du module HSM pour créer un objet blob de transfert de clé (stocké dans un fichier « .byok »). La clé publique KEK (sous forme de fichier .pem) sera l'une des entrées de cet outil.

#### <a name="key-transfer-blob"></a>Objet blob de transfert de clé
À long terme, Microsoft souhaite utiliser le mécanisme CKM_RSA_AES_KEY_WRAP PKCS#11 pour transférer la clé cible vers Azure Key Vault car ce mécanisme produit un seul objet blob et, plus important encore, la clé AES intermédiaire est gérée par les deux modules HSM et est garantie éphémère. Ce mécanisme n'est actuellement pas disponible dans certains modules HSM, mais la combinaison de la protection de la clé cible avec CKM_AES_KEY_WRAP_PAD à l'aide d'une clé AES et de la protection de la clé AES avec CKM_RSA_PKCS_OAEP produit un objet blob équivalent.

Le texte en clair de la clé cible dépend du type de clé : 
* Pour une clé RSA, l'encodage DER ASN.1 de clé privée [RFC3447] enveloppé dans PKCS#8 [RFC5208] 
* Pour une clé EC, l'encodage DER ASN.1 de clé privée [RFC5915] enveloppé dans PKCS#8 [RFC5208]
* Pour une clé d'octet, les octets bruts de la clé

Les octets de la clé en clair sont ensuite transformés à l'aide du mécanisme CKM_RSA_AES_KEY_WRAP :
* Une clé AES éphémère est générée et chiffrée avec la clé RSA d'enveloppement en utilisant RSA-OAEP avec SHA1.
* La clé en clair encodée est chiffrée à l'aide de la clé AES en utilisant AES Key Wrap avec remplissage.
* La clé AES chiffrée et la clé en clair chiffrée sont concaténées pour produire l'objet blob de texte chiffré final.

Le format de l'objet blob de transfert utilise la sérialisation compacte JSON Web Encryption (RFC7516) principalement comme moyen de fournir les métadonnées requises au service pour un déchiffrement adéquat.

Si CKM_RSA_AES_KEY_WRAP_PAD est utilisé, la sérialisation JSON de l'objet blob de transfert sera :

```json
{
  "schema_version": "1.0.0",
  "header":
  {
    "kid": "<key identifier of the KEK>",
    "alg": "dir",
    "enc": "CKM_RSA_AES_KEY_WRAP"
  },
  "ciphertext":"BASE64URL(<ciphertext contents>)",
  "generator": "BYOK tool name and version; source HSM name and firmware version"
}

```

* kid = identificateur de clé de la KEK. Pour les clés Key Vault, il se présente comme suit : https://ContosoKeyVaultHSM.vault.azure.net/keys/mykek/eba63d27e4e34e028839b53fac905621
* alg = algorithme. 
* dir = mode direct, c'est-à-dire que le kid référencé est utilisé pour protéger directement le texte chiffré qui est une représentation précise de CKM_RSA_AES_KEY_WRAP
* generator = champ d'informations qui indique le nom et la version de l'outil BYOK ainsi que le fabricant et le modèle du module HSM source. Ces informations sont destinées à être utilisées pour la résolution des problèmes et le support.

L'objet blob JSON est stocké dans un fichier « .byok » afin que les clients Azure PowerShell/CLI le traitent correctement lorsque les commandes « Add-AzKeyVaultKey » (PSH) ou « az keyvault key import » (CLI) sont utilisées.

### <a name="step-4-upload-key-transfer-blob-to-import-hsm-key"></a>Étape 4 : Charger l'objet blob de transfert de clé pour importer la clé du module HSM

Le client transférera l'objet blob de transfert de clé (fichier « .byok ») vers une station de travail en ligne, puis exécutera une commande **az keyvault key import** pour importer cet objet blob en tant que nouvelle clé HSM dans Key Vault. 

```azurecli
az keyvault key import --vault-name ContosoKeyVaultHSM --name ContosoFirstHSMkey --byok-file KeyTransferPackage-ContosoFirstHSMkey.byok --ops encrypt decrypt
```

Lorsque la commande ci-dessus est exécutée, elle se traduit par l'envoi d'une requête d'API REST :

```
PUT https://contosokeyvaulthsm.vault.azure.net/keys/ContosoFirstHSMKey?api-version=7.0
```

Corps de la requête :
```json
{
  "key": {
    "kty": "RSA-HSM",
    "key_ops": [
      "decrypt",
      "encrypt"
    ],
    "key_hsm": "<Base64 encoded BYOK_BLOB>"
  },
  "attributes": {
    "enabled": true
  }
}
```
la valeur « key_hsm » est le contenu entier du fichier KeyTransferPackage-ContosoFirstHSMkey.byok encodé au format Base64.

## <a name="references"></a>References
- [Guide du développeur Key Vault](../general/developers-guide.md)

## <a name="next-steps"></a>Étapes suivantes
* BYOK - Instructions pas à pas : [Importer des clés protégées par HSM dans un coffre de clés (BYOK)](hsm-protected-keys-byok.md)

