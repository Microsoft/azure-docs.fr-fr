---
title: Génération et transfert de clés protégées par HSM pour les HSM managés Azure Key Vault - Azure Key Vault | Microsoft Docs
description: Utilisez les informations de cet article pour planifier, générer, puis transférer vos propres clés protégées par HSM à utiliser avec les HSM managés. Cette méthode est également appelée BYOK (Bring Your Own Key, ou apportez votre propre clé).
services: key-vault
author: amitbapat
tags: azure-resource-manager
ms.service: key-vault
ms.topic: conceptual
ms.date: 09/17/2020
ms.author: ambapat
ms.openlocfilehash: 3c727b75e0d1b1e05638617d6e460dade15fc3c5
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94413477"
---
# <a name="import-hsm-protected-keys-to-managed-hsm-byok"></a>Importer des clés protégées par HSM dans un HSM managé (BYOK)

 Le HSM géré Azure Key Vault prend en charge l’importation des clés générées dans votre module de sécurité matérielle (HSM) local. Les clés ne sortent jamais des limites de protection du HSM. Il est souvent fait référence à ce scénario sous le terme *BYOK* (Bring Your Own Key, ou apportez votre propre clé). Un HSM managé utilise les adaptateurs HSM Marvell LiquidSecurity (FIPS 140-2 niveau 3 validé) pour protéger vos clés.

Les informations de cet article vous aident à planifier, à générer puis à transférer vos propres clés protégées par HSM à utiliser avec un HSM managé.

