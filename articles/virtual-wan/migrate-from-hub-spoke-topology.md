---
title: 'Architecture : Migration vers Azure Virtual WAN'
description: Découvrez comment migrer vers Azure Virtual WAN.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 09/30/2020
ms.author: cherylmc
ms.openlocfilehash: e602905b461e370189cefed706ddc3a47e0199fe
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91839637"
---
# <a name="migrate-to-azure-virtual-wan"></a>Migration vers Azure Virtual WAN

Azure Virtual WAN permet aux entreprises de simplifier leur connectivité internationale afin de tirer parti de l’étendue du réseau mondial de Microsoft. Cet article fournit des détails techniques pour les entreprises souhaitant migrer d’une topologie hub-and-spoke existante managée par le client à une conception tirant parti des hubs Virtual WAN managés par Microsoft.

Pour plus d’informations sur les avantages offerts par Azure Virtual WAN pour les entreprises adoptant un réseau international d’entreprise moderne centré sur le cloud, consultez [Architecture du réseau de transit global Virtual WAN](virtual-wan-global-transit-network-architecture.md).

:::image type="content" source="./media/migrate-from-hub-spoke-topology/hub-spoke.png" alt-text="Hub-and-spoke":::
**Figure : Azure Virtual WAN**

Le modèle de connectivité hub-and-spoke d’Azure a été adopté par des milliers de clients pour tirer parti du comportement de routage transitif par défaut d’Azure Networking afin de créer des réseaux cloud simples et évolutifs. Azure Virtual WAN est basé sur ces concepts et introduit de nouvelles fonctionnalités qui permettent des topologies de connectivité internationale (pas uniquement entre des emplacements locaux et Azure) et qui permettent aussi aux clients de tirer parti de toute l’étendue du réseau Microsoft pour augmenter leurs réseaux mondiaux existants.

Cet article explique comment migrer un environnement hub-and-spoke géré par le client vers une topologie basée sur Azure Virtual WAN.

## <a name="scenario"></a>Scénario

Contoso est une organisation financière internationale qui possède des bureaux en Europe et en Asie. Elle envisage de déplacer ses applications existantes d’un centre de données local vers Azure et a développé une conception de base qui s’appuie sur l’architecture hub-and-spoke gérée par le client, notamment des réseaux virtuels régionaux de type hub pour la connectivité hybride. Dans le cadre du passage aux technologies informatiques, l’équipe réseau a été chargée de veiller à ce que la connectivité soit optimisée pour les activités à venir de l’organisation.

La figure suivante illustre une vue générale du réseau mondial existant, y compris la connectivité à plusieurs régions Azure.

:::image type="content" source="./media/migrate-from-hub-spoke-topology/contoso-pre-migration.png" alt-text="Hub-and-spoke":::
**Figure : La topologie du réseau existant chez Contoso**

Dans la topologie de réseau existante, on peut noter les points suivants :

* Une topologie hub-and-spoke est utilisée dans plusieurs régions, notamment dans les circuits ExpressRoute pour la connectivité à un réseau étendu (WAN) privé commun.

* Certains de ces sites disposent également de tunnels VPN vers Azure pour atteindre les applications hébergées dans le cloud.

## <a name="requirements"></a>Spécifications

L’équipe qui s’occupe du réseau a été chargée de mettre en œuvre un modèle de réseau global capable de prendre en charge la migration de Contoso vers le cloud et d’optimiser les coûts, la mise à l’échelle et les performances. Voici un résumé des exigences à satisfaire :

* Fournir au siège social (HQ) et aux filiales un chemin optimisé vers les applications hébergées dans le cloud
* Supprimer la dépendance aux centres de données locaux existants afin d’arrêter le VPN tout en conservant les chemins de connectivité suivants :
  * **Branche vers réseau virtuel** : Les bureaux connectés à un VPN doivent être en mesure d’accéder aux applications migrées vers le cloud dans la région Azure locale.
  * **Branche vers hub à Hub vers réseau virtuel** : Les bureaux connectés à un VPN doivent être en mesure d’accéder aux applications migrées vers le cloud dans la région Azure distante.
  * **Branche vers branche** : Les bureaux régionaux connectés à un VPN doivent être en mesure de communiquer entre eux et avec les sites HQ/DC ExpressRoute connectés.
  * **Branche vers hub à Hub vers branche** : Les bureaux distants connectés à un VPN doivent être en mesure de communiquer entre eux et avec tous les sites HQ/DC ExpressRoute connectés.
  * **Branche vers Internet** : Les sites connectés doivent être en mesure de communiquer avec Internet. Ce trafic doit être filtré et journalisé.
  * **Réseau virtuel vers réseau virtuel** : Les réseaux virtuels de type spoke d’une même région doivent être en mesure de communiquer entre eux.
  * **Réseau virtuel vers hub à Hub vers réseau virtuel** : Les réseaux virtuels de type spoke de régions différentes doivent être en mesure de communiquer entre eux.
