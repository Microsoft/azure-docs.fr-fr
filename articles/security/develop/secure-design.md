---
title: Concevoir des applications sécurisées sur Microsoft Azure
description: Cet article décrit les bonnes pratiques à prendre en compte pendant les phases de conception et de configuration des exigences de votre projet d’application web.
author: TerryLanfear
manager: barbkess
ms.author: terrylan
ms.date: 06/11/2019
ms.topic: article
ms.service: security
ms.subservice: security-develop
services: azure
ms.assetid: 521180dc-2cc9-43f1-ae87-2701de7ca6b8
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
ms.openlocfilehash: 743412b7602e5781911cdf190e41a5ee15bfddd4
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96487675"
---
# <a name="design-secure-applications-on-azure"></a>Concevoir des applications sécurisées sur Azure
Cet article présente les activités et contrôles de sécurité à envisager lorsque vous concevez des applications pour le cloud. Les ressources de formation ainsi que les questions et concepts de sécurité à examiner pendant les phases de conception et de configuration des exigences du [Microsoft Security Development Lifecycle (SDL)](/previous-versions/windows/desktop/cc307891(v=msdn.10)) y sont abordés. L’objectif est de vous aider à définir les activités et services Azure que vous pouvez utiliser pour concevoir une application plus sécurisée.

Les phases de Microsoft Security Development Lifecycle suivantes sont traitées dans cet article :

- Entrainement
- Spécifications
- Conception

## <a name="training"></a>Entrainement
Avant de commencer à développer votre application cloud, prenez le temps de comprendre la sécurité et la confidentialité dans Azure. En adoptant cette démarche, vous pouvez réduire le nombre et la gravité des vulnérabilités exploitables dans votre application. Vous serez mieux préparé pour réagir de façon appropriée dans ce paysage de menaces informatiques en perpétuelle évolution.

