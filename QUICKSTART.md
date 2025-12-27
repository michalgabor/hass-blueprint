# Quick Start Guide - Vitae Light Control

Get up and running with Vitae Light Control in 5 minutes!

## Prerequisites

- Home Assistant 2024.1 or newer
- A Vitae biodynamic light connected to a relay switch
- Switch entity available in Home Assistant (e.g., `switch.vitae_relay`)

## Installation Steps

### 1. Install the Blueprint (30 seconds)

**Option A: Manual**
```bash
cd /config/blueprints/automation/
wget https://raw.githubusercontent.com/yourusername/vitae-light-blueprint/main/blueprints/automation/vitae_light_control.yaml
```

**Option B: Download**
Download [vitae_light_control.yaml](blueprints/automation/vitae_light_control.yaml) and place it in `/config/blueprints/automation/`

### 2. Create Input Select (1 minute)

Add to your `configuration.yaml`:

```yaml
input_select:
  vitae_light_mode:
    name: "Vitae Light Mode"
    options:
      - "off"
      - "evening"
      - "day"
      - "night"
    initial: "off"
    icon: mdi:lightbulb-group
```

Restart Home Assistant or reload: **Developer Tools** → **YAML** → **Input Select**

### 3. Create Blueprint Automation (2 minutes)

**Via UI:**
1. Go to **Settings** → **Automations & Scenes** → **Blueprints**
2. Find **Vitae Light Control**
3. Click **Create Automation**
4. Fill in:
   - **Relay Switch**: Your switch entity (e.g., `switch.vitae_relay`)
   - **Mode Input Select**: `input_select.vitae_light_mode`
   - **Reset Delay**: 15 (default)
   - **Switch Delay**: 1 (default)
5. Click **Save**

**Via YAML:**
Add to `automations.yaml`:

```yaml
- use_blueprint:
    path: vitae_light_control.yaml
    input:
      relay_switch: switch.vitae_relay  # ← Change to your switch
      mode_input_select: input_select.vitae_light_mode
      reset_delay: 15
      switch_delay: 1
```

### 4. Add Dashboard Control (1 minute)

Edit your dashboard, add a new card with this YAML:

```yaml
type: vertical-stack
cards:
  - type: entities
    title: Vitae Light
    entities:
      - entity: input_select.vitae_light_mode
        name: Mode

  - type: horizontal-stack
    cards:
      - type: button
        name: "Off"
        icon: mdi:lightbulb-off
        tap_action:
          action: call-service
          service: input_select.select_option
          target:
            entity_id: input_select.vitae_light_mode
          data:
            option: "off"

      - type: button
        name: "Evening"
        icon: mdi:weather-sunset
        tap_action:
          action: call-service
          service: input_select.select_option
          target:
            entity_id: input_select.vitae_light_mode
          data:
            option: "evening"

      - type: button
        name: "Day"
        icon: mdi:white-balance-sunny
        tap_action:
          action: call-service
          service: input_select.select_option
          target:
            entity_id: input_select.vitae_light_mode
          data:
            option: "day"

      - type: button
        name: "Night"
        icon: mdi:weather-night
        tap_action:
          action: call-service
          service: input_select.select_option
          target:
            entity_id: input_select.vitae_light_mode
          data:
            option: "night"
```

## Test It Out!

1. Click the **Evening** button - light should turn on in warm orange
2. Click **Day** - light should switch to neutral white (~5 seconds)
3. Click **Night** - light should switch to cool blue (~3 seconds)
4. Click **Off** - light should turn off

## Mode Reference

| Mode | Color | Icon | Best For |
|------|-------|------|----------|
| **Evening** | Warm Orange | 🌅 | Evening, before bed |
| **Day** | Neutral White | ☀️ | Daytime, work |
| **Night** | Cool Blue | 🌙 | Night reading, focus |
| **Off** | - | 💡 | No light needed |

## Common Customizations

### Add Daily Schedule

Add to `automations.yaml`:

```yaml
- alias: "Vitae - Morning"
  trigger:
    - platform: time
      at: "07:00:00"
  action:
    - service: input_select.select_option
      target:
        entity_id: input_select.vitae_light_mode
      data:
        option: "day"

- alias: "Vitae - Evening"
  trigger:
    - platform: sun
      event: sunset
  action:
    - service: input_select.select_option
      target:
        entity_id: input_select.vitae_light_mode
      data:
        option: "evening"
```

### Adjust Timing

If your light needs different timing, edit the blueprint automation:
- **Reset Delay**: Increase to 20 if light doesn't reset reliably
- **Switch Delay**: Increase to 1.5 for slower, more reliable switching

## Troubleshooting

**Light doesn't change mode?**
- Check that switch entity is correct
- Verify automation is enabled (Settings → Automations)
- Look at logbook for errors

**Mode out of sync after manual toggle?**
- Wait 15 seconds for auto-sync, or
- Manually select correct mode from dropdown

**Switching too slow?**
- This is normal - blueprint optimizes automatically
- evening→day takes ~5s, day→night takes ~3s
- Much faster than traditional method (15-22s)

## Next Steps

- 📖 Read the full [README.md](README.md) for detailed documentation
- 🎯 Check [docs/USAGE.md](docs/USAGE.md) for advanced usage scenarios
- 🎨 Explore [examples/lovelace.yaml](examples/lovelace.yaml) for more dashboard options
- ⚙️ Review [examples/configuration.yaml](examples/configuration.yaml) for automation ideas

## Need Help?

- **Documentation**: See [README.md](README.md)
- **Issues**: Report on [GitHub](https://github.com/yourusername/vitae-light-blueprint/issues)
- **Logs**: Check Settings → System → Logs in Home Assistant

---

**Congratulations!** You now have intelligent Vitae light control with optimized switching. Enjoy your biodynamic lighting! 🎉
