---
title: Orders, damage and buffs
nav_order: 3
parent: Differences from Warcraft III
---

# Orders, damage and buffs
{: .no_toc }

## Orders

Attack orders end when the target can no longer be attacked. A paused unit still accepts orders.

| Native | Status | What differs |
|---|---|---|
| `IssueTargetOrder`, `UnitUseItemTarget` on a target in fog | unverified | Refused (false); WC3 behavior for script orders not confirmed. |
| `IssueTargetOrder`, `IssueTargetOrderById` | bugfix | Return false for a nil target or a removed unit, destructable or item; corpses and killed trees stay valid targets. |
| `IssuePointOrder` | unverified | Same as `IssuePointOrderById`; a nil order string returns false. |
| `IssueImmediateOrder`, `IssueImmediateOrderById` with "stop" | bugfix | Returns true. |
| Orders to a paused unit | gap | A paused unit still accepts orders. |
| Order issued inside an issued-target-order trigger condition | bugfix | The outer order keeps its target (was replaced or crashed). |
| `IssueBuildOrder` | works | Resolves the unit name; an unknown name returns false. |
| Build placement off the pathing grid | bugfix | Refused with the Cantplace error. |
| Channel base order strings that are not WC3 order strings | unverified | Get WarMash-only order ids (0xD8000 to 0xE7FFF), not the real game's. |
| Targeted order on an unseen, immune or ethereal unit | bugfix | The visibility error comes first, so nothing about the unit is revealed. |
| Click on a unit the active spell cannot target | deliberate | Shows the target error and casts nothing; targeting stays active. |
| Attack target turns invulnerable or invisible | unverified | The attacker drops the target and ends the attack order. |
| Attack target lost mid-chase or mid-backswing | deliberate | The attack ends at once; a swing before its damage point deals nothing. |

## Damage and attack events

DAMAGING fires before armor, DAMAGED after. Events fire only for units, not for misses or dead or invulnerable targets.

```lua
-- Halve all damage that is not an attack
local t = CreateTrigger()
for i = 0, bj_MAX_PLAYER_SLOTS - 1 do
    TriggerRegisterPlayerUnitEvent(t, Player(i), EVENT_PLAYER_UNIT_DAMAGED, nil)
end
TriggerAddAction(t, function()
    if not BlzGetEventIsAttack() then
        BlzSetEventDamage(GetEventDamage() * 0.5)
    end
end)
```

| Native | Status | What differs |
|---|---|---|
| `GetEventDamage`, `GetEventDamageSource`, `BlzGetEventDamageTarget`, `BlzSetEventDamage` | works | Outside a damage event the getters return 0 or nil and the setter does nothing. |
| The same getters outside an event | unverified | 0, nil or false, with one log line. |
| `BlzGetEventAttackType`, `BlzGetEventDamageType`, `BlzGetEventWeaponType`, `BlzGetEventIsAttack` and the type setters | works | Types set in DAMAGING feed the armor calculation; setters return false outside an event. |
| `EVENT_UNIT_DAMAGING`, `EVENT_PLAYER_UNIT_DAMAGING` | works | Before armor; setting 0 blocks the hit, a negative amount heals. |
| `EVENT_UNIT_DAMAGED`, `EVENT_PLAYER_UNIT_DAMAGED` | works | After armor; the amount set is the final life loss. |
| When DAMAGED is skipped | unverified | Only when DAMAGING set the amount to 0 or less (behaviour of 1.31.1 and later). |
| Damage dealt inside a damage handler | deliberate | Fires the events again, up to 32 levels deep. |
| Handler makes the target invulnerable or kills it | unverified | Invulnerable: the hit deals nothing; `KillUnit` in DAMAGING still fires DAMAGED. |
| Damage set to 0 or less, seen by lifesteal and thorns | unverified | Counts as 0 damage. |
| Hit by a unit without thorns | bugfix | Exactly one DAMAGING and one DAMAGED per hit. |
| `EVENT_UNIT_ATTACKED`, `EVENT_PLAYER_UNIT_ATTACKED` | works | Fires once per swing, before the damage point. |
| When trigger actions run | deliberate | Damage triggers run actions inside the event; other events queue code actions until the tick ends. |
| `GetHandleId(attacktype)` | bugfix | Returns the `common.j` index, 0 to 6. |
| `UnitDamageTarget` hit sound | bugfix | Plays the weapon type's sound; `WEAPON_TYPE_WHOKNOWS` is silent. |
| `UnitDamagePoint` | works | Honours the delay and collision size, hits the source and allies, as measured. |
| Attack 2 missile art, speed, arc and homing (also `BlzGetUnitWeapon*`) | bugfix | Use attack 2's own values; they used attack 1's. |

## Buffs

The buff filter natives use a mostly inferred classification table: check the buffs your map relies on.

| Native | Status | What differs |
|---|---|---|
| `UnitCountBuffsEx`, `UnitHasBuffsEx`, `UnitRemoveBuffs`, `UnitRemoveBuffsEx` | works | Filter by polarity, magic or physical, aura and timed life on an inferred table. |
| Non-stacking aura buffs from several sources | bugfix | The buff stays listed while another source still applies. |
| Spell buffs with the same buff id | deliberate | A timed spell buff replaces the older one; auras and passives still stack. |
| `UnitRemoveAbility(unit, buffId)` | bugfix | Removes every instance, not only the first. |
| Slow (`Aslo`) on re-level of a disabled ability | bugfix | No longer errors. |
| `UnitApplyTimedLife` | works | A later call replaces the timer without killing the unit. |
| `UnitApplyTimedLife` with 0, negative or huge durations | bugfix | 0 or negative expires on the next tick; huge values no longer overflow. |
| `UnitApplyTimedLife` target art | bugfix | No target art is shown. |

Details:
- Timed-life buff `BTLF` is classed neither magic nor physical, so filters with both false never count it. Removing it kills the summon.
- Polymorph `Bply` counts with `(false, true, false, false, false, false, false)`. Ensnare's net is negative physical and not dispellable (measured).
