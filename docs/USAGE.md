# Vitae Light Control - Usage Guide

This comprehensive guide explains how to use the Vitae Light Control blueprint effectively.

## Table of Contents

1. [Understanding Vitae Lights](#understanding-vitae-lights)
2. [Mode Descriptions](#mode-descriptions)
3. [How the Blueprint Works](#how-the-blueprint-works)
4. [Timing Optimization](#timing-optimization)
5. [Common Scenarios](#common-scenarios)
6. [Best Practices](#best-practices)
7. [Advanced Usage](#advanced-usage)
8. [Troubleshooting](#troubleshooting)

---

## Understanding Vitae Lights

### What are Vitae Biodynamic Lights?

Vitae lights are special LED lights designed to mimic natural daylight cycles. They cycle through different color temperatures to support your body's natural circadian rhythm.

### How They Work

Vitae lights are controlled by a simple relay switch, but they have built-in intelligence:

1. **Power Cycling**: Each time you toggle the power (off → on), the light advances to the next color mode
2. **Reset Mechanism**: If you turn the light off for 13+ seconds, it automatically resets to the default mode (evening - warm orange)
3. **Mode Sequence**: The light cycles through modes in this order:
   - evening (warm orange) → intermediate → day (neutral white) → night (cool blue) → back to evening

### The Challenge

Without smart control, changing modes is tedious:
- To go from night → evening: Turn off, wait 15 seconds, turn on (slow!)
- To go from evening → day: Turn off, wait 15 seconds, turn on, toggle twice (complex!)

### The Solution

This blueprint automates the entire process and optimizes switching to save time.

---

## Mode Descriptions

### Evening (Evening Mode)
- **Color**: Warm Orange (~2000K)
- **Position**: 0 (default after reset)
- **Icon**: 🌅 Sunset
- **Use Cases**:
  - Evening relaxation
  - Winding down before bed
  - Cozy atmosphere
  - Reading in the evening
- **Biological Effect**: Promotes melatonin production, helps prepare for sleep

### Day (Day Mode)
- **Color**: Neutral White (~4000K)
- **Position**: 2 (2 cycles from reset)
- **Icon**: ☀️ Sun
- **Use Cases**:
  - Daytime activities
  - Work and productivity
  - General lighting
  - Morning routines
- **Biological Effect**: Supports alertness and energy during the day

### Night (Night Mode)
- **Color**: Cool Blue (~6000K)
- **Position**: 3 (3 cycles from reset)
- **Icon**: 🌙 Moon
- **Use Cases**:
  - Night reading
  - Focus and concentration
  - Late-night work
  - Alert wakefulness
- **Biological Effect**: Suppresses melatonin, increases alertness

### Off
- **State**: Light turned off
- **Position**: -1 (special case)
- **Icon**: 💡 Light Off
- **Use Case**: When you don't need any light

---

## How the Blueprint Works

### Two Automations

The blueprint creates **two automations** that work together:

#### 1. Mode Change Handler
- **Triggered by**: Changes to the input_select
- **Function**: Executes the switching sequence to reach the target mode
- **Intelligence**: Automatically determines the most efficient path

#### 2. Switch State Sync
- **Triggered by**: Relay switch turning off for 15+ seconds
- **Function**: Syncs the input_select back to "evening" (default after reset)
- **Purpose**: Keeps everything in sync when switch is toggled manually

### Decision Logic

When you select a new mode, the blueprint:

1. **Determines current position** based on previous mode
2. **Calculates target position** based on new mode
3. **Checks if reset is needed** (backward movement requires reset)
4. **Optimizes reset** (skips if switch already off 15+ seconds)
5. **Executes switching sequence** with minimal cycles
6. **Logs the transition** for debugging

### Mode Position Mapping

```
evening = 0   (default after reset)
day   = 2   (2 switches from evening)
night   = 3   (3 switches from evening)
off   = -1  (special case, just turn off)
```

---

## Timing Optimization

### The Problem: Traditional Switching

Without optimization, every backward movement requires a full reset:

| From | To | Steps | Time |
|------|----|----|------|
| evening | evening | Reset + 0 cycles | ~15s |
| evening | day | Reset + 2 cycles | ~19s |
| evening | night | Reset + 3 cycles | ~22s |
| day | evening | Reset + 0 cycles | ~15s |
| day | night | 1 cycle forward | ~3s |
| night | evening | Reset + 0 cycles | ~15s |
| night | day | Reset + 2 cycles | ~19s |

**Average time: ~15.3 seconds**

### The Solution: Smart Optimization

The blueprint optimizes in three ways:

#### 1. Forward Movement (No Reset)
When moving forward in the cycle, no reset is needed:

```
evening (0) → day (2): Just 2 cycles = ~5s
day (2) → night (3): Just 1 cycle = ~3s
```

**Time saved: ~74-87%**

#### 2. Reset Skip Detection
If the switch is already off for 15+ seconds, skip the reset wait:

```
Traditional: Turn off → wait 15s → turn on → cycle
Optimized:  Already off 15s → turn on → cycle
```

**Time saved: 15 seconds**

#### 3. Direct Off State
When turning off, just turn off the switch (no cycling needed):

```
Traditional: Might try to cycle to an "off mode"
Optimized:  Just turn off the relay
```

**Time saved: All cycling time**

### Time Savings Table

| Transition | Traditional | Optimized | Saved | % Saved |
|------------|-------------|-----------|-------|---------|
| evening → evening | ~15s | <1s | ~14s | **96%** |
| evening → day | ~19s | ~5s | ~14s | **74%** |
| evening → night | ~22s | ~7s | ~15s | **68%** |
| day → evening | ~15s | ~5s* | ~10s | **67%** |
| day → night | ~22s | ~3s | ~19s | **87%** |
| night → evening | ~15s | <1s* | ~14s | **93%** |
| night → day | ~19s | ~5s* | ~14s | **74%** |
| any → off | ~3s | <1s | ~2s | **67%** |

*Assumes reset skip optimization applies (switch already off 15+ seconds)

**Average time savings: 58%**

---

## Common Scenarios

### Scenario 1: Daily Routine

**Goal**: Automate light modes throughout the day

**Setup**:
```yaml
automation:
  - alias: "Morning - Wake Up"
    trigger:
      - platform: time
        at: "07:00:00"
    action:
      - service: input_select.select_option
        target:
          entity_id: input_select.vitae_light_mode
        data:
          option: "day"  # Neutral white for morning

  - alias: "Evening - Wind Down"
    trigger:
      - platform: sun
        event: sunset
    action:
      - service: input_select.select_option
        target:
          entity_id: input_select.vitae_light_mode
        data:
          option: "evening"  # Warm orange for evening

  - alias: "Night - Sleep Time"
    trigger:
      - platform: time
        at: "23:00:00"
    action:
      - service: input_select.select_option
        target:
          entity_id: input_select.vitae_light_mode
        data:
          option: "off"
```

**Result**: Light automatically adjusts throughout the day to support your circadian rhythm.

---

### Scenario 2: Work From Home

**Goal**: Optimize lighting for productivity and breaks

**Setup**:
```yaml
automation:
  - alias: "Work Start - Focus Mode"
    trigger:
      - platform: time
        at: "09:00:00"
    condition:
      - condition: state
        entity_id: binary_sensor.workday
        state: "on"
    action:
      - service: input_select.select_option
        target:
          entity_id: input_select.vitae_light_mode
        data:
          option: "day"

  - alias: "Lunch Break - Relax"
    trigger:
      - platform: time
        at: "12:00:00"
    action:
      - service: input_select.select_option
        target:
          entity_id: input_select.vitae_light_mode
        data:
          option: "evening"

  - alias: "Back to Work"
    trigger:
      - platform: time
        at: "13:00:00"
    action:
      - service: input_select.select_option
        target:
          entity_id: input_select.vitae_light_mode
        data:
          option: "day"
```

---

### Scenario 3: Motion-Based Control

**Goal**: Turn on appropriate mode when motion detected

**Setup**:
```yaml
automation:
  - alias: "Office Motion - Smart Light"
    trigger:
      - platform: state
        entity_id: binary_sensor.office_motion
        to: "on"
    action:
      - choose:
          # Morning/Day - use day
          - conditions:
              - condition: time
                after: "06:00:00"
                before: "18:00:00"
            sequence:
              - service: input_select.select_option
                target:
                  entity_id: input_select.vitae_light_mode
                data:
                  option: "day"

          # Evening - use evening
          - conditions:
              - condition: time
                after: "18:00:00"
                before: "22:00:00"
            sequence:
              - service: input_select.select_option
                target:
                  entity_id: input_select.vitae_light_mode
                data:
                  option: "evening"

          # Night - use night
          - conditions:
              - condition: time
                after: "22:00:00"
            sequence:
              - service: input_select.select_option
                target:
                  entity_id: input_select.vitae_light_mode
                data:
                  option: "night"

  - alias: "Office No Motion - Turn Off"
    trigger:
      - platform: state
        entity_id: binary_sensor.office_motion
        to: "off"
        for:
          minutes: 15
    action:
      - service: input_select.select_option
        target:
          entity_id: input_select.vitae_light_mode
        data:
          option: "off"
```

---

### Scenario 4: Scene Integration

**Goal**: Include Vitae light in room scenes

**Setup**:
```yaml
scene:
  - name: "Movie Night"
    entities:
      input_select.vitae_light_mode: "evening"
      light.tv_backlight:
        state: on
        brightness: 30
      media_player.tv: on

  - name: "Reading Time"
    entities:
      input_select.vitae_light_mode: "day"
      light.reading_lamp:
        state: on
        brightness: 100

  - name: "Sleep Mode"
    entities:
      input_select.vitae_light_mode: "off"
      light.all_lights: off
      switch.white_noise: on
```

---

### Scenario 5: Manual Override with Auto-Reset

**Goal**: Allow manual control but auto-return to schedule

**Setup**:
```yaml
input_boolean:
  vitae_manual_mode:
    name: "Vitae Manual Mode"
    initial: off

automation:
  - alias: "Vitae - Track Manual Changes"
    trigger:
      - platform: state
        entity_id: input_select.vitae_light_mode
    condition:
      - condition: template
        value_template: "{{ trigger.from_state.state != trigger.to_state.state }}"
    action:
      - service: input_boolean.turn_on
        target:
          entity_id: input_boolean.vitae_manual_mode
      - delay:
          hours: 1
      - service: input_boolean.turn_off
        target:
          entity_id: input_boolean.vitae_manual_mode

  - alias: "Vitae - Auto Schedule (when not manual)"
    trigger:
      - platform: time
        at: "07:00:00"
    condition:
      - condition: state
        entity_id: input_boolean.vitae_manual_mode
        state: "off"
    action:
      - service: input_select.select_option
        target:
          entity_id: input_select.vitae_light_mode
        data:
          option: "day"
```

---

## Best Practices

### 1. Timing Configuration

**Reset Delay (default 15s)**:
- Manufacturer recommends minimum 13 seconds
- Use 15s for reliability
- Increase to 20s if you experience reset issues
- Don't go below 13s

**Switch Delay (default 1s)**:
- 1 second works well for most setups
- Increase to 1.5-2s if cycles are unreliable
- Decrease to 0.5s for faster switching (test carefully)

### 2. Dashboard Design

**Keep it simple**:
- Use button cards for quick access
- Show current mode clearly
- Group related controls together

**Consider user experience**:
- Use intuitive icons (sunset, sun, moon)
- Label buttons clearly
- Provide visual feedback

### 3. Automation Strategy

**Follow natural rhythms**:
- Warm light (evening) in evening
- Neutral light (day) during day
- Cool light (night) for night activities

**Avoid rapid changes**:
- Don't switch modes too frequently
- Allow 30+ seconds between mode changes
- Use conditions to prevent unnecessary switches

### 4. Mode Selection

**Evening (Warm Orange)**:
- Best for: 2-3 hours before bedtime
- Supports: Melatonin production, relaxation
- Avoid: Morning/daytime (may cause drowsiness)

**Day (Neutral White)**:
- Best for: Daytime activities, work
- Supports: Alertness, productivity
- Avoid: Late evening (may disrupt sleep)

**Night (Cool Blue)**:
- Best for: Night shift work, late-night reading
- Supports: Alertness, concentration
- Avoid: Before bedtime (suppresses melatonin)

### 5. Multiple Lights

**Create separate configs**:
- One input_select per light
- One blueprint automation per light
- Independayt control for each room

**Coordinate when needed**:
```yaml
automation:
  - alias: "All Vitae Lights - Evening"
    trigger:
      - platform: sun
        event: sunset
    action:
      - service: input_select.select_option
        target:
          entity_id:
            - input_select.bedroom_vitae_mode
            - input_select.living_room_vitae_mode
            - input_select.office_vitae_mode
        data:
          option: "evening"
```

---

## Advanced Usage

### Custom Timing Templates

Use templates for dynamic timing based on conditions:

```yaml
automation:
  - alias: "Vitae - Smart Reset Delay"
    trigger:
      - platform: state
        entity_id: input_select.vitae_light_mode
    action:
      - variables:
          custom_reset_delay: >
            {% if is_state('input_boolean.fast_mode', 'on') %}
              13
            {% else %}
              20
            {% endif %}
```

### State Tracking

Monitor light state changes:

```yaml
automation:
  - alias: "Vitae - Log Mode Changes"
    trigger:
      - platform: state
        entity_id: input_select.vitae_light_mode
    action:
      - service: notify.mobile_app
        data:
          message: "Vitae light changed to {{ states('input_select.vitae_light_mode') }}"
```

### Integration with Adaptive Lighting

Coordinate with other smart lighting:

```yaml
automation:
  - alias: "Sync Adaptive Lighting with Vitae"
    trigger:
      - platform: state
        entity_id: input_select.vitae_light_mode
    action:
      - choose:
          - conditions:
              - condition: state
                entity_id: input_select.vitae_light_mode
                state: "evening"
            sequence:
              - service: adaptive_lighting.set_manual_control
                data:
                  entity_id: light.other_lights
                  manual_control: false
              - service: adaptive_lighting.apply
                data:
                  entity_id: light.other_lights
                  color_temp_kelvin: 2000
```

---

## Troubleshooting

### Issue: Mode doesn't change

**Symptoms**: Input select changes but light stays the same

**Checks**:
1. Verify relay switch entity is correct
2. Check switch is responding to commands
3. Look at logbook entries (Settings → Logbook)
4. Ensure automation is enabled

**Fix**:
```yaml
# Test the switch manually
service: switch.turn_on
target:
  entity_id: switch.vitae_relay
```

### Issue: Wrong mode after manual toggle

**Symptoms**: Manually toggled switch, mode out of sync

**Explanation**: The sync automation only triggers after 15 seconds off

**Fix**:
- Wait 15+ seconds for auto-sync, or
- Manually set input_select to match current state

### Issue: Switching takes too long

**Symptoms**: Mode changes are slow

**Checks**:
1. Check reset_delay setting (default 15s)
2. Check switch_delay setting (default 1s)
3. Review logbook to see which path was taken

**Optimization**:
- Blueprint already optimizes automatically
- Reduce switch_delay to 0.5s for faster cycles (test reliability)
- Ensure switch isn't being controlled by other automations

### Issue: Light doesn't reset properly

**Symptoms**: Light ends up in wrong mode

**Checks**:
1. Verify reset_delay is at least 13 seconds
2. Check that switch actually turns off
3. Ensure no other automations interfere

**Fix**:
- Increase reset_delay to 20 seconds
- Disable other automations controlling the same switch
- Manually reset: Turn off, wait 20s, turn on

### Issue: Automation not running

**Symptoms**: Nothing happens when changing input_select

**Checks**:
1. Go to Settings → Automations → Check if automation exists
2. Verify automation is enabled (toggle should be on)
3. Check automation traces (click automation → Traces)

**Fix**:
- Enable the automation
- Reload automations (Developer Tools → YAML → Reload Automations)
- Check Home Assistant logs for errors

### Issue: Input select resets unexpectedly

**Symptoms**: Mode changes back to evening on its own

**Explanation**: Switch state sync automation detected light was off 15+ seconds

**This is normal behavior when**:
- Light manually turned off for 15+ seconds
- Power outage occurred
- Switch controlled by another automation

**To prevent**:
- Modify sync automation condition if needed
- Use scenes to coordinate light state

---

## Support

For additional help:
- Check the main [README.md](../README.md)
- Review [configuration examples](../examples/configuration.yaml)
- Report issues on [GitHub](https://github.com/yourusername/vitae-light-blueprint/issues)
- Check Home Assistant logs (Settings → System → Logs)

---

**Last Updated**: 2025-12-27
**Blueprint Version**: 1.0.0
