---
title: Configurer ASE v1
description: Configuration, gestion et supervision de l’environnement App Service v1. Ce document s’adresse uniquement aux clients qui utilisent l’environnement ASE v1 hérité.
author: ccompy
ms.assetid: b5a1da49-4cab-460d-b5d2-edd086ec32f4
ms.topic: article
ms.date: 07/11/2017
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 598e43d07c213cfeb25f0ecbc7bd02b6ec54b7ed
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88962585"
---
# <a name="configuring-an-app-service-environment-v1"></a>Configuration d’un environnement App Service Environment v1

> [!NOTE]
> Cet article traite de l’environnement App Service Environment v1.  Il existe une version plus récente de l’environnement App Service Environment, plus facile à utiliser et qui s’exécute sur des infrastructures plus puissantes. Pour en savoir plus sur la nouvelle version, commencez par la section [Présentation de l’environnement App Service Environment](intro.md).
> 

## <a name="overview"></a>Vue d’ensemble
Globalement, un environnement Azure App Service se compose de plusieurs composants principaux :

* Des ressources de calcul s’exécutant dans le service hébergé d’environnement App Service
* Stockage
* Une base de données
* Un réseau virtuel Azure (VNet) classique (V1) ou Resource Manager (V2) 
* Un sous-réseau sur lequel s’exécute le service hébergé d’environnement App Service

### <a name="compute-resources"></a>Ressources de calcul
Vous utilisez les ressources de calcul pour vos quatre pools de ressources.  Chaque environnement App Service (ASE) possède un ensemble de serveurs frontaux et trois pools de travail possibles. Vous n’avez pas besoin d’utiliser les trois pools de travail. Si vous le souhaitez, vous pouvez n’en utiliser qu’un ou deux.

Les hôtes présents dans les pools de ressources (des serveurs frontaux et de travail) ne sont pas directement accessibles par les clients. Vous ne pouvez pas utiliser le protocole RDP (Remote Desktop Protocol) pour vous y connecter, modifier leur approvisionnement ou agir sur eux en tant qu’administrateur.

