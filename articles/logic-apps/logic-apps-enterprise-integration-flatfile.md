---
title: Encoder ou décoder des fichiers plats
description: Encoder ou décoder des fichiers plats pour l’intégration d’entreprise dans Azure Logic Apps avec Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 05/09/2020
ms.openlocfilehash: aebce8f284ed4bb21d99efffc8dd6d0c51b39533
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87001483"
---
# <a name="encode-and-decode-flat-files-in-azure-logic-apps-by-using-the-enterprise-integration-pack"></a>Encoder et décoder des fichiers plats dans Azure Logic Apps avec Enterprise Integration Pack

Avant de pouvoir envoyer du contenu XML à un partenaire commercial dans un scénario B2B, vous souhaitez peut-être d’abord encoder ce contenu. Avec une application logique, vous pouvez encoder et décoder des fichiers plats à l’aide du connecteur **Fichiers plats**. Votre application logique peut obtenir ce contenu XML de différentes sources, comme le déclencheur Requête, une autre application ou d’autres [connecteurs pris en charge par Azure Logic Apps](../connectors/apis-list.md). Pour plus d’informations, consultez la page [Présentation d’Azure Logic Apps](logic-apps-overview.md).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/).

* L’application logique dans laquelle vous allez utilisez le connecteur **Fichier plat** et un déclencheur qui démarre le flux de travail de votre application logique. Le connecteur **Fichier plat** fournit uniquement des actions, mais pas de déclencheurs. Vous pouvez utiliser le déclencheur ou une autre action pour transmettre le contenu XML dans votre application logique pour l’encoder ou le décoder. Si vous débutez avec les applications logiques, consultez le guide de [Démarrage rapide : Créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

* Un [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) associé à votre abonnement Azure et [lié à l’application logique](./logic-apps-enterprise-integration-create-integration-account.md#link-account) dans laquelle vous prévoyez d’utiliser le connecteur **Fichier plat**. Votre application logique et votre compte d’intégration doivent tous deux exister dans le même emplacement ou dans la même région Azure.

* Un [schéma](logic-apps-enterprise-integration-schemas.md) de fichier plat que vous avez chargé dans votre compte d’intégration pour encoder ou décoder le contenu XML

* Au moins deux [parties](logic-apps-enterprise-integration-partners.md) que vous avez déjà définies dans votre compte d’intégration

## <a name="add-flat-file-encode-action"></a>Ajouter l’action d’encodage de fichier plat

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Dans votre application logique, sous le déclencheur ou l’action, choisissez **Nouvelle étape** > **Ajouter une action**. Cet exemple utilise le déclencheur Requête, appelé **Réception d’une requête HTTP**, et gère les requêtes entrantes provenant de l’extérieur de l’application logique.

   > [!TIP]
   > Le schéma JSON est facultatif. Si vous avez un exemple de charge utile de la requête entrante, sélectionnez **Utiliser l’exemple de charge utile pour générer le schéma**, entrez l’exemple de charge utile, puis sélectionnez **Terminé**. Le schéma figure dans la zone **Schéma JSON du corps de la requête**.

1. Sous **Choisir une action**, entrez `flat file`. Dans la liste des actions, sélectionnez cette action : **Encodage de fichier plat**

   ![Sélectionner l’action « Encodage de fichier plat »](./media/logic-apps-enterprise-integration-flatfile/select-flat-file-encoding.png)

1. Cliquez dans la zone **Contenu** pour afficher la liste du contenu dynamique. Dans la liste, dans la section **Réception d’une requête HTTP**, sélectionnez la propriété **Corps**, qui contient la sortie du corps de la requête du déclencheur et le contenu à encoder.

   ![Sélectionner le contenu à encoder dans la liste de contenu dynamique](./media/logic-apps-enterprise-integration-flatfile/select-content-to-encode.png)

   > [!TIP]
   > Si vous ne voyez pas la propriété **Corps** dans la liste de contenu dynamique, sélectionnez **Voir plus** en regard de l’étiquette de section **Réception d’une requête HTTP**.
   > Vous pouvez également entrer directement le contenu à décoder dans la zone **Contenu**.

1. Dans la liste **Nom du schéma**, sélectionnez le schéma qui se trouve dans votre compte d’intégration lié à utiliser pour l’encodage, par exemple :

   ![Sélectionner le schéma à utiliser pour l’encodage](./media/logic-apps-enterprise-integration-flatfile/select-schema-for-encoding.png)

   > [!NOTE]
   > Si aucun schéma n’apparaît dans la liste, votre compte d’intégration ne contient aucun fichier de schéma à utiliser pour l’encodage. Chargez le schéma que vous voulez utiliser dans votre compte d’intégration.

1. Enregistrez votre application logique. Pour tester votre connecteur, effectuez une requête au point de terminaison HTTPs, qui apparaît dans la propriété **HTTP POST URL** du déclencheur Requête et ajoutez le contenu XML que vous souhaitez encoder dans le corps de la requête.

Vous avez maintenant terminé la configuration de votre action d’encodage de fichier plat. Dans une application réelle, vous souhaiterez peut-être stocker les données encodées dans une application métier telle que Salesforce. Vous pouvez également envoyer ces données encodées à une partie. Pour envoyer la sortie de l’action d’encodage à Salesforce ou à votre partie, utilisez les autres [connecteurs disponibles dans Azure Logic Apps](../connectors/apis-list.md).

## <a name="add-flat-file-decode-action"></a>Ajouter l’action de décodage de fichier plat

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Dans votre application logique, sous le déclencheur ou l’action, choisissez **Nouvelle étape** > **Ajouter une action**. Cet exemple utilise le déclencheur Requête, appelé **Réception d’une requête HTTP**, et gère les requêtes entrantes provenant de l’extérieur de l’application logique.

   > [!TIP]
   > Le schéma JSON est facultatif. Si vous avez un exemple de charge utile de la requête entrante, sélectionnez **Utiliser l’exemple de charge utile pour générer le schéma**, entrez l’exemple de charge utile, puis sélectionnez **Terminé**. Le schéma figure dans la zone **Schéma JSON du corps de la requête**.

1. Sous **Choisir une action**, entrez `flat file`. Dans la liste des actions, sélectionnez cette action : **Décodage de fichier plat**

   ![Sélectionner l’action « Décodage de fichier plat »](./media/logic-apps-enterprise-integration-flatfile/select-flat-file-decoding.png)

1. Cliquez dans la zone **Contenu** pour afficher la liste du contenu dynamique. Dans la liste, dans la section **Réception d’une requête HTTP**, sélectionnez la propriété **Corps**, qui contient la sortie du corps de la requête du déclencheur et le contenu à décoder.

   ![Sélectionner le contenu à décoder à partir de la liste de contenu dynamique](./media/logic-apps-enterprise-integration-flatfile/select-content-to-decode.png)

   > [!TIP]
   > Si vous ne voyez pas la propriété **Corps** dans la liste de contenu dynamique, sélectionnez **Voir plus** en regard de l’étiquette de section **Réception d’une requête HTTP**. Vous pouvez également entrer directement le contenu à décoder dans la zone **Contenu**.

1. Dans la liste **Nom du schéma**, sélectionnez le schéma qui se trouve dans votre compte d’intégration lié à utiliser pour le décodage, par exemple :

   ![Sélectionner le schéma à utiliser pour le décodage](./media/logic-apps-enterprise-integration-flatfile/select-schema-for-decoding.png)

   > [!NOTE]
   > Si aucun schéma n’apparaît dans la liste, votre compte d’intégration ne contient aucun fichier de schéma à utiliser pour le décodage. Chargez le schéma que vous voulez utiliser dans votre compte d’intégration.

1. Enregistrez votre application logique. Pour tester votre connecteur, effectuez une requête au point de terminaison HTTPs, qui apparaît dans la propriété **HTTP POST URL** du déclencheur Requête et ajoutez le contenu XML que vous souhaitez décoder dans le corps de la requête.

Vous avez maintenant terminé la configuration de votre action de décodage de fichier plat. Dans une application réelle, vous souhaiterez peut-être stocker les données décodées dans une application métier telle que Salesforce. Vous pouvez également envoyer les données décodées à une partie. Pour envoyer la sortie de l’action de décodage à Salesforce ou à votre partie, utilisez les autres [connecteurs disponibles dans Azure Logic Apps](../connectors/apis-list.md).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [Enterprise Integration Pack](logic-apps-enterprise-integration-overview.md)
