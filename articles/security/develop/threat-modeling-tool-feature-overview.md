---
title: Vue d’ensemble des fonctionnalités de Microsoft Threat Modeling Tool - Azure
description: Découvrez toutes les fonctionnalités disponibles dans Threat Modeling Tool, comme la vue d’analyse et les rapports.
author: jegeib
ms.author: jegeib
ms.service: security
ms.subservice: security-develop
ms.topic: article
ms.date: 08/17/2017
ms.openlocfilehash: 65a88f309602462e77336181316c0d5bf19a8a1e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90980672"
---
# <a name="threat-modeling-tool-feature-overview"></a>Vue d’ensemble de la fonctionnalité Outil de modélisation des menaces

L’outil de modélisation des menaces peut vous aider à résoudre vos besoins de modélisation des menaces. Pour une présentation de l’outil, consultez [Bien démarrer avec l’outil de modélisation des menaces](threat-modeling-tool-getting-started.md).

> [!NOTE]
>L’outil de modélisation des menaces étant souvent mis à jour, reportez-vous régulièrement à ce guide pour connaitre nos dernières fonctionnalités et améliorations.

Pour ouvrir une page vierge, sélectionnez **Create A Model** (Créer un modèle).

![Page vierge](./media/threat-modeling-tool-feature-overview/tmtstart.png)

Pour voir les fonctionnalités disponibles dans l’outil, utilisez le modèle des menaces créé par notre équipe dans l’exemple [Bien démarrer avec](threat-modeling-tool-getting-started.md).

![Modèle de menaces de base](./media/threat-modeling-tool-feature-overview/basictmt.png)

## <a name="navigation"></a>Navigation

Avant d’aborder les fonctionnalités intégrées, attardons-nous sur les principaux composants de l’outil.

### <a name="menu-items"></a>Éléments de menu

L’expérience est similaire à d’autres produits Microsoft. Passons en revue les éléments de menu de niveau supérieur.

![Éléments de menu](./media/threat-modeling-tool-feature-overview/menuitems.png)

| Étiquette                               | Détails      |
| --------------------------------------- | ------------ |
| **File** | <ul><li>Ouvrir, enregistrer et fermer des fichiers.</li><li>Se connecter aux comptes OneDrive et s’en déconnecter.</li><li>Partager des liens (afficher et modifier).</li><li>Afficher les informations sur le fichier.</li><li>Appliquer un nouveau modèle à des modèles existants.</li></ul> |
| **Modifier** | Annuler et rétablir des actions, ainsi que copier, coller et supprimer. |
| **Afficher** | <ul><li>Basculer entre les vues **Analyse** et **Conception**.</li><li>Ouvrir des fenêtres fermées (par exemple, gabarits, propriétés des éléments et messages).</li><li>Rétablir la disposition des paramètres par défaut.</li></ul> |
| **Diagramme** | Ajouter et supprimer des diagrammes, et parcourir les onglets des diagrammes. |
| **Rapports** | Créer des rapports HTML à partager avec d’autres. |
| **Aide** | Rechercher des guides vous aidant à utiliser l’outil. |

Les symboles sont des raccourcis pour les menus de niveau supérieur :

| Symbole                               | Détails      |
| --------------------------------------- | ------------ |
| **Ouvrir** | Ouvre un nouveau fichier. |
| **Save** | Enregistre le fichier en cours. |
| **Conception** | Ouvre le **Mode création**, où vous pouvez créer des modèles. |
| **Analyser** | Indique les menaces générées et leurs propriétés. |
| **Add diagram** (Ajouter un diagramme) | Ajoute un nouveau diagramme (semblable aux nouveaux onglets dans Excel). |
| **Delete diagram** (Supprimer un diagramme) | Supprime le diagramme actuel. |
| **Copier/Couper/Coller** | Copie, coupe et colle des éléments. |
| **Annuler/Rétablir** | Annule et rétablit des actions. |
| **Zoom in/Zoom out** (Zoom avant/Zoom arrière) | Effectue un zoom avant et arrière dans le diagramme pour une meilleure vue. |
| **Commentaires** | Ouvre le Forum MSDN. |

### <a name="canvas"></a>Canevas

Le canevas est l’espace dans lequel vous faites glisser et déposer les éléments. Glisser et déplacer est le moyen le plus rapide et le plus efficace pour générer des modèles. Vous pouvez également cliquer avec le bouton droit et sélectionner des éléments dans le menu pour ajouter des versions génériques d’éléments, comme illustré ci-après :

#### <a name="drop-the-stencil-on-the-canvas"></a>Déposer le gabarit sur le canevas

![Dépôt du canevas](./media/threat-modeling-tool-feature-overview/canvasdrop1.png)

#### <a name="select-the-stencil"></a>Sélectionner le gabarit

