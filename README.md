##  MINT version 
- will use the Z interrupt handler for NMI processing.

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

##
