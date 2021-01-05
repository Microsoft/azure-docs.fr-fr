---
title: Décoder des messages X12
description: Valider l’EDI et générer les acquittements avec le décodeur de messages X12 dans Azure Logic Apps avec Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 01/27/2017
ms.openlocfilehash: 918516a5629f8570d54c641ffc29f2367937266f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74792374"
---
# <a name="decode-x12-messages-in-azure-logic-apps-with-enterprise-integration-pack"></a>Décoder des messages X12 dans Azure Logic Apps avec Enterprise Integration Pack

Avec le connecteur de messages Decode X12, vous pouvez valider l’enveloppe en fonction d’un accord de partenariat commercial, valider l’EDI et les propriétés spécifiques du partenaire, fractionner des échanges en documents informatisés ou conserver les échanges entiers et générer des acquittements pour les transactions traitées. Pour utiliser ce connecteur, vous devez ajouter le connecteur à un déclencheur existant dans votre application logique.

## <a name="before-you-start"></a>Avant de commencer

Voici les éléments dont vous avez besoin :

* Un compte Azure (que vous pouvez [créer gratuitement)](https://azure.microsoft.com/free)
* Un [compte d’intégration](logic-apps-enterprise-integration-create-integration-account.md) déjà défini et associé à votre abonnement Azure. Vous devez disposer d’un compte d’intégration pour pouvoir utiliser le connecteur Decode X12 Message.
* Au moins deux [partenaires](logic-apps-enterprise-integration-partners.md) déjà définis dans votre compte d’intégration
* Un [contrat X12](logic-apps-enterprise-integration-x12.md) déjà défini dans votre compte d’intégration

## <a name="decode-x12-messages"></a>Décoder des messages X12

1. [Créez une application logique](quickstart-create-first-logic-app-workflow.md).

2. Le connecteur Decode X12 Message ne possède aucun déclencheur, ce qui signifie que vous devez ajouter un déclencheur pour le démarrage de votre application logique, par exemple un déclencheur de requête. Dans le concepteur d’applications logiques, ajoutez un déclencheur, puis ajoutez une action à votre application logique.

3.  Dans la zone de recherche, entrez le filtre « x12 ». Sélectionnez **X12 – Decode X12 Message**.
   
    ![Recherchez « x12 »](media/logic-apps-enterprise-integration-x12-decode/x12decodeimage1.png)  

3. Si vous n’avez pas encore créé de connexions à votre compte d’intégration, vous êtes invité à le faire à cette étape. Donnez un nom à votre connexion, puis sélectionnez le compte d’intégration auquel vous souhaitez vous connecter. 

    ![Fournir les détails de connexion de compte d’intégration](media/logic-apps-enterprise-integration-x12-decode/x12decodeimage4.png)

    Les propriétés marquées d’un astérisque sont obligatoires.

    | Propriété | Détails |
    | --- | --- |
    | Nom de connexion * |Entrez un nom pour votre connexion. |
    | Compte d’intégration * |Entrez un nom pour votre compte d’intégration. Vérifiez que votre compte d’intégration et votre application logique se trouvent dans le même emplacement Azure. |

5.  Lorsque vous avez terminé, les détails de votre connexion doivent apparaître tels qu’indiqués dans l’exemple suivant. Pour terminer la création de votre connexion, sélectionnez l’option **Créer**.
   
    ![détails de connexion de compte d’intégration](media/logic-apps-enterprise-integration-x12-decode/x12decodeimage5.png) 

6. Une fois votre connexion est créée, comme indiqué dans cet exemple, sélectionnez le message de fichier plat X12 à décoder.

    ![connexion de compte d’intégration créée](media/logic-apps-enterprise-integration-x12-decode/x12decodeimage6.png) 

    Par exemple :

    ![Sélectionner le message de fichier plat X12 à décoder](media/logic-apps-enterprise-integration-x12-decode/x12decodeimage7.png) 

   > [!NOTE]
   > Le contenu réel du message ou la charge utile pour le tableau de message, bon ou mauvais, est codé en base64. Ainsi, vous devez spécifier une expression qui traite ce contenu.
   > Voici un exemple qui traite le contenu en tant que XML que vous pouvez entrer en mode Code ou à l’aide du Générateur d’expressions dans le concepteur.
   > ``` json
   > "content": "@xml(base64ToBinary(item()?['Payload']))"
   > ```
   > ![Exemple de contenu](media/logic-apps-enterprise-integration-x12-decode/content-example.png)
   >


## <a name="x12-decode-details"></a>Informations sur X12 Decode

Le connecteur X12 Decode effectue les tâches suivantes :

* Valide l’enveloppe par rapport au contrat de partenariat commercial
* Valide l’EDI et les propriétés spécifiques au partenaire
  * Validation de la structure EDI et validation du schéma étendue
  * Validation de la structure de l’enveloppe d’échange.
  * Validation de schéma de l’enveloppe par rapport au schéma de contrôle.
  * Validation de schéma des éléments de données du document informatisé par rapport au schéma de message.
  * Validation EDI effectuée sur les éléments de données du document informatisé. 
* Vérifie que les numéros de contrôle de l’échange, du groupe et du document informatisé ne sont pas des doublons.
  * Vérifie le numéro de contrôle de l’échange par rapport aux échanges reçus précédemment.
  * Vérifie le numéro de contrôle du groupe par rapport aux autres numéros de contrôle de groupe dans l’échange.
  * Vérifie le numéro de contrôle du document informatisé par rapport aux autres numéros de contrôle de document informatisé dans ce groupe.
* Fractionne l’échange en documents informatisés ou conserve l’échange entier :
  * Scinder l’échange en documents informatisés : suspendre les documents informatisés en cas d’erreur : fractionne l’échange en documents informatisés et analyse chaque document informatisé. 
  L’action X12 Decode ne génère que les documents informatisés qui échouent à la validation `badMessages` et produit les documents informatisés restants en tant que `goodMessages`.
  * Scinder l’échange en documents informatisés : suspendre l’échange en cas d’erreur : fractionne l’échange en documents informatisés et analyse chaque document informatisé. 
  Si la validation d’un ou de plusieurs documents informatisés de l’échange échoue, l’action X12 Decode génère tous les documents informatisés dans cet échange en tant que `badMessages`.
  * Préserver l'échange : suspendre les documents informatisés en cas d'erreur : conserve l’échange et traite l’intégralité de l’échange par lot. 
  L’action X12 Decode ne génère que les documents informatisés qui échouent à la validation `badMessages` et produit les documents informatisés restants en tant que `goodMessages`.
  * Préserver l’échange : suspendre l’échange en cas d’erreur : conserve l’échange et traite l’intégralité de l’échange par lot. 
  Si la validation d’un ou de plusieurs documents informatisés de l’échange échoue, l’action X12 Decode génère tous les documents informatisés dans cet échange en tant que `badMessages`. 
* Génère un accusé de réception fonctionnel et/ou technique (si configuré).
  * Suite à la validation de l’en-tête, un accusé de réception technique est généré. L’accusé de réception technique renvoie l’état du traitement de l’en-tête et du code de fin d’un échange par le récepteur de l’adresse.
  * Suite à la validation du corps, un accusé de réception fonctionnel est généré. L’accusé de réception fonctionnel signale chaque erreur rencontrée lors du traitement du document reçu

## <a name="view-the-swagger"></a>Afficher le swagger
Consultez les [détails sur Swagger](/connectors/x12/). 

## <a name="next-steps"></a>Étapes suivantes
[En savoir plus sur Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md "En savoir plus sur Enterprise Integration Pack") 

