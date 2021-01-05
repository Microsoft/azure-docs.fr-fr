---
title: Utilisation d’Azure AD Connect Health avec AD FS | Microsoft Docs
description: Ceci est la page d’Azure AD Connect Health spécifiant comment surveiller votre infrastructure AD FS locale.
services: active-directory
documentationcenter: ''
ms.reviewer: zhiweiwangmsft
author: billmath
manager: daveba
editor: curtand
ms.assetid: dc0e53d8-403e-462a-9543-164eaa7dd8b3
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 02/26/2019
ms.author: billmath
ms.custom: H1Hack27Feb2017
ms.collection: M365-identity-device-management
ms.openlocfilehash: 26fdf202cb9bcacee94c83578432f7a399f90a0c
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91306274"
---
# <a name="monitor-ad-fs-using-azure-ad-connect-health"></a>Surveiller AD FS avec Azure AD Connect Health
La documentation suivante est spécifique à la surveillance de votre infrastructure AD FS avec Azure AD Connect Health. Pour plus d’informations sur la surveillance de la synchronisation Azure AD Connect avec Azure AD Connect Health, consultez [Utilisation d’Azure AD Connect Health pour la synchronisation](how-to-connect-health-sync.md). En outre, pour plus d’informations sur la surveillance des services de domaine Active Directory avec Azure AD Connect Health, consultez [Utilisation d’Azure AD Connect Health avec AD DS](how-to-connect-health-adds.md).

## <a name="alerts-for-ad-fs"></a>Alertes pour AD FS
Cette section vous fournit une liste des alertes actives. Chaque alerte inclut les informations associées, la procédure de résolution et des liens vers la documentation afférente.

Vous pouvez double-cliquer sur une alerte active ou résolue pour ouvrir un nouveau panneau comportant des informations supplémentaires, une procédure de résolution de l’alerte et des liens vers de la documentation pertinente. Vous pouvez également afficher des données d’historique sur les alertes résolues par le passé.

![Capture d’écran montrant la page « Alertes » d’Azure AD Connect Health avec une alerte sélectionnée et la fenêtre « Détails de l’alerte » affichée.](./media/how-to-connect-health-adfs/alert2.png)

## <a name="usage-analytics-for-ad-fs"></a>Analyse de l’utilisation pour AD FS
L’analyse de l’utilisation d’Azure AD Connect Health observe le trafic d’authentification des serveurs de fédération. Vous pouvez double-cliquer sur la zone Analyse de l’utilisation pour ouvrir le panneau du même nom, qui vous indique plusieurs mesures et regroupements.

