---
title: Analyser les services de stockage Azure avec Azure Monitor pour le stockage | Microsoft Docs
description: Cet article décrit la fonctionnalité Azure Monitor pour le stockage, qui fournit aux administrateurs du stockage une compréhension rapide des problèmes de performances et d’utilisation avec leurs comptes de stockage Azure.
ms.subservice: ''
ms.topic: conceptual
author: mrbullwinkle
ms.author: mbullwin
ms.date: 05/11/2020
ms.openlocfilehash: 2538bc91be59ff12b39ee3f5f629e4c016480a97
ms.sourcegitcommit: 192f9233ba42e3cdda2794f4307e6620adba3ff2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96296333"
---
# <a name="monitoring-your-storage-service-with-azure-monitor-for-storage"></a>Analyse de votre service de stockage avec Azure Monitor pour le stockage

Azure Monitor pour le stockage fournit une analyse complète de vos comptes de stockage Azure grâce à une vue unifiée des performances, des capacités et de la disponibilité de vos services de stockage Azure. Vous pouvez observer la capacité de stockage et les performances de deux manières, en les affichant directement à partir d’un compte de stockage ou d’une vue dans Azure Monitor pour voir les différents groupes de comptes de stockage. 

Cet article vous aidera à comprendre l’expérience Azure Monitor pour le stockage pour en tirer des connaissances exploitables sur l’intégrité et les performances des comptes de stockage à l’échelle, avec la possibilité de se concentrer sur les zones réactives et de diagnostiquer les problèmes de latence, de limitation de bande passante et de disponibilité.

## <a name="introduction-to-azure-monitor-for-storage"></a>Présentation d’Azure Monitor pour le stockage

Avant d’entrer dans les détails de l’expérience avec la fonctionnalité, vous devez comprendre comment cette fonctionnalité présente et affiche les informations. Que vous désélectionniez la fonctionnalité de stockage directement à partir d’un compte de stockage ou d’Azure Monitor, Azure Monitor pour le stockage offre une expérience cohérente. 

La solution offre les avantages suivants :

* **Une perspective à l’échelle**, présentant une vue d’instantané de la disponibilité en fonction de l’intégrité du service de stockage ou de l’opération d’API, de l’utilisation en indiquant le nombre total de requêtes reçues par le service de stockage, et de la latence en indiquant la durée moyenne nécessaire pour que le service de stockage ou le type d’opération d’API traite les requêtes. Vous pouvez également afficher la capacité par objet blob, fichier, table et file d’attente.

* **Explorez l’analyse** d’un compte de stockage particulier pour vous aider à diagnostiquer les problèmes ou à effectuer une analyse détaillée par catégorie : disponibilité, performances, défaillances et capacité. La sélection de l’une de ces options fournit une vue détaillée des métriques.  

* **Personnalisable** vous permet de modifier les métriques que vous souhaitez voir, modifier ou définir des seuils qui s’alignent sur vos limites et les enregistrer comme votre propre classeur. Les graphiques du classeur peuvent être épinglés au tableau de bord Azure.  

Cette fonctionnalité ne vous oblige pas à activer ou à configurer quoi que ce soit : les métriques de stockage de vos comptes de stockage sont collectées par défaut. Si vous n’êtes pas familiarisé avec les métriques disponibles sur le stockage Azure, consultez la description et la définition des métriques de stockage Azure dans [Métriques de stockage Azure](../../storage/blobs/monitor-blob-storage.md).

>[!NOTE]
>Il n’y a pas de frais facturés pour accéder à cette fonctionnalité et vous ne paierez que pour les fonctionnalités essentielles d’Azure Monitor que vous configurez ou activez, comme décrit dans la page [Détails de la tarification d’Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).


## <a name="view-from-azure-monitor"></a>Voir à partir d’Azure Monitor

À partir d’Azure Monitor, vous pouvez afficher les détails des transactions, de la latence et de la capacité à partir de plusieurs comptes de stockage dans votre abonnement, et identifier les problèmes de performances, de capacité et de défaillances.

