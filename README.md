# SudokuSolver

This project implements a simulated annealing optimization algorithm to solve even the hardest Sudoku boards.

<img src="https://github.com/user-attachments/assets/60c6bc25-4464-441c-a825-528dec4a123f" alt="Sudoku Solver" width="680">


The SudokuSolverG.ipynb will give a visual representation of the board using colors to show the progress of cost over iterations, finishing with a color green to show a finished board:

<img src="https://github.com/user-attachments/assets/505757a2-44e9-49fa-8c90-823597ff7a76" alt="Sudoku Solver" width="680">

## Features

The algorithm includes several optimizations and modifications:
- An initial temperature equivalent to the standard deviation of the cost function among thousands of cases.
- A stuck counter which increments the temperature to avoid local minima.
- If the temperature is too low for a multiplicative increment to make a difference, the initial amount is re-added.
- The algorithm does not give up until the Sudoku board is solved.

Iterations for easy boards average around 2,000. The hardest boards can reach iteration counts of over 200,000.

<img src="https://github.com/user-attachments/assets/3e689243-961b-40d1-b18a-89d5f0eef0ff" alt="Sudoku Solver Example" width="680">

## How It Works

### GUI with Tkinter

The Tkinter library is used to create a graphical user interface (GUI) to visualize the Sudoku board. The `MyApp` class is defined to handle the creation and update of the GUI.

```python
class MyApp:
    def __init__(self, root):
        self.root = root
        self.root.title("9x9 Grid of Buttons")
        
        self.buttons = []
        self.create_grid()
    
    def create_grid(self):
        for i in range(9):
            row = []
            for j in range(9):
                if j in [3, 6]:
                    label = ttk.Label(self.root, text="|", width=2)
                    label.grid(row=i, column=j*2-1, padx=0, pady=0)
                button = ttk.Button(self.root, text="", command=lambda i=i, j=j: self.on_button_click(i, j))
                button.grid(row=i, column=j*2, padx=1, pady=1, sticky="nsew")
                button.config(width=3)
                row.append(button)
            self.buttons.append(row)
            if i in [2, 5]:
                separator = ttk.Label(self.root, text="---------------------")
                separator.grid(row=i*2+1, column=0, columnspan=17)
                
        for i in range(9):
            self.root.grid_rowconfigure(i, weight=1)
            self.root.grid_columnconfigure(i*2, weight=1)
            
    def on_button_click(self, i, j):
        print(f"Button {i},{j} clicked")
    
    def update_grid(self, data):
        lines = data.strip().split("\n")
        for i in range(9):
            for j in range(9):
                self.buttons[i][j].config(text=lines[i][j])
```
### Fixing Sudoku Values
Marks the fixed values in the Sudoku board.

```python

def fixSudokuValues(sudoku_str):
    fixed_sudoku = list(sudoku_str.replace('\n', ''))
    for i in range(len(fixed_sudoku)):
        if fixed_sudoku[i] != '0':
            fixed_sudoku[i] = '1'
        else:
            fixed_sudoku[i] = '0'
    return ''.join(fixed_sudoku)
```
## Next Generation
Generates the next state by swapping two random values within the same subgrid.

```python

def nextGen(sudoku_str, fixed_sudoku_str):
    def get_subgrid_indices(subgrid_number):
        row_start = (subgrid_number // 3) * 3
        col_start = (subgrid_number % 3) * 3
        indices = [(row_start + i) * 9 + (col_start + j) for i in range(3) for j in range(3)]
        return indices
    
    sudoku = list(sudoku_str.replace('\n', ''))
    fixed_sudoku = list(fixed_sudoku_str.replace('\n', ''))
    subgrid_number = random.randint(0, 8)
    indices = get_subgrid_indices(subgrid_number)
    
    non_fixed_indices = [idx for idx in indices if fixed_sudoku[idx] == '0']
    
    if len(non_fixed_indices) < 2:
        return '\n'.join([sudoku_str[i:i+9] for i in range(0, 81, 9)])
    
    idx1, idx2 = random.sample(non_fixed_indices, 2)
    sudoku[idx1], sudoku[idx2] = sudoku[idx2], sudoku[idx1]
    
    result = ''.join(sudoku)
    return '\n'.join([result[i:i+9] for i in range(0, 81, 9)])
```
### Calculating Cost
Computes the cost of the current Sudoku board state.