![Propriétés de l’élément](./media/threat-modeling-tool-feature-overview/canvasdrop2.png)

### <a name="stencils"></a>Gabarits

Selon le modèle sélectionné, vous pouvez rechercher tous les gabarits utilisables. Si vous ne trouvez pas les éléments adéquats, utilisez un autre modèle. Ou bien, vous pouvez modifier un modèle selon vos besoins. En règle générale, vous pouvez trouver une combinaison de catégories, telles que celles-ci :

| Nom du gabarit                               | Détails      |
| --------------------------------------- | ------------ |
| **Processus** | Applications, plug-ins de navigateur, menaces, machines virtuelles |
| **External interactor** (Interacteur externe) | Fournisseurs d’authentification, navigateurs, utilisateurs, applications web |
| **Data store** (Magasin de données) | Cache, stockage, fichiers de configuration, bases de données, registre |
| **Data flow** (Flux de données) | Binaire, ALPC, HTTP, HTTPS/TLS/SSL, IOCTL, IPSec, canal nommé, DCOM/RPC, SMB, UDP |
| **Trust line/Border boundary** (Limites de bordure/ligne d’approbation) | Réseaux d’entreprise, Internet, machine, bac à sable, mode utilisateur/noyau |

### <a name="notesmessages"></a>Notes/Messages (Remarques/Messages)

| Composant                               | Détails      |
| --------------------------------------- | ------------ |
| **Messages** | Logique d’outil interne qui avertit les utilisateurs chaque fois qu’il existe une erreur, telle qu’aucun flux de données entre des éléments. |
| **Remarques** | Les remarques manuelles sont ajoutées au fichier par l’équipe des ingénieurs durant le processus de conception et de révision. |

### <a name="element-properties"></a>Propriétés de l’élément

Les propriétés de l’élément dépendent de l’élément que vous sélectionnez. En dehors des limites d’approbation, tous les autres éléments contiennent trois sélections générales :

| Propriété d’élément                               | Détails      |
| --------------------------------------- | ------------ |
| **Nom** | Utile pour nommer vos processus, magasins, interacteurs et flux afin qu’ils soient facilement reconnus. |
| **Out of scope** (Hors de portée) | Si sélectionné, l’élément est extrait de la matrice de génération des menaces (non recommandé). |
| **Reason for out of scope** (Raison de l’hors de portée) | Champ de justification pour informer les utilisateurs de la raison pour laquelle l’option Hors de portée a été sélectionnée. |

Les propriétés sont modifiées pour chaque catégorie d’élément. Sélectionnez chaque élément pour vérifier les options disponibles. Ou bien, vous pouvez ouvrir le modèle pour en savoir plus. Passons en revue les fonctionnalités.

## <a name="welcome-screen"></a>Écran d’accueil

Quand vous ouvrez l’application, vous voyez l’écran **Welcome** (Bienvenue).

### <a name="open-a-model"></a>Ouvrir un modèle

Placez le curseur sur **Ouvrir un modèle** pour afficher deux options : **Ouvrir à partir de cet ordinateur** et **Ouvrir depuis OneDrive**. La première option ouvre l’écran **File Open** (Ouvrir le fichier). La seconde option vous guide dans le processus de connexion pour OneDrive. Une fois l’authentification réussie, vous pouvez sélectionner des fichiers et des dossiers.

![Ouvrir un modèle](./media/threat-modeling-tool-feature-overview/openmodel.png)

![Ouvrir depuis l’ordinateur ou OneDrive](./media/threat-modeling-tool-feature-overview/openmodel2.png)

### <a name="feedback-suggestions-and-issues"></a>Commentaires, suggestions et problèmes

Quand vous sélectionnez **Feedback, Suggestions and Issues** (Commentaires, suggestions et problèmes), vous accédez au forum MSDN sur les outils SDL. Vous pouvez lire les avis d’autres utilisateurs sur l’outil, y compris les solutions de contournement et de nouvelles idées.

![Capture d’écran montrant un bouton avec les commentaires de texte, les suggestions et les problèmes.](./media/threat-modeling-tool-feature-overview/feedback.png)

## <a name="design-view"></a>Mode création

Quand vous ouvrez ou créez un modèle, le **Mode création** s’ouvre.

### <a name="add-elements"></a>Ajouter des éléments

Vous pouvez ajouter des éléments dans la grille de deux manières :

- **Glisser-déplacer** : faites glisser l’élément souhaité dans la grille. Ensuite, utilisez les propriétés de l’élément pour fournir des informations supplémentaires.
- **Cliquer avec le bouton droit** : cliquez avec le bouton droit n’importe où sur la grille, puis sélectionnez des éléments dans le menu déroulant. Une représentation générique de l’élément que vous sélectionnez s’affiche sur l’écran.

