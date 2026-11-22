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
| `IssueTargetOrder`, `IssueTargetOrderById` attack or move on a unit in fog or under the black mask | bugfix | Returns true and walks there, as measured in Reforged. |
| `IssueTargetOrder` smart on an unseen enemy | bugfix | Returns false, as measured; also false for a fogged ally without shared vision and an undetected invisible enemy (inferred). |
| `IssueTargetOrder` attack on an undetected invisible unit | deliberate | Returns false; Reforged returns true but the unit never hits. |
| Unit-target spells and `UnitUseItemTarget` on unseen targets | works | Return false, as measured in Reforged. |
| `GroupTargetOrder`, `GroupTargetOrderById` | partial | Follow the single-unit rules (inferred); nil target returns false, empty group returns true (unmeasured). |
| Script target order queued on a stunned or casting unit | bugfix | Keeps the script rule when it starts (was dropped). |
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
| Smart, move and follow orders on a unit: stop distance | deliberate | The follower stops 75 units from the target's edge (100 for buildings), not Warcraft III's 300; a map's own `FollowRange` wins. |
| Smart, move and follow orders on a unit: settling | deliberate | The follower walks in to about 40 units, restarts only well beyond the range, and stands at least 0.3 s once settled. |
| Following a moving unit | deliberate | The follower walks at the target's speed when close, so it neither flickers between walking and standing nor runs ahead. |
| Following a target that turns or circles close | deliberate | A follower inside about 40 units stands unless the target walks away; on a U-turn it restarts after about 0.5 s. |
| Follower blocked by a crowd | deliberate | It stands and retries after 1 s, then every 2 s. |
| `GetUnitCurrentOrder` while following a friendly unit after smart | works | Reports smart (851971) throughout, as measured; a move order on a unit reads 851986. |
| Smart on your own building | bugfix | The unit walks to contact and the order ends (reads 0), no follow, as measured; an allied building is the same (inferred). |
| Smart on a neutral Passive building (shop, gold mine, waygate) | unverified | The unit keeps following and the order stays smart; Warcraft III is not measured. |
| Follower after an auto-acquired attack | bugfix | Returns to following the target; the order reads smart throughout. |
| Follower snared while in follow range | bugfix | Resumes following when the snare ends (inferred); a follow begun while rooted does not resume after unrooting. |
| `GetUnitMoveFollowBehavior` with a null target | bugfix | Returns null instead of raising an error. |

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
| PLAYER vs UNIT trigger order for death, damaged, damaging, hero level, issued orders and item pickup | bugfix | The player event fires before the unit event whatever the registration order, as measured; sell and attacked still fire the unit first (unmeasured). |
| `IssueImmediateOrder(u, "stop")` and `ISSUED_ORDER` | gap | Fires no `EVENT_UNIT_ISSUED_ORDER`; Reforged fires one (order id 851972); holdposition does fire. |
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
