---
title: Contrôles d’application adaptative dans Azure Security Center
description: Ce document vous aide à utiliser les contrôles d’application adaptatifs dans Azure Security Center pour permettre l’exécution d’applications de liste sur les machines Azure.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 9268b8dd-a327-4e36-918e-0c0b711e99d2
ms.service: security-center
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/06/2020
ms.author: memildin
ms.openlocfilehash: 95c3ad6f689893195e0e5c17c59ab31143cf238d
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93346517"
---
# <a name="use-adaptive-application-controls-to-reduce-your-machines-attack-surfaces"></a>Utiliser des contrôles d’application adaptatifs pour réduire les surfaces d’attaque de vos machines

Découvrez les avantages des contrôles d’application adaptatifs d’Azure Security Center et comment vous pouvez améliorer votre sécurité grâce à cette fonctionnalité intelligente et pilotée par les données.


## <a name="what-are-security-centers-adaptive-application-controls"></a>Que sont les contrôles d’application adaptatifs d’Azure Security Center ?

Les contrôles d’application adaptatifs sont une solution intelligente et automatisée permettant la définition de listes vertes d’applications réputées sécurisées pour vos ordinateurs. 

Souvent, les organisations ont des collections d’ordinateurs qui exécutent régulièrement les mêmes processus. Security Center utilise le Machine Learning pour analyser les applications en cours d’exécution sur vos machines et créer une liste des logiciels réputés sécurisés. Les listes vertes sont basées sur vos charges de travail Azure spécifiques, et vous pouvez personnaliser davantage les recommandations à l’aide des instructions ci-dessous.

Lorsque vous avez activé et configuré des contrôles d’application adaptatifs, vous obtenez des alertes de sécurité si une autre application que celles que vous avez définies comme sécurisées s’exécute.


## <a name="what-are-the-benefits-of-adaptive-application-controls"></a>Quels sont les avantages des contrôles d’application adaptatifs ?

En définissant des listes d’applications réputées sécurisées, et en générant des alertes lorsque d’autres applications sont exécutées, vous pouvez atteindre plusieurs objectifs de renforcement de la sécurité :

- Identifier les programmes malveillants potentiels, y compris ceux qui peuvent être ratés par les logiciels anti-programme malveillant
- Améliorer la conformité avec les stratégies de sécurité locales qui dictent l’utilisation de logiciels sous licence uniquement
- Éviter d’exécuter des applications anciennes ou non prises en charge
- Empêcher les logiciels spécifiques qui sont interdits par votre organisation
- Améliorer la supervision des applications qui accèdent aux données sensibles



## <a name="availability"></a>Disponibilité

|Aspect|Détails|
|----|:----|
|État de sortie :|Disponibilité générale (GA)|
|Prix :|Nécessite [Azure Defender pour les serveurs](defender-for-servers-introduction.md)|
|Machines prises en charge :|![Oui](./media/icons/yes-icon.png) Machines Azure et non-Azure exécutant Windows et Linux<br>![Oui](./media/icons/yes-icon.png) [Machines Azure Arc](../azure-arc/index.yml)|
|Rôles et autorisations obligatoires :|Les rôles **Lecteur de sécurité** et **Lecteur** permettent d’afficher les groupes et les listes d’applications réputées sécurisées<br>Les rôles **Contributeur** et **Administrateur de la sécurité** permettent de modifier les groupes et les listes d’applications réputées sécurisées|
|Clouds :|![Oui](./media/icons/yes-icon.png) Clouds commerciaux<br>![Oui](./media/icons/yes-icon.png) National/souverain (US Gov, Chine Gov, autres Gov)|
|||



## <a name="enable-application-controls-on-a-group-of-machines"></a>Activer les contrôles d’application sur un groupe d’ordinateurs

Si Security Center a identifié des groupes d’ordinateurs dans vos abonnements qui exécutent systématiquement un ensemble similaire d’applications, la recommandation suivante s’affiche : **Les contrôles d’application adaptatifs permettant de définir les applications sécurisées doivent être activés sur vos machines**.

Sélectionnez la recommandation ou ouvrez la page des contrôles d’application adaptatifs pour afficher la liste des applications et des groupes d’ordinateurs à sécurité reconnue recommandés.

