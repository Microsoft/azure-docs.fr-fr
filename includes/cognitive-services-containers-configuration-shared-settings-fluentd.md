---
author: IEvangelist
ms.author: dapine
ms.date: 06/25/2019
ms.service: cognitive-services
ms.topic: include
ms.openlocfilehash: b08516b35a864eae6d15c4c5c928f0550c64c239
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001160"
---
Fluentd est un collecteur de données open source pour la journalisation unifiée. Les paramètres `Fluentd` gèrent la connexion du conteneur à un serveur [Fluentd](https://www.fluentd.org). Le conteneur comprend un fournisseur de journalisation Fluentd qui permet à votre conteneur d’écrire des données de journaux d’activité et, éventuellement, de métriques, sur un serveur Fluentd.

Le tableau suivant décrit les paramètres de configuration pris en charge sous la section `Fluentd`.

| Name | Type de données | Description |
|------|-----------|-------------|
| `Host` | String | Adresse IP ou nom d’hôte DNS du serveur Fluentd. |
| `Port` | Integer | Port du serveur Fluentd.<br/> La valeur par défaut est 24224. |
| `HeartbeatMs` | Integer | Intervalle de pulsation, en millisecondes. Si aucun trafic d’événement n’est envoyé avant l’expiration de cet intervalle, une pulsation est envoyée au serveur Fluentd. La valeur par défaut est de 60 000 millisecondes (1 minute). |
| `SendBufferSize` | Integer | Espace de mémoire tampon réseau, en octets, alloué pour les opérations d’envoi. La valeur par défaut est de 32 768 octets (32 kilo-octets). |
| `TlsConnectionEstablishmentTimeoutMs` | Integer | Délai d’attente, en millisecondes, pour établir une connexion SSL/TLS avec le serveur Fluentd. La valeur par défaut est de 10 000 millisecondes (10 secondes).<br/> Si `UseTLS` est défini sur false, cette valeur est ignorée. |
| `UseTLS` | Boolean | Indique si le conteneur doit utiliser SSL/TLS pour communiquer avec le serveur Fluentd. La valeur par défaut est false. |