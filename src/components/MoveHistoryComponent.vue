<template>
  <div class="move-history">
    <table>
      <thead>
        <tr>
          <th>Move</th>
          <th>White</th>
          <th>Black</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(move, index) in moveList" :key="index">
          <td>{{ moveNumber(index) }}</td>
          <td>{{ move.white }}</td>
          <td>{{ move.black }}</td>
        </tr>
      </tbody>
    </table>
    <button @click="downloadMoves">Download Move List</button>
  </div>
</template>

<script>
export default {
  props: {
    moves: {
      type: Array,
      default: () => []
    }
  },
  methods: {
    moveNumber(index) {
      // Calculate the move number based on the index
      return Math.floor(index / 2) + 1;
    },

    movePiece(fromRank, fromFile, toRank, toFile) {
      const fromSquare = this.getSquareName(fromRank, fromFile);
      const toSquare = this.getSquareName(toRank, toFile);

      const move = {
        fromSquare,
        toSquare
      };

      // Create a new array and push the move to it
      const updatedMoves = [...this.moves, move];

      // Emit an event to notify the parent component of the updated moves
      this.$emit('update:moves', updatedMoves);

      this.updateChessboard();
      this.handleGameLogic();
    },

    downloadMoves() {
      const movesText = this.generateMoveListText();
      const blob = new Blob([movesText], { type: 'text/plain' });

      // Create a temporary URL for the file
      const url = URL.createObjectURL(blob);

      // Create a temporary link element and simulate a click to trigger the download
      const link = document.createElement('a');
      link.href = url;
      link.download = 'move_list.txt';
      link.click();

      // Clean up by revoking the temporary URL
      URL.revokeObjectURL(url);
    },

    generateMoveListText() {
      // Generate the move list text
      const moveListText = this.moves.map((move, index) => {
        const moveNumber = Math.floor(index / 2) + 1;
        const player = index % 2 === 0 ? 'White' : 'Black';
        return `${moveNumber}. ${player}: ${move}`;
      }).join('\n');

      return moveListText;
    },
  },
  computed: {
    moveList() {
      // Create a flattened array of moves
      return this.moves.reduce((flatMoves, move, index) => {
        if (index % 2 === 0) {
          flatMoves.push({ white: move });
        } else {
          flatMoves[Math.floor(index / 2)].black = move;
        }
        return flatMoves;
      }, []);
    },
  }
};
</script>

<style scoped>
.move-history {
  margin-top: 20px;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  padding: 8px;
  text-align: center;
}

th {
  background-color: #f2f2f2;
}

tr:nth-child(even) {
  background-color: #f9f9f9;
}
</style>
