---
title: Azure Disk Encryption sur un réseau isolé
description: Cet article présente des conseils de dépannage pour Microsoft Azure Disk Encryption sur des machines virtuelles Linux.
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 02/27/2020
ms.custom: seodec18
ms.openlocfilehash: 1d7e019e7759e22e945bddee477a4cb77f17350b
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92913821"
---
# <a name="azure-disk-encryption-on-an-isolated-network"></a>Azure Disk Encryption sur un réseau isolé

Lorsque la connectivité est limitée par un pare-feu, une exigence de proxy ou des paramètres de groupe de sécurité réseau (NSG), cela peut interrompre la capacité de l’extension à effectuer les tâches nécessaires. Cette interruption peut entraîner les messages d’état de type « État de l’extension non disponible sur la machine virtuelle ».

## <a name="package-management"></a>Gestion des packages

Azure Disk Encryption dépend de plusieurs composants, qui sont généralement installés dans le cadre de l’activation ADE, s’ils ne sont pas déjà présents. Derrière un pare-feu ou isolés d’Internet, ces packages doivent être préinstallés ou disponibles localement.

Voici les packages nécessaires pour chaque distribution. Pour obtenir la liste complète des distributions et des types de volumes pris en charge, consultez [Machines virtuelles et systèmes d’exploitation pris en charge](disk-encryption-overview.md#supported-vms-and-operating-systems).

- **Ubuntu 14.04, 16.04, 18.04** : lsscsi, psmisc, at, cryptsetup-bin, python-parted, python-six, procps, grub-pc-bin
- **CentOS 7.2-7.7**  : lsscsi, psmisc, lvm2, UUID, at, patch, cryptsetup, cryptsetup-rechiffrer, pyparted, procps-ng, util-linux
- **CentOS 6.8**  : lsscsi, psmisc, lvm2, UUID, à, cryptsetup-rechiffrer, pyparted, Python-six
- **RedHat 7.2 - 7.7**  : lsscsi, psmisc, lvm2, uuid, at, patch, cryptsetup, cryptsetup-reencrypt, procps-ng, util-linux
- **RedHat 6.8**  : lsscsi, psmisc, lvm2, uuid, at, patch, cryptsetup-reencrypt
- **openSUSE 42,3, SLES 12-SP4, 12-SP3**  : lsscsi, cryptsetup

Sous Red Hat, lorsqu’un proxy est requis, vous devez absolument vous assurer que le gestionnaire d’abonnements et yum sont configurés correctement. Pour plus d’informations, consultez [Guide pratique pour résoudre les problèmes du gestionnaire d’abonnement et yum](https://access.redhat.com/solutions/189533).  

Lorsque les packages sont installés manuellement, ils doivent également être mis à niveau manuellement au fur et à mesure de la publication de nouvelles versions.

## <a name="network-security-groups"></a>Groupes de sécurité réseau
Les paramètres de groupe de sécurité réseau appliqués doivent permettre au point de terminaison de remplir les conditions requises de configuration réseau indiquées pour le chiffrement de disque.  Voir [Azure Disk Encryption : Configuration réseau requise](disk-encryption-overview.md#networking-requirements)

## <a name="azure-disk-encryption-with-azure-ad-previous-version"></a>Azure Disk Encryption avec Azure AD (version précédente)

Si vous utilisez [Azure Disk Encryption avec Azure AD (version précédente)](disk-encryption-overview-aad.md), vous devez installer manuellement la bibliothèque [Azure Active Directory](../../active-directory/azuread-dev/active-directory-authentication-libraries.md) pour toutes les distributions (en plus des packages appropriés à la distribution, comme [ci-dessus](#package-management)).

Lorsque le chiffrement est activé avec des [informations d’identification Azure AD](disk-encryption-linux-aad.md), la machine virtuelle cible doit autoriser la connectivité aux points de terminaison Azure Active Directory et à ceux de Key Vault. Les points de terminaison d'authentification Azure Active Directory actuels sont traités dans les sections 56 et 59 de la documentation [URL et plages d'adresses IP Microsoft 365](/microsoft-365/enterprise/urls-and-ip-address-ranges). Des instructions relatives à Key Vault sont fournies dans la documentation sur l’[accès à Azure Key Vault derrière un pare-feu](../../key-vault/general/access-behind-firewall.md).

### <a name="azure-instance-metadata-service"></a>Service de métadonnées d’instance Azure 

La machine virtuelle doit pouvoir accéder au point de terminaison [Azure Instance Metadata Service](instance-metadata-service.md) qui utilise une adresse IP non routable bien connue (`169.254.169.254`), accessible uniquement à partir de la machine virtuelle.  Les configurations de proxy qui modifient le trafic HTTP local à cette adresse (par exemple, en ajoutant un en-tête X-Forwarded-For) ne sont pas prises en charge.

## <a name="next-steps"></a>Étapes suivantes

- Voir davantage d'étapes sur la [résolution des problèmes liés à Azure Disk Encryption](disk-encryption-troubleshooting.md)
- [Chiffrement des données au repos Azure](../../security/fundamentals/encryption-atrest.md)
