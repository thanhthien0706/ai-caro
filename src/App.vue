<style>
.mainContent {
  max-width: 500px;
}

.mainContent .list_item {
  list-style: none;
  display: flex;
  flex-wrap: wrap;
}
.list_item .item_custom {
  flex-basis: 33.33333%;
  flex-grow: 1;
}

.list_item .item_custom .btn_dot {
  width: 100%;
  height: 50px;
  font-weight: 700;
}

.list_item .item_custom .btn_dot:disabled {
}

.list_item .item_custom .btn_dot.red {
  color: red;
}

.list_item .item_custom .btn_dot:disabled.red {
  opacity: 0.5;
}

.list_item .item_custom .btn_dot.blue {
  color: blue;
}

.list_item .item_custom .btn_dot:disabled.blue {
  opacity: 0.5;
}
</style>

<template>
  <div class="mainContent">
    <div class="container-handler">
      <p class="textShow">{{ textShow }}</p>
      <button @click="resetHandler">Reset</button>
    </div>
    <ul class="list_item">
      <li class="item_custom" v-for="(item, index) in state.data" :key="index">
        <button
          class="btn_dot"
          @click="handleChoice(index)"
          :disabled="isDisabled"
          :class="{ red: item == 1 && item != 0, blue: item != 1 && item != 0 }"
        >
          {{ item == 0 ? "" : item == 1 ? "o" : "x" }}
          <p v-if="checkShow(index)">oke</p>
        </button>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: "App",
  components: {},
  data() {
    return {
      player: 1,
      turn: 0,
      state: {
        data: [0, 0, 0, 0, 0, 0, 0, 0, 0],
        size: 3,
      },
      textShow: "",
      isDisabled: false,
      stateWin: {
        index: null,
        form: "",
      },
    };
  },
  methods: {
    handleChoice(index) {
      if ((this.turn % 2) + 1 == this.player) {
        if (this.state.data[index] == 0) {
          this.state.data[index] = 1;
        }
        if (this.Win(this.state)) {
          this.textShow = "Player Won";
          console.log("Player won");
          this.isDisabled = true;
          return;
        }
        this.aiHandleChoice();
        // this.checkDraw();
        this.turn += 1;
      }
    },

    aiHandleChoice() {
      let mn = 2;
      let minChild = null;
      let sz = this.state.size;
      let child;

      for (let i = 0; i < sz; i++) {
        for (let j = 0; j < sz; j++) {
          child = this.move(this.state, i, j);
          if (child == null) {
            continue;
          }
          let tmp = this.MiniMax(child, 1, true);
          if (mn > tmp) {
            mn = tmp;
            minChild = child;
          }
        }
      }
      this.state = minChild;
      if (this.Win(this.state)) {
        this.textShow = "AI Won";
        console.log("AI won");
        this.isDisabled = true;
        return;
      }

      this.checkDraw();
      this.turn += 1;
    },

    checkDraw() {
      if (this.isEndNode(this.state)) {
        this.isDisabled = true;
        console.log("draw");
        return;
      }
    },

    move(state, x, y) {
      let sz = state.size;
      if (x < 0 || x >= sz) {
        return null;
      }

      if (y < 0 || y >= sz) {
        return null;
      }

      if (state.data[x * sz + y] != 0) {
        return null;
      }

      let res = 0;
      for (const value of state.data) {
        if (value != 0) {
          res += 1;
        }
      }

      let sn = JSON.parse(JSON.stringify(state));
      if (res % 2 == 0) {
        sn.data[x * sz + y] = 1;
      } else {
        sn.data[x * sz + y] = 2;
      }
      return sn;
    },

    MiniMax(s, d, mp) {
      return this.AlphaBeta(s, d, -2, 2, mp);
    },

    AlphaBeta(s, d, a, b, mp) {
      if (this.isEndNode(s) || d == 0) {
        return this.Value(s);
      }
      let sz = s.size;

      if (mp) {
        for (let i = 0; i < sz; i++) {
          for (let j = 0; j < sz; j++) {
            let child = this.move(s, i, j);
            if (child == null) {
              continue;
            }
            let tmp = this.AlphaBeta(child, d - 1, a, b, false);
            a = Math.max(a, tmp);
            if (a >= b) {
              break;
            }
          }
        }
        return a;
      } else {
        for (let i = 0; i < sz; i++) {
          for (let j = 0; j < sz; j++) {
            let child = this.move(s, i, j);
            if (child == null) {
              continue;
            }
            let tmp = this.AlphaBeta(child, d - 1, a, b, true);
            a = Math.min(a, tmp);
            if (a >= b) {
              break;
            }
          }
        }
        return b;
      }
    },

    Value(s) {
      if (this.Win(s)) {
        if (this.checkMyTurn(s)) {
          return 1;
        }
        return -1;
      }
      return 0;
    },

    Win(s) {
      if (s.data == null) {
        return false;
      }

      let sz = s.size;
      let data = s.data;

      for (let i = 0; i < sz; i++) {
        // check row
        if (
          data[i * sz + 0] != 0 &&
          data[i * sz + 0] == data[i * sz + 1] &&
          data[i * sz + 0] == data[i * sz + 2]
        ) {
          // this.row = i;
          this.stateWin = {
            index: i,
            form: "row",
          };
          return true;
        }

        // check column
        if (
          data[0 * sz + i] != 0 &&
          data[0 * sz + i] == data[1 * sz + i] &&
          data[0 * sz + i] == data[2 * sz + i]
        ) {
          // this.column = i;
          this.stateWin = {
            index: i,
            form: "column",
          };
          return true;
        }
      }

      if (
        data[0 * sz + 0] != 0 &&
        data[0 * sz + 0] == data[1 * sz + 1] &&
        data[0 * sz + 0] == data[2 * sz + 2]
      ) {
        this.stateWin = {
          index: null,
          form: "main",
        };
        return true;
      }
      if (
        data[0 * sz + 2] != 0 &&
        data[0 * sz + 2] == data[1 * sz + 1] &&
        data[0 * sz + 2] == data[2 * sz + 0]
      ) {
        this.stateWin = {
          index: null,
          form: "sub",
        };
        return true;
      }

      return false;
    },

    checkMyTurn(s) {
      let res = 0;
      for (let x of s.data) {
        if (x == 0) {
          res += 1;
        }
      }
      if (res % 2 == 0) {
        return true;
      }
      return false;
    },

    isEndNode(s) {
      let sz = s.size;
      let data = s.data;
      for (let i = 0; i < sz; i++) {
        if (
          data[i * sz + 0] != 0 &&
          data[i * sz + 0] == data[i * sz + 1] &&
          data[i * sz + 0] == data[i * sz + 2]
        ) {
          return true;
        }

        if (
          data[0 * sz + i] != 0 &&
          data[0 * sz + i] == data[1 * sz + i] &&
          data[0 * sz + i] == data[2 * sz + i]
        ) {
          return true;
        }
      }

      if (
        data[0 * sz + 0] != 0 &&
        data[0 * sz + 0] == data[1 * sz + 1] &&
        data[0 * sz + 0] == data[2 * sz + 2]
      ) {
        return true;
      }
      if (
        data[0 * sz + 2] != 0 &&
        data[0 * sz + 2] == data[1 * sz + 1] &&
        data[0 * sz + 2] == data[2 * sz + 0]
      ) {
        return true;
      }

      for (const value of data) {
        if (value == 0) {
          return false;
        }
      }
      return true;
    },

    resetHandler() {
      this.state.data = [0, 0, 0, 0, 0, 0, 0, 0, 0];
      this.isDisabled = false;
      this.textShow = "";
    },
    checkShow(index) {
      let indexState = this.stateWin.index;
      const sz = this.state.size;
      if (this.textShow != "") {
        if (this.stateWin.form == "column") {
          let arrR = [
            0 * sz + indexState,
            1 * sz + indexState,
            2 * sz + indexState,
          ];
          return arrR.includes(index);
        } else if (this.stateWin.form == "row") {
          let arrR = [
            indexState * sz + 0,
            indexState * sz + 1,
            indexState * sz + 2,
          ];
          return arrR.includes(index);
        } else if (this.stateWin.form == "main") {
          let arrR = [0 * sz + 0, 1 * sz + 1, 2 * sz + 2];
          return arrR.includes(index);
        } else {
          let arrR = [2 * sz + 0, 1 * sz + 1, 0 * sz + 2];
          return arrR.includes(index);
        }
      }
    },
  },
};
</script>
