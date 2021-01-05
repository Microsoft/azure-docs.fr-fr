---
title: Fonctionnalités de repérage dans Azure Sentinel | Microsoft Docs
description: Utilisez les requêtes de chasse intégrées à Azure Sentinel pour vous aider à poser les bonnes questions afin de détecter les problèmes dans vos données.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: 6aa9dd27-6506-49c5-8e97-cc1aebecee87
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/10/2019
ms.author: yelevin
ms.openlocfilehash: ca483d993290e4d66ba257d4cb5de7fc3c952405
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660829"
---
# <a name="hunt-for-threats-with-azure-sentinel"></a>Repérer les menaces avec Azure Sentinel

Si vous êtes chargé d’examiner les menaces pour la sécurité, les puissants outils de repérage et de requête offerts par Azure Sentinel vous aideront à détecter ces menaces de manière proactive dans l’ensemble des sources de données de votre organisation. Toutefois, vos systèmes et appliances de sécurité génèrent des montagnes de données qui peuvent se révéler difficiles à analyser et à convertir en événements significatifs. Afin de faciliter la tâche des analystes de la sécurité dans la recherche proactive des nouvelles anomalies non détectées par vos applications de sécurité, les requêtes de repérage intégrées d’Azure Sentinel vous aident à poser les bonnes questions pour identifier les problèmes au niveau des données déjà présents dans le réseau. 

Par exemple, l’une des requêtes intégrées fournit des données sur les processus qui s’exécutent le moins souvent sur votre infrastructure. Vous ne souhaitez pas recevoir une alerte chaque fois que ces processus s’exécutent, car ils peuvent se révéler totalement inoffensifs, mais vous pouvez examiner cette requête de temps à autre pour vérifier qu’il ne se produit rien d’inhabituel. 



La fonctionnalité de repérage d’Azure Sentinel vous offre les avantages suivants :

- Requêtes intégrées : une page de démarrage fournit des exemples de requêtes préchargés conçus pour vous aider à démarrer et à vous familiariser avec les tables et le langage de requête. Ces requêtes de repérage intégrées sont élaborées en continu par les chercheurs Microsoft en matière de sécurité, qui ajoutent de nouvelles requêtes et affinent les requêtes existantes, de manière à vous offrir un point d’entrée pour la recherche de nouvelles détections et le repérage de l’origine des nouvelles attaques. 

- Langage de requête performant avec IntelliSense : repose sur un langage de requête suffisamment flexible pour garantir des fonctionnalités de repérage de niveau supérieur.

- Création de vos propres signets : lors du processus de repérage, vous pouvez rencontrer des correspondances, conclusions, tableaux de bord ou activités qui vous semblent inhabituels ou suspects. Pour marquer ces éléments afin d’y réaccéder plus facilement par la suite, utilisez la fonctionnalité de création de signets. Les signets vous permettent d’enregistrer des éléments pour plus tard, afin de les utiliser pour créer un incident à des fins d’investigation. Pour plus d’informations sur les signets, consultez l’article [Utiliser des signets dans le repérage](hunting.md).
- Automatisation de l’examen à l’aide de notebooks : les notebooks sont semblables à des playbooks pas à pas que vous pouvez générer pour suivre les différentes étapes d’une procédure d’examen et de repérage.  Les notebooks encapsulent toutes les étapes de repérage dans un playbook réutilisable qui peut être partagé avec d’autres personnes de votre organisation. 
- Interrogation des données stockées : les données sont accessibles dans des tables sur lesquelles vous pouvez exécuter des requêtes. Par exemple, vous pouvez interroger les événements de création de processus, les événements DNS (Domain Name Service), ainsi que de nombreux autres types d’événements.

- Liens vers la communauté : tirez parti de la puissance de l’ensemble de la communauté pour rechercher d’autres requêtes et sources de données.
 
## <a name="get-started-hunting"></a>Démarrer le repérage

1. Dans le Portail Azure Sentinel, cliquez sur **Repérage**.
  ![Début du repérage par Azure Sentinel](media/tutorial-hunting/hunting-start.png)

2. Lorsque vous ouvrez la page **Repérage**, toutes les requêtes de repérage s’affichent dans une même table. Cette table répertorie toutes les requêtes rédigées par l’équipe d’analystes de la sécurité Microsoft, ainsi que toutes les autres requêtes que vous pouvez avoir créées ou modifiées. Chaque requête décrit l’objet du repérage, ainsi que le type de données sur lequel ce processus s’exécute. Ces modèles sont regroupés par tactique : les icônes situées sur la droite classent le type de menace (par exemple, accès initial, persistance et exfiltration). Vous pouvez filtrer ces modèles de requête de repérage à l’aide des champs de votre choix. Vous pouvez enregistrer n’importe quelle requête dans vos favoris. Lorsque vous effectuez cette opération, la requête s’exécute automatiquement à chaque ouverture de la page **Repérage**. Vous pouvez créer votre propre requête de repérage ou cloner et personnaliser un modèle de requête de repérage existant. 
 
2. Cliquez sur **Exécuter la requête** à partir de la page de détails d’une requête de repérage pour exécuter une requête sans quitter la page de repérage.  Le nombre de correspondances s’affiche dans la table. Passez en revue la liste des requêtes de repérage et leurs correspondances. Consultez l’étape de la chaîne de destruction à laquelle est associée la correspondance.