```python

def calculatingCost(board):
    board = board.replace('\n', '')
    total = 0
    
    for i in range(9):
        row_count = [0] * 10
        col_count = [0] * 10
        for j in range(9):
            row_num = int(board[i * 9 + j])
            col_num = int(board[j * 9 + i])
            if row_num != 0:
                row_count[row_num] += 1
            if col_num != 0:
                col_count[col_num] += 1
        
        total += sum(x - 1 for x in row_count if x > 1)
        total += sum(x - 1 for x in col_count if x > 1)
    
    return total
```
### Simulated Annealing
The main loop of the algorithm, which iteratively improves the solution.

```python

def simulatedAnnealing(sudoku_str, initial_temp, cooling_rate, max_iterations):
    current_sudoku = initialStateGen(sudoku_str)
    fixed_sudoku = fixSudokuValues(sudoku_str)
    current_cost = calculatingCost(current_sudoku)
    temperature = initial_temp
    
    best_sudoku = current_sudoku
    best_cost = current_cost

    costs = [current_cost]
    iteration = 0
    stuck_count = 0

    while current_cost != 0 and iteration < max_iterations:
        new_sudoku = nextGen(current_sudoku, fixed_sudoku)
        new_cost = calculatingCost(new_sudoku)
        
        cost_diff = new_cost - current_cost
        acceptance_probability = math.exp(-cost_diff / temperature) if cost_diff > 0 else 1

        if random.random() < acceptance_probability:
            current_sudoku = new_sudoku
            current_cost = new_cost

        if current_cost < best_cost:
            best_sudoku = current_sudoku
            best_cost = current_cost
        
        temperature *= cooling_rate
        costs.append(current_cost)
        iteration += 1

        if current_cost >= best_cost:
            stuck_count += 1
        else:
            stuck_count = 0

        if stuck_count > 100:
            temperature *= 2
            stuck_count = 0

        # Update GUI every 1000 iterations
        if iteration % 1000 == 0:
            app.update_grid(current_sudoku)
            print(f"Iteration {iteration}, Current Cost: {current_cost}")

    return best_sudoku, best_cost, costs
```

### Running the Code
The following code initializes the Tkinter GUI and runs the simulated annealing algorithm.

```python
def start_tkinter_app():
    global app
    root = tk.Tk()
    app = MyApp(root)
    root.mainloop()

gui_thread = threading.Thread(target=start_tkinter_app)
gui_thread.start()

# Parameters for the simulated annealing
initial_temp = 3.5  # Increased initial temperature
cooling_rate = 0.99  # Slower cooling rate
max_iterations = 100000  # Increased number of iterations

# Wait a bit for the GUI to be ready
time.sleep(1)

# Update the GUI inside the annealing function
best_sudoku, best_cost, costs = simulatedAnnealing(sudokuorg, initial_temp, cooling_rate, max_iterations)

print("Best Sudoku State:\n", best_sudoku)
print("Best Cost:", best_cost)

# Plotting the cost reduction over iterations
plt.plot(costs)
plt.xlabel('Iteration')
plt.ylabel('Cost')
plt.title('Cost Reduction Over Iterations')
plt.show()
```
## Example Cost Count
```python
Iteration 231000, Current Cost: 7
Iteration 232000, Current Cost: 6
Iteration 233000, Current Cost: 6
Iteration 234000, Current Cost: 4
Iteration 235000, Current Cost: 4
Iteration 236000, Current Cost: 4
Iteration 237000, Current Cost: 4
Iteration 238000, Current Cost: 4
Iteration 239000, Current Cost: 4
Iteration 240000, Current Cost: 4
Iteration 241000, Current Cost: 6
Iteration 242000, Current Cost: 4
Iteration 243000, Current Cost: 2
Iteration 244000, Current Cost: 2
Iteration 245000, Current Cost: 2
Iteration 246000, Current Cost: 2
Iteration 247000, Current Cost: 2
Iteration 248000, Current Cost: 2
Iteration 249000, Current Cost: 2
Iteration 250000, Current Cost: 2
Iteration 251000, Current Cost: 10
Iteration 252000, Current Cost: 8
Iteration 253000, Current Cost: 8
Iteration 254000, Current Cost: 8
Iteration 255000, Current Cost: 8
Iteration 256000, Current Cost: 8
Iteration 257000, Current Cost: 8
Iteration 258000, Current Cost: 8
Iteration 259000, Current Cost: 8
Iteration 260000, Current Cost: 8
Iteration 261000, Current Cost: 6
Iteration 262000, Current Cost: 2
Iteration 263000, Current Cost: 2
Iteration 264000, Current Cost: 2
Best Sudoku State: 
625197834
781423965
349586217
516379428
498251376
273864591
867935142
132748659
954612783
Best Cost: 0
```


Thank you for Reading. 18-07-2024
MIT License ~ Jamie O Connor
