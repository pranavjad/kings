<script>
import "../app.css";
import King from "../lib/king.svelte";
import X from "../lib/x.svelte";

const EMPTY = 0;
const MARKED = 1;
const KING = 2;

let grid_dim = 8;
let cur_num_queens = 0;
let game_over = false;

let colorRegions = [];
let boardState = [];

let getColorForRegion = (index) => {
    const colors = [
        '#FFADAD', '#FFD6A5', '#FDFFB6', '#CAFFBF', 
        '#9BF6FF', '#A0C4FF', '#BDB2FF', '#FFC6FF'
    ];
    return colors[index % colors.length];
}

let generateCells = () => {
    let coords = [];
    for (let i = 0; i < grid_dim; i++) {
        for (let j = 0; j < grid_dim; j++) {
            coords.push({row: i, col: j});
        }
    }
    return coords;
}

let handleCellClick = (cell) => {
    if (game_over) {
        return;
    }
    // 0 = empty, 1 = x, 2 = king
    let row = cell.row;
    let col = cell.col;
    let currentState = boardState[row][col];
    let color = colorRegions[row][col];
    console.log(color, getColorForRegion(color));

    // validate the move if we are placing a queen
    if (currentState == MARKED && !validateMove(cell)) {
        boardState[row][col] = EMPTY;
    }
    else {
        boardState[row][col] = (boardState[row][col] + 1) % 3;
        if (boardState[row][col] === KING) {
            cur_num_queens++;
        }
        else if (boardState[row][col] === EMPTY) {
            cur_num_queens--;
        }
        if (cur_num_queens === grid_dim) {
            game_over = true;
            setTimeout(() => {
                alert("You win!");
                generatePuzzle();
            }, 1000);
        }
    }
    console.log(cur_num_queens)
    boardState = [...boardState];
}

let validateMove = (cell) => {
    let row = cell.row;
    let col = cell.col;

    // check if row is empty
    for (let i = 0; i < grid_dim; i++) {
        if (boardState[cell.row][i] === KING) {
            return false;
        }
    }

    // check if col is empty
    for (let i = 0; i < grid_dim; i++) {
        if (boardState[i][cell.col] === KING) {
            return false;
        }
    }

    // check if the 8 neighboring cells are empty
    let neighbors = [
        {row: row - 1, col: col},
        {row: row + 1, col: col},
        {row: row, col: col - 1},
        {row: row, col: col + 1},
        {row: row - 1, col: col - 1},
        {row: row - 1, col: col + 1},
        {row: row + 1, col: col - 1},
        {row: row + 1, col: col + 1},
    ];

    for (let neighbor of neighbors) {
        if (neighbor.row >= 0 && neighbor.row < grid_dim && neighbor.col >= 0 && neighbor.col < grid_dim) {
            if (boardState[neighbor.row][neighbor.col] === KING) {
                return false;
            }
        }
    }

    // check if the color region has no other queens
    for (let i = 0; i < grid_dim; i++) {
        for (let j = 0; j < grid_dim; j++) {
            if (colorRegions[i][j] === colorRegions[row][col] && boardState[i][j] === KING) {
                return false;
            }
        }
    }

    return true;
}

// Generate a valid configuration of queens
let generateValidQueens = () => {
    initBoardState();
    let queens = [];
    
    let isValid = (row, col) => {
        for (let queen of queens) {
            if (queen.row === row || queen.col === col || 
                Math.abs(queen.row - row) === Math.abs(queen.col - col) ||
                Math.abs(queen.row - row) <= 1 && Math.abs(queen.col - col) <= 1) {
                return false;
            }
        }
        return true;
    }
    
    let placeQueen = (row) => {
        if (row === grid_dim) {
            return true;
        }
        for (let col = 0; col < grid_dim; col++) {
            if (isValid(row, col)) {
                queens.push({row, col});
                boardState[row][col] = KING;
                if (placeQueen(row + 1)) {
                    return true;
                }
                queens.pop();
                boardState[row][col] = EMPTY;
            }
        }
        return false;
    }
    
    placeQueen(0);
    return queens;
}

