---
title: Bonnes pratiques pour utiliser Azure Key Vault Managed HSM
description: Ce document décrit certaines des meilleures pratiques à employer pour utiliser Key Vault.
services: key-vault
author: amitbapat
tags: azure-key-vault
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: conceptual
ms.date: 09/17/2020
ms.author: ambapat
ms.openlocfilehash: 7a30a7ab6689b602bc9ad4f696a6fe54c80f2151
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90992675"
---
# <a name="best-practices-when-using-managed-hsm"></a>Meilleures pratiques d’utilisation du HSM managé

## <a name="control-access-to-your-managed-hsm"></a>Contrôler l’accès à votre HSM managé

Le HSM managé est un service cloud qui protège les clés de chiffrement. Ces données étant sensibles et critiques, veillez à sécuriser l’accès à vos HSM managés en permettant seulement aux applications et utilisateurs autorisés d’y accéder. Cet [article](access-control.md) fournit une vue d’ensemble du modèle d’accès. Il décrit l’authentification et l’autorisation, ainsi que le contrôle d’accès en fonction du rôle.
- Créez un [groupe de sécurité Azure Active Directory](../../active-directory/fundamentals/active-directory-manage-groups.md) pour les administrateurs HSM (plutôt que de leur attribuer le rôle d’administrateur). Cela permet d’éviter un « verrouillage administratif » en cas de suppression d’un compte individuel.
- Verrouillez l’accès à vos groupes d’administration, abonnements, groupes de ressources et HSM managés. Utilisez Azure RBAC pour contrôler l’accès à vos groupes d’administration, abonnements et groupes de ressources.
- Créer des attributions de rôle par clé à l’aide de [RBAC local HSM managé](access-control.md#data-plane-and-managed-hsm-local-rbac)
- Utiliser le principe des privilèges d’accès minimum pour attribuer des rôles

## <a name="choose-regions-that-support-availability-zones"></a>Choisir les régions prenant en charge les zones de disponibilité

- Pour garantir une haute disponibilité et une résilience de zone optimales, choisissez les régions Azure au sein desquelles les [zones de disponibilité](../../availability-zones/az-overview.md) sont prises en charge. Ces régions apparaissent en tant que « régions recommandées » dans le portail Azure.

## <a name="backup"></a>Backup

- Veillez à régulièrement sauvegarder votre HSM. Les sauvegardes peuvent être effectuées au niveau du HSM, ainsi que pour des clés spécifiques. 

## <a name="turn-on-logging"></a>Activation de la journalisation

- [Activez la journalisation](logging.md) pour votre HSM. Configurez également des alertes.

## <a name="turn-on-recovery-options"></a>Activer les options de récupération

- Le [suppression réversible](../general/soft-delete-overview.md) est activée par défaut.
- Activez la protection contre le vidage si vous souhaitez vous prémunir contre la suppression forcée du HSM, même après activation de la suppression réversible.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur la sauvegarde/restauration HSM complète, consultez [Sauvegarde/Restauration complète](backup-restore.md).
- Pour savoir comment utiliser Azure Monitor pour configurer la journalisation, consultez [Journalisation du HSM managé](logging.md).
- Pour la gestion des clés, consultez [Gérer les clés du HSM managé](key-management.md).
- Pour gérer les attributions de rôles, consultez [Gestion du rôle HSM managé](role-management.md).
