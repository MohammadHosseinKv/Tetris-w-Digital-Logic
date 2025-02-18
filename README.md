<h3 align="center">Tetris (Digital Circuits Project Implemented In Proteus) </h3>

## Introduction
This game is a digital block-stacking puzzle where the player's goal is to strategically guide and align falling blocks to complete rows within a limited time. The game is played on a 10x7 matrix, with blocks randomly generated in the top three rows. Players can control the position and orientation of these blocks using shift and rotation keys.

As rows are completed, they will flash for two seconds before being cleared, causing all blocks above them to shift downward. Players earn points for each cleared row, with the aim of reaching a total of three points to win the game. However, the game ends if:

- A block collides with a fixed block in the top three rows and cannot move out of the area.
- The timer reaches 99 seconds. 

> [!NOTE]
>
> This document explains the overall project, the idea behind its implementation, and the functionality of each section.
>

![project interface pic](resources/project_interface.png)

## Table of Contents

- [Gameplay Overview](#gameplay-overview)
  - [General Rules and Interface Setup](#general-rules-and-interface-setup)
  - [Starting the Game](#starting-the-game)
  - [Player Interactions](#player-interactions)
  - [Objective of the Game](#objective-of-the-game)
  
- [Implementation Idea](#implementation-idea)
  - [Before Game Starts](#before-game-starts)
    - [Blinking Student ID Digits on 7-Segment Displays](#blinking-student-id-digits-on-7-segment-displays)
    - [Light Show (Spiral Pattern)](#light-show-spiral-pattern)
  - [Game State Management](#game-state-management)
    - [Functionality of the States](#functionality-of-the-states)
    - [Game State Table](#game-state-table)
    - [Controlling Circuit Activities](#controlling-circuit-activities)
  - [Game End Condition](#game-end-condition)
    - [Win Condition](#win-condition)
    - [Loss Conditions](#loss-conditions)
    - [Summary of Logic](#summary-of-logic)
  - [Timer and Score](#timer-and-score)
    - [Game Timer](#game-timer)
    - [Player Score](#player-score)
  - [Generating and Placing a Random 3x3 Block in the Game](#generating-and-placing-a-random-3x3-block-in-the-game)
    - [Generating the Random Bits](#generating-the-random-bits)
    - [Using Random Bits to Determine the Block Shape](#using-random-bits-to-determine-the-block-shape)
    - [Using Random Bits to Determine Block Position](#using-random-bits-to-determine-block-position)
    - [Process Summary](#process-summary)
  - [Control Generated Block](#control-generated-block)
    - [Shifting Left and Right](#shifting-left-and-right)
    - [Preventing Invalid Shifts](#preventing-invalid-shifts)
    - [Rotation Implementation](#rotation-implementation)
    - [Center of Rotation](#center-of-rotation)
  - [Downward Shift of the Generated Block](#downward-shift-of-the-generated-block)
    - [Transferring Block Data to the Game Core](#transferring-block-data-to-the-game-core)
    - [Game Core Structure and Shift Register Configuration](#game-core-structure-and-shift-register-configuration)
    - [Functionality of Shift Registers in Rows 1-3](#functionality-of-shift-registers-in-rows-1-3)
    - [Downward Shift in Rows 4-10](#downward-shift-in-rows-4-10)
  - [Collision Handling & Block Locking Mechanism](#collision-handling--block-locking-mechanism)
    - [Step 1: Independent Behavior of Lights (Before Group Dependency)](#step-1-independent-behavior-of-lights-before-group-dependency)
    - [Step 2: Transitioning from Y (Moving) to B (Fixed)](#step-2-transitioning-from-y-moving-to-b-fixed)
    - [Step 3: Adding Group Behavior (Locking Entire Blocks Instead of Individual Lights)](#step-3-adding-group-behavior-locking-entire-blocks-instead-of-individual-lights)
    - [Step 4: Updating Collision Conditions for Block Groups](#step-4-updating-collision-conditions-for-block-groups)
    - [Step 5: Updating Load Conditions](#step-5-updating-load-conditions)
    - [Supplementary Information](#supplementary-information)
  - [Score Calculation and Row Clearing Mechanism](#score-calculation-and-row-clearing-mechanism)
    - [Step 1: Detecting and Blinking Full Rows Before Deletiion](#step-1-detecting-and-blinking-full-rows-before-deletiion)
    - [Step 2: Generating the Blinking Signal for Rows](#step-2-generating-the-blinking-signal-for-rows)
    - [Step 3: Controlling the Blinking Duration (2 Seconds Limit)](#step-3-controlling-the-blinking-duration-2-seconds-limit)
    - [Step 4: Define And Store Row Full Condition](#step-4-define-and-store-row-full-condition)
    - [Step 5: Deleting Full Rows & Shifting Upper Rows Down](#step-5-deleting-full-rows--shifting-upper-rows-down)
      - [5.1 Understanding the Row Deletion Process](#51-understanding-the-row-deletion-process)
      - [5.2 Handling Multiple Full Rows (Priority Encoder & Decoder Mechanism)](#52-handling-multiple-full-rows-priority-encoder--decoder-mechanism)
    - [Step 6: Adding Score After Row Deletion](#step-6-adding-score-after-row-deletion)
  - [Fullboard Condition](#fullboard-condition)
  - [Notes](#notes)
- [Installation](#installation)
- [Running the Game](#running-the-game)
- [Resources](#resources)
- [License](#license)
- [Contact](#contact)


## Gameplay Overview

### General Rules and Interface Setup
The game interface consists of an 10x7 grid of LED , four seven segment responsible for displaying time and score and five control buttons which are Start , Reset , Rotate , S_Right and S_Left.

In the interface, each LED can either be on or off. The LEDs in the first three rows, when lit, are red, while the LEDs in rows four to ten can be either yellow or blue.

Yellow LEDs indicate a moving block, whereas blue LEDs represent a fixed block. If any part of a moving block collides with the bottom of the grid or with a fixed block, the entire moving block becomes fixed at the point of collision.

#### End Condition:

**Winning Condition:** The player wins by successfully collecting 3 points.

**Losing Conditions:**
1. The player loses if a generated block collides with a fixed block while exiting the top 3 rows and cannot fully leave the area.
2. The player loses if the game time reaches 99 seconds.

### Starting the Game

After pressing the start button, the game begins, and the time and score values on the 7-segment displays reset to zero. One of the eight predefined 3x3 blocks is randomly generated and placed in a random position within the top three rows. (All eight predefined 3x3 blocks are displayed in the picture below.) 

![3x3_generate_block](resources/3x3_generate_block.png)

For the first three seconds, while the block is still in the top three rows, the player can use the shift and rotate buttons to change the block's position and orientation. Once the block completely moves out of the top three rows, a new block is generated, and this process continues until the game ends.

### Player Interactions

- **Reset:** Clicking this option resets the game and user interface to a ready state.

- **Start:** Clicking this option starts the game, resets the score and time displays to zero, and initiates the random generation of blocks. This button is only active when the game has been reset and is ready to start.

- **Rotate:** When this key is pressed, the block in the top three rows rotates counterclockwise.

- **S_Right:** When the key is pressed, the block in the top three rows shifts right, if possible.

- **S_Left:** When the key is pressed, the block in the top three rows shifts left, if possible.

### Objective of the Game

The objective of the game is to arrange the blocks so they align in a single row. When a row of LEDs is completed, all the LEDs in that row will blink for two seconds before turning off. After that, all the blocks above the completed row will shift downward, and the player will receive one point added to their total score. The player wins by accumulating 3 points.

# Implementation Idea

## Before Game Starts
After running the simulation and before pressing the start button, the last two digits of the student ID numbers of the two project group members will alternately blink on the four 7-segment displays.

At the same time, the LEDs on the game matrix will illuminate in a spiral pattern from top to bottom, creating an effect similar to a light show. Please see the video below for demonstration. This adds a visually engaging prelude to the game.

![before_game_starts_light_show](resources/before_game_starts.gif)

> [!NOTE]
>
> To simplify and enhance understanding, from now on we will  refer to 'LEDs' as 'lights'.
>

Our implementation approach for this section is clear and systematic:

### Blinking Student ID Digits on 7-Segment Displays

**BCD to 7-Segment Conversion:**

To display the digits of student IDs on four 7-segment displays, we connect a BCD-to-7-segment decoder to each display. The input to each decoder consists of the BCD representation of the corresponding digit from the student ID.

**Blinking Mechanism:**

The outputs of the decoders, which are connected to the 7-segment displays, pass through a tri-state buffer. The enable condition of the tri-state buffer is controlled by an AND gate that combines the following signals:

1. **Game Not Started:** This ensures that the blinking occurs only before the game begins.
2. **2 Hz Clock Signal:** This clock alternates the buffer's enable state every 0.5 seconds, toggling the display output on and off.

This setup creates a blinking effect on the digits of the 7-segment displays.

### Light Show (Spiral Pattern)

**Shift Registers for Light Control:**

We employ several shift registers to control the lights in a spiral pattern from top to bottom. The input to the serial shift registers is set to 1 to simulate sequentially lighting up each light.

**Operation:**

Before the game begins, the shift registers sequentially shift a '1' through the lights, creating the spiral lighting effect. Once all the lights are illuminated, the shift registers are reset, and the process repeats until the game starts.

**Condition for Operation:**

The shift registers remain active as long as the game has not started. Once the game begins, the enabling condition for the shift registers is invalidated, thus stopping the light show.

## Game State Management

To manage the state of the game, we utilize two variables: GameStartState and GameEndState. As their names suggest:

- **GameStartState = 1**: This indicates that the game has started, meaning the player has clicked the Start button.
- **GameEndState = 1**: This indicates that the game has ended, whether the player won or lost.

### Functionality of the States

**GameStartState**:
- When the Start button is pressed, a value of 1 is loaded into the register associated with the GameStartState output.
- This register is reset only when the Reset button is pressed.

**GameEndState**:
- The value of GameEndState is determined by the XOR operation applied to two variables: GameWon and GameLost. 
- These variables are explained in further detail in the section on [Game End Condition](#game-end-condition).

### Game State Table

| GameStartState | GameEndState | Description                                        |
|-----------------|---------------|----------------------------------------------------|
| 0               | X             | The game has not started.                         |
| 1               | 0             | The game has started but not yet ended.          |
| 1               | 1             | The game has started and has ended.              |

### Controlling Circuit Activities

Using these two variables (GameStartState and GameEndState), we can effectively control the operation of the circuits for each part of the game:

- **When GameStartState = 0**, pre-start behaviors such as a blinking display and light show are active.
- **When GameStartState = 1** and GameEndState = 0, the game logic, block movement, and player interactions are active.
- **When GameEndState = 1**, post-game behaviors, such as displaying results, can be triggered.

This modular approach ensures clear control over the different stages of the game.

## Game End Condition
To track whether the player has won or lost, we define two variables: **GameWon** and **GameLost**. These variables indicate the player's win or loss status and are stored in a register. Here’s how their values are managed:

### Reset Condition
Both variables are reset to 0 when the **Reset** button is pressed.

### Load Condition
- **GameWon** is set to 1 when the **WinGame** signal is activated.
- **GameLost** is set to 1 when the **LoseGame** signal is activated.

These signals are generated based on the game's win and loss conditions.

### Win Condition
The player wins if they collect 3 points.

The player's score is displayed on the interface using two 7-segment displays, representing the score as an 8-bit BCD (Binary-Coded Decimal) value.

A comparator circuit compares this 8-bit BCD value with **0000 0011** (the binary representation of 3). If the comparator output indicates "greater than or equal to 3," the **WinGame** signal is set to 1. This activates the **GameWon** variable, marking the game as won and ending it.

### Loss Conditions

A player loses the game if either of the following conditions is met:

**Collision in the Top 3 Rows (FullBoard Condition):**

If a newly generated 3x3 block collides with a fixed block and cannot completely move out of the top three rows, a variable named **FullBoard** is set to 1. 

When **FullBoard** equals 1, the **LoseGame** signal is triggered, which sets **GameLost** to 1 and ends the game. 

Details on how **FullBoard** is determined can be found in the [FullBoard Condition](#fullboard-condition) section.

### Timer Reaches 99 Seconds

The game timer is represented as an 8-bit BCD (Binary-Coded Decimal) value. To detect when the timer reaches 99 (**1001 1001 in BCD**) seconds, the following condition is checked:

If we let eight BCD Timer bits be T0...T7:
```math
(T0 \ \text{AND} \ \neg T1 \ \text{AND} \ \neg T2 \ \text{AND} \ T3) \ \text{AND} \ (T4 \ \text{AND} \ \neg T5 \ \text{AND} \ \neg T6 \ \text{AND} \ T7)
```

If this condition evaluates to true, the `LoseGame` signal is triggered, resulting in `GameLost` being set to 1 and the game ending.

### Combining Loss Conditions

The `LoseGame` signal is determined based on the following conditions:

```math
LoseGame = FullBoard \ \text{OR} \\ ( \ (T0 \ \text{AND} \ \neg T1 \ \text{AND} \ \neg T2 \ \text{AND} \ T3) \ \text{AND} \ (T4 \ \text{AND} \ \neg T5 \ \text{AND} \ \neg T6 \ \text{AND} \ T7) \ )

```

This means the game will be marked as lost if either the board is full or the timer has reached 99 seconds.

To ensure these loss conditions only apply after the game has started, we finalize the `LoseGame` condition by ANDing it with the `GameStartState`:

```math
LoseGame = FullBoard \ \text{OR} \\ ( \ (T0 \ \text{AND} \ \neg T1 \ \text{AND} \ \neg T2 \ \text{AND} \ T3) \ \text{AND} \ (T4 \ \text{AND} \ \neg T5 \ \text{AND} \ \neg T6 \ \text{AND} \ T7) \ )\\
 \ \text{AND} \ GameStartState
```

### Summary of Logic

| Condition                    | Signal Triggered      | Result                            |
|------------------------------|----------------------|-----------------------------------|
| Score >= 3                   | WinGame = 1         | GameWon = 1 (End Game)           |
| Collision in top 3 rows      | FullBoard = 1       | GameLost = 1 (End Game)          |
| Timer reaches 99 seconds     | Timer AND 1001 1001 | GameLost = 1 (End Game)          |

## Timer and Score

To display the game timer and player score, we use four 7-segment displays, each driven by a Binary-Coded Decimal (BCD) value, since humans naturally count in base-10. Below is a detailed implementation:

### Game Timer

The game timer counts the elapsed seconds and is displayed as two digits:

- **Units Place (0–9):** Controlled by Counter A.
- **Tens Place (0–9):** Controlled by Counter B.

**Implementation of Timer Counters**

**Counter A (Units):**

- **Clock Input:** A 1 Hz clock signal serves as the input.
- **Operation:** 
  - Counter A increments from 0 to 9.
  - When it reaches 9, it resets to 0 on the next clock edge.
  - When Counter A resets, Counter B increments by 1.

**Counter B (Tens):**

- **Clock Input:** Triggered whenever Counter A resets after reaching 9.
- **Operation:**
  - Counter B increments from 0 to 9.

**Output:**

The combined output of Counter A and Counter B forms an 8-bit BCD value representing the timer:

**Timer Output = Counter B (MSB) | Counter A (LSB)**

**Connection to 7-Segment Displays:**

The 8-bit BCD output is sent to BCD-to-7-segment decoders. To control when the timer is displayed, we use tri-state buffers with the following enable condition:

```math
GameStartState \ \text{AND} \ \neg GameEndState
```

This condition ensures the timer is displayed only when the game is active (i.e., when it has started but not yet ended).

### Player Score

The player score is displayed using a similar structure to the timer, with a few key differences:

**Clock Input for Counter A (Units):**

- Instead of a 1 Hz clock, Counter A utilizes a signal known as AddScore as its clock input.

**AddScore:**
- The AddScore signal becomes active (1) whenever the player successfully clears a complete row of lights.
- Further details about this signal are provided in the [Score Calculation and Row Clearing Mechanism](#score-calculation-and-row-clearing-mechanism) section.

**Operation:**
- Counter A increments each time it receives an AddScore pulse.
- When Counter A reaches a value of 9, it resets to 0 and increments Counter B, similar to how the timer counters operate.

**Output and Display:**
- The combined output of Counter A and Counter B forms the 8-bit BCD (Binary-Coded Decimal) score value.
- This output is sent to the BCD-to-7-segment decoders, which are controlled by tri-state buffers. The enable condition for these buffers is the same as that for the timer: 
```math
GameStartState \ \text{AND}  \ \neg GameEndState
```
This ensures a clear and logical display of the game timer and player score.

## Generating and Placing a Random 3x3 Block in the Game

To create and position a random 3x3 block on the game board, we utilize a Linear-Feedback Shift Register (LFSR) to generate 6 pseudo-random bits. These bits determine both the shape and position of the block.

### Generating the Random Bits

**LFSR for 6-Bit Output:**

The LFSR continuously generates a 6-bit pseudo-random sequence. When the block generation conditions are met, the current output of the LFSR is stored in a register to ensure stability for subsequent block generation.

**Block Generation Conditions:**

A block is generated only when the following conditions are met:

1. All lights in the top 3 rows are off (indicating that last generated 3x3 block has completely left this area or simply the game just started).
2. `GameStartState = 1` (the game has started).
3. `GameEndState = 0` (the game has not ended).

These conditions ensure that blocks are created only when the game is active and there is space to place a new block.

### Using Random Bits to Determine the Block Shape

**Block Shape Selection:**

The 3 most significant bits (MSBs) of the LFSR output are used as a selector for an 8-to-1 multiplexer. This multiplexer has 8 inputs, each representing one of the predefined 3x3 block shapes, as follows:
```math
S1: \ 010 \ 111 \ 010 \\
S2: \ 100 \ 100 \ 100 \\  
S3: \ 000 \ 001 \ 111 \\ 
S4: \ 000 \ 110 \ 011 \\  
S5: \ 000 \ 111 \ 100 \\  
S6: \ 000 \ 111 \ 010 \\ 
S7: \ 000 \ 101 \ 111 \\  
S8: \ 000 \ 010 \ 111 \\
---------------
\\
S1 =
\begin{matrix}
0 & 1 & 0 \\
1 & 1 & 1 \\
0 & 1 & 0 \\
\end{matrix}
\\
---------------
\\
S2 =
\begin{matrix}
1 & 0 & 0 \\
1 & 0 & 0 \\
1 & 0 & 0 \\
\end{matrix}
\\
---------------
\\
S3 =
\begin{matrix}
0 & 0 & 0 \\
0 & 0 & 1 \\
1 & 1 & 1 \\
\end{matrix}
\\
---------------
\\
S4 =
\begin{matrix}
0 & 0 & 0 \\
1 & 1 & 0 \\
0 & 1 & 1 \\
\end{matrix}
\\
---------------
\\
S5 =
\begin{matrix}
0 & 0 & 0 \\
1 & 1 & 1 \\
1 & 0 & 0 \\
\end{matrix}
\\
---------------
\\
S6 =
\begin{matrix}
0 & 0 & 0 \\
1 & 1 & 1 \\
0 & 1 & 0 \\
\end{matrix}
\\
---------------
\\
S7 =
\begin{matrix}
0 & 0 & 0 \\
1 & 0 & 1 \\
1 & 1 & 1 \\
\end{matrix}
\\
---------------
\\
S8 =
\begin{matrix}
0 & 0 & 0 \\
0 & 1 & 0 \\
1 & 1 & 1 \\
\end{matrix}
```
Each shape is represented by 9 bits (3 bits per row). The output of the multiplexer (9 bits) is stored in variables: `SHAPE0`, `SHAPE1`, ..., `SHAPE8`.

![3x3_generate_block](resources/3x3_generate_block.png)

### Using Random Bits to Determine Block Position

#### Block Positioning:
The three least significant bits (LSBs) of the Linear Feedback Shift Register (LFSR) output are utilized as inputs to a 3-to-8 decoder. This decoder determines which three adjacent columns on the game board the block will occupy.

#### Mapping Decoder Outputs to Positions:
Given that the game board consists of seven columns, the possible positions are as follows:

- **POS0-2**: Columns 0, 1, 2
- **POS1-3**: Columns 1, 2, 3
- **POS2-4**: Columns 2, 3, 4
- **POS3-5**: Columns 3, 4, 5
- **POS4-6**: Columns 4, 5, 6

To account for the eight decoder outputs, the outputs corresponding to positions 5, 6, and 7 are merged with the first five positions using OR gates. This ensures that only valid positions are generated, increasing the likelihood of placing blocks further away from the board edges.

#### Connecting to Buffers:
For each position (POS0-2, POS1-3, ..., POS4-6), a 9-bit tri-state buffer is employed. The enable condition for each buffer corresponds to its respective POS signal. The output of the buffer is then connected to the appropriate columns on the game board.

### Process Summary

**Shape Selection:**  
The top 3 bits of the LFSR output are used to select one of the predefined 8 shapes through an 8-to-1 multiplexer.

**Position Selection:**  
The bottom 3 bits of the LFSR output determine the position of the block using a 3-to-8 decoder.

**Block Placement:**  
The selected shape, which consists of 9 bits, is routed through the corresponding position buffer. This enables the shape to control the lights in the specified 3 columns.

## Control Generated Block

After a block is generated, the player has three seconds to control it using the following actions:

- **Shift Right (S_Right)**
- **Shift Left (S_Left)**
- **Rotate (Rotate)**

### Shifting Left and Right

Each of the top three rows is managed by a separate 7-bit shift register. When a block is created, its shape data is initially stored in temporary variables, which are then loaded into these shift registers. This setup allows for future modifications, such as rotations, without directly altering the display registers.

#### To Shift the Block:

The shift registers move their bits to the left or right. 

The shift operation is clocked by the OR combination of `S_Right`, `S_Left`, `Rotate`, and `CreationCondition` signals.

The shift direction is determined by the order of temporary light variables as shift register inputs:

- **If left-to-right:** S_Right decides shift direction.

- **If right-to-left:** S_Left decides shift direction.

### Preventing Invalid Shifts

To ensure that a block does not shift out of bounds:

- **Shift Right** is permitted only if there are no active bits in last column. This is verified using a NOR gate that checks the bits in 7th column.
- **Shift Left** is permitted only if there are no active bits in first column. This is verified using a NOR gate that checks the bits in 1st column .

Both conditions are combined using an AND operation with the shift signals before execution. Additionally, shifts are allowed only while the ControlCondition signal is active, which means they can only occur within the three-second control window. So The previous output will be combined with **ControlCondition** using an AND operation.

**Shift Registers Clock Signal:**
```math
(S_\text{Left} \ \text{AND} \ (R_{0,0} \ \text{NOR} \ R_{1,0} \ \text{NOR} \ R_{2,0}) \ \text{AND} \ ControlCondition)
\ \text{OR} \\
(S_{\text{Right}} \ \text{AND} \ (R_{0,6} \ \text{NOR} \ R_{1,6} \ \text{NOR} \ R_{2,6}) \ \text{AND} \ ControlCondition)
\ \text{OR} \\
(Rotate \ \text{AND} \ ControlCondition)
\ \text{OR} \\ CreationCondition
\\
```
```math
\\
\text{which [}R_{0,0}-R_{1,0}-R_{2,0}\text{] are red lights in the first column and }\\ \text{[}R_{0,6}-R_{1,6}-R_{2,6}\text{] are red lights in the last column.}
```


### Rotation Implementation

Blocks can rotate counterclockwise in four states:

1. 0° (Initial State)
2. 90° Counterclockwise
3. 180° Counterclockwise
4. 270° Counterclockwise

A 2-bit counter tracks the rotation state (from 0 to 3). Each time a Rotate signal is received, the counter increments, looping back to 0 after reaching 3.

A 2-to-4 decoder maps the counter value to one of four 9-bit tri-state buffers, each storing the block's shape in the correct rotated orientation. The rotations follow this pattern:

**Initial Shape (0°):**

```math
\begin{matrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9 \\
\end{matrix}
```

**After 90° Counterclockwise Rotation:**

```math
\begin{matrix}
3 & 6 & 9 \\
2 & 5 & 8 \\
1 & 4 & 7 \\
\end{matrix}
```

Each further 90° rotation applies the same transformation. The new rotated shape is stored in temporary variables before being loaded into shift registers, ensuring seamless display updates.

### Center of Rotation

The block must rotate around its current column position. A shift register stores the block's current position and updates it when the block shifts left or right. This approach ensures that the rotated shapes remain within their assigned three columns. The position register:

1. Loads the initial position upon block creation.
2. Shifts left or right when the player moves the block.
3. Prevents shifting beyond the edges of the board by applying conditions to POS4-6 (for right shifts) and POS0-2 (for left shifts).

Each rotation applies the updated shape data to the relevant three columns based on the position register.

**Finalizing Block Placement**

When the 3-second control period ends, the block becomes fixed in place.
The block’s shape is transferred to the game board registers, and the falling mechanism begins.
Then when the last generated block completely exits the three first rows, the next block generation process starts.

## Downward Shift of the Generated Block

Once the block is finalized after the 3-second control period, it must transition from the construction phase to the game board. This section explains how the block moves downward within the game grid.

### Transferring Block Data to the Game Core

After a block is created and the 3-second movement/rotation period expires, it becomes static. This is determined when both the ControlCondition and CreationCondition signals are deactivated (set to 0).

At this point, the values stored in the shift registers, which are responsible for constructing and controlling the block, are transferred to the Game Core section, managing the main grid of the game.

To facilitate this transfer:

1. The Enabler for the construction/control registers is set to ControlCondition.
2. The Enabler for the Game Core registers is set to NOT(ControlCondition).
3. The Load signal is activated simultaneously, ensuring proper data transfer.

### Game Core Structure and Shift Register Configuration

In the Game Core section, shift registers are used to store and move blocks downward. The configuration is as follows:

- **First 3 Rows**: Each column in the top three rows has its own 3-bit shift register. Since the board has 7 columns, this results in 7 shift registers handling the 21 lights of the top three rows.
  
- **Rows 4 to 10**: Each light (cell) in these rows has an individual shift register that stores Blue (B) and Yellow (Y) values, representing the block's color state. As the game board contains 49 lights from row 4 onward, this requires 49 shift registers.

### Total Shift Registers

- 7 shift registers for the top 3 rows
- 49 shift registers for rows 4-10
- **Total**: 56 shift registers

### Functionality of Shift Registers in Rows 1-3

The 7 shift registers managing the first three rows have a unique function:

Each register receives three input values (one from each row) from its respective column, along with a 0 (Ground) input at DL (Data Load).

With each shift operation:

- The first bit is replaced with 0.
- The remaining bits shift downward from row 1 to row 3.
- The outputs of these registers are connected to the corresponding column lights.

These shift registers operate with a 1Hz clock, meaning the block moves down once per second.

### Load Signal Conditions

The load signal is activated when the following conditions are met:

- GameStartState = 1 **(Game is running)**
- NOT(GameEndState) = 1 **(Game is not over)**
- ControlCondition = 1 **(Block is still in the control phase)**

However, the Output Enable (OE) is set to NOT(ControlCondition), which means the transferred data remains hidden until the block is finalized. 

**When ControlCondition switches to 0:**

- The Load signal is deactivated.
- The shift registers become active, and the block starts moving downward at each clock pulse.
- The third row's values transition into the fourth-row shift registers, ensuring there is no premature movement into row 4 before finalization.

![first three rows shift register implementation schematic](resources/first_three_rows%20shift_register_implementation_schematic.png)

### Downward Shift in Rows 4-10

From row 4 onward, each shift register behaves independently:

- The Yellow (Y) and Blue (B) values from the current row are loaded into the shift register of the row below.
- The shift occurs at each clock pulse, effectively moving the entire block downward one row per second.

This setup allows full control over each light, mimicking the behavior of a 2D shift register system.

![rows four to six shift register implementation schematic](resources/rows_four_to_six_shift_register_implementation_schematic.png)
> [!NOTE]
>
> In all of the schematic images above, the variables indices begin from 0.
>

### Next Steps: Collision Handling & Block Locking

Now that the downward shift mechanism is in place, the next step involves handling collisions and locking blocks in place by transferring Yellow (Y) values into Blue (B). This will be covered in the next section.

## Collision Handling & Block Locking Mechanism

In the game, a **moving block** can become a **fixed block** under two conditions:  

1. **It reaches the bottom** of the game grid.  
2. **It collides with an existing fixed block.**  

When either of these conditions is met, all parts of the block transition from **Yellow (Y) to Blue (B)**, indicating they are now part of the static game board.

---

### **Step 1: Independent Behavior of Lights (Before Group Dependency)**

To simplify the collision logic, we first assume that **each moving light is independent** and doesn't belong to a block group. That means each individual light will turn into a fixed block only when it **personally collides** with something below it.

### **Detecting Collision for Rows 3 to 9**
A collision is detected by **AND-ing the Yellow (Y) value of the current light** with the **Blue (B) value of the light directly below it**:

```math
\text{Collision Condition} = Y(i, j) \ \text{AND} \ B(i+1, j)
```

For example, in **row 4, column 0**, the collision condition is:

```math
Y(3,0) \ \text{AND} \ B(4,0)
```

If this condition evaluates to **1**, the light must become part of the static grid.

### **Handling Row 10 (Last Row)**
For the bottom row (row 10), there's no row below to check. Instead, we consider a light to have **collided when it simply exists**:

```math
Y(9,j) \ \text{AND} \ 1
```

Since AND with 1 does not change the value, this means **any Yellow light in row 10 automatically triggers a collision.**

![light_pis](resources/light_pis.PNG)

---

### **Step 2: Transitioning from Y (Moving) to B (Fixed)**

Once a collision is detected, we must shift the **Y value** into **B** on the next clock edge. This is done by using the **HOLD** input of the shift register.

- **HOLD = 1:** No shift occurs (light remains moving).
- **HOLD = 0:** The register shifts the **Y value into B**, making the light fixed.

To achieve this:

```math
\text{HOLD}(i, j) = \neg[\text{Collision Condition}(i, j)]
```

That means:

- **If there is no collision** → HOLD remains **1**, allowing the block to keep shifting down.
- **If a collision occurs** → HOLD becomes **0**, causing Y to be shifted into B, effectively "freezing" the light.

> [!NOTE]
>
> **In the implementation, we utilize the collision condition directly rather than its negation. However, we still negate the output of the Collision Condition using NAND instead of AND gate.** This means that we connect the collision condition variable directly to the shift register of the light, eliminating the need to include a NOT gate.
>

---

### **Step 3: Adding Group Behavior (Locking Entire Blocks Instead of Individual Lights)**

So far, only individual lights become fixed when they collide. However, **entire blocks need to freeze together**. Otherwise, a falling Tetris-like piece would break into smaller parts instead of landing as a unit.

### **Solution 1: Unique Block Identifiers (More General Approach)**
A logical approach would be to assign **a unique 2-bit identifier** to each block at creation. This way:

- Each moving light knows which block it belongs to.
- If **any part** of the block collides, all other lights with the **same ID** will also be locked.

The **ID system uses a 2-bit counter**, allowing **up to 4 distinct moving blocks** to exist at the same time (since new blocks only appear every few seconds).

If a block's **ID matches** that of a colliding light, **all its lights must become fixed** as well.

This is the **ideal method** for a more scalable implementation.

---

### **Solution 2: Simple Proximity Locking (Current Implementation)**
For a quick solution before the project deadline, we use **a less precise but functional method**:

- If **any light in a row collides**, all lights in:
  - The **same row**
  - **Two rows above**
  - **Two rows below**
  
  will **also be fixed**.

This works because **all generated blocks are 3×3** in size. Given the game mechanics, this assumption is valid since no other blocks will be in this range.

**Example:**
- If a light in **row 6** collides:
  - **Rows 4, 5, 6, 7, and 8** all become fixed.

While this method **isn't as elegant as the ID-based approach**, it works **within the constraints of the project**.

![pisrow](resources/pisrow.PNG)

---

### **Step 4: Updating Collision Conditions for Block Groups**
Previously, the **collision condition only checked individual lights**. Now, we extend it to apply to **entire block groups**.

For a light at **(i, j)**:

```math
\text{Block Collision Condition} = \neg B(i, j) \ \text{AND} \ [\text{Collision Occurred in Rows (i-2 to i+2)}]
```

If we use:
```math
PIS(i, j)
```
to denote **the independent collision condition** at (i, j). and
```math
PISROW(i-2 \text{ to } i+2)
```
to denote **if any collision happened in rows** (i-2 to i+2).

Then:
```math
\text{Final Collision Condition or IS}(i, j) = \neg B(i, j) \ \text{AND} \ PISROW(i-2 \text{ to } i+2)
```

This ensures that:
- **Only moving lights (not already fixed ones) are affected**.
- **If any light in a block collides, the entire block locks**.

![light_is](resources/light_is.PNG)

> [!NOTE]
>
> The naming of the final light collision condition as "IS" stands for "Inner Shift." This term is used because it describes how the shift register internally shifts the B and Y values, rather than employing a downward shifting mechanism on the game board. Similarly, "PIS" stands for "Partial Inner Shift."
>

---

### **Step 5: Updating Load Conditions**
Now that blocks are freezing properly, we must ensure **fixed blocks don't shift anymore**.

Each shift register should only load new data if:
1. **The current light isn’t already fixed (B = 0).**
2. **The light in the row above isn’t fixed (to prevent shifting downward).**
3. **The collision condition(IS) hasn’t been triggered (to prevent overriding the freeze action).**

Mathematically:

```math
\text{Load Condition}(i, j) = \neg B(i, j) \ \text{AND} \ \neg B(i-1, j) \ \text{AND} \ \neg IS(i, j)
```

**Example:**
For **row 5, column 2 (Light4-1)**:

```math
\neg B(4,1) \ \text{AND} \ \neg B(3,1) \ \text{AND} \ \neg IS(4,1)
```

For row **4**, there’s no light above it that can be static, so as an example for **row 4, column 6** we simplify:

```math
\neg B(3,5) \ \text{AND} \ \neg IS(3,5)
```

This ensures:
- Fixed blocks don’t shift.
- The row below doesn’t keep shifting into a fixed row.
- Blocks that **should freeze** don’t get overridden by a load operation.

![light_load](resources/light_load.PNG)

> [!NOTE]
>
> Since we already negated the 'IS' signal through the NAND gate, we do not negate it again during the load condition check.
>

---

### **Supplementary Information**

The variables **indices all begin from 0** thats why index of a light in row 4, column 3 would be L(3,2).

LXFORCE variables will be introduced in [Score Calculation and Row Clearing Mechanism](#score-calculation-and-row-clearing-mechanism) section which forces load condition to be true.

While the **simple row-freezing method** was used for quick implementation, a **block ID-based method** would be better for a long-term solution. The main **downside** of the current approach is that it **relies on block size assumptions** (3×3), hence limiting flexibility.


That said, given the constraints, this method **worked well for meeting the deadline** and maintaining correct gameplay behavior. 🚀

## Score Calculation and Row Clearing Mechanism

This section explains the **scoring system and row clearing mechanism** in the game, detailing how a **full row is detected, blinked, deleted, and how upper rows shift down accordingly.** It also explains the logic behind score incrementing after a row is cleared.  

### **Step 1: Detecting and Blinking Full Rows Before Deletiion**

A row is considered **full** when all its lights are **Blue (Fixed Blocks)**. This check applies to **rows 4 to 10**.  

- When a row is full, it begins **blinking for 2 seconds** before being deleted.
- During this **blinking phase**, all the blocks in that row toggle between **on and off states**.
- After **2 seconds**, the entire row is removed, and **all rows above it shift down by one row**, including fixed blocks.

To define the **blinking condition** for a row, we use the following approach:

1. **Check if the row is full:**  
   - **AND all 7 B values** (Fixed Blocks) of that row.  
   - If the result is `1`, the row is full, and blinking starts.

2. **Toggling the Output Enable (OE) signal** of shift registers:  
   - The OE input of the shift register toggles between `0` and `1`.  
   - This makes the **B values of that row toggle between `1` and undefined**, creating the blinking effect.

3. **Storing the Blink Condition:**  
   - Each row (4-10) requires a register to store its blinking condition.  
   - The register loads the blinking condition when the row is detected as full.  
   - After 2 seconds, this condition is cleared to allow row deletion.

Thus, **seven registers** (`BLINKROW3` to `BLINKROW9`) are defined, one for each row from 4 to 10.  

- **Load Input of Each Register:** AND of all **B values** in that row.  
- **Reset Input:** Activated **after 2 seconds of blinking**, which is explained later.  

![blinkrow](resources/blinkrow.PNG)

---

### **Step 2: Generating the Blinking Signal for Rows**  

The **Output Enable (OE) input** of the shift registers for each row is controlled using **ENROW3...ENROW9** signals.  

**Blinking Logic:**
1. **Use a 2 Hz clock pulse** (i.e., toggles every 0.5s).  
2. **NAND this clock signal with BLINKROWX** (the blinking flag for that row).  
3. **AND the result with ``GameStartState`` & NOT(``GameEndState``).**  

### **Why NAND?**
- When `BLINKROWX = 0`, the **NAND output is always `1`**, keeping the row visible.  
- When `BLINKROWX = 1`, the **NAND output toggles with the clock pulse**, causing a blinking effect.

### **Truth Table for ENROWX (Blinking Signal Generation)**  

| BLINKROWX | Clock Pulse (2Hz) | NAND Output | ENROWX |
|-----------|-------------------|-------------|--------|
| 0         | 0                 | 1           | 1      |
| 0         | 1                 | 1           | 1      |
| 1         | 0                 | 1           | 1      |
| 1         | 1                 | 0           | 0      |

- **When `BLINKROWX = 0`**, row is always visible.  
- **When `BLINKROWX = 1`**, row toggles visibility at 2 Hz, making it blink.

![ENROW](resources/ENRow.PNG)

---

### **Step 3: Controlling the Blinking Duration (2 Seconds Limit)**  

- The **blinking should last exactly 2 seconds**.
- We use a **3-bit counter** to track this duration.
- The counter starts counting **whenever any row starts blinking**.
- The counter **resets after 2 seconds**, stopping the blinking.

### **Counter Configuration**
- **Clock Enable (CE) Input:**  
  - **OR of all `BLINKROW3...BLINKROW9`** signals.  
  - If any row is blinking, the counter starts.

- **Counter Operation:**  
  - After **1 second**, a signal `StopBlink = 1`.  
  - After **2 seconds**, the counter resets (`StopBlink = 0`).  

### **StopBlink Calculation**
If `C0, C1, C2` are the **3-bit counter outputs** from least to most significant bit:

```math
\text{StopBlink} = C0 \ \text{AND} \ \neg{C1} \ \text{AND} \ \neg{C2}
```

- `StopBlink = 1` after **1 second** of blinking.  
- `StopBlink = 0` after **2 seconds** (counter resets).  

### **Counter Reset Logic**
- **Reset Input:** `StopBlink`
- When `StopBlink = 1`, on the next **counter clock cycle (1Hz)**, the counter loads `000`, effectively resetting.

---

### **Step 4: Define And Store Row Full Condition**  

Once `StopBlink = 1`, full row **must be deleted**.  

To store full row conditions, we define **seven new registers:**  
- `ROW3FULL ... ROW9FULL`  
- These store which row was **full before deletion**.

### **Register Behavior**
- **Load Input:** `BLINKROW3...BLINKROW9`
- **Load Condition:** `StopBlink = 1` (asynchronous)
- **Reset Condition:**  
  - OR of all `ROW3FULL...ROW9FULL`  
  - Reset synchronously with **game shift clock (1Hz).**  

This ensures **ROWXFULL stays `1` for exactly 1 game shift cycle** before resetting.

---

### **Step 5: Deleting Full Rows & Shifting Upper Rows Down**  

To **delete full rows** and shift down all rows **above the deleted row**, we define:  
- `L3FORCE ... L9FORCE`  
- These force the **loading of shift registers for affected rows**, ensuring they move down.

### **Force Load Logic**
- **When `ROWXFULL = 1`, all rows above `X` must shift down.**
- We define:

```math
\text{L9FORCE} = ROW9FULL
\\
L8FORCE = ROW8FULL \ \text{OR} \ ROW9FULL = ROW8FULL \ \text{OR} \ L9FORCE
\\
L7FORCE = ROW7FULL \ \text{OR} \ ... \ \text{OR} \ ROW9FULL = ROW7FULL \ \text{OR} \ L8FORCE
\\
L6FORCE = ROW6FULL \ \text{OR} \ ... \ \text{OR} \ ROW9FULL = ROW6FULL \ \text{OR} \ L7FORCE
\\
L5FORCE = ROW5FULL \ \text{OR} \ ... \ \text{OR} \ ROW9FULL = ROW5FULL \ \text{OR} \ L6FORCE
\\
L4FORCE = ROW4FULL \ \text{OR} \ ... \ \text{OR} \ ROW9FULL = ROW4FULL \ \text{OR} \ L5FORCE
\\
L3FORCE = ROW3FULL \ \text{OR} \ ... \ \text{OR} \ ROW9FULL = ROW3FULL \ \text{OR} \ L4FORCE

```

![LFORCE](resources/LForce.PNG)

### **5.1 Understanding the Row Deletion Process**
When a row (let’s say row X) is completely filled, all its lamps have **B = 1** (indicating they are active). The deletion process must:
- **Shift all rows above it downward by one row**, including both **fixed lamps** and moving ones.
- Ensure that the **previous state of row X does not remain** after shift. (**Overwrite lamps** in row X with the row above it.)

To achieve this, the **shift register loading condition** is carefully designed.

Each row in the system has a **shift register** responsible for storing its lamp states. The loading condition of each row's shift register determines **whether it updates based on the previous row or retains its current state**.

> [!NOTE]
>
> For additional info on how load condition is determined check out [Shift Registers Load Condition](#step-5-updating-load-conditions).
>
- Normally, a row loads its new values from the row above it **only if certain conditions are met**.
- However, when a row is **completely filled and needs to be deleted**, all of its lamps have **B = 1**.  
  - This causes the **shift register loading condition** of that row and the row below to be **false (0)**, meaning that row and the row below it will **not copy any values from the row above it** and will hold its current state.
  - As a result, all rows **above the full row shift down by one position**.
  - Since the **row below the full row is prevented from loading data**, it holds its current state instead of incorrectly inheriting values from the full row.
  - To finalize the deletion, **we force-load the shift registers of the full row** with the values from the row above it.
- This effectively **pushes all upper rows down** by one step.
- The full row is **overwritten and disappears**.

Thus, all shift registers for **the full row and rows above the full row** are **loaded unconditionally** on the next shift clock, **bypassing normal conditions** and ensuring a correct downward shift.

![row_deletion_and_add_score](resources/row_deletion_and_add_score.gif)

### **5.2 Handling Multiple Full Rows (Priority Encoder & Decoder Mechanism)**  

When multiple rows are full at the same time, the system must determine **which row to blink and delete first**. To enforce an **orderly deletion process**, we use a **priority encoder** and a **decoder**.

#### **Priority Encoder (8-to-3)**
- The priority encoder **detects the highest-numbered (deepest) full row** and assigns it the highest priority.
- It takes **seven inputs** (`ROW3FULL` to `ROW9FULL`), each representing whether a row is full.
- The **output is a 3-bit code**, identifying the highest-number full row.

#### **Decoder (3-to-8)**
- The **3-bit output** of the priority encoder is fed into a **3-to-8 decoder**.
- The decoder converts this back into **one-hot format**, activating only the corresponding row’s **BLINKROW** signal (`BLINKROW3` to `BLINKROW9`).
- The **highest-numbered full row** is now the only one that blinks, ensuring orderly deletion.

> [!NOTE] 
>
> The **first output of the decoder (least significant bit)** corresponds to **"no row is full"**. We ignore this signal.
The remaining **seven outputs** control the **blinking process** for `BLINKROW3` to `BLINKROW9`.
>

![blinkrow_rowxfull_addscore_schematic](resources/blinkrow_rowxfull_addscore_schematic.PNG)


---

### **Step 6: Adding Score After Row Deletion**  

To count the score:
- A **D Flip-Flop** with **rising edge trigger** and **asynchronous reset** is used.
- **Clock Input:** `NOR(ROW3FULL...ROW9FULL)`.  
- **Output:** `AddScore` (acts as clock for the [score counter](#player-score)).

#### **Reset Logic**
- **Reset when:**  
  - Game hasn't started (`GameStartState = 0`).  
  - Game is over (`GameEndState = 1`).  
  - `AddScore` is already `1`.
  ```math
  \text{DFF Reset} = AddScore \ \text{OR} \ ( GameStartState \ \text{NAND} \ \neg{GameEndState})  
  ```

#### **Score Incrementation**
- **At game start, the NOR output is `1`.**  
- **When a row gets deleted, `NOR(ROWXFULL) = 0` (falling edge)**.  
- **As soon as ROWXFULL resets (`0 → 1`), NOR becomes `1` again, creating a rising edge.**  
- This **rising edge** triggers `AddScore`, **increasing the player's score**.

![addscore](resources/addscore.PNG)

## Fullboard Condition

## Notes

## Installation

## Running the Game

## Resources

## License

## Contact


