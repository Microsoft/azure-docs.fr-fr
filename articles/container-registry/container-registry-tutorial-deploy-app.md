---
title: Tutoriel - Effectuer un déploiement à partir d’un registre géorépliqué
description: Déployez une application web basée sur Linux sur deux régions Azure différentes à l’aide d’une image conteneur à partir d’un registre de conteneurs Azure géorépliqué. Deuxième partie d’une série de trois.
ms.topic: tutorial
ms.date: 08/20/2018
ms.custom: seodec18, mvc
ms.openlocfilehash: 7a203bfc9b1317bc258e4a93ae4ac03ecbdc7a15
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92148425"
---
# <a name="tutorial-deploy-a-web-app-from-a-geo-replicated-azure-container-registry"></a>Tutoriel : Déployer une application web à partir d’un registre de conteneurs Azure géorépliqué

Ceci est la deuxième partie d’une série de didacticiels qui en compte trois. Dans la [première partie](container-registry-tutorial-prepare-registry.md), un registre de conteneurs privé et géorépliqué a été créé, et une image de conteneur a été générée à partir de la source et envoyée au registre. Cet article explique comment tirer parti de la proximité réseau du registre géorépliqué en déployant le conteneur dans deux instances d’application web situées dans deux régions Azure distinctes. Chaque instance extrait ensuite l’image conteneur du registre le plus proche.

Dans ce didacticiel, la deuxième partie de la série :

> [!div class="checklist"]
> * Déployer une image de conteneur sur deux instances *Web App pour conteneurs*
> * Vérifier l’application déployée

Si vous n’avez pas encore créé de registre géorépliqué et placé l’image de l’exemple d’application en conteneur dans le registre, revenez au didacticiel précédent de la série, [Préparer un registre de conteneurs Azure géorépliqué](container-registry-tutorial-prepare-registry.md).

L’article suivant de la série explique comment mettre à jour l’application, puis envoyer l’image conteneur mise à jour dans le registre. Enfin, vous accédez à chaque instance d’application web en cours d’exécution pour voir la modification répercutée automatiquement dans les deux, montrant la géoréplication et les webhooks d’Azure Container Registry en action.

## <a name="automatic-deployment-to-web-apps-for-containers"></a>Déploiement automatique sur Web App pour conteneurs

Azure Container Registry prend en charge le déploiement d’applications en conteneur directement vers [Web App pour conteneurs](../app-service/index.yml). Ce didacticiel explique comment utiliser le portail Azure pour déployer l’image conteneur créée dans le didacticiel précédent vers deux plans d’application web situés dans des régions Azure différentes.

Lorsque vous déployez une application web à partir d’une image de conteneur dans votre registre, et que vous avez un registre géorépliqué dans la même région, Azure Container Registry crée un [webhook](container-registry-webhook.md) de déploiement d’image pour vous. Lorsque vous envoyez une nouvelle image au référentiel de votre conteneur, le webhook récupère la modification et déploie automatiquement la nouvelle image de conteneur vers votre application web.

## <a name="deploy-a-web-app-for-containers-instance"></a>Déployer une application web pour une instance de conteneurs

Cette étape explique comment créer une instance Web App pour conteneurs dans la région *USA Ouest*.

