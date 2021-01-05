---
title: Rôles RBAC locaux intégrés au HSM managé - Azure Key Vault | Microsoft Docs
description: Vue d’ensemble des rôles intégrés au HSM managé qui peuvent être attribués à des utilisateurs, des principaux de service, des groupes ou des identités managées.
services: key-vault
author: amitbapat
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 09/15/2020
ms.author: ambapat
ms.openlocfilehash: a5ecd8f13a3255a565168f62f358a6a38e3cbab4
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94445215"
---
# <a name="managed-hsm-local-rbac-built-in-roles"></a>Rôles RBAC locaux intégrés au HSM managé

Il existe plusieurs rôles RBAC locaux intégrés au HSM managé que vous pouvez attribuer aux utilisateurs, aux principaux de service, aux groupes et aux identités managées. Pour permettre à un principal d’effectuer une opération, vous devez lui attribuer un rôle qui lui accorde l’autorisation d’effectuer cette opération. L’ensemble de ces rôles et de ces opérations vous permettent uniquement de gérer les autorisations pour les opérations de plan de données. Si vous souhaitez gérer les autorisations de plan de contrôle pour la ressource Managed HSM (par exemple, créer un HSM managé, ou mettre à jour, déplacer ou supprimer un HSM existant), vous devez utiliser le [contrôle d’accès en fonction du rôle (RBAC Azure) Azure](../../role-based-access-control/overview.md).

## <a name="built-in-roles"></a>Rôles intégrés

|Nom du rôle|Description|id|
|---|---|---|
|Administrateur du HSM managé| Accorde un accès complet à toutes les actions de données.|a290e904-7015-4bba-90c8-60543313cdb4|
|Responsable du chiffrement du HSM managé| Octroie un accès complet à toutes les opérations de gestion et de chiffrement des clés.|515eb02d-2335-4d2d-92f2-b1cbdf9c3778|
|Utilisateur du chiffrement du HSM managé|Accorde l’accès à la création et à l’utilisation de clés pour les opérations de chiffrement. Il ne peut pas supprimer définitivement des clés.|21dbd100-6940-42c2-9190-5d6cb909625b|
|Administrateur des stratégies du HSM managé| Accorde l’autorisation de créer et de supprimer des attributions de rôles.|4bd23610-cdcf-4971-bdee-bdc562cc28e4|
|Auditeur du chiffrement du HSM managé|Accorde une autorisation de lecture pour lire (mais pas pour utiliser) les clés|2c18b078-7c48-4d3a-af88-5a3a1b3f82b3|
|Chiffrement du service du HSM managé| Accorde l’autorisation d’utiliser une clé pour le chiffrement du service. |33413926-3206-4cdd-b39a-83574fe37a17|
|Sauvegarde du HSM managé| Accorde l’autorisation d’exécuter la sauvegarde d’une seule clé ou de l’intégralité du HSM. |7b127d3c-77bd-4e3e-bbe0-dbb8971fa7f8|

## <a name="permitted-operations"></a>Opérations autorisées
> [!NOTE]  
> - Un « X » indique qu’un rôle est autorisé à exécuter l’action de données. Une cellule vide indique que le rôle n’est pas autorisé à exécuter cette action de données.
> - Tous les noms d’action de données ont le préfixe « Microsoft.KeyVault/managedHsm », qui a été omis dans les tableaux ci-dessous par souci de concision.
> - Tous les noms de rôles ont le préfixe « Managed HSM », qui a été omis dans le tableau ci-dessous par souci de concision.

|Action de données | Administrateur | Crypto Officer | Utilisateur du chiffrement | Administrateur de la stratégie | Chiffrement du service | Backup | Auditeur du chiffrement|
|---|---|---|---|---|---|---|---|
|**Gestion du domaine de sécurité**|
/securitydomain/download/action|<center>X</center>||||||
/securitydomain/upload/action|<center>X</center>||||||
/securitydomain/upload/read|<center>X</center>||||||
/securitydomain/transferkey/read|<center>X</center>||||||
|**Gestion des clés**|
|/keys/read/action|<center>X</center>|<center>X</center>|<center>X</center>||<center>X</center>||<center>X</center>|
|/keys/write/action|<center>X</center>|<center>X</center>|<center>X</center>||||
|/keys/create|<center>X</center>|<center>X</center>|<center>X</center>||||
|/keys/delete|<center>X</center>|<center>X</center>|||||
|/keys/deletedKeys/read/action|<center>X</center>|<center>X</center>|||||
|/keys/deletedKeys/recover/action|<center>X</center>|<center>X</center>|||||
|/keys/deletedKeys/delete|<center>X</center>|<center>X</center>|||||<center>X</center>|
|/keys/backup/action|<center>X</center>|<center>X</center>|<center>X</center>|||<center>X</center>|
|/keys/restore/action|<center>X</center>|<center>X</center>|||||
|/keys/export/action|<center>X</center>|<center>X</center>|||||
|/keys/import/action|<center>X</center>|<center>X</center>|||||
|**Opérations de chiffrement de clés**|
|/keys/encrypt/action|<center>X</center>|<center>X</center>|<center>X</center>||||
|/keys/decrypt/action|<center>X</center>|<center>X</center>|<center>X</center>||||
|/keys/wrap/action|<center>X</center>|<center>X</center>|<center>X</center>||<center>X</center>||
|/keys/unwrap/action|<center>X</center>|<center>X</center>|<center>X</center>||<center>X</center>||
|/keys/sign/action|<center>X</center>|<center>X</center>|<center>X</center>||||
|/keys/verify/action|<center>X</center>|<center>X</center>|<center>X</center>||||
|**Gestion des rôles**|
|/roleAssignments/delete/action|<center>X</center>|||<center>X</center>|||
|/roleAssignments/read/action|<center>X</center>|||<center>X</center>|||
|/roleAssignments/write/action|<center>X</center>|||<center>X</center>|||
|/roleDefinitions/read/action|<center>X</center>|||<center>X</center>|||
|**Gestion des sauvegardes/restaurations**|
|/backup/start/action|<center>X</center>|||||<center>X</center>|
|/backup/status/action|<center>X</center>|||||<center>X</center>|
|/restore/start/action|<center>X</center>||||||
|/restore/status/action|<center>X</center>||||||
||||||||

## <a name="next-steps"></a>Étapes suivantes

- Lisez la présentation du [Contrôle d’accès en fonction du rôle (Azure RBAC)](../../role-based-access-control/overview.md).
- Suivez un tutoriel sur la [Gestion des rôles dans Managed HSM](role-management.md).
