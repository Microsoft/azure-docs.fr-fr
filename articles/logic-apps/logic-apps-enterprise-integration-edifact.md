---
title: Messages EDIFACT pour l’intégration B2B
description: Échanger des messages EDIFACT au format EDI dans le cadre d’une intégration d’entreprise B2B dans Azure Logic Apps avec Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 04/22/2020
ms.openlocfilehash: b0df55e59bd519a816c4022f2434edfcd4460780
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006521"
---
# <a name="exchange-edifact-messages-for-b2b-enterprise-integration-in-azure-logic-apps-with-enterprise-integration-pack"></a>Échanger des messages EDIFACT dans le cadre d’une intégration d’entreprise B2B dans Azure Logic Apps avec Enterprise Integration Pack

Avant de pouvoir échanger des messages EDIFACT pour Azure Logic Apps, vous devez créer un contrat EDIFACT et le stocker dans votre compte d’intégration. Voici les étapes de création d’un contrat EDIFACT.

> [!NOTE]
> Cette page traite des fonctionnalités EDIFACT d’Azure Logic Apps. Pour plus d’informations, consultez [X12](logic-apps-enterprise-integration-x12.md).

## <a name="before-you-start"></a>Avant de commencer

Voici les éléments dont vous avez besoin :

* Un [compte d’intégration](logic-apps-enterprise-integration-create-integration-account.md) déjà défini et associé à votre abonnement Azure  
* Au moins deux [partenaires](logic-apps-enterprise-integration-partners.md) déjà définis dans votre compte d’intégration

> [!NOTE]
> Lorsque vous créez un contrat, le contenu des messages que vous échangez avec le partenaire doit correspondre au type de contrat.

Une fois que vous avez [créé un compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) et [ajouté des partenaires](logic-apps-enterprise-integration-partners.md), vous pouvez créer un contrat EDIFACT en procédant comme suit.

## <a name="create-an-edifact-agreement"></a>Créer un contrat EDIFACT 

