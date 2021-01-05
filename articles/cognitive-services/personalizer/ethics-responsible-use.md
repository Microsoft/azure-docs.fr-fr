---
title: Déontologie et utilisation responsable - Personalizer
titleSuffix: Azure Cognitive Services
description: Ces instructions ont pour but de vous aider à implémenter une personnalisation de façon à ce que votre entreprise et votre service inspirent confiance. Prenez le temps de faire des recherches, d'en savoir plus et de délibérer sur l'impact de la personnalisation sur la vie des gens. En cas de doute, demandez conseil.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 06/12/2019
ms.openlocfilehash: 602da28f0c235fb0e797a493bc1160631c042a9d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005977"
---
# <a name="guidelines-for-responsible-implementation-of-personalizer"></a>Instructions pour une implémentation responsable de Personalizer

Pour que les individus et la société soient en mesure de tirer pleinement parti du potentiel de l’intelligence artificielle, les implémentations doivent être conçues de manière à gagner la confiance de ceux qui ajoutent l’intelligence artificielle à leurs applications et des utilisateurs d’applications intégrant l’intelligence artificielle. Ces instructions ont pour but de vous aider à implémenter Personalizer de façon à ce que votre entreprise et votre service inspirent confiance. Prenez le temps de faire des recherches, d'en savoir plus et de délibérer sur l'impact de la personnalisation sur la vie des gens. En cas de doute, demandez conseil.

Ces instructions n’ont pas vocation à constituer des conseils juridiques. Vous devez vous assurer par vous-même que votre application est conforme à l’évolution rapide de la législation dans ce domaine et dans votre secteur.

