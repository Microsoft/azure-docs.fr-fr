---
title: Azure AD Connect dans Microsoft Cloud Germany
description: Azure AD Connect intègre vos répertoires locaux à Azure Active Directory. Cela vous permet de fournir une identité commune pour les applications Microsoft 365, Azure et SaaS intégrées à Azure AD.
keywords: introduction à Azure AD Connect, présentation d’Azure AD Connect, qu’est-ce qu’Azure AD Connect, installation d’active directory, Germany, Black Forest
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 2bcb0caf-5d97-46cb-8c32-bda66cc22dad
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 07/12/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 246f8326f7112825de98ee824cf4bab7952a7878
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90014585"
---
# <a name="azure-ad-connect-in-microsoft-cloud-germany---public-preview"></a>Azure AD Connect dans Microsoft Cloud Germany - Version préliminaire
## <a name="introduction"></a>Introduction
Azure AD Connect fournit la synchronisation entre Active Directory local et Azure Active Directory.
Actuellement, la plupart des scénarios dans [Microsoft Cloud Germany](https://azure.microsoft.com/global-infrastructure/germany/
) doivent être réalisés par l’opérateur. Quand vous utilisez Microsoft Cloud Germany, vous devez avoir connaissance des informations suivantes :

* Les URL suivantes doivent être ouvertes sur un serveur proxy pour que la synchronisation réussisse :
  
  * \* .microsoftonline.de
  * \* .windows.net
  * * Listes de révocation de certificat
* Lorsque vous vous connectez à votre annuaire Azure AD, vous devez utiliser un compte du domaine onmicrosoft.de.

 
## <a name="download"></a>Téléchargement
Vous pouvez télécharger Azure AD Connect à partir du panneau Azure AD Connect dans le portail.  Utilisez les instructions ci-dessous pour localiser le panneau Azure AD Connect.

### <a name="the-azure-ad-connect-blade"></a>Panneau Azure AD Connect
Une fois que vous êtes connecté au portail Azure :

1. Accédez à parcourir
2. Sélectionnez Azure Active Directory
3. Puis sélectionnez Azure AD Connect

Vous obtenez ces détails :

![Panneau Azure AD Connect](./media/reference-connect-germany/germany1.png)

Le tableau suivant décrit les fonctionnalités affichées dans le panneau.

| Intitulé | Description |
| --- | --- |
| ÉTAT DE LA SYNCHRONISATION |Vous indique si la synchronisation est activée ou désactivée. |
| DERNIÈRE SYNCHRONISATION |Dernière fois qu’une synchronisation réussie s’est terminée. |
| DOMAINES FÉDÉRÉS |Affiche le nombre de domaines fédérés actuellement configurés. |

## <a name="installation"></a>Installation
Pour installer Azure AD Connect, vous pouvez utiliser la documentation [ici](how-to-connect-install-roadmap.md).

## <a name="advanced-features-and-additional-information"></a>Fonctionnalités avancées et informations supplémentaires
Pour plus d’informations sur les paramètres personnalisés ou les configurations avancées, accédez à [Intégration d’identités locales avec Azure Active Directory](whatis-hybrid-identity.md). Cette page fournit des informations et des liens vers des conseils supplémentaires.

