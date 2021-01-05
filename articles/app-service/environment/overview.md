---
title: Vue d’ensemble d’App Service Environment
description: Vue d’ensemble de l’environnement App Service Environment
author: ccompy
ms.assetid: 3d37f007-d6f2-4e47-8e26-b844e47ee919
ms.topic: article
ms.date: 11/16/2020
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: fbc498fcd654d16936c2548528e2600be68a2ad9
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94663268"
---
# <a name="app-service-environment-overview"></a>Vue d’ensemble d’App Service Environment 

> [!NOTE]
> Cet article traite d’App Service Environment v3 (préversion)
> 

L’environnement Azure App Service est une fonctionnalité d’Azure App Service qui fournit un environnement totalement isolé et dédié pour l’exécution sécurisée de vos applications App Service à grande échelle. Cette fonctionnalité peut héberger vos :

- Applications web Windows
- Applications web Linux
- Conteneurs Docker
- Fonctions

Les environnements App Service (ASE) conviennent aux charges de travail d’application qui nécessitent :

- Scalabilité élevée.
- Une isolation et un accès réseau sécurisé.
- Une utilisation élevée de la mémoire.
- Nombre élevé de demandes par seconde (RPS). Vous pouvez créer plusieurs environnements ASE au sein d’une même région Azure ou dans plusieurs régions Azure. Grâce à cette souplesse, les environnements ASE sont parfaits pour la scalabilité horizontale des applications sans état avec des exigences RPS élevées.

L’environnement ASE héberge des applications d’un seul client, et le fait sur l’un de ses réseaux virtuels. Les clients peuvent contrôler précisément le trafic réseau entrant et sortant des applications. Les applications peuvent établir des connexions sécurisées à haute vitesse aux ressources d’entreprise locales par le biais de réseaux privés virtuels.

ASEv3 est fourni avec son propre niveau tarifaire, Isolé V2.
Les environnements App Service Environment v3 fournissent un cadre qui protège vos applications dans un sous-réseau de votre réseau, et assurent votre propre déploiement privé d’Azure App Service.
Vous pouvez utiliser plusieurs environnements App Service pour une mise à l’échelle horizontale. L’accès aux applications qui s’exécutent sur des environnements App Service peut être contrôlé par des appareils en amont tels que les pare-feu d’applications web (WAF). Pour plus d’informations, consultez l’article Pare-feu d’applications web (WAF).

## <a name="usage-scenarios"></a>Scénarios d’usage

L’environnement App Service Environment peut être utilisé dans de nombreux cas, notamment :

- Applications métier internes
- Applications qui ont besoin de plus de 30 instances ASP
- Système monolocataire pour respecter les exigences de conformité ou de sécurité interne
- Hébergement d’application isolée du réseau
- Applications multicouches

Il existe un certain nombre de fonctionnalités de mise en réseau qui permettent aux applications dans l’instance App Service multilocataire d’atteindre des ressources isolées du réseau ou d’être isolées du réseau elles-mêmes. Ces fonctionnalités sont activées au niveau de l’application.  Avec un environnement ASE, aucune configuration supplémentaire sur les applications n’est nécessaire pour les inclure dans le réseau virtuel. Les applications sont déployées dans un environnement isolé du réseau qui se trouve déjà dans un réseau virtuel. Non seulement l’environnement ASE héberge des applications isolées du réseau, mais il s’agit également d’un système monolocataire. Aucun autre client n’utilise l’environnement ASE. Si vous avez vraiment besoin d’un scénario d’isolation complet, vous pouvez également déployer votre environnement ASE sur du matériel dédié. Entre l’hébergement d’applications isolées du réseau, la caractéristique monolocataire et la capacité 

## <a name="dedicated-environment"></a>Environnement dédié
Un environnement ASE est dédié exclusivement à un seul abonnement et peut héberger 200 instances de plan App Service. Il peut s’agir de 100 instances d’un même plan App Service ou de 100 plans App Service à instance unique, ou de toutes les combinaisons possibles des deux.

Un environnement App Service est composé de frontends et de workers. Les frontends sont responsables de la terminaison du protocole HTTP/HTTPS et de l’équilibrage de charge automatique des demandes d’application dans un environnement App Service. Ils sont ajoutés automatiquement à mesure que les plans App Service dans l’environnement App Service augmentent leur échelle.

Les workers sont des rôles qui hébergent des applications clientes. Ils sont disponibles dans trois tailles fixes :

