---
title: 'Identité hybride : Comparaison des outils d’intégration d’annuaire | Microsoft Docs'
description: Cette page contient un tableau comparatif complet des différents outils d’intégration de répertoire pouvant être utilisés pour l’intégration de répertoire.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 1e62a4bd-4d55-4609-895e-70131dedbf52
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/07/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: d9c8f5a75fea6ddd85026407fe4784c8a4589e33
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89278307"
---
# <a name="hybrid-identity-directory-integration-tools-comparison"></a>Identité hybride : Comparaison des outils d’intégration d’annuaire
Au fil des années, les outils d’intégration de répertoire ont évolué et se sont améliorés.  


- [FIM](/previous-versions/windows/desktop/forefront-2010/ff182370(v=vs.100)) et [MIM](/microsoft-identity-manager/microsoft-identity-manager-2016) sont toujours pris en charge et permettent principalement la synchronisation entre les systèmes locaux.   Le [connecteur Azure AD pour FIM Windows](/previous-versions/mim/dn511001(v=ws.10)) est pris en charge à la fois dans FIM et MIM, mais n’est pas recommandé pour les nouveaux déploiements : les clients avec des sources locales telles que Notes ou SAP HCM doivent utiliser MIM pour remplir Active Directory Domain Services (AD DS), puis utiliser la synchronisation Azure AD Connect ou le provisionnement cloud Azure AD Connect pour effectuer une synchronisation d’AD DS à Azure AD.
- La [synchronisation Azure AD Connect](how-to-connect-sync-whatis.md) intègre les composants et fonctionnalités précédemment publiés dans DirSync et Azure AD Sync pour la synchronisation entre les forêts AD DS et Azure AD.  
- [L’approvisionnement cloud Azure AD Connect](../cloud-provisioning/what-is-cloud-provisioning.md) est un nouvel agent Microsoft pour la synchronisation entre AD DS et Azure AD, utile pour des scénarios tels que les fusions et acquisitions, où les forêts AD de la société acquise sont isolées de celles de l’entreprise parente.

Pour en savoir plus sur les différences entre la synchronisation Azure AD Connect et le provisionnement cloud Azure AD Connect, consultez l’article [Présentation du provisionnement cloud Azure AD Connect](../cloud-provisioning/what-is-cloud-provisioning.md)

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).