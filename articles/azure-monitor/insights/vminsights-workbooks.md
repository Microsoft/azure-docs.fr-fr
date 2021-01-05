---
title: Créer des rapports interactifs Azure Monitor pour machines virtuelles avec des classeurs
description: Simplifier la création de rapports complexes grâce à des classeurs paramétrés prédéfinis et personnalisés pour Azure Monitor pour machines virtuelles.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/12/2020
ms.openlocfilehash: 43cdb9de111bdea5486e49a56d58d38279b685c7
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95985541"
---
# <a name="create-interactive-reports-azure-monitor-for-vms-with-workbooks"></a>Créer des rapports interactifs Azure Monitor pour machines virtuelles avec des classeurs

Les classeurs regroupent du texte, des  [requêtes de journal](/azure/data-explorer/kusto/query/), des métriques et des paramètres sous la forme de rapports interactifs complets. Les classeurs sont modifiables par tous les membres de l’équipe ayant accès aux mêmes ressources Azure.

Les classeurs sont utiles pour les scénarios tels que les suivants :

* Explorer l’utilisation de votre machine virtuelle quand vous ne connaissez pas à l’avance les métriques intéressantes : utilisation du processeur, espace disque, mémoire, dépendances réseau, etc. Contrairement à d’autres outils d’analyse de l’utilisation, les classeurs vous permettent de combiner plusieurs types de visualisations et d’analyses, ce qui les rend très utiles pour ce type d’exploration sous forme libre.
* Expliquer à votre équipe le fonctionnement d’une machine virtuelle provisionnée récemment, en affichant les métriques des compteurs clés et d’autres événements de journal.
* Partager les résultats d’une expérimentation de redimensionnement de votre machine virtuelle avec d’autres membres de votre équipe. Vous pouvez expliquer les objectifs de l’expérimentation avec du texte, puis montrer chaque métrique d’utilisation et des requêtes analytiques utilisées pour évaluer l’expérimentation, en vous aidant de légendes claires indiquant si chaque métrique se situe au-dessus ou en dessous de la cible.
* Créer des rapports relatifs à l’impact d’une panne sur l’utilisation de votre machine virtuelle, en combinant des données, une explication du texte et une présentation des étapes suivantes pour éviter à l’avenir d’éventuelles interruptions.

Le tableau suivant récapitule les classeurs qu’inclut Azure Monitor pour machines virtuelles pour vous aider à commencer.

| Classeur | Description | Étendue |
|----------|-------------|-------|
| Performances | Fournit une version personnalisable de notre Liste des N premiers et la vue Graphiques dans un classeur unique qui exploite tous les compteurs de performances Log Analytics que vous avez activés.| À grande échelle |
| Compteurs de performance | Vue de graphique N premiers sur un vaste ensemble de compteurs de performances. | À grande échelle |
| Connexions | Le classeur Connexions fournit une vue détaillée des connexions entrantes et sortantes de vos machines virtuelles supervisées. | À grande échelle |
| Ports actifs | Fournit une liste des processus qui sont liés aux ports sur les machines virtuelles supervisées et à leur activité dans la plage de temps choisie. | À grande échelle |
| Ouvrir des ports | Fournit le nombre de ports ouverts sur vos machines virtuelles supervisées et les détails concernant ces ports. | À grande échelle |
| Connexions ayant échoué | Permet d’afficher le nombre de connexions ayant échoué sur vos machines virtuelles supervisées, la tendance des échecs et si le pourcentage d’échecs augmente au fil du temps. | À grande échelle |
| Sécurité et audit | Analyse de votre trafic TCP/IP qui génère des rapports sur les connexions globales et les connexions malveillantes, où les points de terminaison IP se trouvent partout dans le monde entier.  Pour activer toutes les fonctionnalités, vous devez activer la Détection de la sécurité. | À grande échelle |
| Trafic TCP | Rapport classé concernant vos machines virtuelles supervisées et leur trafic réseau envoyé, reçu et total dans une grille et affiché sous forme de courbe de tendance. | À grande échelle |
| Comparaison du trafic | Ce classeur vous permet de comparer les tendances du trafic réseau pour une seule machine ou un groupe de machines. | À grande échelle |
| Performances | Fournit une version personnalisable de notre vue Performances qui exploite tous les compteurs de performances Log Analytics que vous avez activés. | Machine virtuelle unique | 
| Connexions | Le classeur Connexions fournit une vue détaillée des connexions entrantes et sortantes de vos machines virtuelles. | Machine virtuelle unique |
 
## <a name="creating-a-new-workbook"></a>Création d’un classeur

