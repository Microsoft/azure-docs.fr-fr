---
title: Rapports principaux de Verizon | Microsoft Docs
description: Découvrez comment accéder aux rapports principaux de Verizon et les afficher via le portail de gestion pour les profils Verizon.
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: 5a0d9018-8bdb-48ff-84df-23648ebcf763
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: 5353fa70be4705b28dab9350fcdf7819b22faf54
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90985534"
---
# <a name="core-reports-from-verizon"></a>Rapports principaux de Verizon

[!INCLUDE [cdn-verizon-only](../../includes/cdn-verizon-only.md)]

En utilisant les rapports principaux de Verizon via le portail de gestion des profils Verizon, vous pouvez afficher des modèles d’utilisation pour votre CDN avec les rapports suivants :

* Bande passante
* Données transférées
* Correspondances
* États du cache
* Taux d'accès au cache
* Données IPV4/IPV6 transférées

## <a name="accessing-verizon-core-reports"></a>Accès aux rapports principaux de Verizon
1. Dans le panneau Profil CDN, cliquez sur le bouton **Gérer** .
   
    ![Bouton Gérer du profil CDN](./media/cdn-reports/cdn-manage-btn.png)
   
    Le portail de gestion CDN s'ouvre.
2. Pointez sur l’onglet **Analytics**, puis sur le menu volant **Core Reports** (Rapports principaux). Cliquez sur un rapport du menu.
   
    ![Portail de gestion CDN - Menu des rapports principaux](./media/cdn-reports/cdn-core-reports.png)

3. Pour chaque rapport, sélectionnez une plage de dates de la liste **Plage de dates**. Vous pouvez sélectionner une plage de dates prédéfinie, comme **Aujourd’hui** ou **Semaine**, ou sélectionner **Personnalisée** puis entrer manuellement une plage de dates en cliquant sur les icônes du calendrier. 

4. Après avoir sélectionné la plage de dates, cliquez sur **OK** pour générer le rapport. 

4. Si vous souhaitez exporter les données au format Excel, cliquez sur l’icône Excel située au-dessus du bouton **OK**.

## <a name="bandwidth"></a>Bande passante
Le rapport relatif à la bande passante consiste en un graphique et un tableau de données indiquant l’utilisation de la bande passante CDN pour HTTP et HTTPS sur une période donnée, en Mbits/s. Vous pouvez afficher l'utilisation de la bande passante sur tous les POP ou un POP particulier. Ce rapport vous permet d’afficher les pics de trafic et la distribution sur les POP.

Dans la liste **Nœuds de périmètre**, sélectionnez **Tous les nœuds Edge** pour afficher le trafic à partir de tous les nœuds ou choisissez une région spécifique.

Le rapport est mis à jour toutes les cinq minutes.

![Rapport de la bande passante](./media/cdn-reports/cdn-bandwidth.png)

## <a name="data-transferred"></a>Données transférées
Ce rapport consiste en un graphique et un tableau de données indiquant l’utilisation de trafic CDN pour HTTP et HTTPS sur une période donnée, en Go. Vous pouvez afficher le trafic de la bande passante sur tous les POP ou un POP particulier. Ce rapport vous permet d’afficher les pics de trafic et la distribution sur les POP.

Dans la liste **Nœuds de périmètre**, sélectionnez **Tous les nœuds Edge** pour afficher le trafic à partir de tous les nœuds ou choisissez une région spécifique.

Le rapport est mis à jour toutes les cinq minutes.

![Rapport des données transférées](./media/cdn-reports/cdn-data-transferred.png)

## <a name="hits-status-codes"></a>Correspondances (codes d'état)
Ce rapport décrit la distribution des codes d'état de demande pour votre contenu. Chaque demande de contenu génère un code d’état HTTP. Le code d'état décrit comment les POP Edge ont géré la demande. Par exemple, un code d’état 2xx indique que la demande a été correctement servie à un client, tandis qu’un code d’état 4xx indique qu’une erreur s’est produite. Pour plus d’informations sur les codes d’état HTTP, consultez la [Liste des codes d’état HTTP](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes).

![Rapport de correspondances](./media/cdn-reports/cdn-hits.png)

## <a name="cache-statuses"></a>États du cache
Ce rapport décrit la distribution des correspondances et des absences dans le cache pour les requêtes du client. Étant donné que les meilleures performances proviennent de correspondances dans le cache, vous pouvez optimiser les vitesses de remise de données en réduisant les absences dans le cache et les correspondances dans le cache expirées. 

Pour réduire les absences dans le cache, configurez votre serveur d’origine afin de réduire l’utilisation des éléments suivants : 
 * En-têtes de réponse `no-cache`
 * Mise en cache des chaînes de requête, sauf si strictement requise  
 * Codes de réponse non mis en cache

Pour réduire les correspondances de mise en cache expirées, définissez la valeur `max-age` d’une ressource sur une longue période afin de réduire le nombre de requêtes dirigées vers le serveur d’origine.

![Rapport des états du cache](./media/cdn-reports/cdn-cache-statuses.png)

