---
title: 'Azure ExpressRoute : Routage asymétrique'
description: Cet article présente des solutions aux problèmes courants en cas de routage asymétrique dans un réseau doté de plusieurs connexions vers une même destination.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: article
ms.date: 12/14/2020
ms.author: duau
ms.openlocfilehash: 0713c52c7f07db93d9ae9084062ef2c3b25a9074
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97560506"
---
# <a name="asymmetric-routing-with-multiple-network-paths"></a>Routage asymétrique avec chemins d’accès réseau multiples
Cet article explique de quelle façon le trafic peut emprunter différentes voies lorsque plusieurs itinéraires sont disponibles entre la source et la destination réseau.

Il existe deux concepts que vous devez connaître pour comprendre le routage asymétrique. Le premier est l’effet de plusieurs chemins d’accès réseau. Le deuxième est la manière dont les appareils (par exemple, un pare-feu) maintiennent l’état. Ces types d’appareils sont appelés appareils avec état. Lorsque ces deux facteurs sont combinés, ils peuvent créer un scénario dans lequel le trafic est annulé par l’appareil avec état.  Le trafic est annulé, car il n’a pas détecté qu’il provenait de lui-même.

## <a name="multiple-network-paths"></a>Chemins d’accès réseau multiples
Lorsqu’un réseau d’entreprise ne dispose que d’une seule connexion à Internet fournie via le fournisseur d’accès à Internet, l’intégralité du trafic Internet entrant et sortant transite par le même chemin. Il est courant que les entreprises acquièrent plusieurs circuits afin de créer des chemins redondants et d’améliorer la disponibilité de leur réseau. Avec ce type de configuration, il est possible que le trafic sorte par une connexion à Internet et revienne par une connexion différente. Ce scénario est communément appelé routage asymétrique. Dans le routage asymétrique, le trafic de retour emprunte un chemin différent du flux sortant d’origine.

![Réseau avec plusieurs chemins d’accès](./media/expressroute-asymmetric-routing/AsymmetricRouting3.png)

Bien que le routage asymétrique se produise généralement lors de l’accès à Internet, il se produit également lorsqu’une combinaison de plusieurs chemins est introduite. Le premier exemple est lorsque vous avez un chemin Internet et un chemin privé qui accèdent à la même destination. Le deuxième exemple est lorsque vous avez plusieurs chemins privés qui accèdent également à la même destination.

Chaque routeur le long du chemin entre la source et la destination calcule le meilleur chemin à prendre pour atteindre la destination. Le routeur détermine le meilleur chemin possible en fonction de deux facteurs principaux :

* Le routage entre des réseaux externes repose sur le protocole de routage BGP (protocole de passerelle frontière). Le BGP collecte les annonces voisines et les soumet à plusieurs opérations afin de déterminer le meilleur chemin d’accès à la destination prévue. Il stocke le meilleur chemin d’accès dans sa table de routage.
* La longueur d’un masque de sous-réseau associé à un itinéraire influence les chemins de routage. Si un routeur reçoit plusieurs publications pour la même adresse IP, le routeur sélectionne le chemin présentant le masque de sous-réseau le plus long, car l’itinéraire est considéré comme plus spécifique.

## <a name="stateful-devices"></a>Appareils avec état
Les routeurs examinent l’en-tête IP d’un paquet pour les besoins du routage. Certains appareils procèdent à une analyse plus en profondeur du paquet. En règle générale, ces appareils examinent les en-têtes de la couche 4 (protocole TCP [Transmission Control Protocol] ou UDP [User Datagram Protocol]), ou même de la couche 7 (couche Application). Ces types d’appareils sont soit des appareils de sécurité, soit des appareils d’optimisation de la bande passante. 

Un pare-feu est un exemple courant d’appareil avec état. Un pare-feu autorise ou rejette les paquets traversant ses interfaces en fonction de différents critères. Ces critères incluent notamment les en-têtes de protocole, de port TCP/UDP et d’URL. Ce niveau d’inspection des paquets peut soumettre l’appareil à une forte charge de traitement. 

Afin d’améliorer les performances, le pare-feu inspecte le premier paquet d’un flux. S’il autorise le paquet à traverser ses interfaces, il conserve les informations de flux dans sa table des états. Les paquets résultants associés à ce flux sont ensuite autorisés en fonction de la détermination initiale. Un paquet qui fait partie d’un flux existant peut arriver au pare-feu dont il ne provient pas. Dans la mesure où il ne dispose d’aucune information d’état préalable sur le flux initial, le pare-feu annule le paquet.

## <a name="asymmetric-routing-with-expressroute"></a>Routage asymétrique avec ExpressRoute
Lorsque vous vous connectez à Microsoft via Azure ExpressRoute, votre réseau change de la manière suivante :

* Vous disposez de plusieurs connexions à Microsoft. L’une d’elles est votre connexion Internet existante, et l’autre est votre connexion ExpressRoute. Certains trafics destinés à Microsoft peuvent passer par la connexion Internet, mais revenir via votre connexion ExpressRoute. Cela peut également se produire lorsque le trafic passe par ExpressRoute, mais revient par le chemin Internet.
* Vous avez reçu des adresses IP plus spécifiques du circuit ExpressRoute. Ainsi, lorsque le trafic de votre réseau accède à Microsoft pour des services offerts par le biais d’ExpressRoute, vos routeurs préfèrent toujours la connexion ExpressRoute.

