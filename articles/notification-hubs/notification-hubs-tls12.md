---
title: Mises à jour de TLS dans Notification Hubs
description: Découvrez-en plus sur la prise en charge de TLS dans Azure Notification Hubs.
services: notification-hubs
documentationcenter: .net
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: multiple
ms.topic: article
ms.date: 04/29/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 01/28/2020
ms.openlocfilehash: 87a3627d7820f9f456ac08e2f20b70af961f817e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87084243"
---
# <a name="transport-layer-security-tls"></a>TLS (Transport Layer Security)

Pour garantir un niveau de sécurité élevé, Notification Hubs désactivera la prise en charge des versions 1.0 et 1.1 de TLS le **31 décembre 2020** (et non le 30 avril 2020). Ces protocoles anciens offrent un faible niveau de chiffrement et sont vulnérables aux attaques BEAST et POODLE. Les applications déployées sur des appareils exécutant Android version 5 ou ultérieure, ou iOS version 5 ou ultérieure, ne sont pas concernées par ce changement car ces systèmes d'exploitation prennent en charge TLS 1.2. Le client et le serveur négocieront la version la plus élevée mutuellement prise en charge lors de la connexion.

Nous vous recommandons d'examiner toutes les applications qui utilisent Azure Notification Hubs pour vous assurer qu'elles utilisent les bibliothèques les plus pertinentes et des piles TLS qui prennent en charge TLS 1.2.

## <a name="update-apps"></a>Mise à jour des applications

Vous pouvez vérifier que vos applications iOS utilisent TLS 1.2 à l'aide de la fonctionnalité de sécurité réseau ATS (App Transport Security) d'Apple. La fonctionnalité ATS ne peut pas être utilisée pour les kits de développement logiciel (SDK) antérieurs à iOS 9.0 ou macOS 10.11. Pour plus d'informations, consultez la [documentation d'Apple](https://developer.apple.com/documentation/security/preventing_insecure_network_connections).

Pour les applications Android qui utilisent des instances de SSLSocket, définissez les protocoles activés sur chaque instance de SSLSocket, comme indiqué dans [setEnabledProtocols](https://developer.android.com/reference/javax/net/ssl/SSLSocket#setEnabledProtocols(java.lang.String%5B%5D)).

Le tableau de la page de support [Compatibilité du protocole TLS](https://support.globalsign.com/customer/portal/articles/2934392-tls-protocol-compatibility) permet de mapper les systèmes d'exploitation avec les versions compatibles de TLS.

Pour plus d'informations, consultez la vue d'ensemble de la [prise en charge des protocoles TLS sous Windows](/archive/blogs/kaushal/support-for-ssltls-protocols-on-windows).

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble de Notification Hubs](notification-hubs-push-notification-overview.md)
