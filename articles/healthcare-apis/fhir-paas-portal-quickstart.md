---
title: 'Démarrage rapide : Déployer l’API Azure pour FHIR à l’aide du portail Azure'
description: Dans ce guide de démarrage rapide, vous allez apprendre à déployer l’API Azure pour FHIR et à configurer les paramètres à l’aide du portail Azure.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: quickstart
ms.date: 03/15/2020
ms.author: cavoeg
ms.openlocfilehash: cabacd5204f4a9ac5c17c7bd66924482b5bf688a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023463"
---
# <a name="quickstart-deploy-azure-api-for-fhir-using-azure-portal"></a>Démarrage rapide : Déployer l’API Azure pour FHIR à l’aide du portail Azure

Dans ce guide de démarrage rapide, vous allez apprendre à déployer l’API Azure pour FHIR à l’aide du portail Azure.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="create-new-resource"></a>Créer une ressource

Ouvrez le [portail Azure](https://portal.azure.com), puis cliquez sur **Créer une ressource**

![Créer une ressource](media/quickstart-paas-portal/portal-create-resource.png)

## <a name="search-for-azure-api-for-fhir"></a>Rechercher l’API Azure pour FHIR

Vous pouvez trouver l’API Azure pour FHIR en tapant « FHIR » dans la zone de recherche :

:::image type="content" source="media/quickstart-paas-portal/portal-search-healthcare-apis.png" alt-text="Rechercher des API médicales":::

## <a name="create-azure-api-for-fhir-account"></a>Créer un compte d’API Azure pour FHIR

Sélectionnez **Créer** afin de créer un compte d’API Azure pour FHIR :

:::image type="content" source="media/quickstart-paas-portal/portal-create-healthcare-apis.png" alt-text="Créer un compte d’API Azure pour FHIR":::

## <a name="enter-account-details"></a>Entrer les détails du compte

Sélectionnez un groupe de ressources existant, ou créez-en un, choisissez un nom pour le compte, puis cliquez sur **Vérifier + créer** :

:::image type="content" source="media/quickstart-paas-portal/portal-new-healthcareapi-details.png" alt-text="Détails de la nouvelle API médicale":::

Confirmez la création et attendez le déploiement de l’API FHIR.

## <a name="additional-settings-optional"></a>Paramètres supplémentaires (facultatif)

Vous pouvez également cliquer sur **suivant : Paramètres supplémentaires** pour afficher les paramètres d’authentification. La configuration par défaut de l’API Azure pour FHIR consiste à [utiliser Azure RBAC pour attribuer des rôles de plan de données](configure-azure-rbac.md). Dans ce mode de configuration, l’ « Autorité » pour le service FHIR est définie sur le locataire Azure Active Directory de l’abonnement :

:::image type="content" source="media/rbac/confirm-azure-rbac-mode-create.png" alt-text="Paramètres d’authentification par défaut":::

Notez que la zone de saisie des ID d’objet autorisés est grisée, car nous utilisons Azure RBAC pour configurer des attributions de rôles dans ce cas.

Si vous souhaitez configurer le service FHIR pour utiliser un locataire Azure Active Directory externe ou secondaire, vous pouvez modifier l’Autorité et entrer des ID d’objet pour un utilisateur et des groupes qui doivent être autorisés à accéder au serveur. Pour plus d’informations, consultez le guide de [configuration de RBAC local](configure-local-rbac.md).

## <a name="fetch-fhir-api-capability-statement"></a>Récupérer une déclaration de capacité relative à l’API FHIR

Pour vérifier que le nouveau compte d’API FHIR est provisionné, récupérez une déclaration de capacité en faisant pointer un navigateur vers `https://<ACCOUNT-NAME>.azurehealthcareapis.com/metadata`.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, vous pouvez supprimer le groupe de ressources, l’API Azure pour FHIR et toutes les ressources associées. Pour ce faire, sélectionnez le groupe de ressources contenant le compte d’API Azure pour FHIR, sélectionnez **Supprimer le groupe de ressources**, puis confirmez le nom du groupe de ressources à supprimer.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez déployé l’API Azure pour FHIR sur votre abonnement. Pour définir des paramètres supplémentaires dans votre API Azure pour FHIR, consultez le guide pratique relatif aux paramètres supplémentaires. Si vous êtes prêt à commencer à utiliser l’API Azure pour FHIR, découvrez plus en détail comment inscrire des applications.

>[!div class="nextstepaction"]
>[Paramètres supplémentaires dans l’API Azure pour FHIR](azure-api-for-fhir-additional-settings.md)

>[!div class="nextstepaction"]
>[Présentation de l’inscription d’applications](fhir-app-registration.md)