3. Examinez rapidement la requête sous-jacente dans le volet de détails de requête ou cliquez sur **Afficher le résultat de la requête** pour ouvrir la requête dans Log Analytics. Dans la zone inférieure, passez en revue les correspondances de la requête.

4.    Pour marquer toutes les lignes suspectes qui doivent être examinées, cliquez sur chaque ligne et sélectionnez **Ajouter un signet**. 

5. Ensuite, réaccédez à la page **Repérage** principale et cliquez sur l’onglet **Signets** pour visualiser toutes les activités suspectes. 

6. Sélectionnez un signet, puis cliquez sur **Examiner** pour démarrer l’expérience d’examen. Vous pouvez filtrer les signets. Par exemple, si vous examinez une campagne, vous pouvez créer une balise pour cette dernière, puis filtrer tous les signets en fonction de cette campagne.

1. Une fois que vous avez découvert la requête de repérage qui fournit de précieux insights sur les attaques possibles, vous pouvez également créer des règles de détection personnalisées basées sur cette requête et communiquer ces insights sous forme d’alertes aux gestionnaires des incidents de sécurité.

 

## <a name="query-language"></a>Langage de requête 

La fonctionnalité de repérage d’Azure Sentinel repose sur le langage de requête Kusto. Pour plus d’informations sur le langage de requête et sur les opérateurs pris en charge, consultez l’article [Référence sur le langage de requête](../azure-monitor/log-query/get-started-queries.md).

## <a name="public-hunting-query-github-repository"></a>Référentiel public GitHub de requêtes de repérage

Consultez le [référentiel de requêtes de repérage](https://github.com/Azure/Orion). Complétez et utilisez les exemples de requêtes partagés par nos clients.

 

## <a name="sample-query"></a>Exemple de requête

Une requête classique commence par un nom de table suivi d’une série d’opérateurs séparés par \|.

Dans l’exemple ci-dessus, démarrez par le nom de table SecurityEvent et ajoutez les éléments enchaînés à l’aide d’une barre verticale selon vos besoins.

1. Définissez un filtre de temps pour consulter uniquement les enregistrements des sept jours qui précèdent.

2. Ajoutez un filtre à la requête pour qu’elle présente uniquement l’ID d’événement 4688.

3. Ajoutez un filtre à la requête sur l’élément CommandLine pour qu’il contienne uniquement les instances de cscript.exe.

4. Projetez uniquement les colonnes que vous souhaitez explorer et limitez les résultats à 1 000, puis cliquez sur **Exécuter la requête**.
5. Cliquez sur le triangle vert et exécutez la requête. Vous pouvez tester la requête et l’exécuter pour rechercher un comportement anormal.

## <a name="useful-operators"></a>Opérateurs utiles

Le langage de requête est très puissant grâce à ses nombreux opérateurs. Voici une liste de certains opérateurs particulièrement utiles :

**where** : filtre une table en fonction du sous-ensemble de lignes correspondant à un prédicat.

**summarize** : génère une table qui agrège le contenu de la table d’entrée.

**join** : fusionne les lignes de deux tables pour former une nouvelle table en mettant en correspondance les valeurs des colonnes spécifiées de chaque table.

**count** : renvoie le nombre d’enregistrements indiqué dans le jeu d’enregistrements d’entrée.

**top** : renvoie les N premiers enregistrements triés d’après les colonnes spécifiées.

**limit** : renvoie au maximum le nombre de lignes spécifié.

**project** : sélectionne les colonnes à inclure, renommer ou supprimer, puis insère les nouvelles colonnes calculées.

**extend** : crée des colonnes calculées et les ajoute au jeu de résultats.

**makeset** : renvoie un tableau (JSON) dynamique de l’ensemble de valeurs distinctes prises par Expr dans le groupe.

**find** : recherche les lignes qui correspondent à un prédicat dans un ensemble de tables.

## <a name="save-a-query"></a>Enregistrement d’une requête

Vous pouvez créer ou modifier une requête et l’enregistrer comme votre propre requête ou la partager avec des utilisateurs situés dans le même locataire.

   ![Enregistrer la requête](./media/tutorial-hunting/save-query.png)

Créez une requête de repérage :

1. Cliquez sur **Nouvelle requête**, puis sélectionnez **Enregistrer**.
2. Renseignez tous les champs vides, puis sélectionnez **Enregistrer**.

   ![Nouvelle requête](./media/tutorial-hunting/new-query.png)

Clonez et modifiez une requête de repérage existante :

1. Dans la table, sélectionnez la requête à modifier.
2. Sélectionnez les points de suspension (...) sur la ligne de la requête que vous souhaitez modifier, puis sélectionnez **Cloner la requête**.

   ![Clonage d’une requête](./media/tutorial-hunting/clone-query.png)
 

3. Modifiez la requête, puis sélectionnez **Créer**.

   ![Requête personnalisée](./media/tutorial-hunting/custom-query.png)

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez appris à effectuer un examen de repérage à l’aide d’Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :


- [Use notebooks to run automated hunting campaigns](notebooks.md) (Utiliser des notebooks pour exécuter des campagnes de repérage automatisées)
- [Keep track of data during hunting](bookmarks.md) (Suivre des données pendant la recherche)