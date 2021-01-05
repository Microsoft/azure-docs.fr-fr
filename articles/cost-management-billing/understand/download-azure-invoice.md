---
title: Consulter et télécharger votre facture Azure
description: Décrit comment afficher et télécharger votre facture Azure.
keywords: facturation, facture, téléchargement de facture, facture Azure, utilisation d’Azure
author: bandersmsft
ms.reviewer: amberb
tags: billing
ms.service: cost-management-billing
ms.topic: conceptual
ms.date: 05/28/2020
ms.author: banders
ms.openlocfilehash: 4e77b167f00e2cfa3838439143c6074bd4122976
ms.sourcegitcommit: 1f48ad3c83467a6ffac4e23093ef288fea592eb5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/29/2020
ms.locfileid: "84191448"
---
# <a name="view-and-download-your-microsoft-azure-invoice"></a>Afficher et télécharger votre facture Microsoft Azure

Vous pouvez télécharger votre facture dans le [portail Azure](https://portal.azure.com/) ou la faire envoyer par e-mail. Si vous êtes un client Azure avec un Contrat Entreprise (client EA), vous ne pouvez pas télécharger les factures de votre organisation. Au lieu de cela, les factures sont envoyées à la personne chargée de recevoir les factures pour l’inscription.

## <a name="when-invoices-are-generated"></a>Quand les factures sont-elles générées ?

Une facture est générée en fonction du type de compte de facturation. Des factures sont créées pour les comptes de facturation Microsoft Online Service Program (MOSP), Contrat client Microsoft (MCA) et Contrat Partenaire Microsoft (MPA). Des factures sont également générées pour les comptes de facturation Contrat Entreprise (EA). Toutefois, les factures pour les comptes de facturation EA n’apparaissent pas dans le portail Azure.

Pour plus d’informations sur les comptes de facturation et savoir comment identifier le type de votre compte de facturation, consultez [Afficher les comptes de facturation dans le portail Azure](../manage/view-all-accounts.md).

## <a name="invoices-for-mosp-billing-accounts"></a>Factures pour les comptes de facturation MOSP

Un compte de facturation MOSP est créé quand vous vous inscrivez à Azure par le biais du site web Azure. Par exemple, quand vous souscrivez à un [compte gratuit Azure](https://azure.microsoft.com/offers/ms-azr-0044p/), un [compte avec tarifs de paiement à l’utilisation](https://azure.microsoft.com/offers/ms-azr-0003p/) ou en tant qu’[abonné Visual Studio](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/).

Les clients de certaines régions, qui s’inscrivent par le biais du site web Azure pour obtenir un [compte avec tarifs de paiement à l’utilisation](https://azure.microsoft.com/offers/ms-azr-0003p/) ou un [compte gratuit Azure](https://azure.microsoft.com/offers/ms-azr-0044p/) peuvent avoir un compte de facturation pour un contrat MCA.

Si vous n’êtes pas sûr du type de compte de facturation dont vous disposez, consultez [Vérifier votre type de compte de facturation](../manage/view-all-accounts.md#check-the-type-of-your-account) avant de suivre les instructions de cet article. 

Un compte de facturation MOSP peut avoir les factures suivantes :

**Frais de service Azure** : Une facture est générée pour chaque abonnement Azure contenant des ressources Azure utilisées par l’abonnement. La facture recense les frais associés à une période de facturation donnée. La période de facturation est déterminée par le jour du mois où l’abonnement est créé.

Par exemple, John crée *Azure sub 01* le 5 mars et *Azure sub 02* le 10 mars. La facture pour *Azure sub 01* recense les frais engagés du cinquième jour du mois au quatrième jour du mois suivant. La facture pour *Azure sub 02* inclut les frais engagés du dixième jour du mois au neuvième jour du mois suivant. Les factures de tous les abonnements Azure sont normalement générées le jour du mois où le compte a été créé, mais elles peuvent être produites un ou deux jours après cette date. Dans cet exemple, si John a créé son compte le 2 février, les factures pour *Azure sub 01* et *Azure sub 02* sont normalement générées le deuxième jour de chaque mois. Toutefois, elles peuvent être produites un ou deux jours plus tard.

**Place de marché Azure, réservations et machines virtuelles Spot** : une facture est générée pour les réservations, les produits de la Place de marché et les machines virtuelles Spot achetés par le biais d’un abonnement. La facture inclut les frais respectifs du mois précédent. Par exemple, John a acheté une réservation le 1er mars et une autre le 30 mars. Une seule facture est générée pour les deux réservations en avril. La facture pour la Place de marché Azure, les réservations et les machines virtuelles Spot est toujours générée vers le neuvième jour du mois.

Si vous utilisez une carte de crédit comme mode de paiement dans Azure et que vous achetez une réservation, Azure génère une facture immédiate. Toutefois, si vous payez sur facture, la réservation apparaît sur la facture du mois suivant.

**Plan de support Azure** : Une facture est générée chaque mois pour votre abonnement au plan de support. La première facture est générée le jour de l’achat ou jusqu’à deux jours plus tard. Les factures ultérieures du plan de support sont normalement générées le jour du mois où le compte a été créé, mais elles peuvent être produites un ou deux jours après cette date.

## <a name="download-your-mosp-azure-subscription-invoice"></a>Télécharger votre facture d’abonnement Azure MOSP

Une facture est générée uniquement pour un abonnement qui appartient à un compte de facturation pour un MOSP. [Vérifiez votre accès à un compte MOSP](../manage/view-all-accounts.md#check-the-type-of-your-account). 

Vous devez disposer d’un rôle d’administrateur de compte sur un abonnement pour télécharger sa facture. Les utilisateurs disposant des rôles de propriétaire, de contributeur ou de lecteur peuvent télécharger leur facture si l’administrateur de compte leur a donné l’autorisation. Pour plus d’informations, consultez [Autoriser les utilisateurs à télécharger des factures](../manage/manage-billing-access.md#opt-in).

1. Sélectionnez votre abonnement dans la [page Abonnements](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) du portail Azure.
1. Sélectionnez **Factures** dans la section de facturation.  
    ![Capture d’écran montrant un utilisateur sélectionnant l’option Factures pour un abonnement](./media/download-azure-invoice/select-subscription-invoice.png)
1. Sélectionnez **Télécharger** pour télécharger une version PDF de votre facture, puis **Télécharger** sous la section de la facture.  
    ![Capture d’écran montrant les périodes de facturation, l’option de téléchargement et le total des frais pour chaque période de facturation](./media/download-azure-invoice/downloadinvoice-subscription.png)
1. Vous pouvez également télécharger une analyse quotidienne des quantités consommées et des frais en sélectionnant **Télécharger** sous la section de détails de l’utilisation. La préparation du fichier CSV peut prendre quelques minutes.  
    ![Capture d’écran montrant la page Télécharger les données sur l’utilisation et la facturation](./media/download-azure-invoice/usage-and-invoice-subscription.png)

Pour plus d’informations sur votre facture, consultez la page [Comprendre votre facture Microsoft Azure](../understand/review-individual-bill.md). Pour vous aider à gérer vos coûts, consultez [Éviter les coûts inattendus avec la gestion de la facturation et des coûts Azure](../manage/getting-started.md).

## <a name="download-your-mosp-support-plan-invoice"></a>Télécharger votre facture de plan de support MOSP

Une facture est générée uniquement pour un abonnement de plan de support qui appartient à un compte de facturation MOSP. [Vérifiez votre accès à un compte MOSP](../manage/view-all-accounts.md#check-the-type-of-your-account).

Vous devez disposer d’un rôle d’administrateur de compte sur l’abonnement au plan de support pour télécharger sa facture.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Recherchez **Gestion des coûts + facturation**.  
    ![Capture d’écran montrant la recherche dans le portail pour Gestion des coûts + facturation](./media/download-azure-invoice/search-cmb.png)
1. Sélectionnez **Factures** dans la partie gauche.
1. Sélectionnez votre abonnement au plan de support, puis **Télécharger**.  
    [![Capture d’écran montrant la liste des profils de facturation](./media/download-azure-invoice/cmb-invoices.png)](./media/download-azure-invoice/cmb-invoices-zoomed-in.png#lightbox)
1. Sélectionnez **Télécharger** pour télécharger une version PDF de votre facture.  
    ![Capture d’écran montrant les périodes de facturation, l’option de téléchargement et le total des frais pour chaque période de facturation](./media/download-azure-invoice/download-invoice-support-plan.png)

## <a name="allow-others-to-download-the-your-subscription-invoice"></a>Autoriser les autres utilisateurs à télécharger la facture de votre abonnement

Pour télécharger une facture :

1.  Connectez-vous au [portail Microsoft Azure](https://portal.azure.com) en tant qu’administrateur de compte pour l’abonnement.

2.  Recherchez **Gestion des coûts + facturation**.

    ![Capture d’écran montrant la recherche dans le portail pour gestion des coûts + facturation](./media/download-azure-invoice/search-cmb.png)

3.  Sélectionnez **Factures** dans la partie gauche.

4.  Sélectionnez votre abonnement Azure, puis cliquez sur **Autoriser les autres utilisateurs à télécharger la facture**.

    [![Capture d’écran montrant comment sélectionner Accès à la facture](./media/download-azure-invoice/cmb-select-access-to-invoice.png)](./media/download-azure-invoice/cmb-select-access-to-invoice-zoomed-in.png#lightbox)
1.  Sélectionnez **Activé**, puis **Enregistrer** en haut de la page.  
    ![Capture d’écran montrant comment sélectionner Activé dans Accès à la facture](./media/download-azure-invoice/cmb-access-to-invoice.png)

## <a name="get-mosp-subscription-invoice-in-email"></a>Recevoir la facture d’un abonnement MOSP par e-mail

Vous devez être titulaire d’un rôle d’administrateur de compte sur un abonnement ou un plan de support pour recevoir une facture par e-mail. L’envoi de factures par e-mail est uniquement disponible pour les abonnements et les plans de support. Les réservations et les achats réalisés dans la Place de marché Azure ne proposent pas cette option. Après avoir choisi cette option, vous pouvez ajouter des destinataires qui recevront également la facture par e-mail.

1.  Connectez-vous au [portail Azure](https://portal.azure.com).
2.  Recherchez **Gestion des coûts + facturation**.  
3.  Sélectionnez **Factures** dans la partie gauche.
4.  Sélectionnez votre abonnement Azure ou votre abonnement à un plan de support, puis sélectionnez **Recevoir la facture par e-mail**.  
    [![Capture d’écran montrant la liste des profils de facturation](./media/download-azure-invoice/cmb-email-invoice.png)](./media/download-azure-invoice/cmb-email-invoice-zoomed-in.png#lightbox)
5. Cliquez sur **Facture par e-mail** et acceptez les conditions.  
    ![Capture d’écran montrant l’étape 2 du flux d’activation](./media/download-azure-invoice/invoicearticlestep02.png)
6. La facture est envoyée à votre adresse e-mail de communication par défaut. Sélectionnez **Mettre à jour le profil** pour mettre l’adresse e-mail à jour.  
    ![Capture d’écran montrant l’étape 3 du flux d’activation](./media/download-azure-invoice/invoicearticlestep03-verifyemail.png)

## <a name="share-subscription-and-support-plan-invoices"></a>Partager des factures d’abonnement et de plan de support

Chaque mois, vous pouvez partager les factures de votre abonnement et de votre plan de support avec votre équipe de comptabilité ou les envoyer à l’une de vos autres adresses e-mail.

1. Suivez les étapes décrites dans [Recevoir les factures de votre abonnement et de votre plan de support par e-mail](#get-mosp-subscription-invoice-in-email) et sélectionnez **Configurer les destinataires**.  
    ![Capture d’écran montrant un utilisateur qui sélectionne Configurer les destinataires](./media/download-azure-invoice/invoice-article-step03.png)
1. Entrez une adresse e-mail, puis sélectionnez **Ajouter un destinataire**. Vous pouvez ajouter plusieurs adresses e-mail.  
    ![Capture d’écran montrant un utilisateur qui ajoute des destinataires supplémentaires](./media/download-azure-invoice/invoice-article-step04.png)
1. Une fois que vous avez ajouté toutes les adresses e-mail, sélectionnez **Terminé** en bas de l’écran.

## <a name="invoices-for-mca-and-mpa-billing-accounts"></a>Factures pour les comptes de facturation MCA et MPA

Un compte de facturation MCA est créé quand votre organisation travaille avec un représentant Microsoft pour signer un contrat MCA. Les clients de certaines régions, qui s’inscrivent par le biais du site web Azure pour obtenir un [compte avec tarifs de paiement à l’utilisation](https://azure.microsoft.com/offers/ms-azr-0003p/) ou un [compte gratuit Azure](https://azure.microsoft.com/offers/ms-azr-0044p/), peuvent également avoir un compte de facturation pour un contrat MCA. Pour plus d’informations, consultez [Bien démarrer avec votre compte de facturation MCA](../understand/mca-overview.md).

Un compte de facturation MPA est créé pour les partenaires du fournisseur de solutions cloud (CSP) afin de gérer leurs clients dans la nouvelle expérience de commerce. Les partenaires doivent avoir au moins un client avec un [plan Azure](https://docs.microsoft.com/partner-center/purchase-azure-plan) pour gérer leur compte de facturation dans le portail Azure. Pour plus d’informations, consultez [Bien démarrer avec votre compte de facturation MPA](../understand/mpa-overview.md).

Une facture mensuelle est générée au début du mois pour chaque profil de facturation associé à votre compte. La facture contient les frais respectifs pour tous les abonnements Azure et les autres achats du mois précédent. Par exemple, John crée *Azure sub 01* le 5 mars et *Azure sub 02* le 10 mars. Il achète un abonnement *Azure support 01* le 28 mars avec le *profil de facturation 01*. Début avril, John reçoit une facture qui inclut les frais engendrés par les deux abonnements Azure et le plan de support.

##  <a name="download-an-mca-or-mpa-billing-profile-invoice"></a>Télécharger une facture pour un profil de facturation MCA ou MPA

Vous devez être titulaire du rôle de propriétaire, de contributeur, de lecteur ou de gestionnaire de factures sur un profil de facturation pour télécharger sa facture dans le portail Azure. Les utilisateurs titulaire d’un rôle de propriétaire, de contributeur ou de lecteur sur un compte de facturation peuvent télécharger des factures pour tous les profils de facturation dans le compte.

1.  Connectez-vous au [portail Azure](https://portal.azure.com).

2.  Recherchez **Gestion des coûts + facturation**.

    ![Capture d’écran montrant la recherche dans le portail pour gestion des coûts + facturation](./media/download-azure-invoice/search-cmb.png)

3. Sélectionnez **Factures** dans la partie gauche.

    [![Capture d’écran montrant une page de factures pour un compte de facturation MCA](./media/download-azure-invoice/mca-billingprofile-invoices.png)](./media/download-azure-invoice/mca-billingprofile-invoices-zoomed-in.png#lightbox)

4. Dans la table des factures, sélectionnez la facture à télécharger.

5. Cliquez sur le bouton **Télécharger la facture au format PDF** en haut de la page.

    [![Capture d’écran montrant le téléchargement d’une facture au format PDF](./media/download-azure-invoice/mca-billingprofile-download-invoice.png)](./media/download-azure-invoice/mca-billingprofile-download-invoice-zoomed-in.png#lightbox)

6. Vous pouvez également télécharger une analyse quotidienne des quantités consommées et des frais estimés en cliquant sur **Télécharger l’utilisation Azure**. La préparation du fichier CSV peut prendre quelques minutes.

## <a name="get-your-billing-profiles-invoice-in-email"></a>Recevoir la facture de votre profil de facturation par e-mail

Vous devez disposer d’un rôle de propriétaire ou de contributeur sur le profil de facturation ou son compte de facturation pour mettre à jour la préférence d’envoi par e-mail de la facture. Une fois cette option choisie, tous les utilisateurs disposant d’un rôle de propriétaire, de collaborateur, de lecteur ou de gestionnaire de factures sur un profil de facturation reçoivent la facture par e-mail. 

1.  Connectez-vous au [portail Azure](https://portal.azure.com).
1.  Recherchez **Gestion des coûts + facturation**.  
1.  Sélectionnez **Factures** dans le volet gauche, puis **Facture par e-mail** en haut de la page.  
    [![Capture d’écran montrant une page de factures pour un compte de facturation MCA](./media/download-azure-invoice/mca-billing-profile-select-email-invoice.png)](./media/download-azure-invoice/mca-billing-profile-select-email-invoice-zoomed.png#lightbox)
1.  Si vous avez plusieurs profils de facturation, sélectionnez-en un, puis sélectionnez **Activer**.  
    ![Capture d’écran montrant une page de factures pour un compte de facturation MCA](./media/download-azure-invoice/mca-billing-profile-email-invoice.png)
1.  Sélectionnez **Update**.

2.  Recherchez **Gestion des coûts + facturation**.

    ![Capture d’écran montrant la recherche dans le portail pour gestion des coûts + facturation](./media/download-azure-invoice/search-cmb.png)

3.  Sélectionnez **Factures** dans le volet gauche, puis **Facture par e-mail** en haut de la page.

    [![Capture d’écran montrant une page de factures pour un compte de facturation MCA](./media/download-azure-invoice/mca-billingprofile-select-emailinvoice.png)](./media/download-azure-invoice/mca-billingprofile-select-emailinvoice-zoomed-in.png)

4.  Si vous avez plusieurs profils de facturation, sélectionnez-en un, puis sélectionnez **Activer**.

Vous pouvez autoriser d’autres utilisateurs à consulter, télécharger et payer des factures en leur attribuant le rôle de gestionnaire de factures pour un profil de facturation MCA ou MPA. Si vous avez choisi d’obtenir vos factures par e-mail, ces utilisateurs reçoivent également les factures de cette façon.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Recherchez **Gestion des coûts + facturation**.  
1. Sélectionnez **Profils de facturation** sur le côté gauche. Dans la liste des profils de facturation, sélectionnez un profil de facturation auquel vous souhaitez affecter un rôle de gestionnaire de factures.  
   ![Capture d’écran montrant la liste des profils de facturation](./media/download-azure-invoice/mca-select-profile-zoomed-in.png)
1. Sélectionnez **Contrôle d’accès (IAM)** dans le volet gauche, puis sélectionnez **Ajouter** en haut de la page.  
    ![Capture d’écran montrant la page de contrôle d’accès](./media/download-azure-invoice/mca-select-access-control-zoomed-in.png)
1. Dans la liste déroulante Rôle, sélectionnez **Gestionnaire de factures**. Entrez l’adresse e-mail de l’utilisateur auquel vous souhaitez accorder l’accès. Sélectionnez **Enregistrer** pour attribuer le rôle.  
   ![Capture d’écran montrant l’ajout d’un utilisateur en tant que gestionnaire de factures](./media/download-azure-invoice/mca-added-invoice-manager.png)

1. Recherchez **Gestion des coûts + facturation**.

   ![Capture d’écran montrant la recherche d’abonnements dans le portail](./media/download-azure-invoice/search-cmb.png)

1. Sélectionnez **Profils de facturation** sur le côté gauche. Dans la liste des profils de facturation, sélectionnez un profil de facturation auquel vous souhaitez affecter un rôle de gestionnaire de factures.

   ![Capture d’écran montrant la liste des profils de facturation](./media/download-azure-invoice/mca-select-profile-zoomed-in.png)

1. Sélectionnez **Contrôle d’accès (IAM)** dans le volet gauche, puis sélectionnez **Ajouter** en haut de la page.

   [![Capture d’écran montrant la page de contrôle d’accès](./media/download-azure-invoice/mca-select-access-control-zoomed-in.png)

1. Dans la liste déroulante Rôle, sélectionnez **Gestionnaire de factures**. Entrez l’adresse e-mail de l’utilisateur auquel vous souhaitez accorder l’accès. Sélectionnez **Enregistrer** pour attribuer le rôle.

   [![Capture d’écran montrant l’ajout d’un utilisateur en tant que gestionnaire de factures](./media/download-azure-invoice/mca-added-invoice-manager.png)](./media/download-azure-invoice/mca-added-invoice-manager.png#lightbox)
   
   
##  <a name="why-you-might-not-see-an-invoice"></a>Je ne vois pas ma facture. Pourquoi ?

<a name="noinvoice"></a>

Si vous ne voyez pas de facture, cela peut être pour plusieurs raisons :

- La facture n'est pas encore prête
    
    - Cela fait moins de 30 jours que vous êtes inscrit à Azure. 

    - Azure facture quelques jours après la fin de votre période de facturation. Par conséquent, une facture n’a peut-être pas encore été générée.

- Vous n’êtes pas autorisé à afficher les factures. 
    
    - Si vous avez un compte de facturation MCA ou MPA, vous devez avoir un rôle Propriétaire, Contributeur, Lecteur ou Gestionnaire de factures sur un profil de facturation ou bien un rôle Propriétaire, Contributeur ou Lecteur sur le compte de facturation pour voir les factures. 
    
    - Pour les autres comptes de facturation, il est possible que vous ne voyiez pas les factures si vous n’êtes pas Administrateur de compte.

- Votre compte ne prend pas une facture en charge.

    - Si vous avez un compte de facturation pour Microsoft Online Subscription Program (MOSP) et que vous vous êtes inscrit à un compte Azure gratuit ou à un abonnement avec un montant de crédit mensuel, vous ne recevez une facture que lorsque vous dépassez le montant de crédit mensuel.

    - Si vous avez un compte de facturation pour un Contrat Client Microsoft ou un Contrat Partenaire Microsoft, vous recevez toujours une facture.

- Vous avez accès à la facture via l’un de vos autres comptes.

    - Cette situation se produit généralement lorsque vous cliquez sur un lien dans l’e-mail, vous demandant d’afficher votre facture dans le portail. Vous cliquez sur le lien et vous affichez un message d’erreur - `We can't display your invoices. Please try again`. Vérifiez que vous êtes connecté avec l’adresse e-mail qui dispose des autorisations d’affichage des factures.

- Vous avez accès à la facture via une identité différente. 

    - Certains clients ont deux identités avec la même adresse e-mail : un compte professionnel et un compte Microsoft. En règle générale, une seule de leurs identités dispose des autorisations d’affichage des factures. S’ils se connectent avec l’identité qui ne dispose pas de l’autorisation, ils ne voient pas les factures. Vérifiez que vous utilisez l’identité correcte pour vous connecter.

- Vous vous êtes connecté au mauvais locataire Azure Active Directory (AAD). 

    - Votre compte de facturation est associé à un locataire AAD. Si vous êtes connecté à un mauvais locataire, vous ne verrez pas la facture des abonnements dans votre compte de facturation. Vérifiez que vous êtes connecté au bon locataire Azure Active Directory (AAD). Si vous n’êtes pas connecté au bon locataire, utilisez la commande suivante pour changer de locataire dans le Portail Azure :

        1. Sélectionnez votre e-mail dans l’angle supérieur droit de la page.

        2. Sélectionnez **Changer de répertoire**.

           ![Capture d’écran montrant la sélection du changement de répertoire dans le portail](./media/download-azure-invoice/select-switch-directory.png)

        3. Sélectionnez un répertoire dans la section **Tous les répertoires**.

           ![Capture d’écran montrant la sélection d’un répertoire dans le portail](./media/download-azure-invoice/select-directory.png)

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur vos facture et frais, consultez :

- [Afficher et télécharger votre utilisation et vos frais Microsoft Azure](download-azure-daily-usage.md)
- [Comprendre votre facture Microsoft Azure](review-individual-bill.md)
- [Comprendre les modalités sur votre facture Azure](understand-invoice.md)

Si vous avez un contrat MCA, consultez :

- [Comprendre les frais indiqués sur la facture de votre profil de facturation](review-customer-agreement-bill.md)
- [Comprendre les termes indiqués sur la facture de votre profil de facturation](mca-understand-your-invoice.md)
- [Comprendre le fichier d’utilisation et de frais Azure pour votre profil de facturation](mca-understand-your-usage.md)