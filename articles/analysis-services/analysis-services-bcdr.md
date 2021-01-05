---
title: Haute disponibilité Azure Analysis Services | Microsoft Docs
description: Cet article explique comment Azure Analysis Services assure la haute disponibilité au cours d’une interruption de service.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 03/30/2020
ms.author: owend
ms.reviewer: minewiskan
ms.custom: references_regions
ms.openlocfilehash: abfcc38601887cd14509ac436e0344b681e3577e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86506750"
---
# <a name="analysis-services-high-availability"></a>Haute disponibilité Analysis Services

Cet article décrit la garantie d’une haute disponibilité pour les serveurs Azure Analysis Services. 

## <a name="assuring-high-availability-during-a-service-disruption"></a>Garantissez une haute disponibilité au cours d’une interruption de service

Bien que le fait soit rare, un centre de données Azure peut subir une panne. En cas de panne, il entraîne une interruption de service qui peut durer de quelques minutes à plusieurs heures. La haute disponibilité est souvent obtenue grâce à la redondance des serveurs. Avec Azure Analysis Services, vous pouvez obtenir une redondance en créant des serveurs secondaires supplémentaires dans une ou plusieurs régions. Lorsque vous créez des serveurs redondants pour garantir la synchronisation des données et des métadonnées sur ces serveurs avec le serveur d’une région hors connexion, vous pouvez :

* Déployer des modèles sur des serveurs redondants dans d’autres régions. Cette méthode requiert le traitement des données sur votre serveur principal et sur les serveurs redondants en parallèle, garantissant ainsi la synchronisation de tous les serveurs.

* [Sauvegardez](analysis-services-backup.md) les bases de données à partir de votre serveur principal et restaurez-les sur des serveurs redondants. Par exemple, vous pouvez automatiser les sauvegardes nocturnes vers le stockage Azure et procéder à la restauration vers d’autres serveurs redondants dans d’autres régions. 

Dans les deux cas, si votre serveur principal subit une panne, vous devez modifier les chaînes de connexion des clients de rapport afin de vous connecter au serveur dans un autre centre de données régional. Cette modification doit être envisagée en dernier recours et uniquement en cas de panne très grave du centre de données régional. Il est plus probable qu’une panne du centre de données hébergeant votre serveur principal réapparaisse en ligne avant que vous ne puissiez mettre à jour les connexions sur tous les clients. 

Pour éviter de devoir modifier les chaînes de connexion sur les clients de création de rapports, vous pouvez créer un [alias](analysis-services-server-alias.md) de serveur pour votre serveur principal. Si le serveur principal tombe en panne, vous pouvez modifier l’alias pour qu’il pointe vers un serveur redondant dans une autre région. Vous pouvez automatiser un alias à un nom de serveur en codant un contrôle d’intégrité de point de terminaison sur le serveur principal. Si le contrôle d’intégrité échoue, le même point de terminaison peut diriger vers un serveur redondant dans une autre région. 

## <a name="related-information"></a>Informations connexes

[Sauvegarde et restauration](analysis-services-backup.md)   
[Gérer Analysis Services](analysis-services-manage.md)   
[Alias de noms de serveurs](analysis-services-server-alias.md) 

