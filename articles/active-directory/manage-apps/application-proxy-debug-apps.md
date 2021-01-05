---
title: Déboguer les applications de proxy d'application - Azure Active Directory | Microsoft Docs
description: Déboguez les problèmes liés aux applications de proxy d'application Azure Active Directory (Azure AD).
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 05/21/2019
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: e61ea30f01e8ae141d24c9bd91b08edef4dbe74e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85555058"
---
# <a name="debug-application-proxy-application-issues"></a>Déboguer les problèmes d’application de proxy d’application 

Cet article vous aide à résoudre les problèmes liés aux applications de proxy d'application Azure Active Directory (Azure AD). Si vous utilisez le service Proxy d’application pour accéder à distance à une application web local sans parvenir à vous y connecter, utilisez cet organigramme pour déboguer les problèmes de l'application. 

## <a name="before-you-begin"></a>Avant de commencer

Lors de la résolution des problèmes liés au service Proxy d’application, nous vous recommandons de commencer par les connecteurs. Tout d’abord, suivez le flux de résolution des problèmes dans [Déboguer les problèmes liés aux connecteurs du service Proxy d’application](application-proxy-debug-connectors.md) pour vous assurer que les connecteurs sont correctement configurés. Si les problèmes persistent, consultez cet article pour résoudre les problèmes liés à l'application.  

Pour plus d’informations sur le proxy d'application, consultez :

- [Accès à distance aux applications locales par le bais du service Proxy d'application](application-proxy.md)
- [Connecteurs du service Proxy d’application](application-proxy-connectors.md)
- [Installer et inscrire un connecteur](application-proxy-add-on-premises-application.md)
- [Résoudre les problèmes de proxy d’application et les messages d’erreur](application-proxy-troubleshoot.md)

## <a name="flowchart-for-application-issues"></a>Organigramme des problèmes liés à l'application

Cet organigramme vous guide tout au long des étapes de débogage des problèmes de connexion à l'application les plus courants. Pour plus d’informations sur chaque étape, consultez le tableau suivant l’organigramme.

![Organigramme illustrant les étapes de débogage d’une application](media/application-proxy-debug-apps/application-proxy-apps-debugging-flowchart.png)

| Étape | Action | Description |
|---------|---------|---------|
|1 | Ouvrez un navigateur, accédez à l’application et entrez vos informations d’identification | Essayez d’utiliser vos informations d’identification pour vous connecter à l’application et consultez les éventuelles erreurs liées à l'utilisateur comme [Impossible d'accéder à cet application d'entreprise](application-proxy-sign-in-bad-gateway-timeout-error.md). |
|2 | Vérifiez l’affectation d'utilisateurs à l'application | Assurez-vous que votre compte d’utilisateur est autorisé à accéder à l’application à partir du réseau d’entreprise, puis effectuez un test de signature en suivant la procédure [Tester l’application](application-proxy-add-on-premises-application.md#test-the-application). Si les problèmes de connexion persistent, consultez [Résoudre les erreurs de connexion](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context).  |
|3 | Ouvrez un navigateur et essayez d’accéder à l’application | Si une erreur s’affiche immédiatement, vérifiez que le service Proxy d’application est correctement configuré. Pour plus d'informations sur des messages d'erreur spécifiques, consultez [Résoudre les problèmes liés au service Proxy d'application et les messages d'erreur](application-proxy-troubleshoot.md).  |
|4 | Vérifiez la configuration de votre domaine personnalisé ou corrigez l’erreur | Si la page ne s’affiche pas, vérifiez que votre domaine personnalisé est correctement configuré en consultant [Utilisation des domaines personnalisés](application-proxy-configure-custom-domain.md).<br></br>Si la page ne se charge et qu'un message d’erreur s'affiche, corrigez l’erreur en consultant [Résoudre les problèmes liés au service Proxy d'application et les messages d’erreur](application-proxy-troubleshoot.md). <br></br>Si un message d'erreur met plus de 20 secondes à s'afficher, il peut s'agir d'un problème de connectivité. Consultez l'article [Déboguer les connecteurs Proxy d'application](application-proxy-debug-connectors.md).  |
|5 | Si les problèmes persistent, procédez au débogage du connecteur | Il peut s'agir d'un problème de connectivité entre le proxy et le connecteur ou le connecteur et le serveur principal. Consultez l'article [Déboguer les connecteurs Proxy d'application](application-proxy-debug-connectors.md). |
|6 | Publiez toutes les ressources, vérifiez les outils de développement du navigateur et corrigez les liens | Assurez-vous que le chemin de publication inclut l’ensemble des images, scripts et feuilles de style nécessaires à votre application. Pour plus d'informations, consultez [Ajouter une application locale à Azure AD](application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad). <br></br>Utilisez les outils de développement du navigateur (F12 dans Internet Explorer ou Microsoft Edge) et vérifiez s'il existe d'éventuels problèmes de publication, comme décrit dans [La page d'application ne s'affiche pas correctement](application-proxy-page-appearance-broken-problem.md). <br></br>Pour plus d'informations sur la résolution des liens rompus, consultez [Les liens de la page ne fonctionnent pas](application-proxy-page-links-broken-problem.md). |
|7 | Vérifiez la latence du réseau | Si la page se charge lentement, découvrez comment réduire la latence du réseau dans [Considérations relatives à la réduction de la latence](application-proxy-network-topology.md#considerations-for-reducing-latency). | 
|8 | Consultez l'aide ayant trait à la résolution de problèmes supplémentaires | Si les problèmes persistent, recherchez d’autres articles de résolution des problèmes dans la [documentation de résolution des problèmes liés au service Proxy d'application](application-proxy-troubleshoot.md). |

## <a name="next-steps"></a>Étapes suivantes


* [Publier des applications sur des réseaux et emplacements distincts à l’aide de groupes de connecteurs](application-proxy-connector-groups.md)
* [Travailler avec des serveurs proxy locaux existants](application-proxy-configure-connectors-with-proxy-servers.md)
* [Résoudre les erreurs du proxy d’application et du connecteur](application-proxy-troubleshoot.md)
* [Comment installer silencieusement le connecteur du Proxy d'application Azure AD](application-proxy-register-connector-powershell.md)