* Offrir aux utilisateurs itinérants de Contoso (ordinateur portable et téléphone) la possibilité d’accéder aux ressources de l’entreprise, même sans connexion au réseau de l’entreprise.

## <a name="azure-virtual-wan-architecture"></a><a name="architecture"></a>Architecture d’Azure Virtual WAN

La figure suivante illustre une vue d’ensemble de la topologie cible mise à jour à l’aide d’Azure Virtual WAN pour répondre aux exigences décrites dans la section précédente.

:::image type="content" source="./media/migrate-from-hub-spoke-topology/vwan-architecture.png" alt-text="Hub-and-spoke":::
**Figure : Architecture d’Azure Virtual WAN**

Résumé :

* Le siège social en Europe reste connecté à ExpressRoute, tandis que les contrôleurs de domaine locaux en Europe ont été entièrement migrés vers Azure, puis désactivés.
* Le contrôleur de domaine et le siège social d’Asie restent connectés à un WAN privé. Azure Virtual WAN est désormais utilisé pour augmenter le réseau de l’opérateur local et fournir une connectivité mondiale.
* Des hubs Azure Virtual WAN ont été déployés dans les régions Azure Europe Ouest et Asie Sud-Est pour fournir un hub de connectivité aux appareils connectés à ExpressRoute et au VPN.
* Les hubs fournissent également un arrêt VPN pour les utilisateurs itinérants sur plusieurs types de clients à l’aide de la connectivité OpenVPN au réseau de maillage global. Ainsi, ils peuvent accéder non seulement aux applications migrées vers Azure, mais également à toutes les ressources locales.
* Connectivité Internet pour les ressources au sein d’un réseau virtuel fournie par Azure Virtual WAN.

Connectivité Internet pour les sites distants également fournie par Azure Virtual WAN. Passerelle de sortie vers Internet (local internet breakout) prise en charge via l’intégration des partenaires pour un accès optimisé aux services SaaS tels que Microsoft 365.

## <a name="migrate-to-virtual-wan"></a>Migrer vers Virtual WAN

Cette section présente les différentes étapes de la migration vers Azure Virtual WAN.

### <a name="step-1-single-region-customer-managed-hub-and-spoke"></a>Étape 1 : Une seule région hub-and-spoke gérée par le client

La figure suivante illustre une topologie à une seule région pour Contoso avant le lancement d’Azure Virtual WAN :

:::image type="content" source="./media/migrate-from-hub-spoke-topology/figure1.png" alt-text="Hub-and-spoke":::
**Figure 1 : Une seule région hub-and-spoke manuelle**

Conformément à l’approche hub-and-spoke, le réseau virtuel hub managé par le client contient plusieurs blocs de fonctions :

* Services partagés (toute fonction commune requise par plusieurs spokes). Exemple : Contoso utilise des contrôleurs de domaine Windows Server sur des machines virtuelles IaaS (infrastructure-as-a-service).
* Les services de pare-feu d’adresse IP/de routage sont fournis par une appliance virtuelle de réseau tierce, ce qui permet un routage d’adresse IP de couche 3 de type « spoke à spoke ».
* Services d’entrée/sortie Internet incluant la passerelle Azure Application pour les requêtes HTTPS entrantes et les services proxy tiers exécutés sur des machines virtuelles pour un accès sortant filtré aux ressources Internet.
* Passerelle de réseau virtuel VPN et ExpressRoute pour la connectivité aux réseaux locaux.

### <a name="step-2-deploy-virtual-wan-hubs"></a>Étape 2 : Déploiement des hubs Virtual WAN

Déployez un hub Virtual WAN dans chaque région. Configurez le hub Virtual WAN avec les fonctionnalités VPN et ExpressRoute, comme décrit dans les articles suivants :

* [Tutoriel : Créer une connexion de site à site à l’aide d’Azure Virtual WAN](virtual-wan-site-to-site-portal.md)
* [Tutoriel : Créer une association ExpressRoute avec Azure Virtual WAN](virtual-wan-expressroute-portal.md)

> [!NOTE]
> Azure Virtual WAN doit utiliser la référence SKU Standard pour activer certains des chemins de trafic indiqués dans cet article.
>

