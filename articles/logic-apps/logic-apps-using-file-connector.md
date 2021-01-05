---
title: Se connecter à des systèmes de fichiers locaux
description: Automatiser des tâches et des flux de travail qui se connectent aux systèmes de fichiers locaux avec le connecteur de système de fichiers via la passerelle de données locale dans Azure Logic Apps
services: logic-apps
ms.suite: integration
author: derek1ee
ms.author: deli
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 10/08/2020
ms.openlocfilehash: 4715d7173dd959d12350229e457717c908a83756
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91873232"
---
# <a name="connect-to-on-premises-file-systems-with-azure-logic-apps"></a>Se connecter aux systèmes de fichiers locaux avec Azure Logic Apps

Avec Azure Logic Apps et le connecteur de système de fichiers, vous pouvez créer des tâches et des workflows automatisés qui créent et gèrent des fichiers sur un partage de fichiers local, par exemple :

- Créer, obtenir, ajouter, mettre à jour et supprimer des fichiers.
- Répertorier des fichiers dans des dossiers ou des dossiers racine.
- Obtenir les métadonnées et le contenu des fichiers.

  > [!IMPORTANT]
  > Actuellement, le connecteur de système de fichiers prend uniquement en charge les systèmes de fichiers Windows sur les systèmes d’exploitation Windows.  

Cet article vous explique comment vous connecter à un système de fichiers local, conformément à la description faite dans cet exemple de scénario : copier un fichier chargé dans Dropbox vers un partage de fichiers, puis envoyer un e-mail. Pour vous connecter en toute sécurité et d’accéder aux systèmes locaux, les applications logiques utilisent la [passerelle de données locale](../logic-apps/logic-apps-gateway-connection.md). Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md). Pour obtenir des informations techniques spécifiques aux connecteurs, consultez la [référence du connecteur de système de fichiers](/connectors/filesystem/).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Pour pouvoir connecter les applications logiques à des systèmes locaux tels que votre serveur de système de fichiers, vous devez [installer et configurer une passerelle de données locale](../logic-apps/logic-apps-gateway-install.md). De cette façon, vous pouvez spécifier que votre installation de passerelle soit utilisée lorsque vous créez la connexion du système de fichiers depuis votre application logique.

