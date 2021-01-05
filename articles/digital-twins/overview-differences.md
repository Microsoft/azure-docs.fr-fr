---
title: Différences par rapport à la première version
titleSuffix: Azure Digital Twins
description: Découvrez ce qui a changé dans la nouvelle version d’Azure Digital Twins
author: baanders
ms.author: baanders
ms.date: 3/12/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: af8d768ceeaacdf1c8a0f6cdc8be0041e4ae4ed1
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93099077"
---
# <a name="what-is-the-new-azure-digital-twins-how-is-it-different-from-the-previous-version-2018"></a>Qu’est-ce que la nouvelle version Azure Digital Twins ? En quoi est-elle différente de la version précédente (2018) ?

La première préversion publique d’Azure Digital Twins a été publiée en octobre 2018. Bien que les principaux concepts de cette première version soient toujours valables dans le nouveau service, la plupart des interfaces et détails d’implémentation ont été modifiés et le service a ainsi gagné en flexibilité et en accessibilité. Ces modifications ont été motivées par les commentaires envoyés par les clients.

> [!IMPORTANT]
> En raison des capacités étendues du nouveau service, le précédent service Azure Digital Twins sera supprimé à la fin de l’année 2020.

Si vous avez utilisé la première version d’Azure Digital Twins au cours de la première préversion publique, utilisez les informations et les bonnes pratiques de cet article pour savoir comment utiliser le nouveau service et tirer parti de ses fonctionnalités.

## <a name="differences-by-topic"></a>Différences par rubrique

Le graphique ci-dessous fournit une vue côte à côte des concepts qui ont changé entre la version précédente du service et le nouveau service (actuel).

| Rubrique | Version précédente | Nouvelle version |
| --- | --- | --- | --- |
| **Modélisation**<br>*Flexibilité accrue* | La version précédente était conçue autour des espaces intelligents et était donc fournie avec un vocabulaire intégré pour les bâtiments. | La nouvelle version d’Azure Digital Twins n’est pas réservée à un domaine spécifique. Vous pouvez définir votre propre vocabulaire personnalisé et vos propres modèles personnalisés pour votre solution, afin de représenter plus de types d’environnements de manière plus flexible.<br><br>Pour plus d’informations, consultez [*Concepts : Modèles personnalisés*](concepts-models.md). |
| **Topologie**<br>*Flexibilité accrue*| La version précédente prenait en charge une structure de données sous forme d’arborescence, adaptée aux espaces intelligents. Les jumeaux numériques étaient connectés avec des relations hiérarchiques. | Avec la nouvelle version, votre jumeau numérique peut être connecté à des topologies de graphe arbitraires, organisées comme vous le souhaitez. Vous bénéficiez ainsi d’une plus grande flexibilité pour exprimer les relations complexes du monde réel.<br><br>Pour plus d’informations, consultez [*Concepts : Jumeaux numériques et graphe de jumeaux*](concepts-twins-graph.md). |
| **Calcul**<br>*Plus riche et plus flexible* | Dans la version précédente, la logique de traitement des événements et de télémétrie était définie dans des fonctions JavaScript définies par l’utilisateur. Le débogage avec les fonctions définies par l’utilisateur était limité. | La nouvelle version a un modèle de calcul ouvert : vous fournissez une logique personnalisée en attachant des ressources de calcul externes comme [Azure Functions](../azure-functions/functions-overview.md). Cela vous permet d’utiliser le langage de programmation de votre choix, d’accéder à des bibliothèques de code personnalisées sans restriction et de tirer parti des éventuelles ressources de développement et de débogage du service externe.<br><br>Pour plus d’informations, consultez [*Guide pratique : Configurer une fonction Azure pour le traitement des données*](how-to-create-azure-function.md). |
| **Gestion des appareils avec IoT Hub**<br>*Accessibilité accrue* | La version précédente gérait les appareils avec une instance [IoT Hub](../iot-hub/about-iot-hub.md) qui était interne au service Azure Digital Twins. Ce hub intégré n’était pas entièrement accessible aux développeurs. | Dans la nouvelle version, vous « apportez votre propre IoT Hub » en attachant une instance IoT Hub créée de manière indépendante (ainsi que les appareils qu’elle gère déjà). Cela vous donne un accès complet aux fonctionnalités IoT Hub et vous permet de contrôler la gestion des appareils.<br><br>Pour plus d’informations, consultez [*Guide pratique : Ingérer la télémétrie depuis IoT Hub*](how-to-ingest-iot-hub-data.md). |
| **Sécurité**<br>*Plus standard* | La version précédente comportait des rôles prédéfinis que vous pouviez utiliser pour gérer l’accès à votre instance. | La nouvelle version s’intègre avec le même service principal de [Contrôle d’accès en fonction du rôle (RBAC) Azure](../role-based-access-control/overview.md) que celui qu’utilisent les autres services Azure. Cela simplifie l’authentification entre les autres services Azure de votre solution, comme IoT Hub, Azure Functions, Event Grid et d’autres.<br>Avec le contrôle d’accès en fonction du rôle (RBAC), vous pouvez utiliser des rôles prédéfinis ou créer et configurer des rôles personnalisés.<br><br>Pour plus d’informations, consultez [*Concepts : Sécurité pour les solutions Azure Digital Twins*](concepts-security.md). |
| **Extensibilité**<br>*Une échelle plus grande* | La version précédente avait des limitations d’échelle pour les appareils, les messages, les graphes et les unités d’échelle. Une seule instance d’Azure Digital Twins était prise en charge par abonnement.  | La nouvelle version s’appuie sur une nouvelle architecture avec une scalabilité améliorée et offre une plus grande puissance de calcul. Elle prend également en charge 10 instances par région et par abonnement.<br><br>Consultez [*Référence : limites de service*](reference-service-limits.md) pour plus d’informations sur les limites de la version actuelle. |

## <a name="service-limits"></a>Limites du service

Pour obtenir la liste des limites Azure Digital Twins, consultez [*Référence : limites de service*](reference-service-limits.md).

## <a name="next-steps"></a>Étapes suivantes

Apprenez à utiliser Azure Digital Twins en suivant le premier tutoriel :

[*Tutoriel : Coder une application cliente*](tutorial-code.md)