---
title: Gérer des messages volumineux à l’aide d’une segmentation
description: Apprenez à gérer les messages volumineux en les segmentant en tâches et flux de travail automatisés que vous créez avec Azure Logic Apps
services: logic-apps
ms.suite: integration
author: DavidCBerry13
ms.author: daberry
ms.topic: article
ms.date: 12/03/2019
ms.openlocfilehash: 54828dded5196c86946d99a9cd8cec7a42533661
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83117561"
---
# <a name="handle-large-messages-with-chunking-in-azure-logic-apps"></a>Gérer les messages volumineux avec la segmentation dans Azure Logic Apps

Pour traiter les messages, Logic Apps limite leur contenu à une taille maximale. Cette limite permet de réduire la surcharge créée par le stockage et le traitement des messages volumineux. Pour gérer les messages dépassant cette limite de taille, Logic Apps peut *segmenter* un message volumineux en messages de plus petite taille. De cette façon, vous pouvez toujours transférer des fichiers volumineux à l’aide de Logic Apps dans des conditions spécifiques. Lors de la communication avec d’autres services via des connecteurs ou HTTP, Logic Apps peut traiter des messages volumineux mais *uniquement* sous forme de segments. Cette condition signifie que les connecteurs doivent également prendre en charge la segmentation, ou que le protocole d’échange de messages HTTP sous-jacent entre Logic Apps et ces services doit utiliser la segmentation.

Cet article explique comment configurer la segmentation des actions de gestion des messages qui dépassent la taille maximale autorisée. Les déclencheurs d’application logique ne prennent pas en charge la segmentation en raison de la surcharge liée à l’échange de nombreux messages. 

## <a name="what-makes-messages-large"></a>Qu’est-ce qui rend les messages « volumineux » ?

Les messages sont considérés comme « volumineux » selon le service qui les gère. La taille maximale réelle des messages volumineux varie selon les connecteurs et Logic Apps. Logic Apps et les différents connecteurs sont incapables de gérer directement les messages volumineux, qui doivent donc être segmentés. Pour connaître la taille maximale de message pour Logic Apps, consultez l’article [Limites et configuration de Logic Apps](../logic-apps/logic-apps-limits-and-config.md).
Pour connaître la taille maximale de message pour les différents connecteurs, consultez les [détails techniques spécifiques à chaque connecteur](../connectors/apis-list.md).

### <a name="chunked-message-handling-for-logic-apps"></a>Gestion des messages segmentés pour Logic Apps

Logic Apps ne peut pas utiliser directement les sorties provenant de messages segmentés dont la taille dépasse la limite autorisée. Seules les actions qui prennent en charge la segmentation peuvent accéder au contenu du message dans ces sorties. Par conséquent, une action qui gère les messages volumineux doit répondre à *l’un* de ces critères :

* Prendre en charge la segmentation de manière native lorsque cette action appartient à un connecteur. 
* Avoir la prise en charge de la segmentation activée dans la configuration d’exécution de cette action. 

