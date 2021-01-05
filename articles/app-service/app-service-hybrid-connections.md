---
title: Connexions hybrides
description: Découvrez comment créer et utiliser des connexions hybrides dans Azure App Service pour accéder aux ressources de réseaux hétérogènes.
author: ccompy
ms.assetid: 66774bde-13f5-45d0-9a70-4e9536a4f619
ms.topic: article
ms.date: 06/08/2020
ms.author: ccompy
ms.custom: seodec18, fasttrack-edit
ms.openlocfilehash: 16f6a0660fa9aa20f636ee412f3f337bd5dea9b5
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825978"
---
# <a name="azure-app-service-hybrid-connections"></a>Connexions hybrides d’Azure App Service

Les connexions hybrides sont un service dans Azure et une fonctionnalité dans Azure App Service. En tant que service, il exploite et inclut des fonctionnalités qui vont au-delà de celles utilisées dans App Service. Pour en savoir plus sur les connexions hybrides et leur utilisation en dehors d’App Service, consultez [Connexions hybrides Azure Relay][HCService].

Dans App Service, les connexions hybrides peuvent être utilisées pour accéder aux ressources d’application dans les réseaux qui peuvent effectuer des appels sortants vers Azure via le port 443. Les connexions hybrides permettent l’accès entre votre application et un point de terminaison TCP, et ne permettent pas d’accéder autrement à votre application. Utilisée dans App Service, chaque connexion hybride correspond à une combinaison d’hôte et de port TCP unique. Cela permet aux applications d’accéder à des ressources sur n’importe quel système d’exploitation, à condition qu’il s’agisse d’un point de terminaison TCP. La fonctionnalité Connexions hybrides ne détectent pas et ne prennent pas en compte le protocole d’application ou les ressources auxquels vous accédez. Elle fournit simplement un accès réseau.  

## <a name="how-it-works"></a>Fonctionnement ##
Les connexions hybrides nécessitent qu’un agent de relais soit déployé à un emplacement d’où il pourra accéder à la fois au point de terminaison souhaité et à Azure. L’agent de relais, Hybrid Connection Manager (HCM), appelle Azure Relay sur le port 443. À partir du site de l’application web, l’infrastructure App Service se connecte également à Azure Relay au nom de votre application. Grâce aux connexions jointes, l’application accède au point de terminaison souhaité. La connexion utilise TLS 1.2 pour la sécurité et des clés de signature d’accès partagé (SAP) pour l’authentification et l’autorisation.    

![Diagramme du flux de haut niveau de la connexion hybride][1]

Lorsque votre application effectue une requête DNS qui correspond à un point de terminaison de connexion hybride configuré, le trafic TCP sortant est redirigé via la connexion hybride.  

> [!NOTE]
> Cela signifie que vous devez toujours utiliser un nom DNS pour votre connexion hybride. Certains logiciels clients n’effectuent une recherche DNS que si le point de terminaison utilise une adresse IP à la place. 
>

### <a name="app-service-hybrid-connection-benefits"></a>Avantages d’une connexion hybride App Service ###

La fonctionnalité de connexions hybrides offre un certain nombre d’avantages, notamment :

- Les applications peuvent accéder en toute sécurité aux systèmes et services locaux.
- La fonctionnalité ne nécessite pas un point de terminaison accessible par Internet.
- sa configuration est simple et rapide ; Aucune passerelle requise
- Chaque connexion hybride correspond à une combinaison hôte:port unique, gage de sécurité.
- Normalement, elle ne nécessite pas de trous de pare-feu. Les connexions sont toutes sortantes via des ports web standard.
- la fonctionnalité se situant au niveau du réseau, elle n’est pas spécifique au langage utilisé par votre application et à la technologie utilisée par le point de terminaison ;
- elle peut être utilisée pour fournir un accès à plusieurs réseaux à partir d’une même application. 
- Elle est prise en charge en disponibilité générale pour les applications natives Windows et en préversion pour les applications Linux. Elle n’est pas prise en charge pour les applications de conteneurs Windows.

### <a name="things-you-cannot-do-with-hybrid-connections"></a>Ce que vous ne pouvez pas faire avec les connexions hybrides ###

Ce que vous ne pouvez pas faire avec les connexions hybrides, notamment :

- Monter un lecteur.
- Utiliser UDP.
- Accéder à des services TCP qui utilisent des ports dynamiques tels que le mode FTP passif ou le mode passif étendu.
- Prendre en charge LDAP, car cela peut utiliser UDP.
- Prendre en charge Active Directory, car vous ne pouvez pas effectuer la jonction de domaine avec un rôle de travail App Service. 

## <a name="add-and-create-hybrid-connections-in-your-app"></a>Ajouter et créer des connexions hybrides dans votre application ##

