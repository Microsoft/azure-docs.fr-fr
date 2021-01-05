---
title: Génération et transfert de clés protégées par HSM - Azure Key Vault
description: Découvrez comment planifier, générer, puis transférer vos propres clés protégées par HSM à utiliser avec Azure Key Vault. Cette méthode est également appelée BYOK (Bring Your Own Key, ou apportez votre propre clé).
services: key-vault
author: amitbapat
manager: devtiw
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: keys
ms.topic: tutorial
ms.date: 05/29/2020
ms.author: ambapat
ms.openlocfilehash: 99339914b66dc40f686cc46cacbcea15438c9004
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94413341"
---
# <a name="import-hsm-protected-keys-to-key-vault"></a>Importer des clés protégées par HSM dans un coffre de clés

Pour une meilleure garantie, lorsque vous utilisez le coffre de clés Azure, vous pouvez importer ou générer des clés dans des modules de sécurité matériels (HSM) qui ne franchissent jamais les limites HSM. Ce scénario est souvent appelé *Apportez votre propre clé* ou désigné par l’acronyme BYOK. Azure Key Vault utilise la famille nShield de modules HSM (FIPS 140-2 niveau 2 validé) de nCipher pour protéger vos clés.

Cette fonctionnalité n’est pas disponible pour Azure Chine 21Vianet.

> [!NOTE]
> Pour plus d’informations sur le coffre de clés Azure, consultez la page [Présentation du coffre de clés Azure](../general/overview.md)  
> Pour voir un didacticiel de mise en route incluant un coffre de clés pour les clés protégées par HSM, consultez la section [ Présentation d’Azure Key Vault](../general/overview.md).

## <a name="supported-hsms"></a>Modules HSM pris en charge

Le transfert de clés protégées par HSM vers un coffre de clés est pris en charge par deux méthodes. La méthode à employer dépend des modules HSM que vous utilisez. Utilisez le tableau ci-dessous pour déterminer la méthode à utiliser pour vos modules HSM afin de générer, puis transférer vos propres clés protégées par HSM, que vous utiliserez avec Azure Key Vault. 

|Nom du fournisseur|Type de fournisseur|Modèles HSM pris en charge|Méthode de transfert de clé HSM prise en charge|
|---|---|---|---|
|[nCipher](https://www.ncipher.com/products/key-management/cloud-microsoft-azure)|Fabricant,<br/>HSM en tant que service|<ul><li>Famille nShield de modules HSM</li><li>nShield en tant que service</ul>|**Méthode 1 :** [nCipher BYOK](hsm-protected-keys-ncipher.md) (avec une attestation forte pour l'importation de clés et la validation HSM)<br/>**Méthode 2 :** [Utiliser la nouvelle méthode BYOK](hsm-protected-keys-byok.md) |
|Thales|Fabricant|<ul><li>Famille Luna HSM 7 avec microprogramme version 7.3 ou ultérieure</li></ul>| [Utiliser la nouvelle méthode BYOK](hsm-protected-keys-byok.md)|
|Fortanix|Fabricant,<br/>HSM en tant que service|<ul><li>SDKMS (Self-Defending Key Management Service)</li><li>Equinix SmartKey</li></ul>|[Utiliser la nouvelle méthode BYOK](hsm-protected-keys-byok.md)|
|Marvell|Fabricant|Tous les modules HSM LiquidSecurity avec<ul><li>Version 2.0.4 ou ultérieure du microprogramme</li><li>Version 3.2 ou ultérieure du microprogramme</li></ul>|[Utiliser la nouvelle méthode BYOK](hsm-protected-keys-byok.md)|
|Cryptomathic|ISV (système de gestion de clés sécurisé)|Plusieurs marques et modèles de modules HSM, dont<ul><li>nCipher</li><li>Thales</li><li>Utimaco</li></ul>Pour plus d'informations, consultez le [site de Cryptomathic](https://www.cryptomathic.com/azurebyok)|[Utiliser la nouvelle méthode BYOK](hsm-protected-keys-byok.md)|
|Securosys SA|Fabricant, HSM en tant que service|Famille HSM de Primus, HSM Clouds de Securosys|[Utiliser la nouvelle méthode BYOK](hsm-protected-keys-byok.md)|
|StorMagic|ISV (système de gestion de clés sécurisé)|Plusieurs marques et modèles de modules HSM, dont<ul><li>Utimaco</li><li>Thales</li><li>nCipher</li></ul>Pour plus d’informations, consultez le [site StorMagic](https://stormagic.com/doc/svkms/Content/Integrations/Azure_KeyVault_BYOK.htm)|[Utiliser la nouvelle méthode BYOK](hsm-protected-keys-byok.md)|
|||||

## <a name="next-steps"></a>Étapes suivantes

* Appliquez les [bonnes pratiques relatives à Key Vault](../general/best-practices.md) pour assurer la sécurité, la durabilité et la supervision de vos clés.
* Pour une description complète de la nouvelle méthode BYOK, reportez-vous à la [Spécification BYOK](./byok-specification.md).