:::image type="content" source="./media/migrate-from-hub-spoke-topology/figure2.png" alt-text="Hub-and-spoke":::
**Figure 2 : Migration d’un hub-and-spoke géré par le client vers Virtual WAN**

### <a name="step-3-connect-remote-sites-expressroute-and-vpn-to-virtual-wan"></a>Étape 3 : Connexion des sites distants (ExpressRoute et VPN) à Virtual WAN

Connectez le hub Virtual WAN aux circuits ExpressRoute existants et configurez des VPN de site à site via Internet vers toutes les branches distantes.

:::image type="content" source="./media/migrate-from-hub-spoke-topology/figure3.png" alt-text="Hub-and-spoke":::
**Figure 3 : Migration d’un hub-and-spoke géré par le client vers Virtual WAN**

À ce stade, l’équipement réseau local commencera à recevoir des itinéraires reflétant l’espace d’adressage IP attribué au réseau virtuel hub managé par Virtual WAN. À ce stade, les branches distantes connectées via un VPN voient deux chemins d’accès à toutes les applications existantes sur les réseaux virtuels spoke. Ces appareils doivent être configurés pour continuer à utiliser le tunnel vers le hub géré par le client pour garantir un routage symétrique pendant la phase de transition.

### <a name="step-4-test-hybrid-connectivity-via-virtual-wan"></a>Étape 4 : Test de la connectivité hybride par le biais de Virtual WAN

Avant d’utiliser le hub Virtual WAN managé pour la connectivité en production, nous vous recommandons de configurer un réseau virtuel spoke de test et une connexion de réseau virtuel Virtual WAN. Vérifiez que les connexions à cet environnement de test fonctionnent via ExpressRoute et le VPN de site à site avant de passer aux étapes suivantes.

:::image type="content" source="./media/migrate-from-hub-spoke-topology/figure4.png" alt-text="Hub-and-spoke":::
**Figure 4 : Migration d’un hub-and-spoke géré par le client vers Virtual WAN**

À ce stade, il est important de reconnaître que le réseau virtuel hub d’origine géré par le client et le nouveau hub Virtual WAN sont tous deux connectés au même circuit ExpressRoute. Grâce à cela, nous disposons d’un chemin d’accès du trafic qui peut être utilisé pour permettre aux spokes des deux environnements de communiquer. Par exemple, le trafic provenant d’un spoke attaché au réseau virtuel hub géré par le client traverse les appareils MSEE utilisés pour le circuit ExpressRoute afin d’atteindre tout spoke connecté via une connexion de réseau virtuel au nouveau hub Virtual WAN. Cela permet une migration intermédiaire des spokes à l’étape 5.

### <a name="step-5-transition-connectivity-to-virtual-wan-hub"></a>Étape 5 : Transition de connectivité vers le hub Virtual WAN

:::image type="content" source="./media/migrate-from-hub-spoke-topology/figure5.png" alt-text="Hub-and-spoke":::
**Figure 5 : Migration d’un hub-and-spoke géré par le client vers Virtual WAN**

**a**. Supprimez les connexions de Peering existantes des réseaux virtuels spoke vers l’ancien hub géré par le client. L’accès aux applications sur les réseaux virtuels spoke n’est pas possible tant que les étapes a à c ne sont pas terminées.

**b**. Connectez les réseaux virtuels spoke au hub Virtual WAN par le biais de connexions de réseau virtuel.

**c**. Supprimez tous les itinéraires définis par l’utilisateur précédemment utilisés sur les réseaux virtuels spoke pour les communications de type spoke à spoke. Ce chemin est désormais accessible par le routage dynamique disponible dans le hub Virtual WAN.

**d**. Les passerelles VPN et ExpressRoute existantes dans le hub géré par le client sont maintenant désactivées pour autoriser le passage à l’étape suivante (e).

**e**. Connectez l’ancien hub géré par le client (réseau virtuel hub) au hub Virtual WAN par le biais d’une nouvelle connexion de réseau virtuel.

### <a name="step-6-old-hub-becomes-shared-services-spoke"></a>Étape 6 : L’ancien hub devient un spoke de services partagés

Nous avons maintenant repensé notre réseau Azure pour que le hub Virtual WAN soit le point central de notre nouvelle topologie.

:::image type="content" source="./media/migrate-from-hub-spoke-topology/figure6.png" alt-text="Hub-and-spoke":::
**Figure 6 : Migration d’un hub-and-spoke géré par le client vers Virtual WAN**

