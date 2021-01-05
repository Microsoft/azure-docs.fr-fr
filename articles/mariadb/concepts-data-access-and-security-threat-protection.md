---
title: Advanced Threat Protection - Azure Database for MariaDB
description: Azure ATP détecte les activités de base de données anormales qui indiquent la présence potentielle de menaces de sécurité pour la base de données.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: 018ba33271f8a3a81c3b7109607334b076d8609c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95977024"
---
# <a name="azure-database-for-mariadb-advanced-threat-protection"></a>Advanced Threat Protection pour Azure Database for MariaDB

La protection Advanced Threat Protection pour Azure Database for MariaDB détecte les activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses de vos bases de données.

> [!IMPORTANT]
> Advanced Threat Protection est disponible en préversion publique.

La protection avancée contre les menaces fait partie de l’offre Advanced Data Security qui est un package unifié de fonctionnalités de sécurité avancées. Vous pouvez accéder à Advanced Threat Protection et le gérer via le [portail Azure](https://portal.azure.com). Cette fonctionnalité est disponible pour les serveurs Usage général et Mémoire optimisée.

> [!NOTE]
> La fonctionnalité Advanced Threat Protection **n’est pas** disponible dans les régions administratives et souveraines suivantes du cloud Azure : US Gov Texas, US Gov Arizona, US Gov Iowa, US, Gov Virginie, US DoD Est, US DoD Central, Allemagne Centre, Allemagne Nord, Chine Est, Chine Est 2. Consultez les [produits disponibles par région](https://azure.microsoft.com/global-infrastructure/services/) pour connaître la disponibilité générale des produits.


## <a name="what-is-advanced-threat-protection"></a>Présentation d’Azure ATP

Azure ATP pour Azure Database for MariaDB fournit une nouvelle couche de sécurité qui permet aux clients de détecter les menaces potentielles et d’y répondre à mesure qu’elles se présentent en générant des alertes de sécurité sur les activités anormales de bases de données. Les utilisateurs reçoivent une alerte en cas d’activités de base de données suspectes, de vulnérabilités potentielles et de modèles d’accès et de requêtes anormaux à la base de données. Azure ATP pour Azure Database for MariaDB intègre des alertes à [Azure Security Center](https://azure.microsoft.com/services/security-center/), avec des détails sur les activités suspectes et les actions recommandées à entreprendre pour analyser et prévenir la menace. Azure ATP pour Azure Database for MariaDB vous permet de réagir facilement aux menaces potentielles visant la base de données sans devenir un expert en sécurité ou gérer des systèmes avancés de surveillance de la sécurité. 

![Concept Azure ATP](media/concepts-data-access-and-security-threat-protection/advanced-threat-protection-concept.png)

## <a name="advanced-threat-protection-alerts"></a>Alertes Azure ATP 
Advanced Threat Protection pour Azure Database for MariaDB détecte les activités anormales indiquant des tentatives inhabituelles ou potentiellement dangereuses visant à accéder aux bases de données ou à les exploiter et peut déclencher les alertes suivantes :
- **Accès à partir d’un emplacement inhabituel** : cette alerte est déclenchée en cas de modification du modèle d’accès à un serveur Azure Database for MariaDB, quand un utilisateur s’est connecté au serveur Azure Database for MariaDB à partir d’un emplacement géographique inhabituel. Dans certains cas, l’alerte détecte une action légitime (une nouvelle application ou opération de maintenance du développeur). Dans d’autres cas, l’alerte détecte une action malveillante (ancien employé, attaquant externe).
- **Accès à partir d’un centre de données Azure inhabituel** : cette alerte est déclenchée en cas de modification du modèle d’accès au serveur Azure Database for MariaDB, quand un utilisateur s’est connecté au serveur à partir d’un centre de données Azure inhabituel observé sur ce serveur récemment. Dans certains cas, l’alerte détecte une action légitime (votre nouvelle application dans Azure, Power BI). Dans d’autres cas, l’alerte détecte une action malveillante provenant d’une ressource/d’un service Azure (ancien employé, attaquant externe).
- **Accès à partir d’une entité de sécurité inconnue** : cette alerte est déclenchée en cas de modification du modèle d’accès au serveur Azure Database for MariaDB, quand un utilisateur s’est connecté au serveur avec un principal inhabituel (utilisateur Azure Database for MariaDB). Dans certains cas, l’alerte détecte une action légitime (nouvelle application, opération de maintenance du développeur). Dans d’autres cas, l’alerte détecte une action malveillante (ancien employé, attaquant externe).
- **Accès à partir d’une application potentiellement dangereuse** : cette alerte est déclenchée quand une application potentiellement dangereuse est utilisée pour accéder à la base de données. Dans certains cas, l’alerte détecte le test d’intrusion en action. Dans d’autres cas, l’alerte détecte une attaque à l’aide d’outils d’attaque courants.
- **Attaque par force brute pour obtenir les informations d’identification d’Azure Database for MariaDB** : cette alerte est déclenchée quand il existe un nombre anormalement élevé d’échecs de connexion avec des informations d’identification différentes. Dans certains cas, l’alerte détecte le test d’intrusion en action. Dans d’autres cas, l’alerte détecte une attaque par force brute.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [Azure Security Center](../security-center/security-center-introduction.md)
* Pour plus d’informations sur les prix, consultez la page [Tarification d’Azure Database for MariaDB](https://azure.microsoft.com/pricing/details/mariadb/). 
* Configurer [Azure ATP pour Azure Database for MariaDB](howto-database-threat-protection-portal.md) avec le portail Azure