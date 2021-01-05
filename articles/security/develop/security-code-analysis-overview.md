---
title: Vue d'ensemble de la documentation de Microsoft Security Code Analysis
description: En savoir plus sur l’extension Microsoft Security Code Analysis. Grâce à cette extension, vous pouvez ajouter l’analyse du code de sécurité aux pipelines CI/CD Azure DevOps.
author: sukhans
manager: sukhans
ms.author: terrylan
ms.date: 07/31/2019
ms.topic: article
ms.service: security
services: azure
ms.assetid: 521180dc-2cc9-43f1-ae87-2701de7ca6b8
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
ms.openlocfilehash: a3af3307892e82ecd0697124954e99837ad5eb9e
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94515691"
---
# <a name="about-microsoft-security-code-analysis"></a>À propos de Microsoft Security Code Analysis

Grâce à l'extension Microsoft Security Code Analysis, les équipes peuvent ajouter l'analyse du code de sécurité à leurs pipelines d'intégration et de livraison continues (CI/CD) Azure DevOps. Cette analyse est recommandée par les experts [Secure Development Lifecycle (SDL)](https://www.microsoft.com/securityengineering/sdl/practices) de Microsoft.

Une expérience utilisateur cohérente simplifie la prise en charge de la sécurité en masquant les complexités liées à l'exécution des outils. Grâce à une livraison des outils basée sur NuGet, les équipes n'ont plus à gérer l'installation ou la mise à jour des outils. Grâce aux interfaces de ligne de commande et aux interfaces de base disponibles pour les tâches de build, tous les utilisateurs bénéficient d'autant de contrôle qu'ils le souhaitent sur les outils.

Les équipes peuvent également utiliser de puissantes fonctionnalités de post-traitement telles que :

- Publication des journaux à des fins de rétention.
- Génération de rapports exploitables et axés sur les développeurs.
- Configuration des échecs de build sur les tests de régression.

## <a name="why-should-i-use-microsoft-security-code-analysis"></a>Pourquoi devrais-je utiliser Microsoft Security Code Analysis ?

### <a name="security-simplified"></a>Sécurité simplifiée

L'ajout d'outils Microsoft Security Code Analysis à votre pipeline Azure DevOps est aussi simple que l'ajout de nouvelles tâches. Personnalisez les tâches ou utilisez leur comportement par défaut. Les tâches s'exécutent dans le cadre de votre pipeline Azure DevOps et produisent des journaux détaillant différents types de résultats.

### <a name="clean-builds"></a>Des builds propres

Après avoir résolu les problèmes initiaux signalés par les outils, vous pouvez configurer l’extension pour interrompre les builds en cas de nouveaux problèmes.  La configuration de builds d'intégration continue sur chaque demande de tirage (pull request) est simple.

### <a name="set-it-and-forget-it"></a>Définissez-la, puis oubliez-la

Par défaut, les tâches et les outils de build restent à jour. S'il existe une version mise à jour d'un outil, vous n'avez pas besoin de la télécharger et de l'installer. L'extension procède à la mise à jour pour vous.

### <a name="under-the-hood"></a>Sous le capot

Les tâches de build de l'extension masquent les complexités liées aux opérations suivantes :
  - Exécution d'outils d'analyse statique de la sécurité.
  - Traitement des résultats des fichiers journaux pour créer un rapport récapitulatif ou interrompre la build.

## <a name="microsoft-security-code-analysis-tool-set"></a>Ensemble d'outils Microsoft Security Code Analysis

L'extension Microsoft Security Code Analysis met à votre disposition les dernières versions des outils d'analyse importants. L'extension comprend à la fois des outils gérés par Microsoft et des outils open source.

Ces outils sont automatiquement téléchargés sur l'agent hébergé dans le cloud une fois que vous avez utilisé la tâche de build correspondante pour configurer et exécuter le pipeline.

Cette section répertorie l'ensemble des outils actuellement disponibles dans l'extension. Surveillez l'ajout d'autres outils. En outre, si vous souhaitez que nous ajoutions des outils spécifiques, n'hésitez pas à nous faire part de vos suggestions.

### <a name="anti-malware-scanner"></a>Analyseur anti-programme malveillant

