---
services: storage
author: normesta
ms.service: storage
ms.topic: include
ms.date: 09/29/2020
ms.author: normesta
ms.custom: include file
ms.openlocfilehash: 97b47b48a183951002d04cd58b08d5e2b9051eda
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017670"
---
| Mechanism | Étendue |Limites | Niveau d’autorisation pris en charge |
|---|---|---|---|
| Azure RBAC | Comptes de stockage, conteneurs. <br>Attributions de rôles Azure de ressources croisées au niveau de l’abonnement ou du groupe de ressources. | 2000 attributions de rôles Azure dans un abonnement | Rôles Azure (intégrés ou personnalisés) |
| ACL| Répertoire, fichier |32 entrées ACL (28 entrées ACL effectives) par fichier et par annuaire. Les ACL d’accès et par défaut disposent chacune de leur propre limite d’entrée de 32 ACL. |Autorisation ACL|
