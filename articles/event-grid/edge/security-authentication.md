---
title: Sécurité et authentification – Azure Event Grid sur IoT Edge | Microsoft Docs
description: Sécurité et authentification dans Event Grid sur IoT Edge.
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 42f6107ff79d6262cdc0a35cf972cf65d3a9a802
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86171378"
---
# <a name="security-and-authentication"></a>Sécurité et authentification

La sécurité et l’authentification constituent un concept avancé qui requiert une certain connaissance des concepts de base d’Event Grid. Si vous ignorez tout d’Event Grid sur IoT Edge, démarrez [ici](concepts.md). Le module Event Grid s’appuie sur l’infrastructure de sécurité existante de IoT Edge. Pour plus de détails et pour configurer, consultez [cette documentation](../../iot-edge/security.md).

Les sections suivantes décrivent en détail comment ces paramètres sont sécurisés et authentifiés :

* Configuration TLS
* Authentification du client entrant
* Authentification du serveur sortant
* Authentification du client sortant

>[!IMPORTANT]
>Le module de sécurité et d’authentification Event Grid tire parti de l’infrastructure existante disponible sur IoT Edge. L’hypothèse est que le sous-système IoT Edge est sécurisé.

>[!IMPORTANT]
>La configuration d’Event Grid est **sécurisée par défaut**. Les sous-sections suivantes décrivent toutes les options et les valeurs que vous pouvez utiliser pour remplacer certains aspects de l’authentification. Avant d’apporter des modifications, vous devez comprendre leur impact. Pour que les modifications prennent effet, le module Event Grid doit être redéployé.

## <a name="tls-configuration-aka-server-authentication"></a>Configuration TLS (également appelée authentification du serveur)

Le module Event Grid héberge des points de terminaison HTTP et HTTPs. Chaque module IoT Edge reçoit un certificat de serveur du démon de sécurité de l’IoT Edge. Nous utilisons le certificat de serveur pour sécuriser le point de terminaison. À l’expiration du certificat, le module est automatiquement actualisé avec un nouveau certificat du démon de sécurité d’IoT Edge.

Par défaut, seules les communications HTTPs sont autorisées. Vous pouvez modifier ce comportement via une configuration **inbound__serverAuth__tlsPolicy**. Le tableau suivant indique les valeurs possibles de cette propriété.

| Valeurs possibles | Description |
| ---------------- | ------------ |
| Strict | Par défaut. Active HTTPS uniquement
| activé | Active HTTP et HTTPs
| Désactivé | Active HTTP uniquement

## <a name="inbound-client-authentication"></a>Authentification du client entrant

Les clients sont des entités effectuant des opérations de gestion ou d’exécution. Les clients peuvent être d’autres modules IoT Edge, mais pas des applications IoT.

Le module Event Grid prend en charge deux types d’authentifications du client :

* Signature d’accès partagé (SAP) basée sur une clé
* basée sur un certificat

Par défaut, le module Event Grid est configuré pour accepter uniquement une authentification basée sur un certificat. Au démarrage, le module Event Grid récupère « TrustBundle » à partir du démon de sécurité d’IoT Edge et l’utilise pour valider tout certificat client. Les certificats clients qui ne se résolvent pas dans cette chaîne sont rejetés avec `UnAuthorized`.

### <a name="certificate-based-client-authentication"></a>Authentification du client basée sur un certificat

L’authentification basée sur un certificat est activée par défaut. Vous pouvez désactiver l’authentification basée sur un certificat via la propriété **inbound__clientAuth__clientCert__enabled**. Le tableau suivant présente les valeurs possibles.

| Valeurs possibles | Description |
| ----------------  | ------------ |
| true | Par défaut. Exige que toutes les demandes dans le module Event Grid présentent un certificat client. Vous devez en outre configurer **inbound__clientAuth__clientCert__source**.
| false | Ne force pas un client à présenter un certificat.

