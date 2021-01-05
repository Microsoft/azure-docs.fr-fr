---
title: Prérequis d’application Azure Disk Encryption avec Azure AD (version précédente)
description: Cet article apporte un complément à l’article Azure Disk Encryption pour les machines virtuelles Linux, avec des exigences et des prérequis supplémentaires pour Azure Disk Encryption avec Azure AD.
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 03/15/2019
ms.custom: seodec18
ms.openlocfilehash: b258d499c78aa5fb734cbee01fb753c292bf2678
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90970892"
---
# <a name="azure-disk-encryption-with-azure-active-directory-ad-previous-release"></a>Azure Disk Encryption avec Azure Active Directory (AD) (version précédente)

La nouvelle version d’Azure Disk Encryption évite d’avoir à indiquer un paramètre d’application Azure Active Directory (Azure AD) pour activer le chiffrement de disque de machine virtuelle. Avec la nouvelle version, vous n'êtes plus obligé de fournir les informations d'identification Azure AD lors de l'étape d’activation du chiffrement. À l’aide de la nouvelle version, toutes les nouvelles machines virtuelles doivent être chiffrées sans les paramètres d’application Azure AD. Pour obtenir des instructions sur l’activation du chiffrement de disque de machine virtuelle à l’aide de la nouvelle version, consultez [Azure Disk Encryption pour les machines virtuelles Linux](disk-encryption-overview.md). Les machines virtuelles déjà chiffrées avec les paramètres d’application Azure AD sont toujours prises en charge et doivent continuer à être gérées avec la syntaxe AAD.

Cet article apporte un complément à l’article [Azure Disk Encryption pour les machines virtuelles Linux](disk-encryption-overview.md), avec des exigences et des prérequis supplémentaires pour Azure Disk Encryption avec Azure AD (version précédente).

Les informations contenues dans ces sections restent les mêmes :

- [Machines virtuelles et systèmes d’exploitation pris en charge](disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Configuration requise supplémentaire pour les machines virtuelles](disk-encryption-overview.md#additional-vm-requirements)


## <a name="networking-and-group-policy"></a>Réseau et stratégie de groupe

Pour activer la fonctionnalité Azure Disk Encryption en utilisant l’ancienne syntaxe des paramètres AAD, les machines virtuelles IaaS (d’infrastructure en tant que service) doivent répondre aux exigences de configuration de point de terminaison réseau suivantes : 
  - Pour obtenir un jeton afin de se connecter à votre coffre de clés, la machine virtuelle IaaS doit être en mesure de se connecter à un point de terminaison Azure AD, \[login.microsoftonline.com\].
  - Pour écrire les clés de chiffrement dans votre coffre de clés, la machine virtuelle IaaS doit être en mesure de se connecter au point de terminaison Key Vault.
  - La machine virtuelle IaaS doit être en mesure de se connecter au point de terminaison de stockage Azure qui héberge le référentiel d’extensions Azure et au compte de stockage Azure qui héberge les fichiers de disque dur virtuel.
  -  Si votre stratégie de sécurité limite l’accès à Internet à partir des machines virtuelles Azure, vous pouvez résoudre l’URI ci-dessus et configurer une règle spécifique pour autoriser les connexions sortantes vers les adresses IP. Pour plus d’informations, consultez l’article [Azure Key Vault derrière un pare-feu](../../key-vault/general/access-behind-firewall.md).
  - Sur Windows, si le protocole TLS 1.0 est explicitement désactivé et que la version .NET n’est pas mise à jour vers la version 4.6 ou une version ultérieure, la modification suivante du Registre active Azure Disk Encryption pour sélectionner la version la plus récente du protocole TLS :

  ```config-registry
  [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319]
  "SystemDefaultTlsVersions"=dword:00000001
  "SchUseStrongCrypto"=dword:00000001
    
  [HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319]
  "SystemDefaultTlsVersions"=dword:00000001
  "SchUseStrongCrypto"=dword:00000001` 
  ```

### <a name="group-policy"></a>Stratégie de groupe
 - La solution Azure Disk Encryption utilise le protecteur de clé externe BitLocker pour les machines virtuelles IaaS Windows. Pour les machines virtuelles jointes à un domaine, n’envoyez (push) pas de stratégies de groupe qui appliquent des protecteurs de module de plateforme sécurisée (TPM). Pour en savoir plus sur la stratégie de groupe pour l’option **Autoriser BitLocker sans un module de plateforme sécurisée compatible**, consultez la [Référence de stratégie de groupe BitLocker](/windows/security/information-protection/bitlocker/bitlocker-group-policy-settings#bkmk-unlockpol1).

- La stratégie BitLocker sur les machines virtuelles jointes à un domaine avec une stratégie de groupe personnalisée doit inclure le paramètre suivant : [Configurer le stockage par les utilisateurs des informations de récupération BitLocker -> Autoriser une clé de récupération de 256 bits](/windows/security/information-protection/bitlocker/bitlocker-group-policy-settings). Azure Disk Encryption échoue quand les paramètres de stratégie de groupe personnalisés pour BitLocker sont incompatibles. Sur les machines dont le paramètre de stratégie est incorrect, appliquez la nouvelle stratégie, forcez la mise à jour de cette dernière (gpupdate.exe /force), puis redémarrez si nécessaire. 

## <a name="encryption-key-storage-requirements"></a>Exigences liées au stockage des clés de chiffrement 

Azure Disk Encryption requiert Azure Key Vault pour contrôler et gérer les secrets et les clés de chiffrement de disque. Votre coffre de clés et vos machines virtuelles doivent se trouver dans la même région et le même abonnement Azure.

Pour plus d’informations, consultez [Création et configuration d’un coffre de clés pour Azure Disk Encryption avec Azure AD (version précédente)](disk-encryption-key-vault-aad.md).
 
## <a name="next-steps"></a>Étapes suivantes

- [Création et configuration d’un coffre de clés pour Azure Disk Encryption avec Azure AD (version précédente)](disk-encryption-key-vault-aad.md)
- [Activer Azure Disk Encryption avec Azure AD sur des machines virtuelles Linux (version précédente)](disk-encryption-linux-aad.md)
- [Script d’interface CLI des prérequis Azure Disk Encryption](https://github.com/ejarvi/ade-cli-getting-started)
- [Script PowerShell des prérequis Azure Disk Encryption](https://github.com/Azure/azure-powershell/tree/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts)
