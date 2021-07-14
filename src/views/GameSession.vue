<template>
    <section class="board">
      <div v-for="n in 64" :key="n">
        <div v-if="board[n - 1]" v-on:click="makeMove(n)" :id="`${n}-square`" class="bg-white square" :class="{'-clicked': detectClick[0] === n || detectClick[1] === n}">{{n}} {{getCoordinates(n)}}</div>
        <div v-else v-on:click="makeMove(n)" :id="`${n}-square`" class="bg-black square" :class="{'-clicked': detectClick[0] === n || detectClick[1] === n}">{{n}} {{getCoordinates(n)}}</div>
      </div>
    </section>
    <section v-if="squares !== undefined" class="pieces">
      <div v-for="square in squares">
        <piece :id="square[0]" :owner="square[1]" :piece="square[2]"></piece>
      </div>
    </section>
</template>

<script>
import piece from "../components/piece";

export default {
  name: 'GameSession',
  components: { piece },
  data () {
    return {
      board: [],
      squares: undefined,
      moveLog: [],
      moveChoice: [],
    }
  },
  computed: {
    detectClick() {
      return this.moveChoice
    }
  },
  methods: {
    makeMove(n) {
      this.moveChoice.push(n)
      if (this.moveChoice[1] === this.moveChoice[0]) {
        this.moveChoice = []
      } else if (this.moveChoice.length >= 2) {
        this.sendMove(this.moveChoice)
        this.moveChoice = []
      }
    },
    sendMove(message) {
      console.log(this.moveChoice)
      this.chatSocket.send(JSON.stringify({
        'message': message,
        'command': 'new_message'
      }))
      this.message = ';'
    },
    buildBoard() {
      const board = []
      let squareColor = true
      for (let n = 1; n <= 64; n++) {
        if ((n - 1) % 8 !== 0) {
          squareColor = !squareColor
        }
      board.push(squareColor)
      }
      this.board = board
    },
    getCoordinates(n) {
      let x = 0
        if (n % 8 !== 0) {
          x = ((n % 8) - 9) * -1
        } else {
          x = 1
        }
        const y = Math.ceil(n / 8)
      return `${x}, ${y}`
    },
    createGame() {
      const squares = []
      let squareColor = true
      for (let n = 1; n <= 64; n++) {
        if ((n - 1) % 8 !== 0) {
          squareColor = !squareColor
        }
        let x = 0
        if (n % 8 !== 0) {
          x = ((n % 8) - 9) * -1
        } else {
          x = 1
        }
        const y = Math.ceil(n / 8)
        let owner = 0
        switch (y) {
          case 1: case 2:
            owner = 1
            //  white pieces
            break
          case 7: case 8:
            owner = 2
            //  black pieces
            break
          default:
            owner = 0
            break
        }
        let piece = 0
        if (owner === 0) {
          piece = 0
          //  empty squares
        } else if (y === 2 || y === 7) {
          piece = 1
          //  pawns
        } else if (x === 8 || x === 1) {
          piece = 2
          //  rooks
        } else if (x === 7 || x === 2) {
          piece = 3
          //  knights
        } else if (x === 6 || x === 3) {
          piece = 4
          //  bishops
        } else if (x === 4) {
          piece = 5
          //  queens
        } else if (x === 5) {
          piece = 6
          //  kings
        }
        const square = {
          id: n,
          coordinates: `(${x}, ${y})`,
          color: squareColor,
          owner: owner,
          //  0 = none
          //  1 = white
          //  2 = black
          piece: piece
          //  0 = none
          //  1 = pawn
          //  2 = rook
          //  3 = knight
          //  4 = bishop
          //  5 = queen
          //  6 = king
        }
        squares.push(square)
      }
      this.squares = squares
    }
  },
  created () {
    this.chatSocket = new WebSocket(`ws://127.0.0.1:8000/ws/${this.$route.params.gameId}/`)

    this.chatSocket.onopen = function (event) {
      console.log(event)
      console.log("Successful connection to websocket")
    }
    this.chatSocket.onmessage = (event) => {
      const data = JSON.parse(event.data)
      if (data.squares !== undefined) {
        this.squares = JSON.parse(data.squares)
        console.log(this.squares)
      }
    }
    this.chatSocket.onclose = function (event) {
      console.log(event)
      console.log('Connection to websocket closed')
    }
    this.buildBoard()
  }
}
</script>

<style scoped>
.board {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 696px;
  height: 600px;
  display: grid;
  grid-template-columns: repeat(8, 1fr);
}
.pieces {
  pointer-events: none;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 696px;
  height: 600px;
  display: grid;
  grid-template-columns: repeat(8, 1fr);
}
.square {
  color: purple;
  text-align: center;
  height: 100%;
  width: 100%;
}
.bg-black {
  background-color: #000000;
}
.bg-white {
  background-color: #E0E0CE;
}
.-clicked {
  background-color: cornflowerblue;
  opacity: 90%;
}
</style>