Un classeur est composé de sections comprenant des graphiques, des tableaux, du texte et des commandes de saisie modifiables de manière indépendante. Pour mieux comprendre les classeurs, commençons par ouvrir un modèle et décrivons la procédure à suivre pour créer un classeur personnalisé. 

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Sélectionnez **Machines virtuelles**.

3. Sélectionnez une machine virtuelle dans la liste.

4. Dans la page de la machine virtuelle, dans la section **Supervision**, sélectionnez **Insights**.

5. Dans la page Insights de machine virtuelle, sélectionnez l’onglet **Performances** ou **Cartes**, puis sélectionnez **Voir les classeurs** à partir du lien affiché sur la page. Dans la liste déroulante, sélectionnez **Accéder à la galerie**.

    ![Capture d’écran de la liste déroulante des classeurs](media/vminsights-workbooks/workbook-dropdown-gallery-01.png)

    La galerie de classeurs est alors lancée avec plusieurs classeurs prédéfinis pour vous aider à commencer.

7. Créez un classeur en sélectionnant **Nouveau**.

    ![Capture d’écran de la galerie de classeurs](media/vminsights-workbooks/workbook-gallery-01.png)

## <a name="editing-workbook-sections"></a>Modification de sections de classeur

Les classeurs présentent deux modes : le **mode d’édition** et le **mode lecture**. Quand un nouveau classeur est lancé pour la première fois, il s’ouvre en **mode d’édition**. Tout le contenu du classeur s’affiche, notamment les étapes et les paramètres qui sont généralement masqués. Le **mode lecture** présente une vue de rapport simplifiée. Le mode lecture vous permet de masquer les éléments complexes de création du rapport, tout en accédant en quelques clics aux paramètres et mécanismes sous-jacents si vous avez besoin de les modifier.

![Capture d’écran de la section de Classeur des machines virtuelles dans Azure Monitor montrant un nouveau classeur en mode d’édition avec les contrôles d’édition mis en évidence.](media/vminsights-workbooks/workbook-new-workbook-editor-01.png)

1. Quand vous avez fini de modifier une section, cliquez sur **Fin de l’édition** en bas à gauche de la section.

2. Pour créer un doublon de section, cliquez sur l’icône **Cloner cette section**. La création de sections en double est une bonne façon d’itérer sur une requête sans perdre les itérations précédentes.

3. Pour déplacer une section vers un classeur, cliquez sur l’icône **Déplacer vers le haut** ou **Déplacer vers le bas**.

4. Pour supprimer une section de façon permanente, cliquez sur l’icône **Supprimer**.

## <a name="adding-text-and-markdown-sections"></a>Ajout de texte et de sections Markdown

