---
title: Résoudre les problèmes de connexion - Azure Database pour PostgreSQL - Serveur unique
description: Découvrez comment résoudre les problèmes de connexion à la base de données Azure Database pour PostgreSQL - Serveur unique.
keywords: connexion postgresql,chaîne de connexion,problèmes de connectivité,erreur temporaire,erreur de connexion
author: niklarin
ms.author: nlarin
ms.service: postgresql
ms.topic: how-to
ms.date: 5/6/2019
ms.openlocfilehash: bff930153dc8941fbfe561edf963d5b1c1e7811f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96014616"
---
# <a name="troubleshoot-connection-issues-to-azure-database-for-postgresql---single-server"></a>Résoudre les problèmes de connexion à la base de données Azure Database pour PostgreSQL - Serveur unique

Les problèmes de connexion peuvent avoir l’une des causes suivantes :

* Paramètres du pare-feu
* Expiration du délai de connexion
* Informations de connexion incorrectes
* Dépassement de la limite maximale sur certaines ressources de la base de données Azure Database pour PostgreSQL
* Problèmes liés à l’infrastructure du service
* Maintenance en cours dans le service
* L’allocation de calcul du serveur est modifiée par la mise à l’échelle du nombre de vCores ou par la migration vers un niveau de service différent.

En général, les problèmes de connexion à la base de données Azure Database pour PostgreSQL peuvent être classés ainsi :

* Erreurs temporaires (de courte durée ou intermittentes)
* Erreurs persistantes ou non temporaires (erreurs qui se produisent régulièrement)

## <a name="troubleshoot-transient-errors"></a>Résoudre les erreurs temporaires

Des erreurs transitoires se produisent lorsque la maintenance est effectuée, lorsque le système rencontre une erreur avec le matériel ou le logiciel ou lorsque vous modifiez les vCores ou le niveau de service de votre serveur. Le service Azure Database pour PostgreSQL intègre la haute disponibilité et est conçu pour atténuer ces types de problèmes automatiquement. Cependant, votre application perd sa connexion au serveur pendant une courte période de moins de 60 secondes au maximum. Certains événements peuvent parfois prendre plus de temps à s’atténuer, par exemple lorsqu’une transaction volumineuse entraîne une récupération de longue durée.

### <a name="steps-to-resolve-transient-connectivity-issues"></a>Étapes pour résoudre les problèmes de connectivité transitoire

1. Consultez le [tableau de bord du service Microsoft Azure](https://azure.microsoft.com/status) pour obtenir la liste des coupures prévues qui se sont produites au moment où les erreurs ont été signalées par l’application.
2. Les applications qui se connectent à un service cloud, tel que la base de données Azure Database pour PostgreSQL, doivent s’attendre à des erreurs périodiques de reconfiguration et implémenter une logique de nouvelle tentative pour gérer ces erreurs au lieu d’afficher ces événements en tant qu’erreurs de l’application aux utilisateurs. Reportez-vous à [Gestion des erreurs de connectivité temporaires pour Azure Database pour PostgreSQL](concepts-connectivity.md) afin de connaître les meilleures pratiques et les instructions de conception pour la gestion des erreurs temporaires.
3. Lorsqu’un serveur approche des limites de ressources, les erreurs peuvent s’apparenter à un problème de connectivité temporaire. Reportez-vous à [Limitations des bases de données Azure pour PostgreSQL](concepts-limits.md).
4. Si les problèmes de connectivité persistent ou si la durée pendant laquelle votre application rencontre une erreur dépasse les 60 secondes ou si plusieurs occurrences de l’erreur s’affichent dans un jour donné, créez une demande de support Azure en sélectionnant **Obtenir de l’aide** sur le site du [support Azure](https://azure.microsoft.com/support/options) .

## <a name="troubleshoot-persistent-errors"></a>Résoudre les erreurs persistantes

Si l’application échoue de façon permanente à se connecter à la base de données Azure Database pour PostgreSQL, cela indique généralement un problème avec l’un des éléments suivants :

* Configuration du pare-feu côté serveur : le pare-feu du serveur Azure Database pour PostgreSQL doit être configuré de façon à autoriser les connexions provenant du client, y compris les serveurs proxy et les passerelles.
* Configuration du pare-feu côté client : le pare-feu du client doit autoriser les connexions au serveur de base de données. Les adresses IP et les ports du serveur auquel vous ne pouvez pas vous connecter doivent être autorisés, ainsi que les noms d’application, comme PostgreSQL, sur certains pare-feu.
* Erreur utilisateur : il se peut que vous ayez fait une erreur lors de la saisie des paramètres de connexion, comme le nom du serveur dans la chaîne de connexion ou le suffixe *\@servername* manquant dans le nom de l’utilisateur.
* Si l’erreur _Le serveur n’est pas configuré pour autoriser les connexions IPv6_, s’affiche, notez que le niveau de base ne prend pas en charge les points de terminaison de service VNet. Vous devez supprimer le point de terminaison Microsoft.Sql du sous-réseau qui tente de se connecter au serveur de base.
* Si vous voyez le message d’erreur de connexion _valeur sslmode « * * * » non valide lorsque la prise en charge du protocole SSL n’est pas compilée_, cela signifie que votre client PostgreSQL ne prend pas en charge le protocole SSL. Le plus souvent, le libpq côté client n’a pas été compilé avec l’indicateur « --with-openssl ». Essayez de vous connecter avec un client PostgreSQL qui prend en charge le protocole SSL. 

### <a name="steps-to-resolve-persistent-connectivity-issues"></a>Étapes permettant résoudre les problèmes de connectivité persistants

1. Configurez les [règles de pare-feu](howto-manage-firewall-using-portal.md) pour autoriser l’adresse IP du client. Définissez une règle de pare-feu avec 0.0.0.0 pour adresse IP initiale et 255.255.255.255 pour adresse IP finale à des fins de test temporaire uniquement. Cette opération ouvrira le serveur à toutes les adresses IP. Si elle résout votre problème de connectivité, supprimez cette règle et créez une règle de pare-feu pour une adresse ou une plage d’adresses IP correctement bornée.
2. Sur tous les pare-feu situés entre le client et Internet, assurez-vous que le port 5432 est ouvert pour les connexions sortantes.
3. Vérifiez votre chaîne de connexion et d’autres paramètres de connexion.
4. Vérifiez l’état du service dans le tableau de bord. Si vous soupçonnez une panne régionale, consultez [Vue d’ensemble de la continuité d’activité avec Azure Database pour PostgreSQL](concepts-business-continuity.md) pour connaître les étapes de restauration vers une nouvelle région.

## <a name="next-steps"></a>Étapes suivantes

* [Gestion des erreurs de connectivité temporaires pour Azure Database pour PostgreSQL](concepts-connectivity.md)
