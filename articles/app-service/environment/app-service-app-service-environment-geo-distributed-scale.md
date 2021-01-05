---
title: Échelle géodistribuée
description: Découvrez comment mettre à l’échelle des applications horizontalement en utilisant Traffic Manager et les environnements App Service.
author: stefsch
ms.assetid: c1b05ca8-3703-4d87-a9ae-819d741787fb
ms.topic: article
ms.date: 09/07/2016
ms.author: stefsch
ms.custom: seodec18, references_regions
ms.openlocfilehash: 004b32118521f72c5b59ad7bab2d4e41244b85c4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85833602"
---
# <a name="geo-distributed-scale-with-app-service-environments"></a>Mise à l’échelle géolocalisée avec les environnements App Service
## <a name="overview"></a>Vue d’ensemble

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Les scénarios d’application à très grande échelle peuvent dépasser la capacité de ressources de calcul disponible pour un seul déploiement d’application.  À titre d’exemple, les applications de vote, les événements sportifs et les programmes de divertissement télévisés sont des scénarios qui fonctionnent à très grande échelle. Les exigences de fonctionnement à grande échelle peuvent être satisfaites par un scale-out horizontal des applications. Pour gérer les charges extrêmes, beaucoup de déploiements d’applications peuvent être effectués dans une seule ou plusieurs régions.

Les environnements ASE (App Service Environment) sont une plateforme idéale pour le scale-out horizontal. Après avoir choisi une configuration d’environnement ASE capable de prendre en charge un taux de requêtes connu, les développeurs peuvent déployer des environnements ASE supplémentaires en mode « découpage » (cookie cutter) pour atteindre la capacité de charge maximale souhaitée.

Par exemple, supposons qu’une application s’exécutant sur une configuration d’environnement App Service a été testée pour traiter 20 Ko de requêtes par seconde (RPS).  Si la capacité de charge maximale souhaitée est de 100 Ko RPS, il est possible de créer et configurer cinq (5) environnements App Service pour s’assurer que l’application pourra gérer la charge maximale prévue.

Les clients accédant généralement aux applications par le biais d’un domaine personnalisé (ou personnel), les développeurs ont besoin d’un moyen de distribuer les demandes d’application parmi toutes les instances d’environnement App Service.  Un excellent moyen d’atteindre ce but consiste à résoudre le domaine personnalisé à l’aide d’un [profil Azure Traffic Manager][AzureTrafficManagerProfile].  Le profil Traffic Manager peut être configuré pour pointer sur tous les environnements App Service.  Traffic Manager gérera automatiquement la distribution aux clients sur tous les environnements ASE en fonction des paramètres d’équilibrage de la charge définis dans le profil Traffic Manager.  Cette approche fonctionne que les environnements App Service soient déployés dans une seule région Azure ou dans plusieurs régions Azure à travers le monde.

En outre, comme les clients accèdent aux applications via le domaine personnel, les clients ne connaissent pas le nombre d’environnements App Service exécutant une application.  Par conséquent les développeurs peuvent rapidement et facilement ajouter et supprimer des environnements App Service en fonction de la charge du trafic observée.

Le schéma conceptuel ci-dessous décrit une application mise à l’échelle horizontalement dans trois environnements App Service au sein d’une seule région.

![Architecture conceptuelle][ConceptualArchitecture] 

Le reste de cette rubrique décrit les étapes nécessaires à la configuration d’une topologie distribuée pour l’exemple d’application à l’aide de plusieurs environnements App Service.

## <a name="planning-the-topology"></a>Planification de la topologie
Avant de créer une empreinte d’application distribuée, il peut s’avérer utile de disposer de quelques éléments d’informations.