Étant donné que le hub Virtual WAN est une entité managée et qu’il n’autorise pas le déploiement de ressources personnalisées telles que des machines virtuelles, le bloc de services partagés existe désormais en tant que réseau virtuel spoke et héberge des fonctions telles que l’entrée Internet par le biais de la passerelle Azure Application ou de l’appliance virtualisée de réseau. Le trafic entre l’environnement de services partagés et les machines virtuelles back-end transite désormais par le hub managé par Virtual WAN.

### <a name="step-7-optimize-on-premises-connectivity-to-fully-utilize-virtual-wan"></a>Étape 7 : Optimisation de la connectivité locale pour exploiter pleinement Virtual WAN

À ce niveau, Contoso a migré la plupart de ses applications métier dans le cloud Microsoft. Il ne reste plus que quelques applications héritées dans le contrôleur de domaine local.

:::image type="content" source="./media/migrate-from-hub-spoke-topology/figure7.png" alt-text="Hub-and-spoke":::
**Figure 7 : Migration d’un hub-and-spoke géré par le client vers Virtual WAN**

Pour tirer parti de toutes les fonctionnalités d’Azure Virtual WAN, Contoso a décidé de désactiver ses connexions VPN locales héritées. Toutes les branches qui continuent d’accéder aux réseaux du siège social ou du contrôleur de domaine sont en mesure de transiter par le réseau global de Microsoft à l’aide du routage de transit intégré à Azure Virtual WAN.

> [!NOTE]
> ExpressRoute Global Reach est requis pour les clients qui souhaitent tirer parti du réseau principal de Microsoft pour fournir un transit ExpressRoute vers ExpressRoute (non représenté dans la figure 7).
>

## <a name="end-state-architecture-and-traffic-paths"></a>Architecture finalisée et chemins du trafic

:::image type="content" source="./media/migrate-from-hub-spoke-topology/figure8.png" alt-text="Hub-and-spoke":::
**Figure : Virtual WAN à deux régions**

Cette section fournit un résumé de la façon dont cette topologie répond aux exigences d’origine en examinant des exemples de flux de trafic.

### <a name="path-1"></a>Chemin 1

Le chemin 1 affiche le flux du trafic entre une branche connectée VPN site à site en Asie et un réseau virtuel Azure dans la région Asie Sud-Est.

Le trafic est routé comme suit :

* La branche Asie est connectée par le biais de tunnels BGP site à site résilients dans le hub Virtual WAN de la région Asie Sud-Est.

* Le hub Virtual WAN en Asie achemine le trafic localement vers le réseau virtuel connecté.

:::image type="content" source="./media/migrate-from-hub-spoke-topology/flow1.png" alt-text="Hub-and-spoke":::

### <a name="path-2"></a>Chemin 2

Le chemin 2 affiche le flux du trafic entre le siège en Europe connecté via ExpressRoute et le réseau virtuel Azure de la région Asie Sud-Est.

Le trafic est routé comme suit :

* Le siège européen est connecté via le circuit ExpressRoute au hub Virtual WAN de la région Europe Ouest.

* La connectivité globale de hub à hub Virtual WAN permet un transit du trafic vers le réseau virtuel connecté dans une région distante.

:::image type="content" source="./media/migrate-from-hub-spoke-topology/flow2.png" alt-text="Hub-and-spoke":::

### <a name="path-3"></a>Chemin 3

Le chemin 3 affiche le flux du trafic entre le centre de données local en Asie connecté au WAN privé jusqu’à une branche européenne connectée site à site.

Le trafic est routé comme suit :

* Le contrôleur de domaine en Asie est connecté à l’opérateur du WAN privé local.

* Le circuit ExpressRoute se termine localement là où le WAN privé se connecte au hub Virtual WAN de la région Asie Sud-Est.

* La connectivité mondiale hub à hub Virtual WAN permet le transit du trafic.

:::image type="content" source="./media/migrate-from-hub-spoke-topology/flow3.png" alt-text="Hub-and-spoke":::

### <a name="path-4"></a>Chemin 4

Le chemin 4 affiche le flux du trafic entre un réseau virtuel Azure de la région Asie Sud-Est et un réseau virtuel Azure de la région Europe Ouest.

Le trafic est routé comme suit :

* La connectivité globale de hub à hub Virtual WAN permet le transit natif de tous les réseaux virtuels Azure connectés sans configuration utilisateur supplémentaire.

:::image type="content" source="./media/migrate-from-hub-spoke-topology/flow4.png" alt-text="Hub-and-spoke":::