Connectez-vous au [portail Azure](https://portal.azure.com), puis accédez au registre que vous avez créé dans le didacticiel précédent.

Sélectionnez **Dépôts** > **acr-helloworld**, cliquez avec le bouton droit sur la balise **v1** sous **Balises**, puis sélectionnez **Déployer sur l’application web** :

![Déployer vers le service d’application dans le portail Azure][deploy-app-portal-01]

Si l’option « Déployer sur l’application web » est désactivée, vous n’avez peut-être pas activé l’utilisateur administrateur de registre comme indiqué dans [Créer un registre de conteneurs](container-registry-tutorial-prepare-registry.md#create-a-container-registry) dans le premier didacticiel. Vous pouvez activer l’utilisateur administrateur dans **Paramètres** > **Clés d’accès** dans le portail Azure.

Sous **Web App pour conteneurs** qui s’affiche lorsque vous sélectionnez « Déployer sur l’application web », spécifiez les valeurs suivantes pour chaque paramètre :

| Paramètre | Valeur |
|---|---|
| **Nom du site** | Nom global unique de l’application web. Dans cet exemple, nous utilisons le format `<acrName>-westus` pour identifier facilement le registre et la région à partir de laquelle l’application web est déployée. |
| **Groupe de ressources** | **Utiliser l’existant** > `myResourceGroup` |
| **Plan/emplacement du service d’application** | Créez un plan nommé `plan-westus` dans la région **USA Ouest**. |
| **Image** | `acr-helloworld:v1` |
| **Système d’exploitation** | Linux |

> [!NOTE]
> Quand vous créez un nouveau plan App Service pour déployer votre application conteneurisée, un plan par défaut est sélectionné automatiquement pour héberger votre application. Le plan par défaut dépend du paramètre du système d’exploitation.

Sélectionnez **Créer** pour approvisionner l’application web dans la région *USA Ouest*.

![Capture d’écran montrant Web App pour conteneurs avec le bouton Créer mis en évidence.][deploy-app-portal-02]

## <a name="view-the-deployed-web-app"></a>Afficher l’application web déployée

Une fois le déploiement terminé, vous pouvez afficher l’application en cours d’exécution en accédant à son URL dans votre navigateur.

Dans le portail, sélectionnez **App Services**, puis l’application web que vous avez approvisionnée à l’étape précédente. Dans cet exemple, l’application web est nommée *uniqueregistryname-westus*.

Sélectionnez le lien hypertexte de l’URL de l’application web dans l’angle supérieur droit de la vue d’ensemble **App Service** pour afficher l’application en cours d’exécution dans votre navigateur.

![Capture d’écran montrant la vue d’ensemble App Service avec l’URL de l’application web mise en évidence.][deploy-app-portal-04]

Une fois l’image Docker déployée à partir de votre registre de conteneurs géorépliqué, le site affiche une image représentant la région Azure hébergeant le registre de conteneurs.

![Capture d’écran montrant l’application web déployée dans un navigateur.][deployed-app-westus]

## <a name="deploy-second-web-app-for-containers-instance"></a>Déployer une deuxième instance Web App pour conteneurs

Suivez la procédure décrite dans la section précédente pour déployer une deuxième application web dans la région *USA Est*. Sous **Web App pour conteneurs**, spécifiez les valeurs suivantes :

| Paramètre | Valeur |
|---|---|
| **Nom du site** | Nom global unique de l’application web. Dans cet exemple, nous utilisons le format `<acrName>-eastus` pour identifier facilement le registre et la région à partir de laquelle l’application web est déployée. |
| **Groupe de ressources** | **Utiliser l’existant** > `myResourceGroup` |
| **Plan/emplacement du service d’application** | Créez un plan nommé `plan-eastus` dans la région **USA Est**. |
| **Image** | `acr-helloworld:v1` |
| **Système d’exploitation** | Linux |

Sélectionnez **Créer** pour approvisionner l’application web dans la région *USA Est*.

![Capture d’écran montrant la fenêtre Créer de Web App pour conteneurs avec le bouton Créer mis en évidence.][deploy-app-portal-06]

## <a name="view-the-second-deployed-web-app"></a>Afficher la seconde application web déployée

Comme auparavant, vous pouvez afficher l’application en cours d’exécution en accédant à son URL dans votre navigateur.

Dans le portail, sélectionnez **App Services**, puis l’application web que vous avez approvisionnée à l’étape précédente. Dans cet exemple, l’application web est nommée *uniqueregistryname-eastus*.

Sélectionnez le lien hypertexte de l’URL de l’application web dans l’angle supérieur droit de la **vue d’ensemble ’App Service** pour afficher l’application en cours d’exécution dans votre navigateur.

![Application web sur une configuration Linux dans le portail Azure][deploy-app-portal-07]

Une fois l’image Docker déployée à partir de votre registre de conteneurs géorépliqué, le site affiche une image représentant la région Azure hébergeant le registre de conteneurs.

![Application web déployée affichée dans un navigateur][deployed-app-eastus]

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez déployé deux instances Web App pour conteneurs à partir d’un registre de conteneurs Azure géorépliqué.

Passez au didacticiel suivant pour mettre à jour, puis déployer une nouvelle image de conteneur dans le registre de conteneurs, puis vérifier que les applications web en cours d’exécution dans les deux régions ont été mises à jour automatiquement.

> [!div class="nextstepaction"]
> [Déployer une mise à jour vers l’image de conteneur géorépliqué](./container-registry-tutorial-deploy-update.md)

<!-- IMAGES -->
[deploy-app-portal-01]: ./media/container-registry-tutorial-deploy-app/deploy-app-portal-01.png
[deploy-app-portal-02]: ./media/container-registry-tutorial-deploy-app/deploy-app-portal-02.png
[deploy-app-portal-03]: ./media/container-registry-tutorial-deploy-app/deploy-app-portal-03.png
[deploy-app-portal-04]: ./media/container-registry-tutorial-deploy-app/deploy-app-portal-04.png
[deploy-app-portal-05]: ./media/container-registry-tutorial-deploy-app/deploy-app-portal-05.png
[deploy-app-portal-06]: ./media/container-registry-tutorial-deploy-app/deploy-app-portal-06.png
[deploy-app-portal-07]: ./media/container-registry-tutorial-deploy-app/deploy-app-portal-07.png
[deployed-app-westus]: ./media/container-registry-tutorial-deploy-app/deployed-app-westus.png
[deployed-app-eastus]: ./media/container-registry-tutorial-deploy-app/deployed-app-eastus.png