---
title: Résoudre les problèmes de connexion – Hyperscale (Citus) – Azure Database pour PostgreSQL
description: Découvrez comme résoudre les problèmes de connexion à Azure Database pour PostgreSQL - Hyperscale (Citus)
keywords: connexion postgresql,chaîne de connexion,problèmes de connectivité,erreur temporaire,erreur de connexion
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 10/8/2019
ms.openlocfilehash: e1c6825820ae943d10157279dfe93922a7521b75
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91295615"
---
# <a name="troubleshoot-connection-issues-to-azure-database-for-postgresql---hyperscale-citus"></a>Résoudre les problèmes de connexion à Azure Database pour PostgreSQL - Hyperscale (Citus)

Les problèmes de connexion peuvent avoir plusieurs causes, telles que :

* Paramètres du pare-feu
* Expiration du délai de connexion
* Informations de connexion incorrectes
* Limite de connexion atteinte pour le groupe de serveurs
* Problèmes liés à l’infrastructure du service
* Maintenance des services
* Le nœud coordinateur a basculé sur le nouveau matériel

En général, les problèmes de connexion à Hyperscale (Citus) peuvent être classifiés ainsi :

* Erreurs temporaires (de courte durée ou intermittentes)
* Erreurs persistantes ou non temporaires (erreurs qui se produisent régulièrement)

## <a name="troubleshoot-transient-errors"></a>Résoudre les erreurs temporaires

Les erreurs temporaires se produisent pour plusieurs raisons. Parmi les plus courantes : la maintenance du système, les erreurs liées au matériel ou aux logiciels et les mises à niveau vCore des nœuds coordinateurs.

L’activation de la haute disponibilité pour les nœuds de groupe de serveurs Hyperscale (Citus) peut atténuer automatiquement ces types de problèmes. Toutefois, votre application doit toujours être préparée à perdre brièvement sa connexion. D’autres événements peuvent également prendre plus de temps à s’atténuer, par exemple lorsqu’une transaction volumineuse entraîne une récupération de longue durée.

### <a name="steps-to-resolve-transient-connectivity-issues"></a>Étapes pour résoudre les problèmes de connectivité transitoire

1. Consultez le [tableau de bord du service Microsoft Azure](https://azure.microsoft.com/status) pour obtenir la liste des interruptions connues qui se sont produites au moment où l’application a signalé des erreurs.
2. Les applications qui se connectent à un service cloud tel que Hyperscale (Citus) doivent attendre des erreurs temporaires et réagir correctement. Par exemple, les applications doivent implémenter une logique de nouvelle tentative pour gérer ces erreurs au lieu de les exposer en tant qu’erreurs d’application aux utilisateurs.
3. Lorsqu’un groupe de serveurs approche des limites de ressources, les erreurs peuvent s’apparenter à des problèmes de connectivité temporaires. L’augmentation de la RAM du nœud, ou l’ajout de nœuds Worker et le rééquilibrage des données peuvent vous aider.
4. Si les problèmes de connectivité persistent ou durent plus de 60 secondes, ou encore se produisent plusieurs fois par jour, effectuez une requête de support Azure en sélectionnant **Obtenir de l’aide** sur le site [Support Azure](https://azure.microsoft.com/support/options).

## <a name="troubleshoot-persistent-errors"></a>Résoudre les erreurs persistantes

Si, à plusieurs reprises, l’application ne parvient pas à se connecter à Hyperscale (Citus), les causes les plus courantes sont un problème de configuration du pare-feu ou une erreur de l’utilisateur.

* Configuration du pare-feu du nœud coordinateur : Vérifiez que le pare-feu du serveur Hyperscale (Citus) est configuré pour autoriser les connexions côté client, y compris les serveurs proxy et les passerelles.
* Configuration du pare-feu côté client : le pare-feu du client doit autoriser les connexions au serveur de base de données. Certains pare-feu nécessitent non seulement l’autorisation d’une application par son nom, mais aussi des adresses IP et des ports du serveur.
* Erreur utilisateur : Vérifiez la chaîne de connexion. Vous avez peut-être mal saisi des paramètres comme le nom du serveur. Vous pouvez rechercher des chaînes de connexion pour différentes infrastructures de langage et psql dans le Portail Azure. Accédez à la page **Chaînes de connexion** dans votre groupe de serveurs Hyperscale (Citus). Gardez également à l’esprit que les clusters Citus ne possèdent qu’une seule base de données et que son nom prédéfini est **citus**.

### <a name="steps-to-resolve-persistent-connectivity-issues"></a>Étapes permettant résoudre les problèmes de connectivité persistants

1. Configurez les [règles de pare-feu](howto-hyperscale-manage-firewall-using-portal.md) pour autoriser l’adresse IP du client. Définissez une règle de pare-feu avec 0.0.0.0 pour adresse IP initiale et 255.255.255.255 pour adresse IP finale à des fins de test temporaire uniquement. Cette règle ouvre le serveur à toutes les adresses IP. Si la règle résout votre problème de connectivité, supprimez-la et créez une règle de pare-feu pour une adresse ou une plage d’adresses IP correctement bornée.
2. Sur tous les pare-feu situés entre le client et Internet, assurez-vous que le port 5432 est ouvert pour les connexions sortantes.
3. Vérifiez votre chaîne de connexion et d’autres paramètres de connexion.
4. Vérifiez l’état du service dans le tableau de bord.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez les concepts de [Règles de pare-feu dans Azure Database pour PostgreSQL - Hyperscale (Citus)](concepts-hyperscale-firewall-rules.md)
* Voir comment [Gérer des règles de pare-feu dans Azure Database pour PostgreSQL - Hyperscale (Citus)](howto-hyperscale-manage-firewall-using-portal.md)
