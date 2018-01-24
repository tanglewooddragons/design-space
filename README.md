# Tanglewood

## Notes:

- `((2/pi) * arctan (x/100))` for luck calculation

## Todos:

- describe hieroglyphs (per-dragon modifiers)
- describe more aspects
	- Time
	- Sand
	- Fate
	- God

## Dragon aspects:
### Basic dragon aspects:
#### Tier 0 (Progenitors):
- Earth
- Water
- Fire
- Air
- Order
- Chaos

Those are further combined.
#### Tier 1:
- Ice = Water + Order
- Crystal = Earth + Order
- Energy = Fire + Order
- Light = Air + Fire
- Void = Air + Chaos
- Life = Water + Earth

#### Tier 2:
- Darkness = Light + Void
- Death = Life + Void
- Metal = Crystal + Ice
- Tempest = Air + Energy
- Beast = Energy + Life
- Stone = Earth + Ice

#### Tier 3:
- Space = Air + Void
- Spirit = Life + Death
- Wisdom = Light + Darkness
- Envy = Beast + Crystal
- Sky = Water + Tempest
- Love = Life + Tempest

#### Tier 4:
- Soul = Spirit + Wisdom
- Universe = Space + Order
- Greed = Envy + Beast
- Necromancy = Spirit + Death

#### Tier 5:
- Golem = Soul + Stone
- Clockwork = Metal + Soul
- Gold = Greed + Metal
- Star = Universe + Light

## Breeding:
Tier 0 dragons (Progenitors) can be acquired by buying dragon eggs.

Dragons of further tiers can be acquired by breeding.

To breed a dragon of a particular aspect, the player needs to own two dragons of opposite genders which are of aspects making up the desired aspect as well as at least level 20. For instance, to breed a Light dragon, the player must either own a male Fire dragon and a female Air dragon or a female Fire dragon and a male Air dragon.

There is a chance for the breeding to succeed and breed a dragon of the particular aspect or not and breed a dragon with the aspect of one of its parents. The chance for successful breeding is `1/tier`. That chance can be increased using hearts.

Dragons can breed once a week.

## Combat model notes:

Each basic aspect is associated with a stat and gets a `+1` bonus when leveling that stat, respectively:
- `CON` (Constitution) - fixed mutliplier to the dragon's hp
- `INT` (Intelligence) - fixed multiplier to the strength of magic attack
- `STR` (Strength) - fixed multiplier to the strength of physical attacks
- `AGL` (Agility) - fixed multiplier to the strength of defence
- `WLP` (Willpower) - smaller effects to picking wrong actions
- `LCK` (Luck) - larger effects to picking correct actions

**Example:** A Fire dragon will receive bonuses as follows: `CON: 0` `INT: 0` `STR: 1` `AGL: 0` `WLP: 0` `LCK: 0`

Dragons of higher tiers will receive bonuses according to the following algorithm:
- Determine all the progenitor aspects of a dragon by taking its aspect (row 1) and splitting it into its components. Call those components row 2. If there are non-base aspects, repeat and call those row 3. Continue until there are only base aspects in the last row.

- For each base aspect `a`:
  - For each row `n`:
    - Add up all instances of the aspect `a` on the row `n` and call this sum `Sn`.
    - A row total `Sr = Sn/sqrt(n)`
  - Sum up all the row totals `Sr` to get the bonus for the particular stat. Round it off to 2 decimal places.

----
**Example 0 (Trivial):** A Fire dragon is made up only of a single aspect - Fire. Therefore:

- `n = 1 =>` `[Fire*]`

We are disregarding all aspects which aren't Fire because they aren't present (`Sn = 0`).

For row `n = 1` we sum up all instances of Fire: `S1 = 1`.

Since there are no more rows `Sr = S1 = 1`, we determine that the stat bonus for `STR` should be `1/sqrt(1) = 1`.

`CON: 0` `INT: 0` `STR: 1` `AGL: 0` `WLP: 0` `LCK: 0`

**Example 1:** A Darkness dragon is made up of Light and Void. Light is made up of Fire and Air, and Void is made up of Air and Chaos. Therefore:
- `n = 1 =>` `[Darkness]`
- `n = 2 =>` `[Light, Void]`
- `n = 3 =>` `[[Fire*, Air*], [Air*, Chaos*]]`

Since there are no base aspects in rows `n = 1` and `n = 2`, we disregard them (`Sn = 0`).

For row `n = 2` we sum up all instances of Air: `S3 = 2`.

Since there are no more rows `Sr = 2`, we determine that the stat bonus for `AGL` should be `2/sqrt(3) ~= 1.15`.

For row `n = 2` we sum up all instances of Fire: `S3 = 1`. We determine that the `STR` bonus should be `1/sqrt(3) ~= 0.58`. Same is true for Earth.