Utilisez les ressources suivantes pendant la phase de formation, afin de vous familiariser avec les services Azure disponibles pour les développeurs, et avec les bonnes pratiques en matière de sécurité sur Azure :

  - Le [Guide sur Azure pour les développeurs](https://azure.microsoft.com/campaigns/developer-guide/) montre comment bien démarrer avec Azure. Ce guide vous présente les services que vous pouvez utiliser pour exécuter vos applications, stocker vos données, intégrer l’intelligence, générer des applications IoT et déployer vos solutions de manière plus efficace et sécurisée.

  - Le [Guide de prise en main pour les développeurs Azure](../../guides/developer/azure-developer-guide.md) fournit des informations essentielles aux développeurs qui souhaitent commencer à utiliser la plateforme Azure pour leurs besoins de développement.

  - L’ensemble des [kits SDK et outils](../../index.yml?pivot=sdkstools) décrit les outils qui sont disponibles sur Azure.

  - La suite [Azure DevOps Services](/azure/devops/) fournit des outils collaboratifs de développement. Ces outils comportent des pipelines haute performance, des dépôts Git gratuits, des tableaux Kanban configurables et des tests de charge basés sur le cloud, automatisés et complets.
    Le [Centre de ressources DevOps](/azure/devops/learn/) combine nos ressources d’apprentissage des pratiques DevOps, le contrôle de version Git, les méthodes agiles, notre manière de travailler avec DevOps chez Microsoft et la façon dont vous pouvez évaluer la progression de vos propres opérations de développement.

  - Le [Top 5 des éléments de sécurité à prendre en compte avant d’envoyer en production](/learn/modules/top-5-security-items-to-consider/index?WT.mc_id=Learn-Blog-tajanca) vous montre comment sécuriser vos applications web sur Azure, et comment protéger vos applications contre les attaques les plus courantes et les plus dangereuses menées sur les applications web.

  - Le [Secure DevOps Kit pour Azure](https://azsk.azurewebsites.net/index.html) est une collection de scripts, d’outils, d’extensions et d’automatisations répondant aux besoins étendus en matière de sécurité des ressources et des abonnements Azure pour les équipes DevOps ayant recours à l’automatisation complète. Le Secure DevOps Kit pour Azure peut vous montrer comment intégrer en douceur la sécurité dans vos workflows DevOps natifs. Le kit aborde des outils, tels que les tests de vérification de sécurité (SVT) ; ceux-ci peuvent aider les développeurs à écrire du code sécurisé et à tester la configuration protégée de leurs applications cloud dans les phases de développement initial et de codage.

  - L’ensemble des [Bonnes pratiques de sécurité pour les solutions Azure](https://azure.microsoft.com/resources/security-best-practices-for-azure-solutions) s’utilise au fur et à mesure que vous concevez, déployez et gérez vos solutions cloud avec Azure.

## <a name="requirements"></a>Spécifications
La phase de détermination des exigences est une étape essentielle dans la définition de ce que votre application est, et de ce qu’elle fera lorsqu’elle sera publiée. La phase de configuration des exigences représente également un moment de réflexion à mener sur les contrôles de sécurité que vous allez créer dans votre application. Pendant cette phase, vous initiez également les étapes que vous allez suivre tout au long du SDL pour garantir la publication et le déploiement d’une application sécurisée.

### <a name="consider-security-and-privacy-issues"></a>Examiner les questions de confidentialité et de sécurité
Cette phase est le moment idéal pour prendre en compte les problèmes fondamentaux de la sécurité et de la confidentialité. La définition des niveaux acceptables de sécurité et de confidentialité au début d’un projet aide une équipe à :

- Comprendre les risques associés à des problèmes de sécurité.
- Identifier et résoudre les bogues de sécurité pendant le développement.
- Appliquer des niveaux établis de sécurité et de confidentialité tout au long de l’ensemble du projet.

Lorsque vous écrivez la configuration exigée pour votre application, veillez à tenir compte des contrôles de sécurité qui peuvent participer à la protection de votre application et de vos données.

### <a name="ask-security-questions"></a>Poser des questions de sécurité
Posez des questions de sécurité, telles que :

  - Mon application contient-elle des données sensibles ?

  - Mon application collecte-t-elle ou stocke-t-elle des données qui exigent que je me conforme aux normes du secteur d’activité et aux programmes de conformité, tels que le [FFIEC (Federal Financial Institution examen Council)](/previous-versions/azure/security/blueprints/ffiec-analytics-overview) ou les [normes PCI DSS (Payment Card Industry Data Security Standards)](/previous-versions/azure/security/blueprints/pcidss-analytics-overview) ?

  - Mon application collecte-t-elle ou contient-elle des données clientes ou personnelles sensibles qui peuvent être utilisées, seules ou avec d’autres informations, pour identifier, contacter ou localiser une personne ?

  - Mon application collecte-t-elle ou contient-elle des données qui peuvent être utilisées pour accéder à des renseignements médicaux, financiers ou à toute autre information relative à la carrière ou à la formation d’un individu ? L’identification de la sensibilité de vos données pendant la phase de configuration des exigences vous permet d’organiser vos données et d’identifier la méthode de protection des données que vous utiliserez pour votre application.

  - Où et comment sont stockées mes données ? Réfléchissez à la façon dont vous devez superviser les services de stockage utilisés par votre application, afin de détecter tout changement inattendu (par exemple, des temps de réponse plus longs). Serez-vous en mesure d’influencer la journalisation pour collecter des données plus détaillées et d’analyser un problème de façon approfondie ?

  - Mon application sera-t-elle disponible au public (sur internet) ou en interne uniquement ? Si votre application est disponible au public, comment protégez-vous les données qui peuvent être collectées d’être utilisées à mauvais escient ? Si votre application est disponible en interne uniquement, réfléchissez aux personnes dans votre entreprise qui doivent avoir accès à l’application, et la durée pendant laquelle elles doivent y accéder.

  - Comprenez-vous votre modèle d’identité avant de démarrer la conception de votre application ? Comment allez-vous déterminer que les utilisateurs en présence sont bien ceux qu’ils prétendent être, et ce que ces utilisateurs sont autorisés à faire ?

  - Mon application effectue-t-elle des tâches sensibles ou capitales (par exemple, le transfert d’argent, le déverrouillage de portes ou la remise de médicaments) ?
    Réfléchissez à la façon dont vous allez valider que l’utilisateur en train d’effectuer une tâche sensible est autorisé à la faire, et à la façon dont vous allez authentifier que cette personne est bien celle qu’elle prétend être. L’autorisation (AuthZ) correspond à l’action d’accorder à un principal de sécurité authentifié le droit de faire quelque chose. L’authentification (AuthN) consiste à demander des informations d’identification légitimes à une partie.

  - Mon application déploie-t-elle des activités logicielles à risque, comme permettre aux utilisateurs de charger ou de télécharger des fichiers ou d’autres données ? Si votre application développe des activités à risque, songez à la façon dont elle protégera les utilisateurs contre l’utilisation de données ou de fichiers malveillants.

### <a name="review-owasp-top-10"></a>Consulter le Top 10 OWASP
Prévoyez de passer en revue le [<span class="underline">Top 10 OWASP des risques liés à la sécurité des applications</span>](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project).
Le Top 10 OWASP traite des risques de sécurité critiques pour les applications web.
La connaissance de ces risques de sécurité peut vous aider dans vos prises de décision en matière d’exigences et de conception pour diminuer ces risques dans votre application.

Réfléchir aux contrôles de sécurité pour empêcher les violations est important.
Vous voulez néanmoins également [envisager qu’une violation de la sécurité](/azure/devops/learn/devops-at-microsoft/security-in-devops) puisse se produise. Partir du principe qu’une faille existe permet de répondre en avance à certaines questions cruciales sur la sécurité, pour ne pas avoir à y répondre dans l’urgence :

  - Comment détecter une attaque ?

  - Que faire en cas d’attaque ou de violation ?

  - Comment récupérer après une attaque, par exemple après une fuite ou une falsification de données ?

## <a name="design"></a>Conception

La phase de conception est primordiale dans l’établissement de bonnes pratiques se rapportant aux spécifications de conception et de fonctionnement. Il est également capital d’effectuer une analyse des risques, car elle permet d’atténuer les problèmes de sécurité et de confidentialité sur l’ensemble d’un projet.

Lorsque des exigences de sécurité sont en place et que vous utilisez les concepts d’une conception sécurisée, vous pouvez éviter ou réduire au minimum les possibilités d’une faille de sécurité. Une faille de sécurité est une omission, au niveau de la conception de l’application, qui peut éventuellement autoriser un utilisateur à commettre des actions malveillantes ou inattendues après la publication de votre application.

Pendant la phase de conception, pensez aussi à la façon dont vous pouvez appliquer la sécurité en couches, un niveau de défense ne suffisant pas nécessairement. Que se passe-t-il si un attaquant franchit votre pare-feu d’applications web (WAF) ? Vous voulez un autre contrôle de sécurité en place pour vous défendre contre cette attaque.

C’est en gardant cette idée à l’esprit que nous abordons les concepts de la conception sécurisée suivants, ainsi que les contrôles de sécurité dont vous devez tenir compte lorsque vous concevez des applications sécurisées :

- Utilisation d’une bibliothèque de codages sécurisés et d’un framework logiciel.
- Recherche des composants vulnérables.
- Utilisation de la modélisation des menaces lors de la conception d’une application.
- Réduction de votre surface d’attaque.
- Adoption d’une stratégie d‘identité en tant que périmètre de sécurité principal.
- Obligation d’une nouvelle authentification pour les transactions importantes.
- Utilisation d’une solution de gestion de clés pour sécuriser les clés, les informations d’identification et d’autres secrets.
- Protection des données sensibles.
- Implémentation de mesures de prévention de défaillance.
- Mise à profit de la gestion des erreurs et des exceptions.
- Utilisation de la journalisation et des alertes.

### <a name="use-a-secure-coding-library-and-a-software-framework"></a>Utilisation d’une bibliothèque de codages sécurisés et d’un framework logiciel

En ce qui concerne le développement, utilisez une bibliothèque de codages sécurisés et un framework logiciel doté d’une sécurité intégrée. Les développeurs peuvent utiliser des fonctionnalités existantes éprouvées (chiffrement, assainissement d’entrée, encodage de sortie, clés ou chaînes de connexion et tout autre élément pouvant être considéré comme contrôle de sécurité) plutôt que de développer des contrôles de sécurité à partir de zéro. Cette façon de faire permet de se prémunir contre les défauts d’implémentation et de conception liés à la sécurité.

Veillez à utiliser la toute dernière version de votre framework, ainsi que toutes les fonctionnalités de sécurité disponibles dans le framework. Microsoft propose un [ensemble complet d’outils de développement](https://azure.microsoft.com/product-categories/developer-tools/) destiné à tous les développeurs, quels que soient la plateforme ou le langage dont ils se servent, pour fournir des applications cloud. Vous pouvez développer dans le langage de votre choix en choisissant parmi différents [kits SDK](https://azure.microsoft.com/downloads/).
Vous pouvez profiter d’environnements de développement intégrés (IDE) complets et d’éditeurs équipés de fonctions de débogage avancé, ainsi que du support Azure intégré.

Microsoft offre un éventail de [langages, frameworks et outils](../../index.yml?panel=sdkstools-all&pivot=sdkstools) que vous pouvez utiliser pour développer des applications sur Azure. Par exemple, [Azure pour les développeurs .NET et .NET Core](/dotnet/azure/). À chaque langage et framework que nous proposons, correspondent des guides de démarrage rapide, des tutoriels et des informations de référence sur les API pour vous mettre rapidement le pied à l’étrier.

Azure présente toute une gamme de services à utiliser pour héberger des applications web et des sites web. Ces services vous permettent de développer dans votre langage de prédilection, que ce soit .NET, .NET Core, Java, Ruby, Node.js, PHP ou Python.
[Azure App Service Web Apps](../../app-service/overview.md) (Web Apps) est un de ces services.

Web Apps ajoute la puissance de Microsoft Azure à votre application. Il inclut la sécurité, l’équilibrage de charge, la mise à l’échelle automatique et la gestion automatisée. Vous pouvez également bénéficier des fonctionnalités DevOps dans Web Apps, comme la gestion des packages, les environnements de préproduction, les domaines personnalisés, les certificats SSL/TLS et le déploiement continu à partir d’Azure DevOps, GitHub, Docker Hub et d’autres sources.

Azure propose d’autres services que vous pouvez utiliser pour héberger des applications web et des sites web. Pour la plupart des scénarios, Web Apps est le meilleur choix. Pour une architecture de microservices, pensez à [Azure Service Fabric](https://azure.microsoft.com/documentation/services/service-fabric).
Si vous avez besoin de contrôler davantage les machines virtuelles sur lesquelles votre code s’exécute, utilisez plutôt [Azure Virtual Machines](https://azure.microsoft.com/documentation/services/virtual-machines/).
Pour plus d’informations sur le choix à opérer entre ces services Azure, consultez une [comparaison entre Azure App Service, Virtual Machines, Service Fabric et Cloud Services](/azure/architecture/guide/technology-choices/compute-decision-tree).

### <a name="apply-updates-to-components"></a>Appliquer des mises à jour aux composants

Afin d’éviter les vulnérabilités, vous devez inventorier en permanence vos composants côté client et côté serveur (par exemple, les frameworks et les bibliothèques) ainsi que leurs dépendances pour les mettre à jour. De nouvelles vulnérabilités et versions de logiciels mis à jour sont publiées sans relâche. Veillez à toujours avoir un plan vous permettant de superviser, de procéder au triage et d’appliquer mises à jour ou changements de configuration aux bibliothèques et aux composants que vous utilisez.

Consultez la page [OWASP (Open Web Application Security Project )](https://www.owasp.org/index.php/Main_Page) sur l’[utilisation de composants avec des vulnérabilités connues](https://www.owasp.org/index.php/Top_10-2017_A9-Using_Components_with_Known_Vulnerabilities) pour obtenir des suggestions d’outils. Vous pouvez également vous abonner aux alertes par e-mail pour les vulnérabilités de sécurité qui sont liées aux composants que vous utilisez.

### <a name="use-threat-modeling-during-application-design"></a>Utiliser la modélisation des menaces lors de la conception d’une application

La modélisation des menaces est le processus consistant à identifier les menaces de sécurité potentielles pour votre entreprise et votre application, puis à vérifier que les atténuations appropriées sont en place. Le SDL précise que les équipes doivent s’impliquer dans la modélisation des menaces au cours de la phase de conception, lorsque la résolution de problèmes potentiels est relativement facile et économique. L’utilisation de la modélisation des menaces dans la phase de conception peut réduire considérablement le coût total de développement.

Pour faciliter le processus de modélisation des menaces, nous avons conçu l’[Outil SDL de modélisation des menaces](threat-modeling-tool.md) en pensant à ceux qui ne sont pas experts en sécurité. Cet outil simplifie la modélisation des menaces pour tous les développeurs, en fournissant des conseils clairs sur la façon de créer et d’analyser les modèles de menace.

La modélisation de la conception de l’application et l’énumération des menaces [STRIDE](https://docs.google.com/viewer?a=v&pid=sites&srcid=ZGVmYXVsdGRvbWFpbnxzZWN1cmVwcm9ncmFtbWluZ3xneDo0MTY1MmM0ZDI0ZjQ4ZDMy) (usurpation d’identité, falsification, répudiation, divulgation d’informations, déni de service et élévation de privilège) sur toutes les limites d’approbation s’avèrent un moyen efficace pour comprendre dès le début les erreurs de conception. Le tableau suivant liste les menaces STRIDE et donne des exemples d’atténuation des risques qu’utilisent les fonctionnalités fournies par Azure. Ces atténuations ne fonctionnent pas dans tous les cas.

| Menace | Propriété de sécurité | Atténuation des risques potentiels sur la plateforme Azure |
| ---------------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Usurpation d’identité               | Authentification        | [Exiger des connexions HTTPS](/aspnet/core/security/enforcing-ssl?tabs=visual-studio&view=aspnetcore-2.1). |
| Falsification              | Intégrité             | Valider des certificats SSL/TLS. Les applications qui utilisent les protocoles SSL/TLS doivent intégralement vérifier les certificats X.509 des entités auxquelles elles se connectent. Utiliser des certificats Azure Key Vault pour [gérer vos certificats x509](../../key-vault/general/about-keys-secrets-certificates.md). |
| Répudiation            | Non-répudiation       | Activez [la surveillance et les diagnostics Azure](/azure/architecture/best-practices/monitoring).|
| Divulgation d’informations | Confidentialité       | Chiffrer les données sensibles [au repos](../fundamentals/encryption-atrest.md) et [en transit](../fundamentals/data-encryption-best-practices.md#protect-data-in-transit). |
| Déni de service      | Disponibilité          | Superviser les métriques de performances pour des conditions potentielles de déni de service. Implémentez des filtres de connexion. Combiné aux bonnes pratiques de la conception d’application, le [service de protection DDoS Azure](../../ddos-protection/ddos-protection-overview.md#next-steps) assure une excellente protection contre les attaques DDoS.|
| Élévation de privilège | Autorisation         | Utiliser Azure Active Directory <span class="underline"> </span> [Privileged Identity Management](../../active-directory/privileged-identity-management/pim-configure.md).|

### <a name="reduce-your-attack-surface"></a>Réduction de votre surface d’attaque

La somme totale des emplacements où des vulnérabilités potentielles peuvent se produire constitue une surface d’attaque. Dans ce document, nous allons nous concentrer sur la surface d’attaque d’une application.
L’accent est mis sur la protection d’une application contre l’attaque. Un moyen simple et rapide de réduire votre surface d’attaque consiste à supprimer ressources et code inutilisés de votre application. Plus votre application est petite, plus sa surface d’attaque est petite. Par exemple, supprimez :

- Le code des fonctionnalités que vous n’avez pas encore publiées.
- Le code de prise en charge du débogage.
- Les interfaces et protocoles réseau qui ne sont pas utilisés ou qui ont été dépréciés.
- Les machines virtuelles et d’autres ressources que vous n’utilisez pas.

Le nettoyage régulier de vos ressources et la vérification que tout code inutilisé est supprimé sont deux excellents moyens de veiller à ce que les opportunités d’attaque par des acteurs malveillants soient moins nombreuses.

Une façon tout aussi détaillée qu’approfondie de réduire la surface d’attaque consiste à effectuer une analyse de la surface d’attaque. Une analyse de surface d’attaque vous permet de mapper les parties d’un système qui doivent être examinées et testées à la recherche de vulnérabilités de sécurité.

La finalité d’une analyse de surface d’attaque se résume à la compréhension des zones de risque existant dans une application, afin que les développeurs et les spécialistes de la sécurité sachent quelles parties de l’application sont ouvertes aux attaques. Il suffit ensuite de trouver des moyens de limiter ce potentiel, de suivre les modifications de la surface d’attaque (quand et comment elles sont effectuées) et de savoir ce que cela signifie en termes de risque.

Une analyse de surface d’attaque permet d’identifier :

- Les fonctions et parties du système que vous devez passer en revue et tester à la recherche de failles de sécurité.
- Les zones de code à haut risque qui nécessitent une protection de défense en profondeur (les parties du système que vous devez défendre).
- Lorsque vous modifiez la surface d’attaque et que vous avez besoin d’actualiser une évaluation des menaces.

Pour réduire les opportunités qu’un point faible potentiel ou qu’une vulnérabilité soient exploités par des attaquants, vous devez analyser de manière approfondie l’ensemble de la surface d’attaque de votre application. Doivent être également inclus la désactivation ou la restriction de l’accès aux services système, l’application du principe du privilège minimum et l’emploi de défenses en couches, autant que possible.

Nous abordons [la conduite d’une analyse de surface d’attaque](secure-develop.md#conduct-attack-surface-review) pendant la phase de vérification du SDL.

> [!NOTE]
> **Quelle différence y a-t-il entre la modélisation des menaces et l’analyse de la surface d’attaque ?**
La modélisation des menaces consiste à identifier les menaces de sécurité potentielles de votre application, et de vérifier ensuite que les atténuations appropriées contre les menaces sont en place. L’analyse de la surface d’attaque identifie les zones à haut risque du code qui sont ouvertes aux attaques. Elle implique de trouver des moyens de protéger les zones à haut risque de votre application, puis de passer en revue et de tester ces zones de code avant de déployer l’application.

### <a name="adopt-a-policy-of-identity-as-the-primary-security-perimeter"></a>Adopter une stratégie d‘identité en tant que périmètre de sécurité principal

Lorsque vous concevez des applications cloud, il est important d’élargir votre focalisation du périmètre de sécurité en partant d’une approche centrée sur le réseau pour vous orienter vers une approche axée sur l’identité. Historiquement, le périmètre principal de la sécurité locale était le réseau d’entreprise. La plupart des conceptions de sécurité locale utilisent le réseau comme pivot principal de la sécurité. Pour les applications cloud, il est préférable de considérer l’identité comme périmètre principal de la sécurité.

Voici les choses que vous pouvez faire pour développer une approche centrée sur l’identité dans le développement d’applications web :

- Mettre en application l’authentification multifacteur pour les utilisateurs.
- utilisez une authentification forte et des plateformes d’autorisation.
- Appliquer le principe du privilège minimum.
- Implémenter l’accès juste-à-temps.

#### <a name="enforce-multi-factor-authentication-for-users"></a>Mettre en application l’authentification multifacteur pour les utilisateurs

Utilisez une authentification à deux facteurs. L’authentification à deux facteurs représente la norme actuelle pour l’authentification et l’autorisation, car elle évite les failles de sécurité qui sont inhérentes aux types d’authentification par nom d’utilisateur et mot de passe. Les accès aux interfaces de gestion Azure (portail Azure/PowerShell à distance) et aux services destinés aux clients doivent être conçus et configurés pour utiliser [Azure multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md).

#### <a name="use-strong-authentication-and-authorization-platforms"></a>Utiliser une authentification forte et des plateformes d’autorisation

Utilisez les mécanismes d‘authentification et d‘autorisation fournis par les plateformes au lieu d‘un code personnalisé. En effet, le développement d’un code d’authentification personnalisé peut être sujet aux erreurs. Le code commercial (par exemple, de Microsoft) est souvent très contrôlé au niveau de la sécurité. [Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md) est la solution Azure pour la gestion des identités et des accès. Ces services et outils Azure AD contribuent au développement sécurisé :

- La [Plateforme d’identités Microsoft](../../active-directory/develop/index.yml) est un ensemble de composants utilisé par les développeurs pour créer des applications qui connectent les utilisateurs de manière sécurisée. Elle aide les développeurs qui créent des applications métier monolocataires et ceux qui cherchent à développer des applications multilocataires. En plus de la connexion de base, les applications créées à l’aide de la Plateforme d’identités Microsoft peuvent appeler des API Microsoft et des API personnalisées. La plateforme d’identités Microsoft prend en charge les protocoles standard, tels qu’OAuth 2.0 et OpenID Connect.

- [Azure Active Directory B2C (Azure AD B2C)](../../active-directory-b2c/index.yml) est un service de gestion des identités que vous pouvez utiliser pour personnaliser et contrôler la façon dont les clients s’inscrivent, se connectent et gèrent leurs profils lorsqu’ils utilisent vos applications. Sont notamment incluses les applications développées pour iOS, Android et .NET. Azure AD B2C autorise ces actions tout en protégeant les identités de vos clients.

#### <a name="apply-the-principle-of-least-privilege"></a>Appliquer le principe du privilège minimum

Le concept du [privilège minimum](https://en.wikipedia.org/wiki/Principle_of_least_privilege) implique de donner aux utilisateurs le niveau précis d’accès et de contrôle dont ils ont besoin pour effectuer leurs tâches, et rien de plus.

Un développeur de logiciels a-t-il besoin de droits d’administrateur de domaine ? Un assistant administratif doit-il avoir accès aux contrôles d’administration sur son ordinateur personnel ? L’évaluation de l’accès aux logiciels n’est pas différent. Si vous utilisez un [contrôle d’accès en fonction du rôle Azure (RBAC Azure)](../../role-based-access-control/overview.md) pour donner une autorité et des capacités différentes aux utilisateurs dans votre application, vous ne donnez pas à tous l’accès à tout. En limitant l’accès à ce qui est nécessaire pour chaque rôle, vous limitez le risque qu’un problème de sécurité se produise.

Assurez-vous que votre application suit le principe du [privilège minimum](/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models#in-applications) tout au long de ses modèles d’accès.

> [!NOTE]
> Les règles du privilège minimum doivent s’appliquer au logiciel et aux personnes créant le logiciel. Les développeurs de logiciels peuvent représenter un risque considérable pour la sécurité informatique s’ils ont trop d’accès. Si un développeur est animé d’intentions malveillantes, ou qu’il bénéficie de trop d’accès, les conséquences peuvent être graves. Nous recommandons que les règles du privilège minimum soient appliquées aux développeurs tout au long du cycle de vie de développement.

#### <a name="implement-just-in-time-access"></a>Implémenter l’accès juste-à-temps

Implémentez l’accès *juste-à-temps* (JIT) pour réduire davantage le temps d’exposition des privilèges. Utilisez [Azure AD Privileged Identity Management](../../active-directory/roles/security-planning.md#stage-3-take-control-of-admin-activity) pour :

- Donner aux utilisateurs les autorisations dont ils ont besoin uniquement juste-à-temps.
- Attribuer des rôles pour une durée plus courte en sachant que les privilèges sont automatiquement révoqués.

### <a name="require-re-authentication-for-important-transactions"></a>Obligation d’une nouvelle authentification pour les transactions importantes

Une [falsification de requête intersites](/aspnet/core/security/anti-request-forgery?view=aspnetcore-2.1) (également connue sous le nom de *XSRF* ou *CSRF*) est une attaque contre des applications hébergées sur le web, dans lesquelles un site web malveillant influence l’interaction entre un navigateur client et une application web qui fait confiance à ce navigateur. Ces attaques de falsification de requête intersites sont possibles, car les navigateurs web envoient automatiquement certains types de jetons d’authentification avec chaque requête vers un site web.
Cette forme d’exploitation est également appelée *one-click attack* (attaque en un clic) ou *session riding* (détournement de session) du fait que l’attaque profite de la session précédemment authentifiée de l’utilisateur.

La meilleure façon de se défendre contre ce genre d’attaque revient à demander à l’utilisateur quelque chose que lui seul peut fournir avant chaque transaction importante, comme un achat, une désactivation de compte ou une modification de mot de passe. Vous pouvez demander à l’utilisateur d’entrer de nouveau son mot de passe, d’effectuer un test captcha ou d’envoyer un jeton secret dont il est l’unique détenteur. La méthode la plus courante est le jeton secret.

### <a name="use-a-key-management-solution-to-secure-keys-credentials-and-other-secrets"></a>Utilisation d’une solution de gestion de clés pour sécuriser les clés, les informations d’identification et d’autres secrets

La perte de clés ou d'informations d’identification est un problème courant. la seule chose qui est pire que la perte de vos clés et informations d’identification serait qu’un tiers non autorisé y accède. Les attaquants peuvent exploiter des techniques automatisées et manuelles pour trouver des clés et des secrets qui sont stockés dans des dépôts de code, comme GitHub. Ne placez pas de clés ni de secrets dans ces dépôts de code public, ou sur tout autre serveur.

Placez toujours vos clés, certificats, secrets et chaînes de connexion dans une solution de gestion de clés. Vous pouvez utiliser une solution centralisée, dans laquelle clés et secrets sont stockés dans des modules de sécurité matériels (HSM). Azure vous fournit un module HSM dans le cloud avec [Azure Key Vault](../../key-vault/general/overview.md).

Key Vault est un *magasin des secrets* : c’est un service cloud centralisé pour stocker des secrets d’application. Key Vault préserve la sécurité de vos données confidentielles en conservant les secrets de l’application dans un emplacement central unique et en fournissant l’accès sécurisé, le contrôle des autorisations et la journalisation des accès.

Les secrets sont stockés dans des *coffres* individuels. Chaque coffre a sa propre configuration et ses stratégies de sécurité pour en contrôler l’accès. Vous accédez à vos données par le biais d’une API REST, ou d’un kit SDK client disponible pour la plupart des langages de programmation.

> [!IMPORTANT]
> Azure Key Vault est conçu pour stocker les secrets de configuration des applications serveur. Il n’est pas destiné au stockage des données appartenant aux utilisateurs d’applications. Ses caractéristiques de performances, API et modèle de coût reflètent cette condition.
>
> Les données utilisateur doivent être stockées ailleurs, par exemple dans une instance Azure SQL Database dotée de la technologie TDE (Transparent Data Encryption), ou dans un compte de stockage qui utilise Azure Storage Service Encryption. Les secrets utilisés par votre application pour accéder à ces magasins de données peuvent être conservés dans Azure Key Vault.

### <a name="protect-sensitive-data"></a>Protection des données sensibles

La protection des données représente une partie primordiale de votre stratégie de sécurité.
Classer vos données et identifier vos besoins en matière de protection de données vous permet de concevoir votre application en gardant à l’esprit la sécurité des données. Classer (catégoriser) les données stockées par niveau de sensibilité et d’impact sur l’activité aide les développeurs à déterminer les risques qui sont associés aux données.

Étiquetez comme sensibles toutes les données concernées lorsque vous concevez vos formats de données. Assurez-vous que l’application traite les données concernées comme sensibles. Ces pratiques peuvent vous aider à protéger vos données sensibles :

- Utiliser le chiffrement.
- Éviter de coder de manière irréversible des secrets, tels que des clés et des mots de passe.
- S’assurer que les contrôles d’accès et l’audit sont en place.

#### <a name="use-encryption"></a>Utiliser le chiffrement

La protection des données doit constituer une partie essentielle de votre stratégie de sécurité.
Si vos données sont stockées dans une base de données, ou si elles se déplacent dans les deux sens entre des emplacements, utilisez le chiffrement des [données au repos](../fundamentals/encryption-atrest.md) (lorsqu’elles sont dans la base de données) et le chiffrement des [données en transit](../fundamentals/data-encryption-best-practices.md#protect-data-in-transit) (en cours d’acheminement vers et à partir de l’utilisateur, la base de données, une API ou un point de terminaison de service). Nous vous conseillons de toujours utiliser les protocoles SSL/TLS pour échanger des données. Vérifiez que vous utilisez la dernière version en date de TLS pour le chiffrement (actuellement, il s’agit de la version 1.2).

#### <a name="avoid-hard-coding"></a>Éviter le codage effectué de manière irréversible

Certaines choses ne devraient jamais être codées de manière irréversible dans votre logiciel. Par exemple, les noms d’hôte ou les adresses IP, les URL, adresses e-mail, noms d’utilisateur, mots de passe, clés de compte de stockage et d’autres clés de chiffrement. Songez à implémenter des exigences par rapport à ce qui peut ou ne peut pas être codé de manière irréversible dans votre code, y compris dans les sections de commentaire de votre code.

Quand vous insérez des commentaires dans votre code, assurez-vous que vous n’enregistrez aucune information sensible. Par exemple, votre adresse e-mail, des mots de passe ou des chaînes de connexion, des informations sur votre application qui ne seraient connues que d’une seule personne dans votre organisation, et tout autre élément dont pourrait tirer profit un pirate pour attaquer votre application ou votre organisation.

En fait, partez du principe que tous les éléments de votre projet de développement seront rendus publics lorsqu’il sera déployé. Évitez d’inclure des données sensibles de quelque type que ce soit dans le projet.

Précédemment, nous avons abordé [Azure Key Vault](../../key-vault/general/overview.md). Vous pouvez vous servir de Key Vault pour stocker des secrets, tels que des clés et des mots de passe, au lieu de les coder de manière irréversible. Lorsque vous utilisez Key Vault en association avec des identités managées pour les ressources Azure, votre application web Azure peut accéder aux valeurs de configuration secrètes facilement et de façon sécurisée, sans stocker de secrets dans votre configuration ou votre contrôle de code source. Pour en savoir plus, voir [Gérer des secrets dans vos applications serveur avec Azure Key Vault](/learn/modules/manage-secrets-with-azure-key-vault/).

### <a name="implement-fail-safe-measures"></a>Implémentation de mesures de prévention de défaillance

Votre application doit être en mesure de gérer de manière systématique les [erreurs](/dotnet/standard/exceptions/) qui se produisent pendant l’exécution. L’application doit intercepter toutes les erreurs, et soit être fiable ou se fermer.

Vous devez également vous assurer que les erreurs sont enregistrées avec un contexte utilisateur suffisant pour identifier toute activité suspecte ou malveillante. Les journaux doivent être conservés pendant une durée suffisante pour permettre l’analyse d’investigation différée. Les journaux doivent se présenter dans un format facilement consommable par une solution de gestion des journaux. Assurez-vous que les alertes des erreurs qui sont liées à la sécurité sont déclenchées. Une journalisation et une surveillance insuffisantes permettent aux attaquants de lancer de nouvelles attaques sur les systèmes et de maintenir une persistance.

### <a name="take-advantage-of-error-and-exception-handling"></a>Mise à profit de la gestion des erreurs et des exceptions

L’implémentation de la correction des erreurs et de la [gestion des exceptions](/dotnet/standard/exceptions/best-practices-for-exceptions) représentent une partie importante du codage défensif. La gestion des exceptions et des erreurs est cruciale pour fiabiliser et sécuriser un système. Des fautes commises dans la gestion des erreurs peuvent entraîner différents types de failles de sécurité, comme la fuite d’informations vers des personnes malveillantes ou la possibilité donnée aux pirates de mieux comprendre votre plateforme et sa conception.

Assurez-vous que :

- Vous gérez les exceptions de manière centralisée, afin d’éviter des [blocs try/catch](/dotnet/standard/exceptions/how-to-use-the-try-catch-block-to-catch-exceptions) dupliqués dans le code.

- Tous les comportements inattendus sont gérés à l’intérieur de l’application.

- Les messages qui sont présentés aux utilisateurs ne divulguent aucune donnée critique, mais fournissent suffisamment d’informations pour expliquer le problème.

- Les exceptions sont enregistrées et qu’elles fournissent suffisamment d’informations aux équipes de forensique ou de réponse aux incidents pour investiguer.

[Azure Logic Apps](../../logic-apps/logic-apps-overview.md) fournit une expérience hors pair pour la [gestion des erreurs et des exceptions](../../logic-apps/logic-apps-exception-handling.md) qui sont provoqués par des systèmes dépendants. Vous pouvez utiliser Logic Apps, pour créer des workflows et automatiser tâches et processus qui s’intègrent à des applications, des données, des systèmes et des services, dans les entreprises et les organisations.

### <a name="use-logging-and-alerting"></a>Utiliser la journalisation et les alertes

[Enregistrez dans un journal](/aspnet/core/fundamentals/logging/?view=aspnetcore-2.1) vos problèmes de sécurité pour les besoins des enquêtes de sécurité, et déclenchez des alertes par rapport à ces problèmes pour que les personnes aient bien connaissance à temps de ces problèmes. Activez l’audit et la journalisation sur tous les composants. Les journaux d’audit doivent capturer le contexte de l’utilisateur et identifier tous les événements importants.

Vérifiez que vous n’enregistrez aucune donnée sensible fournie par un utilisateur sur votre site. Voici quelques exemples de données sensibles :

- Informations d’identification de l’utilisateur
- Numéros de sécurité sociale ou autres informations d’identification
- Numéros de carte de crédit ou autres informations financières
- Informations de santé
- Clés privées ou autres données pouvant être utilisées pour déchiffrer des informations chiffrées
- Informations système ou d’application pouvant être utilisées pour attaquer plus efficacement l’application

Vérifiez que l’application supervise les événements de la gestion des utilisateurs, tels que les connexions utilisateur ayant réussi ou échoué, les réinitialisations de mot de passe, les modifications de mot de passe, le verrouillage de compte et l’inscription utilisateur. L’enregistrement de ces événements dans des journaux vous permet de détecter un comportement potentiellement suspect et de réagir. Cela vous permet également de recueillir des données d’exploitation /d’opérations, par exemple savoir qui accède à l’application.

## <a name="next-steps"></a>Étapes suivantes
Dans les articles suivants, nous recommandons des contrôles et des activités de sécurité qui peuvent vous aider à développer et déployer des applications sécurisées.

- [Développer des applications sécurisées](secure-develop.md)
- [Déployer des applications sécurisées](secure-deploy.md)