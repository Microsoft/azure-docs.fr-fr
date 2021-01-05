---
title: Résoudre les erreurs relatives à l’API de création de rapports Azure Active Directory | Microsoft Docs
description: Fournit des solutions aux erreurs rencontrées lors de l’appel des API de création de rapports Azure Active Directory.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 0030c5a4-16f0-46f4-ad30-782e7fea7e40
ms.service: active-directory
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 11/13/2018
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: abc8badf261e631dd6ceb7af9a6a0cb3676ae25d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017591"
---
# <a name="troubleshoot-errors-in-azure-active-directory-reporting-api"></a>Résoudre les erreurs relatives à l’API de création de rapports Azure Active Directory

Cet article répertorie les messages d’erreur couramment rencontrés en accédant aux rapports d’activité à l’aide de l’API Microsoft Graph. Elle indique également les étapes à suivre pour les résoudre.

### <a name="500-http-internal-server-error-while-accessing-microsoft-graph-v2-endpoint"></a>Erreur interne du serveur HTTP 500 lors de l’accès au point de terminaison Microsoft Graph V2

Le point de terminaison Microsoft Graph v2 n’est pas pris en charge. Vous devez accéder aux journaux d’activité à l’aide du point de terminaison Microsoft Graph v1.

### <a name="error-neither-tenant-is-b2c-or-tenant-doesnt-have-premium-license"></a>Erreur : Aucun des locataires n'est B2C ou le locataire n'a pas de licence premium

L’accès aux rapports de connexion nécessite une licence Azure Active Directory Premium 1 (P1). Si vous voyez ce message d’erreur lorsque vous accédez aux connexions, vérifiez que votre locataire dispose bien d’une licence Azure AD P1.

### <a name="error-user-is-not-in-the-allowed-roles"></a>Erreur : L'utilisateur ne dispose pas d'un rôle autorisé 

Si ce message d’erreur s’affiche quand vous tentez d’accéder aux journaux d’audit ou aux connexions à l’aide de l’API, vérifiez que votre compte est associé au rôle **Lecteur Sécurité** ou **Lecteur de rapport** dans votre locataire Azure Active Directory. 

### <a name="error-application-missing-aad-read-directory-data-permission"></a>Erreur : L'application ne dispose pas de l'autorisation AAD « Lire les données de l'annuaire » 

Pour vérifier que votre application est exécutée avec le bon ensemble d’autorisations, suivez les étapes décrites dans les [Prérequis pour accéder à l’API de création de rapports Azure Active Directory](howto-configure-prerequisites-for-reporting-api.md). 

### <a name="error-application-missing-microsoft-graph-api-read-all-audit-log-data-permission"></a>Erreur : L’application ne dispose pas de l’autorisation de l’API Microsoft Graph « Lire toutes les données du journal d’audit »

Pour vérifier que votre application est exécutée avec le bon ensemble d’autorisations, suivez les étapes décrites dans les [Prérequis pour accéder à l’API de création de rapports Azure Active Directory](howto-configure-prerequisites-for-reporting-api.md). 

## <a name="next-steps"></a>Étapes suivantes

[Utiliser la documentation de référence de l’API d’audit](/graph/api/resources/directoryaudit?view=graph-rest-beta)
[Utiliser la documentation de référence de l’API de rapport sur les activités de connexion](/graph/api/resources/signin?view=graph-rest-beta)