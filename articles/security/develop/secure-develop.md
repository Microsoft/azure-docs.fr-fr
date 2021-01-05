---
title: Développer des applications sécurisées sur Microsoft Azure
description: Cet article présente les meilleures pratiques à prendre en compte pendant les phases d’implémentation et de vérification de votre projet d’application web.
author: TerryLanfear
manager: barbkess
ms.author: terrylan
ms.date: 06/12/2019
ms.topic: article
ms.service: security
ms.subservice: security-develop
services: azure
ms.assetid: 521180dc-2cc9-43f1-ae87-2701de7ca6b8
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
ms.openlocfilehash: 421fb7b0c91171756f55ad25c918955870054e3e
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97511278"
---
# <a name="develop-secure-applications-on-azure"></a>Développer des applications sécurisées sur Azure
Cet article présente les activités et contrôles de sécurité à prendre en compte lorsque vous développez des applications pour le cloud. Les questions et concepts de sécurité à prendre en compte pendant les phases d’implémentation et de vérification du [Microsoft Security Development Lifecycle](/previous-versions/windows/desktop/cc307891(v=msdn.10)) y sont abordées. L’objectif est de vous aider à définir les activités et services Azure que vous pouvez utiliser pour développer une application plus sécurisée.

Les phases de Microsoft Security Development Lifecycle suivantes sont traitées dans cet article :

- Implémentation
- Vérification

## <a name="implementation"></a>Implémentation
L’objectif de la phase d’implémentation est d’établir les meilleures pratiques en matière de prévention anticipée et de détecter et supprimer les problèmes de sécurité du code.
Supposons que votre application soit utilisée d’une manière que vous ne souhaitiez pas. Cela permet de se prémunir contre une mauvaise utilisation accidentelle ou intentionnelle de votre application.

### <a name="perform-code-reviews"></a>Effectuer des révisions de code

Avant d’archiver du code, [révisez-le](/azure/devops/learn/devops-at-microsoft/code-reviews-not-primarily-finding-bugs) pour augmenter la qualité globale du code et réduire le risque de création de bogues. Vous pouvez utiliser [Visual Studio](/azure/devops/repos/tfvc/get-code-reviewed-vs?view=vsts) pour gérer le processus de révision de code.

### <a name="perform-static-code-analysis"></a>Effectuer une analyse du code statique