- Deux processeurs virtuels/8 Go de RAM
- Quatre processeurs virtuels/16 Go de RAM
- Huit processeurs virtuels/32 Go de RAM

Les clients n’ont pas besoin de gérer les front-ends et les workers. Toute l’infrastructure est automatique. À mesure que des plans App Service sont créés ou mis à l’échelle dans un environnement App Service, l’infrastructure requise est ajoutée ou supprimée en conséquence.

Les instances de plan App Service Isolé v2 sont sujettes à des frais. Si vous n’avez pas de plan App Service dans votre environnement ASE, vous êtes facturé comme si vous aviez un plan App Service avec une instance des deux principaux workers.

## <a name="virtual-network-support"></a>Prise en charge des réseaux virtuels
La fonctionnalité ASE est un déploiement d’Azure App Service effectué directement dans le réseau virtuel Azure Resource Manager d’un client. Un environnement ASE existe toujours dans un sous-réseau d’un réseau virtuel. Vous pouvez utiliser les fonctionnalités de sécurité des réseaux virtuels pour contrôler les communications réseau entrantes et sortantes de vos applications.

Les groupes de sécurité réseau limitent les communications réseau entrantes vers le sous-réseau sur lequel réside l’environnement App Service. Vous pouvez utiliser des groupes de sécurité réseau pour exécuter des applications derrière des appareils et services en amont tels que des pare-feu d’applications web et des fournisseurs SaaS réseau.

Les applications doivent souvent accéder à des ressources d'entreprise telles que des bases de données internes et des services web. Si vous déployez l’environnement App Service sur un réseau virtuel qui a une connexion VPN au réseau local, les applications de l’environnement App Service peuvent accéder aux ressources locales. Cela est vrai, que le réseau privé virtuel soit un VPN de site à site ou un VPN Azure ExpressRoute.

## <a name="preview"></a>Préversion
App Service Environment v3 est en préversion publique.  Certaines fonctionnalités sont ajoutées à mesure de l’évolution de la préversion. Les limites actuelles d’ASEv3 incluent :

- Impossibilité à mettre à l’échelle un plan App Service au-delà de cinq instances
- Impossibilité d’obtenir un conteneur à partir d’un registre privé
- Impossibilité des fonctionnalités App Service actuellement non prises en charge de passer par le réseau virtuel client
- Aucun modèle de déploiement externe avec un point de terminaison accessible sur Internet
- Pas de prise en charge de la ligne de commande (AZ CLI et PowerShell)
- Aucune possibilité de mise à niveau d’ASEv2 vers ASEv3
- Pas de prise en charge FTP
- Pas de prise en charge du passage de certaines fonctionnalités App Service vers le réseau virtuel client. La sauvegarde/restauration, les références Key Vault dans les paramètres d’application, l’utilisation d’un registre de conteneurs privé et la journalisation des diagnostics dans le stockage ne fonctionnent pas avec les points de terminaison de service ou les points de terminaison privés
    
### <a name="asev3-preview-architecture"></a>Architecture de la préversion d’ASEv3
Dans la préversion d’ASEv3, l’environnement ASE utilise des points de terminaison privés pour prendre en charge le trafic entrant. Le point de terminaison privé sera remplacé par des équilibreurs de charge dans la version en disponibilité générale. En préversion, l’environnement ASE n’offre pas de prise en charge intégrée pour un point de terminaison accessible via Internet. Pour ce faire, vous pouvez ajouter une passerelle Application Gateway. L’environnement ASE nécessite d’avoir des ressources dans deux sous-réseaux.  Le trafic entrant est transmis via un point de terminaison privé. Le point de terminaison privé peut être placé dans n’importe quel sous-réseau, à condition qu’il dispose d’une adresse disponible pouvant être utilisée par les points de terminaison privés.  Le sous-réseau sortant doit être vide et délégué à Microsoft.Web/hostingEnvironments. Lorsqu’il est utilisé par l’environnement ASE, le sous-réseau sortant ne peut pas être utilisé pour autre chose.

Avec ASEv3, il n’existe aucune exigence de mise en réseau du trafic entrant ou sortant sur le sous-réseau ASE. Vous pouvez contrôler le trafic avec des groupes de sécurité réseau et des tables de routage, et cela affecte uniquement le trafic de votre application. Ne supprimez pas le point de terminaison privé associé à votre environnement ASE, car cette action ne peut pas être annulée. Le point de terminaison privé utilisé pour l’environnement ASE est utilisé pour toutes les applications dans celui-ci. 
