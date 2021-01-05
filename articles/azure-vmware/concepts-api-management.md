---
title: Concepts - Gestion des API
description: Découvrez comment la Gestion des API protège les API s’exécutant sur des machines virtuelles Azure VMware Solution.
ms.topic: conceptual
ms.date: 10/27/2020
ms.openlocfilehash: 958cc52c48d1121a69dca2fc901289ad1ed671cb
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541961"
---
# <a name="api-management-to-publish-and-protect-apis-running-on-azure-vmware-solution-based-vms"></a>Gestion des API pour publier et protéger des API qui s’exécutent sur des machines virtuelles Azure VMware Solution

Microsoft Azure [Gestion des API](https://azure.microsoft.com/services/api-management/) vous permet de publier en toute sécurité des contrôles serveur consommateurs internes ou externes.  Seuls les SKU Développeur et Premium permettent l’intégration de Réseau virtuel Azure pour publier des API qui s’exécutent sur des charges de travail Azure VMware Solution.  Les deux SKU activent en toute sécurité la connectivité entre le service de Gestion des API et le serveur principal. 

>[!NOTE]
>La référence SKU Développeur est destinée au développement et aux tests, tandis que la référence SKU Premium est destinée aux déploiements de production.

La configuration de Gestion des API est la même pour les services principaux qui s’exécutent sur des machines virtuelles Azure VMware Solution et localement. Pour les deux types de déploiements, Gestion des API configure l’adresse IP virtuelle (VIPA) sur l’équilibreur de charge comme point de terminaison principal lorsque le serveur principal est placé derrière un équilibreur de charge NSX sur Azure VMware Solution. 


## <a name="external-deployment"></a>Déploiement externe

Un déploiement externe publie les API consommées par les utilisateurs externes à l’aide d’un point de terminaison public. Les développeurs et les ingénieurs DevOps peuvent gérer les API via le portail Azure ou PowerShell et le portail des développeurs de Gestion des API.

Le diagramme de déploiement externe montre l’ensemble du processus et des acteurs impliqués (affiché en haut). Les acteurs sont les suivants :

- **Administrateur(s) :** représente l’équipe d’administrateurs ou DevOps qui gère Azure VMware Solution au moyen du Portail Azure et de mécanismes d’automatisation comme PowerShell ou Azure DevOps.

- **Utilisateurs :**  Représente les contrôles serveur consommateurs des API exposées et représente à la fois les utilisateurs et les services qui consomment les API.

Le flux de trafic passe par l’instance de Gestion des API, qui soustrait les services principaux, branchés au réseau virtuel hub. La passerelle ExpressRoute achemine le trafic vers le canal ExpressRoute Global Reach et atteint un équilibreur de charge NSX distribuant le trafic entrant aux différentes instances des services principaux.

Gestion des API a une API publique Azure, et l’activation du service Azure DDoS Protection est recommandée. 

:::image type="content" source="media/api-management/external-deployment.png" alt-text="Déploiement externe – Gestion des API pour Azure VMware Solution":::


## <a name="internal-deployment"></a>Déploiement interne

Un déploiement interne publie des API consommées par des utilisateurs ou des systèmes internes. L’équipe DevOps et les développeurs d’API utilisent les mêmes outils de gestion et le même portail des développeurs que dans le cadre du déploiement externe.

Les déploiements internes peuvent être effectués [avec Azure Application Gateway](../api-management/api-management-howto-integrate-internal-vnet-appgateway.md) pour créer un point de terminaison public et sécurisé pour l’API.  Les capacités de la passerelle sont utilisées pour créer un déploiement hybride qui permet différents scénarios.  

* Utilisez la même ressource de Gestion des API pour la consommation à la fois par les consommateurs internes et externes.

* Utilisez une seule ressource de Gestion des API avec un sous-ensemble d’API défini et disponible pour les consommateurs externes.

* Fournissez un moyen simple pour activer et désactiver l’accès à la Gestion des API à partir de l’Internet public.

Le diagramme de déploiement ci-dessous montre les consommateurs qui peuvent être internes ou externes, chaque type accédant à des API identiques ou différentes.

Dans un déploiement interne, les API sont exposées à la même instance de Gestion des API. Devant Gestion des API, Application Gateway est déployé avec la capacité Azure Web Application Firewall (WAF) activée. Un ensemble d’écouteurs et de règles HTTP est également déployé pour filtrer le trafic, en exposant uniquement un sous-ensemble des services principaux fonctionnant sur Azure VMware Solution.


* Le trafic interne est acheminé via la passerelle ExpressRoute vers Pare-feu Azure, puis vers Gestion des API, directement ou via les règles de trafic.   

* Le trafic externe entre dans Azure via Application Gateway, qui utilise la couche de protection externe pour la Gestion des API.


:::image type="content" source="media/api-management/internal-deployment.png" alt-text="Déploiement interne – Gestion des API pour Azure VMware Solution" lightbox="media/api-management/internal-deployment.png":::