* **Domaine personnalisé pour l’application :**  quel est le nom de domaine personnalisé que les clients utiliseront pour accéder à l’application ?  Pour l’exemple d’application, le nom de domaine personnalisé est `www.scalableasedemo.com`.
* **Domaine Traffic Manager :** choisissez un nom de domaine au moment de la création d’un [profil Azure Traffic Manager][AzureTrafficManagerProfile].  Ce nom est associé au suffixe *trafficmanager.net* pour enregistrer une entrée de domaine gérée par Traffic Manager.  Dans l’exemple d’application, le nom choisi est *scalable-ase-demo*.  Par conséquent, le nom de domaine complet géré par Traffic Manager est *scalable-ase-demo.trafficmanager.net*.
* **Stratégie de mise à l’échelle de l’empreinte de l’application :**  l’empreinte de l’application sera-t-elle distribuée sur plusieurs environnements App Service dans une seule région ?  Plusieurs régions ?  Une combinaison des deux approches ?  Prenez la décision en fonction de l’emplacement d’où proviendra probablement le trafic client, ainsi que de la capacité de mise à l’échelle du reste de l’infrastructure back-end sur laquelle repose l’application.  Par exemple, avec une application à 100 % sans état, une application peut être adaptée à très grande échelle en utilisant une combinaison de nombreux environnements AES dans chaque région Azure, multipliée par les environnements AES déployés dans beaucoup de régions Azure.  Avec plus de 15 régions Azure globales disponibles, les clients peuvent véritablement créer une empreinte d’application à échelle mondiale.  Pour l’exemple d’application utilisé dans cet article, trois environnements ASE ont été créés dans une seule région Azure (USA Centre Sud).
* **Convention de nommage pour les environnements App Service :**  chaque environnement App Service requiert un nom unique.  Au-delà d’un ou deux environnements App Service, une convention d’affectation de noms identifiant chaque environnement App Service peut s’avérer utile.  Pour l’exemple d’application, une convention de nommage simple a été utilisée.  Les noms des trois environnements App Service sont respectivement *fe1ase*, *fe2ase* et *fe3ase*.
* **Convention d’affectation de noms pour les applications :**  comme plusieurs instances de l’application vont être déployées, un nom est requis pour chacune d’entre elles.  Les environnements ASE proposent une fonctionnalité peu connue, mais pratique, car elle permet d’utiliser le même nom d’application dans de multiples environnements ASE.  Étant donné que chaque environnement App Service comporte un suffixe de domaine unique, les développeurs peuvent choisir d’utiliser le même nom d’application dans chaque environnement.  Par exemple, un développeur peut avoir des applications nommées comme suit : *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, etc.  Toutefois, pour l’exemple d’application, chaque instance de l’application a également un nom unique.  Les noms d’instance application utilisés sont *webfrontend1*, *webfrontend2* et *webfrontend3*.

## <a name="setting-up-the-traffic-manager-profile"></a>Configuration d’un profil Traffic Manager
Une fois que plusieurs instances d’une application sont déployées sur plusieurs environnements App Service, les instances d’application individuelles peuvent être enregistrées avec Traffic Manager.  Dans l’exemple d’application, un profil Traffic Manager qui peut router les clients vers les instances d’application qui suivent est nécessaire pour *scalable-ase-demo.trafficmanager.net* :

* **webfrontend1.fe1ase.p.azurewebsites.net :**  instance de l’exemple d’application déployée sur le premier environnement App Service.
* **webfrontend2.fe2ase.p.azurewebsites.net :**  instance de l’exemple d’application déployée sur le deuxième environnement App Service.
* **webfrontend3.fe3ase.p.azurewebsites.net :**  instance de l’exemple d’application déployée sur le troisième environnement App Service.

Le moyen le plus simple d’enregistrer plusieurs points de terminaison Azure App Service qui s’exécutent tous dans la **même** région Azure est d’utiliser la [prise en charge d’Azure Resource Manager pour Traffic Manager][ARMTrafficManager] dans PowerShell.  

La première étape consiste à créer un profil Azure Traffic Manager.  Le code suivant montre comment le profil a été créé pour l’exemple d’application :

```azurepowershell-interactive
$profile = New-AzureTrafficManagerProfile –Name scalableasedemo -ResourceGroupName yourRGNameHere -TrafficRoutingMethod Weighted -RelativeDnsName scalable-ase-demo -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
```

Notez la manière dont le paramètre *RelativeDnsName* a été défini sur *scalable-ase-demo*.  Ce paramètre crée le nom de domaine *scalable-ase-demo.trafficmanager.net* et l’associe à un profil Traffic Manager.

Le paramètre *TrafficRoutingMethod* définit la stratégie d’équilibrage de la charge selon laquelle Traffic Manager répartira la charge cliente entre tous les points de terminaison disponibles.  Dans cet exemple, la méthode *Weighted* a été choisie.  Du fait de ce choix, les demandes clientes seront réparties entre tous les points de terminaison d’application enregistrés en fonction de la pondération relative associée à chacun d’eux. 

Avec le profil créé, chaque instance de l’application est ajoutée au profil en tant que point de terminaison Azure natif.  Le code suivant extrait une référence à chaque application web front-end. Il ajoute ensuite chaque application en tant que point de terminaison Traffic Manager par le biais du paramètre *TargetResourceId*.

```azurepowershell-interactive
$webapp1 = Get-AzWebApp -Name webfrontend1
Add-AzureTrafficManagerEndpointConfig –EndpointName webfrontend1 –TrafficManagerProfile $profile –Type AzureEndpoints -TargetResourceId $webapp1.Id –EndpointStatus Enabled –Weight 10

$webapp2 = Get-AzWebApp -Name webfrontend2
Add-AzureTrafficManagerEndpointConfig –EndpointName webfrontend2 –TrafficManagerProfile $profile –Type AzureEndpoints -TargetResourceId $webapp2.Id –EndpointStatus Enabled –Weight 10

$webapp3 = Get-AzWebApp -Name webfrontend3
Add-AzureTrafficManagerEndpointConfig –EndpointName webfrontend3 –TrafficManagerProfile $profile –Type AzureEndpoints -TargetResourceId $webapp3.Id –EndpointStatus Enabled –Weight 10

Set-AzureTrafficManagerProfile –TrafficManagerProfile $profile
```

