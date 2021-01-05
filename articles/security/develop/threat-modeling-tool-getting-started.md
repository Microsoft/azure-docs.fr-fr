---
title: Bien démarrer - Outil Microsoft de modélisation des menaces - Azure | Documents Microsoft
description: Découvrez comment commencer à utiliser Threat Modeling Tool. Créez un diagramme, identifiez des menaces, atténuez des menaces et validez chaque atténuation.
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.subservice: security-develop
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/17/2017
ms.author: jegeib
ms.openlocfilehash: 322f5f6a79bdce23706b2211ccc04ef2451675d0
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94515725"
---
# <a name="getting-started-with-the-threat-modeling-tool"></a>Bien démarrer avec l’outil de modélisation des menaces

Microsoft Threat Modeling Tool 2018 a été publié par GA en septembre 2018 sous forme de **[téléchargement](https://aka.ms/threatmodelingtool)** gratuit. La modification du mécanisme de livraison nous permet de transmettre les dernières améliorations et correctifs de bogues pour les clients chaque fois qu’ils ouvrent l’outil, rendant ainsi la gestion et l’utilisation plus facile.
Cet article vous guide tout au long du processus de mise en route avec l’approche de modélisation des menaces SDL Microsoft et vous montre comment utiliser l’outil pour développer d’excellents modèles de menaces comme structure fondamentale de votre processus de sécurité.

Cet article s’appuie sur les connaissances existantes de l’approche de modélisation des menaces SDL. Pour une vérification rapide, reportez-vous à **[Applications Web de la modélisation des menaces](/previous-versions/msp-n-p/ff648006(v=pandp.10))** et une version archivée de l’article MSDN **[Découvrir des failles de sécurité à l’aide de l’approche STRIDE](https://docs.google.com/viewer?a=v&pid=sites&srcid=ZGVmYXVsdGRvbWFpbnxzZWN1cmVwcm9ncmFtbWluZ3xneDo0MTY1MmM0ZDI0ZjQ4ZDMy)** publiée en 2006.

En résumé, l’approche implique la création d’un diagramme, l’identification des menaces, leur atténuation et la validation de chaque atténuation. Voici un diagramme qui met en évidence ce processus :

![Traitement SDL](./media/threat-modeling-tool-getting-started/sdlapproach.png)

## <a name="starting-the-threat-modeling-process"></a>Démarrage du processus de modélisation des menaces

Lorsque vous lancez l’outil de modélisation des menaces, vous remarquerez quelques éléments, comme indiqué dans l’image :

![Page de démarrage vierge](./media/threat-modeling-tool-feature-overview/tmtstart.png)

### <a name="threat-model-section"></a>Section de modèle des menaces

| Composant                                   | Détails                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Bouton de commentaires, suggestions et problèmes** | Vous conduit au **[Forum MSDN](https://social.msdn.microsoft.com/Forums/en-US/home?forum=sdlprocess)** pour tous les éléments SDL. Il vous donne la possibilité de consulter ce que font les autres utilisateurs, ainsi que les solutions de contournement et les recommandations. Si vous ne trouvez toujours pas ce que vous cherchez, envoyez un e-mail tmtextsupport@microsoft.com pour que notre équipe de support technique vous aide                                                                                                                            |
| **Créer un modèle**                          | Ouvre un canevas vide pour que vous puissiez dessiner votre diagramme. Veillez à sélectionner le modèle que vous souhaitez utiliser pour votre modèle                                                                                                                                                                                                                                                                                                                                                                       |
| **Modèle pour de nouveaux modèles**                 | Vous devez sélectionner le modèle à utiliser avant de créer un modèle. Notre modèle principal est le modèle du modèle des menaces Azure, qui contient les gabarits spécifiques à Azure, les menaces et les atténuations. Pour les modèles génériques, sélectionnez la Base de connaissances SDL TM dans le menu déroulant. Vous souhaitez créer votre propre modèle ou en envoyer un autre pour tous les utilisateurs ? Consultez notre page GitHub du **[Modèle de référentiel](https://github.com/Microsoft/threat-modeling-templates)** pour en savoir plus                              |
| **Ouvrir un modèle**                            | <p>Ouvre des modèles de menaces précédemment enregistrés. La fonctionnalité Modèles récemment ouverts est idéale si vous devez ouvrir vos fichiers les plus récents. En passant la souris sur la sélection, vous trouverez 2 méthodes pour ouvrir des modèles :</p><p><ul><li>Ouvrir depuis cet ordinateur : un moyen classique d’ouvrir un fichier à l’aide du stockage local</li><li>Ouvrir depuis OneDrive : les équipes peuvent utiliser des dossiers dans OneDrive pour enregistrer et partager tous leurs modèles de menaces dans un emplacement unique pour augmenter la productivité et la collaboration</li></ul></p> |
| **Guide de démarrage**                   | Permet d’ouvrir la page principale **[Outil Microsoft de modélisation des menaces](threat-modeling-tool.md)**                                                                                                                                                                                                                                                                                                                                                                                            |

### <a name="template-section"></a>Section de modèle

| Composant               | Détails                                                                                                                                                          |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Créer un nouveau modèle** | Ouvre un modèle vide à générer. À moins d’avoir une connaissance approfondie de la création de modèles à partir de rien, nous vous recommandons de construire à partir de types existants |
| **Ouvrir un modèle**       | Ouvre des modèles existants pour vous permettre d’y apporter des modifications                                                                                                              |

L’équipe de l’Outil de modélisation des menaces fonctionne en permanence pour améliorer les fonctionnalités de l’outil et l’expérience. Quelques modifications mineures peuvent être apportées au cours de l’année, mais toutes les modifications majeures nécessitent des réécritures dans le guide. Consultez-les souvent pour vous assurer d'obtenir les dernières annonces.

## <a name="building-a-model"></a>Création d’un modèle

Dans cette section, nous parlons de :

- Cristina (un développeur)
- Ricardo (un responsable des programmes) et
- Ashish (un testeur)

Ils suivent le processus de développement de leur premier modèle de menaces.

> Ricardo : Bonjour Cristina, j’ai travaillé sur le diagramme de modèle des menaces et je souhaite m'assurer que tout est correct. Pouvez-vous m’aider à vérifier ?
> Cristina : Absolument. Jetons un œil sur cette configuration.
> Ricardo ouvre l’outil et partage son écran avec Cristina.

![Modèle de menaces de base](./media/threat-modeling-tool-feature-overview/basictmt.png)

> Cristina : OK, ça semble simple, mais pouvez-vous m'aider ?
> Ricardo : Bien sûr ! Voici la répartition :
> - Notre utilisateur humain est dessiné comme une entité extérieure, un carré
> - Ils envoient des commandes vers notre serveur Web, le cercle
> - Le serveur Web consulte une base de données (deux lignes parallèles)

Ce que Ricardo vient de montrer à Cristina est un DFD, abréviation de **[diagramme de flux de données](https://en.wikipedia.org/wiki/Data_flow_diagram)** . L’outil de modélisation des menaces permet aux utilisateurs de spécifier des limites d’approbation, indiqués par les lignes en pointillés rouges, pour indiquer où les différentes entités sont sous contrôle. Par exemple, les administrateurs informatiques requièrent un système Active Directory à des fins d’authentification, par conséquent, Active Directory est en dehors de leur contrôle.

> Cristina : Ça me paraît correct. Qu’en est-il des menaces ?
> Ricardo : Je vais vous montrer.

## <a name="analyzing-threats"></a>Analyse des menaces

Après avoir cliqué sur la vue d’analyse à partir de la sélection du menu icône (fichier avec la loupe), il est dirigé vers la liste des menaces générées trouvées par l’outil Threat Modeling Tool en fonction du modèle par défaut, qui utilise l’approche SDL appelée **[STRIDE (usurpation d’identité, falsification, répudiation, divulgation d’informations, déni de service et élévation de privilèges)](https://en.wikipedia.org/wiki/STRIDE_(security))** . L’idée est que le logiciel relève d’un jeu de menaces prévisibles, qui peut être trouvé à l’aide de ces 6 catégories.

Cette approche est similaire à la sécurisation de votre maison en vous assurant que chaque porte et fenêtre a un mécanisme de verrouillage en place avant d’ajouter un système d’alarme ou de pourchasser le voleur.

![Menaces de base](./media/threat-modeling-tool-getting-started/basicthreats.png)

Ricardo commence par sélectionner le premier élément de la liste. Voici ce qui se passe :

Tout d’abord, l’interaction entre les deux gabarits est améliorée

![La capture d’écran montre deux gabarits et la flèche courbe qui les relie de manière plus prononcée.](./media/threat-modeling-tool-getting-started/interaction.png)

Ensuite, des informations supplémentaires sur les menaces s’affichent dans la fenêtre Propriétés de la menace

![Capture d’écran de la fenêtre Propriétés de la menace, qui contient le titre, la catégorie, la description, l’interaction et la priorité](./media/threat-modeling-tool-getting-started/interactioninfo.png)

La menace générée lui permet de comprendre les failles potentielles de la conception. La catégorisation STRIDE lui donne une idée des vecteurs d’attaque potentiels, tandis que la description supplémentaire lui indique exactement ce qui est incorrect, ainsi que des méthodes possibles pour l’atténuer. Il peut utiliser des champs modifiables pour écrire des commentaires dans les détails de la justification ou modifier des classements de priorité en fonction de la barre de bogues de son organisation.

Les modèles Azure disposent d’informations supplémentaires pour aider les utilisateurs à comprendre non seulement ce qui est incorrect, mais également comment résoudre le problème en ajoutant des descriptions, des exemples et des liens hypertexte à la documentation spécifique à Azure.

La description lui permet de mesurer l’importance de l’ajout d’un mécanisme d’authentification pour empêcher l’usurpation des utilisateurs, en révélant la première menace sur laquelle travailler. Après quelques minutes de discussion avec Cristina, ils comprennent l’importance de la mise en œuvre du contrôle d’accès et des rôles. Ricardo rempli quelques notes rapides pour s’assurer des mises en œuvre.

Lorsque Ricardo parcoure les menaces sous Divulgation des informations, il réalise que le plan de contrôle d’accès requiert certains comptes en lecture seule pour la génération de rapports et d’audits. Il se demande si cela peut être une nouvelle menace, mais les atténuations sont identiques, il note alors la menace en conséquence.
Il réfléchit un peu plus à la divulgation des informations et réalise que les bandes de sauvegarde vont avoir besoin de chiffrement, un travail pour l’équipe des opérations.

Les menaces non applicables à la conception en raison des atténuations existantes ou les garanties de sécurité peuvent être remplacées par « Non applicable » dans la liste déroulante État. Il existe trois autres possibilités : Non démarré (sélection par défaut), Besoin d'investigation (pour le suivi des éléments) et Mitigé (une fois que tout est terminé).

## <a name="reports--sharing"></a>Rapports et partage

Ricardo parcourt la liste avec Cristina et ajoute des remarques importantes, des atténuations /justifications, des modifications de la priorité et de l’état, il sélectionne Rapports -> Créer un rapport complet -> Enregistrer le rapport, qui imprime un rapport intéressant pour lui permettre de collaborer avec ses collègues et garantir la mise en œuvre d’un travail correctement sécurisé.

![Capture d’écran d’un rapport Threat Modeling représentatif](./media/threat-modeling-tool-feature-overview/report.png)

Si Ricardo souhaite partager le fichier à la place, il peut facilement le faire en enregistrant le compte OneDrive de son organisation. Une fois cela fait, il peut copier le lien du document et le partager avec ses collègues. 

## <a name="threat-modeling-meetings"></a>Réunion sur la modélisation des menaces

Lorsque Ricardo a envoyé son modèle de menaces à son collègue à l’aide de OneDrive, Ashish, le testeur, a été impressionné. Il semblait que Ricardo et Cristina avaient manqué quelques cas particuliers importants, qui pouvaient être facilement compromis. Son scepticisme est un complément aux modèles de menaces.

Dans ce scénario, une fois qu’Ashish a consulté le modèle des menaces, il a programmé deux réunions de modélisation des menaces : une réunion pour synchroniser sur le processus et organiser les diagrammes et une deuxième réunion pour examiner les menaces et signer.

Lors de la première réunion, Ashish a passé 10 minutes à présenter à chacun le processus de modélisation des menaces SDL. Puis, il a extrait le diagramme de modèle de menaces et commencé à l’expliquer en détail. Au bout de cinq minutes, un important composant manquant a été identifié.

Quelques minutes plus tard, Ashish et Ricardo ont engagé une discussion prolongée sur le mode de génération du serveur Web. Ce n’était pas le déroulement idéal d’une réunion, mais chacun d’eux par la suite se sont mis d’accord sur le fait que la découverte précoce de l’écart allait permettre de leur faire gagner du temps à l’avenir.

Lors de la deuxième réunion, l’équipe a passé en revue les menaces, décrit les différentes façons d’y remédier et signé le modèle des menaces. Ils ont vérifié le document dans le contrôle source et ont poursuivi le développement.

## <a name="thinking-about-assets"></a>Réfléchir sur les ressources

Certains lecteurs dont les menaces sont modélisées peuvent remarquer que nous n’avons pas du tout parlé des ressources. Nous avons découvert que de nombreux ingénieurs logiciels comprennent mieux leur logiciel qu’ils comprennent le concept de ressources et les ressources qui peuvent intéresser un attaquant.

Si vous allez modéliser des menaces pour une maison, vous pouvez commencer à réfléchir à votre famille, photos irremplaçables ou pièces de valeur. Par exemple, vous pouvez commencer à réfléchir à qui peut entrer en infraction et interrompre le système de sécurité actuel. Ou bien, vous pouvez commencer par considérer les caractéristiques physiques, telles que la piscine ou la véranda. Ceci est identique à réfléchir aux ressources, aux attaquants ou au concept de logiciels. Ces trois approches fonctionnent.

L’approche de modélisation des menaces présentée est considérablement plus simple que ce que Microsoft a fait dans le passé. Nous avons constaté que l’approche de conception de logiciels fonctionne bien pour de nombreuses équipes. Nous espérons inclure la vôtre.

## <a name="next-steps"></a>Étapes suivantes

Envoyez vos questions, commentaires et vos préoccupations à tmtextsupport@microsoft.com. **[Téléchargez](https://aka.ms/threatmodelingtool)** l’outil de modélisation des menaces pour démarrer.