Pour créer une connexion hybride, accédez au [portail Azure][portal] et sélectionnez votre application. Sélectionnez **Mise en réseau** > **Configurer vos points de terminaison de connexion hybride**. Vous pouvez alors voir les connexions hybrides configurées pour votre application.  

![Capture d’écran de la liste des connexions hybrides][2]

Pour ajouter une nouvelle connexion hybride, sélectionnez **[+] Ajouter une connexion hybride**.  La liste des connexions hybrides que vous avez déjà créées apparaît. Pour en ajouter une ou plusieurs à votre application, sélectionnez celles de votre choix, puis **Ajouter la connexion hybride sélectionnée**.  

![Capture d’écran du portail des connexions hybrides][3]

Si vous souhaitez créer une nouvelle connexion hybride, sélectionnez **Créer une connexion hybride**. Spécifiez les éléments suivants : 

- Nom de la connexion hybride.
- Nom d’hôte du point de terminaison.
- Port du point de terminaison.
- Espace de noms Service Bus que vous voulez utiliser.

![Capture d’écran de la boîte de dialogue de création d’une connexion hybride][4]

Chaque connexion hybride est liée à un espace de noms Service Bus et chaque espace de noms Service Bus se trouve dans une région Azure. Il est important d’essayer d’utiliser un espace de noms Service Bus dans la même région que votre application pour éviter une latence du réseau.

Si vous souhaitez supprimer votre connexion hybride de votre application, cliquez avec le bouton droit dessus et sélectionnez **Déconnecter**.  

Lorsqu’une connexion hybride est ajoutée à votre application, vous pouvez afficher ses détails simplement en la sélectionnant. 

![Capture d’écran des détails des connexions hybrides][5]

### <a name="create-a-hybrid-connection-in-the-azure-relay-portal"></a>Créer une connexion hybride dans le portail Azure Relay ###

En plus d’utiliser le portail à partir de votre application, vous pouvez créer des connexions hybrides depuis le portail Azure Relay. Pour qu’App Service utilise une connexion hybride, celle-ci doit :

* Exiger une autorisation du client.
* Comporter un élément de métadonnées, un point de terminaison nommé contenant une combinaison hôte:port comme valeur.

## <a name="hybrid-connections-and-app-service-plans"></a>Connexions hybrides et plans App Service ##

Les connexions hybrides App Service sont uniquement disponibles dans les références SKU de tarification De base, Standard, Premium et Isolé. Des limites sont liées au plan de tarification.  

| Plan tarifaire | Nombre de connexions hybrides utilisables dans le plan |
|----|----|
| De base | 5 par plan |
| Standard | 25 par plan |
| PremiumV2 | 200 par application |
| Isolé | 200 par application |

L’IU du plan App Service vous indique combien de connexions hybrides sont utilisées et par quelles applications.  

![Capture d’écran des propriétés du plan App Service][6]

Sélectionnez la connexion hybride pour voir les détails. Vous pouvez voir toutes les informations que vous avez vues dans la vue de l’application. Vous pouvez également voir combien d’autres applications utilisent cette connexion hybride dans le même plan.

Le nombre de points de terminaison de connexion hybride utilisables dans un plan App Service est limité. Chaque connexion hybride utilisée peut, toutefois, être utilisée sur autant d’applications que souhaité dans ce plan. Par exemple, une même connexion hybride utilisée dans cinq applications distinctes dans un plan App Service compte comme une seule connexion hybride.

### <a name="pricing"></a>Tarifs ###

En plus des exigences de référence SKU de plan App Service, il existe un coût supplémentaire lors de l’utilisation de connexions hybrides. Il existe des frais chaque écouteur utilisé par une connexion hybride. L’écouteur est l’Hybrid Connection Manager. Si vous aviez cinq connexions hybrides prises en charge par deux Hybrid Connection Manager, vous disposeriez de 10 écouteurs. Pour plus d’informations, consultez [Tarification Service Bus][sbpricing].

## <a name="hybrid-connection-manager"></a>Gestionnaire de connexion hybride ##

La fonctionnalité Connexions hybrides exige un agent de relais dans le réseau qui héberge votre point de terminaison de connexion hybride. Cet agent de relais est appelé le Gestionnaire de connexion hybride (GCH). Pour télécharger HCM, à partir de votre application dans le [portail Azure][portal], sélectionnez **Mise en réseau** > **Configurer vos points de terminaison de connexion hybride**.  

Cet outil s’exécute sur Windows Server 2012 et version ultérieure. Le HCM s’exécute en tant que service et connecte le trafic sortant à Azure Relay sur le port 443.  