let generateInterestingRegions = (queens) => {
    let regions = Array(grid_dim).fill(0).map(() => Array(grid_dim).fill(null));
    
    // Assign initial regions to queens
    queens.forEach((queen, index) => {
        regions[queen.row][queen.col] = index;
    });

    // Define directions for growth (including diagonals)
    const directions = [
        [-1, 0], 
        [0, -1],           [0, 1],
        [1, 0]
    ];

    // Function to get valid neighboring cells
    const getNeighbors = (row, col) => {
        return directions
            .map(([dx, dy]) => [row + dx, col + dy])
            .filter(([r, c]) => r >= 0 && r < grid_dim && c >= 0 && c < grid_dim);
    };

    // Growth phase
    let unassigned = grid_dim * grid_dim - queens.length;
    while (unassigned > 0) {
        console.log(unassigned);
        for (let i = 0; i < grid_dim; i++) {
            for (let j = 0; j < grid_dim; j++) {
                if (regions[i][j] !== null) {
                    let neighbors = getNeighbors(i, j);
                    for (let [r, c] of neighbors) {
                        if (regions[r][c] === null && Math.random() < 0.8) { // Adjust probability for more interesting shapes
                            regions[r][c] = regions[i][j];
                            unassigned--;
                        }
                    }
                }
            }
        }
    }

    // // Fill any remaining unassigned cells
    // for (let i = 0; i < grid_dim; i++) {
    //     for (let j = 0; j < grid_dim; j++) {
    //         if (regions[i][j] === null) {
    //             console.log("unfilled");
    //             let neighbors = getNeighbors(i, j);
    //             let assignedNeighbor = neighbors.find(([r, c]) => regions[r][c] !== null);
    //             if (assignedNeighbor) {
    //                 regions[i][j] = regions[assignedNeighbor[0]][assignedNeighbor[1]];
    //             } else {
    //                 regions[i][j] = Math.floor(Math.random() * queens.length);
    //             }
    //         }
    //     }
    // }

    return regions;
};

let generateVoronoiLikeRegions = (queens) => {
    let regions = Array(grid_dim).fill(0).map(() => Array(grid_dim).fill(null));
    let queue = [];

    // Initialize the queue with all queen positions
    queens.forEach((queen, index) => {
        regions[queen.row][queen.col] = index;
        queue.push({row: queen.row, col: queen.col, region: index, distance: 0});
    });

    // Directions for adjacent cells (including diagonals)
    const directions = [
        [-1, -1], [-1, 0], [-1, 1],
        [0, -1],           [0, 1],
        [1, -1],  [1, 0],  [1, 1]
    ];

    while (queue.length > 0) {
        let {row, col, region, distance} = queue.shift();
        let randomIndex = Math.floor(Math.random() * directions.length);
        let randomDirection = directions[randomIndex];
        for (let [dx, dy] of directions) {
            let newRow = row + dx;
            let newCol = col + dy;

            if (newRow >= 0 && newRow < grid_dim && newCol >= 0 && newCol < grid_dim) {
                if (regions[newRow][newCol] === null) {
                    regions[newRow][newCol] = region;
                    queue.push({row: newRow, col: newCol, region: region, distance: distance + 1});
                }
            }
        }
        queue.push({row, col, region, distance});
    }

    return regions;
}

let initBoardState = () => {
    boardState = Array(grid_dim).fill(0).map(() => Array(grid_dim).fill(0));
    cur_num_queens = 0;
}



let generatePuzzle = () => {
    game_over = false;
    initBoardState();
    const queens = generateValidQueens();
    colorRegions = generateInterestingRegions(queens);
    initBoardState();
}

generatePuzzle();

</script>
<style>

</style>
<div class="w-screen h-screen flex flex-col items-center justify-center">
    <div
        class="board outline outline-2 w-[80vmin] max-w-[500px] aspect-square grid grid-rows-8 grid-cols-8"
        style="grid-template-columns: repeat({grid_dim}, 1fr); grid-template-rows: repeat({grid_dim}, 1fr);"
    >
        {#each generateCells() as cell}
            <button on:click={() => handleCellClick(cell)}
                class="cell bg-gray-200 border border-gray-300 p-3"
                style="background-color: {getColorForRegion(colorRegions[cell.row][cell.col])};"
            >
                {#if boardState[cell.row][cell.col] === MARKED}
                    <X />
                {:else if boardState[cell.row][cell.col] === KING}
                    <King />
                {/if}
            </button>
        {/each}
    </div>
</div>

