---
title: Guide de sécurité d’Azure Stack Edge Pro R | Microsoft Docs
description: Décrit les conventions, les instructions et les considérations de sécurité, et explique comment installer et utiliser de manière sécurisée un appareil Azure Stack Edge Pro R.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 10/26/2020
ms.author: alkohli
ms.openlocfilehash: ac3f2cc1c68ea552b2858d932217a28055fee0fd
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465371"
---
# <a name="azure-stack-edge-pro-r-safety-instructions"></a>Instructions de sécurité pour Azure Stack Edge Pro R

![Icône Avertissement](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)
![Icône Lire l’avis de sécurité](./media/azure-stack-edge-pro-r-safety/icon-safety-read-all-instructions.png)
**LIRE LES INFORMATIONS SUR LA SANTÉ ET LA SÉCURITÉ**

Lisez l’ensemble des informations de sécurité contenues dans cet article avant d’utiliser votre appareil Azure Stack Edge Pro R. Le non-respect des instructions peut entraîner un incendie ou une dégradation de vos biens et être à l’origine d’électrocutions et de blessures. Lisez toutes les informations de sécurité ci-dessous avant d’utiliser Azure Stack Edge Pro R.

## <a name="safety-icon-conventions"></a>Conventions des icônes de sécurité

Voici les termes signalétiques correspondant aux panneaux d’alerte de danger :

| Icône | Description |
|:--- |:--- |
| ![Symbole de danger](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)| **DANGER :** Signale une situation dangereuse qui, si elle n’est pas évitée, entraînera la mort ou des blessures graves. <br> **AVERTISSEMENT :** Signale une situation dangereuse qui, si elle n’est pas évitée, risque d’entraîner la mort ou des blessures graves. <br> **ATTENTION :** Signale une situation dangereuse qui, si elle n’est pas évitée, risque d’entraîner des blessures légères ou moyennement graves.|
|

Les icônes de danger suivantes doivent être observées lors de la configuration et de l’exécution de votre appareil Edge Azure Stack Edge Pro R :

| Icône | Description |
|:--- |:--- |
| ![Lire toutes les instructions au préalable](./media/azure-stack-edge-pro-r-safety/icon-safety-read-all-instructions.png) | Lire toutes les instructions au préalable |
| ![Symbole de danger](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png) | Symbole de danger |
| ![Icône de risque de basculement](./media/azure-stack-edge-pro-r-safety/icon-safety-tip-hazard.png)  | Risque de basculement|
| ![Icône de poids important](./media/azure-stack-edge-pro-r-safety/icon-safety-heavy-weight.png)  | Risque de poids important|
| ![Icône de choc électrique](./media/azure-stack-edge-pro-r-safety/icon-safety-electric-shock.png) | Risque d’électrocution |
| ![Icône d'absence de pièces réparables par l'utilisateur](./media/azure-stack-edge-pro-r-safety/icon-safety-do-not-access.png) | Aucune pièce remplaçable par l’utilisateur. Accès réservé au personnel formé à cet effet. |
| ![Icône Plusieurs sources d’alimentation](./media/azure-stack-edge-pro-r-safety/icon-safety-disconnect-all-power.png)  | Plusieurs sources d’alimentation. Débranchez tous les cordons d’alimentation pour supprimer toute alimentation de l’équipement. |
| ![Icône Points de pincement](./media/azure-stack-edge-pro-r-safety/icon-pinching-points.png)  | Des points de pincement sont présents. |
| ![Icône Composants ou surfaces chauds](./media/azure-stack-edge-pro-r-safety/icon-hot-component-surface.png)  | Indique des composants ou des surfaces chauds. |
|

## <a name="handling-precautions-and-site-selection"></a>Précautions de manipulation et sélection du site

![Icône Avertissement](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png) **AVERTISSEMENT :**

* L’équipement approprié (par exemple, un transpalette) et l’équipement de protection individuelle, par exemple, des gants, doivent être utilisés pour déplacer et manipuler l’appareil tel qu’il est livré.

