---
title: Gestion des prospects dans Salesforce – Marketplace commercial de Microsoft
description: Découvrez comment utiliser Salesforce pour configurer des prospects pour Microsoft AppSource et la Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: trkeya
ms.author: trkeya
ms.date: 03/30/2020
ms.openlocfilehash: 73caf848ab5c6f8e973469066ce4612a075a52f5
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94489316"
---
# <a name="configure-lead-management-for-salesforce"></a>Configurer la gestion des prospects pour Salesforce

Cet article explique comment configurer votre système Salesforce de manière à traiter les prospects provenant de vos offres publiées sur Microsoft AppSource et la Place de marché commerciale.

> [!NOTE]
> La Place de marché Azure ne prend pas en charge les listes préremplies, telles qu’une liste de valeurs pour le champ **Pays**. Assurez-vous qu’aucune liste n’est configurée avant de continuer. Vous pouvez également configurer un [point de terminaison HTTPS](./commercial-marketplace-lead-management-instructions-https.md) ou une [table Azure](./commercial-marketplace-lead-management-instructions-azure-table.md) pour qu’ils reçoivent les prospects.

## <a name="set-up-your-salesforce-system"></a>Configurer votre système Salesforce

1. Connectez-vous à Salesforce.
1. Accédez aux paramètres de **Web-to-Lead**. 
    
    Si vous utilisez l’expérience Salesforce Lightning
    1. Sélectionnez **Setup** (Configurer) dans la page d’accueil de Salesforce.

       ![Configuration Salesforce](./media/commercial-marketplace-lead-management-instructions-salesforce/salesforce-1.png)

    1. Dans la page **Setup** (Configurer), accédez à **Platform Tools** (Outils de plateforme) > **Feature Settings** (Paramètres des fonctionnalités) > **Marketing** > **Web-to-Lead**.

        ![Web-to-Lead Salesforce](./media/commercial-marketplace-lead-management-instructions-salesforce/salesforce-2.png)

    Si vous utilisez l’expérience Salesforce standard :

    1. Sélectionnez **Setup** (Configurer) dans la page d’accueil de Salesforce.

       ![Configuration Salesforce standard](./media/commercial-marketplace-lead-management-instructions-salesforce/salesforce-classic-setup.png)

    1. Dans la page **Setup** (Configurer), sélectionnez **Build** (Générer) > **Customize** (Personnaliser) > **Leads** (Prospects) > **Web-to-Lead**.

        ![Salesforce standard – Web-to-Lead](./media/commercial-marketplace-lead-management-instructions-salesforce/salesforce-classic-web-to-lead.png)

   Les étapes restantes sont les mêmes pour les deux expériences Salesforce.

1. Dans la page **Web-to-Lead Setup** (Configuration de Web-to-Lead), sélectionnez le bouton **Create Web-to-Lead Form** (Créer un formulaire Web-to-Lead).
1. Dans **Web-to-Lead Setup** (Configuration de Web-to-Lead), sélectionnez **Create a Web-to-Lead Form** (Créer un formulaire Web-to-Lead).

    ![Salesforce – configuration de Web-to-Lead](./media/commercial-marketplace-lead-management-instructions-salesforce/salesforce-3.png)

1. Dans **Create a Web-to-Lead Form** (Créer un formulaire Web-to-Lead), vérifiez que le paramètre `Include reCAPTCHA in HTML` est désélectionné et sélectionnez **Generate** (Générer).

    ![Salesforce – volet Create a Web-to-Lead Form (Créer un formulaire Web-to-Lead)](./media/commercial-marketplace-lead-management-instructions-salesforce/salesforce-4.png)

1. Du texte HTML s’affiche. Recherchez le texte « oid », copiez la **valeur « oid »** à partir du texte HTML (uniquement le texte entre guillemets) et enregistrez-la. Vous collerez cette valeur dans le champ **Identificateur de l'organisation** du portail de publication.

    ![Salesforce – volet Create a Web-to-Lead Form (Créer un formulaire Web-to-Lead) affichant la valeur HTML « oid »](./media/commercial-marketplace-lead-management-instructions-salesforce/salesforce-5.png)

1. Sélectionnez **Finished**.

## <a name="configure-your-offer-to-send-leads-to-salesforce"></a>Configurer votre offre pour envoyer des prospects à Salesforce

Lorsque vous êtes prêt à configurer les informations de gestion des prospects pour votre offre dans le portail de publication, procédez comme suit.

1. Connectez-vous à l’[Espace partenaires](https://partner.microsoft.com/dashboard/home).

1. Sélectionnez votre offre, puis accédez à l’onglet **Configuration de l’offre**.

1. Dans la section **Prospects** , sélectionnez **Se connecter**.

    :::image type="content" source="./media/commercial-marketplace-lead-management-instructions-salesforce/customer-leads.png" alt-text="Prospects":::

1. Dans la fenêtre contextuelle **Détails de la connexion** , sélectionnez **Salesforce** pour définir **Destination du prospect** et collez la valeur `oid` du formulaire Web-to-Lead que vous avez créé dans le champ **Identificateur de l’organisation**.

    ![Fenêtre contextuelle Détails de la connexion – zone de validation de l’adresse e-mail du contact](./media/commercial-marketplace-lead-management-instructions-salesforce/salesforce-connection-details.png)

1. Sous **E-mail du contact** , entrez les adresses e-mail des personnes de votre société qui doivent recevoir des notifications par e-mail lors de la réception d’un nouveau prospect. Vous pouvez spécifier plusieurs e-mails en les séparant par un point-virgule.

1. Sélectionnez **OK**.

Pour vérifier que vous êtes bien connecté à une destination de prospect, sélectionnez **Valider**. En cas de réussite, un prospect de test se trouve dans la destination de prospect.

>[!NOTE]
>Vous devez terminer la configuration du reste de l'offre et la publier avant de pouvoir recevoir des prospects pour cette offre.
