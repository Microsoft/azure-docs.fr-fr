---
title: 'Gestion des applications : Bonnes pratiques et recommandations | Microsoft Docs'
description: Apprenez les bonnes pratiques et les recommandations associées à la gestion des applications dans Azure Active Directory. Découvrez l’utilisation du provisionnement automatique et la publication d’applications locales avec Proxy d’application.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/13/2019
ms.subservice: app-mgmt
ms.author: kenwith
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2f5aaf4bbff4c093872228d647d3850667fb9777
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94656405"
---
# <a name="application-management-best-practices"></a>Bonnes pratiques relatives à la gestion des applications

Cet article contient des recommandations et des bonnes pratiques pour la gestion des applications dans Azure Active Directory (Azure AD), lors de l’utilisation du provisionnement automatique et de la publication d’applications locales avec Proxy d’application.

## <a name="cloud-app-and-single-sign-on-recommendations"></a>Recommandations relatives à l’authentification unique et aux applications cloud
| Recommandation | Commentaires |
| --- | --- |
| Vérifier la galerie d’applications Azure AD à la recherche d’applications  | Azure AD comprend une galerie contenant des milliers d’applications pré-intégrées pour lesquelles Enterprise Single Sign-On (SSO de l’entreprise) est activé. Pour obtenir des instructions de configuration propres à une application, consultez la [Liste des tutoriels sur les applications SaaS](../saas-apps/tutorial-list.md).  | 
| Utiliser l’authentification unique SAML fédérée  | Lorsqu’elle est prise en charge par l’application, utilisez l’authentification unique SAML fédérée avec Azure AD plutôt que l’authentification unique basée sur les mots de passe et ADFS.  | 
| Utiliser SHA-256 pour la signature de certificat  | Azure AD utilise l’algorithme SHA-256 par défaut pour signer la réponse SAML. Utilisez SHA-256, sauf si l’application exige SHA-1 (voir [Options de signature de certificat](certificate-signing-options.md) et [Problème de connexion à l’application](application-sign-in-problem-application-error.md).)  | 
| Exiger l’affectation des utilisateurs  | Par défaut, les utilisateurs peuvent accéder aux applications de votre entreprise sans affectation. Toutefois, si l’application expose des rôles, ou si vous souhaitez qu’elle s’affiche sur le panneau Mes applications d’un utilisateur, demandez l’affectation d’utilisateurs. (Consultez le [Guide du développeur pour l’intégration de l’application](developer-guidance-for-integrating-applications.md).)  | 
| Déployer Mes applications pour vos utilisateurs | [Mes applications](end-user-experiences.md) sur `https://myapps.microsoft.com` est un portail web qui fournit aux utilisateurs un point unique d’entrée pour leurs applications cloud affectées. À mesure que des fonctionnalités supplémentaires, telles que la gestion des groupes et la réinitialisation du mot de passe en libre-service, sont ajoutées, les utilisateurs peuvent les trouver dans Mes applications. Consultez [Planifier le déploiement de Mes applications](access-panel-deployment-plan.md).
| Utiliser l’affectation de groupe  | Si cette possibilité est incluse dans votre abonnement, affectez des groupes à une application afin de pouvoir déléguer la gestion des accès en cours au propriétaire du groupe. (Consultez le [Guide du développeur pour l’intégration de l’application](developer-guidance-for-integrating-applications.md).)   | 
| Établir un processus de gestion des certificats | La durée de vie maximale d’un certificat de signature est de trois ans. Pour éviter ou réduire au minimum l’interruption due à l’expiration d’un certificat, utilisez des rôles et des listes de distribution d’e-mails pour vous assurer que les notifications de modifications liées aux certificats sont étroitement supervisées. |

## <a name="provisioning-recommendations"></a>Recommandations relatives au provisionnement
| Recommandation | Commentaires |
| --- | --- |
| Utiliser des tutoriels pour configurer le provisionnement avec les applications cloud | Consultez la [Liste des tutoriels sur les applications SaaS](../saas-apps/tutorial-list.md) pour obtenir des instructions pas à pas sur la configuration du provisionnement de l’application de galerie que vous souhaitez ajouter. |
| Utiliser les journaux de provisionnement (préversion) pour superviser l’état | Les [journaux de provisionnement](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context) fournissent des détails sur toutes les actions effectuées par le service de provisionnement, y compris l’état des utilisateurs individuels. |
| Affecter un groupe de distribution à l’e-mail de notification de provisionnement | Pour augmenter la visibilité des alertes critiques envoyées par le service de provisionnement, attribuez un groupe de distribution au paramètre E-mails de notification. |


