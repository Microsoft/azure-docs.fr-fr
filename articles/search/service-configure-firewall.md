---
title: Configurer un pare-feu IP pour votre service Recherche cognitive Azure
titleSuffix: Azure Cognitive Search
description: Configurez les stratégies de contrôle IP pour restreindre l’accès à votre service Recherche cognitive Azure.
manager: nitinme
author: mrcarter8
ms.author: mcarter
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 05/11/2020
ms.openlocfilehash: 3e8a94b6b9b71d2d71b634edd70ea4150652b143
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88932795"
---
# <a name="configure-ip-firewall-for-azure-cognitive-search"></a>Configurer le pare-feu IP pour Recherche cognitive Azure

Recherche cognitive Azure prend en charge les règles IP pour la prise en charge du pare-feu entrant. Ce modèle fournit une couche supplémentaire de sécurité pour votre service de recherche, similaire aux règles IP que vous trouverez dans un groupe de sécurité de réseau virtuel Azure. Avec ces règles IP, vous pouvez configurer votre service de recherche pour qu’il soit accessible uniquement à partir d’un ensemble d’ordinateurs et/ou de services cloud approuvés. L’accès aux données stockées dans votre service de recherche à partir de ces ensembles d’ordinateurs et de services approuvés nécessite toujours que l’appelant présente un jeton d’autorisation valide.

> [!Important]
> Les règles IP sur votre service Recherche cognitive Azure peut être configurée à l’aide du Portail Azure ou de l’[API REST de gestion version 2020-03-13](/rest/api/searchmanagement/).

## <a name="configure-an-ip-firewall-using-the-azure-portal"></a><a id="configure-ip-policy"></a> Configurer un pare-feu IP à l’aide du Portail Azure

Pour définir la stratégie de contrôle d’accès IP dans le Portail Azure, accédez à la page de votre service Recherche cognitive Azure et sélectionnez **Mise en réseau** dans le menu de navigation. La connectivité réseau des points de terminaison doit être **publique**. Si votre connectivité est définie sur **Privée**, vous pouvez accéder à votre service de recherche uniquement via un point de terminaison privé.

![Capture d’écran montrant comment configurer le pare-feu IP dans le Portail Azure](./media/service-configure-firewall/azure-portal-firewall.png)

Le Portail Azure permet de spécifier des adresses IP et des plages d’adresses IP au format CIDR. Un exemple de notation CIDR est 8.8.8.0/24, qui représente les adresses IP comprises entre 8.8.8.0 et 8.8.8.255.

> [!NOTE]
> Une fois que vous avez activé la stratégie de contrôle d’accès IP pour votre service Recherche cognitive Azure, toutes les demandes adressées au plan de données à partir d’ordinateurs ne figurant pas dans la liste des plages d’adresses IP autorisées sont rejetées. Lorsque des règles IP sont configurées, certaines fonctionnalités du Portail Azure sont désactivées. Vous pouvez voir et gérer les informations au niveau du service, mais l'accès du portail aux données d'index et aux divers composants de ce service, comme les définitions d'index, d'indexeur et d'ensemble de compétences, est limité pour des raisons de sécurité.

### <a name="requests-from-your-current-ip"></a>Demandes à partir de votre adresse IP actuelle

Pour simplifier le développement, le portail Azure vous aide à identifier et à ajouter l’adresse IP de votre ordinateur client à la liste autorisée, Les applications qui s’exécutent sur votre ordinateur peuvent ensuite accéder à votre service Recherche cognitive Azure.

Le portail détecte automatiquement l’adresse IP de votre client. Il peut s’agir de l’adresse IP du client de votre ordinateur ou de la passerelle réseau. N’oubliez pas de supprimer cette adresse IP avant de mettre vos charges de travail en production.

Pour ajouter votre adresse IP actuelle à la liste des adresses IP, cochez **Ajouter l’adresse IP de votre client**. Ensuite, sélectionnez **Enregistrer**.

![Capture d’écran montrant comment configurer les paramètres de pare-feu IP pour autoriser l’adresse IP actuelle](./media/service-configure-firewall/enable-current-ip.png)

## <a name="troubleshoot-issues-with-an-ip-access-control-policy"></a><a id="troubleshoot-ip-firewall"></a>Résoudre les problèmes de stratégie de contrôle d’accès IP

Vous pouvez résoudre les problèmes de stratégie de contrôle d’accès IP en utilisant les options suivantes :

### <a name="azure-portal"></a>Portail Azure

L’activation d’une stratégie de contrôle d’accès IP pour votre service Recherche cognitive Azure bloque toutes les demandes des ordinateurs en dehors de la liste des plages d’adresses IP autorisées, y compris le Portail Azure.  Vous pouvez voir et gérer les informations au niveau du service, mais l'accès du portail aux données d'index et aux divers composants de ce service, comme les définitions d'index, d'indexeur et d'ensemble de compétences, est limité pour des raisons de sécurité. 

### <a name="sdks"></a>Kits SDK

Quand vous accédez au service Recherche cognitive Azure à l’aide du Kit de développement logiciel (SDK) à partir d’ordinateurs ne figurant pas dans la liste autorisée, une réponse **403 interdit** générique est retournée sans aucun détail supplémentaire. Vérifiez la liste des adresses IP autorisées pour votre compte et assurez-vous que la configuration appropriée a été mise à jour pour votre service de recherche.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’accès à votre service de recherche via Azure Private Link, consultez l’article suivant :

* [Créer un point de terminaison privé pour une connexion sécurisée à Recherche cognitive Azure](service-create-private-endpoint.md)