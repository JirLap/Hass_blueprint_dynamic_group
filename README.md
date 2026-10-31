# Home Assistant: Dynamická skupina

Blueprint vytváří skupinu pomocí `group.set`. Rozšiřuje původní Create Group od [mwolter805](https://gist.github.com/mwolter805/965fb5c3b4d6bf8e9aeba8acd82fd224) o filtrování podle oblastí.

Úprava filtrování podle oblastí a její dokumentace: **JirLap**. Původní autor blueprintu: **mwolter805**.

## Soubory

- `create_group.yaml`: upravený blueprint.
- `create_group_backup.yaml`: přesná původní verze před úpravou.

## Vstupní pole (v pořadí editoru)

### 1. Name to assign to the group — `group_name`

Zobrazovaný název skupiny, například `Skupina: Světla patro`.

### 2. object_id to assign to the group — `group_object_id`

Identifikátor skupiny bez domény, například `skupina_svetla_patro`. Výsledná entita bude `group.skupina_svetla_patro`.

### 3. Dynamic Domain List — `search_domain`

Seznam domén pro dynamický výběr, například `[light]` nebo `[light, switch]`. Bez omezení domény mohou být vybrány entity všech domén, pokud je použit filtr oblastí, include nebo exclude. Pokud jsou všechny dynamické filtry prázdné, skupina obsahuje pouze staticky přidané entity.

### 4. Dynamic Area List — `filtr_areas`

Vyberte jednu nebo více oblastí. Dynamický výběr se omezí na entity z kterékoli vybrané oblasti pomocí `area_entities()`. Výběr se kombinuje s doménami a include/exclude. Prázdný výběr oblastí zachovává původní chování.

Editor očekává přímo seznam ID oblastí:

```yaml
filtr_areas:
  - obyvak
  - kuchyne
```

Starší zápis `filtr_areas: {area_id: obyvak}` i objekt s `area_id` obsahujícím seznam fungují při běhu automatizace. Pro správné zobrazení vybraných oblastí v editoru je převeďte na přímý seznam uvedený výše.

Členství oblastí se načítá při každém spuštění automatizace. Po změně přiřazení entit do oblastí spusťte automatizaci znovu. Staticky přidané entity mohou ležet i mimo vybrané oblasti.

### 5. Dynamic Include List — `search_includes`

Seznam hledaných vzorů v `object_id` entity (část za tečkou). Pokud je seznam vyplněn, ponechají se entity odpovídající alespoň jednomu vzoru. Prázdný seznam výběr dále neomezuje.

Používají se regulární výrazy, nikoli souborové zástupné znaky. Například `^obyvak_` hledá začátek názvu; `bedroom` hledá výskyt textu kdekoli v názvu.

### 6. Dynamic Exclude List — `search_excludes`

Seznam regulárních výrazů v `object_id`. Entity odpovídající alespoň jednomu vzoru se odeberou z dynamického výběru. Například pro prefixy použijte:

```yaml
search_excludes:
  - ^vanoce
  - ^topeni
  - ^vetrak
```

V regulárním výrazu `*` opakuje předchozí znak; pro libovolné pokračování se používá `.*`. Pro pouhé hledání prefixu zde stačí `^vanoce`.

### 7. Static Entities To Add — `entities_to_add`

Konkrétní entity přidané po dynamickém filtrování, i když neodpovídají doméně, oblasti nebo include/exclude. Pomocí tohoto pole lze spravovat i čistě statickou skupinu. Při výběru zařízení zkontrolujte jednotlivé entity.

### 8. Static Entities To Remove — `entities_to_remove`

Konkrétní entity odebrané jako poslední krok, včetně staticky přidaných. Odebrání má přednost před přidáním. Výsledný seznam je zbaven duplicit a seřazen.

## Pořadí zpracování

1. Výběr podle domén; pokud domény nejsou vyplněny, vychází se ze všech stavových entit při použití některého dalšího dynamického filtru.
2. Omezení na vybrané oblasti.
3. Ponechání entit odpovídajících include.
4. Odebrání entit odpovídajících exclude.
5. Statické přidání a následné statické odebrání.

Automatizace se spouští při startu Home Assistantu a při reloadu skupin. Skupinu lze aktualizovat také ručním spuštěním automatizace. Členy ověřte v atributu `entity_id` výsledné skupiny.

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

## Původ a poděkování

Původní Create Group: [mwolter805](https://gist.github.com/mwolter805/965fb5c3b4d6bf8e9aeba8acd82fd224). Rozšíření filtru oblastí a dokumentace: **JirLap**.

Původní blueprint dále uvádí [Frigate Notification Blueprint](https://gist.github.com/hunterjm/23c1588a9f2b8b9c2a62ffc364e17f8c), [Tarasův příspěvek o vytváření skupin](https://community.home-assistant.io/t/template-loop-detected-while-processing-event/243057/22?u=mwolter) a pomoc uživatele [petro](https://community.home-assistant.io/u/petro).
