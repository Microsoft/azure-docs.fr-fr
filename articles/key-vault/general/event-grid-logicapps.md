---
title: Envoyer un e-mail en cas de changement de l’état Key Vault du secret
description: Guide d’utilisation de Logic Apps pour répondre aux changements apportés aux secrets Key Vault
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 11/11/2019
ms.author: mbaldwin
ms.openlocfilehash: 9c522d870a25b3df34ab6a0cf1c1e944a6462685
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013987"
---
# <a name="use-logic-apps-to-receive-email-about-status-changes-of-key-vault-secrets"></a>Utilisez Logic Apps pour recevoir un e-mail sur les changements d’état des secrets du coffre de clés

Dans ce guide, vous allez apprendre à répondre aux événements Azure Key Vault reçus via [Azure Event Grid](../../event-grid/index.yml) à l’aide d’[Azure Logic Apps](../../logic-apps/index.yml). Une fois que vous aurez fini de le parcourir, vous disposerez d’une application logique Azure configurée pour envoyer un e-mail de notification chaque fois qu’un secret est créé dans Azure Key Vault.

Pour obtenir une vue d’ensemble de l’intégration d’Azure Key Vault/Azure Event Grid, consultez [Supervision de Key Vault avec Azure Event Grid](event-grid-overview.md).

## <a name="prerequisites"></a>Prérequis

- Un compte e-mail d’un fournisseur d’e-mail pris en charge par Azure Logic Apps (par exemple Office 365 Outlook). Ce compte e-mail permet d’envoyer les notifications d’événements. Pour obtenir la liste complète des connecteurs d’application logique pris en charge, consultez la [Vue d’ensemble des connecteurs](/connectors).
- Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.
- Un coffre de clés dans votre abonnement Azure. Vous pouvez créer rapidement un coffre de clés en suivant les étapes décrites dans [Définir et récupérer un secret depuis Azure Key Vault à l’aide d’Azure CLI](../secrets/quick-create-cli.md).
- Event Grid inscrit en tant que fournisseur de ressources. Consultez [Inscriptions de fournisseurs de ressources](../../azure-resource-manager/management/resource-providers-and-types.md)

## <a name="create-a-logic-app-via-event-grid"></a>Créer une application logique via Event Grid

Tout d’abord, créez une application logique avec un gestionnaire Event Grid et abonnez-vous aux événements « SecretNewVersionCreated » d’Azure Key Vault.

Pour créer un abonnement Azure Event Grid, effectuez les étapes suivantes :

1. Dans le portail Azure, accédez à votre coffre de clés, sélectionnez **Événements > Démarrer**, puis cliquez sur **Logic Apps**

    
    ![Key Vault - Page des événements](../media/eventgrid-logicapps-kvsubs.png)

1. Dans le **Concepteur Logic Apps** validez la connexion, puis cliquez sur **Continuer** 
 
    ![Concepteur d’application logique - Connexion](../media/eventgrid-logicappdesigner1.png)

1. Dans l’écran **Quand un événement de ressource se produit**, effectuez ce qui suit :
    - Conservez les valeurs contenues dans **Abonnement** et **Nom de la ressource** en tant que valeurs par défaut.
    - Sélectionnez Select **Microsoft.KeyVault.vaults** pour **Type de ressource**.
    - Sélectionnez **Microsoft.KeyVault.SecretNewVersionCreated** pour **Élément de type d’événement - 1**.

    ![Concepteur d’application logique - Gestionnaire d’événements](../media/eventgrid-logicappdesigner2.png)

1. Sélectionnez **+ Nouvelle étape**. Une fenêtre s’ouvre pour vous permettre de choisir une action.
1. Recherchez **E-mail**. Selon votre fournisseur de messagerie, recherchez et sélectionnez le connecteur correspondant. Ce didacticiel utilise **Office 365 Outlook**. Les étapes pour les autres fournisseurs d’e-mail sont similaires.
1. Sélectionnez l’action **Envoyer un e-mail (V2)**.

   ![Concepteur d’application logique – Envoyer un e-mail](../media/eventgrid-logicappdesigner3.png)

1. Créez votre modèle d’e-mail :
    - **À :** entrez l’adresse e-mail à laquelle recevoir les e-mails de notification. Pour ce didacticiel, utilisez un compte de messagerie auquel vous pouvez accéder pour les tests.
    - **Objet** et **Corps** : écrivez le texte de votre e-mail. Sélectionnez Propriétés JSON dans l’outil de sélection pour inclure du contenu dynamique basé sur les données d’événement. Vous pouvez récupérer les données de l’événement via `@{triggerBody()?['Data']}`.

    Votre modèle d’e-mail peut ressembler à l’exemple suivant.

    ![Concepteur d’application logique – Corps de l’e-mail](../media/eventgrid-logicappdesigner4.png)

8. Cliquez sur **Enregistrer sous**.
9. Entrez un **nom** pour la nouvelle application logique, puis cliquez sur **Créer**.
    
    ![Concepteur d’application logique – Créer](../media/eventgrid-logicappdesigner5.png)

## <a name="test-and-verify"></a>Tester et vérifier

1.  Accédez à votre coffre de clés dans le portail Azure, puis sélectionnez **Événements > Abonnements aux événements**.  Vérifier la création d’un abonnement
    
    ![Concepteur d’application logique – Tester et vérifier](../media/eventgrid-logicapps-kvnewsubs.png)

1.  Accédez à votre coffre de clés, sélectionnez **Secrets**, puis **+ Générer/Importer**. Créez un secret à des fins de test, nommez la clé et conservez les autres paramètres avec leurs valeurs par défaut.

    ![Key Vault - Créer un secret](../media/eventgrid-logicapps-kv-create-secret.png)

1. Dans l’écran **Créer un secret**, indiquez un nom, une valeur, puis sélectionnez **Créer**.

Une fois le secret créé, un e-mail est reçu aux adresses configurées.

## <a name="next-steps"></a>Étapes suivantes

- Présentation : [Supervision de Key Vault avec Azure Event Grid](event-grid-overview.md)
- Procédure : [Routez les notifications du coffre de clés vers Azure Automation](event-grid-tutorial.md).
- [Schéma des événements Azure Event Grid pour Azure Key Vault](../../event-grid/event-schema-key-vault.md)
- Apprenez-en plus sur [Azure Event Grid](../../event-grid/index.yml).
- Apprenez-en plus sur [la fonctionnalité Logic Apps d’Azure App Service](../../logic-apps/index.yml).