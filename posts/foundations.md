Before the AI pipelines and the client sites, there were the projects where I learned how computers actually work. None of these are flashy. All of them are why the later stuff was possible.

## A Sudoku solver in C

This is where I really learned pointers and dynamic memory. The solver models the puzzle with three structures (squares, boxes, and the whole grid) and attacks it two ways.

**Elimination.** When a number gets placed, every other square in that box is told "you can't be that number anymore." Any square left with only one possibility gets filled, which triggers more eliminations:

```c
int updateBox(Square*** sudoku, int row, int col)
{
    int number = sudoku[row][col]->number;   // number found
    Box* box = sudoku[row][col]->whichBox;

    for (int i = 0; i < 9; i++) {
        if (box->squares[i]->possible[number - 1] == 0) {
            box->squares[i]->solvable--;
            box->squares[i]->possible[number - 1] = 1;  // can't be that number
        }
    }
}
```

**Backtracking** for when elimination stalls: try a number, check the rules, move on, and back up when you hit a dead end.

## An x16 assembler and emulator

A whole 16-bit computer in C: an assembler that turns x16 assembly into object files, a disassembler, and an emulator that runs them, plus a few small games written in x16 assembly to prove it works. The heart of it is the fetch–decode–execute loop:

```c
uint16_t opcode = getopcode(instruction);
switch (opcode) {
case OP_ADD: {
    reg_t dst  = getdr(instruction);
    reg_t src1 = getsr1(instruction);
    if (getimmediate(instruction) == 0) {
        reg_t src2 = getsr2(instruction);
        x16_set(machine, dst, x16_reg(machine, src1) + x16_reg(machine, src2));
    } else {
        uint16_t imm = getbits(instruction, 0, 5);
        x16_set(machine, dst, x16_reg(machine, src1) + sign_extend(imm, 5));
    }
    update_cond(machine, dst);   // set the condition codes
    break;
}
```

After you've hand-decoded a branch instruction's condition bits, nothing feels like magic anymore. Which is the point.

## Bond Identifier

This is where my chemistry minor and computer science first collided. In organic chem lab you read IR spectra by hand: find a peak, note its shape and strength, look up which bond it could be. I turned that into a Java Swing app. You enter the peak's shape (narrow or broad), strength (1–3), and approximate frequency, and it tells you the bond.

Under the hood it's a **binary search tree** keyed on frequency ranges, with a hash map alongside for constant-time lookups. The hard part was overlapping frequency ranges in a BST, which forced me to rethink how keys compare. I also learned to separate the tree and node classes after first cramming everything into one class. Lesson learned.

## First sites

My first real web work was practical. A multi-page site for **Midland Urgent Care** (I was also running IT for the clinic at the time) covering services, insurance and wellness pages, in plain HTML, CSS and JS. And my first portfolio, which listed "Philanthropy ⭐⭐⭐⭐⭐" as a skill. Some things you grow out of. You're reading the replacement.

**Stack:** C, Make, Java, Swing, HTML/CSS/JS.
