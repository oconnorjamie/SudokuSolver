# SudokuSolver

This is a simulated annealing optimization algorithm implementation designed to solve even the hardest Sudoku boards.



<img src="https://github.com/user-attachments/assets/60c6bc25-4464-441c-a825-528dec4a123f" width="480">

## Features

The algorithm includes several optimizations and modifications:

- An initial temperature equivalent to the standard deviation of the cost function among thousands of cases.
- A stuck counter which increments the temperature to avoid local minima.
- If the temperature is too low for a multiplicative increment to make a difference, the initial amount is re-added.
- The algorithm does not give up until the Sudoku board is solved.

Iterations for easy boards average around 2,000. The hardest boards can reach iteration counts of over 200,000.



<img src="https://github.com/user-attachments/assets/3e689243-961b-40d1-b18a-89d5f0eef0ff" width="680">

## Example Cost Count Over Iterations 



<img src="https://github.com/user-attachments/assets/a7971e9e-c92b-40f0-8736-797dc48b0b78" width="700" height="1000">


Thank you for Reading. 18-07-2024
MIT License ~ Jamie O Connor
