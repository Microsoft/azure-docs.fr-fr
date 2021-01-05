---
title: Liste des classifications prises en charge
description: Cette page répertorie les classifications système prises en charge dans Azure Purview.
author: animukherjee
ms.author: anmuk
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: reference
ms.date: 11/20/2020
ms.openlocfilehash: 7458b027add8be86d9491c674c2f1a0bc9fbc68c
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550201"
---
# <a name="supported-classifications-in-azure-purview"></a>Classifications prises en charge dans Azure Purview

Cet article répertorie les classifications système prises en charge et définies dans Azure Purview (préversion).

## <a name="defined-system-classifications"></a>Classifications système définies

Azure Purview classifie les données en se basant sur des [expressions régulières](https://wikipedia.org/wiki/Regular_expression) et le [filtre de Bloom](https://wikipedia.org/wiki/Bloom_filter). Les listes suivantes décrivent le format, le modèle et les mots clés pour les classifications système définies par Azure Purview.

Chaque nom de classification est précédé du préfixe MICROSOFT.

## <a name="bloom-filter-classifications"></a>Classifications Filtre de Bloom

## <a name="city-country-and-place"></a>Ville, pays et lieu

Les filtres de ville, de pays et de lieu ont été préparés à l’aide des meilleurs jeux de données disponibles pour la préparation des données.

## <a name="person"></a>Personne

Le filtre de Bloom Person a été préparé à l’aide des deux jeux de données ci-dessous.

- [Données de recensement des noms de famille aux États-Unis en 2010 (162 000 entrées)](https://www.census.gov/topics/population/genealogy/data/2010_surnames.html)
- [Noms de bébés populaires (à partir des numéros de sécurité sociale aux USA) entre 1880 et 2019 (98 000 entrées)](https://www.ssa.gov/oact/babynames/limits.html)

## <a name="regex-classifications"></a>Classifications RegEx

## <a name="aba-routing"></a>Routage ABA

### <a name="format"></a>Format

Neuf chiffres, qui peuvent être dans un modèle mis en forme ou non mis en forme

### <a name="pattern"></a>Modèle

Mis en forme :

- quatre chiffres commençant par 0, 1, 2, 3, 6, 7 ou 8
- trait d’union
- quatre chiffres
- trait d’union
- chiffre non mis en forme : neuf chiffres consécutifs commençant par 0, 1, 2, 3, 6, 7 ou 8

### <a name="keywords"></a>Mots clés

#### <a name="keyword_aba_routing"></a>Keyword_aba_routing

```
amba number
aba #
aba
abarouting#
abaroutingnumber
americanbankassociationrouting#
americanbankassociationroutingnumber
bank routing#
bankroutingnumber
routing #
routing no
routing number
routing transit number
routing#
RTN
```

## <a name="argentina-national-identity-dni-number"></a>Numéro d’identité nationale (DNI) en Argentine

### <a name="format"></a>Format

Huit chiffres avec ou sans points

### <a name="pattern"></a>Modèle

Huit chiffres :

- deux chiffres
- point facultatif
- trois chiffres
- point facultatif
- trois chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_argentina_national_id"></a>Keyword_argentina_national_id

```
Argentina National Identity number
cedula
cédula
dni
documento nacional de identidad
documento número
documento numero
registro nacional de las personas
rnp
```

## <a name="australia-bank-account-number"></a>Numéro de compte bancaire en Australie

### <a name="format"></a>Format

6 à 10 chiffres avec ou sans numéro de succursale d’établissement bancaire

### <a name="pattern"></a>Modèle

Le numéro de compte inclut entre 6 et 10 chiffres.

Numéro de succursale d’établissement bancaire en Australie :

- trois chiffres
- trait d’union
- trois chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_australia_bank_account_number"></a>Keyword_australia_bank_account_number

```
swift bank code
correspondent bank
base currency
usa account
holder address
bank address
information account
fund transfers
bank charges
bank details
banking information
full names
iaea
```

## <a name="australia-drivers-license-number"></a>Numéro de permis de conduire en Australie

### <a name="format"></a>Format
Neufs lettres et chiffres

### <a name="pattern"></a>Modèle
neufs lettres et chiffres :

- deux chiffres ou lettres (ne respectant pas la casse)
- deux chiffres
- cinq chiffres ou lettres (ne respectant pas la casse)

OU

- une à deux lettres facultatives (ne respectant pas la casse)
- quatre à neuf chiffres

OU

- neuf chiffres ou lettres (ne respectant pas la casse)

### <a name="keywords"></a>Mots clés

#### <a name="keyword_australia_drivers_license_number"></a>Keyword_australia_drivers_license_number

```
international driving permits
australian automobile association
international driving permit
DriverLicence
DriverLicences
Driver Lic
Driver License
Driver Licenses
DriversLic
DriversLicence
DriversLicences
Drivers Lic
Drivers Lics
Drivers License
Drivers Licenses
Driver'Lic
Driver'Lics
Driver'License
Driver'Licenses
Driver' Lic
Driver' Lics
Driver' License
Driver' Licenses
Driver'sLic
Driver'sLics
Driver'sLicence
Driver'sLicences
Driver's Lic
Driver's Lics
Driver's License
Driver's Licenses
DriverLic#
DriverLics#
DriverLicence#
DriverLicences#
Driver Lic#
Driver Lics#
Driver License#
Driver Licenses#
DriversLic#
DriversLics#
DriversLicence#
DriversLicences#
Drivers Lic#
Drivers Lics#
Drivers License#
Drivers Licenses#
Driver'Lic#
Driver'Lics#
Driver'License#
Driver'Licenses#
Driver' Lic#
Driver' Lics#
Driver' License#
Driver' Licenses#
Driver'sLic#
Driver'sLics#
Driver'sLicence#
Driver'sLicences#
Driver's Lic#
Driver's Lics#
Driver's License#
Driver's Licenses#
```

#### <a name="keyword_australia_drivers_license_number_exclusions"></a>Keyword_australia_drivers_license_number_exclusions

```
aaa
DriverLicense
DriverLicenses
Driver License
Driver Licenses
DriversLicense
DriversLicenses
Drivers License
Drivers Licenses
Driver'License
Driver'Licenses
Driver' License
Driver' Licenses
Driver'sLicense
Driver'sLicenses
Driver's License
Driver's Licenses
DriverLicense#
DriverLicenses#
Driver License#
Driver Licenses#
DriversLicense#
DriversLicenses#
Drivers License#
Drivers Licenses#
Driver'License#
Driver'Licenses#
Driver' License#
Driver' Licenses#
Driver'sLicense#
Driver'sLicenses#
Driver's License#
Driver's Licenses#
```

## <a name="australia-medical-account-number"></a>Numéro de compte médical en Australie

### <a name="format"></a>Format

10-11 chiffres

### <a name="pattern"></a>Modèle

10-11 chiffres :

- le premier chiffre est compris entre 2 et 6
- le neuvième chiffre est un chiffre de vérification
- le dixième chiffre est le chiffre de l’émission
- le onzième chiffre (facultatif) est le numéro individuel

### <a name="keywords"></a>Mots clés

#### <a name="keyword_australia_medical_account_number"></a>Keyword_Australia_Medical_Account_Number

```
bank account details
medicare payments
mortgage account
bank payments
information branch
credit card loan
department of human services
local service
medicare
```

## <a name="australia-passport-number"></a>Numéro de passeport en Australie

### <a name="format"></a>Format

Lettre suivie de sept chiffres

### <a name="pattern"></a>Modèle

Une lettre (sans respect de la casse) suivie de sept chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_passport"></a>Keyword\_passport

```
Passport Number
Passport No
Passport #
Passport#
PassportID
Passport no
passport number
パスポート
パスポート番号
パスポートのNum
パスポート＃
Numéro de passeport
Passeport n °
Passeport Non
Passeport #
Passeport#
PasseportNon
Passeportn °
```

#### <a name="keyword_australia_passport_number"></a>Keyword\_australia\_passport\_number

```
passport
passport details
immigration and citizenship
commonwealth of australia
department of immigration
residential address
department of immigration and citizenship
visa
national identity card
passport number
travel document
issuing authority
```

## <a name="australia-tax-file-number"></a>Numéro d’identification fiscale en Australie

### <a name="format"></a>Format

huit à neuf chiffres

### <a name="pattern"></a>Modèle

huit à neuf chiffres généralement présentés avec des espaces, comme suit :

- trois chiffres
- espace facultatif
- trois chiffres
- espace facultatif
- deux à trois chiffres où le dernier chiffre est un chiffre de vérification

### <a name="keywords"></a>Mots clés

#### <a name="keyword_australia_tax_file_number"></a>Keyword\_australia\_tax\_file\_number

```
australian business number
marginal tax rate
medicare levy
portfolio number
service veterans
withholding tax
individual tax return
tax file number
tfn
```

## <a name="belgium-national-number"></a>Numéro national en Belgique

### <a name="format"></a>Format

11 chiffres plus délimiteurs facultatifs

### <a name="pattern"></a>Modèle

11 chiffres plus délimiteurs :

- six chiffres et deux points facultatifs au format YY.MM.DD pour la date de naissance
- Délimiteur facultatif (point, tiret, espace)
- trois chiffres séquentiels (impair pour les hommes, pair pour les femmes)
- Délimiteur facultatif (point, tiret, espace)
- deux chiffres de vérification

### <a name="keywords"></a>Mots clés

#### <a name="keyword_belgium_national_number"></a>Keyword\_belgium\_national\_number

```
be lasting aantal
bnn#
bnn
carte d'identité
identifiant national
identifiantnational#
identificatie
identification
identifikation
identifikationsnummer
identifizierung
identité
identiteit
identiteitskaart
identity
inscription
national number
national register
national number#
national number
nif#
nif
numéro d'assuré
numéro de registre national
numéro de sécurité
numéro d'identification
numéro d'immatriculation
numéro national
numéronational#
personal ID number
personalausweis
personalidnumber#
registratie
registration
registrationsnumme
registrierung
social security number
ssn#
ssn
steuernummer
tax ID
tax identification no
tax identification number
tax no#
tax no
tax number
tax registration number
tax ID#
tax ID no#
tax ID number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
```

## <a name="brazil-cpf-number"></a>Numéro CPF au Brésil

### <a name="format"></a>Format

11 chiffres incluant un chiffre de vérification et pouvant être mis en forme ou non

### <a name="pattern"></a>Modèle

Mis en forme :

- trois chiffres
- point
- trois chiffres
- point
- trois chiffres
- trait d’union
- deux chiffres, qui sont des chiffres de vérification

Sans mise en forme :

- 11 chiffres où les deux derniers chiffres sont des chiffres de vérification

### <a name="keywords"></a>Mots clés

#### <a name="keyword_brazil_cpf"></a>Keyword\_brazil\_cpf

```
CPF
Identification
Registration
Revenue
Cadastro de Pessoas Físicas
Imposto
Identificação
Inscrição
Receita
```

## <a name="brazil-legal-entity-number-cnpj"></a>Numéro d’entité légale (CNPJ) au Brésil

### <a name="format"></a>Format

14 chiffres incluant un numéro d’enregistrement, un numéro de succursale et des chiffres de vérification, plus des délimiteurs

### <a name="pattern"></a>Modèle

14 chiffres plus délimiteurs :

- deux chiffres
- point
- trois chiffres
- point
- trois chiffres (les huit premiers chiffres correspondent au numéro d’enregistrement)
- barre oblique
- numéro de succursale à quatre chiffres
- trait d’union
- deux chiffres, qui sont des chiffres de vérification

### <a name="keywords"></a>Mots clés

#### <a name="keyword_brazil_cnpj"></a>Keyword\_brazil\_cnpj

```
CNPJ
CNPJ/MF
CNPJ-MF
National Registry of Legal Entities
Taxpayers Registry
Legal entity
Legal entities
Registration Status
Business
Company
CNPJ
Cadastro Nacional da Pessoa Jurídica
Cadastro Geral de Contribuintes
CGC
Pessoa jurídica
Pessoas jurídicas
Situação cadastral
Inscrição
Empresa
```

## <a name="brazil-national-identification-card-rg"></a>Numéro d’identification nationale au Brésil (RG)

### <a name="format"></a>Format

:::no-loc text="Registro Geral (old format): Nine digits":::

:::no-loc text="Registro de Identidade (RIC) (new format): 11 digits":::

### <a name="pattern"></a>Modèle

:::no-loc text="Registro Geral (old format):":::

- deux chiffres
- point
- trois chiffres
- point
- trois chiffres
- trait d’union
- un chiffre, qui est un chiffre de vérification

:::no-loc text="Registro de Identidade (RIC) (new format):":::

- 10 chiffres
- trait d’union
- un chiffre, qui est un chiffre de vérification

### <a name="keywords"></a>Mots clés

#### <a name="keyword_brazil_rg"></a>Keyword\_brazil\_rg

```
Cédula de identidade
identity card
national ID
número de rregistro
registro de Iidentidade
registro geral
RG (this keyword is case-sensitive)
RIC (this keyword is case-sensitive)
```

## <a name="canada-bank-account-number"></a>Numéro de compte bancaire au Canada

### <a name="format"></a>Format

7 ou 12 chiffres

### <a name="pattern"></a>Modèle

Le numéro de compte bancaire du Canada inclut 7 ou 12 chiffres.

Le numéro de transit d’un compte bancaire au Canada est le suivant :

- cinq chiffres
- trait d’union
- trois chiffres OU
- un zéro &quot;0&quot;
- huit chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_canada_bank_account_number"></a>Keyword\_canada\_bank\_account\_number

```
canada savings bonds
canada revenue agency
canadian financial institution
direct deposit form
canadian citizen
legal representative
notary public
commissioner for oaths
child care benefit
universal child care
canada child tax benefit
income tax benefit
harmonized sales tax
social insurance number
income tax refund
child tax benefit
territorial payments
institution number
deposit request
banking information
direct deposit
```

## <a name="canada-drivers-license-number"></a>Numéro de permis de conduire au Canada

### <a name="format"></a>Format

Varie selon la province

### <a name="pattern"></a>Modèle

Différents modèles couvrant l’Alberta, la Colombie-Britannique, le Manitoba, le Nouveau-Brunswick, Terre-Neuve-et-Labrador, la Nouvelle-Écosse, l’Ontario, l’île du Prince Edward, le Québec et Saskatchewan

### <a name="keywords"></a>Mots clés

#### <a name="keyword_province_name_drivers_license_name"></a>Keyword\_[province\_name]\_drivers\_license\_name

- Abréviation de la province, par exemple AB
- Nom de la province, par exemple Alberta

#### <a name="keyword_canada_drivers_license"></a>Keyword\_canada\_drivers\_license

```
DL
DLS
CDL
CDLS
DriverLic
DriverLics
DriverLicense
DriverLicenses
DriverLicence
DriverLicences
Driver Lic
Driver Lics
Driver License
Driver Licenses
Driver License
Driver Licenses
DriversLic
DriversLics
DriversLicence
DriversLicences
DriversLicense
DriversLicenses
Drivers Lic
Drivers Lics
Drivers License
Drivers Licenses
Drivers License
Drivers Licenses
Driver'Lic
Driver'Lics
Driver'License
Driver'Licenses
Driver'License
Driver'Licenses
Driver' Lic
Driver' Lics
Driver' License
Driver' Licenses
Driver' License
Driver' Licenses
Driver'sLic
Driver'sLics
Driver'sLicense
Driver'sLicenses
Driver'sLicence
Driver'sLicences
Driver's Lic
Driver's Lics
Driver's License
Driver's Licenses
Driver's License
Driver's Licenses
Permis de Conduire
ID
IDs
ID card number
ID card numbers
ID card #
ID card #s
ID card
ID cards
ID card
identification number
identification numbers
identification #
identification #s
identification card
identification cards
identification
DL#
DLS#
CDL#
CDLS#
DriverLic#
DriverLics#
DriverLicense#
DriverLicenses#
DriverLicence#
DriverLicences#
Driver Lic#
Driver Lics#
Driver License#
Driver Licenses#
Driver License#
Driver Licenses#
DriversLic#
DriversLics#
DriversLicense#
DriversLicenses#
DriversLicence#
DriversLicences#
Drivers Lic#
Drivers Lics#
Drivers License#
Drivers Licenses#
Drivers License#
Drivers Licenses#
Driver'Lic#
Driver'Lics#
Driver'License#
Driver'Licenses#
Driver'License#
Driver'Licenses#
Driver' Lic#
Driver' Lics#
Driver' License#
Driver' Licenses#
Driver' License#
Driver' Licenses#
Driver'sLic#
Driver'sLics#
Driver'sLicense#
Driver'sLicenses#
Driver'sLicence#
Driver'sLicences#
Driver's Lic#
Driver's Lics#
Driver's License#
Driver's Licenses#
Driver's License#
Driver's Licenses#
Permis de Conduire#
ID#
IDs#
ID card#
ID cards#
ID card#
identification card#
identification cards#
identification#
```

## <a name="canada-health-service-number"></a>Numéro de service médical au Canada

### <a name="format"></a>Format

10 chiffres

### <a name="pattern"></a>Modèle

10 chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_canada_health_service_number"></a>Keyword\_canada\_health\_service\_number

```
personal health number
patient information
health services
specialty services
automobile accident
patient hospital
psychiatrist
workers compensation
disability
```

## <a name="canada-passport-number"></a>Numéro de passeport au Canada

### <a name="format"></a>Format

deux lettres majuscules suivies de six chiffres

### <a name="pattern"></a>Modèle

deux lettres majuscules suivies de six chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_canada_passport_number"></a>Keyword\_canada\_passport\_number

```
canadian citizenship
canadian passport
passport application
passport photos
certified translator
canadian citizens
processing times
renewal application
```

#### <a name="keyword_passport"></a>Keyword\_passport

```
Passport Number
Passport No
Passport #
Passport#
PassportID
Passport no
passport number
パスポート
パスポート番号
パスポートのNum
パスポート＃
Numéro de passeport
Passeport n °
Passeport Non
Passeport #
Passeport#
PasseportNon
Passeportn °
```

## <a name="canada-personal-health-identification-number-phin"></a>Numéro d’identification médicale personnelle (PHIN) au Canada

### <a name="format"></a>Format

neuf chiffres

### <a name="pattern"></a>Modèle

neuf chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_canada_phin"></a>Keyword\_canada\_phin

```
social insurance number
health information act
income tax information
manitoba health
health registration
prescription purchases
benefit eligibility
personal health
power of attorney
registration number
personal health number
practitioner referral
wellness professional
patient referral
health and wellness
```

#### <a name="keyword_canada_provinces"></a>Keyword\_canada\_provinces

```
Nunavut
Quebec
Northwest Territories
Ontario
British Columbia
Alberta
Saskatchewan
Manitoba
Yukon
Newfoundland and Labrador
New Brunswick
Nova Scotia
Prince Edward Island
Canada
```

## <a name="canada-social-insurance-number"></a>Numéro d’assurance sociale au Canada

### <a name="format"></a>Format

neuf chiffres avec traits d’union ou espaces facultatifs

### <a name="pattern"></a>Modèle

Mis en forme :

- trois chiffres
- trait d’union ou espace
- trois chiffres
- trait d’union ou espace
- trois chiffres

Non formaté : neuf chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_sin"></a>Keyword\_sin

```
sin
social insurance
numero d'assurance sociale
sins
ssn
ssns
social security
numero d'assurance social
national identification number
national ID
sin#
soc ins
social ins
```

#### <a name="keyword_sin_collaborative"></a>Keyword\_sin\_collaborative

```
driver's license
drivers license
driver's license
drivers license
DOB
Birthdate
Birthday
Date of Birth
```

## <a name="chile-identity-card-number"></a>Numéro de carte d’identité au Chili

### <a name="format"></a>Format

sept à huit chiffres plus délimiteurs, un chiffre ou une lettre de vérification

### <a name="pattern"></a>Modèle

sept à huit chiffres plus les délimiteurs :

- un à deux chiffres
- point facultatif
- trois chiffres
- point facultatif
- trois chiffres
- tiret
- un chiffre ou une lettre (ne respectant pas la casse), qui est un chiffre de vérification

### <a name="keywords"></a>Mots clés

#### <a name="keyword_chile_id_card"></a>Keyword\_chile\_id\_card

```
cédula de identidad
identificación
national identification
national identification number
national ID
número de identificación nacional
rol único nacional
rol único tributario
RUN
RUT
tarjeta de identificación
Rol Unico Nacional
Rol Unico Tributario
RUN#
RUT#
nationaluniqueroleID#
nacional identidad
número identificación
identidad número
numero identificacion
identidad numero
Chilean identity no.
Chilean identity number
Chilean identity #
Unique Tax Registry
Unique Tributary Role
Unique Tax Role
Unique Tributary Number
Unique National Number
Unique National Role
National unique role
Chile identity no.
Chile identity number
Chile identity #
```

## <a name="china-resident-identity-card-prc-number"></a>Numéro de carte d’identité de résident (RPC) en Chine

### <a name="format"></a>Format

18 chiffres

### <a name="pattern"></a>Modèle

18 chiffres :

- six chiffres, qui sont un code d’adresse
- huit chiffres au format YYYYMMDD (date de naissance)
- trois chiffres (code de commande)
- un chiffre, qui est un chiffre de vérification

### <a name="keywords"></a>Mots clés

#### <a name="keyword_china_resident_id"></a>Keyword\_china\_resident\_id

```
Resident Identity Card
PRC
National Identification Card
身份证
居民身份证
居民身份证
鉴定
身分證
居民身份證
鑑定
```

## <a name="credit-card-number"></a>Numéro de carte de crédit

### <a name="format"></a>Format

14 à 16 chiffres, qui peuvent être mis en forme ou non mis en forme (dddddddddddddddd) et qui doivent réussir le test Luhn.

### <a name="pattern"></a>Modèle

Modèle complexe et robuste qui détecte les cartes de toutes les principales marques dans le monde entier, notamment les cartes Visa, MasterCard, Discover Card, JCB, American Express, cartes cadeaux et Diner Cards.

### <a name="keywords"></a>Mots clés

#### <a name="keyword_cc_verification"></a>Keyword\_cc\_verification

```
card verification
card identification number
cvn
cid
cvc2
cvv2
pin block
security code
security number
security no
issue number
issue no
cryptogramme
numéro de sécurité
numero de securite
kreditkartenprüfnummer
kreditkartenprufnummer
prüfziffer
prufziffer
sicherheits Kode
sicherheitscode
sicherheitsnummer
verfalldatum
codice di verifica
cod. sicurezza
cod sicurezza
n autorizzazione
código
codigo
cod. seg
cod seg
código de segurança
codigo de seguranca
codigo de segurança
código de seguranca
cód. segurança
cod. seguranca
cod. segurança
cód. seguranca
cód segurança
cod seguranca
cod segurança
cód seguranca
número de verificação
numero de verificacao
ablauf
gültig bis
gültigkeitsdatum
gultig bis
gultigkeitsdatum
scadenza
data scad
fecha de expiracion
fecha de venc
vencimiento
válido hasta
valido hasta
vto
data de expiração
data de expiracao
data em que expira
validade
valor
vencimento
transaction
transaction number
reference number
セキュリティコード
セキュリティ コード
セキュリティナンバー
セキュリティ ナンバー
セキュリティ番号
```

#### <a name="keyword_cc_name"></a>Keyword\_cc\_name

```
amex
Americans express
Americans express
americano espresso
Visa
Mastercard
master card
mc
master cards
master cards
diner's Club
diners club
diners club
discover
discover card
discover card
discover cards
JCB
BrandSmart
japanese card bureau
carte blanche
carteblanche
credit card
cc#
cc#:
expiration date
exp date
expiry date
date d'expiration
date d'exp
date expiration
bank card
bank card
card number
card num
card number
card numbers
card numbers
credit card
credit cards
credit cards
ccn
card holder
cardholder
card holders
cardholders
check card
check card
check cards
check cards
debit card
debit card
debit cards
debit cards
atm card
atmcard
atm cards
atmcards
enroute
en route
card type
Cardmember Acct
cardmember account
Card no
Corporate Card
Corporate cards
Type of card
card account number
card member account
Cardmember Acct.
card no.
card no
card number
carte bancaire
carte de crédit
carte de credit
numéro de carte
numero de carte
nº de la carte
nº de carte
kreditkarte
karte
karteninhaber
karteninhabers
kreditkarteninhaber
kreditkarteninstitut
kreditkartentyp
eigentümername
kartennr
kartennummer
kreditkartennummer
kreditkarten-nummer
carta di credito
carta credito
n. carta
n carta
nr. carta
nr carta
numero carta
numero della carta
numero di carta
tarjeta credito
tarjeta de credito
tarjeta crédito
tarjeta de crédito
tarjeta de atm
tarjeta atm
tarjeta debito
tarjeta de debito
tarjeta débito
tarjeta de débito
nº de tarjeta
no. de tarjeta
no de tarjeta
numero de tarjeta
número de tarjeta
tarjeta no
tarjetahabiente
cartão de crédito
cartão de credito
cartao de crédito
cartao de credito
cartão de débito
cartao de débito
cartão de debito
cartao de debito
débito automático
debito automatico
número do cartão
numero do cartão
número do cartao
numero do cartao
número de cartão
numero de cartão
número de cartao
numero de cartao
nº do cartão
nº do cartao
nº. do cartão
no do cartão
no do cartao
no. do cartão
no. do cartao
クレジットカード番号
クレジットカードナンバー
クレジットカード＃
クレジットカード
クレジット
クレカ
カード番号
カードナンバー
カード＃
アメックス
アメリカンエクスプレス
アメリカン エクスプレス
Visaカード
Visa カード
マスターカード
マスター カード
マスター
ダイナースクラブ
ダイナース クラブ
ダイナース
有効期限
期限
キャッシュカード
キャッシュ カード
カード名義人
カードの名義人
カードの名義
デビット カード
デビットカード
```

## <a name="croatia-drivers-license-number"></a>Numéro de permis de conduire de Croatie

Cette entité de type d’informations sensibles est uniquement disponible dans le type d’informations sensibles Numéro de licence de conducteur européen.

### <a name="format"></a>Format

huit chiffres sans espaces ni délimiteurs

### <a name="pattern"></a>Modèle

huit chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keywords_eu_drivers_license_number"></a>Keywords\_eu\_driver's\_license\_number

```
driverlic
driverlics
driver license
driver licenses
driverlicence
driverlicences
driver lic
driver lics
driver license
driver licenses
driver license
driver licenses
driverslic
driverslics
driverslicence
driverslicences
drivers license
drivers licenses
drivers lic
drivers lics
drivers license
drivers licenses
drivers license
drivers licenses
driver'lic
driver'lics
driver'license
driver'licenses
driver'license
driver'licenses
driver' lic
driver' lics
driver' license
driver' licenses
driver' license
driver' licenses
driver'slic
driver'slics
driver'license
driver'licenses
driver'license
driver'licenses
driver's lic
driver's lics
driver's license
driver's licenses
driver's license
driver's licenses
dl#
dls#
driverlic#
driverlics#
driver license#
driver licenses#
driverlicence#
driverlicences#
driver lic#
driver lics#
driver license#
driver licenses#
driver licenses#
driverslic#
driverslics#
drivers license#
drivers licenses#
driverslicence#
driverslicences#
drivers lic#
drivers lics#
drivers license#
drivers licenses#
drivers license#
drivers licenses#
driver'lic#
driver'lics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver' lic#
driver' lics#
driver' license#
driver' licenses#
driver' license#
driver' licenses#
driver'slic#
driver'slics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver's lic#
driver's lics#
driver's license#
driver's licenses#
driver's license#
driver's licenses#
driving license
driving license
dlno#
driv lic
driv licen
driv license
driv licenses
driv license
driv licenses
driver licen
drivers licen
driver's licen
driving lic
driving licen
driving licenses
driving license
driving licenses
driving permit
dl no
dlno
dl number
```

#### <a name="keywords_croatia_eu_drivers_license_number"></a>Keywords\_croatia\_eu\_driver's\_license\_number

```
vozačka dozvola
vozačke dozvole
```

## <a name="croatia-identity-card-number"></a>Numéro de carte d’identité en Croatie

Cette entité de type d’informations sensibles est incluse dans le type d’information sensible Numéro d’identification national de l’UE et elle est disponible en tant qu’entité de type d’informations sensibles autonome.

### <a name="format"></a>Format

neuf chiffres

### <a name="pattern"></a>Modèle

neuf chiffres consécutifs

### <a name="keywords"></a>Mots clés

#### <a name="keyword_croatia_id_card"></a>Keyword\_croatia\_id\_card

```
majstorski broj građana
master citizen number
nacionalni identifikacijski broj
national identification number
oib#
oib
osobna iskaznica
osobni ID
osobni identifikacijski broj
personal identification number
porezni broj
porezni identifikacijski broj
tax ID
tax identification no
tax identification number
tax no#
tax no
tax number
tax registration number
tax ID#
tax ID no#
tax ID number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
```

## <a name="croatia-personal-identification-oib-number"></a>Numéro d’identification personnelle (OIB) en Croatie

### <a name="format"></a>Format

11 chiffres

### <a name="pattern"></a>Modèle

11 chiffres :

- 10 chiffres
- le dernier chiffre est un chiffre de vérification

### <a name="keywords"></a>Mots clés

#### <a name="keyword_croatia_oib_number"></a>Keyword\_croatia\_oib\_number

```
majstorski broj građana
master citizen number
nacionalni identifikacijski broj
national identification number
oib#
oib
osobna iskaznica
osobni ID
osobni identifikacijski broj
personal identification number
porezni broj
porezni identifikacijski broj
tax ID
tax identification no
tax identification number
tax no#
tax no
tax number
tax registration number
tax ID#
tax ID no#
tax Id number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
```

## <a name="denmark-personal-identification-number"></a>Numéro d’identification personnelle au Danemark

### <a name="format"></a>Format

10 chiffres contenant un trait d’union

### <a name="pattern"></a>Modèle

10 chiffres :

- six chiffres au format DDMMYY (date de naissance)
- trait d’union
- quatre chiffres où le dernier chiffre est un chiffre de vérification

### <a name="keywords"></a>Mots clés

#### <a name="keyword_denmark_id"></a>Keyword\_denmark\_id

```
centrale person register
civilt registreringssystem
cpr
cpr#
gesundheitskarte nummer
gesundheitsversicherungkarte nummer
health card
health insurance card number
health insurance number
identification number
identifikationsnummer
identifikationsnummer#
identity number
krankenkassennummer
national ID#
national number#
national number
personalidnumber#
personalidentityno#
personal ID number
personnummer
personnummer#
reisekrankenversicherungskartenummer
rejsesygesikringskort
ssn
ssn#
skat ID
skat kode
skat nummer
skattenummer
social security number
sundhedsforsikringskort
sundhedsforsikringsnummer
sundhedskort
sundhedskortnummer
sygesikring
sygesikringkortnummer
tax code
travel health insurance card
uniqueidentityno#
tax number
tax registration number
tax ID
tax identification number
tax ID#
tax number#
tax no
tax no#
tax number
tax identification no
tin#
tax ID no#
tax ID number#
tax no#
tin ID
tin no
cpr.nr
cprnr
cprnummer
personnr
person register
sygesikringsbevis
sygesikringsbevisnr
sygesikringsbevisnummer
sygesikringskort
sygesikringskortnr
sygesikringskortnummer
sygesikringsnr
sygesikringsnummer
```

## <a name="eu-debit-card-number"></a>Numéro de carte de débit UE

### <a name="format"></a>Format

16 chiffres

### <a name="pattern"></a>Modèle

Modèle complexe et robuste

### <a name="keywords"></a>Mots clés

#### <a name="keyword_eu_debit_card"></a>Keyword\_eu\_debit\_card

```
account number
card number
card no.
security number
cc#
```

#### <a name="keyword_card_terms_dict"></a>Keyword\_card\_terms\_dict

```
acct nbr
acct num
acct no
Americans express
Americans express
americano espresso
amex
atm card
atm cards
atm kaart
atmcard
atmcards
atmkaart
atmkaarten
ban contact
bank card
bankkaart
card holder
card holders
card num
card number
card numbers
card type
cardano numerico
cardholder
cardholders
card number
card numbers
carta bianca
carta credito
carta di credito
cartao de credito
cartao de crédito
cartao de debito
cartao de débito
carte bancaire
carte blanche
carte bleue
carte de credit
carte de crédit
carte di credito
carteblanche
cartão de credito
cartão de crédito
cartão de debito
cartão de débito
cb
ccn
check card
check cards
check card
check cards
chequekaart
cirrus
cirrus-edc-maestro
controlekaart
controlekaarten
credit card
credit cards
credit card
credit cards
debetkaart
debetkaarten
debit card
debit cards
debit card
debit cards
debito automatico
diners club
diners club
discover
discover card
discover cards
discover card
discover cards
débito automático
edc
eigentümername
european debit card
hoofdkaart
hoofdkaarten
in viaggio
japanese card bureau
japanse kaartdienst
jcb
kaart
kaart num
kaartaantal
kaartaantallen
kaarthouder
kaarthouders
karte
karteninhaber
karteninhabers
kartennr
kartennummer
kreditkarte
kreditkarten-nummer
kreditkarteninhaber
kreditkarteninstitut
kreditkartennummer
kreditkartentyp
maestro
master card
master cards
Mastercard
master cards
mc
mister cash
n carta
carta
no de tarjeta
no do cartao
no do cartão
no. de tarjeta
no. do cartao
no. do cartão
nr carta
nr. carta
numeri di scheda
numero carta
numero de cartao
numero de carte
numero de cartão
numero de tarjeta
numero della carta
numero di carta
numero di scheda
numero do cartao
numero do cartão
numéro de carte
nº carta
nº de carte
nº de la carte
nº de tarjeta
nº do cartao
nº do cartão
nº. do cartão
número de cartao
número de cartão
número de tarjeta
número do cartao
scheda dell'assegno
scheda dell'atmosfera
scheda dell'atmosfera
scheda della banca
scheda di controllo
scheda di debito
scheda matrice
schede dell'atmosfera
schede di controllo
schede di debito
schede matrici
scoprono la scheda
scoprono le schede
solo
supporti di scheda
supporto di scheda
switch
tarjeta atm
tarjeta credito
tarjeta de atm
tarjeta de credito
tarjeta de debito
tarjeta debito
tarjeta no
tarjetahabiente
tipo della scheda
ufficio giapponese della
scheda
v pay
v-pay
visa
visa plus
visa electron
visto
visum
vpay
```

#### <a name="keyword_card_security_terms_dict"></a>Keyword\_card\_security\_terms\_dict

```
card identification number
card verification
cardi la verifica
cid
cod seg
cod seguranca
cod segurança
cod sicurezza
cod. seg
cod. seguranca
cod. segurança
cod. sicurezza
codice di sicurezza
codice di verifica
codigo
codigo de seguranca
codigo de segurança
crittogramma
cryptogram
cryptogramme
cv2
cvc
cvc2
cvn
cvv
cvv2
cód seguranca
cód segurança
cód. seguranca
cód. segurança
código
código de seguranca
código de segurança
de kaart controle
geeft nr uit
issue no
issue number
kaartidentificatienummer
kreditkartenprufnummer
kreditkartenprüfnummer
kwestieaantal
no. dell'edizione
no. di sicurezza
numero de securite
numero de verificacao
numero dell'edizione
numero di identificazione della
scheda
numero di sicurezza
numero van veiligheid
numéro de sécurité
nº autorizzazione
número de verificação
perno il blocco
pin block
prufziffer
prüfziffer
security code
security no
security number
sicherheits kode
sicherheitscode
sicherheitsnummer
speldblok
veiligheid nr
veiligheidsaantal
veiligheidscode
veiligheidsnummer
verfalldatum
```

#### <a name="keyword_card_expiration_terms_dict"></a>Keyword\_card\_expiration\_terms\_dict

```
ablauf
data de expiracao
data de expiração
data del exp
data di exp
data di scadenza
data em que expira
data scad
data scadenza
date de validité
datum afloop
datum van exp
de afloop
espira
espira
exp date
exp datum
expiration
expire
expires
expiry
fecha de expiracion
fecha de venc
gultig bis
gultigkeitsdatum
gültig bis
gültigkeitsdatum
la scadenza
scadenza
valable
validade
valido hasta
valor
venc
vencimento
vencimiento
verloopt
vervaldag
vervaldatum
vto
válido hasta
```

## <a name="eu-drivers-license-number"></a>Numéro de permis de conduire dans l’Union européenne

Il s’agit des entités du type d’informations sensibles Numéro de permis de conduire dans l’Union européenne.

- Autriche
- Belgique
- Bulgarie
- Croatie
- Chypre
- Tchèque
- Danemark
- Estonie
- Finlande
- France
- Allemagne
- Grèce
- Hongrie
- Irlande
- Italie
- Lettonie
- Lituanie
- Luxembourg
- Malte
- Pays-Bas
- Pologne
- Portugal
- Roumanie
- Slovaquie
- Slovénie
- Espagne
- Suède
- au Royaume-Uni

## <a name="eu-national-identification-number"></a>Numéro d’identification nationale dans l’Union européenne

Il s’agit des entités du type d’informations sensibles Numéro d’identification nationale dans l’Union européenne.

- Autriche
- Belgique
- Bulgarie
- Croatie
- Chypre
- Tchèque
- Danemark
- Estonie
- Finlande
- France
- Allemagne
- Grèce
- Hongrie
- Irlande
- Italie
- Lettonie
- Lituanie
- Luxembourg
- Malte
- Pays-Bas
- Pologne
- Portugal
- Roumanie
- Slovaquie
- Slovénie
- Espagne
- au Royaume-Uni

## <a name="eu-passport-number"></a>Numéro de passeport dans l’Union européenne

Il s’agit des entités du type d’informations sensibles Numéro de passeport dans l’Union européenne. Il s’agit des entités du groupe Numéro de passeport dans l’Union européenne.

- Autriche
- Belgique
- Bulgarie
- Croatie
- Chypre
- Tchèque
- Danemark
- Estonie
- Finlande
- France
- Allemagne
- Grèce
- Hongrie
- Irlande
- Italie
- Lettonie
- Lituanie
- Luxembourg
- Malte
- Pays-Bas
- Pologne
- Portugal
- Roumanie
- Slovaquie
- Slovénie
- Espagne
- Suède
- au Royaume-Uni

## <a name="eu-social-security-number-or-equivalent-identification"></a>Numéro de sécurité sociale dans l’Union européenne (ou équivalent)

Il s’agit des entités du type d’informations sensibles Numéro de sécurité sociale dans l’Union européenne (ou équivalent).

- Autriche
- Belgique
- Croatie
- Tchèque
- Danemark
- Finlande
- France
- Allemagne
- Grèce
- Hongrie
- Portugal
- Espagne
- Suède

## <a name="eu-tax-identification-number"></a>Numéros d’identification fiscale dans l’Union européenne

Il s’agit des entités du type d’informations sensibles Numéros d’identification fiscale dans l’Union européenne.

- Autriche
- Belgique
- Bulgarie
- Croatie
- Chypre
- Tchèque
- Danemark
- Estonie
- Finlande
- France
- Allemagne
- Grèce
- Hongrie
- Irlande
- Italie
- Lettonie
- Lituanie
- Luxembourg
- Malte
- Pays-Bas
- Pologne
- Portugal
- Roumanie
- Slovaquie
- Slovénie
- Espagne
- Suède
- au Royaume-Uni



## <a name="finland-national-id"></a>Numéro d’identité nationale en Finlande

### <a name="format"></a>Format

six chiffres plus un caractère indiquant un siècle plus trois chiffres plus un chiffre de vérification

### <a name="pattern"></a>Modèle

Le modèle doit inclure tous les éléments suivants :

- six chiffres au format DDMMYY (date de naissance)
- marqueur de siècle (-, + ou a)
- numéro d’identification personnelle à 3 chiffres
- un chiffre ou une lettre (respectant la casse), qui est un chiffre de vérification

### <a name="keywords"></a>Mots clés

```
ainutlaatuinen henkilökohtainen tunnus
henkilökohtainen tunnus
henkilötunnus
henkilötunnusnumero#
henkilötunnusnumero
hetu
ID no
ID number
identification number
identiteetti numero
identity number
ID number
kansallinen henkilötunnus
kansallisen henkilökortin
national ID card
national ID no.
personal ID
personal identity code
personalidnumber#
personbeteckning
personnummer
social security number
sosiaaliturvatunnus
tax ID
tax identification no
tax identification number
tax no#
tax no
tax number
tax registration number
tax ID#
tax ID no#
tax ID number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
tunnistenumero
tunnus numero
tunnusluku
tunnusnumero
verokortti
veronumero
verotunniste
verotunnus
```

## <a name="finland-passport-number"></a>Numéro de passeport en Finlande

Cette entité de type d’informations sensibles est disponible dans le type d’informations sensibles Numéro de passeport européen et elle est disponible en tant qu’entité de type d’informations sensibles autonome.

### <a name="format"></a>Format

combinaison de neuf lettres et chiffres

### <a name="pattern"></a>Modèle

combinaison de neuf lettres et chiffres :

- deux lettres (ne respectant pas la casse)
- sept chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keywords_eu_passport_number_common"></a>Keywords\_eu\_passport\_number\_common

```
passport#
passport #
passport ID
passports
passport no
passport no
passport number
passport number
passport numbers
passport numbers
```

#### <a name="keyword_finland_passport_number"></a>Keyword\_finland\_passport\_number

```
suomalainen passi
passin numero
passin numero.#
passin numero#
passin numero.
passi#
passi number
```

## <a name="france-drivers-license-number"></a>Numéro de permis de conduire en France

Cette entité de type d’informations sensibles est incluse dans le type d’information sensible Numéro de permis de conduire européen et elle est disponible en tant qu’entité de type d’informations sensibles autonome.

### <a name="format"></a>Format

12 chiffres

### <a name="pattern"></a>Modèle

12 chiffres avec validation pour tenir compte des modèles similaires tels que les numéros de téléphone français

### <a name="keywords"></a>Mots clés

#### <a name="keyword_french_drivers_license"></a>Keyword\_french\_drivers\_license

```
driverlic
driverlics
driver license
driver licenses
driverlicence
driverlicences
driver lic
driver lics
driver license
driver licenses
driver license
driver licenses
driverslic
driverslics
driverslicence
driverslicences
drivers license
drivers licenses
drivers lic
drivers lics
drivers license
drivers licenses
drivers license
drivers licenses
driver'lic
driver'lics
driver'license
driver'licenses
driver'license
driver'licenses
driver' lic
driver' lics
driver' license
driver' licenses
driver' license
driver' licenses
driver'slic
driver'slics
driver'license
driver'licenses
driver'license
driver'licenses
driver's lic
driver's lics
driver's license
driver's licenses
driver's license
driver's licenses
dl#
dls#
driverlic#
driverlics#
driver license#
driver licenses#
driverlicence#
driverlicences#
driver lic#
driver lics#
driver license#
driver licenses#
driver licenses#
driverslic#
driverslics#
drivers license#
drivers licenses#
driverslicence#
driverslicences#
drivers lic#
drivers lics#
drivers license#
drivers licenses#
drivers license#
drivers licenses#
driver'lic#
driver'lics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver' lic#
driver' lics#
driver' license#
driver' licenses#
driver' license#
driver' licenses#
driver'slic#
driver'slics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver's lic#
driver's lics#
driver's license#
driver's licenses#
driver's license#
driver's licenses#
driving license
driving license
dlno#
driv lic
driv licen
driv license
driv licenses
driv license
driv licenses
driver licen
drivers licen
driver's licen
driving lic
driving licen
driving licenses
driving license
driving licenses
driving permit
dl no
dlno
dl number
permis de conduire
license number
license number
license numbers
license numbers
numéros de license
```

## <a name="france-national-id-card-cni"></a>Carte nationale d’identité (CNI) en France

### <a name="format"></a>Format

12 chiffres

### <a name="pattern"></a>Modèle

12 chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keywords_france_eu_national_id_card"></a>Keywords\_france\_eu\_national\_id\_card

```
card number
carte nationale d'identité
carte nationale d'idenite no
cni#
cni
compte bancaire
national identification number
national identity
nationalidno#
numéro d'assurance maladie
numéro de carte vitale
```

## <a name="france-passport-number"></a>Numéro de passeport en France

Cette entité de type d’informations sensibles est disponible dans le type d’informations sensibles Numéro de passeport européen et elle est disponible en tant qu’entité de type d’informations sensibles autonome.

### <a name="format"></a>Format

neuf chiffres et lettres

### <a name="pattern"></a>Modèle

neuf chiffres et lettres :

- deux chiffres
- deux lettres (ne respectant pas la casse)
- cinq chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_passport"></a>Keyword\_passport

```
Passport Number
Passport No
Passport #
Passport#
PassportID
Passport no
passport number
パスポート
パスポート番号
パスポートのNum
パスポート＃
Numéro de passeport
Passeport n °
Passeport Non
Passeport #
Passeport#
PasseportNon
Passeportn °
```

## <a name="france-social-security-number-insee-or-equivalent-identification"></a>Numéro de sécurité sociale en France (INSEE) ou équivalent

Cette entité de type d’informations sensibles est incluse dans le type d’informations sensibles Numéro de sécurité sociale en France (INSEE) ou équivalent et elle est disponible en tant qu’entité de type d’informations sensibles autonome.

### <a name="format"></a>Format

15 chiffres

### <a name="pattern"></a>Modèle

Doit correspondre à l’un des deux modèles suivants :

- 13 chiffres suivis d’un espace suivi de deux chiffres ou
- 15 chiffres consécutifs

### <a name="keywords"></a>Mots clés

#### <a name="keyword_fr_insee"></a>Keyword\_fr\_insee

```
insee
securité sociale
securite sociale
national ID
national identification
numéro d'identité
no d'identité
no. d'identité
numero d'identite
no d'identite
no. d'identite
social security number
social security code
social insurance number
le numéro d'identification nationale
d'identité nationale
numéro de sécurité sociale
le code de la sécurité sociale
numéro d'assurance sociale
numéro de sécu
code sécu
```

## <a name="germany-drivers-license-number"></a>Numéro de permis de conduire en Allemagne

Cette entité de type d’informations sensibles est incluse dans le type d’information sensible Numéro de permis de conduire européen et elle est disponible en tant qu’entité de type d’informations sensibles autonome.

### <a name="format"></a>Format

combinaison de 11 chiffres et lettres

### <a name="pattern"></a>Modèle

11 chiffres et lettres (ne respectant pas la casse) :

- un chiffre ou une lettre
- deux chiffres
- six chiffres ou lettres
- un chiffre
- un chiffre ou une lettre

### <a name="keywords"></a>Mots clés

#### <a name="keyword_german_drivers_license_number"></a>Keyword\_german\_drivers\_license\_number

```
ausstellungsdatum
ausstellungsort
ausstellende behöde
ausstellende behorde
ausstellende behoerde
führerschein
fuhrerschein
fuehrerschein
führerscheinnummer
fuhrerscheinnummer
fuehrerscheinnummer
führerschein-
fuhrerschein-
fuehrerschein-
führerscheinnummernr
fuhrerscheinnummernr
fuehrerscheinnummernr
führerscheinnummerklasse
fuhrerscheinnummerklasse
fuehrerscheinnummerklasse
nr-führerschein
nr-fuhrerschein
nr-fuehrerschein
no-führerschein
no-fuhrerschein
no-fuehrerschein
n-führerschein
n-fuhrerschein
n-fuehrerschein
permis de conduire
driverlic
driverlics
driver license
driver licenses
driverlicence
driverlicences
driver lic
driver lics
driver license
driver licenses
driver license
driver licenses
driverslic
driverslics
driverslicence
driverslicences
drivers license
drivers licenses
drivers lic
drivers lics
drivers license
drivers licenses
drivers license
drivers licenses
driver'lic
driver'lics
driver'license
driver'licenses
driver'license
driver'licenses
driver' lic
driver' lics
driver' license
driver' licenses
driver' license
driver' licenses
driver'slic
driver'slics
driver'license
driver'licenses
driver'license
driver'licenses
driver's lic
driver's lics
driver's license
driver's licenses
driver's license
driver's licenses
dl#
dls#
driverlic#
driverlics#
driver license#
driver licenses#
driverlicence#
driverlicences#
driver lic#
driver lics#
driver license#
driver licenses#
driver licenses#
driverslic#
driverslics#
drivers license#
drivers licenses#
driverslicence#
driverslicences#
drivers lic#
drivers lics#
drivers license#
drivers licenses#
drivers license#
drivers licenses#
driver'lic#
driver'lics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver' lic#
driver' lics#
driver' license#
driver' licenses#
driver' license#
driver' licenses#
driver'slic#
driver'slics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver's lic#
driver's lics#
driver's license#
driver's licenses#
driver's license#
driver's licenses#
driving license
driving license
dlno#
driv lic
driv licen
driv license
driv licenses
driv license
driv licenses
driver licen
drivers licen
driver's licen
driving lic
driving licen
driving licenses
driving license
driving licenses
driving permit
dlno
```

## <a name="germany-identity-card-number"></a>Numéro de carte d’identité en Allemagne

### <a name="format"></a>Format

depuis le 1er novembre 2010 : Neufs lettres et chiffres

du 1er avril 1987 jusqu’au 31 octobre 2010 : 10 chiffres

### <a name="pattern"></a>Modèle

depuis le 1er novembre 2010 :

- une lettre (ne respectant pas la casse)
- huit chiffres

du 1er avril 1987 jusqu’au 31 octobre 2010 :

- 10 chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_germany_id_card"></a>Keyword\_germany\_id\_card

```
ausweis
gpid
identification
identifikation
identifizierungsnummer
identity card
identity number
id-nummer
personal ID
personalausweis
persönliche ID nummer
persönliche identifikationsnummer
persönliche-id-nummer
```

## <a name="germany-passport-number"></a>Numéro de passeport en Allemagne

Cette entité de type d’informations sensibles est incluse dans le type d’informations sensibles Numéro de passeport européen et elle est disponible en tant qu’entité de type d’informations sensibles autonome.

### <a name="format"></a>Format

10 chiffres ou lettres

### <a name="pattern"></a>Modèle

Le modèle doit inclure tous les éléments suivants :

- le premier caractère est un chiffre ou une lettre de cet ensemble (C, F, G, H, J, K)
- trois chiffres
- cinq chiffres ou lettres de cet ensemble (C,H, J-N, P, R, T, V-Z)
- un chiffre

### <a name="keywords"></a>Mots clés

#### <a name="keyword_german_passport"></a>Keyword\_german\_passport

```
reisepasse
reisepassnummer
No-Reisepass
Nr-Reisepass
Reisepass-Nr
Passnummer
reisepässe
passeport no.
passeport no
```

## <a name="greece-national-id-card"></a>Numéro de carte d’identité nationale en Grèce

### <a name="format"></a>Format

Combinaison de 7-8 lettres et chiffres plus un tiret

### <a name="pattern"></a>Modèle

Sept lettres et chiffres (ancien format) :

- Une lettre (n’importe quelle lettre de l’alphabet grec)
- Tiret
- Six chiffres

Huit lettres et chiffres (nouveau format) :

- Deux lettres dont les caractères majuscules se trouvent à la fois dans les alphabets grecs et latins (ABEZHIKMNOPTYX)
- Tiret
- Six chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_greece_id_card"></a>Keyword\_greece\_id\_card

```
greek ID
greek national ID
greek personal ID card
greek police ID
identity card
tautotita
ταυτότητα
ταυτότητας
```

## <a name="hong-kong-identity-card-hkid-number"></a>Numéro de carte d’identité à Hong Kong (R.A.S.)

### <a name="format"></a>Format

Combinaison de 8-9 lettres et chiffres plus les parenthèses facultatives autour du caractère final

### <a name="pattern"></a>Modèle

Combinaison de 8-9 lettres :

- 1-2 lettres (ne respectant pas la casse)
- Six chiffres
- Le caractère final (n’importe quel chiffre ou lettre A), qui est le chiffre de vérification et est éventuellement placé entre parenthèses.

### <a name="keywords"></a>Mots clés

#### <a name="keyword_hong_kong_id_card"></a>Keyword\_hong\_kong\_id\_card

```
hkid
Hong Kong identity card
HKIDC
ID card
identity card
hk identity card
Hong Kong ID
香港身份證
香港永久性居民身份證
身份證
身份証
身分證
身分証
香港身份証
香港身分證
香港身分証
香港身份證
香港居民身份證
香港居民身份証
香港居民身分證
香港居民身分証
香港永久性居民身份証
香港永久性居民身分證
香港永久性居民身分証
香港永久性居民身份證
香港非永久性居民身份證
香港非永久性居民身份証
香港非永久性居民身分證
香港非永久性居民身分証
香港特別行政區永久性居民身份證
香港特別行政區永久性居民身份証
香港特別行政區永久性居民身分證
香港特別行政區永久性居民身分証
香港特別行政區非永久性居民身份證
香港特別行政區非永久性居民身份証
香港特別行政區非永久性居民身分證
香港特別行政區非永久性居民身分証
```

## <a name="ip-address"></a>Adresse IP

### <a name="format"></a>Format

#### <a name="ipv4"></a>IPv4 :

Modèle complexe, qui tient compte des versions mises en forme (points) et non mises en forme (absence de point) des adresses IPv4

#### <a name="ipv6"></a>IPv6 :

Modèle complexe qui compte les numéros IPv6 mis en forme (y compris les signes deux-points)

### <a name="pattern"></a>Modèle

### <a name="keywords"></a>Mots clés

#### <a name="keyword_ipaddress"></a>Keyword\_ipaddress

```
IP (this keyword is case-sensitive)
ip address
ip addresses
internet protocol
IP-כתובת ה
```

## <a name="india-permanent-account-number-pan"></a>Numéro de compte permanent (PAN) en Inde

### <a name="format"></a>Format

10 lettres ou chiffres

### <a name="pattern"></a>Modèle

10 lettres ou chiffres :

- Trois lettres (ne respectant pas la casse)
- Une lettre parmi C, P, H, F, A, T, B, L, J, G (ne respecte pas la casse)
- Une lettre
- Quatre chiffres
- Une lettre (ne respectant pas la casse)

### <a name="keywords"></a>Mots clés

#### <a name="keyword_india_permanent_account_number"></a>Keyword\_india\_permanent\_account\_number

```
Permanent Account Number
PAN
```

## <a name="india-unique-identification-aadhaar-number"></a>Numéro d’identification unique (Aadhaar) en Inde

### <a name="format"></a>Format

12 chiffres contenant des espaces ou des tirets facultatifs

### <a name="pattern"></a>Modèle

12 chiffres :

- Un chiffre, qui n’est pas 0 ou 1
- Trois chiffres
- Espace ou tiret facultatif
- Quatre chiffres
- Espace ou tiret facultatif
- Le chiffre final, qui est le chiffre de vérification

### <a name="keywords"></a>Mots clés

#### <a name="keyword_india_aadhar"></a>Keyword\_india\_aadhar

```
aadhaar
aadhar
aadhar#
uid
आधार
uidai
```

## <a name="indonesia-identity-card-ktp-number"></a>Numéro de carte d’identité (KTP) en Indonésie

### <a name="format"></a>Format

16 chiffres contenant des points facultatifs

### <a name="pattern"></a>Modèle

16 chiffres :

- Code de province à deux chiffres
- Point (facultatif)
- Code de régence ou ville à deux chiffres
- Code de sous-secteur à deux chiffres
- Point (facultatif)
- Six chiffres au format DDMMYY (date de naissance)
- Point (facultatif)
- Quatre chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_indonesia_id_card"></a>Keyword\_indonesia\_id\_card

```
KTP
Kartu Tanda Penduduk
Nomor Induk Kependudukan
```

## <a name="international-banking-account-number-iban"></a>Numéro de compte bancaire international (IBAN)

### <a name="format"></a>Format

Code pays (deux lettres) plus chiffres de vérification (deux chiffres) plus :::no-loc text="bban"::: nombre (jusqu’à 30 caractères)

### <a name="pattern"></a>Modèle

Le modèle doit inclure tous les éléments suivants :

- Code pays à deux lettres
- Deux chiffres de vérification (suivis d’un espace facultatif)
- 1-7 groupes de quatre lettres ou chiffres (peuvent être séparés par des espaces)
- 1-3 lettres ou chiffres

Le format de chaque pays est légèrement différent. Le type d’informations sensibles IBAN couvre les 60 pays suivants :

```
ad, ae, al, at, az, ba, be, bg, bh, ch, cr, cy, cz, de, dk, do, ee, es, fi, fo, fr, gb, ge, gi, gl, gr, hr, hu, that is, il, is, it, kw, kz, lb, li, lt, lu, lv, mc, md, me, mk, mr, mt, mu, nl, no, pl, pt, ro, rs, sa, se, si, sk, sm, tn, tr, vg
```

### <a name="keywords"></a>Mots clés

Aucun

## <a name="ireland-personal-public-service-pps-number"></a>Numéro de service public personnel (PPS) en Irlande

### <a name="format"></a>Format

Ancien format (jusqu’au 31 décembre 2012) :

- sept chiffres suivis de 1-2 lettres

Nouveau format (à partir du 1er janvier 2013) :

- sept chiffres suivis de deux lettres

### <a name="pattern"></a>Modèle

Ancien format (jusqu’au 31 décembre 2012) :

- sept chiffres
- une à deux lettres (sans respect de la casse)

Nouveau format (à partir du 1er janvier 2013) :

- sept chiffres
- lettre (ne respectant pas la casse) qui est un chiffre de vérification alphabétique
- Une lettre facultative dans la plage A-I ou &quot;W&quot;

### <a name="keywords"></a>Mots clés

#### <a name="keywords_ireland_eu_national_id_card"></a>Keywords\_ireland\_eu\_national\_id\_card

```
client identity service
identification number
personal ID number
personal public service number
personal service no
phearsanta seirbhíse poiblí
pps no
pps number
pps num
pps service no
ppsn
ppsno#
ppsno
psp
public service no
publicserviceno#
publicserviceno
revenue and social insurance number
rsi no
rsi number
rsin
seirbhís aitheantais cliant
uimh
uimhir aitheantais chánach
uimhir aitheantais phearsanta
uimhir phearsanta seirbhíse poiblí
tax ID
tax identification no
tax identification number
tax no#
tax no
tax number
tax registration number
tax ID#
tax ID no#
tax ID number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
```

## <a name="israel-bank-account-number"></a>Numéro de compte bancaire en Israël

### <a name="format"></a>Format

13 chiffres

### <a name="pattern"></a>Modèle

Mis en forme :

- deux chiffres
- tiret
- trois chiffres
- tiret
- huit chiffres

Sans mise en forme :

- 13 chiffres consécutifs

### <a name="keywords"></a>Mots clés

#### <a name="keyword_israel_bank_account_number"></a>Keyword\_israel\_bank\_account\_number

```
Bank Account Number
Bank Account
Account Number
מספר חשבון בנק
```

## <a name="israel-national-identification-number"></a>Numéro d’identification nationale en Israël

### <a name="format"></a>Format

neuf chiffres

### <a name="pattern"></a>Modèle

neuf chiffres consécutifs

### <a name="keywords"></a>Mots clés

#### <a name="keyword_israel_national_id"></a>Keyword\_Israel\_National\_ID

```
מספר זהות
מספר זיה וי
מספר זיהוי ישר אלי
זהותישר אלית
هو ية اسرائيل ية عدد
هوية إسرائ يلية
رقم الهوية
عدد هوية فريدة من نوعها
ID number#
ID number
identity no
identity number#
identity number
israeliidentitynumber
personal ID
unique ID
```

## <a name="italy-drivers-license-number"></a>Numéro de permis de conduire en Italie

Cette entité de type d’informations sensibles est incluse dans le type d’information sensible Numéro de permis de conduire européen et elle est disponible en tant qu’entité de type d’informations sensibles autonome.

### <a name="format"></a>Format

combinaison de 10 lettres et chiffres

### <a name="pattern"></a>Modèle

combinaison de 10 lettres et chiffres :

- une lettre (ne respectant pas la casse)
- la lettre &quot;A&quot; ou &quot;V&quot; (sans respect de la casse)
- sept chiffres
- une lettre (ne respectant pas la casse)

### <a name="keywords"></a>Mots clés

#### <a name="keyword_italy_drivers_license_number"></a>Keyword\_italy\_drivers\_license\_number

```
numero di patente
patente di guida
patente guida
patenti di guida
patenti guida
```

## <a name="japan-bank-account-number"></a>Numéro de compte bancaire au Japon

### <a name="format"></a>Format

sept ou huit chiffres

### <a name="pattern"></a>Modèle

Numéro de compte bancaire :

- sept ou huit chiffres
- Code de la succursale du compte bancaire :
- quatre chiffres
- espace ou tiret (facultatif)
- trois chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_jp_bank_account"></a>Keyword\_jp\_bank\_account

```
Checking Account Number
Checking Account
Checking Account #
Checking Acct Number
Checking Acct #
Checking Acct No.
Checking Account No.
Bank Account Number
Bank Account
Bank Account #
Bank Acct Number
Bank Acct #
Bank Acct No.
Bank Account No.
Savings Account Number
Savings Account
Savings Account #
Savings Acct Number
Savings Acct #
Savings Acct No.
Savings Account No.
Debit Account Number
Debit Account
Debit Account #
Debit Acct Number
Debit Acct #
Debit Acct No.
Debit Account No.
口座番号
銀行口座
銀行口座番号
総合口座
普通預金口座
普通口座
当座預金口座
当座口座
預金口座
振替口座
銀行
バンク
```

#### <a name="keyword_jp_bank_branch_code"></a>Keyword\_jp\_bank\_branch\_code

```
支店番号
支店コード
店番号
```

## <a name="japan-drivers-license-number"></a>Numéro de permis de conduire au Japon

### <a name="format"></a>Format

12 chiffres

### <a name="pattern"></a>Modèle

12 chiffres consécutifs

### <a name="keywords"></a>Mots clés

#### <a name="keyword_jp_drivers_license_number"></a>Keyword\_jp\_drivers\_license\_number

```
driver license
drivers license
driver'license
drivers licenses
driver'licenses
driver licenses
dl#
dls#
lic#
lics#
運転免許証
運転免許
免許証
免許
運転免許証番号
運転免許番号
免許証番号
免許番号
運転免許証ナンバー
運転免許ナンバー
免許証ナンバー
運転免許証no
運転免許no
免許証no
免許no
運転経歴証明書番号
運転経歴証明書
運転免許証No.
運転免許No.
免許証No.
免許No.
運転免許証#
運転免許#
免許証#
免許#
```

## <a name="japan-passport-number"></a>Numéro de passeport au Japon

### <a name="format"></a>Format

deux lettres suivies de sept chiffres

### <a name="pattern"></a>Modèle

deux lettres (ne respectant pas la casse) suivies de sept chiffres

#### <a name="keyword_jp_passport"></a>Keyword\_jp\_passport

```
Passport
Passport Number
Passport No.
Passport #
パスポート
パスポート番号
パスポートナンバー
パスポート＃
パスポート#
パスポートNo.
旅券番号
旅券番号＃
旅券番号♯
旅券ナンバー
```

## <a name="japan-residence-card-number"></a>Numéro de carte de résidence au Japon

### <a name="format"></a>Format

12 lettres et chiffres

### <a name="pattern"></a>Modèle

12 lettres et chiffres :

- deux lettres (ne respectant pas la casse)
- huit chiffres
- deux lettres (ne respectant pas la casse)

### <a name="keywords"></a>Mots clés

#### <a name="keyword_jp_residence_card_number"></a>Keyword\_jp\_residence\_card\_number

```
Residence card number
Residence card no
Residence card #
在留カード番号
在留カード
在留番号
```

## <a name="japan-resident-registration-number"></a>Numéro d’inscription de résident au Japon

### <a name="format"></a>Format

11 chiffres

### <a name="pattern"></a>Modèle

11 chiffres consécutifs

### <a name="keywords"></a>Mots clés

#### <a name="keyword_jp_resident_registration_number"></a>Keyword\_jp\_resident\_registration\_number

```
Resident Registration Number
Residents Basic Registry Number
Resident Registration No.
Resident Register No.
Residents Basic Registry No.
Basic Resident Register No.
外国人登録証明書番号
証明書番号
登録番号
外国人登録証
```

## <a name="japan-social-insurance-number-sin"></a>Numéro d’assurance sociale (SIN) au Japon

### <a name="format"></a>Format

7-12 chiffres

### <a name="pattern"></a>Modèle

7-12 chiffres :

- quatre chiffres
- un trait d’union (facultatif)
- six chiffres OU
- 7-12 chiffres consécutifs

### <a name="keywords"></a>Mots clés

#### <a name="keyword_jp_sin"></a>Keyword\_jp\_sin

```
Social Insurance No.
Social Insurance Num
Social Insurance Number
健康保険被保険者番号
健保番号
基礎年金番号
雇用保険被保険者番号
雇用保険番号
保険証番号
社会保険番号
社会保険No.
社会保険
介護保険
介護保険被保険者番号
健康保険被保険者整理番号
雇用保険被保険者整理番号
厚生年金
厚生年金被保険者整理番号
```

## <a name="malaysia-identification-card-number"></a>Numéro de carte d’identification de Malaisie

### <a name="format"></a>Format

12 chiffres contenant des tirets facultatifs

### <a name="pattern"></a>Modèle

12 chiffres :

- six chiffres au format YYMMDD (date de naissance)
- tiret (facultatif)
- code de lieu de naissance à deux lettres
- tiret (facultatif)
- trois chiffres aléatoires
- code de sexe à un chiffre

### <a name="keywords"></a>Mots clés

#### <a name="keyword_malaysia_id_card_number"></a>Keyword\_malaysia\_id\_card\_number

```
digital application card
i/c
i/c no
ic
ic no
ID card
identification Card
identity card
k/p
k/p no
kad akuan diri
kad aplikasi digital
kad pengenalan malaysia
kp
kp no
mykad
mykas
mykid
mypr
mytentera
malaysia identity card
malaysian identity card
nric
personal identification card
```

## <a name="netherlands-citizens-service-bsn-number"></a>Numéro de service citoyen (BSN) au Pays-Bas

### <a name="format"></a>Format

huit à neuf chiffres contenant des espaces facultatifs

### <a name="pattern"></a>Modèle

huit à neuf chiffres :

- trois chiffres
- espace (facultatif)
- trois chiffres
- espace (facultatif)
- deux à trois chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keywords_netherlands_eu_national_id_card"></a>Keywords\_netherlands\_eu\_national\_id\_card

```
bsn#
bsn
burgerservicenummer
citizen service number
person number
personal number
personal numeric code
person-related number
persoonlijk nummer
persoonlijke numerieke code
persoonsgebonden
persoonsnummer
sociaal-fiscaal nummer
social-fiscal number
sofi
sofinummer
uniek identificatienummer
uniek identiteitsnummer
unique identification number
unique identity number
uniqueidentityno#
```

## <a name="new-zealand-ministry-of-health-number"></a>Numéro du ministère de la santé en Nouvelle-Zélande

### <a name="format"></a>Format

trois lettres, un espace (facultatif) et quatre chiffres

### <a name="pattern"></a>Modèle

- trois lettres (ne respectant pas la casse) sauf « I » et « O »
- espace (facultatif)
- quatre chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_nz_terms"></a>Keyword\_nz\_terms

```
NHI
New Zealand
Health
treatment
National Health Index Number
nhi number
nhi no.
NHI#
National Health Index No.
National Health Index Id
National Health Index #
```

## <a name="norway-identification-number"></a>Numéro d’identification de la Norvège

### <a name="format"></a>Format

11 chiffres

### <a name="pattern"></a>Modèle

11 chiffres :

- six chiffres au format DDMMYY (date de naissance)
- nombre individuel à trois chiffres
- deux chiffres de vérification

### <a name="keywords"></a>Mots clés

#### <a name="keyword_norway_id_number"></a>Keyword\_norway\_id\_number

```
Personal identification number
Norwegian ID Number
ID Number
Identification
Personnummer
Fødselsnummer
```

## <a name="philippines-unified-multi-purpose-identification-number"></a>Numéro d’identité polyvalent unifié aux Philippines

### <a name="format"></a>Format

12 chiffres séparés par des traits d’union

### <a name="pattern"></a>Modèle

12 chiffres :

- quatre chiffres
- trait d’union
- sept chiffres
- trait d’union
- un chiffre

### <a name="keywords"></a>Mots clés

#### <a name="keyword_philippines_id"></a>Keyword\_philippines\_id

```
Unified Multi-Purpose ID
UMID
Identity Card
Pinag-isang Multi-Layunin ID
```

## <a name="poland-identity-card"></a>Carte d’identité en Pologne

### <a name="format"></a>Format

trois lettres et six chiffres

### <a name="pattern"></a>Modèle

trois lettres (ne respectant pas la casse) suivies de six chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_poland_national_id_passport_number"></a>Keyword\_poland\_national\_id\_passport\_number

```
Dowód osobisty
Numer dowodu osobistego
Nazwa i numer dowodu osobistego
Nazwa i nr dowodu osobistego
Nazwa i nr dowodu tożsamości
Dowód Tożsamości
dow. os.
```

## <a name="poland-national-id-pesel"></a>Numéro d’identité nationale (PESEL) en Pologne

### <a name="format"></a>Format

11 chiffres

### <a name="pattern"></a>Modèle

- 6 chiffres représentant la date de naissance au format AAMMJJ
- 4 chiffres
- 1 chiffre de vérification

### <a name="keywords"></a>Mots clés

#### <a name="keyword_pesel_identification_number"></a>Keyword\_pesel\_identification\_number

```
dowód osobisty
dowódosobisty
niepowtarzalny numer
niepowtarzalnynumer
nr.-pesel
nr-pesel
numer identyfikacyjny
pesel
tożsamości narodowej
```

## <a name="poland-passport-number"></a>Numéro de passeport en Pologne

Cette entité de type d’informations sensibles est incluse dans le type d’informations sensibles Numéro de passeport européen et elle est disponible en tant qu’entité de type d’informations sensibles autonome.

### <a name="format"></a>Format

deux lettres et sept chiffres

### <a name="pattern"></a>Modèle

Deux lettres (ne respectant pas la casse) suivies de sept chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_poland_national_id_passport_number"></a>Keyword\_poland\_national\_id\_passport\_number

```
Numer paszportu
Nr. Paszportu
Paszport
```

## <a name="portugal-citizen-card-number"></a>Numéro de carte de citoyen au Portugal

### <a name="format"></a>Format

huit chiffres

### <a name="pattern"></a>Modèle

huit chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_portugal_citizen_card"></a>Keyword\_portugal\_citizen\_card

```
bilhete de identidade
cartão de cidadão
citizen card
document number
documento de identificação
ID number
identification no
identification number
identity card no
identity card number
national ID card
nic
número bi de portugal
número de identificação civil
número de identificação fiscal
número do documento
portugal bi number
```

## <a name="portugal-drivers-license-number"></a>Numéro de permis de conduire au Portugal

Cette entité de type d’informations sensibles est uniquement disponible dans le type d’informations sensibles Numéro de licence de conducteur européen.

### <a name="format"></a>Format

deux modèles : deux lettres suivies de 5-8 chiffres avec des caractères spéciaux

### <a name="pattern"></a>Modèle

Modèle 1 : Deux lettres suivies de 5/6 avec des caractères spéciaux :

- Deux lettres (ne respectant pas la casse)
- Un trait d’union
- Cinq ou six chiffres
- Un espace
- Un chiffre

Modèle 2 : Une lettre suivie de 6/8 chiffres avec des caractères spéciaux :

- Une lettre (ne respectant pas la casse)
- Un trait d’union
- Six ou huit chiffres
- Un espace
- Un chiffre

### <a name="keywords"></a>Mots clés

#### <a name="keywords_eu_drivers_license_number"></a>Keywords\_eu\_driver's\_license\_number

```
driverlic
driverlics
driver license
driver licenses
driverlicence
driverlicences
driver lic
driver lics
driver license
driver licenses
driver license
driver licenses
driverslic
driverslics
driverslicence
driverslicences
drivers license
drivers licenses
drivers lic
drivers lics
drivers license
drivers licenses
drivers license
drivers licenses
driver'lic
driver'lics
driver'license
driver'licenses
driver'license
driver'licenses
driver' lic
driver' lics
driver' license
driver' licenses
driver' license
driver' licenses
driver'slic
driver'slics
driver'license
driver'licenses
driver'license
driver'licenses
driver's lic
driver's lics
driver's license
driver's licenses
driver's license
driver's licenses
dl#
dls#
driverlic#
driverlics#
driver license#
driver licenses#
driverlicence#
driverlicences#
driver lic#
driver lics#
driver license#
driver licenses#
driver licenses#
driverslic#
driverslics#
drivers license#
drivers licenses#
driverslicence#
driverslicences#
drivers lic#
drivers lics#
drivers license#
drivers licenses#
drivers license#
drivers licenses#
driver'lic#
driver'lics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver' lic#
driver' lics#
driver' license#
driver' licenses#
driver' license#
driver' licenses#
driver'slic#
driver'slics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver's lic#
driver's lics#
driver's license#
driver's licenses#
driver's license#
driver's licenses#
driving license
driving license
dlno#
driv lic
driv licen
driv license
driv licenses
driv license
driv licenses
driver licen
drivers licen
driver's licen
driving lic
driving licen
driving licenses
driving license
driving licenses
driving permit
dl no
dlno
dl number
```

#### <a name="keywords_portugal_eu_drivers_license_number"></a>Keywords\_portugal\_eu\_driver's\_license\_number

```
carteira de motorista
carteira motorista
carteira de habilitação
carteira habilitação
número de licença
número licença
permissão de condução
permissão condução
Licença condução Portugal
carta de condução
```

## <a name="saudi-arabia-national-id"></a>Numéro national en Arabie saoudite

### <a name="format"></a>Format

10 chiffres

### <a name="pattern"></a>Modèle

10 chiffres consécutifs

### <a name="keywords"></a>Mots clés

#### <a name="keyword_saudi_arabia_national_id"></a>Keyword\_saudi\_arabia\_national\_id

```
Identification Card
I card number
ID number
الوطنية الهوية بطاقة رقم
```

## <a name="singapore-national-registration-identity-card-nric-number"></a>Numéro de carte d’identité d’inscription nationale (NRIC) à Singapour

### <a name="format"></a>Format

neufs lettres et chiffres

### <a name="pattern"></a>Modèle

- neufs lettres et chiffres :
- lettre &quot;F&quot;, &quot;G&quot;, &quot;S&quot; ou &quot;T&quot; (sans respect de la casse)
- sept chiffres
- chiffre de vérification alphabétique

### <a name="keywords"></a>Mots clés

#### <a name="keyword_singapore_nric"></a>Keyword\_singapore\_nric

```
National Registration Identity Card
Identity Card Number
NRIC
IC
Foreign Identification Number
FIN
身份证
身份證
```

## <a name="south-africa-identification-number"></a>Numéro d’identification en Afrique du Sud

### <a name="format"></a>Format

13 chiffres pouvant contenir des espaces

### <a name="pattern"></a>Modèle

13 chiffres :

- six chiffres au format YYMMDD (date de naissance)
- quatre chiffres
- un indicateur de citoyenneté à un seul chiffre
- le chiffre &quot;8&quot; ou &quot;9&quot;
- un chiffre, qui est un chiffre de somme de contrôle

### <a name="keywords"></a>Mots clés

#### <a name="keyword_south_africa_identification_number"></a>Keyword\_south\_africa\_identification\_number

```
Identity card
ID
Identification
```

## <a name="south-korea-resident-registration-number"></a>Numéro d’inscription de résident en Corée du Sud

### <a name="format"></a>Format

13 chiffres contenant un trait d’union

### <a name="pattern"></a>Modèle

13 chiffres :

- six chiffres au format YYMMDD (date de naissance)
- trait d’union
- un chiffre déterminé par le siècle et le sexe
- Code de région de naissance à quatre chiffres
- un chiffre utilisé pour différencier les personnes dont les nombres précédents sont identiques
- 1 chiffre de vérification.

### <a name="keywords"></a>Mots clés

#### <a name="keyword_south_korea_resident_number"></a>Keyword\_south\_korea\_resident\_number

```
National ID card
Citizen's Registration Number
Jumin deungnok beonho
RRN
주민등록번호
```

## <a name="spain-social-security-number-ssn"></a>Numéro de sécurité sociale (SSN) en Espagne

Cette entité de type d’informations sensibles est incluse dans le type d’informations sensibles Numéro de sécurité sociale en France (INSEE) ou équivalent et elle est disponible en tant qu’entité de type d’informations sensibles autonome.

### <a name="format"></a>Format

11-12 chiffres

### <a name="pattern"></a>Modèle

11-12 chiffres :

- deux chiffres
- barre oblique (facultatif)
- sept ou huit chiffres
- barre oblique (facultatif)
- deux chiffres

### <a name="keywords"></a>Mots clés

Aucun

## <a name="sweden-national-id"></a>Numéro d’identité nationale en Suède

### <a name="format"></a>Format

10 ou 12 chiffres et un délimiteur facultatif

### <a name="pattern"></a>Modèle

10 ou 12 chiffres et un délimiteur facultatif :

- deux chiffres (facultatif)
- Six chiffres au format de date AAMMJJ
- délimiteur de &quot;-&quot; ou &quot;+&quot; (facultatif)
- quatre chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keywords_swedish_national_identifier"></a>Keywords\_swedish\_national\_identifier

```
ID no
ID number
ID#
identification no
identification number
identifikationsnumret#
identifikationsnumret
identitetshandling
identity document
identity no
identity number
id-nummer
personal ID
personnummer#
personnummer
skatteidentifikationsnummer
```

## <a name="sweden-passport-number"></a>Numéro de passeport en Suède

Cette entité de type d’informations sensibles est incluse dans le type d’informations sensibles Numéro de passeport européen et elle est disponible en tant qu’entité de type d’informations sensibles autonome.

### <a name="format"></a>Format

huit chiffres

### <a name="pattern"></a>Modèle

huit chiffres consécutifs

### <a name="keywords"></a>Mots clés

#### <a name="keyword_sweden_passport"></a>Keyword\_sweden\_passport

```
visa requirements
Alien Registration Card
Schengen visas
Schengen visa
Visa Processing
Visa Type
Single Entry
Multiple Entries
G3 Processing Fees
```

#### <a name="keyword_passport"></a>Keyword\_passport

```
Passport Number
Passport No
Passport #
Passport#
PassportID
Passport no
passport number
パスポート
パスポート番号
パスポートのNum
パスポート＃
Numéro de passeport
Passeport n °
Passeport Non
Passeport #
Passeport#
PasseportNon
Passeportn °
```

## <a name="swift-code"></a>Code Swift

### <a name="format"></a>Format

quatre lettres suivies de 5 à 31 lettres ou chiffres

### <a name="pattern"></a>Modèle

quatre lettres suivies de 5 à 31 lettres ou chiffres :

- code bancaire à quatre lettres (sans respect de la casse)
- espace facultatif
- 4 à 28 lettres ou chiffres (numéro de compte bancaire de base (IBAN))
- espace facultatif
- un à trois lettres ou chiffres (le reste du IBAN)

### <a name="keywords"></a>Mots clés

#### <a name="keyword_swift"></a>Keyword\_swift

```
international organization for standardization 9362
iso 9362
iso9362
swift#
swift code
swift number
swiftroutingnumber
swift code
swift number #
swift routing number
bic number
bic code
bic #
bic#
bank identifier code
Organization internationale de normalization 9362
rapide #
code SWIFT
le numéro de swift
swift numéro d'acheminement
le numéro BIC
#BIC
code identificateur de banque
SWIFTコード
SWIFT番号
BIC番号
BICコード
SWIFT コード
SWIFT 番号
BIC 番号
BIC コード
金融機関識別コード
金融機関コード
銀行コード
```

## <a name="taiwan-national-identification-number"></a>Numéro d’identification nationale à Taïwan

### <a name="format"></a>Format

une lettre (en anglais) suivie de neuf chiffres

### <a name="pattern"></a>Modèle

une lettre (en anglais) suivie de neuf chiffres :

- une lettre (en anglais) (sans respect de la casse)
- le chiffre &quot;1&quot; ou &quot;2&quot;
- huit chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_taiwan_national_id"></a>Keyword\_taiwan\_national\_id

```
身份證字號
身份證
身份證號碼
身份證號
身分證字號
身分證
身分證號碼
身份證號
身分證統一編號
國民身分證統一編號
簽名
蓋章
簽名或蓋章
簽章
```

## <a name="taiwan-passport-number"></a>Numéro de passeport à Taïwan

### <a name="format"></a>Format

- numéro de passeport biométrique : neuf chiffres
- numéro de passeport non biométrique : neuf chiffres

### <a name="pattern"></a>Modèle

numéro de passeport biométrique :

- le caractère &quot;3&quot;
- huit chiffres

numéro de passeport non biométrique :

- neuf chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_taiwan_passport"></a>Keyword\_taiwan\_passport

```
ROC passport number
Passport number
Passport no
Passport Num
Passport #
护照
中華民國護照
Zhōnghuá Mínguó hùzhào
```

## <a name="taiwan-resident-certificate-arctarc-number"></a>Numéro de certificat de résident (ARC/TARC) à Taïwan

### <a name="format"></a>Format

10 lettres et chiffres

### <a name="pattern"></a>Modèle

10 lettres et chiffres :

- deux lettres (ne respectant pas la casse)
- huit chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_taiwan_resident_certificate"></a>Keyword\_taiwan\_resident\_certificate

```
Resident Certificate
Resident Cert
Resident Cert.
Identification card
Alien Resident Certificate
ARC
Taiwan Area Resident Certificate
TARC
居留證
外僑居留證
台灣地區居留證
```

## <a name="uk-drivers-license-number"></a>au Royaume-Uni Numéro de permis de conduire

Cette entité de type d’informations sensibles est incluse dans le type d’information sensible Numéro de permis de conduire européen et elle est disponible en tant qu’entité de type d’informations sensibles autonome.

### <a name="format"></a>Format

Combinaison de 18 lettres et chiffres dans le format spécifié

### <a name="pattern"></a>Modèle

18 lettres et chiffres :

- cinq lettres (ne respectant pas la casse) ou le chiffre &quot;9&quot; à la place d’une lettre
- un chiffre
- cinq chiffres au format de date MMDDY pour la date de naissance (le septième caractère est incrémenté de 50 si le conducteur est une femme, c’est-à-dire 51 à 62 au lieu de 01 à 12)
- deux lettres (ne respectant pas la casse) ou le chiffre &quot;9&quot; à la place d’une lettre
- cinq chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_uk_drivers_license"></a>Keyword\_uk\_drivers\_license

```
DVLA
light vans
quad bikes
motor cars
125cc
sidecar
tricycles
motorcycles
photo card license
learner drivers
license holder
license holders
driving licenses
driving license
dual control car
```

## <a name="uk-electoral-roll-number"></a>au Royaume-Uni Numéro d’électeur

### <a name="format"></a>Format

deux lettres suivies de 1-4 chiffres

### <a name="pattern"></a>Modèle

deux lettres (ne respectant pas la casse) suivies de 1-4 chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_uk_electoral"></a>Keyword\_uk\_electoral

- Nomination au Conseil
- Formulaire de nomination
- registre électoral
- liste électorale

## <a name="uk-national-health-service-number"></a>au Royaume-Uni Numéro du service de santé national

### <a name="format"></a>Format

10-17 chiffres séparés par des espaces

### <a name="pattern"></a>Modèle

10-17 chiffres :

- trois ou 10 chiffres
- espace
- trois chiffres
- espace
- quatre chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_uk_nhs_number"></a>Keyword\_uk\_nhs\_number

```
national health service
nhs
health services authority
health authority
```

#### <a name="keyword_uk_nhs_number1"></a>Keyword\_uk\_nhs\_number1

```
patient ID
patient identification
patient no
patient number
```

#### <a name="keyword_uk_nhs_number_dob"></a>Keyword\_uk\_nhs\_number\_dob

```
GP
DOB
D.O.B
Date of Birth
Birth Date
```

## <a name="uk-national-insurance-number-nino"></a>au Royaume-Uni Numéro d’assurance nationale (NINO)

Cette entité de type d’informations sensibles est incluse dans le type d’information sensible Numéro d’identification national de l’UE et elle est disponible en tant qu’entité de type d’informations sensibles autonome.

### <a name="format"></a>Format

sept caractères ou neuf caractères séparés par des espaces ou des tirets

### <a name="pattern"></a>Modèle

deux modèles possibles :

- deux lettres (les numéros NINO valide utilisent uniquement certains caractères dans ce préfixe, que ce modèle valide ; ne respecte pas la casse)
- six chiffres
- « A », « B », « C » ou « D » (comme le préfixe, seuls certains caractères sont autorisés dans le suffixe ; ne respecte pas la casse)

OU

- deux lettres
- espace ou tiret
- deux chiffres
- espace ou tiret
- deux chiffres
- espace ou tiret
- deux chiffres
- espace ou tiret
- « A », « B », « C » ou « D »


### <a name="keywords"></a>Mots clés

#### <a name="keyword_uk_nino"></a>Keyword\_uk\_nino

```
national insurance number
national insurance contributions
protection act
insurance
social security number
insurance application
medical application
social insurance
medical attention
social security
Great Britain
NI Number
NI No.
NI #
NI#
insurance#
insurance number
national insurance#
nationalinsurancenumber
```

## <a name="uk-unique-taxpayer-reference-number"></a>au Royaume-Uni Numéro de référence fiscal unique (UTR)

Ce type d’informations sensibles n’est disponible que pour une utilisation dans :

- stratégies de protection contre la perte de données
- stratégies de conformité des communications
- gouvernance des informations
- gestion des enregistrements
- Microsoft Cloud App Security

### <a name="format"></a>Format

10 chiffres sans espaces ni délimiteurs

### <a name="pattern"></a>Modèle

10 chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keywords_uk_eu_tax_file_number"></a>Keywords\_uk\_eu\_tax\_file\_number

```
tax number
tax file
tax ID
tax identification no
tax identification number
tax no#
tax no
tax registration number
tax ID#
tax ID no#
tax ID number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
```

## <a name="us-bank-account-number"></a>Numéro de compte bancaire aux USA

### <a name="format"></a>Format

6-17 chiffres

### <a name="pattern"></a>Modèle

6-17 chiffres consécutifs

### <a name="keywords"></a>Mots clés

#### <a name="keyword_usa_bank_account"></a>Keyword\_usa\_Bank\_Account

```
Checking Account Number
Checking Account
Checking Account #
Checking Acct Number
Checking Acct #
Checking Acct No.
Checking Account No.
Bank Account Number
Bank Account #
Bank Acct Number
Bank Acct #
Bank Acct No.
Bank Account No.
Savings Account Number
Savings Account.
Savings Account #
Savings Acct Number
Savings Acct #
Savings Acct No.
Savings Account No.
Debit Account Number
Debit Account
Debit Account #
Debit Acct Number
Debit Acct #
Debit Acct No.
Debit Account No.
```

## <a name="us-drivers-license-number"></a>Numéro de permis de conduire aux États-Unis

### <a name="format"></a>Format

Dépend de l’État

### <a name="pattern"></a>Modèle

dépend de l’État (par exemple, New York) :

- neuf chiffres mis en forme, comme ccc, correspondent.
- neuf chiffres tels que ccccccccc ne correspondent pas.

### <a name="keywords"></a>Mots clés

#### <a name="keyword_us_drivers_license_abbreviations"></a>Keyword\_us\_drivers\_license\_abbreviations

```
DL
DLS
CDL
CDLS
ID
IDs
DL#
DLS#
CDL#
CDLS#
ID#
IDs#
ID number
ID numbers
LIC
LIC#
```

#### <a name="keyword_us_drivers_license"></a>Keyword\_us\_drivers\_license

```
DriverLic
DriverLics
DriverLicense
DriverLicenses
Driver Lic
Driver Lics
Driver License
Driver Licenses
DriversLic
DriversLics
DriversLicense
DriversLicenses
Drivers Lic
Drivers Lics
Drivers License
Drivers Licenses
Driver'Lic
Driver'Lics
Driver'License
Driver'Licenses
Driver' Lic
Driver' Lics
Driver' License
Driver' Licenses
Driver'sLic
Driver'sLics
Driver'sLicense
Driver'sLicenses
Driver's Lic
Driver's Lics
Driver's License
Driver's Licenses
identification number
identification numbers
identification #
ID card
ID cards
identification card
identification cards
DriverLic#
DriverLics#
DriverLicense#
DriverLicenses#
Driver Lic#
Driver Lics#
Driver License#
Driver Licenses#
DriversLic#
DriversLics#
DriversLicense#
DriversLicenses#
Drivers Lic#
Drivers Lics#
Drivers License#
Drivers Licenses#
Driver'Lic#
Driver'Lics#
Driver'License#
Driver'Licenses#
Driver' Lic#
Driver' Lics#
Driver' License#
Driver' Licenses#
Driver'sLic#
Driver'sLics#
Driver'sLicense#
Driver'sLicenses#
Driver's Lic#
Driver's Lics#
Driver's License#
Driver's Licenses#
ID card#
ID cards#
identification card#
identification cards#
```

#### <a name="keyword_state_name_drivers_license_name"></a>Keyword\_[state\_name]\_drivers\_license\_name

- Abréviation d’État (par exemple, &quot;NY&quot;)
- Nom de l’État (par exemple, &quot;New York&quot;)

## <a name="us-individual-taxpayer-identification-number-itin"></a>Numéro d'identification de contribuable pour les États-Unis (ITIN)

### <a name="format"></a>Format

neuf chiffres commençant par un &quot;9&quot; et contenant un &quot;7&quot; ou &quot;8&quot; comme quatrième chiffre, éventuellement mis en forme avec des espaces ou des tirets

### <a name="pattern"></a>Modèle

mis en forme :

- le chiffre &quot;9&quot;
- deux chiffres
- espace ou tiret
- &quot;7&quot; ou &quot;8&quot;
- un chiffre
- espace ou tiret
- quatre chiffres

non mis en forme :

- le chiffre &quot;9&quot;
- deux chiffres
- &quot;7&quot; ou &quot;8&quot;
- cinq chiffres

### <a name="keywords"></a>Mots clés

#### <a name="keyword_itin"></a>Keyword\_itin

```
taxpayer
tax ID
tax identification
itin
i.t.i.n.
ssn
tin
social security
tax payer
itins
tax ID
individual taxpayer
```

## <a name="us-social-security-number-ssn"></a>Numéro de sécurité sociale (SSN) aux États-Unis

### <a name="format"></a>Format

neuf chiffres, qui peuvent être dans un modèle mis en forme ou non mis en forme

>[!Note]
> S’il est émis avant le mi-2011, un SSN a une mise en forme forte dans laquelle certaines parties du nombre doivent être comprises dans certaines plages pour être valides (mais il n’y a pas de somme de contrôle).
>

### <a name="pattern"></a>Modèle

quatre fonctions recherchent les numéros de sécurité sociale dans quatre modèles différents :

- Func\_ssn trouve les numéros de sécurité sociale avec une mise en forme forte de 2011 qui est mise en forme avec des tirets ou des espaces (ddd-dd-dddd OU ddd dd dddd)
- Func\_non formaté\_ssn trouve des numéros de sécurité sociale avec une mise en forme forte de pré-2011 qui n’est pas mise en forme avec neuf chiffres consécutifs (ccccccccc)
- Func\_mise en forme\_aléatoire\_ssn trouve les numéros de sécurité sociales postérieurs à 2011 qui sont mis en forme avec des tirets ou des espaces (ddd-dd-dddd OR ddd dd dddd)
- Func\_aléatoire\_non formaté\_ssn trouve des numéros de sécurité sociale de pré-2011 qui ne sont pas mise en forme avec neuf chiffres consécutifs (ccccccccc)

### <a name="keywords"></a>Mots clés

#### <a name="keyword_ssn"></a>Keyword\_ssn

```
SSA Number
social security number
social security #
social security#
social security no
Social Security#
Soc Sec
SSN
SSNS
SSN#
SS#
SSID
```

## <a name="us--uk-passport-number"></a>États-Unis/Royaume-Uni Numéro de passeport

L’ entité de type d’informations sensibles est disponible dans le type d’informations sensibles Numéro de passeport britannique et elle est disponible en tant qu’entité de type d’informations sensibles autonome.

### <a name="format"></a>Format

neuf chiffres

### <a name="pattern"></a>Modèle

neuf chiffres consécutifs

### <a name="keywords"></a>Mots clés

#### <a name="keyword_passport"></a>Keyword\_passport

```
Passport Number
Passport No
Passport #
Passport#
PassportID
Passport no
passport number
パスポート
パスポート番号
パスポートのNum
パスポート＃
Numéro de passeport
Passeport n °
Passeport Non
Passeport #
Passeport#
PasseportNon
Passeportn °
```
