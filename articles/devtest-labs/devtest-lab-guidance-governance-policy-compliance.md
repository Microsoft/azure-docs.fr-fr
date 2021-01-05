---
title: Stratégie et conformité de l’entreprise dans Azure DevTest Labs
description: Cet article fournit des conseils concernant la stratégie de l’entreprise et la conformité de l’infrastructure Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.reviewer: christianreddington,anthdela,juselph
ms.openlocfilehash: fe370ec882fd0462bba6afdf11c718cf46618f0c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85480930"
---
# <a name="governance-of-azure-devtest-labs-infrastructure---company-policy-and-compliance"></a>Gouvernance de l’infrastructure Azure DevTest Labs : stratégie d’entreprise et conformité
Cet article fournit des conseils concernant la stratégie de l’entreprise et la conformité de l’infrastructure Azure DevTest Labs. 

## <a name="public-vs-private-artifact-repository"></a>Référentiel d’artefacts public/privé

### <a name="question"></a>Question
Quand une organisation doit-elle utiliser un référentiel d’artefacts public/privé dans DevTest Labs ?

### <a name="answer"></a>Réponse
Le [référentiel d’artefacts public](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts) fournit un ensemble initial de progiciels les plus couramment utilisés. Il permet un déploiement rapide sans avoir besoin de consacrer beaucoup de temps à la reproduction d’outils de développement et de compléments courants. Vous pouvez choisir de déployer leur propre référentiel privé. Vous pouvez utiliser un référentiel public avec un référentiel privé en parallèle. Vous pouvez également choisir de désactiver le référentiel public. Les critères de déploiement d’un référentiel privé doivent reposer sur les questions et les considérations suivantes :

- L’organisation doit-elle disposer de logiciels d’entreprise sous licence dans le cadre de son offre DevTest Labs ? Dans l’affirmative, un référentiel privé doit être créé.
- L’organisation développe-t-elle un logiciel personnalisé assurant une fonction particulière nécessaire dans le cadre du processus général d’approvisionnement ? Dans l’affirmative, un référentiel privé doit être déployé.
- Si la stratégie de gouvernance de l’organisation exige l’isolation et que la configuration des référentiels externes n’est pas directement gérée par l’organisation, un référentiel privé d’artefacts doit être déployé. Dans le cadre de ce processus, il est possible de copier une version initiale du référentiel public et de l’intégrer au référentiel privé. Le référentiel public peut ensuite être désactivé afin que personne ne puisse plus y accéder au sein de l’organisation. Cette approche oblige tous les utilisateurs de l’organisation à ne disposer que d’un seul référentiel approuvé par l’organisation et à limiter la dérive des configurations.

### <a name="single-repository-or-multiple-repositories"></a>Référentiel unique ou référentiels multiples 

### <a name="question"></a>Question
Une organisation doit-elle planifier un référentiel unique ou autoriser plusieurs référentiels ?

### <a name="answer"></a>Réponse
Dans le cadre de la stratégie générale de gouvernance et de la gestion des configurations au sein de votre organisation, nous vous recommandons d’utiliser un référentiel centralisé. Lorsque plusieurs référentiels sont utilisés, ils peuvent au fil du temps se transformer en silos de logiciels non managés. Avec un référentiel central, plusieurs équipes peuvent utiliser les artefacts de ce référentiel pour leurs projets. Il renforce la normalisation, la sécurité et la facilité de gestion. Cela permet également d’éviter de faire double emploi. Dans le cadre de la centralisation, nous recommandons les mesures suivantes pour la gestion et la viabilité à long terme :

- Associez Azure Repos au même locataire Azure Active Directory que celui utilisé par l’abonnement Azure pour l’authentification et l’autorisation.
- Dans Azure Active Directory, créez un groupe nommé **Tous les développeurs DevTest Labs** qui soit géré de façon centralisée. Tous les développeurs qui contribuent au développement d’artefacts doivent être placés dans ce groupe.
- Le même groupe Azure Active Directory peut être utilisé pour donner accès au dépôt Azure Repos et au lab.
- Dans Azure Repos, la création de branches ou la duplication doit permettre de séparer un dépôt de développement du dépôt de production principal. Le contenu n’est ajouté à la branche maître qu’avec une requête d’extraction après un examen approprié du code. Une fois la modification approuvée par l’examinateur du code, le code actualisé est fusionné par un développeur principal responsable de la maintenance de la branche maître. 

