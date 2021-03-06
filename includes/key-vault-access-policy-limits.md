---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 08/27/2020
ms.author: msmbaldwin
ms.openlocfilehash: 20ef9a4f30aafee562096e584c4b80fef236b062
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/21/2021
ms.locfileid: "108749719"
---
Le coffre de clés prend en charge jusqu’à 1024 entrées de stratégie d’accès, chaque entrée accordant un ensemble distinct d’autorisations à un principal de sécurité particulier. En raison de cette limitation, nous vous recommandons d’attribuer des stratégies d’accès, autant que possible, à des groupes d’utilisateurs plutôt qu’à des utilisateurs individuels. L’utilisation de groupes facilite grandement la gestion des autorisations pour plusieurs personnes au sein de votre organisation. Pour plus d’informations, consultez [Gérer l’accès aux applications et aux ressources à l’aide de groupes Azure Active Directory](../articles/active-directory/fundamentals/active-directory-manage-groups.md).

Pour plus d’informations sur le contrôle d’accès Key Vault, consultez [Fonctionnalités de sécurité d’Azure Key Vault : Gestion de l’identité et de l’accès](../articles/key-vault/general/security-features.md#identity-management).