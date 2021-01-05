---
title: Rapports personnalisés de Verizon | Microsoft Docs
description: 'Vous pouvez afficher les modèles d’utilisation pour votre CDN en utilisant les rapports suivants : la bande passante, les données transférées, les correspondances, les statuts de cache, le taux d’accès au cache, les données transférées IPV4/IPV6.'
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 10/11/2017
ms.author: allensu
ms.openlocfilehash: cff355aeb74efc9dec2780840ae57fea4c5764cc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84888704"
---
# <a name="custom-reports-from-verizon"></a>Rapports personnalisés de Verizon

[!INCLUDE [cdn-verizon-only](../../includes/cdn-verizon-only.md)]

Les rapports personnalisés de Verizon accessibles via le portail de gestion pour les profils Verizon vous permettent de définir le type des données à collecter pour les rapports d’enregistrements Edge CNAMEs.


## <a name="accessing-verizon-custom-reports"></a>Accès aux rapports personnalisés de Verizon
1. Dans le panneau Profil CDN, cliquez sur le bouton **Gérer** .
   
    ![Bouton Gérer du profil CDN](./media/cdn-reports/cdn-manage-btn.png)
   
    Le portail de gestion CDN s'ouvre.
2. Pointez sur l’onglet **Analytics**, puis sur le menu volant **Custom Reports** (Rapports personnalisés). Cliquez sur **Edge CNAMEs**.
   
    ![Portail de gestion CDN - Menu des rapports personnalisés](./media/cdn-reports/cdn-custom-reports.png)

## <a name="edge-cnames-custom-report"></a>Rapport personnalisé Edge CNAMEs
Le rapport personnalisé Edge CNAMEs fournit des statistiques d’accès et de transfert de données pour les Edge CNAMEs sur lesquels la journalisation de rapport personnalisé a été activée. Les Edge CNAMEs consistent en noms d’hôte de point de terminaison CDN Azure et d’éventuels noms d’hôte de domaine personnalisé associés. 

Le journalisation de données de rapport personnalisé commence une heure après que vous avez activé la fonctionnalité de création de rapports personnalisés d’un Edge CNAME. Vous pouvez afficher les données du rapport en générant un rapport Edge CNAMEs pour une plateforme spécifique ou pour toutes les plateformes. La couverture de ce rapport est limitée aux Edge CNAMEs pour lesquels les données de rapport personnalisé ont été collectées pendant la période spécifiée. Le rapport Edge CNAMEs consiste en un graphique et une table de données pour 10 premiers Edge CNAMEs en fonction de la métrique définie dans l’option Métriques. 

Générez un rapport personnalisé en définissant les options de rapport suivantes :

- Métriques : les options prises en charge sont les suivantes :

   - Accès : indique le nombre total de demandes adressées à un Edge CNAME sur lequel la fonctionnalité de création de rapports personnalisés est activée. Cette mesure n’inclut pas le code d’état retourné au client.

   - Données transférées : indique la quantité totale de données transférées des serveurs Edge aux clients HTTP (par exemple, des navigateurs web) pour des demandes dirigées vers un Edge CNAME sur lequel la fonctionnalité de création de rapports personnalisés est activée. La quantité de données transférées est calculée en ajoutant des en-têtes de réponse HTTP au corps de la réponse. Par conséquent, la quantité des données transférées pour chaque composant est supérieure à sa taille de fichier réelle.

- Regroupements : détermine le type des statistiques affichées sous le graphique à barres. Les options suivantes sont prises en charge :

   - Codes de réponse HTTP : organise les statistiques par code de réponse HTTP (par exemple, 200, 403, etc.) retourné au client. 

   - État du cache : organise les statistiques par état du cache.


Pour définir la plage de dates pour le rapport, vous pouvez sélectionner dans la liste déroulante une plage de dates prédéfinie, telle que **Aujourd’hui** ou **Semaine**, ou sélectionner **Personnalisée**, puis entrer manuellement une plage de dates en cliquant sur les icônes du calendrier. 

