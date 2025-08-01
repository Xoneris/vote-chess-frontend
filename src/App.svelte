<script lang="ts">
  import { Chess } from 'chess.js';
  import { onDestroy, onMount } from 'svelte';
  import type { Square, Color, PieceSymbol } from 'chess.js';

  const SIMULATED_FEN_FROM_SERVER = "rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1"
  // const SIMULATED_FEN_FROM_SERVER = "1p6/P7/8/8/8/8/8/7k w - - 0 1"
  const SIMULATED_LAST_MOVE_FROM_SERVER = null

  const chess = new Chess(SIMULATED_FEN_FROM_SERVER, { skipValidation : true })
  const currentPlayer:"w"|"b" = chess.turn() 

  const playerChosenColor = localStorage.getItem("Chosen-Color")

  const YOU = playerChosenColor === "w" ? "w" : "b"
  const OPPONENT = playerChosenColor === "w" ? "b" : "w"


  const handleSquareClick = (clickedSquare:string) => {

    if (chosenColor !== currentPlayer || hasVoted) {
      return
    }

    // Promotion move
    if (moveableFields.some(field => field.includes("="))) {
      // promotion = true
      confirmPromotionModal = true
      targetSquare = clickedSquare
      return
    }

    // Regular move
    if (moveableFields.some(field => field.includes(clickedSquare)) ) {
      confirmMoveModal = true
      targetSquare = clickedSquare
      // makeMove(selectedSquare, clickedSquare)
      return
    } 
    
    // Kingside castling
    if (moveableFields.some(field => field === "O-O") && ((clickedSquare === "g1" && chess.turn() === "w") || clickedSquare === "g8" && chess.turn() === "b")) {
      chess.move("0-0")
      chessBoard = fenToBoard(chess.fen())
      moveableFields = []
      selectedSquare = ""
      return
    } 
    
    // Queenside castling
    if (moveableFields.some(field => field === "O-O-O") && ((clickedSquare === "c1" && chess.turn() === "w") || clickedSquare === "c8" && chess.turn() === "b")) {
      chess.move("0-0-0")
      chessBoard = fenToBoard(chess.fen())
      moveableFields = []
      selectedSquare = ""
      return
    } 

    moveableFields = chess.moves({square: clickedSquare as Square})
    selectedSquare = clickedSquare
    console.log(moveableFields)
    
  }

  const makeMove = (selectedField:string, targetField:string) => {
    chess.move({from: selectedField, to: targetField})
    chessBoard = fenToBoard(chess.fen())
    moveableFields = []
    selectedSquare = ""
  }

  const handlePromotion = (toPromoteColor:Color, toPromoteType:PieceSymbol) => {
    chess.put({type: toPromoteType, color: toPromoteColor}, targetSquare as Square)
    chess.remove(selectedSquare as Square)
    chess.setTurn(toPromoteColor === "w" ? "b" : "w")
    chessBoard = fenToBoard(chess.fen())
    promotion = false
    moveableFields = []
    selectedSquare = ""
    targetSquare = ""
  }

  const fenToArray = (fen:string) => {
    const rows = fen.split(' ')[0].split('/');
    return rows.map(fenRowToArray);
  }
  
  const fenRowToArray = (fenRow:string) => {
    
    const result = [];
    for (const char of fenRow) {
      if (/[1-8]/.test(char)) {
        const emptyCount = parseInt(char, 10);
        for (let i = 0; i < emptyCount; i++) {
          result.push(null);  
        }
      } else {
        result.push(char);
      }
    }
    return result;
  }

  const fenToBoard = (fenString:string) => {

    const fenArray = fenToArray(fenString)
    const board = []

    for (let i=0 ; i<fenArray.length ; i++) {

      const fullRow = []

      let row:number = 0
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

        let col:string = ""
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
        
        square.name = col + row

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

const handleChoosingColor = (selectedColor:"w"|"b") => {

  if (!localStorage.getItem("Chosen-Color")) {
    localStorage.setItem("Chosen-Color", selectedColor)
    chosenColor = selectedColor
  }
  showChooseColorModal = false
}

const handleConfirmMove = (confirm:boolean) => {

  if(confirm){

    // API Call to send your vote here
    makeMove(selectedSquare, targetSquare)

    // localStorage.setItem("hasVoted","d")

    confirmMoveModal = false
    hasVoted = true
  } else {
    confirmMoveModal = false
  }
}

const selectPromotionPiece = (selectedPiece:PieceSymbol) => {
  pieceToPromote = selectedPiece
}

const handleConfirmPromotion = (confirm:boolean) => {

  if (confirm) {
    if (pieceToPromote === undefined) {
      selectPieceToPromoteError = "Please select a piece to promote!"
      return
    }

    handlePromotion(currentPlayer,pieceToPromote)

    confirmPromotionModal = false
    selectPieceToPromoteError = ""
    hasVoted = true
  } else {
    confirmPromotionModal = false
    selectPieceToPromoteError = ""
  }
  
  
}

const getTargetTime = (player:"w"|"b") => {
  const target = new Date();
  const hourOfColor = player === "w" ? 12 : 24
  target.setHours(hourOfColor, 0, 0);

  return target;
}

function updateCountdown() {
  const now = new Date();
  const target = hasVoted ? getTargetTime(OPPONENT) : getTargetTime(YOU);
  const diff = target.getTime() - now.getTime();


  const totalSeconds = Math.floor(diff / 1000);
  const hours = Math.floor(totalSeconds / 3600);
  const minutes = Math.floor((totalSeconds % 3600) / 60);
  const seconds = totalSeconds % 60;

  countdown = [
    hours.toString().padStart(2, '0'),
    minutes.toString().padStart(2, '0'),
    seconds.toString().padStart(2, '0')
  ].join(':');
}

let chessBoard = $state(fenToBoard(chess.fen()))
let selectedSquare = $state<string>("")
let targetSquare = $state<string>("")
let moveableFields = $state<string[]>([]);
let showChooseColorModal = $state<boolean>(
  localStorage.getItem("Chosen-Color") === null ? true : false
)
let chosenColor = $state<string>(
  localStorage.getItem("Chosen-Color") || currentPlayer
)
let confirmMoveModal = $state<boolean>(false)
let confirmPromotionModal = $state<boolean>(false)
let hasVoted = $state<boolean>(false)
let promotion = $state<boolean>(false)
let pieceToPromote = $state<PieceSymbol>()
let selectPieceToPromoteError = $state<string>("")

let countdown = $state<string>("")

let interval: ReturnType<typeof setInterval>

onMount(() => {
  updateCountdown();
  interval = setInterval(updateCountdown, 1000);
});

onDestroy(() => {
  clearInterval(interval);
});
</script>

<main class="min-h-screen min-w-screen flex flex-col justify-start items-center bg-gray-500">

  {#if showChooseColorModal}
    <div class="fixed min-h-screen min-w-screen flex justify-center items-center bg-black/60">
      
      <div class="flex flex-col gap-2 items-center w-120 h-50 p-4 rounded-lg border bg-gray-300">
        <h1 class="text-4xl">Welcome to Vote Chess!</h1>
        <h2 class="text-xl">Which side do you want to vote for?</h2>
        <div class="flex">
          <button 
            class="w-12 h-12 transition-all hover:cursor-pointer hover:scale-110"
            aria-label="Button to choose the white color"
            onclick={() => handleChoosingColor("w")}
          >
            <img src="chess-figures/wK.svg" alt="white king icon"/>
          </button>
          <button 
            class="w-12 h-12 transition-all hover:cursor-pointer hover:scale-110"
            aria-label="Button to choose the black color"
            onclick={() => handleChoosingColor("b")}
          >
            <img src="chess-figures/bK.svg" alt="black king icon"/>
          </button>
        </div>
      </div>
      
    </div>
  {/if}

  {#if confirmMoveModal}
    <div class="fixed min-h-screen min-w-screen flex justify-center items-center bg-black/60">
      
      <div class="flex flex-col gap-2 items-center w-120 h-50 p-4 rounded-lg border bg-gray-300">
        <h1 class="text-4xl">Confirm move!</h1>
        <h2 class="text-xl">You are about to vote for the following move:</h2>
        <p>{"from " + selectedSquare + " to " + targetSquare}</p>
        <div class="flex gap-2">
          <button 
            class="w-20 h-8 rounded-sm border bg-green-300 transition-all hover:cursor-pointer hover:scale-110"
            onclick={() => handleConfirmMove(true)}
          >
            Yes!
          </button>
          <button 
            class="w-20 h-8 rounded-sm border bg-red-300 transition-all hover:cursor-pointer hover:scale-110"
            onclick={() => handleConfirmMove(false)}
          >
            No!
          </button>
        </div>
      </div>
      
    </div>
  {/if}

  {#if confirmPromotionModal}
    <div class="fixed min-h-screen min-w-screen flex justify-center items-center bg-black/60">
      
      <div class="flex flex-col gap-2 items-center w-120 min-h-50 p-4 rounded-lg border bg-gray-300">
        <h1 class="text-4xl">Confirm Promotion!</h1>
        <h2 class="text-xl">You are about to vote on a promotion! Please select a piece to promote and confirm your vote:</h2>
        <div class="flex">
          <button 
            class={`w-16 h-16 transition-all hover:cursor-pointer ${pieceToPromote === "q" ? "scale-110" : pieceToPromote === undefined ? "hover:scale-110" : "hover:scale-110 opacity-25"}`}
            onclick={() => selectPromotionPiece("q")}
          >
            <img src={`chess-figures/${chess.turn()+"Q"}.svg`} alt="Queen"/>
          </button>
          <button 
            class={`w-16 h-16 transition-all hover:cursor-pointer ${pieceToPromote === "r" ? "scale-110" : pieceToPromote === undefined ? "hover:scale-110" : "hover:scale-110 opacity-25"}`}
            onclick={() => selectPromotionPiece("r")}
          >
            <img src={`chess-figures/${chess.turn()+"R"}.svg`} alt="Rook"/>
          </button>  
          <button 
            class={`w-16 h-16 transition-all hover:cursor-pointer ${pieceToPromote === "b" ? "scale-110" : pieceToPromote === undefined ? "hover:scale-110" : "hover:scale-110 opacity-25"}`}
            onclick={() => selectPromotionPiece("b")}
          >
            <img src={`chess-figures/${chess.turn()+"B"}.svg`} alt="Bishop"/>
          </button>
          <button 
            class={`w-16 h-16 transition-all hover:cursor-pointer ${pieceToPromote === "n" ? "scale-110" : pieceToPromote === undefined ? "hover:scale-110" : "hover:scale-110 opacity-25"}`}
            onclick={() => selectPromotionPiece("n")}
          >
            <img src={`chess-figures/${chess.turn()+"N"}.svg`} alt="Knight"/>
          </button>
        </div>
        <p class="text-red-500">{selectPieceToPromoteError}</p>
        <p>{"from " + selectedSquare + " to " + targetSquare}</p>
        <div class="flex gap-2">
          <button 
            class="w-20 h-8 rounded-sm border bg-green-300 transition-all hover:cursor-pointer hover:scale-110"
            onclick={() => handleConfirmPromotion(true)}
          >
            Yes!
          </button>
          <button 
            class="w-20 h-8 rounded-sm border bg-red-300 transition-all hover:cursor-pointer hover:scale-110"
            onclick={() => handleConfirmPromotion(false)}
          >
            No!
          </button>
        </div>
      </div>
      
    </div>
  {/if}

  <div class="flex flex-col gap-2">
  
    <div class="flex justify-between items-center">
      <h1 class="text-4xl">VoteChess</h1>
      <button onclick={() => localStorage.clear()}>
        reset color
      </button>
    </div>

    <p class="w-full flex justify-center">
    {#if currentPlayer === chosenColor}
      {#if hasVoted}
        {`You can vote again in ${countdown}!`}
      {:else}
        {`You have ${countdown} left to vote!`}
      {/if}
    {:else}
      {`You can vote again in ${countdown}!`}
    {/if}

    
    </p>

    <div class="h-128 w-128">
      {#each (chosenColor === null ? chessBoard : chosenColor === "w" ? chessBoard : [...chessBoard].reverse()) as row, rowIndex}
        <div class="flex">
          {#each row as square, squareIndex}
            <div
              aria-hidden="true" 
              class={`
                w-16 h-16 flex justify-center items-center border
                ${
                  selectedSquare === square.name 
                  ? "bg-yellow-500"
                  : moveableFields.some(field => field.includes(square.name)) 
                  ? "bg-green-400" 
                  : moveableFields.some(field => field === "O-O") && ((square.name === "g1" && chess.turn() === "w") || square.name === "g8" && chess.turn() === "b")
                  ? "bg-green-600"
                  : moveableFields.some(field => field === "O-O-O") && ((square.name === "c1" && chess.turn() === "w") || square.name === "c8" && chess.turn() === "b")
                  ? "bg-green-600"
                  : square.type === "k" && chess.inCheck() && chess.turn() === square.color
                  ? "bg-red-700"
                  : (rowIndex + squareIndex) % 2 === 0 
                  ? "bg-gray-700" 
                  : "bg-white"
                }
              `}
              onclick={() => handleSquareClick(square.name)}  
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

    {#if hasVoted}
      
      <div class="w-128 h-128 border p-4 bg-gray-600 rounded-lg">
        
        <h2>You have voted!</h2>
        {"from " + selectedSquare + " to " + targetSquare}
        <!-- {#if chooseColorModal}
          {"Chosen color is: " + localStorage.getItem("Chosen-Color")}
        {/if} -->

        <!-- {#if promotion}
          <div class="flex border">
              <img src={`chess-figures/${chess.turn()+"Q"}.svg`} alt="" onclick={() => handlePromotion(chess.turn(), "q")}/>
              <img src={`chess-figures/${chess.turn()+"R"}.svg`} alt="" onclick={() => handlePromotion(chess.turn(), "r")}/>
              <img src={`chess-figures/${chess.turn()+"B"}.svg`} alt="" onclick={() => handlePromotion(chess.turn(), "b")}/>
              <img src={`chess-figures/${chess.turn()+"N"}.svg`} alt="" onclick={() => handlePromotion(chess.turn(), "n")}/>
          </div>
        {/if} -->

      </div>
    {/if}
  </div>
</main>

