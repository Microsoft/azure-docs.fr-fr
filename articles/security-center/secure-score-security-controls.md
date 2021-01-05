---
title: Degré de sécurisation dans Azure Security Center
description: Description du degré de sécurisation d’Azure Security Center et de ses contrôles de sécurité
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetd: c42d02e4-201d-4a95-8527-253af903a5c6
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/10/2020
ms.author: memildin
ms.openlocfilehash: 3cd536d051f3e227ba86429ae3f1633bf6c2e82f
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96490531"
---
# <a name="secure-score-in-azure-security-center"></a>Degré de sécurisation dans Azure Security Center

## <a name="introduction-to-secure-score"></a>Présentation du degré de sécurisation

Les deux principaux objectifs d'Azure Security Center sont les suivants : 

- Vous aider à comprendre votre situation actuelle en matière de sécurité
- Vous aider à améliorer efficacement votre sécurité

Pour vous permettre d'atteindre ces objectifs, Security Center met à votre disposition une fonctionnalité appelée **niveau de sécurité**.

Security Center évalue continuellement vos ressources, vos abonnements et votre organisation en recherchant d’éventuels problèmes de sécurité. Il agrège ensuite toutes ses découvertes sous la forme d’un score qui vous permet de déterminer d’un coup d’œil votre niveau de sécurité actuel : plus le score est élevé, plus le niveau de risque identifié est faible.

Sur les pages du portail Azure, le niveau de sécurité est indiqué sous forme de pourcentage, mais les valeurs sous-jacentes sont également clairement présentées :

:::image type="content" source="./media/secure-score-security-controls/single-secure-score-via-ui.png" alt-text="Score de sécurité global comme indiqué dans le portail":::

Pour renforcer votre sécurité, consultez la page des recommandations de Security Center. Chaque recommandation comprend des instructions pour vous aider à résoudre un problème spécifique.

Les recommandations sont regroupées en **contrôles de sécurité**. Chaque contrôle est un groupe logique de recommandations de sécurité associées, et reflète les surfaces d'attaque vulnérables. Votre degré de sécurisation n’augmente que si vous avez suivi *toutes* les recommandations fournies pour une même ressource au sein d’un contrôle. Pour connaître le degré de sécurisation de chacune des surfaces d'attaque de votre organisation, examinez le niveau de sécurité de chaque contrôle de sécurité.

