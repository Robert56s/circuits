NOTE DE DESIGN - Programmeur ATtiny10 (logique de sélection)
=============================================================

Ce que le design doit faire
---------------------------
Le programmeur USBasp parle à un seul ATtiny10 à la fois, par l'interface TPI
(RESET, CLK, DATA). La carte a deux emplacements, A et B. Avec SW301 (SEL), je
choisis l'emplacement : seul celui-là reçoit le 5 V sur VCC et les signaux du
programmeur. L'autre reste à 0 V et débranché, pour qu'on ne parle jamais à une
puce éteinte. Avec SW302 (HV), je choisis le mode de programmation : normal
(RESET à 0 V pour programmer) ou 12 V (RESET à 12 V pour programmer, pour une
puce dont le fusible RSTDISBL est programmé). Quand le programmeur ne programme
pas (RST = 1), le RESET de la puce choisie est à 5 V et elle exécute son
programme.

Programmation normale et programmation 12 V
-------------------------------------------
- Normale : la broche RESET fonctionne comme un reset. Pour entrer en mode
  programmation, le programmeur met RESET à 0 V et le garde à 0 V pendant toute
  la programmation.
- 12 V : si le fusible RSTDISBL est programmé, la broche RESET devient une
  broche normale (PB3). Mettre RESET à 0 V ne fait plus rien. La seule façon
  d'entrer en mode programmation est de mettre 12 V sur RESET, pendant toute la
  programmation (datasheet ATtiny4/5/9/10, section 15.3.1).
- Dans les deux cas, VCC doit être à 5 V.

Décision : quand le 12 V arrive sur RESET
-----------------------------------------
J'ai choisi qu'en mode 12 V, le 12 V n'arrive sur RESET que quand le programmeur
met RST à 0. Il n'est pas branché en continu.

Raisons :
1. Le datasheet demande un ordre : d'abord le 5 V sur VCC, ensuite le 12 V sur
   RESET. Comme le programmeur ne met RST à 0 qu'une fois la puce alimentée,
   l'ordre est respecté automatiquement.
2. Le programmeur garde le contrôle du début et de la fin de la programmation,
   comme en mode normal. Le USBasp et avrdude n'ont pas besoin de savoir que le
   12 V existe : ils mettent juste RST à 0, et la logique traduit ça en 0 V ou
   en 12 V selon SW302.
3. Le 12 V n'est présent que pendant la programmation, et pas quand la puce
   tourne.

C'est différent du design de Félix, où SW302 en position 12 V branche le 12 V
en continu sur RESET, peu importe ce que fait le programmeur.

Équation qui en découle :
  RESET (emplacement choisi) = 5 V   si RST = 1
                             = 0 V   si RST = 0 et HV = 0
                             = 12 V  si RST = 0 et HV = 1
