---
title: Resource Health pour Azure Cloud Services (classique)
description: Cet article traite de la prise en charge de Resource Health Check (RHC) pour Microsoft Azure Cloud Services (classique).
services: cloud-services
author: tanmaygore
ms.service: cloud-services
ms.topic: article
ms.date: 9/1/2020
ms.author: tagore
ms.openlocfilehash: f99dd8131df9f8bc5d3e4013d4438faa8c25e53b
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92072711"
---
# <a name="resource-health-check-rhc-support-for-azure-cloud-services-classic"></a>Prise en charge de Resource Health Check (RHC) pour Azure Cloud Services (classique)
Cet article traite de la prise en charge de Resource Health Check (RHC) pour [Microsoft Azure Cloud Services (classique)](https://azure.microsoft.com/services/cloud-services).

[Azure Resource Health](../service-health/resource-health-overview.md) pour les services cloud vous aide à diagnostiquer et à obtenir de l’aide pour les problèmes de service qui ont un impact sur votre déploiement, vos rôles et vos instances de rôle Azure Cloud Services. La fonctionnalité rend compte de l’intégrité actuelle et passée de vos services cloud aux niveaux Déploiement, Rôle et Instance de rôle.

Azure fait état des problèmes qui touchent un large éventail de clients Azure. Resource Health vous donne un tableau de bord personnalisé de l’intégrité de vos ressources. Resource Health vous montre toutes les fois où vos ressources ont été non disponibles en raison de problèmes de service Azure. Ces données vous permettent de voir facilement si un contrat SLA n’a pas été respecté.

:::image type="content" source="media/cloud-services-allocation-failure/rhc-blade-cloud-services.png" alt-text="Image affichant le panneau de vérification de resource health dans le portail Azure.":::

## <a name="how-health-is-checked-and-reported"></a>Comment l’intégrité est-elle vérifiée et signalée ?
L’intégrité des ressources est signalée au niveau du déploiement ou du rôle. Le contrôle d’intégrité se produit au niveau de l’instance de rôle, nous agrégeons l’état et nous le signalons au niveau du rôle. Par exemple, si toutes les instances de rôle sont disponibles, l’état du rôle est disponible. De même, nous agrégeons l’état d’intégrité de tous les rôles et nous le signalons au niveau du déploiement. Par exemple, si tous les rôles sont disponibles, l’état du déploiement devient disponible. 

## <a name="why-i-cannot-see-health-status-for-my-staging-slot-deployment"></a>Pourquoi ne puis-je pas voir l’état d’intégrité de mon déploiement d’emplacement de préproduction ?
Les contrôles d’intégrité des ressources fonctionnent uniquement pour le déploiement d’emplacement de production. Le déploiement d’emplacement de préproduction n’est pas encore pris en charge. 

## <a name="does-resource-health-check-also-check-the-health-of-the-application"></a>Resource Health Check vérifie-t-il également l’intégrité de l’application ?
Non, le contrôle d’intégrité ne se produit que pour les instances de rôle et n’analyse pas l’intégrité de l’application. Par exemple, même si une instance de rôle sur trois est non saine, l’application peut continuer à être disponible. RHC n’utilise pas de [sondes d’équilibrage de charge](../load-balancer/load-balancer-custom-probe-overview.md) ni de sonde d’agent invité. Par conséquent, les clients doivent continuer à utiliser des sondes d’équilibrage de charge pour surveiller l’intégrité de leur application. 

## <a name="what-are-the-annotations-for-cloud-services"></a>Quelles sont les annotations pour Azure Cloud Services ?
Les annotations sont l’état d’intégrité du déploiement ou des rôles. Il existe des annotations différentes en fonction de l’état d’intégrité, de la raison du changement d’état, etc. 

## <a name="what-does-it-mean-by-role-instance-being-unavailable"></a>Que signifie le fait que l’instance de rôle soit « non disponible » ?
Cela signifie que l’instance de rôle n’envoie pas de signal sain à la plateforme. Vérifiez l’état de l’instance de rôle pour obtenir une explication détaillée de la raison pour laquelle le signal sain n’est pas émis.

## <a name="what-does-it-mean-by-deployment-being-unknown"></a>Que signifie le fait que le déploiement soit « inconnu » ?
« Inconnu » signifie que l’intégrité agrégée du déploiement d’Azure Cloud Services ne peut pas être déterminée. Cela indique généralement qu’aucun déploiement de production n’a été créé pour le service cloud, que le déploiement vient d’être créé (et qu’Azure commence à collecter les événements d’intégrité) ou que la plateforme rencontre des problèmes lors de la collecte des événements d’intégrité pour ce déploiement.

## <a name="why-does-role-instance-annotations-mentions-vms-instead-of-role-instances"></a>Pourquoi les annotations des instances de rôle mentionnent-elles des machines virtuelles plutôt que des instances de rôle ?
Étant donné que les instances de rôle sont fondamentalement des machines virtuelles et que le contrôle d’intégrité des machines virtuelles est réutilisé pour les instances de rôle, le terme « machine virtuelle » est utilisé pour représenter les instances de rôle. 

## <a name="cloud-services-deployment-level-annotations--their-meanings"></a>Annotations Azure Cloud Services (au niveau du déploiement) & leurs significations
| Annotation | Description | 
| --- | --- | 
| Disponible| Aucun problème connu de la plateforme Azure n’affecte ce déploiement Azure Cloud Services |
| Unknown | Nous ne pouvons pas déterminer l’intégrité de ce déploiement Azure Cloud Services pour l’instant | 
| Configuration de Resource Health | Configuration de Resource Health pour cette ressource. Resource Health surveille vos ressources Azure pour fournir des détails sur les événements passés et présents qui les impactent|
| Détérioré | Votre déploiement de service cloud est détérioré. Nous essayons de récupérer automatiquement votre déploiement de service cloud et de déterminer la source du problème. Vous n’avez aucune action supplémentaire à effectuer pour l’instant |
| Unhealthy | Votre déploiement Azure Cloud Services est non sain, car {0} instances de rôle sur un total de {1} sont indisponibles |
| Détérioré | Votre déploiement Azure Cloud Services est non sain, car {0} instances de rôle sur un total de {1} sont indisponibles | 
| Disponible et peut-être impacté | Votre déploiement Azure Cloud Services est en cours d’exécution, mais une interruption de service Azure peut survenir et vous empêcher de vous y connecter. La connectivité sera restaurée une fois la panne résolue |
| Non disponible et peut-être impacté | Une panne du service Azure peut avoir un impact sur l’intégrité de ce déploiement Azure Cloud Services. Votre déploiement Azure Cloud Services sera automatiquement récupéré lorsque la panne sera résolue |
| Inconnu et peut-être impacté | Nous ne pouvons pas déterminer l’intégrité de ce déploiement Azure Cloud Services pour l’instant. Il est possible qu’une panne du service Azure ait un impact sur cette machine virtuelle. Celle-ci sera automatiquement récupérée une fois la panne résolue |

## <a name="cloud-services-role-instance-level-annotations--their-meanings"></a>Annotations Azure Cloud Services (au niveau de l’instance de rôle) & leurs significations
| Annotation | Description | 
| --- | --- | 
| Disponible | Aucun problème connu de la plateforme Azure n’a d’impact sur cette machine virtuelle | 
| Unknown | Nous ne pouvons pas déterminer l’intégrité de cette machine virtuelle pour l’instant |
| Arrêtée et libérée | Cette machine virtuelle s’arrête et est désallouée à la demande d’un utilisateur ou d’un processus autorisé |
| Configuration de Resource Health | Configuration de Resource Health pour cette ressource. Resource Health surveille vos ressources Azure pour fournir des détails sur les événements passés et présents qui les impactent |
| Indisponible | Votre machine virtuelle n’est pas disponible. Nous essayons de récupérer automatiquement votre machine virtuelle et de déterminer la source du problème. Vous n’avez aucune action supplémentaire à effectuer pour l’instant |
| Détérioré | Votre machine virtuelle est détériorée. Nous essayons de récupérer automatiquement votre machine virtuelle et de déterminer la source du problème. Vous n’avez aucune action supplémentaire à effectuer pour l’instant |
| Défaillance matérielle du serveur hôte | Cette machine virtuelle subit une défaillance {HardwareCategory} irrécupérable sur le serveur hôte. Azure va redéployer votre machine virtuelle sur un serveur hôte sain |
| Migration planifiée en raison d’un matériel détérioré | Azure a identifié que le serveur hôte a une {0} dégradée qui sera bientôt défaillante. Si possible, nous allons opérer une migration dynamique de votre machine virtuelle au plus tôt. Dans le cas contraire, nous allons la redéployer après {1} heures UTC. Pour minimiser les risques auxquels votre service est exposé et au cas où le matériel tomberait en panne avant la migration initiée par le système, nous vous recommandons de redéployer votre machine virtuelle dès que possible |
| Disponible et peut-être impacté | Votre machine virtuelle est en cours d’exécution, mais une interruption de service Azure peut survenir et vous empêcher de vous y connecter. La connectivité sera restaurée une fois la panne résolue |
| Non disponible et peut-être impacté | Une panne du service Azure peut avoir un impact sur l’intégrité de cette machine virtuelle. Votre machine virtuelle sera automatiquement récupérée lorsque la panne sera résolue |
| Inconnu et peut-être impacté | Nous ne pouvons pas déterminer l’intégrité de cette machine virtuelle pour l’instant. Il est possible qu’une panne du service Azure ait un impact sur cette machine virtuelle. Celle-ci sera automatiquement récupérée une fois la panne résolue |
| Ressources matérielles allouées | Des ressources matérielles ont été affectées à la machine virtuelle, qui sera en ligne d’ici peu |
| Arrêt et libération | Cette machine virtuelle s’arrête et est désallouée à la demande d’un utilisateur ou d’un processus autorisé |
| Configuration en cours de mise à jour | La configuration de cette machine virtuelle est en cours de mise à jour à la demande d’un utilisateur ou d’un processus autorisé |
| Sélectionné par l’utilisateur | Cette machine virtuelle redémarre à la demande d’un utilisateur ou d’un processus autorisé. Elle sera de nouveau en ligne après le redémarrage |
| Redéploiement sur un hôte différent | Cette machine virtuelle est en cours de redéploiement sur un autre serveur hôte à la demande d’un utilisateur autorisé ou d’un processus. Elle sera de nouveau en ligne après le redéploiement |
| Arrêtée par l’utilisateur | Cette machine virtuelle s’arrête à la demande d’un utilisateur ou d’un processus autorisé |
| Arrêtée par l’utilisateur ou le processus | Cette machine virtuelle s’arrête à la demande d’un utilisateur ou d’un processus autorisé s’exécutant à l’intérieur de la machine virtuelle |
| Démarrée par l’utilisateur | Cette machine virtuelle démarre à la demande d’un utilisateur ou d’un processus autorisé. Elle sera en ligne d’ici peu |
| Redéploiement de la maintenance sur un hôte différent | Cette machine virtuelle est en cours de redéploiement sur un autre serveur hôte à la demande d’un utilisateur autorisé ou d’un processus. Elle sera de nouveau en ligne après le redéploiement |
| Redémarrage initié depuis l’intérieur de la machine | Un redémarrage a été déclenché depuis l’intérieur de la machine virtuelle. La raison peut en être un échec du système d’exploitation de la machine virtuelle, ou une demande faite par un utilisateur ou un processus autorisé. La machine virtuelle sera à nouveau en ligne une fois le redémarrage terminé |
| Redimensionnée par l’utilisateur | Cette machine virtuelle est redimensionnée à la demande d’un utilisateur autorisé ou d’un processus. Elle sera de nouveau en ligne une fois le redimensionnement terminé |
| Incident de machine | Un redémarrage a été déclenché depuis l’intérieur de la machine virtuelle. La raison peut en être un échec du système d’exploitation de la machine virtuelle, ou une demande faite par un utilisateur ou un processus autorisé. La machine virtuelle sera à nouveau en ligne une fois le redémarrage terminé |
| Redémarrage de maintenance pour la mise à jour de l’hôte | Des mises à jour de maintenance sont appliquées au serveur hôte exécutant cette machine virtuelle. Vous n’avez aucune action supplémentaire à effectuer pour l’instant. Elle sera de nouveau en ligne une fois la maintenance terminée |
| Redéploiement de maintenance sur un nouveau matériel | Cette machine virtuelle n’est pas disponible, car elle est en cours de redéploiement sur du matériel plus récent dans le cadre d’un événement de maintenance planifiée. Vous n’avez aucune action supplémentaire à effectuer pour l’instant. Elle sera de nouveau en ligne une fois la maintenance planifiée terminée |
| Machine basse priorité préemptée | Cette machine virtuelle a été préemptée. Cette machine virtuelle basse priorité est en cours d’arrêt et de libération |
| Redémarrage du serveur hôte | Nous sommes désolés, votre machine virtuelle n’est pas disponible en raison d’un redémarrage inattendu du serveur hôte. Le serveur hôte est en cours de redémarrage. La machine virtuelle sera à nouveau en ligne une fois le redémarrage terminé. Vous n’avez aucune action supplémentaire à effectuer pour l’instant |
| Redéploiement en raison d’une défaillance de l’hôte | Nous sommes désolés, votre machine virtuelle n’est pas disponible et est en cours de redéploiement en raison d’une défaillance inattendue sur le serveur hôte. Azure a commencé le processus de récupération automatique et démarre actuellement la machine virtuelle sur un autre hôte. Vous n’avez aucune action supplémentaire à effectuer pour l’instant |
| Défaillance inattendue de l’hôte | Nous sommes désolés, votre machine virtuelle n’est pas disponible en raison d’une défaillance inattendue du serveur hôte. Azure a commencé le processus de récupération automatique et est en train de redémarrer le serveur hôte. Vous n’avez aucune action supplémentaire à effectuer pour l’instant. La machine virtuelle sera à nouveau en ligne une fois le redémarrage terminé |
| Redéploiement en raison d’une maintenance de l’hôte non planifiée | Nous sommes désolés, votre machine virtuelle n’est pas disponible et est en cours de redéploiement en raison d’une défaillance inattendue sur le serveur hôte. Azure a commencé le processus de récupération automatique et démarre actuellement la machine virtuelle sur un autre serveur hôte. Vous n’avez aucune action supplémentaire à effectuer pour l’instant |
| Échec de l’approvisionnement | Nous sommes désolés, votre machine virtuelle n’est pas disponible en raison de problèmes d’approvisionnement inattendus. L’approvisionnement de votre machine virtuelle a échoué en raison d’une erreur inattendue |
| Migration dynamique | Cette machine virtuelle est suspendue en raison d’une opération Migration dynamique préservant la mémoire. La machine virtuelle reprend généralement dans les 10 secondes. Vous n’avez aucune action supplémentaire à effectuer pour l’instant |
| Migration dynamique | Cette machine virtuelle est suspendue en raison d’une opération Migration dynamique préservant la mémoire. La machine virtuelle reprend généralement dans les 10 secondes. Vous n’avez aucune action supplémentaire à effectuer pour l’instant | 
| Disque distant déconnecté | Nous sommes désolés, votre machine virtuelle n’est pas disponible en raison d’une perte de connexion au disque distant. Nous travaillons actuellement au rétablissement de la connexion au disque. Vous n’avez aucune action supplémentaire à effectuer pour l’instant |
| Problème de service Azure | Votre machine virtuelle est concernée par un problème de service Azure |
| Problème réseau | Cette machine virtuelle est concernée par un périphérique réseau TOR (top-of-rack) |
| Indisponible | Votre machine virtuelle n’est pas disponible. Nous ne pouvons pas déterminer la raison de ce temps d’arrêt pour l’instant |
| Redémarrage du serveur hôte | Nous sommes désolés, votre machine virtuelle n’est pas disponible en raison d’un redémarrage inattendu du serveur hôte. Un problème inattendu avec le serveur hôte nous empêche de récupérer automatiquement votre machine virtuelle |
| Redéploiement en raison d’une défaillance de l’hôte | Nous sommes désolés, votre machine virtuelle n’est pas disponible en raison d’une défaillance inattendue du serveur hôte. Un problème inattendu avec l’hôte nous empêche de récupérer automatiquement votre machine virtuelle |
| Défaillance inattendue de l’hôte | Nous sommes désolés, votre machine virtuelle n’est pas disponible en raison d’une défaillance inattendue du serveur hôte. Un problème inattendu avec l’hôte nous empêche de récupérer automatiquement votre machine virtuelle |
| Redéploiement en raison d’une maintenance de l’hôte non planifiée | Nous sommes désolés, votre machine virtuelle n’est pas disponible en raison d’une défaillance inattendue du serveur hôte. Un problème inattendu avec l’hôte nous empêche de récupérer automatiquement votre machine virtuelle |
| Échec de l’approvisionnement | Nous sommes désolés, votre machine virtuelle n’est pas disponible en raison de problèmes d’approvisionnement inattendus. L’approvisionnement de votre machine virtuelle a échoué en raison d’une erreur inattendue |
| Disque distant déconnecté | Nous sommes désolés, votre machine virtuelle n’est pas disponible en raison d’une perte de connexion au disque distant. Un problème inattendu nous empêche de récupérer automatiquement votre machine virtuelle |
| Redémarrage en raison de la mise à jour du système d’exploitation invité | Un redémarrage a été initié par la plateforme Azure pour appliquer une nouvelle mise à jour du système d’exploitation invité. La machine virtuelle sera à nouveau en ligne une fois le redémarrage terminé |