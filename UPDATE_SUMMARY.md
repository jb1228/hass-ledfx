# LedFX Home Assistant Integration - Update Summary

## Version 3.1.0 - Reload and Reconnection Support

### Overview
This update modernizes the LedFX Home Assistant integration after 3 years of inactivity, adding critical features for reliability and ease of maintenance:

1. **Integration Reload Support** - Allows reloading the integration without restarting Home Assistant
2. **Automatic Reconnection** - Intelligently reconnects to LedFX when it restarts or becomes unavailable
3. **Exponential Backoff** - Uses exponential backoff to reduce load during prolonged disconnections

---

## Key Changes

### 1. Integration Reload Support

#### Modified: `custom_components/ledfx/__init__.py`
- **Added**: `async_reload_entry()` function to support integration reloading
- **Purpose**: Allows users to reload the LedFX integration from the Home Assistant UI without restarting the entire system
- **How it works**: 
  - Unloads all platforms and stops the updater
  - Reinitializes the integration from scratch
  - Re-establishes connection to LedFX

**Example Usage:**
```
Settings > Devices & Services > LedFx > [Options Menu] > Reload
```

#### Modified: `custom_components/ledfx/manifest.json`
- **Updated**: Version bumped to `3.1.0`
- This manifests proper support for the reload feature

---

### 2. Automatic Reconnection with Intelligent Backoff

#### Modified: `custom_components/ledfx/updater.py`

**New Constants:**
```python
MAX_RECONNECT_DELAY: Final = 300  # 5 minutes
MIN_RECONNECT_DELAY: Final = 5    # 5 seconds
```

**New Instance Variable:**
- `_connection_failures: int = 0` - Tracks consecutive connection failures

**Enhanced Methods:**

##### `update()` method
- **Connection Error Handling**: 
  - Increments failure counter on connection errors
  - Logs warnings with attempt count
  - Resets counter on successful reconnection
  - Logs info message when connection is restored

##### `_get_reconnect_delay()` method (New)
Calculates exponential backoff delay:
- **1st failure**: 5 seconds
- **2nd failure**: 10 seconds
- **3rd failure**: 20 seconds
- **4th failure**: 40 seconds
- **5th+ failures**: Capped at 300 seconds (5 minutes)

Formula: `MIN_RECONNECT_DELAY * (2 ^ (failures - 1))`

##### `async_config_entry_first_refresh()` method (Override)
- Adjusts update interval based on connection failures
- Applies exponential backoff delay when failures are detected
- Logs debug messages for troubleshooting

##### `async_request_refresh()` method (Override)
- Resets to normal update interval on manual refresh requests
- Allows users to force immediate reconnection attempts
- Useful for testing and manual recovery

---

## Benefits

### For Users
✅ **No Restart Required**: Reload integration without restarting Home Assistant
✅ **Automatic Recovery**: Seamlessly reconnects when LedFX comes back online
✅ **Smart Retry Logic**: Intelligent exponential backoff prevents overwhelming failed connections
✅ **Better Diagnostics**: Warning and info logs help troubleshoot connection issues

### For System Health
✅ **Reduced Load**: Exponential backoff prevents constant reconnection attempts
✅ **Graceful Degradation**: Integration becomes unavailable during disconnection, entities reflect this state
✅ **Better Observability**: Connection failures and recoveries are logged appropriately

---

## Behavior Examples

### Scenario 1: LedFX Restarts
1. Integration detects connection error
2. Increments failure counter
3. Sets update interval to 5 seconds (exponential backoff)
4. Attempts reconnection every 5 seconds
5. When LedFX comes back online, connection succeeds
6. Failure counter resets to 0
7. Update interval returns to normal (default 7 seconds)
8. User receives info log: "Successfully reconnected to LedFX at 192.168.1.100:8080"

### Scenario 2: Network Cable Unplugged
1. Multiple connection errors detected
2. Failure counter increments: 1, 2, 3, 4, ...
3. Update intervals extend: 5s, 10s, 20s, 40s, 80s, ... (capped at 5 min)
4. User reloads integration manually
5. Integration unloads and reloads cleanly
6. Fresh connection attempt with failure counter reset

### Scenario 3: User Reloads Integration
1. User clicks "Reload" in Home Assistant UI
2. `async_reload_entry()` is called
3. All entities are removed and unloaded
4. Updater is stopped and cleaned up
5. Integration is set up fresh
6. All entities are re-created and restored

---

## Migration from Previous Version

No breaking changes. The update is fully backward compatible:
- Existing configurations work without modification
- No config.yaml changes required
- Manual reload feature is optional - use it only when needed

---

## Technical Implementation Details

### Exponential Backoff Formula
```python
def _get_reconnect_delay(self) -> timedelta:
    if self._connection_failures == 0:
        return self._update_interval
    
    delay = min(
        MIN_RECONNECT_DELAY * (2 ** (self._connection_failures - 1)),
        MAX_RECONNECT_DELAY,
    )
    return timedelta(seconds=delay)
```

### Connection Failure Tracking
- **Incremented**: When `LedFxConnectionError` is caught during update
- **Logged**: Warning message with failure count and device address
- **Reset**: When update succeeds (code is success)
- **Applied**: Update interval is adjusted based on failure count

### Logging
- **WARNING**: Connection errors with attempt count
- **INFO**: Successful reconnection after failures
- **DEBUG**: Update interval adjustments for troubleshooting

---

## Testing Recommendations

### Test 1: Basic Reload
1. Configure LedFX integration
2. Navigate to Settings > Devices & Services
3. Find LedFX entry
4. Click the options menu (three dots)
5. Select "Reload"
6. Verify entities are still available after reload

### Test 2: Connection Loss and Recovery
1. LedFX running and connected
2. Verify integration shows "connected"
3. Stop or disconnect LedFX service
4. Verify Home Assistant logs show connection error
5. Verify update interval increases (exponential backoff)
6. Restart LedFX
7. Verify connection is restored
8. Verify update interval returns to normal

### Test 3: Manual Refresh During Disconnection
1. Disconnect LedFX
2. Wait for exponential backoff to increase interval to ~40+ seconds
3. Call "Reload" service or use Reload button
4. Verify update interval resets to normal
5. Verify reconnection attempt happens immediately

---

## Future Enhancements

Potential improvements for future versions:
- Configuration options for exponential backoff parameters
- Heartbeat/ping check before full update
- Separate connectivity sensor
- MQTT-style last_will_and_testament detection
- Configurable max retry attempts with permanent failure state

---

## Files Modified

1. **custom_components/ledfx/__init__.py**
   - Added: `async_reload_entry()` function

2. **custom_components/ledfx/updater.py**
   - Added: Connection failure tracking
   - Added: Exponential backoff calculation
   - Enhanced: `update()` method with reconnection logic
   - Added: `_get_reconnect_delay()` method
   - Added: `async_config_entry_first_refresh()` override
   - Added: `async_request_refresh()` override

3. **custom_components/ledfx/manifest.json**
   - Updated: Version to 3.1.0

---

## Questions or Issues?

If you encounter any issues:
1. Check Home Assistant logs for LedFX connection errors
2. Ensure LedFX is running and accessible at configured address
3. Verify network connectivity between Home Assistant and LedFX
4. Try manually reloading the integration
5. Check existing GitHub issues or create a new one

---

## Version History

- **3.1.0** (Current) - Added reload support and automatic reconnection with exponential backoff
- **3.0.0** - Previous stable release
