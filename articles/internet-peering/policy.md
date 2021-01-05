---
title: Stratégie de Peering Microsoft
titleSuffix: Azure
description: Stratégie de Peering Microsoft
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: conceptual
ms.date: 12/15/2020
ms.author: prmitiki
ms.openlocfilehash: bee41bb8e5beb4df3086ab50499cb185a83e4efe
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97592327"
---
# <a name="peering-policy"></a>Stratégie de peering
Microsoft gère une stratégie de peering sélective conçue pour garantir la meilleure expérience client possible adossée à des normes sectorielles et des meilleures pratiques, adaptant son échelle à la demande future et au positionnement stratégique du peering. Par conséquent, Microsoft se réserve le droit de faire des exceptions à la stratégie si elles sont jugées nécessaires. Les exigences générales de Microsoft concernant votre réseau sont décrites dans les sections ci-dessous. Elles s’appliquent à la fois aux requêtes de Peering direct et de Peering Exchange. 

## <a name="technical-requirements"></a>Exigences techniques

* Un réseau entièrement redondant avec une capacité suffisante pour échanger du trafic sans congestion.
* L’homologue aura un numéro de système autonome (NSA) routable publiquement.
* Les protocoles IPv4 et IPv6 sont pris en charge, et Microsoft s’attend à établir des sessions des deux types dans chaque emplacement de Peering.
* MD5 n’est pas pris en charge.
* **Détails concernant les NSA :**

    * Microsoft gère AS8075 avec les NSA suivants : AS8068, AS8069, AS12076. Pour obtenir la liste complète des NSA avec Peering AS8075, consultez AS-SET MICROSOFT.
    * Toutes les parties qui établissent un peering avec Microsoft s’engagent à n’accepter en aucune circonstance les routes à partir de AS12076 (Express Route), et doivent filtrer AS12076 sur tous les homologues.

* **Stratégie de routage :**
    * L’homologue aura au moins un bloc /24 routable public.
    * Microsoft remplacera les MED (Multi-Exit Discriminators) reçus.
    * Microsoft préfère recevoir des balises de communauté BGP en provenance des homologues pour indiquer l’origine de la route.
    * Nous recommandons aux homologues de définir un max-prefix de 2000 (IPv4) et 500 (IPv6) routes sur les sessions de peering avec Microsoft.
    * Sauf accord spécifique préalable, les homologues sont censés annoncer des routes cohérentes dans tous les emplacements où ils établissent un Peering avec Microsoft.
    * En général, les sessions de Peering avec AS8075 publieront toutes les routes AS-MICROSOFT. Microsoft peut annoncer certaines spécificités régionales.
    * Aucune des parties n’établira une route statique, une route de dernier recours, ni n’enverra le trafic à l’autre partie pour une route non annoncée par le biais du protocole BGP.
    * L’homologue doit inscrire ses routes dans une base de données de routage Internet (IRR, Internet Routing Registry) publique, à des fins de filtrage, et tiendra ces informations à jour.      
    * Les homologues adhèrent aux normes sectorielles MANRS pour la sécurité des itinéraires.  Microsoft peut choisir à sa seule discrétion : 1.) de ne pas établir de peering avec des sociétés qui n’ont pas d’itinéraires signés et inscrits ; 2.) de supprimer des itinéraires RPKI non valides ; 3.) de ne pas accepter d’itinéraires d’homologues établis qui ne sont pas inscrits et signés. 

## <a name="operational-requirements"></a>Exigences opérationnelles
* Un centre d’opérations réseau 24x7, capable d’aider à la résolution de tous les problèmes techniques et de performances, les violations de sécurité, les attaques par déni de service ou tout autre abus provenant de l’homologue ou de ses clients.
* Les homologues sont censés disposer d’un profil complet et à jour sur [PeeringDB](https://www.peeringdb.com), y compris un e-mail NOC 24x7 du domaine d’entreprise et un numéro de téléphone. Nous utilisons ces informations pour valider les détails de l’homologue, tels que les informations NOC, les informations de contact technique, leur présence dans les installations de Peering, et ainsi de suite. Les comptes personnels Yahoo, Gmail et Hotmail ne sont pas acceptés.

## <a name="physical-connection-requirements"></a>Exigences relatives aux connexions physiques
* Les emplacements où vous pouvez vous connecter à Microsoft pour le Peering direct ou le Peering Exchange sont listés dans [PeeringDB](https://www.peeringdb.com/net/694).

* **Peering Exchange :**
    * Les homologues doivent avoir au minimum une connexion de 10 Go à l’échange.
    * Les homologues sont censés mettre à niveau leurs ports quand le pic d’utilisation dépasse 50 %.
    * Microsoft encourage les homologues à maintenir une connectivité diversifiée à l’échange pour prendre en charge les scénarios de basculement.

* **Peering direct :**
    * L’interconnexion doit être établie sur une fibre optique monomode offrant un débit de 100 Gbits/s.
    * Microsoft n’établira un Peering direct qu’avec des fournisseurs de services Internet (ISP) ou des fournisseurs de services réseau.
    * Les homologues sont censés mettre à niveau leurs ports quand le pic d’utilisation dépasse 50 %, et maintenir une capacité diversifiée dans chaque métro, soit dans un même emplacement, soit parmi plusieurs emplacements dans un métro.
    * Chaque Peering direct se compose de deux connexions à deux routeurs Microsoft Edge à partir des routeurs de l’homologue situés dans la périphérie de l’homologue. Microsoft exige des sessions BGP doubles sur ces connexions. L’homologue peut choisir de ne pas déployer d’appareils redondants à son extrémité.


## <a name="traffic-requirements"></a>Exigences relatives au trafic

* Les homologues sur un peering d’échange doivent avoir au moins 500 Mo de trafic, et au plus 2 Go. Pour le trafic qui dépasse 2 Go, un peering direct doit être établi.
* Microsoft requiert au moins 2 Go pour le peering direct. Chaque emplacement de peering convenu mutuellement doit prendre en charge un basculement qui garantit que le peering reste localisé au cours d’un scénario de basculement. 

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les étapes de configuration du Peering direct avec Microsoft, suivez la [procédure pas à pas de configuration du Peering direct](walkthrough-direct-all.md).
* Pour plus d’informations sur les étapes de configuration d’un Peering Exchange avec Microsoft, suivez la [procédure pas à pas de configuration d’un Peering Exchange](walkthrough-exchange-all.md).
