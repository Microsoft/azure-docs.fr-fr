---
author: memildin
ms.author: memildin
manager: rkarlin
ms.date: 06/30/2020
ms.topic: include
ms.openlocfilehash: 051cd8646718c17c288b06a535d35e1eed5e9259
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95561154"
---
Security Center fournit une protection contre les menaces en temps réel pour vos environnements conteneurisés, et génère des alertes en cas d’activités suspectes. Vous pouvez utiliser ces informations pour remédier rapidement aux problèmes de sécurité et améliorer la sécurité de vos conteneurs.

Security Center offre une protection contre les menaces à différents niveaux : 

* **Niveau de l’hôte (fourni par Azure Defender pour les serveurs)** : à l’aide du même agent Log Analytics que celui utilisé par Security Center sur d’autres machines virtuelles, Azure Defender effectue un monitoring des nœuds AKS Linux pour repérer les activités suspectes, par exemple, la détection de web shell et la connexion avec des adresses IP suspectes connues. L’agent surveille également les analyses propres au conteneur, comme la création de conteneurs privilégiés, l’accès suspect à des serveurs d’API et à des serveurs Secure Shell (SSH) s’exécutant dans un conteneur Docker.

    >[!IMPORTANT]
    > Si vous choisissez de ne pas installer les agents sur vos machines hôtes, vous ne recevrez qu’une partie des avantages et alertes de sécurité relatifs à la protection contre les menaces. Vous recevrez toujours les alertes liées à l’analyse réseau et aux communications avec des serveurs malveillants.

    Pour obtenir la liste des alertes au niveau de l’hôte AKS, consultez la [Table de référence des alertes](../articles/security-center/alerts-reference.md#alerts-containerhost).


* **Niveau de cluster AKS (fourni par Azure Defender pour Kubernetes)** : au niveau du cluster, la protection contre les menaces est basée sur l’analyse des journaux d’audit Kubernetes. Pour activer cette analyse **sans agent**, activez Azure Defender. Pour générer des alertes à ce niveau, Security Center supervise vos services managés par AKS à l’aide des journaux collectés par AKS. Les tableaux de bord Kubernetes exposés, la création de rôles dotés de privilèges élevés et la création de montages sensibles sont des exemples d’événements à ce niveau.

    >[!NOTE]
    > Security Center génère des alertes de sécurité pour les actions et les déploiements Azure Kubernetes Service, qui se produisent après l’activation de l’option Kubernetes dans les paramètres de l’abonnement. 

    Pour obtenir la liste des alertes au niveau du cluster AKS, consultez la [table de référence des alertes](../articles/security-center/alerts-reference.md#alerts-akscluster).

De plus, notre équipe mondiale d’experts en sécurité surveille en permanence l’évolution des menaces. Ils ajoutent des alertes et des vulnérabilités propres aux conteneurs au fur et à mesure de leur découverte.

> [!TIP]
> Vous pouvez simuler des alertes de conteneur en suivant les instructions données dans [ce billet de blog](https://techcommunity.microsoft.com/t5/azure-security-center/how-to-demonstrate-the-new-containers-features-in-azure-security/ba-p/1011270).