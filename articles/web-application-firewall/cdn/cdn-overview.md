---
title: Qu’est-ce que le pare-feu d’applications web Azure pour Azure CDN ?
description: Découvrez comment le pare-feu d’applications web Azure sur le service Azure CDN protège vos applications web contre les attaques malveillantes.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: victorh
ms.openlocfilehash: 6949c1e8f83ebf47878a3d449796ccc03920756a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89225150"
---
# <a name="azure-web-application-firewall-on-azure-content-delivery-network"></a>Pare-feu d’applications web Azure sur Azure Content Delivery Network

Le pare-feu d’applications web (WAF) Azure sur Azure Content Delivery Network (CDN) de Microsoft offre une protection centralisée pour votre contenu web. WAF protège vos services web contre les vulnérabilités et les attaques courantes. Il assure la haute disponibilité de votre service pour vos utilisateurs et vous aide à répondre aux exigences de conformité.

> [!IMPORTANT]
> WAF sur Azure CDN de Microsoft est actuellement en préversion publique et est fourni avec un contrat de niveau de service en préversion. Certaines fonctionnalités peuvent être limitées ou non prises en charge.  Consultez les [Conditions d’utilisation supplémentaires des préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

WAF sur Azure CDN est une solution mondiale et centralisée. Il est déployé sur les sites de périphérie du réseau Azure dans le monde entier. WAF empêche les attaques malveillantes proches des sources d’attaque avant qu’elles atteignent votre origine. Vous bénéficiez d’une protection globale à grande échelle sans sacrifier les performances. 

Une stratégie WAF est facile à lier à un point de terminaison CDN dans votre abonnement. De nouvelles règles peuvent être déployées en quelques minutes : vous pouvez donc répondre rapidement aux modèles de menaces changeants.

![Pare-feu d’applications web (WAF) d’Azure](../media/cdn-overview/waf-cdn-overview.png)

## <a name="waf-policy-and-rules"></a>Stratégie et règles WAF

Vous pouvez configurer une stratégie WAF et l’associer à un ou plusieurs points de terminaison CDN pour la protection. Une stratégie WAF se compose de deux types de règles de sécurité :

- des règles personnalisées que vous pouvez créer.

- des ensembles de règles managées qui rassemblent des règles préconfigurées managées par Azure.

Quand ces deux types de règles sont utilisés conjointement, les règles personnalisées sont appliquées avant celles d’un ensemble de règles managées. Une règle est constituée d’une condition de correspondance, d’une priorité et d’une action. Les types d’actions pris en charge sont : *AUTORISER*, *BLOQUER*, *JOURNALISER* et *REDIRIGER*. Vous pouvez créer une stratégie entièrement personnalisée qui répond aux exigences de protection spécifiques de votre application en combinant des règles personnalisées et des règles managées.

Les règles d’une stratégie sont traitées selon un ordre de priorité. La priorité est représentée par un nombre unique qui définit l’ordre des règles à traiter. Plus le nombre est petit, plus la priorité est élevée. Les règles ayant des valeurs inférieures sont évaluées avant les règles ayant des valeurs plus élevées. Une fois qu'une correspondance de règle est trouvée, l’action associée définie dans la règle est appliquée à la requête. Une fois qu’une telle correspondance est traitée, aucune autre règle avec une priorité inférieure n’est traitée.

Une application web hébergée sur Azure CDN peut être associée à une seule stratégie WAF. Toutefois, vous pouvez avoir un point de terminaison CDN sans aucune stratégie WAF associée. Si une stratégie WAF est présente, elle est répliquée sur tous nos emplacements de périphérie pour garantir des stratégies de sécurité cohérentes à travers le monde.

## <a name="waf-modes"></a>Modes WAF

Une stratégie WAF peut être configurée pour s’exécuter dans les deux modes suivants :

- *Mode de détection* : Quand il est exécuté en mode Détection, le pare-feu d’applications web n’entreprend pas d’autre action que la supervision et la journalisation de la requête et de la règle WAF correspondante dans les journaux WAF. Vous pouvez activer la journalisation des diagnostics pour CDN. Quand vous utilisez le portail, accédez à la section **Diagnostics**.

- *Mode de prévention* : En mode Prévention, le pare-feu d’applications web entreprend l’action spécifiée si une demande correspond à une règle. Si une correspondance est trouvée, aucune autre règle avec une priorité inférieure n’est évaluée. Toutes les requêtes mises en correspondance sont également enregistrées dans les journaux WAF.

## <a name="waf-actions"></a>Actions WAF

Vous pouvez choisir d’exécuter l’une des actions suivantes quand une demande correspond aux conditions d’une règle :

- *Autoriser* : La demande est transférée au back-end par le biais du WAF. Aucune autre règle de priorité inférieure ne peut bloquer cette requête.
- *Bloquer* : la requête est bloquée et le pare-feu d'applications web envoie une réponse au client sans transférer la requête au back-end.
- *Log* :  La demande est journalisée dans les journaux WAF, puis le WAF continue l’évaluation des règles de priorité inférieure.
- *Rediriger :* le pare-feu d'applications web redirige la requête vers l’URI spécifié. L’URI spécifié est un paramètre défini au niveau de la stratégie. Une fois l’URI configuré, toutes les demandes qui correspondent à l’action *Rediriger* sont envoyées à cet URI.

## <a name="waf-rules"></a>Règles WAF

Une stratégie WAF se compose de deux types de règles de sécurité :

- des *règles personnalisées* : règles que vous créez vous-même 
- des *ensembles de règles managées* : ensemble de règles préconfigurées et managées par Azure

### <a name="custom-rules"></a>Règles personnalisées

Les règles personnalisées peuvent avoir des règles de correspondance et des règles de contrôle de débit.

Vous pouvez configurer les règles de correspondance personnalisées suivantes :

- *Listes d’adresses IP autorisées et bloquées* : Vous pouvez contrôler l’accès à vos applications web en fonction d’une liste d’adresses IP de clients ou de plages d’adresses IP. Les deux types d’adresses IPv4 et IPv6 sont pris en charge. Cette liste peut être configurée pour bloquer ou autoriser les requêtes dont l’adresse IP source correspond à l’une des adresses IP listées.

- *Contrôle d’accès en fonction de l’emplacement géographique* : Vous pouvez contrôler l’accès à vos applications web en fonction du code de pays qui est associé à l’adresse IP d’un client.

- *Contrôle d’accès en fonction des paramètres HTTP* : Vous pouvez baser des règles sur des correspondances de chaîne dans les paramètres de requête HTTP/HTTPS.  Par exemple, les chaînes de requête, les arguments POST, l’URI de la demande, l’en-tête de la demande et le corps de la demande.

- *Contrôle d’accès en fonction de la méthode de demande* : Vous basez les règles sur la méthode de demande HTTP de la demande. Par exemple, GET, PUT ou HEAD.

- *Contrainte de taille* : Vous pouvez baser les règles sur la longueur de parties spécifiques d’une requête, comme la chaîne de requête, l’URI ou le corps de la demande.

Une règle de contrôle de débit limite le trafic anormalement élevé en provenance de n’importe quelle adresse IP cliente.

- *Règles de limitation de débit* : Vous pouvez configurer un seuil pour le nombre de demandes web autorisées à partir d’une adresse IP cliente pendant une durée d’une minute. Cette règle est différente d’une règle personnalisée basée sur une liste d’adresses IP autorisées ou bloquées, qui autorise ou bloque toutes les demandes provenant d’une adresse IP cliente. Des limitations de débit peuvent être combinées avec des conditions de correspondance supplémentaires, comme la mise en correspondance de paramètres HTTP/HTTPS pour un contrôle du débit plus précis.

### <a name="azure-managed-rule-sets"></a>Ensembles de règles managées par Azure

Les ensembles de règles managées par Azure fournissent un moyen simple de déployer une solution de protection contre diverses menaces de sécurité courantes. Dans la mesure où ces ensembles de règles sont managés par Azure, les règles sont mises à jour comme il convient pour vous protéger contre les nouvelles signatures d’attaque. L’ensemble de règles par défaut managé par Azure comprend des règles de protection contre les catégories de menaces suivantes :

- Scripts intersites (XSS)
- Attaques Java
- Inclusion de fichier local (LFI)
- Injection de code PHP
- Exécution de commande à distance
- Inclusion de fichier distant (RFI)
- Fixation de session
- Protection contre les injections de code SQL
- Attaquants de protocole

Le numéro de version de l’ensemble de règles par défaut s’incrémente quand de nouvelles signatures d’attaque sont ajoutées à l’ensemble de règles.
L’ensemble de règles par défaut est activé en mode *Détection* par défaut dans vos stratégies WAF. Vous pouvez activer ou désactiver des règles individuellement dans l’ensemble de règles par défaut en fonction des exigences propres à votre application. Vous pouvez également définir des actions spécifiques (AUTORISER/BLOQUER/REDIRIGER/JOURNALISER) pour chaque règle. L’action par défaut de l’ensemble de règles par défaut managé est *Bloquer*.

Les règles personnalisées sont toujours appliquées avant l’évaluation des règles de l’ensemble de règles par défaut. Si une demande correspond à une règle personnalisée, l’action de la règle correspondante est appliquée. La demande est bloquée ou transmise au back-end. Aucune autre règle personnalisée ou les règles de l’ensemble de règles par défaut ne sont traitées. Vous pouvez aussi supprimer l’ensemble de règles par défaut de vos stratégies WAF.

## <a name="configuration"></a>Configuration

Vous pouvez configurer et déployer tous les types de règles WAF en utilisant le portail Azure, des API REST, des modèles Azure Resource Manager ou Azure PowerShell.

## <a name="monitoring"></a>Surveillance

La supervision de WAF avec CDN est intégrée à Azure Monitor pour pouvoir suivre les alertes et superviser facilement les tendances du trafic.

## <a name="next-steps"></a>Étapes suivantes

- [Tutoriel : Créer une stratégie WAF avec Azure CDN à l’aide du portail Azure](waf-cdn-create-portal.md)