### <a name="path-5"></a>Chemin 5

Le chemin 5 affiche le flux du trafic entre les utilisateurs VPN (P2S) itinérants et un réseau virtuel Azure de la région Europe Ouest.

Le trafic est routé comme suit :

* Les utilisateurs d’ordinateurs portables et d’appareils mobiles utilisent le client OpenVPN pour une connectivité transparente vers la passerelle VPN P2S de la région Europe Ouest.

* Le hub Virtual WAN de la région Europe Ouest achemine le trafic localement vers le réseau virtuel connecté.

:::image type="content" source="./media/migrate-from-hub-spoke-topology/flow5.png" alt-text="Hub-and-spoke":::

## <a name="security-and-policy-control-via-azure-firewall"></a>Sécurité et contrôle de stratégie via le pare-feu Azure

Contoso dispose maintenant d’une connectivité validée entre toutes les branches et tous les réseaux virtuels, qui est conforme aux exigences abordées précédemment dans cet article. Pour répondre à ses exigences en matière de contrôle de sécurité et d’isolement réseau, l’entreprise doit continuer à séparer et à enregistrer le trafic via le réseau hub. Auparavant, cette fonction était exécutée par une appliance virtuelle réseau (NVA). Contoso souhaite également désactiver ses services proxy existants et utiliser les services Azure natifs pour le filtrage Internet sortant.

:::image type="content" source="./media/migrate-from-hub-spoke-topology/security-policy.png" alt-text="Hub-and-spoke":::
**Figure : Pare-feu Azure dans Virtual WAN (hub virtuel sécurisé)**

Les étapes générales suivantes sont requises pour introduire Pare-feu Azure dans les hubs Virtual WAN afin d’obtenir un point unifié de contrôle des stratégies. Pour plus d’informations sur ce processus et le concept de hubs virtuels sécurisés, consultez [Azure Firewall Manager](../firewall-manager/index.yml).

1. Créez une stratégie de pare-feu Azure.
2. Liez la stratégie de pare-feu au hub Azure Virtual WAN. Cette étape permet au hub Virtual WAN existant de fonctionner en tant que hub virtuel sécurisé et déploie les ressources Pare-feu Azure nécessaires.

> [!NOTE]
> Il existe des contraintes relatives à l’utilisation de hubs virtuels sécurisés, notamment le trafic interrégional. Pour plus d’informations, consultez [Firewall Manager : problèmes connus](../firewall-manager/overview.md#known-issues).
>

Les chemins d’accès suivants montrent les chemins de connectivité activés grâce aux hubs virtuels sécurisés Azure :

### <a name="path-6"></a>Chemin 6

Le chemin 6 affiche le flux du trafic sécurisé entre les réseaux virtuels de la même région.

Le trafic est routé comme suit :

* Les réseaux virtuels connectés au même hub virtuel sécurisé acheminent désormais le trafic via le pare-feu Azure.

* Le pare-feu Azure peut appliquer une stratégie à ces flux.

:::image type="content" source="./media/migrate-from-hub-spoke-topology/flow6.png" alt-text="Hub-and-spoke":::

### <a name="path-7"></a>Chemin 7

Le chemin 7 affiche le flux du trafic entre un réseau virtuel Azure et Internet ou un service de sécurité tiers.

Le trafic est routé comme suit :

* Les réseaux virtuels connectés au hub virtuel sécurisé peuvent envoyer du trafic au public (des destinations sur Internet) en utilisant le hub sécurisé comme point central d’accès à Internet.

* Ce trafic peut être filtré localement à l’aide des règles FQDN de Pare-feu Azure ou envoyé à un service de sécurité tiers à des fins d’inspection.

:::image type="content" source="./media/migrate-from-hub-spoke-topology/flow7.png" alt-text="Hub-and-spoke":::

### <a name="path-8"></a>Chemin 8

Le chemin 8 affiche le flux du trafic d’une branche vers Internet ou un service de sécurité tiers.

Le trafic est routé comme suit :

* Les branches connectées au hub virtuel sécurisé peuvent envoyer du trafic vers des destinations publiques sur Internet en utilisant le hub sécurisé comme point central d’accès à Internet.

* Ce trafic peut être filtré localement à l’aide des règles FQDN de Pare-feu Azure ou envoyé à un service de sécurité tiers à des fins d’inspection.

:::image type="content" source="./media/migrate-from-hub-spoke-topology/flow8.png" alt-text="Hub-and-spoke":::

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur [Azure Virtual WAN](virtual-wan-about.md).