![Icône Avertissement](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)![Icône Risque de basculement](./media/azure-stack-edge-pro-r-safety/icon-safety-tip-hazard.png) **AVERTISSEMENT :**

* Placez l’équipement sur une surface plane, dure et stable pour éviter tout risque de basculement ou d’écrasement.
* N’empilez pas plus de 2 appareils les uns sur les autres.
* Assurez-vous que le système est sécurisé avant de procéder à un empilement.
* Ne déplacez pas des appareils empilés.
* N’empilez pas des appareils qui sont alimentés avec des câbles externes.
* Assurez-vous que les cordons d’alimentation ne sont pas écrasés ou endommagés pendant les opérations d’empilement.
* Les appareils ne doivent pas être utilisés comme tables ou espaces de travail.

![Icône Avertissement](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)
![Icône Risque d’électrocution](./media/azure-stack-edge-pro-r-safety/icon-safety-electric-shock.png)
![Icône Aucune pièce remplaçable par l’utilisateur](./media/azure-stack-edge-pro-r-safety/icon-safety-do-not-access.png) **ATTENTION :**

* Inspectez l’appareil *à réception* pour identifier les dommages éventuels. Si le boîtier de l’appareil est endommagé, [contactez le Support Microsoft](azure-stack-edge-contact-microsoft-support.md) pour bénéficier d’un remplacement. N’essayez pas d’utiliser l’appareil.
* Si vous pensez que l’appareil est défectueux, [contactez le Support Microsoft](azure-stack-edge-contact-microsoft-support.md) pour bénéficier d’un remplacement. N’essayez pas de réparer l’appareil.
* L’appareil ne contient aucune pièce remplaçable par l’utilisateur. L’intérieur de l’appareil présente des niveaux de tension, de courant et d’énergie dangereux. N’ouvrez pas l’appareil. Retournez l’appareil à Microsoft pour toute réparation.

![Icône Avertissement](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png) ![Icône Objet lourd](./media/azure-stack-edge-pro-r-safety/icon-safety-heavy-weight.png) **AVERTISSEMENT :**

* Le retrait du module d’alimentation de l’onduleur expose des composants de ce dernier qui sont sous tension. N’insérez pas d’objets étrangers dans le compartiment du module d’alimentation.
* N’essayez pas de soulever un appareil Edge Azure Stack Edge Pro R tout seul. Un boîtier peut peser entre 52 et 93 kg (115 à 205 livres) ; utilisez une assistance mécanique ou toute autre assistance appropriée lors du déplacement et du levage de l’équipement. Respectez les exigences locales en matière de santé et de sécurité au travail lors du déplacement et du levage de l’équipement.
* N’essayez pas de soulever l’équipement sans l’aide mécanique appropriée. Sachez que le fait de tenter de soulever ce poids peut entraîner des blessures graves.

![Icône Avertissement](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)  **AVERTISSEMENT :**

* Le système est conçu pour fonctionner dans un environnement contrôlé. Choisissez un site qui est :
  * Bien ventilé et loin des sources de chaleur, y compris les radiateurs et le soleil.
  * Non exposé à l’humidité, ni à la pluie.
  * Dans un espace dans lequel les vibrations et les chocs physiques sont limités.  Le système est conçu pour les chocs et les vibrations conformément à la norme MIL-STD-810G.
  * Isolé des champs électromagnétiques forts produits par des appareils électriques.
  * Muni de prises correctement mises à la terre.
  * Dotés d’un espace adéquat pour accéder au(x) cordon(s) d’alimentation, car ils servent de sectionneur d’alimentation principal de l’appareil.
* Installez l’équipement dans une zone de travail suffisamment aérée. Vérifiez que les capots avant et arrière sont entièrement retirés pendant que l’appareil est en cours de fonctionnement.
* Installez l’équipement dans une zone à température contrôlée exempte de particules conductrices, et veillez à ce que la circulation d’air autour de l’équipement soit adéquate.
* Conservez l’équipement à l’écart de sources de liquide et d’environnements excessivement humides.
* Ne laissez aucun liquide ni aucun objet étranger entrer dans le système. Ne placez pas de boissons ni d’autres récipients contenant des liquides sur le système ou à proximité.

