You are helping calibrate a split-flap display by analyzing photos. The display is a 3x15 grid (45 modules total) controlled by a Flask API on a Raspberry Pi.

The user will provide the Pi's hostname or IP as an argument: $ARGUMENTS

## Grid Layout

```
Row 0:  [ 00 ][ 01 ][ 02 ][ 03 ][ 04 ][ 05 ][ 06 ][ 07 ][ 08 ][ 09 ][ 10 ][ 11 ][ 12 ][ 13 ][ 14 ]
Row 1:  [ 15 ][ 16 ][ 17 ][ 18 ][ 19 ][ 20 ][ 21 ][ 22 ][ 23 ][ 24 ][ 25 ][ 26 ][ 27 ][ 28 ][ 29 ]
Row 2:  [ 30 ][ 31 ][ 32 ][ 33 ][ 34 ][ 35 ][ 36 ][ 37 ][ 38 ][ 39 ][ 40 ][ 41 ][ 42 ][ 43 ][ 44 ]
```

## Character Set (64 characters, index 0-63)

```
 0: (space)   1: A   2: B   3: C   4: D   5: E   6: F   7: G
 8: H   9: I  10: J  11: K  12: L  13: M  14: N  15: O
16: P  17: Q  18: R  19: S  20: T  21: U  22: V  23: W
24: X  25: Y  26: Z  27: 0  28: 1  29: 2  30: 3  31: 4
32: 5  33: 6  34: 7  35: 8  36: 9  37: !  38: @  39: #
40: $  41: &  42: (  43: )  44: -  45: +  46: =  47: ;
48: "  49: :  50: %  51: '  52: .  53: ,  54: /  55: ?
56: *  57: r  58: o  59: y  60: g  61: b  62: p  63: w
```

Characters 57-63 are color flaps (red, orange, yellow, green, blue, purple, white).

## API Reference

Base URL: `http://$ARGUMENTS`

### Home all modules
```bash
curl -X POST http://$ARGUMENTS/auto_tune \
  -H 'Content-Type: application/json' \
  -d '{"action":"home"}'
```

### Send all modules to a character
```bash
curl -X POST http://$ARGUMENTS/auto_tune \
  -H 'Content-Type: application/json' \
  -d '{"action":"goto_char","char_index":1}'
```

### Get current tuning positions for a character
```bash
curl http://$ARGUMENTS/tuning_status?char_index=1
```

### Nudge modules (positive = more steps, negative = fewer)
```bash
curl -X POST http://$ARGUMENTS/auto_tune \
  -H 'Content-Type: application/json' \
  -d '{"action":"adjust","modules":[3,17,42],"char_index":1,"delta":-5}'
```

## Workflow

1. First, verify connectivity by fetching `http://$ARGUMENTS/tuning_status?char_index=0`
2. Home all modules: POST `/auto_tune` with `{"action":"home"}` — wait 15 seconds
3. Start with character index 1 (A) — send `goto_char`
4. Ask the user to take a straight-on photo of the full display and provide it
5. Analyze the photo:
   - Map each visible module to its grid position (row 0 = top, left to right)
   - Identify modules showing the WRONG character (completely different letter)
   - Identify modules where the character is slightly misaligned (half-showing, split between two)
6. Apply corrections:
   - Wrong character entirely: nudge by ±15-25 steps
   - Slightly off (character visible but not centered): nudge by ±3-8 steps
   - If a module shows the NEXT character (overshot): use negative delta
   - If a module shows the PREVIOUS character (undershot): use positive delta
7. After adjusting, re-send `goto_char` for the same character
8. Ask for another photo to verify
9. When all 45 modules look correct, advance to the next character index
10. Repeat for all characters (or as many as the user wants to tune)

## Photo Analysis Tips

- The display has white/light characters on dark flaps
- Each module shows one character in a small rectangular window
- A correctly aligned character is fully visible and centered in the window
- A misaligned character may show parts of two adjacent characters (split flap visible)
- Color flaps (indices 57-63) will show solid colored panels instead of text
- Focus on getting letters and numbers right first (indices 1-36), then punctuation

## Important

- Always re-send `goto_char` after adjusting, so the modules move to their new positions
- Work through one character at a time — don't skip ahead
- Be conservative with adjustments — it's better to nudge twice than overshoot
- The user can stop at any time; all adjustments are saved to EEPROM immediately
