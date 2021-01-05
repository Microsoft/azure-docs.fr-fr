---
title: Fermer un compte professionnel ou scolaire dans une organisation Azure AD non gérée
description: Comment fermer votre compte professionnel ou scolaire dans un annuaire Azure Active Directory non managé.
services: active-directory
author: rolyon
manager: daveba
ms.service: active-directory
ms.subservice: enterprise-users
ms.topic: how-to
ms.workload: identity
ms.date: 12/03/2020
ms.author: rolyon
ms.reviewer: ''
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 252c993f5679d80cda97a6fade08f5f92489840d
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575582"
---
# <a name="close-your-work-or-school-account-in-an-unmanaged-azure-ad-organization"></a>Fermer votre compte professionnel ou scolaire dans une organisation Azure AD non gérée

Si vous êtes un utilisateur d’une organisation Azure Active Directory (Azure AD) non managée et que vous n’avez plus besoin d’utiliser des applications à partir de cette organisation ou de conserver toute association, vous pouvez fermer votre compte à tout moment. Une organisation non gérée n’a pas d’Administrateur général. Les utilisateurs d’une organisation non gérée peuvent fermer leurs comptes eux-mêmes, sans contacter d’administrateur.

Les utilisateurs d’une organisation non gérée sont souvent créés au cours d’une inscription en libre-service. Un exemple peut être un travailleur d’informations dans une organisation qui s’inscrit pour un service gratuit. Pour plus d’informations sur l’inscription en libre-service, consultez [Présentation de l’inscription en libre-service pour Azure Active Directory](directory-self-service-signup.md).

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

## <a name="before-you-begin"></a>Avant de commencer

Avant que vous puissiez fermer votre compte, vous devez vérifier les éléments suivants :

* Assurez-vous d’être un utilisateur d’une organisation Azure AD non gérée. Vous ne pouvez pas fermer votre compte si vous appartenez à une organisation gérée. Si vous appartenez à une organisation gérée et souhaitez fermer votre compte, vous devez contacter votre administrateur. Pour plus d’informations sur la façon de déterminer si vous appartenez à une organisation gérée, consultez [Supprimer l’utilisateur d’un locataire non géré](/flow/gdpr-dsr-delete#delete-the-user-from-unmanaged-tenant).

* Enregistrez toutes les données que vous souhaitez conserver. Pour plus d’informations sur la façon de soumettre une demande d’exportation, consultez [Accès et exportation des journaux générés par le système pour les locataires non managés](/power-platform/admin/powerapps-gdpr-dsr-guide-systemlogs#accessing-and-exporting-system-generated-logs-for-unmanaged-tenants).

> [!WARNING]
> La fermeture de votre compte est irréversible. Lorsque vous fermez votre compte, toutes les données personnelles sont supprimées. Vous n’aurez plus accès à votre compte et aux données associées.

## <a name="close-your-account"></a>Fermer votre compte

Pour fermer un compte professionnel ou scolaire non managé, suivez ces étapes :

1. Connectez-vous pour [fermer votre compte](https://go.microsoft.com/fwlink/?linkid=873123), en utilisant le compte que vous souhaitez fermer.

1. Sur **Mes demandes de données**, sélectionnez **Fermer le compte**.

    ![Mes demandes de données - Fermer le compte](./media/users-close-account/close-account.png)

1. Passez en revue le message de confirmation, puis sélectionnez **Oui**.

    ![Mes demandes de données - Confirmer la fermeture](./media/users-close-account/confirm-close.png)

## <a name="next-steps"></a>Étapes suivantes

- [Présentation de l’inscription en libre-service pour Azure Active Directory](directory-self-service-signup.md)
- [Supprimer l’utilisateur d’un locataire non managé](/flow/gdpr-dsr-delete#delete-the-user-from-unmanaged-tenant)
- [Accès et exportation des journaux générés par le système pour les locataires non managés](/power-platform/admin/powerapps-gdpr-dsr-guide-systemlogs#accessing-and-exporting-system-generated-logs-for-unmanaged-tenants)