Pour plus d’informations, consultez [Mode de calcul de votre degré de sécurisation](secure-score-security-controls.md#how-your-secure-score-is-calculated), ci-dessous. 


## <a name="access-your-secure-score"></a>Accédez à votre score de sécurité

Vous pouvez connaître votre niveau de sécurité global, ainsi que votre niveau de sécurité par abonnement, via le portail Azure ou par programmation comme décrit dans les sections suivantes :

- [Obtenir votre score de sécurité à partir du portail](#get-your-secure-score-from-the-portal)
- [Obtenir votre score de sécurité à partir de l’API REST](#get-your-secure-score-from-the-rest-api)
- [Obtenir le niveau de sécurité à partir d'Azure Resource Graph (ARG)](#get-your-secure-score-from-azure-resource-graph-arg)

### <a name="get-your-secure-score-from-the-portal"></a>Obtenir votre score de sécurité à partir du portail

Security Center indique le niveau de sécurité de manière bien visible sur le portail puisqu'il s'agit de la première vignette que l'on voit sur la page de présentation. En sélectionnant cette vignette, vous accédez à la page consacrée au niveau de sécurité, où celui-ci est présenté par abonnement. Sélectionnez un abonnement pour voir la liste détaillée des recommandations classées par ordre de priorité ainsi que l'impact que peut avoir leur prise en compte sur le niveau de sécurité de l'abonnement.

Pour récapituler, votre niveau de sécurité apparaît aux emplacements suivants sur les pages du portail de Security Center.

- Sur une vignette de la **page de présentation** de Security Center (tableau de bord principal) :

    :::image type="content" source="./media/secure-score-security-controls/score-on-main-dashboard.png" alt-text="Niveau de sécurité sur le tableau de bord de Security Center":::

- Sur la page **Niveau de sécurité** :

    :::image type="content" source="./media/secure-score-security-controls/score-on-dedicated-dashboard.png" alt-text="Niveau de sécurité sur la page Niveau de sécurité de Security Center":::

- En haut de la page **Recommandations** :

    :::image type="content" source="./media/secure-score-security-controls/score-on-recommendations-page.png" alt-text="Niveau de sécurité sur la page Recommandations de Security Center":::

### <a name="get-your-secure-score-from-the-rest-api"></a>Obtenir votre score de sécurité à partir de l’API REST

Vous pouvez accéder à votre degré de sécurisation par le biais de l’API Degré de sécurisation (actuellement en préversion). Les méthodes de l’API offrent la flexibilité nécessaire pour interroger les données et créer votre propre mécanisme de création de rapports sur vos degrés de sécurisation au fil du temps. Par exemple, vous pouvez utiliser l’API [Degré de sécurisation](/rest/api/securitycenter/securescores) pour obtenir le degré de sécurisation d’un abonnement spécifique. En outre, vous pouvez utiliser l’API [Contrôles du degré de sécurisation](/rest/api/securitycenter/securescorecontrols) pour répertorier les contrôles de sécurité et le degré de sécurisation actuel de vos abonnements.

![Récupération d’un score sécurisé unique via l’API](media/secure-score-security-controls/single-secure-score-via-api.png)

Pour obtenir des exemples d’outils créés sur la base de l’API de degré de sécurisation, consultez [la zone consacrée au degré de sécurisation de notre communauté GitHub](https://github.com/Azure/Azure-Security-Center/tree/master/Secure%20Score). 

### <a name="get-your-secure-score-from-azure-resource-graph-arg"></a>Obtenir le niveau de sécurité à partir d'Azure Resource Graph (ARG)

Azure Resource Graph fournit un accès instantané aux informations relatives aux ressources de vos environnements cloud avec des fonctionnalités robustes de filtrage, de regroupement et de tri. Il s’agit d’un moyen rapide et efficace de demander des informations dans les abonnements Azure par programmation ou depuis le Portail Azure. [Apprenez-en davantage sur Azure Resource Graph](../governance/resource-graph/index.yml).

Pour accéder au niveau de sécurité de plusieurs abonnements avec ARG :

1. Sur le portail Azure, ouvrez l'**Explorateur Azure Resource Graph**.

    :::image type="content" source="./media/security-center-identity-access/opening-resource-graph-explorer.png" alt-text="Lancement de la page de recommandations de l’Explorateur Azure Resource Graph**" :::

1. Entrez votre requête Kusto (en utilisant les exemples ci-dessous à titre indicatif).

    - Cette requête renvoie l'ID d'abonnement, le niveau de sécurité actuel en points et en pourcentage, et le niveau de sécurité maximum de l'abonnement. 

        ```kusto
        SecurityResources 
        | where type == 'microsoft.security/securescores' 
        | extend current = properties.score.current, max = todouble(properties.score.max)
        | project subscriptionId, current, max, percentage = ((current / max)*100)
        ```

    - Cette requête renvoie l'état de tous les contrôles de sécurité. Pour chaque contrôle, vous obtiendrez le nombre de ressources non saines, le niveau de sécurité actuel et le niveau de sécurité maximum. 

        ```kusto
        SecurityResources 
        | where type == 'microsoft.security/securescores/securescorecontrols'
        | extend SecureControl = properties.displayName, unhealthy = properties.unhealthyResourceCount, currentscore = properties.score.current, maxscore = properties.score.max
        | project SecureControl , unhealthy, currentscore, maxscore
        ```

1. Sélectionnez **Exécuter la requête**.




## <a name="tracking-your-secure-score-over-time"></a>Suivi de votre score sécurisé au fil du temps

Si vous êtes un utilisateur Power BI avec un compte Pro, vous pouvez utiliser le tableau de bord Power BI **Score sécurisé au fil du temps** pour suivre votre score sécurisé au fil du temps et examiner les modifications.

> [!TIP]
> Ce tableau de bord, ainsi que d’autres outils permettant de travailler par programmation avec un score sécurisé, sont disponibles dans le domaine dédié de la communauté Azure Security Center sur GitHub : https://github.com/Azure/Azure-Security-Center/tree/master/Secure%20Score

Le tableau de bord contient les deux rapports suivants pour vous aider à analyser votre état de sécurité :

- **Résumé des ressources** : fournit des données résumées concernant l’intégrité de vos ressources.
- **Synthèse des scores sécurisés** : fournit des données résumées concernant la progression de votre score. Utilisez le graphique « Score sécurisé dans le temps par abonnement » pour afficher les modifications apportées au score. Si vous remarquez une modification spectaculaire de votre score, consultez la table « modifications détectées qui peuvent affecter votre score sécurisé » pour les éventuelles modifications susceptibles d’avoir provoqué la modification. Ce tableau présente les ressources supprimées, les ressources récemment déployées ou les ressources dont le statut de sécurité a changé pour l’une des recommandations.

:::image type="content" source="./media/secure-score-security-controls/power-bi-secure-score-dashboard.png" alt-text="Le tableau de bord PowerBI de score sécurisé facultatif pour le suivi de vos scores sécurisés au fil du temps et l’examen des modifications":::





## <a name="how-your-secure-score-is-calculated"></a>Mode de calcul de votre degré de sécurisation 

La contribution de chaque contrôle de sécurité au degré de sécurisation global est indiquée clairement dans la page des recommandations.

[![La version améliorée du degré de sécurisation comprend désormais des contrôles de sécurité](media/secure-score-security-controls/security-controls.png)](media/secure-score-security-controls/security-controls.png#lightbox)

Pour obtenir le nombre maximal de points que peut avoir un contrôle de sécurité, toutes vos ressources doivent se conformer à l’intégralité des recommandations de sécurité fournies dans le contrôle de sécurité. Par exemple, Security Center fournit plusieurs recommandations relatives à la sécurisation de vos ports de gestion. Vous devrez toutes les appliquer pour voir votre degré de sécurisation s’améliorer.

Par exemple, le contrôle de sécurité « Appliquer les mises à jour système » a un degré maximal de six points, que vous pouvez voir dans l’info-bulle concernant l’augmentation potentielle de la valeur du contrôle :

[![Contrôle de sécurité « Appliquer les mises à jour système »](media/secure-score-security-controls/apply-system-updates-control.png)](media/secure-score-security-controls/apply-system-updates-control.png#lightbox)

Le score maximal pour ce contrôle, Appliquer des mises à jour système, est toujours 6. Dans cet exemple, il y a 50 ressources. Nous avons donc divisé le score maximal par 50 et le résultat est que chaque ressource contribue à 0,12 points. 

* **Augmentation potentielle** (0,12 x 8 ressources non saines = 0,96) - Les points disponibles que vous conservez dans le cadre du contrôle. Si vous corrigez toutes les suggestions de ce contrôle, votre score augmentera de 2 % (dans ce cas, 0,96 points arrondis à 1 point). 
* **Score courant** (0,12 x 42 ressources saines = 5,04) - Le score courant pour ce contrôle. Chaque contrôle contribue au score total. Dans cet exemple, le contrôle contribue à 5,04 points au total sécurisé courant.
* **Score maximal** - Nombre maximal de points que vous pouvez gagner en appliquant toutes les recommandations d’un contrôle. Le score maximal d’un contrôle indique l’importance relative de ce contrôle. Utilisez les valeurs du score maximal pour que le triage des problèmes fonctionne en premier. 


### <a name="calculations---understanding-your-score"></a>Calculs : comprendre votre score

|Métrique|Formule et exemple|
|-|-|
|**Degré de sécurisation actuel du contrôle de sécurité**|<br>![Équation pour le calcul du degré de sécurisation d’un contrôle de sécurité](media/secure-score-security-controls/secure-score-equation-single-control.png)<br><br>Chaque contrôle de sécurité individuel contribue au degré de sécurisation global. Chaque ressource concernée par une recommandation dans le contrôle contribue au degré de sécurisation actuel du contrôle. Le degré de sécurisation actuel de chaque contrôle correspond à la mesure de l’état des ressources *comprises* dans ce contrôle.<br>![Info-bulles présentant les valeurs utilisées lors du calcul du degré de sécurisation actuel du contrôle de sécurité](media/secure-score-security-controls/security-control-scoring-tooltips.png)<br>Dans cet exemple, le score (ou degré) maximal de 6 est divisé par 78, car il s’agit de la somme des ressources saines et non saines.<br>6 / 78 = 0,0769<br>En multipliant ce score par le nombre de ressources saines (4), vous obtenez le score actuel :<br>0,0769 \* 4 = **0,31**<br><br>|
|**Degré de sécurisation**<br>Abonnement unique|<br>![Équation pour le calcul du score sécurisé d’un abonnement](media/secure-score-security-controls/secure-score-equation-single-sub.png)<br><br>![Degré de sécurisation d’un abonnement avec tous les contrôles activés](media/secure-score-security-controls/secure-score-example-single-sub.png)<br>Dans cet exemple, vous pouvez voir un abonnement pour lequel tous les contrôles de sécurité sont disponibles (avec un degré de sécurisation maximal potentiel de 60 points). Le degré de sécurisation indique 28 points sur 60 points possibles, et les 32 points restants correspondent aux valeurs figurant dans la colonne « Potential score increase » (Augmentation potentielle du degré de sécurisation) des contrôles de sécurité.<br>![Liste des contrôles et augmentation potentielle du score](media/secure-score-security-controls/secure-score-example-single-sub-recs.png)|
|**Degré de sécurisation**<br>Abonnements multiples|<br>![Équation pour le calcul du score sécurisé pour plusieurs abonnements](media/secure-score-security-controls/secure-score-equation-multiple-subs.png)<br><br>Lors du calcul du score combiné pour plusieurs abonnements, Security Center comprend un *poids* pour chaque abonnement. Les pondérations relatives de vos abonnements sont déterminées par Security Center en fonction de facteurs tels que le nombre de ressources.<br>Le score actuel de chaque abonnement est calculé de la même façon que pour un abonnement unique, mais le poids est appliqué comme indiqué dans l’équation.<br>Quand vous consultez plusieurs abonnements, le degré de sécurisation évalue toutes les ressources de toutes les stratégies activées, et regroupe leur impact combiné sur le degré maximal de chaque contrôle de sécurité.<br>![Degré de sécurisation pour plusieurs abonnements avec tous les contrôles activés](media/secure-score-security-controls/secure-score-example-multiple-subs.png)<br>Le score combiné **n’est pas** une moyenne. Il s’agit plutôt d’une évaluation de l’état de toutes les ressources de tous les abonnements.<br>Ici aussi, si vous accédez à la page des recommandations et si vous ajoutez les points que vous pouvez potentiellement gagner, vous constaterez qu’il s’agit de la différence entre le score actuel (24) et le score maximal possible (60).|
||||

### <a name="which-recommendations-are-included-in-the-secure-score-calculations"></a>Quelles sont les recommandations incluses dans les calculs du degré de sécurisation ?

Seules les recommandations intégrées ont un impact sur le degré de sécurisation.

Les recommandations marquées **Preview (Préversion)** ne sont pas incluses dans les calculs de votre degré de sécurisation. Elles doivent tout de même être corrigées dans la mesure du possible, ainsi lorsque la période de préversion se termine, elles seront prises en compte dans le calcul.

Exemple de recommandation de préversion :

:::image type="content" source="./media/secure-score-security-controls/example-of-preview-recommendation.png" alt-text="Recommandation portant l’indicateur Preview (Préversion)":::

## <a name="improve-your-secure-score"></a>Améliorer votre score de sécurité

Pour améliorer votre degré de sécurisation, appliquez les recommandations de sécurité qui figurent dans la liste des recommandations. Vous pouvez appliquer chaque recommandation manuellement à chaque ressource, ou à l’aide de l’option **Correction rapide !** (si celle-ci est disponible) pour appliquer rapidement une recommandation à un groupe de ressources. Pour plus d’informations, consultez [Appliquer des recommandations](security-center-remediate-recommendations.md).

Une autre façon d’améliorer votre score et de vous assurer que vos utilisateurs ne créent pas de ressources ayant un impact négatif sur votre score consiste à configurer les options « Appliquer » et « Refuser » sur les recommandations pertinentes. Pour plus d’informations, consultez [Empêcher des configurations incorrectes à l’aide des recommandations Appliquer/Refuser](prevent-misconfigurations.md).

## <a name="security-controls-and-their-recommendations"></a>Contrôles de sécurité et leurs recommandations

Le tableau ci-dessous liste les contrôles de sécurité d’Azure Security Center. Pour chaque contrôle, vous pouvez voir le nombre maximal de points que vous pouvez ajouter à votre degré de sécurisation si vous appliquez *toutes* les recommandations indiquées dans le contrôle, pour *toutes* vos ressources. 

L’ensemble de recommandations de sécurité fournies par Security Center est adapté aux ressources disponibles dans l’environnement de chaque organisation. Les recommandations peuvent être personnalisées davantage en [désactivant les stratégies](tutorial-security-policy.md#disable-security-policies-and-disable-recommendations) et en [exemptant des ressources spécifiques d’une recommandation](exempt-resource.md). 
 
Nous recommandons à chaque organisation d’examiner attentivement les initiatives Azure Policy qui lui ont été attribuées. 

> [!TIP]
> Pour plus d’informations sur l’examen et la modification de vos initiatives, consultez [Utilisation de stratégies de sécurité](tutorial-security-policy.md). 

Bien que l’initiative de sécurité par défaut de Security Center soit basée sur les meilleures pratiques et les normes du secteur, il existe des scénarios dans lesquels les recommandations intégrées répertoriées ci-dessous peuvent ne pas être entièrement adaptées à votre organisation. Par conséquent, il est parfois nécessaire d’ajuster l’initiative par défaut (sans compromettre la sécurité) pour s’assurer qu’elle est alignée sur les propres stratégies de votre organisation, les normes du secteur, les normes réglementaires et les benchmarks que vous êtes tenu de respecter.<br><br>
<div class="foo">

<style type="text/css"> .tg  {border-collapse:collapse;border-spacing:0;} .tg td{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px; overflow:hidden;padding:10px 5px;word-break:normal;} .tg th{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:18px; font-weight:normal;overflow:hidden;padding:10px 5px;word-break:normal;} .tg .tg-cly1{text-align:left;vertical-align:middle} .tg .tg-lboi{border-color:inherit;text-align:left;vertical-align:middle} </style>
<table class="tg">
<thead>
  <tr>
    <th class="tg-cly1"><b>Contrôle de sécurité, score et description</b><br></th>
    <th class="tg-cly1"><b>Recommandations</b></th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Activer MFA (score max. de 10)</p></strong>Si vous n’utilisez qu’un mot de passe pour authentifier un utilisateur, le vecteur d’attaque reste ouvert. Si le mot de passe est faible ou s’il a été exposé ailleurs, est-ce vraiment l’utilisateur qui se connecte avec le nom d’utilisateur et le mot de passe ?<br>Lorsque <a href="https://www.microsoft.com/security/business/identity/mfa">MFA</a> est activée, vos comptes sont plus sûrs et les utilisateurs peuvent toujours s’authentifier auprès de presque n’importe quelle application avec l’authentification unique (SSO).</td>
    <td class="tg-lboi"; width=55%>- MFA doit être activée sur les comptes disposant d’autorisations de propriétaire sur votre abonnement<br>- MFA doit être activée sur les comptes disposant d’autorisations d’écriture sur votre abonnement</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Sécuriser les ports de gestion (score maximal de 8)</p></strong>Les attaques par force brute ciblent des ports de gestion pour accéder à une machine virtuelle. Étant donné que les ports ne doivent pas être toujours ouverts, une stratégie d’atténuation consiste à réduire l’exposition aux ports à l’aide de contrôles d’accès réseau juste-à-temps, de groupes de sécurité réseau et de la gestion des ports de machines virtuelles.<br>Étant donné que de nombreuses organisations informatiques ne bloquent pas les communications SSH sortantes de leur réseau, les attaquants peuvent créer des tunnels chiffrés qui autorisent les ports RDP sur les systèmes infectés à communiquer en retour avec la commande de l’attaquant pour contrôler les serveurs. Les attaquants peuvent utiliser le sous-système Windows Remote Management pour se déplacer latéralement dans votre environnement et utiliser des informations d'identification volées pour accéder à d’autres ressources sur un réseau.</td>
    <td class="tg-lboi"; width=55%>- Les ports de gestion des machines virtuelles doivent être protégés par un contrôle d’accès réseau juste-à-temps<br>- Les machines virtuelles doivent être associées à un groupe de sécurité réseau<br>- Les ports de gestion doivent être fermés sur vos machines virtuelles</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Appliquer des mises à jour système (score maximal de 6)</p></strong>Les mises à jour système offrent aux organisations la possibilité de maintenir l’efficacité opérationnelle, de réduire les vulnérabilités de sécurité et de fournir un environnement plus stable aux utilisateurs finaux. Si vous n’appliquez pas de mises à jour, les vulnérabilités et les résultats des environnements qui sont susceptibles d’être attaqués ne sont pas corrigés. Ces vulnérabilités peuvent être exploitées et entraîner une perte de données, une exfiltration des données, un rançongiciel et un abus de ressources. Pour déployer ces mises à jour système, vous pouvez utiliser la <a href="/azure/automation/update-management/overview">solution Update Management pour gérer les mises à jour et les correctifs</a> pour vos machines virtuelles. La gestion des mises à jour consiste à contrôler le déploiement et la maintenance des versions logicielles.</td>
    <td class="tg-lboi"; width=55%>- Les problèmes d’intégrité de l’agent d’analyse doivent être résolus sur vos machines<br>- L’agent d’analyse doit être installé sur des groupes de machines virtuelles identiques<br>- L’agent d’analyse doit être installé sur vos machines<br>- La version du système d’exploitation doit être mise à jour pour vos rôles de service cloud<br>- Les mises à jour système sur les groupes de machines virtuelles identiques doivent être installées<br>- Les mises à jour système doivent être installées sur vos machines<br>- Vos machines doivent être redémarrées pour appliquer les mises à jour système<br>- Les services Kubernetes doivent être mis à niveau vers une version non vulnérable de Kubernetes<br>- L’agent d’analyse doit être installé sur vos machines virtuelles<br>- L’agent Log Analytics doit être installé sur vos machines Azure Arc basées sur Windows (préversion)<br>- L’agent Log Analytics doit être installé sur vos machines Azure Arc basées sur Linux (préversion)</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Corriger les vulnérabilités (score maximal de 6)</p></strong>Une vulnérabilité est une faiblesse qui peut être exploitée par un intervenant de menace, pour compromettre la confidentialité, la disponibilité ou l’intégrité d’une ressource. <a href="/windows/security/threat-protection/microsoft-defender-atp/next-gen-threat-and-vuln-mgt">La gestion des vulnérabilités</a> réduit l’exposition de l’organisation, renforce la surface d’exposition des points de terminaison, augmente la résilience de l’organisation et réduit la surface d’attaque de vos ressources. La gestion des menaces et des vulnérabilités permet de visualiser les erreurs de configuration des logiciels et de la sécurité et de fournir des suggestions en matière d’atténuation.</td>
    <td class="tg-lboi"; width=55%>- La sécurité avancée des données doit être activée sur SQL Database<br>- Les vulnérabilités dans les images Azure Container Registry doivent être corrigées<br>- Les vulnérabilités sur vos bases de données SQL doivent être corrigées<br>- Les vulnérabilités doivent être corrigées avec une solution d’évaluation des vulnérabilités<br>- L’évaluation des vulnérabilités doit être activée sur SQL Managed Instance<br>- L’évaluation des vulnérabilités doit être activée sur vos serveurs SQL<br>- La solution d’évaluation des vulnérabilités doit être installée sur vos machines virtuelles<br>- Les images conteneur doivent être déployées à partir de registres approuvés uniquement (préversion)<br>- Le module complémentaire Azure Policy pour Kubernetes doit être installé et activé sur vos clusters (préversion)</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Activer le chiffrement au repos (score maximal de 4)</p></strong><a href="/azure/security/fundamentals/encryption-atrest">Le chiffrement au repos</a> offre une protection des données pour les données stockées. Les attaques contre les données au repos sont notamment des tentatives d’obtenir un accès physique au matériel sur lequel les données sont stockées. Azure utilise le chiffrement symétrique pour chiffrer et déchiffrer de grandes quantités de données au repos. Une clé de chiffrement symétrique est utilisée pour chiffrer les données au fil de leur stockage. Cette clé de chiffrement est utilisée pour déchiffrer ces données au fil de leur préparation à une utilisation en mémoire. Les clés doivent être stockées dans un emplacement sécurisé doté du contrôle d’accès basé sur l’identité et de stratégies d’audit. L’un de ces emplacements sécurisés est Azure Key Vault. Si un attaquant obtient les données chiffrées, mais sans les clés de chiffrement, il doit pirater le chiffrement pour avoir accès aux données.</td>
    <td class="tg-lboi"; width=55%>- Le chiffrement de disque doit être appliqué aux machines virtuelles<br>- Transparent Data Encryption sur les bases de données SQL doit être activé<br>- Les variables de compte Automation doivent être chiffrées<br>- La propriété ClusterProtectionLevel doit être définie sur EncryptAndSign pour les clusters Service Fabric<br>- Le protecteur TDE du serveur SQL doit être chiffré avec votre propre clé</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Chiffrer des données en transit (score maximal de 4)</p></strong>Les données sont « en transit » lorsqu’elles sont transférées entre des composants, des emplacements ou des programmes. Les organisations qui ne parviennent pas à protéger les données en transit sont plus sensibles aux attaques de l’intercepteur, aux écoutes électroniques et au piratage de session. Les protocoles SSL/TLS doivent être utilisés pour échanger des données et un VPN est recommandé. Lors de l’envoi de données chiffrées entre une machine virtuelle Azure et un emplacement local, via Internet, vous pouvez utiliser une passerelle de réseau virtuel telle que la <a href="/azure/vpn-gateway/vpn-gateway-about-vpngateways">Passerelle VPN Azure</a> pour envoyer du trafic chiffré.</td>
    <td class="tg-lboi"; width=55%>- L’application API doit uniquement être accessible via HTTPS<br>- L’application de fonction doit uniquement être accessible via HTTPS<br>- Seules les connexions sécurisées à votre cache Redis doivent être activées<br>- La sécurisation du transfert vers des comptes de stockage doit être activée<br>- L’application web doit uniquement être accessible via HTTPS<br>- Le point de terminaison privé doit être activé pour les serveurs PostgreSQL<br>- L’application de la connexion SSL doit être activée pour les serveurs de base de données PostgreSQL<br>- L’application de la connexion SSL doit être activée pour les serveurs de base de données MySQL<br>- TLS doit être mis à jour vers la dernière version pour votre application API<br>- TLS doit être mis à jour vers la dernière version pour votre application de fonction<br>- TLS doit être mis à jour vers la dernière version pour votre application web<br>- FTPS doit être exigé dans votre application API<br>- FTPS doit être exigé dans votre application de fonction<br>- FTPS doit être exigé dans votre application web</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Gérer l’accès et les autorisations (score maximal de 4)</p></strong>L’un des principaux éléments d’un programme de sécurité est de s’assurer que vos utilisateurs bénéficient de l’accès nécessaire pour effectuer leurs tâches, mais pas plus que ce qui suit : le <a href="/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models">modèle d’accès au moindre privilège</a>.<br>Contrôlez l’accès à vos ressources en créant des attributions de rôles avec le <a href="/azure/role-based-access-control/overview">contrôle d’accès en fonction du rôle Azure (Azure RBAC)</a>. Une attribution de rôle se compose de trois éléments :<br>- <strong>Principal de sécurité</strong> : l’objet auquel l’utilisateur demande l’accès<br>- <strong>Définition du rôle</strong> : leurs autorisations<br>- <strong>Étendue</strong> : l’ensemble des ressources auxquelles les autorisations s’appliquent</td>
    <td class="tg-lboi"; width=55%>- Les comptes déconseillés doivent être supprimés de votre abonnement (préversion)<br>- Les comptes déconseillés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement (préversion)<br>- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement (préversion)<br>- Les comptes externes disposant d’autorisations d’écriture doivent être supprimés de votre abonnement (préversion)<br>- Plusieurs propriétaires doivent être attribués à votre abonnement<br>- Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) doit être utilisé sur les services Kubernetes (préversion)<br>- Les clusters Service Fabric ne doivent utiliser Azure Active Directory que pour l’authentification des clients<br>-Des principaux de service doivent être utilisés pour protéger vos abonnements à la place des certificats de gestion<br>- Les fonctionnalités Linux les moins privilégiées doivent être appliquées pour les conteneurs (préversion)<br>- Le système de fichiers racine immuable (en lecture seule) doit être appliqué pour les conteneurs (préversion)<br>- Tout conteneur avec une élévation des privilèges doit être évité (préversion)<br>- L’exécution des conteneurs en tant qu’utilisateur racine doit être évitée (préversion)<br>- Éviter les conteneurs partageant des espaces de noms d’hôte sensibles (préversion)<br>- L’utilisation des montages de volume HostPath de pad doit être limitée à une liste connue (préversion)<br>- Les conteneurs privilégiés doivent être évités (préversion)<br>- Le module complémentaire Azure Policy pour Kubernetes doit être installé et activé sur vos clusters (préversion)<br>- Les applications web doivent exiger un certificat SSL pour toutes les demandes entrantes<br>- Une identité managée doit être utilisée dans votre application API<br>- Une identité managée doit être utilisée dans votre application de fonction<br>- Une identité managée doit être utilisée dans votre application web</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Corriger les configurations de sécurité (score maximal de 4)</p></strong>Les ressources informatiques mal configurées présentent un risque plus élevé d’attaque. Les actions de renforcement de base sont souvent oubliées lorsque des ressources sont déployées et que des échéances doivent être respectées. Les erreurs de configuration de la sécurité peuvent être à n’importe quel niveau de l’infrastructure : des systèmes d’exploitation et des appareils réseau aux ressources cloud.<br>Azure Security Center compare continuellement la configuration de vos ressources avec les exigences des normes, réglementations et tests d’évaluation du secteur. Lorsque vous avez configuré les « packages de conformité » appropriés (normes et lignes de base) qui comptent dans le cadre de votre organisation, tous les écarts entraînent des suggestions en matière de sécurité qui incluent le CCEID et une explication de l’impact potentiel sur la sécurité.<br>Les packages les plus fréquemment utilisés sont <a href="/azure/security/benchmarks/introduction">Azure Security Benchmark</a> et <a href="https://www.cisecurity.org/benchmark/azure/">CIS Microsoft Azure Foundations Benchmark version 1.1.0</a></td>
    <td class="tg-lboi"; width=55%>- Les vulnérabilités dans les configurations de la sécurité des conteneurs doivent être corrigées<br>- Les vulnérabilités dans la configuration de la sécurité sur vos machines doivent être corrigées<br>- Les vulnérabilités dans la configuration de la sécurité de vos groupes de machines virtuelles identiques doivent être corrigées<br>- L’agent d’analyse doit être installé sur vos machines virtuelles<br>- L’agent d’analyse doit être installé sur vos machines<br>- L’agent Log Analytics doit être installé sur vos machines Azure Arc basées sur Windows (préversion)<br>- L’agent Log Analytics doit être installé sur vos machines Azure Arc basées sur Linux (préversion)<br>- L’agent d’analyse doit être installé sur des groupes de machines virtuelles identiques<br>- Les problèmes d’intégrité de l’agent d’analyse doivent être résolus sur vos machines<br>- Le remplacement ou la désactivation du profil AppArmor des conteneurs doit être limité (préversion)<br>- Le module complémentaire Azure Policy pour Kubernetes doit être installé et activé sur vos clusters (préversion)</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Restreindre l’accès réseau non autorisé (score maximal de 4)</p></strong>Les points de terminaison au sein d’une organisation fournissent une connexion directe entre votre réseau virtuel et les services Azure pris en charge. Les machines virtuelles d’un sous-réseau peuvent communiquer avec toutes les ressources. Vous pouvez limiter les communications vers et à partir de toutes les ressources d’un sous-réseau par la création d’un groupe de sécurité réseau et son association au sous-réseau. Les organisations peuvent limiter et se protéger contre le trafic non autorisé en créant des règles de trafic entrant et sortant.</td>
    <td class="tg-lboi"; width=55%>- Le transfert IP doit être désactivé sur votre machine virtuelle<br>- Des plages d’adresses IP autorisées doivent être définies sur les services Kubernetes (préversion)<br>- (DÉCONSEILLÉ) L’accès à App Services doit être restreint (préversion)<br>- (DÉCONSEILLÉ) Les règles relatives aux applications web doivent être renforcées sur les groupes de sécurité réseau IaaS<br>- Les machines virtuelles doivent être associées à un groupe de sécurité réseau<br>- CORS ne doit pas autoriser toutes les ressources à accéder à votre application API<br>- CORS ne doit pas autoriser toutes les ressources à accéder à votre application de fonction<br>- CORS ne doit pas autoriser toutes les ressources à accéder à vos applications web<br>- Le débogage à distance doit être désactivé pour l’application API<br>- Le débogage à distance doit être désactivé pour l’application de fonction<br>- Le débogage à distance doit être désactivé pour les applications web<br>- L’accès doit être limité pour les groupes de sécurité réseau permissifs avec machines virtuelles connectées à Internet<br>- Les règles de groupes de sécurité réseau pour les machines virtuelles connectées à Internet doivent être renforcées<br>- Le module complémentaire Azure Policy pour Kubernetes doit être installé et activé sur vos clusters (préversion)<br>- Les conteneurs doivent écouter uniquement sur les ports autorisés (préversion)<br>- Les services doivent écouter uniquement sur les ports autorisés (préversion)<br>- L’utilisation du réseau hôte et des ports doit être limitée (préversion)<br>Les réseaux virtuels - doivent être protégés par le pare-feu Azure (préversion)<br>- Le point de terminaison privé doit être activé pour les serveurs MariaDB<br>- Le point de terminaison privé doit être activé pour les serveurs MySQL<br>- Le point de terminaison privé doit être activé pour les serveurs PostgreSQL</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Appliquer un contrôle d’application adaptatif (score maximal de 3)</p></strong>Le contrôle d’application adaptatif (AAC) est une solution de bout en bout intelligente et automatisée qui vous permet de contrôler les applications qui peuvent s’exécuter sur vos machines Azure et non-Azure. Il permet également de renforcer vos ordinateurs contre les logiciels malveillants.<br>Security Center utilise le Machine Learning pour créer une liste d’applications connues pour un groupe d’ordinateurs.<br>Cette approche innovante de la liste verte d’applications offre des avantages en matière de sécurité sans la complexité de la gestion.<br>AAC est particulièrement utile pour les serveurs spécialement conçus qui doivent exécuter un ensemble spécifique d’applications.</td>
    <td class="tg-lboi"; width=55%>- Des contrôles d’application adaptatifs doivent être activés sur les machines virtuelles<br>- L’agent d’analyse doit être installé sur vos machines virtuelles<br>- L’agent d’analyse doit être installé sur vos machines<br>- L’agent Log Analytics doit être installé sur vos machines Azure Arc basées sur Windows (préversion)<br>- L’agent Log Analytics doit être installé sur vos machines Azure Arc basées sur Linux (préversion)<br>- Les problèmes d’intégrité de l’agent d’analyse doivent être résolus sur vos machines</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Appliquer la classification des données (score maximal de 2)</p></strong>La classification des données de votre organisation par la sensibilité et l’impact sur l’activité vous permet de déterminer et d’attribuer la valeur aux données et fournit la stratégie et la base de la gouvernance.<br><a href="/azure/information-protection/what-is-information-protection">Azure Information Protection</a> peut aider à classer des données. Il utilise des stratégies de chiffrement, d’identité et d’autorisation pour éviter et limiter la perte de données. Certaines classifications utilisées par Microsoft sont non professionnelles, publiques, générales, confidentielles et hautement confidentielles.</td>
    <td class="tg-lboi"; width=55%>- Les données sensibles de vos bases de données SQL doivent être classifiées (préversion)</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Protéger les applications contre les attaques DDoS (score maximal de 2)</p></strong>Les attaques par déni de service distribué (DDoS) saturent les ressources et rendent les applications inutilisables. Utilisez <a href="/azure/virtual-network/ddos-protection-overview">Azure DDoS Protection Standard</a> pour défendre votre organisation contre les trois principaux types d’attaques DDoS :<br>Les - <strong>attaques volumétriques</strong> saturent le réseau avec un trafic légitime. La protection DDoS Standard limite ces attaques en les absorbant ou en les lisant à vitesse variable automatiquement.<br>Les - <strong>attaques de protocole</strong> rendent une cible inaccessible, en exploitant des faiblesses dans la pile de protocoles des couches 3 et 4. La protection DDoS Standard limite ces attaques en bloquant le trafic malveillant.<br>Les - <strong>attaques par couche de ressource (application)</strong> ciblent les paquets d’application web. Protégez-vous contre ce type avec un pare-feu d’applications web et une protection DDoS Standard.</td>
    <td class="tg-lboi"; width=55%>- La protection DDoS Standard doit être activée<br>- Les limites de mémoire et du processeur du conteneur doivent être appliquées (préversion)<br>- Le module complémentaire Azure Policy pour Kubernetes doit être installé et activé sur vos clusters (préversion)</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Activer la protection du point de terminaison (score maximal de 2)</p></strong>Pour garantir la protection de vos points de terminaison contre les programmes malveillants, les capteurs comportementaux collectent et traitent les données des systèmes d’exploitation de vos points de terminaison et envoient ces données au cloud privé pour analyse. L’analytique de sécurité exploite les Big Data, le Machine Learning et d’autres sources pour suggérer des réponses aux menaces. Par exemple, <a href="/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection">Microsoft Defender ATP</a> utilise l’intelligence des menaces pour identifier les méthodes d’attaque et générer des alertes de sécurité.<br>Security Center prend en charge les solutions de protection des points de terminaison suivantes : Windows Defender, System Center Endpoint Protection, Trend Micro, Symantec v12.1.1.1100, McAfee v10 pour Windows, McAfee v10 pour Linux et Sophos v9 pour Linux. Si Security Center détecte l’une de ces solutions, la suggestion relative à l’installation de protection du point de terminaison n’apparaîtra plus.</td>
    <td class="tg-lboi"; width=55%>- Les échecs d’intégrité de protection du point de terminaison doivent être corrigés sur les groupes de machines virtuelles identiques<br>- Les problèmes d’intégrité de la protection du point de terminaison doivent être résolus sur vos machines<br>- La solution de protection du point de terminaison doit être installée sur les groupes de machines virtuelles identiques<br>- Installer la solution de protection du point de terminaison sur les machines virtuelles<br>- Les problèmes d’intégrité de l’agent d’analyse doivent être résolus sur vos machines<br>- L’agent d’analyse doit être installé sur des groupes de machines virtuelles identiques<br>- L’agent d’analyse doit être installé sur vos machines<br>- L’agent d’analyse doit être installé sur vos machines virtuelles<br>- L’agent Log Analytics doit être installé sur vos machines Azure Arc basées sur Windows (préversion)<br>- L’agent Log Analytics doit être installé sur vos machines Azure Arc basées sur Linux (préversion)<br>- Installer la solution de protection du point de terminaison sur vos machines</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Activer l’audit et la journalisation (score maximal de 1)</p></strong>Les données de journalisation fournissent des aperçus sur les problèmes précédents, empêchent les éventuels autres, peuvent améliorer les performances de l’application et permettent d’automatiser des actions qui seraient autrement manuelles.<br>Les - <strong>journaux d’activité de contrôle et de gestion</strong> fournissent des informations sur les opérations <a href="/azure/azure-resource-manager/management/overview">Azure Resource Manager</a>.<br>Les - <strong>journaux de plan de données</strong> fournissent des informations sur les événements déclenchés lors de l’utilisation des ressources Azure.<br>Les - <strong>événements traités</strong> fournissent des informations sur les événements/alertes analysés qui ont été traités.</td>
    <td class="tg-lboi"; width=55%>- L’audit sur le serveur SQL doit être activé<br>- Les journaux de diagnostic dans App Services doivent être activés<br>- Les journaux de diagnostic dans Azure Data Lake Store doivent être activés<br>- Les journaux de diagnostic dans Azure Stream Analytics doivent être activés<br>- Les journaux de diagnostic dans les comptes Batch doivent être activés<br>- Les journaux de diagnostic dans Data Lake Analytics doivent être activés<br>- Les journaux de diagnostic dans Event Hub doivent être activés<br>- Les journaux de diagnostic dans IoT Hub doivent être activés<br>- Les journaux de diagnostic dans Key Vault doivent être activés<br>- Les journaux de diagnostic dans Logic Apps doivent être activés<br>- Les journaux de diagnostic dans les services Search doivent être activés<br>- Les journaux de diagnostic dans Service Bus doivent être activés<br>- Les journaux de diagnostic dans les groupes identiques de machines virtuelles doivent être activés<br>- Les règles d'alerte de métriques doivent être configurées sur des comptes Batch<br>- Les paramètres d’audit SQL doivent avoir des groupes d’actions configurés pour capturer les activités critiques<br>- Les serveurs SQL doivent être configurés avec une période de rétention d’audit supérieure à 90 jours.</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Activer la protection avancée contre les menaces (score maximal 0)</p></strong>Les plans facultatifs de protection contre les menaces Azure Defender d’Azure Security Center fournissent des défenses complètes pour votre environnement. Quand Security Center détecte une menace dans un domaines de votre environnement, il génère une alerte. Ces alertes décrivent les détails des ressources affectées, les étapes de correction suggérées et, dans certains cas, l’option permettant de déclencher une application logique en réponse.<br>Chaque plan Azure Defender est une offre distincte et facultative que vous pouvez activer à l’aide de la recommandation correspondante dans ce contrôle de sécurité.<br><a href="/azure/security-center/threat-protection">En savoir davantage sur la protection contre les menaces de Security Center</a>.</td>
    <td class="tg-lboi"; width=55%>- Advanced Data Security doit être activé sur les serveurs Azure SQL Database<br>- Advanced Data Security doit être activé sur les serveurs SQL sur les machines<br>- Advanced Threat Protection doit être activé sur les machines virtuelles<br>- Advanced Threat Protection doit être activé sur les plans Azure App Service<br>- Advanced Threat Protection doit être activé sur les comptes Stockage Azure<br>- Advanced Threat Protection doit être activé sur les clusters Azure Kubernetes Service<br>- Advanced Threat Protection doit être activé sur les registres Azure Container Registry<br>- Advanced Threat Protection doit être activé sur les coffres Azure Key Vault</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Implémenter les meilleures pratiques relatives à la sécurité (score maximal de 0)</p></strong>Les pratiques de sécurité modernes « supposent une violation » du périmètre du réseau. Pour cette raison, un grand nombre des meilleures pratiques dans ce contrôle se focalisent sur la gestion des identités.<br>La perte de clés ou d'informations d’identification est un problème courant. <a href="/azure/key-vault/key-vault-overview">Azure Key Vault</a> protège les clés et les secrets en chiffrant les clés, les fichiers. pfx et les mots de passe.<br>Les réseaux privés virtuels (VPN) sont un moyen sécurisé d’accéder à vos machines virtuelles. Si les VPN ne sont pas disponibles, utilisez des phrases secrètes complexes et une authentification à deux facteurs, notamment <a href="/azure/active-directory/authentication/concept-mfa-howitworks">l’authentification multifacteur Azure AD</a>. L’authentification à deux facteurs évite les faiblesses inhérentes à la confiance uniquement basée sur les noms d’utilisateur et les mots de passe.<br>L’utilisation d’une authentification forte et des plateformes d’autorisation est une autre meilleure pratique. L’utilisation d’identités fédérées permet aux organisations de déléguer la gestion des identités autorisées. Cela est également important lorsque le contrat des employés est résilié et que leur accès doit être révoqué.</td>
    <td class="tg-lboi"; width=55%>- 3 propriétaires au maximum doivent être désignés pour votre abonnement<br>- Les comptes externes disposant d’autorisations de lecture doivent être supprimés de votre abonnement<br>- L’authentification multifacteur (MFA) doit être activée sur les comptes disposant d’autorisations de lecture sur votre abonnement<br>- L’accès aux comptes de stockage avec des configurations de pare-feu et de réseau virtuel doit être limité<br>- Toutes les règles d’autorisation, sauf RootManageSharedAccessKey, doivent être supprimées de l’espace de noms Event Hub<br>- Un administrateur Azure Active Directory doit être provisionné pour les serveurs SQL<br>- La sécurité avancée des données doit être activée sur vos instances gérées<br>- Les règles d’autorisation sur l’instance Event Hub doivent être définies<br>- Les comptes de stockage doivent être migrés vers de nouvelles ressources Azure Resource Manager<br>- Les machines virtuelles doivent être migrées vers de nouvelles ressources Azure Resource Manager<br>- Les sous-réseaux doivent être associés à un groupe de sécurité réseau<br>- [Préversion] Exploit Guar Windows doit être activé <br>- [Préversion] L’agent de configuration Invité doit être installé<br>- Les machines virtuelles non connectées à Internet doivent être protégées avec des groupes de sécurité réseau<br>- Sauvegarde Azure doit être activé pour les machines virtuelles<br>- La sauvegarde géoredondante doit être activée pour Azure Database for MariaDB<br>- La sauvegarde géoredondante doit être activée pour Azure Database pour MySQL<br>- La sauvegarde géoredondante doit être activée pour Azure Database pour PostgreSQL<br>- PHP doit être mis à jour vers la dernière version pour votre application API<br>- PHP doit être mis à jour vers la dernière version pour votre application web<br>- Java doit être mis à jour vers la dernière version pour votre application API<br>- Java doit être mis à jour vers la dernière version pour votre application de fonction<br>- Java doit être mis à jour vers la dernière version pour votre application web<br>- Python doit être mis à jour vers la dernière version pour votre application API<br>- Python doit être mis à jour vers la dernière version pour votre application de fonction<br>- Python doit être mis à jour vers la dernière version pour votre application web<br>- La conservation des audits pour les serveurs SQL Server doit être définie sur au moins 90 jours</td>
  </tr>
</tbody>
</table>


</div>




## <a name="secure-score-faq"></a>Questions fréquentes (FAQ) sur le degré de sécurisation

### <a name="if-i-address-only-three-out-of-four-recommendations-in-a-security-control-will-my-secure-score-change"></a>Si je n’applique que trois recommandations sur quatre dans un contrôle de sécurité, mon degré de sécurisation changera-t-il ?
Non. Il ne changera pas tant que vous n’aurez pas appliqué toutes les recommandations fournies pour une même ressource. Pour obtenir le score maximal d’un contrôle, vous devez appliquer toutes les recommandations de l’ensemble des ressources.

### <a name="if-a-recommendation-isnt-applicable-to-me-and-i-disable-it-in-the-policy-will-my-security-control-be-fulfilled-and-my-secure-score-updated"></a>Si je désactive une recommandation dans ma stratégie car elle ne s’applique pas à mon cas, mon contrôle de sécurité sera-t-il respecté et mon degré de sécurisation sera-t-il mis à jour ?
Oui. Nous vous recommandons de désactiver les recommandations qui ne s’appliquent pas à votre environnement. Pour obtenir des instructions sur la désactivation d’une recommandation, consultez [Désactiver des stratégies de sécurité](./tutorial-security-policy.md#disable-security-policies-and-disable-recommendations).

### <a name="if-a-security-control-offers-me-zero-points-towards-my-secure-score-should-i-ignore-it"></a>Si un contrôle de sécurité n’ajoute aucun point à mon degré de sécurisation, dois-je l’ignorer ?
Dans certains cas, vous verrez un score de contrôle maximal supérieur à zéro, mais son impact est égal à zéro. Lorsque le score incrémentiel relatif à la correction de ressources est négligeable, celui-ci est arrondi à zéro. N’ignorez pas ces recommandations, car elles offrent malgré tout des améliorations au niveau de la sécurité. La seule exception concerne le contrôle « Bonne pratique supplémentaire ». L’application de ces recommandations n’augmentera pas votre score, mais elle améliorera votre sécurité globale.

## <a name="next-steps"></a>Étapes suivantes

Cet article a décrit le degré de sécurisation, ainsi que les contrôles de sécurité qu’il introduit. Pour des informations connexes, consultez les articles suivants :

- [En savoir plus sur les différents éléments d’une recommandation](security-center-recommendations.md)
- [Découvrez comment appliquer les recommandations](security-center-remediate-recommendations.md)
- [Consulter les outils basés sur GitHub pour travailler par programmation avec un score sécurisé](https://github.com/Azure/Azure-Security-Center/tree/master/Secure%20Score)