<template>
  <v-app>
    <v-main>
      <v-container fill-height>
        <v-row align="center" justify="center">
          <v-col cols="12" md="auto">
            <v-card :color="currentPlayer == 'x' ? 'blue' : 'grey'" dark tile>
              <v-card-text>
                <v-icon size="100">mdi-close</v-icon>
                <h1 class="display-3 text-center">{{ xWin }}</h1>
              </v-card-text>
            </v-card>
          </v-col>
          <v-col cols="12" md="auto">
            <v-card tile>
              <div v-for="(row, y) in board" :key="y" :style="tile > 20 ? 'height: 20px' : ''">
                <v-btn 
                  v-for="(box, x) in row"
                  :key="x"
                  outlined icon tile 
                  :x-large="tile <= 10"
                  :large="tile > 10 && tile <= 12"
                  :small="tile > 15 && tile <= 20"
                  :x-small="tile > 20"
                  @click="play(x,y)"
                >
                  <v-icon v-if="box == 'x'" color="blue">mdi-close</v-icon>
                  <v-icon v-else-if="box == 'o'" color="red">mdi-circle-outline</v-icon>
                </v-btn>
              </div>
            </v-card>
          </v-col>
          <v-col cols="12" md="auto">
            <v-card :color="currentPlayer == 'o' ? 'red' : 'grey'" dark tile>
              <v-card-text>
                <v-icon size="100">mdi-circle-outline</v-icon>
                <h1 class="display-3 text-center">{{ oWin }}</h1>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
    <v-dialog v-model="startDialog" persistent max-width="600">
      <v-card tile>
        <v-container class="py-5">
          <h1 class="display-4 text-center">Tic Tac Toe</h1>
          <v-row justify="center">
            <v-col cols="auto">
              <v-icon size="100" color="red">mdi-circle-outline</v-icon>
            </v-col>
            <v-col cols="auto">
              <v-icon size="100" color="blue">mdi-close</v-icon>
            </v-col>
          </v-row>
          <v-row justify="center">
            <v-col cols="4">
              <v-slider
                v-model="tile"
                label="Tile"
                thumb-label="always"
                min="3"
                max="30"
              ></v-slider>
              <v-btn color="primary" :loading="loading" block tile @click="start">Start</v-btn>
            </v-col>
          </v-row>
          <v-row>
            <v-col class="text-center">
              Make with <v-icon color="red">mdi-heart</v-icon> By <a href="https://github.com/corneliusventi" target="blank">Cornelius Venti</a>
            </v-col>
          </v-row>
        </v-container>
      </v-card>
    </v-dialog>

    <v-dialog v-model="startDialog" persistent max-width="600">
      <v-card tile>
        <v-container class="py-5">
          <h1 class="display-4 text-center">Tic Tac Toe</h1>
          <v-row justify="center">
            <v-col cols="auto">
              <v-icon size="100" color="red">mdi-circle-outline</v-icon>
            </v-col>
            <v-col cols="auto">
              <v-icon size="100" color="blue">mdi-close</v-icon>
            </v-col>
          </v-row>
          <v-row justify="center">
            <v-col cols="4">
              <v-slider
                v-model="tile"
                label="Tile"
                thumb-label="always"
                min="3"
                max="30"
              ></v-slider>
              <v-btn color="primary" :loading="loading" block tile @click="start">Start</v-btn>
            </v-col>
          </v-row>
          <v-row>
            <v-col class="text-center">
              Make with <v-icon color="red">mdi-heart</v-icon> By <a href="https://github.com/corneliusventi" target="blank">Cornelius Venti</a>
            </v-col>
          </v-row>
        </v-container>
      </v-card>
    </v-dialog>

    <v-dialog v-model="finishDialog" persistent max-width="600">
      <v-card tile>
        <v-container class="py-5">
          <h1 class="display-4 text-center">{{ winner ? 'Winner' : 'Draw' }}</h1>
          <v-row v-if="winner" justify="center">
            <v-col v-if="winner == 'o'" cols="auto">
              <v-icon size="100" color="red">mdi-circle-outline</v-icon>
            </v-col>
            <v-col v-else cols="auto">
              <v-icon size="100" color="blue">mdi-close</v-icon>
            </v-col>
          </v-row>
          <v-row justify="center">
            <v-col cols="auto">
              <v-btn color="primary" :loading="loading" block tile @click="restart">Restart</v-btn>
            </v-col>
            <v-col cols="auto">
              <v-btn :loading="loading" text block tile @click="reset">Reset</v-btn>
            </v-col>
          </v-row>
          <v-row>
            <v-col class="text-center">
              Make with <v-icon color="red">mdi-heart</v-icon> By <a href="https://github.com/corneliusventi" target="blank">Cornelius Venti</a>
            </v-col>
          </v-row>
        </v-container>
      </v-card>
    </v-dialog>

  </v-app>