### <a name="connect-elements"></a>Connecter des éléments

Vous pouvez connecter des éléments de deux manières :

- **Glisser-déplacer** : faites glisser le flux de données de votre choix vers la grille et connectez les deux extrémités aux éléments appropriés.
- **Cliquer + Maj** : cliquez sur le premier élément (envoi de données), appuyez et maintenez la pression sur la touche Maj, puis sélectionnez le deuxième élément (réception de données). Cliquez avec le bouton droit et sélectionnez **Connect** (Se connecter). Si vous utilisez un flux de données bidirectionnel, l’ordre n’est pas aussi important.

### <a name="properties"></a>Propriétés

 Pour afficher les propriétés pouvant être modifiées dans les gabarits, sélectionnez le gabarit ; les informations apparaissent alors. L’exemple ci-dessous présente un gabarit de **Database** (Base de données) avant et après être glissé vers le diagramme :

#### <a name="before"></a>Avant

![Avant](./media/threat-modeling-tool-feature-overview/properties1.png)

#### <a name="after"></a>After

![After](./media/threat-modeling-tool-feature-overview/properties2.png)

### <a name="messages"></a>Messages

Si vous créez un modèle de menaces et oubliez de connecter les flux de données aux éléments, une notification s’affiche. Vous pouvez ignorer le message, ou bien suivre les instructions pour résoudre le problème. 

![Capture d’écran montrant un connecteur de modèle de menace non connecté aux éléments, avec le message généré par ce problème.](./media/threat-modeling-tool-feature-overview/messages.png)

### <a name="notes"></a>Notes

Pour ajouter des remarques à votre diagramme, passez de l’onglet **Messages** à l’onglet **Notes** (Remarques).

## <a name="analysis-view"></a>Vue d’analyse

Après avoir généré votre diagramme, sélectionnez le symbole **Analyse** (la loupe) dans la barre d’outils de raccourcis pour basculer en mode **Analyse**.

![Vue d’analyse](./media/threat-modeling-tool-feature-overview/analysisview.png)

### <a name="generated-threat-selection"></a>Sélection de menaces générées

Quand vous sélectionnez une menace, vous pouvez utiliser trois fonctions distinctes :

| Fonctionnalité                               | Information      |
| --------------------------------------- | ------------ |
| **Indicateur de lecture** | <p>La menace est marquée comme lue, ce qui facilite le suivi des éléments que vous avez examinés.</p><p>![Indicateur lu/non lu](./media/threat-modeling-tool-feature-overview/readmode.png)</p> |
| **Focus d’interaction** | <p>L’interaction dans le diagramme qui appartient à une menace est mise en surbrillance.</p><p>![Focus d’interaction](./media/threat-modeling-tool-feature-overview/interactionfocus.png)</p> |
| **Propriétés de la menace** | <p>Des informations supplémentaires sur la menace s’affichent dans la fenêtre **Threat Properties** (Propriétés de la menace).</p><p>![Propriétés de la menace](./media/threat-modeling-tool-feature-overview/threatproperties.png)</p> |

### <a name="priority-change"></a>Modification de priorité

Vous pouvez changer le niveau de priorité de chaque menace générée. Des couleurs différentes permettent d’identifier le niveau de priorité (élevé, moyen et bas).

![Modification de priorité](./media/threat-modeling-tool-feature-overview/prioritychange.png)

### <a name="threat-properties-editable-fields"></a>Champs modifiables de propriétés de menaces

Comme le montre l’image précédente, vous pouvez changer les informations générées par l’outil. Vous pouvez également ajouter des informations à certains champs, telles qu’une justification. Le modèle génère ces champs. Si vous avez besoin de plus d’informations pour chaque menace, vous pouvez apporter des modifications.

![Propriétés de la menace](./media/threat-modeling-tool-feature-overview/threatproperties.png)

## <a name="reports"></a>Rapports

Après avoir changé les priorités et mis à jour l’état de chaque menace générée, vous pouvez enregistrer le fichier et/ou imprimer un rapport. Accédez à **Report** (Rapport) > **Create Full Report** (Créer un rapport complet). Attribuez un nom au rapport. Vous devez obtenir un graphique similaire à l’image suivante :

![Capture d’écran montrant un exemple de rapport de modélisation des menaces, notamment un résumé, des schémas et d’autres informations.](./media/threat-modeling-tool-feature-overview/report.png)

## <a name="next-steps"></a>Étapes suivantes

- Envoyez vos questions, commentaires et vos préoccupations à tmtextsupport@microsoft.com. **[Téléchargez](https://aka.ms/threatmodelingtool)** l’outil de modélisation des menaces pour démarrer.
- Pour contribuer au modèle pour la communauté, accédez à notre page [GitHub](https://github.com/Microsoft/threat-modeling-templates).