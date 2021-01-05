---
author: baanders
description: Fichier include pour l’étape des autorisations d’accès dans la configuration d’Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 7/17/2020
ms.author: baanders
ms.openlocfilehash: cbcaf4b4ad1b6c00f8c452582b986b6ee3b2806e
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92478834"
---
Azure Digital Twins utilise [Azure Active Directory (AAD)](../articles/active-directory/fundamentals/active-directory-whatis.md) pour le contrôle d’accès en fonction du rôle (RBAC). Cela signifie qu’avant qu’un utilisateur puisse effectuer des appels de plan de données à votre instance Azure Digital Twins, il doit se voir attribuer un rôle disposant des autorisations appropriées.

Pour Azure Digital Twins, ce rôle est _**Propriétaire de données Azure Digital Twins**_ . Pour en savoir plus sur les rôles et la sécurité, consultez [*Concepts : Sécurité pour les solutions Azure Digital Twins*](../articles/digital-twins/concepts-security.md).

[!INCLUDE [digital-twins-role-rename-note.md](digital-twins-role-rename-note.md)]

Cette section montre comment créer une attribution de rôle pour un utilisateur dans votre instance Azure Digital Twins, en utilisant l’adresse e-mail de cet utilisateur dans le locataire Azure AD sur votre abonnement Azure. Selon votre rôle dans votre organisation, vous pouvez configurer cette autorisation pour vous-même ou pour le compte d’une autre personne appelée à gérer l’instance Azure Digital Twins.

### <a name="assign-the-role"></a>Attribuer le rôle

Pour accorder à un utilisateur les autorisations nécessaires pour gérer une instance Azure Digital Twins, vous devez lui attribuer le rôle _**Propriétaire de données Azure Digital Twins**_ au sein de l’instance.

> [!NOTE]
> Notez que ce rôle diffère du rôle Azure AD *Propriétaire* qui peut également être attribué à l’étendue de l’instance Azure Digital Twins. Il s’agit de deux rôles de gestion distincts. Le rôle Azure AD *Propriétaire* n’octroie pas d’accès aux fonctionnalités de plan de données qui sont accordées avec le rôle *Propriétaire de données Azure Digital Twins* .