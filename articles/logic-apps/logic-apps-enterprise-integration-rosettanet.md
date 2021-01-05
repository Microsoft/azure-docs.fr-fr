---
title: Messages RosettaNet pour l’intégration B2B
description: Échanger des messages RosettaNet dans Azure Logic Apps avec Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 06/22/2019
ms.openlocfilehash: f02cbdc7ca8822c5fcc91b106856d7f8f547536b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91565103"
---
# <a name="exchange-rosettanet-messages-for-b2b-enterprise-integration-in-azure-logic-apps"></a>Échanger des messages RosettaNet pour l’intégration d’entreprise B2B dans Azure Logic Apps

[RosettaNet](https://resources.gs1us.org) est un consortium à but non lucratif qui a établi un processus standard pour le partage des informations métier. Ces normes sont couramment utilisées dans les processus de chaîne d’approvisionnement et sont répandues dans les secteurs des semi-conducteurs, de l’électronique ainsi que de la logistique. Le consortium RosettaNet crée et gère des processus PIP (Partner Interface Process) qui fournissent des définitions de processus métier communes pour tous les échanges de messages RosettaNet. RosettaNet repose sur XML et définit des directives de messages, des interfaces pour les processus métier, ainsi que des cadres d’implémentations à des fins de communication entre les entreprises.

Dans [Azure Logic Apps](../logic-apps/logic-apps-overview.md), le connecteur RosettaNet vous permet de créer des solutions d’intégration qui prennent en charge les normes RosettaNet. Le connecteur est basé sur RosettaNet Implementation Framework (RNIF) version 2.0.01. RNIF est une infrastructure d’application réseau ouverte qui permet aux partenaires professionnels d’exécuter de façon collaborative des processus PIP RosettaNet. Cette infrastructure définit la structure des messages, la nécessité d’accusés de réception, le codage MIME (Multipurpose Internet Mail Extensions) et la signature numérique.

Le connecteur fournit plus précisément les fonctionnalités suivantes :

* Encoder ou recevoir des messages RosettaNet.
* Décoder ou envoyer des messages RosettaNet.
* Attendre la réponse et la génération d’une notification d’échec.

Pour ces fonctionnalités, le connecteur prend en charge tous les processus PIP définis par RNIF 2.0.01. La communication avec le partenaire peut être synchrone ou asynchrone.

## <a name="rosettanet-concepts"></a>Concepts RosettaNet

Voici certains concepts et termes propres à la spécification RosettaNet et qui sont importants lors de la génération d’intégrations basées sur RosettaNet :

* **PIP**

  L’organisation RosettaNet crée et gère des processus PIP (Partner Interface Process) qui fournissent des définitions de processus métier communes pour tous les échanges de messages RosettaNet. Chaque spécification PIP fournit un fichier de définition de type de document (DTD) et un document d’orientation de message. Le fichier DTD définit la structure des messages de contenu de service. Le document d’orientation de message, un fichier HTML contrôlable de visu, spécifie des contraintes au niveau de l’élément. Ensemble, ces fichiers fournissent une définition complète du processus d’entreprise.

   Les processus PIP sont classés par fonction d’entreprise de haut niveau ou cluster, et sous-fonction ou segment. Par exemple, « 3A4 » est le processus PIP lié au bon de commande, où « 3 » correspond à la fonction de gestion des commandes et « 3A » à la sous-fonction de saisie de devis et de commande. Pour plus d’informations, consultez le [site de RosettaNet](https://resources.gs1us.org).

* **Action**

  Dans le cadre d’un processus PIP, les messages d’action sont les messages professionnels échangés entre partenaires.

* **Signal**

   Dans le cadre d’un processus PIP, les messages de signal sont les accusés de réception envoyés en réponse aux messages d’action.

* **Action unique et double action**

  Dans le cadre d’un processus PIP à action unique, la seule réponse est un message de signal d’accusé de réception. Dans le cadre d’un processus PIP à double action, l’initiateur reçoit un message de réponse et y répond avec un accusé de réception en plus du flux de messages d’action unique.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Un [compte d'intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) pour stocker votre accord et d'autres artefacts B2B. Ce compte d'intégration doit être associé à votre abonnement Azure.

* Au moins deux [partenaires](../logic-apps/logic-apps-enterprise-integration-partners.md) définis dans votre compte d’intégration et configurés avec le qualificateur « DUNS » sous **Identités d’entreprise**

* Une configuration de processus PIP, qui est requise pour envoyer ou recevoir des messages RosettaNet, dans votre compte d’intégration. La configuration du processus stocke toutes les caractéristiques de configuration du processus PIP. Vous pouvez ensuite faire référence à cette configuration lorsque vous créez un contrat avec le partenaire. Pour créer une configuration de processus PIP dans votre compte d’intégration, consultez [Ajouter une configuration de processus PIP](#add-pip).

* Des [certificats](../logic-apps/logic-apps-enterprise-integration-certificates.md) facultatifs pour chiffrer, déchiffrer ou signer les messages que vous chargez sur le compte d’intégration. Des certificats ne sont requis que si vous utilisez la signature ou le chiffrement.

<a name="add-pip"></a>

## <a name="add-pip-process-configuration"></a>Ajouter une configuration de processus PIP

Pour ajouter une configuration de processus PIP à votre compte d’intégration, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com), recherchez et ouvrez votre compte d’intégration.

1. Sur le volet **Vue d’ensemble**, sélectionnez la vignette **RosettaNet PIP** (PIP RosettaNet).

   ![Choisir la vignette RosettaNet](media/logic-apps-enterprise-integration-rosettanet/select-rosettanet-tile.png)

1. Sous **RosettaNet PIP** (PIP RosettaNet), choisissez **Ajouter**. Fournissez les informations de votre PIP.

   ![Ajouter des détails du processus PIP RosettaNet](media/logic-apps-enterprise-integration-rosettanet/add-rosettanet-pip.png)

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Nom** | Oui | Nom de votre PIP |
   | **PIP Code** (Code de PIP) | Oui | Code à trois chiffres du processus PIP. Pour plus d’informations, consultez les [processus PIP RosettaNet](/biztalk/adapters-and-accelerators/accelerator-rosettanet/rosettanet-pips). |
   | **PIP Version** (Version de PIP) | Oui | Numéro de version du processus PIP, disponible en fonction de votre code de PIP sélectionné |
   ||||

   Pour plus d’informations sur ces propriétés de processus PIP, visitez le [site web de RosettaNet](https://resources.gs1us.org/RosettaNet-Standards/Standards-Library/PIP-Directory#1043208-pipsreg).

1. Lorsque vous avez terminé, choisissez **OK**. La configuration de processus PIP est créée.

1. Pour afficher ou modifier la configuration du processus, sélectionnez le PIP, puis choisissez **Edit as JSON** (Modifier en tant que JSON).

   Tous les paramètres de configuration du processus proviennent les spécifications du processus PIP. Logic Apps remplit la plupart des paramètres avec les valeurs par défaut, qui sont les valeurs généralement les plus utilisées pour ces propriétés.

   ![Modifier la configuration de processus PIP RosettaNet](media/logic-apps-enterprise-integration-rosettanet/edit-rosettanet-pip.png)

1. Vérifiez que les paramètres correspondent aux valeurs dans la spécification de processus PIP appropriée et répondent aux besoins de votre entreprise. Si nécessaire, mettez à jour les valeurs dans JSON et enregistrez ces modifications.

## <a name="create-rosettanet-agreement"></a>Créer un contrat RosettaNet

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre compte d’intégration, s’il n’est pas déjà ouvert.

1. Sur le volet **Vue d’ensemble**, sélectionnez la vignette **Contrats**.

   ![Choisir la vignette Contrats](media/logic-apps-enterprise-integration-rosettanet/select-agreement-tile.png)

1. Sous **Contrats**, choisissez **Ajouter**. Fournissez les détails de votre contrat.

   ![Ajouter des détails du contrat](media/logic-apps-enterprise-integration-rosettanet/add-agreement-details.png)

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Nom** | Oui | Nom du contrat |
   | **Type d'accord** | Oui | Sélectionnez **RosettaNet**. |
   | **Partenaire hôte** | Oui | Un contrat nécessite un partenaire hôte et un partenaire invité. Le partenaire hôte représente l’organisation qui configure le contrat. |
   | **Identité de l'hôte** | Oui | Identificateur du partenaire hôte. |
   | **Partenaire invité** | Oui | Un contrat nécessite un partenaire hôte et un partenaire invité. Le partenaire invité représente l’organisation qui travaille avec le partenaire hôte. |
   | **Identité de l'invité** | Oui | Identificateur du partenaire invité. |
   | **Paramètres de réception** | Variable | Ces propriétés s’appliquent à tous les messages reçus par le partenaire hôte |
   | **Paramètres d'envoi** | Variable | Ces propriétés s’appliquent à tous les messages envoyés par le partenaire hôte |  
   | **RosettaNet PIP references** (Références de processus PIP RosettaNet) | Oui | Références de processus PIP du contrat. Tous les messages RosettaNet nécessitent des configurations de processus PIP. |
   ||||

1. Pour configurer votre contrat pour recevoir des messages entrants du partenaire invité, sélectionnez **Paramètres de réception**.

   ![Paramètres de réception](media/logic-apps-enterprise-integration-rosettanet/add-agreement-receive-details.png)

   1. Pour activer la signature ou le chiffrement pour les messages entrants, sous **Messages**, sélectionnez **Le message doit être signé** ou **Le message doit être chiffré**, respectivement.

      | Propriété | Obligatoire | Description |
      |----------|----------|-------------|
      | **Le message doit être signé** | Non | Signez les messages entrants avec le certificat sélectionné. |
      | **Certificate** | Oui, si la signature est activée | Certificat à utiliser pour la signature |
      | **Activer le chiffrement des messages** | Non | Chiffrez les messages entrants avec le certificat sélectionné. |
      | **Certificate** | Oui, si le chiffrement est activé | Certificat à utiliser pour le chiffrement |
      ||||

   1. Sous chaque sélection, sélectionnez le [certificat](./logic-apps-enterprise-integration-certificates.md) respectif, que vous avez ajouté précédemment à votre compte d’intégration, à utiliser pour la signature ou le chiffrement.

1. Pour configurer votre contrat pour envoyer des messages au partenaire invité, sélectionnez **Paramètres d’envoi**.

   ![Paramètres d’envoi](media/logic-apps-enterprise-integration-rosettanet/add-agreement-send-details.png)

   1. Pour activer la signature ou chiffrement pour les messages sortants, sous **Messages**, sélectionnez **Enable message signing** (Activer la signature des messages) ou **Enable message encryption** (Activer le chiffrement des messages), respectivement. Sous chaque sélection, sélectionnez l’algorithme et le [certificat](./logic-apps-enterprise-integration-certificates.md) respectifs, que vous avez ajoutés précédemment à votre compte d’intégration, à utiliser pour la signature ou le chiffrement.

      | Propriété | Obligatoire | Description |
      |----------|----------|-------------|
      | **Activer la signature des messages** | Non | Signez les messages sortants avec l’algorithme de signature et le certificat sélectionnés. |
      | **Algorithme de signature** | Oui, si la signature est activée | Algorithme de signature à utiliser, basé sur le certificat sélectionné |
      | **Certificate** | Oui, si la signature est activée | Certificat à utiliser pour la signature |
      | **Activer le chiffrement des messages** | Non | Chiffrez les messages sortants avec l’algorithme de chiffrement et le certificat sélectionnés. |
      | **Algorithme de chiffrement** | Oui, si le chiffrement est activé | Algorithme de chiffrement à utiliser, basé sur le certificat sélectionné |
      | **Certificate** | Oui, si le chiffrement est activé | Certificat à utiliser pour le chiffrement |
      ||||

   1. Sous **Points de terminaison**, spécifiez les URL requises à utiliser pour envoyer des messages d’action et des accusés de réception.

      | Propriété | Obligatoire | Description |
      |----------|----------|-------------|
      | **URL d’action** |  Oui | URL à utiliser pour envoyer des messages d’action. L’URL est un champ obligatoire pour les messages synchrones et asynchrones. |
      | **Acknowledgment URL** (URL d’accusé de réception) | Oui | URL à utiliser pour envoyer des messages d’accusé de réception. L’URL est un champ obligatoire pour les messages synchrones. |
      ||||

1. Pour configurer votre contrat avec les références de processus PIP RosettaNet pour les partenaires, sélectionnez **RosettaNet PIP references** (Références de processus PIP RosettaNet). Sous **PIP Name** (Nom de PIP), sélectionnez le nom de votre processus PIP créé précédemment.

   ![Références de processus PIP](media/logic-apps-enterprise-integration-rosettanet/add-agreement-pip-details.png)

   Votre sélection remplit les propriétés restantes, qui sont basées sur le processus PIP que vous avez configuré dans votre compte d’intégration. Si nécessaire, vous pouvez modifier le **rôle du PIP**.

   ![PIP sélectionné](media/logic-apps-enterprise-integration-rosettanet/add-agreement-selected-pip.png)

Lorsque vous avez terminé ces étapes, vous êtes prêt à envoyer ou recevoir des messages RosettaNet.

## <a name="rosettanet-templates"></a>Modèles RosettaNet

Pour accélérer le développement et recommander des modèles d’intégration, vous pouvez utiliser des modèles d’application logique pour encoder et décoder des messages RosettaNet. Lorsque vous créez une application logique, vous pouvez sélectionner dans la galerie de modèles du concepteur d’application logique. Vous pouvez également retrouver ces modèles dans le [référentiel GitHub pour Azure Logic Apps](https://github.com/Azure/logicapps).

![Modèles RosettaNet](media/logic-apps-enterprise-integration-rosettanet/decode-encode-rosettanet-templates.png)

## <a name="receive-or-decode-rosettanet-messages"></a>Recevoir et décoder des messages RosettaNet

1. [Créez une application logique vide](quickstart-create-first-logic-app-workflow.md).

1. [Liez votre compte d’intégration](logic-apps-enterprise-integration-create-integration-account.md#link-account) à votre application logique.

1. Avant de pouvoir ajouter une action pour décoder le message RosettaNet, vous devez ajouter un déclencheur de démarrage de votre application logique, un déclencheur de requête par exemple.

1. Une fois le déclencheur ajouté, choisissez **Nouvelle étape**.

   ![Ajouter un déclencheur de requête](media/logic-apps-enterprise-integration-rosettanet/request-trigger.png)

1. Dans la zone de recherche, entrez « rosettanet », puis sélectionnez cette action : **RosettaNet Decode**

   ![Recherchez et sélectionnez l’action « RosettaNet Decode »](media/logic-apps-enterprise-integration-rosettanet/select-decode-rosettanet-action.png)

1. Entrez les informations pour les propriétés de l’action :

   ![Capture d'écran montrant où entrer les informations relatives aux propriétés de l'action.](media/logic-apps-enterprise-integration-rosettanet/decode-action-details.png)

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Message** | Oui | Message RosettaNet à décoder  |
   | **En-têtes** | Oui | En-têtes HTTP fournissant les valeurs de version, à savoir la version RNIF, et de type de réponse, qui indique le type de communication entre les partenaires et peut être synchrone ou asynchrone |
   | **Rôle** | Oui | Rôle du partenaire hôte dans le processus PIP |
   ||||

   Dans l’action RosettaNet Decode, la sortie, ainsi que d’autres propriétés, inclut **Outbound signal** (Signal sortant), que vous pouvez choisir d’encoder et renvoyer au partenaire, ou d’effectuer toute autre action sur cette sortie.

## <a name="send-or-encode-rosettanet-messages"></a>Envoyer et encoder des messages RosettaNet

1. [Créez une application logique vide](quickstart-create-first-logic-app-workflow.md).

1. [Liez votre compte d’intégration](logic-apps-enterprise-integration-create-integration-account.md#link-account) à votre application logique.

1. Avant de pouvoir ajouter une action pour encoder le message RosettaNet, vous devez ajouter un déclencheur de démarrage de votre application logique, un déclencheur de requête par exemple.

1. Une fois le déclencheur ajouté, choisissez **Nouvelle étape**.

   ![Ajouter un déclencheur de requête](media/logic-apps-enterprise-integration-rosettanet/request-trigger.png)

1. Dans la zone de recherche, entrez « rosettanet », puis sélectionnez cette action : **RosettaNet Encode**

   ![Recherchez et sélectionnez l’action « RosettaNet Encode »](media/logic-apps-enterprise-integration-rosettanet/select-encode-rosettanet-action.png)

1. Entrez les informations pour les propriétés de l’action :

   ![Fournir les détails de l’action](media/logic-apps-enterprise-integration-rosettanet/encode-action-details.png)

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Message** | Oui | Message RosettaNet à encoder  |
   | **Partenaire hôte** | Oui | Nom du partenaire hôte |
   | **Partenaire invité** | Oui | Nom du partenaire invité |
   | **PIP Code** (Code de PIP) | Oui | Code de processus PIP |
   | **PIP Version** (Version de PIP) | Oui | Version de processus PIP |  
   | **PIP instance identity** (Identité d’instance PIP) | Oui | Identificateur unique de ce message de processus PIP |  
   | **Type de message** | Oui | Type du message à encoder |  
   | **Rôle** | Oui | Rôle du partenaire hôte |
   ||||

   Le message encodé est maintenant prêt à envoyer au partenaire.

1. Pour envoyer le message encodé, cet exemple utilise l’action **HTTP**, qui est renommé « HTTP - Send encoded message to partner ».

   ![Action HTTP pour l’envoi de message RosettaNet](media/logic-apps-enterprise-integration-rosettanet/send-rosettanet-message-to-partner.png)

   Conformément aux normes RosettaNet, les transactions commerciales ne sont considérées comme terminées que lorsque toutes les étapes définies par le processus PIP sont terminées.

1. Une fois que l’hôte envoie le message encodé au partenaire, l’hôte attend le signal et l’accusé de réception. Pour effectuer cette tâche, ajoutez l’action **RosettaNet wait for response**.

   ![Ajouter une action « RosettaNet wait for response »](media/logic-apps-enterprise-integration-rosettanet/rosettanet-wait-for-response-action.png)

   La durée d’attente et le nombre de tentatives sont basés sur la configuration du processus PIP dans votre compte d’intégration. Si aucune réponse n’est reçue, cette action génère une notification d’échec. Pour gérer les nouvelles tentatives, placez toujours les actions **Encode** and **Wait for response** dans une boucle **Until**.

   ![Boucle Until avec des actions RosettaNet](media/logic-apps-enterprise-integration-rosettanet/rosettanet-loop.png)

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment valider, transformer et utiliser d’autres opérations message dans [Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md).
* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)
