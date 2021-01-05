---
title: Fonctionnalités de demande de données client pour Azure Digital Twins
titleSuffix: Azure Digital Twins
description: Cet article présente les processus d’exportation et de suppression de données personnelles dans Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 05/05/2020
ms.topic: conceptual
ms.service: digital-twins
services: digital-twins
ms.openlocfilehash: 9942056e59a46c61b4e9da32aab191d4b72db968
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92461757"
---
# <a name="azure-digital-twins-customer-data-request-features"></a>Fonctionnalités de demande de données client Azure Digital Twins

Azure Digital Twins est une plateforme de développement permettant de créer des représentations numériques sécurisées d’un environnement d’entreprise. Les représentations sont pilotées par les données d’état actives des sources de données sélectionnées par les utilisateurs.

[!INCLUDE [gdpr-intro-sentence](../../includes/gdpr-intro-sentence.md)]

Les représentations numériques appelées *jumeaux numériques* dans Azure Digital Twins représentent des entités dans des environnements réels et sont associées à des identificateurs. Microsoft ne conserve aucune information et n’a pas accès aux données qui permettraient la corrélation entre les identificateurs et les utilisateurs. 

La plupart des jumeaux numériques dans Azure Digital Twins ne représentent pas directement les entités personnelles : les objets classiques représentés peuvent être une salle de réunion Office ou une usine. Toutefois, les utilisateurs peuvent décider que certaines entités sont personnelles et gérer eux-mêmes leurs propres méthodes de suivi des ressources ou de l’inventaire afin d’associer des jumeaux numériques à des personnes. Azure Digital Twins gère et stocke toutes les données associées aux jumeaux numériques comme s’il s’agissait de données personnelles.

Pour afficher, exporter et supprimer les données personnelles qui peuvent être référencées dans une demande d’objet de données, un administrateur Azure Digital Twins peut utiliser le [**portail Azure**](https://portal.azure.com/) pour les utilisateurs et les rôles ou les [**API REST Azure Digital Twins**](/rest/api/azure-digitaltwins/) pour les jumeaux numériques. Le portail Azure et les API REST offrent différentes méthodes permettant aux utilisateurs de traiter de telles demandes d’objet de données.

## <a name="identifying-customer-data"></a>Identification des données client

Azure Digital Twins considère les *données personnelles* comme des données associées à ses administrateurs et à ses utilisateurs. 

Azure Digital Twins stocke l’ *ID d’objet* [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) d’utilisateurs ayant accès à l’environnement. Azure Digital Twins dans le portail Azure affiche les adresses e-mail des utilisateurs, mais ces adresses e-mail ne sont pas stockées dans Azure Digital Twins. Elles sont recherchées dynamiquement dans Azure Active Directory à l’aide de l’ID d’objet Azure Active Directory.

## <a name="deleting-customer-data"></a>Suppression des données client

Les administrateurs Azure Digital Twins peuvent utiliser le portail Azure pour supprimer des données relatives aux utilisateurs. Il est également possible d’effectuer des opérations de suppression sur des jumeaux numériques individuels à l’aide des API REST Azure Digital Twins. Pour plus d’informations sur les API disponibles, consultez la [documentation sur les API REST Azure Digital Twins](/rest/api/azure-digitaltwins/).

## <a name="exporting-customer-data"></a>Exportation des données client

Azure Digital Twins stocke les données relatives aux jumeaux numériques. Les utilisateurs peuvent récupérer et afficher ces données via des API REST, et exporter ces données à l’aide de la copie et du collage. 

Les données client, notamment les rôles d’utilisateur et les attributions de rôle, peuvent être sélectionnées, copiées et collées à partir du portail Azure. 

## <a name="links-to-additional-documentation"></a>Liens vers la documentation complémentaire

Pour obtenir la liste complète des API du service Azure Digital Twins, consultez la [documentation sur les API REST Azure Digital Twins](/rest/api/azure-digitaltwins/).