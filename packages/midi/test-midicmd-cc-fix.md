# MIDI CC Bug Fix Validation

## Bug Description
When `midicmd` was passed as an array (e.g., from `midicmd("cc:74:1")`), the CC branch only executed when `midicmd.length === 2` and then called `sendCC(midicmd[0], midicmd[1] / 127, ...)`, sending a control change for controller 'cc' with the wrong value while dropping the actual controller number/value.

## Expected Behavior
For `midicmd("cc:74:1")`:
- Parse into array: `["cc", 74, 1]`
- Send CC message for controller #74 with value 1/127 ≈ 0.0079

## Bug Details

### Before Fix (packages/midi/midi.mjs:449-452)
```javascript
} else if (midicmd[0] === 'cc') {
  if (midicmd.length === 2) {  // ❌ Wrong: should be 3
    sendCC(midicmd[0], midicmd[1] / 127, device, midichan, timeOffsetString);  // ❌ Wrong indices
  }
}
```

**Problems:**
1. Checked for `length === 2` instead of `3`
2. Used `midicmd[0]` ("cc") as controller number instead of `midicmd[1]` (74)
3. Used `midicmd[1]` (74) as value instead of `midicmd[2]` (1)
4. Result: CC messages with 3-element arrays were never sent

### After Fix (packages/midi/midi.mjs:449-452)
```javascript
} else if (midicmd[0] === 'cc') {
  if (midicmd.length === 3) {  // ✅ Correct: checks for 3 elements
    sendCC(midicmd[1], midicmd[2] / 127, device, midichan, timeOffsetString);  // ✅ Correct indices
  }
}
```

**Fixes:**
1. Now checks for `length === 3` to match the format `["cc", ccn, ccv]`
2. Uses `midicmd[1]` as controller number (e.g., 74)
3. Uses `midicmd[2]` as value (e.g., 1), normalized to 0-1 range by dividing by 127
4. Result: CC messages are now correctly sent

## Test Cases

### Test 1: Basic CC Message
```javascript
midicmd("cc:74:1").midi('IAC Driver')
```
- **Input Array:** `["cc", 74, 1]`
- **Expected:** Send CC for controller 74, value 1/127 ≈ 0.0079
- **Status:** ✅ Should work with fix

### Test 2: CC with Different Controller Numbers
```javascript
stack(
  midicmd("cc:1:64").midi('IAC Driver'),   // Modulation wheel, mid value
  midicmd("cc:7:127").midi('IAC Driver'),  // Volume, max value
  midicmd("cc:10:0").midi('IAC Driver')    // Pan, min value
)
```
- **Expected:** All three CC messages sent correctly
- **Status:** ✅ Should work with fix

### Test 3: Mixed MIDI Commands
```javascript
stack(
  midicmd("cc:74:1").midi('IAC Driver'),        // CC message
  midicmd("progNum:1").midi('IAC Driver'),       // Program change
  midicmd("sysex:[0x43]:[0x79]").midi('IAC Driver')  // Sysex
)
```
- **Expected:** All three command types work correctly
- **Status:** ✅ Should work with fix (other commands unchanged)

## Format Reference
From packages/midi/README.md:186-187:
```javascript
// "cc:ccn:ccv"
midicmd("cc:74:1").midi('IAC Driver')
```

Where:
- `cc` = command type
- `ccn` = controller number (0-127)
- `ccv` = controller value (0-127)

## Mini Notation Parsing
The colon operator (`:`) in mini notation appends elements to create arrays:
- `"cc"` → `["cc"]`
- `"cc:74"` → `["cc", 74]`
- `"cc:74:1"` → `["cc", 74, 1]`

Source: packages/mini/krill.pegjs:159-160 (op_tail)

## Validation Checklist
- [x] Bug identified in packages/midi/midi.mjs:449-452
- [x] Fix applied: changed length check from 2 to 3
- [x] Fix applied: changed sendCC arguments to use correct array indices
- [x] Fix aligns with documented format "cc:ccn:ccv"
- [x] Fix follows same pattern as progNum and sysex handlers
- [x] No other code paths affected

## Related Code
- **MIDI handler:** packages/midi/midi.mjs:289-461 (Pattern.prototype.midi)
- **Documentation:** packages/midi/README.md:180-192
- **Mini notation parser:** packages/mini/krill.pegjs (op_tail operator)
- **Control definition:** packages/core/controls.mjs:1706 (midicmd)

## Conclusion
The fix correctly implements the documented behavior for CC messages via midicmd. Three-element arrays from `midicmd("cc:ccn:ccv")` will now properly send MIDI control change messages.
