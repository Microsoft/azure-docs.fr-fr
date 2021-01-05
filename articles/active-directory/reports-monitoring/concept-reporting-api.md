---
title: Prise en main de l’API de création de rapports Azure AD | Microsoft Docs
description: Prise en main de l'API de création de rapports Azure Active Directory
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 8813b911-a4ec-4234-8474-2eef9afea11e
ms.service: active-directory
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 11/13/2018
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: af448d5d28c5add39176500b988824b17f85b9fb
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861967"
---
# <a name="get-started-with-the-azure-active-directory-reporting-api"></a>Prise en main de l’API de création de rapports Azure Active Directory

Azure Active Directory vous offre un large éventail de [rapports](overview-reports.md), contenant des informations utiles pour les applications telles que les systèmes SIEM, d’audit et les outils de Business Intelligence. 

En utilisant l’API Microsoft Graph pour les rapports Azure AD, vous pouvez obtenir un accès par programme aux données via un ensemble d’API REST. Vous pouvez appeler ces API à partir de divers outils et langages de programmation.

Cet article fournit une vue d’ensemble de l’API de création de rapports, et indique notamment comment y accéder.

Si vous rencontrez des problèmes, consultez [Comment obtenir une assistance pour Azure Active Directory](../fundamentals/active-directory-troubleshooting-support-howto.md).

## <a name="prerequisites"></a>Conditions préalables requises

Pour accéder à l’API de création de rapports, avec ou sans intervention de l’utilisateur, vous devez :

1. Affecter des rôles (lecteur Sécurité, administrateur de la sécurité, administrateur général)
2. Inscrire une application
3. Accorder des autorisations
4. Rassembler les paramètres de configuration

Pour plus d’instructions, consultez les [prérequis à l’accès à l’API de création de rapports Azure Active Directory](howto-configure-prerequisites-for-reporting-api.md) sont satisfaits. 

## <a name="api-endpoints"></a>Points de terminaison d’API 

Le point de terminaison de l’API Microsoft Graph pour les journaux d’audit est `https://graph.microsoft.com/v1.0/auditLogs/directoryAudits` et le point de terminaison de l’API Microsoft Graph pour les connexions est `https://graph.microsoft.com/v1.0/auditLogs/signIns`. Pour plus d’informations, consultez les [informations de référence sur l’API d’audit](/graph/api/resources/directoryaudit) et [sur l’API de connexion](/graph/api/resources/signIn).

De plus, vous pouvez utiliser l’[API Identity Protection relative aux détections d’événements à risque](/graph/api/resources/identityriskevent?view=graph-rest-beta) pour accéder par programmation aux détections de sécurité à l’aide de Microsoft Graph. Pour plus d’informations, consultez [Prise en main d’Azure Active Directory Identity Protection et de Microsoft Graph](../identity-protection/howto-identity-protection-graph-api.md). 
  
## <a name="apis-with-microsoft-graph-explorer"></a>API avec l'Afficheur Microsoft Graph

Vous pouvez utiliser l'[Afficheur Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer) pour vérifier vos données de connexion et d'API d'audit. Veillez à vous connecter à votre compte à l’aide des deux boutons de connexion disponibles dans l’interface utilisateur de l’Explorateur Graph, puis à définir les autorisations **AuditLog.Read.All** et **Directory.ReadAll** pour votre abonné, comme indiqué.   

![Afficheur Graph](./media/concept-reporting-api/graph-explorer.png)

![Modifier l’interface utilisateur des autorisations](./media/concept-reporting-api/modify-permissions.png)

## <a name="use-certificates-to-access-the-azure-ad-reporting-api"></a>Utilisez les certificats pour accéder à l’API de création de rapports Azure AD 

Utilisez l’API de création de rapports Azure AD avec des certificats si vous envisagez de récupérer des données de rapports sans intervention de l’utilisateur.

Pour plus d’instructions, consultez [Obtenir des données à l’aide de l’API de création de rapports Azure AD avec des certificats](tutorial-access-api-with-certificates.md).

## <a name="next-steps"></a>Étapes suivantes

 * [Configuration requise pour accéder à l’API de création de rapports](howto-configure-prerequisites-for-reporting-api.md) 
 * [Obtenir des données à l’aide de l’API Génération de rapports Azure AD avec des certificats](tutorial-access-api-with-certificates.md)
 * [Résoudre les erreurs dans l’API de création de rapports Azure AD](troubleshoot-graph-api.md)
