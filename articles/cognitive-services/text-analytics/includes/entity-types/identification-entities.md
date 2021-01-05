---
title: Entités d’identification
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 04/29/2020
ms.author: aahi
ms.openlocfilehash: 49a2b285d9494dfc5ca0863c021733b4bc874efa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91779377"
---
Cette catégorie d’entité comprend les informations financières et les formes officielles d’identification. Disponible à partir de la version `2019-10-01` du modèle. Les sous-types sont listés ci-dessous. 

### <a name="financial-account-identification"></a>Identification de compte financier

| Nom de sous-type               | Description                                                                |
|----------------------------|----------------------------------------------------------------------------|
| Numéro de routage ABA        | Numéros de routage de transit ABA (American Banker Association).                  |
| Code SWIFT                 | Codes SWIFT pour les informations sur les instructions de paiement.                           |
| Carte de crédit                | Numéros de carte de crédit.                                                       |
| Numéro de compte bancaire international (IBAN)                  | Codes IBAN pour les informations sur les instructions de paiement.                            |


### <a name="government-and-countryregion-specific-identification"></a>Identification propre aux gouvernements et pays/régions

> [!NOTE]
> Les entités financières et propres au pays suivantes ne sont pas retournées avec le paramètre `domain=phi` :
> * Numéros de passeport
> * Numéros d’identification fiscale

Les entités suivantes sont regroupées et listées par pays :

Argentine
* Numéro d’identité nationale (DNI) en Argentine

Autriche
* Carte d’identité en Autriche
* Numéros d’identification fiscale autrichien
* Numéro de TVA en Autriche

Australie
* Numéro de compte bancaire en Australie
* Numéro professionnel australien
* Numéro de société australien
* Numéro de permis de conduire en Australie
* Numéro de compte médical en Australie
* Numéro de passeport en Australie
* Numéro d’identification fiscale en Australie

Belgique
* Numéro national en Belgique
* Numéro de TVA en Belgique

Brésil 
* Numéro d’entité légale (CNPJ) au Brésil
* Numéro CPF au Brésil
* Carte d’identité nationale (RG) au Brésil

Bulgarie
* Numéro civil unifié en Bulgarie

Canada
* Numéro de compte bancaire au Canada
* Numéro de permis de conduire au Canada
* Numéro de service médical au Canada
* Numéro de passeport au Canada
* Numéro d’identification médicale personnelle (PHIN) au Canada
* Numéro d’assurance sociale au Canada

Chili
* Numéro de carte d’identité 

Chine
* Numéro de carte d’identité de résident (RPC) en Chine

Croatie
* Numéro de carte d’identité en Croatie
* Numéro de carte d’identité nationale en Croatie
* Numéro d’identification personnelle (OIB) en Croatie

Chypre
* Numéro de carte d’identité à Chypre
* Numéros d’identification fiscale à Chypre

République tchèque
* Numéro d’identité personnelle en Tchéquie

Danemark
* Numéro d’identification personnelle au Danemark

Estonie
* Code d’identification personnelle en Estonie

Union européenne (UE)
* Numéro de carte de débit UE
* Numéro de permis de conduire dans l’Union européenne
* Numéro d’identification nationale dans l’Union européenne
* Numéro de passeport dans l’Union européenne
* Numéro de sécurité sociale (SSN) dans l’Union européenne (ou équivalent)
* Numéros d’identification fiscale UE (TIN)

Finlande
* Numéro d’assurance maladie européen en Finlande
* Numéro d’identité nationale en Finlande
* Numéro de passeport en Finlande

France
* Numéro de permis de conduire en France
* Numéro d’assurance maladie en France
* Carte nationale d’identité (CNI) en France
* Numéro de passeport en France
* Numéro de sécurité sociale (INSEE) en France
* Numéro d’identification fiscale en France (numéro SPI)
* Numéro de TVA en France

Allemagne
* Numéro de permis de conduire en Allemagne
* Numéro de carte d’identité en Allemagne
* Numéro de passeport en Allemagne
* Numéros d’identification fiscale en Allemagne
* Numéro de TVA en Allemagne

Grèce 
* Numéro de carte d’identité nationale en Grèce
* Numéros d’identification fiscale en Grèce

Hong Kong (R.A.S.)
* Numéro de carte d’identité à Hong Kong (R.A.S.)

Hongrie
* Numéro d’identification nationale en Hongrie
* Numéro d’identification fiscale en Hongrie
* Numéro de TVA en Hongrie

