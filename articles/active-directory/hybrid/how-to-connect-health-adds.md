---
title: Utilisation d’Azure AD Connect Health avec AD DS | Microsoft Docs
description: Cette page dédiée à Azure AD Connect Health explique comment surveiller AD DS.
services: active-directory
documentationcenter: ''
author: zhiweiwangmsft
manager: daveba
editor: curtand
ms.assetid: 19e3cf15-f150-46a3-a10c-2990702cd700
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/18/2017
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: d75b439c2753db17cffecdc97b6389a7b31b833e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89279871"
---
# <a name="using-azure-ad-connect-health-with-ad-ds"></a>Utilisation d’Azure AD Connect Health avec AD DS
La documentation suivante s’applique à la surveillance des services de domaine Active Directory avec Azure AD Connect Health. Les versions prises en charge d’AD DS sont les suivantes : Windows Server 2008 R2, Windows Server 2012, Windows Server 2012 R2 et Windows Server 2016.

Pour plus d’informations sur la surveillance AD FS avec Azure AD Connect Health, consultez [Utilisation d’Azure AD Connect Health avec AD FS](how-to-connect-health-adfs.md). En outre, pour plus d’informations sur la surveillance de la synchronisation Azure AD Connect avec Azure AD Connect Health, consultez [Utilisation d’Azure AD Connect Health pour la synchronisation](how-to-connect-health-sync.md).

![Azure AD Connect Health pour AD DS](./media/how-to-connect-health-adds/domainservicesnapshot.PNG)

## <a name="alerts-for-azure-ad-connect-health-for-ad-ds"></a>Alertes d’Azure AD Connect Health pour AD DS
La section Alertes disponible dans Azure AD Connect Health pour AD DS vous propose une liste d’alertes actives et résolues, liées à vos contrôleurs de domaine. Si vous sélectionnez une alerte active ou résolue, un nouveau panneau contenant des informations supplémentaires, ainsi que les étapes de résolution et des liens vers la documentation de support, s’ouvre. Chaque type d’alerte peut avoir une ou plusieurs instances, qui correspondent à chacun des contrôleurs de domaine affectés par cette alerte. Au bas du panneau d’alerte, vous pouvez double-cliquer sur un contrôleur de domaine affecté pour ouvrir un autre panneau contenant des informations supplémentaires sur cette instance d’alerte.

Dans ce panneau, vous pouvez activer les notifications par courrier électronique pour les alertes et modifier l’intervalle de temps dans la vue. Étendez l’intervalle de temps pour voir les alertes résolues antérieures.

![Erreur de synchronisation d’Azure AD Connect](./media/how-to-connect-health-adds/aadconnect-health-adds-alerts.png)

## <a name="domain-controllers-dashboard"></a>Tableau de bord Contrôleurs de domaine
Ce tableau de bord fournit une vue de topologie de votre environnement, ainsi que les mesures opérationnelles essentielles et l’état d’intégrité de chacun de vos contrôleurs de domaine surveillés. Les mesures présentées permettent d’identifier rapidement des contrôleurs de domaine qui peuvent nécessiter un examen supplémentaire. Par défaut, seul un sous-ensemble des colonnes est affiché. Cependant, vous pouvez accéder à l’ensemble des colonnes disponibles en double-cliquant sur la commande Colonnes. Sélectionnez les colonnes qui vous intéressent le plus pour faire de ce tableau de bord un emplacement unique et facile pour afficher l’état de votre environnement AD DS.

![Contrôleurs de domaine](./media/how-to-connect-health-adds/aadconnect-health-adds-domainsandsites-dashboard.png)

Les contrôleurs de domaine peuvent être regroupés par leur domaine ou site respectif, ce qui est utile pour comprendre la topologie de l’environnement. Enfin, si vous double-cliquez sur l’en-tête du panneau, le tableau de bord est agrandi pour pouvoir utiliser l’écran réel disponible. Cette vue plus large est utile lorsque plusieurs colonnes sont affichées.

## <a name="replication-status-dashboard"></a>Tableau de bord État de la réplication
Ce tableau de bord fournit une vue de l’état de réplication et de la topologie de réplication de vos contrôleurs de domaine surveillés. L’état de la dernière tentative de réplication est indiqué, ainsi que la documentation utile en cas de détection d’erreur. Vous pouvez double-cliquer sur un contrôleur de domaine qui présente une erreur pour ouvrir un nouveau panneau contenant des informations telles que des détails sur l’erreur, les étapes de résolution recommandées et des liens vers la documentation de dépannage.

![État de la réplication](./media/how-to-connect-health-adds/aadconnect-health-adds-replication.png)

## <a name="monitoring"></a>Surveillance
Cette fonctionnalité fournit les tendances graphiques de différents compteurs de performances, qui sont collectées en permanence à partir de chacun des contrôleurs de domaine surveillés. Les performances d’un contrôleur de domaine peuvent être facilement comparées avec tous les autres contrôleurs de domaine surveillés de votre forêt. En outre, vous pouvez afficher plusieurs compteurs de performances côte à côte, ce qui est utile lors du dépannage de problèmes dans votre environnement.

![Surveillance](./media/how-to-connect-health-adds/aadconnect-health-adds-monitoring.png)

Par défaut, nous avons présélectionné quatre compteurs de performances. Vous pouvez toutefois en inclure d’autres en cliquant sur la commande de filtre et en sélectionnant ou en désélectionnant les compteurs de performances de votre choix. En outre, vous pouvez double-cliquer sur un graphique de compteur de performances pour ouvrir un nouveau panneau incluant les points de données de chacun des contrôleurs de domaine surveillés.

## <a name="related-links"></a>Liens connexes
* [Azure AD Connect Health](./whatis-azure-ad-connect.md)
* [Installation de l’agent Azure AD Connect Health](how-to-connect-health-agent-install.md)
* [Opérations Azure AD Connect Health](how-to-connect-health-operations.md)
* [Utilisation d’Azure AD Connect Health avec AD FS](how-to-connect-health-adfs.md)
* [Utilisation d'Azure AD Connect Health pour la synchronisation (en Anglais)](how-to-connect-health-sync.md)
* [Forum Aux Questions (FAQ) Azure AD Connect Health](reference-connect-health-faq.md)
* [Historique de publication des versions d’Azure AD Connect Health](reference-connect-health-version-history.md)