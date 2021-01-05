---
title: Se connecter à un compte SFTP (déconseillé)
description: Automatiser les tâches et les processus qui surveillent, créent, gèrent, envoient et reçoivent des fichiers pour un serveur SFTP à l’aide d’Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.date: 11/01/2019
tags: connectors
ROBOTS: NOINDEX
ms.openlocfilehash: 70fb956af7ff45c7b54f04d7ed441ec39f9d80a5
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92673818"
---
# <a name="monitor-create-and-manage-sftp-files-in-azure-logic-apps"></a>Surveiller, créer et gérer des fichiers SFTP dans Azure Logic Apps

> [!IMPORTANT]
> Veuillez utiliser le [connecteur SFTP-SSH](../connectors/connectors-sftp-ssh.md), car le connecteur SFTP est déconseillé. Vous ne pouvez plus sélectionner les déclencheurs et actions SFTP dans le concepteur d’applications logiques.

Pour automatiser les tâches qui surveillent, créent, envoient et reçoivent des fichiers sur un serveur [Secure File Transfer Protocol (SFTP)](https://www.ssh.com/ssh/sftp/), vous pouvez générer et automatiser les workflows d’intégration à l’aide d’Azure Logic Apps et du connecteur SFTP. SFTP est un protocole réseau qui fournit un accès aux fichiers, le transfert de fichiers et la gestion des fichiers sur n’importe quel flux de données fiable. Voici quelques exemples de tâches que vous pouvez automatiser :

* Superviser à quel moment des fichiers sont ajoutés ou changés.
* Obtenir, créer, copier, mettre à jour, lister et supprimer des fichiers.
* Obtenir les métadonnées et le contenu des fichiers.
* Extraire des archives dans des dossiers.

Vous pouvez utiliser des déclencheurs qui surveillent les événements sur votre serveur SFTP et mettent la sortie à la disposition d’autres actions. Vous pouvez utiliser des actions qui effectuent diverses tâches sur votre serveur SFTP. Vous pouvez également faire en sorte que d’autres actions de votre application logique utilisent la sortie d’actions SFTP. Par exemple, si vous récupérez régulièrement des fichiers de votre serveur SFTP, vous pouvez envoyer des alertes par e-mail au sujet de ces fichiers et de leur contenu en utilisant le connecteur Office 365 Outlook ou le connecteur Outlook.com. Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md)

## <a name="limits"></a>limites

