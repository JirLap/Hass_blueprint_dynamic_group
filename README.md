# Home Assistant: Dynamic Group

[English](README.md) | [Čeština](README_cs.md)

This blueprint creates a group using `group.set`. It extends the original [Create Group by mwolter805](https://gist.github.com/mwolter805/965fb5c3b4d6bf8e9aeba8acd82fd224) with area filtering.

Area filtering extension and documentation: **JirLap**. Original blueprint author: **mwolter805**.

## Files

- `create_group.yaml`: the modified blueprint.
- `create_group_backup.yaml`: an exact copy of the original blueprint before modification.
- `README_cs.md`: Czech documentation.

## Input fields (in editor order)

### 1. Name to assign to the group — `group_name`

The display name of the group, for example `Group: Upstairs Lights`.

### 2. object_id to assign to the group — `group_object_id`

The group ID without the domain, for example `upstairs_lights`. The resulting entity is `group.upstairs_lights`.

### 3. Dynamic Domain List — `search_domain`

A list of domains for dynamic selection, such as `[light]` or `[light, switch]`. Without a domain restriction, entities from any domain may be selected when an area, include, or exclude filter is used. If all dynamic filters are empty, only statically added entities are included.

### 4. Dynamic Area List — `filtr_areas`

Select one or more areas. Dynamic selection is restricted to entities from any selected area using `area_entities()`. Areas combine with domain and include/exclude filters. An empty area selection preserves the original behavior.

The editor expects a direct list of area IDs:

```yaml
filtr_areas:
  - living_room
  - kitchen
```

The legacy format `filtr_areas: {area_id: living_room}`, including an object whose `area_id` contains a list, is supported when the automation runs. Convert it to the direct list above to display the selected areas correctly in the editor.

Area membership is evaluated each time the automation runs. Run it again after changing area assignments. Statically added entities may belong to areas outside the selection.

### 5. Dynamic Include List — `search_includes`

A list of search patterns matched against an entity's `object_id` (the part after the dot). When provided, only entities matching at least one pattern remain. An empty list adds no restriction.

Patterns are regular expressions, not file wildcards. For example, `^living_room_` matches the start of a name; `bedroom` matches that text anywhere in the name.

### 6. Dynamic Exclude List — `search_excludes`

A list of regular expressions matched against `object_id`. Entities matching any pattern are removed from dynamic selection. For example, to exclude prefixes:

```yaml
search_excludes:
  - ^christmas
  - ^heating
  - ^fan
```

In a regular expression, `*` repeats the preceding character; `.*` matches any continuation. For a prefix search here, `^christmas` is sufficient.

### 7. Static Entities To Add — `entities_to_add`

Specific entities added after dynamic filtering, even if they do not match the domain, area, or include/exclude filters. Use this field alone to manage a static group. When selecting devices, review the individual entities.

### 8. Static Entities To Remove — `entities_to_remove`

Specific entities removed as the final step, including any static additions. Removal takes precedence over addition. The final list is deduplicated and sorted.

## Processing order

1. Select entities by domain. If domains are empty but another dynamic filter is used, start with all entities that have states.
2. Restrict selection to the chosen areas.
3. Keep entities matching include patterns.
4. Remove entities matching exclude patterns.
5. Apply static additions, then static removals.

The automation runs at Home Assistant startup and when groups are reloaded. You can also run it manually to refresh the group. Check membership in the resulting group's `entity_id` attribute.

## Deploying to an existing installation

1. Replace `/config/blueprints/automation/JirLap/create_group/create_group.yaml` with this repository's `create_group.yaml`. This preserves the blueprint path used by existing automations.
2. Reload automations and manually run the relevant automation.
3. Verify group membership. If necessary, select areas again in the editor to convert the legacy object format to a list.

For a new installation, you can save the file as `/config/blueprints/automation/JirLap/Hass_blueprint_dynamic_group/create_group.yaml` and use the matching path in `use_blueprint.path`.

## Example inputs

```yaml
group_name: "Group: Upstairs Lights"
group_object_id: upstairs_lights
search_domain:
  - light
filtr_areas:
  - living_room
```

Without `search_domain: [light]`, selection is not restricted to lights. Replace example area IDs with IDs from your installation.

## Validation and testing

YAML syntax was checked, including `!input` tags. The backup was compared byte for byte with the original. A total of 116 Jinja template checks passed using mock data, covering legacy inputs, multiple areas, domains, include/exclude filters, static entities, and unchanged behavior with empty areas. These were not integration tests in a running Home Assistant instance.

After deployment, test a single area, multiple areas, an empty area selection, and static additions/removals. Run the automation again after changing area assignments.

To revert, restore `create_group_backup.yaml` to the active blueprint path and reload automations. The backup itself does not contain area filtering.

## Credits

Original Create Group: [mwolter805](https://gist.github.com/mwolter805/965fb5c3b4d6bf8e9aeba8acd82fd224). Area filtering extension and documentation: **JirLap**.

The original blueprint also credits [Frigate Notification Blueprint](https://gist.github.com/hunterjm/23c1588a9f2b8b9c2a62ffc364e17f8c), [Taras' post about group creation](https://community.home-assistant.io/t/template-loop-detected-while-processing-event/243057/22?u=mwolter), and assistance from [petro](https://community.home-assistant.io/u/petro).
