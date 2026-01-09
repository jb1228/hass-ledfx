# Quick Start Guide - Updated LedFX Integration v3.1.0

## What Changed?

Your LedFX integration has been updated to support:
1. **Integration reload** without Home Assistant restart
2. **Automatic reconnection** when LedFX restarts
3. **Smart retry logic** that backs off gradually

## How to Use the New Features

### Reload the Integration (No Restart Needed!)

**Via Web Interface:**
1. Go to Settings → Devices & Services
2. Find the LedFx entry
3. Click the three-dot menu → **Reload**
4. Done! Integration reloads in seconds

**Via Service Call (Automation/Script):**
```yaml
service: homeassistant.reload_config_entry
data:
  entry_id: "YOUR_ENTRY_ID_HERE"
```

### When LedFX Service Restarts

- ✅ Integration detects disconnection automatically
- ✅ Attempts reconnection every 5 seconds
- ✅ Updates interval if connection takes longer
- ✅ Auto-recovers when LedFX comes back online
- ✅ Shows as unavailable to Home Assistant during downtime (entities gray out)

**No action needed!** The integration handles it automatically.

## Monitoring Connection Status

Check Home Assistant logs:

```bash
tail -f /home/homeassistant/.homeassistant/home-assistant.log | grep ledfx
```

Look for messages like:
- `Connection error to LedFX...` - Disconnected
- `Successfully reconnected to LedFX...` - Back online
- `Adjusting update interval...` - Using exponential backoff

## Troubleshooting

### Integration still shows unavailable after LedFX restarts
1. Verify LedFX is actually running: `http://YOUR_IP:8080/api/info`
2. Check network connectivity from Home Assistant
3. Check firewall rules
4. Manually reload the integration via UI (see above)

### Want immediate reconnection attempt
1. Go to Settings → Devices & Services → LedFx
2. Click the three-dot menu → **Reload**
3. This resets the retry backoff and attempts immediately

### Seeing lots of "Connection error" messages
- This is normal if LedFX is offline
- Integration will keep trying with increasing delays
- Once LedFX is back online, it will reconnect automatically

## What's NOT Changed

- ❌ Configuration format - same as before
- ❌ Entity names and behaviors - same as before
- ❌ Automations and scripts - will still work
- ❌ Any dependencies or requirements - same as before

**Just drop in the new files and go!**

## Version Info

- Current version: **3.1.0**
- Previous version: 3.0.0
- Release type: Maintenance update with new features

## Need Help?

1. Check the full documentation: [UPDATE_SUMMARY.md](UPDATE_SUMMARY.md)
2. Check connection logs for error messages
3. Manually reload the integration
4. Check GitHub issues: https://github.com/dmamontov/hass-ledfx/issues
