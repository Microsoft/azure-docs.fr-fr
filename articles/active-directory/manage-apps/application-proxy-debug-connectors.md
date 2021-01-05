---
title: Déboguer les connecteurs Proxy d’application - Azure Active Directory | Microsoft Docs
description: Déboguez les problèmes liés aux connecteurs Proxy d’application Azure Active Directory (Azure AD).
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
ms.openlocfilehash: cbbb5328f2d7e814be9b5b94ec522bbb01df39e5
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658245"
---
# <a name="debug-application-proxy-connector-issues"></a>Déboguer les problèmes d’un connecteur Proxy d’application 

Cet article vous aide à résoudre les problèmes liés aux connecteurs Proxy d’application Azure Active Directory (Azure AD). Si vous utilisez le service Proxy d’application pour l’accès à distance à une application web en local, mais que vous ne parvenez pas à vous connecter à l’application, utilisez cet organigramme pour déboguer les problèmes du connecteur. 

## <a name="before-you-begin"></a>Avant de commencer

Cet article suppose que vous avez installé le connecteur Proxy d’application et que vous rencontrez un problème. Lors de la résolution des problèmes du service Proxy d’application, nous vous recommandons de commencer par cette procédure de résolution des problèmes pour déterminer si les connecteurs Proxy d’application sont configurés correctement. Si vous rencontrez toujours des problèmes de connexion à l’application, suivez la procédure de résolution des problèmes dans [Déboguer les problèmes d’application d’un proxy d’application](application-proxy-debug-apps.md).  


Pour plus d’informations sur le service Proxy d’application et l’utilisation de ses connecteurs, consultez :

- [Accès à distance aux applications locales par le bais du service Proxy d'application](application-proxy.md)
- [Connecteurs du service Proxy d’application](application-proxy-connectors.md)
- [Installer et inscrire un connecteur](application-proxy-add-on-premises-application.md)
- [Résoudre les problèmes de proxy d’application et les messages d’erreur](application-proxy-troubleshoot.md)

## <a name="flowchart-for-connector-issues"></a>Organigramme des problèmes de connecteur

Cet organigramme vous guide à travers les étapes du débogage des problèmes de connecteur plus courants. Pour plus d’informations sur chaque étape, consultez le tableau suivant l’organigramme.

![Organigramme illustrant les étapes du débogage d’un connecteur](media/application-proxy-debug-connectors/application-proxy-connector-debugging-flowchart.png)

| Étape | Action | Description |
|---------|---------|---------|
|1 | Recherchez le groupe de connecteurs attribué à l’application | Vous avez probablement un connecteur installé sur plusieurs serveurs, auquel cas les connecteurs doivent être [attribués à des groupes de connecteurs](application-proxy-connector-groups.md#assign-applications-to-your-connector-groups). Pour en savoir plus sur les groupes de connecteurs, consultez [Publier des applications sur des réseaux et emplacements distincts à l’aide de groupes de connecteurs](application-proxy-connector-groups.md). |
|2 | Installez le connecteur et attribuez-lui un groupe | Si vous n’avez pas installé de connecteur, consultez [Installer et inscrire un connecteur](application-proxy-add-on-premises-application.md#install-and-register-a-connector).<br></br> Si vous rencontrez des problèmes lors de l’installation du connecteur, consultez [Problème lors de l’installation du connecteur](application-proxy-connector-installation-problem.md).<br></br> Si le connecteur n’est pas attribué à un groupe, consultez [Attribuer le connecteur à un groupe](application-proxy-connector-groups.md#create-connector-groups).<br></br>Si l’application n’est pas attribuée à un groupe de connecteurs, consultez [Attribuer l’application à un groupe de connecteurs](application-proxy-connector-groups.md#assign-applications-to-your-connector-groups).|
|3 | Exécutez un test de port sur le serveur du connecteur | Sur le serveur du connecteur, exécutez un test de port à l’aide de [telnet](/windows-server/administration/windows-commands/telnet) ou d’un autre outil de test de port pour vérifier que les ports [443 et 80](application-proxy-add-on-premises-application.md#open-ports) sont ouverts.|
|4 | Configurez les domaines et les ports | [Assurez-vous que vos domaines et vos ports sont configurés correctement](application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment). Pour que le connecteur fonctionne correctement, certains ports doivent être ouverts et votre serveur doit pouvoir accéder à certains URL. |
|5 | Vérifiez si un proxy de serveur principal est en cours d’utilisation | Vérifiez si les connecteurs utilisent des serveurs proxy principaux ou les ignorent. Pour en savoir plus, consultez [Résoudre les problèmes courants de proxy de connecteur et de connectivité du service](application-proxy-configure-connectors-with-proxy-servers.md#troubleshoot-connector-proxy-problems-and-service-connectivity-issues). |
|6 | Mettez à jour le connecteur et le programme de mise à jour pour utiliser le proxy principal | Si un proxy de serveur principal est en cours d’utilisation, nous vous recommandons de vérifier que le connecteur utilise le même proxy. Pour plus d’informations sur le dépannage et la configuration des connecteurs pour les utiliser avec des serveurs proxy, consultez [Travailler avec des serveurs proxy locaux existants](application-proxy-configure-connectors-with-proxy-servers.md). |
|7 | Chargez l’URL interne de l’application sur le serveur du connecteur | Sur le serveur de connecteur, chargez l’URL interne de l’application. |
|8 | Vérifiez la connectivité de réseau interne | Votre réseau interne est soumis à un problème de connectivité que cette procédure de débogage ne peut pas diagnostiquer. L’application doit être accessible en interne pour que les connecteurs puissent fonctionner. Vous pouvez activer et afficher les journaux des événements du connecteur, comme décrit dans la section [Connecteurs de proxy d’application](application-proxy-connectors.md#under-the-hood). |
|9 | Augmentez la valeur du délai d’attente sur le serveur principal | Dans les **Paramètres supplémentaires** de votre application, définissez la valeur du champ **Expiration de l’application principale** sur **Long**. Pour en savoir plus, consultez [Ajouter une application locale à Azure AD](application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad). |
|10 | Si les problèmes persistent, penchez-vous sur des points précis de la procédure posant problème, examinez les procédures de débogage de l’application et de l’authentification unique | Suivez la procédure de résolution des problèmes [Déboguer les problèmes d’application d’un proxy d’application](application-proxy-debug-apps.md). |

## <a name="next-steps"></a>Étapes suivantes


* [Publier des applications sur des réseaux et emplacements distincts à l’aide de groupes de connecteurs](application-proxy-connector-groups.md)
* [Travailler avec des serveurs proxy locaux existants](application-proxy-configure-connectors-with-proxy-servers.md)
* [Résoudre les erreurs du proxy d’application et du connecteur](application-proxy-troubleshoot.md)
* [Comment installer silencieusement le connecteur du Proxy d'application Azure AD](application-proxy-register-connector-powershell.md)