---
title: Développement et intégration de l’API Azure ExpressRoute CrossConnnections
description: Cet article fournit aux partenaires ExpressRoute une vue d’ensemble détaillée sur le type de ressource expressRouteCrossConnections.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 02/06/2020
ms.author: duau
ms.openlocfilehash: b80392231dba26a10141dcd8247b092e8171894c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011980"
---
# <a name="expressroute-crossconnnections-api-development-and-integration"></a>Développement et intégration de l’API ExpressRoute CrossConnnections

L’API ExpressRoute Partner Resource Manager permet aux partenaires ExpressRoute de gérer la configuration de couche 2 et de couche 3 des circuits ExpressRoute clients. L’API ExpressRoute Partner Resource Manager introduit un nouveau type de ressource, **expressRouteCrossConnections**. Les partenaires utilisent cette ressource pour gérer les circuits ExpressRoute clients.

## <a name="workflow"></a>Workflow

La ressource expressRouteCrossConnections est une ressource cachée du circuit ExpressRoute. Lorsqu’un client Azure crée un circuit ExpressRoute et sélectionne un partenaire ExpressRoute spécifique, Microsoft crée une ressource expressRouteCrossConnections dans l’abonnement de gestion d’Azure ExpressRoute du partenaire. Ce faisant, Microsoft définit un groupe de ressources dans lequel créer la ressource expressRouteCrossConnections. La norme d’attribution de nom pour le groupe de ressources est **CrossConnection-* PeeringLocation*,** où PeeringLocation = l’emplacement d’ExpressRoute. Par exemple, si un client crée un circuit ExpressRoute à Denver, la CrossConnection est créée dans l’abonnement Azure du partenaire dans le groupe de ressources suivant : **CrossConnnection-Denver**.

Les partenaires ExpressRoute gèrent la configuration des couches 2 et 3 en émettant des opérations REST sur la ressource expressRouteCrossConnections.

## <a name="benefits"></a>Avantages

Avantages de la migration vers la ressource expressRouteCrossConnections :

* Toutes les futures améliorations apportées aux partenaires ExpressRoute seront disponibles sur la ressource ExpressRouteCrossConnection.

* Les partenaires peuvent appliquer un [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md) à la ressource expressRouteCrossConnection. Ces contrôles peuvent définir des autorisations pour lesquelles les comptes d’utilisateurs peuvent modifier la ressource expressRouteCrossConnection et ajouter/mettre à jour/supprimer des configurations de Peering.

* La ressource expressRouteCrossConnection expose des API qui peuvent être utiles pour la résolution des problèmes de connexion ExpressRoute. Cela comprend la table ARP, le résumé de la table de route BGP et les détails de la table de route BGP. Cette capacité n’est pas prise en charge par les API de déploiement classiques.

* Les partenaires peuvent également rechercher les communautés publiées concernant le Peering Microsoft à l’aide de la ressource *RouteFilter*.

## <a name="api-development-and-integration-steps"></a>Étapes de développement et d’intégration d’API

Pour se développer par rapport à l’API partenaire, les partenaires ExpressRoute s’appuient sur une configuration cliente et partenaire de test. La configuration cliente de test sera utilisée pour créer des circuits ExpressRoute dans des emplacements tests de Peering qui sont mappés à des appareils et ports factices. La configuration partenaire de test est utilisée pour gérer les circuits ExpressRoute créés à l’emplacement test de Peering.

### <a name="1-enlist-subscriptions"></a>1. Inscrire des abonnements

Pour demander la configuration cliente et partenaire de test, inscrivez deux abonnements Azure en Paiement à l’utilisation à votre contact d’ingénierie ExpressRoute :
* **ExpressRoute_API_Dev_Provider_Sub :** Cet abonnement est utilisé pour gérer les circuits ExpressRoute créés dans des emplacements tests de Peering sur des appareils et des ports factices.

* **ExpressRoute_API_Dev_Customer_Sub :** Cet abonnement est utilisé pour créer des circuits ExpressRoute dans des emplacements tests de Peering qui sont mappés à des appareils et ports factices.

Emplacements tests de Peering : les appareils et ports factices ne sont pas exposés aux clients de production par défaut. Pour créer des circuits ExpressRoute qui mappent à la configuration de test, vous devez activer un indicateur de fonctionnalité d’abonnement.

### <a name="2-register-the-dev_provider-subscription-to-access-the-expressroutecrossconnections-api"></a>2. Inscrire l’abonnement Dev_Provider pour accéder à l’API expressRouteCrossConnections

Pour accéder à l’API expressRouteCrossConnections, l’abonnement partenaire doit être inscrit dans le **fournisseur de ressources Microsoft.Network**. Pour terminer le processus d’inscription, suivez les étapes décrites dans l’article [Fournisseurs et types de ressources Azure](../azure-resource-manager/management/resource-providers-and-types.md#azure-portal).

### <a name="3-set-up-authentication-for-azure-resource-manager-rest-api-calls"></a>3. Configurer l’authentification pour les appels d’API REST Azure Resource Manager

La plupart des services Azure requièrent que le code client s’authentifie avec Resource Manager, à l’aide d’informations d’identification valides, avant d’appeler les API de service. L’authentification est coordonnée entre les différents intervenants par Azure AD et fournit au client un jeton d’accès comme preuve d’authentification.

Le processus d’authentification implique deux étapes principales :

1. [Inscrire le client](/rest/api/azure/#register-your-client-application-with-azure-ad).
2. [Créez la demande d’accès](/rest/api/azure/#create-the-request).

### <a name="4-provide-network-contributor-permission-to-the-client-application"></a>4. Accorder l’autorisation Contributeur de réseau à l’application cliente

Une fois que l’authentification a été correctement configurée, vous devez accorder à votre application cliente l’accès au contributeur de réseau, sous le Dev_Provider_Sub. Pour accorder l’autorisation, connectez-vous au [Portail Azure](https://ms.portal.azure.com/#home) et procédez comme suit :

1. Accéder à Abonnements et sélectionner le Dev_Provider_Sub
2. Accéder à Contrôle d’accès (IAM)
3. Ajouter une attribution de rôle
4. Sélectionner le rôle Contributeur de réseau
5. Attribuer l’accès à l’utilisateur, au groupe ou au principal de service Azure AD
6. Sélectionner votre application cliente
7. Enregistrer les modifications

### <a name="5-develop"></a>5. Développer

Développez en fonction de l’[API expressRouteCrossConnections](/rest/api/expressroute/expressroutecrossconnections).

## <a name="rest-api"></a>API REST

Pour obtenir de la documentation sur l’API REST, consultez [API REST ExpressRoute CrossConnections](/rest/api/expressroute/expressroutecrossconnections).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur toutes les API REST ExpressRoute, consultez [API REST ExpressRoute](/rest/api/expressroute/).