---
title: Gestion des certificats de fédération dans Azure AD | Microsoft Docs
description: Apprenez à personnaliser la date d’expiration pour vos certificats de fédération, mais aussi à renouveler les certificats arrivant à expiration.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/04/2019
ms.author: kenwith
ms.reviewer: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 87190e9a7aa3c3a26b77220dd0332d65c640bc2e
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658993"
---
# <a name="manage-certificates-for-federated-single-sign-on-in-azure-active-directory"></a>Gérer des certificats pour l’authentification unique fédérée sur Azure Active Directory

Dans cet article, nous abordons les questions et informations relatives aux certificats créés par Azure Active Directory (Azure AD) pour établir une authentification unique fédérée vers vos applications Software as a service (SaaS). Ajoutez des applications à partir de la galerie d’applications Azure AD ou à l’aide de modèle d’applications ne figurant pas dans la galerie. Configurez l’application à l’aide de l’option d’authentification unique fédérée.

Cet article concerne uniquement les applications configurées pour utiliser l’authentification unique Azure AD via une fédération [Security Assertion Markup Language](https://wikipedia.org/wiki/Security_Assertion_Markup_Language) (SAML).

## <a name="auto-generated-certificate-for-gallery-and-non-gallery-applications"></a>Certificat généré automatiquement pour les applications de la galerie et celles ne figurant pas dans la galerie

Lorsque vous ajoutez une nouvelle application à partir de la galerie et configurez une authentification SAML (en sélectionnant **Authentification unique** > **SAML** dans la page de présentation de l’application), Azure AD génère un certificat d’une validité de trois ans pour l’application. Pour télécharger le certificat actif en tant que fichier de certificat de sécurité ( **.cer**), retournez à cette page (**Authentification basée sur SAML**) et sélectionnez un lien de téléchargement dans l’en-tête **Certificat de signature SAML**. Vous pouvez choisir entre le certificat brut (binaire) ou le certificat Base64 (texte codé en base 64). Pour les applications de la galerie, cette section peut également afficher un lien permettant de télécharger le certificat en tant qu’élément XML de métadonnées de fédération (un fichier **.xml**), selon la configuration de l’application.

![Options de téléchargement du certificat de signature SAML actif](./media/manage-certificates-for-federated-single-sign-on/active-certificate-download-options.png)

Vous pouvez également télécharger un certificat actif ou inactif en sélectionnant l'icône **Modifier** **Certificat de signature SAML** de l'en-tête (un crayon), qui affiche la page **Certificat de signature SAML**. Sélectionnez les points de suspension ( **...** ) en regard du certificat que vous voulez télécharger, puis choisissez un format de certificat. Vous avez aussi la possibilité de télécharger le certificat au format PEM (Privacy Enhanced Mail). Ce format est identique à Base64 mais avec une extension de nom de fichier **.pem**, qui n'est pas reconnue dans Windows comme un format de certificat.

![Options de téléchargement du certificat de signature SAML (actif et inactif)](./media/manage-certificates-for-federated-single-sign-on/all-certificate-download-options.png)

## <a name="customize-the-expiration-date-for-your-federation-certificate-and-roll-it-over-to-a-new-certificate"></a>Personnaliser la date d’expiration de votre certificat de fédération et activer le nouveau certificat

Par défaut, Azure configure un certificat pour qu'il expire après trois ans lorsqu'il est créé automatiquement lors de la configuration de l’authentification unique SAML. Étant donné que vous ne pouvez pas changer la date d'un certificat après l'avoir enregistré, vous devez :

1. Créer un certificat avec la date souhaitée.
1. Enregistrer le nouveau certificat.
1. Télécharger le nouveau certificat au bon format.
1. Charger le nouveau certificat dans l'application.
1. Activer le nouveau certificat dans le portail Azure Active Directory.

Les deux sections suivantes vous aident à effectuer ces étapes.

### <a name="create-a-new-certificate"></a>Créer un certificat

Tout d'abord, créez et enregistrez un nouveau certificat avec une date d'expiration différente :

1. Connectez-vous au [portail Azure Active Directory](https://aad.portal.azure.com/). La page du **Centre d’administration Azure Active Directory** s’affiche.
1. Dans le volet gauche, sélectionnez **Applications d’entreprise**. Une liste des applications d’entreprise de votre compte s’affiche.
1. Sélectionnez l’application affectée. Une page de vue d’ensemble de l’application s’affiche.
1. Dans le volet gauche de la page de vue d’ensemble de l’application, sélectionnez **Authentification unique**.
1. Si la page **Sélectionner une méthode d’authentification unique** apparaît, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML - Préversion**, recherchez l’en-tête **Certificat de signature SAML** et sélectionnez l’icône **Modifier** (associée à un crayon). La page **Certificat de signature SAML** apparaît et indique l'état (**Actif** ou **Inactif**), la date d'expiration et l'empreinte (chaîne de hachage) de chaque certificat.
1. Sélectionnez **Nouveau certificat**. Une nouvelle ligne apparaît sous la liste des certificats, où la date d'expiration par défaut est fixée à exactement trois ans après la date du jour. (Vos modifications n'ont pas encore été enregistrées et vous pouvez donc toujours modifier la date d'expiration.)
1. Dans la ligne du nouveau certificat, pointez sur la colonne de date d'expiration, puis sélectionnez l'icône **Sélectionner une date** (calendrier). Un contrôle de calendrier apparaît, affichant les jours du mois pour la date d'expiration actuelle de la nouvelle ligne.
1. Utilisez le contrôle du calendrier pour définir une nouvelle date. Vous pouvez choisir n'importe quelle date entre la date du jour et trois ans après aujourd’hui.
1. Sélectionnez **Enregistrer**. Le nouveau certificat affiche maintenant un statut **Inactif**, la date d'expiration que vous avez choisie et une empreinte.
1. Sélectionnez **X** pour revenir à la page **Configurer l’authentification unique avec SAML - Préversion**.

### <a name="upload-and-activate-a-certificate"></a>Charger et activer un certificat

Téléchargez ensuite le nouveau certificat au bon format, chargez-le dans l'application, puis activez-le dans Azure Active Directory :

1. Consultez les instructions supplémentaires de configuration de l’authentification SAML de l'application à l’aide d’une des méthodes suivantes :

   - sélectionnez le lien **guide de configuration** pour afficher une fenêtre ou un onglet de navigateur distincts, ou
   - accédez à l’en-tête **configurer**, puis sélectionnez **Afficher les instructions détaillées** pour afficher dans une barre latérale.

1. Dans les instructions, notez le format d'encodage requis pour le chargement du certificat.
1. Suivez les instructions de la section précédente intitulée [Certificat généré automatiquement pour les applications de la galerie et celles ne figurant pas dans la galerie](#auto-generated-certificate-for-gallery-and-non-gallery-applications). Cette étape télécharge le certificat au format d'encodage requis pour le chargement par l'application.
1. Pour passer au nouveau certificat, retournez à la page **Certificat de signature SAML** puis, dans la nouvelle ligne de certificat enregistrée, sélectionnez les points de suspension ( **...** ) et choisissez **Définir comme certificat actif**. L'état du nouveau certificat devient **Actif**, et le certificat précédemment actif devient **Inactif**.
1. Continuez à suivre les instructions de configuration de l’authentification SAML de l'application que vous avez affichées précédemment afin de charger le certificat de signature SAML au bon format d'encodage.

## <a name="add-email-notification-addresses-for-certificate-expiration"></a>Ajouter des adresses e-mail de notification d’expiration du certificat

Azure AD enverra un e-mail de notification 60, 30 et 7 jours avant expiration du certificat SAML. Vous pouvez ajouter plusieurs adresses e-mail pour recevoir des notifications. Pour spécifier les adresses e-mail auxquelles les notifications doivent être envoyées :

1. Dans la page **Certificat de signature SAML**, accédez à la rubrique **Adresses e-mail de notification**. Par défaut, cette rubrique utilise uniquement l’adresse e-mail de l’administrateur qui a ajouté l’application.
1. Sous l'adresse e-mail finale, tapez l'adresse e-mail qui devrait recevoir la notification d'expiration du certificat, puis appuyez sur Entrée.
1. Répétez l'étape précédente pour chaque adresse e-mail à ajouter.
1. Pour chaque adresse e-mail à supprimer, sélectionnez l'icône **Supprimer**(une poubelle) en regard de l'adresse e-mail.
1. Sélectionnez **Enregistrer**.

Vous recevez l’e-mail de notification de la part de aadnotification@microsoft.com. Pour éviter que l’e-mail ne soit traité comme un courrier indésirable, ajoutez cette adresse e-mail à vos contacts.

## <a name="renew-a-certificate-that-will-soon-expire"></a>Renouvellement d’un certificat arrivant à expiration

Si un certificat est sur le point d'expirer, vous pouvez le renouveler en utilisant une procédure qui n'entraîne aucun temps d'arrêt significatif pour vos utilisateurs. Pour renouveler un certificat sur le point d’expirer :

1. Suivez les instructions de la section [Créer un certificat](#create-a-new-certificate) précédente, en utilisant une date qui chevauche le certificat existant. Cette date limite le temps d'arrêt causé par l'expiration du certificat.
1. Si l’application peut automatiquement remplacer un certificat, activez le nouveau certificat en procédant comme suit :
   1. Retournez à la page **Certificat de signature SAML**.
   1. Dans la ligne de certificat nouvellement enregistrée, sélectionnez les points de suspension ( **...** ), puis choisissez **Définir comme certificat actif**.
   1. Ignorez les deux étapes suivantes.

1. Si l'application ne peut gérer qu'un seul certificat à la fois, choisissez un intervalle de temps d'arrêt pour effectuer l'étape suivante. (Sinon, si l'application ne sélectionne pas automatiquement le nouveau certificat mais peut gérer plusieurs certificats de signature, vous pouvez effectuer l'étape suivante à tout moment.)
1. Avant l'expiration de l'ancien certificat, suivez les instructions de la section [Charger et activer un certificat](#upload-and-activate-a-certificate) précédente.
1. Connectez-vous à l'application pour vérifier que le certificat fonctionne correctement.

## <a name="related-articles"></a>Articles connexes

- [Tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](../saas-apps/tutorial-list.md)
- [Gestion des applications avec Azure Active Directory](what-is-application-management.md)
- [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](what-is-single-sign-on.md)
- [Déboguer l’authentification unique SAML aux applications dans Azure Active Directory](./debug-saml-sso-issues.md)