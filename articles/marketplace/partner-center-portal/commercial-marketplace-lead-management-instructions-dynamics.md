---
title: Gestion des prospects pour Dynamics 365 Customer Engagement | Marketplace commercial de Microsoft
description: Découvrez comment configurer Dynamics 365 Customer Engagement pour gérer les prospects de Microsoft AppSource et de la Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: trkeya
ms.author: trkeya
ms.date: 03/30/2020
ms.openlocfilehash: 5d07dda82361ff59a43aa6753669bf38f4463059
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94491135"
---
# <a name="configure-lead-management-for-dynamics-365-customer-engagement"></a>Configurer la gestion des prospects pour Dynamics 365 Customer Engagement

Cet article décrit comment configurer Dynamics 365 Customer Engagement (précédemment appelé Dynamics CRM Online). Pour en savoir plus sur cette modification, consultez la section [Configurer l'authentification basée sur le serveur avec Customer Engagement et SharePoint Online](/dynamics365/customerengagement/on-premises/admin/on-prem-server-based-sharepoint-online) pour traiter les prospects de votre offre commerciale.

>[!NOTE]
>Ces instructions sont spécifiques à l’environnement Dynamics 365 Customer Engagement dans le cloud hébergé par Microsoft. La connexion directe à un environnement Dynamics local n'est pas prise en charge actuellement. Il existe d'autres options pour vous permettre de recevoir des prospects, notamment la configuration d'un [point de terminaison HTTPS](./commercial-marketplace-lead-management-instructions-https.md) ou d'une [table Azure](./commercial-marketplace-lead-management-instructions-azure-table.md).

## <a name="prerequisites"></a>Prérequis

Pour suivre les étapes décrites dans cet article, vous devez disposer des autorisations utilisateur suivantes :

* Droits d’administrateur sur votre instance Dynamics 365 Customer Engagement pour pouvoir installer une solution.
* Droits d’administrateur de locataire pour créer un compte pour le service utilisé afin d’envoyer des prospects à partir des offres de marketplace commercial.
* Accès au portail d’administration.
* Accès au Portail Azure.

## <a name="install-the-solution"></a>Installer la solution

1. Téléchargez la [solution Microsoft Marketplace Lead Writer](https://mpsapiprodwus.blob.core.windows.net/documentation/MicrosoftMarketplacesLeadIntegrationSolution_1_0_0_0_target_CRM_6.1_managed.zip) et enregistrez-la sur votre ordinateur.

1. Ouvrez Dynamics 365 Customer Engagement en accédant à l’URL de votre instance Dynamics, par exemple, `https://tenant.crm.dynamics.com`.

1. Sélectionnez l'icône d'engrenage sur la barre supérieure, puis sélectionnez **Paramètres avancés**.
 
    ![Élément de menu des paramètres avancés Dynamics 365](./media/commercial-marketplace-lead-management-instructions-dynamics/dynamics-advanced-settings.png)

1. Sur la page **Paramètres** , ouvrez le menu **Paramètres** sur la barre supérieure et sélectionnez **Solutions**.

    >[!NOTE]
    >Si vous ne voyez pas les options dans l’écran suivant, cela signifie que vous ne disposez pas des autorisations requises. Contactez un administrateur sur votre instance Dynamics 365 Customer Engagement.

    ![Dynamics 365 - Option Solutions](./media/commercial-marketplace-lead-management-instructions-dynamics/dynamics-solutions.png)

1. Dans la page **Solutions** , sélectionnez **Importer** , puis accédez à l’emplacement où vous avez enregistré la solution **Microsoft Marketplace Lead Writer** que vous avez téléchargée à l’étape 1.

    ![Bouton Importer](./media/commercial-marketplace-lead-management-instructions-dynamics/dynamics-crm-import.png)

1. Terminez l’importation de la solution en suivant les instructions de l’Assistant Importation de solution.

## <a name="configure-user-permissions"></a>Configurer les autorisations utilisateur

Pour écrire des prospects dans votre instance Dynamics 365 Customer Engagement, vous devez partager un compte de service avec Microsoft et configurer des autorisations pour celui-ci.

Pour créer le compte de service et attribuer les autorisations, procédez comme suit. Vous pouvez utiliser Azure Active Directory ou Office 365.

>[!NOTE]
>Passez aux instructions correspondantes selon l’option d’authentification que vous avez sélectionnée. Voir [Azure Active Directory](#azure-active-directory) ou [Office 365](#office-365).

### <a name="azure-active-directory"></a>Azure Active Directory

Nous recommandons cette option car vous n’aurez ainsi jamais besoin de mettre à jour votre nom d’utilisateur ou mot de passe afin de continuer à obtenir des prospects. Pour utiliser l’option Azure Active Directory, vous devez fournir l’ID de l’application, la Clé de l’application et l’ID de répertoire de votre application Active Directory.

Pour configurer Azure Active Directory pour Dynamics 365 Customer Engagement :

1. Connectez-vous au [portail Azure](https://portal.azure.com/). Sélectionnez **Azure Active Directory** dans le volet de gauche.

1. Sélectionnez **Propriétés** , puis copiez la valeur **ID de répertoire** dans la page **Propriétés du répertoire**. Enregistrez cette valeur car vous devrez la fournir dans le portail de publication afin de recevoir des prospects pour votre offre de la Place de marché.

    ![Élément de menu des propriétés Azure Active Directory](./media/commercial-marketplace-lead-management-instructions-dynamics/aad-properties.png)

1. Dans le volet gauche d’Azure Active Directory, sélectionnez **Inscriptions d’applications** , puis **Nouvelle inscription** dans la page qui s’affiche.
1. Entrez un nom explicite pour l’application.
1. Sous **Types de comptes pris en charge** , sélectionnez **Comptes dans un annuaire organisationnel**.
1. Sous **URI de redirection (facultatif)** , sélectionnez **Web** , puis entrez un URI, par exemple `https://contosoapp1/auth`. 
1. Sélectionnez **Inscription**.

    ![Page Inscription d’une application](./media/commercial-marketplace-lead-management-instructions-dynamics/register-an-application.png)

1. Maintenant que votre application est enregistrée, accédez à la page de présentation de l’application. Copiez la valeur **ID d’application (client)** sur cette page. Enregistrez cette valeur car vous devrez la fournir dans le portail de publication et dans Dynamics 365 afin de recevoir des prospects pour votre offre de la Place de marché.

    ![Zone ID d’application (client)](./media/commercial-marketplace-lead-management-instructions-dynamics/application-id.png)

1. Choisissez **Certificates et secrets** dans le volet gauche de l’application, puis sélectionnez le bouton **Nouveau secret client**. Entrez une description explicite de la clé secrète client, puis, sous **Expire** , sélectionnez l’option **Jamais**. Sélectionnez **Ajouter** pour créer la clé secrète client.

    ![Élément de menu Certificats et secrets](./media/commercial-marketplace-lead-management-instructions-dynamics/aad-certificates-secrets.png)

1. Dès que la clé secrète client est correctement créée, **copiez sa valeur**. Vous ne pourrez pas récupérer la valeur une fois que vous aurez quitté la page. Enregistrez cette valeur car vous devrez la fournir dans le portail de publication afin de recevoir des prospects pour votre offre de la Place de marché. 
1. Dans le volet gauche de l’application, sélectionnez **Autorisations de l’API** , puis sélectionnez **+ Ajouter une autorisation**.
1. Sélectionnez **API Microsoft** , puis l’API **Dynamics CRM**.
1. Sous **Quel type d’autorisation votre application nécessite-t-elle ?** , assurez-vous que l’option **Autorisations déléguées** est sélectionnée. 
1. Sous **Autorisation** , cochez la case **user_impersonation** pour **Accéder à Common Data Service en tant qu'utilisateurs de l'organisation**. Sélectionnez ensuite **Ajouter des autorisations**.

    ![Bouton Ajouter des autorisations](./media/commercial-marketplace-lead-management-instructions-dynamics/api-permissions.png)

1. Après avoir effectué les étapes 1 à 14 dans le portail Azure, accédez à votre instance Dynamics 365 Customer Engagement en allant à l'URL, par exemple `https://tenant.crm.dynamics.com`.
1. Sélectionnez l'icône d'engrenage sur la barre supérieure, puis sélectionnez **Paramètres avancés**.
1. Sur la page **Paramètres** , ouvrez le menu **Paramètres** sur la barre supérieure et sélectionnez **Sécurité**.
1. Sur la page **Sécurité** , sélectionnez **Utilisateurs**. Dans la page **Utilisateurs** , sélectionnez la liste déroulante **Utilisateurs activés** , puis **Utilisateurs de l’application**.
1. Sélectionnez **Nouveau** pour créer un utilisateur. 

    ![Créer un utilisateur](./media/commercial-marketplace-lead-management-instructions-dynamics/application-users.png)

1. Dans le volet **Nouvel utilisateur** , assurez-vous que l’option **UTILISATEUR : UTILISATEUR DE L'APPLICATION** est sélectionnée. Spécifiez un nom d’utilisateur, un nom complet et une adresse e-mail pour l’utilisateur que vous souhaitez utiliser avec cette connexion. Par ailleurs, collez **l’ID d’application** de l’application que vous avez créée dans le portail Azure à l’étape 8. Sélectionnez **Enregistrer et fermer** pour terminer l'ajout de l'utilisateur.

    ![Volet Nouvel utilisateur](./media/commercial-marketplace-lead-management-instructions-dynamics/new-user-info.png)

1. Accédez à la section « Paramètres de sécurité » de cet article pour terminer la configuration de la connexion pour cet utilisateur.

### <a name="office-365"></a>Office 365

Si vous ne souhaitez pas utiliser Azure Active Directory, vous pouvez inscrire un nouvel utilisateur dans le centre d’administration Microsoft 365. Vous devrez mettre à jour votre nom d’utilisateur et votre mot de passe tous les 90 jours pour continuer à obtenir des prospects.

Pour configurer Office 365 pour Dynamics 365 Customer Engagement :

1. Connectez-vous au [centre d’administration Microsoft 365](https://admin.microsoft.com).

1. Sélectionnez **Ajouter un utilisateur**.

    ![Centre d’administration Microsoft 365 - option Ajouter un utilisateur](./media/commercial-marketplace-lead-management-instructions-dynamics/ms-365-add-user.png)

1. Créez un utilisateur pour le service d’écriture de prospect. Configurez les paramètres suivants :

    * Entrez un nom d’utilisateur.
    * Entrez un mot de passe, puis désactivez la case à cocher **Amener cet utilisateur à modifier son mot de passe lors de sa première connexion**.
    * Sélectionnez **Utilisateur (pas d’accès administrateur)** en tant que rôle de l’utilisateur.
    * Sélectionnez **Offre Dynamics 365 Customer Engagement** en tant que licence de produit, comme dans l’écran suivant. Vous êtes facturé pour la licence sélectionnée. 

Enregistrez ces valeurs car vous devrez fournir les valeurs **Nom d’utilisateur** et **Mot de passe** sur le portail de publication afin de recevoir des prospects pour votre offre de la Place de marché.

![Centre d’administration Microsoft 365 - volet Nouvel utilisateur](./media/commercial-marketplace-lead-management-instructions-dynamics/ms-365-new-user.png)

## <a name="security-settings"></a>Paramètres de sécurité

L’étape finale consiste à permettre à l’utilisateur que vous avez créé d’écrire les prospects.

1. Ouvrez Dynamics 365 Customer Engagement en accédant à l’URL de votre instance Dynamics, par exemple, `https://tenant.crm.dynamics.com`.
1. Sélectionnez l'icône d'engrenage sur la barre supérieure, puis sélectionnez **Paramètres avancés**.
1. Sur la page **Paramètres** , ouvrez le menu **Paramètres** sur la barre supérieure et sélectionnez **Sécurité**.
1. Dans la page **Sécurité** , sélectionnez **Utilisateurs** , choisissez l’utilisateur que vous avez créé dans la section « Configurer les autorisations utilisateur » de ce document. Sélectionnez ensuite **Gérer les rôles**. 

    ![Onglet Gérer les rôles](./media/commercial-marketplace-lead-management-instructions-dynamics/security-manage-roles.png)

1. Recherchez le nom de rôle **Microsoft Marketplace Lead Writer** , puis sélectionnez-le pour attribuer le rôle à l’utilisateur.

    ![Volet Gérer les rôles d’utilisateur](./media/commercial-marketplace-lead-management-instructions-dynamics/security-manage-user-roles.png)

    >[!NOTE]
    >Ce rôle est créé par la solution que vous avez importée, et dispose uniquement des autorisations d’écrire les prospects et de suivre la version de la solution pour garantir la compatibilité.

1. Revenez à la page **Sécurité** , puis sélectionnez **Rôles de sécurité**. Recherchez le rôle **Microsoft Marketplace Lead Writer** et sélectionnez-le.

    ![Volet Rôles de sécurité](./media/commercial-marketplace-lead-management-instructions-dynamics/security-roles.png)

1. Dans le rôle de sécurité, sélectionnez l’onglet **Enregistrements principaux**. Recherchez l’entité **Paramètres d’interface utilisateur de l’entité utilisateur**. Activez les autorisations Créer, Lire et Écrire pour l’utilisateur (1/4 de cercle jaune) pour cette entité en cliquant une fois dans chacun des cercles correspondants.

    ![Microsoft Marketplace Lead Writer - onglet Enregistrements principaux](./media/commercial-marketplace-lead-management-instructions-dynamics/marketplace-lead-writer.png)

1. Dans l'onglet **Personnalisation** , recherchez l'élément **Tâche système**. Activez les autorisations Lire, Écrire et AppendTo pour Organisation (cercles verts en aplat) pour cette entité en cliquant quatre fois sur chacun des cercles correspondants.

    ![Microsoft Marketplace Lead Writer - onglet Personnalisation](./media/commercial-marketplace-lead-management-instructions-dynamics/marketplace-lead-writer-customization.png)

1. Sélectionnez **Enregistrer et fermer**.

## <a name="configure-your-offer-to-send-leads-to-dynamics-365-customer-engagement"></a>Configurer votre offre pour envoyer des prospects à Dynamics 365 Customer Engagement 

Pour configurer les informations de gestion des prospects pour votre offre dans le portail de publication :

1. Accédez à la page **Configuration de l’offre** de votre offre.
1. Dans la section **Prospects** , sélectionnez **Se connecter**.

    :::image type="content" source="./media/commercial-marketplace-lead-management-instructions-dynamics/customer-leads.png" alt-text="Prospects":::

1. Dans la fenêtre contextuelle Détails de la connexion, pour Destination du prospect, sélectionnez **Dynamics 365 Customer Engagement**.

    ![Zone Destination du prospect](./media/commercial-marketplace-lead-management-instructions-dynamics/connection-details-lead-destination.png)

1. Entrez l’ **URL** pour l'instance Dynamics 365, par exemple `https://contoso.crm4.dynamics.com`.

1. Sélectionnez la méthode d’ **authentification** , Azure Active Directory ou Office 365. 
1. Si vous avez sélectionné **Azure Active Directory** , entrez l’ **ID d’application (client)** (par exemple, `23456052-aaaa-bbbb-8662-1234df56788f`), l’ **ID de répertoire** (par exemple, `12345678-8af1-4asf-1234-12234d01db47`), puis **Secret client** (par exemple, `1234ABCDEDFRZ/G/FdY0aUABCEDcqhbLn/ST122345nBc=`).

    ![Authentification à l’aide d’Azure Active Directory sélectionné](./media/commercial-marketplace-lead-management-instructions-dynamics/connection-details-application-id.png)

1. Si vous avez sélectionné **Office 365** , entrez le **Nom d’utilisateur** (par exemple, `contoso@contoso.onmicrosoft.com`) et **Mot de passe** (par exemple, `P@ssw0rd`).

    ![Zone Nom d’utilisateur Office 365](./media/commercial-marketplace-lead-management-instructions-dynamics/connection-details-authentication.png)

1. Pour **E-mail du contact** , entrez les adresses e-mail des personnes de votre société qui doivent recevoir des notifications par e-mail lors de la réception d’un nouveau prospect. Vous pouvez saisir plusieurs adresses e-mail en les séparant par un point-virgule.
1. Sélectionnez **OK**.

Pour vérifier que vous êtes bien connecté à une destination de prospect, sélectionnez le bouton **Valider**. En cas de réussite, un prospect de test se trouve dans la destination du prospect.

![Zone E-mail du contact](./media/commercial-marketplace-lead-management-instructions-dynamics/dynamics-connection-details.png)

>[!NOTE]
>Vous devez terminer la configuration du reste de l'offre et la publier avant de pouvoir recevoir des prospects pour cette offre.