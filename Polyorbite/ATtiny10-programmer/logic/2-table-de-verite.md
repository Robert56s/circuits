# Programmeur ATtiny10 - table de vérité

## Les entrées

| Signal | Vient de | 0 veut dire | 1 veut dire |
|---|---|---|---|
| SEL | SW301 | emplacement A | emplacement B |
| HV | SW302 | mode normal | mode 12 V (fusible RSTDISBL programmé) |
| RST | programmeur | « je programme » | « laisse la puce tourner » |
| CLK | programmeur | horloge TPI | horloge TPI |
| DATA | programmeur | données TPI | données TPI |

## La table

CLK et DATA ne changent pas de valeur : ils sont seulement **branchés** (ils passent) ou **coupés** (0).

| SEL |  HV | RST | VCC_A | RESET_A | CLK, DATA A | VCC_B | RESET_B | CLK, DATA B | Ce qui se passe |
|---|---|---|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 5 V | 0 V | branchés | 0 V | 0 V | coupés | A : programmation normale |
| 0 | 0 | 1 | 5 V | 5 V | branchés | 0 V | 0 V | coupés | A : la puce tourne |
| 0 | 1 | 0 | 5 V | 12 V | branchés | 0 V | 0 V | coupés | A : programmation 12 V |
| 0 | 1 | 1 | 5 V | 5 V | branchés | 0 V | 0 V | coupés | A : la puce tourne |
| 1 | 0 | 0 | 0 V | 0 V | coupés | 5 V | 0 V | branchés | B : programmation normale |
| 1 | 0 | 1 | 0 V | 0 V | coupés | 5 V | 5 V | branchés | B : la puce tourne |
| 1 | 1 | 0 | 0 V | 0 V | coupés | 5 V | 12 V | branchés | B : programmation 12 V |
| 1 | 1 | 1 | 0 V | 0 V | coupés | 5 V | 5 V | branchés | B : la puce tourne |

## Les équations (ce que fait le circuit logique)

| Signal | Équation | En mots |
|---|---|---|
| EN_A | NOT SEL | A est choisi |
| EN_B | SEL | B est choisi |
| VPROG | MUX(HV : 0 V, 12 V) | tension pour programmer |
| VRESET | MUX(RST : VPROG, 5 V) | tension du RESET du côté choisi |
| VCC_A | MUX(EN_A : 0 V, 5 V) | alimentation de A |
| RESET_A | MUX(EN_A : 0 V, VRESET) | RESET de A |
| CLK_A | CLK AND EN_A | horloge vers A |
| DATA_A | DATA AND EN_A | données vers A |

Pour B, c'est pareil avec EN_B.

Lecture d'un MUX : `MUX(S : X, Y)` donne X si S = 0, et Y si S = 1.