> [!NOTE]
> Pour utiliser l’analyse de l’utilisation avec AD FS, vous devez vous assurer que l’audit AD FS est activé. Pour plus d’informations, consultez [Activer l’audit pour AD FS](how-to-connect-health-agent-install.md#enable-auditing-for-ad-fs).
>
>

![Capture d’écran montrant la page « Analyse de l’utilisation » d’Azure AD Connect Health.](./media/how-to-connect-health-adfs/report1.png)

Pour sélectionner des mesures supplémentaires, spécifiez un intervalle de temps ou, pour modifier le regroupement, cliquez avec le bouton droit sur le graphique d’analyse de l’utilisation, puis sélectionnez Modifier le graphique. Vous pouvez ensuite spécifier l’intervalle de temps, sélectionner une autre mesure et modifier le regroupement. Vous pouvez afficher la distribution du trafic d’authentification en fonction de « mesures » différentes et regrouper les mesures en fonction des paramètres de « regroupement » appropriés décrits dans la section suivante :

**Métrique : Nombre total de demandes** : nombre total de demandes traitées par les serveurs AD FS.

|Regrouper par | En quoi consiste le regroupement ? Quel est son intérêt ? |
| --- | --- |
| Tous | Montre le nombre total de demandes traitées par tous les serveurs AD FS.|
| Application | Regroupe la totalité des demandes en fonction de la partie de confiance cible. Ce regroupement permet de comprendre le pourcentage du trafic total reçu par chaque application. |
|  Serveur |Regroupe la totalité des demandes en fonction du serveur qui a traité la demande. Ce regroupement permet de mieux comprendre la distribution de charge du trafic total.
| Jonction d’espace de travail |Regroupe la totalité des demandes en fonction de la provenance des demandes, selon qu’elles soient ou non issues d’appareils joints à un espace de travail (connu). Ce regroupement permet de comprendre si vos ressources sont consultées à l’aide d’appareils inconnus de l’infrastructure d’identité. |
|  Méthode d'authentification | Regroupe la totalité des demandes en fonction de la méthode d’authentification utilisée. Ce regroupement permet de mieux comprendre la méthode courante d’authentification utilisée. Voici les méthodes d’authentification possibles : <ol> <li>Authentification Windows intégrée (Windows)</li> <li>Authentification basée sur les formulaires (Formulaires)</li> <li>SSO (Authentification unique)</li> <li>Authentification par certificat X509 (Certificat)</li> <br>Si les serveurs de fédération reçoivent la requête avec un cookie d’authentification unique, cette requête est considérée comme une authentification unique (SSO). Le cas échéant, si le cookie est valide, l’utilisateur ne doit pas fournir d’informations d’identification et bénéficie d’un accès transparent à l’application. Ce comportement est courant si vous disposez de multiples parties de confiance protégées par les serveurs de fédération. |
| Emplacement réseau | Regroupe la totalité des demandes en fonction de l’emplacement du réseau de l’utilisateur. Il peut s’agir d’un réseau intranet ou extranet. Ce regroupement permet de comparer les pourcentages de trafic issus de l’intranet et de l’extranet. |


**Métrique : Nombre total de requêtes échouées** : nombre total de requêtes échouées traitées par le service de fédération. (Cette métrique est disponible uniquement sur AD FS pour Windows Server 2012 R2)

|Regrouper par | En quoi consiste le regroupement ? Quel est son intérêt ? |
| --- | --- |
| Type d’erreur | Indique le nombre d’erreurs en fonction des types d’erreurs prédéfinis. Ce regroupement permet de mieux comprendre les types courants d’erreurs. <ul><li>Nom d’utilisateur ou mot de passe incorrect : erreurs dues à un nom d’utilisateur ou mot de passe incorrect.</li> <li>« Verrouillage extranet » : défaillances provoquées par les requêtes reçues d’un utilisateur ne disposant d’aucun accès à l’extranet. </li><li> « Mot de passe expiré » : défaillances provoquées par des utilisateurs se connectant avec des mots de passe expirés.</li><li>« Compte désactivé » : défaillances dues aux utilisateurs se connectant avec un compte désactivé.</li><li>« Authentification de l’appareil » : défaillances provoquées par les utilisateurs ne réussissant pas à s’authentifier à l’aide de l’authentification de l’appareil.</li><li>« Authentification par certificat utilisateur » : défaillances dues aux utilisateurs ne réussissant pas à se connecter à cause d’un certificat non valide.</li><li>« MFA » : défaillances dues aux utilisateurs ne réussissant pas à s’identifier à l’aide de l’authentification multifacteur (Multi Factor Authentication, MFA).</li><li>« Autres informations d’identification » : « Autorisation d’émission » : défaillances dues à des défauts d’autorisation.</li><li>« Délégation d’émission » : défaillances dues à des erreurs de délégation d’émission.</li><li>« Acceptation de jetons » : défaillances dues au rejet par ADFS du jeton d’un fournisseur d’identité tiers.</li><li>« Protocole » : défaillances dues à des erreurs de protocoles.</li><li>« Inconnu » : intercepte tout. Toutes les autres défaillances qui n’entrent pas dans les catégories définies.</li> |
| Serveur | Regroupe les erreurs en fonction du serveur. Ce regroupement permet de mieux comprendre la distribution des erreurs entre les serveurs. Une distribution inégale peut indiquer qu’un serveur présente un état défectueux. |
| Emplacement réseau | Regroupe les erreurs en fonction de l’emplacement réseau des demandes (intranet et extranet). Ce regroupement procure une meilleure visibilité sur le type des demandes défaillantes. |
|  Application | Regroupe les défaillances en fonction de l’application cible (partie de confiance). Ce regroupement permet d’identifier l’application cible présentant le nombre le plus important d’erreurs. |

**Métrique : Nombre d’utilisateurs** : nombre moyen d’utilisateurs uniques qui utilisent activement l’authentification AD FS

|Regrouper par | En quoi consiste le regroupement ? Quel est son intérêt ? |
| --- | --- |
|Tous |Cette mesure fournit le nombre moyen de personnes utilisant le service de fédération durant l’intervalle sélectionné. Les utilisateurs ne sont pas regroupés. <br>La moyenne dépend de la tranche horaire sélectionnée. |
| Application |Regroupe le nombre moyen d’utilisateurs sur l’application cible (partie de confiance). Ce regroupement procure une plus grande visibilité sur le nombre d’utilisateurs exécutant chaque application. |

## <a name="performance-monitoring-for-ad-fs"></a>Surveillance des performances pour AD FS
L’analyse des performances Azure AD Connect Health fournit des informations d’analyse sur les mesures. Si vous sélectionnez la zone Surveillance, un panneau contenant des informations détaillées sur les mesures s’ouvre.

![Capture d’écran montrant la page « Surveillance des performances » d’Azure AD Connect Health.](./media/how-to-connect-health-adfs/perf1.png)

Si vous sélectionnez l’option Filtre en haut du panneau, vous pouvez filtrer par serveur afin d’afficher les mesures spécifiques à chacun d’entre eux. Pour modifier la métrique, cliquez avec le bouton droit sur le graphique de surveillance situé sous le panneau de surveillance, puis sélectionnez Modifier le graphique (ou cliquez sur le bouton Modifier le graphique). Dans le nouveau panneau ouvert, vous pouvez sélectionner des mesures supplémentaires à partir de la liste déroulante et spécifier un intervalle de temps pour l’affichage des données de performances.

## <a name="top-50-users-with-failed-usernamepassword-logins"></a>Les 50 utilisateurs dont la combinaison nom d’utilisateur/mot de passe échoue le plus souvent.
Des informations d’identification erronées sont souvent à l’origine d’une demande d’authentification ayant échoué sur un serveur AD, autrement dit, un nom d’utilisateur ou un mot de passe incorrect. Ces erreurs sont généralement le résultat de mots de passe trop complexes, oubliés ou de fautes de frappe.

Mais il existe également d’autres raisons pouvant entraîner un nombre anormalement élevé de demandes traitées par vos serveurs AD FS, par exemple : une application qui met en cache des informations d’identification utilisateur, qui expirent par la suite, ou un utilisateur malveillant qui tente de se connecter à un compte à l’aide d’une série de mots de passe bien connus. Ces deux exemples peuvent conduire à une augmentation considérable des demandes.

Azure AD Connect Health pour AD FS fournit un rapport sur les 50 utilisateurs dont les tentatives de connexion échouent le plus fréquemment en raison d’un mot de passe ou d’un nom d’utilisateur non valide. Ce rapport est le résultat du traitement de tous les événements d’audit générés par les serveurs AD FS dans les batteries de serveurs.

![Capture d’écran montrant la section « Rapports » avec le nombre de tentatives d’entrée de mot de passe incorrect au cours des 30 derniers jours.](./media/how-to-connect-health-adfs/report1a.png)

Dans ce rapport, vous pouvez facilement retrouver les informations suivantes :

* Nombre total de demandes ayant échoué en raison d’un nom d’utilisateur/mot de passe incorrect au cours des 30 derniers jours
* Nombre moyen d’utilisateurs dont la connexion a échoué en raison d’un nom d’utilisateur/mot de passe incorrect par jour.

Lorsque vous cliquez sur cette section, vous accédez au panneau de rapport principal, qui fournit des détails supplémentaires. Ce panneau inclut un graphique contenant des informations sur les tendances qui permettent d’établir un point de comparaison des demandes dont le nom d’utilisateur ou le mot de passe est incorrect. En outre, il fournit la liste des 50 utilisateurs présentant le plus grand nombre de tentatives infructueuses au cours de la dernière semaine. Remarquez que les 50 premiers utilisateurs de la dernière semaine ont aidé à l’identification des pics de mots de passe incorrects.  

Ce graphique fournit les informations suivantes :

* Le nombre total d’échecs de connexion en raison d’un nom d’utilisateur/mot de passe incorrect sur une base quotidienne.
* Le nombre total d’utilisateurs uniques dont les tentatives de connexion ont échoué sur une base quotidienne.
* Adresse IP du client pour la dernière demande

![portail Azure AD Connect Health](./media/how-to-connect-health-adfs/report3a.png)

Ce rapport fournit les informations suivantes :

| Élément de rapport | Description |
| --- | --- |
| ID d'utilisateur |Affiche l’ID d’utilisateur qui a été utilisé. Cette valeur correspond à ce que l’utilisateur a tapé, ce qui n’est pas toujours l’ID d’utilisateur correct. |
| Tentatives ayant échoué |Affiche le nombre total de tentatives ayant échoué pour cet ID d’utilisateur. Le tableau est trié par nombre décroissant de tentatives ayant échoué. |
| Dernier échec |Affiche l’horodatage du dernier échec. |
| IP du dernier échec |Affiche l’adresse IP du client à partir de la dernière requête incorrecte. Si vous voyez plusieurs adresses IP dans cette valeur, elle peut inclure l’adresse IP cliente de transfert avec l’adresse IP de requête de dernière tentative de l’utilisateur.  |

> [!NOTE]
> Ce rapport est automatiquement mis à jour toutes les 12 heures avec les nouvelles informations collectées durant ce laps de temps. En conséquence, les tentatives de connexion effectuées dans les 12 dernières heures peuvent ne pas être incluses dans le rapport.

## <a name="related-links"></a>Liens connexes
* [Azure AD Connect Health](./whatis-azure-ad-connect.md)
* [Installation de l’agent Azure AD Connect Health](how-to-connect-health-agent-install.md)
* [Rapport de l’adresse IP risquée](how-to-connect-health-adfs-risky-ip.md)