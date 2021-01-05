---
title: Azure Disk Encryption pour les machines virtuelles et les groupes de machines virtuelles identiques
description: En savoir plus sur Azure Disk Encryption pour les machines virtuelles et les groupes de machines virtuelles identiques. Azure Disk Encryption fonctionne pour les machines virtuelles Linux et Windows.
author: msmbaldwin
ms.service: security
ms.topic: article
ms.author: mbaldwin
ms.date: 10/15/2019
ms.custom: seodec18
ms.openlocfilehash: 21194bf2fe76a7eb0ee034d4a502c20ee3032dd9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87543671"
---
# <a name="azure-disk-encryption-for-virtual-machines-and-virtual-machine-scale-sets"></a>Azure Disk Encryption pour les machines virtuelles et les groupes de machines virtuelles identiques

Azure Disk Encryption peut être appliqué à des machines virtuelles Linux et Windows, ainsi qu’à des groupes de machines virtuelles identiques. 

## <a name="linux-virtual-machines"></a>Machines virtuelles Linux

Les articles suivants fournissent des conseils pour le chiffrement de machines virtuelles Linux.

### <a name="current-version-of-azure-disk-encryption"></a>Version actuelle d’Azure Disk Encryption

- [Vue d’ensemble d’Azure Disk Encryption pour machines virtuelles Linux](../../virtual-machines/linux/disk-encryption-overview.md)
- [Scénarios Azure Disk Encryption sur les machines virtuelles Linux](../../virtual-machines/linux/disk-encryption-linux.md)
- [Créer et chiffrer une machine virtuelle Linux avec Azure CLI](../../virtual-machines/linux/disk-encryption-cli-quickstart.md)
- [Créer et chiffrer une machine virtuelle Linux avec Azure PowerShell](../../virtual-machines/linux/disk-encryption-powershell-quickstart.md)
- [Créer et chiffrer une machine virtuelle Linux via le portail Azure](../../virtual-machines/linux/disk-encryption-portal-quickstart.md)
- [Schéma de l’extension Azure Disk Encryption pour Linux](../../virtual-machines/extensions/azure-disk-enc-linux.md)
- [Création et configuration d’un coffre de clés pour Azure Disk Encryption](../../virtual-machines/linux/disk-encryption-key-vault.md)
- [Exemples de scripts Azure Disk Encryption](../../virtual-machines/linux/disk-encryption-sample-scripts.md)
- [Résolution des problèmes liés à Azure Disk Encryption](../../virtual-machines/linux/disk-encryption-troubleshooting.md)
- [Forum aux questions sur Azure Disk Encryption](../../virtual-machines/linux/disk-encryption-faq.md)

### <a name="azure-disk-encryption-with-azure-ad-previous-version"></a>Azure Disk Encryption avec Azure AD (version précédente)

- [Vue d’ensemble d’Azure Disk Encryption avec Azure AD pour machines virtuelles Linux](../../virtual-machines/linux/disk-encryption-overview-aad.md)
- [Scénarios Azure Disk Encryption avec Azure AD sur machines virtuelles Linux](../../virtual-machines/linux/disk-encryption-linux.md)
- [Création et configuration d’un coffre de clés pour Azure Disk Encryption avec Azure AD (version précédente)](../../virtual-machines/linux/disk-encryption-key-vault-aad.md)

## <a name="windows-virtual-machines"></a>Machines virtuelles Windows

Les articles suivants fournissent des conseils pour le chiffrement de machines virtuelles Windows.

### <a name="current-version-of-azure-disk-encryption"></a>Version actuelle d’Azure Disk Encryption

- [Vue d’ensemble d’Azure Disk Encryption pour machines virtuelles Windows](../../virtual-machines/windows/disk-encryption-overview.md)
- [Scénarios Azure Disk Encryption sur les machines virtuelles Windows](../../virtual-machines/windows/disk-encryption-windows.md)
- [Créer et chiffrer une machine virtuelle Windows avec Azure CLI](../../virtual-machines/windows/disk-encryption-cli-quickstart.md)
- [Créer et chiffrer une machine virtuelle Windows avec Azure PowerShell](../../virtual-machines/windows/disk-encryption-powershell-quickstart.md)
- [Créer et chiffrer une machine virtuelle Windows via le portail Azure](../../virtual-machines/windows/disk-encryption-portal-quickstart.md)
- [Schéma de l’extension Azure Disk Encryption pour Windows](../../virtual-machines/extensions/azure-disk-enc-windows.md)
- [Création et configuration d’un coffre de clés pour Azure Disk Encryption](../../virtual-machines/windows/disk-encryption-key-vault.md)
- [Exemples de scripts Azure Disk Encryption](../../virtual-machines/windows/disk-encryption-sample-scripts.md)
- [Résolution des problèmes liés à Azure Disk Encryption](../../virtual-machines/windows/disk-encryption-troubleshooting.md)
- [Forum aux questions sur Azure Disk Encryption](../../virtual-machines/windows/disk-encryption-faq.md)

### <a name="azure-disk-encryption-with-azure-ad-previous-version"></a>Azure Disk Encryption avec Azure AD (version précédente)

- [Vue d’ensemble des Azure Disk Encryption avec Azure AD pour machines virtuelles Windows](../../virtual-machines/windows/disk-encryption-overview-aad.md)
- [Scénarios Azure Disk Encryption avec Azure AD sur machines virtuelles Windows](../../virtual-machines/windows/disk-encryption-windows.md)
- [Création et configuration d’un coffre de clés pour Azure Disk Encryption avec Azure AD (version précédente)](../../virtual-machines/windows/disk-encryption-key-vault-aad.md)

## <a name="virtual-machine-scale-sets"></a>Groupes identiques de machines virtuelles

Les articles suivants fournissent des conseils pour le chiffrement de groupes de machines virtuelles identiques.

- [Vue d’ensemble d’Azure Disk Encryption pour groupes de machines virtuelles identiques](../../virtual-machine-scale-sets/disk-encryption-overview.md) 
- [Chiffrer un groupe de machines virtuelles identiques à l’aide d’Azure CLI](../../virtual-machine-scale-sets/disk-encryption-cli.md) 
- [Chiffrer un groupe de machines virtuelles identiques à l’aide d’Azure PowerShell](../../virtual-machine-scale-sets/disk-encryption-powershell.md).
- [Chiffrer un groupe de machines virtuelles identiques à l’aide d’Azure Resource Manager](../../virtual-machine-scale-sets/disk-encryption-azure-resource-manager.md)
- [Créer et configurer un coffre de clés pour Azure Disk Encryption](../../virtual-machine-scale-sets/disk-encryption-key-vault.md)
- [Utiliser Azure Disk Encryption avec le séquencement d’extensions de groupes de machines virtuelles identiques](../../virtual-machine-scale-sets/disk-encryption-extension-sequencing.md)

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble du chiffrement Azure](encryption-overview.md)
- [Chiffrement des données au repos](encryption-atrest.md)
- [Meilleures pratiques en matière de chiffrement et de sécurité des données](data-encryption-best-practices.md)
