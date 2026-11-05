# XML schema

This document describes the structure of the XML template files in this repository. They are
the serialized form (`ArchiveVer="4"`) of Prime World Editor's game templates, and describe
the script objects, structs, enums and flags of each supported game.

All files are UTF-8 with an `<?xml version="1.0" encoding="UTF-8"?>` header and four-space
indentation.

## Contents

- [Repository layout](#repository-layout)
- [Common conventions](#common-conventions)
- [GameList.xml](#gamelistxml)
- [PropertyMap.xml](#propertymapxml)
- [Game.xml](#gamexml)
- [Script objects (`Script/`, `Misc/`)](#script-objects-script-misc)
- [Property templates (`Structs/`, `Enums/`, `Flags/`)](#property-templates-structs-enums-flags)
- [Properties](#properties)
- [Default values](#default-values)
- [Property paths](#property-paths)

## Repository layout

```
GameList.xml          List of games and where their Game.xml lives
PropertyMap.xml       Names for hashed property IDs (shared by all hashed games)
<GameDir>/
    Game.xml          Index of everything else in this folder
    Script/           One ScriptObject per script object type (Actor, Relay, ...)
    Misc/             ScriptObject files that are not script instances (tweaks, scan data)
    Structs/          PropertyTemplate files with a Struct archetype
    Enums/            PropertyTemplate files with an Enum or Choice archetype
    Flags/            PropertyTemplate files with a Flags archetype
```

| Game folder | `Game` ID          | Game                                               |
|-------------|--------------------|----------------------------------------------------|
| `MP1`       | `Prime`            | Metroid Prime                                      |
| `MP2Demo`   | `EchoesDemo`       | Metroid Prime 2: Echoes Bonus Disc                 |
| `MP2`       | `Echoes`           | Metroid Prime 2: Echoes                            |
| `MP3Proto`  | `CorruptionProto`  | Metroid Prime 3: Corruption E3 2006 Prototype      |
| `MP3`       | `Corruption`       | Metroid Prime 3: Corruption                        |
| `DKCR`      | `DKCReturns`       | Donkey Kong Country Returns                        |
| `MP1R`      | `PrimeRemastered`  | Metroid Prime Remastered                           |

Not every folder has every subfolder (for example `MP1` has no `Flags/`, `MP1R` has no `Misc/`).

## Common conventions

### Root attributes

Every file's root element carries:

| Attribute    | Meaning                                                                      |
|--------------|------------------------------------------------------------------------------|
| `ArchiveVer` | Serialization version. Always `4`.                                           |
| `Game`       | The game ID from the table above. Not present on `GameList` or `PropertyMap`. |

Known inconsistency: seven files in `MP1R/Structs/` use `Game="PrimeRemaster"` instead of
`PrimeRemastered`.

### Maps

Dictionaries are written as a list of `Element` children, each with a `Key` and a `Value`:

```xml
<SomeMap>
    <Element>
        <Key>...</Key>
        <Value .../>
    </Element>
</SomeMap>
```

### Lists

Lists are written as repeated `Element` children. Simple values go in the element's text
(`<Element>CMDL</Element>`), compound values in attributes or child elements.

### IDs

Property IDs are hexadecimal strings with a `0x` prefix and no leading zeros
(`0x0`, `0xFC966DC`). Their meaning depends on the game:

- **Metroid Prime (`MP1`)**: the property's index inside its parent struct (`0x0`, `0x1`, ...).
- **Echoes and later**: a 32-bit hash of the property name. The name itself is looked up in
  `PropertyMap.xml` unless the template provides one.
- **Metroid Prime Remastered (`MP1R`)**: hashes written in lower case, with an inline `Name`
  attribute.

Four-character codes (script object types, messages, states, asset types) are written as plain
text, for example `ACTR` or `CMDL`. Script object types are the exception in two games: `MP1`
writes them as hex numbers (`0x0`, `0x2`, ...) and `MP1R` as lower-case 32-bit hashes
(`0xb6200be6`).

## GameList.xml

Lists the supported games.

```xml
<GameList ArchiveVer="4">
    <Game ID="Prime">
        <Name>Metroid Prime</Name>
        <GameTemplate>MP1/Game.xml</GameTemplate>
    </Game>
</GameList>
```

| Element / attribute | Meaning                                       |
|---------------------|-----------------------------------------------|
| `Game@ID`           | Game ID, used in every other file's `Game`.   |
| `Name`              | Display name.                                 |
| `GameTemplate`      | Path to the game's `Game.xml`, from the root. |

## PropertyMap.xml

Maps hashed property IDs to names. Used by every game that hashes property names.

```xml
<PropertyMap ArchiveVer="4">
    <PropertyMap>
        <Element>
            <Key ID="0x2444C6" Type="WeaponVulnerability"/>
            <Value Name="Light"/>
        </Element>
    </PropertyMap>
</PropertyMap>
```

| Element / attribute | Meaning                                                                    |
|---------------------|----------------------------------------------------------------------------|
| `Key@ID`            | The property ID hash.                                                      |
| `Key@Type`          | The property's type name: a lower-case primitive name (`float`, `bool`, `int`, `asset`, `Color`, `Vector`, `spline`, `sound`, `string`, `enum`, `choice`, `characteranimation`, ...) or, for structs, enums and flags, the archetype name. |
| `Value@Name`        | The property name.                                                         |

The same ID can appear more than once with different types.

## Game.xml

Index of one game's templates. Each section is a [map](#maps).

```xml
<Game ArchiveVer="4" Game="Echoes">
    <ScriptObjects>...</ScriptObjects>
    <PropertyArchetypes>...</PropertyArchetypes>
    <MiscTemplates>...</MiscTemplates>
    <States>...</States>
    <Messages>...</Messages>
</Game>
```

| Section              | Key                                    | Value                                     |
|----------------------|----------------------------------------|-------------------------------------------|
| `ScriptObjects`      | Script object type (four-character code, or a number in `MP1`/`MP1R`) | `<Value Path="Script/Actor.xml"/>` |
| `PropertyArchetypes` | Archetype name, as used by `Archetype` attributes | `<Value Path="Structs/HealthInfo.xml"/>` |
| `MiscTemplates`      | Template name (`TweakBall`, `ScannableObjectInfo`, ...) | `<Value Path="Misc/TweakBall.xml"/>` |
| `States`             | State ID (four-character code)         | `<Value>Active</Value>` (display name)    |
| `Messages`           | Message ID (four-character code)       | `<Value>Activate</Value>` (display name)  |

`Path` values are relative to the folder holding `Game.xml`.

## Script objects (`Script/`, `Misc/`)

One file per script object type. The root is `ScriptObject`:

```xml
<ScriptObject ArchiveVer="4" Game="Echoes">
    <Properties Type="Struct">
        <Name>Actor</Name>
        <SubProperties>...</SubProperties>
    </Properties>
    <ExpectedPropertyCount>24+</ExpectedPropertyCount>
    <EditorProperties>...</EditorProperties>
    <Assets>...</Assets>
    <Attachments>...</Attachments>
    <Modules>...</Modules>
    <RotationType>RotationDisabled</RotationType>
    <ScaleType>ScaleVolume</ScaleType>
    <PreviewScale>0.5</PreviewScale>
    <VolumeShape>BoxShape</VolumeShape>
    <VolumeConditionProperty>0xD5869B0B</VolumeConditionProperty>
    <VolumeConditions>...</VolumeConditions>
</ScriptObject>
```

Only `Properties` is required; the other elements appear when needed, in the order shown.

| Element                   | Meaning |
|---------------------------|---------|
| `Properties`              | The object's root property. Always `Type="Struct"`, with a `Name` (the object name) and `SubProperties`. See [Properties](#properties). May carry `Incomplete="true"` and `<Atomic>true</Atomic>`. |
| `ExpectedPropertyCount`   | How many properties the object has in the game data. Mainly used for `MP1`, where properties are not hashed. Formats: `N` (exact), `N-M` (range), `N+` (at least N). |
| `EditorProperties`        | Which properties the editor uses for the object's name, transform and state. See below. |
| `Assets`                  | Assets the editor uses to draw the object. See below. |
| `Attachments`             | Other properties attached to the object's skeleton. See below. |
| `Modules`                 | List of code modules (`.rel`/`.rso` files) the object needs, as `<Element>WallCrawler.rel</Element>`. |
| `RotationType`            | `RotationDisabled` if the object can't be rotated in the editor. Rotation is enabled when absent. |
| `ScaleType`               | `ScaleDisabled` (no scaling) or `ScaleVolume` (scaling changes the volume only). Normal scaling when absent. |
| `PreviewScale`            | Scale applied to the preview model. |
| `VolumeShape`             | Shape of the object's volume: `BoxShape`, `AxisAlignedBoxShape`, `EllipsoidShape`, `CylinderShape` or `ConditionalShape`. |
| `VolumeConditionProperty` | With `ConditionalShape`: the [path](#property-paths) of the property that selects the shape. |
| `VolumeConditions`        | With `ConditionalShape`: list of `Element`s with `Value` (property value), `Shape` and an optional `Scale`. |

### `EditorProperties`

Each child holds a [property path](#property-paths):

| Element                    | Property it points to                     |
|----------------------------|-------------------------------------------|
| `NameProperty`             | Instance name (string)                    |
| `PositionProperty`         | Position (vector)                         |
| `RotationProperty`         | Rotation (vector)                         |
| `ScaleProperty`            | Scale (vector)                            |
| `ActiveProperty`           | Active flag (bool)                        |
| `LightParametersProperty`  | Lighting parameters struct                |

```xml
<EditorProperties>
    <NameProperty>0x255A4580:0x494E414D</NameProperty>
    <PositionProperty>0x255A4580:0x5846524D:0x00</PositionProperty>
    <ActiveProperty>0x255A4580:0x41435456</ActiveProperty>
</EditorProperties>
```

### `Assets`

List of `Element`s:

| Attribute             | Meaning |
|-----------------------|---------|
| `Type`                | `Model`, `AnimParams`, `Billboard` or `Collision`. |
| `Source`              | `Property` (read the asset from a property) or `File` (a fixed file). |
| `Location`            | With `Property`: a [property path](#property-paths). With `File`: a path such as `script/common/AIWaypoint.CMDL`. |
| `ForceCharacterIndex` | Optional. With `AnimParams`: use this character of the animation set. |

The editor uses the first entry that resolves to a valid asset.

### `Attachments`

List of `Element`s:

| Attribute        | Meaning |
|------------------|---------|
| `AttachProperty` | [Path](#property-paths) of the property holding the attached model or animation. |
| `LocatorName`    | Name of the skeleton locator to attach to. |
| `AttachType`     | `Attach` or `Follow`. |

## Property templates (`Structs/`, `Enums/`, `Flags/`)

Reusable property types, referenced from other properties by their `Archetype` attribute. The
root is `PropertyTemplate` with one `PropertyArchetype`:

```xml
<PropertyTemplate ArchiveVer="4" Game="Echoes">
    <PropertyArchetype Type="Struct">
        <Name>HealthInfo</Name>
        <SubProperties>...</SubProperties>
    </PropertyArchetype>
    <ExpectedPropertyCount>2</ExpectedPropertyCount>
</PropertyTemplate>
```

`PropertyArchetype` is a [property](#properties) without an `ID`. Its `Type` is:

| Type     | Folder     | Content |
|----------|------------|---------|
| `Struct` | `Structs/` | `SubProperties`. May have `<Atomic>true</Atomic>` and `Incomplete="true"`. |
| `Enum`   | `Enums/`   | `Values`, where each ID is a hash. |
| `Choice` | `Enums/`   | `Values`, where each ID is an index (`0x0`, `0x1`, ...). |
| `Flags`  | `Flags/`   | `Flags` and an optional `DefaultValue`. |

`ExpectedPropertyCount` works as in script objects.

`<OverrideTypeName>true</OverrideTypeName>` on an `Enum` archetype makes properties of that
type use the archetype name as their type name (for example `PlayerItem`) instead of the generic
`enum`, both in `PropertyMap.xml` keys and in name hashing. Only `PlayerItem` uses it.

## Properties

A property is an `Element` inside a `SubProperties` list (or the root `Properties` /
`PropertyArchetype`):

```xml
<Element Type="Float" ID="0x75DBB375">
    <Name>Mass</Name>
    <Description>...</Description>
    <CookPreference>OnlyIfModified</CookPreference>
    <DefaultValue>1.0</DefaultValue>
</Element>
```

### Attributes

| Attribute   | Meaning |
|-------------|---------|
| `Type`      | Property type. See the table below. |
| `ID`        | Property ID. See [IDs](#ids). |
| `Archetype` | Name of a property template (a `PropertyArchetypes` key in `Game.xml`). The property starts from that template, and any children here override it. |
| `Name`      | Property name as an attribute. Only used in `MP1R`; other games use the `Name` child element. |

### Child elements

All are optional.

| Element          | Meaning |
|------------------|---------|
| `Name`           | Property name. Usually omitted for hashed IDs that are named in `PropertyMap.xml`. |
| `Description`    | Free text for the editor's tooltip. |
| `CookPreference` | When to write the property to game files: `Default`, `Always`, `Never` or `OnlyIfModified`. |
| `DefaultValue`   | See [Default values](#default-values). |
| `Suffix`         | Unit shown after the value in the editor, for example `%`. |
| `LoadCondition`  | Expression controlling whether the property is read, for example `propCount &gt; 14`. Used for `MP1` objects whose property count changed between versions. |
| `SubProperties`  | `Struct` only: list of child properties. When combined with `Archetype`, lists only the children that are overridden. |
| `Values`         | `Enum`/`Choice` only: list of `<Element Name="..." ID="0x..."/>`. |
| `Flags`          | `Flags` only: list of `<Element Name="..." Mask="0x..."/>`. |
| `TypeFilter`     | `Asset` only: list of allowed asset types, as `<Element>CMDL</Element>`. |
| `ItemArchetype`  | `Array` only: the item type. A property with `Type` and optionally `ID`, `Archetype`, `Name` and `DefaultValue`. It can contain another `ItemArchetype` for nested arrays. |
| `Atomic`         | `Struct` only: `true` if the struct is written as one block with no per-property IDs. |

### Types

| Type           | Notes |
|----------------|-------|
| `Bool`         | |
| `Short`        | 16-bit integer. |
| `Int`          | 32-bit integer. |
| `Float`        | |
| `String`       | |
| `PooledString` | String stored in a string pool (DKCR, MP1R). |
| `Vector`       | 3D vector. |
| `Color`        | RGBA color. |
| `Enum`         | Value from a list of hashed IDs. Uses `Values` or an `Enum` archetype. |
| `Choice`       | Value from a list of indexes. Uses `Values` or a `Choice` archetype. |
| `Flags`        | Bit field. Uses `Flags` or a `Flags` archetype. |
| `Asset`        | Asset ID. Uses `TypeFilter`. |
| `Sound`        | Sound ID. |
| `Animation`    | Animation ID. |
| `AnimationSet` | Character/animation set reference. |
| `Spline`       | Spline curve. No default value. |
| `Guid`         | 128-bit ID. |
| `Struct`       | Group of properties. Uses `SubProperties` and/or `Archetype`. |
| `Array`        | Variable-length list. Uses `ItemArchetype`. |

## Default values

The format of `DefaultValue` depends on the property type:

| Type                             | Format | Example |
|----------------------------------|--------|---------|
| `Bool`                           | `true` / `false` | `<DefaultValue>false</DefaultValue>` |
| `Short`, `Int`, `Sound`, `Animation` | Decimal integer (`-1` for none) | `<DefaultValue>-1</DefaultValue>` |
| `Float`                          | Decimal number | `<DefaultValue>1.0</DefaultValue>` |
| `String`                         | Text | `<DefaultValue>Hello</DefaultValue>` |
| `Choice`                         | Decimal index | `<DefaultValue>3</DefaultValue>` |
| `Enum`                           | Hex value ID | `<DefaultValue>0xD086634A</DefaultValue>` |
| `Flags`                          | Decimal mask | `<DefaultValue>20</DefaultValue>` |
| `Vector`                         | `X`, `Y`, `Z` children | see below |
| `Color`                          | `R`, `G`, `B` and optional `A` children, from 0.0 to 1.0 | see below |

```xml
<DefaultValue>
    <X>0.0</X>
    <Y>0.0</Y>
    <Z>0.0</Z>
</DefaultValue>

<DefaultValue>
    <R>1.0</R>
    <G>1.0</G>
    <B>1.0</B>
    <A>1.0</A>
</DefaultValue>
```

`Struct`, `Array`, `Asset`, `AnimationSet`, `Spline`, `Guid` and `PooledString` properties have
no `DefaultValue`.

## Property paths

Elements and attributes that point to a property (`EditorProperties`, `Assets@Location`,
`Attachments@AttachProperty`, `VolumeConditionProperty`) use a colon-separated list of property
IDs, starting from the object's root struct:

```
0x7E397FED:0xC0BA9E18     ActorParameters (0x7E397FED) -> property 0xC0BA9E18 inside it
0x0C:0x02                 MP1: property 12 -> its property 2
```
