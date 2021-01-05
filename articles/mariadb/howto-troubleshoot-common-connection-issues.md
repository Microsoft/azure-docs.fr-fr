---
title: Résoudre les problèmes de connexion - Azure Database for MariaDB
description: Découvrez comment résoudre les problèmes de connexion à Azure Database for MariaDB, notamment les erreurs temporaires nécessitant de nouvelles tentatives, les problèmes de pare-feu et les interruptions.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: troubleshooting
ms.date: 3/18/2020
ms.openlocfilehash: b9a670472529dccd51cbfa71e385151bc63cff9b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96020736"
---
# <a name="troubleshoot-connection-issues-to-azure-database-for-mariadb"></a>Résoudre les problèmes de connexion à la base de données Azure Database pour MariaDB

Les problèmes de connexion peuvent avoir l’une des causes suivantes :

* Paramètres du pare-feu
* Expiration du délai de connexion
* Informations de connexion incorrectes
* Dépassement de la limite maximale sur certaines ressources de la base de données Azure Database pour MariaDB
* Problèmes liés à l’infrastructure du service
* Maintenance en cours dans le service
* L’allocation de calcul du serveur est modifiée par la mise à l’échelle du nombre de vCores ou par la migration vers un niveau de service différent.

En général, les problèmes de connexion à la base de données Azure Database pour MariaDB peuvent être classés ainsi :

* Erreurs temporaires (de courte durée ou intermittentes)
* Erreurs persistantes ou non temporaires (erreurs qui se produisent régulièrement)

## <a name="troubleshoot-transient-errors"></a>Résoudre les erreurs temporaires

Des erreurs transitoires se produisent lorsque la maintenance est effectuée, lorsque le système rencontre une erreur avec le matériel ou le logiciel ou lorsque vous modifiez les vCores ou le niveau de service de votre serveur. Le service Azure Database pour MariaDB intègre la haute disponibilité et est conçu pour atténuer ces types de problèmes automatiquement. Cependant, votre application perd sa connexion au serveur pendant une courte période de moins de 60 secondes au maximum. Certains événements peuvent parfois prendre plus de temps à s’atténuer, par exemple lorsqu’une transaction volumineuse entraîne une récupération de longue durée.

### <a name="steps-to-resolve-transient-connectivity-issues"></a>Étapes pour résoudre les problèmes de connectivité transitoire

1. Consultez le [tableau de bord du service Microsoft Azure](https://azure.microsoft.com/status) pour obtenir la liste des coupures prévues qui se sont produites au moment où les erreurs ont été signalées par l’application.
2. Les applications qui se connectent à un service cloud, tel que la base de données Azure Database pour MariaDB, doivent s’attendre à des erreurs périodiques de reconfiguration et implémenter une logique de nouvelle tentative pour gérer ces erreurs au lieu d’afficher ces événements en tant qu’erreurs de l’application aux utilisateurs. Reportez-vous à [Gestion des erreurs de connectivité temporaires pour Azure Database pour MariaDB](concepts-connectivity.md) afin de connaître les meilleures pratiques et les instructions de conception pour la gestion des erreurs temporaires.
3. Lorsqu’un serveur approche des limites de ressources, les erreurs peuvent s’apparenter à un problème de connectivité temporaire. Consultez [Limitations dans Azure Database for MariaDB](concepts-limits.md).
4. Si les problèmes de connectivité persistent ou si la durée pendant laquelle votre application rencontre une erreur dépasse les 60 secondes ou si plusieurs occurrences de l’erreur s’affichent dans un jour donné, créez une demande de support Azure en sélectionnant **Obtenir de l’aide** sur le site du [support Azure](https://azure.microsoft.com/support/options) .

## <a name="troubleshoot-persistent-errors"></a>Résoudre les erreurs persistantes

Si l’application échoue de façon permanente à se connecter à la base de données Azure Database pour MariaDB, cela indique généralement un problème avec l’un des éléments suivants :

* Configuration du pare-feu : le pare-feu Azure Database for MariaDB côté serveur ou côté client bloque les connexions.
* Reconfiguration du réseau côté client : une nouvelle adresse IP ou un serveur proxy a été ajouté.
* Erreur utilisateur : par exemple, il se peut que vous ayez fait une erreur lors de la saisie des paramètres de connexion, comme le nom du serveur dans la chaîne de connexion ou le suffixe *\@servername* manquant dans le nom de l’utilisateur.

### <a name="steps-to-resolve-persistent-connectivity-issues"></a>Étapes permettant résoudre les problèmes de connectivité persistants

1. Configurez les [règles de pare-feu](howto-manage-firewall-portal.md) pour autoriser l’adresse IP du client. Définissez une règle de pare-feu avec 0.0.0.0 pour adresse IP initiale et 255.255.255.255 pour adresse IP finale à des fins de test temporaire uniquement. Cette opération ouvrira le serveur à toutes les adresses IP. Si elle résout votre problème de connectivité, supprimez cette règle et créez une règle de pare-feu pour une adresse ou une plage d’adresses IP correctement bornée.
2. Sur tous les pare-feu situés entre le client et Internet, assurez-vous que le port 3306 est ouvert pour les connexions sortantes.
3. Vérifiez votre chaîne de connexion et d’autres paramètres de connexion. Consultez [Connexion d’applications à la base de données Azure Database pour MariaDB](howto-connection-string.md).
4. Vérifiez l’état du service dans le tableau de bord. Si vous soupçonnez une panne régionale, consultez [Vue d’ensemble de la continuité d’activité avec Azure Database pour MariaDB](concepts-business-continuity.md) pour connaître les étapes de restauration vers une nouvelle région.

## <a name="next-steps"></a>Étapes suivantes

* [Gestion des erreurs de connectivité temporaires pour la base de données Azure Database pour MariaDB](concepts-connectivity.md)
