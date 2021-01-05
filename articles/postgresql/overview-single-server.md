---
title: Azure Database pour PostgreSQL - Serveur unique
description: Fournit une vue d'ensemble d'Azure Database pour PostgreSQL - Serveur flexible.
author: sunilagarwal
ms.author: sunila
ms.custom: mvc
ms.service: postgresql
ms.topic: overview
ms.date: 09/21/2020
ms.openlocfilehash: 098d6e3712a928392547fcd2cba68b68dcca5ef3
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92476142"
---
# <a name="azure-database-for-postgresql-single-server"></a>Azure Database pour PostgreSQL - Serveur unique

Trois modes de déploiement sont disponibles pour [Azure Database pour PostgreSQL](./overview.md) optimisé par PostgreSQL Community Edition :

- Serveur unique
- Serveur flexible (préversion)
- Hyperscale (Citus)

Cet article contient une vue d'ensemble et une présentation des concepts de base du modèle de déploiement Serveur unique. Pour en savoir plus sur le mode de déploiement Serveur flexible, consultez dans l'ordre [Vue d'ensemble du mode Serveur flexible](./flexible-server/overview.md) et Vue d'ensemble d'Hyperscale (Citus).

## <a name="overview"></a>Vue d’ensemble

Serveur unique est un service de base de données complètement managé avec des exigences minimales pour les personnalisations de la base de données. La plateforme de serveur unique est conçue pour prendre en charge la plupart des fonctions de gestion de base de données, comme les correctifs, les sauvegardes, la haute disponibilité et la sécurité, avec un minimum de configuration et de contrôle de la part de l'utilisateur. L'architecture est optimisée pour fournir une disponibilité de 99,99 % sur une seule zone de disponibilité. Elle prend en charge la version de la communauté de PostgreSQL 9.5, 9.6, 10 et 11. Le service est aujourd'hui généralement disponible dans un grand nombre de [régions Azure](https://azure.microsoft.com/global-infrastructure/services/).

Les serveurs uniques sont particulièrement adaptés aux applications natives Cloud conçues pour gérer les correctifs automatisés sans qu'il soit nécessaire d'exercer un contrôle précis sur le calendrier des correctifs et les paramètres de configuration PostgreSQL personnalisés.

## <a name="high-availability"></a>Haute disponibilité

Le modèle de déploiement Serveur unique est optimisé pour offrir une haute disponibilité intégrée et une élasticité à moindre coût. L'architecture distingue le calcul du stockage. Le moteur de base de données s'exécute sur un conteneur de calcul propriétaire, tandis que les fichiers de données résident sur le stockage Azure. Le stockage gère trois copies synchrones localement redondantes des fichiers de base de données, ce qui garantit la durabilité des données.

Lors des événements de basculement, qu'ils soient planifiés ou non, si le serveur tombe en panne, le service maintient la haute disponibilité des serveurs grâce à la procédure automatisée suivante :

1. Un nouveau conteneur de calcul est approvisionné.
2. Le stockage contenant les fichiers de données est mappé avec le nouveau conteneur.
3. Le moteur de base de données PostgreSQL est mis en ligne sur le nouveau conteneur de calcul.
4. Le service de passerelle assure un basculement transparent qui ne nécessite aucune modification du côté des applications.
   
 :::image type="content" source="./media/overview/overview-azure-postgres-single-server.png" alt-text="Azure Database pour PostgreSQL - Serveur unique":::

Le temps de basculement standard est compris entre 60 et 120 secondes. La conception native Cloud du service à serveur unique lui permet de garantir 99,99 % de disponibilité, ce qui élimine les coûts engendrés par un serveur de secours passif.

Avec un temps de disponibilité de 99,99 %, l'excellent contrat de niveau de service (SLA) d'Azure s'appuie sur un réseau mondial de centres de données gérés par Microsoft et permet d'exécuter vos applications 24 heures sur 24, 7 jours sur 7.

## <a name="automated-patching"></a>Mise à jour corrective automatisée

Le service effectue une mise à jour corrective automatisée du matériel, du système d’exploitation et du moteur de base de données sous-jacents. Le correctif comprend les mises à jour de sécurité et de logiciel. Pour le moteur PostgreSQL, la mise à niveau des versions mineures est automatique et incluse dans le cadre du cycle de mise à jour corrective. Aucune action de l’utilisateur ni aucun paramètre de configuration ne sont requis pour la mise à jour corrective. La fréquence des mises à jour correctives est gérée par le service en fonction du caractère critique de la charge utile. En général, le service suit un calendrier de publication mensuel dans le cadre de l’intégration et de la publication continues. Les utilisateurs peuvent s’abonner à la [notification de maintenance planifiée]() pour recevoir une notification de la maintenance à venir 72 heures avant l’événement.

## <a name="automatic-backups"></a>Sauvegardes automatiques

Le service à serveur unique crée automatiquement des sauvegardes de serveur et les conserve sur un stockage géoredondant ou redondant localement configuré par l'utilisateur. Les sauvegardes peuvent être utilisées pour restaurer votre serveur à n'importe quel point dans le temps au cours de la période de rétention des sauvegardes. La période de rétention de sauvegarde par défaut est de sept jours. La rétention peut être configurée sur une durée maximum de 35 jours. Toutes les sauvegardes sont chiffrées à l’aide du chiffrement AES de 256 bits. Pour plus d'informations, consultez [Sauvegardes](./concepts-backup.md).

## <a name="adjust-performance-and-scale-within-seconds"></a>Ajustez les performances et la mise à l’échelle en quelques secondes

Trois références SKU sont disponibles pour le service à serveur unique : De base, Usage général et À mémoire optimisée. Le niveau De base conviendra à un développement à faible coût et à des charges de travail à faible simultanéité. Les niveaux Usage général et À mémoire optimisée conviendront quant à eux aux charges de travail de production nécessitant une simultanéité et une mise à l'échelle de haut niveau, ainsi que des performances prévisibles. Vous pouvez créer votre première application sur une petite base de données pour un faible coût mensuel, puis adapter l’échelle aux besoins de votre solution. La mise à l'échelle du stockage s'effectue en ligne et prend en charge la croissance automatique du stockage. L’évolutivité dynamique permet de répondre en toute transparence à l’évolution rapide des besoins en ressources de votre base de données. Vous ne payez que pour les ressources que vous consommez. Pour plus d’informations, consultez [Niveaux tarifaires]().

## <a name="enterprise-grade-security-compliance-and-governance"></a>Sécurité, conformité et gouvernance de niveau entreprise

Le service à serveur unique utilise le module de chiffrement conforme à la norme FIPS 140-2 pour chiffrer le stockage des données au repos. Toutes les données sont chiffrées, y compris les sauvegardes et les fichiers temporaires créés lors de l'exécution des requêtes. Le service utilise le chiffrement AES 256 bits inclus dans le chiffrement du stockage Azure, et les clés peuvent être gérées par le système (par défaut) ou [par le client](). Le service chiffre les données en mouvement avec les protocoles SSL/TLS appliqués par défaut. Le service prend en charge les versions 1.2, 1.1 et 1.0 du protocole TLS, avec la possibilité d'appliquer la [version minimum de TLS]().

Le service permet un accès privé aux serveurs via une liaison privée et fournit la fonctionnalité Advanced Threat Protection. Advanced Threat Protection détecte les activités anormales indiquant des tentatives inhabituelles et potentiellement dangereuses d'accès à vos bases de données ou d'exploitation de celles-ci.

Outre l'authentification native, le service à serveur unique prend en charge l'authentification Azure Active Directory. L'authentification Azure AD est un mécanisme de connexion aux serveurs PostgreSQL utilisant des identités définies et gérées dans Azure AD. Avec l'authentification Azure AD, vous pouvez gérer les identités des utilisateurs de la base de données et d'autres services Azure à un emplacement central, ce qui simplifie et centralise le contrôle d'accès.

La fonctionnalité d'[enregistrement d'audit]() (en préversion) est disponible pour suivre toute l'activité au niveau de la base de données.

