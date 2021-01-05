---
title: Envoyer et recevoir des messages AS2 pour l’intégration B2B
description: Échanger des messages AS2 dans le cadre d’une intégration d’entreprise B2B dans Azure Logic Apps avec Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 10/08/2020
ms.openlocfilehash: 1845de8332287f85e96b4e29c40caa66cb8704d9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91856668"
---
# <a name="exchange-as2-messages-for-b2b-enterprise-integration-in-azure-logic-apps-with-enterprise-integration-pack"></a>Échanger des messages AS2 dans le cadre d’une intégration d’entreprise B2B dans Azure Logic Apps avec Enterprise Integration Pack

> [!IMPORTANT]
> Le connecteur AS2 d’origine est déconseillé. Veillez à utiliser le connecteur **AS2 (v2)** à la place, sauf si vous avez besoin de capacités de suivi. À l’exception du suivi, le connecteur v2 offre les mêmes capacités que la version d’origine, est natif du runtime Logic Apps et apporte d’importantes améliorations des performances en ce qui concerne le débit, la taille et la latence des messages. De plus, le connecteur v2 ne nécessite pas la création d’une connexion à votre compte d’intégration. Comme indiqué dans les prérequis; assurez-vous plutôt de lier votre compte d’intégration à l’application logique dans laquelle vous envisagez d’utiliser le connecteur.

Pour utiliser des messages AS2 dans Azure Logic Apps, vous pouvez vous servir du connecteur AS2, qui fournit des déclencheurs et des actions qui prennent en charge et gèrent la communication AS2 (version 1.2). Par exemple, pour garantir la sécurité et la fiabilité lors de la transmission des messages, vous pouvez utiliser ces actions :

