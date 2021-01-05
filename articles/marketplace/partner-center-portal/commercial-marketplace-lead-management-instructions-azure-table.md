---
title: Gestion des prospects avec Stockage Table Azure – Marketplace commercial de Microsoft
description: Découvrez comment utiliser Stockage Table Azure pour configurer des prospects pour Microsoft AppSource et la Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: trkeya
ms.author: trkeya
ms.date: 08/25/2020
ms.openlocfilehash: 71b9c96c1855180106f7dfa9a31f0ee8b06ceb67
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347861"
---
# <a name="use-azure-table-storage-to-manage-commercial-marketplace-leads"></a>Utiliser Stockage Table Azure pour gérer les prospects de marketplaces commerciaux

Si votre système Gestion des relations avec la clientèle (CRM) n’est pas explicitement pris en charge dans l’Espace partenaires pour recevoir des prospects de Microsoft AppSource et de la Place de marché Azure, vous pouvez utiliser Stockage Table Azure pour gérer ces prospects. Vous pouvez ensuite décider d’exporter les données et de les importer dans votre système CRM. Cet article explique comment créer un compte de stockage Azure et une table sous ce compte. De plus, vous pouvez créer un flux à l’aide de Power Automate pour envoyer une notification par e-mail quand votre offre reçoit un prospect.

## <a name="configure-an-azure-storage-account"></a>Configurer un compte de stockage Azure