Pour afficher l’utilisation et la disponibilité de vos comptes de stockage pour l’ensemble de vos abonnements, procédez comme suit.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Sélectionnez **Analyse** dans le volet gauche du Portail Azure, puis sous la section **Insights**, sélectionnez **Comptes de stockage**.

    ![Vue de comptes de stockage multiples](./media/storage-insights-overview/multiple-storage-accounts-view-01.png)

### <a name="overview-workbook"></a>Classeur de présentation

Dans le classeur **Vue d’ensemble** de l’abonnement sélectionné, le tableau affiche les métriques de stockage interactives et l’état de disponibilité du service pour jusqu’à 5 comptes de stockage regroupés au sein de l’abonnement. Vous pouvez filtrer les résultats en fonction des options que vous sélectionnez dans les listes déroulantes suivantes :

* **Abonnements** : seuls les abonnements qui ont des comptes de stockage sont répertoriés.  

* **Comptes de stockage** : par défaut, 5 comptes de stockage sont présélectionnés. Si vous sélectionnez plusieurs comptes de stockage ou tous les comptes dans le sélecteur de portée, jusqu’à 200 comptes de stockage sont retournés. Par exemple, si vous avez un total de 573 comptes de stockage sur trois abonnements que vous avez sélectionnés, seuls 200 comptes sont affichés. 

* **Intervalle de temps** : par défaut, affiche les quatre dernières heures d’informations en fonction des sélections correspondantes effectuées.

La vignette de compteur sous les listes déroulantes cumule le total de comptes de stockage dans l’abonnement et indique le nombre total d’opérations sélectionnées. Il existe des codes de couleur conditionnels ou des cartes thermiques pour les colonnes du classeur qui signalent des métriques ou des erreurs de transaction. La couleur la plus profonde a la valeur la plus élevée et la couleur la plus claire désigne les valeurs les plus basses. Pour les colonnes basées sur une erreur, la valeur est en rouge, et pour les colonnes basées sur les métriques, la valeur est en bleu.

