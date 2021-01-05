---
title: Comment configurer une application du proxy d’application | Microsoft Docs
description: Apprenez à configurer une application du proxy d’application en quelques étapes simples
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 05/18/2018
ms.author: kenwith
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: c63137c6943d9adc0ea7c19f7551d1f31587f42a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84764993"
---
# <a name="how-to-configure-an-application-proxy-application"></a>Comment configurer une application du proxy d’application

Cet article vous aide à comprendre comment configurer une application de proxy d’application dans Azure AD afin de rendre vos applications locales accessibles au cloud.

## <a name="recommended-documents"></a>Documents recommandés

Pour en savoir plus sur les configurations initiales et la création d’une application de proxy d’application par le biais du portail d’administration, consultez [Publier des applications avec le Proxy d’application Azure AD](application-proxy-add-on-premises-application.md).

Pour plus d’informations sur la configuration des connecteurs, consultez [Activer le proxy d’application dans le portail Azure](application-proxy-add-on-premises-application.md).

Pour plus d’informations sur le chargement des certificats et l’utilisation des domaines personnalisés, consultez [Utilisation des domaines personnalisés dans le proxy d’application Azure AD](application-proxy-configure-custom-domain.md).

## <a name="create-the-applicationsetting-the-urls"></a>Créer l’application/Définir les URL

Si vous suivez les étapes décrites dans la documentation [Publier des applications avec le Proxy d’application Azure AD](application-proxy-add-on-premises-application.md) et si vous obtenez une erreur lors de création de l’application, consultez les détails de l’erreur pour en savoir plus et découvrir des suggestions afin de corriger l’application. La plupart des messages d’erreur incluent la suggestion d’un correctif. Pour éviter les erreurs courantes, vérifiez les points suivants :

- Vous êtes un administrateur autorisé à créer une application de proxy d’application
- L’URL interne est unique
- L’URL externe est unique
- Les URL commencent par http ou https et se terminent par un signe « / »
- L’URL doit être un nom de domaine, et non une adresse IP

Le message d’erreur s’affiche en haut à droite lorsque vous créez l’application. Vous pouvez également sélectionner l’icône de notification pour afficher les messages d’erreur.

![Indique où trouver l’invite de notification sur le portail Azure](./media/application-proxy-config-how-to/error-message.png)

## <a name="configure-connectorsconnector-groups"></a>Configurer des connecteurs/groupes de connecteurs

Si vous rencontrez des difficultés pour configurer votre application en raison d’un avertissement concernant les connecteurs et les groupes de connecteurs, consultez les instructions dédiées à l’activation du proxy d’application pour en savoir plus sur le téléchargement des connecteurs. Si vous souhaitez en savoir plus sur les connecteurs, consultez la [documentation consacrée aux connecteurs](application-proxy-connectors.md).

Si vos connecteurs sont inactifs, cela signifie qu’ils ne peuvent pas atteindre le service. Cela est souvent dû au fait que tous les ports requis ne sont pas ouverts. Pour afficher la liste des ports requis, consultez la section relative aux conditions préalables dans la documentation d’activation du proxy d’application.

## <a name="upload-certificates-for-custom-domains"></a>Charger des certificats pour des domaines personnalisés

Les domaines personnalisés vous permettent de spécifier le domaine de vos URL externes. Pour utiliser des domaines personnalisés, vous devez charger le certificat relatif à ce domaine. Pour plus d’informations sur l’utilisation des domaines et des certificats personnalisés, consultez [Utilisation des domaines personnalisés dans le proxy d’application Azure AD](application-proxy-configure-custom-domain.md).

Si vous rencontrez des problèmes de chargement du certificat, recherchez les messages d’erreur dans le portail pour en savoir plus. Voici les problèmes courants liés aux certificats :

- Expiration du certificat
- Certificat auto-signé
- Clé privée manquante pour le certificat

Le message d’erreur s’affiche en haut à droite lorsque vous tentez de charger le certificat. Vous pouvez également sélectionner l’icône de notification pour afficher les messages d’erreur.

## <a name="next-steps"></a>Étapes suivantes

[Publier des applications avec le Proxy d’application Azure AD](application-proxy-add-on-premises-application.md)