Le tableau suivant capture les valeurs possibles pour **inbound__clientAuth__clientCert__source**

| Valeurs possibles | Description |
| ---------------- | ------------ |
| IoT Edge | Par défaut. Utilise le Trustbundle de l’IoT Edge pour valider tous les certificats clients.

Si un client présente un certificat auto-signé, par défaut, le module Event Grid rejette la demande. Vous pouvez autoriser les certificats clients auto-signés à l’aide de la propriété **inbound__clientAuth__clientCert__allowUnknownCA**. Le tableau suivant présente les valeurs possibles.

| Valeurs possibles | Description |
| ----------------  | ------------|
| true | Par défaut. Autorise la présentation de certificats auto-signés.
| false | Les demandes échouent en cas de présentation de certificats auto-signés.

>[!IMPORTANT]
>Dans des scénarios de production, vous pouvez définir **inbound__clientAuth__clientCert__allowUnknownCA** sur **false**.

### <a name="sas-key-based-client-authentication"></a>Authentification du client basée sur une clé SAP

Outre l’authentification basée sur un certificat, le module Event Grid peut également effectuer une authentification basée sur une clé SAP. Une clé SAP est semblable à un secret configuré dans le module Event Grid que celui-ci doit utiliser pour valider tous les appels entrants. Les clients doivent spécifier le secret dans l’en-tête HTTP ’aeg-sas-key’. Si la demande ne correspond pas, est rejetée avec l’erreur `UnAuthorized`.

La configuration permettant de contrôler l’authentification basée sur une clé SAP est **inbound__clientAuth__sasKeys__enabled**.

| Valeurs possibles | Description  |
| ----------------  | ------------ |
| true | Autorise une authentification basée sur une clé SAP. Requiert **inbound__clientAuth__sasKeys__key1** ou **inbound__clientAuth__sasKeys__key2**
| false | Par défaut. L’authentification basée sur une clé SAP est désactivée.

 **inbound__clientAuth__sasKeys__key1** et **inbound__clientAuth__sasKeys__key2** sont des clés que vous configurez sur le module Event Grid pour vérifier les demandes entrantes. Au moins une des clés doit être configurée. Le client qui effectue la demande doit présenter la clé dans l’en-tête de demande ’**aeg-sas-key**’. Si les deux clés sont configurées, le client peut présenter n’importe laquelle.

> [!NOTE]
>Vous pouvez configurer les deux méthodes d’authentification. Dans ce cas, la clé SAP est vérifiée en premier et l’authentification basée sur le certificat n’est effectuée qu’en cas d’échec. Pour qu’une demande aboutisse, une seule méthode d’authentification doit réussir.

## <a name="outbound-client-authentication"></a>Authentification du client sortant

Un client dans un contexte sortant fait référence au module Event Grid. L’opération consiste à livrer des événements à des abonnés. Les modules qui s’abonnent sont considérés comme le serveur.

Chaque module IoT Edge reçoit un certificat d’identité du démon de sécurité de l’IoT Edge. Nous utilisons le certificat d’identité pour les appels sortants. À l’expiration du certificat, le module est automatiquement actualisé avec un nouveau certificat du démon de sécurité d’IoT Edge.

La configuration permettant de contrôler l’authentification du client sortant est **outbound__clientAuth__clientCert__enabled**.

| Valeurs possibles | Description |
| ----------------  | ------------ |
| true | Par défaut. Exige que toutes les demandes sortantes du module Event Grid présentent un certificat. Nécessite de configurer **outbound__clientAuth__clientCert__source**.
| false | N’exige pas que le module Event Grid présente son certificat.

La configuration qui contrôle la source du certificat est **outbound__clientAuth__clientCert__source**.

| Valeurs possibles | Description |
| ---------------- | ------------ |
| IoT Edge | Par défaut. Utilise le certificat d’identité du module configuré par le démon de sécurité d’IoT Edge.