Sélectionnez une valeur dans les colonnes **Disponibilité**, **Latence E2E**, **Latence du serveur** et **Type d’erreur de transaction/Erreurs** pour aller au rapport adapté au type spécifique de métriques de stockage correspondant à la colonne sélectionnée pour ce compte de stockage. Pour plus d’informations sur les classeurs pour chaque catégorie, consultez la section [Classeurs de stockage détaillés](#detailed-storage-workbooks) ci-dessous. 

>[!NOTE]
>Pour plus d’informations sur les erreurs pouvant être affichées dans le rapport, consultez [Schéma de type de réponse](../../storage/blobs/monitor-blob-storage-reference.md#metrics-dimensions) et recherchez les types de réponse, comme **ServerOtherError**, **ClientOtherError** et **ClientThrottlingError**. Selon les comptes de stockage sélectionnés, si plus de trois types d’erreurs sont signalés, toutes les autres erreurs sont représentées sous la catégorie **Autres**.

Le seuil de **Disponibilité** par défaut est le suivant :

* Avertissement - 99 %
* Critique - 90 %

Pour définir un seuil de disponibilité en fonction des résultats de votre observation ou de vos besoins, consultez [Modifier le seuil de disponibilité](#modify-the-availability-threshold). 

### <a name="capacity-workbook"></a>Classeur de capacité

Sélectionnez **Capacité** en haut de la page et le classeur **Capacité** s’ouvrira. Il vous indique la quantité totale de stockage utilisée dans le compte et la capacité utilisée par chaque service de données dans le compte afin de faciliter l’identification des stockages sur- ou sous-utilisés.

![Classeur de capacité de comptes de stockage multiples](./media/storage-insights-overview/storage-account-capacity-02.png) 

Il existe des codes de couleur conditionnels ou des cartes thermiques pour les colonnes du classeur qui signalent les métriques de capacité avec une valeur bleue. La couleur la plus profonde a la valeur la plus élevée et la couleur la plus claire désigne les valeurs les plus basses.

Lorsque vous sélectionnez une valeur sous l’une des colonnes du classeur, vous accédez au classeur **Capacité** du compte de stockage. Pour plus d’informations sur l’analyse du rapport, consultez la section [Classeurs de stockage détaillés](#detailed-storage-workbooks) ci-dessous. 

## <a name="view-from-a-storage-account"></a>Afficher à partir d’un compte de stockage

Pour accéder à Azure Monitor pour machines virtuelles directement à partir d’un compte de stockage :

1. Dans le portail Azure, sélectionnez Comptes de stockage.

2. Dans la liste, choisissez un compte de stockage. Dans la section Analyse, choisissez Insights.

    ![Capture d’écran montrant la page du classeur Vue d’ensemble pour le compte de stockage.](./media/storage-insights-overview/storage-account-direct-overview-01.png)

Dans le classeur **Vue d’ensemble** du compte de stockage, plusieurs métriques de performances du stockage s’affichent pour vous aider à évaluer rapidement :

* L’intégrité du service de stockage, pour voir immédiatement si un problème en dehors de votre contrôle affecte le service de stockage dans la région dans laquelle il est déployé, indiquée sous la colonne **Résumé**.

* Les graphiques de performances interactifs présentant les détails les plus importants relatifs à la capacité de stockage, à la disponibilité, aux transactions et à la latence.  

* Les vignettes Métriques et État mettent en évidence la disponibilité du service, le nombre total de transactions pour le service de stockage, la latence E2E et la latence du serveur.

Sélectionnez un des boutons parmi **Défaillances**, **Performances**, **Disponibilité** et **Capacité** pour ouvrir le classeur respectif. 

![Page Vue d’ensemble du compte de stockage sélectionné](./media/storage-insights-overview/storage-account-capacity-01.png)

## <a name="detailed-storage-workbooks"></a>Classeurs de stockage détaillés

Si vous avez sélectionné une valeur dans les colonnes **Disponibilité**, **Latence E2E**, **Latence du serveur** ou **Type d’erreur de transaction/Erreurs** à partir du classeur **Vue d’ensemble** du compte de stockage multiple, ou en sélectionnant un des boutons parmi **Défaillances**, **Performances**, **Disponibilité** et **Capacité** du classeur **Vue d’ensemble** à partir d’un compte de stockage spécifique, vous obtenez un ensemble d’informations interactives relatives au stockage adaptées à cette catégorie.  

* **Disponibilité** ouvre le classeur **Disponibilité**. Il indique l’état d’intégrité actuel du service de stockage Azure, un tableau montrant l’état d’intégrité disponible de chaque objet catégorisé par le service de données défini dans le compte de stockage, avec une courbe de tendance représentant la plage horaire sélectionnée et un graphique de tendance de disponibilité pour chaque service de données dans le compte.  

    ![Exemple de rapport de disponibilité](./media/storage-insights-overview/storage-account-availability-01.png)

* **Latence E2E** et **Latence du serveur** ouvrent le classeur **Performances**. Il comprend une vignette de statut de cumul, qui indique la latence E2E et la latence du serveur, un graphique de performances de E2E et de latence du serveur, ainsi qu’une table détaillant les appels réussis par l’API, catégorisée par service de données défini dans le compte de stockage.

    ![Exemple de rapport de performances](./media/storage-insights-overview/storage-account-performance-01.png)

* En sélectionnant une des catégories d’erreur figurant dans la grille, vous ouvrez le classeur **Défaillances**. Le rapport affiche les vignettes métriques de toutes les autres erreurs côté client à l’exception des erreurs décrites et des requêtes réussies, les erreurs de limitation du client, un graphique de performances pour la métrique de dimension **Type de réponse** de la transaction spécifique à l’attribut ClientOtherError et deux tables : **Transactions par nom d’API** et **Transactions par type de réponse**.

   ![Exemple de rapport de défaillance](./media/storage-insights-overview/storage-account-failures-01.png)

* **Capacité** ouvre le classeur **Capacité**. Il indique la quantité totale de stockage utilisée pour chaque objet de données de stockage dans le compte dans les vignettes et le graphique, ainsi que le nombre d’objets de données stockés dans le compte.  

    ![Page Capacité du compte de stockage sélectionné](./media/storage-insights-overview/storage-account-capacity-01.png) 

## <a name="pin-and-export"></a>Épingler et exporter

Vous pouvez épingler une des sections métriques à un tableau de bord Azure en sélectionnant l’icône de punaise en haut à droite de la section.

![Exemple d’épinglage de section de métrique au tableau de bord](./media/storage-insights-overview/workbook-pin-example.png)

Les classeurs **Vue d’ensemble** et **Capacité** du compte de stockage et multi-abonnement prennent en charge l’exportation des résultats au format Excel en sélectionnant l’icône de flèche vers le bas à droite de l’icône de punaise.

![Exemple d’exportation de résultats de grille de classeur](./media/storage-insights-overview/workbook-export-example.png)

## <a name="customize-azure-monitor-for-storage"></a>Personnaliser Azure Monitor pour le stockage

Cette section présente les scénarios courants de modification du classeur pour le personnaliser en fonction de vos besoins d’analyse de données :

* Régler la portée du classeur pour qu’il sélectionne toujours un abonnement ou des comptes de stockage particuliers
* Modifier les métriques dans la grille
* Modifier le seuil de disponibilité
* Modifier le rendu des couleurs

Les personnalisations sont enregistrées dans un classeur personnalisé pour empêcher le remplacement de la configuration par défaut dans notre classeur publié. Les classeurs sont enregistrés au sein d’un groupe de ressources, soit dans la section **Mes rapports** qui est privée ou dans la section **Rapports partagés** accessible à tous les utilisateurs ayant accès au groupe de ressources. Une fois que vous avez enregistré le classeur personnalisé, vous devez accéder à la galerie de classeurs pour le lancer.

![Lancez la galerie de classeurs à partir de la barre de commandes](./media/storage-insights-overview/workbook-command-bar-gallery.png)

### <a name="specifying-a-subscription-or-storage-account"></a>Spécification d’un abonnement ou d’un compte de stockage

Vous pouvez configurer les classeurs **Vue d’ensemble** et **Capacité** du compte de stockage et multi-abonnement pour les étendre à un ou plusieurs abonnements ou comptes de stockage à chaque exécution. Pour cela, procédez comme suit.

1. Sélectionnez **Analyse** à partir du portail, puis sélectionnez **Comptes de stockage** dans le volet gauche.

2. Dans le classeur **Vue d’ensemble**, dans la barre de commandes, sélectionnez **Modifier**.

3. Dans la liste déroulante **Abonnements**, sélectionnez un ou plusieurs abonnements à utiliser par défaut. N’oubliez pas que le classeur prend en charge la sélection de jusqu’à 10 abonnements au total.  

4. Dans la liste déroulante **Comptes de stockage**, sélectionnez un ou plusieurs comptes à utiliser par défaut. N’oubliez pas que le classeur prend en charge la sélection d’un total de jusqu’à 200 comptes de stockage. 

5. Sélectionnez **Enregistrer sous** dans la barre de commandes pour enregistrer une copie du classeur avec vos personnalisations, puis cliquez sur **Modifications terminées** pour revenir au mode lecture.  

### <a name="modify-metrics-and-colors-in-the-workbook"></a>Modifier les métriques et les couleurs dans le classeur

Les classeurs prédéfinis contiennent des données de métriques et vous avez la possibilité de modifier ou de supprimer les visualisations et de les personnaliser en fonction des besoins spécifiques de votre équipe.

Dans notre exemple, nous travaillons avec le classeur de capacité du compte de stockage et multi-abonnement pour montrer comment :

* Supprimer une métrique
* Modifier le rendu des couleurs

Vous pouvez effectuer les mêmes modifications sur n’importe lequel des classeurs **Défaillances**, **Performances**, **Disponibilité** et **Capacité**.

1. Sélectionnez **Analyse** à partir du portail, puis sélectionnez **Comptes de stockage** dans le volet gauche.

2. Sélectionnez **Capacité** pour basculer vers le classeur Capacité et, à partir de la barre de commandes, sélectionnez **Modifier**.

    ![Sélectionnez Modifier pour modifier un classeur](./media/storage-insights-overview/workbook-edit-workbook.png)

3. En regard de la section Métriques, sélectionnez **Modifier**.

    ![Sélectionnez Modifier pour modifier les métriques des classeurs de capacité](./media/storage-insights-overview/edit-metrics-capacity-workbook-01.png)

4. Nous allons supprimer la colonne **Chronologie de la capacité utilisée du compte**. Sélectionnez donc **Paramètres de colonne** dans la grille de métriques.

    ![Modifier les paramètres de colonne](./media/storage-insights-overview/edit-capacity-workbook-resource-grid.png)

5. Dans le volet **Modifier les paramètres de la colonne**, sélectionnez sous la section **Colonnes** **microsoft.storage/storageaccounts-Capacity-UsedCapacity Timeline$|Chronologie de la capacité utilisée du compte$** , et dans la liste déroulante **Convertisseur de colonne** sélectionnez **Masqué**.

6. Sélectionnez **Enregistrer et fermer** pour valider vos modifications.

À présent, nous allons modifier le thème de couleur pour les métriques de capacité dans le rapport afin d’utiliser le vert au lieu du bleu.

1. Sélectionnez **Paramètres de colonne** dans la grille de métriques.

2. Dans le volet **Modifier les paramètres de colonne**, sous la section **Colonnes**, sélectionnez **microsoft.storage/storageaccounts-Capacity-UsedCapacity$`|`microsoft.storage/storageaccounts/blobservices-Capacity-BlobCapacity$`|`microsoft.storage/storageaccounts/fileservices-Capacity-FileCapacity$`|`microsoft.storage/storageaccounts/queueservices-Capacity-QueueCapacity$`|`microsoft.storage/storageaccounts/tableservices-Capacity-TableCapacity$** . Dans la liste déroulante **Palette de couleurs**, sélectionnez **Vert**.

3. Sélectionnez **Enregistrer et fermer** pour valider vos modifications.

4. Sélectionnez **Enregistrer sous** dans la barre de commandes pour enregistrer une copie du classeur avec vos personnalisations, puis cliquez sur **Modifications terminées** pour revenir au mode lecture.  

### <a name="modify-the-availability-threshold"></a>Modifier le seuil de disponibilité

Dans cet exemple, nous travaillons avec le classeur Capacité du compte de stockage et expliquons comment modifier le seuil de disponibilité. Par défaut, les pourcentages de disponibilité des rapports de vignette et de grille sont configurés avec un seuil minimal de 90 et maximum de 99. Nous allons modifier la valeur de seuil minimum de disponibilité **% de disponibilité** dans la grille **Disponibilité par nom d’API** sur 85 %, ce qui signifie que l’état d’intégrité passe à Critique si le seuil est inférieur à 85 %. 

1. Sélectionnez **Comptes de stockage** dans le portail, puis sélectionnez un compte de stockage dans la liste.

2. Sélectionnez **Insights** dans le volet gauche.

3. Dans le classeur, sélectionnez **Disponibilité** pour basculer vers le classeur de disponibilité , puis sélectionnez **Modifier** dans la barre de commandes. 

4. Faites défiler vers le bas de la page et, dans la partie gauche en regard de la grille **Disponibilité par API**, sélectionnez **Modifier**.

    ![Modifier les paramètres de la grille de disponibilité par nom d’API](./media/storage-insights-overview/availability-workbook-avail-by-apiname.png)

5. Sélectionnez **Paramètres de colonne** puis, dans le volet **Modifier les paramètres de colonne**, sous la section **Colonnes**, sélectionnez **Disponibilité (%).(Seuils + formatés)** .

6. Modifiez la valeur de l'état d’intégrité **Critique** de **90** à **85**, puis cliquez sur **Enregistrer et fermer**.

    ![Modifier la valeur du seuil de disponibilité pour l’état critique](./media/storage-insights-overview/edit-column-settings-capacity-workbook-01.png)

7. Sélectionnez **Enregistrer sous** dans la barre de commandes pour enregistrer une copie du classeur avec vos personnalisations, puis cliquez sur **Modifications terminées** pour revenir au mode lecture.

## <a name="troubleshooting"></a>Dépannage

Pour obtenir des conseils généraux sur la résolution des problèmes, reportez-vous à [l’article de résolution des problèmes](troubleshoot-workbooks.md) pour les insights basés sur les workbooks.

Cette section est destinée à vous aider à diagnostiquer et résoudre certains des problèmes communs que vous êtes susceptible de rencontrer lors de l’utilisation d’Azure Monitor pour le stockage. La liste ci-dessous permet d’identifier les informations pertinentes pour un problème spécifique.

### <a name="resolving-performance-capacity-or-availability-issues"></a>Résolution des problèmes de performances, de capacité et de disponibilité

Pour résoudre les problèmes liés au stockage que vous identifiez avec Azure Monitor pour le stockage, consultez le [guide de résolution des problèmes](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md#troubleshooting-guidance) pour le stockage Azure.  

### <a name="why-can-i-only-see-200-storage-accounts"></a>Pourquoi ne puis-je voir que 200 comptes de stockage ?

Le nombre de comptes de stockage sélectionnés est limité à 200, quel que soit le nombre d’abonnements sélectionnés.

### <a name="how-to-change-the-coloring-and-threshold-for-availability"></a>Comment modifier la couleur et le seuil de disponibilité ?

Reportez-vous à la section [Modifier le seuil de disponibilité](#modify-the-availability-threshold) pour obtenir des instructions détaillées sur la façon de modifier les couleurs et seuils de disponibilité.

### <a name="how-to-analyze-and-troubleshoot-the-data-shown-in-azure-monitor-for-storage"></a>Comment analyser et dépanner les données affichées dans Azure Monitor pour le stockage ?

 Pour plus d’informations sur l’analyse et la résolution des problèmes liés aux données de stockage Azure affichées dans Azure Monitor pour le stockage, consultez l’article [Analyser, diagnostiquer et dépanner Microsoft Azure Storage](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md).

### <a name="why-dont-i-see-all-the-types-of-errors-in-metrics"></a>Pourquoi ne puis-je pas voir tous les types d’erreur dans les métriques ?

Actuellement, jusqu’à trois types d’erreurs différents sont affichés et le reste des erreurs est regroupé dans un seul compartiment. Ce comportement est contrôlé par splitByLimit et peut être modifié. Pour modifier cette propriété :

1. Cliquez sur Modifier le classeur.
2. Accédez aux métriques, cliquez sur Modifier, puis sélectionnez **Transactions, sommes** ou les métriques que vous souhaitez modifier.

    ![Accédez aux métriques, puis cliquez sur Modifier, puis sur Transactions, sommes](./media/storage-insights-overview/fqa7.png)

3. Modifiez ensuite le nombre de fractionnements.

    ![Sélectionner Paramètres de métrique](./media/storage-insights-overview/fqa7-2.png)

Si vous souhaitez afficher n types d’erreur différents, spécifiez une valeur splitByLimit de n + 1, avec 1 en plus pour le reste des erreurs.

###  <a name="i-saved-my-workbook-while-on-some-storage-account-why-cant-i-find-it-now"></a>J’ai enregistré mon classeur sur un compte de stockage. Pourquoi ne puis-je pas le trouver maintenant ?

Chaque classeur est enregistré dans le compte de stockage dans lequel vous l’avez enregistré. Essayez de trouver le compte de stockage spécifique dans lequel l’utilisateur a enregistré le classeur. Sans cela, il n’existe aucun moyen de rechercher un classeur spécifique sans connaître la ressource (le compte de stockage).

## <a name="next-steps"></a>Étapes suivantes

* Configurez les [alertes de métriques](../platform/alerts-metric.md) et les [notifications d’intégrité du service](../../service-health/alerts-activity-log-service-notifications-portal.md) pour configurer l’alerte automatisée afin de faciliter la détection des problèmes.

* Découvrez les scénarios que les classeurs sont conçus pour prendre en charge, comment créer et personnaliser des rapports existants, et bien plus encore en consultant la rubrique [Créer des rapports interactifs avec les classeurs Azure Monitor](../platform/workbooks-overview.md).

* Pour obtenir un guide détaillé concernant l'utilisation de Storage Analytics et d'autres outils permettant d'analyser, de diagnostiquer et de résoudre les problèmes d'Azure Storage, consultez [Analyse, diagnostic et résolution des problèmes rencontrés sur Microsoft Azure Storage](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md).
