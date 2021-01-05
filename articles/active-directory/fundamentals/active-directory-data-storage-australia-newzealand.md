---
title: Stockage de données client pour les clients australiens et néo-zélandais – Azure AD
description: Découvrez où Azure Active Directory stocke les données client pour les clients australiens et néo-zélandais.
services: active-directory
author: ajburnle
manager: daveba
ms.author: ajburnle
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 7/21/2020
ms.custom: it-pro, seodec18, references_regions
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4d7c37e64e4f1b339ae66fe3d9135b1874476eb3
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94836969"
---
# <a name="customer-data-storage-for-australian-and-new-zealand-customers-in-azure-active-directory"></a>Stockage de données client pour les clients australiens et néo-zélandais dans Azure Active Directory 

Azure Active Directory (Azure AD) stocke ses données client dans un emplacement géographique qui dépend du pays indiqué lors de l’inscription à un service Microsoft Online Services. Microsoft Online Services inclut Office 365 et Azure. 

Pour plus d’informations sur l’emplacement des données Azure AD et des autres services Microsoft, consultez la section [Où se trouvent vos données ?](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) du Centre de gestion de la confidentialité Microsoft.

À partir du 26 février 2020, Microsoft a commencé à stocker les données client Azure AD des nouveaux locataires présentant une adresse de facturation australienne ou néo-zélandaise dans les centres de données australiens. Entre le 1er mai 2020 et le 31 mars 2021, Microsoft migrera les locataires existants disposant d’une adresse de facturation australienne ou néo-zélandaise vers les centres de données australiens sans aucune intervention du client. Le processus de migration n’impliquera pas de temps d’arrêt pour les clients et n’aura aucun impact sur les fonctionnalités des locataires.

Par ailleurs, certaines fonctionnalités Azure AD ne prennent pas encore en charge le stockage des données client en Australie. Pour obtenir des informations spécifiques sur les fonctionnalités, accédez à [Mappage de données Azure AD](https://msit.powerbi.com/view?r=eyJrIjoiYzEyZTc5OTgtNTdlZS00ZTVkLWExN2ItOTM0OWU4NjljOGVjIiwidCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsImMiOjV9). Par exemple, Microsoft Azure AD Multi-Factor Authentication stocke les données client aux États-Unis et les traite à l’échelle mondiale. Consultez [Résidence des données et données client pour Azure AD Multi-Factor Authentication](../authentication/concept-mfa-data-residency.md).

> [!NOTE]
> Les produits et services Microsoft ainsi que les applications tierces qui s’intègrent à Azure AD ont accès aux données client. Évaluez chacun des produits, services et applications que vous utilisez pour déterminer comment ils traitent les données client et s’ils répondent aux besoins de stockage de données de votre entreprise. Pour plus d’informations sur la résidence des données des services de Microsoft, consultez la section [Où se trouvent vos données ?](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) du Microsoft Trust Center.