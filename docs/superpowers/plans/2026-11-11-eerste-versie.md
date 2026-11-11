# Open FM Studio, eerste versie — plan

**Doel:** een eigenaar met een handvol gebouwen beheert die in de klantomgeving. Per gebouw is er een register van
ruimtes, installaties en onderdelen, met meldingen, onderhoud en keuringen, documenten, een meerjarenonderhoudsplan
(MJOP) met conditiemeting, de jaarkosten en meetwaarden per ruimte. Waar een IFC-model is, hangt dat register aan het
model, en het blijft daaraan hangen als er een nieuwe versie van het model komt.

**Ontwerp:** er is nog geen ontwerpdocument. Dit plan legt de afbakening en de volgorde vast. Elk onderdeel krijgt
vóór de bouw een eigen ontwerp en implementatieplan in `docs/superpowers/`. Achtergrond: de FM-toepassing
(OpenAEC-Foundation/open-bim-platform#10) en het uitbreidingssysteem van het platform
(OpenAEC-Foundation/open-bim-platform#9).

**Issues:** mijlpaal "Eerste versie" in deze repo, #1 tot en met #13.

## Uitgangspunten

Deze uitgangspunten volgen uit een marktverkenning van november 2026: onderhoudsbeheer, meldingen, vastgoed- en
ruimtebeheer, sensoren en energie, digital twins op BIM, MJOP en NEN 2767, en de internationale en Nederlandse
standaarden voor beheer.

1. **Het ERP doet wat het al kan.** Assets met afschrijving, de locatieboom, periodiek onderhoud tot drie jaar met
   logboek en certificaat, reparaties met kosten, inkoop, kostenplaatsen en meerjarige budgetten staan in het ERP van de
   klantomgeving. Open FM Studio voegt alleen toe wat daar ontbreekt:
   - een elementenboom per gebouw met classificatie (een asset heeft geen ouder);
   - de koppeling met het model;
   - meldingen per ruimte of onderdeel;
   - conditiemeting;
   - een MJOP met cycli langer dan drie jaar;
   - meetwaarden.
2. **Het werkt ook zonder model.** Nieuwbouw is per jaar maar 1 à 2 procent van de voorraad, en de meeste gebouwen
   hebben geen bruikbaar as-built model. Ruimtes en onderdelen kunnen met de hand worden aangemaakt; een model kan er
   later bij komen.
3. **Het model is het onderscheid.** Pakketten voor onderhoudsbeheer in het middensegment gebruiken geen BIM. De grote
   pakketten lezen een model eenmalig in bij de oplevering, en vrijwel geen enkel product legt uit hoe beheergegevens
   gekoppeld blijven als de GlobalIds in een nieuwe versie veranderen. Dat maken wij de proef op de som (#4).
4. **Open standaarden in plaats van eigen formaten.**
   - IFC: lezen in IFC2x3, IFC4 en IFC4.3, intern werken met de begrippen van IFC4.3.
   - IDS 1.0 voor de eisen bij oplevering, BCF 3.0 voor een melding op een element, en bSDD voor classificaties.
   - NL-SfB als sleutel tussen het model, NEN 2767 en het MJOP.
   - Brick (`ref:IFCReference`) voor meetpunten, en MQTT voor meetwaarden.
   - COBie alleen als export, en pas als iemand erom vraagt.
5. **Sensoren koppelen, geen eigen IoT-platform.** Per gebouw een edge-gateway, en centraal een broker en een
   tijdreeksdatabase als losse diensten. Open FM Studio bewaart alleen welk meetpunt bij welk element of welke ruimte
   hoort.

## Randvoorwaarden (gelden voor elk onderdeel)

- **Geen productnamen.** Nergens in de repo, ook niet in issues en commits, staan namen van andere softwareproducten
  of verwijzingen naar gesprekken.
- **Taal en commits.** Code (namen, docstrings, commentaar) is Engels en documentatie Nederlands. Commits volgen
  `type(scope): Nederlandse omschrijving` en eindigen op de Co-Authored-By-regel.
- **Alleen de openbare lagen van het platform** (OpenAEC-Foundation/open-bim-platform#9): de API, de gebeurtenissen,
  eigen gegevens aan elementen en de uitbreidingen in de viewer. Geen interne methoden van `openaec_bim`.
- **Geen tweede administratie.** Wat in het ERP kan, staat in het ERP.
- **Licenties.**
  - De app is LGPL-3.0-or-later.
  - Geen bouwstenen onder BSL, SSPL of een andere source-available licentie. AGPL alleen als losse, ongewijzigde
    dienst.
  - De inhoud van betaalde normen (NEN 2767-gebrekenlijsten, NEN 2580) leveren we niet mee.
  - NL-SfB halen we tijdens het gebruik op uit bSDD, tot duidelijk is of we de codes mogen meeleveren.
- **IFC-valkuilen.** Pset_Warranty heeft in IFC4.3 geen einddatum meer, dus die berekenen we zelf. ModelReference en
  ModelLabel in Pset_ManufacturerTypeInformation zijn tussen IFC2x3 en IFC4 van betekenis omgewisseld.
- **Testen.**
  - Testmodellen maken we in de tests met IfcOpenShell; geen modellen van klanten.
  - Frappe-tests draaien alleen op een testsite.

## Gegevensmodel (voorstel, vast te leggen in het ontwerp van #1)

| Wat | Waar | Opmerking |
|---|---|---|
| Gebouw, verdieping, ruimte | ERP: locatieboom, met extra velden (elementsleutel, ruimtecode, functie, oppervlakte) | Een gebouw is ook een asset als het financieel wordt geadministreerd |
| Beheerobject (installatie, onderdeel, bouwdeel) | Open FM Studio: eigen DocType met een stabiele sleutel, een ouder, classificatie en een verwijzing naar het element (project, model, GlobalId) | Alleen financieel relevante objecten krijgen ook een asset in het ERP |
| Fabrikant, type, serienummer, tag, garantie, levensduur | Uit het model (property sets), daarna bij het beheerobject of de asset | De mapping staat in #3 |
| Periodiek onderhoud en keuringen tot drie jaar | ERP: onderhoud van assets, met logboek en certificaat | Gaten in #7 |
| Melding | Ticket in het ERP (de keuze valt in #6), met beheerobject of ruimte; met model ook een BCF-standpunt op het element | |
| Reparatie, werkorder, inkoop | ERP | |
| Inspectie, gebrek, conditiescore | Open FM Studio | #10 |
| MJOP-regel (object × activiteit × cyclus × hoeveelheid × prijs) | Open FM Studio; jaartotalen als budget in het ERP | #11 |
| Kosten | ERP: grootboek met een kostenplaats of dimensie per gebouw | #12 |
| Document | Storage, gekoppeld aan gebouw, ruimte, type of beheerobject | #8 |
| Meetpunt | Open FM Studio: verwijzing van topic of reeks naar element of ruimte; de waarden staan in de tijdreeksdatabase | #13 |

## Stappen

| Stap | Issues | Wat de eigenaar daarna kan | Hangt af van |
|---|---|---|---|
| 1. Fundament | #1 | Een gebouw met ruimtes en onderdelen met de hand opzetten, gekoppeld aan het ERP | — |
| 2. Beheer zonder model | #6, #7, #8 | Meldingen via een QR-code, onderhoud en keuringen op een kalender, documenten per ruimte en onderdeel | stap 1 |
| 3. Beheer op het model | #2, #3, #4, #5, #9 | Het as-built model inlezen, een installatie aanklikken en de beheergegevens zien, ook na een nieuwe versie | stap 1; lagen 1–4 van het platform (OpenAEC-Foundation/open-bim-platform#9) |
| 4. MJOP en conditie | #10, #11, #12 | Een conditiemeting volgens NEN 2767, een MJOP over 30 jaar, de jaarkosten per gebouw | stap 1 |
| 5. Sensoren | #13 | Meetwaarden per ruimte, ook in de viewer, met een melding bij een overschrijding | stap 1, #6; stap 3 voor de viewer |

Stap 2 en stap 4 kunnen naast stap 3 lopen; die wacht op de openbare API en de gebeurtenissen van het platform.
Binnen stap 3 is de volgorde #2, #3, dan #4 en #9 naast elkaar; #5 kan na #3.

## Klaar als (de eerste versie)

1. **Zonder model.** Een eigenaar zet een gebouw op met ruimtes, onderdelen, keuringen en documenten. Een gebruiker
   meldt via de QR-code in een ruimte een storing. Die komt als werkopdracht in het ERP terecht, en de melder ziet de
   status.
2. **Met model.** Een beheerder opent het as-built model, klikt op een installatie en ziet de gegevens, het onderhoud,
   de meldingen en de conditie. De viewer kleurt op achterstallig onderhoud.
3. **Een nieuwe modelversie.** Na een verbouwing komt er een nieuwe versie met deels nieuwe GlobalIds. Meldingen,
   onderhoudshistorie, documenten, conditiescores en meetpunten staan daarna nog bij het goede element, of liggen
   klaar om te beoordelen. Er gaat niets ongemerkt verloren.
4. **MJOP.** Een gebouw heeft een MJOP over 30 jaar met cycli en prijzen per onderdeel, dat na een conditiemeting is
   bijgewerkt. De jaartotalen staan als budget in het ERP.
5. **Jaarkosten.** De jaarkosten per gebouw en per m² staan naast de begroting uit het MJOP.
6. **Sensoren.** Een meetwaarde per ruimte is zichtbaar in de viewer, en een overschrijding geeft één melding.

## Bewust niet in de eerste versie

- **Installaties en wettelijke plicht.** Geen foutdetectie op installaties. Ook geen rapportage voor de verplichte
  gebouwautomatisering: die plicht vult het gebouwbeheersysteem van de installateur in. Open FM Studio geeft inzicht en
  bewijs, maar is geen compliance-oplossing.
- **Vastgoed en ruimtegebruik.** Geen verhuur met indexatie en servicekosten, geen ruimtereservering en geen
  leaseboekhouding.
- **Techniek.** Geen eigen offline-app, geen COBie-import (een export pas als er vraag naar is) en geen IFC5.
- **Classificatie.** Geen automatische omzetting naar de nieuwe Nederlands-Belgische classificatie. Twee coderingen
  naast elkaar opslaan kan wel.

## Besluiten voor de eigenaar (niet in dit plan genomen)

- **Gebruikers en rechten:** de beheerder van de eigenaar, het onderhoudsbedrijf en de huurder. Mag melden zonder
  account? (#1, #6)
- **Ticketmodel voor meldingen:** het lichte ticket van het ERP (GPL) of een rijkere helpdesk-app uit het
  Frappe-ecosysteem (AGPL). (#6)
- **Licentietoets:** een LGPL-app die het GPL-ERP uitbreidt, valt bij verspreiding waarschijnlijk onder de GPL. Dit
  laten toetsen, of de koppeling met het model als losse dienst houden. (#1)
- **Inhoud van normen:** mogen NL-SfB-codes en NEN 2767-gebrekenlijsten mee, of halen we ze op en voert de gebruiker
  ze in? (#2, #3, #10)
- **Beheerproject in Storage:** een eigen ruimte per gebouw of per portefeuille, met de as-built modellen? (#2)
- **Gaten in het ERP:** één onderhoudsplan per asset, cycli tot drie jaar, geen leverancier per taak. Vullen we die
  met extra velden of met eigen DocTypes? (#7)
- **Naam van de Frappe-app:** de werktitel is `openaec_fm`. (#1)
