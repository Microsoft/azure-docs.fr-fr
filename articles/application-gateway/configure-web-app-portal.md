---
title: Gérer le trafic vers les applications multilocataires à l’aide du portail
titleSuffix: Azure Application Gateway
description: Cet article fournit de l’aide sur la configuration des applications web Azure App Service en tant que membres du pool de back-ends sur une passerelle d’application nouvelle ou existante.
services: application-gateway
author: surajmb
ms.service: application-gateway
ms.topic: how-to
ms.date: 09/23/2020
ms.author: victorh
ms.openlocfilehash: a72f0106088d26eb2ff53456840c598c3d9619a7
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397550"
---
# <a name="configure-app-service-with-application-gateway"></a>Configurer App Service avec Application Gateway

Étant donné que le service d’application est un service multi-locataire et non un déploiement dédié, il utilise l’en-tête de l’hôte dans la requête entrante pour résoudre la requête au point de terminaison de service d’application approprié. En règle générale, le nom DNS de l’application, qui à son tour est le nom DNS associé à la passerelle d’application qui expose le service d’application, est différent du nom de domaine du service d’application principal. Par conséquent, l’en-tête d’hôte de la requête d’origine reçu par la passerelle d’application n’est pas le même que le nom d’hôte du service principal. Pour cette raison, sauf si l’en-tête d’hôte de la requête provenant de la passerelle d’application vers le serveur principal est modifié pour porter le nom d’hôte du service principal, les back-ends multilocataires ne sont pas en mesure d’effectuer la résolution. de la requête vers le point de terminaison approprié.

Application Gateway fournit un commutateur appelé `Pick host name from backend target` qui remplace l'en-tête de l'hôte dans la requête par le nom d'hôte du back-end lorsque la requête est acheminée d'Application Gateway vers le back-end. Cette fonctionnalité permet la prise en charge des back-ends multilocataires tels que Azure App Service et la gestion des API. 

Dans cet article, vous apprendrez comment :

- Modifier un pool principal et y ajouter un App Service
- Modification des paramètres HTTP avec le commutateur « Choisir le nom d’hôte » activé

## <a name="prerequisites"></a>Prérequis

- Passerelle d’application : Créez une passerelle applicative sans cible de pool principal. Pour plus d’informations, consultez [Démarrage rapide : diriger le trafic web avec Azure Application Gateway - Portail Azure](quick-create-portal.md)

- Service d’application : Si vous n’avez pas encore de service d’application, consultez la [documentation relative à App Service](../app-service/index.yml).

## <a name="add-app-service-as-backend-pool"></a>Ajouter un service d’application comme pool de back-ends

1. Dans le portail Azure, sélectionnez votre passerelle applicative.

2. Sous **Pools principaux** , sélectionnez le pool principal.

4. Sous **Type de cible** , sélectionnez **App Services**.

5. Sous **Cible** , sélectionnez votre App Service.

   :::image type="content" source="./media/configure-web-app-portal/backend-pool.png" alt-text="Service d’application comme back-end":::
   
   > [!NOTE]
   > Le menu déroulant ne renseigne que les services d’application figurant dans le même abonnement que votre Application Gateway. Si vous voulez utiliser un service d'application d’un autre abonnement que celui où se trouve Application Gateway, au lieu de choisir **App Services** dans le menu déroulant **Cibles** , choisissez l’option **Adresse IP ou nom d'hôte** , puis entrez le nom d'hôte (example. azurewebsites.net) d’App Service.
1. Sélectionnez **Enregistrer**.

## <a name="edit-http-settings-for-app-service"></a>Modifier les paramètres HTTP pour App Service

1. Sous **Paramètres HTTP** , sélectionnez le paramètre HTTP existant.

2. Sous **Remplacer par le nouveau nom d’hôte** , sélectionnez **Oui**.
3. Sous **Remplacement du nom d’hôte** , sélectionnez **Choisir un nom d’hôte à partir d’une cible de backend**.
4. Sélectionnez **Enregistrer**.

   :::image type="content" source="./media/configure-web-app-portal/http-settings.png" alt-text="Choisir un nom d’hôte à partir des paramètres HTTP du backend":::

## <a name="additional-configuration-in-case-of-redirection-to-app-services-relative-path"></a>Configuration supplémentaire en cas de redirection vers le chemin relatif d’App Service

Quand App Service envoie une réponse de redirection au client pour le rediriger vers son chemin relatif (par exemple, une redirection de `contoso.azurewebsites.net/path1` vers `contoso.azurewebsites.net/path2`), il utilise le même nom d’hôte dans l’en-tête d’emplacement de sa réponse que celui de la requête reçue de la passerelle applicative. Le client adresse donc la demande directement à `contoso.azurewebsites.net/path2` au lieu de passer par la passerelle applicative (`contoso.com/path2`). Or, il n’est pas souhaitable de contourner la passerelle d’application.

Si dans votre cas d'utilisation, il y a des scénarios où App Service devra envoyer une réponse de redirection au client, exécutez les étapes [supplémentaires pour réécrire l'en-tête d’emplacement](./troubleshoot-app-service-redirection-app-service-url.md#sample-configuration).

## <a name="restrict-access"></a>Restriction de l’accès

Les applications web déployées dans ces exemples utilisent des adresses IP publiques qui sont accessibles directement à partir d’Internet. La résolution des problèmes s’en trouve facilitée quand vous découvrez une nouvelle fonctionnalité et essayez de nouvelles choses. Mais si vous prévoyez de déployer une fonctionnalité sur l’environnement de production, l’ajout de restrictions peut vous sembler approprié.

Une façon de restreindre l’accès à vos applications web consiste à utiliser des [restrictions d’adresse IP statique avec Azure App Service](../app-service/app-service-ip-restrictions.md). Par exemple, vous pouvez restreindre l’accès à l’application web au trafic transitant par la passerelle d’application. Utilisez la fonctionnalité de restriction IP avec App Service pour répertorier l’adresse IP virtuelle de la passerelle d’application comme étant la seule adresse bénéficiant d’un accès.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur App Service et sur la prise en charge multilocataire avec Application Gateway, consultez [Prise en charge de service multilocataire avec Application Gateway](./application-gateway-web-app-overview.md).