### <a name="outbound-server-authentication"></a>Authentification du serveur sortant

L’un des types de destinations pour un abonné Event Grid est « Webhook ». Par défaut, seuls des points de terminaison HTTPs sont acceptés pour de tels abonnés.

La configuration permettant de contrôler la stratégie de destination webhook est **outbound__webhook__httpsOnly**.

| Valeurs possibles | Description |
| ----------------  | ------------ |
| true | Par défaut. Autorise uniquement les abonnés disposant d’un point de terminaison HTTPs.
| false | Autorise les abonnés disposant d’un point de terminaison HTTP ou HTTPs.

Par défaut, le module Event Grid valide le certificat de serveur de l’abonné. Vous pouvez ignorer la validation en modifiant **outbound__webhook__skipServerCertValidation**. Les valeurs possibles sont les suivantes :

| Valeurs possibles | Description |
| ----------------  | ------------ |
| true | Ne pas valider pas le certificat de serveur de l’abonné.
| false | Par défaut. Valider le certificat de serveur de l’abonné.

Si le certificat de l’abonné est auto-signé, par défaut, le module Event Grid rejette l’abonné. Pour autoriser l’usage d’un certificat auto-signé, vous pouvez modifier **outbound__webhook__allowUnknownCA**. Le tableau suivant présente les valeurs possibles.

| Valeurs possibles | Description |
| ----------------  | ------------ |
| true | Par défaut. Autorise la présentation de certificats auto-signés.
| false | Les demandes échouent en cas de présentation de certificats auto-signés.

>[!IMPORTANT]
>Dans des scénarios de production, vous pouvez définir **outbound__webhook__allowUnknownCA** sur **false**.

> [!NOTE]
>L’environnement IoT Edge génère des certificats auto-signés. Il est recommandé de générer des certificats émis par des autorités de certification autorisées pour les charges de travail de production, et de définir la propriété **allowUnknownCA** sur **false** pour les trafics tant entrant que sortant.

## <a name="summary"></a>Résumé

Un module Event Grid est **sécurisé par défaut**. Nous vous recommandons de conserver ces valeurs par défaut pour vos déploiements en production.

Voici les principes directeurs à suivre lors de la configuration :

* Autoriser uniquement des requêtes HTTPs dans le module.
* Autoriser uniquement l’authentification du client basée sur un certificat. Autoriser uniquement des certificats émis par des autorités de certification bien connues. Ne pas autoriser de certificats auto-signés.
* Ne pas autoriser d’authentification du client basée sur une clé SAP.
* Toujours présenter le certificat d’identité du module Event Grid sur les appels sortants.
* Autoriser uniquement des abonnés HTTPs pour des types de destinations Webhook.
* Toujours valider le certificat de serveur de l’abonné pour des types de destinations Webhook. Autoriser uniquement des certificats émis par des autorités de certification bien connues. Ne pas autoriser de certificats auto-signés.

Par défaut, le module Event Grid est déployé avec la configuration suivante :

 ```json
 {
  "Env": [
    "inbound__serverAuth__tlsPolicy=strict",
    "inbound__serverAuth__serverCert__source=IoTEdge",
    "inbound__clientAuth__sasKeys__enabled=false",
    "inbound__clientAuth__clientCert__enabled=true",
    "inbound__clientAuth__clientCert__source=IoTEdge",
    "inbound__clientAuth__clientCert__allowUnknownCA=true",
    "outbound__clientAuth__clientCert__enabled=true",
    "outbound__clientAuth__clientCert__source=IoTEdge",
    "outbound__webhook__httpsOnly=true",
    "outbound__webhook__skipServerCertValidation=false",
    "outbound__webhook__allowUnknownCA=true"
  ],
  "HostConfig": {
    "PortBindings": {
      "4438/tcp": [
        {
          "HostPort": "4438"
        }
      ]
    }
  }
}
```