## <a name="heater-precautions"></a>Précautions liées au chauffage

![Icône Avertissement](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png) ![Icône Composants ou surfaces chauds 1](./media/azure-stack-edge-pro-r-safety/icon-safety-electric-shock.png) **AVERTISSEMENT :** 

* L’opération de chauffage automatique pendant que le système est sous tension peut créer un danger au toucher en raison des températures élevées à la surface du capot de l’ensemble de chauffage. Ne touchez pas cette surface pendant que le système est sous tension. Prévoyez une période de refroidissement de 10 minutes après la mise hors tension du système.

![Icône Avertissement](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png) ![Icône Points de pincement 1](./media/azure-stack-edge-pro-r-safety/icon-safety-electric-shock.png) **AVERTISSEMENT :** 

* Quand le système est sous tension, l’actionnement automatique de la porte de ventilation arrière peut créer un risque de point de pincement. Ne touchez pas cette zone quand le système est sous tension.

## <a name="electrical-precautions"></a>Consignes de sécurité électrique

![Icône Avertissement](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png) ![Icône Risque d’électrocution](./media/azure-stack-edge-pro-r-safety/icon-safety-electric-shock.png) **AVERTISSEMENT :**

* Fournissez un raccordement à la masse sûr pour le cordon d’alimentation. Le cordon de courant alternatif (CA) est doté d’une fiche de terre à trois fils (fiche présentant une broche de mise à la terre). Cette fiche convient uniquement aux prises secteur mises à la terre. Veillez à ne pas compromettre le rôle de la broche de mise à la terre.
* La fiche du cordon d’alimentation étant le moyen principal de débrancher l’alimentation, assurez-vous que les prises de courant sont situées à proximité de l’appareil et facilement accessibles.
* En présence de l’une des conditions suivantes, débranchez le ou les cordons d’alimentation (en tirant sur la fiche et non sur le cordon) et déconnectez tous les câbles :

  * Le cordon d’alimentation est effiloché ou la prise ou le cordon est endommagé.
  * Un liquide a pénétré dans le boîtier de l’appareil.
  * L’appareil est exposé à la pluie ou à une humidité excessive.
  * L’appareil a chuté et son boîtier est endommagé.
  * Vous pensez que l’appareil doit faire l’objet d’une maintenance ou d’une réparation.
* Débranchez l’unité de façon permanente avant de la déplacer ou si vous pensez qu’elle a été endommagée d’une quelconque façon.
* Prévoyez une source d’alimentation appropriée offrant une protection contre les surcharges, conforme aux spécifications d’alimentation suivantes :

  * Tension : 100 à 240 volts CA
  * Intensité : 20 A, nombre maximal par cordon d’alimentation. Les câbles d’alimentation sont fournis.
  * Fréquence : 50 à 60 Hz

![Icône Avertissement](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)
![Icône Risque d’électrocution](./media/azure-stack-edge-pro-r-safety/icon-safety-electric-shock.png)
![Plusieurs sources d’alimentation](./media/azure-stack-edge-pro-r-safety/icon-safety-disconnect-all-power.png) **AVERTISSEMENT :**

* Pour les systèmes dépourvus d’onduleur, débranchez tous les cordons d’alimentation CA afin de supprimer toute alimentation CA de l’équipement.
* Pour les systèmes avec onduleur, débranchez tous les cordons d’alimentation CA et utilisez le commutateur d’alimentation de l’onduleur pour purger le système de toute énergie. L’onduleur contient des tensions CA et CC dangereuses.

![Icône Avertissement](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)
![Icône Risque d’électrocution](./media/azure-stack-edge-pro-r-safety/icon-safety-electric-shock.png)**AVERTISSEMENT :**