Pour comprendre l’impact de ces deux modifications sur un réseau, étudions quelques scénarios. Supposons que vous disposez d’un circuit de connexion à Internet et que vous utilisez tous les services Microsoft par le biais d’Internet. Le trafic circulant de votre réseau depuis et vers Microsoft passe par la même connexion Internet et transite par un pare-feu. Le pare-feu enregistre le flux lorsqu’il voit le premier paquet. Chaque paquet résultant de cette conversation est autorisé, car le flux existe dans la table des états.

![Routage asymétrique avec ExpressRoute](./media/expressroute-asymmetric-routing/AsymmetricRouting1.png)

Vous affichez ensuite un circuit ExpressRoute pour consommer les services proposés par Microsoft via ExpressRoute. Tous les autres services de Microsoft sont consommés via Internet. Vous déployez un pare-feu distinct sur votre périphérie connecté à la connexion ExpressRoute. Microsoft publie des préfixes plus spécifiques à votre réseau via ExpressRoute pour certains services. Votre infrastructure de routage choisit ExpressRoute comme chemin d’accès par défaut pour ces préfixes. 

Si vous ne publiez pas vos IP publiques auprès de Microsoft via ExpressRoute, Microsoft communiquera avec vos IP publiques via Internet. Le trafic envoyé de votre réseau à Microsoft utilise la connexion ExpressRoute, mais le trafic de retour de Microsoft utilisera ensuite le chemin Internet. Lorsque le pare-feu situé à votre périphérie voit un paquet de réponse pour un flux qu’il ne connaît pas, il annule ces paquets.

Si vous choisissez de publier le même pool de traduction d’adresses réseau (NAT) pour ExpressRoute et Internet, vous constaterez des problèmes similaires avec les clients de votre réseau sur les adresses IP privées. Les requêtes de services tels que Windows Update passeront par Internet, car les adresses IP pour ces services ne sont pas publiées via ExpressRoute. Toutefois, le trafic de retour reviendra via ExpressRoute. Comme Microsoft a reçu une adresse IP avec le même masque de sous-réseau à partir d’Internet et d’ExpressRoute, le chemin d’accès par défaut est toujours ExpressRoute. Si un pare-feu ou un autre appareil avec état situé à la périphérie de votre réseau et faisant face à la connexion ExpressRoute ne dispose d’aucune information préalable sur un flux, il annulera ces paquets.

## <a name="asymmetric-routing-solutions"></a>Solutions de routage asymétrique
Deux options sont disponibles pour résoudre le problème de routage asymétrique. L’une est le routage et l’autre est l’utilisation d’une NAT basée sur la source (SNAT).

### <a name="routing"></a>Routage
Assurez-vous que vos IP publiques sont publiées sur les connexions WAN appropriées. Par exemple, si vous souhaitez utiliser Internet pour le trafic d’authentification et ExpressRoute pour votre trafic de messagerie, ne publiez pas vos IP publiques des services de fédération Active Directory (AD FS) sur ExpressRoute. Veillez également à ne pas exposer votre serveur AD FS local aux adresses IP reçues par le routeur via ExpressRoute. Les itinéraires reçus via ExpressRoute étant plus spécifiques, ils entraînent la prévalence d’ExpressRoute comme chemin du trafic d’authentification vers Microsoft. Si vous ne faites pas attention à la façon dont le routage s’effectue dans votre réseau, des problèmes de routage asymétrique peuvent survenir.

Si vous souhaitez utiliser ExpressRoute pour l’authentification, assurez-vous que vous publiez les IP publiques AD FS via ExpressRoute sans NAT. Dans cette configuration, le trafic provenant de Microsoft est dirigé vers votre serveur AD FS local via ExpressRoute. Le trafic de retour de votre réseau qui est dirigé vers Microsoft utilisera ExpressRoute, car il s’agit de l’itinéraire de prédilection par rapport à Internet.

### <a name="source-based-nat"></a>NAT basé sur la source
Une autre façon de résoudre le problème de routage asymétrique consiste à utiliser une SNAT. Par exemple, vous choisissez de ne pas publier l’IP publique d’un serveur SMTP (Simple Mail Transfer Protocol) local via ExpressRoute. Au lieu de cela, vous avez l’intention d’utiliser Internet pour ce type de communication. Une requête émanant de Microsoft qui accède à votre serveur SMTP local transite via Internet. Vous définissez l’adresse SNAT de la requête entrante sous une adresse IP interne. Le trafic de retour du serveur SMTP sera dirigé vers le pare-feu de périmètre (utilisé pour la NAT) au lieu de passer via ExpressRoute. En conséquence, le trafic de retour prendra le chemin Internet.

![Configuration réseau avec NAT basée sur la source](./media/expressroute-asymmetric-routing/AsymmetricRouting2.png)

## <a name="asymmetric-routing-detection"></a>Détection du routage asymétrique
La détermination d’itinéraire constitue le meilleur moyen de vous assurer que le trafic réseau circule par le chemin attendu. Si vous souhaitez que le trafic entre votre serveur SMTP local et Microsoft passe par Internet, déterminez son itinéraire attendu à partir du serveur SMTP vers Microsoft 365. Le résultat valide le passage du trafic sortant de votre réseau par Internet et non par ExpressRoute.