Le fait d’ajouter des en-têtes, des explications et des commentaires à vos classeurs permet de donner un aspect narratif à un ensemble de tables et de graphiques. Les sections de texte des classeurs prennent en charge la [syntaxe Markdown](https://daringfireball.net/projects/markdown/) pour la mise en forme du texte (en-têtes, gras, italique et listes à puces, par exemple).

Pour ajouter une section de textes à votre classeur, utilisez le bouton **Ajouter du texte** situé en bas du classeur, ou de chaque section.

## <a name="adding-query-sections"></a>Ajout de sections de requête

![Section de requête d’un classeur](media/vminsights-workbooks/005-workbook-query-section.png)

Pour ajouter une section de requête à votre classeur, utilisez le bouton **Ajouter une requête** situé en bas du classeur ou en bas de chaque section.

Les sections de requête sont extrêmement flexibles et peuvent être utilisées pour répondre à des questions telles que :

* Comment était mon utilisation du processeur au cours d’une période d’augmentation du trafic réseau ?
* Quelle était la tendance concernant l’espace disque disponible au cours du mois dernier ?
* Combien d’échecs de connexion réseau ma machine virtuelle a rencontrés au cours des deux dernières semaines ? 

Vous pouvez également formuler des requêtes concernant d’autres machines virtuelles que celle à partir de laquelle vous avez lancé le classeur. Vous pouvez lancer des requêtes sur plusieurs machines virtuelles, ainsi que sur des espaces de travail Log Analytics, à condition de disposer d’une autorisation d’accès à ces ressources.

Pour inclure des données d’autres espaces de travail Log Analytics ou d’une application Application Insights spécifique, utilisez l’identificateur de l’**espace de travail**. Pour en savoir plus sur les requêtes inter-ressources, reportez-vous aux [instructions officielles](../log-query/cross-workspace-query.md).

### <a name="advanced-analytic-query-settings"></a>Paramètres de requête analytique avancés

Chaque section a ses propres paramètres avancés, accessibles par le biais de l’icône ![Contrôles d’édition de la section Classeurs](media/vminsights-workbooks/006-settings.png) des paramètres située à droite du bouton **Ajouter des paramètres**.

![Capture d’écran de la boîte de dialogue Paramètres avancés dans la section Classeur des machines virtuelles d’Azure Monitor. L’icône qui ouvre la boîte de dialogue est mise en évidence.](media/vminsights-workbooks/007-settings-expanded.png)

|         |          |
| ---------------- |:-----|
| **Largeur personnalisée**    | Personnalise la taille d’un élément afin de faire tenir un grand nombre d’éléments sur une seule ligne. Vous pouvez ainsi mieux organiser vos tableaux et graphiques pour créer des rapports interactifs complets.  |
| **Visible sous conditions** | Spécifiez cette option pour masquer les étapes en fonction d’une valeur de paramètre en mode lecture. |
| **Exporter un paramètre**| Permet de sélectionner une ligne dans le tableau ou le graphique pour faire en sorte que des valeurs soient modifiées ou deviennent visibles.  |
| **Afficher la requête en mode lecture** | Affiche la requête au-dessus du graphique ou du tableau même en mode lecture.
| **Afficher le bouton Ouvrir dans Analytics en mode lecture** | Ajoute l’icône bleue Analytics dans le coin droit du graphique pour vous permettre d’y accéder en un seul clic.|

La plupart de ces paramètres sont relativement intuitifs, mais pour comprendre comment fonctionne l’option **Exporter un paramètre**, il est préférable d’examiner un classeur qui utilise cette fonctionnalité.

L’un des classeurs prédéfini, **Trafic TCP**, fournit des informations sur les métriques de connexion d’une machine virtuelle.

La première section du classeur est basée sur les données de requête de journal. La deuxième section est également basée sur les données de requête de journal, mais la sélection d’une ligne dans le premier tableau met à jour de façon interactive le contenu des graphiques :

![Capture d’écran de la section Machines virtuelles dans Azure Monitor montrant le trafic TCP du classeur prédéfini.](media/vminsights-workbooks/008-workbook-tcp-traffic.png)

Ce comportement est possible grâce à l’utilisation des paramètres avancés **Quand un élément est sélectionné, exporter un paramètre**, qui sont activés dans la requête de journal du tableau.

![Capture d’écran de la boîte de dialogue Paramètres avancés pour un classeur de machines virtuelles avec l’option « Quand un élément est sélectionné, exporter un paramètre » activée.](media/vminsights-workbooks/009-settings-export.png)

La deuxième requête de journal utilise alors les valeurs exportées quand une ligne est sélectionnée pour créer un ensemble de valeurs qui sont ensuite utilisés par l’en-tête de section et les graphiques. Si aucune ligne n’est sélectionnée, elle masque l’en-tête de section et les graphiques. 

Par exemple, le paramètre masqué dans la deuxième section utilise la référence suivante à partir de la ligne sélectionnée dans la grille :

```
VMConnection
| where TimeGenerated {TimeRange}
| where Computer in ("{ComputerName}") or '*' in ("{ComputerName}") 
| summarize Sent = sum(BytesSent), Received = sum(BytesReceived) by bin(TimeGenerated, {TimeRange:grain})
```

## <a name="adding-metrics-sections"></a>Ajout de sections de métriques

Les sections de métriques vous offrent un accès complet pour intégrer des données de métriques Azure Monitor dans vos rapports interactifs. Dans Azure Monitor pour machines virtuelles, les classeurs prédéfinis contiennent généralement des données de requête analytique plutôt que des données métriques.  Vous pouvez décider de créer des classeurs avec des données de métriques, ce qui vous permet de combiner les avantages des deux fonctionnalités dans un emplacement unique. Vous avez également la possibilité d’extraire des données de métriques à partir des ressources de l’ensemble des abonnements auxquels vous avez accès.

Voici un exemple de données de machine virtuelle extraites dans un classeur en vue d’afficher les performances du processeur sous forme de grille :

![Capture d’écran de la section Métriques d’un classeur de machine virtuelle dans Azure Monitor. Les performances de l’UC pour chaque machine virtuelle s’affichent sous forme de graphique.](media/vminsights-workbooks/010-metrics-grid.png)

## <a name="adding-parameter-sections"></a>Ajout de sections de paramètres

Les paramètres de classeur vous permettent de modifier les valeurs dans le classeur sans avoir à modifier manuellement les sections de requête ou de texte. Davantage de personnes peuvent ainsi utiliser ce type de rapports basés sur des classeurs, car il n’est pas nécessaire de comprendre le langage de requête analytique sous-jacent.

Pour remplacer la valeur d’un paramètre dans les sections de requête, de texte ou de paramètre, le nom du paramètre doit être saisi entre accolades, par exemple ``{parameterName}``. Les noms de paramètre doivent respecter des règles similaires à celles des identificateurs JavaScript (des caractères alphabétiques ou des traits de soulignement, suivis par des caractères alphanumériques ou des traits de soulignement). Par exemple, **a1** est autorisé, mais pas **1a**.

Les paramètres sont linéaires et appliqués aux différentes étapes du classeur, en partant du haut.  Des paramètres déclarés ultérieurement dans un classeur peuvent remplacer les paramètres déclarés précédemment. Cela permet également aux paramètres qui utilisent des requêtes d’accéder aux valeurs de paramètres définis précédemment. Au sein d’une étape de paramètre, les paramètres sont également linéaires et appliqués de gauche à droite, les paramètres de droite pouvant dépendre d’un paramètre déclaré antérieurement au sein de cette même étape.
 
Quatre types de paramètres différents sont actuellement pris en charge :

|                  |      |
| ---------------- |:-----|
| **Text**    | Permet à l’utilisateur de modifier une zone de texte. Vous pouvez éventuellement fournir une requête pour renseigner la valeur par défaut. |
| **Liste déroulante** | Permet à l’utilisateur de choisir parmi un ensemble de valeurs. |
| **Sélecteur d’intervalle de temps**| Permet à l’utilisateur de choisir parmi un ensemble prédéfini de valeurs d’intervalle de temps, ou de sélectionner un élément à partir d’un intervalle de temps personnalisé.|
| **Sélecteur de ressource** | Permet à l’utilisateur de choisir parmi les ressources sélectionnées pour le classeur.|

### <a name="using-a-text-parameter"></a>Utilisation d’un paramètre de texte

La valeur qu’un utilisateur entre dans la zone de texte est directement remplacée dans la requête, sans caractère d’échappement ni guillemets. Si la valeur dont vous avez besoin est une chaîne, le paramètre doit être saisi entre guillemets dans la requête (par exemple **‘{paramètre}’**).

Le paramètre de texte que la valeur entrée dans une zone de texte soit utilisée n’importe où. Cela peut être un nom de tableau, un nom de colonne, un nom de fonction, un opérateur, etc.  Le type de paramètre texte dispose d’un paramètre **Obtenir la valeur par défaut à partir d’une requête Analytics** qui permet au créateur du classeur d’utiliser une requête pour renseigner la valeur par défaut de cette zone de texte.

Quand vous utilisez la valeur par défaut obtenue à partir d’une requête de journal, seule la première valeur de la première ligne (ligne 0, colonne 0) est utilisée comme valeur par défaut. Par conséquent, il est recommandé de limiter votre requête afin qu’elle renvoie une seule ligne et une seule colonne. Toutes les autres données retournées par la requête sont ignorées. 

La valeur retournée par la requête est remplacée directement sans espace ni guillemets. Si la requête ne retourne aucune ligne, le résultat du paramètre est une chaîne vide (si le paramètre n’est pas obligatoire) ou une chaîne non définie (si le paramètre est obligatoire).

### <a name="using-a-drop-down"></a>Utilisation d’une liste déroulante

Le type de paramètre liste déroulante vous permet de créer un contrôle de liste déroulante pour pouvoir sélectionner une ou plusieurs valeurs.

La liste déroulante est remplie par une requête de journal ou JSON. Si la requête retourne une colonne, les valeurs de cette colonne correspondent à la fois à la valeur et à l’étiquette figurant dans le contrôle de liste déroulante. Si la requête retourne deux colonnes, la première colonne correspond à la valeur, et la deuxième colonne à l’étiquette affichée dans la liste déroulante. Si la requête retourne trois colonnes, la troisième colonne est utilisée pour indiquer la sélection par défaut dans cette liste déroulante. Cette colonne peut être de n’importe quel type, mais le plus simple est d’utiliser des valeurs booléennes ou numériques, où 0 correspond à false et 1 à true.

Si la colonne est de type chaîne, la chaîne null/vide est considérée comme fausse (false), et toute autre valeur est considérée comme vraie (true). Pour les listes déroulantes à sélection unique, la première valeur ayant la valeur true est utilisée comme sélection par défaut.  Pour les listes déroulantes à sélection multiple, toutes les valeurs ayant la valeur true forment le jeu de valeurs sélectionné par défaut. Les éléments de la liste déroulante s’affichent dans l’ordre où la requête a retourné les lignes. 

Examinons les paramètres présents dans le rapport Vue d’ensemble des connexions. Cliquez sur le symbole de modification en regard de **Direction**.

![Capture d’écran de la section permettant l’ajout et la modification des paramètres de rapport dans Azure Monitor. L’icône de modification du paramètre Direction est sélectionnée.](media/vminsights-workbooks/011-workbook-using-dropdown.png)

Cette action lance l’élément de menu **Modifier le paramètre**.

![Capture d’écran de la boîte de dialogue Modifier le paramètre. Le nom du paramètre est Direction, le type de paramètre est Liste déroulante et l’option Récupérer les données à partir de JSON est sélectionnée.](media/vminsights-workbooks/012-workbook-edit-parameter.png)

Le code JSON vous permet de générer une table arbitraire contenant des données. Par exemple, le code JSON suivant génère deux valeurs dans la liste déroulante :

```
[
    { "value": "inbound", "label": "Inbound"},
    { "value": "outbound", "label": "Outbound"}
]
```

Vous pouvez également utiliser une liste déroulante pour sélectionner un ensemble de compteurs de performances par nom :

```
Perf
| summarize by CounterName, ObjectName
| order by ObjectName asc, CounterName asc
| project Counter = pack('counter', CounterName, 'object', ObjectName), CounterName, group = ObjectName
```

La requête affiche les résultats comme suit :

![Liste déroulante de compteurs de performances](media/vminsights-workbooks/013-workbook-edit-parameter-perf-counters.png)

Les listes déroulantes sont des outils incroyablement efficaces pour créer et personnaliser des rapports interactifs.

### <a name="time-range-parameters"></a>Paramètres d’intervalle de temps

Bien que vous puissiez définir votre propre paramètre d’intervalle de temps personnalisé via le type de paramètre liste déroulante, vous pouvez également utiliser le type de paramètre intervalle de temps prêt à l’emploi si vous n’avez pas besoin du même niveau de flexibilité. 

Les types de paramètre intervalle de temps ont 15 plages par défaut qui vont de cinq minutes aux 90 derniers jours. Il est également possible de sélectionner un intervalle de temps personnalisé, ce qui permet au créateur du rapport de sélectionner des valeurs de début et de fin spécifiques pour l’intervalle de temps.

### <a name="resource-picker"></a>Sélecteur de ressource

Le type de paramètre de sélecteur de ressource vous donne la possibilité de limiter ou d’étendre votre rapport à certains types de ressources. Le classeur **Performances** est un exemple de classeur prédéfini qui utilise le type de sélecteur de ressource.

![Liste déroulante d’espaces de travail](media/vminsights-workbooks/014-workbook-edit-parameter-workspaces.png)

## <a name="saving-and-sharing-workbooks-with-your-team"></a>Enregistrer et partager des classeurs avec votre équipe

Les classeurs sont enregistrés dans un espace de travail Log Analytics ou une ressource de machine virtuelle, selon la façon dont vous accédez à la galerie de classeurs. Le classeur peut être sont enregistré dans la section **Mes rapports** qui est privée ou dans la section **Rapports partagés** qui est accessible à tous les utilisateurs ayant accès à la ressource. Pour afficher tous les classeurs de la ressource, cliquez sur le bouton **Ouvrir** dans la barre d’action.

Pour partager un classeur qui se trouve actuellement dans **Mes rapports** :

1. Dans la barre d’action, cliquez sur **Ouvrir**.
2. Cliquez sur le bouton «... » en regard du classeur que vous souhaitez partager.
3. Cliquez sur **Déplacer vers les rapports partagés**.

Pour partager un classeur avec un lien ou par e-mail, cliquez sur **Partager** dans la barre d’action. Gardez à l’esprit que les destinataires du lien ont besoin d’accéder à cette ressource dans le portail Azure pour afficher le classeur. Pour apporter des modifications, les destinataires doivent au moins disposer des autorisations de collaborateur pour la ressource.

Pour épingler un lien à un classeur dans un tableau de bord Azure :

1. Dans la barre d’action, cliquez sur **Ouvrir**.
2. Cliquez sur le bouton «... » en regard du classeur que vous souhaitez épingler.
3. Cliquez sur **Épingler au tableau de bord**.

## <a name="next-steps"></a>Étapes suivantes

- Pour connaître les limitations et les performances globales des machines virtuelles, consultez [Afficher les performances des machines virtuelles Azure](vminsights-performance.md).

- Pour en savoir plus sur les dépendances des applications détectées, consultez [View Azure Monitor for VMs Map](vminsights-maps.md) (Afficher la carte d’Azure Monitor pour machines virtuelles).