> [!NOTE]
> Cette fonctionnalité n’est pas disponible pour Azure Chine 21Vianet. Cette méthode d’importation n’est disponible que pour les [HSM pris en charge](#supported-hsms). 

Pour plus d’informations et pour obtenir un didacticiel sur la prise en main de HSM géré, consultez [Qu’est-ce que HSM managé ?](overview.md).

## <a name="overview"></a>Vue d’ensemble

Voici une vue d’ensemble du processus. Les étapes spécifiques à effectuer sont décrites plus loin dans cet article.

* Dans un HSM managé, générez une clé (*Key Exchange Key*, KEK). Cette KEK doit être une clé RSA-HSM qui n’a que l’opération de clé `import`. 
* Téléchargez la clé publique KEK en tant que fichier. pem.
* Transférez la clé publique KEK sur un ordinateur hors connexion connecté à un HSM local.
* Sur l’ordinateur hors connexion, utilisez l’outil BYOK fourni par votre fournisseur de HSM pour créer un fichier BYOK. 
* La clé cible est chiffrée à l’aide d’une clé KEK qui reste chiffrée jusqu’à ce qu’elle soit transférée vers le HSM managé. Seule la version chiffrée de la clé quitte le HSM local.
* Une clé KEK générée à l’intérieur d’un HSM managé n’est pas exportable. Les HSM appliquent la règle en vertu de laquelle aucune version claire d’une clé KEK n’existe en dehors d’un HSM managé.
* La clé KEK doit se trouver dans le HSM managé dans lequel la clé cible doit être importée.
* Lors du chargement du fichier BYOK sur le HSM managé, un HSM managé utilise la clé KEK privée pour déchiffrer le matériel de la clé cible et l’importer comme clé HSM. Cette opération se produit entièrement à l’intérieur du HSM. La clé cible reste toujours dans la limite de protection du HSM.


## <a name="prerequisites"></a>Prérequis

Pour utiliser les commandes Azure CLI de cet article, vous devez disposer des éléments suivants :

* Un abonnement à Microsoft Azure. Si vous n’en avez pas, vous pouvez vous inscrire pour bénéficier d’un [essai gratuit](https://azure.microsoft.com/pricing/free-trial).
* Azure CLI 2.12.0 ou une version ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI]( /cli/azure/install-azure-cli).
* Un HSM géré de la [liste des modules HSM pris en charge](#supported-hsms) dans votre abonnement. Consultez [Démarrage rapide : Approvisionnez et activez un HSM géré à l’aide d’Azure CLI](quick-create-cli.md) pour approvisionner et activer un HSM géré.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Pour vous connecter à Azure à l’aide de l’interface CLI, vous pouvez taper la commande suivante :

```azurecli
az login
```

Pour plus d’informations sur les options de connexion par le biais de l’interface CLI, consultez [Se connecter avec Azure CLI](/cli/azure/authenticate-azure-cli?view=azure-cli-latest&preserve-view=true)

## <a name="supported-hsms"></a>Modules HSM pris en charge

|Nom du fournisseur|Type de fournisseur|Modèles HSM pris en charge|Informations complémentaires|
|---|---|---|---|
|nCipher|Fabricant,<br/>HSM en tant que service|<ul><li>Famille nShield de modules HSM</li><li>nShield en tant que service</ul>|[Outil et documentation BYOK nCipher (nouvelle méthode)](https://www.ncipher.com/products/key-management/cloud-microsoft-azure)|
|Thales|Fabricant|<ul><li>Famille Luna HSM 7 avec microprogramme version 7.3 ou ultérieure</li></ul>| [Outil BYOK Luna et documentation](https://supportportal.thalesgroup.com/csm?id=kb_article_view&sys_kb_id=3892db6ddb8fc45005c9143b0b961987&sysparm_article=KB0021016)|
|Fortanix|Fabricant,<br/>HSM en tant que service|<ul><li>SDKMS (Self-Defending Key Management Service)</li><li>Equinix SmartKey</li></ul>|[Exportation de clés SDKMS vers des fournisseurs cloud pour BYOK - Azure Key Vault](https://support.fortanix.com/hc/en-us/articles/360040071192-Exporting-SDKMS-keys-to-Cloud-Providers-for-BYOK-Azure-Key-Vault)|
|Marvell|Fabricant|Tous les modules HSM LiquidSecurity avec<ul><li>Version 2.0.4 ou ultérieure du microprogramme</li><li>Version 3.2 ou ultérieure du microprogramme</li></ul>|[Outil et documentation BYOK Marvell](https://www.marvell.com/products/security-solutions/nitrox-hs-adapters/exporting-marvell-hsm-keys-to-cloud-azure-key-vault.html)|
|Cryptomathic|ISV (système de gestion de clés sécurisé)|Plusieurs marques et modèles de modules HSM, dont<ul><li>nCipher</li><li>Thales</li><li>Utimaco</li></ul>Pour plus d’informations, consultez le [site de Cryptomathic](https://www.cryptomathic.com/azurebyok)|[Outil et documentation BYOK Cryptomathic](https://www.cryptomathic.com/azurebyok)|
|Securosys SA|Fabricant, HSM en tant que service|Famille HSM de Primus, HSM Clouds de Securosys|[Outil et documentation BYOK Primus](https://www.securosys.com/primus-azure-byok)|
|StorMagic|ISV (système de gestion de clés sécurisé)|Plusieurs marques et modèles de modules HSM, dont<ul><li>Utimaco</li><li>Thales</li><li>nCipher</li></ul>Pour plus d'informations, consultez le [site StorMagic](https://stormagic.com/doc/svkms/Content/Integrations/Azure_KeyVault_BYOK.htm)|[SvKMS et BYOK d’Azure Key Vault](https://stormagic.com/doc/svkms/Content/Integrations/Azure_KeyVault_BYOK.htm)|
||||


## <a name="supported-key-types"></a>Types de clés pris en charge

|Nom de clé|Type de clé|Taille de la clé|Origine|Description|
|---|---|---|---|---|
|Key Exchange Key (KEK)|RSA| 2 048 bits<br />3 072 bits<br />4 096 bits|HSM managé|Paire de clés RSA-HSM générée dans un HSM managé|
|Clé cible|RSA|2 048 bits<br />3 072 bits<br />4 096 bits|HSM du fournisseur|La clé à transférer au HSM géré|

## <a name="generate-and-transfer-your-key-to-the-managed-hsm"></a>Générer et transférer votre clé vers le HSM managé

Pour générer et transférer votre clé vers un HSM managé :

* [Étape 1 : Générer une clé KEK](#step-1-generate-a-kek)
* [Étape 2 : Télécharger la clé publique KEK](#step-2-download-the-kek-public-key)
* [Étape 3 : Générer votre clé et la préparer pour le transfert](#step-3-generate-and-prepare-your-key-for-transfer)
* [Étape 4 : Transférer votre clé vers le HSM géré](#step-4-transfer-your-key-to-managed-hsm)

### <a name="step-1-generate-a-kek"></a>Étape 1 : Générer une clé KEK

Une KEK est une clé RSA générée dans un HSM managé. La KEK est utilisée pour chiffrer la clé que vous souhaitez importer (clé *cible*).

La clé KEK doit être :
- une clé RSA-HSM (2 048 bits, 3 072 bits ou 4 096 bits)
- Générée dans le HSM managé dans lequel vous envisagez d’importer la clé cible
- Créée avec les opérations de clé autorisées définies sur `import`

> [!NOTE]
> La clé KEK doit avoir « Importer » comme seule opération de clé autorisée. « Importer » est incompatible avec toutes les autres opérations de clé.

Pour créer une clé KEK avec les opérations de clé définies sur `import`, utilisez la commande [az keyvault key create](/cli/azure/keyvault/key?view=azure-cli-latest&preserve-view=true#az-keyvault-key-create). Enregistrez l’identificateur de clé (`kid`) qui est retourné par la commande suivante. (Vous allez utiliser la valeur `kid` à l’[étape 3](#step-3-generate-and-prepare-your-key-for-transfer).)

```azurecli-interactive
az keyvault key create --kty RSA-HSM --size 4096 --name KEKforBYOK --ops import --hsm-name ContosoKeyVaultHSM
```
---


### <a name="step-2-download-the-kek-public-key"></a>Étape 2 : Télécharger la clé publique KEK

Utilisez la commande [az keyvault key download](/cli/azure/keyvault/key?view=azure-cli-latest&preserve-view=true#az-keyvault-key-download) pour télécharger la clé publique KEK dans un fichier .pem. La clé cible que vous importez est chiffrée à l’aide de la clé publique KEK.

```azurecli-interactive
az keyvault key download --name KEKforBYOK --hsm-name ContosoKeyVaultHSM --file KEKforBYOK.publickey.pem
```
---

Transférez le fichier KEKforBYOK.publickey.pem sur votre ordiateur hors connexion. Vous en aurez besoin à l’étape suivante.

### <a name="step-3-generate-and-prepare-your-key-for-transfer"></a>Étape 3 : Générer votre clé et la préparer pour le transfert

Reportez-vous à la documentation de votre fournisseur de HSM pour télécharger et installer l’outil BYOK. Suivez les instructions de votre fournisseur de HSM pour générer une clé cible, puis créez un package de transfert de clé (fichier BYOK). L’outil BYOK utilise le `kid` de l’[étape 1](#step-1-generate-a-kek) et le fichier KEKforBYOK.publickey.pem que vous avez téléchargé à l’[étape 2](#step-2-download-the-kek-public-key) pour générer une clé cible chiffrée dans un fichier BYOK.

Transférez le fichier BYOK sur votre ordinateur connecté.

> [!NOTE] 
> L’importation de clés RSA 1 024 bits n’est pas prise en charge. Actuellement, l’importation d’une clé Elliptic Curve (EC) n’est pas prise en charge.
>
> **Problème connu** : L’importation d’une clé cible RSA 4K à partir de modules HSM Luna est uniquement prise en charge avec le microprogramme 7.4.0 ou une version ultérieure.

### <a name="step-4-transfer-your-key-to-managed-hsm"></a>Étape 4 : Transférer votre clé vers le HSM géré

Pour terminer l’importation de la clé, transférez le package de transfert de clé (fichier BYOK) de votre ordinateur déconnecté vers l’ordinateur connecté à Internet. Pour charger le fichier BYOK dans le HSM managé, utilisez la commande [az keyvault key import](/cli/azure/keyvault/key?view=azure-cli-latest&preserve-view=true#az-keyvault-key-import).

```azurecli-interactive
az keyvault key import --hsm-name ContosoKeyVaultHSM --name ContosoFirstHSMkey --byok-file KeyTransferPackage-ContosoFirstHSMkey.byok
```

Si le chargement réussit, Azure CLI affiche les propriétés de la clé importée.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez maintenant utiliser cette clé protégée HSM dans votre HSM managé. Pour plus d’informations, voir cette [comparaison de prix et de fonctionnalité](https://azure.microsoft.com/pricing/details/key-vault/).