Inde
* Numéro de compte permanent (PAN) en Inde
* Numéro d’identification unique (Aadhaar) en Inde

Indonésie
* Numéro de carte d’identité (KTP) en Indonésie

Irlande
* Numéro de service public personnel (PPS) en Irlande

Israël
* Numéro d’identité nationale en Israël
* Numéro de compte bancaire en Israël

Italie
* Numéro de permis de conduire en Italie
* Code fiscal en Italie
* Numéro de TVA en Italie

Japon
* Numéro de compte bancaire au Japon
* Numéro de permis de conduire au Japon
* My Number japonais - Personnel
* My Number japonais - Entreprise
* Numéro d’inscription de résident au Japon
* Numéro de carte de résidence japonaise
* Numéro d’assurance sociale (SIN) au Japon
* Numéro de passeport au Japon

Lettonie
* Code personnel en Lettonie

Lituanie
* Code personnel en Lituanie

Luxembourg
* Numéro d’identification nationale au Luxembourg (personnes physiques)
* Numéro d’identification nationale au Luxembourg (personnes morales)

Malaisie
* Numéro de carte d’identité en Malaisie

Malte
* Numéro de carte d’identité à Malte
* Numéros d’identification fiscale à Malte

Pays-Bas
* Numéro de service citoyen (BSN) au Pays-Bas
* Numéros d’identification fiscale aux Pays-Bas
* Numéro de TVA aux Pays-Bas

Nouvelle-Zélande
* Numéro de compte bancaire en Nouvelle-Zélande
* Numéro de permis de conduire en Nouvelle-Zélande
* Numéro Inland Revenue Department (IRD) en Nouvelle-Zélande
* Numéro du ministère de la santé en Nouvelle-Zélande
* Numéro de protection sociale (Social Welfare) en Nouvelle-Zélande

Norvège
* Numéro d’identité en Norvège

Philippines
* Numéro d’identité polyvalent unifié aux Philippines

Pologne
* Carte d’identité en Pologne
* Numéro d’identité nationale (PESEL) en Pologne
* Numéro de passeport en Pologne
* Numéro REGON en Pologne
* Numéros d’identification fiscale en Pologne

Portugal 
* Numéro de carte de citoyen au Portugal
* Numéros d’identification fiscale au Portugal

Roumanie
* Code numérique personnel (CNP) en Roumanie

Russie
* Numéro de passeport russe (national)
* Numéro de passeport russe (international)

Arabie Saoudite
* Numéro national en Arabie saoudite

Singapour
* Numéro de carte d’identité d’inscription nationale (NRIC) à Singapour

Slovaquie 
* Numéro personnel en Slovaquie

Slovénie
* Numéros d’identification fiscale en Slovénie
* Numéro d’identification nationale de Slovénie de Slovénie

Afrique du Sud
* Numéro d’identification en Afrique du Sud

Corée du Sud
* Numéro d’inscription de résident en Corée du Sud

Espagne 
* Numéro d’identification nationale en Espagne
* Numéro de sécurité sociale (SSN) en Espagne
* Numéros d’identification fiscale en Espagne

Suède
* Numéro d’identité nationale en Suède
* Numéro de passeport en Suède
* Numéros d’identification fiscale en Suède

Suisse
* Numéro d'assurance sociale (AVS) en Suisse

Taïwan 
* Numéro d’identité nationale à Taïwan
* Certificat de résident (ARC/TARC) à Taïwan
* Numéro de passeport à Taïwan

Thaïlande
* Code d’identification de population thaï

Turquie
* Numéro d’identification nationale en Turquie

Ukraine
* Numéro de passeport en Ukraine (national)
* Numéro de passeport en Ukraine (international)

Royaume-Uni
* au Royaume-Uni Numéro de permis de conduire
* au Royaume-Uni Numéro d’électeur
* au Royaume-Uni Numéro du service de santé national (NHS)
* au Royaume-Uni Numéro d’assurance nationale (NINO)
* au Royaume-Uni Numéro de passeport
* au Royaume-Uni Numéro de référence fiscal unique (UTR)

États-Unis
* États-Unis Numéro de sécurité sociale (SSN)
* États-Unis Numéro de permis de conduire
* États-Unis Numéro de passeport
* États-Unis Numéro d’identification de contribuable individuel (ITIN)
* États-Unis Numéro DEA (Drug Enforcement Agency)
* États-Unis Numéro de compte bancaire