* Pour les systèmes équipés d’onduleur, les tensions CA et/ou CC impliquent toujours un risque potentiel de tension CA à la sortie de l’onduleur générée par des batteries ou autre dispositif. Pour éviter des dégâts sur l’équipement ou des blessures corporelles, partez systématiquement du principe qu’il peut y avoir une tension à la sortie de l’onduleur, même s’il est déconnecté de la source d’alimentation principale.
* Pour les systèmes équipés d’onduleur, toutes les connexions externes alimentées par onduleur sont de type femelle. Ne retirez pas le boîtier ou n’insérez rien dans ces connecteurs ou d’autres connecteurs sur l’onduleur.

![Icône Avertissement](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)
![Icône Risque d’électrocution](./media/azure-stack-edge-pro-r-safety/icon-safety-electric-shock.png)**AVERTISSEMENT :**

* N’essayez pas de modifier ou d’utiliser des cordons d’alimentation AC autres que ceux fournis avec l’équipement.

![Icône Avertissement](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)
![Icône Aucune pièce réparable par l’utilisateur](./media/azure-stack-edge-pro-r-safety/icon-safety-do-not-access.png) **ATTENTION :**

* Cet équipement contient des piles boutons au lithium et/ou des batteries lithium fer phosphate. N’essayez pas de réparer l’équipement. Les batteries de cet équipement ne sont pas remplaçables par l’utilisateur. risque d’explosion si la batterie est remplacée par une batterie de type incorrect.
* Le retrait du module de batteries de l’onduleur expose des composants de ce dernier qui sont sous tension. N’insérez pas d’objets étrangers dans le compartiment du module de batteries.

## <a name="regulatory-information"></a>Informations réglementaires

Cette section contient des informations réglementaires sur l’appareil Azure Stack Edge Pro R, dont le numéro de modèle réglementaire est Azure Stack Edge Pro R.

L’appareil Edge Azure Stack Edge Pro R est conçu pour être utilisé avec un équipement de technologies de l’information répertorié par un laboratoire d’essai reconnu nationalement (UL, CSA, ETL, etc.) et conforme à la norme CEI/EN 60950-1 ou CEI/EN 62368-1 (marquage CE).

L’appareil est conçu pour fonctionner dans les environnements suivants :

| Environnement | Spécifications |
|:--- |:--- |
|Spécifications relatives à la température | <ul><li>Température de stockage : -33&deg;C–63&deg;C (-28&deg;F–145&deg;F) </li><li>En régime continu : 5&deg;C–43&deg;C (41&deg;F–110&deg;F)</li><li>Gradient de température maximal (fonctionnement et stockage) : 20&deg;C/h (68&deg;F/h)</li></ul> |
|Spécifications relatives à l’humidité relative | <ul><li>Stockage : De 5 à 95 % d’humidité relative avec un point de rosée maximal de 33&deg;C (91&deg;F). L’atmosphère doit être en permanence sans condensation.</li><li>En fonctionnement : De 5 à 85 % d’humidité relative avec un point de rosée maximal de 29&deg;C (84,2&deg;F)</li></ul> |
| Spécifications relatives à l’altitude maximale | <ul><li>En fonctionnement (sans onduleur) : 15 000 pieds (4 572 mètres)</li><li>En fonctionnement (avec onduleur) : 6 561 pieds (2 000 mètres)</li><li>Stockage : 40 000 pieds (12 192 mètres)</li></ul> |

<!--|Standard operating temperature specifications | <ul>Continuous operation (for altitude less than 950 m or 3117 ft): +5&deg;C–45&deg;C (41&deg;F–113&deg;F) with no direct sunlight on the equipment</ui>
|Expanded operating temperature specifications | <ul><li>Continuous operation: 5&deg;C to 45&deg;C at 5% to 85% RH with 29&deg;C dew point.</li><li></= 1% of annual operating hours: –5&deg;C to 55&deg;C at 5% to 90% RH with 29&deg;C dew point. |
| Expanded operating temperature restrictions | <ul><li>Do not perform cold start below -15&deg;C according to IEC 60945.</li><li>The operating temperature specified is for a maximum altitude of 950 meters.</li></ul> |
| Gaseous contamination specifications | <ul><li>Copper coupon corrosion rate: < 300 &Aring;/month per Class G1 as defined by ANSI/ISA71.04-1985.</li><li>Silver coupon corrosion rate: < 200 &Aring;/month as defined by AHSRAE TC9.9.</li></ul>|