1. Connectez-vous au [portail Azure](https://portal.azure.com "Portail Azure"). 

2. Dans le menu principal Azure, sélectionnez **Tous les services**. Dans la zone de recherche, entrez « intégration », puis sélectionnez **Comptes d’intégration**.

   ![Recherche du compte d’intégration](./media/logic-apps-enterprise-integration-edifact/edifact-0.png)

   > [!TIP]
   > Si l’option **Tous les services** n’est pas visible, vous devez d’abord peut-être développer le menu. En haut du menu réduit, sélectionnez **Afficher les étiquettes de texte**.

3. Sous **Comptes d’intégration**, sélectionnez le compte d’intégration dans lequel vous voulez créer le contrat.

   ![Sélectionnez le compte d’intégration dans lequel vous souhaitez créer le contrat](./media/logic-apps-enterprise-integration-edifact/edifact-1-4.png)

4. Choisissez **Contrats**. Si vous ne voyez pas la mosaïque Contrats, commencez par ajouter la mosaïque.   

   ![Choisissez la mosaïque « Contrats »](./media/logic-apps-enterprise-integration-edifact/edifact-1-5.png)

5. Dans la page Contrats, choisissez **Ajouter**.

   ![Choisir « Ajouter »](./media/logic-apps-enterprise-integration-edifact/edifact-agreement-2.png)

6. Sous **Ajouter**, entrez le **nom** de votre contrat. Pour le **type de contrat**, sélectionnez **EDIFACT**. Sélectionnez le **partenaire hôte**, **l’identité de l’hôte**, le **partenaire invité** et **l’identité de l’invité** pour votre contrat.

   ![Renseigner les détails relatifs au contrat](./media/logic-apps-enterprise-integration-edifact/edifact-1.png)

   | Propriété | Description |
   | --- | --- |
   | Nom |Nom du contrat. |
   | Type de contrat | Doit être EDIFACT |
   | Partenaire hôte |Un contrat nécessite un partenaire hôte et un partenaire invité. Le partenaire hôte représente l’organisation qui configure le contrat. |
   | Identité de l’hôte |Identificateur du partenaire hôte. |
   | Partenaire invité |Un contrat nécessite un partenaire hôte et un partenaire invité. Le partenaire invité représente l’organisation qui travaille avec le partenaire hôte. |
   | identité de l’invité |Identificateur du partenaire invité. |
   | Paramètres de réception |Ces propriétés s’appliquent à tous les messages reçus par un contrat. |
   | Paramètres d’envoi |Ces propriétés s’appliquent à tous les messages envoyés par un contrat. |
   ||| 

## <a name="configure-how-your-agreement-handles-received-messages"></a>Configuration du traitement des messages reçus

Maintenant que vous avez défini les propriétés du contrat, vous pouvez configurer la manière dont ce contrat identifie et traite les messages entrants reçus par votre partenaire par l’intermédiaire de ce contrat.

> [!IMPORTANT]
> Le connecteur EDIFACT prend en charge uniquement les caractères UTF-8.
> Si votre sortie contient des caractères inattendus, vérifiez que vos messages EDIFACT utilisent le jeu de caractères UTF-8.

1. Sous **Ajouter**, sélectionnez **Paramètres de réception**.
Configurez ces propriétés selon le contrat conclu avec le partenaire qui échange des messages avec vous. Pour obtenir les descriptions des propriétés, consultez les tableaux de cette section.

   Les **paramètres de réception** sont organisés en plusieurs sections : identificateurs, accusé de réception, schémas, numéros de contrôle, validations et paramètres internes.

   ![Configurer les « Paramètres de réception »](./media/logic-apps-enterprise-integration-edifact/edifact-2.png)  

2. Une fois que vous avez terminé, cliquez sur **OK** pour enregistrer vos paramètres.

Votre contrat est maintenant prêt à traiter les messages entrants qui sont conformes aux paramètres que vous avez sélectionnés.

### <a name="identifiers"></a>Identificateurs

| Propriété | Description |
| --- | --- |
| UNB6.1 (mot de passe de référence du destinataire) |Entrez une valeur alphanumérique comprise entre 1 et 14 caractères. |
| UNB6.2 (qualificateur de référence du destinataire) |Entrez une valeur numérique contenant deux caractères maximum. |

### <a name="acknowledgments"></a>Remerciements

| Propriété | Description |
| --- | --- |
| Réception des messages (CONTRL) |Activez cette case à cocher pour renvoyer un accusé de réception technique (CONTRL) à l’expéditeur. Cet accusé de réception est envoyé à l’expéditeur en fonction des paramètres d’envoi du contrat. |
| Accusé de réception (CONTRL) |Cochez cette case pour renvoyer un accusé de réception (CONTRL) fonctionnel à l’expéditeur. L’accusé de réception est envoyé à l’expéditeur en fonction des paramètres d’envoi du contrat. |

### <a name="schemas"></a>Schémas

| Propriété | Description |
| --- | --- |
| UNH2.1 (TYPE) |Sélectionnez un type de document informatisé. |
| UNH2.2 (VERSION) |Entrez le numéro de version du message. (un caractère minimum ; 3 caractères maximum). |
| UNH2.3 (VERSION FINALE) |Entrez le numéro de version finale du message. (un caractère minimum ; 3 caractères maximum). |
| UNH2.5 (CODE AFFECTÉ ASSOCIÉ) |Entrez le code affecté. (6 caractères maximum. Doit contenir des valeurs alphanumériques). |
| UNG2.1 (ID DE L’EXPÉDITEUR D’APPLICATION) |Entrez une valeur numérique comprenant entre un et 35 caractères. |
| UNG2.2 (QUALIFICATEUR DE L’EXPÉDITEUR D’APPLICATION) |Entrez une valeur alphanumérique comprenant quatre caractères maximum. |
| SCHEMA |Sélectionnez le schéma que vous avez déjà téléchargé et que vous souhaitez utiliser à partir de votre compte d’intégration associé. |

### <a name="control-numbers"></a>Numéros de contrôle

| Propriété | Description |
| --- | --- |
| Interdire les doublons de numéro de contrôle d’échange |Pour bloquer les échanges en double, sélectionnez cette propriété. Si cette option est sélectionnée, l’action de décodage EDIFACT vérifie que le numéro de contrôle d’échange (UNB5) pour l’échange reçu ne correspond pas à un numéro de contrôle déjà traité. Si une correspondance est détectée, l’échange n’est pas traité. |
| Check for duplicate UNB5 every (days) (Vérifier les doublons UNB5 tous les (jours)) |Si vous choisissez de ne pas autoriser les numéros de contrôle d’échange en double, vous pouvez spécifier le nombre de jours après lequel le contrôle est effectué en définissant la valeur appropriée pour ce paramètre. |
| Interdire les numéros de contrôle de groupe en double |Pour bloquer les échanges avec des numéros de contrôle de groupe en double (UNG5), sélectionnez cette propriété. |
| Interdire les numéros de contrôle de document informatisé en double |Pour bloquer les échanges avec des numéros de contrôle de document informatisé en double (UNH1), sélectionnez cette propriété. |
| Numéro de contrôle d’accusé de réception EDIFACT |Pour désigner les numéros de référence de document informatisé à utiliser dans un accusé de réception, entrez une valeur de préfixe, une plage de numéros de référence et un suffixe. |

### <a name="validation"></a>Validation

Une nouvelle ligne de validation est automatiquement ajoutée dès que la ligne précédente est terminée. Si vous ne spécifiez aucune règle, la validation utilise la ligne « Par défaut ».

| Propriété | Description |
| --- | --- |
| Type de message |Sélectionnez le type de message EDI. |
| Validation EDI |Effectuez une validation EDI sur les types de données selon les propriétés EDI du schéma, les restrictions de longueur, les éléments de données vides et les séparateurs de fin. |
| Validation étendue |Si le type de données n’est pas EDI, la validation s’effectue sur la spécification d’élément de données et la répétition autorisée, les énumérations et la validation de la longueur d’élément de données (min/max). |
| Autoriser les zéros de début ou de fin |Conservez les zéros de début ou de fin ainsi que les caractères d’espace supplémentaires. Ne supprimez pas ces caractères. |
| Supprimer les zéros de début ou de fin |Supprimez les zéros de début ou de fin ainsi que les caractères d’espace. |
| Stratégie de séparateur de fin |Générez des séparateurs de fin. <p>Sélectionnez **Non autorisé** pour interdire les délimiteurs et les séparateurs dans l’échange reçu. Si l’échange contient des délimiteurs et des séparateurs, il est déclaré non valide. <p>Sélectionnez **Facultatif** pour accepter les échanges avec ou sans délimiteurs et séparateurs. <p>Sélectionnez **Obligatoire** lorsque l’échange reçu doit contenir des délimiteurs et des séparateurs. |

### <a name="internal-settings"></a>Paramètres internes

| Propriété | Description |
| --- | --- |
| Créer des balises XML vides si les séparateurs de fin sont autorisés |Cochez cette case afin d’inclure des balises XML vides pour les séparateurs de l’expéditeur. |
| Scinder l’échange en documents informatisés : suspendre les documents informatisés en cas d’erreur|Analyse chaque document informatisé d’un échange dans un document XML distinct en appliquant l’enveloppe appropriée au document informatisé. Suspendez uniquement les documents informatisés dont la validation échoue. |
| Scinder l’échange en documents informatisés : suspendre l’échange en cas d’erreur|Analyse chaque document informatisé d’un échange dans un document XML distinct en appliquant l’enveloppe appropriée. Suspendez l’intégralité de l’échange lorsque la validation d’un ou plusieurs documents informatisés de l’échange échoue. | 
| Préserver l'échange : suspendre les documents informatisés en cas d’erreur |Préserve l’échange : crée un document XML pour l’intégralité de l’échange par lot. Suspendez uniquement les documents informatisés dont la validation échoue, tout en continuant à traiter tous les autres documents informatisés. |
| Préserver l’échange : suspendre l’échange en cas d’erreur |Préserve l’échange : crée un document XML pour l’intégralité de l’échange par lot. Suspendez l’intégralité de l’échange lorsque la validation d’un ou plusieurs documents informatisés de l’échange échoue. |

## <a name="configure-how-your-agreement-sends-messages"></a>Configuration de l’envoi des messages

Vous pouvez configurer la manière dont votre contrat identifie et traite les messages sortants que vous envoyez à vos partenaires par l’intermédiaire de ce contrat.

1.  Sous **Ajouter**, sélectionnez **Paramètres d’envoi**.
Configurez ces propriétés selon le contrat conclu avec le partenaire qui échange des messages avec vous. Pour obtenir les descriptions des propriétés, consultez les tableaux de cette section.

    Les **paramètres d’envoi** sont organisés en plusieurs sections : identificateurs, accusé de réception, schémas, enveloppes, jeux de caractères et séparateurs, numéros de contrôle et validations.

    ![Configurer « Paramètres d’envoi »](./media/logic-apps-enterprise-integration-edifact/edifact-3.png)    

2. Une fois que vous avez terminé, cliquez sur **OK** pour enregistrer vos paramètres.

Votre contrat est maintenant prêt à traiter les messages sortants qui sont conformes aux paramètres que vous avez sélectionnés.

### <a name="identifiers"></a>Identificateurs

| Propriété | Description |
| --- | --- |
| UNB1.2 (version de la syntaxe) |Sélectionnez une valeur comprise entre **1** et **4**. |
| UNB2.3 (adresse de routage inverse de l’expéditeur) |Entrez une valeur numérique comprenant entre un et 14 caractères. |
| UNB3.3 (adresse de routage inverse du destinataire) |Entrez une valeur numérique comprenant entre un et 14 caractères. |
| UNB6.1 (mot de passe de référence du destinataire) |Entrez une valeur numérique comprenant entre un et 14 caractères. |
| UNB6.2 (qualificateur de référence du destinataire) |Entrez une valeur numérique contenant deux caractères maximum. |
| UNB7 (identifiant de référence de l’application) |Entrez une valeur numérique comprenant entre un et 14 caractères |

### <a name="acknowledgment"></a>Accusé de réception

| Propriété | Description |
| --- | --- |
| Réception des messages (CONTRL) |Activez cette case à cocher si le partenaire hébergé s’attend à recevoir un accusé de réception (CONTRL) technique. Ce paramètre spécifie que le partenaire hébergé qui envoie le message demande un accusé de réception de la part du partenaire invité. |
| Accusé de réception (CONTRL) |Activez cette case à cocher si le partenaire hébergé s’attend à recevoir un accusé de réception (CONTRL) fonctionnel. Ce paramètre spécifie que le partenaire hébergé qui envoie le message demande un accusé de réception de la part du partenaire invité. |
| Générer une boucle SG1/SG4 pour les documents informatisés acceptés |Si vous choisissez de demander un accusé de réception fonctionnel, activez cette case à cocher pour forcer la génération de boucles SG1/SG4 dans les accusés de réception CONTRL fonctionnels pour les documents informatisés acceptés. |

### <a name="schemas"></a>Schémas

| Propriété | Description |
| --- | --- |
| UNH2.1 (TYPE) |Sélectionnez un type de document informatisé. |
| UNH2.2 (VERSION) |Entrez le numéro de version du message. |
| UNH2.3 (VERSION FINALE) |Entrez le numéro de version finale du message. |
| SCHEMA |Sélectionnez le schéma à utiliser. Les schémas se trouvent dans votre compte d’intégration. Pour accéder à vos schémas, vous devez tout d’abord lier votre compte d’intégration à votre application logique. |

### <a name="envelopes"></a>Enveloppes

| Propriété | Description |
| --- | --- |
| UNB8 (code de priorité de traitement) |Entrez une valeur alphabétique ne contenant pas plus d’un caractère. |
| UNB10 (contrat de communication) |Entrez une valeur numérique comprenant entre un et 40 caractères. |
| UNB11 (indicateur de test) |Activez cette case à cocher pour indiquer que l’échange généré contient des données de test |
| Appliquer le segment UNA (conseil de service de chaîne) |Activez cette case à cocher pour générer un segment UNA pour l’échange à envoyer. |
| Appliquer des segments UNG (En-tête de groupe fonctionnel) |Activez cette case à cocher pour créer des segments de regroupement dans l’en-tête de groupe fonctionnel dans les messages envoyés au partenaire invité. Les valeurs suivantes sont utilisées pour créer les segments UNG : <p>Pour **UNG1**, entrez une valeur alphanumérique comportant un caractère minimum et six caractères maximum. <p>Pour **UNG2.1**, entrez une valeur alphanumérique comportant un caractère minimum et 35 caractères maximum. <p>Pour **UNG2.2**, entrez une valeur alphanumérique comprenant quatre caractères maximum. <p>Pour **UNG3.1**, entrez une valeur alphanumérique comportant un caractère minimum et 35 caractères maximum. <p>Pour **UNG3.2**, entrez une valeur alphanumérique comprenant quatre caractères maximum. <p>Pour **UNG6**, entrez une valeur alphanumérique comportant un caractère minimum et trois caractères maximum. <p>Pour **UNG7.1**, entrez une valeur alphanumérique comportant un caractère minimum et trois caractères maximum. <p>Pour **UNG7.2**, entrez une valeur alphanumérique comportant un caractère minimum et trois caractères maximum. <p>Pour **UNG7.3**, entrez une valeur alphanumérique comportant un caractère minimum et six caractères maximum. <p>Pour **UNG8**, entrez une valeur alphanumérique comportant un caractère minimum et 14 caractères maximum. |

### <a name="character-sets-and-separators"></a>Jeux de caractères et séparateurs

outre le jeu de caractères, vous pouvez entrer un autre ensemble de délimiteurs à utiliser pour chaque type de message. Si un jeu de caractères n’est pas spécifié pour un schéma de message donné, le jeu de caractères par défaut est utilisé.

| Propriété | Description |
| --- | --- |
| UNB1.1 (identificateur système) |Sélectionnez le jeu de caractères EDIFACT à appliquer à l’échange sortant. |
| schéma |Dans la liste déroulante, sélectionnez un schéma. Une nouvelle ligne est automatiquement ajoutée lorsque la ligne précédente est terminée. Pour le schéma sélectionné, sélectionnez le jeu de séparateurs à utiliser en fonction de leur description ci-dessous. |
| Type d’entrée |Dans la liste déroulante, sélectionnez un type d’entrée. |
| Séparateur de composants |Entrez un caractère unique pour séparer les éléments de données composites. |
| Séparateur d'éléments de données |Entrez un caractère unique pour séparer les éléments de données simples au sein des éléments de données composites. |
| Terminateur de segment |Entrez un caractère unique pour indiquer la fin d’un segment EDI. |
| Suffixe |Sélectionnez le caractère utilisé avec l’identificateur de segment. Si vous désignez un suffixe, l’élément de données de terminateur de segment peut être vide. Si le terminateur de segment est laissé vide, vous devez désigner un suffixe. |

### <a name="control-numbers"></a>Numéros de contrôle

| Propriété | Description |
| --- | --- |
| UNB5 (Numéro de contrôle de l’échange) |Entrez un préfixe, une plage de valeurs pour le numéro de contrôle de l’échange, ainsi qu’un suffixe. Ces valeurs sont utilisées pour générer un échange sortant. Le préfixe et le suffixe sont facultatifs, tandis que le numéro de contrôle est obligatoire. Le numéro de contrôle est incrémenté pour chaque nouveau message ; le préfixe et le suffixe restent les mêmes. |
| UNG5 (Numéro de contrôle de groupe) |Entrez un préfixe, une plage de valeurs pour le numéro de contrôle de l’échange, ainsi qu’un suffixe. Ces valeurs sont utilisées pour générer le numéro de contrôle du groupe. Le préfixe et le suffixe sont facultatifs, tandis que le numéro de contrôle est obligatoire. Le numéro de contrôle est incrémenté pour chaque nouveau message jusqu’à ce que la valeur maximale soit atteinte ; le préfixe et le suffixe restent les mêmes. |
| UNH1 (Numéro de référence de l’en-tête de message) |Entrez un préfixe, une plage de valeurs pour le numéro de contrôle de l’échange, ainsi qu’un suffixe. Ces valeurs sont utilisées pour générer le numéro de référence de l’en-tête de message. Le préfixe et le suffixe sont facultatifs, tandis que le numéro de référence est obligatoire. Le numéro de référence est incrémenté pour chaque nouveau message ; le préfixe et le suffixe restent les mêmes. |

### <a name="validation"></a>Validation

Une nouvelle ligne de validation est automatiquement ajoutée dès que la ligne précédente est terminée. Si vous ne spécifiez aucune règle, la validation utilise la ligne « Par défaut ».

| Propriété | Description |
| --- | --- |
| Type de message |Sélectionnez le type de message EDI. |
| Validation EDI |Effectuez une validation EDI sur les types de données selon les propriétés EDI du schéma, les restrictions de longueur, les éléments de données vides et les séparateurs de fin. |
| Validation étendue |Si le type de données n’est pas EDI, la validation s’effectue sur la spécification d’élément de données et la répétition autorisée, les énumérations et la validation de la longueur d’élément de données (min/max). |
| Autoriser les zéros de début ou de fin |Conservez les zéros de début ou de fin ainsi que les caractères d’espace supplémentaires. Ne supprimez pas ces caractères. |
| Supprimer les zéros de début ou de fin |Supprimez les zéros de début ou de fin. |
| Stratégie de séparateur de fin |Générez des séparateurs de fin. <p>Sélectionnez **Non autorisé** pour interdire les délimiteurs et les séparateurs dans l’échange envoyé. Si l’échange contient des délimiteurs et des séparateurs, il est déclaré non valide. <p>Sélectionnez **Facultatif** pour envoyer des échanges avec ou sans délimiteurs et séparateurs. <p>Sélectionnez **Obligatoire** si l’échange envoyé doit contenir des délimiteurs et des séparateurs. |

## <a name="find-your-created-agreement"></a>Comment retrouver le contrat que vous avez créé

1.  Après avoir défini toutes les propriétés de votre contrat, dans la page **Ajouter**, choisissez **OK** pour finaliser la création de votre contrat et revenir à votre compte d’intégration.

    Le contrat que vous venez d’ajouter s’affiche dans votre liste **Contrats**.

2.  Vous pouvez également afficher vos contrats dans la vue d’ensemble de votre compte d’intégration. Dans le menu de votre compte d’intégration, choisissez **Vue d’ensemble**, puis sélectionnez la mosaïque **Contrats**. 

    ![Capture d’écran montrant la vignette Contrats.](./media/logic-apps-enterprise-integration-edifact/edifact-4.png)   

## <a name="connector-reference"></a>Référence de connecteur

Pour plus d’informations techniques sur ce connecteur, notamment au sujet des actions et des limites décrites dans le fichier Swagger du connecteur, consultez la [page de référence du connecteur](/connectors/edifact/).

> [!NOTE]
> Pour les applications logiques utilisées dans un [environnement de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la version de ce connecteur avec l’étiquette ISE applique les [limites de messages B2B de l’ISE](../logic-apps/logic-apps-limits-and-config.md#b2b-protocol-limits).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)