La tâche de build Analyseur anti-programme malveillant est désormais incluse dans l'extension Microsoft Security Code Analysis. Cette tâche doit être exécutée sur un agent de build sur lequel Windows Defender est déjà installé. Pour plus d'informations, consultez le [site web de Windows Defender](https://aka.ms/defender)

### <a name="binskim"></a>BinSkim

BinSkim est un outil d'analyse léger et exécutable qui vérifie les paramètres du compilateur, les paramètres de l'éditeur de liens et d'autres caractéristiques de sécurité des fichiers binaires. Cette tâche de build fournit un wrapper de ligne de commande autour de l'application console binskim.exe. BinSkim est un outil open source. Pour plus d'informations, consultez [BinSkim sur GitHub](https://github.com/Microsoft/binskim).

### <a name="credential-scanner"></a>Credential Scanner

Les mots de passe et autres secrets stockés dans le code source constituent un problème important. Credential Scanner est un outil d'analyse statique propriétaire qui permet de résoudre ce problème. Cet outil détecte les informations d'identification, les secrets, les certificats et tout autre contenu sensible dans votre code source et dans la sortie de votre build.

### <a name="roslyn-analyzers"></a>Analyseurs Roslyn

Roslyn Analyzers est un outil intégré au compilateur de Microsoft qui permet une analyse statique du code C# et Visual Basic managé. Pour plus d'informations, consultez [Analyseurs basés sur Roslyn](/dotnet/fundamentals/code-analysis/quality-rules/security-warnings).

### <a name="tslint"></a>TSLint

TSLint est un outil d'analyse statique extensible qui recherche les erreurs de lisibilité, de maintenabilité et de fonctionnalités dans le code TypeScript. Il est largement pris en charge par les éditeurs et les systèmes de build modernes. Vous pouvez le personnaliser avec vos propres règles, configurations et formateurs Lint. TSLint est un outil open source. Pour plus d'informations, consultez [TSLint sur GitHub](https://github.com/palantir/tslint).

## <a name="analysis-and-post-processing-of-results"></a>Analyse et post-traitement des résultats

L'extension Microsoft Security Code Analysis comporte également trois tâches de post-traitement. Ces tâches vous aident à analyser les résultats trouvés par les tâches de l'outil de sécurité. Lorsqu'elles sont ajoutées à un pipeline, ces tâches suivent généralement toutes les autres tâches de l'outil.

### <a name="publish-security-analysis-logs"></a>Publier les journaux d’analyse de la sécurité

La tâche de build Publier les journaux d'analyse de la sécurité conserve les fichiers journaux des outils de sécurité exécutés pendant la build. Vous pouvez lire ces journaux à des fins d'examen et de suivi.

Vous pouvez publier les fichiers journaux dans Azure Artifacts sous forme de fichier .zip. Vous pouvez également les copier sur un partage de fichiers accessible à partir de votre agent de build privé.

### <a name="security-report"></a>Rapport de sécurité

La tâche de build Rapport de sécurité analyse les fichiers journaux. Ces fichiers sont créés par les outils de sécurité exécutés pendant la build. La tâche de build crée ensuite un rapport de synthèse. Ce fichier présente tous les problèmes détectés par les outils d'analyse.

Vous pouvez configurer cette tâche de manière à fournir les résultats de certains outils ou de tous les outils. Vous pouvez également choisir le niveau de problème à signaler, comme les erreurs uniquement ou à la fois les erreurs et les avertissements.

### <a name="post-analysis-build-break"></a>Post-analyse (interruption de la build)

Avec la tâche de build Post-analyse, vous pouvez injecter un échec de build qui provoque délibérément l'échec d'une build. Vous injectez un échec de build si un ou plusieurs outils d'analyse signalent des problèmes dans le code.

Vous pouvez configurer cette tâche de manière à faire échouer la build lorsque des problèmes sont détectés par des certains outils ou par tous les outils. Vous pouvez également la configurer en fonction de la gravité des problèmes détectés, comme les erreurs ou les avertissements.

>[!NOTE]
>Par défaut, chaque tâche de build aboutit si elle se termine avec succès. Cette règle s'applique aussi bien lorsqu'un outil détecte des problèmes que lorsqu'il n'en détecte pas, afin que la build puisse être exécutée jusqu'à la fin en autorisant l'exécution de tous les outils.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des instructions sur l'intégration et l'installation de Microsoft Security Code Analysis, consultez notre [Guide d'intégration et d'installation](security-code-analysis-onboard.md).

Pour plus d’informations sur la configuration des tâches de build, consultez notre [Guide de configuration](security-code-analysis-customize.md) ou notre [Guide de configuration YAML](yaml-configuration.md).

Si vous avez d'autres questions sur l'extension et les outils proposés, consultez notre [page Forum aux questions](security-code-analysis-faq.md).