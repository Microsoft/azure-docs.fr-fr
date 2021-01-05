---
title: FAQ - Azure Disk Encryption pour les machines virtuelles Windows
description: Cet article offre des réponses aux questions fréquentes sur Microsoft Azure Disk Encryption pour les machines virtuelles IaaS Windows.
author: msmbaldwin
ms.service: virtual-machines-windows
ms.subservice: security
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 11/01/2019
ms.custom: seodec18
ms.openlocfilehash: 3f02c90dd6c751909434f3f2af9b8eb81ee1c4b9
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96488372"
---
# <a name="azure-disk-encryption-for-windows-virtual-machines-faq"></a>FAQ sur Azure Disk Encryption pour machines virtuelles Windows

Cet article offre des réponses aux questions fréquentes (FAQ) sur Azure Disk Encryption pour les machines virtuelles Windows. Pour plus d’informations sur ce service, consultez [Vue d’ensemble d’Azure Disk Encryption](disk-encryption-overview.md).

## <a name="what-is-azure-disk-encryption-for-windows-vms"></a>Qu’est-ce qu’Azure Disk Encryption pour machines virtuelles Windows ?

Le service Azure Disk Encryption pour machines virtuelles Windows utilise la fonctionnalité BitLocker de Windows pour effectuer un chiffrement complet du disque du système d’exploitation et des disques de données. Il assure en outre le chiffrement du disque temporaire lorsque le paramètre [VolumeType est défini sur Tous](disk-encryption-windows.md#enable-encryption-on-a-newly-added-data-disk).  Le contenu est chiffré à partir de la machine virtuelle vers le serveur back-end de stockage. Donc, un chiffrement de bout en bout est assuré avec une clé gérée par le client.
 
Consultez [Machines virtuelles et systèmes d’exploitation pris en charge](disk-encryption-overview.md#supported-vms-and-operating-systems).
 
## <a name="where-is-azure-disk-encryption-in-general-availability-ga"></a>Où se trouve Azure Disk Encryption en disponibilité générale (GA) ?

Azure Disk Encryption est en disponibilité générale dans toutes les régions publiques Azure.

## <a name="what-user-experiences-are-available-with-azure-disk-encryption"></a>Quelles expériences utilisateur sont disponibles avec Azure Disk Encryption ?

La disponibilité générale (GA) Azure Disk Encryption prend en charge les modèles Azure Resource Manager, Azure PowerShell et Azure CLI. Les différentes expériences utilisateur vous donnent de la souplesse. Vous disposez de trois options différentes pour activer le chiffrement des disques sur vos machines virtuelles. Pour plus d’informations sur l’expérience utilisateur et pour des instructions étape par étape disponibles dans Azure Disk Encryption, consultez [Scénarios Azure Disk Encryption pour Windows](disk-encryption-windows.md).

## <a name="how-much-does-azure-disk-encryption-cost"></a>Combien coûte Azure Disk Encryption ?

Il n’y a pas de frais associés au chiffrement des disques de machine virtuelle avec Azure Disk Encryption, mais l’utilisation d’Azure Key Vault engendre des frais. Pour plus d’informations sur les coûts d’Azure Key Vault, consultez la page sur la [Tarification de Key Vault](https://azure.microsoft.com/pricing/details/key-vault/).

## <a name="how-can-i-start-using-azure-disk-encryption"></a>Comment puis-je commencer à utiliser Azure Disk Encryption ?

Pour commencer, consultez la [Vue d’ensemble d’Azure Disk Encryption](disk-encryption-overview.md).

## <a name="what-vm-sizes-and-operating-systems-support-azure-disk-encryption"></a>Quels sont les tailles et les systèmes d’exploitation de machine virtuelle qui prennent en charge Azure Disk Encryption ?

L’article [Vue d’ensemble d’Azure Disk Encryption](disk-encryption-overview.md) liste les [tailles de machine virtuelle](disk-encryption-overview.md#supported-vms) et les [systèmes d’exploitation de machine virtuelle](disk-encryption-overview.md#supported-operating-systems) qui prennent en charge Azure Disk Encryption.

## <a name="can-i-encrypt-both-boot-and-data-volumes-with-azure-disk-encryption"></a>Puis-je chiffrer des volumes de démarrage et de données avec Azure Disk Encryption ?

Vous pouvez chiffrer les volumes de démarrage et de données, mais vous ne pouvez pas chiffrer les données sans avoir chiffré au préalable le volume du système d’exploitation.

## <a name="can-i-encrypt-an-unmounted-volume-with-azure-disk-encryption"></a>Puis-je chiffrer un volume démonté avec Azure Disk Encryption ?

Non, Azure Disk Encryption chiffre uniquement les volumes montés.

## <a name="what-is-storage-server-side-encryption"></a>Qu’est-ce que le chiffrement côté serveur de stockage ?

Le chiffrement côté serveur de stockage chiffre les disques managés Azure dans Stockage Azure. Les disques managés sont chiffrés par défaut avec le chiffrement côté serveur à l’aide d’une clé gérée par la plateforme (à compter du 10 juin 2017). Vous pouvez gérer le chiffrement des disques managés avec vos propres clés en spécifiant une clé gérée par le client. Pour plus d’informations, consultez [Chiffrement côté serveur de disques managés Azure](../disk-encryption.md).
 
## <a name="how-is-azure-disk-encryption-different-from-storage-server-side-encryption-with-customer-managed-key-and-when-should-i-use-each-solution"></a>En quoi Azure Disk Encryption diffère-t-il du chiffrement côté serveur de stockage avec clé gérée par le client, et quand dois-je utiliser chaque solution ?

Azure Disk Encryption assure un chiffrement de bout en bout pour le disque du système d’exploitation, les disques de données et le disque temporaire en utilisant une clé gérée par le client.

- Si vos exigences incluent tout ce qui précède de bout en bout, optez pour Azure Disk Encryption. 
- Si vos exigences incluent uniquement le chiffrement des données au repos avec clé gérée par le client, utilisez un [chiffrement côté serveur avec clés gérées par le client](../disk-encryption.md). Vous ne pouvez pas chiffrer un disque en utilisant Azure Disk Encryption et un chiffrement côté serveur de stockage avec clés gérées par le client.
- Si vous utilisez un scénario répertorié dans les [scénarios non pris en charge pour Windows](disk-encryption-windows.md#unsupported-scenarios), envisagez d’utiliser le [chiffrement côté serveur avec clés gérées par le client](../disk-encryption.md). 
- Si la stratégie de votre organisation vous permet de chiffrer du contenu au repos avec une clé gérée par Azure, aucune action n’est nécessaire. Le contenu est chiffré par défaut. Pour les disques managés, le contenu du stockage est chiffré par défaut à l’aide d’un chiffrement côté serveur avec clé gérée par la plateforme. La clé est gérée par le service Stockage Azure. 

## <a name="how-do-i-rotate-secrets-or-encryption-keys"></a>Comment effectuer la rotation des secrets ou des clés de chiffrement ?

Pour effectuer la rotation des secrets, appelez simplement la même commande que celle utilisée initialement pour activer le chiffrement de disque, en spécifiant un autre coffre de clés. Pour effectuer la rotation de la clé de chiffrement principale, appelez la même commande que celle utilisée initialement pour activer le chiffrement de disque, en spécifiant le nouveau chiffrement principal. 

>[!WARNING]
> - Si vous avez déjà utilisé [Azure Disk Encryption avec une application Azure AD](disk-encryption-windows-aad.md) avec des informations d’identification Azure AD pour chiffrer cette machine virtuelle, vous devrez continuer à utiliser cette option. Vous ne pouvez pas utiliser Azure Disk Encryption sur cette machine virtuelle chiffrée car ce scénario n’est pas pris en charge. Autrement dit, l’utilisation d’une application autre qu’AAD pour cette machine virtuelle n’est pas encore prise en charge.

## <a name="how-do-i-add-or-remove-a-key-encryption-key-if-i-didnt-originally-use-one"></a>Comment ajouter ou supprimer une clé de chiffrement principale si je n’en utilisais pas à l’origine ?

Pour ajouter une clé de chiffrement principale, appelez la commande enable à nouveau en y passant le paramètre de clé de chiffrement principale. Pour supprimer une clé de chiffrement principale, appelez la commande enable à nouveau sans le paramètre de clé de chiffrement principale.

## <a name="does-azure-disk-encryption-allow-you-to-bring-your-own-key-byok"></a>Azure Disk Encryption vous permet-il d’apporter votre propre clé « BYOK » ?

Oui, vous pouvez fournir vos propres clés de chiffrement principales. Ces clés sont sauvegardées dans Azure Key Vault, le magasin de clés d’Azure Disk Encryption. Pour plus d’informations sur les scénarios de prise en charge des clés de chiffrement principales, consultez [Création et configuration d’un coffre de clés pour Azure Disk Encryption](disk-encryption-key-vault.md).

## <a name="can-i-use-an-azure-created-key-encryption-key"></a>Puis-je utiliser une clé de chiffrement principale créée par Azure ?

Oui, vous pouvez utiliser Azure Key Vault pour générer la clé de chiffrement principale pour une utilisation du chiffrement de disques Azure. Ces clés sont sauvegardées dans Azure Key Vault, le magasin de clés d’Azure Disk Encryption. Pour plus d’informations sur la clé de chiffrement principale, consultez [Création et configuration d’un coffre de clés pour Azure Disk Encryption](disk-encryption-key-vault.md).

## <a name="can-i-use-an-on-premises-key-management-service-or-hsm-to-safeguard-the-encryption-keys"></a>Puis-je utiliser un service de gestion de clés local ou HSM pour sauvegarder les clés de chiffrement ?

Vous ne pouvez pas utiliser le service de gestion de clés local ou HSM pour protéger les clés de chiffrement avec Azure Disk Encryption. Vous pouvez uniquement utiliser le service Azure Key Vault pour sauvegarder les clés de chiffrement. Pour plus d’informations sur les scénarios de prise en charge des clés de chiffrement principales, consultez [Création et configuration d’un coffre de clés pour Azure Disk Encryption](disk-encryption-key-vault.md).

## <a name="what-are-the-prerequisites-to-configure-azure-disk-encryption"></a>Quels sont les composants requis pour configurer Azure Disk Encryption ?

Azure Disk Encryption a des prérequis. Consultez l’article [Création et configuration d’un coffre de clés pour Azure Disk Encryption](disk-encryption-key-vault.md) pour créer un coffre de clés ou en configurer un existant afin d’activer l’accès au chiffrement de disque et protéger des secrets et des clés. Pour plus d’informations sur les scénarios de prise en charge des clés de chiffrement principales, consultez [Création et configuration d’un coffre de clés pour Azure Disk Encryption](disk-encryption-key-vault.md).

## <a name="what-are-the-prerequisites-to-configure-azure-disk-encryption-with-an-azure-ad-app-previous-release"></a>Quels sont les prérequis pour configurer Azure Disk Encryption avec une application Azure AD (version précédente) ?

Azure Disk Encryption a des prérequis. Consultez l’article [Azure Disk Encryption avec Azure AD](disk-encryption-windows-aad.md) pour créer une application Azure Active Directory et créer un coffre de clés ou en configurer un existant afin d’activer l’accès au chiffrement de disque et protéger des secrets et des clés. Pour plus d’informations sur les scénarios de prise en charge des clés de chiffrement principales, consultez [Création et configuration d’un coffre de clés pour Azure Disk Encryption avec Azure AD](disk-encryption-key-vault-aad.md).

## <a name="is-azure-disk-encryption-using-an-azure-ad-app-previous-release-still-supported"></a>La technologie Azure Disk Encryption utilisant une application Azure AD (version précédente) est-elle toujours pris en charge ?
Oui. Le chiffrement de disque à l’aide d’une application Azure AD est toujours pris en charge. Cependant, lors du chiffrement de nouvelles machines virtuelles, il est recommandé d’utiliser la nouvelle méthode plutôt que le chiffrement avec une application Azure AD. 

## <a name="can-i-migrate-vms-that-were-encrypted-with-an-azure-ad-app-to-encryption-without-an-azure-ad-app"></a>Puis-je migrer des machines virtuelles qui ont été chiffrées avec une application Azure AD pour les chiffrer sans application Azure AD ?
  Actuellement, il n’existe pas de chemin d’accès de migration directe pour des machines chiffrées avec une application Azure AD qui permette le chiffrement sans application Azure AD. Il n’y a pas non plus de chemin d'accès direct du chiffrement sans application Azure AD au chiffrement avec une application AD. 

## <a name="what-version-of-azure-powershell-does-azure-disk-encryption-support"></a>Quelle version d’Azure PowerShell est prise en charge par Azure Disk Encryption ?

Utilisez la dernière version du Kit de développement logiciel (SDK) Azure PowerShell pour configurer Azure Disk Encryption. Téléchargez la dernière version d’[Azure PowerShell](https://github.com/Azure/azure-powershell/releases). Azure Disk Encryption n’est *pas* pris en charge par le Kit SDK Azure version 1.1.0.

## <a name="what-is-the-disk-bek-volume-or-mntazure_bek_disk"></a>À quoi correspond le disque « volume Bek » ou « /mnt/azure_bek_disk » ?

Le « volume Bek » est un volume de données local qui stocke de façon sécurisée les clés de chiffrement pour les machines virtuelles Azure chiffrées.

> [!NOTE]
> Vous ne devez pas supprimer ni modifier le contenu de ce disque. Vous ne devez pas non plus démonter ce disque, car les clés de chiffrement qui y sont stockées sont nécessaires pour effectuer les opérations de chiffrement sur les machines virtuelles IaaS.

## <a name="what-encryption-method-does-azure-disk-encryption-use"></a>Quelle méthode de chiffrement Azure Disk Encryption utilise-t-il ?

Azure Disk Encryption sélectionne la méthode de chiffrement dans BitLocker en fonction de la version de Windows comme suit :

| Versions de Windows                 | Version | Méthode de chiffrement        |
|----------------------------------|--------|--------------------------|
| Windows Server 2012, Windows 10 ou ultérieur  | >=1511 |XTS-AES 256 bits           |
| Windows Server 2012, Windows 8, 8.1, 10 | < 1511 |AES 256 bits *              |
| Windows Server 2008R2            |        |AES 256 bits avec diffuseur |

\* La version AES 256 bits avec diffuseur n’est pas prise en charge dans Windows 2012 et versions ultérieures.

Pour déterminer la version du système d’exploitation Windows, exécutez l’outil « winver » sur votre machine virtuelle.

## <a name="can-i-backup-and-restore-an-encrypted-vm"></a>Puis-je sauvegarder et restaurer une machine virtuelle chiffrée ? 

Sauvegarde Azure fournit un mécanisme permettant de sauvegarder et de restaurer les machines virtuelles chiffrées dans le même abonnement et la même région.  Pour obtenir les instructions correspondantes, consultez [Sauvegarder et restaurer des machines virtuelles chiffrées avec Sauvegarde Azure](../../backup/backup-azure-vms-encryption.md).  La restauration d’une machine virtuelle chiffrée dans une autre région n’est pas prise en charge actuellement.  

## <a name="where-can-i-go-to-ask-questions-or-provide-feedback"></a>Où puis-je poser des questions ou envoyer des commentaires ?

Vous pouvez poser vos questions ou envoyer vos commentaires sur la [Page de questions Microsoft Q&A pour Azure Disk Encryption](/answers/topics/azure-disk-encryption.html).

## <a name="next-steps"></a>Étapes suivantes
Ce document vous a fourni les réponses aux questions les plus courantes concernant Azure Disk Encryption. Pour plus d'informations sur ce service, consultez les articles suivants :

- [Vue d’ensemble d’Azure Disk Encryption](disk-encryption-overview.md)
- [Appliquer le chiffrement de disque dans Azure Security Center](../../security-center/asset-inventory.md)
- [Chiffrement des données au repos Azure](../../security/fundamentals/encryption-atrest.md)