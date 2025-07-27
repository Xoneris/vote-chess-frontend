<script lang="ts">

  import { Chess } from 'chess.js';

  const chess = new Chess()

  const handleSquareClick = (clickedSquare) => {

    if (moveableFields.some(field => field.includes("="))) {
      promotion = true
      return
    }

    if (moveableFields.some(field => field.includes(clickedSquare)) ) {
      makeMove(selectedSquare, clickedSquare)
    } else if (moveableFields.some(field => field.includes("O-O")) && ((clickedSquare === "g1" && chess.turn() === "w") || clickedSquare === "g8" && chess.turn() === "b")) {
      chess.move("0-0")
      chessBoard = fenToBoard(chess.fen())
      moveableFields = []
      selectedSquare = ""
    } else {
      moveableFields = chess.moves({square: clickedSquare})
      selectedSquare = clickedSquare
      console.log(moveableFields)
    }
  }

  const makeMove = (selectedField, targetField) => {
    chess.move({from: selectedField, to: targetField})
    chessBoard = fenToBoard(chess.fen())
    moveableFields = []
    selectedSquare = ""
  }

  const fenToArray = (fen:string) => {
    const rows = fen.split(' ')[0].split('/'); // take only board layout before spaces
    return rows.map(fenRowToArray);
  }
  
  const fenRowToArray = (fenRow:string) => {
    
    const result = [];
    for (const char of fenRow) {
      if (/[1-8]/.test(char)) {
        // It's a number - add that many empty squares (e.g., null or '')
        const emptyCount = parseInt(char, 10);
        for (let i = 0; i < emptyCount; i++) {
          result.push(null);  // or '' if you prefer
        }
      } else {
        // It's a piece, add as-is
        result.push(char);
      }
    }
    return result;
  }

  const fenToBoard = (fenString) => {

    const fenArray = fenToArray(fenString)
    const board = []

    for (let i=0 ; i<fenArray.length ; i++) {

      const fullRow = []

      let row:number
      switch (i) {
          case 0:
            row = 8
            break
          case 1:
            row = 7
            break
          case 2:
            row = 6
            break
          case 3:
            row = 5
            break
          case 4:
            row = 4
            break
          case 5:
            row = 3
            break
          case 6:
            row = 2
            break
          case 7:
            row = 1
            break
        }

      for (let j=0 ; j<fenArray[i].length ; j++) {

        const square = {
          name: "",
          type: "",
          color: "",
        }

        let col:string
        switch (j) {
        case 0:
          col = "a"
          break
        case 1:
          col = "b"
          break
        case 2:
          col = "c"
          break
        case 3:
          col = "d"
          break
        case 4:
          col = "e"
          break
        case 5:
          col = "f"
          break
        case 6:
          col = "g"
          break
        case 7:
          col = "h"
          break
      }
        
        square.name = col+row

        if (fenArray[i][j] !== null) {

          switch (fenArray[i][j]) {
            case "r":
              square.type = "r"
              square.color = "b"
              break
            case "n":
              square.type = "n"
              square.color = "b"
              break
            case "b":
              square.type = "b"
              square.color = "b"
              break
            case "k":
              square.type = "k"
              square.color = "b"
              break
            case "q":
              square.type = "q"
              square.color = "b"
              break
            case "p":
              square.type = "p"
              square.color = "b"
              break
            case "R":
              square.type = "r"
              square.color = "w"
              break
            case "N":
              square.type = "n"
              square.color = "w"
              break
            case "B":
              square.type = "b"
              square.color = "w"
              break
            case "K":
              square.type = "k"
              square.color = "w"
              break
            case "Q":
              square.type = "q"
              square.color = "w"
              break
            case "P":
              square.type = "p"
              square.color = "w"
              break
          }
        }
        fullRow.push(square)
      }
      board.push(fullRow)
    }
    return board
  }

let chessBoard = $state(fenToBoard(chess.fen()))
let selectedSquare = $state<string>("")
let moveableFields = $state<string[]>([]);

let promotion = $state<boolean>(false)

</script>

<main class="min-h-screen min-w-screen flex justify-center items-center bg-gray-500">
  {#if promotion}
    <div class="flex border">
      <img src={`chess-figures/${chess.turn()+"Q"}.svg`} alt=""/>
      <img src={`chess-figures/${chess.turn()+"R"}.svg`} alt=""/>
      <img src={`chess-figures/${chess.turn()+"B"}.svg`} alt=""/>
      <img src={`chess-figures/${chess.turn()+"N"}.svg`} alt=""/>
    </div>
  {/if}
  <div class="h-128 w-128">
    {#each chessBoard as row, rowIndex}
      <div class="flex">
        {#each row as square, squareIndex}
          <div 
            class={`
              w-16 h-16 flex justify-center items-center border
              ${
                selectedSquare === square.name 
                ? "bg-yellow-500"
                : moveableFields.some(field => field.includes(square.name)) 
                ? "bg-green-400" 
                : moveableFields.some(field => field.includes("O-O")) && ((square.name === "g1" && chess.turn() === "w") || square.name === "g8" && chess.turn() === "b")
                ? "bg-green-600"
                : square.type === "k" && chess.inCheck() && chess.turn() === square.color
                ? "bg-red-700"
                : (rowIndex + squareIndex) % 2 === 0 
                ? "bg-gray-700" 
                : "bg-white"
              }
            `}
            on:click={() => handleSquareClick(square.name)}  
          >
            <img 
              src={`chess-figures/${square.color+square.type.toUpperCase()}.svg`}
              alt={square.color + square.type}
            />
          </div>
        {/each}
      </div>
    {/each}
  </div>
</main>