Vous pouvez définir la taille et la quantité des pools de ressources. Dans un environnement App Service, vous disposez de quatre options de taille (P1 à P4). Pour plus de détails sur ces tailles et leur tarification, voir [Tarification de App Service](https://azure.microsoft.com/pricing/details/app-service/).
La modification de la quantité ou de la taille est une opération appelée « mise à l’échelle ».  Il ne peut y avoir qu’une seule opération de mise à l’échelle en cours à la fois.

**Serveurs frontaux :** les front-ends sont les points de terminaison HTTP/HTTPS pour vos applications contenues dans votre environnement App Service (ASE). Les charges de travail ne sont pas exécutées dans les serveurs frontaux.

* Un environnement App Service commence avec deux P2, ce qui est suffisant pour les charges de travail de développement et de test et les charges de travail de production de bas niveau. Nous vous recommandons vivement d’utiliser des P3 pour des charges de travail de production moyennes et lourdes.
* Il est recommandé d’avoir au moins quatre P3 pour les charges de travail moyennes et lourdes, afin de garantir l’exécution d’un nombre suffisant de serveurs frontaux pendant la maintenance planifiée. Les activités de maintenance planifiées interrompront un seul serveur frontal à la fois. Cela réduit la capacité globale disponible des serveurs frontaux pendant les activités de maintenance.
* L’approvisionnement des serveurs frontaux peut prendre jusqu’à une heure. 
* Pour un ajustement optimal de la mise à l’échelle, vous devez analyser le pourcentage UC, le pourcentage de mémoire ainsi que les mesures des demandes actives pour le pool frontal. Si le pourcentage UC ou de mémoire est supérieur à 70 % lorsque vous exécutez des P3, ajoutez davantage de serveurs frontaux. Si la valeur moyenne des demandes actives oscille entre 15 000 et 20 000 demandes par serveur frontal, vous devez également ajouter davantage de serveurs frontaux. L’objectif est de maintenir le pourcentage UC et de mémoire en dessous de 70 % et le nombre moyen des demandes actives en dessous de 15 000 par serveur frontal lorsque vous exécutez des P3.  

**Workers** : les workers sont l’emplacement où s’exécutent en réalité vos applications. Lorsque vous faites monter en puissance vos plans App Service, cette opération utilise des Workers dans le pool de travail associé.

* L’ajout instantané de Workers n’est pas possible. Leur approvisionnement peut prendre jusqu’à une heure.
* La mise à l’échelle d’une ressource de calcul pour un pool quelconque prendra moins de 1 heure par domaine de mise à jour. Un environnement App Service contient 20 domaines de mise à jour. Si vous mettez à l’échelle la taille de calcul d’un pool de travail contenant 10 instances, cette opération peut prendre jusqu’à 10 heures.
* Si vous modifiez la taille des ressources de calcul utilisées dans un pool de travail, vous pouvez provoquer des démarrages à froid des applications en cours d’exécution dans ce pool de travail.

La façon la plus rapide de modifier la taille des ressources de calcul d’un pool de travail qui n’exécute pas d’applications est de :

* Réduire la quantité d’employés à 2.  L’échelle minimale de diminution dans le portail est de 2. La libération de vos instances prendra quelques minutes. 
* Sélectionner la nouvelle taille de calcul et le nombre d’instances. À partir de là, l’opération prendra jusqu’à 2 heures.

Si vos applications nécessitent une plus grande taille de ressources de calcul, les étapes précédentes ne vous concernent pas. Au lieu de modifier la taille du pool de travail qui héberge ces applications, vous pouvez remplir un autre pool de travail avec des Workers de la taille souhaitée, puis déplacer vos applications vers ce pool.

* Créez les instances supplémentaires de la taille de calcul nécessaire dans un autre pool de travail. L’opération prendra jusqu’à 1 heure.
* Réaffectez vos plans App Service hébergeant les applications qui nécessitent une plus grande taille au pool de travail nouvellement configuré. Il s’agit d’une opération rapide qui devrait prendre moins d’une minute.  
* Descendez en puissance le premier pool de travail si ces instances inutilisées ne vous servent plus. Cette opération prend quelques minutes.

**Mise à l’échelle automatique** : la mise à l’échelle automatique constitue l’un des outils pouvant vous aider à gérer la consommation des ressources de calcul. Vous pouvez utiliser la mise à l’échelle pour les serveurs frontaux ou les pools de travail. Vous pouvez effectuer des opérations comme augmenter vos instances de n’importe quel type de pool le matin et les réduire le soir. Vous pouvez également par exemple ajouter des instances lorsque le nombre de Workers disponibles dans un pool de travail passe en dessous d’un certain seuil.

Si vous souhaitez définir des règles de mise à l’échelle automatique en vous basant sur des mesures de pool de ressources de calcul, gardez en tête le temps d’approvisionnement nécessaire. Pour en savoir plus sur les environnements App Service, voir [Mise à l’échelle automatique et environnement App Service][ASEAutoscale].

### <a name="storage"></a>Stockage
Chaque ASE est configuré avec 500 Go de stockage. Cet espace est utilisé par toutes les applications dans l’ASE. Cet espace de stockage est une partie de l’ASE ; actuellement, aucun basculement pour utiliser votre espace de stockage n’est possible. Si vous ajustez le routage ou la sécurité de votre réseau virtuel, vous devez autoriser l’accès à Azure Storage ; sinon, l’ASE ne fonctionnera pas.

### <a name="database"></a>Base de données
La base de données conserve les informations qui définissent l’environnement, ainsi que des détails sur les applications exécutées. Cela est également inclus dans l’abonnement Azure. Il ne s’agit pas d’un élément que vous avez la possibilité de manipuler directement. Si vous ajustez le routage ou la sécurité de votre réseau virtuel, vous devez autoriser l’accès à SQL Azure ; sinon, l’ASE ne fonctionnera pas.

### <a name="network"></a>Réseau
Le réseau virtuel qui est utilisé avec votre ASE peut être un réseau que vous avez créé lors de la création de l’ASE ou un réseau que vous aviez créé au préalable. Si vous créez le sous-réseau lors de la création de l’ASE, cela force l’ASE à être dans le même groupe de ressources que le réseau virtuel. Si vous avez besoin que le groupe de ressources utilisé par votre ASE soit différent de celui de votre réseau virtuel, alors vous devez créer votre ASE à l’aide d’un modèle Resource Manager.

Un réseau virtuel contient quelques restrictions utilisées pour un environnement App Service :

* Le réseau virtuel doit être un réseau virtuel régional.
* Un sous-réseau avec 8 adresses ou plus où est déployé l’ASE doit exister.
* Lorsqu’un sous-réseau est utilisé pour héberger un ASE, la plage d’adresses du sous-réseau ne peut être modifiée. Pour cette raison, il est recommandé que le sous-réseau contienne au moins 64 adresses pour la prise en charge d’ASE supplémentaires potentiels.
* Le sous-réseau doit contenir uniquement l’ASE.

Contrairement au service hébergé qui contient l’ASE, le [réseau virtuel][virtualnetwork] et le sous-réseau sont tous les deux contrôlés par l’utilisateur.  Vous pouvez administrer votre réseau virtuel via l’interface utilisateur de réseau virtuel ou Powershell.  Un ASE peut être déployé dans un réseau virtuel classique ou Resource Manager.  Le portail et les expériences API varient légèrement entre les réseaux virtuels classique et Resource Manager, mais l’expérience ASE est identique.

Le réseau virtuel utilisé pour héberger un ASE peut utiliser des adresses IP RFC1918 privées ou des adresses IP publiques.  Si vous souhaitez utiliser une plage IP qui n’est pas couverte par RFC1918 (10.0.0.0/8, 172.16.0.0/12 et 192.168.0.0/16), vous devez créer le réseau virtuel et le sous-réseau utilisés par votre ASE préalablement à sa création.

Étant donné que cette fonctionnalité place Azure App Service sur votre réseau virtuel, vos applications hébergées dans votre ASE peuvent désormais accéder aux ressources rendues disponibles directement via des réseaux virtuels VPN ExpressRoute ou site à site. Les applications de vos environnements App Service ne nécessitent pas de fonctionnalités de mise en réseau supplémentaires pour accéder aux ressources disponibles sur le réseau virtuel qui héberge votre environnement App Service. Cela signifie que vous n’avez pas besoin d’utiliser l’intégration de réseau virtuel ou les connexions hybrides pour accéder aux ressources dans ou connectées à votre réseau virtuel. Vous pouvez toujours utiliser ces deux fonctionnalités pour accéder aux ressources dans les réseaux qui ne sont pas connectés à votre réseau virtuel.

Par exemple, vous pouvez utiliser l’intégration de réseau virtuel pour vous intégrer à un réseau virtuel qui figure dans votre abonnement mais qui n’est pas connecté au réseau virtuel dans lequel se trouve votre ASE. Vous pouvez toujours également utiliser des connexions hybrides pour accéder aux ressources dans d’autres réseaux, comme vous le faites normalement.  

Si votre réseau virtuel est configuré avec un VPN ExpressRoute, vous devez être informé de certains des besoins de routage d’un ASE. Certaines configurations d’itinéraire défini par l’utilisateur ne sont pas compatibles avec un ASE. Pour plus de détails concernant l’exécution d’un ASE dans un réseau virtuel avec ExpressRoute, voir [Exécution d’un environnement App Service dans un réseau virtuel avec ExpressRoute][ExpressRoute].

#### <a name="securing-inbound-traffic"></a>Sécurisation du trafic entrant
Il existe deux méthodes principales pour contrôler le trafic entrant dans votre ASE.  Vous pouvez utiliser des groupes de sécurité réseau (NSG) pour contrôler les adresses IP pouvant accéder à votre ASE comme décrit dans [Comment contrôler le trafic entrant dans un environnement App Service](app-service-app-service-environment-control-inbound-traffic.md) et vous pouvez également configurer votre ASE avec un équilibreur de charge interne (ILB).  Ces fonctionnalités peuvent par ailleurs être utilisées simultanément si vous souhaitez restreindre l’accès à votre ILB ASE à l’aide de groupes de sécurité réseau.

Lorsque vous créez un ASE, une adresse IP virtuelle sera créée dans votre réseau virtuel.  Il existe deux types d’adresses IP virtuelles : internes et externes.  Lorsque vous créez un ASE avec une adresse IP virtuelle externe, les applications dans votre ASE sont accessibles via une adresse IP Internet routable. Si vous optez pour une adresse interne, votre ASE sera configuré avec un ILB et ne sera pas directement accessible par Internet.  Un ILB ASE nécessite malgré tout une adresse IP virtuelle externe, mais celle-ci est utilisée uniquement pour l’accès lié à la gestion et à la maintenance Azure.  

Lors de la création de l’ILB ASE, vous indiquez le sous-domaine qu’il utilise et vous devez gérer votre propre serveur DNS pour le sous-domaine que vous spécifiez.  Étant donné que vous définissez le nom du sous-domaine, vous devez également gérer le certificat utilisé pour l’accès HTTPS.  Suite à la création d’un ASE, vous êtes invité à fournir le certificat.  Pour en savoir plus sur la création et l’utilisation d’un ILB ASE, voir [Utilisation d’un équilibreur de charge interne avec un environnement App Service][ILBASE]. 

## <a name="portal"></a>Portail
L’interface utilisateur sur le portail Azure vous permet de gérer et surveiller votre environnement App Service. Si vous possédez un ASE, il est possible que vous voyiez le symbole App Service dans votre barre latérale. Ce symbole est utilisé pour représenter les environnements App Service dans le portail Azure :

![Symbole d’environnement App Service][1]

Pour ouvrir l’interface utilisateur qui répertorie tous vos environnements App Service, vous pouvez utiliser l’icône ou sélectionnez le chevron (symbole « > ») en bas de la barre latérale pour sélectionner les environnements App Service. La sélection de l’un des ASE répertoriés ouvre l’interface utilisateur utilisée pour le surveiller et le gérer.

![Interface utilisateur de surveillance et de gestion de votre environnement App Service][2]

Ce premier panneau présente certaines propriétés de votre ASE avec un graphique de mesures par pool de ressources. Certaines des propriétés affichées dans le bloc **Essentiels** sont également des liens hypertexte qui ouvrent le panneau associé. Par exemple, vous pouvez sélectionner le nom de **réseau virtuel** pour ouvrir l’interface utilisateur associée au réseau virtuel dans lequel votre ASE est exécuté. Les **plans App Service** et les **applications** ouvrent des panneaux qui répertorient les éléments figurant dans votre ASE.  

### <a name="monitoring"></a>Surveillance
Les graphiques permettent de voir les différentes mesures de performances dans chaque pool de ressources. Vous pouvez analyser l’utilisation moyenne de l’UC et de la mémoire pour le pool frontal. Pour les pools de travail, vous pouvez surveiller la quantité utilisée et la quantité disponible.

Plusieurs plans App Service peuvent utiliser les Workers dans un pool de travail. La charge de travail n’est pas distribuée de la même manière que pour les serveurs frontaux. L’analyse de l’utilisation de l’UC et de la mémoire ne fournit donc pas des informations pertinentes. Il est plus important de suivre le nombre de Workers que vous avez utilisés et qui sont disponibles, en particulier si vous gérez ce système pour d’autres utilisateurs.  

Vous pouvez également utiliser toutes les mesures qui peuvent être suivies dans les graphiques pour configurer des alertes. La configuration d’alertes ici fonctionne de la même manière qu’ailleurs dans App Service. Vous pouvez définir une alerte à partir de l’interface utilisateur **Alertes** ou dans toute interface utilisateur de mesures en cliquant sur **Ajouter une alerte**.

![Interface utilisateur Mesures][3]

Les mesures dont nous venons de parler sont les mesures de l’environnement App Service. Des mesures sont également disponibles au niveau du plan App Service. C’est là que la surveillance de l’utilisation de l’UC et de la mémoire se révèle intéressante.

Dans un ASE, tous les plans App Service sont des plans App Service dédiés. Cela signifie que les seules applications qui sont exécutées sur les hôtes alloués à ce plan App Service sont les applications de ce plan App Service. Pour afficher des détails sur votre plan App Service, affichez-le simplement à partir d’une liste quelconque dans l’interface utilisateur de l’ASE, voire à partir des **plans App Service** qui les répertorient tous.   

### <a name="settings"></a>Paramètres
Le panneau de l’ASE contient une section **Paramètres**. Cette dernière comprend plusieurs fonctionnalités importantes :

**Paramètres** > **Propriétés** : le panneau **Paramètres** s’ouvre automatiquement quand vous affichez le panneau de votre environnement App Service (ASE). Les **Propriétés**se trouvent dans la partie supérieure. Ici, plusieurs éléments sont redondants avec ce que vous voyez dans le bloc **Essentiels**, mais deux propriétés sont très utiles : **Adresse IP virtuelle** et **Adresses IP sortantes**.

![Panneau Paramètres et Propriétés][4]

**Paramètres** > **Adresses IP** : une adresse IP SSL est nécessaire quand vous créez une application IP SSL (Secure Sockets Layer) dans votre environnement App Service (ASE). Pour en obtenir une, votre ASE doit posséder des adresses IP SSL qui peuvent être allouées. Lorsqu’un ASE est créé, il dispose d’une adresse IP SSL unique à cette fin, mais vous pouvez en ajouter d’autres. Ces adresses IP SSL supplémentaires sont payantes. Consultez les prix affichés sur la page [Tarification de App Service][AppServicePricing] (dans la rubrique Connexions SSL). Le supplément de prix correspond au coût IP SSL.

**Paramètres** > **Pool frontal** / **Pools de workers** : chacun de ces panneaux de pools de ressources offre la possibilité de voir uniquement des informations sur ce pool de ressources, en plus de proposer des commandes permettant de le mettre entièrement à l’échelle.  

Le panneau de base pour chaque pool de ressources fournit un graphique avec des mesures pour ce pool de ressources. À l’instar des graphiques du panneau de l’ASE, vous pouvez entrer dans le graphique et configurer les alertes de votre choix. La configuration d’une alerte à partir du panneau de l’ASE pour un pool de ressources spécifique produit le même résultat que d’effectuer l’opération à partir du pool de ressources. Dans le panneau **Paramètres** du pool de travail, vous avez accès à l’ensemble des applications ou plans App Service qui s’exécutent dans ce pool de travail.

![Interface utilisateur Paramètres du pool de travail][5]

### <a name="portal-scale-capabilities"></a>Fonctionnalités de mise à l’échelle du portail
Il existe trois opérations de mise à l’échelle :

* Modifier le nombre d’adresses IP dans l’ASE qui sont disponibles pour l’utilisation IP SSL.
* Modifier la taille de la ressource de calcul utilisée dans un pool de ressources.
* Modifier le nombre de ressources de calcul utilisées dans un pool de ressources manuellement ou via la mise à l’échelle automatique.

Dans le portail, il existe trois façons de contrôler le nombre de serveurs dont vous disposez dans vos pools de ressources :

* Une opération de mise à l’échelle à partir du panneau principal de l’ASE dans la partie supérieure. Vous pouvez modifier la configuration de mise à l’échelle du pool frontal et des pools de travail de plusieurs façons. Toutes les modifications sont appliquées en une seule opération.
* Une opération de mise à l’échelle manuelle à partir du panneau **Mise à l’échelle** du pool de ressources individuel, sous **Paramètres**.
* Une mise à l’échelle automatique configurée à partir du panneau **Mise à l’échelle** du pool de ressources individuel.

Pour utiliser l’opération de mise à l’échelle sur le panneau de l’ASE, faites glisser le curseur vers la quantité souhaitée et enregistrez. Cette interface utilisateur prend également en charge la modification de la taille.  

![Interface utilisateur Mise à l’échelle][6]

Pour utiliser les fonctionnalités de mise à l’échelle automatique ou manuelle dans un pool de ressources spécifique, accédez à **Paramètres** > **Pool frontal** / **Pools de travail**, selon le cas. Ouvrez ensuite le pool que vous souhaitez modifier. Accédez à **Paramètres** > **Scale-out** ou **Paramètres** > **Scale-up**. Le panneau **Scale-out** vous permet de contrôler la quantité des instances. **Monter en puissance** vous permet de contrôler la taille de la ressource.  

![Interface utilisateur Paramètres de mise à l’échelle][7]

## <a name="fault-tolerance-considerations"></a>Éléments à prendre en compte en matière de tolérance de pannes
Vous pouvez configurer un environnement App Service pour utiliser jusqu’à 55 ressources de calcul au total. De ces 55 ressources de calcul, seules 50 peuvent être utilisées pour héberger des charges de travail. Il y a deux raisons à cela. Il existe au minimum 2 ressources de calcul frontales,  ce qui en laisse 53 au maximum pour prendre en charge d’allocation des pools de travail. Pour fournir une tolérance de panne, une ressource de calcul supplémentaire doit être allouée en respectant les règles suivantes :

* Chaque pool de travail nécessite au moins une ressource de calcul supplémentaire qui n’est pas disponible pour qu’une charge de travail lui soit affectée.
* Lorsque la quantité de ressources de calcul dans un pool de travail dépasse une certaine valeur, une autre ressource de calcul est requise pour la tolérance de panne. Cela n’est pas le cas dans le pool de serveurs frontaux.

Dans tout pool de travaux unique, les conditions requises pour la tolérance de pannes sont que pour une valeur donnée de X ressources affectées à un pool de travaux :

* si X est compris entre 2 et 20, la quantité de ressources de calcul utilisables que vous pouvez utiliser pour les charges de travail est X-1 ;
* si X est compris entre 21 et 40, la quantité de ressources de calcul utilisables que vous pouvez utiliser pour les charges de travail est X-2 ;
* si X est compris entre 41 et 53, la quantité de ressources de calcul utilisables que vous pouvez utiliser pour les charges de travail est X-3.

L’encombrement minimal comprend 2 serveurs frontaux et 2 Workers.  Voici quelques exemples pour clarifier les affirmations ci-dessus :  

* Si vous avez 30 Workers dans un pool unique, 28 d’entre eux peuvent servir à héberger les charges de travail.
* Si vous avez 2 Workers dans un pool unique, 1 peut servir à héberger les charges de travail.
* Si vous avez 20 Workers dans un pool unique, 19 peuvent servir à héberger les charges de travail.  
* Si vous avez 21 Workers dans un pool unique, seuls 19 peuvent servir à héberger les charges de travail.  

L’aspect tolérance de panne est important, et vous devez le garder à l’esprit à mesure que votre mise à l’échelle dépasse certains seuils. Si vous souhaitez augmenter la capacité au-delà de 20 instances, alors passez à 22 instances ou plus, car un nombre de 21 instances n’ajoute pas plus de capacité. Cette remarque est également valable au-delà de 40 instances : le nombre suivant qui ajoute de la capacité est 42.  

## <a name="deleting-an-app-service-environment"></a>Suppression d'un environnement App Service
Si vous voulez supprimer un environnement App Service, utilisez simplement l’action **Supprimer** en haut du panneau Environnement App Service. Lorsque vous effectuez cette opération, vous êtes invité à entrer le nom de votre environnement App Service pour confirmer votre action. Lorsque vous supprimez un environnement App Service, vous supprimez également l’ensemble de son contenu.  

![Interface utilisateur Supprimer un environnement App Service][9]  

## <a name="getting-started"></a>Prise en main
Pour prendre en main les environnements App Service, consultez [Comment créer un environnement App Service](app-service-web-how-to-create-an-app-service-environment.md).

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!--Image references-->
[1]: ./media/app-service-web-configure-an-app-service-environment/ase-icon.png
[2]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-aseblade.png
[3]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-poolchart.png
[4]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-properties.png
[5]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-poolblade.png
[6]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-scalecommand.png
[7]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-poolscale.png
[8]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-pricingtiers.png
[9]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-deletease.png

<!--Links-->
[WhatisASE]: app-service-app-service-environment-intro.md
[Appserviceplans]: ../overview-hosting-plans.md
[HowtoCreateASE]: app-service-web-how-to-create-an-app-service-environment.md
[HowtoScale]: app-service-web-scale-a-web-app-in-an-app-service-environment.md
[ControlInbound]: app-service-app-service-environment-control-inbound-traffic.md
[virtualnetwork]: ../../virtual-network/virtual-networks-faq.md
[AppServicePricing]: https://azure.microsoft.com/pricing/details/app-service/
[ASEAutoscale]: app-service-environment-auto-scale.md
[ExpressRoute]: app-service-app-service-environment-network-configuration-expressroute.md
[ILBASE]: app-service-environment-with-internal-load-balancer.md