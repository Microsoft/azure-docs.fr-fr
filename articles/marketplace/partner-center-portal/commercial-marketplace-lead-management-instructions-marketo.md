---
title: Gestion des prospects dans Marketo – Marketplace commercial de Microsoft
description: Découvrez comment utiliser un système CRM Marketo pour gérer les prospects de Microsoft AppSource et de la Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: trkeya
ms.author: trkeya
ms.date: 03/30/2020
ms.openlocfilehash: cbd12481312d310add15bf3d41e21e9c416f1c39
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94491073"
---
# <a name="use-marketo-to-manage-commercial-marketplace-leads"></a>Utiliser Marketo pour gérer les prospects de marketplaces commerciaux

Cet article explique comment configurer votre système CRM Marketo de manière à traiter les prospects provenant de vos offres publiées sur Microsoft AppSource et la Place de marché commerciale.

## <a name="set-up-your-marketo-crm-system"></a>Configurer votre système CRM Marketo

1. Connectez-vous à Marketo.

1. Sélectionnez **Design Studio**.

    ![Marketo Design Studio](./media/commercial-marketplace-lead-management-instructions-marketo/marketo-1.png)

1.  Sélectionnez **Nouveau formulaire**.

    ![Nouveau formulaire Marketo](./media/commercial-marketplace-lead-management-instructions-marketo/marketo-2.png)

1.  Renseignez les champs requis dans la boîte de dialogue **Nouveau formulaire** , puis sélectionnez **Créer**.

    ![Nouveau formulaire de création Marketo](./media/commercial-marketplace-lead-management-instructions-marketo/marketo-3.png)

1.  Dans la page **Détails du champ** , sélectionnez **Terminer**.

    ![Formulaire d’achèvement Marketo](./media/commercial-marketplace-lead-management-instructions-marketo/marketo-4.png)

1.  Approuvez et fermez.

1. Sous l’onglet **MarketplaceLeadBackend** , sélectionnez **Code incorporé**. 

    ![Code incorporé Marketo](./media/commercial-marketplace-lead-management-instructions-marketo/marketo-6.png)

1. Le code incorporé Marketo affiche du code qui ressemble à l’exemple suivant.

    ```html
    <form id="mktoForm_1179"></form>
    <script>MktoForms2.loadForm("("//app-ys12.marketo.com", "123-PQR-789", 1179);</script>
    ```

1. Copiez les valeurs des champs suivants, affichés dans le formulaire Code incorporé. Vous utiliserez ces valeurs pour configurer votre offre afin de recevoir des prospects à l’étape suivante. Utilisez l’exemple suivant comme guide pour obtenir les identifiants dont vous avez besoin dans l’exemple de code incorporé Marketo.

    - Server ID = **ys12**
    - Munchkin ID = **123-PQR-789**
    - Form ID = **1179**

    Autre façon de déterminer ces valeurs :

    - L’ID de serveur se trouve dans l’URL de votre instance Marketo, par exemple, `serverID.marketo.com`.
    - Procurez-vous l’ID Munchkin de votre abonnement en accédant à votre menu **Admin** > **Munchkin** dans le champ **ID du compte Munchkin** , ou en consultant la première partie de votre sous-domaine d’hôte de l’API REST Marketo : `https://{Munchkin ID}.mktorest.com`.
    - L’ID du formulaire correspond à l’ID du formulaire Code incorporé que vous avez créé à l’étape 7 pour router les prospects à partir de la Place de marché.

## <a name="configure-your-offer-to-send-leads-to-marketo"></a>Configurer votre offre pour envoyer des prospects à Marketo

Lorsque vous êtes prêt à configurer les informations de gestion des prospects pour votre offre dans le portail de publication, procédez comme suit. 

1. Connectez-vous à l’[Espace partenaires](https://partner.microsoft.com/dashboard/home).

1. Sélectionnez votre offre, puis accédez à l’onglet **Configuration de l’offre**.

1. Dans la section **Prospects** , sélectionnez **Se connecter**.

    :::image type="content" source="./media/commercial-marketplace-lead-management-instructions-marketo/customer-leads.png" alt-text="Prospects":::

1. Dans la fenêtre contextuelle **Détails de la connexion** , sélectionnez **Marketo** comme **Destination du prospect**.

    ![Choisir une destination de prospect](./media/commercial-marketplace-lead-management-instructions-marketo/choose-lead-destination.png)

1. Fournissez l’ **ID du serveur** , l’ **ID du compte Munchkin** et l’ **ID du formulaire**.

    > [!NOTE]
    > Vous devez terminer la configuration du reste de l'offre et la publier avant de pouvoir recevoir des prospects pour cette offre. 

1. Sous **E-mail du contact** , entrez les adresses e-mail des personnes de votre société qui doivent recevoir des notifications par e-mail lors de la réception d’un nouveau prospect. Vous pouvez fournir plusieurs adresses e-mail en les séparant par des points-virgules.

1. Sélectionnez **OK**.

   Pour vérifier que vous êtes bien connecté à une destination de prospect, sélectionnez **Valider**. En cas de réussite, un prospect de test se trouve dans la destination de prospect.

   ![Fenêtre contextuelle Détails de la connexion](./media/commercial-marketplace-lead-management-instructions-marketo/marketo-connection-details.png)
