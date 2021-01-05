---
title: FAQ sur Internet Analyzer | Microsoft Docs
description: Il s’agit de la FAQ sur Azure Internet Analyzer.
services: internet-analyzer
author: diego-perez-botero
ms.service: internet-analyzer
ms.topic: guide
ms.date: 10/16/2019
ms.author: mebeatty
ms.openlocfilehash: a4a5b058666fab3e9048a7d92726dccd1360ff37
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74184250"
---
# <a name="azure-internet-analyzer-faq-preview"></a>FAQ sur Azure Internet Analyzer (Préversion)

Il s’agit de la FAQ sur Azure Internet Analyzer. Si vous avez des questions supplémentaires, veuillez accéder au [Forum de commentaires](https://aka.ms/internetAnalyzerFeedbackForum). Vous pourrez y publier votre question. Lorsqu’une question est fréquemment posée, nous l’ajoutons à cet article pour qu’elle soit facile et rapide à trouver.

## <a name="how-do-i-participate-in-the-preview"></a>Comment faire pour participer à la préversion ?

La préversion est disponible pour certains clients. Si vous souhaitez disposer de la préversion, veuillez procéder ainsi :

1. Connectez-vous au [portail Azure](https://ms.portal.azure.com).
2. Accédez à la page **Abonnements**.
3. Cliquez sur l’abonnement Azure avec lequel vous envisagez d’utiliser l’Internet Analyzer.
4. Accédez aux paramètres des **Fournisseurs de ressources** de l’abonnement.
5. Recherchez **Microsoft.Network**, puis cliquez sur le bouton **S’inscrire** (ou **Se réinscrire**).
![Demande d’accès](./media/ia-faq/request-preview-access.png)

6. [Faites approuver votre demande](https://aka.ms/internetAnalyzerContact) en nous fournissant votre adresse e-mail et l’ID de l’abonnement Azure utilisé pour la demande d’accès.
7. Une fois votre demande approuvée, vous recevrez une confirmation par e-mail qui vous permettra de créer/mettre à jour/modifier des ressources Internet Analyzer de l’abonnement Azure autorisé.

## <a name="do-i-need-to-embed-the-client-to-run-a-test"></a>Dois-je incorporer un client pour exécuter un test ?

Oui, le client Internet Analyzer doit être installé dans votre application pour collecter des mesures propres à vos utilisateurs. [Découvrez comment Installer le client.](internet-analyzer-embed-client.md) 

## <a name="do-i-get-billed-for-participating-in-the-preview"></a>Suis-je facturé pour ma participation à la préversion ?
Non, l’utilisation de la préversion d’Azure Internet Analyzer est gratuite. Pendant la période de préversion, il n’existe aucun contrat de niveau de service.

## <a name="what-scenarios-is-internet-analyzer-designed-to-address"></a>Quels sont les scénarios qu’Internet Analyzer peut résoudre ?

Internet Analyzer est conçu pour vous fournir des insights relatives aux performances de votre réseau en fonction de votre population d’utilisateurs. Pour vous aider à prendre les meilleures décisions en matière de performances pour vos utilisateurs, Internet Analyzer compare les performances de deux points de terminaison Internet à l’aide de votre population d’utilisateurs distincts. Bien qu’Internet Analyzer puisse répondre à une multitude de questions, voici les plus courantes :

* Quel est l’impact d’une migration vers le cloud sur les performances ? 
    * *Test suggéré : Personnalisé (votre infrastructure locale actuelle) par rapport à Azure (n’importe quel point de terminaison préconfiguré)*
* Quelle est la différence en termes de valeur d’un placement de mes données en périphérie, par rapport à un placement dans un centre de données ? 
    *  *Test suggéré : Azure, Azure Front Door, Azure par rapport à Azure CDN de Microsoft*
* Quel est l’avantage d’Azure Front Door en matière de performances ?
    *  *Test suggéré : Custom/ Azure/ CDN par rapport à Azure Front Door*
* Quel est l’avantage d’Azure CDN de Microsoft en matière de performances ? 
    *  *Test suggéré : Custom/ Azure/ AFD par rapport à Azure CDN de Microsoft*
* Quels sont les avantages d’Azure CDN de Microsoft ? 
    *  *Test suggéré : Personnalisé (autre point de terminaison CDN) par rapport à Azure CDN de Microsoft*
* Quel est le meilleur cloud pour votre population d’utilisateurs finaux dans chaque région ? 
    *  *Test suggéré : Personnalisé (autre service cloud) par rapport à Azure (n’importe quel point de terminaison préconfiguré)*

## <a name="which-tests-can-i-run-in-preview"></a>Quels tests puis-je exécuter en préversion ?

Chaque test que vous créez dans Internet Analyzer est composé de deux points de terminaison : le point de terminaison A et le point de terminaison B. Chacune des combinaisons suivantes peut être exécutée en tant que test :  
* Deux points de terminaison préconfigurés,
* Un point de terminaison personnalisé et un point de terminaison préconfiguré, ou
* Deux [points de terminaison personnalisés](internet-analyzer-custom-endpoint.md) (un point de terminaison personnalisé doit résider dans Azure).

Les points de terminaison préconfigurés suivants sont disponibles durant la période de préversion :
* **Régions Azure**
    * Brésil Sud
    * Inde centrale
    * USA Centre
    * Asie Est
    * USA Est
    * OuJapon Est
    * Europe Nord
    * Afrique du Sud Nord
    * Asie Sud-Est
    * Émirats arabes unis Nord
    * Ouest du Royaume-Uni  
    * Europe Ouest
    * USA Ouest
    * USA Ouest 2
* **Plusieurs combinaisons de régions Azure**
    * USA Est, Brésil Sud
    * USA Est, Asie Est
    * Europe Ouest, Brésil Sud
    * Europe Ouest, Asie Sud-Est
    * Europe Ouest, Émirats arabes unis Nord
    * USA Ouest, USA Est
    * USA Ouest, Europe Ouest
    * USA Ouest, Émirats arabes unis Nord
    * Europe Ouest, Émirats arabes unis Nord, Asie Sud-Est
    * USA Ouest, Europe Ouest, Asie Est
    * USA Ouest, Europe Nord, Asie Sud-Est, Émirats arabes unis Nord, Afrique du Sud Nord 
* **Azure + Azure Front Door** : déployé sur une ou plusieurs combinaisons de régions Azure listées ci-dessus
* **Azure + Azure CDN from Microsoft** : déployé sur une combinaison de régions Azure listées ci-dessus
* **Azure + Azure Traffic Manager** : déployé sur plusieurs combinaisons de régions Azure listées ci-dessus

## <a name="how-is-internet-analyzer-different-from-other-monitoring-services-provided-by-azure"></a>En quoi Internet Analyzer diffère-t-il des autres services d’analyse d’Azure ?

Internet Analyzer vous aide à comprendre les performances de vos utilisateurs finaux, mais prendre les meilleures décisions pour améliorer leurs performances. Bien que d’autres outils d’analyse d’Azure fournissent des insights sur vos services Azure, Internet Analyzer se concentre sur la mesure des performances Internet de bout en bout pour vos utilisateurs.

## <a name="how-is-measurement-data-handled-by-internet-analyzer"></a>Comment les données de mesure sont-elles gérées par Internet Analyzer ?

Azure dispose de [processus de sécurité renforcés, il est conforme à une large gamme de normes de conformité](https://azure.microsoft.com/support/trust-center/). Seuls vous et votre équipe avez accès à vos données. Le personnel Microsoft peut avoir un accès restreint uniquement dans certaines circonstances définies, avec votre consentement. Les données sont chiffrées, en transit comme au repos.

## <a name="next-steps"></a>Étapes suivantes

Si vous souhaitez en savoir plus, veuillez consulter la page [Qu’est-ce qu’Internet Analyzer ? (Préversion)](internet-analyzer-overview.md).