Le service à serveur unique est conforme aux principales certifications du secteur, telles que FedRAMP, HIPAA, PCI DSS. Visitez le [Centre de confidentialité Azure]() pour plus d’informations sur la sécurité de la plateforme Azure.

Pour plus d'informations sur les fonctionnalités de sécurité d'Azure Database pour PostgreSQL, consultez la [vue d'ensemble de la sécurité]().

## <a name="monitoring-and-alerting"></a>Surveillance et alerte

Le service à serveur unique est équipé de fonctionnalités intégrées d'analyse des performances et d'alerte. Toutes les métriques Azure présentent une fréquence d’une minute et chaque métrique fournit 30 jours d’historique. Vous pouvez configurer des alertes basées sur les métriques. Le service permet de configurer des journaux de requêtes lentes et est fourni avec une fonctionnalité [Magasin de requêtes](./concepts-query-store.md) distincte. Le Magasin des requêtes simplifie la résolution des problèmes de performances en vous aidant à identifier rapidement les requête dont l’exécution est la plus longue et qui consomment le plus de ressources. Grâce à ces outils, vous pouvez rapidement optimiser vos charges de travail et configurer votre serveur pour bénéficier de performances optimales. Pour plus d'informations, consultez [Analyse](./concepts-monitoring.md).

## <a name="migration"></a>Migration

Le service exécute la version de la communauté de PostgreSQL. Cela permet une compatibilité totale des applications et requiert un coût de refactorisation minimal pour migrer une application existante développée sur le moteur PostgreSQL vers un service à serveur unique. La migration vers le serveur unique peut être effectuée à l'aide de l'une des options suivantes :

- **Image mémoire et restauration**  : pour les migrations hors connexion, au cours desquelles les utilisateurs peuvent se permettre des temps d'arrêt, l'image mémoire et la restauration à l'aide d'outils communautaires tels que Pg_dump et Pg_restore peuvent constituer la méthode de migration la plus rapide. Pour plus d'informations, consultez [Migrer à l'aide de l'image mémoire et de la restauration](./howto-migrate-using-dump-and-restore.md).
- **Azure Database Migration Service**  : pour les migrations transparentes et simplifiées vers un serveur unique avec temps d'arrêt minimal, Azure Database Migration Service peut être utilisé. Consultez [DMS via le portail](../dms/tutorial-postgresql-azure-postgresql-online-portal.md) et [DMS via l'interface CLI](../dms/tutorial-postgresql-azure-postgresql-online.md).

## <a name="contacts"></a>Contacts

Pour toute question ou suggestion au sujet d’Azure Database pour PostgreSQL, envoyez un e-mail à l’équipe Azure Database pour PostgreSQL ([@Ask Azure DB pour PostgreSQL](mailto:AskAzureDBforPostgreSQL@service.microsoft.com)). Cette adresse e-mail n’est pas un alias de support technique.

En outre, tenez compte des points de contact suivants le cas échéant :

- Pour contacter le support technique Azure, [émettez un ticket à partir du Portail Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).
- Pour résoudre un problème relatif à votre compte, enregistrez une [demande de support](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) sur le portail Azure.
- Pour donner votre avis ou demander de nouvelles fonctionnalités, créez une entrée via [UserVoice](https://feedback.azure.com/forums/597976-azure-database-for-postgresql).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez lu l'introduction au mode de déploiement Azure Database pour PostgreSQL - Serveur unique, vous êtes prêt à :
- Créer votre premier serveur