* Un [compte Dropbox](https://www.dropbox.com/), que vous pouvez obtenir gratuitement. Les informations d'identification de votre compte sont nécessaires pour établir une connexion entre votre application logique et votre compte Dropbox.

* Un accès à l’ordinateur qui héberge le système de fichiers que vous souhaitez utiliser. Par exemple, si vous installez la passerelle de données sur le même ordinateur que votre système de fichiers, vous avez besoin des informations d'identification du compte pour cet ordinateur.

* Un compte de messagerie d'un fournisseur pris en charge par Azure Logic Apps, comme Office 365 Outlook, Outlook.com ou Gmail. Pour les autres fournisseurs, [passez en revue la liste des connecteurs ici](/connectors/). Cette application logique utilise un compte professionnel ou scolaire. Si vous utilisez un autre compte de messagerie, les étapes générales sont identiques, mais votre interface utilisateur peut-être légèrement différente.

  > [!IMPORTANT]
  > Si vous souhaitez utiliser le connecteur Gmail, seuls les comptes professionnels G-Suite peuvent utiliser ce connecteur sans restriction dans Logic Apps. Si vous disposez d’un compte de consommateur Gmail, vous pouvez utiliser ce connecteur uniquement avec certains services approuvés par Google, ou vous pouvez [créer une application cliente Google pour servir lors de l’authentification avec votre connecteur Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Pour plus d’informations, consultez [Stratégies de confidentialité et de sécurité des données pour les connecteurs Google dans Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md). Pour cet exemple, vous avez besoin d’une application logique vide.

## <a name="add-trigger"></a>Ajouter un déclencheur

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Connectez-vous au [portail Azure](https://portal.azure.com) et ouvrez votre application logique dans le concepteur d’application logique, si elle n’est pas déjà ouverte.

1. Dans la zone de recherche, entrez « dropbox » comme filtre. Dans la liste des déclencheurs, sélectionnez ce déclencheur : **Quand un fichier est créé**

   ![Sélectionner un déclencheur Dropbox](media/logic-apps-using-file-connector/select-dropbox-trigger.png)

1. Connectez-vous avec vos informations d’identification de compte Dropbox et autorisez l’accès à vos données Dropbox pour Azure Logic Apps.

1. Fournissez les informations requises pour votre déclencheur.

   ![Déclencheur Dropbox](media/logic-apps-using-file-connector/dropbox-trigger.png)

## <a name="add-actions"></a>Ajouter des actions

1. Sous le déclencheur, choisissez **Étape suivante**. Dans la zone de recherche, entrez « système de fichiers » en tant que filtre. Dans la liste des actions, sélectionnez cette action : **Créer un fichier**

   ![Trouver le connecteur de système de fichiers](media/logic-apps-using-file-connector/find-file-system-action.png)

1. Si vous ne disposez pas encore d’une connexion à votre système de fichiers, vous êtes invité à créer une connexion.

   ![Créer une connexion](media/logic-apps-using-file-connector/file-system-connection.png)

   | Propriété | Obligatoire | Valeur | Description |
   | -------- | -------- | ----- | ----------- |
   | **Nom de connexion** | Oui | <*connection-name*> | Le nom souhaité pour votre connexion |
   | **Dossier racine** | Oui | <*root-folder-name*> | Dossier racine de votre système de fichiers, par exemple, si vous avez installé votre passerelle de données locale, comme un dossier local sur l'ordinateur où la passerelle de données locale est installée, ou le dossier d'un partage réseau auquel l'ordinateur peut accéder. <p>Par exemple : `\\PublicShare\\DropboxFiles` <p>Le dossier racine est le dossier parent principal qui est utilisé pour les chemins relatifs de toutes les actions liées aux fichiers. |
   | **Type d'authentification** | Non | <*auth-type*> | Type d’authentification utilisé par votre système de fichiers : **Windows** |
   | **Nom d’utilisateur** | Oui | <*domaine*>\\<*nom d’utilisateur*> <p>-ou- <p><*ordinateur local*>\\<*nom d’utilisateur*> | Nom d’utilisateur de l’ordinateur qui héberge le dossier de votre système de fichiers. <p>Si le dossier de votre système de fichiers se trouve sur le même ordinateur que la passerelle de données locale, vous pouvez utiliser <*ordinateur local*>\\<*nom d’utilisateur*>. |
   | **Mot de passe** | Oui | <*your-password*> | Mot de passe de l'ordinateur qui héberge votre système de fichiers |
   | **gateway** | Oui | <*installed-gateway-name*> | Le nom de la passerelle que vous avez installée précédemment |
   |||||

1. Lorsque vous êtes prêt, choisissez **Créer**.

   Logic Apps configure et teste votre connexion pour vérifier son bon fonctionnement. Si la connexion est configurée correctement, des options apparaissent pour l’action que vous avez sélectionnée précédemment.

1. Configurez l’action **Créer un fichier**, fournissez les détails pour la copie de fichiers de Dropbox vers le dossier racine de votre partage de fichiers local. Pour ajouter des résultats des étapes précédentes, cliquez dans les zones, puis effectuez la sélection dans les champs disponibles lorsque la liste de contenu dynamique s’affiche.

   ![Créer une action de fichier](media/logic-apps-using-file-connector/create-file-filled.png)

1. Maintenant, ajoutez une action Outlook qui envoie un e-mail, afin que les utilisateurs concernés soient informés du nouveau fichier. Saisissez les destinataires, l’objet et le corps de l’e-mail. Pour tester, vous pouvez utiliser votre propre adresse e-mail.

   ![Envoyer l’action de messagerie](media/logic-apps-using-file-connector/send-email.png)

1. Enregistrez votre application logique. Testez votre application en chargeant un fichier sur Dropbox.

   Votre application logique doit copier le fichier dans votre partage de fichiers en local et envoyer un e-mail aux destinataires pour les informer du fichier copié.

## <a name="connector-reference"></a>Référence de connecteur

Pour plus d’informations techniques sur ce connecteur, notamment au sujet des déclencheurs, des actions et des limites décrits dans le fichier Swagger du connecteur, consultez la [page de référence du connecteur](/connectors/fileconnector/).

> [!NOTE]
> Pour les applications logiques utilisées dans un [environnement de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la version de ce connecteur avec l’étiquette ISE applique les [limites de messages de l’ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits) à la place.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [vous connecter à des données locales](../logic-apps/logic-apps-gateway-connection.md) 
* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)