> [!NOTE]
> Outside the standard operating temperature (10&deg;C to 35&deg;C):
> - The system can operate continuously in temperatures as low as 5&deg;C and as high as 45&deg;C. For temperatures between 35&deg;C and 45&deg;C, de-rate maximum allowable temperature by 1&deg;C per 175 m above 950 m (1&deg;F per 319 ft).
> - The system can operate down to –5&deg;C or up to 55&deg;C for a maximum of 1% of its annual operating hours. For temperatures between 45&deg;C and 55&deg;C, de-rate maximum allowable temperature by 1&deg;C per 125 m above 950 m (1&deg;F per 228 ft).
>
> When operating in the expanded temperature range:
> - System performance may be impacted.
> - Ambient temperature warnings may be reported in the System Event Log.     
>
> Maximum corrosive contaminant levels measured at &lt;/= 50% relative humidity. 


A device that has a UPS installed is designed to operate in the following environments:

| Environment | Specifications |
|:---  |:--- |
|Temperature specifications | <ul><li>Storage temperature: –40&deg;C–70&deg;C (–40&deg;F-158&deg;F) per Mil-Std 810G Method 501.5, Proc 1</li><li>Continuous operation (for altitude less than 950 m or 3117 ft): 5&deg;C–45&deg;C (41&deg;F–113&deg;F), with no direct sunlight on the equipment</li><li>Excursion temperature: 50&deg;C per Mil-Std 810G</li><li>Maximum temperature gradient (operating and storage): 20&deg;C/h (68&deg;F/h)</li></ul> |
|Relative humidity specifications | <ul><li>Storage: 5% to 95% RH with 33&deg;C (91&deg;F) maximum dew point. Atmosphere must be non-condensing at all times.</li><li>Operating: 5% to 85% relative humidity with 29&deg;C (84.2&deg;F) maximum dew point</li></ul>|
|Maximum altitude specifications | <ul><li>Storage: Mil-Std 810G method 500.5, Proc. I, 40,000 ft for 1 hour after stabilization </li><li>Operating: Mil-Std 810G method 500.5, Proc. II, air carriage, 15,000 ft for 1 hour after stabilization</li></ul>|
|Standard operating temperature specifications |<ul><li>Continuous operation (for altitude less than 950 m or 3117 ft)</li><li>+5&deg;C–45&deg;C (41&deg;F–113&deg;F) with no direct sunlight on the equipment</li></ul>|
|Expanded operating temperature specifications | <ul><li>Continuous operation: 5&deg;C to 45&deg;C at 5% to 85% RH with 29&deg;C dew point.</li><li>&lt;/= 1% of annual operating hours: 0&deg;C to 50&deg;C at 5% to 90% RH with 29&deg;C dew point.</li></ul>|
|Expanded operating temperature restrictions | <ul><li>Do not perform cold start below -15C Per IEC 60945</li><li>The operating temperature specified is for a maximum altitude of 950 meters</li></ul> |
|Gaseous contamination specifications | <ul><li>Copper coupon corrosion rate: < 300 &Aring;/month per Class G1 as defined by ANSI/ISA71.04-1985.</li><li>Silver coupon corrosion rate: < 200 &Aring;/month as defined by AHSRAE TC9.9.|-->

<!--
> [!NOTE]
> Outside the standard operating temperature (10&deg;C to 35&deg;C):
> - The system can operate continuously in temperatures as low as 5&deg;C and as high as 45&deg;C. For temperatures between 35&deg;C and 45&deg;C, de-rate maximum allowable temperature by 1&deg;C per 175 m above 950 m (1&deg;F per 319 ft).
> - The system can operate down to 0&deg;C or up to 50&deg;C for a maximum of 1% of its annual operating hours. For temperatures between 45&deg;C and 55&deg;C, de-rate maximum allowable temperature by 1&deg;C per 125 m above 950 m (1&deg;F per 228 ft).
> 
> When operating in the expanded temperature range:
> - System performance may be impacted.
> - Ambient temperature warnings may be reported in the System Event Log.
>
> Maximum corrosive contaminant levels measured at &lt;/= 50% relative humidity. --> 