Après avoir sélectionné la plage de dates, cliquez sur **OK** pour générer le rapport.

Vous pouvez exporter les données au format Excel en cliquant sur le symbole Excel à droite du bouton **OK**.

![Rapport CNAMEs](./media/cdn-reports/cdn-cnames-report.png)

## <a name="edge-cnames-custom-report-fields"></a>Champs de rapport personnalisé d’Edge CNAMEs

| Champ                     | Description   |
|---------------------------|---------------|
| 2xx                       | Indique le nombre total des demandes ou des données transférées (Mo) pour l’Edge CNAME qui produit un code d’état HTTP 2xx (par exemple, 200 OK). |
| 3xx                       | Indique le nombre total des demandes ou des données transférées (Mo) pour l’Edge CNAME qui produit un code d’état HTTP 3xx (par exemple, 302 Trouvé ou 304 Non modifié). |
| 4xx                       | Indique le nombre total des demandes ou des données transférées (Mo) pour l’Edge CNAME qui produit un code d’état HTTP 4xx (par exemple, 400 Requête incorrecte, 403 Interdit ou 404 Non trouvé). |
| 5xx                       | Indique le nombre total des demandes ou des données transférées (Mo) pour l’Edge CNAME qui produit un code d’état HTTP 5xx (par exemple, 500 Erreur interne du serveur ou 502 Passerelle incorrecte). |
| % de correspondance dans le cache               | Indique le pourcentage de demandes pouvant être mises en cache traitées directement à partir du cache du demandeur. |
| Présences dans le cache                | Indique le nombre total des demandes ou des données transférées (Mo) pour l’Edge CNAME qui produit un correspondance dans le cache (par exemple, TCP_EXPIRED_HIT, TCP_HIT ou TCP_PARTIAL_HIT). Un correspondance dans le cache se produit quand une version mise en cache du contenu demandé est trouvée. |
| Données transférées (Mo)     | Indique la quantité totale de données transférées (Mo) à partir des serveurs Edge pour les clients HTTP (navigateurs web) pour l’Edge CNAME. La quantité de données transférées est calculée en ajoutant les en-têtes de réponse HTTP au corps de la réponse. Par conséquent, la quantité des données transférées pour chaque composant est supérieure à sa taille de fichier réelle. |
| Description               | Identifie un Edge CNAME par son nom d’hôte. |
| Correspondances                      | Indique le nombre total de demandes adressées à l’Edge CNAME. |
| Absences                    | Indique le nombre total des demandes ou des données transférées (Mo) pour l’Edge CNAME qui produit un absence du cache (par exemple, TCP_CLIENT_REFRESH_MISS, TCP_EXPIRED_MISS, or TCP_MISS). Une absence dans le cache se produit lorsque le contenu demandé n’a pas mis en cache sur le serveur Edge qui a satisfait la demande. | 
| Aucun cache                  | Indique le nombre total des demandes ou des données transférées (Mo) pour l’Edge CNAME qui produit un code d’état CONFIG_NOCACHE.  |
| Autres                     | Indique le nombre total des demandes ou des données transférées (Mo) pour l’Edge CNAME indiqué, qui produit un code d’état HTTP ne s’inscrivant pas dans la plage 2xx - 5xx. |
| Plateforme                  | Indique la plateforme qui gère le trafic de l’Edge CNAME. |
| Non affecté               | Indique le nombre total des demandes ou des données transférées (Mo) pour l’Edge CNAME pour lequel les informations de code d’état du cache ou de code d’état HTTP n’ont pas été consignées.  |
| Non mis en cache               | Indique le nombre total des demandes ou des données transférées (Mo) pour l’Edge CNAME, qui entraîne un code d’état de cache UNCACHEABLE.  |


## <a name="considerations"></a>Considérations
Des rapports peuvent uniquement être générés pour les 18 derniers mois.

