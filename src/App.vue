<script setup>
  import { ref, onMounted } from 'vue';
  import Player from './components/Player.vue';
  import gsap from 'gsap';

  let currentPlayer = ref(1);
  let colors = ref(["red", "yellow"]);
  let names = ref(["Red", "Yellow"]);
  let selectedMoves = ref(["drop", "drop"]);
  let hoveredSquare = ref(null);
  let hoveredMove = ref(null);
  let winner = ref(null);
  let canMove = ref(true);

  let boardSize = 7;
  let winLength = 4;
  document.querySelector(":root").style.setProperty("--board-size", boardSize)

  let board = [];
  let tempBoard = [];
  for (let x = 0; x < boardSize; x++) {
    board.push([]);
    tempBoard.push([]);
    for (let y = 0; y < boardSize; y++) { 
      board[x].push(0);
      tempBoard[x].push(0);
    }
  }

  function switchPlayers() {
    currentPlayer.value %= 2;
    currentPlayer.value++;
  }

  function reset() {
    document.querySelectorAll(".piece").forEach(piece => piece.remove());
    for (let x = 0; x < boardSize; x++) {
      for (let y = 0; y < boardSize; y++) {
        board[x][y] = 0;
      }
    }
    selectedMoves.value = ["drop", "drop"];
    switchPlayers();
    winner.value = null;
    canMove.value = true;
    window.dialog.close();
    document.getElementById("arrowsWrapper").style.display = "flex";
  }

  function checkWins() {
    function checkLine(pieces) { 
      if (pieces[0][0] == 0) {
        return;
      }

      for (let x = 1; x < winLength; x++) {
        if (pieces[0][0] != pieces[x][0]) {
          return;
        }
      }

      pieces.forEach((piece) => {
        document.getElementById("p" + piece[1] + "_" + piece[2]).style.borderColor = "white";
      })

      if (winner.value != null && winner.value != pieces[0][0]) {
        winner.value = "tie"
      } else {
        winner.value = pieces[0][0];
      }
    }

    let checkingLine;

    for (let x = 0; x < boardSize; x++) {
      for (let y = 0; y < boardSize - winLength + 1; y++) {
        checkingLine = [];
        for (let checkedCells = 0; checkedCells < winLength; checkedCells++) {
          checkingLine.push([board[x][y + checkedCells], x, y + checkedCells]);
        }
        checkLine(checkingLine);
      }
    }
    for (let x = 0; x < boardSize - winLength + 1; x++) {
      for (let y = 0; y < boardSize; y++) {
        checkingLine = [];
        for (let checkedCells = 0; checkedCells < winLength; checkedCells++) {
          checkingLine.push([board[x + checkedCells][y], x + checkedCells, y]);
        }
        checkLine(checkingLine);
      }
    }
    for (let x = 0; x < boardSize - winLength + 1; x++) {
      for (let y = 0; y < boardSize - winLength + 1; y++) {
        checkingLine = [];
        for (let checkedCells = 0; checkedCells < winLength; checkedCells++) {
          checkingLine.push([board[x + checkedCells][y + checkedCells], x + checkedCells, y + checkedCells]);
        }
        checkLine(checkingLine);
      }
    }
    for (let x = boardSize + 1; x < boardSize; x++) {
      for (let y = 0; y < boardSize - winLength + 1; y++) {
        checkingLine = [];
        for (let checkedCells = 0; checkedCells < winLength; checkedCells++) {
          checkingLine.push([board[x - checkedCells][y + checkedCells], x - checkedCells, y + checkedCells]);
        }
        checkLine(checkingLine);
      }
    }

    if (winner.value == null) {
      selectedMoves.value[currentPlayer.value - 1] = "drop";
      switchPlayers();
      canMove.value = true;
      document.getElementById("arrowsWrapper").style.display = "flex";
    } else {
      window.dialog.showModal();
    }
  }

  function resetHoles() {
    for (let x = -1; x < boardSize + 1; x++) {
      for (let y = -1; y < boardSize + 1; y++) {
        gsap.set("#h" + x + "_" + y, {
          x: x * 10 + "rem",
          y: y * 10 + "rem"
        });
      }
    }
  }
  function gravity() {
    let falling = false;
    for (let x = 0; x < boardSize; x++) {
      for (let y = boardSize - 1; y > -1; y--) {
        if (board[x][y] > 0) {
          let fall = 0;
          while (y + fall +  1 < boardSize && board[x][y + fall + 1] == 0) {
            fall++;
          }
          if (fall > 0) {
            falling = true;
            gsap.to("#pf" + x + "_" + y, {
              duration: 1,
              ease: "bounce.out",
              y: "+=" + fall * 10 + "rem"
            }) 
            board[x][y + fall] = board[x][y];
            board[x][y] = 0;
          }
          document.getElementById("pf" + x + "_" + y).id = "p" + x + "_" + (y + fall);
        }
      }
    }
    let delay = falling ? 1000 : 0;
    setTimeout(() => {
      checkWins();
      resetHoles();
    }, delay)
  }
  function updateBoard() {
    for (let x = 0; x < boardSize; x++) {
      for (let y = 0; y < boardSize; y++) {
        board[x][y] = tempBoard[x][y];
      }
    }
    gravity();
  }

  function updateMove(hoveredSquare) {
    if (!canMove.value) return;
    const image = document.getElementById("moveImage");
    if (hoveredSquare == null) {
      hoveredMove.value = null;
      image.style.backgroundImage = null;
      return;
    } 

    switch (selectedMoves.value[currentPlayer.value - 1]) {
      case "rotate":
        switch (hoveredSquare[0]) {
          case "l":
            hoveredMove.value = "rotateCCW";
            break;
          case "m":
            hoveredMove.value = "flip";
            break;
          case "r":
            hoveredMove.value = "rotateCW";
            break; 
        }
        break;
      case "revolve":
        switch (hoveredSquare[0]) {
          case "l":
            hoveredMove.value = "revolveCCW";
            break;
          case "m":
            image.style.backgroundImage = null;
            return;
          case "r":
            hoveredMove.value = "revolveCW";
            break; 
        }
        break;
      case "shift":
        if (hoveredSquare == "mm") {
          image.style.backgroundImage = null;
          return;
        }
        hoveredMove.value = "shift";
        switch (hoveredSquare[1]) {
          case "u":
            hoveredMove.value += "U";
            break;
          case "m":
            break;
          case "d":
            hoveredMove.value += "D";
            break; 
        }
        switch (hoveredSquare[0]) {
          case "l":
            hoveredMove.value += "L";
            break;
          case "m":
            break;
          case "r":
            hoveredMove.value += "R";
            break; 
        }
        break;
    }
    image.style.backgroundImage = "url(src/assets/" + hoveredMove.value + ".png)";
  }

  function rotateCCW() {
    gsap.to("#board", {
      duration: 1,
      ease: "expo.inOut",
      rotate: "-=90",
      onComplete: () => { 
        gsap.set("#board", {rotate: 0}); 
        for (let x = 0; x < boardSize; x++) {
          for (let y = 0; y < boardSize; y++) {
            tempBoard[y][boardSize - x - 1] = board[x][y];
            if (board[x][y] > 0) {
              gsap.set("#p" + x + "_" + y, {x: y * 10 + "rem", y: (boardSize - x - 1) * 10 + "rem"});
              document.getElementById("p" + x + "_" + y).id = "pf" + y + "_" + (boardSize - x - 1);
            }
          }
        }
        updateBoard();
      }
    });
  }
  function flip() {
    gsap.to("#board", {
      duration: 1,
      ease: "expo.inOut",
      rotateX: "+=180",
      onComplete: () => { 
        gsap.set("#board", {rotateX: 0}); 
        for (let x = 0; x < boardSize; x++) {
          for (let y = 0; y < boardSize; y++) {
            tempBoard[x][boardSize - y - 1] = board[x][y];
            if (board[x][y] > 0) {
              gsap.set("#p" + x + "_" + y, {y: (boardSize - y - 1) * 10 + "rem"});
              document.getElementById("p" + x + "_" + y).id = "pf" + x + "_" + (boardSize - y - 1);
            }
          }
        }
        updateBoard();
      }
    });
  }
  function rotateCW() {
    gsap.to("#board", {
      duration: 1,
      ease: "expo.inOut", 
      rotate: "+=90",
      onComplete: () => { 
        gsap.set("#board", {rotate: 0}); 
        for (let x = 0; x < boardSize; x++) {
          for (let y = 0; y < boardSize; y++) {
            tempBoard[boardSize - y - 1][x] = board[x][y];
            if (board[x][y] > 0) {
              gsap.set("#p" + x + "_" + y, {x: (boardSize - y - 1) * 10 + "rem", y: x * 10 + "rem"});
              document.getElementById("p" + x + "_" + y).id = "pf" + (boardSize - y - 1) + "_" + x;
            }
          }
        }
        updateBoard();
      }
    });
  }

  function shiftUL() {
    for (let x = 0; x < boardSize; x++) {
      for (let y = 0; y < boardSize; y++) {
        let sX = x;
        let sY = y;
        let oob = false;
        if (x == 0) {
          sX = boardSize;
          oob = true;
        }
        if (y == 0) {
          sY = boardSize;
          oob = true;
        }
        tempBoard[sX - 1][sY - 1] = board[x][y];
        if (board[x][y] > 0) {
          if (oob) {
            const piece = document.createElement("div");
            piece.className = "piece";
            piece.style.backgroundColor = colors.value[board[x][y] - 1];
            piece.id = "pf" + (sX - 1) + "_" + (sY - 1);
            gsap.set(piece, {
              x: sX * 10 + "rem",
              y: sY * 10 + "rem"
            });
            document.getElementById("pieces").appendChild(piece);
            document.getElementById("p" + x + "_" + y).classList.add("oob");
          } else {
            document.getElementById("p" + x + "_" + y).id = "pf" + (x - 1) + "_" + (y - 1);
          }
        }
      }
    }
    gsap.to(".piece, .hole", {
      duration: 1,
      ease: "expo.inOut",
      x: "-=10rem",
      y: "-=10rem",
      modifiers: {
        x: gsap.utils.unitize(x => gsap.utils.wrap(-10, (boardSize + 1) * 10, x)),
        y: gsap.utils.unitize(y => gsap.utils.wrap(-10, (boardSize + 1) * 10, y))
      },
      onComplete: () => {
        updateBoard();
        document.querySelectorAll(".oob").forEach(e => e.remove());
      }
    });
  }
  function shiftU() {
    for (let x = 0; x < boardSize; x++) {
      for (let y = 0; y < boardSize; y++) {
        if (y == 0) {
          tempBoard[x][boardSize - 1] = board[x][y];
        } else {
          tempBoard[x][y - 1] = board[x][y];
        }
        if (board[x][y] > 0) {
          if (y == 0) {
            const piece = document.createElement("div");
            piece.className = "piece";
            piece.style.backgroundColor = colors.value[board[x][y] - 1];
            piece.id = "pf" + x + "_" + (boardSize - 1);
            gsap.set(piece, {
              x: x * 10 + "rem",
              y: boardSize * 10 + "rem"
            });
            document.getElementById("pieces").appendChild(piece);
            document.getElementById("p" + x + "_" + y).classList.add("oob");
          } else {
            document.getElementById("p" + x + "_" + y).id = "pf" + x + "_" + (y - 1);
          }
        }
      }
    }
    gsap.to(".piece, .hole", {
      duration: 1,
      ease: "expo.inOut",
      y: "-=10rem",
      modifiers: {
        y: gsap.utils.unitize(y => gsap.utils.wrap(-10, (boardSize + 1) * 10, y))
      },
      onComplete: () => {
        updateBoard();
        document.querySelectorAll(".oob").forEach(e => e.remove());
      }
    });
  }
  function shiftUR() {
    for (let x = 0; x < boardSize; x++) {
      for (let y = 0; y < boardSize; y++) {
        let sX = x;
        let sY = y;
        let oob = false;
        if (x == boardSize - 1) {
          sX = -1;
          oob = true;
        }
        if (y == 0) {
          sY = boardSize;
          oob = true;
        }
        tempBoard[sX + 1][sY - 1] = board[x][y];
        if (board[x][y] > 0) {
          if (oob) {
            const piece = document.createElement("div");
            piece.className = "piece";
            piece.style.backgroundColor = colors.value[board[x][y] - 1];
            piece.id = "pf" + (sX + 1) + "_" + (sY - 1);
            gsap.set(piece, {
              x: sX * 10 + "rem",
              y: sY * 10 + "rem"
            });
            document.getElementById("pieces").appendChild(piece);
            document.getElementById("p" + x + "_" + y).classList.add("oob");
          } else {
            document.getElementById("p" + x + "_" + y).id = "pf" + (x + 1) + "_" + (y - 1);
          }
        }
      }
    }
    gsap.to(".piece, .hole", {
      duration: 1,
      ease: "expo.inOut",
      x: "+=10rem",
      y: "-=10rem",
      modifiers: {
        x: gsap.utils.unitize(x => gsap.utils.wrap(-10, (boardSize + 1) * 10, x)),
        y: gsap.utils.unitize(y => gsap.utils.wrap(-10, (boardSize + 1) * 10, y))
      },
      onComplete: () => {
        updateBoard();
        document.querySelectorAll(".oob").forEach(e => e.remove());
      }
    });
  }
  function shiftL() {
    for (let x = 0; x < boardSize; x++) {
      for (let y = 0; y < boardSize; y++) {
        if (x == 0) {
          tempBoard[boardSize - 1][y] = board[x][y];
        } else {
          tempBoard[x - 1][y] = board[x][y];
        }
        if (board[x][y] > 0) {
          if (x == 0) {
            const piece = document.createElement("div");
            piece.className = "piece";
            piece.style.backgroundColor = colors.value[board[x][y] - 1];
            piece.id = "pf" + (boardSize - 1) + "_" + y;
            gsap.set(piece, {
              x: boardSize * 10 + "rem",
              y: y * 10 + "rem"
            });
            document.getElementById("pieces").appendChild(piece);
            document.getElementById("p0_" + y).classList.add("oob");
          } else {
            document.getElementById("p" + x + "_" + y).id = "pf" + (x - 1) + "_" + y;
          }
        }
      }
    }
    gsap.to(".piece, .hole", {
      duration: 1,
      ease: "expo.inOut",
      x: "-=10rem",
      modifiers: {
        x: gsap.utils.unitize(x => gsap.utils.wrap(-10, (boardSize + 1) * 10, x))
      },
      onComplete: () => {
        updateBoard();
        document.querySelectorAll(".oob").forEach(e => e.remove());
      }
    });
  }
  function shiftR() {
    for (let x = 0; x < boardSize; x++) {
      for (let y = 0; y < boardSize; y++) {
        if (x == boardSize - 1) {
          tempBoard[0][y] = board[x][y];
        } else {
          tempBoard[x + 1][y] = board[x][y];
        }
        if (board[x][y] > 0) {
          if (x == boardSize - 1) {
            const piece = document.createElement("div");
            piece.className = "piece";
            piece.style.backgroundColor = colors.value[board[x][y] - 1];
            piece.id = "pf0_" + y;
            gsap.set(piece, {
              x: "-10rem",
              y: y * 10 + "rem"
            });
            document.getElementById("pieces").appendChild(piece);
            document.getElementById("p" + x + "_" + y).classList.add("oob");
          } else {
            document.getElementById("p" + x + "_" + y).id = "pf" + (x + 1) + "_" + y;
          }
        }
      }
    }
    gsap.to(".piece, .hole", {
      duration: 1,
      ease: "expo.inOut",
      x: "+=10rem",
      modifiers: {
        x: gsap.utils.unitize(x => gsap.utils.wrap(-10, (boardSize + 1) * 10, x))
      },
      onComplete: () => {
        updateBoard();
        document.querySelectorAll(".oob").forEach(e => e.remove());
      }
    });
  }
  function shiftDL() {
    for (let x = 0; x < boardSize; x++) {
      for (let y = 0; y < boardSize; y++) {
        let sX = x;
        let sY = y;
        let oob = false;
        if (x == 0) {
          sX = boardSize;
          oob = true;
        }
        if (y == boardSize - 1) {
          sY = -1;
          oob = true;
        }
        tempBoard[sX - 1][sY + 1] = board[x][y];
        if (board[x][y] > 0) {
          if (oob) {
            const piece = document.createElement("div");
            piece.className = "piece";
            piece.style.backgroundColor = colors.value[board[x][y] - 1];
            piece.id = "pf" + (sX - 1) + "_" + (sY + 1);
            gsap.set(piece, {
              x: sX * 10 + "rem",
              y: sY * 10 + "rem"
            });
            document.getElementById("pieces").appendChild(piece);
            document.getElementById("p" + x + "_" + y).classList.add("oob");
          } else {
            document.getElementById("p" + x + "_" + y).id = "pf" + (x - 1) + "_" + (y + 1);
          }
        }
      }
    }
    gsap.to(".piece, .hole", {
      duration: 1,
      ease: "expo.inOut",
      x: "-=10rem",
      y: "+=10rem",
      modifiers: {
        x: gsap.utils.unitize(x => gsap.utils.wrap(-10, (boardSize + 1) * 10, x)),
        y: gsap.utils.unitize(y => gsap.utils.wrap(-10, (boardSize + 1) * 10, y))
      },
      onComplete: () => {
        updateBoard();
        document.querySelectorAll(".oob").forEach(e => e.remove());
      }
    });
  }
  function shiftD() {  
    for (let x = 0; x < boardSize; x++) {
      for (let y = 0; y < boardSize; y++) {
        if (y == boardSize - 1) {
          tempBoard[x][0] = board[x][y];
        } else {
          tempBoard[x][y + 1] = board[x][y];
        }
        if (board[x][y] > 0) {
          if (y == boardSize - 1) {
            const piece = document.createElement("div");
            piece.className = "piece";
            piece.style.backgroundColor = colors.value[board[x][y] - 1];
            piece.id = "pf" + x + "_0";
            gsap.set(piece, {
              x: x * 10 + "rem",
              y: "-10rem"
            });
            document.getElementById("pieces").appendChild(piece);
            document.getElementById("p" + x + "_" + y).classList.add("oob");
          } else {
            document.getElementById("p" + x + "_" + y).id = "pf" + x + "_" + (y + 1);
          }
        }
      }
    }
    gsap.to(".piece, .hole", {
      duration: 1,
      ease: "expo.inOut",
      y: "+=10rem",
      modifiers: {
        y: gsap.utils.unitize(y => gsap.utils.wrap(-10, (boardSize + 1) * 10, y))
      },
      onComplete: () => {
        updateBoard();
        document.querySelectorAll(".oob").forEach(e => e.remove());
      }
    });
  }
  function shiftDR() {
    for (let x = 0; x < boardSize; x++) {
      for (let y = 0; y < boardSize; y++) {
        let sX = x;
        let sY = y;
        let oob = false;
        if (x == boardSize - 1) {
          sX = -1;
          oob = true;
        }
        if (y == boardSize - 1) {
          sY = -1;
          oob = true;
        }
        tempBoard[sX + 1][sY + 1] = board[x][y];
        if (board[x][y] > 0) {
          if (oob) {
            const piece = document.createElement("div");
            piece.className = "piece";
            piece.style.backgroundColor = colors.value[board[x][y] - 1];
            piece.id = "pf" + (sX + 1) + "_" + (sY + 1);
            gsap.set(piece, {
              x: sX * 10 + "rem",
              y: sY * 10 + "rem"
            });
            document.getElementById("pieces").appendChild(piece);
            document.getElementById("p" + x + "_" + y).classList.add("oob");
          } else {
            document.getElementById("p" + x + "_" + y).id = "pf" + (x + 1) + "_" + (y + 1);
          }
        }
      }
    }
    gsap.to(".piece, .hole", {
      duration: 1,
      ease: "expo.inOut",
      x: "+=10rem",
      y: "+=10rem",
      modifiers: {
        x: gsap.utils.unitize(x => gsap.utils.wrap(-10, (boardSize + 1) * 10, x)),
        y: gsap.utils.unitize(y => gsap.utils.wrap(-10, (boardSize + 1) * 10, y))
      },
      onComplete: () => {
        updateBoard();
        document.querySelectorAll(".oob").forEach(e => e.remove());
      }
    });
  }

  let revolveDirections = [];
  let r = 1;
  let v = boardSize - 1;
  for (let x = 0; x < boardSize; x++) {
    revolveDirections.push([]);
    let rLeft = r;
    let vLeft = v;
    let placing = "r";
    for (let y = 0; y < boardSize; y++) {
      if (x == boardSize / 2 - 0.5 && y == x) {
        revolveDirections[x].push(null);
        continue;
      }
      if (rLeft == 0) placing = "v";
      if (vLeft == 0 && y > boardSize / 2 - 0.5) placing = "l";
      switch (placing) {
        case "r":
          revolveDirections[x].push("r");
          rLeft--;
          break;
        case "v":
          if (v > 0) {
            revolveDirections[x].push("u");
            vLeft--;
          } else {
            revolveDirections[x].push("d");
            vLeft++;
          }
          break;
        case "l":
          revolveDirections[x].push("l");
          break;
      } 
    }
    if (x < boardSize / 2 - 1.5) {
      r++;
    } else if (x > boardSize / 2 - 1.5) {
      r--
    }
    v -= 2
  }


  function revolveCW() {
    gsap.to(".hole", {
      duration: 0.5,
      ease: "expo.in",
      scale: "0.5"
    });
    gsap.to(".hole", {
      duration: 0.5,
      ease: "expo.Out",
      scale: "1",
      delay: 0.5,
    });
    for (let x = 0; x < boardSize; x++) {
      for (let y = 0; y < boardSize; y++) {
        switch (revolveDirections[x][y]) {
          case "l":
            tempBoard[x - 1][y] = board[x][y];
            if (board[x][y] > 0) {
              gsap.to("#p" + x + "_" + y, {
                duration: 1,
                ease: "expo.inOut",
                x: "-=10rem",
              });
              document.getElementById("p" + x + "_" + y).id = "pf" + (x - 1) + "_" + y;
            }
            gsap.to("#h" + x + "_" + y, {
              duration: 1,
              ease: "expo.inOut",
              x: "-=10rem",
            });
            break;
          case "u":
            tempBoard[x][y - 1] = board[x][y];
            if (board[x][y] > 0) {
              gsap.to("#p" + x + "_" + y, {
                duration: 1,
                ease: "expo.inOut",
                y: "-=10rem",
              });
              document.getElementById("p" + x + "_" + y).id = "pf" + x + "_" + (y - 1);
            }
            gsap.to("#h" + x + "_" + y, {
              duration: 1,
              ease: "expo.inOut",
              y: "-=10rem",
            });
            break;
          case "r":
            tempBoard[x + 1][y] = board[x][y];
            if (board[x][y] > 0) {
              gsap.to("#p" + x + "_" + y, {
                duration: 1,
                ease: "expo.inOut",
                x: "+=10rem",
              });
              document.getElementById("p" + x + "_" + y).id = "pf" + (x + 1) + "_" + y;
            }
            gsap.to("#h" + x + "_" + y, {
              duration: 1,
              ease: "expo.inOut",
              x: "+=10rem",
            });
            break;
          case "d":
            tempBoard[x][y + 1] = board[x][y];
            if (board[x][y] > 0) {
              gsap.to("#p" + x + "_" + y, {
                duration: 1,
                ease: "expo.inOut",
                y: "+=10rem",
              });
              document.getElementById("p" + x + "_" + y).id = "pf" + x + "_" + (y + 1);
            }
            gsap.to("#h" + x + "_" + y, {
              duration: 1,
              ease: "expo.inOut",
              y: "+=10rem",
            });
            break;
          default:
            tempBoard[x][y] = board[x][y]
            if (board[x][y] > 0) {
              document.getElementById("p" + x + "_" + y).id = "pf" + x + "_" + y;
            }
            break;
        }
      }
    }
    gsap.to(document, {
      duration: 1,
      onComplete: updateBoard,
    });
  }
  function revolveCCW() {
    gsap.to(".hole", {
      duration: 0.5,
      ease: "expo.in",
      scale: "0.5",
    });
    gsap.to(".hole", {
      duration: 0.5,
      ease: "expo.Out",
      scale: "1",
      delay: 0.5,
    });
    for (let x = 0; x < boardSize; x++) {
      for (let y = 0; y < boardSize; y++) {
        switch (revolveDirections[boardSize - x - 1][y]) {
          case "r":
            tempBoard[x - 1][y] = board[x][y];
            if (board[x][y] > 0) {
              gsap.to("#p" + x + "_" + y, {
                duration: 1,
                ease: "expo.inOut",
                x: "-=10rem",
              });
              document.getElementById("p" + x + "_" + y).id = "pf" + (x - 1) + "_" + y;
            }
            gsap.to("#h" + x + "_" + y, {
              duration: 1,
              ease: "expo.inOut",
              x: "-=10rem",
            });
            break;
          case "u":
            tempBoard[x][y - 1] = board[x][y];
            if (board[x][y] > 0) {
              gsap.to("#p" + x + "_" + y, {
                duration: 1,
                ease: "expo.inOut",
                y: "-=10rem",
              });
              document.getElementById("p" + x + "_" + y).id = "pf" + x + "_" + (y - 1);
            }
            gsap.to("#h" + x + "_" + y, {
              duration: 1,
              ease: "expo.inOut",
              y: "-=10rem",
            });
            break;
          case "l":
            tempBoard[x + 1][y] = board[x][y];
            if (board[x][y] > 0) {
              gsap.to("#p" + x + "_" + y, {
                duration: 1,
                ease: "expo.inOut",
                x: "+=10rem",
              });
              document.getElementById("p" + x + "_" + y).id = "pf" + (x + 1) + "_" + y;
            }
            gsap.to("#h" + x + "_" + y, {
              duration: 1,
              ease: "expo.inOut",
              x: "+=10rem",
            });
            break;
          case "d":
            tempBoard[x][y + 1] = board[x][y];
            if (board[x][y] > 0) {
              gsap.to("#p" + x + "_" + y, {
                duration: 1,
                ease: "expo.inOut",
                y: "+=10rem",
              });
              document.getElementById("p" + x + "_" + y).id = "pf" + x + "_" + (y + 1);
            }
            gsap.to("#h" + x + "_" + y, {
              duration: 1,
              ease: "expo.inOut",
              y: "+=10rem",
            });
            break;
          default:
            tempBoard[x][y] = board[x][y]
            if (board[x][y] > 0) {
              document.getElementById("p" + x + "_" + y).id = "pf" + x + "_" + y;
            }
            break;
        }
      }
    }
    gsap.to(document, {
      duration: 1,
      onComplete: updateBoard,
    });
  }

  function makeMove(move) {
    if (!canMove.value || winner.value != null || move == null || (typeof move === "number" && board[move][0] != 0)) return;
    document.getElementById("arrowsWrapper").style.display = "none";
    canMove.value = false;
    if (typeof move === "number") {
      let y = boardSize - 1;
      while (board[move][y] != 0) {
        y--
      }
      board[move][y] = currentPlayer.value;
      const piece = document.createElement("div");
      piece.className = "piece";
      piece.style.backgroundColor = colors.value[currentPlayer.value - 1];
      piece.id = "p" + move + "_" + y;
      gsap.set(piece, {
        x: move * 10 + "rem",
        y: "-10rem"
      });
      document.getElementById("pieces").appendChild(piece);
      gsap.to(piece, {
        duration: 1,
        ease: "bounce.out",
        y: "+=" + (y + 1) * 10 + "rem",
        onComplete: checkWins
      });
    } else {
      this[move]();
      updateMove(null);
    }
  }

  onMounted(() => {
    for (let x = -1; x < boardSize + 1; x++) {
      for (let y = -1; y < boardSize + 1; y++) {
        const hole = document.createElement("div");
        hole.className = "hole";
        hole.id = "h" + x + "_" + y;
        gsap.set(hole, {
          x: x * 10 + "rem",
          y: y * 10 + "rem"
        });
        document.getElementById("holes").appendChild(hole);
      }
    }

    const canvas = document.getElementById("holeCanvas");
    canvas.width = boardSize * 200;
    canvas.height = canvas.width;
    const ctx = canvas.getContext("2d");

    function drawHole(x, y, scale) {
      ctx.globalCompositeOperation = "source-over"; 
      ctx.fillStyle = "darkblue";
      ctx.beginPath();
      ctx.arc(100 + 20 * x, 100 + 20 * y, 90 * scale, 0, 2 * Math.PI);
      ctx.fill();
      ctx.globalCompositeOperation = "destination-out";
      ctx.beginPath();
      ctx.arc(100 + 20 * x, 100 + 20 * y, 80 * scale, 0, 2 * Math.PI);
      ctx.fill();
      ctx.beginPath();
    }

    function drawHoles() {
      ctx.globalCompositeOperation = "source-over"; 
      ctx.fillStyle = "blue";
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      document.querySelectorAll(".hole").forEach((hole) => drawHole(hole._gsap.x.slice(0, -3), hole._gsap.y.slice(0, -3), hole._gsap.scaleX));
      requestAnimationFrame(drawHoles);
    }

    requestAnimationFrame(drawHoles);

    for (let x = 0; x < boardSize; x++) {
      const arrows = document.createElement("div");
      arrows.className = "arrows";
      arrows.id = "a" + x;
      document.getElementById("arrowsWrapper").appendChild(arrows);
      const column = document.createElement("div");
      column.className = "column";
      column.addEventListener("mouseover", () => document.getElementById("a" + x).style.visibility = "visible");
      column.addEventListener("mouseout", () => document.getElementById("a" + x).style.visibility = "hidden");
      column.addEventListener("click", () => {makeMove(x)});
      document.getElementById("columns").appendChild(column);
    }
  });
