---
title: Génération et transfert de clés protégées par HSM – BYOK – Azure Key Vault
description: Utilisez les informations de cet article pour planifier, générer, puis transférer vos propres clés protégées par HSM à utiliser avec Azure Key Vault. Cette méthode est également appelée BYOK (Bring Your Own Key, ou apportez votre propre clé).
services: key-vault
author: amitbapat
manager: devtiw
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: keys
ms.topic: tutorial
ms.date: 05/29/2020
ms.author: ambapat
ms.openlocfilehash: a1c6b054a9caac8ba223bc81e164e7ebf34bd267
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94413324"
---
# <a name="import-hsm-protected-keys-to-key-vault-byok"></a>Importer des clés protégées par HSM dans Key Vault (BYOK)

Pour une meilleure garantie, lorsque vous utilisez Azure Key Vault, vous pouvez importer ou générer une clé dans un module de sécurité matériel (HSM), qui ne franchit jamais les limites de celui-ci. Il est souvent fait référence à ce scénario sous le terme *BYOK* (Bring Your Own Key, ou apportez votre propre clé). Azure Key Vault utilise la famille nShield de HSM (FIPS 140-2 niveau 2 validé) de nCipher pour protéger vos clés.

Les informations de cet article vous aident à planifier, à générer puis à transférer vos propres clés protégées par HSM à utiliser avec Azure Key Vault.