* [**Action** Encodage AS2](#encode) pour fournir les fonctionnalités de chiffrement, de signature numérique et d’accusés de réception par notification de réception du message (MDN), ce qui renforce la non-répudiation. Par exemple, cette action applique des en-têtes AS2/HTTP et effectue les tâches suivantes lorsqu’elles sont configurées :

  * Signature des messages sortants.
  * Chiffrement des messages sortants.
  * Compression du message.
  * Transmission du nom de fichier dans l’en-tête MIME.

* [**Action** Décodage AS2](#decode) pour fournir les fonctionnalités de déchiffrement, de signature numérique et d’accusés de réception par notification de réception du message (MDN). Par exemple, cette action effectue les tâches suivantes :

  * Traitement des en-têtes AS2/HTTP.
  * Rapprochement des MDN reçues avec le message sortant d’origine.
  * Mise à jour et mise en corrélation les enregistrements dans la base de données de non-répudiation.
  * Écriture des enregistrements pour le rapport d’état AS2.
  * Génération des contenus de charge codés en base64.
  * Détermination du caractère obligatoire des MDN. Selon le contrat AS2, détermination du caractère synchrone ou asynchrone des MDN.
  * Génération des MDN synchrones ou asynchrones en fonction du contrat AS2.
  * Définition des propriétés et des jetons de corrélation sur les MDN.

  Cette action effectue également les tâches suivantes lorsqu’elles sont configurées :

  * Vérification de la signature.
  * Déchiffrement des messages.
  * Décompression du message.
  * Vérification et interdiction des doublons d’ID de message.

Cet article explique comment ajouter des actions de codage et de décodage AS2 à une application logique existante.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* L’application logique à partir de laquelle vous souhaitez utiliser le connecteur AS2 et un déclencheur qui démarre le flux de travail de votre application logique. Le connecteur AS2 fournit uniquement des actions, pas des déclencheurs. Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md) et [Démarrage rapide : Créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

* Un [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) associé à votre abonnement Azure et lié à l’application logique dans laquelle vous prévoyez d’utiliser le connecteur AS2. Votre application logique et votre compte d’intégration doivent tous deux exister dans le même emplacement ou dans la même région Azure.

* Au moins deux [partenaires commerciaux](../logic-apps/logic-apps-enterprise-integration-partners.md) déjà définis dans votre compte d’intégration à l’aide du qualificateur d’identité AS2.

* Pour pouvoir utiliser le connecteur AS2, vous devez créer un [contrat](../logic-apps/logic-apps-enterprise-integration-agreements.md) AS2 entre vos partenaires commerciaux et le stocker dans votre compte d’intégration.

* Si vous utilisez **Azure Key Vault** pour la gestion des certificats, vérifiez que les clés de coffre autorisent les opérations [Chiffrer](../key-vault/general/overview.md) et **Déchiffrer**. Sinon, le codage et le décodage échouent.

  Dans le portail Azure, accédez à la clé de votre coffre de clés, examinez les **Opérations autorisées** et vérifiez que les actions **Chiffrer** et **Déchiffrer** sont sélectionnées. Par exemple :

  ![Vérifier les opérations sur les clés de coffre](media/logic-apps-enterprise-integration-as2/key-vault-permitted-operations.png)

<a name="encode"></a>

## <a name="encode-as2-messages"></a>Encoder des messages AS2

1. Si ce n’est déjà fait, dans le [Portail Microsoft Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’application logique.

1. Dans le concepteur, ajoutez une nouvelle action à votre application logique.

1. Sous **Choisir une action** et la zone de recherche, sélectionnez **Tous**. Dans la zone de recherche, entrez « encodage AS2 » et assurez-vous de sélectionner l’action AS2 (v2) : **Encodage AS2**

   ![Sélectionnez « encodage AS2 »](./media/logic-apps-enterprise-integration-as2/select-as2-encode.png)

1. Maintenant, fournissez des informations pour les propriétés suivantes :

   | Propriété | Description |
   |----------|-------------|
   | **Message à encoder** | Charge utile du message |
   | **AS2 à partir de** | Identificateur de l’expéditeur du message tel que spécifié par votre contrat AS2 |
   | **AS2 vers** | Identificateur du destinataire du message tel que spécifié par votre contrat AS2 |
   |||

   Par exemple :

   ![Propriétés de codage de message](./media/logic-apps-enterprise-integration-as2/as2-message-encoding-details.png)

> [!TIP]
> Si vous rencontrez des problèmes lors de l’envoi de messages signés ou chiffrés, pensez à essayer différents formats de l’algorithme SHA256. Comme la spécification AS2 ne fournit pas d’informations sur les formats SHA256, chaque fournisseur utilise sa propre implémentation ou son propre format.

<a name="decode"></a>

## <a name="decode-as2-messages"></a>Décoder des messages AS2

1. Si ce n’est déjà fait, dans le [Portail Microsoft Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’application logique.

1. Dans le concepteur, ajoutez une nouvelle action à votre application logique.

1. Sous **Choisir une action** et la zone de recherche, sélectionnez **Tous**. Dans la zone de recherche, entrez « décodage AS2 » et assurez-vous de sélectionner l’action AS2 (v2) : **Décodage AS2**

   ![Sélectionnez « décodage AS2 »](media/logic-apps-enterprise-integration-as2/select-as2-decode.png)

1. Pour les propriétés du **Message à encoder** et les **En-têtes de message**, sélectionnez ces valeurs à partir de sorties de déclencheur ou d’action précédentes.

   Par exemple, supposons que votre application logique reçoive des messages via un déclencheur de requête. Vous pouvez sélectionner les sorties de ce déclencheur.

   ![Sélectionnez le corps et les en-têtes à partir des sorties de requête.](media/logic-apps-enterprise-integration-as2/as2-message-decoding-details.png)

## <a name="sample"></a>Exemple

Pour déployer une application logique totalement opérationnelle dans le cadre d’un exemple de scénario AS2, consultez le [scénario et le modèle d’application logique AS2](https://azure.microsoft.com/documentation/templates/201-logic-app-as2-send-receive/).

## <a name="connector-reference"></a>Référence de connecteur

Pour plus d’informations techniques sur ce connecteur, notamment au sujet des actions et des limites décrites dans le fichier Swagger du connecteur, consultez la [page de référence du connecteur](/connectors/as2/). 

> [!NOTE]
> Pour les applications logiques utilisées dans un [environnement de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la version de ce connecteur avec l’étiquette ISE applique les [limites de messages B2B de l’ISE](../logic-apps/logic-apps-limits-and-config.md#b2b-protocol-limits).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)
