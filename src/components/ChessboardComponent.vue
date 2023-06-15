<template>

  <div class="chessboard">
    <div class="rank">
      <div class="square"></div>
      <div v-for="file in files" :key="file" class="square file-label">{{ file }}</div>
    </div>
    <div v-for="rank in 8" :key="9 - rank" class="rank">
      <div class="square rank-label">{{ 9 - rank }}</div>
      <div v-for="file in 8" :key="file" class="square" :class="{ dark: isSquareDark(rank, file) }" @click="handleSquareClick(rank, file)">
        <!-- Render chess piece if present -->
        <div v-if="hasPiece(rank, file)" class="piece">

          <img :src="getPieceImage(rank, file)" :alt="getPieceSymbol(rank, file)" class="piece-image">
        </div>

      </div>
    </div>
  </div>
  <MoveHistoryComponent :moves="moves" />
</template>


<script>
import {Chess} from 'chess.js';
import {coordinatesToSquare, pieceImages, SQUARES} from "@/utils/ChessboardUtils";
import MoveHistoryComponent from '@/components/MoveHistoryComponent';


export default {
  name: 'ChessboardComponent',
  components: {
    MoveHistoryComponent
  },
  data() {
    return {
      chess: new Chess(),
      ranks: [1, 2, 3, 4, 5, 6, 7, 8],
      files: ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'],
      selectedSquare: null,
      moves: [],
    };
  },
  methods: {
    isSquareDark: (rank, file) => (rank + file) % 2 !== 0,

    hasPiece(rank, file) {
      const square = coordinatesToSquare(rank, file);
      return this.chess.get(square) !== null;
    },
    getPieceSymbol(rank, file) {
      const square = coordinatesToSquare(rank, file);
      const piece = this.chess.get(square);
      return piece ? piece.type.toUpperCase() : '';
    },
    getPieceImage(rank, file) {
      const square = coordinatesToSquare(rank, file);
      const piece = this.chess.get(square);
      if (piece) {
        const color = piece.color === 'w' ? 'white' : 'black'
        const type = pieceImages[piece.type.toLowerCase()];
        return require(`@/assets/images/${color}_${type}.svg`);
      }
      return null;
    },

    handleSquareClick(rank, file) {
      const square = coordinatesToSquare(rank, file);
      const piece = this.chess.get(square);
      let validMove;

      if (this.selectedSquare) {
        // Handle move for the previously selected piece
        const move = {
          from: this.selectedSquare,
          to: square,
          promotion: 'q', // Default to promoting to a queen
        };
        try {
          this.chess.move(move);
          validMove = true;
        }catch (e) {
          validMove = false;
        }

        if (validMove) {
          const newMove = `${move.from}-${move.to}`;
          this.moves.push(newMove);
          // Move was valid, update chessboard state
          this.updateChessboard();
          // Handle game logic
          this.handleGameLogic();
        }

        this.selectedSquare = null; // Reset the selected square
      } else if (piece && piece.color === this.chess.turn()) {
        // Select the clicked piece if it belongs to the current player
        this.selectedSquare = square;
      }
    },

    updateChessboard() {
      // Clear the chessboard
      for (let rank = 1; rank <= 8; rank++) {
        for (let file = 1; file <= 8; file++) {
          const square = coordinatesToSquare(rank, file);
          const elementId = `square-${square}`;
          const squareElement = document.getElementById(elementId);

          if (squareElement) {
            squareElement.innerHTML = ''; // Clear the content of the square
          }
        }
      }

      // Render the pieces on the chessboard
      for (const square of SQUARES) {
        const piece = this.chess.get(square);

        if (piece) {
          const elementId = `square-${square}`;
          const squareElement = document.getElementById(elementId);

          if (squareElement) {
            const pieceImage = document.createElement('img');
            pieceImage.src = require(`@/assets/images/${this.getPieceImage(piece)}`);
            pieceImage.alt = this.getPieceSymbol(piece);
            pieceImage.classList.add('piece-image');

            squareElement.appendChild(pieceImage);
          }
        }
      }
    },

    handleGameLogic() {
      // Check for checkmate
      if (this.chess.isCheckmate()) {
        const currentPlayer = this.chess.turn() === 'w' ? 'White' : 'Black';
        alert(`${currentPlayer} is in checkmate. Game over!`);
        // Perform any necessary actions after the game ends
        return;
      }

      // Check for stalemate
      if (this.chess.isStalemate()) {
        alert('Stalemate. Game over!');
        // Perform any necessary actions after the game ends
        return;
      }

      // Check for draw by insufficient material
      if (this.chess.isInsufficientMaterial()) {
        alert('Draw by insufficient material. Game over!');
        // Perform any necessary actions after the game ends
        return;
      }

      // Check for draw by threefold repetition
      if (this.chess.isThreefoldRepetition()) {
        alert('Draw by threefold repetition. Game over!');
        // Perform any necessary actions after the game ends
        return;
      }
      // Check for draw by stalemate
      if (this.chess.isDraw()) {
        alert('Stalemate. Game over!');
        // Perform any necessary actions after the game ends
      }
      // No game-ending conditions detected, continue the game
      // Perform any other necessary actions or logic here
    },
    
  },

}
</script>


<style scoped>

.chessboard {
  display: flex;
  flex-direction: column;
  width: max-content;
}

.rank {
  display: flex;
}
.rank-label {
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
}
.square {
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.file-label {
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
}
.piece-image {
  width: 60%;
  height: auto;
}
.dark {
  background-color: #b58863;
}

.piece {
  display: contents;
  position: relative;
  font-size: 24px;
  font-weight: bold;
  color: #000;
}
</style>

