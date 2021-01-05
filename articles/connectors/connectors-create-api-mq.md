---
title: Se connecter à un serveur IBM MQ
description: Envoyer et récupérer des messages avec un serveur Azure ou un serveur IBM MQ local et Azure Logic Apps
services: logic-apps
ms.suite: integration
author: ChristopherHouser
ms.author: chrishou
ms.reviewer: valthom, estfan, logicappspm
ms.topic: article
ms.date: 05/14/2020
tags: connectors
ms.openlocfilehash: e9e554fdc092e49f5a87049de0e3dc3163105f58
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85609501"
---
# <a name="connect-to-an-ibm-mq-server-from-azure-logic-apps"></a>Se connecter à un serveur IBM MQ depuis Azure Logic Apps

Le connecteur IBM MQ envoie et récupère les messages stockés dans un serveur IBM MQ local ou dans Azure. Ce connecteur inclut un client Microsoft MQ qui communique avec un serveur IBM MQ distant sur un réseau TCP/IP. Cet article offre un guide de démarrage pour l’utilisation du connecteur MQ. Vous pouvez commencer par parcourir un message dans une file d’attente, avant de tenter les autres actions.

Le connecteur IBM MQ inclut ces actions, mais ne fournit aucun déclencheur :

- Parcourez un seul message sans le supprimer du serveur IBM MQ.
- Parcourez un lot de messages sans supprimer ceux-ci du serveur IBM MQ.
- Recevez un message unique et supprimez le message à partir du serveur IBM MQ.
- Recevez un lot de messages et supprimez les messages du serveur IBM MQ.
- Envoyez un message unique au serveur IBM MQ.

Voici les versions d’IBM WebSphere MQ officiellement prises en charge :

  * MQ 7.5
  * MQ 8.0
  * MQ 9.0
  * MQ 9.1

## <a name="prerequisites"></a>Prérequis

* Si vous utilisez un serveur MQ local, [installez la passerelle de données locale](../logic-apps/logic-apps-gateway-install.md) sur un serveur au sein de votre réseau. Pour que le connecteur MQ fonctionne, .NET Framework 4.6 doit également être installé sur le serveur sur lequel la passerelle de données locale est installée.

  Une fois que vous avez terminé l’installation de la passerelle, vous devez également créer une ressource dans Azure pour la passerelle de données locale. Pour plus d’informations, consultez [Configurer la connexion à la passerelle de données](../logic-apps/logic-apps-gateway-connection.md).

  Si votre serveur MQ est disponible publiquement ou dans Azure, vous n’avez pas à utiliser la passerelle de données.