1. Ouvrez le tableau de bord Azure Defender et, dans la zone de protection avancée, sélectionnez **Contrôles d’application adaptatifs**.

    :::image type="content" source="./media/security-center-adaptive-application/opening-adaptive-application-control.png" alt-text="Ouverture de contrôles d’application adaptatifs à partir du tableau de bord Azure" lightbox="./media/security-center-adaptive-application/opening-adaptive-application-control.png":::

    La page **Contrôles d’application adaptatifs** s’ouvre avec vos machines virtuelles regroupées dans les onglets suivants :

    - **Configurés**  : groupes d’ordinateurs qui disposent déjà d’une liste verte d’applications définie. Pour chaque groupe, l’onglet Configurés affiche :
        - le nombre de machines dans le groupe ;
        - les alertes récentes.

    - **Recommandés**  : groupes d’ordinateurs qui exécutent régulièrement les mêmes applications et qui n’ont pas de liste verte configurée. Nous vous recommandons d’activer les contrôles d’application adaptatifs pour ces groupes.
    
      > [!TIP]
      > Si vous voyez un nom de groupe avec le préfixe « REVIEWGROUP », il contient des ordinateurs avec une liste d’applications partiellement cohérente. Security Center ne voit pas de modèle, mais recommande d’examiner ce groupe pour voir si _vous_ pouvez définir manuellement certaines règles de contrôles d’application adaptatifs, comme décrit dans [Modification de la règle de contrôle d’application adaptative d’un groupe](#edit-a-groups-adaptive-application-controls-rule).
      >
      > Vous pouvez également déplacer des machines de ce groupe vers d’autres groupes, comme décrit dans [Déplacer une machine d’un groupe vers un autre](#move-a-machine-from-one-group-to-another).

    - **Aucune recommandation**  : les machines sans liste verte d’applications définie et qui ne prennent pas en charge la fonctionnalité. Votre machine peut se trouver dans cet onglet pour les raisons suivantes :
      - Il manque un agent Log Analytics
      - L’agent Log Analytics n’envoie pas d’événements
      - Il s’agit d’un ordinateur Windows avec une stratégie [AppLocker](/windows/security/threat-protection/windows-defender-application-control/applocker/applocker-overview) préexistante activée par un GPO ou une stratégie de sécurité locale

      > [!TIP]
      > Security Center a besoin d’au moins deux semaines de données pour définir les recommandations uniques par groupe d’ordinateurs. Les machines qui ont été créées récemment, ou qui appartiennent à des abonnements qui ont été récemment activés avec Azure Defender, s’affichent sous l’onglet **Aucune recommandation**.


1. Ouvrez l’onglet **Recommandés**. Les groupes de machines avec les listes vertes recommandées s’affichent.

   ![Onglet Recommandés](./media/security-center-adaptive-application/adaptive-application-recommended-tab.png)

1. Sélectionnez un groupe. 

1. Pour configurer votre nouvelle règle, passez en revue les différentes sections de cette page **Configurer des règles de contrôle d’applications** et le contenu, qui sera propre à ce groupe spécifique de machines :

   ![Configurer une nouvelle règle](./media/security-center-adaptive-application/adaptive-application-create-rule.png)

   1. **Sélectionner les machines**  : par défaut, toutes les machines du groupe identifié sont sélectionnées. Désélectionnez tout pour les supprimer de cette règle.
   
   1. **Applications recommandées**  : passez en revue cette liste d’applications qui sont communes aux ordinateurs au sein de ce groupe et dont l’autorisation d’exécution est recommandée.
   
   1. **Plus d’applications**  : passez en revue cette liste d’applications qui sont vues moins fréquemment sur les ordinateurs de ce groupe ou qui sont susceptibles d’être exploitées. Une icône d’avertissement indique qu’une application spécifique peut être utilisée par un attaquant pour ignorer une liste verte d’applications. Nous vous recommandons de passer attentivement en revue ces applications.

      > [!TIP]
      > Les deux listes d’applications incluent l’option permettant de limiter une application spécifique à certains utilisateurs. Adoptez le principe du moindre privilège dans la mesure du possible.
      > 
      > Les applications sont définies par leur éditeur, si une application ne possède pas d’informations d’éditeur (elle n’est pas signée), une règle de chemin d’accès est créée pour le chemin d’accès complet de l’application spécifique.

   1. Pour appliquer la règle, sélectionnez **Audit**. 




## <a name="edit-a-groups-adaptive-application-controls-rule"></a>Modifier la règle de contrôles d’application adaptatifs d’un groupe

Vous pouvez décider de modifier la liste verte pour un groupe d’ordinateurs en raison des modifications connues dans votre organisation. 

Pour modifier les règles d’un groupe d’ordinateurs :

1. Ouvrez le tableau de bord Azure Defender et, dans la zone de protection avancée, sélectionnez **Contrôles d’application adaptatifs**.

1. Dans l’onglet **Configurés** , sélectionnez le groupe dont vous souhaitez modifier la règle.

1. Passez en revue les différentes sections de la page **Configurer les règles de contrôle d’applications** , comme décrit dans [Activer les contrôles d’application adaptatifs sur un groupe d’ordinateurs](#enable-application-controls-on-a-group-of-machines).

1. Si vous le souhaitez, ajoutez une ou plusieurs règles personnalisées :

   1. Sélectionnez **Ajouter une règle**.

      ![Ajouter une règle personnalisée](./media/security-center-adaptive-application/adaptive-application-add-custom-rule.png)

   1. Si vous définissez un chemin d’accès réputé sécurisé, changez le **Type de règle** en « Path » et entrez un chemin d’accès unique. Vous pouvez inclure des caractères génériques dans le chemin d’accès.
   
      > [!TIP]
      > Voici quelques scénarios pour lesquels les caractères génériques dans un chemin d’accès peuvent être utiles :
      > 
      > * Utilisation d’un caractère générique à la fin d’un chemin d’accès pour autoriser tous les exécutables dans ce dossier et ses sous-dossiers.
      > * Utilisation d’un caractère générique au milieu d’un chemin d’accès pour activer un nom d’exécutable connu avec un nom de dossier variable (par exemple, des dossiers utilisateur personnels contenant un exécutable connu, des noms de dossiers générés automatiquement, etc.).
  
   1. Définissez les utilisateurs autorisés et les types de fichiers protégés.

   1. Une fois que vous avez fini de définir la règle, sélectionnez **Ajouter**.

1. Pour appliquer les modifications, sélectionnez **Enregistrer**.


## <a name="review-and-edit-a-groups-settings"></a>Examiner et modifier les paramètres d’un groupe

1. Pour afficher les détails et les paramètres de votre groupe, sélectionnez **Paramètres du groupe**

    Ce volet affiche le nom du groupe (qui peut être modifié), le type de système d’exploitation, l’emplacement et d’autres détails pertinents.

    :::image type="content" source="./media/security-center-adaptive-application/adaptive-application-group-settings.png" alt-text="Page des paramètres de groupe pour les commandes d’applications adaptatives" lightbox="./media/security-center-adaptive-application/adaptive-application-group-settings.png":::

    > [!IMPORTANT]
    > L’option **Appliquer** , dans les paramètres du mode de protection du type de fichier, est grisée dans **tous** les scénarios. Aucune option de mise en application n’est disponible pour l’instant. 
    >
    > :::image type="content" source="./media/security-center-adaptive-application/adaptive-application-modes.png" alt-text="Le mode d’application de la protection des fichiers est définitivement grisé. Aucune option de mise en application n’est disponible.":::

1. Si vous le souhaitez, modifiez le nom du groupe ou les modes de protection du type de fichier.

1. Sélectionnez **Appliquer** et **Enregistrer**.



## <a name="respond-to-the-allowlist-rules-in-your-adaptive-application-control-policy-should-be-updated-recommendation"></a>Répondre à la recommandation « Les règles de liste verte de la stratégie de contrôles d’application adaptatifs doivent être mises à jour »

Cette recommandation s’affiche lorsque l’apprentissage automatique de Security Center identifie un comportement potentiellement légitime qui n’a pas été autorisé précédemment. La recommandation suggère de nouvelles règles pour vos définitions existantes afin de réduire le nombre d’alertes de faux positifs.

Pour corriger les problèmes :

1. Dans la page des recommandations, sélectionnez la recommandation **Les règles de liste verte de la stratégie de contrôles d’application adaptatifs doivent être mises à jour** pour afficher les groupes avec un comportement récemment identifié et potentiellement légitime.

1. Sélectionnez le groupe dont vous souhaitez modifier la règle.

1. Passez en revue les différentes sections de la page **Configurer les règles de contrôle d’applications** , comme décrit dans [Activer les contrôles d’application adaptatifs sur un groupe d’ordinateurs](#enable-application-controls-on-a-group-of-machines).

1. Pour appliquer les modifications, sélectionnez **Audit**.




## <a name="audit-alerts-and-violations"></a>Auditer les alertes et les violations

1. Ouvrez le tableau de bord Azure Defender et, dans la zone de protection avancée, sélectionnez **Contrôles d’application adaptatifs**.

1. Pour afficher les groupes avec des ordinateurs qui ont des alertes récentes, passez en revue les groupes listés dans l’onglet **Configurés**.

1. Pour approfondir vos recherches, sélectionnez un groupe.

   ![Alertes récentes](./media/security-center-adaptive-application/recent-alerts.png)

1. Pour plus d’informations et pour obtenir la liste des machines affectées, sélectionnez une alerte.



## <a name="move-a-machine-from-one-group-to-another"></a>Déplacer une machine d’un groupe à un autre

Lorsque vous déplacez une machine d’un groupe à un autre, la stratégie de contrôle des applications qui lui est appliquée change en fonction des paramètres du groupe de destination. Vous pouvez également déplacer une machine d’un groupe configuré vers un groupe non configuré, ce qui supprime toutes les règles de contrôle d’application qui ont été appliquées à la machine.

1. Ouvrez le tableau de bord Azure Defender et, dans la zone de protection avancée, sélectionnez **Contrôles d’application adaptatifs**.

1. Sur la page **Contrôles d’application adaptatifs** , sous l’onglet **Configurés** , sélectionnez le groupe contenant l’ordinateur à déplacer.

1. Ouvrez la liste des **Machines configurées**.

1. Ouvrez le menu de l’ordinateur à partir de trois points à la fin de la ligne, puis sélectionnez **Déplacer**. Le volet **Déplacer l’ordinateur vers un autre groupe** s’ouvre.

1. Sélectionnez le groupe de destination, puis sélectionnez **Déplacer l’ordinateur**.

1. Pour enregistrer vos modifications, sélectionnez **Enregistrer**.





## <a name="manage-application-controls-via-the-rest-api"></a>Gérer les contrôles d’application via l’API REST 

Pour gérer vos contrôles d’application adaptatifs par programmation, utilisez notre API REST. 

La documentation complète de l’API se trouve [ici](/rest/api/securitycenter/adaptiveapplicationcontrols).

Voici certaines des fonctions disponibles à partir de l’API REST :

* **List** récupère toutes vos recommandations de groupe et fournit un JSON avec un objet pour chaque groupe.

* **Get** récupère le JSON avec les données de recommandation complètes (c’est-à-dire la liste des machines, les règles d’éditeur/de chemin d’accès, etc.).

* **Put** configure votre règle (utilisez le JSON que vous avez récupéré avec **Get** comme corps pour cette demande).
 
   > [!IMPORTANT]
   > La fonction **Put** attend moins de paramètres que le JSON retourné par la commande Get.
   >
   > Supprimez les propriétés suivantes avant d’utiliser le JSON dans la requête Put : recommendationStatus, configurationStatus, issues, location et sourceSystem.




## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez découvert comment utiliser les contrôles d’application adaptatifs dans Azure Security Center pour définir des listes vertes des applications s’exécutant sur vos les machines Azure et non Azure. Pour en savoir plus sur certaines des autres fonctionnalités de protection de la charge de travail cloud de Security Center, consultez :

* [Fonctionnement de l’accès aux machines virtuelles juste-à-temps (JAT)](just-in-time-explained.md)
* [Sécurisation de vos clusters Azure Kubernetes](defender-for-kubernetes-introduction.md)