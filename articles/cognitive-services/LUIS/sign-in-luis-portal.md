---
title: Se connecter au portail LUIS
description: Si vous êtes un nouvel utilisateur qui se connecte au portail LUIS, l’expérience de connexion varie légèrement en fonction de votre compte d’utilisateur actuel.
services: cognitive-services
ms.custom: ''
ms.service: cognitive-services
ms.subservice: language-understanding
ms.date: 09/08/2020
ms.topic: how-to
ms.author: nitinme
author: nitinme
ms.openlocfilehash: b8382b76496976054ebb452e39866765d986ccbb
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368174"
---
# <a name="sign-in-to-luis-portal"></a>Se connecter au portail LUIS

[!INCLUDE [LUIS Free account](includes/luis-portal-note.md)]

Cet article va vous aider à prendre en main le portail LUIS et à créer une ressource de création. Une fois que vous aurez effectué les étapes décrites dans cet article, vous serez en mesure de créer et de publier des applications LUIS.

## <a name="access-the-portal"></a>Accéder au portail


1. Pour vous familiariser avec LUIS, accédez au [Portail LUIS](https://www.luis.ai). Si vous n’avez pas encore d’abonnement, vous êtes invité à créer un compte [gratuit](https://azure.microsoft.com//free/cognitive-services/) et à revenir au portail.
2. Actualisez la page pour la mettre à jour avec l’abonnement que vous venez de créer.
3. Sélectionnez votre abonnement dans la liste déroulante.

    > [!div class="mx-imgBorder"]
    > ![sélectionner les abonnements](./media/migrate-authoring-key/select-subscription-sign-in-2.png)

4. Si votre abonnement se trouve sous un autre locataire, vous ne pouvez pas changer de locataire à partir de la fenêtre existante. Vous pouvez changer de locataire en fermant cette fenêtre et en sélectionnant l’avatar le plus à droite contenant vos initiales dans la barre supérieure. Cliquez sur **Choisir une autre ressource de création** dans la partie supérieure pour rouvrir la fenêtre.

    > [!div class="mx-imgBorder"]
    > ![Changer de répertoire](./media/migrate-authoring-key/switch-directories.png)

5. Si vous disposez déjà d’une ressource de création LUIS associée à votre abonnement, choisissez-la dans la liste déroulante. Vous pouvez afficher toutes les applications qui sont créées sous cette ressource de création.
6. Si ce n’est pas le cas, cliquez sur **Créer une ressource de création** en bas de ce modal.
7.  Quand vous créez une ressource de création, fournissez les informations suivantes :

    > [!div class="mx-imgBorder"]
    > ![Créer une ressource](./media/migrate-authoring-key/create-new-authoring-resource-2.png)

    * **Nom du locataire** : locataire auquel votre abonnement Azure est associé. Vous ne serez pas en mesure de basculer les locataires à partir de la fenêtre existante. Vous pouvez changer de locataire en fermant cette fenêtre et en sélectionnant l’avatar contenant vos initiales dans le coin supérieur droit de l’écran. Sélectionnez **Choisir une autre ressource de création** dans la partie supérieure pour rouvrir la fenêtre.
    * **Nom du groupe de ressources Azure** : nom de groupe de ressources personnalisé que vous choisissez dans votre abonnement. Les groupes de ressources vous permettent de regrouper des ressources Azure pour l’accès et la gestion. Si vous n’avez pas de groupe de ressources dans votre abonnement, vous ne serez pas autorisé à en créer un dans le portail LUIS. Accédez au [Portail Azure](https://ms.portal.azure.com/#create/Microsoft.ResourceGroup) pour en créer un, puis cliquez sur LUIS pour continuer le processus de connexion.
    * **Nom de la ressource Azure** : nom personnalisé que vous choisissez, utilisé comme élément de l’URL pour vos transactions de création. Le nom de votre ressource doit contenir uniquement des caractères alphanumériques (`-`) et ne peut pas commencer ni se terminer par `-`. Si d’autres symboles sont inclus dans le nom, la création d’une ressource échoue.
    * **Emplacement** : choisissez de créer vos applications dans l’un des [trois emplacements de création](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-regions) actuellement pris en charge par LUIS, notamment : USA Ouest, Europe Ouest et Australie Est.
    * **Niveau tarifaire** : par défaut, le niveau tarifaire de création F0 est sélectionné, car il est recommandé. Créez une [clé gérée par le client](https://docs.microsoft.com/azure/cognitive-services/luis/luis-encryption-of-data-at-rest#customer-managed-keys-for-language-understanding) à partir du portail Azure si vous recherchez une couche supplémentaire de sécurité.
8. Vous êtes maintenant connecté à LUIS. Vous pouvez dès à présent commencer à créer des applications.

## <a name="troubleshooting"></a>Dépannage

* Lors de la création d’une ressource, assurez-vous que le nom de la ressource ne contient que des caractères alphanumériques, '-', et ne commence ou ne se termine pas par '-'. Dans le cas contraire, elle échoue.
* Assurez-vous que vous disposez des [autorisations appropriées sur votre abonnement pour créer une ressource Azure](../../role-based-access-control/rbac-and-directory-admin-roles.md#azure-roles). Si vous ne disposez pas des autorisations appropriées, contactez l’administrateur de votre abonnement pour obtenir des autorisations suffisantes.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [démarrer une nouvelle application](luis-how-to-start-new-app.md)
