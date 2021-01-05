---
title: Modifier les informations de contact d’un compte de facturation Azure
description: Décrit comment modifier les informations de contact de votre compte de facturation Azure
author: genlin
ms.reviewer: dcscontentpm
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 10/26/2020
ms.author: banders
ms.custom: contperf-fy21q2
ms.openlocfilehash: 776c207387ee55b998615131baf77e6c13655b49
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033935"
---
# <a name="change-contact-information-for-an-azure-billing-account"></a>Modifier les informations de contact d’un compte de facturation Azure

Cet article vous aide à mettre à jour les informations de contact d’un *compte de facturation* dans le Portail Azure. Les instructions permettant de mettre à jour les informations de contact varient en fonction du type de compte de facturation. Pour plus d’informations sur les comptes de facturation et savoir comment identifier le type de votre compte de facturation, consultez [Afficher les comptes de facturation dans le portail Azure](view-all-accounts.md). Un compte de facturation Azure est distinct du compte d’utilisateur Azure et du [compte Microsoft](https://account.microsoft.com/).

Si vous voulez mettre à jour les informations de votre profil d’utilisateur Azure Active Directory, seul un administrateur d’utilisateurs peut apporter des modifications. Si vous n’avez pas le rôle d’administrateur d’utilisateurs, contactez la personne concernée. Pour plus d’informations sur la modification du profil d’utilisateur, consultez [Ajouter ou mettre à jour les informations du profil de l’utilisateur avec Azure Active Directory](../../active-directory/fundamentals/active-directory-users-profile-azure-portal.md).

*Adresse du donneur d’ordre* : l’adresse du donneur d’ordre est composée de l’adresse et des coordonnées de l’organisation ou de la personne responsable d’un compte de facturation. Elle s’affiche dans toutes les factures générées pour le compte de facturation.

*Adresse de facturation* : l’adresse de facturation est composée de l’adresse et des coordonnées de l’organisation ou de la personne responsable des factures générées pour un compte de facturation. Pour un compte de facturation d’un MOSP (Microsoft Online Service Program), il existe une seule adresse de facturation, elle s’affiche sur toutes les factures générées pour le compte. Pour un compte de facturation d’un contrat client Microsoft (MCA), une adresse de facturation existe pour chaque profil de facturation ; elle est affichée dans la facture générée pour le profil de facturation.

*Adresse e-mail du contact pour les e-mails marketing et de service* : vous pouvez spécifier une adresse e-mail différente de l’adresse avec laquelle vous vous connectez pour recevoir des notifications importantes de facturation, de service et de recommandation concernant votre compte Azure. Les e-mails de notification de service, tels que les problèmes de sécurité urgents, les changements de prix ou les changements cassants apportés aux services en cours d’utilisation par votre compte, sont toujours envoyés à votre adresse de connexion.

## <a name="update-an-mosp-billing-account-address"></a>Mettre à jour une adresse de compte de facturation MOSP

1. Connectez-vous au portail Azure à l’aide de l’adresse e-mail disposant de l’autorisation d’administration de compte sur ce compte.
1. Recherchez **Gestion des coûts + facturation**.  
    ![Capture d’écran montrant la recherche de « gestion des coûts + facturation » dans le Portail Azure](./media/change-azure-account-profile/search-cmb.png)
1. Sélectionnez **Propriétés** dans la partie gauche.  
    ![Capture d’écran montrant les propriétés d’un compte de facturation MOSP](./media/change-azure-account-profile/update-contact-information-select-properties.png)
1. Sélectionnez **Mettre à jour l’adresse de facturation** pour mettre à jour l’adresse du donneur d’ordre et l’adresse de facturation. Entrez la nouvelle adresse, puis sélectionnez **Enregistrer**.  
    ![Capture d’écran illustrant la mise à jour de l’adresse pour le compte de facturation MOSP](./media/change-azure-account-profile/update-contact-information-mosp.png)

## <a name="update-an-mca-billing-account-sold-to-address"></a>Mettre à jour une adresse de donneur d’ordre d’un compte de facturation MCA

1. Connectez-vous au portail Azure à l’aide de l’adresse e-mail disposant d’un rôle propriétaire ou contributeur sur le compte de facturation d’un contrat client Microsoft.
1. Recherchez **Gestion des coûts + facturation**.  
    ![Capture d’écran montrant la zone de recherche dans le Portail Azure](./media/change-azure-account-profile/search-cmb.png)
1. Sélectionnez **Propriétés** dans la partie gauche, puis sélectionnez **Mettre à jour le donneur d’ordre**.  
    ![Capture d’écran montrant les propriétés d’un compte de facturation MCA, dans lesquelles vous pouvez modifier l’adresse du donneur d’ordre](./media/change-azure-account-profile/update-sold-to-list-properties-mca.png)
1. Entrez la nouvelle adresse, puis sélectionnez **Enregistrer**.  
    ![Capture d’écran montrant la modification de l’adresse du donneur d’ordre pour un compte MCA](./media/change-azure-account-profile/update-sold-to-save-mca.png)

    > [!IMPORTANT]
    > Certains comptes nécessitent une vérification supplémentaire avant de pouvoir mettre à jour leur donneur d’ordre. Si votre compte a besoin d’une approbation manuelle, il vous sera demandé de contacter le support Azure.

## <a name="update-an-mca-billing-account-address"></a>Mettre à jour une adresse d’un compte de facturation MCA

1. Connectez-vous au portail Azure à l’aide de l’adresse e-mail disposant d’un rôle propriétaire ou collaborateur sur un compte de facturation ou un profil de facturation pour un MCA.
1. Recherchez **Gestion des coûts + facturation**.  
1. Sélectionnez **Profils de facturation** sur le côté gauche.
1. Sélectionnez un profil de facturation pour mettre à jour l’adresse de facturation.  
    ![Capture d’écran montrant la page Profils de facturation, où vous pouvez sélectionner un profil de facturation](./media/change-azure-account-profile/update-bill-to-list-profiles-mca.png)
1. Sélectionnez **Propriétés** dans la partie gauche.
1. Sélectionnez **Mettre à jour l’adresse**.  
    ![Capture d’écran montrant où modifier l’adresse](./media/change-azure-account-profile/update-bill-to-list-properties-mca.png)
1. Entrez la nouvelle adresse, puis sélectionnez **Enregistrer**.  
    ![Capture d’écran montrant la mise à jour de l’adresse](./media/change-azure-account-profile/update-bill-to-save-mca.png)

## <a name="service-and-marketing-emails"></a>E-mails marketing et de service

Dans le portail Azure, vous êtes invité à vérifier ou mettre à jour votre adresse e-mail tous les 90 jours. Microsoft envoie à cette adresse des e-mails contenant des informations relatives aux comptes Azure au sujet de :

- Notifications de service
- Alertes de sécurité
- Facturation
- Support
- Communications marketing
- Meilleures pratiques recommandées en fonction de votre utilisation d’Azure

Entrez l’adresse e-mail sur laquelle vous souhaitez recevoir des communications relatives à votre compte. En entrant une adresse e-mail, vous acceptez de recevoir des communications de Microsoft.

![Exemple de l’invite pour mettre à jour vos informations de contact](./media/change-azure-account-profile/update-contact-information.png)

### <a name="change-your-contact-email-address"></a>Modifier votre adresse e-mail de contact

Vous pouvez modifier votre adresse e-mail de contact en utilisant l’une des méthodes suivantes. La mise à jour de votre adresse e-mail de contact ne met pas à jour l’adresse e-mail avec laquelle vous vous connectez.

1. Si vous êtes administrateur de compte d’un compte MOSP, suivez les instructions dans [Mettre à jour une adresse de compte de facturation MOSP](#update-an-mosp-billing-account-address) et sélectionnez **Mettre à jour les informations de contact** à la dernière étape. Ensuite, entrez la nouvelle adresse e-mail.
1. Accédez à la section [Informations de contact](https://portal.azure.com/#blade/HubsExtension/ContactInfoBlade) du portail Azure et entrez la nouvelle adresse e-mail. 
1. Dans le portail Azure, sélectionnez l’icône avec vos initiales ou votre photo. Puis, sélectionnez le menu contextuel ( **...** ). Ensuite, sélectionnez **Mes informations de contact** dans le menu et entrez la nouvelle adresse e-mail.

![Exemple de mise à jour d’une adresse e-mail dans Azure](./media/change-azure-account-profile/azure-contact-information.png)

### <a name="opt-out-of-marketing-emails"></a>Refuser les e-mails marketing

Pour arrêter de recevoir des e-mails marketing :

1. Accédez au [formulaire de requête](https://account.microsoft.com/profile/permissions-link-request) pour envoyer une requête via l’adresse e-mail de votre profil. Vous allez recevoir un lien par e-mail pour mettre à jour vos préférences.
1. Sélectionnez le lien pour ouvrir la page **Manage communication permissions** (Gérer les autorisations de communication). Cette page affiche les types de communications marketing que vous avez accepté de recevoir sur cette adresse e-mail. Décochez toutes les sélections que vous souhaitez exclure, puis sélectionnez **Enregistrer**.  
    ![Exemple de la page de gestion des autorisations de communication](./media/change-azure-account-profile/manage-communication-permissions.png)

Même si vous refusez de recevoir des communications marketing, vous recevrez toujours des notifications de service, en fonction de votre compte.

## <a name="update-the-email-address-that-you-sign-in-with"></a>Mettre à jour l’adresse e-mail avec laquelle vous vous connectez

Vous ne pouvez pas mettre à jour l’adresse e-mail que vous utilisez pour accéder à votre compte. Toutefois, si vous disposez d’un compte de facturation pour un MOSP, vous pouvez vous inscrire à un autre compte à l’aide de la nouvelle adresse e-mail, puis transférer la propriété de vos abonnements sur le nouveau compte. Pour un compte de facturation MCA, vous pouvez attribuer à la nouvelle adresse e-mail les autorisations sur votre compte.

## <a name="update-your-credit-card"></a>Mettre à jour votre carte de crédit

Pour savoir comment mettre à jour votre carte de crédit, consultez [Modifier la carte de crédit utilisée pour régler un abonnement Azure](change-credit-card.md).

## <a name="update-your-country-or-region"></a>Mettre à jour votre pays ou région

La modification du pays ou de la région d’un compte existant n’est pas prise en charge. Toutefois, vous pouvez créer un nouveau compte dans un autre pays ou une autre région, puis contacter le support technique Azure pour transférer votre abonnement sur le nouveau compte.

## <a name="change-the-subscription-name"></a>Modifier le nom d’abonnement

1. Connectez-vous au portail Azure, sélectionnez **Abonnement** dans le volet gauche, puis sélectionnez l’abonnement que vous voulez renommer.
1. Sélectionnez **Vue d’ensemble**, puis **Renommer** dans la barre de commandes.  
    ![Exemple de changement de nom d’un abonnement Azure](./media/change-azure-account-profile/rename-sub.png)
1. Après avoir modifié le nom, sélectionnez **Enregistrer**.

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

- [Afficher vos comptes de facturation](view-all-accounts.md)