Notez qu’il existe un appel de *Add-AzureTrafficManagerEndpointConfig* pour chaque instance d’application individuelle.  Le paramètre *TargetResourceId* dans chaque commande PowerShell référence l’une des trois instances d’application déployées.  Le profil Traffic Manager répartira la charge entre les trois points de terminaison enregistrés dans le profil.

Les trois points de terminaison utilisent la même valeur (10) pour le paramètre *Poids* .  Le résultat est que Traffic Manager répartit les requêtes clientes entre les trois instances d’application de façon relativement uniforme. 

## <a name="pointing-the-apps-custom-domain-at-the-traffic-manager-domain"></a>Pointer le domaine personnalisé de l’application sur le domaine Traffic Manager
La dernière étape obligatoire consiste à pointer le domaine personnalisé de l’application sur le domaine Traffic Manager.  Pour l’exemple d’application, pointez `www.scalableasedemo.com` sur `scalable-ase-demo.trafficmanager.net`.  Effectuez cette étape avec le service de registre de domaine (registrar) qui gère le domaine personnalisé.  

Avec les outils de gestion d’enregistrement de domaine, vous devez créer un enregistrement CNAME pointant sur le domaine personnalisé du domaine Traffic Manager.  L’illustration ci-dessous montre un exemple de cette configuration CNAME :

![Enregistrement CNAME pour le domaine personnalisé][CNAMEforCustomDomain] 

Bien que cet aspect ne soit pas traité dans cette rubrique, n’oubliez pas que le domaine personnalisé doit être enregistré avec chaque instance d’application individuelle.  Si ce n’est pas le cas, quand une requête est effectuée sur une instance d’application et que l’application n’a pas enregistré le domaine personnalisé avec l’application, la requête échoue.

Dans cet exemple, le domaine personnalisé est `www.scalableasedemo.com`, et chaque instance d’application est associée au domaine personnalisé.

![Domaine personnalisé][CustomDomain] 

Pour avoir un récapitulatif de l’enregistrement d’un domaine personnalisé avec les applications Azure App Service, consultez l’article sur l’[enregistrement de domaines personnalisés][RegisterCustomDomain].

## <a name="trying-out-the-distributed-topology"></a>Essai de la topologie distribuée
Le résultat final de la configuration de Traffic Manager et de DNS est que les demandes de `www.scalableasedemo.com` circulent dans la séquence suivante :

1. Un navigateur ou un appareil effectue une recherche DNS sur `www.scalableasedemo.com`
2. L’entrée CNAME sur l’enregistrement de domaine fait que la recherche DNS est redirigée vers Azure Traffic Manager.
3. Une recherche DNS sur *scalable-ase-demo.trafficmanager.net* est effectuée sur l’un des serveurs DNS Traffic Manager d’Azure.
4. Sur la base de la stratégie d’équilibrage de la charge spécifiée précédemment dans le paramètre *TrafficRoutingMethod*, Traffic Manager sélectionne l’un des points de terminaison configurés. Il retourne ensuite le nom de domaine complet (FQDN) du point de terminaison choisi au navigateur ou à l’appareil.
5. Étant donné que le nom de domaine complet du point de terminaison est l’URL d’une instance d’application exécutée dans un environnement ASE, le navigateur ou l’appareil demandera à un serveur Microsoft Azure DNS de résoudre le nom de domaine complet sur une adresse IP. 
6. Le navigateur ou le périphérique envoie la demande HTTP/S à l’adresse IP.  
7. La demande arrive dans une des instances d’application en cours d’exécution sur un des environnements App Service.

L’image de console ci-dessous montre une recherche DNS pour trouver le domaine personnalisé de l’exemple d’application. La recherche s’est résolue en une instance d’application qui s’exécute sur l’un des trois exemples d’environnements ASE (dans ce cas, le deuxième des trois environnements ASE) :

![Recherche DNS][DNSLookup] 

## <a name="additional-links-and-information"></a>Informations et liens supplémentaires
Documentation sur la [prise en charge d’Azure Resource Manager pour Azure Traffic Manager][ARMTrafficManager] dans PowerShell.  

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[AzureTrafficManagerProfile]: ../../traffic-manager/traffic-manager-manage-profiles.md
[ARMTrafficManager]: ../../traffic-manager/traffic-manager-powershell-arm.md
[RegisterCustomDomain]: ../app-service-web-tutorial-custom-domain.md


<!-- IMAGES -->
[ConceptualArchitecture]: ./media/app-service-app-service-environment-geo-distributed-scale/ConceptualArchitecture-1.png
[CNAMEforCustomDomain]:  ./media/app-service-app-service-environment-geo-distributed-scale/CNAMECustomDomain-1.png
[DNSLookup]:  ./media/app-service-app-service-environment-geo-distributed-scale/DNSLookup-1.png
[CustomDomain]:  ./media/app-service-app-service-environment-geo-distributed-scale/CustomDomain-1.png 
