---
title: Stratégies de confidentialité et de sécurité des données pour les connecteurs Google
description: En savoir plus sur l’impact des stratégies de sécurité et de confidentialité Google sur les connecteurs Google, tels que Gmail, dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: divswa, logicappspm
ms.topic: conceptual
ms.date: 06/05/2020
ms.openlocfilehash: 2a5204be638f108b40e431b148c9cb97788c4a52
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91400755"
---
# <a name="data-security-and-privacy-policies-for-google-connectors-in-azure-logic-apps"></a>Stratégies de confidentialité et de sécurité des données pour les connecteurs Google dans Azure Logic Apps

À partir du **1er mai 2020**, les modifications apportées en raison des [stratégies de confidentialité et de sécurité des données](https://www.blog.google/technology/safety-security/project-strobe/) de Google peuvent perturber vos workflows d’application logique qui utilisent le [connecteur Gmail](/connectors/gmail/). Si vos applications logiques utilisent le connecteur Gmail avec un compte Gmail grand public (adresse e-mail qui se termine par @gmail.com ou @googlemail.com), vos applications logiques peuvent uniquement utiliser [des déclencheurs, des actions et des connecteurs approuvés par Google](#approved-connectors).

> [!NOTE]
> Si vos applications logiques utilisent le connecteur Gmail avec un compte professionnel G-suite (adresse e-mail avec un domaine personnalisé), vos applications logiques ne sont pas concernées et n’ont aucune restriction quant à l’utilisation du connecteur Gmail.

## <a name="affected-logic-apps"></a>Applications logiques concernées

Si vous avez des applications logiques qui utilisent le connecteur Gmail, vous recevrez un e-mail concernant les applications logiques potentiellement concernées. Toutefois, à partir du **15 juin 2020**, tous les workflows non conformes seront désactivés. Vous pouvez effectuer l’une des actions suivantes :

* Mettre à jour les applications logiques concernées en [suivant la procédure décrite dans cette rubrique](#update-affected-workflows). Vous devez créer une application cliente Google, qui fournit un ID client et une clé secrète client que vous utilisez pour l’authentification dans votre déclencheur ou action Gmail.

* Mettre à jour les applications logiques concernées afin qu’elles utilisent uniquement les [connecteurs approuvés par Google](#approved-connectors) avant de réactiver les applications logiques désactivées.

<a name="approved-connectors"></a>

## <a name="google-approved-connectors"></a>Connecteurs approuvés par Google

Dans le cadre de cette stratégie, lorsque vous utilisez un compte Gmail grand public, vous pouvez utiliser le connecteur Gmail uniquement avec certains services approuvés par Google, susceptibles d’être modifiés. Nos équipes d’ingénieurs continuent à travailler avec Google pour ajouter des services à cette liste. Pour l’instant, voici les déclencheurs, actions et connecteurs approuvés par Google que vous pouvez utiliser dans le même workflow d’application logique que le connecteur Gmail lorsque vous utilisez un compte Gmail grand public :

* Déclencheurs et actions intégrés à Logic Apps : Lot, contrôle, opérations de données, délai d’exécution, Fichier plat, liquid, requête, calendrier, variables et XML

  Les déclencheurs intégrés et les actions qui ne sont pas approuvés par Google, comme HTTP, Azure Functions, Azure Logic Apps et autres, rendent une application logique non conforme au connecteur Gmail, car celle-ci peut envoyer ou recevoir des données depuis n’importe où.

* Services Google : Gmail, Google Agenda, Google Contacts, Google Drive, Google Sheets et Google Tasks

* Services Microsoft approuvés : Dynamics 365, Excel Online, Microsoft Teams, Microsoft 365, OneDrive et SharePoint Online

* Connecteurs pour les sources de données gérées par le client : FTP, RSS, SFTP, SMTP et SQL Server

## <a name="non-compliant-examples"></a>Exemples non conformes

Voici quelques exemples qui utilisent le connecteur Gmail avec des déclencheurs et des actions intégrés ou des connecteurs managés qui ne sont pas approuvés par Google :

* Cette application logique utilise le connecteur Gmail avec le déclencheur intégré HTTP :

  ![Application logique non conforme : exemple 1](./media/connectors-google-data-security-privacy-policy/not-compliant-logic-app-1.png)
  
  L’application logique utilise également le connecteur Google Calendar, qui est approuvé.

* Cette application logique utilise le connecteur Gmail avec le connecteur Stockage Blob Azure :

  ![Application logique non conforme : exemple 2](./media/connectors-google-data-security-privacy-policy/not-compliant-logic-app-2.png)

* Cette application logique utilise le connecteur Gmail avec le connecteur Twitter :

  ![Application logique non conforme : exemple 3](./media/connectors-google-data-security-privacy-policy/not-compliant-logic-app-3.png)

Pour obtenir les informations les plus récentes, consultez la [documentation technique de référence du connecteur Gmail](/connectors/gmail/).

<a name="update-affected-workflows"></a>

## <a name="steps-for-affected-logic-apps"></a>Étapes pour les applications logiques concernées

Si vous devez utiliser le connecteur Gmail avec un compte Gmail grand public et des connecteurs non approuvés par Google dans une application logique, vous pouvez créer votre propre application Google pour une utilisation personnelle ou interne à votre entreprise. Pour ce scénario, voici la procédure générale que vous devez suivre :

1. Créez une application cliente Google à l’aide de la [console d’API Google](https://console.developers.google.com).

1. Dans votre connecteur Gmail, utilisez les valeurs de l’ID client et de la clé secrète client de votre application cliente Google.

Pour obtenir plus d’informations, consultez la [documentation technique de référence du connecteur Gmail](/connectors/gmail/#authentication-and-bring-your-own-application).

### <a name="create-google-client-app"></a>Créer une application cliente Google

Afin de configurer un projet pour votre application cliente, utilisez l’[Assistant de la console d’API Google](https://console.developers.google.com/start/api?id=gmail&credential=client_key) et suivez les instructions. Ou, pour obtenir des instructions détaillées, consultez les instructions de la [documentation technique de référence du connecteur Gmail](/connectors/gmail/#authentication-and-bring-your-own-application).

Lorsque vous avez terminé, votre écran ressemble à cet exemple, sauf que vous disposez de vos propres valeurs **ID client** et **Clé secrète client**, que vous utiliserez ultérieurement dans votre application logique.

![ID client et clé secrète client pour votre application cliente Google](./media/connectors-google-data-security-privacy-policy/google-api-console.png)

### <a name="use-client-app-settings-in-logic-app"></a>Utiliser les paramètres de l’application cliente dans l’application logique

Pour utiliser l’ID client et la clé secrète client de votre application cliente Google dans votre déclencheur ou action Gmail, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Si vous ajoutez une nouvelle action ou un nouveau déclencheur Gmail et que vous créez une connexion à partir de zéro, passez à l’étape suivante. Dans le cas contraire, dans le déclencheur ou l’action Gmail, sélectionnez **Modifier la connexion** > **Ajouter nouveau**, par exemple :

   ![Sélectionner « Modifier la connexion » > « Ajouter nouveau »](./media/connectors-google-data-security-privacy-policy/change-gmail-connection.png)

1. Fournissez vos informations de connexion, par exemple :

   ![Fournir les informations de connexion](./media/connectors-google-data-security-privacy-policy/authentication-type-bring-your-own.png)

   | Propriété | Valeur | Description |
   |----------|-------|-------------|
   | **Type d'authentification** | **Apportez votre propre application** | Spécifie que vous allez utiliser votre propre application cliente pour l’authentification |
   | **ID client** | <*ID client*> | ID client de votre application cliente Google |
   | **Clé secrète client** | <*client-secret*> | Clé secrète client de votre application cliente Google |
   ||||

1. Lorsque vous avez terminé, sélectionnez **Se connecter**.

   Une page apparaît pour afficher l’application cliente que vous avez créée. Si vous utilisez un compte Gmail grand public, vous pouvez obtenir une page qui indique que votre application cliente n’est pas vérifiée par Google et vous invite à autoriser d’abord l’accès à votre compte Google.

   ![Demande d’accès à votre compte Google](./media/connectors-google-data-security-privacy-policy/allow-access-authorized-domain.png)

1. Le cas échéant, sélectionnez **Autoriser**.

   Vous pouvez maintenant utiliser le connecteur Gmail sans restrictions dans votre application logique.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur le [connecteur Gmail](/connectors/gmail/)

