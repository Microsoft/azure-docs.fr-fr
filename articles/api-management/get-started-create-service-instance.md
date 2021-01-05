---
title: 'Démarrage rapide : Créer une instance du service Gestion des API Azure'
description: Créez une instance du service Gestion des API Azure avec le portail Azure.
author: vladvino
ms.service: api-management
ms.topic: quickstart
ms.custom: mvc
ms.date: 09/08/2020
ms.author: apimpm
ms.openlocfilehash: 66d0874536b358b720b43780d6ecce879ac866ee
ms.sourcegitcommit: 7374b41bb1469f2e3ef119ffaf735f03f5fad484
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/16/2020
ms.locfileid: "90708204"
---
# <a name="quickstart-create-a-new-azure-api-management-service-instance-by-using-the-azure-portal"></a>Démarrage rapide : Créer une instance du service Gestion des API Azure avec le portail Azure

Le service Gestion des API Azure (API Management, APIM) permet aux organisations de publier des API pour des développeurs externes, partenaires et internes afin de libérer le potentiel de leurs données et de leurs services. Il offre les compétences essentielles qui garantissent un programme d’API réussi au travers de l’engagement des développeurs, des perspectives commerciales, de l’analytique, de la sécurité et de la protection. APIM vous permet de créer et de gérer des passerelles d’API modernes pour des services back-end existants, où qu’ils soient hébergés. Pour plus d'informations, consultez la [Vue d’ensemble](api-management-key-concepts.md).

Ce guide de démarrage rapide décrit les étapes permettant de créer une instance de la Gestion des API à l’aide du portail Azure.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

Instance Gestion des API

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="create-a-new-service"></a>Créer un service

1. Dans le menu du Portail Azure, sélectionnez **Créer une ressource**. Vous pouvez également sélectionner **Créer une ressource** sur la page d’**accueil** Azure. 
   
   Sélectionnez :::image type="content" source="media/get-started-create-service-instance/00-CreateResource-01.png" alt-text="Créer une ressource":::

   
1. Dans la page **Nouveau**, sélectionnez **Integration** > **Gestion des API**.

   Nouvelle instance du service Gestion des API Azure
   
1. Dans la page **Service Gestion des API**, entrez les paramètres.

   :::image type="content" source="media/get-started-create-service-instance/get-started-create-service-instance-create-new.png" alt-text="Créer une ressource":::
   
   | Paramètre                 | Description   |                                                                     
   |-------------------------|-----------------------------------------------|
   | Name                | Nom unique pour votre service Gestion des API. Vous ne pourrez plus changer ce nom. Le nom du service fait référence au service et à la ressource Azure correspondante. <br/> Le nom du service sert à générer un nom de domaine par défaut : *\<name\>.azure-api.net.* Si vous souhaitez utiliser un nom de domaine personnalisé, consultez [Configurer un domaine personnalisé](configure-custom-domain.md). |
   | **Abonnement** :          | Abonnement sous lequel cette nouvelle instance de service sera créée.   |
   | **Groupe de ressources**      |  Sélectionnez un nouveau groupe de ressources ou un groupe existant. Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. |
   | **Lieu**          | Sélectionnez une zone géographique près de chez vous parmi les localisations du service Gestion des API disponibles. | 
   | **Nom de l’organisation**   | Nom de votre organisation. Ce nom est utilisé à plusieurs endroits, notamment dans le titre du portail des développeurs et comme expéditeur des e-mails de notification. |                                                         
   | **E-mail de l’administrateur** | Adresse e-mail à laquelle toutes les notifications de la **Gestion des API** seront envoyées.   |  
   | **Niveau tarifaire**        | Sélectionnez le niveau **Développeur** pour évaluer le service. Ce niveau n’est pas destiné à la production. Pour en savoir plus sur la mise à l’échelle des niveaux du service Gestion des API, consultez [Mettre à niveau et mettre à l’échelle](upgrade-and-scale.md). |

3. Sélectionnez **Create** (Créer).

    > [!TIP]
    > La création et l’activation d’un service Gestion des API à ce niveau peuvent prendre entre 30 et 40 minutes. Sélectionnez **Épingler au tableau de bord** pour rechercher plus rapidement un service qui vient d’être créé.

[!INCLUDE [api-management-navigate-to-instance](../../includes/api-management-navigate-to-instance.md)]

Passez en revue les propriétés de votre service dans la page **Vue d’ensemble**.

   Instance Gestion des API

Une fois votre instance du service Gestion des API en ligne, vous êtes prêt à l’utiliser. Commencez par le tutoriel [Importer et publier votre première API](import-and-publish.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, vous pouvez supprimer le groupe de ressources et toutes les ressources associées en suivant ces étapes :

1. Depuis le portail Azure, recherchez et sélectionnez **Groupes de ressources**. Vous pouvez également sélectionner **Groupes de ressources** depuis la page **d’accueil**. 

   :::image type="content" source="media/get-started-create-service-instance/00-DeleteResource-01.png" alt-text="Créer une ressource":::

1. Sur la page **Groupes de ressources**, sélectionnez votre groupe de ressources.

   :::image type="content" source="media/get-started-create-service-instance/00-DeleteResource-02.png" alt-text="Créer une ressource":::

1. Sur la page Groupe de ressources, sélectionnez **Supprimer un groupe de ressources**. 
   
1. Saisissez le nom de votre groupe de ressources, puis sélectionnez **Supprimer**.

   Supprimer le groupe de ressources

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Importer et publier votre première API](import-and-publish.md)