`CON: 0.58` `INT: 0` `STR: 0.58` `AGL: 1.15` `WLP: 0` `LCK: 0`

**Example 2:** A Clockwork dragon is made up of the following aspects:

- `n = 1 =>` `[Clockwork]`
- `n = 2 =>` `[Metal, Soul]`
- `n = 3 =>` `[[Crystal, Earth*], [Wisdom, Spirit]]`
- `n = 4 =>` `[[Earth*, Order*], [[Darkness, Light], [Life, Void]]]`
- `n = 5 =>` `[[[Light, Void], [Air*, Fire*]], [[Earth*, Water*], [Air*, Chaos*]]]`
- `n = 6 =>` `[[Fire*, Air*], [Air*, Chaos*]]`

Due to the overall length of the process, we will only consider Earth and the `CON` stat.

Earth appears exactly once on each row `3, 4, 5`. So `S3 = S4 = S5 = 1`. `Sr` for row `n` is gonna be therefore equal to `Sr = 1/sqrt(n)`.

Sum of the aspect bonus is therefore going to be `1/sqrt(3) + 1/sqrt(4) + 1/sqrt(5) ~= 1.52`

Repeating this process for each aspect we determine that the Clockwork dragon will have the following stat bonuses.

`CON: 1.52` `INT: 0.45` `STR: 0.86` `AGL: 1.71` `WLP: 0.5` `LCK: 0.86`

### Combat
Starting a combat (either a wild dragon - NPC, or a selected player's dragon on the Leaderboard or through the dragon page) makes the players enter a turn-based combat. Both outcomes happen simultaneously.

The opponent is always using an AI (based on the stats of the dragon it will random an action).

If a player closes the window, they will automatically lose the fight.

The fight starts with each dragon having the HP of `level * CON`

The player can select one of three types of attacks:
- Swipe - counters Magic, `F = ATK`
- Magic - counters Feint, `F = INT`
- Feint - counters Swipe, `F = AGL`

#### Attack resolution
Every attack can be "correct", "wrong" or "same" - rock-paper-scissors style.

First calculate deal luck constant `dL` and take luck constant `tL`:

- `dL = 0.03 * sqrt(LCK)`
- `tL = 0.026 * sqrt(LCK)`

Then resolve the damage. `dF` corresponds to the stat value chosen by you, `tF` - by the enemy.

- Correct:
	- Deal: `(1.2 + dL) * dF`
	- Take: `(0.7 - tL) * tF`
- Wrong:
	- Deal: `(0.6 + dL) * dF`
	- Take: `(1.3 + tL) * tF`
- Same:
	- Deal: `dF`
	- Take: `tF`

#### Post-combat
After both sides of the attack are resolved, check if any HP fell below 0, if yes then determine the outcome of the fight (win, loss, draw). Draw means loss outcome to both players.

## Tasks

Dragons can be sent on tasks to the following locations:

| Name | Aspect | Description | Mechanical Description | Unlocks |
|-----|-----|-----|----|
| Fields of Summer | Earth | Meadow/Orchard | Chances to drop Earth-related items, chance bonus for `CON` stat. | Initial |
| Phiaron | Water | Underwater City | Chances to drop Water-related items, bonus for `INT` stat. | Initial |
| Githan Volcano | Fire | | Chances to drop Fire-related items, bonus for `STR` stat. | Initial |
| Diamond Mountains | Air | | Chances to drop Air-related items, bonus for `AGL` stat. | Initial |
| Sword Coast | Order | | Chances to drop Order-related items, bonus for `WLP` stat. | Initial |
| Eastern Deserts | Chaos | | Chances to drop Chaos-related items, bonus for `LCK` stat. | Initial |
| Delos | Any | Ancient ruins | Chances to drop any of the above items, total chance slightly higher than others. | Dragons above lvl 50
| Horizon Roots | Any | End of the world | Rarest items | Tier 4 dragons or above
| Maelstrom | Any | A rare phenomenon | 100% chance to return with at least one rare item, 10x bonus to all silver brought | Unlocks once a month for an hour, randomly. |

## Inventory

Inventory is divided into two sections - ingredients and items. Ingredients are bulk things which are used in crafting. Items are unique items which are used for specialised things.

### Ingredients

These are the ingredients which the player is able to acquire on tasks:

#### Aspect ingredients:

  Aspect ingredients are used in training a dragon's respective stat as well as crafting certain aspect-related items.

  - Poison Ivy (Chaos/LCK)
  - Sword Pearls (Order/WLP)
  - Cherry Petals (Earth/CON)
  - Spring Water (Water/INT)
  - Gunpowder (Fire/STR)
  - Eastern Wind (Air/AGL)

#### Crafting ingredients:

Crafting ingredients are used to craft Hearts which increase the chance of breeding a dragon of a particular aspect.

  - Forget-me-nots
  - Silver Sand
  - Red Honey
  - Quintessence