Après avoir installé HCM, vous pouvez exécuter HybridConnectionManagerUi.exe pour utiliser l’interface utilisateur de l’outil. Ce fichier se trouve dans le répertoire d’installation de Hybrid Connection Manager. Dans Windows 10, vous pouvez aussi simplement rechercher *Hybrid Connection Manager UI* dans votre zone de recherche.  

![Capture d’écran de Hybrid Connection Manager][7]

Quand vous démarrez l’interface utilisateur HCM, vous voyez tout d’abord un tableau qui répertorie toutes les connexions hybrides configurées avec cette instance de HCM. Pour apporter des modifications, commencez par vous authentifier auprès d’Azure. 

Pour ajouter une ou plusieurs connexions hybrides à votre GCH :

1. Démarrez l’interface utilisateur de HCM.
2. Sélectionnez **Configurer une autre connexion hybride**.
![Capture d’écran de la configuration de nouvelles connexions hybrides][8]

1. Connectez-vous à votre compte Azure pour obtenir les connexions hybrides disponibles avec vos abonnements. Le HCM ne continue pas d’utiliser votre compte Azure au-delà de cela. 
1. Choisissez un abonnement.
1. Sélectionnez les connexions hybrides à faire relayer par HCM.
![Capture d’écran des connexions hybrides][9]

1. Sélectionnez **Enregistrer**.

Vous pouvez maintenant voir les connexions hybrides que vous avez ajoutées. Vous pouvez également sélectionner la connexion hybride configurée pour en afficher les détails.

![Capture d’écran des détails de la connexion hybride][10]

Pour prendre en charge les connexions hybrides avec lesquelles il est configuré, HCM exige les éléments suivants :

- Accès TCP à Azure sur le port 443.
- Accès TCP au point de terminaison de connexion hybride.
- Possibilité de recherches DNS sur l’hôte du point de terminaison et l’espace de noms Service Bus.

> [!NOTE]
> Azure Relay s’appuie sur les sockets web pour assurer la connectivité. Cette fonctionnalité est uniquement disponible sur Windows Server 2012 ou version ultérieure. C’est pourquoi HCM n’est pas pris en charge sur les systèmes antérieurs à Windows Server 2012.
>

### <a name="redundancy"></a>Redondance ###

Chaque GCH peut prendre en charge plusieurs connexions hybrides. De plus, toute connexion hybride peut être prise en charge par plusieurs GCH. Le comportement par défaut consiste à acheminer le trafic entre les HCM configurés pour n’importe quel point de terminaison donné. Si vous souhaitez une haute disponibilité sur vos connexions hybrides à partir de votre réseau, exécutez plusieurs HCM sur des ordinateurs distincts. L’algorithme de répartition de charge utilisé par le service Relay pour répartir le trafic vers les HCM est une affectation aléatoire. 

### <a name="manually-add-a-hybrid-connection"></a>Ajouter manuellement une connexion hybride ###

Pour permettre à une personne extérieure à votre abonnement d’héberger une instance HCM pour une connexion hybride donnée, communiquez-lui la chaîne de connexion de passerelle de la connexion hybride. Vous pouvez voir la chaîne de connexion de passerelle dans les propriétés de la connexion hybride dans le [portail Azure][portal]. Pour utiliser cette chaîne, sélectionnez **Saisir manuellement** dans HCM et collez la chaîne de connexion de passerelle.

![Ajouter manuellement une connexion hybride][11]

### <a name="upgrade"></a>Mettre à niveau ###

Le Hybrid Connection Manager est mis à jour périodiquement pour corriger des problèmes ou fournir des améliorations. Lors de la publication des mises à niveau, une fenêtre contextuelle s’affichera dans l’IU du HCM. Appliquer la mise à niveau appliquera les modifications et redémarrera le HCM. 

## <a name="adding-a-hybrid-connection-to-your-app-programmatically"></a>Ajout d’une connexion hybride à votre application par programmation ##

Les connexions hybrides sont prises en charge dans Azure CLI. Les commandes fournies s’exécutent à deux niveaux, celui le l’application et celui du plan App Service.  Les commandes au niveau de l’application sont les suivantes :

```azurecli
az webapp hybrid-connection

Group
    az webapp hybrid-connection : Methods that list, add and remove hybrid-connections from webapps.
        This command group is in preview. It may be changed/removed in a future release.
Commands:
    add    : Add a hybrid-connection to a webapp.
    list   : List the hybrid-connections on a webapp.
    remove : Remove a hybrid-connection from a webapp.
```

Les commandes au niveau du plan App Service vous permettent de définir la clé à utiliser par une connexion hybride donnée. Deux clés sont définies sur chaque connexion hybride : une clé primaire et une clé secondaire. Vous pouvez choisir d’utiliser la clé primaire ou secondaire avec les commandes ci-dessous. Cela vous permet de changer de clé quand vous regénérez vos clés régulièrement. 