L’[analyse du code statique](https://owasp.org/www-community/controls/Static_Code_Analysis) (également appelée *analyse du code source*) est généralement effectuée dans le cadre d’une révision du code. L’analyse du code statique consiste à exécuter des outils d’analyse du code statique pour rechercher les vulnérabilités potentielles dans le code qui n’est pas en cours d’exécution à l’aide de techniques, telles que la [vérification de teinte](https://en.wikipedia.org/wiki/Taint_checking) et l’[analyse du flux de données](https://en.wikipedia.org/wiki/Data-flow_analysis).

Place de marché Azure propose des [outils de développement](https://azuremarketplace.microsoft.com/marketplace/apps/category/developer-tools?page=1&search=code%20review) qui permettent d’effectuer une analyse du code statique et de réviser le code.

### <a name="validate-and-sanitize-every-input-for-your-application"></a>Valider et nettoyer chaque entrée de votre application

Traitez toutes les entrées comme si elles n’étaient pas fiables pour protéger votre application des vulnérabilités d’applications web les plus courantes. Les données non fiables véhiculent les attaques par injection. Parmi les entrées de votre application figurent les paramètres d’URL, les saisies de l’utilisateur, les données issues de la base de données ou d’une API, et tout élément transmis qu’un utilisateur pourrait potentiellement manipuler. Une application doit [confirmer](https://owasp.org/www-project-proactive-controls/v3/en/c5-validate-inputs) la validité des données d’un point de vue syntaxique et sémantique avant que l’application ne les utilise de quelque manière que ce soit (y compris pour les réafficher pour l’utilisateur).

Validez les entrées au début du flux de données pour vous assurer que seules des données correctement formatées pénètre le flux de travail. Vous ne voulez pas que des données incorrectes demeurent dans votre base de données ou déclenchent une défaillance dans un composant en aval.

La mise en liste rouge et en mise en liste verte sont deux approches générales de la validation de la syntaxe d’entrée :

  - La mise en liste rouge tente de vérifier qu’une entrée utilisateur donnée ne contient pas de contenu « réputé malveillant ».

  - La mise en liste verte tente de vérifier qu’une entrée utilisateur donnée correspond à un ensemble d’entrées « réputées bonnes ». La mise en liste verte basée sur des caractères est une forme de mise en liste verte où une application vérifie que la saisie de l’utilisateur ne contient que des caractères « réputés bons » ou correspond à un format connu.
    Par exemple, cela peut impliquer la vérification qu’un nom d’utilisateur contient uniquement des caractères alphanumériques ou qu’il contient exactement deux chiffres.

La mise en liste verte est la meilleure approche en matière de création de logiciels sécurisés.
La mise en liste rouge est sujette à erreur, car il est impossible d’établir une liste exhaustive des entrées potentiellement incorrectes.

Faites ce travail sur le serveur, et non côté client (ou sur le serveur et côté client).

### <a name="verify-your-applications-outputs"></a>Vérifiez les sorties de votre application

Toute sortie présentée visuellement ou au sein d’un document doit toujours être encodée et placée dans une séquence d’échappement. L’[échappement](https://owasp.org/www-community/Injection_Theory#Escaping_.28aka_Output_Encoding.29), également appelé *encodage de sortie*, permet de garantir que les données non fiables ne véhiculent pas d’attaque par injection de code. L’échappement, associé à la validation des données, offre des défenses multiniveau pour améliorer la sécurité du système dans son ensemble.

L’échappement garantit que tout est affiché sous forme de *sortie.* L’échappement informe également l’interpréteur que les données ne sont pas destinées à être exécutées pour éviter l’exécution des attaques. Il s’agit d’une autre technique d’attaque courants appelée *script intersites* (XSS).

Si vous utilisez une infrastructure web tierce, vous pouvez vérifier vos options pour l’encodage de sortie sur les sites Web à l’aide de l’[aide-mémoire de prévention OWASP XSS](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.md).

### <a name="use-parameterized-queries-when-you-contact-the-database"></a>Utiliser des requêtes paramétrables lorsque vous contactez la base de données

Ne créez jamais de requête de base de données en ligne « à la volée » dans votre code et envoyez-la directement à la base de données. Du code malveillant inséré dans votre application pourrait éventuellement provoquer le vol, la réinitialisation ou la modification de votre base de données. Votre application pourrait également être utilisée pour exécuter des commandes de système d’exploitation malveillantes sur le système d’exploitation hébergeant votre base de données.

Pour éviter cela, utilisez des requêtes paramétrables ou des procédures stockées. Lorsque vous utilisez des requêtes paramétrables, vous pouvez appeler la procédure à partir de votre code en toute sécurité et lui transmettre une chaîne sans vous préoccuper de savoir si elle sera traitée dans le cadre de l’instruction de la requête.

### <a name="remove-standard-server-headers"></a>Supprimer des en-têtes de serveur standard

Les en-têtes tels que Server, X-Powered-By et X-AspNet-Version affichent des informations relatives au serveur et aux technologies sous-jacentes. Nous vous recommandons de supprimer ces en-têtes afin d’éviter la prise d’empreinte de l’application.
Consultez [Removing standard server headers on Windows Azure Web Sites (Supprimer les en-têtes de serveur standard des sites web Azure)](https://azure.microsoft.com/blog/removing-standard-server-headers-on-windows-azure-web-sites/).

### <a name="segregate-your-production-data"></a>Séparer les données de production

Vos données de production, ou données « réelles », ne doivent pas servir au développement, au test ou à d’autres fins que celles liées à votre activité. Un jeu de données masqué ([anonyme](https://en.wikipedia.org/wiki/Data_anonymization)) doit être utilisé pour tout développement et test.

En d’autres termes, moins de personnes ont accès à vos données réelles, plus la surface d’attaque se trouve réduite. Cela signifie également qu’un nombre réduit d’employés a accès aux données personnelles, ce qui élimine une violation potentielle de la confidentialité.

### <a name="implement-a-strong-password-policy"></a>Implémenter une stratégie de mot de passe forte

Pour se défendre contre les attaques en force brute et les vols basés sur le dictionnaire, vous devez implémenter une stratégie de mot de passe fort pour garantir que les utilisateurs créent des mots de passe complexes (par exemple, longueur minimale de 12 caractères, caractères spéciaux et alphanumériques).

Vous pouvez utiliser une infrastructure d’identité pour créer et appliquer des stratégies de mot de passe. Azure AD B2C permet de gérer les mots de passe en proposant des [stratégies intégrées](../../active-directory-b2c/tutorial-create-user-flows.md#create-a-password-reset-user-flow), la [réinitialisation de mot de passe en libre-service](../../active-directory-b2c/user-flow-self-service-password-reset.md) et bien plus encore.

Pour se défendre contre les attaques sur les comptes par défaut, vérifiez que toutes les clés et que tous les mots de passe sont remplaçables et qu’ils sont générés ou remplacés après l’installation des ressources.

Si l’application doit générer automatiquement des mots de passe, assurez-vous que les mots de passe générés sont aléatoires et ont une entropie élevée.

### <a name="validate-file-uploads"></a>Valider des chargements de fichiers

Si votre application autorise les [chargements de fichiers](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload), envisagez les précautions que vous pouvez prendre pour cette activité à risque. La première étape de nombreuses attaques consiste à obtenir un code malveillant dans un système subissant une attaque. Un chargement de fichier permet aux personnes malveillantes d’accomplir cette étape. OWASP propose des solutions permettant de valider un fichier afin de garantir que le fichier chargé est sécurisé.

Une solution de protection contre les programmes malveillants permet d’identifier et de supprimer les virus, logiciels espions et autres logiciels malveillants. Vous pouvez installer [Microsoft Antimalware](../fundamentals/antimalware.md) ou une solution de protection des points de terminaison d’un partenaire de Microsoft ([Trend Micro](https://www.trendmicro.com/azure/), [Broadcom](https://www.broadcom.com/products), [McAfee](https://www.mcafee.com/us/products.aspx), [Windows Defender](/windows/security/threat-protection/windows-defender-antivirus/windows-defender-antivirus-in-windows-10) ou [Endpoint Protection](/configmgr/protect/deploy-use/endpoint-protection)).

Le logiciel [Microsoft Antimalware](../fundamentals/antimalware.md) inclut des fonctionnalités telles que la protection en temps réel, l’analyse planifiée, la correction des logiciels malveillants, la mise à jour des signatures, la mise à jour des moteurs, des exemples de création de rapport et la collecte d’événements d’exclusion. Vous pouvez intégrer Microsoft Antimalware et des solutions de partenaires avec [Azure Security Center](../../security-center/security-center-partner-integration.md) pour bénéficier d’un déploiement simplifié et de fonctionnalités de détection intégrées (alertes et incidents).

### <a name="dont-cache-sensitive-content"></a>Ne pas mettre en cache de contenu sensible

Ne mettez pas en cache de contenu sensible sur le navigateur. Les navigateurs peuvent stocker des informations à des fins de mise en cache et d’historique. Les fichiers mis en cache sont stockés dans un dossier, comme le dossier Fichiers Internet temporaires d’Internet Explorer. Lorsque ces pages sont de nouveau consultées, le navigateur les affiche à partir du cache. Si l’utilisateur peut voir des informations sensibles (par exemple, adresse, numéro de carte de crédit, numéro de sécurité sociale ou nom d’utilisateur), ces informations sont peut-être stockées dans le cache du navigateur et elles peuvent par conséquent être récupérées lors de l’examen du cache du navigateur ou en appuyant simplement sur le bouton **Précédent** du navigateur.

## <a name="verification"></a>Vérification
La phase de vérification implique un effort complet pour s’assurer que le code respecte les principes de sécurité et confidentialité établis au cours des phases précédentes.

### <a name="find-and-fix-vulnerabilities-in-your-application-dependencies"></a>Rechercher et corriger les vulnérabilités de vos dépendances d’application

Vous analysez votre application et ses bibliothèques dépendantes pour identifier tous les composants vulnérables connus. Les produits disponibles pour effectuer cette analyse sont les suivants : [OWASP Dependency Check](https://www.owasp.org/index.php/OWASP_Dependency_Check), [Snyk](https://snyk.io/), et [Black Duck](https://www.blackducksoftware.com/).

L’analyse des vulnérabilités s’appuyant sur [Tinfoil Security](https://www.tinfoilsecurity.com/) est disponible pour Azure App Service Web Apps. [L’analyse Tinfoil Security via App Service](https://azure.microsoft.com/blog/web-vulnerability-scanning-for-azure-app-service-powered-by-tinfoil-security/) offre aux développeurs et administrateurs un moyen rapide, économique et intégré de détecter et corriger les vulnérabilités avant qu’un acteur malveillant puisse en tirer parti.

> [!NOTE]
> Vous pouvez également [intégrer Tinfoil Security à Azure AD](../../active-directory/saas-apps/tinfoil-security-tutorial.md). L’intégration de Tinfoil Security à Azure AD vous fait bénéficier des avantages suivants :
>  - Dans Azure AD, vous pouvez contrôler qui a accès à Tinfoil Security.
>  - Vos utilisateurs peuvent se connecter automatiquement à Tinfoil Security (par le biais de l’authentification unique) avec leur compte Azure AD.
>  - Vous pouvez centraliser la gestion de vos comptes à un seul emplacement : le Portail Azure.

### <a name="test-your-application-in-an-operating-state"></a>Tester votre application en fonctionnement

Le test de sécurité des applications dynamique (DAST) est un processus de test d’une application en fonctionnement permettant de rechercher les vulnérabilités de sécurité. Les outils DAST analysent les programmes en cours d’exécution à la recherche de vulnérabilités de sécurité, telles que l’altération de la mémoire, une configuration de serveur non sécurisée, les scripts intersites, les problèmes liés au privilège utilisateur, l’injection de code SQL et d’autres problèmes de sécurité critiques.

Les tests DAST sont différents des tests de sécurité d’application statiques (SAST). Les outils SAST analysent le code source ou les versions compilées de code lorsque le code n’est pas exécuté afin de détecter des failles de sécurité.

Effectuez les tests DAST de préférence avec l’aide d’un professionnel de la sécurité (un [testeur d’intrusion](../fundamentals/pen-testing.md) ou un évaluateur des vulnérabilités). Si aucun professionnel de la sécurité n’est disponible, vous pouvez effectuer vous-même les tests DAST avec un scanneur de proxy web et une formation préalable. Connectez un scanneur DAST suffisamment tôt afin de vous assurer de ne pas introduire de problèmes de sécurité évidents dans votre code. Consultez le site [OWASP](https://owasp.org/www-community/Vulnerability_Scanning_Tools) pour obtenir la liste des scanneurs de vulnérabilité d’application web.

### <a name="perform-fuzz-testing"></a>Effectuer un test à données aléatoires (fuzzing)

Au cours du [test à données aléatoires (fuzzing)](https://cloudblogs.microsoft.com/microsoftsecure/2007/09/20/fuzz-testing-at-microsoft-and-the-triage-process/), vous introduisez délibérément des données incorrectes ou aléatoires dans une application. Le fait d’induire l’échec du programme permet de révéler des problèmes potentiels de sécurité avant la publication de l’application.

La [détection des risques de sécurité](https://www.microsoft.com/en-us/security-risk-detection/) est le service de test à données aléatoires (fuzzing) unique Microsoft permettant de rechercher des bogues critiques de sécurité dans le logiciel.

### <a name="conduct-attack-surface-review"></a>Examiner la surface d’attaque

L’examen de la surface d’attaque après l’exécution de code permet de garantir que toute modification de conception ou d’implémentation apportée à une application ou un système a été prise en compte. Cela permet de s’assurer que les nouveaux vecteurs d’attaque créés à la suite de modifications, y compris les modèles de menace, on été examinés et atténués.

Vous pouvez créer une image de la surface d’attaque en analysant l’application. Microsoft propose un outil d’analyse de la surface d’attaque appelé [Attack Surface Analyzer](https://www.microsoft.com/download/details.aspx?id=24487). Vous pouvez choisir parmi de nombreux outils d’analyse des vulnérabilités et de test dynamique commerciaux, notamment [OWASP Zed Attack Proxy Project](https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project), [Arachni](http://arachni-scanner.com/), [Skipfish](https://code.google.com/p/skipfish/), et [w3af](http://w3af.sourceforge.net/). Ces outils d’analyse parcourent votre application et mappent les parties de l’application accessibles via le web. Vous pouvez également effectuer des recherches dans Place de marché Azure pour trouver des [outils de développement](https://azuremarketplace.microsoft.com/marketplace/apps/category/developer-tools?page=1) similaires.

### <a name="perform-security-penetration-testing"></a>Effectuer des tests d’intrusion sécurisés

La sécurité de votre application est aussi importante que le bon fonctionnement de ses fonctionnalités. Intégrez les [tests d’intrusion](../fundamentals/pen-testing.md) aux processus de génération et de déploiement. Planifiez régulièrement des tests de sécurité et des analyses de vulnérabilité sur les applications déployées, et surveillez les ports ouverts, les points de terminaison et les attaques.

### <a name="run-security-verification-tests"></a>Exécuter des tests de vérification de sécurité

[Secure DevOps Kit pour Azure](https://azsk.azurewebsites.net/index.html) (AzSK) contient des SVT pour plusieurs services de la plateforme Azure. Vous exécutez ces SVT régulièrement pour garantir que votre abonnement Azure et les différentes ressources qui composent votre application sont sécurisées. Vous pouvez également automatiser ces tests à l’aide de la fonctionnalité des extensions de déploiement/d’intégration continus (CI/CD) d’AzSK, qui rend les SVT disponibles sous forme d’extension Visual Studio.

## <a name="next-steps"></a>Étapes suivantes
Dans les articles suivants, nous recommandons des contrôles et des activités de sécurité qui peuvent vous aider à concevoir et déployer des applications sécurisées.

- [Concevoir des applications sécurisées](secure-design.md)
- [Déployer des applications sécurisées](secure-deploy.md)