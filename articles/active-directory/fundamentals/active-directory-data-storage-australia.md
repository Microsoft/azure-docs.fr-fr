---
title: Stockage de données d’identité pour les clients australiens et néo-zélandais – Azure AD
description: Découvrez où Azure Active Directory stocke les données d’identité pour les clients australiens et néo-zélandais.
services: active-directory
author: ajburnle
manager: daveba
ms.author: ajburnle
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 12/13/2019
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 498b75a6541da97e57e23465ae6eb23a6c854727
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94836952"
---
# <a name="identity-data-storage-for-australian-and-new-zealand-customers-in-azure-active-directory"></a>Stockage de données d’identité pour les clients australiens et néo-zélandais dans Azure Active Directory

Les données d’identité sont stockées par Azure AD à un emplacement géographique en fonction de l’adresse fournie par votre organisation lors de l’abonnement à un service Microsoft Online tel que Microsoft 365 ou Azure. Pour obtenir des informations sur le lieu de stockage de vos données d’identité client, vous pouvez utiliser la section [Où se trouvent vos données ?](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) du Centre de gestion de la confidentialité Microsoft.

> [!NOTE]
> Les services et applications qui s’intègrent à Azure AD ont accès aux données d’identité client. Évaluez chaque service et application que vous utilisez pour déterminer comment ils traitent les données d’identité client, et s’ils répondent aux besoins de stockage de données de votre entreprise. Pour plus d’informations sur la résidence des données des services Microsoft, consultez la section « Où se trouvent vos données ? » du Centre de gestion de la confidentialité Microsoft.

Pour les clients qui ont fourni une adresse en Australie ou en Nouvelle-Zélande, Azure AD conserve les données d’identité pour ces services dans des centres de données australiens : 
- Gestion des annuaires Azure AD 
- Authentication

Tous les autres services Azure AD stockent les données client dans des centres de données dans le monde entier. Pour localiser le centre de données d’un service, consultez [Azure Active Directory - Où se trouvent vos données ?](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located)

## <a name="microsoft-azure-ad-multi-factor-authentication-mfa"></a>Microsoft Azure AD Multi-Factor Authentication (MFA)

MFA stocke les données d’identité client dans des centres de données dans le monde entier. Pour en savoir plus sur les informations utilisateur collectées et stockées par les serveurs Azure AD MFA et Azure MFA basés sur le cloud, consultez [Collecte de données utilisateur avec Microsoft Azure Multi-Factor Authentication](../authentication/concept-mfa-data-residency.md).

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les fonctions et fonctionnalités décrites ci-dessus, consultez les articles suivants :
- [Présentation d’authentification multifacteur](../authentication/concept-mfa-howitworks.md)