# Pack-Generator-Data

Public, versioned data for the Pack Generator service.

This repository intentionally contains **data only**. The generator/website code can live in a separate repository and fetch this data at runtime.

## Structure

```text
manifest.json

games/
  heroclix/
    game.json
    sets/
      thunderbolts/
        configs/
          v1.json
        catalogs/
          v1.json
```

The layout is deliberately game-agnostic.

A second game can be added without changing the existing HeroClix files:

```text
games/
  another-game/
    game.json
    sets/
      first-set/
        configs/
          v1.json
        catalogs/
          v1.json
```

Then add that game/set to `manifest.json`.

## manifest.json

The manifest is the entry point used by the generator.

It identifies:

- available games
- game codes
- available sets within each game
- set codes
- the current config version for new openings
- the generator engine version
- the path to that set's current config

Example:

```json
{
  "id": "thunderbolts",
  "code": "TB",
  "current_config_version": 1,
  "engine_version": 1,
  "config_path": "games/heroclix/sets/thunderbolts/configs/v1.json"
}
```

The generator should use `config_path` from the manifest rather than assuming a fixed directory structure.

## Game metadata

`games/<game>/game.json` contains data that applies to the whole game rather than a particular set.

For HeroClix this currently includes the OCTGN game GUID and default deck section.

Keeping that at the game level prevents those values from being repeated in every set config.

## Versioning rule

Published config versions are immutable.

Do **not** edit `v1.json` after V1 share codes have been used publicly.

If the Thunderbolts collation model changes:

1. Copy `configs/v1.json` to `configs/v2.json`.
2. Make the changes in V2.
3. If the catalog changes in a way that affects generation, create `catalogs/v2.json` as well.
4. Update the V2 config's `catalog_url`.
5. Change `current_config_version` in `manifest.json` to `2`.

Old V1 codes continue using V1. New generated pools use V2.

## Catalog

A catalog contains the models the generator may select.

For each model it currently stores:

- model GUID
- name
- collector number
- unit type
- printed rarity
- Prime flag
- OCTGN properties string

The catalog is separate from distribution rules so card data and collation logic can be updated/versioned independently.

## Config

The set config describes how sealed product is collated.

Thunderbolts V1 currently defines:

- 2 bricks per case
- 12 boosters per brick
- 5 game-piece slots per booster
- 8 base boosters at 3 Common / 1 Uncommon / 1 Rare
- 4 base boosters at 2 Common / 2 Uncommon / 1 Rare
- exactly 3 Character SR substitutions per brick
- exactly 1 standard Chase substitution per brick
- Prime as an overlay on Rare or SR
- SR equipment replacing Common slots
- extra Chase variants replacing Common slots
- One-Shot and terrain insert distribution

These values are data, not generator code.

## Public repository

This repository is intended to be public.

That allows the Worker to read the raw files without a GitHub token and lets users inspect the exact configuration behind a share code.

If the GitHub repository is named:

`Pack-Generator-Data`

the raw base URL will be:

```text
https://raw.githubusercontent.com/YOUR-GITHUB-USER/Pack-Generator-Data/main/
```

The generator should point its `DATA_BASE_URL` at that root.

## Adding another set

For another HeroClix set:

```text
games/heroclix/sets/new-set/
  configs/
    v1.json
  catalogs/
    v1.json
```

Then add it to the HeroClix `sets` array in `manifest.json`.

## Adding another game

Create:

```text
games/new-game/game.json
games/new-game/sets/...
```

and add a new game object to `manifest.json`.

No HeroClix-specific assumptions should be required by the data repository itself.