> [!NOTE]
> Cette fonctionnalité n’est pas disponible pour Azure Chine 21Vianet. 
> 
> Cette méthode d’importation n’est disponible que pour les [HSM pris en charge](#supported-hsms). 

Pour plus d’informations et pour accéder à un tutoriel sur la prise en main de Key Vault (y compris la création d’un coffre de clés pour des clés protégées par HSM), voir [Qu’est-ce qu’Azure Key Vault ?](../general/overview.md).

## <a name="overview"></a>Vue d’ensemble

Voici une vue d’ensemble du processus. Les étapes spécifiques à effectuer sont décrites plus loin dans cet article.

* Dans Azure Key Vault, générez une clé (*Key Exchange Key*, KEK). Cette KEK doit être une clé RSA-HSM qui n’a que l’opération de clé `import`. Seule la référence SKU Premium d’Azure Key Vault prend en charge les clés RSA-HSM.
* Téléchargez la clé publique KEK en tant que fichier. pem.
* Transférez la clé publique KEK sur un ordinateur hors connexion connecté à un HSM local.
* Sur l’ordinateur hors connexion, utilisez l’outil BYOK fourni par votre fournisseur de HSM pour créer un fichier BYOK. 
* La clé cible est chiffrée à l’aide d’une clé KEK qui reste chiffrée jusqu’à ce qu’elle soit transférée vers le HSM d’Azure Key Vault. Seule la version chiffrée de la clé quitte le HSM local.
* Une clé KEK générée à l’intérieur d’un HSM de Key Vault n’est pas exportable. Les HSM appliquent la règle en vertu de laquelle aucune version claire d’une clé KEK n’existe en dehors d’un HSM de Key Vault.
* La clé KEK doit se trouver dans le coffre de clés dans lequel la clé cible doit être importée.
* Lors du chargement du fichier BYOK sur le coffre de clés, un HSM de Key Vault utilise la clé KEK privée pour déchiffrer le matériel de la clé cible et l’importer comme clé HSM. Cette opération se produit entièrement à l’intérieur d’un HSM de Key Vault. La clé cible reste toujours dans la limite de protection du HSM.

## <a name="prerequisites"></a>Prérequis

Le tableau suivant répertorie les conditions préalables à l’utilisation de BYOK dans Azure Key Vault :

| Condition requise | Informations complémentaires |
| --- | --- |
| Abonnement Azure |Pour créer un coffre de clés dans Azure Key Vault, vous avez besoin d’un abonnement Azure. [Inscrivez-vous pour un essai gratuit](https://azure.microsoft.com/pricing/free-trial/). |
| SKU Premium Key Vault pour l’importation de clés protégées par HSM |Pour plus d’informations sur les niveaux de service et les capacités d’Azure Key Vault, voir [Tarification d’Azure Key Vault](https://azure.microsoft.com/pricing/details/key-vault/). |
| HSM figurant dans la liste des HSM pris en charge, avec un outil BYOK et des instructions fournies par votre fournisseur de HSM | Vous devez disposer d’autorisations pour un HSM et d’une connaissance de base de l’utilisation de votre HSM. Consultez [Modules HSM pris en charge](#supported-hsms). |
| Azure CLI version 2.1.0 ou ultérieure | Voir [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).|

## <a name="supported-hsms"></a>Modules HSM pris en charge

|Nom du fournisseur|Type de fournisseur|Modèles HSM pris en charge|Informations complémentaires|
|---|---|---|---|
|nCipher|Fabricant,<br/>HSM en tant que service|<ul><li>Famille nShield de modules HSM</li><li>nShield en tant que service</ul>|[Outil et documentation BYOK nCipher (nouvelle méthode)](https://www.ncipher.com/products/key-management/cloud-microsoft-azure)|
|Thales|Fabricant|<ul><li>Famille Luna HSM 7 avec microprogramme version 7.3 ou ultérieure</li></ul>| [Outil BYOK Luna et documentation](https://supportportal.thalesgroup.com/csm?id=kb_article_view&sys_kb_id=3892db6ddb8fc45005c9143b0b961987&sysparm_article=KB0021016)|
|Fortanix|Fabricant,<br/>HSM en tant que service|<ul><li>SDKMS (Self-Defending Key Management Service)</li><li>Equinix SmartKey</li></ul>|[Exportation de clés SDKMS vers des fournisseurs cloud pour BYOK - Azure Key Vault](https://support.fortanix.com/hc/en-us/articles/360040071192-Exporting-SDKMS-keys-to-Cloud-Providers-for-BYOK-Azure-Key-Vault)|
|Marvell|Fabricant|Tous les modules HSM LiquidSecurity avec<ul><li>Version 2.0.4 ou ultérieure du microprogramme</li><li>Version 3.2 ou ultérieure du microprogramme</li></ul>|[Outil et documentation BYOK Marvell](https://www.marvell.com/products/security-solutions/nitrox-hs-adapters/exporting-marvell-hsm-keys-to-cloud-azure-key-vault.html)|
|Cryptomathic|ISV (système de gestion de clés sécurisé)|Plusieurs marques et modèles de modules HSM, dont<ul><li>nCipher</li><li>Thales</li><li>Utimaco</li></ul>Pour plus d’informations, consultez le [site de Cryptomathic](https://www.cryptomathic.com/azurebyok)|[Outil et documentation BYOK Cryptomathic](https://www.cryptomathic.com/azurebyok)|
|Securosys SA|Fabricant, HSM en tant que service|Famille HSM de Primus, HSM Clouds de Securosys|[Outil et documentation BYOK Primus](https://www.securosys.com/primus-azure-byok)|
|StorMagic|ISV (système de gestion de clés sécurisé)|Plusieurs marques et modèles de modules HSM, dont<ul><li>Utimaco</li><li>Thales</li><li>nCipher</li></ul>Pour plus d’informations, consultez le [site StorMagic](https://stormagic.com/doc/svkms/Content/Integrations/Azure_KeyVault_BYOK.htm)|[SvKMS et BYOK d’Azure Key Vault](https://stormagic.com/doc/svkms/Content/Integrations/Azure_KeyVault_BYOK.htm)|
||||


## <a name="supported-key-types"></a>Types de clés pris en charge

|Nom de clé|Type de clé|Taille de la clé|Origine|Description|
|---|---|---|---|---|
|Key Exchange Key (KEK)|RSA| 2 048 bits<br />3 072 bits<br />4 096 bits|HSM Azure Key Vault|Paire de clés RSA sauvegardée par HSM générée dans Azure Key Vault|
|Clé cible|RSA|2 048 bits<br />3 072 bits<br />4 096 bits|HSM du fournisseur|Clé à transférer au HSM d’Azure Key Vault|

## <a name="generate-and-transfer-your-key-to-the-key-vault-hsm"></a>Générer et transférer votre clé vers le HSM du Key Vault

Pour générer et transférer votre clé vers un HDM de Key Vault :

* [Étape 1 : Générer une clé KEK](#step-1-generate-a-kek)
* [Étape 2 : Télécharger la clé publique KEK](#step-2-download-the-kek-public-key)
* [Étape 3 : Générer votre clé et la préparer pour le transfert](#step-3-generate-and-prepare-your-key-for-transfer)
* [Étape 4 : Transférer votre clé vers Azure Key Vault](#step-4-transfer-your-key-to-azure-key-vault)

### <a name="step-1-generate-a-kek"></a>Étape 1 : Générer une clé KEK

Une KEK est une clé RSA générée dans un HSM de Key Vault. La KEK est utilisée pour chiffrer la clé que vous souhaitez importer (clé *cible*).

La clé KEK doit être :
- une clé RSA-HSM (2 048 bits, 3 072 bits ou 4 096 bits)
- Générée dans le coffre de clés dans lequel vous envisagez d’importer la clé cible
- Créée avec les opérations de clé autorisées définies sur `import`

> [!NOTE]
> La clé KEK doit avoir « Importer » comme seule opération de clé autorisée. « Importer » est incompatible avec toutes les autres opérations de clé.

Pour créer une clé KEK avec les opérations de clé définies sur `import`, utilisez la commande [az keyvault key create](/cli/azure/keyvault/key?view=azure-cli-latest#az-keyvault-key-create). Enregistrez l’identificateur de clé (`kid`) qui est retourné par la commande suivante. (Vous allez utiliser la valeur `kid` à l’[étape 3](#step-3-generate-and-prepare-your-key-for-transfer).)

```azurecli
az keyvault key create --kty RSA-HSM --size 4096 --name KEKforBYOK --ops import --vault-name ContosoKeyVaultHSM
```

### <a name="step-2-download-the-kek-public-key"></a>Étape 2 : Télécharger la clé publique KEK

Utilisez la commande [az keyvault key download](/cli/azure/keyvault/key?view=azure-cli-latest#az-keyvault-key-download) pour télécharger la clé publique KEK dans un fichier .pem. La clé cible que vous importez est chiffrée à l’aide de la clé publique KEK.

```azurecli
az keyvault key download --name KEKforBYOK --vault-name ContosoKeyVaultHSM --file KEKforBYOK.publickey.pem
```

Transférez le fichier KEKforBYOK.publickey.pem sur votre ordiateur hors connexion. Vous en aurez besoin à l’étape suivante.

### <a name="step-3-generate-and-prepare-your-key-for-transfer"></a>Étape 3 : Générer votre clé et la préparer pour le transfert

Reportez-vous à la documentation de votre fournisseur de HSM pour télécharger et installer l’outil BYOK. Suivez les instructions de votre fournisseur de HSM pour générer une clé cible, puis créez un package de transfert de clé (fichier BYOK). L’outil BYOK utilise le `kid` de l’[étape 1](#step-1-generate-a-kek) et le fichier KEKforBYOK.publickey.pem que vous avez téléchargé à l’[étape 2](#step-2-download-the-kek-public-key) pour générer une clé cible chiffrée dans un fichier BYOK.

Transférez le fichier BYOK sur votre ordinateur connecté.

> [!NOTE] 
> L’importation de clés RSA 1 024 bits n’est pas prise en charge. Actuellement, l’importation d’une clé Elliptic Curve (EC) n’est pas prise en charge.
> 
> **Problème connu** : L’importation d’une clé cible RSA 4K à partir de modules HSM Luna est uniquement prise en charge avec le microprogramme 7.4.0 ou une version ultérieure.

### <a name="step-4-transfer-your-key-to-azure-key-vault"></a>Étape 4 : Transférer votre clé vers Azure Key Vault

Pour terminer l’importation de la clé, transférez le package de transfert de clé (fichier BYOK) de votre ordinateur déconnecté vers l’ordinateur connecté à Internet. Pour charger le fichier BYOK dans le HSM de Key Vault, utilisez la commande [az keyvault key import](/cli/azure/keyvault/key?view=azure-cli-latest#az-keyvault-key-import).

```azurecli
az keyvault key import --vault-name ContosoKeyVaultHSM --name ContosoFirstHSMkey --byok-file KeyTransferPackage-ContosoFirstHSMkey.byok
```

Si le chargement réussit, Azure CLI affiche les propriétés de la clé importée.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez maintenant utiliser cette clé protégée HSM dans votre coffre de clés. Pour plus d’informations, voir cette [comparaison de prix et de fonctionnalité](https://azure.microsoft.com/pricing/details/key-vault/).