Dans le cas contraire, vous obtenez une erreur d’exécution lorsque vous essayez d’accéder à un contenu volumineux. Pour activer la segmentation, consultez l’article [Set up chunking support](#set-up-chunking) (Configurer la prise en charge de la segmentation).

### <a name="chunked-message-handling-for-connectors"></a>Gestion des messages segmentés pour les connecteurs

Les services qui communiquent avec Logic Apps peuvent avoir leurs propres limites de taille de message. Ces limites sont souvent inférieures à la limite de Logic Apps. Par exemple, en supposant qu’un connecteur prend en charge la segmentation, celui-ci peut considérer un message de 30 Mo comme volumineux, alors que Logic Apps ne le considérera pas comme tel. Pour respecter la limite de ce connecteur, Logic Apps fractionne tout message supérieur à 30 Mo en segments plus petits.

Pour les connecteurs qui prennent en charge la segmentation, le protocole de segmentation sous-jacent n’est pas visible par les utilisateurs finaux. Toutefois, tous les connecteurs ne prennent pas en charge la segmentation. Ces connecteurs génèrent alors des erreurs d’exécution lorsque les messages entrants dépassent la taille maximale autorisée pour ces connecteurs.

> [!NOTE]
> Pour les actions qui utilisent la segmentation, vous ne pouvez pas passer le corps du déclencheur ou utiliser des expressions telles que `@triggerBody()?['Content']` dans ces actions. Au lieu de cela, pour le contenu de fichier texte ou JSON, vous pouvez essayer d’utiliser l’[action **Compose**](../logic-apps/logic-apps-perform-data-operations.md#compose-action) ou [créer une variable](../logic-apps/logic-apps-create-variables-store-values.md) pour gérer ce contenu. Si le corps du déclencheur contient d’autres types de contenu, tels que des fichiers multimédias, vous devez effectuer d’autres étapes pour gérer ce contenu.

<a name="set-up-chunking"></a>

## <a name="set-up-chunking-over-http"></a>Configurer la segmentation via HTTP

Dans les scénarios HTTP génériques, vous pouvez fractionner les téléchargements et chargements de contenu volumineux via HTTP, afin que votre application logique et un point de terminaison puissent échanger des messages volumineux. Toutefois, vous devez segmenter ces messages conformément à la limite autorisée par Logic Apps. 

Si un point de terminaison prend en charge la segmentation pour les téléchargements ou les chargements, les actions HTTP dans votre application logique segmenteront automatiquement les messages volumineux. Dans le cas contraire, vous devez configurer la prise en charge de la segmentation sur le point de terminaison. Si vous ne possédez ou ne contrôlez pas le point de terminaison ou le connecteur, vous ne serez peut-être pas en mesure de configurer la segmentation.

En outre, si une action HTTP ne prend pas encore en charge la segmentation, vous devez également configurer la segmentation dans la propriété `runTimeConfiguration` de l’action. Vous pouvez définir cette propriété à l’intérieur de l’action, soit directement dans l’éditeur en mode Code comme décrit plus loin, soit dans le Concepteur Logic Apps comme décrit ci-après :

1. Dans le coin supérieur droit de l’action HTTP, sélectionnez le bouton représentant des points de suspension ( **...** ), puis **Paramètres**.

   ![Dans l’action, ouvrez le menu Paramètres](./media/logic-apps-handle-large-messages/http-settings.png)

2. Sous **Transfert de contenu**, définissez **Autoriser la segmentation** sur **Activé**.

   ![Activer la segmentation](./media/logic-apps-handle-large-messages/set-up-chunking.png)

3. Pour poursuivre la configuration de la segmentation pour les téléchargements ou les chargements, consultez les sections suivantes.

<a name="download-chunks"></a>

## <a name="download-content-in-chunks"></a>Télécharger du contenu sous forme de segments

De nombreux points de terminaison envoient automatiquement les messages volumineux sous forme de segments lorsque ces derniers sont téléchargés via une requête HTTP GET. Pour télécharger des messages segmentés depuis un point de terminaison via HTTP, le point de terminaison doit prendre en charge les requêtes de contenu partiel, ou *téléchargements segmentés*. Lorsque votre application logique envoie une requête HTTP GET à un point de terminaison pour télécharger du contenu et que le point de terminaison répond avec un code d’état « 206 », la réponse contient le contenu segmenté. Logic Apps ne peut pas déterminer si un point de terminaison prend en charge les requêtes partielles. Toutefois, lorsque votre application logique reçoit la première réponse « 206 », votre application logique envoie automatiquement plusieurs requêtes pour télécharger l’ensemble du contenu.

Pour vérifier si un point de terminaison peut prendre en charge le contenu partiel, envoyez une requête HEAD. Cette requête vous permet de savoir si la réponse contient l’en-tête `Accept-Ranges`. De cette façon, si le point de terminaison prend en charge les téléchargements segmentés, mais n’envoie pas de contenu segmenté, vous pouvez *suggérer* cette option en définissant l’en-tête `Range` dans votre requête HTTP GET. 

Ces étapes décrivent le processus détaillé utilisé par Logic Apps pour télécharger le contenu segmenté d’un point de terminaison vers votre application logique :

1. Votre application logique envoie une requête HTTP GET au point de terminaison.

   L’en-tête de la requête peut éventuellement inclure un champ `Range` qui définit une plage d’octets pour la requête de segments de contenu.

2. Le point de terminaison renvoie le code d’état « 206 » et un corps de message HTTP.

    Les informations détaillées sur le contenu de ce segment s’affichent dans l’en-tête `Content-Range` de la réponse, notamment des informations qui permettent à Logic Apps d’identifier le début et la fin du segment, ainsi que la taille totale du contenu avant segmentation.

3. Votre application logique envoie automatiquement les requêtes HTTP GET suivantes.

    Votre application logique envoie des requêtes GET jusqu’à ce que l’ensemble du contenu soit récupéré.

Par exemple, cette définition d’action affiche une requête HTTP GET qui définit l’en-tête `Range`. L’en-tête *suggère* au point de terminaison de répondre en envoyant du contenu segmenté :

```json
"getAction": {
    "inputs": {
        "headers": {
            "Range": "bytes=0-1023"
        },
       "method": "GET",
       "uri": "http://myAPIendpoint/api/downloadContent"
    },
    "runAfter": {},
    "type": "Http"
}
```

La requête GET définit l’en-tête « Range » sur « bytes=0-1023 », qui correspond à la plage d’octets. Si le point de terminaison prend en charge les requêtes de contenu partiel, le point de terminaison répond en envoyant un segment de contenu dans la plage demandée. Le format exact du champ d’en-tête « Range » peut varier selon les points de terminaison.

<a name="upload-chunks"></a>

## <a name="upload-content-in-chunks"></a>Charger du contenu sous forme de segments

Pour charger du contenu segmenté à partir d’une action HTTP, la prise en charge de la segmentation doit être activée dans la propriété `runtimeConfiguration` de l’action. Ce paramètre permet à l’action de démarrer le protocole de segmentation. Votre logique application peut ensuite envoyer un message POST ou PUT initial au point de terminaison cible. Une fois que le point de terminaison répond en envoyant une suggestion de taille de segment, votre application logique poursuit le processus en envoyant des requêtes HTTP PATCH qui contiennent les segments de contenu.

Ces étapes décrivent le processus détaillé utilisé par Logic Apps pour charger le contenu segmenté de votre application logique vers un point de terminaison :

1. Votre application logique envoie une requête HTTP POST ou PUT initiale avec un corps de message vide. L’en-tête de la requête inclut les informations sur le contenu que votre application logique souhaite envoyer sous forme de segments :

   | Champ d’en-tête de la requête Logic Apps | Valeur | Type | Description |
   |---------------------------------|-------|------|-------------|
   | **x-ms-transfer-mode** | segmenté | String | Indique que le contenu est chargé sous forme de segments |
   | **x-ms-content-length** | <*content-length*> | Integer | La taille, en octets, de l’intégralité du contenu avant segmentation |
   ||||

2. Le point de terminaison répond avec le code d’état de réussite « 200 » et ces informations facultatives :

   | Champ d’en-tête de réponse de point de terminaison | Type | Obligatoire | Description |
   |--------------------------------|------|----------|-------------|
   | **x-ms-chunk-size** | Integer | Non | La taille de segment suggérée en octets |
   | **Lieu** | String | Oui | L’adresse URL vers laquelle envoyer les messages HTTP PATCH |
   ||||

3. Votre application logique crée et envoie des messages HTTP PATCH de suivi, chacun contenant les informations suivantes :

   * Un segment de contenu correspondant à la taille **x-ms-chunk-size** ou à une taille calculée en interne, jusqu’à ce que tout le contenu correspondant à la valeur **x-ms-content-length** soit chargé de manière séquentielle

   * Ces informations d’en-tête relatives au segment de contenu envoyé dans chaque message PATCH :

     | Champ d’en-tête de la requête Logic Apps | Valeur | Type | Description |
     |---------------------------------|-------|------|-------------|
     | **Content-Range** | <*range*> | String | La plage d’octets pour le segment de contenu actuel, y compris la valeur de début, la valeur de fin et la taille totale du contenu, par exemple : « bytes=0-1023/10100 » |
     | **Content-Type** | <*content-type*> | String | Le type de contenu segmenté |
     | **Content-Length** | <*content-length*> | String | La longueur, en octets, du segment actuel |
     |||||

4. Après chaque requête PATCH, le point de terminaison confirme la réception de chaque segment en renvoyant le code d’état « 200 » et les en-têtes de réponse suivants :

   | Champ d’en-tête de réponse de point de terminaison | Type | Obligatoire | Description |
   |--------------------------------|------|----------|-------------|
   | **Plage** | String | Oui | La plage d’octets pour le contenu qui a été reçu par le point de terminaison, par exemple: « octets = 0-1023 » |   
   | **x-ms-chunk-size** | Integer | Non | La taille de segment suggérée en octets |
   ||||

Par exemple, cette définition d’action affiche une requête HTTP POST pour le chargement de contenu segmenté vers un point de terminaison. Dans la propriété `runTimeConfiguration` de l’action, la propriété `contentTransfer` définit `transferMode` sur `chunked` :

```json
"postAction": {
    "runtimeConfiguration": {
        "contentTransfer": {
            "transferMode": "chunked"
        }
    },
    "inputs": {
        "method": "POST",
        "uri": "http://myAPIendpoint/api/action",
        "body": "@body('getAction')"
    },
    "runAfter": {
    "getAction": ["Succeeded"]
    },
    "type": "Http"
}
```