* L’application logique dans laquelle vous souhaitez ajouter l’action MQ. Cette application logique doit utiliser le même emplacement que la connexion de votre passerelle de données locale et vous devez avoir un déclencheur qui démarre votre workflow.

  Le connecteur MQ ne possède aucun déclencheur. Vous devez donc d’abord en ajouter un à votre application logique. Par exemple, vous pouvez utiliser le déclencheur Recurrence. Si vous débutez avec les applications logiques, essayez ce [Démarrage rapide pour créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

<a name="create-connection"></a>

## <a name="create-mq-connection"></a>Créer une connexion MQ

Si vous n’avez pas encore de connexion à MQ lorsque vous ajoutez une action MQ, vous êtes invité à la créer, par exemple :

![Fournir les informations de connexion](media/connectors-create-api-mq/connection-properties.png)

1. Si vous vous connectez à un serveur MQ local, sélectionnez **Se connecter via une passerelle de données locale**.

1. Fournissez les informations de connexion pour votre serveur MQ.

   * Pour **Serveur**, vous pouvez entrer le nom du serveur MQ, ou l’adresse IP suivie par un signe deux-points et le numéro de port.

   * Pour utiliser le protocole SSL, sélectionnez **Activer SSL ?** .

     Actuellement, le connecteur MQ prend uniquement en charge l’authentification serveur, et pas l’authentification client. Pour plus d’informations, consultez [Problèmes de connexion et d’authentification](#connection-problems).

1. Dans la section **Passerelle**, procédez comme suit :

   1. Dans la liste **Abonnement**, sélectionnez l’abonnement Azure associé à votre ressource de passerelle Azure.

   1. Dans la liste **Passerelle de connexion**, sélectionnez la ressource de passerelle Azure que vous souhaitez utiliser.

1. Sélectionnez **Créer** lorsque vous avez terminé.

<a name="connection-problems"></a>

### <a name="connection-and-authentication-problems"></a>Problèmes de connexion et d’authentification

Lorsque votre application logique tente de se connecter à votre serveur MQ local, vous pourriez recevoir cette erreur :

`"MQ: Could not Connect the Queue Manager '<queue-manager-name>': The Server was expecting an SSL connection."`

* Si vous utilisez le connecteur MQ directement dans Azure, le serveur MQ doit utiliser un certificat émis par une [autorité de certification](https://www.ssl.com/faqs/what-is-a-certificate-authority/) approuvée.

* Si vous utilisez la passerelle de données locale, essayez d’utiliser un certificat émis par une [autorité de certification](https://www.ssl.com/faqs/what-is-a-certificate-authority/) approuvée dans la mesure du possible. Toutefois, si cette option n’est pas possible, vous pouvez utiliser un certificat auto-signé, qui n’est pas émis par une [autorité de certification](https://www.ssl.com/faqs/what-is-a-certificate-authority/) approuvée et est considéré comme moins sécurisé.

  Pour installer le certificat auto-signé du serveur, vous pouvez utiliser l’outil **Windows Certification Manager** (Certmgr.msc). Pour ce scénario, sur votre ordinateur local sur lequel le service de passerelle de données locale est en cours d’exécution, vous devez installer le certificat dans votre magasin de certificats **Ordinateur local** au niveau **Autorités de certification racines de confiance**.

  1. Sur l’ordinateur exécutant le service de passerelle de données locale, ouvrez le menu Démarrer, puis recherchez et sélectionnez **Gérer les certificats utilisateur**.

  1. Après l’ouverture de l’outil Windows Certification Manager, accédez au dossier **Certificats - Ordinateur local** >  **Autorités de certification racines de confiance**, puis installez le certificat.

     > [!IMPORTANT]
     > Veillez à installer le certificat dans le magasin **Certificats - Ordinateur local** > **Autorités de certification racines de confiance**.

* Le serveur MQ vous demande de définir la spécification de chiffrement que vous souhaitez utiliser pour les connexions TLS/SSL. Toutefois, SslStream dans .NET ne vous permet pas de spécifier l’ordre des spécifications de chiffrement. Pour contourner cette limitation, vous pouvez modifier la configuration de votre serveur MQ afin qu’elle corresponde à la première spécification de chiffrement de la suite que le connecteur envoie dans la négociation TLS/SSL.

  Lorsque vous essayez la connexion, le serveur MQ enregistre un message d’événement qui indique que la connexion a échoué, car l’autre terminaison a utilisé la mauvaise spécification de chiffrement. Le message d’événement contient la spécification de chiffrement qui apparaît en premier dans la liste. Mettez à jour la spécification de chiffrement dans la configuration du canal pour qu’elle corresponde à la spécification de chiffrement du message d’événement.

## <a name="browse-single-message"></a>Parcourir un seul message

1. Dans votre application logique, sous le déclencheur ou une autre action, sélectionnez **Nouvelle étape**.

1. Dans la zone de recherche, saisissez `mq`, puis sélectionnez l’action **Parcourir le message**.

   ![Sélectionner l’action « Parcourir le message »](media/connectors-create-api-mq/browse-message.png)

1. Si vous n’avez pas encore créé de connexion MQ, vous êtes invité à [créer cette connexion](#create-connection).

1. Après avoir créé la connexion, configurez les propriétés de l’action **Parcourir le message** :

   | Propriété | Description |
   |----------|-------------|
   | **File d'attente** | Si la file d'attente est différente de celle spécifiée dans la connexion, spécifiez-la. |
   | **MessageId**, **CorrelationId**, **GroupId** et d’autres propriétés | Recherchez un message basé sur les différentes propriétés de message MQ |
   | **IncludeInfo** | Sélectionnez **true** pour inclure des informations supplémentaires dans la sortie. Pour omettre les informations supplémentaires sur les messages dans la sortie, sélectionnez **false**. |
   | **Délai d'expiration** | Entrer une valeur pour déterminer la durée d’attente de l’arrivée d’un message dans une file d’attente vide. Si aucune valeur n’est entrée, le premier message dans la file d’attente est récupéré et aucun temps n’est consacré à l’attente de l’affichage d’un message. |
   |||

   Par exemple :

   ![Propriétés de l’action « Parcourir le message »](media/connectors-create-api-mq/browse-message-properties.png)

1. Lorsque c’est chose faite, dans la barre d’outils du concepteur, sélectionnez **Enregistrer**. Pour tester votre application, sélectionnez **Exécuter**.

   Une fois l’exécution terminée, le concepteur affiche les étapes du flux de travail et leur état afin que vous puissiez examiner la sortie.

1. Pour afficher les détails de chaque étape, cliquez sur la barre de titre de l’étape. Pour consulter plus d’informations sur la sortie d’une étape, sélectionnez **Afficher les sorties brutes**.

   ![Parcourir une sortie de message](media/connectors-create-api-mq/browse-message-output.png)

   Voici un exemple de sortie brute :

   ![Parcourir une sortie brute de message](media/connectors-create-api-mq/browse-message-raw-output.png)

1. Si vous définissez **IncludeInfo** sur **true**, une sortie supplémentaire s’affiche :

   ![Parcourir les informations include d’un message](media/connectors-create-api-mq/browse-message-include-info.png)

## <a name="browse-multiple-messages"></a>Parcourir plusieurs messages

L’action **Parcourir les messages** inclut une option **BatchSize** permettant d’indiquer le nombre de messages à retourner à partir de la file d’attente. Si l’option **BatchSize** ne comporte aucune valeur, tous les messages sont retournés. La sortie retournée est un tableau de messages.

1. Suivez les étapes précédentes, mais ajoutez l’action **Parcourir les messages** à la place.

1. Si vous n’avez pas encore créé de connexion MQ, vous êtes invité à [créer cette connexion](#create-connection). Sinon, par défaut, la première connexion précédemment configurée est utilisée. Pour créer une nouvelle connexion, sélectionnez **Modifier la connexion**. Ou sélectionnez une autre connexion.

1. Entrez les informations pour l’action.

1. Enregistrez et exécutez l’application logique.

   Lorsque l’exécution de l’application logique se termine, voici un exemple de sortie de l’action **Parcourir les messages** :

   ![Exemple de sortie « Parcourir les messages »](media/connectors-create-api-mq/browse-messages-output.png)

## <a name="receive-single-message"></a>Recevoir un seul message

L’action **Recevoir un message** a les mêmes entrées et sorties que l’action **Parcourir un message**. Lorsque vous utilisez l’action **Recevoir un message**, le message est supprimé de la file d’attente.

## <a name="receive-multiple-messages"></a>Recevoir plusieurs messages

L’action **Recevoir des messages** a les mêmes entrées et sorties que l’action **Parcourir des messages**. Lorsque vous utilisez l’action **Recevoir des messages**, les messages sont supprimés de la file d’attente.

> [!NOTE]
> Lors de l’exécution d’une action de navigation ou de réception sur une file d’attente qui n’a pas de messages, l’action échoue avec la sortie suivante :
>
> ![Erreur MQ « Aucun message »](media/connectors-create-api-mq/mq-no-message-error.png)

## <a name="send-message"></a>Envoyer un message

1. Suivez les étapes précédentes, mais ajoutez à la place l’action **Envoyer un message**.

1. Si vous n’avez pas encore créé de connexion MQ, vous êtes invité à [créer cette connexion](#create-connection). Sinon, par défaut, la première connexion précédemment configurée est utilisée. Pour créer une nouvelle connexion, sélectionnez **Modifier la connexion**. Ou sélectionnez une autre connexion.

1. Entrez les informations pour l’action. Pour **MessageType**, sélectionnez un type de message valide : **Datagramme**, **Réponse**, ou **Demande**

   ![Propriétés de l’action « Envoyer un message »](media/connectors-create-api-mq/send-message-properties.png)

1. Enregistrez et exécutez l’application logique.

   Lorsque l’exécution de l’application logique se termine, voici un exemple de sortie de l’action **Envoyer des messages** :

   ![Exemple de sortie « Envoyer un message »](media/connectors-create-api-mq/send-message-output.png)

## <a name="connector-reference"></a>Référence de connecteur

Pour obtenir des détails techniques, les actions et les limites, qui sont fournis par la description OpenAPI du connecteur (anciennement Swagger), consultez la [page de référence du connecteur](/connectors/mq/).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)
