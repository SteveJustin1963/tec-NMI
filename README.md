## Z interrupt handler for NMI processing in MINT

![image](https://github.com/user-attachments/assets/fc7ae6e0-36f8-4cf5-a8e8-9fcba6299b07)


This MINT2 implementation includes:

1. System Setup:
- Port initialization
- Interrupt control setup
- Key storage initialization

2. Key Features:
- Uses MINT's Z function for interrupt handling
- Continuous polling in main loop
- Interrupt-driven key capture
- Key display capability

3. Key Functions:
- I: Initialize system
- R: Read keypad
- M: Main processing loop
- Z: Interrupt handler
- P: Run program forever
- T: Test program

The main differences from the Z80 assembly:
1. MINT automatically handles register saving/restoration
2. NMI vector setup is handled by MINT
3. Simpler interrupt handling using Z function
4. Built-in port I/O functions


```
// 74C923 Keypad Handler
// Variables:
// p = port number (#10)
// k = last key read
// t = temporary storage
// i = interrupt control port (#FF)

// Initialize system
:I #10 p!          // Set port number for 74C923
   #FF i!          // Set interrupt control port
   #20 i /O        // Set interrupt mode (NMI enabled)
   [ 0 ] k! ;      // Initialize key storage

// Read keypad - returns value in k
:R p /I k! ;       // Read port to k

// Main processing function
:M R              // Read keypad
   k .            // Display key value
   100() ;        // Small delay

// Interrupt handler - called on NMI
:Z R              // Read keypad on interrupt
   k .            // Show key value
   /N ;           // New line

// Main program - initialize and run forever
:P I              // Initialize
   /U( M ) ;      // Loop forever

// Test program
:T I 1000( M ) ;  // Run for 1000 cycles
```


To use:
```mint
:P   // Run continuously
```
or
```mint
:T   // Run test program
```


## Enhanced version of the keypad handler.

New features added:

1. Debouncing:
- Counts stable readings before accepting key
- Configurable debounce threshold (current = 3)
- Maintains last key state for comparison

2. Key Buffer:
- 8-byte circular buffer implementation
- Buffer full/empty checking
- FIFO (First In, First Out) operation
- Buffer shifting on read

3. Special Key Processing:
- Handles Enter, Escape, Function keys
- Customizable actions for special keys
- Separate processing path for standard vs special keys

4. Scan Code Conversion:
- Maps 74C923 codes to ASCII
- Supports 0-9 and A-F keys
- Extensible mapping table
- Conversion happens before buffering


```
// 74C923 Keypad Handler
// Variables:
// p = port number (#10)
// k = last key read
// t = temporary storage
// i = interrupt control port (#FF)

// Initialize system
:I #10 p!          // Set port number for 74C923
   #FF i!          // Set interrupt control port
   #20 i /O        // Set interrupt mode (NMI enabled)
   [ 0 ] k! ;      // Initialize key storage

// Read keypad - returns value in k
:R p /I k! ;       // Read port to k

// Main processing function
:M R              // Read keypad
   k .            // Display key value
   100() ;        // Small delay

// Interrupt handler - called on NMI
:Z R              // Read keypad on interrupt
   k .            // Show key value
   /N ;           // New line

// Main program - initialize and run forever
:P I              // Initialize
   /U( M ) ;      // Loop forever

// Test program
:T I 1000( M ) ;  // Run for 1000 cycles
```


Usage Examples:
```mint
:X    // Run continuous program with all features
```
or
```mint
:T    // Run test program
```

Additional features:
- Buffer overflow protection
- Automatic key repeat prevention
- Clean separation of functions
- Easy to modify timing/thresholds

## 74C923 and 4x5 matrix plus 2 extra keys (shift and another key) specifically for the TEC-1.

1. Matrix Support:
- Handles 4x5 main matrix (20 keys)
- Supports SHIFT and FUNCTION keys
- Matches TEC-1 keyboard layout

2. Key Mapping:
- 0-9 mapped to ASCII 30-39
- A-F mapped to ASCII 41-46
- Special keys (., AD, GO, +) with specific mappings
- Shifted keys support
- Function key support

3. Special Functions:
- SHIFT key toggles shifted state
- FUNCTION key toggles function state
- Auto-clear shift/function after use

4. TEC-1 Specific:
- Matches port usage of TEC-1
- Compatible with interrupt system
- Proper key debouncing for mechanical keys

```
// TEC-1 74C923 Keypad Handler
// Matrix is 4x5 plus SHIFT and FUNCTION keys
// Matrix Layout:
// 7 8 9 A F
// 4 5 6 B E
// 1 2 3 C D
// 0 . AD GO +
// Plus SHIFT and FN keys

// Variables:
// p = port number (#10)
// k = last key read
// t = temporary storage
// s = shift state
// f = function key state
// b = key buffer array
// c = buffer count
// l = last key state
// m = scan code map array

// Scan code conversion map for standard and shifted keys
:M [ // Regular keys (0-19)
     #30 #31 #32 #33 #34 #35 #36 #37 #38 #39   // 0-9
     #41 #42 #43 #44 #45                        // A-F
     #2E #41 #47 #2B                            // . AD GO +
     // Shifted keys (20-39)
     #40 #41 #42 #43 #44 #45 #46 #47 #48 #49   // Shifted 0-9
     #51 #52 #53 #54 #55                        // Shifted A-F
     #3E #44 #53 #3D                            // Shifted . AD GO +
     0 ] m! ;                                   // End marker

// Initialize system
:I #10 p!           // Set port number for 74C923
   [ 0 0 0 0 0 0 0 0 ] b!  // Initialize 8-byte buffer
   0 s!             // Clear shift state
   0 f!             // Clear function state
   0 c!             // Clear buffer count
   0 l!             // Clear last key state
   M ;              // Initialize scan code map

// Add key to buffer if space available
:A c 8 < (          // If buffer not full
     k b c ? !      // Store key at current position
     c 1+ c!        // Increment count
   ) ;

// Get key from buffer
:G c 0 > (          // If buffer not empty
     b 0 ?          // Get first key
     c 1- c!        // Decrement count
     // Shift buffer left
     c ( /i b /i 1+ ? b /i ?! )
   ) ;

// Process special keys
:S k #14 = ( /T s! )     // Set shift state if SHIFT key
   k #15 = ( /T f! )     // Set function state if FN key
   k #14 < (             // If regular key
     s @ (               // If shift active
       k 20 + k!         // Add offset for shifted keys
       0 s!              // Clear shift state
     )
     f @ (               // If function active
       k #40 + k!        // Add offset for function keys
       0 f!              // Clear function state
     )
   ) ;

// Convert scan code to ASCII using map
:C m k ? ;

// Read keypad with debounce
:R p /I k!          // Read port to k
   k l = /F (       // If different from last state
     S              // Process special keys
     C              // Convert scan code
     A              // Add to buffer if valid
   )
   k l! ;           // Save last state

// Main processing function
:P R               // Read keypad
   c 0 > (         // If buffer has keys
     G .           // Get and show key
   )
   50() ;          // Delay for stability

// Interrupt handler - called on NMI
:Z R               // Read keypad on interrupt
   /N ;            // New line

// Main program - initialize and run forever
:X I               // Initialize
   /U( P ) ;       // Loop forever

// Test program
:T I 100( P ) ;    // Run for 100 cycles
```

## added key combinations, key repeat, and LED segment display support for the TEC-1.

Added features:

1. Key Combinations/Macros:
- Programmable macro sequences
- Auto-detection of macro patterns
- Built-in macros for common operations
- Example macros: "ADG", "REST"

2. Key Repeat:
- Configurable initial delay (20 cycles)
- Adjustable repeat rate (every 5 cycles)
- Repeat for held keys
- Auto-reset on key release

3. LED Display Support:
- 7-segment patterns for 0-F
- Direct port output for display
- Display current key pressed
- Demo mode to test display

4. New Functions:
- D: Display hexadecimal digit
- K: Check for macro matches
- R: Handle key repeat
- Y: Display test sequence

```
// Enhanced TEC-1 74C923 Keypad Handler
// Variables:
// p = port number (#10) for keypad
// d = port number (#00) for display
// k = last key read
// t = temporary storage
// s = shift state
// f = function state
// b = key buffer array
// c = buffer count
// l = last key state
// m = scan code map array
// r = repeat counter
// h = key hold time
// n = macro buffer
// e = segment patterns

// 7-segment patterns for 0-F
:E [ #3F #06 #5B #4F #66 #6D #7D #07 
     #7F #6F #77 #7C #39 #5E #79 #71 ] e! ;

// Macro definitions (each macro is: count, sequence of keys)
:M [ 3 #41 #44 #47 0    // Macro 1: "ADG" - Auto address+go
     4 #52 #45 #53 #54 0 // Macro 2: "REST" - Reset command
     0 ] n! ;            // End of macros

// Initialize system
:I #10 p!           // Set keypad port
   #00 d!           // Set display port
   [ 0 0 0 0 0 0 0 0 ] b!  // Initialize key buffer
   0 s! 0 f!        // Clear states
   0 c! 0 l!        // Clear counters
   0 r! 0 h!        // Clear repeat vars
   E ;              // Initialize display patterns

// Display value in leftmost digit
:D e + ? d /O ;     // Look up pattern and output

// Check for macro match
:K n! // Start at macro buffer
   /U (
     n ? 0 = /W     // Exit if end marker
     n ? t!         // Get macro length
     /T v!          // Set valid flag
     t ( // Check each key
       n /i 1+ ? 
       b /i ? = /F ( 
         /F v!      // Invalid if mismatch
       )
     )
     v @ ( // If valid macro
       t ( // Output macro sequence
         n /i 1+ ? A // Add to buffer
       )
       c t - c!     // Remove matched keys
     )
     // Skip to next macro
     n t + 2 + n!
   ) ;

// Key repeat handler
:R k l = (          // If same key
     h 1+ h!        // Increment hold time
     h 20 > (       // If held long enough
       r 1+ r!      // Increment repeat count
       r 5 > (      // If repeat delay passed
         k A        // Add key to buffer
         0 r!       // Reset repeat counter
       )
     )
   ) /E (
     0 h!          // Reset hold time
     0 r!          // Reset repeat counter
   ) ;

// Process keypad with all features
:P p /I k!          // Read keypad
   R               // Handle key repeat
   k l = /F (      // If new key
     S             // Process special keys
     C             // Convert scan code
     A             // Add to buffer
   )
   k l!            // Save last state
   K               // Check for macros
   c 0 > (         // If buffer has keys
     G "           // Get key and duplicate
     D             // Show on display
     .             // Print to terminal
   )
   50() ;          // Delay for stability

// Main program - initialize and run forever
:X I /U( P ) ;

// Test program
:T I 100( P ) ;

// Demo showing all digits
:Y I 16 (          // Show 0-F
     /i D          // Display value
     500()         // Delay
   ) ;
```


Example usage:
```mint
:X    // Run full program
:T    // Test program
:Y    // Display test
```