</script>

<template>
  <Player v-bind:active="currentPlayer == 2 && winner == null" v-bind:color="colors[1]" v-bind:name="names[1]" v-bind:selected-move="selectedMoves[1]" @select-move="(move) => selectedMoves[1] = move"></Player>
  <div id="board">
    <div id="arrowsWrapper"></div>
    <div id="pieces"></div>
    <canvas id="holeCanvas"></canvas>
    <div id="holes"></div>
    <div id="columns"></div>
    <div id="moves" @click="makeMove(hoveredMove)" @mouseleave="updateMove(null)" :class="{hidden: !canMove || selectedMoves[currentPlayer - 1] == 'drop'}">
      <div id="moveImage"></div>
      <div class="moveColumn">
        <div @mouseenter="updateMove('lu')"></div>
        <div @mouseenter="updateMove('lm')"></div>
        <div @mouseenter="updateMove('ld')"></div>
      </div>
      <div class="moveColumn">
        <div @mouseenter="updateMove('mu')"></div>
        <div @mouseenter="updateMove('mm')"></div>
        <div @mouseenter="updateMove('md')"></div>
      </div>
      <div class="moveColumn">
        <div @mouseenter="updateMove('ru')"></div>
        <div @mouseenter="updateMove('rm')"></div>
        <div @mouseenter="updateMove('rd')"></div>
      </div>
    </div>
  </div>
  <dialog id="dialog">
    <h1 v-if="winner == 'tie'">Game Tied</h1>
    <h1 v-else>{{ names[winner - 1] }} Won!</h1>
    <div id="reset" @click="reset">Play Again</div>
  </dialog>
  <Player v-bind:active="currentPlayer == 1 && winner == null" v-bind:color="colors[0]" v-bind:name="names[0]" v-bind:selected-move="selectedMoves[0]" @select-move="(move) => selectedMoves[0] = move"></Player>