## <a name="corporate-security-policies"></a>Stratégies de sécurité de l’entreprise

### <a name="question"></a>Question
Comment une organisation peut-elle veiller à la mise en place des stratégies de sécurité de l’entreprise ?

### <a name="answer"></a>Réponse
Une organisation peut s’en charger en prenant les mesures suivantes :

1. Élaboration et publication d’une stratégie de sécurité exhaustive. La stratégie énonce les règles d’utilisation acceptable en ce qui a trait à l’utilisation des logiciels et des ressources du cloud. Elle définit également ce qui la contrevient de façon évidente. 
2. Développer une image personnalisée, des artefacts personnalisés et un processus de déploiement permettant l’orchestration dans le domaine de la sécurité qui est défini avec un répertoire actif. Cette démarche permet de faire respecter les limites de l’entreprise et d’établir un ensemble commun de contrôles en matière d’environnement. Il s’agit des contrôles relatifs à l’environnement qu’un développeur peut prendre en compte dans le cadre de l’élaboration et du suivi d’un cycle de vie de développement sécurisé dans l’ensemble de son processus. L’objectif n’est pas de fournir un environnement qui soit trop contraignant et susceptible d’entraver le développement, mais un ensemble raisonnable de contrôles. Les stratégies de groupe de l’unité d’organisation (OU) qui héberge les machines virtuelles du laboratoire peuvent constituer un sous-ensemble de toutes les stratégies de groupe qui se trouvent en production. Elles peuvent également constituer un ensemble supplémentaire permettant de limiter de façon appropriée tous les risques identifiés.

## <a name="data-integrity"></a>Intégrité des données

### <a name="question"></a>Question
Comment une organisation peut-elle garantir l’intégrité des données de telle sorte que les développeurs distants ne puissent ni supprimer du code ni installer de logiciels malveillants ou non approuvés ?

### <a name="answer"></a>Réponse
Il existe plusieurs niveaux de contrôle pour atténuer les risques liés aux consultants externes, aux entrepreneurs ou aux employés qui se connectent à distance afin de collaborer dans DevTest Labs. 

Comme indiqué précédemment, la première étape consiste à élaborer et à définir une stratégie d’utilisation acceptable qui décrit clairement les sanctions en cas de violation de celle-ci. 

La première couche de contrôles de l’accès à distance consiste à appliquer une stratégie d’accès à distance via une connexion VPN qui n’est pas directement connectée au laboratoire. 

La deuxième couche de contrôles consiste à appliquer un ensemble d’objets de stratégie de groupe qui empêchent de copier-coller sur le bureau distant. Une stratégie de réseau peut être implémentée afin de ne pas autoriser les services sortants de l’environnement comme les services FTP et RDP ne faisant pas partie de l’environnement. Le routage défini par l’utilisateur peut contraindre l’ensemble du trafic réseau Azure à revenir en local, mais le routage ne peut pas tenir compte de toutes les URL susceptibles d’autoriser des téléchargements de données sans un contrôle via proxy permettant d’analyser les contenus et les sessions. Les adresses IP publiques peuvent être restreintes au sein du réseau virtuel prenant en charge DevTest Labs afin d’éviter le pontage d’une ressource réseau externe.

En définitive, le même type de restrictions doit être appliqué dans toute l’organisation qui doit également tenir compte de toutes les méthodes possibles de supports amovibles ou d’URL externes susceptibles d’accepter une publication de contenu. Consultez votre professionnel de la sécurité afin d’examiner et de mettre en œuvre une stratégie en matière de sécurité. Pour obtenir davantage de suggestions, consultez la section [Cybersécurité Microsoft](https://www.microsoft.com/security/default.aspx?&WT.srch=1&wt.mc_id=AID623240_SEM_sNYnsZDs).


## <a name="next-steps"></a>Étapes suivantes
Consultez la section [Intégration et migration d’applications](devtest-lab-guidance-governance-application-migration-integration.md).
