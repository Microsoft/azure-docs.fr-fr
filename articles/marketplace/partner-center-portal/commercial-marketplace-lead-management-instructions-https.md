---
title: Gestion des prospects avec un point de terminaison HTTPS – Marketplace commercial de Microsoft
description: Découvrez comment utiliser Power Automate et un point de terminaison HTTPS pour gérer les prospects de Microsoft AppSource et de la Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: trkeya
ms.author: trkeya
ms.date: 03/30/2020
ms.openlocfilehash: b4dabee6a725fd02b7be6dd12c8a032e3d816835
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94489367"
---
# <a name="use-an-https-endpoint-to-manage-commercial-marketplace-leads"></a>Utiliser un point de terminaison HTTPS pour gérer les prospects de marketplaces commerciaux

Si votre système Gestion des relations avec la clientèle (CRM) n’est pas explicitement pris en charge dans l’Espace partenaires pour recevoir les prospects de la Place de marché Azure et de Microsoft AppSource, vous pouvez utiliser un point de terminaison HTTPS dans [Power Automate](https://powerapps.microsoft.com/automate-processes/) pour gérer ces prospects. Avec un point de terminaison HTTPS, les prospects de marketplaces commerciaux peuvent être envoyés sous forme de notification par e-mail ou être écrits dans un système CRM pris en charge par Power Automate.

Cet article explique comment créer un flux dans Power Automate pour générer l’URL POST HTTP que vous allez utiliser pour configurer des prospects dans l’Espace partenaires. Il comprend également les étapes permettant de tester votre flux à l’aide de [Postman](https://www.getpostman.com/downloads/).

>[!NOTE]
>Le connecteur Power Automate utilisé dans ces instructions nécessite un abonnement payant à Power Automate. Veillez à prendre cela en compte avant de configurer ce flux.

## <a name="create-a-flow-by-using-power-automate"></a>Créer un flux à l’aide de Power Automate

1. Ouvrez la page web [Power Automate](https://flow.microsoft.com/). Sélectionnez **Connexion**. Si vous n’avez pas encore de compte, sélectionnez **S’inscrire gratuitement** pour créer un compte Power Automate gratuit.

1. Connectez-vous et sélectionnez **Mes flux** dans le menu.

    ![Connexion à Mes flux](./media/commercial-marketplace-lead-management-instructions-https/my-flows-automated.png)

1. Sous **+ Nouveau** , sélectionnez **+ Instantané - à partir de zéro**.

    ![Mes flux + Automatisé – à partir de zéro](./media/commercial-marketplace-lead-management-instructions-https/https-myflows-create-fromblank.png)

1. Nommez votre flux, puis sous **Choisir comment déclencher ce flux** , sélectionnez **Lors de la réception d’une requête HTTP**.

    ![Bouton Ignorer dans la fenêtre Générer un flux automatisé](./media/commercial-marketplace-lead-management-instructions-https/https-myflows-pick-request-trigger.png)

1. Cliquez sur l’étape du flux pour la développer.

    ![Développer l’étape du flux](./media/commercial-marketplace-lead-management-instructions-https/expand-flow-step.png)

1. Utilisez l’une des méthodes suivantes pour configurer le **schéma JSON du corps de la demande**  :

    - Copiez le schéma JSON dans la zone de texte **Schéma JSON du corps de la demande**.
    - Sélectionnez **Utiliser l’exemple de charge utile pour générer le schéma**. Dans la zone de texte **Entrer ou coller un exemple de charge utile JSON** , collez l’exemple JSON. Sélectionnez **Terminé** pour créer le schéma.

    **Schéma JSON**

    ```JSON
    {
      "$schema": "https://json-schema.org/draft-04/schema#",
      "definitions": {},
      "id": "http://example.com/example.json",
      "properties": {
        "ActionCode": {
          "id": "/properties/ActionCode",
          "type": "string"
        },
        "OfferTitle": {
          "id": "/properties/OfferTitle",
          "type": "string"
        },
        "LeadSource": {
          "id": "/properties/LeadSource",
          "type": "string"
        },
        "UserDetails": {
          "id": "/properties/UserDetails",
          "properties": {
            "Company": {
              "id": "/properties/UserDetails/properties/Company",
              "type": "string"
            },
            "Country": {
              "id": "/properties/UserDetails/properties/Country",
              "type": "string"
            },
            "Email": {
              "id": "/properties/UserDetails/properties/Email",
              "type": "string"
            },
            "FirstName": {
              "id": "/properties/UserDetails/properties/FirstName",
              "type": "string"
            },
            "LastName": {
              "id": "/properties/UserDetails/properties/LastName",
              "type": "string"
            },
            "Phone": {
              "id": "/properties/UserDetails/properties/Phone",
              "type": "string"
            },
            "Title": {
              "id": "/properties/UserDetails/properties/Title",
              "type": "string"
            }
          },
          "type": "object"
        }
      },
      "type": "object"
    }
    ```

    **Exemple JSON**
    
    ```json
    {
      "UserDetails": {
        "FirstName": "Some",
        "LastName": "One",
        "Email": "someone@contoso.com",
        "Phone": "16175555555",
        "Country": "USA",
        "Company": "Contoso",
        "Title": "Esquire"
     },
      "LeadSource": "AzureMarketplace",
      "ActionCode": "INS",
      "OfferTitle": "Test Microsoft",
      "Description": "Test run through Power Automate"
    }
    ```

>[!NOTE]
>À ce stade de la configuration, vous pouvez choisir de vous connecter à un système CRM ou de configurer une notification par e-mail. Suivez les instructions restantes en fonction de votre choix.

### <a name="connect-to-a-crm-system"></a>Se connecter à un système CRM

1. Sélectionnez **+ Nouvelle étape**.
1. Choisissez le système CRM de votre choix en le recherchant dans **Rechercher parmi les actions et les connecteurs**. Sélectionnez-le sous l’onglet **Actions** avec l’action pour créer un nouvel enregistrement. La capture d’écran suivante montre **Créer un enregistrement (Dynamics 365)** comme exemple.

    ![Créer un enregistrement](./media/commercial-marketplace-lead-management-instructions-https/create-new-record.png)

1. Indiquez le **nom de l’organisation** associé au système CRM. Sélectionnez **Prospects** dans la liste déroulante **Nom de l’entité**.

    ![Sélection de prospects](./media/commercial-marketplace-lead-management-instructions-https/select-leads.png)

1. Power Automate affiche un formulaire dans lequel fournir des informations sur les prospects. Vous pouvez mapper des éléments de la demande d’entrée en choisissant d’ajouter du contenu dynamique. La capture d’écran suivante présente **OfferTitle** comme exemple.

    ![Ajout de contenu dynamique](./media/commercial-marketplace-lead-management-instructions-https/add-dynamic-content.png)

1. Mappez les champs de votre choix, puis sélectionnez **Enregistrer** pour enregistrer votre flux. Une URL HTTP POST est créée et accessible dans la fenêtre **Quand une requête HTTP arrive**. Copiez cette URL en utilisant le contrôle de copie, situé à droite de l’URL HTTP POST. Il est important d’utiliser le contrôle de copie pour ne manquer aucune partie de l’URL complète. Enregistrez cette URL, car vous en aurez besoin lors de la configuration de la gestion des prospects dans le portail de publication.

    ![Quand une requête HTTP est reçue](./media/commercial-marketplace-lead-management-instructions-https/when-http-request-received.png)

### <a name="set-up-email-notification"></a>Configurer une notification par e-mail

1. Maintenant que vous avez terminé le schéma JSON, sélectionnez **+ Nouvelle étape**.
1. Sous **Choisir une action** , sélectionnez **Actions**.
1. Sous l’onglet **Actions** , sélectionnez **Envoyer un e-mail (Office 365 Outlook)** .

    >[!NOTE]
    >Si vous souhaitez utiliser un autre fournisseur de messagerie, recherchez et sélectionnez **Envoyer une notification par e-mail (Courrier)** comme action à la place.

    ![Ajouter une action de messagerie](./media/commercial-marketplace-lead-management-instructions-https/https-request-received-send-email.png)

1. Dans la fenêtre **Envoyer un e-mail** , configurez les champs obligatoires suivants :

   - **À**  : entrez au moins une adresse e-mail valide à laquelle les prospects seront envoyés.
   - **Objet** : Power Automate vous donne la possibilité d’ajouter du contenu dynamique, comme **LeadSource** , illustré dans la capture d’écran suivante. Commencez par entrer un nom de champ. Ensuite, sélectionnez la liste de sélection du contenu dynamique dans la fenêtre contextuelle. 

        >[!NOTE] 
        > Lorsque vous ajoutez des noms de champs, vous pouvez faire suivre chaque nom d’un signe deux-points (:) puis sélectionner **Entrée** pour créer une nouvelle ligne. Une fois que vous avez ajouté vos noms de champs, vous pouvez ajouter chaque paramètre associé à partir de la liste de sélection dynamique.

        ![Ajouter une action de messagerie à l’aide de contenu dynamique](./media/commercial-marketplace-lead-management-instructions-https/add-email-using-dynamic-content.png)

   - **Corps** : dans la liste de sélection de contenu dynamique, ajoutez les informations souhaitées dans le corps de l’e-mail. Par exemple, utilisez LastName, FirstName, Email et Company. Quand vous aurez fini de configurer la notification par e-mail, elle ressemblera à l’exemple affiché dans la capture d’écran suivante.


       ![Exemple de notification par e-mail](./media/commercial-marketplace-lead-management-instructions-https/send-an-email.png)

1. Sélectionnez **Enregistrer** pour terminer votre flux. Une URL HTTP POST est créée et accessible dans la fenêtre **Quand une requête HTTP arrive**. Copiez cette URL en utilisant le contrôle de copie, situé à droite de l’URL HTTP POST. Il est important d’utiliser ce contrôle pour ne manquer aucune partie de l’URL complète. Enregistrez cette URL, car vous en aurez besoin lors de la configuration de la gestion des prospects dans le portail de publication.

   ![URL HTTP POST](./media/commercial-marketplace-lead-management-instructions-https/http-post-url.png)

### <a name="testing"></a>Test

Vous pouvez tester votre configuration à l’aide de [Postman](https://app.getpostman.com/app/download/win64). Un téléchargement en ligne de Postman est disponible pour Windows. 

1. Lancez Postman et sélectionnez **Nouveau** > **Requête** pour configurer votre outil de test. 

   ![Demande de configuration de votre outil de test](./media/commercial-marketplace-lead-management-instructions-https/postman-request.png)

1. Renseignez le formulaire **Enregistrer la requête** , puis enregistrez-le dans le dossier que vous avez créé.

   ![Formulaire Enregistrer la requête](./media/commercial-marketplace-lead-management-instructions-https/postman-save-to-test.png)

1. Sélectionnez **POST** dans la liste déroulante. 

   ![Tester mon flux](./media/commercial-marketplace-lead-management-instructions-https/test-my-flow.png)

1. Collez l’URL HTTP POST du flux que vous avez créé dans Power Automate, là où il est mentionné **Entrer l’URL de la demande**.

   ![Coller l’URL HTTP POST](./media/commercial-marketplace-lead-management-instructions-https/paste-http-post-url.png)

1. Retournez dans [Power Automate](https://flow.microsoft.com/). Recherchez le flux que vous avez créé pour envoyer des prospects en accédant à **Mes flux** à partir de la barre de menus Power Automate. Sélectionnez les points de suspension en regard du nom du flux pour afficher plus d’options et sélectionnez **Modifier**.


1. Sélectionnez **Tester** en haut à droite, puis **J’effectuerai l’action de déclenchement** , et enfin **Tester**. Vous verrez une indication en haut de l’écran indiquant que le test a démarré.

   ![Option J’effectuerai l’action de déclenchement](./media/commercial-marketplace-lead-management-instructions-https/test-flow-trigger-action.png)

1. Retournez dans votre application Postman et sélectionnez **Envoyer**.

   ![Bouton Envoyer](./media/commercial-marketplace-lead-management-instructions-https/postman-send.png)

1. Revenez à votre flux et vérifiez le résultat. Si tout fonctionne comme prévu, vous verrez un message indiquant que le flux a réussi.

   ![Vérifier les résultats](./media/commercial-marketplace-lead-management-instructions-https/my-flow-check-results.png)

1. Vous devez avoir également reçu un e-mail. Consultez votre messagerie. 

    >[!NOTE] 
    >Si vous ne voyez pas de message de test, vérifiez vos dossiers de courrier indésirable. Dans la capture d’écran ci-dessous, vous remarquerez les étiquettes de champ que vous avez ajoutées lors de la configuration de la notification par e-mail. S’il s’agissait d’un prospect réel généré à partir de votre offre, vous pourriez aussi voir les coordonnées réelles du prospect dans le corps et dans la ligne d’objet.

   ![E-mail reçu](./media/commercial-marketplace-lead-management-instructions-https/email-received.png)

## <a name="configure-your-offer-to-send-leads-to-the-https-endpoint"></a>Configuration de votre offre pour l’envoi des prospects au point de terminaison HTTPS

Lorsque vous êtes prêt à configurer les informations de gestion des prospects pour votre offre dans le portail de publication, procédez comme suit.

1. Connectez-vous à l’[Espace partenaires](https://partner.microsoft.com/dashboard/home).

1. Sélectionnez votre offre, puis accédez à l’onglet **Configuration de l’offre**.

1. Dans la section **Prospects** , sélectionnez **Se connecter**.

    :::image type="content" source="./media/commercial-marketplace-lead-management-instructions-https/customer-leads.png" alt-text="Prospects":::

1. Dans la fenêtre contextuelle **Détails de la connexion** , sélectionnez **Point de terminaison HTTPS** comme **Destination du prospect**. Collez l’URL HTTP POST à partir du flux que vous avez créé en suivant les étapes précédentes dans le champ **URL du point de terminaison HTTPS**.
    ![Connection details Contact email](./media/commercial-marketplace-lead-management-instructions-https/https-connection-details.png)

1. Sous **E-mail du contact** , entrez les adresses e-mail des personnes de votre société qui doivent recevoir des notifications par e-mail lors de la réception d’un nouveau prospect. Vous pouvez spécifier plusieurs e-mails en les séparant par un point-virgule.

1. Sélectionnez **OK**.

Pour vérifier que vous êtes bien connecté à une destination de prospect, sélectionnez le bouton **Valider**. En cas de réussite, un prospect de test se trouve dans la destination de prospect.

>[!NOTE] 
>Vous devez terminer la configuration du reste de l'offre et la publier avant de pouvoir recevoir des prospects pour cette offre.

Quand des prospects sont générés, Microsoft les envoie au flux. Les prospects sont routés vers le système CRM ou vers l’adresse e-mail que vous avez configurée.
