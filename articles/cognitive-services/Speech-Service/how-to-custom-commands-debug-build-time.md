---
title: Déboguer les erreurs lors de la création d’une application Commandes personnalisées (préversion)
titleSuffix: Azure Cognitive Services
description: Dans cet article, vous apprendrez à déboguer les erreurs lors de la création d’une application Commandes personnalisées.
services: cognitive-services
author: xiaojul
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: xiaojul
ms.openlocfilehash: aeb90e8e064c44f4d17f920261ed58310f0e55f0
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025699"
---
# <a name="debug-errors-when-authoring-a-custom-commands-application"></a>Déboguer les erreurs lors de la création d’une application Commandes personnalisées

Cet article explique comment déboguer les erreurs qui apparaissent lors de la création d’une application Commandes personnalisées. 

## <a name="errors-when-creating-an-application"></a>Erreurs lors de la création d’une application
Les commandes personnalisées créent également une application dans [LUIS](https://www.luis.ai/) lors de la génération d’une application Commandes personnalisée. 

[LUIS comporte une limite de 500 applications par ressource de création](../luis/luis-limits.md). La création d’une application LUIS peut échouer si vous utilisez une ressource de création qui possède déjà 500 applications. 

Assurez-vous que la ressource de création LUIS sélectionnée possède moins de 500 applications. Si ce n’est pas le cas, vous pouvez créer une ressource de création LUIS, basculer vers une autre ressource, ou essayer de nettoyer vos applications LUIS.  

## <a name="errors-when-deleting-an-application"></a>Erreurs lors de la suppression d’une application
### <a name="cant-delete-luis-application"></a>Impossible de supprimer l’application LUIS
Lors de la suppression d’une application Commandes personnalisées, les commandes personnalisées peuvent également tenter de supprimer l’application LUIS associée à l’application Commandes personnalisées.

Si la suppression de l’application LUIS échoue, accédez à votre compte [LUIS](https://www.luis.ai/) pour la supprimer manuellement.

### <a name="toomanyrequests"></a>TooManyRequests
Lorsque vous tenterez de supprimer un grand nombre d’applications en même temps, des erreurs « TooManyRequests » s’afficheront probablement. Ces messages d’erreur signifient que vos demandes de suppression sont limitées par Azure. 

Actualisez votre page et essayez de supprimer moins d’applications.

## <a name="errors-when-modifying-an-application"></a>Erreurs lors de la modification d’une application

### <a name="cant-delete-a-parameter-or-a-web-endpoint"></a>Impossible de supprimer un paramètre ou un point de terminaison Web
Vous n’êtes pas autorisé à supprimer un paramètre lorsqu’il est utilisé. Supprimez toute référence du paramètre dans les réponses vocales, les exemples de phrases, les conditions, les actions, puis réessayez.

### <a name="cant-delete-a-web-endpoint"></a>Impossible de supprimer un point de terminaison Web
Vous n’êtes pas autorisé à supprimer un point de terminaison Web lorsqu’il est utilisé. Supprimez toute action **Appeler le point de terminaison Web** qui utilise ce point de terminaison Web avant de supprimer un point de terminaison Web.

## <a name="errors-when-training-an-application"></a>Erreurs lors de l’apprentissage d’une application
### <a name="built-in-intents"></a>Intentions intégrées
LUIS intègre des intentions Oui/Non. L’apprentissage échoue si les exemples de phrases contiennent uniquement « Oui » ou « Non ». 

| Mot clé | Variantes | 
| ------- | --------- | 
| Oui | Entendu, OK |
| Non | Pas du tout, Pas | 

### <a name="common-sample-sentences"></a>Exemples de phrases courantes
Les commandes personnalisées n’autorisent pas le partage d’exemples de phrases courantes entre différentes commandes. L’apprentissage d’une application peut échouer si des exemples de phrases d’une commande sont déjà définis dans une autre commande. 

Assurez-vous qu’aucun exemple de phrase courante n’est partagé entre différentes commandes. 

Pour connaître les meilleures pratiques en matière d’équilibrage des exemples de phrases sur différentes commandes, reportez-vous aux [Meilleures pratiques LUIS](../luis/luis-concept-best-practices.md).

### <a name="empty-sample-sentences"></a>Exemples de phrases vides
Vous devez avoir au moins un exemple de phrase pour chaque commande.

### <a name="undefined-parameter-in-sample-sentences"></a>Paramètre non défini dans les exemples de phrases
Un ou plusieurs paramètres sont utilisés dans les exemples de phrases, mais ils ne sont pas définis.

### <a name="training-takes-too-long"></a>L’apprentissage prend trop de temps
L’apprentissage LUIS est conçu pour apprendre rapidement avec peu d’exemples. N’ajoutez pas trop d’exemples de phrases. 

Si vous avez de nombreux exemples de phrases qui sont similaires, définissez un paramètre, extrayez-les vers un modèle, puis ajoutez-les aux exemples de phrases.

Par exemple, vous pouvez définir un paramètre {vehicle} pour les exemples de phrases ci-dessous, et ajouter uniquement « Réserver un(e) {vehicle} » à des exemples de phrases.

| Exemples de phrases | Modèle | 
| ------- | ------- | 
| Réserver une voiture | Réserver un(e) {vehicle} | 
| Book a flight | Réserver un(e) {vehicle} |
| Réserver un taxi | Réserver un(e) {vehicle} |

Pour connaître les meilleures pratiques en matière d’apprentissage LUIS, reportez-vous aux [Meilleures pratiques LUIS](../luis/luis-concept-best-practices.md).

## <a name="cant-update-luis-key"></a>Impossible de mettre à jour la clé LUIS
### <a name="reassign-to-e0-authoring-resource"></a>Réaffecter à la ressource de création E0
LUIS ne prend pas en charge la réaffectation d’une application LUIS à la ressource de création E0.

Si vous avez besoin de remplacer votre ressource de création F0 par E0 ou par une autre ressource E0, recréez l’application. 

Pour exporter rapidement une application existante et l’importer dans une nouvelle application, reportez-vous à [déploiement continu avec Azure DevOps](./how-to-custom-commands-deploy-cicd.md).

### <a name="save-button-is-disabled"></a>Le bouton Enregistrer est désactivé
Si vous n’affectez jamais une ressource de prédiction LUIS à votre application, le bouton Enregistrer est désactivé lorsque vous essayez de modifier votre ressource de création sans ajouter de ressource de prédiction.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Voir des exemples sur GitHub](https://aka.ms/speech/cc-samples)