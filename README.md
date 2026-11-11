# Open FM Studio

Gebouwbeheer op het BIM-model, als uitbreiding op [Open BIM Platform](https://github.com/OpenAEC-Foundation/open-bim-platform)
in de OpenAEC SuperCloud.

Na de oplevering werkt de gebouwbeheerder met het model van het gebouw: ruimtes, installaties en onderdelen opzoeken in de
viewer, per onderdeel het onderhoud, de meldingen en de documenten zien, en dat bijhouden gedurende de hele levensduur van het
gebouw.

**Status:** opgezet, nog geen code. Het doel en de aanpak staan in
[FM-toepassing: gebouwbeheer op het model](https://github.com/OpenAEC-Foundation/open-bim-platform/issues/10).

## Wat het wordt

- **Beheerobjecten** (gebouw, ruimte, installatie, onderdeel), elk gekoppeld aan een element in het model: project, model en
  GlobalId.
- **Overnemen uit het model**: ruimtes, installaties en hun eigenschappen, zoals merk, type, serienummer, garantie en
  onderhoudsinterval. Ze komen uit het IFC, of uit COBie als de opdrachtgever dat vraagt.
- **Onderhoud en meldingen**: een meerjarenonderhoudsplan per onderdeel, conditiemetingen (bijvoorbeeld volgens NEN 2767), en
  storingsmeldingen op een element uit de viewer of vanaf een tablet ter plekke.
- **Koppeling met het ERP** van de klantomgeving: onderdelen als assets, en onderhoud en kosten daar, niet in een tweede
  administratie.
- **Een paneel in de viewer** met de beheergegevens van het geselecteerde element, en kleuring op staat of op achterstallig
  onderhoud.

## Hoe het op Open BIM Platform aansluit

Open FM Studio is de eerste toepassing op het
[uitbreidingssysteem van Open BIM Platform](https://github.com/OpenAEC-Foundation/open-bim-platform/issues/9). Het gebruikt
alleen wat het platform openbaar aanbiedt, geen interne functies:

| Laag van het platform | Wat Open FM Studio ermee doet |
|---|---|
| Openbare API (projecten, modellen en versies, elementen, eigenschappen, ruimtelijke boom, issues) | het model doorzoeken en beheerobjecten overnemen |
| Gebeurtenissen (nieuwe versie omgezet, versie gekoppeld, issue gewijzigd) | bijwerken als er een nieuwe versie van het model is |
| Eigen gegevens aan elementen, die het platform meeneemt naar een nieuwe versie | beheergegevens blijven bij het goede element, ook bij deels nieuwe GlobalIds |
| Uitbreidingen in de viewer | het beheerpaneel en de kleuring |

Die lagen bestaan nog niet; ze worden samen met deze toepassing ontworpen. Tot die tijd bouwt Open FM Studio niet op interne
methoden van `openaec_bim`, zodat het later niet herschreven hoeft te worden.

## Open vragen

- **Gebruikers.** Zijn dat de beheerder van de eigenaar, een onderhoudsbedrijf, of een huurder die een melding doet? Dat bepaalt
  de rechten, en of er een eenvoudige meldpagina zonder viewer nodig is.
- **Gegevensmodel.** Eigen DocTypes, of de assets en het onderhoud van het ERP zelf? De voorkeur is zo veel mogelijk het ERP, met
  in deze app alleen de koppeling naar het model.
- **As-built eisen.** Welke eigenschappen moet een as-built model hebben om bruikbaar te zijn? Een IDS voor de oplevering kan dat
  vooraf controleren.
- **Beheerproject.** Een gebouw in beheer heeft vaak geen actief bouwproject meer. Hoe ziet zo'n project eruit in de opslag?
- **In het veld.** Werken op een tablet, ook met slecht bereik.

## Licentie

[LGPL-3.0-or-later](COPYING.LESSER), net als Open BIM Platform.
