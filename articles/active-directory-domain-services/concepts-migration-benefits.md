---
title: Avantages de la migration d'un déploiement Classic dans Azure AD Domain Services | Microsoft Docs
description: En savoir plus sur les avantages de la migration d'un déploiement Classic d'Azure Active Directory Domain Services vers le modèle de déploiement Resource Manager
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 05/26/2020
ms.author: justinha
ms.openlocfilehash: 8cc5f5ebf389d35df02474d0561dc7827cde4d0b
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96620084"
---
# <a name="benefits-of-migration-from-the-classic-to-resource-manager-deployment-model-in-azure-active-directory-domain-services"></a>Avantages de la migration du modèle de déploiement Classic vers le modèle de déploiement Resource Manager dans Azure Active Directory Domain Services

Azure Active Directory Domain Services (AD DS) vous permet de migrer un domaine managé existant qui utilise le modèle de déploiement Classic vers le modèle de déploiement Resource Manager. Les domaines managés Azure AD DS qui utilisent le modèle de déploiement Resource Manager offrent des fonctionnalités supplémentaires telles que la stratégie de mot de passe affinée, les journaux d’audit et la protection par verrouillage de compte.

Cet article décrit les avantages de la migration. Pour commencer, consultez [Migrer Azure Active Directory Domain Services du modèle de réseau virtuel Classic vers Resource Manager][howto-migrate].

> [!NOTE]
> En 2017, le service Azure AD Domain Services est devenu disponible pour l’hébergement dans un réseau Azure Resource Manager. Depuis lors, nous avons pu créer un service plus sécurisé utilisant les fonctionnalités modernes d’Azure Resource Manager. Étant donné que les déploiements d’Azure Resource Manager remplacent entièrement les déploiements d’Azure Classic, les déploiements de réseau virtuel Classic d’Azure AD DS seront mis hors service le 1er mars 2023.
>
> Pour plus d’informations, voir l’[avis de désapprobation officiel](https://azure.microsoft.com/updates/we-are-retiring-azure-ad-domain-services-classic-vnet-support-on-march-1-2023/)

## <a name="migration-benefits"></a>Avantages de la migration

Le processus de migration permet à un domaine managé existant qui utilise le modèle de déploiement Classic de passer au modèle de déploiement Resource Manager. Lorsque vous migrez un domaine managé du modèle de déploiement Classic vers le modèle Resource Manager, vous n'êtes pas obligé de rattacher les machines au domaine managé, ou de supprimer le domaine managé et d'en créer un à partir de zéro. Les machines virtuelles restent jointes au domaine managé à la fin du processus de migration.

À l'issue de la migration, Azure AD DS fournit de nombreuses fonctionnalités qui ne sont disponibles que pour les domaines utilisant le modèle de déploiement Resource Manager, par exemple :

* [Prise en charge de la stratégie de mot de passe affinée][password-policy]
* Synchronisation plus rapide entre Azure AD et Azure AD Domain Services.
* Deux nouveaux [attributs qui se synchronisent à partir d'Azure AD][attributes] - *manager* et *employeeID*
* Accès à des contrôleurs de domaine plus puissants lors de la [mise à niveau de la référence SKU][skus]
* Protection par verrouillage de compte Active Directory
* [Notifications par e-mail pour les alertes liées à votre domaine managé][email-alerts]
* [Utilisation d'Azure Workbooks et d'Azure Monitor pour consulter les journaux d'audit et l'activité de connexion][workbooks]
* Dans les régions prises en charge, [Zones de disponibilité Azure][availability-zones]
* Intégration à d'autres produits Azure, comme [Azure Files][azure-files], [HD Insights][hd-insights] et [Windows Virtual Desktop][wvd]
* Le support a accès à plus de données de télémétrie, ce qui lui permet de résoudre les problèmes plus efficacement
* Chiffrement au repos à l'aide de [disques managés Azure][managed-disks] pour les données des contrôleurs de domaines managés

Les domaines managés qui utilisent un modèle de déploiement Resource Manager vous permettent de rester à jour en profitant des fonctionnalités les plus récentes. Les nouvelles fonctionnalités ne sont pas disponibles pour les domaines managés qui utilisent le modèle de déploiement Classic.

## <a name="next-steps"></a>Étapes suivantes

Pour commencer, consultez [Migrer Azure Active Directory Domain Services du modèle de réseau virtuel Classic vers Resource Manager][howto-migrate].

<!-- LINKS - INTERNAL -->
[password-policy]: password-policy.md
[skus]: change-sku.md
[email-alerts]: notifications.md
[workbooks]: use-azure-monitor-workbooks.md
[azure-files]: ../storage/files/storage-files-identity-auth-active-directory-domain-service-enable.md
[hd-insights]: ../hdinsight/domain-joined/apache-domain-joined-configure-using-azure-adds.md
[wvd]: ../virtual-desktop/overview.md
[availability-zones]: ../availability-zones/az-overview.md
[howto-migrate]: migrate-from-classic-vnet.md
[attributes]: synchronization.md#attribute-synchronization-and-mapping-to-azure-ad-ds
[managed-disks]: ../virtual-machines/managed-disks-overview.md
