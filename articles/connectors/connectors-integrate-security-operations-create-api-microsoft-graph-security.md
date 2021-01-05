---
title: Intégrer et gérer des opérations de sécurité et Microsoft Graph Security
description: Améliorer la protection de votre application contre les menaces et la façon dont elle détecte les menaces et y répond avec Microsoft Graph Security et Azure Logic Apps
services: logic-apps
ms.suite: integration
author: ecfan
ms.author: preetikr
ms.reviewer: v-ching, estfan, logicappspm
ms.topic: article
ms.date: 02/21/2020
tags: connectors
ms.openlocfilehash: a83cd68df2f1d722517d6239bf6959075860d0b8
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94888536"
---
# <a name="improve-threat-protection-by-integrating-security-operations-with-microsoft-graph-security--azure-logic-apps"></a>Améliorer la protection contre les menaces en intégrant des opérations de sécurité avec Microsoft Graph Security et Azure Logic Apps

Avec [Azure Logic Apps](../logic-apps/logic-apps-overview.md) et le connecteur [Microsoft Graph Security](/graph/security-concept-overview), vous pouvez améliorer la façon dont votre application détecte les menaces, y répond et vous protège contre celles-ci en créant des workflows automatisés pour intégrer des partenaires, des services et des produits de sécurité Microsoft. Par exemple, vous pouvez créer des [playbooks Azure Security Center](../security-center/workflow-automation.md) qui supervisent et gèrent des entités Microsoft Graph Security, telles que des alertes. Voici quelques scénarios pris en charge par le connecteur Microsoft Graph Security :

* Obtenir des alertes basées sur des requêtes ou par ID d’alerte. Par exemple, vous pouvez obtenir une liste qui inclut des alertes d’un niveau de gravité élevé.

* Mettre à jour des alertes. Par exemple, vous pouvez mettre à jour des affectations d’alerte, ajouter des commentaires aux alertes ou étiqueter des alertes.

* Superviser quand les alertes sont générées ou modifiées en créant des [abonnements d’alerte (webhooks)](/graph/api/resources/webhooks).

* Gérer vos abonnements d’alerte. Par exemple, vous pouvez obtenir des abonnements actifs, étendre le délai d’expiration pour un abonnement ou supprimer des abonnements.

Le workflow de votre application logique peut utiliser des actions visant à obtenir des réponses du connecteur Microsoft Graph Security et à rendre cette sortie disponible pour d’autres actions de votre workflow. Il est également possible que d’autres actions de votre workflow utilisent la sortie des actions du connecteur Microsoft Graph Security. Par exemple, si vous obtenez des alertes d’un niveau de gravité élevé par le biais du connecteur Microsoft Graph Security, vous pouvez envoyer ces alertes dans un e-mail à l’aide du connecteur Outlook. 