1. Si vous ne possédez pas de compte Azure, vous pouvez [créer un compte d’évaluation gratuit](https://azure.microsoft.com/pricing/free-trial/).
1. Une fois que votre compte Azure est actif, connectez-vous au [portail Microsoft Azure](https://portal.azure.com).
1. Dans le portail Microsoft Azure, créez un compte de stockage à l’aide de la procédure suivante :

    1. Dans la barre de menus de gauche, sélectionnez **+ Créer une ressource**. Le volet **Nouveau** s’affiche à droite.
    1. Sélectionnez **Stockage** dans le volet **Nouveau**. Une liste **Éléments proposés** apparaît à droite.
    1. Sélectionnez **Compte de stockage** pour commencer la création du compte. Suivez les instructions pour [créer un compte de stockage](../../storage/common/storage-account-create.md?tabs=azure-portal).

        :::image type="content" source="media/commercial-marketplace-lead-management-instructions-azure-table/azure-storage-create.png" alt-text="Procédure de création d'un compte de stockage Azure.":::

        Pour en savoir plus sur les comptes de stockage, consultez [Tutoriel de démarrage rapide](../../storage/index.yml). Pour plus d’informations sur la tarification du stockage, consultez [Tarification de Storage](https://azure.microsoft.com/pricing/details/storage/).

1. Attendez que votre compte de stockage soit approvisionné. Ce processus prend généralement quelques minutes.

## <a name="create-a-table-in-your-storage-account"></a>Créer une table dans votre compte de stockage

1. Sur la **page d'accueil** du portail Azure, sélectionnez **Voir toutes vos ressources** pour accéder à votre compte de stockage. Vous pouvez également sélectionner **Toutes les ressources** dans la barre de menu de gauche du portail Azure.

    :::image type="content" source="media/commercial-marketplace-lead-management-instructions-azure-table/azure-storage-access.png" alt-text="Accéder à votre compte de stockage Azure.":::

1. Dans le volet de votre compte de stockage, sélectionnez **Clés d’accès**, puis copiez la valeur **Chaîne de connexion** pour la clé. Enregistrez cette valeur, car il s’agit de la valeur de **Chaîne de connexion de compte de stockage** que vous devrez fournir dans le portail de publication afin de recevoir des prospects pour votre offre de la Place de marché Azure.

    Voici un exemple de chaîne de connexion.

    ```sql
    DefaultEndpointsProtocol=https;AccountName=myAccountName;AccountKey=myAccountKey;EndpointSuffix=core.screens.net
    ```

    :::image type="content" source="media/commercial-marketplace-lead-management-instructions-azure-table/azure-storage-keys.png" alt-text="Clé de stockage Azure.":::


1. Dans le volet de votre compte de stockage, sélectionnez **Tables**, puis **+ Table** pour créer une table. Entrez un nom pour votre table et sélectionnez **OK**. Enregistrez cette valeur car vous en aurez besoin si vous voulez configurer un flux pour qu’il reçoive des notifications par e-mail lors de la réception de prospects.

    ![Tables Azure](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-tables.png)

    Vous pouvez utiliser l’[Explorateur de stockage Microsoft Azure](https://www.storageexplorer.com) ou tout autre outil pour afficher de votre table de stockage. Vous pouvez également exporter les données de la table Azure.

## <a name="optional-use-power-automate-to-get-lead-notifications"></a>(Facultatif) Utiliser Power Automate pour obtenir des notifications de prospect

Vous pouvez utiliser [Power Automate](/flow/) pour automatiser les notifications chaque fois qu’un prospect est ajouté à votre table Stockage Azure. Si vous ne possédez pas encore de compte, vous pouvez [vous inscrire pour en obtenir un gratuitement](https://flow.microsoft.com/).

### <a name="lead-notification-example"></a>Exemple de notification de prospect

L’exemple crée un flux qui envoie automatiquement une notification par e-mail lorsqu’un prospect est ajouté à Stockage Table Azure. Cet exemple définit une récurrence pour envoyer toutes les heures des informations sur le prospect si le stockage de la table est mis à jour.

1. Connectez-vous à votre compte Power Automate.
1. Dans la barre de gauche, sélectionnez **Mes flux**.
1. Dans la barre supérieure, sélectionnez **+ Nouveau**.
1. Dans la liste déroulante, sélectionnez **+ Planifié entièrement**.

   ![Mes flux + Planifié entièrement](./media/commercial-marketplace-lead-management-instructions-azure-table/ms-flow-scheduled-from-blank.png)

1. Dans la fenêtre **Créer un flux planifié**, pour **Répéter tou(te)s les**, sélectionnez l’intervalle **1** et la fréquence **Heure**. Par ailleurs, attribuez un nom au flux si vous le souhaitez. Sélectionnez **Create** (Créer).

   >[!NOTE]
   >Même si cet exemple utilise un intervalle d’une heure, vous pouvez sélectionner l’intervalle et la fréquence les plus adaptés à vos besoins professionnels.

   ![Créer un flux planifié](./media/commercial-marketplace-lead-management-instructions-azure-table/build-scheduled-flow.png)

1. Sélectionnez **+ Nouvelle étape**.
1. Dans la fenêtre **Choisir une action**, recherchez **Obtenir l'heure passée**. Puis sous **Actions**, sélectionnez **Obtenir l'heure passée**.

   ![Choisir une action](./media/commercial-marketplace-lead-management-instructions-azure-table/choose-an-action.png)

1. Dans la fenêtre **Obtenir l’heure passée**, définissez l'**intervalle** sur **1**. Dans la liste déroulante **Unité de temps**, sélectionnez **Heure**.

    >[!IMPORTANT]
    >Assurez-vous que l’intervalle et l’unité de temps que vous avez envoyés à l’étape 8 correspondent à l’intervalle et à la fréquence que vous avez configurés pour la récurrence à l’étape 5.

    ![Définir l’intervalle Obtenir l'heure passée](./media/commercial-marketplace-lead-management-instructions-azure-table/ms-flow-getpast-time.png)

   >[!TIP]
   >Vous pouvez vérifier votre flux à tout moment pour confirmer que chaque étape est correctement configurée. Pour vérifier votre flux, sélectionnez **Vérificateur de flux** dans la barre de menus **Flux**.

   Dans les étapes suivantes, vous allez vous connecter à votre table et configurer la logique de traitement des nouveaux prospects.

1. Sélectionnez **+ Nouvelle étape**. Ensuite, recherchez **Obtenir des entités** dans la fenêtre **Choisir une action**.
1. Sous **Actions**, sélectionnez **Obtenir des entités (Stockage Table Azure)** .
1. Dans la fenêtre **Stockage Table Azure**, fournissez les informations dans les zones suivantes, puis sélectionnez **Créer** :

    * **Nom de la connexion** : donnez un nom significatif à la connexion que vous établissez entre ce flux et la table.
    * **Nom du compte de stockage** : indiquez le nom du compte de stockage pour votre table. Vous trouverez ce nom sur la page **Clés d’accès** du compte de stockage.
    * **Clé de stockage partagée** : indiquez la valeur de clé de votre compte de stockage pour votre table. Vous trouverez cette valeur sur la page **Clés d’accès** du compte de stockage.

      ![Fenêtre Stockage de table Azure](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-storage.png)

   Après avoir sélectionné **Créer**, la fenêtre **Obtenir des entités** apparaît. Sélectionnez **Afficher les options avancées** et remplissez les zones suivantes :

   * **Table**: sélectionnez le nom de votre table (dans [Créer une table](#create-a-table-in-your-storage-account)). L’image suivante montre l’invite quand la table `marketplaceleads` est sélectionnée pour cet exemple.

     ![Fenêtre Obtenir des entités](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-get-entities.png)

   * **Requête de filtre** : activez cette case à cocher, puis collez cette fonction dans la zone : `Timestamp gt datetime'@{body('Get_past_time')}'`

     ![Obtenir des entités, zone Requête de filtre](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-get-entities-filter-query.png)

1. Maintenant que vous avez effectué la configuration de la connexion à la table Azure, sélectionnez **Nouvelle étape** pour ajouter une condition afin de rechercher de nouveaux prospects dans la table Azure.

1. Dans la fenêtre **Choisir une action**, sélectionnez **Actions**. Puis sélectionnez **Contrôle de condition**.

    ![Choisir une fenêtre d’action](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-choose-an-action.png)

1. Dans la fenêtre **Condition**, sélectionnez **Choisissez une valeur**. Sélectionnez ensuite **Expression** dans la fenêtre contextuelle.

1. Collez `length(body('Get_entities')?['value'])` dans la zone **fx**. Sélectionnez **OK** pour ajouter cette fonction.

1. Pour terminer la configuration de la condition :
    1. Sélectionnez **est supérieur à** dans la liste déroulante.
    2. Entrez **0** comme valeur.

        ![Fenêtre Condition](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-condition.png)

   Dans les étapes suivantes, vous définissez l’action à exécuter selon le résultat de la condition :

   * Si la condition donne **If no**, vous n’avez rien à faire.
   * Si la condition affiche **If yes**, déclenchez une action pour vous connecter à votre compte professionnel ou scolaire afin d’envoyer un e-mail. 

1. Sélectionnez **Ajouter une action** sous **Si oui**.

    ![Fenêtre Condition, Si oui, ajouter une action](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-condition-if-yes.png)

1. Sélectionnez **Envoyer un e-mail (Office 365 Outlook)** .

    ![Fenêtre Condition, Si oui, envoyer un e-mail](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-condition-if-yes-send-email.png)

    >[!NOTE]
    >Pour utiliser un autre fournisseur de messagerie, sélectionnez **Envoyer une notification par e-mail** comme action à la place. Les instructions vous montrent comment effectuer la configuration avec Office 365 Outlook, mais elles sont similaires pour un autre fournisseur de courrier.

1. Dans la fenêtre Office 365 Outlook, fournissez les informations dans les zones suivantes :

    1. **À** : entrez l’adresse e-mail de toutes les personnes qui doivent recevoir cette notification.
    1. **Objet** : indiquez un objet pour l’e-mail. Par exemple, **Nouveaux prospects !**
    1. **Corps**: ajoutez le texte que vous souhaitez inclure dans chaque message électronique (facultatif) et collez-le dans `body('Get_entities')?['value']`.

    >[!NOTE]
    >Vous pouvez insérer d’autres points de données statiques ou dynamiques au corps de ce message électronique.

    ![Fenêtre Condition, Si oui, fenêtre Office 365 Outlook](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-condition-if-yes-outlook.png)

1. Sélectionnez **Enregistrer** pour enregistrer le flux. Power Automate teste automatiquement le flux pour y détecter d’éventuelles erreurs. Si aucune erreur n’est détectée, le flux s’exécute dès qu’il est enregistré.

L'image suivante montre à quoi doit ressembler le flux final.

![Exemple de flux final](./media/commercial-marketplace-lead-management-instructions-azure-table/final-flow.png)

### <a name="manage-your-flow"></a>Gérer votre flux

Il est facile de gérer un flux en cours d’exécution. Vous disposez d’un contrôle total sur votre flux. Par exemple, vous pouvez l’arrêter le modifier, afficher un historique des exécutions et obtenir des données analytiques. L’image suivante montre les options disponibles pour gérer un flux.

 ![Gérer un flux](./media/commercial-marketplace-lead-management-instructions-azure-table/ms-flow-manage-completed.png)

Le flux continue de s’exécuter jusqu'à ce que vous l’arrêtiez à l’aide de l’option **Désactiver le flux**.

Si vous ne recevez aucune notification de prospect par e-mail, cela signifie qu’aucun nouveau prospect n’a été ajouté à la table Azure. Si votre flux contient des erreurs, vous recevrez un e-mail similaire à celui de cet exemple.

 ![Notification par e-mail d’une erreur de flux](./media/commercial-marketplace-lead-management-instructions-azure-table/ms-flow-failure-note.png)

## <a name="configure-your-offer-to-send-leads-to-the-azure-table"></a>Configurer votre offre pour envoyer des prospects à la table Azure

Lorsque vous êtes prêt à configurer les informations de gestion des leads pour votre offre dans le portail de publication, procédez comme suit.

1. Accédez à la page **Configuration de l’offre** de votre offre.

1. Dans la section **Prospects**, sélectionnez **Se connecter**.

    :::image type="content" source="./media/commercial-marketplace-lead-management-instructions-azure-table/customer-leads.png" alt-text="Prospects":::

1. Dans la fenêtre contextuelle **Détails de la connexion**, sélectionnez **Table Azure** comme **Destination du prospect**. 
     ![Lead management, Connection details](./media/commercial-marketplace-lead-management-instructions-azure-table/connection-details.png)

1. Collez la chaîne de connexion du compte de stockage Azure que vous avez créé en suivant les étapes précédentes dans la zone **Chaîne de connexion du compte de stockage**.
     ![Lead management, Connection details storage account](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-connection-details.png)

1. **E-mail du contact** : fournissez les e-mails des personnes de votre société qui doivent recevoir des notifications par e-mail lors de la réception d’un nouveau prospect. Vous pouvez fournir plusieurs e-mails en les séparant par un point-virgule.

1. Sélectionnez **OK**.

Pour vérifier que vous êtes bien connecté à une destination de prospect, sélectionnez le bouton **Valider**. En cas de réussite, un prospect de test se trouve dans la destination de prospect.

>[!NOTE]
>Vous devez terminer la configuration du reste de l'offre et la publier avant de pouvoir recevoir des prospects pour cette offre.

Quand des prospects sont générés, Microsoft les envoie à la table Azure. Si vous avez configuré un flux, un message est également envoyé à l’adresse e-mail que vous avez configurée.
