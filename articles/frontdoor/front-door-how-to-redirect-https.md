---
title: Créer une porte d’entrée avec redirection de HTTP vers HTTPS à l’aide du portail Microsoft Azure
description: Apprenez à créer une porte d’entrée avec redirection du trafic de HTTP vers HTTPS à l’aide du portail Microsoft Azure.
services: front-door
author: duongau
ms.service: frontdoor
ms.topic: how-to
ms.date: 09/30/2020
ms.author: duau
ms.openlocfilehash: 19908b3cba63bc76a205097ef8d16e612d58503b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91626636"
---
# <a name="create-a-front-door-with-http-to-https-redirection-using-the-azure-portal"></a>Créer une porte d’entrée avec redirection de HTTP vers HTTPS à l’aide du portail Microsoft Azure

Vous pouvez utiliser le portail Azure afin de [créer une instance Front Door](quickstart-create-front-door.md) avec un certificat pour la terminaison TLS. Une règle d’acheminement est utilisée pour rediriger le trafic HTTP vers HTTPS.

## <a name="create-a-front-door-with-an-existing-web-app-resource"></a>Créer une porte d’entrée avec une ressource d’application web existante

1. Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

1. Sélectionnez  **Créer une ressource** dans le coin supérieur gauche du portail Azure.

1. Recherchez **Front Door** à l’aide de la barre de recherche. Après avoir trouvé le type de ressource, sélectionnez **Créer**.

1. Choisissez un *abonnement*, puis utilisez un groupe de ressources existant ou créez-en un. Sélectionnez **Suivant** pour accéder à l’onglet de configuration.

    > [!NOTE]
    > L’emplacement demandé dans l’interface utilisateur est pour le groupe de ressources uniquement. Votre configuration de porte d’entrée va être déployée sur l’ensemble des [emplacements POP d’Azure Front Door](front-door-faq.md#what-are-the-pop-locations-for-azure-front-door).

    :::image type="content" source="./media/front-door-url-redirect/front-door-create-basics.png" alt-text="Configurer les paramètres de base pour la nouvelle porte d’entrée":::

1. La configuration de la porte d’entrée se déroule en trois étapes : l’ajout d’un hôte frontend par défaut, l’ajout de back ends dans un pool de back ends et la création de règles d’acheminement pour mapper le comportement d’acheminement à l’hôte frontend. Sélectionnez l’icône « **+**  » sur les _Hôtes front-end_ pour créer un hôte front-end.

    :::image type="content" source="./media/front-door-url-redirect/front-door-designer.png" alt-text="Configurer les paramètres de base pour la nouvelle porte d’entrée":::

1. Entrez un nom global unique destiné à votre hôte front-end par défaut pour votre instance Front Door. Sélectionnez **Ajouter** pour passer à l’étape suivante.

    :::image type="content" source="./media/front-door-url-redirect/front-door-create-frontend-host.png" alt-text="Configurer les paramètres de base pour la nouvelle porte d’entrée":::

### <a name="create-backend-pool"></a>Créer un pool de back-ends

1. Cliquez sur l’icône « **+**  » dans _Pools de back-ends_ pour créer un pool de back-ends. Fournissez un nom pour le pool, puis sélectionnez « **Ajouter un back-end** ».

    :::image type="content" source="./media/front-door-url-redirect/front-door-designer-backend-pool.png" alt-text="Configurer les paramètres de base pour la nouvelle porte d’entrée":::

1. Sélectionnez le type d’hôte back end en tant que _service d’application_. Sélectionnez l’abonnement dans lequel votre application web est hébergée, puis sélectionnez l’application web spécifique dans la liste déroulante pour **Nom de l’hôte back end**.

    :::image type="content" source="./media/front-door-url-redirect/front-door-create-backend-pool.png" alt-text="Configurer les paramètres de base pour la nouvelle porte d’entrée":::

1. Sélectionnez **Ajouter** pour enregistrer le back-end et sélectionnez de nouveau **Ajouter** pour enregistrer la configuration du pool de back-ends. 

## <a name="create-http-to-https-redirect-rule"></a>Ajouter une règle de redirection HTTP vers HTTPS

1. Sélectionnez l’icône « **+**  » dans *Règles de routage* pour créer une route. Fournissez un nom pour la route, par exemple « HttpToHttpsRedirect », puis définissez le champ *Protocole accepté* sur **« HTTP uniquement »** . Vérifiez que la sélection des *Front-ends/domaines* est appropriée.  

    :::image type="content" source="./media/front-door-url-redirect/front-door-designer-routing-rule.png" alt-text="Configurer les paramètres de base pour la nouvelle porte d’entrée":::

1. Sous la section *Détails de la route*, définissez le *Type de route* sur **Rediriger**. Assurez-vous que le *Type de redirection* est défini sur **Trouvé (302)** , et le *Protocole de redirection* sur **HTTPS uniquement**. 

    :::image type="content" source="./media/front-door-url-redirect/front-door-redirect-config-example.png" alt-text="Configurer les paramètres de base pour la nouvelle porte d’entrée":::

1. Sélectionnez **Ajouter** pour enregistrer la règle de routage de la redirection HTTP vers HTTPS.

## <a name="create-forwarding-rule"></a>Créer une règle de transfert

1. Ajoutez une autre règle de routage pour gérer le trafic HTTPS. Sélectionnez le signe « **+**  » dans *Règles de routage* et fournissez un nom pour la route, par exemple « DefaultForwardingRoute ». Définissez ensuite le champ *Protocoles acceptés* sur **HTTPS uniquement**. Vérifiez que la sélection des *Front-ends/domaines* est appropriée.

1. Dans la section Détails de la route, définissez le *Type de route* sur **Transférer**. Assurez-vous que le pool de back-ends approprié est sélectionné, et que le *Protocole de transfert* est défini sur **HTTPS uniquement**. 

    :::image type="content" source="./media/front-door-url-redirect/front-door-forward-route-example.png" alt-text="Configurer les paramètres de base pour la nouvelle porte d’entrée" border="false":::

1. Sélectionnez **Ajouter** pour enregistrer la règle de routage du transfert de requêtes.

1. Cliquez sur **Vérifier + créer**, puis sur **Créer** pour créer votre profil Front Door. Accédez à la ressource après sa création.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez [comment fonctionne Front Door](front-door-routing-architecture.md).
- Découvrez plus en détail la [redirection d’URL sur la porte d’entrée](front-door-url-redirect.md).