</template>

<script>
export default {
  name: 'App',

  data() {
    return {
      loading: false,
      tile: 3,
      startDialog: true,
      finishDialog: false,
      board: [],
      currentPlayer: 'x',
      count: 0,
      winner: null,
      xWin: 0,
      oWin: 0,
    };
  },

  methods: {
    async start() {
      this.loading = true;
      this.board = await new Array(this.tile).fill(new Array(this.tile));
      this.loading = false;
      this.startDialog = false;
    },

    async play(x, y) {
      if (!this.board[y][x]) {
        // Javascript caveats and can not use Vue $set
        //this.$set(this.board[y], x, this.currentPlayer);
        //this.board[y].splice(x, 1, this.currentPlayer);
        this.board[y][x] = this.currentPlayer;
        this.$forceUpdate();

        this.count += 1;

        await this.checkWin(x, y);

        if (!this.winner) {
          if (this.count == this.tile * this.tile) {
            this.finishDialog = true;
          }
          this.currentPlayer = this.currentPlayer == 'x' ? 'o' : 'x';
        }
      }
    },

    checkWin(x, y) {
      if(this.checkHorizontal(y) || this.checkVertical(x) || this.checkDiagonal(x, y)) {
        if (this.currentPlayer == 'x') {
          this.xWin += 1;
          this.winner = 'x';
        } else {
          this.oWin += 1;
          this.winner = 'o';
        }
        this.displayWinner();
      }
    },

    checkHorizontal(y) {
      let checklist = [];
      let x;

      for (x = 0; x < this.tile; x++) {
        checklist.push(this.board[y][x] == this.currentPlayer);
      }

      return checklist.every(check => !!check);
    },

    checkVertical(x) {
      let checklist = [];
      let y;

      for (y = 0; y < this.tile; y++) {
        checklist.push(this.board[y][x] == this.currentPlayer);
      }

      return checklist.every(check => !!check);
    },

    checkDiagonal(x, y) {
      let checklist = [];
      let i;

      if (x == y) {
        for (i = 0; i < this.tile; i++) {
          checklist.push(this.board[i][i] == this.currentPlayer);
        }
      } else {
        for (i = 0; i < this.tile; i++) {
          checklist.push(this.board[i][ this.tile - 1 - i] == this.currentPlayer);
        }
      }

      return checklist.every(check => !!check);
    },

    displayWinner() {
      this.finishDialog = true;
    },

    async restart() {
      this.currentPlayer = 'x';
      this.count = 0;
      this.winner = null;
      this.loading = true;
      this.board = await new Array(this.tile).fill(new Array(this.tile));
      this.loading = false;
      this.finishDialog = false;
    },

    reset() {
      this.tile = 3;
      this.currentPlayer = 'x';
      this.count = 0;
      this.winner = null;
      this.board = [];
      this.xWin = 0;
      this.oWin = 0;
      this.finishDialog = false;
      this.startDialog = true;
    }
  },
};
</script>
