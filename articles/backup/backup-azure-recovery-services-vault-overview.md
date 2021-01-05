---
title: Vue d’ensemble des coffres Recovery Services
description: Une vue d’ensemble des coffres Recovery Services.
ms.topic: conceptual
ms.date: 08/17/2020
ms.openlocfilehash: ab6b27bdc7ac9b01385ed43830d0af5900210d43
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017319"
---
# <a name="recovery-services-vaults-overview"></a>Vue d’ensemble des coffres Recovery Services

Cet article décrit les fonctionnalités d’un coffre Recovery Services. Un coffre Recovery Services est une entité de stockage dans Azure qui héberge des données. Les données sont généralement des copies de données ou des informations de configuration pour des machines virtuelles, des charges de travail, des serveurs ou des stations de travail. Vous pouvez utiliser des coffres Recovery Services afin de stocker des données de sauvegarde pour divers services Azure tels que des machines virtuelles IaaS (Windows ou Linux) et des bases de données Azure SQL. Les coffres Recovery Services prennent en charge System Center DPM, Windows Server, le serveur de sauvegarde Azure et bien plus. Les coffres Recovery Services facilitent l’organisation de vos données de sauvegarde, tout en réduisant le temps de gestion. Les coffres Recovery Services sont basés sur le modèle Resource Manager d’Azure, qui fournit des fonctionnalités telles que :

- **Fonctionnalités enrichies pour sécuriser les données de sauvegarde** : avec les coffres Recovery Services, la sauvegarde Azure offre des fonctionnalités de sécurité pour protéger les sauvegardes cloud. Ces fonctionnalités de sécurité vous garantissent de pouvoir sécuriser vos sauvegardes et récupérer en toute sécurité des données même si des serveurs de production et de sauvegarde sont compromis. [En savoir plus](backup-azure-security-feature.md)

