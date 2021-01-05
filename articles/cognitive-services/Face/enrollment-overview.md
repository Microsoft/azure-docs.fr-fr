---
title: Vue d’ensemble de l’inscription à l’API Visage
titleSuffix: Azure Cognitive Services
description: Découvrez le processus d’inscription à Visage pour inscrire des utilisateurs à un service de reconnaissance faciale.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: overview
ms.date: 11/17/2020
ms.author: pafarley
ms.openlocfilehash: ac5106aa661cb2baea31ee15d57e9c6fac8c7192
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350328"
---
# <a name="face-api-enrollment"></a>Inscription à l’API Visage

Pour pouvoir utiliser l’API Visage de Cognitive Services pour la vérification ou l’identification des visages, vous devez inscrire des visages dans un **LargePersonGroup**. Cette présentation approfondie montre les bonnes pratiques pour collecter un consentement explicite des utilisateurs ainsi qu’un exemple de logique pour créer des inscriptions de qualité qui vont optimiser la précision de la reconnaissance.  

## <a name="meaningful-consent"></a>Consentement explicite 

Un des principaux objectifs d’une application d’inscription pour la reconnaissance faciale est de donner aux utilisateurs la possibilité de consentir à l’utilisation d’images de leur visage à des fins spécifiques, comme accéder à un lieu de travail. Étant donné que les technologies de reconnaissance faciale peuvent être perçues comme une collecte de données personnelles sensibles, il est particulièrement important de demander le consentement d’une façon à la fois transparente et respectueuse. Le consentement est explicite pour les utilisateurs quand il leur permet de prendre la décision qui, selon eux, leur convient le mieux.   