```azurecli
az appservice hybrid-connection --help

Group
    az appservice hybrid-connection : A method that sets the key a hybrid-connection uses.
        This command group is in preview. It may be changed/removed in a future release.
Commands:
    set-key : Set the key that all apps in an appservice plan use to connect to the hybrid-
                connections in that appservice plan.
```

## <a name="secure-your-hybrid-connections"></a>Sécuriser vos connexions hybrides ##

Tout utilisateur disposant des autorisations suffisantes sur le relais Azure Service Bus Relay sous-jacent peut ajouter une connexion hybride existante à d’autres App Service Web Apps. Cela signifie que si vous devez empêcher d’autres utilisateurs de réutiliser cette même connexion hybride (par exemple, lorsque la ressource cible est un service ne présentant pas de mesures de sécurité supplémentaires pour empêcher les accès non autorisés), vous devez verrouiller l’accès à Azure Service Bus Relay.

Toute personne bénéficiant d’un accès `Reader` au relais peut _voir_ la connexion hybride lorsqu’elle essaye de l’ajouter à son application web dans le portail Azure. Toutefois, elle ne peut pas l’_ajouter_, car elle ne dispose pas des autorisations nécessaires pour récupérer la chaîne de connexion utilisée pour établir la connexion au relais. Pour ajouter la connexion hybride, les utilisateurs doivent disposer de l’autorisation `listKeys` (`Microsoft.Relay/namespaces/hybridConnections/authorizationRules/listKeys/action`). Le rôle `Contributor`, ou tout autre rôle comprenant cette autorisation sur le relais, permettra aux utilisateurs d’utiliser la connexion hybride et de l’ajouter à leurs propres applications web.

## <a name="troubleshooting"></a>Dépannage ##

L’état « Connecté » signifie qu’au moins un HCM est configuré avec cette connexion hybride et qu’il est en mesure d’atteindre Azure. Si l’état de votre connexion hybride n’indique pas **Connecté**, votre connexion hybride n’est configurée sur aucun HCM ayant accès à Azure.

La principale raison pour laquelle les clients ne peuvent pas se connecter à leur point de terminaison est que le point de terminaison a été spécifié à l’aide d’une adresse IP au lieu d’un nom DNS. Si votre application ne peut pas accéder au point de terminaison souhaité et que vous avez utilisé une adresse IP, utilisez un nom DNS valide sur l’hôte sur lequel le GCH est exécuté. Vérifiez également que le nom DNS est correctement résolu sur l’hôte sur lequel le HCM est en cours d’exécution. Vérifiez qu’il existe une connectivité à partir de l’hôte où le HCM est en cours d’exécution vers le point de terminaison de connexion hybride.  

Dans App Service, l’outil de ligne de commande **tcpping** peut être appelé à partir de la console Outils avancés (Kudu). Cet outil peut vous indiquer si vous avez accès à un point de terminaison TCP, mais ne vous dit pas si vous avez accès à un point de terminaison de connexion hybride. Lorsque vous utilisez l’outil dans la console par rapport à un point de terminaison de connexion hybride, vous confirmez seulement qu’il utilise une combinaison hôte:port.  

Si vous avez un client de ligne de commande pour votre point de terminaison, vous pouvez tester la connectivité à partir de la console de l’application. Par exemple, vous pouvez tester l’accès aux points de terminaison de serveur web à l’aide de curl.


<!--Image references-->
[1]: ./media/app-service-hybrid-connections/hybridconn-connectiondiagram.png
[2]: ./media/app-service-hybrid-connections/hybridconn-portal.png
[3]: ./media/app-service-hybrid-connections/hybridconn-addhc.png
[4]: ./media/app-service-hybrid-connections/hybridconn-createhc.png
[5]: ./media/app-service-hybrid-connections/hybridconn-properties.png
[6]: ./media/app-service-hybrid-connections/hybridconn-aspproperties.png
[7]: ./media/app-service-hybrid-connections/hybridconn-hcm.png
[8]: ./media/app-service-hybrid-connections/hybridconn-hcmadd.png
[9]: ./media/app-service-hybrid-connections/hybridconn-hcmadded.png
[10]: ./media/app-service-hybrid-connections/hybridconn-hcmdetails.png
[11]: ./media/app-service-hybrid-connections/hybridconn-manual.png
[12]: ./media/app-service-hybrid-connections/hybridconn-bt.png

<!--Links-->
[HCService]: /azure/service-bus-relay/relay-hybrid-connections-protocol/
[portal]: https://portal.azure.com/
[oldhc]: /azure/biztalk-services/integration-hybrid-connection-overview/
[sbpricing]: https://azure.microsoft.com/pricing/details/service-bus/
[armclient]: https://github.com/projectkudu/ARMClient/
