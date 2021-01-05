---
title: Extraction de données d’instructions d’Azure Active Directory Connect Health | Microsoft Docs
description: Cette page décrit comment récupérer des données à partir d’Azure Active Directory Connect Health.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 09/02/2020
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8d147d2c094923e971e52e1dbfe3f7a19776d38c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89463217"
---
# <a name="azure-ad-connect-health-instructions-for-data-retrieval"></a>Instructions d’Azure Active Directory Connect Health pour l’extraction de données

Ce document explique comment utiliser Azure AD Connect pour récupérer des données d’Azure AD Connect Health.

[!INCLUDE [active-directory-app-provisioning.md](../../../includes/gdpr-intro-sentence.md)]

## <a name="retrieve-all-email-addresses-for-users-configured-for-health-alerts"></a>Récupérez toutes les adresses e-mail des utilisateurs configurés pour les alertes d’intégrité.

Pour récupérer les adresses e-mail de tous les utilisateurs configurés dans Azure AD Connect Health pour recevoir des alertes, procédez comme suit.

1.  Commencez par le panneau Azure Active Directory Connect Health et sélectionnez **Sync Services** dans la barre de navigation de gauche.
 ![Sync Services](./media/how-to-connect-health-data-retrieval/retrieve1.png)

2.  Cliquez sur la vignette **Alertes**.</br>
 ![Alert](./media/how-to-connect-health-data-retrieval/retrieve3.png)

3.  Cliquez sur **Paramètres de notification**.
 ![Notification](./media/how-to-connect-health-data-retrieval/retrieve4.png)

4.  Le panneau **Paramètre de notification** contient la liste des adresses e-mail activées en tant que destinataires pour les notifications d’alerte d’intégrité.
 ![E-mails](./media/how-to-connect-health-data-retrieval/retrieve5a.png)
 
## <a name="retrieve-accounts-that-were-flagged-with-ad-fs-bad-password-attempts"></a>Récupérer les comptes marqués avec des tentatives de mot de passe incorrect AD FS

Pour récupérer les comptes marqués avec des tentatives de mot de passe incorrect AD FS, procédez comme suit.

1.  Dans le panneau Azure Active Directory Health, sélectionnez **Erreurs de synchronisation**.
 ![Erreurs de synchronisation](./media/how-to-connect-health-data-retrieval/retrieve6.png)

2.  Dans le panneau **Erreurs de synchronisation**, cliquez sur **Exporter**. Cette opération a pour effet d’exporter une liste des erreurs de synchronisation enregistrées.
 ![Export](./media/how-to-connect-health-data-retrieval/retrieve7.png)

## <a name="next-steps"></a>Étapes suivantes
* [Azure AD Connect Health](./whatis-azure-ad-connect.md)
* [Installation de l’agent Azure AD Connect Health](how-to-connect-health-agent-install.md)
* [Opérations Azure AD Connect Health](how-to-connect-health-operations.md)
* [Forum Aux Questions (FAQ) Azure AD Connect Health](reference-connect-health-faq.md)
* [Historique de publication des versions d’Azure AD Connect Health](reference-connect-health-version-history.md)