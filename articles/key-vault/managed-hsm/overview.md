---
title: Vue d’ensemble d’Azure Managed HSM – Azure Managed HSM | Microsoft Docs
description: Azure Managed HSM est un service cloud qui protège vos clés de chiffrement pour les applications cloud.
services: key-vault
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: overview
ms.custom: mvc
ms.date: 09/15/2020
ms.author: mbaldwin
author: msmbaldwin
ms.openlocfilehash: 95feeebf009cae22cf3952df80596c12e75cbe4b
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94357570"
---
# <a name="what-is-azure-key-vault-managed-hsm-preview"></a>Qu’est-ce qu’Azure Key Vault Managed HSM (préversion) ?

Azure Key Vault Managed HSM est un service cloud complètement managé, à haut niveau de disponibilité et monolocataire qui vous permet de protéger les clés de chiffrement de vos applications cloud à l’aide de HSM certifiés **FIPS 140-2 de niveau 3**.  

## <a name="why-use-managed-hsm"></a>Pourquoi utiliser Managed HSM ?

### <a name="fully-managed-highly-available-single-tenant-hsm-as-a-service"></a>HSM complètement managé, à haut niveau de disponibilité et monolocataire en tant que service

- **Complètement managé**  : le provisionnement, la configuration, les mises à jour correctives et la maintenance de HSM sont assurées par le service. 
- **Haut niveau de disponibilité et résilience dans la zone** (où les zones de disponibilité sont prises en charge) : chaque cluster HSM est constitué de plusieurs partitions HSM qui s’étendent sur au moins deux zones de disponibilité. Si le matériel tombe en panne, les partitions membres de votre cluster HSM sont automatiquement migrées vers des nœuds sains.
- **Monolocataire**  : chaque instance Managed HSM est dédiée à un seul client et se compose d’un cluster de plusieurs partitions HSM. Chaque cluster HSM utilise un domaine de sécurité distinct propre au client qui isole par chiffrement le cluster HSM de chaque client.


### <a name="access-control-enhanced-data-protection--compliance"></a>Contrôle d’accès, protection améliorée des données et conformité

- **Gestion centralisée des clés**  : gérez les clés critiques et à forte valeur à l’échelle de votre organisation depuis un seul et même emplacement. Avec des autorisations précises par clé, contrôlez l’accès à chaque clé selon le principe de l’accès à privilèges minimum.
- **Contrôle d’accès isolé**  : le modèle de contrôle d’accès « RBAC local » de Managed HSM permet aux administrateurs de cluster HSM désignés de disposer d’un contrôle total sur les HSM que même les administrateurs de groupes d’administration, d’abonnements ou de groupes de ressources ne peuvent pas remplacer.
- **HSM certifiés FIPS 140-2 de niveau 3**  : protégez vos données et respectez les exigences de conformité avec des HSM certifiés FIPS (Federal Information Protection Standard) 140-2 de niveau 3. Les HSM managés utilisent les adaptateurs HSM Marvell LiquidSecurity.
- **Superviser et auditer**  : entièrement intégré à Azure Monitor. Obtenez des journaux complets de toutes les activités via Azure Monitor. Utilisez Azure Log Analytics pour l’analytique et les alertes.

### <a name="integrated-with-azure-and-microsoft-paassaas-services"></a>Intégré aux services PaaS/SaaS d’Azure et Microsoft 

- Générez des clés (ou importez-en avec [BYOK](hsm-protected-keys-byok.md)) et servez-vous-en pour chiffrer vos données au repos dans des services Azure comme [Stockage Azure](../../storage/common/customer-managed-keys-overview.md), [Azure SQL](../../azure-sql/database/transparent-data-encryption-byok-overview.md) et [Azure Information Protection](/azure/information-protection/byok-price-restrictions).

### <a name="uses-same-api-and-management-interfaces-as-key-vault"></a>Utilise les mêmes interfaces d’API et de gestion que Key Vault

- Migrez facilement vos applications existantes qui utilisent un coffre (multilocataire) pour utiliser des HSM managés.
- Utilisez les mêmes modèles de développement et de déploiement d’applications pour toutes vos applications, quelle que soit la solution de gestion de clés utilisée : coffres multilocataires ou HSM managés monolocataires.

### <a name="import-keys-from-your-on-premise-hsms"></a>Importer des clés à partir de vos HSM locaux

- Générez des clés protégées par HSM dans votre HSM local et importez-les de manière sécurisée dans Managed HSM

## <a name="next-steps"></a>Étapes suivantes
- Consultez [Démarrage rapide : Provisionner et activer un HSM managé à l’aide d’Azure CLI](quick-create-cli.md) pour créer et activer un HSM managé
- Consultez [Bonnes pratiques pour utiliser Azure Key Vault Managed HSM](best-practices.md)