> ![Icône Avis](./media/azure-stack-edge-pro-r-safety/icon-safety-notice.png) **AVIS :** &nbsp;Les changements et modifications apportés à l’équipement non expressément approuvés par Microsoft peuvent annuler l’autorisation de l’utilisateur à se servir de l’équipement.

CANADA et ÉTATS-UNIS :

AVIS : Cet équipement a été testé et déclaré conforme aux limites applicables aux appareils numériques de classe A, selon la section 15 des règlements de la FCC. Ces limites sont conçues pour offrir une protection raisonnable contre les interférences nuisibles lorsque l’équipement est utilisé dans un environnement commercial. Cet appareil génère, utilise et peut émettre une énergie de fréquence radio. S’il n’est pas installé et utilisé conformément au manuel d’instructions, il peut être source d’interférences nuisibles pour les communications radio. L’utilisation de cet appareil dans une zone résidentielle est susceptible de causer des interférences nuisibles, auquel cas l’utilisateur sera tenu d’y remédier à ses propres frais.

Cet appareil est conforme à la section 15 des règlements de la FCC et aux normes RSS d’Industrie Canada relatives à une utilisation sans licence. L’utilisation de l’appareil est soumise aux deux conditions suivantes : (1) cet appareil ne peut pas provoquer d’interférences nuisibles et (2) cet appareil doit accepter toute interférence reçue, y compris les interférences pouvant engendrer un fonctionnement indésirable de l’appareil.

![Avertissement relatif aux informations réglementaires 1](./media/azure-stack-edge-mini-r-safety/regulatory-information-1.png)


CAN ICES-3(A)/NMB-3(A) Microsoft Corporation, One Microsoft Way, Redmond, WA 98052, États-Unis.
États-Unis : (800) 426 9400 Canada : (800) 933 4750

UNION EUROPÉENNE : Demander une copie de la déclaration de conformité de l’Union européenne. 

![Icône d'avertissement](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)

Cet appareil est un produit de classe A. Dans un environnement domestique, ce produit peut provoquer des interférences radio, auquel cas l’utilisateur peut être tenu de prendre les mesures adéquates.

Élimination de batteries usagées et d’équipements électriques et électroniques :

![Icône Avertissement 14](./media/azure-stack-edge-mini-r-safety/icon-ewaste-disposal.png)

Ce symbole sur le produit, ses batteries ou son emballage signifie que ce produit et toute batterie qu’il peut contenir ne doivent pas être éliminés avec vos déchets ménagers. Il vous incombe de confier le recyclage des batteries et équipements électriques et électroniques à un point de collecte approprié. Ce processus de collecte et de recyclage spécifique contribue à préserver les ressources naturelles et à éviter les conséquences négatives potentielles sur la santé humaine et l’environnement que pourrait entraîner l’élimination inappropriée de batteries et d’équipements électriques et électroniques susceptibles de contenir des substances dangereuses. Pour plus d’informations sur les lieux où vous pouvez déposer vos batteries et déchets électriques et électroniques, contactez votre service municipal, votre service d’élimination des déchets ménagers ou le magasin dans lequel vous avez acheté ce produit. Contactez erecycle@microsoft.com pour obtenir des informations supplémentaires sur les déchets d’équipements électriques et électroniques.

Ce produit contient une ou plusieurs batteries boutons.

Microsoft Ireland Sandyford Ind Est Dublin D18 KX32 IRL Numéro de téléphone : + 353 1 295 3826 Numéro de fax : + 353 1 706 4110



## <a name="next-steps"></a>Étapes suivantes

- [Préparer le déploiement d’Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-prep.md)