Le connecteur SFTP gère uniquement les fichiers dont la taille est *inférieure ou égale à 50 Mo* et ne prend pas en charge la [segmentation des messages](../logic-apps/logic-apps-handle-large-messages.md). Pour les fichiers plus grands, utilisez le [connecteur SFTP-SSH](../connectors/connectors-sftp-ssh.md). Pour connaître les différences entre le connecteur SFTP et le connecteur SFTP-SSH, consultez [Comparer SFTP/SSH par rapport à SFTP](../connectors/connectors-sftp-ssh.md#comparison) dans l’article SFTP-SSH.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Vos informations d’identification de compte et adresse du serveur SFTP, qui permettent à votre application logique d’accéder à votre compte SFTP. Pour utiliser le protocole [SSH (Secure Shell)](https://www.ssh.com/ssh/protocol/), vous devez également accéder à une clé privée SSH et au mot clé privé SSH.

  > [!NOTE]
  >
  > Le connecteur SFTP prend en charge ces formats de clé privée : OpenSSH, ssh.com et PuTTY
  >
  > Lorsque vous créez votre application logique, après avoir ajouté le déclencheur SFTP ou une action , vous devrez fournir les informations de connexion pour votre serveur SFTP. 
  > Si vous utilisez une clé privée SSH, assurez-vous de * **copier** _ la clé à partir de votre fichier de clé privée SSH, puis _*_collez_*_ cette clé dans les détails de connexion. _*_N’entrez pas ni ne modifiez manuellement la clé_*_ , car cela peut entraîner l’échec de la connexion. 
  > Pour plus d’informations, consultez les étapes détaillées plus loin dans cet article.

_ Des connaissances de base [en création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md).

* L’application logique à partir de laquelle vous souhaitez accéder à votre compte SFTP. Pour démarrer avec un déclencheur SFTP, [créez une application logique vide](../logic-apps/quickstart-create-first-logic-app-workflow.md). Pour utiliser une action SFTP, démarrez votre application logique avec un autre déclencheur, par exemple, le déclencheur **Périodicité**.

## <a name="how-sftp-triggers-work"></a>Fonctionnement des déclencheurs SFTP

Les déclencheurs SFTP fonctionnent en interrogeant le système de fichiers SFTP et en recherchant tout fichier changé depuis la dernière interrogation. Certains outils permettent de conserver l’horodatage lorsque les fichiers sont modifiés. Dans ce cas, vous devez désactiver cette fonctionnalité pour permettre l’exécution du déclencheur. Voici quelques paramètres communs :

| Client SFTP | Action |
|-------------|--------|
| Winscp | Accédez à **Options** > **Préférences** > **Transférer** > **Modifier** > **Conserver l’horodatage** > **Désactiver** |
| FileZilla | Accédez à **Transférer** > **Conserver les horodatages des fichiers transférés** > **Désactiver** |
|||

Quand un déclencheur détecte un nouveau fichier, il vérifie que le nouveau fichier est complet et non partiellement écrit. Par exemple, un fichier peut être en cours de modification lorsque le déclencheur vérifie le serveur de fichiers. Pour éviter de retourner un fichier partiellement écrit, le déclencheur note l’horodatage du fichier qui comporte des modifications récentes, mais ne retourne pas immédiatement ce fichier. Le déclencheur retourne le fichier uniquement lors d’une nouvelle interrogation du serveur. Parfois, ce comportement peut entraîner un retard correspondant à jusqu’à deux fois l’intervalle d’interrogation du déclencheur.

## <a name="connect-to-sftp"></a>Se connecter à SFTP

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Connectez-vous au [portail Azure](https://portal.azure.com) et ouvrez votre application logique dans le concepteur d’application logique, si elle n’est pas déjà ouverte.

1. Pour les applications logiques vides, dans la zone de recherche, entrez « sftp » comme filtre. Dans la liste des déclencheurs, sélectionnez le déclencheur souhaité.

   -ou-

   Pour les applications logiques existantes, sous la dernière étape où vous souhaitez ajouter une action, choisissez **Nouvelle étape**. Dans la zone de recherche, entrez « sftp » comme filtre. Dans la liste des actions, sélectionnez l’action souhaitée.

   Pour ajouter une action entre des étapes, placez votre pointeur au-dessus de la flèche qui les sépare. Cliquez sur le signe plus ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action**.

1. Fournissez les informations nécessaires pour votre connexion.

   > [!IMPORTANT]
   >
   > Lorsque vous entrez votre clé privée SSH dans la propriété **Clé privée SSH** , suivez ces étapes supplémentaires pour vous assurer que vous fournissez la valeur complète et correcte pour cette propriété. 
   > Une clé non valide entraîne l’échec de la connexion.

   Bien qu’il est possible d’utiliser n’importe quel éditeur de texte, vous trouverez ici des exemples d’étapes montrant comment copier et coller correctement votre clé à l’aide de Notepad.exe.

   1. Ouvrez votre fichier de clé privée SSH dans un éditeur de texte. Dans cet exemple, les étapes utilisent le Bloc-notes.

   1. Dans le menu **Edition** du Bloc-notes, choisissez **Sélectionner tout**.

   1. Sélectionnez **Edition** > **Copier**.

   1. Dans le déclencheur SFTP ou l’action que vous avez ajoutés, collez la clé *complète* que vous avez copiée dans la propriété **Clé privée SSH** , qui prend en charge plusieurs lignes. **_Veillez à coller_* _ la clé. _*_N’entrez pas ni ne modifiez la clé manuellement_*_.

1. Après avoir entré les informations de connexion, choisissez _*Créer**.

1. Fournissez les informations nécessaires pour le déclencheur ou l’action sélectionnés et continuez à générer le flux de travail de votre application logique.

## <a name="examples"></a>Exemples

<a name="file-add-modified"></a>

### <a name="sftp-trigger-when-a-file-is-added-or-modified"></a>Déclencheur SFTP : Lors de l’ajout ou de la modification d’un fichier

Ce déclencheur démarre un flux de travail d’application logique quand un fichier est ajouté ou changé sur un serveur SFTP. Par exemple, vous pouvez ajouter une condition qui vérifie et extrait le contenu du fichier si ce contenu répond à une condition spécifiée. Vous pouvez ensuite ajouter une action qui obtient le contenu du fichier et le place dans un dossier sur le serveur SFTP.

**Exemple en entreprise**  : vous pouvez utiliser ce déclencheur pour superviser l’apparition dans un dossier SFTP de nouveaux fichiers représentant les commandes des clients. Vous pouvez ensuite utiliser une action SFTP comme **Obtenir le contenu du fichier** afin d’obtenir le contenu de la commande à des fins de traitement et stocker cette commande dans une base de données de commandes.

<a name="get-content"></a>

### <a name="sftp-action-get-content"></a>Action SFTP : Get content

Cette action obtient le contenu d’un fichier sur un serveur SFTP. Par exemple, vous pouvez ajouter le déclencheur de l’exemple précédent et une condition à laquelle le contenu du fichier doit satisfaire. Si la condition est vérifiée, l’action qui obtient le contenu peut s’exécuter.

## <a name="connector-reference"></a>Référence de connecteur

Pour obtenir des détails techniques sur les déclencheurs, les actions et les limites, qui sont décrits par la description OpenAPI du connecteur (anciennement Swagger), consultez la [page de référence](../data-factory/connector-sftp.md) du connecteur.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)