### <a name="main-cache-statuses-include"></a>Les états de cache principaux sont les suivants :
* TCP_HIT : traité à partir du serveur Edge. L’objet était dans le cache et n’avait pas dépassé son max-age.
* TCP_MISS : traité à partir du serveur d’origine. L’objet n’était pas dans le cache et la réponse a été renvoyée à l’origine.
* TCP_EXPIRED _MISS : traités à partir du serveur d’origine après la revalidation avec l’origine. L’objet était dans le cache mais avait dépassé son max-age. Une revalidation avec l’origine a entraîné un remplacement de l'objet de cache par une nouvelle réponse de l'origine.
* TCP_EXPIRED _HIT : traités à partir de Edge après la revalidation avec l'origine. L'objet était en cache mais avait dépassé son max-age. Une revalidation avec le serveur d'origine n’a entraîné aucune modification de l’objet de cache.

### <a name="full-list-of-cache-statuses"></a>Liste complète des états de cache
* TCP_HIT : cet état est signalé lorsqu'une requête est traitée directement du POP au client. Une ressource est immédiatement traitée à partir d’un POP lorsqu’elle est mise en cache sur le serveur POP le plus proche du client et que sa durée de vie (TTL) est valide. La durée de vie est déterminée par les en-têtes de réponse suivants :
  
  * Cache-Control: s-maxage
  * Cache-Control: max-age
  * Expires
* TCP_MISS : cet état indique qu'une version mise en cache de la ressource demandée est introuvable sur le serveur POP le plus proche du client. La ressource est demandée à partir d’un serveur d’origine ou d’un serveur de protection d’origine. Si le serveur d’origine ou le serveur de protection d’origine renvoie une ressource, elle est envoyée au client et mise en cache sur le client et le serveur Edge. Sinon, un code d’état autre que 200 (par exemple, 403 Interdit ou 404 Introuvable) est renvoyé.
* TCP_EXPIRED _HIT : cet état est signalé lorsqu’une requête ciblant une ressource avec une durée de vie (TTL) expirée a été envoyée directement du POP au client. Par exemple, lorsque la propriété max-age de la ressource a expiré. 
  
   Une requête expirée entraîne généralement une demande de revalidation au serveur d'origine. Pour qu’un état TCP_EXPIRED_HIT se produise, le serveur d’origine doit indiquer qu’il n’existe pas de version plus récente de la ressource. Cette situation se produit généralement lors d’une mise à jour des en-têtes Cache-Control et Expires de la ressource.
* TCP_EXPIRED _MISS : cet état est signalé lorsqu'une version plus récente d’une ressource mise en cache expirée est envoyée du POP au client. Cet état se produit lorsque la durée de vie d’une ressource mise en cache a expiré (par exemple, la propriété max-age a expiré) et que le serveur d’origine retourne une version plus récente de la ressource. Cette nouvelle version de la ressource est envoyée au client, plutôt que la version mise en cache. En outre, elle est mise en cache sur le serveur Edge et le client.
* CONFIG_NOCACHE : cet état indique qu'une configuration spécifique au client sur le POP Edge a empêché la mise en cache de la ressource.
* NONE : cet état indique qu'une vérification de la fraîcheur du contenu du cache n'a pas été effectuée.
* TCP_ CLIENT_REFRESH _MISS : cet état est signalé lorsqu’un client HTTP (par exemple, le navigateur) force un POP Edge à récupérer une nouvelle version d’une ressource obsolète à partir du serveur d’origine. Par défaut, les serveurs empêchent un client HTTP de forcer nos serveurs Edge à récupérer une nouvelle version de la ressource à partir du serveur d'origine.
* TCP_ PARTIAL_HIT : cet état est signalé lorsqu'une demande de plage d'octets entraîne un accès à une ressource partiellement en cache. La plage d'octets demandée est immédiatement envoyée du serveur POP au client.
* UNCACHEABLE : cet état est signalé lorsque les en-têtes `Cache-Control` et `Expires` d'une ressource indiquent qu'elle ne doit pas être mise en cache sur un POP ou par le client HTTP. Ces types de requêtes sont traités à partir du serveur d’origine.

## <a name="cache-hit-ratio"></a>Taux d'accès au cache
Ce rapport indique le pourcentage de requêtes mises en cache traitées directement à partir de la mémoire cache.

Le rapport fournit les informations suivantes :

* Le contenu demandé a été mis en cache sur le serveur POP le plus proche du demandeur.
* La requête a été traitée directement à partir du Edge de notre réseau.
* La demande ne nécessitait pas de revalidation avec le serveur d'origine.

Le rapport n'inclut pas :

* Les requêtes refusées en raison des options de filtrage par pays/région.
* Les demandes de ressources dont les en-têtes indiquent qu'elles ne doivent pas être mises en cache. Par exemple, les en-têtes `Cache-Control: private`, `Cache-Control: no-cache` ou `Pragma: no-cache` préviennent la mise en cache d’une ressource.
* Les demandes de plage d'octets pour le contenu partiellement mis en cache.

La formule est : (TCP_ HIT/(TCP_ HIT+TCP_MISS))*100

![Rapport des taux d'accès au cache](./media/cdn-reports/cdn-cache-hit-ratio.png)

## <a name="ipv4ipv6-data-transferred"></a>Données IPV4/IPV6 transférées
Ce rapport affiche la distribution de l'utilisation du trafic entre IPV4 et IPV6.

![Données IPV4/IPV6 transférées](./media/cdn-reports/cdn-ipv4-ipv6.png)

## <a name="considerations"></a>Considérations
Les rapports peuvent uniquement être générés pour les 18 derniers mois.

