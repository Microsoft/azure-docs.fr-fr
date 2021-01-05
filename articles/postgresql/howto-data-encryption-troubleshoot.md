---
title: Résoudre les problèmes de chiffrement des données – Azure Database pour PostgreSQL – Serveur unique
description: Découvrir comment résoudre les problèmes du chiffrement des données dans Azure Database pour PostgreSQL - Serveur unique
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: how-to
ms.date: 02/13/2020
ms.openlocfilehash: c315e1df473f3d23bab7e2a78ce166f22272ee70
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93242243"
---
# <a name="troubleshoot-data-encryption-in-azure-database-for-postgresql---single-server"></a>Résoudre les problèmes de chiffrement des données dans Azure Database pour PostgreSQL - Serveur unique

Cet article vous aide à identifier et à résoudre les problèmes courants susceptibles de se produire dans le déploiement de serveur unique Azure Database pour PostgreSQL quand il est configuré avec le chiffrement des données à l’aide d’une clé gérée par le client.

## <a name="introduction"></a>Introduction

Lorsque vous configurez le chiffrement des données pour utiliser une clé gérée par le client dans Azure Key Vault, le serveur nécessite un accès continu à la clé. Si le serveur perd l’accès à la clé gérée par le client dans Azure Key Vault, il rejette toutes les connexions, retourne le message d’erreur associé, et son état passe à ***Inaccessible** _ dans le portail Azure.

Si vous n’avez plus besoin d’un serveur Azure Database pour PostgreSQL inaccessible, vous pouvez le supprimer pour arrêter les coûts. Aucune autre action sur le serveur n’est autorisée tant que l’accès au coffre de clés n’a pas été restauré et que le serveur n’est pas disponible. Il n’est pas non plus possible de définir l’option de chiffrement des données sur `No` (managée par le service) au lieu de `Yes` (gérée par le client) sur un serveur inaccessible lorsqu'il est chiffré à l’aide d’une clé gérée par le client. Vous devez revalider la clé manuellement pour que le serveur soit à nouveau accessible. Cette action est nécessaire pour protéger les données contre tout accès non autorisé pendant que les autorisations sur la clé gérée par le client sont révoquées.

## <a name="common-errors-causing-server-to-become-inaccessible"></a>Erreurs courantes provoquant l’inaccessibilité du serveur

Les erreurs de configuration suivantes provoquent la plupart des problèmes liés au chiffrement des données avec des clés Azure Key Vault :

- Le coffre de clés n’est pas disponible ou n’existe pas :
  - Le coffre de clés a été supprimé accidentellement.
  - Une erreur réseau intermittente entraîne l’indisponibilité du coffre de clés.

- Vous n’êtes pas autorisé à accéder au coffre de clés ou la clé n’existe pas :
  - La clé a expiré ou a été accidentellement supprimée ou désactivée.
- L’identité managée de l’instance Azure Database pour PostgreSQL a été supprimée par erreur.
  - L’identité managée de l’instance Azure Database pour PostgreSQL ne dispose pas des autorisations de clé suffisantes. Par exemple, les autorisations n’incluent pas Get, Wrap et Unwrap.
  - Les autorisations de l’identité managée de l’instance Azure Database pour PostgreSQL ont été révoquées ou supprimées.

## <a name="identify-and-resolve-common-errors"></a>Identifier et résoudre les erreurs courantes

### <a name="errors-on-the-key-vault"></a>Erreurs sur le coffre de clés

#### <a name="disabled-key-vault"></a>Coffre de clés désactivé

- `AzureKeyVaultKeyDisabledMessage`
- **Explication**: impossible d’effectuer l’opération sur le serveur, car la clé Azure Key Vault est désactivée.

#### <a name="missing-key-vault-permissions"></a>Autorisations d’accès au coffre de clés manquantes

- `AzureKeyVaultMissingPermissionsMessage`
- **Explication** : le serveur ne dispose pas des autorisations Get, Wrap et Unwrap requises pour Azure Key Vault. Accordez les autorisations manquantes au principal du service avec l’ID.

### <a name="mitigation"></a>Limitation des risques

- Vérifiez que la clé gérée par le client est présente dans le coffre de clés.
- Identifiez le coffre de clés et accédez à celui-ci dans le portail Azure.
- Vérifiez que l’URI de la clé identifie une clé qui est présente.

## <a name="next-steps"></a>Étapes suivantes

[Utiliser le portail Azure pour configurer le chiffrement des données avec une clé gérée par le client sur Azure Database pour PostgreSQL](howto-data-encryption-portal.md)
