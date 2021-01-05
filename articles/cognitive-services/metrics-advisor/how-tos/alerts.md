---
title: Configurer les alertes de Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Comment configurer vos alertes Metrics Advisor à l’aide de hooks d’e-mail, web et Azure DevOps.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 09/14/2020
ms.author: mbullwin
ms.openlocfilehash: 30d8fdf99da7a4854db0985bed6256ecd6f7a366
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93420918"
---
# <a name="how-to-configure-alerts-and-get-notifications-using-a-hook"></a>Procédure : Configurer des alertes et obtenir des notifications à l’aide d’un hook

Une fois qu’une anomalie a été détectée par Metrics Advisor, une notification d’alerte est déclenchée à l’aide d’un hook en fonction des paramètres d’alerte. Un paramètre d’alerte peut être utilisé avec plusieurs configurations de détection, différents paramètres étant disponibles pour personnaliser votre règle d’alerte.

## <a name="create-a-hook"></a>Créer un hook

Metrics Advisor prend en charge trois types de hooks différents : les hooks par e-mail, les webhooks et Azure DevOps. Vous pouvez choisir celui qui fonctionne pour votre scénario spécifique.       

### <a name="email-hook"></a>Hook par e-mail

> [!Note]
> Avant que des alertes d’anomalie ne puissent être envoyées, les administrateurs de ressources de Metrics Advisor doivent configurer les paramètres de messagerie et entrer les informations relatives à SMTP dans Metrics Advisor. L’administrateur du groupe de ressources ou l’administrateur des abonnements doit attribuer au moins un rôle *Administrateur Metrics Advisor Cognitive Services* dans l’onglet Contrôle d’accès de la ressource Metrics Advisor. [Découvrez-en plus sur la configuration des paramètres de messagerie](../faq.md#how-to-set-up-email-settings-and-enable-alerting-by-email).

Pour créer un hook d’e-mail, les paramètres suivants sont disponibles : 

Un hook d’e-mail est le canal des alertes d’anomalies à envoyer aux adresses e-mail spécifiées dans la section **E-mail à**. Deux types d’e-mails d’alerte sont envoyés : Des alertes *Le flux de données n’est pas disponible* et des *rapports d’incidents* qui contiennent une ou plusieurs anomalies. 

|Paramètre |Description  |
|---------|---------|
| Nom | Nom du hook d’e-mail |
| E-mail à| Adresses e-mail auxquelles envoyer une alerte|
| Lien externe | Champ facultatif qui active une redirection personnalisée, par exemple pour les notes relatives à la résolution des problèmes. |
| Titre d’alerte d’anomalie personnalisé | Le modèle de titre prend en charge `${severity}`, `${alertSettingName}`, `${datafeedName}`, `${metricName}`, `${detectConfigName}`, `${timestamp}`, `${topDimension}`, `${incidentCount}`, `${anomalyCount}`

Une fois que vous avez cliqué sur **OK**, un hook d’e-mail est créé. Vous pouvez l’utiliser dans n’importe quel paramètre d’alerte pour recevoir des alertes d’anomalie. 

### <a name="web-hook"></a>Webhook

> [!Note]
> * Utilisez la méthode de demande **POST**.
> * Le corps de la demande ressemble à ce qui suit :  
    `{"timestamp":"2019-09-11T00:00:00Z","alertSettingGuid":"49635104-1234-4c1c-b94a-744fc920a9eb"}`
> * Lors de la création ou de la modification d’un webhook, l’API est appelée comme test avec un corps de demande vide. Votre API doit retourner un code HTTP 200.

Un webhook est le point d’entrée pour toutes les informations disponibles du service Metrics Advisor et appelle une API fournie par l’utilisateur lorsqu’une alerte est déclenchée. Toutes les alertes peuvent être envoyées via un webhook.

Pour créer un webhook, vous devrez ajouter les informations suivantes :

|Paramètre |Description  |
|---------|---------|
|Point de terminaison     | L’adresse API à appeler lorsqu’une alerte est déclenchée.        |
|Nom d’utilisateur / mot de passe | Pour l’authentification auprès de l’adresse d’API. Ne pas remplir si l’authentification n’est pas nécessaire.         |
|En-tête     | En-têtes personnalisés dans l’appel d’API.        |

:::image type="content" source="../media/alerts/create-web-hook.png" alt-text="fenêtre de création de webhook.":::

Lorsqu’une notification est envoyée via un webhook, vous pouvez utiliser les API suivantes pour obtenir les détails de l’alerte. Définissez l’*horodateur* et *alertSettingGuid* dans votre service API qui fait l’objet d’un envoi (push), puis utilisez les requêtes suivantes : 
- `query_alert_result_anomalies`
- `query_alert_result_incidents`

### <a name="azure-devops"></a>Azure DevOps

Metrics Advisor prend également en charge la création automatique d’un élément de travail dans Azure DevOps pour suivre les problèmes/bogues si une anomalie est détectée. Toutes les alertes peuvent être envoyées via des hooks DevOps.

Pour créer un hook Azure DevOps, vous devrez ajouter les informations suivantes

|Paramètre |Description  |
|---------|---------|
| Nom | Nom du hook |
| Organisation | Organisation à laquelle votre DevOps appartient |
| Projet | Projet spécifique dans DevOps. |
| Jeton d'accès |  Jeton pour l’authentification auprès de DevOps. | 

> [!Note]
> Vous devez accorder des autorisations d’écriture si vous souhaitez que Metrics Advisor crée des éléments de travail en fonction des alertes d’anomalie. Après avoir créé des hooks, vous pouvez les utiliser dans tous vos paramètres d’alerte. Gérez vos hooks à la page **Paramètres de hooks**.

## <a name="add-or-edit-alert-settings"></a>Ajouter ou modifier des paramètres d’alerte

Accédez à la page de détails des métriques pour y rechercher la section **Paramètres d’alerte** en bas à gauche. Elle répertorie tous les paramètres d’alerte qui s’appliquent à la configuration de détection sélectionnée. Lorsqu’une nouvelle configuration de détection est créée, il n’existe aucun paramètre d’alerte et aucune alerte n’est envoyée.  
Vous pouvez utiliser les icônes **Ajouter**, **Modifier** et **Supprimer** pour modifier les paramètres d’alerte.

:::image type="content" source="../media/alerts/alert-setting.png" alt-text="Élément de menu Paramètres d’alerte.":::

Cliquez sur le bouton **ajouter** ou **Modifier** pour obtenir une fenêtre permettant d’ajouter ou de modifier vos paramètres d’alerte.

:::image type="content" source="../media/alerts/edit-alert.png" alt-text="Ajouter ou modifier des paramètres d’alerte":::

**Nom du paramètre d’alerte** : nom du paramètre d’alerte. Il s’affichera dans le titre de l’e-mail d’alerte.

**Hooks** : liste des hooks auxquels envoyer des alertes.

La section marquée dans la capture d’écran ci-dessus est celle des paramètres pour une configuration de détection. Vous pouvez définir différents paramètres d’alerte pour différentes configurations de détection. Choisissez la configuration cible à l’aide de la troisième liste déroulante dans cette fenêtre. 

### <a name="filter-settings"></a>Paramètres du filtre 

Voici les paramètres de filtre pour une configuration de détection.

**Alerte pour** a 4 options pour filtrer les anomalies :

* **Anomalies dans toutes les séries** : toutes les anomalies seront incluses dans l’alerte.         
* **Anomalies dans le groupe de séries** : filtrer les séries par valeurs de dimension. Définir des valeurs spécifiques pour certaines dimensions. Les anomalies ne seront incluses dans l’alerte que si la série correspond à la valeur spécifiée.       
* **Anomalies dans les séries de favoris** : seule la série marquée comme favori sera incluse dans l’alerte.        |
* **Anomalies dans les N premières séries** : ce filtre doit être utilisé si vous vous intéressez uniquement à la série dont la valeur se trouve parmi le N premières. Nous allons examiner certains horodatages et vérifier si la valeur de la série sur ces horodatages est dans les N premières. Si le nombre de « dans les n premières » est supérieur au nombre spécifié, l’anomalie sera incluse dans une alerte.        |

**Filtrer les options d’anomalie** est un filtre supplémentaire qui offre les options suivantes :

- **gravité** : l’anomalie sera incluse uniquement si la gravité de l’anomalie est comprise dans la plage spécifiée.
- **Répéter** : arrête temporairement les alertes pour les anomalies aux N points suivants (période) lorsqu’elles sont déclenchées dans une alerte.
    - **type de répétition** : lorsqu’elle est définie sur **Série**, une anomalie déclenchée se répétera uniquement dans sa série. Pour **Métrique**, une anomalie déclenchée se répétera dans toutes les séries de cette mesure.
    - **nombre de répétitions** : nombre de points (périodes) à répéter.
    - **réinitialisation pour les anomalies non successives** : une anomalie sélectionnée qui est déclenchée ne répétera que les n anomalies successives suivantes. Si l’un des points de données suivants n’est pas une anomalie, la répétition sera réinitialisée à partir de ce point. Lorsqu’elle est désélectionnée, une anomalie déclenchée se répétera pour les n points suivants (période), même si les points de données successifs ne sont pas des anomalies.
- **valeur** (facultatif) : filtrer par valeur. Uniquement les valeurs de points qui remplissent la condition, l’anomalie sera incluse. Si vous utilisez la valeur correspondante d’une autre mesure, les noms des dimensions des deux métriques doivent être cohérents.

Les anomalies non éliminées par le filtrage sont envoyées dans une alerte.

### <a name="add-cross-metric-settings"></a>Ajouter des paramètres de métriques croisées

Cliquez sur **+ Ajouter des paramètres de métriques** dans la page Paramètres d’alerte pour ajouter une autre section.

Le sélecteur d’**Opérateur** est la relation logique de chaque section pour déterminer si une alerte doit être envoyée.


|Opérateur  |Description  |
|---------|---------|
|AND     | Envoyer une alerte uniquement si une série correspond à chaque section d’alerte et que tous les points de données sont des anomalies. Si les métriques ont des noms de dimensions différents, une alerte ne sera jamais déclenchée.         |
|OU     | Envoyer l’alerte si au moins une section contient des anomalies.         |

:::image type="content" source="../media/alerts/alert-setting-operator.png" alt-text="Section Opérateur pour plusieurs paramètres d’alerte":::

## <a name="next-steps"></a>Étapes suivantes

- [Ajustement de la détection d’anomalie selon les commentaires](anomaly-feedback.md)
- [Diagnostic d’un incident](diagnose-incident.md).
- [Configuration des métriques et affinage de la configuration de la détection](configure-metrics.md)