Pour en savoir plus sur Microsoft Graph Security, consultez la [vue d’ensemble de l’API Microsoft Graph Security](/graph/security-concept-overview). Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md). Si vous recherchez Power Automate ou PowerApps, consultez [Présentation de Power Automate](https://flow.microsoft.com/) ou [Présentation de Power Apps](https://powerapps.microsoft.com/).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/). 

* Pour utiliser le connecteur Microsoft Graph Security, vous devez avoir *donné explicitement* votre consentement à l’administrateur du locataire Azure Active Directory (AD), ce qui fait partie des [exigences d’authentification Microsoft Graph Security](/graph/security-authorization). Ce consentement nécessite le nom et l’ID d’application du connecteur Microsoft Graph Security, que vous pouvez également trouver dans le [portail Azure](https://portal.azure.com) :

  | Propriété | Valeur |
  |----------|-------|
  | **Nom d’application** | `MicrosoftGraphSecurityConnector` |
  | **ID d’application** | `c4829704-0edc-4c3d-a347-7c4a67586f3c` |
  |||

  Afin de donner le consentement pour le connecteur, l’administrateur de locataire Azure AD peut suivre l’une de ces procédures :

  * [Accorder à l’administrateur de locataire son consentement pour les applications Azure AD](../active-directory/develop/v2-permissions-and-consent.md).

  * Au cours de la première exécution de l’application logique, l’application peut demander le consentement de l’administrateur de locataire Azure AD via l’[expérience du consentement d’application](../active-directory/develop/application-consent-experience.md).
   
* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* Application logique où vous souhaitez accéder aux entités Microsoft Graph Security, telles que des alertes. Pour utiliser un déclencheur Microsoft Graph Security, vous avez besoin d’une application logique vide. Pour utiliser une action Microsoft Graph Security, vous avez besoin d’une application logique qui démarre avec le déclencheur approprié pour votre scénario.

## <a name="connect-to-microsoft-graph-security"></a>Se connecter à Microsoft Graph Security 

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Connectez-vous au [portail Azure](https://portal.azure.com/) et ouvrez votre application logique dans le concepteur d’application logique, si elle n’est pas déjà ouverte.

1. Pour les applications logiques vides, ajoutez le déclencheur et toute autre action souhaitée avant d’ajouter une action Microsoft Graph Security.

   -ou-

   Pour les applications logiques existantes, sous la dernière étape où vous souhaitez ajouter une action Microsoft Graph Security, sélectionnez **Nouvelle étape**.

   -ou-

   Pour ajouter une action entre des étapes, placez votre pointeur au-dessus de la flèche qui les sépare. Sélectionnez le signe plus (+) qui s’affiche, puis sélectionnez **Ajouter une action**.

1. Dans la zone de recherche, entrez « microsoft graph security » comme filtre. Dans la liste des actions, sélectionnez l’action souhaitée.

1. Connectez-vous avec vos informations d’identification Microsoft Graph Security.

1. Fournissez les informations nécessaires pour votre action sélectionné et continuez à générer le flux de travail de votre application logique.

## <a name="add-triggers"></a>Ajouter des déclencheurs

Dans Azure Logic Apps, chaque application logique doit démarrer avec un [déclencheur](../logic-apps/logic-apps-overview.md#logic-app-concepts), qui s’active lorsqu’un événement spécifique se produit ou lorsqu’une condition particulière est remplie. Chaque fois que le déclencheur s’active, le moteur Logic Apps crée une instance d’application logique et lance l’exécution du flux de travail de votre application.

> [!NOTE] 
> Lorsqu’un déclencheur est activé, il traite toutes les nouvelles alertes. Si aucune alerte n’est reçue, l’exécution du déclencheur est ignorée. La prochaine interrogation de déclencheur est basée sur l’intervalle de récurrence que vous spécifiez dans les propriétés du déclencheur.

Cet exemple montre comment vous pouvez démarrer un flux de travail d’application logique lorsque de nouvelles alertes sont envoyées à votre application.

1.  Dans le portail Azure ou Visual Studio, créez une application logique vide, qui ouvre le Concepteur d’applications logiques. Cet exemple utilise le portail Azure.

1.  Dans la zone de recherche du concepteur, entrez « microsoft graph security » comme filtre. Dans la liste des déclencheurs, sélectionnez ce déclencheur : **Sur toutes les nouvelles alertes**

1.  Dans le déclencheur, fournissez des informations sur les alertes que vous souhaitez surveiller. Pour plus de propriétés, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez un paramètre afin d'ajouter cette propriété au déclencheur.

   | Propriété | Propriété (JSON) | Obligatoire | Type | Description |
   |----------|-----------------|----------|------|-------------|
   | **Intervalle** | `interval` | Oui | Integer | Nombre entier positif qui décrit la fréquence à laquelle le flux de travail s’exécute en fonction de la fréquence. Les intervalles minimaux et maximaux sont les suivants : <p><p>- Mois : 1-16 mois <br>Jour : 1-500 jours <br>- Heure : 1-12 000 heures <br>- Minute : 1-72 000 minutes <br>- Seconde : 1-9 999 999 secondes <p>Par exemple, si l’intervalle est de 6 et que la fréquence soit définie sur « Mois », la périodicité est alors tous les 6 mois. |
   | **Fréquence** | `frequency` | Oui | String | Unité de temps à utiliser pour la récurrence : **Seconde**, **Minute**, **Heure**, **Jour**, **Semaine** ou **Mois** |
   | **Fuseau horaire** | `timeZone` | Non | String | S’applique uniquement quand vous spécifiez une heure de début, car ce déclencheur n’accepte pas le [décalage UTC](https://en.wikipedia.org/wiki/UTC_offset). Sélectionnez le fuseau horaire à appliquer. |
   | **Heure de début** | `startTime` | Non | String | Indiquez une date et une heure de début au format suivant : <p><p>AAAA-MM-JJThh:mm:ss si vous sélectionnez un fuseau horaire <p>-ou- <p>AAAA-MM-JJThh:mm:ssZ si vous ne sélectionnez pas de fuseau horaire <p>Par exemple, si vous choisissez le 18 septembre 2017 à 14h00, alors spécifiez « 2017-09-18T14:00:00 » et sélectionnez un fuseau horaire tel que Heure standard du Pacifique. Vous pouvez également spécifier « 2017-09-18T14:00:00Z » sans fuseau horaire. <p>**Remarque :** Cette heure de début ne peut pas dépasser 49 ans dans le futur, et doit être conforme à la [spécification de date/heure ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations) au [format de date/heure UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time), mais sans [décalage UTC](https://en.wikipedia.org/wiki/UTC_offset). Si vous ne sélectionnez pas de fuseau horaire, vous devez ajouter la lettre « Z » à la fin sans espace. Ce « Z » fait référence au [temps nautique](https://en.wikipedia.org/wiki/Nautical_time) équivalent. <p>Pour les planifications simples, l’heure de début est la première occurrence, tandis que pour les planifications complexes, le déclencheur ne s’active pas avant l’heure de début. [*Comment puis-je utiliser la date et l’heure de début ?*](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md#start-time) |
   ||||||

1.  Lorsque c’est chose faite, dans la barre d’outils du concepteur, sélectionnez **Enregistrer**.

1.  Continuez maintenant à ajouter une ou plusieurs actions à votre application logique pour les tâches à effectuer avec les résultats du déclencheur.

## <a name="add-actions"></a>Ajouter des actions

Voici des détails plus spécifiques sur l’utilisation des différentes actions disponibles avec le connecteur Microsoft Graph Security.

### <a name="manage-alerts"></a>Gérer les alertes

Pour filtrer, trier ou obtenir les résultats les plus récents, fournissez *uniquement* les [paramètres de requête ODATA pris en charge par Microsoft Graph](/graph/query-parameters). *Ne spécifiez pas* l’URL de base complète ni l’action HTTP, par exemple `https://graph.microsoft.com/v1.0/security/alerts`, ni l’opération `GET` ou `PATCH`. Voici un exemple spécifique qui montre les paramètres pour une action **Obtenir des alertes** quand vous voulez une liste avec des alertes d’un niveau de gravité élevé :

`Filter alerts value as Severity eq 'high'`

Pour plus d’informations sur les requêtes que vous pouvez utiliser avec ce connecteur, consultez la [documentation de référence des alertes Microsoft Graph Security](/graph/api/alert-list). Pour générer des expériences améliorées avec ce connecteur, découvrez plus en détail les [alertes des propriétés de schéma](/graph/api/resources/alert) prises en charge par le connecteur.

| Action | Description |
|--------|-------------|
| **Obtenir des alertes** | Obtenez des alertes filtrées sur une ou plusieurs [propriétés d’alerte](/graph/api/resources/alert), par exemple, `Provider eq 'Azure Security Center' or 'Palo Alto Networks'`. | 
| **Obtenir une alerte par ID** | Obtenez une alerte spécifique basée sur l’ID d’alerte. | 
| **Mettre à jour une alerte** | Mettez à jour une alerte spécifique basée sur l’ID d’alerte. Pour vous assurer que vous passez les propriétés obligatoires et modifiables dans votre demande, consultez les [propriétés modifiables pour les alertes](/graph/api/alert-update). Par exemple, pour affecter une alerte à un analyste de la sécurité afin qu’il puisse enquêter, vous pouvez mettre à jour la propriété **Affecté à** de l’alerte. |
|||

### <a name="manage-alert-subscriptions"></a>Gérer les abonnements d’alerte

Microsoft Graph prend en charge les [*abonnements*](/graph/api/resources/subscription), ou [*webhooks*](/graph/api/resources/webhooks). Pour obtenir, mettre à jour ou supprimer des abonnements, fournissez les [paramètres de requête ODATA pris en charge par Microsoft Graph](/graph/query-parameters) à la construction d’entité Microsoft Graph et ajoutez `security/alerts`, puis la requête ODATA. *N’incluez pas* l’URL de base, par exemple `https://graph.microsoft.com/v1.0`. Au lieu de cela, utilisez le format dans cet exemple :

`security/alerts?$filter=status eq 'NewAlert'`

| Action | Description |
|--------|-------------|
| **Création d’abonnements** | [Créez un abonnement](/graph/api/subscription-post-subscriptions) qui vous informe de toutes les modifications. Vous pouvez filtrer cet abonnement selon les types d’alertes spécifiques souhaités. Par exemple, vous pouvez créer un abonnement qui vous informe des alertes d’un niveau de gravité élevé. |
| **Obtenir des abonnements actifs** | [Obtenez les abonnements non expirés](/graph/api/subscription-list). | 
| **Mettre à jour un abonnement** | [Mettez à jour un abonnement](/graph/api/subscription-update) en indiquant l’ID d’abonnement. Par exemple, pour étendre votre abonnement, vous pouvez mettre à jour la propriété `expirationDateTime` de l’abonnement. | 
| **Supprimer un abonnement** | [Supprimez un abonnement](/graph/api/subscription-delete) en indiquant l’ID d’abonnement. | 
||| 

### <a name="manage-threat-intelligence-indicators"></a>Gérer les indicateurs Threat Intelligence

Pour filtrer, trier ou obtenir les résultats les plus récents, fournissez *uniquement* les [paramètres de requête ODATA pris en charge par Microsoft Graph](/graph/query-parameters). *Ne spécifiez pas* l’URL de base complète ni l’action HTTP, par exemple `https://graph.microsoft.com/beta/security/tiIndicators`, ni l’opération `GET` ou `PATCH`. Voici un exemple qui montre les paramètres pour une action **Get tiIndicators** quand vous voulez obtenir une liste avec le type de menace `DDoS` :

`Filter threat intelligence indicator value as threatType eq 'DDoS'`

Pour plus d’informations sur les requêtes utilisables avec ce connecteur, consultez [« Paramètres de requête facultatifs » dans la documentation de référence sur les indicateurs Threat Intelligence dans Microsoft Graph Security](/graph/api/tiindicators-list). Pour créer des expériences améliorées avec ce connecteur, découvrez plus en détail les [indicateurs Threat Intelligence des propriétés de schéma](/graph/api/resources/tiindicator) que le connecteur prend en charge.

| Action | Description |
|--------|-------------|
| **Obtenir les indicateurs Threat Intelligence** | Obtenez les tiIndicators filtrées sur une ou plusieurs [propriétés tiIndicator](/graph/api/resources/tiindicator), par exemple, `threatType eq 'MaliciousUrl' or 'DDoS'` |
| **Obtenir un indicateur Threat Intelligence par ID** | Obtenez un tiIndicator spécifique en fonction de l’ID de tiIndicator. | 
| **Créer un indicateur Threat Intelligence** | Créez un tiIndicator en publiant dans la collection tiIndicators. Pour être sûr de passer les propriétés obligatoires dans votre requête, reportez-vous à la section sur les [propriétés requises pour créer un tiIndicator](/graph/api/tiindicators-post). |
| **Envoyer plusieurs indicateurs Threat Intelligence** | Créez plusieurs tiIndicators en publiant dans une collection tiIndicators. Pour être sûr de passer les propriétés obligatoires dans votre requête, reportez-vous à la section sur les [propriétés requises pour envoyer plusieurs tiIndicators](/graph/api/tiindicator-submittiindicators). |
| **Mettre à jour un indicateur Threat Intelligence** | Mettez à jour un tiIndicator spécifique en fonction de l’ID de tiIndicator. Pour être sûr de passer les propriétés obligatoires et modifiables dans votre requête, consultez les [propriétés modifiables pour les tiIndicators](/graph/api/tiindicator-update). Par exemple, pour mettre à jour l’action à appliquer si l’indicateur est mis en correspondance à partir de l’outil de sécurité targetProduct, vous pouvez mettre à jour la propriété **action** du tiIndicator. |
| **Mettre à jour plusieurs indicateurs Threat Intelligence** | Mettez à jour plusieurs tiIndicators. Pour être sûr de passer les propriétés obligatoires dans votre requête, reportez-vous à la section sur les [propriétés requises pour mettre à jour plusieurs tiIndicators](/graph/api/tiindicator-updatetiindicators). |
| **Supprimer un indicateur Threat Intelligence par ID** | Supprimez un tiIndicator spécifique en fonction de l’ID de tiIndicator. |
| **Supprimer plusieurs indicateurs Threat Intelligence par ID** | Supprimez plusieurs tiIndicators en fonction de leur ID. Pour être sûr de passer les propriétés obligatoires dans votre requête, reportez-vous à la section sur les [propriétés requises pour supprimer plusieurs tiIndicators par ID](/graph/api/tiindicator-deletetiindicators). |
| **Supprimer plusieurs indicateurs Threat Intelligence par ID externe** | Supprimez plusieurs tiIndicators en fonction de leur ID externe. Pour être sûr de passer les propriétés obligatoires dans votre requête, reportez-vous à la section sur les [propriétés requises pour supprimer plusieurs tiIndicators par ID externe](/graph/api/tiindicator-deletetiindicatorsbyexternalid). |
|||

## <a name="connector-reference"></a>Référence de connecteur

Pour obtenir des détails techniques sur les déclencheurs, les actions et les limites, qui sont décrits par la description OpenAPI du connecteur (anciennement Swagger), consultez la [page de référence](/connectors/microsoftgraphsecurity/) du connecteur.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)
