---
layout: post
title:  "ggchess: UI and Random Player"
date:   2022-07-24 20:00:00 -0700
categories: chess algorithms go ggchess ui javascript
--- 
This week, I built a simple chess player and web UI to visualize the game state. Eventually, I will implement a multiple players based on different game playing algorithms (Minimax, Alpha-beta pruning, Monte-carlo tree search, etc.). 

![Chessboard showing random play between both sides](https://storage.googleapis.com/oluko-blog/ggchess-random-play.gif){:style="display:block; margin-left:auto; margin-right:auto"}

## Setup
I used the following libraries:
* [chessboardjs][chessboardjs]: Client-side chessboard visualization
* [jhlywa/chess.js][jhlywa]: Client-side game state management
* [notnil/chess][notnil]: Server-side game state management

The folder structure looks like this:

```sh
ggchess/
├── static/
│   ├── css/
│   │   └── main.css # Minimal stylesheet for the board 
│   ├── img/
│   │   └── chesspieces/
│   │       └── wikipedia/
│   │           ├── bB.png 
│   │           └── # Images of chess pieces
│   ├── js/
│   │   └── main.js # Handles moves and updates client state 
│   └── index.html
└── server.go # Serves UI and handles move requests
```

## RandomPlayer
Every `AIPlayer` implements a `move` method that takes as input the game state (currently in [Forsyth–Edwards Notation][fen]) and outputs a proposed move. 

```go
// server.go

type AIPlayer interface {
  move(fen string) string
}
```

The `RandomPlayer` returns a random move from valid moves for the given state.
```go
// server.go

var (
  rp     = RandomPlayer{}
  ag     = chess.AlgebraicNotation{}
)

type RandomPlayer struct{}

func (rp *RandomPlayer) move(state string) string {
  fen, _ := chess.FEN(state)

  game := chess.NewGame(chess.UseNotation(ag), fen)
  moves := game.ValidMoves()
  // Select a random move.
  move := moves[rand.Intn(len(moves))]

  // Encode in standard algebraic notation (SAN).
  return ag.Encode(game.Position(), move)
}
```

## UI

At each turn, the client sends a `POST` request (`/move`) to get the next move. The request body is just the FEN of the game state. The server passes this state to the `RandomPlayer` to get the next move and returns the move to the client.

### Client
```js
// js/main.js

// Client-side state of the game.
var game = new Chess();
var board = Chessboard("#main-board", "start");

// Applies a move proposed by the server. 
function move() {
  if (game.game_over()) return;

  // Send a POST request to server. Server returns a proposed move.
  fetch('/move', {
    method: 'post',
    body: JSON.stringify({
      fen: game.fen(),
    })
  }).then(response => response.json())
    .then(data => {
      console.log("Received response: ", data);
      var move = data["Move"];

      // Apply the server's proposed move.
      game.move(move);
      board.position(game.fen());
    }).catch(function (err) {
      console.log(err)
    });
}
```

### Server
*I found [cosmtrek/air][air] useful for live reload during development.*
 
```go
// server.go

type MoveRequest struct {
  Fen string
}

type MoveResponse struct {
  Move string
}

func move(w http.ResponseWriter, r *http.Request) {
  switch r.Method {
  case "POST":
    var b MoveRequest
    // Try to decode the request body into the struct. If there is an error,
    // respond to the client with the error message and a 400 status code.
    if err := json.NewDecoder(r.Body).Decode(&b); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    mv := MoveResponse{
        Move: rp.move(b.Fen),
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(mv)
  default:
    fmt.Fprintf(w, "Only GET and POST methods are supported.")
  }
}

func main() {
  fs := http.FileServer(http.Dir("./static"))

  http.Handle("/", fs)
  http.HandleFunc("/move", move)

  fmt.Println("Running at localhost:8080")
  if err := http.ListenAndServe(":8080", nil); err != nil {
    log.Fatal(err)
  }
}
```

## Next Steps

I will be working on building better AI players, starting off with a [Minimax][mx] implementation.  

[fen]: https://en.wikipedia.org/wiki/Forsyth%E2%80%93Edwards_Notation 
[chessboardjs]: https://chessboardjs.com/ 
[jhlywa]: https://github.com/jhlywa/chess.js 
[notnil]: https://github.com/notnil/chess
[air]: https://github.com/cosmtrek/air
[mx]: https://en.wikipedia.org/wiki/Minimax