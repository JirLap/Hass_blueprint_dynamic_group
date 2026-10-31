# Home Assistant: Dynamická skupina

Blueprint vytváří skupinu pomocí `group.set`. Rozšiřuje původní Create Group od [mwolter805](https://gist.github.com/mwolter805/965fb5c3b4d6bf8e9aeba8acd82fd224) o filtrování podle oblastí.

Úprava filtrování podle oblastí a její dokumentace: **JirLap**. Původní autor blueprintu: **mwolter805**.

## Soubory

- `create_group.yaml`: upravený blueprint.
- `create_group_backup.yaml`: přesná původní verze před úpravou.

## Funkce

- Volitelný výběr více oblastí (`filtr_areas`, area selector).
- Při každém spuštění načítá členy oblastí přes `area_entities()`.
- Podporuje také starší zápis `filtr_areas: {area_id: obyvak}`.
- Zachovává filtrování domén, include/exclude a statické přidávání/odebírání.
- Prázdný výběr oblastí zachovává původní chování.
- Statické entity se přidávají a odebírají po dynamickém filtrování; lze tak přidat i entitu mimo vybrané oblasti.
- Spouští se při startu Home Assistantu a při reloadu skupin. Změny členství lze aplikovat i ručním spuštěním automatizace.

## Nasazení do existující instalace

1. Nahraďte `/config/blueprints/automation/JirLap/create_group/create_group.yaml` souborem `create_group.yaml` z tohoto repozitáře. Existující cesta automatizace tak zůstane platná.
2. Znovu načtěte automatizace a spusťte příslušnou automatizaci ručně.
3. Ověřte členství výsledné skupiny. V editoru případně znovu vyberte oblasti, aby se starý objektový zápis převedl na seznam.

Pro novou instalaci lze soubor uložit například jako `/config/blueprints/automation/JirLap/Hass_blueprint_dynamic_group/create_group.yaml` a použít odpovídající cestu v `use_blueprint.path`.

## Příklad vstupů

```yaml
group_name: "Skupina: Světla patro"
group_object_id: skupina_svetla_patro
search_domain:
  - light
filtr_areas:
  - obyvak
```

Bez `search_domain: [light]` není výběr omezen pouze na světla.

## Ověření a test

Syntaxe YAML byla ověřena včetně značek `!input`. Záloha byla porovnána po bajtech s originálem. Prošlo 116 kontrol Jinja šablon s modelovými daty, včetně starého vstupu, více oblastí, domén, include/exclude, statických entit a zachování chování při prázdných oblastech. Nejde o integrační test v běžícím Home Assistantu.

Po nasazení ověřte jednu oblast, více oblastí, prázdný výběr a statické přidání/odebrání. Po změně přiřazení entity do oblasti spusťte automatizaci znovu.

Pro návrat obnovte obsah `create_group_backup.yaml` do používané cesty blueprintu a znovu načtěte automatizace. Záloha sama filtr oblastí neobsahuje.
