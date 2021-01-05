---
title: FAQ sur les rapports Azure Active Directory | Microsoft Docs
description: Forum aux questions (FAQ) sur les rapports Azure Active Directory.
services: active-directory
documentationcenter: ''
author: cawrites
manager: MarkusVi
ms.assetid: 534da0b1-7858-4167-9986-7a62fbd10439
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: report-monitor
ms.date: 05/12/2020
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: c274424641ee0173d68f0de9061415815b0df588
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97107428"
---
# <a name="frequently-asked-questions-around-azure-active-directory-reports"></a>Forum aux questions (FAQ) sur les rapports Azure Active Directory

Cet article répond aux questions fréquemment posées sur les rapports Azure Active Directory (Azure AD). Pour plus d’informations, consultez la page [Génération de rapports Azure Active Directory](overview-reports.md). 

## <a name="getting-started"></a>Prise en main 

**Q : J’utilise les API de point de terminaison `https://graph.windows.net/<tenant-name>/reports/` pour tirer (pull) par programmation les rapports d’audit Azure AD et les rapports d’utilisation des applications intégrées dans nos systèmes de création de rapports. Que dois-je utiliser à présent ?**

**R :** Consultez les [Informations de référence sur les API](https://developer.microsoft.com/graph/) pour savoir comment [utiliser les API dans le but d’accéder aux rapports d’activité](concept-reporting-api.md). Ce point de terminaison comporte deux rapports (un d’**audit** et un autre sur les **connexions**) qui fournissent toutes les données dont vous disposiez dans l’ancien point de terminaison d’API. Ce nouveau point de terminaison comporte également un rapport sur les connexions, relatif à la licence Azure AD Premium, que vous pouvez utiliser pour obtenir des informations sur l’utilisation des applications, l’utilisation des appareils et les connexions utilisateur.

---

**Q : J’utilise les API de point de terminaison `https://graph.windows.net/<tenant-name>/reports/` pour tirer (pull) par programmation les rapports de sécurité Azure AD (concernant certains types de détections, tels que les fuites d’informations d’identification ou les connexions à partir d’adresses IP anonymes) dans notre système de création de rapports. Que dois-je utiliser à présent ?**

**R :** Vous pouvez utiliser [l’API Identity Protection relative à la détection des risques](../identity-protection/howto-identity-protection-graph-api.md) pour accéder aux détections de sécurité via Microsoft Graph. Ce nouveau format offre davantage de flexibilité dans la façon dont vous pouvez interroger les données, c’est-à-dire avec un filtrage avancé, une sélection des champs, etc. De plus, il rassemble les détections de risques sous un même type pour faciliter l’intégration aux serveurs SIEM et autres outils de collecte de données. Étant donné que les données ont différents formats, vous ne pouvez pas remplacer vos anciennes requêtes par une nouvelle requête. Toutefois, [la nouvelle API utilise Microsoft Graph](/graph/api/resources/identityriskevent?view=graph-rest-beta), qui est désormais la plateforme standard de Microsoft pour les API telles que Microsoft 365 et Azure AD. Le travail nécessaire peut donc développer vos investissements Microsoft Graph actuels ou vous aider à démarrer la transition vers cette nouvelle plateforme standard.

---

**Q : Comment obtenir une licence Premium ?**

**R :** Consultez [Bien démarrer avec Azure Active Directory Premium](../fundamentals/active-directory-get-started-premium.md) pour mettre à niveau votre édition d’Azure Active Directory.

---

**Q : Au bout de combien de temps puis-je consulter les données d’activité après avoir obtenu une licence Premium ?**

**R :** Si vous avez déjà des données d’activité sous une licence gratuite, vous pouvez les voir immédiatement. Si vous n’avez pas de données, l’affichage des données dans les rapports prend jusqu’à 3 jours.

---

**Q : Puis-je voir les données du mois dernier après avoir obtenu une licence Azure AD Premium ?**

**R :** Si vous êtes récemment passé à une version Premium (y compris une version d’essai), vous pouvez voir les données jusqu’à 7 jours dans un premier temps. Lorsque les données s’accumulent, vous pouvez voir les données au cours des 30 derniers jours.

---

**Q : Faut-il être administrateur général pour voir les activités de connexion sur le portail Azure ou pour obtenir des données par le biais de l’API ?**

**R :** Non, vous pouvez également accéder aux données des rapports via le portail ou via l’API si vous êtes un **Lecteur Sécurité** ou un **Administrateur de sécurité** pour le locataire. Bien sûr, les **administrateurs généraux** ont également accès à ces données.

---


## <a name="activity-logs"></a>Journaux d’activité


**Q : Quelle est la conservation des données pour les journaux d’activité (audit et connexions) dans le portail Azure ?** 

**R :** Pour en savoir plus, consultez [Stratégies de rétention de données des rapports Azure AD](reference-reports-data-retention.md).

---

**Q : Au bout de combien de temps puis-je voir les données d’activité une fois la tâche terminée ?**

**R :** La latence des journaux d’audit est comprise entre 15 minutes et 1 heure. Pour certains enregistrements, la mise à jour des journaux d’activité de connexion peut prendre de 15 minutes à 2 heures.

---

**Q : Puis-je obtenir des informations sur les journaux d’activité de Microsoft 365 par le biais du portail Azure ?**

**R :** Bien que les journaux d’activité de Microsoft 365 et d’Azure AD partagent une grande partie des ressources de l’annuaire, vous devez accéder au [Centre d’administration Microsoft 365](https://admin.microsoft.com) pour obtenir une vue complète des journaux d’activité de Microsoft 365.

---

**Q : Quelles API faut-il utiliser pour obtenir des informations sur les journaux d’activité de Microsoft 365 ?**

**R :** Utilisez les [API Gestion de Microsoft 365](/office/office-365-management-api/office-365-management-apis-overview) pour accéder aux journaux d’activité de Microsoft 365 par le biais d’une API.

---

**Q : Combien d’enregistrements puis-je télécharger à partir du portail Azure ?**

**R :** Vous pouvez télécharger jusqu’à 5 000 enregistrements à partir du portail Azure. Les enregistrements sont triés par *date* et, par défaut, vous obtenez les 5 000 enregistrements les plus récents.

---

## <a name="risky-sign-ins"></a>Connexions risquées

**Q : ldentity Protection m’informe d’une détection de risque, mais je ne vois pas à quelle connexion il correspond dans le rapport sur les connexions. Est-ce normal ?**

**R :** Oui, Identity Protection évalue les risques pour tous les flux d’authentification, qu’ils soient interactifs ou non. Toutefois, seules les connexions interactives sont répertoriées.

---

**Q : Comment faire pour savoir pourquoi une connexion ou un utilisateur ont été signalés comme étant « à risque » dans le portail Azure ?**

**R :** Si vous avez un abonnement **Azure AD Premium**, vous pouvez en savoir plus sur les détections de risques sous-jacentes en sélectionnant l’utilisateur dans **Utilisateurs associés à un indicateur de risque** ou en sélectionnant un enregistrement dans le rapport **Connexions risquées**. Si vous avez un abonnement **Gratuit** ou **De base**, vous pouvez afficher les utilisateurs à risque et les rapports de connexions risquées, mais vous pouvez pas consulter les détections de risques sous-jacentes.

---

**Q : Comment les adresses IP sont-elles calculées dans le rapport des connexions et des connexions à risque ?**

**R :** Les adresses IP sont émises de manière à ce qu’il n’existe aucune connexion définitive entre une adresse IP et l’endroit où se trouve physiquement l’ordinateur avec cette adresse. Le mappage des adresses IP est compliqué par des facteurs tels que les fournisseurs mobiles et les VPN qui émettent des adresses IP à partir de pools centraux souvent très éloignés de l’endroit où l’appareil client est réellement utilisé. Pour le moment, la conversion de l’adresse IP en un emplacement physique constitue la meilleure solution pour les suivis, les données de registre, les recherches inversées et d’autres informations dans les rapports Azure AD. 

---

**Q : Que signifie la détection de risque « Connexion avec un risque supplémentaire détectée » ?**

**R :** Pour vous donner une idée de toutes les connexions à risque dans votre environnement, « Connexion avec un risque supplémentaire détectée » fonctionne comme un conteneur de connexions pour les détections propres aux abonnés d’Azure AD Identity Protection.

---

## <a name="conditional-access"></a>Accès conditionnel

**Q : Quelles sont les nouveautés apportées par cette fonctionnalité ?**

**R :** Les clients peuvent maintenant résoudre les problèmes de stratégies d’accès conditionnel grâce à tous les rapports de connexion. Les clients peuvent examiner l’état de l’accès conditionnel et consulter en détail les stratégies applicables à la connexion, ainsi que les résultats de chaque stratégie.

**Q : Comment faire pour démarrer ?**

**R :** Pour commencer :

* Accédez au rapport de connexion dans le [portail Azure](https://portal.azure.com).
* Cliquez sur la connexion que vous souhaitez dépanner.
* Accédez à l’onglet **Accès conditionnel**. Cet onglet permet de consulter toutes les stratégies ayant eu une incidence sur des connexions, ainsi que le résultat de chaque stratégie. 
    
**Q : Quelles sont toutes les valeurs acceptées par l’état de l’accès conditionnel ?**

**R :** L’état de l’accès conditionnel peut avoir les valeurs suivantes :

* **Non applicable** : Aucune stratégie d’accès conditionnel ne s’appliquait à l’utilisateur et à l’application. 
* **Réussite** : Une stratégie d’accès conditionnel s’appliquait à l’utilisateur et à l’application, et les stratégies d’accès conditionnel ont été respectées. 
* **Échec** : la connexion a satisfait à la condition d’utilisateur et d’application d’au moins une stratégie d’accès conditionnel et les contrôles d’octroi ne sont pas satisfaisants ou ne sont pas configurés pour bloquer l’accès.
    
**Q : Quelles sont toutes les valeurs acceptées par les résultats de la stratégie d’accès conditionnel ?**

**R :** Une stratégie d’accès conditionnel peut aboutir aux résultats suivants :

* **Réussite** : La stratégie a été respectée.
* **Échec** : La stratégie n’a pas été respectée.
* **Non applicable** : Ce résultat peut découler du non-respect des conditions de la stratégie.
* **Inactive** : La stratégie se trouve dans un état désactivé. 
    
**Q : Le nom de la stratégie dans le rapport de connexion ne correspond pas tout le temps au nom de la stratégie d’accès conditionnel. Pourquoi ?**

**R :** Dans tout le rapport de connexion, le nom de la stratégie correspond au nom qu’elle avait au moment de la connexion. Il peut différer si vous l’avez mis à jour ultérieurement, c’est-à-dire après la connexion.

**Q : Mon authentification a été bloquée en raison d’une stratégie d’accès conditionnel, mais le rapport d’activité de connexion indique que la connexion a réussi. Pourquoi ?**

**R :** Le rapport de connexion peut ne pas afficher de résultats précis pour les scénarios Exchange ActiveSync lors de l’application d’un accès conditionnel. Cela peut arriver lorsque le résultat de connexion dans le rapport montre l’établissement d’une connexion, alors que la connexion a en réalité échoué à cause d’une stratégie d’accès conditionnel.