En nous appuyant sur Microsoft User Research, sur les principes de l’IA responsable de Microsoft et sur une [recherche externe](ftp://ftp.cs.washington.edu/tr/2000/12/UW-CSE-00-12-02.pdf), nous avons constaté que le consentement est explicite quand il offre les éléments suivants aux utilisateurs qui s’inscrivent dans la technologie :

* Sensibilisation : Les utilisateurs ne doivent pas avoir de doute quand il leur est demandé de fournir leur modèle de visage ou leurs photos d’inscription. 
* Compréhension : Les utilisateurs doivent pouvoir décrire avec précision dans leurs propres termes ce qui leur est demandé, par qui et avec quelles garanties. 
* Liberté de choix : Les utilisateurs ne doivent pas se sentir forcés ou manipulés quand ils choisissent de donner leur consentement et de s’inscrire à la reconnaissance faciale. 
* Contrôle : Les utilisateurs doivent être en mesure de révoquer leur consentement et de supprimer leurs données à tout moment. 

Cette section contient des conseils sur le développement d’une application d’inscription pour la reconnaissance faciale. Ces conseils ont été élaborés à partir de Microsoft User Research dans le contexte de l’inscription de personnes individuelles dans la reconnaissance faciale pour la création d’une entrée. Par conséquent, ces recommandations peuvent ne pas s’appliquer à toutes les solutions de reconnaissance faciale. L’utilisation responsable de l’API Visage dépend fortement du contexte spécifique dans lequel elle est intégrée : les priorités et l’application de ces recommandations doivent donc être adaptées à votre scénario. 

> [!NOTE]
> Il vous incombe d’aligner votre application d’inscription avec les conditions légales applicables dans votre pays, et de refléter avec précision l’ensemble de vos pratiques de collecte et de traitement des données.

## <a name="application-development"></a>Développement d’applications 

Avant de concevoir un flux d’inscription, pensez à la façon dont l’application que vous créez peut respecter les promesses que vous faites aux utilisateurs quant à la façon dont leurs données sont protégées. Les recommandations suivantes peuvent vous aider à créer une expérience d’inscription qui inclut des approches responsables de la sécurisation des données personnelles, de la gestion de la confidentialité des utilisateurs et de la garantie que l’application est accessible à tous les utilisateurs.  

|Catégorie | Recommandations |
|---|---|
|Matériel | Tenez compte de la qualité de l’appareil photo utilisé pour l’inscription. |
|Fonctionnalités d’inscription recommandées | Incluez une étape de connexion avec une authentification multifacteur.</br></br>Liez des informations utilisateur comme un alias ou un numéro d’identification à l’ID de modèle de visage provenant de l’API Visage (appelé ID de personne). Cette association est nécessaire pour récupérer et gérer l’inscription d’un utilisateur. Remarque : L’ID de personne doit être traité comme un secret dans l’application.</br></br>Configurez un processus automatisé pour supprimer toutes les données d’inscription, notamment les modèles de visage et les photos d’inscription des personnes qui ne sont plus des utilisateurs de la technologie de reconnaissance faciale, par exemple les anciens employés.</br></br>Évitez l’inscription automatique, car elle ne donne pas à l’utilisateur la sensibilisation, la compréhension, la liberté de choix ou le contrôle qui est recommandé pour obtenir le consentement. </br></br>Demandez aux utilisateurs l’autorisation d’enregistrer les images utilisées pour l’inscription. C’est utile quand il y a une mise à jour du modèle, car de nouvelles photos d’inscription devront être réinscrites dans le nouveau modèle environ tous les 10 mois. Si les images d’origine ne sont pas enregistrées, les utilisateurs devront suivre le processus d’inscription depuis le début.</br></br>Permettez aux utilisateurs de refuser le stockage des photos dans le système. Pour rendre le choix plus clair, vous pouvez ajouter un deuxième écran de demande de consentement pour l’enregistrement des photos d’inscription. </br></br>Si des photos sont enregistrées, créez un processus automatisé pour réinscrire tous les utilisateurs quand un modèle est mis à jour. Les personnes qui ont enregistré leurs photos d’inscription ne devront pas se réinscrire. </br></br>Créez une fonctionnalité d’application qui permet aux administrateurs désignés de remplacer certains filtres de qualité si un utilisateur a des difficultés à s’inscrire. |
|Sécurité | Cognitive Services suit les [bonnes pratiques](../cognitive-services-virtual-networks.md?tabs=portal) pour le chiffrement des données utilisateur au repos et en transit. Vous trouverez ci-dessous des pratiques supplémentaires qui peuvent vous aider à respecter les promesses en matière de sécurité que vous faites aux utilisateurs lors de l’inscription. </br></br>Prenez les mesures de sécurité nécessaires pour garantir que personne n’a accès à aucun moment à l’ID de personne pendant l’inscription. Remarque : L’ID de personne doit être traité comme un secret dans le système d’inscription. </br></br>Utilisez le [contrôle d’accès en fonction du rôle](../../role-based-access-control/overview.md) dans Cognitive Services. </br></br>Utilisez l’authentification basée sur les jetons et/ou les signatures d’accès partagé (SAS) sur les clés et les secrets pour accéder à des ressources comme les bases de données. En utilisant des jetons de demande ou SAS, vous pouvez accorder un accès limité aux données sans compromettre vos clés de compte, et vous pouvez spécifier un délai d’expiration sur le jeton. </br></br>Ne stockez jamais de secrets, de clés ou de mots de passe dans votre application. |
|Confidentialité de l'utilisateur |Fournissez une gamme d’options d’inscription pour offrir une réponse aux différents niveaux de problèmes de confidentialité. N’obligez pas les utilisateurs à utiliser leurs appareils personnels pour s’inscrire dans un système de reconnaissance faciale. </br></br>Permettez aux utilisateurs de se réinscrire, de révoquer leur consentement et de supprimer les données de l’application d’inscription à tout moment et pour n’importe quelle raison. |
|Accessibilité |Suivez les normes d’accessibilité (par exemple [ADA](https://www.ada.gov/regs2010/2010ADAStandards/2010ADAstandards.htm) ou [W3C](https://www.w3.org/TR/WCAG21/)) pour garantir que l’application est utilisable par des personnes ayant des déficiences visuelles ou de mobilité. |

## <a name="next-steps"></a>Étapes suivantes  

Pour commencer avec un exemple d’application d’inscription, suivez le guide [Créer une application d’inscription](build-enrollment-app.md). Ensuite, personnalisez-la ou écrivez votre propre application en fonction des besoins de votre produit.