- **Supervision centralisée de votre environnement informatique hybride** : avec les coffres Recovery Services, vous pouvez non seulement superviser vos [machines virtuelles Azure IaaS](backup-azure-manage-vms.md), mais aussi vos [ressources locales](backup-azure-manage-windows-server.md#manage-backup-items) à partir d’un portail centralisé. [En savoir plus](backup-azure-monitoring-built-in-monitor.md)

- **Contrôle d’accès en fonction du rôle Azure (Azure RBAC)**  : Azure RBAC offre un contrôle très précis de la gestion des accès dans Azure. [Azure offre différents rôles intégrés](../role-based-access-control/built-in-roles.md), et Sauvegarde Microsoft Azure comprend trois [rôles intégrés pour gérer les points de récupération](backup-rbac-rs-vault.md). Les coffres Recovery Services sont compatibles avec Azure RBAC, ce qui limite les accès en sauvegarde et en restauration à l’ensemble défini des rôles d’utilisateur. [En savoir plus](backup-rbac-rs-vault.md)

- **Suppression réversible** :  Avec la suppression réversible, même si un intervenant malveillant supprime une sauvegarde (ou même si les données de sauvegarde sont accidentellement supprimées), les données de sauvegarde sont conservées pendant 14 jours supplémentaires, ce qui permet la récupération de cet élément de sauvegarde sans perte de données. La conservation des données de sauvegarde pendant 14 jours supplémentaires à l’état « suppression réversible » n’engendre aucun frais pour vous. [Plus d’informations](backup-azure-security-feature-cloud.md)

- **Restauration inter-région** :  La restauration inter-région (CRR) peut être utilisée pour restaurer des machines virtuelles Azure dans une région secondaire, qui est une région jumelée Azure. Si Azure déclare un incident dans la région primaire, les données répliquées dans la région secondaire peuvent être restaurées dans la région secondaire afin d’atténuer le temps d’arrêt réel de leur environnement dans la région principale. [Plus d’informations](backup-azure-arm-restore-vms.md#cross-region-restore)

## <a name="storage-settings-in-the-recovery-services-vault"></a>Paramètres de stockage dans le coffre Recovery Services

Un coffre Recovery Services est une entité qui stocke les sauvegardes et les points de récupération créés au fil du temps. Le coffre Recovery Services contient également les stratégies de sauvegarde associées aux machines virtuelles protégées.

- La Sauvegarde Azure gère automatiquement le stockage du coffre. Découvrez comment les [paramètres de stockage peuvent être modifiés](./backup-create-rs-vault.md#set-storage-redundancy).

- Pour en savoir plus sur la redondance du stockage, consultez les articles suivants sur la redondance [géographique](../storage/common/storage-redundancy.md#geo-zone-redundant-storage), [locale](../storage/common/storage-redundancy.md#locally-redundant-storage) et [zonale](../storage/common/storage-redundancy.md#zone-redundant-storage).

## <a name="encryption-settings-in-the-recovery-services-vault"></a>Paramètres de chiffrement dans le coffre Recovery Services

Cette section décrit les options disponibles pour le chiffrement de vos données de sauvegarde stockées dans le coffre Recovery Services.

### <a name="encryption-of-backup-data-using-platform-managed-keys"></a>Chiffrement des données de sauvegarde à l’aide de clés gérées par la plateforme

Par défaut, toutes vos données sont chiffrées à l’aide de clés gérées par la plateforme. Vous n’avez aucune action explicite à effectuer pour activer ce chiffrement. Il s’applique à toutes les charges de travail sauvegardées dans votre coffre Recovery Services.

### <a name="encryption-of-backup-data-using-customer-managed-keys"></a>Chiffrement des données de sauvegarde à l’aide de clés gérées par le client

Vous pouvez chiffrer vos données à l’aide de clés de chiffrement détenues et gérées par vous. Sauvegarde Azure vous permet d’utiliser vos clés RSA stockées dans le coffre Azure Key Vault pour le chiffrement de vos sauvegardes. La clé de chiffrement utilisée pour le chiffrement des sauvegardes peut être différente de celle utilisée pour la source. Les données sont protégées à l’aide d’une clé de chiffrement des données basée sur l’algorithme AES 256, qui est à son tour protégée à l’aide de vos clés. Cela vous donne un contrôle total sur les données et les clés. Pour permettre le chiffrement, le coffre Recovery Services doit être autorisé à accéder à la clé de chiffrement dans Azure Key Vault. Vous pouvez désactiver la clé ou révoquer l’accès à tout moment. Toutefois, vous devez activer le chiffrement à l’aide de vos clés avant d’essayer de protéger les éléments du coffre.

Apprenez-en davantage sur la façon de chiffrer vos données de sauvegarde à l’aide de [clés gérées par le client](encryption-at-rest-with-cmk.md).

## <a name="azure-advisor"></a>Azure Advisor

[Azure Advisor](../advisor/index.yml) est un consultant cloud personnalisé qui permet d’optimiser l’utilisation d’Azure. Il analyse votre utilisation d’Azure et fournit des recommandations en temps utile pour vous permettre d’optimiser et de sécuriser vos déploiements. Les recommandations sont divisées en quatre catégories : Haute disponibilité, sécurité, performances et coût.

Azure Advisor fournit des [recommandations](../advisor/advisor-high-availability-recommendations.md#protect-your-virtual-machine-data-from-accidental-deletion) toutes les heures pour les machines virtuelles qui ne sont pas sauvegardées. Ainsi, vous ne risquez pas d’oublier de sauvegarder des machines virtuelles importantes. Vous pouvez également contrôler les recommandations en les répétant.  Vous pouvez sélectionner la recommandation et activer la sauvegarde sur les machines virtuelles en ligne en spécifiant le coffre (où les sauvegardes vont être stockées) et la stratégie de sauvegarde (planification des sauvegardes et rétention des copies de sauvegarde).

![Azure Advisor](./media/backup-azure-recovery-services-vault-overview/azure-advisor.png)

## <a name="additional-resources"></a>Ressources complémentaires

- [Scénarios pris en charge et non pris en charge par le coffre](backup-support-matrix.md#vault-support)
- [Foire aux questions sur le coffre](backup-azure-backup-faq.md)

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour :

- [Sauvegarder une machine virtuelle IaaS](backup-azure-arm-vms-prepare.md)
- [Sauvegarder un serveur de sauvegarde Azure](backup-azure-microsoft-azure-backup.md)
- [Sauvegarder un serveur Windows Server](backup-windows-with-mars-agent.md)