De plus, dans le cadre de la conception de votre application avec Personalizer, vous devez prendre compte le large éventail de responsabilités qui vous incombent lors du développement de tout système d’intelligence artificielle centré sur les données, notamment en matière de déontologie, de confidentialité, de sécurité, d’inclusion et de transparence. Vous trouverez plus d’informations sur ces éléments dans la section [Lectures recommandées](#recommended-reading).

Vous pouvez utiliser le contenu suivant comme check-list de démarrage, et la personnaliser et l’affiner en fonction de votre scénario. Ce document comporte deux sections principales : La première est dédiée à la mise en avant des considérations d’utilisation responsable lors du choix des scénarios, des caractéristiques et des récompenses pour Personalizer. La deuxième présente un ensemble de valeurs qui, selon Microsoft, doivent être prises en considération lors de la création de systèmes d’intelligence artificielle, et expose des suggestions actionnables et les risques quant à la manière dont votre utilisation de Personalizer peut avoir un impact sur ces valeurs.


## <a name="your-responsibility"></a>Votre responsabilité

Toutes les instructions pour une implémentation responsable reposent sur la base selon laquelle les développeurs et les entreprises utilisant Personalizer sont responsables des effets de l’utilisation de ces algorithmes dans la société. Si vous développez une application qui sera déployée par votre organisation, vous devez reconnaître votre rôle et votre responsabilité dans son fonctionnement et comment elle affecte les gens. Si vous concevez une application qui sera déployée par un tiers, vous devez définir d’un commun accord qui sera finalement responsable du comportement de l’application et documenter les modalités de cet accord.

La confiance repose sur la notion d’engagements pris : réfléchissez à vos utilisateurs, à la société et au cadre juridique dans lequel évolueront vos applications, afin d’identifier les engagements explicites et implicites qu’elles peuvent entraîner.

Microsoft améliore sans cesse ses outils et sa documentation pour vous aider à vous conformer à ces responsabilités. [Envoyez vos commentaires à Microsoft](mailto:cogsvcs-RL-feedback@microsoft.com?subject%3DPersonalizer%20Responsible%20Use%20Feedback&body%3D%5BPlease%20share%20any%20question%2C%20idea%20or%20concern%5D) si vous pensez que des outils, des fonctionnalités de produit et de la documentation supplémentaires vous aideraient à implémenter ces instructions pour l’utilisation de Personalizer.


## <a name="factors-for-responsibly-implementing-personalizer"></a>Facteurs pour l’implémentation responsable de Personalizer

L’implémentation de Personalizer peut apporter une belle valeur ajoutée à vos utilisateurs et à votre entreprise. Pour implémenter Personalizer de façon responsable, commencez par prendre en compte les consignes suivantes lors :

* Du choix de cas d’usage pour appliquer la personnalisation.
* De la création de [fonctions de récompense](concept-rewards.md).
* Du choix des [caractéristiques](concepts-features.md) relatives au contexte et des actions possibles que vous utiliserez pour la personnalisation.


## <a name="choosing-use-cases-for-personalizer"></a>Choix de cas d’usage pour Personalizer

L’utilisation d’un service qui apprend à personnaliser le contenu et les interfaces utilisateur est très utile. Elle peut aussi être néfaste si la personnalisation crée des effets secondaires négatifs dans le monde réel, notamment si les utilisateurs n’ont pas connaissance de la personnalisation du contenu.

L’utilisation de Personalizer peut présenter un risque accru d’effets secondaires négatifs ou un manque de transparence dans certains scénarios, par exemple si la « récompense » dépend de nombreux facteurs complexes à long terme qui, en cas de simplification exagérée en une récompense immédiate, peuvent générer des résultats défavorables pour les individus. Ces choix ont tendance à être considérés comme « ayant des conséquences », c'est-à-dire impliquant un risque de préjudice. Par exemple :


* **Finances** : offres de personnalisation sur des produits de prêt, d’assurance ou financiers, où les facteurs de risque sont basés sur des données que les individus ne connaissent pas, ne peuvent pas obtenir ou ne peuvent pas contester.
* **Formation :** classement personnalisé pour des formations et des établissements d’enseignement, où les recommandations peuvent propager des préjugés et réduire la connaissance des utilisateurs quant aux autres options disponibles.
* **Démocratie et participation civique** : la personnalisation du contenu pour les utilisateurs dans le but d’influencer leurs opinions a des conséquences et un caractère manipulatoire.
* **Évaluation des récompenses par un tiers** : personnalisation d’éléments où la récompense est basée sur une évaluation ultérieure de l’utilisateur par un tiers, au lieu de faire en sorte que la récompense soit générée par le comportement de l’utilisateur.
* **Intolérance à l’exploration** : toute situation où le comportement d’exploration de Personalizer peut nuire.

Lors du choix de cas d’usage pour Personalizer :

* Commencez le processus de conception en réfléchissant à la façon dont la personnalisation aide vos utilisateurs.
* Considérez les conséquences négatives dans le monde réel si certains éléments ne sont pas classés pour les utilisateurs en raison de l’exploration ou des modèles de personnalisation.
* Déterminez si votre cas d’usage constitue un traitement automatisé affectant de manière significative les personnes concernées et régi par l’article 22 du [RGPD](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32016R0679) ou d’autres lois.
* Prenez en compte les boucles de prophétie à réalisation automatique. Cela peut se produire si une récompense de personnalisation effectue l’entraînement d’un modèle de telle sorte que celui-ci puisse par la suite exclure un groupe démographique de l’accès au contenu pertinent. Par exemple, la plupart des gens habitant dans un quartier à faibles revenus ne reçoivent pas d’offre d’assurance premium, et petit à petit plus personne dans le quartier n’aura tendance à voir l’offre si l’exploration est insuffisante.
* Enregistrez des copies des modèles et des stratégies d’apprentissage pour le cas où il serait nécessaire de reproduire Personalizer par la suite. Vous pouvez le faire régulièrement ou lors de chaque période d’actualisation du modèle.
* Réfléchissez au niveau d’exploration adéquat pour l’espace, et à la façon de l’utiliser en tant qu’outil pour atténuer les effets de « chambre d’écho ».


## <a name="selecting-features-for-personalizer"></a>Sélection des caractéristiques pour Personalizer

La personnalisation du contenu nécessite de disposer d’informations utiles sur le contenu et l’utilisateur. N’oubliez pas que pour certaines applications et secteurs, certaines caractéristiques des utilisateurs peuvent être directement ou indirectement considérées comme discriminatoires et potentiellement illégales.

Réfléchissez à l’impact de ces caractéristiques :

* **Données démographiques des utilisateurs** : Caractéristiques relatives au sexe, à l’âge, à la race et à la religion : Ces caractéristiques peuvent être interdites dans certaines applications pour des raisons légales, et leur appliquer une personnalisation peut ne pas être éthique car celle-ci transmettrait des généralisations et des préjugés. Exemple de ce genre de propagation biaisée : une offre d’emploi d’ingénieur qui ne serait pas présentée aux personnes d’un certain âge ou d’un certain sexe.
* **Informations de localisation** : Dans de nombreuses régions du monde, les informations de localisation (par exemple un code postal ou un nom de quartier) peuvent être fortement corrélées avec un niveau de revenus, une race ou une religion.
* **Perception de l’équité par l’utilisateur** : Même dans les cas où votre application prend des décisions saines, prenez en compte que les utilisateurs peuvent percevoir que le contenu affiché dans votre application change d’une manière qui semble être liée à des caractéristiques qui seraient discriminatoires.
* **Préjugé involontaire dans les caractéristiques** :       Il existe des types de préjugés qui peuvent être introduits par l’utilisation de caractéristiques affectant uniquement une partie de la population. Vous devez faire doublement attention si des caractéristiques sont générées par un algorithme, par exemple en cas d’utilisation d’analyse d’images pour extraire des éléments dans une image ou d’analytique d’un texte pour découvrir des entités dans le texte. Familiarisez-vous avec les caractéristiques des services que vous utilisez pour créer ces caractéristiques.

Appliquez les pratiques suivantes lors du choix des caractéristiques à envoyer dans les contextes et les actions à Personalizer :

* Réfléchissez au caractère légal et éthique de l’utilisation de certaines caractéristiques pour certaines applications, et si certaines caractéristiques en apparence innocentes sont susceptibles d’être prises comme proxys pour d’autres caractéristiques que vous souhaitez utiliser ou éviter.
* Soyez transparent vis-à-vis des utilisateurs en leur précisant que des algorithmes et une analyse des données sont utilisés pour personnaliser les options qu’ils voient.
* Posez-vous les questions suivantes : Mes utilisateurs se soucieraient-ils et seraient-ils contents si j’utilisais ces informations pour personnaliser leur contenu ? Me sentirais-je à l’aise de leur montrer comment la décision a été prise de souligner ou de masquer certains éléments ?
* Utilisez des données comportementales plutôt que des données de classification ou de segmentation basées sur d’autres caractéristiques. Avant, les informations démographiques étaient utilisées par les détaillants pour des raisons historiques : il semblait simple de recueillir et d’agir sur des attributs démographiques avant l’ère numérique. Mais il convient aujourd’hui de questionner la pertinence des informations démographiques quand on dispose de données historiques, contextuelles et interactives réelles plus étroitement liées aux préférences et à l’identité des utilisateurs.
* Réfléchissez à la façon d’empêcher que des caractéristiques ne soient « usurpées » par des utilisateurs malveillants. En cas d’exploitation à grande échelle, Personalizer risquerait d’être entraîné de manière trompeuse afin de perturber, embarrasser et harceler intentionnellement certaines classes d’utilisateurs.
* Le cas échéant et dans la mesure du possible, concevez votre application de façon à autoriser les utilisateurs à accepter ou à refuser l’utilisation de certaines caractéristiques personnelles. Celles-ci pourraient être regroupées, par exemple dans des catégories telles que « Informations de localisation », « Informations d’appareil », « Historique des achats », et ainsi de suite.


## <a name="computing-rewards-for-personalizer"></a>Calcul des récompenses pour Personalizer

Personalizer s’efforce d’améliorer le choix de l’action à récompenser en fonction du score de récompense fourni par votre logique métier d’application.

Un score de récompense bien conçu agira comme un proxy à court terme vers un objectif professionnel, lié à la mission d’une organisation.

Par exemple, si vous récompensez les clics, le service Personalizer recherchera les clics au détriment de tout le reste, même si l’objet du clic n’est ni pertinent ni lié à un objectif professionnel.

Un site d’info, quant à lui, souhaitera peut-être définir des récompenses liées à quelque chose de plus significatif que les clics, par exemple « L’utilisateur a-t-il consacré assez de temps à la lecture du contenu ? » ou « A-t-il cliqué sur des références ou des articles pertinents ? ». Avec Personalizer, il est facile de lier étroitement des métriques à des récompenses. Mais veillez à ne pas confondre l’engagement utilisateur à court terme avec des résultats satisfaisants.

### <a name="unintended-consequences-from-reward-scores"></a>Conséquences inattendues des scores de récompense
Même si les scores de récompense sont conçus avec la meilleure des intentions, ils peuvent toujours entraîner des conséquences inattendues ou des résultats indésirables quant à la façon dont Personalizer évalue le contenu.

Voici quelques exemples :

* Le fait de récompenser la personnalisation du contenu vidéo d’après le pourcentage de la longueur de vidéo regardé aura probablement tendance à affecter un rang plus élevé aux vidéos plus courtes.
* Le fait de récompenser les partages de médias sociaux, sans analyser les sentiments et la façon dont les partages sont effectués ou le contenu lui-même, peut conduire à obtenir un classement de contenu offensant, non modéré ou incendiaire, qui tend à générer une quantité élevée d’« engagement » mais ajoute peu d’intérêt.
* Le fait de récompenser l’action sur des éléments d’interface utilisateur que les utilisateurs ne s’attendent pas à voir changer risque d’interférer avec la facilité d’utilisation et la prévisibilité de l’interface utilisateur, où des boutons changent étonnamment de place ou de fonction sans avertissement, rendant plus difficile le maintien de la productivité pour certains groupes d’utilisateurs.

Implémentez ces bonnes pratiques :

* Effectuez des expériences hors connexion avec votre système à l’aide de différentes approches de récompense afin de comprendre l’impact et les effets secondaires.
* Évaluez vos fonctions de récompense et demandez-vous comment une personne extrêmement naïve pourrait déformer leur interprétation et atteindre des résultats indésirables avec elles.


## <a name="responsible-design-considerations"></a>Considérations en matière de conception responsable

Voici une liste des domaines de conception pour des implémentations responsables de l’intelligence artificielle. Pour en savoir plus sur ce framework, consultez [The Future Computed](https://news.microsoft.com/futurecomputed/).

![Valeurs d’intelligence artificielle tirées de Future Computed](media/ethics-and-responsible-use/ai-values-future-computed.png)

### <a name="accountability"></a>Responsabilité
*Les personnes qui conçoivent et déploient des systèmes d’intelligence artificielle doivent être responsables du fonctionnement de leurs systèmes*.

* Créez des instructions internes expliquant comment implémenter Personalizer, documentez-les et communiquez-les à votre équipe, aux cadres dirigeants et aux fournisseurs.
* Effectuez des revues périodiques du mode de calcul des scores de récompenses, effectuez des évaluations hors connexion pour identifier les caractéristiques qui ont un impact sur Personalizer, et utilisez les résultats pour éliminer les caractéristiques inutiles et superflues.
* Indiquez clairement à vos utilisateurs comment Personalizer est utilisé, dans quel but et avec quelles données.
* Archivez les informations et les ressources (telles que les modèles, les stratégies d’apprentissage et autres données) utilisées par Personalizer dans le cadre de son fonctionnement, afin de pouvoir reproduire les résultats.

### <a name="transparency"></a>Transparence
*Les systèmes d’intelligence artificielle doivent être compréhensibles*. Avec Personalizer :

* *Donnez aux utilisateurs des informations sur la façon dont le contenu a été personnalisé.* Par exemple, vous pouvez proposer à vos utilisateurs un bouton nommé `Why These Suggestions?` présentant les principales actions et caractéristiques de l’utilisateur ayant joué un rôle dans les résultats de Personalizer.
* Assurez-vous que vos conditions d’utilisation mentionnent le fait que vous utiliserez les informations relatives aux utilisateurs et à leur comportement pour personnaliser l’expérience.

### <a name="fairness"></a>Équité
*Les systèmes d’IA doivent traiter tous les individus de manière équitable*.

* N’utilisez pas Personalizer pour les cas d’usage où les résultats ont des conséquences à long terme ou impliquent des dommages réels.
* N’utilisez pas de caractéristiques avec lesquelles il n’est pas approprié de personnaliser le contenu, ou qui peuvent aider à propager des préjugés indésirables. Par exemple, tous les individus ayant des circonstances financières semblables doivent voir les mêmes recommandations personnalisées pour les produits financiers.
* Veillez à bien comprendre les préjugés susceptibles d’exister dans des caractéristiques issues d’éditeurs, d’outils algorithmiques ou des utilisateurs eux-mêmes.

### <a name="reliability-and-safety"></a>Sécurité et fiabilité
*Les systèmes d’intelligence artificielle doivent être fiables et sécurisés*. Pour Personalizer :

* *Ne fournissez pas à Personalizer des actions qui ne doivent pas être choisies*. Par exemple, les films inappropriés doivent être filtrés et exclus des actions à personnaliser si vous effectuez une recommandation pour un utilisateur anonyme trop jeune.
* *Gérez votre modèle Personalizer comme une ressource d’entreprise*.  Réfléchissez à la fréquence d’enregistrement et de sauvegarde du modèle et des stratégies d’apprentissage sur lesquels repose votre boucle Personalizer, et traitez-la comme une ressource commerciale importante. La reproduction des résultats antérieurs est importante pour l’audit automatique et la mesure des améliorations.
* *Fournissez des canaux pour obtenir des commentaires directs des utilisateurs*. En plus de coder des contrôles de sécurité pour vous assurer que les audiences voient uniquement du contenu approprié, fournissez un mécanisme de commentaires afin que les utilisateurs puissent signaler le contenu susceptible d’être surprenant ou choquant. Pensez à utiliser Microsoft Content Moderator ou des outils supplémentaires pour examiner et valider votre contenu, en particulier s’il provient d’utilisateurs ou de tierces parties.
* *Effectuez des évaluations hors connexion fréquentes*. Elles vous aideront à superviser les tendances et à vous assurer que l’efficacité est connue.
* *Établissez un processus pour détecter les manipulations malveillantes et agir en conséquence*. Il existe des acteurs qui tireront parti de la capacité des systèmes de Machine Learning et d’intelligence artificielle à apprendre de leur environnement afin de diriger les résultats vers leurs objectifs. Si votre utilisation de Personalizer vous met en position d’influencer des décisions importantes, veillez à disposer de moyens appropriés pour détecter et atténuer ces classes d’attaques, notamment par un examen humain le cas échéant.

### <a name="security-and-privacy"></a>Sécurité et confidentialité
*Les systèmes d’intelligence artificielle doivent être sécurisés et respecter la confidentialité*. Lors de l’utilisation de Personalizer :

* *Informez immédiatement les utilisateurs des données qui seront collectées et de la façon dont elles seront utilisées, et obtenez leur accord au préalable*, conformément aux réglementations locales et de l’industrie.
* *Fournissez des contrôles utilisateur de protection de la confidentialité.* Pour les applications qui stockent des informations personnelles, pensez à fournir un bouton facile à trouver pour des fonctions telles que :
   * `Show me all you know about me`
   * `Forget my last interaction`
   * `Delete all you know about me`

Dans certains cas, il pourra s’agir d’une exigence juridique. Évaluez les compromis liés à un ré-entraînement régulier des modèles afin qu’ils ne contiennent pas de traces de données supprimées.

### <a name="inclusiveness"></a>Intégration
*Gérez un large éventail d’expériences et de besoins humains*.
* *Fournissez des expériences personnalisées pour les interfaces d’accessibilité.* L’efficacité qui provient d’une bonne personnalisation - appliquée afin de réduire la quantité d’effort et de mouvements et la répétition inutile des interactions, peut être particulièrement bénéfique aux personnes présentant un handicap.
* *Adaptez le comportement de l’application au contexte*. Vous pouvez utiliser Personalizer pour lever l’ambiguïté entre des intentions dans un bot conversationnel, par exemple quand l’interprétation correcte dépend du contexte.


## <a name="proactive-readiness-for-increased-data-protection-and-governance"></a>Préparation proactive pour une gouvernance et une protection des données accrues

Il est difficile de prédire les changements spécifiques dans des contextes réglementaires, mais en général il est judicieux d’aller au-delà du cadre juridique minimal concernant la garantie d’une utilisation respectueuse des données personnelles et l’offre d’une transparence et de choix liés à la prise de décision algorithmique.


* Planifiez à l’avance afin de faire face aux situations dans lesquelles il pourrait y avoir de nouvelles restrictions sur les données collectées auprès des utilisateurs et il pourrait être nécessaire de montrer comment ces données ont été utilisées pour prendre des décisions.
* Effectuez une préparation supplémentaire pour les cas où les utilisateurs peuvent inclure des populations vulnérables marginalisées, des enfants, des utilisateurs à vulnérabilité économique ou des utilisateurs susceptibles aux influences par manipulation algorithmique.
* Prenez en considération l’historique éventuel de mécontentement généralisé lié aux algorithmes et aux programmes de collecte des données avec influence et ciblage de l’audience, et réfléchissez à la manière d’éviter les erreurs stratégiques.


## <a name="proactive-assessments-during-your-project-lifecycle"></a>Évaluations proactives pendant le cycle de vie de votre projet

Pensez à créer des méthodes permettant aux membres d’équipe, aux utilisateurs et aux responsables d’entreprise de partager leurs inquiétudes relatives à l’utilisation responsable, et à créer un processus qui affecte un ordre de priorité à leur résolution et empêche toutes représailles.

Toute personne réfléchissant aux effets secondaires de l’utilisation d’une technologie est limité par son point de vue et son vécu. Étendez la gamme d’opinions disponibles en introduisant des voix plus diversifiées dans vos équipes, utilisateurs ou comités consultatifs, en faisant en sorte de les encourager à partager leur avis. Utilisez des supports de formation afin de développer les connaissances de votre équipe dans ce domaine et de favoriser la discussion de sujets complexes et sensibles.

Traitez les tâches relatives à une utilisation responsable comme toute autre tâche transversale dans le cycle de vie d’application, telles que celles liées à l’expérience utilisateur, à la sécurité ou à DevOps. Ces tâches et leurs exigences ne doivent pas être prises en considération à posteriori. L’utilisation responsable doit être discutée et vérifiée tout au long du cycle de vie d’application.

## <a name="questions-and-feedback"></a>Questions et commentaires

Microsoft améliore sans cesse ses outils et sa documentation pour vous aider à vous conformer à ces responsabilités. Notre équipe vous invite à [envoyer vos commentaires à Microsoft](mailto:cogsvcs-RL-feedback@microsoft.com?subject%3DPersonalizer%20Responsible%20Use%20Feedback&body%3D%5BPlease%20share%20any%20question%2C%20idea%20or%20concern%5D) si vous pensez que des outils, des fonctionnalités de produit et de la documentation supplémentaires vous aideraient à implémenter ces instructions pour l’utilisation de Personalizer.

## <a name="recommended-reading"></a>Lectures recommandées

* Consultez les six principes de Microsoft pour le développement responsable de l'intelligence artificielle publiés dans l'ouvrage de janvier 2018, [The Future Computed](https://news.microsoft.com/futurecomputed/).
* [Who Owns the Future?](https://www.goodreads.com/book/show/15802693-who-owns-the-future) de Jaron Lanier.
* [Weapons of Math Destruction](https://www.goodreads.com/book/show/28186015-weapons-of-math-destruction) de Cathy O’Neil.
* [Ethics and Data Science](https://www.oreilly.com/library/view/ethics-and-data/9781492043898/) de DJ Patil, Hilary Mason, Mike Loukides.
* [Code de déontologie ACM](https://www.acm.org/code-of-ethics)
* [Genetic Information Nondiscrimination Act - GINA](https://en.wikipedia.org/wiki/Genetic_Information_Nondiscrimination_Act)
* [FATML Principles for Accountable Algorithms](https://www.fatml.org/resources/principles-for-accountable-algorithms)


## <a name="next-steps"></a>Étapes suivantes

[Caractéristiques : action et contexte](concepts-features.md).