</template>

<style>
  #board {
    display: flex;
    width: calc(var(--board-size) * 10rem);
    height: calc(var(--board-size) * 10rem);;
    background-color: white;
    border: 0.5rem solid darkblue;
  }

  @keyframes fall {
    from {
      background-position: 0 0;
    }
    to {
      background-position: 0 10rem;
    }
  }

  #arrowsWrapper {
    position: absolute;
    display: flex;
  }

  .arrows {
    visibility: hidden;
    width: 10rem;
    height: calc(var(--board-size) * 10rem);
    background-image: url(assets/arrows.png);
    background-size: contain;
    animation: 1s linear infinite fall;
  }

  #pieces {
    position: absolute;
    width: calc(var(--board-size) * 10rem);
    height: calc(var(--board-size) * 10rem);
    overflow: hidden;
  }

  .piece {
    position: absolute;
    width: 7rem;
    height: 7rem;
    margin: 0.5rem;
    border-radius: 50%;
    border: 1rem solid rgba(0,0,0, 0.25);
    transition: border-color 0.5s;
  }

  #holeCanvas {
    position: absolute;
    width: calc(var(--board-size) * 10rem);
    height: calc(var(--board-size) * 10rem);
  }

  #holes {
    position: absolute;
    display: flex;
    flex-wrap: wrap;
    width: calc(var(--board-size) * 10rem);
    height: calc(var(--board-size) * 10rem);
    overflow: hidden;
  }

  .hole {
    position: absolute;
    width: 10rem;
    height: 10rem;
  }

  #columns {
    display: flex;
    z-index: 1;
  }

  .column {
    width: 10rem;
    height: calc(var(--board-size) * 10rem);
  }

  #moves {
    position: absolute;
    display: flex;
    width: calc(var(--board-size) * 10rem);
    height: calc(var(--board-size) * 10rem);
    z-index: 2;
    opacity: 0.75;
  }

  #moveImage {
    position: absolute;
    z-index: -1;
    background-size: cover;
    width: 100%;
    height: 100%;
  }
  
  .moveColumn {
    display: flex;
    flex-direction: column;
    width: 33.33%;
    height: 100%;

    div {
      height: 100%;
    }
  }

  .hidden {
    visibility: hidden;
  }

  dialog {
    background-color: rgba(0, 0, 0, 0.85);
    outline: none;
    border: 0.5rem solid darkblue;
  }

  #reset {
    background-color: green;
    font-size: 1.5rem;
    text-align: center;
    cursor: pointer;
  }
</style>