## <a name="application-proxy-recommendations"></a>Recommandations relatives à Proxy d’application
| Recommandation | Commentaires |
| --- | --- |
| Utiliser Proxy d’application pour l’accès distant aux ressources internes | Proxy d’application est l’outil recommandé pour permettre aux utilisateurs distants d’accéder aux ressources internes, en se substituant au besoin d’un VPN ou d’un proxy inversé. Il n’est pas prévu pour l’accès aux ressources depuis le réseau d’entreprise, car il peut ajouter de la latence.
| Utiliser des domaines personnalisés | Configurez des domaines personnalisés pour vos applications (voir [Configurer des domaines personnalisés](application-proxy-configure-custom-domain.md)), afin que les URL pour les utilisateurs et entre les applications fonctionnent de l’intérieur ou de l’extérieur de votre réseau. Vous serez également en mesure de contrôler l’identification par votre marque, et de personnaliser vos URL.  Lorsque vous utilisez des noms de domaine personnalisés, prévoyez d’acquérir un certificat public auprès d’une autorité de certification non Microsoft ayant été approuvée. Proxy d’application Azure prend en charge les certificats standard, ([génériques](application-proxy-wildcard.md)) et SAN. (Voir [Planification de Proxy d’application](application-proxy-deployment-plan.md).) |
| Synchroniser les utilisateurs avant de déployer Proxy d’application | Avant le déploiement du proxy d’application, synchronisez les identités utilisateur à partir d’un annuaire local, ou créez-les directement dans Azure AD. La synchronisation des identités permet à Azure AD de pré-authentifier les utilisateurs avant de leur accorder l’accès aux applications publiées via Proxy d’application. Il fournit également les informations d’identifiant utilisateur nécessaires pour effectuer une authentification unique (SSO). (Voir [Planification de Proxy d’application](application-proxy-deployment-plan.md).) |
| Suivre nos conseils sur la haute disponibilité et l’équilibrage de charge | Pour savoir comment le trafic circule entre les utilisateurs, les connecteurs Proxy d’application et les serveurs d’applications principaux, et pour obtenir des conseils sur l’optimisation des performances et de l’équilibrage de charge, consultez [Haute disponibilité et équilibrage de charge de vos applications et connecteurs Proxy d’application](application-proxy-high-availability-load-balancing.md). |
| Utiliser plusieurs connecteurs | Utilisez au moins deux connecteurs Proxy d’application pour une plus grande résilience, disponibilité et mise à l’échelle (voir [Connecteurs Proxy d’application](application-proxy-connectors.md)). Créez des groupes de connecteurs et assurez-vous que chaque groupe de connecteurs possède au moins deux connecteurs (trois connecteurs est le nombre idéal). |
| Localiser les serveurs de connecteur à proximité des serveurs d’applications et s’assurer qu’ils se trouvent dans le même domaine | Pour optimiser les performances, placez physiquement le serveur de connecteur à proximité des serveurs d’applications (voir [Considérations sur la topologie réseau](application-proxy-network-topology.md)). De plus, le serveur de connecteur et les serveurs d’applications web doivent appartenir au même domaine Active Directory, ou couvrir des domaines autorisés. Cette configuration est exigée pour l’authentification unique avec l’authentification Windows intégrée (IWA) et la délégation Kerberos contrainte (KCD). Si les serveurs se trouvent dans des domaines différents, vous devez utiliser la délégation basée sur les ressources pour l’authentification unique (voir [KCD pour l’authentification unique avec Proxy d’application](application-proxy-configure-single-sign-on-with-kcd.md)). |
| Activer les mises à jour automatiques pour les connecteurs | Activez les mises à jour automatiques de vos connecteurs pour les fonctionnalités et résolutions de bogues les plus récentes. Microsoft fournit une prise en charge directe de la version la plus récente du connecteur et de la version précédente. (Voir [Historique des versions Proxy d’application](application-proxy-release-version-history.md).) |
| Ignorer votre proxy local | Pour faciliter la maintenance, configurez le connecteur de manière à ce qu’il contourne votre proxy local et se connecte ainsi directement aux services Azure. (Voir [Connecteurs de Proxy d’application et serveurs proxy](application-proxy-configure-connectors-with-proxy-servers.md).) |
| Utiliser Proxy d’application Azure AD sur Proxy d’application web | Utilisez Proxy d’application Azure AD pour la plupart des scénarios locaux. Proxy d’application web est préférable uniquement dans les scénarios qui demandent un serveur proxy pour AD FS, et dans lesquels vous ne pouvez pas utiliser de domaines personnalisés dans Azure Active Directory. (Voir [Migration de Proxy d’application](application-proxy-migration.md).) |