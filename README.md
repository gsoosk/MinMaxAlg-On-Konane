<h1>Table of Contents<span class="tocSkip"></span></h1>
<div class="toc"><ul class="toc-item"><li><span><a href="#Introduction" data-toc-modified-id="Introduction-1"><span class="toc-item-num">1&nbsp;&nbsp;</span>Introduction</a></span></li><li><span><a href="#Prepared-Codes" data-toc-modified-id="Prepared-Codes-2"><span class="toc-item-num">2&nbsp;&nbsp;</span>Prepared Codes</a></span><ul class="toc-item"><li><span><a href="#Game-and-Player-Classes" data-toc-modified-id="Game-and-Player-Classes-2.1"><span class="toc-item-num">2.1&nbsp;&nbsp;</span>Game and Player Classes</a></span></li></ul></li><li><span><a href="#Implementation" data-toc-modified-id="Implementation-3"><span class="toc-item-num">3&nbsp;&nbsp;</span>Implementation</a></span><ul class="toc-item"><li><span><a href="#How-Algorithm-Works" data-toc-modified-id="How-Algorithm-Works-3.1"><span class="toc-item-num">3.1&nbsp;&nbsp;</span>How Algorithm Works</a></span><ul class="toc-item"><li><span><a href="#MIN-MAX" data-toc-modified-id="MIN-MAX-3.1.1"><span class="toc-item-num">3.1.1&nbsp;&nbsp;</span>MIN-MAX</a></span></li><li><span><a href="#Alpha-Betha" data-toc-modified-id="Alpha-Betha-3.1.2"><span class="toc-item-num">3.1.2&nbsp;&nbsp;</span>Alpha Betha</a></span></li></ul></li><li><span><a href="#Evaluation-Function" data-toc-modified-id="Evaluation-Function-3.2"><span class="toc-item-num">3.2&nbsp;&nbsp;</span>Evaluation Function</a></span></li><li><span><a href="#Does-Alpha-Betha-moves-are-different-from-Min-Max-?" data-toc-modified-id="Does-Alpha-Betha-moves-are-different-from-Min-Max-?-3.3"><span class="toc-item-num">3.3&nbsp;&nbsp;</span>Does Alpha Betha moves are different from Min-Max ?</a></span></li><li><span><a href="#Comparing-Alpha-Betha-And-Min-Max-Execution-time" data-toc-modified-id="Comparing-Alpha-Betha-And-Min-Max-Execution-time-3.4"><span class="toc-item-num">3.4&nbsp;&nbsp;</span>Comparing Alpha-Betha And Min-Max Execution time</a></span></li></ul></li></ul></div>

# Computer Assignment 2
Farzad Habibi - 810195383

## Introduction
In this project we started to learn games. For this project we used a prepared implementation of Konane Game. Then we implement our player with min-max methodology.

You can find this game in [this page](https://github.com/tommy-russoniello/konane/blob/master/konane.py).


## Prepared Codes

### Game and Player Classes


```python
import random
import copy

class GameError(AttributeError):
	pass

class Game:

	def __init__(self, n):
		self.size = n
		self.half_the_size = int(n/2)
		self.reset()

	def reset(self):
		self.board = []
		value = 'B'
		for i in range(self.size):
			row = []
			for j in range(self.size):
				row.append(value)
				value = self.opponent(value)
			self.board.append(row)
			if self.size%2 == 0:
				value = self.opponent(value)

	def __str__(self):
		result = "  "
		for i in range(self.size):
			result += str(i) + " "
		result += "\n"
		for i in range(self.size):
			result += str(i) + " "
			for j in range(self.size):
				result += str(self.board[i][j]) + " "
			result += "\n"
		return result

	def valid(self, row, col):
		return row >= 0 and col >= 0 and row < self.size and col < self.size

	def contains(self, board, row, col, symbol):
		return self.valid(row,col) and board[row][col]==symbol

	def countSymbol(self, board, symbol):
		count = 0
		for r in range(self.size):
			for c in range(self.size):
				if board[r][c] == symbol:
					count += 1
		return count

	def opponent(self, player):
		if player == 'B':
			return 'W'
		else:
			return 'B'

	def distance(self, r1, c1, r2, c2):
		return abs(r1-r2 + c1-c2)

	def makeMove(self, player, move):
		self.board = self.nextBoard(self.board, player, move)

	def nextBoard(self, board, player, move):
		r1 = move[0]
		c1 = move[1]
		r2 = move[2]
		c2 = move[3]
		next = copy.deepcopy(board)
		if not (self.valid(r1, c1) and self.valid(r2, c2)):
			raise GameError
		if next[r1][c1] != player:
			raise GameError
		dist = self.distance(r1, c1, r2, c2)
		if dist == 0:
			if self.openingMove(board):
				next[r1][c1] = "."
				return next
			raise GameError
		if next[r2][c2] != ".":
			raise GameError
		jumps = int(dist/2)
		dr = int((r2 - r1)/dist)
		dc = int((c2 - c1)/dist)
		for i in range(jumps):
			if next[r1+dr][c1+dc] != self.opponent(player):
				raise GameError
			next[r1][c1] = "."
			next[r1+dr][c1+dc] = "."
			r1 += 2*dr
			c1 += 2*dc
			next[r1][c1] = player
		return next

	def openingMove(self, board):
		return self.countSymbol(board, ".") <= 1

	def generateFirstMoves(self, board):
		moves = []
		moves.append([0]*4)
		moves.append([self.size-1]*4)
		moves.append([self.half_the_size]*4)
		moves.append([(self.half_the_size)-1]*4)
		return moves

	def generateSecondMoves(self, board):
		moves = []
		if board[0][0] == ".":
			moves.append([0,1]*2)
			moves.append([1,0]*2)
			return moves
		elif board[self.size-1][self.size-1] == ".":
			moves.append([self.size-1,self.size-2]*2)
			moves.append([self.size-2,self.size-1]*2)
			return moves
		elif board[self.half_the_size-1][self.half_the_size-1] == ".":
			pos = self.half_the_size -1
		else:
			pos = self.half_the_size
		moves.append([pos,pos-1]*2)
		moves.append([pos+1,pos]*2)
		moves.append([pos,pos+1]*2)
		moves.append([pos-1,pos]*2)
		return moves

	def check(self, board, r, c, rd, cd, factor, opponent):
		if self.contains(board,r+factor*rd,c+factor*cd,opponent) and \
		   self.contains(board,r+(factor+1)*rd,c+(factor+1)*cd,'.'):
			return [[r,c,r+(factor+1)*rd,c+(factor+1)*cd]] + \
				   self.check(board,r,c,rd,cd,factor+2,opponent)
		else:
			return []

	def generateMoves(self, board, player):
		if self.openingMove(board):
			if player=='B':
				return self.generateFirstMoves(board)
			else:
				return self.generateSecondMoves(board)
		else:
			moves = []
			rd = [-1,0,1,0]
			cd = [0,1,0,-1]
			for r in range(self.size):
				for c in range(self.size):
					if board[r][c] == player:
						for i in range(len(rd)):
							moves += self.check(board,r,c,rd[i],cd[i],1,
												self.opponent(player))
			return moves

	def playOneGame(self, p1, p2, show):
		self.reset()
		while True:
			if show:
				print(self)
				print("player B's turn")
			move = p1.getMove(self.board)
			if move == []:
				print("Game over")
				return 'W'
			try:
				self.makeMove('B', move)
			except GameError:
				print("Game over: Invalid move by", p1.name)
				print(move)
				print(self)
				return 'W'
			if show:
				print(move)
				print(self)
				print("player W's turn")
			move = p2.getMove(self.board)
			if move == []:
				print("Game over")
				return 'B'
			try:
				self.makeMove('W', move)
			except GameError:
				print("Game over: Invalid move by", p2.name)
				print(move)
				print(self)
				return 'B'
			if show:
				print(move)

	def playNGames(self, n, p1, p2, show):
		first = p1
		second = p2
		for i in range(n):
			print("Game", i)
			winner = self.playOneGame(first, second, show)
			if winner == 'B':
				first.won()
				second.lost()
				print(first.name, "wins")
			else:
				first.lost()
				second.won()
				print(second.name, "wins")
			first.side, second.side = second.side, first.side
			first, second = second, first


class Player:
	name = "Player"
	wins = 0
	losses = 0
	def results(self):
		result = self.name
		result += " Wins:" + str(self.wins)
		result += " Losses:" + str(self.losses)
		return result
	def lost(self):
		self.losses += 1
	def won(self):
		self.wins += 1
	def reset(self):
		self.wins = 0
		self.losses = 0

	def initialize(self, side):
		abstract()

	def getMove(self, board):
		abstract()


class SimplePlayer(Game, Player):
	def initialize(self, side):
		self.side = side
		self.name = "Simple"
	def getMove(self, board):
		moves = self.generateMoves(board, self.side)
		n = len(moves)
		if n == 0:
			return []
		else:
			return moves[0]

class RandomPlayer(Game, Player):
	def initialize(self, side):
		self.side = side
		self.name = "Random"
	def getMove(self, board):
		moves = self.generateMoves(board, self.side)
		n = len(moves)
		if n == 0:
			return []
		else:
			return moves[random.randrange(0, n)]

class HumanPlayer(Game, Player):
	def initialize(self, side):
		self.side = side
		self.name = "Human"
	def getMove(self, board):
		moves = self.generateMoves(board, self.side)
		while True:
			print("Possible moves:", moves)
			n = len(moves)
			if n == 0:
				print("You must concede")
				return []
			index = input("Enter index of chosen move (0-"+ str(n-1) +
						  ") or -1 to concede: ")
			try:
				index = int(index)
				if index == -1:
					return []
				if 0 <= index <= (n-1):
					print("returning", moves[index])
					return moves[index]
				else:
					print("Invalid choice, try again.")
			except Exception as e:
				print("Invalid choice, try again.")
			
```

## Implementation


```python
import math # For INF
import time # For verbose
```


```python
class MinimaxPlayer(Game, Player):
    def initialize(self,
                   side,
                   verboseMovesTime=False,
                   maxDepth=3,
                   pruning=False,
                   name="MinMax",
                   playerNumberDiff=True,
                   playerMovementDiff=True,
                   playerCanMoveDiff=True,
                   moveCoeff=10, 
                   numCoeff=0.05,
                   canMoveCoeff=2):
        self.side = side
        self.name = name
        self.MAX_DEPTH = maxDepth
        self.playerNumberDiff = playerNumberDiff
        self.pruning = pruning
        self.playerMovementDiff = playerMovementDiff
        self.moveCoeff = moveCoeff
        self.numCoeff = numCoeff
        self.verboseMovesTime = verboseMovesTime
        self.moveCount = 0
        self.playerCanMoveDiff = playerCanMoveDiff
        self.canMoveCoeff = canMoveCoeff


    def getMove(self, board):
        s  = time.time()
        x =  self.minMaxValue(self.side, board, 0)[1]
        f = time.time()
        self.moveCount += 1
        if self.verboseMovesTime: print(f"\033[34m***\t move number \033[31m{self.moveCount}\033[34m in \033[31m{self.name}:{self.side}\033[34m \t\033[32m{f - s}\033[34m \t***\t".expandtabs(40))
        return x


    def minMaxValue(self, side, board, depth, maximum=True, alpha=-math.inf, betha=+math.inf):
        possibleMoves = self.generateMoves(board, side)
        if len(possibleMoves) == 0 or depth >= self.MAX_DEPTH:
            return self.evaluation(board, side, possibleMoves), []
        bestValue = -math.inf if maximum else math.inf
        bestMove = []
        for successorMove in possibleMoves:
            successorBoard = self.nextBoard(board, side, successorMove)
            successorFinalValue, successorBestMove = self.minMaxValue(self.opponent(side), successorBoard,
                                                                      depth+1, not maximum, alpha, betha)
            if (maximum and bestValue < successorFinalValue) or (not maximum and bestValue > successorFinalValue):
                bestValue, bestMove = successorFinalValue, successorMove
            if maximum and self.pruning: 
                if bestValue >= betha: return  bestValue, bestMove
                alpha = max(alpha, bestValue)
            if not maximum and self.pruning:
                if bestValue <= alpha: return bestValue, bestMove
                betha = min(betha, bestValue)
        return bestValue, bestMove

    def evaluation(self, board, side, possibleMoves):
        evalue = 0
        if self.playerNumberDiff: 
            sideNum =  sum([i.count(side) for i in board])
            opponentNum =  sum([i.count(self.opponent(side)) for i in board])
            evalue += self.numCoeff * (sideNum - opponentNum)
        if self.playerMovementDiff or self.playerCanMoveDiff:
            sideMoves = possibleMoves
            sidePlayerNumberCanMove = self.numberOfPlayersCanMove
            opponentMoves = self.generateMoves(board, self.opponent(side))
            opponentPlayerNumberCanMove = self.numberOfPlayersCanMove
            if self.playerMovementDiff:
                sideMovesNum = len(sideMoves)
                opponentMovesNum = len(opponentMoves)
                evalue += self.moveCoeff * (sideMovesNum - opponentMovesNum)
            if self.playerCanMoveDiff:
                evalue += self.canMoveCoeff * (sidePlayerNumberCanMove - opponentPlayerNumberCanMove)
        return evalue

    def generateMoves(self, board, player):
        self.numberOfPlayersCanMove = 0
        if self.openingMove(board):
            if player=='B':
                return self.generateFirstMoves(board)
            else:
                return self.generateSecondMoves(board)
        else:
            moves = []
            rd = [-1,0,1,0]
            cd = [0,1,0,-1]
            for r in range(self.size):
                for c in range(self.size):
                    if board[r][c] == player:
                        new_moves = []
                        for i in range(len(rd)):
                            new_moves += self.check(board,r,c,rd[i],cd[i],1,
                                                self.opponent(player))
                        moves += new_moves
                        self.numberOfPlayersCanMove += (1 if len(new_moves) > 0 else 0)      
            return moves
```


```python
%%time
game = Game(8)
minMax = MinimaxPlayer(8)
minMax.initialize('B',
                  name="min max",
                  pruning=False,
                  maxDepth=3,
                  verboseMovesTime=True)
alphaBetha = MinimaxPlayer(8)
alphaBetha.initialize('W',
                  name="alpha betha",
                  pruning=True,
                  maxDepth=4)

game.playNGames(1, minMax, alphaBetha, False)
```

    Game 0
    [34m***                                 move number [31m1[34m in [31mmin max:B[34m                                 [32m0.02996683120727539[34m           ***                                     
    [34m***                                 move number [31m2[34m in [31mmin max:B[34m                                 [32m0.08310294151306152[34m           ***                                     
    [34m***                                 move number [31m3[34m in [31mmin max:B[34m                                 [32m0.2507510185241699[34m            ***                                     
    [34m***                                 move number [31m4[34m in [31mmin max:B[34m                                 [32m0.3825361728668213[34m            ***                                     
    [34m***                                 move number [31m5[34m in [31mmin max:B[34m                                 [32m0.4077789783477783[34m            ***                                     
    [34m***                                 move number [31m6[34m in [31mmin max:B[34m                                 [32m0.17318081855773926[34m           ***                                     
    [34m***                                 move number [31m7[34m in [31mmin max:B[34m                                 [32m0.3354179859161377[34m            ***                                     
    [34m***                                 move number [31m8[34m in [31mmin max:B[34m                                 [32m0.2324199676513672[34m            ***                                     
    [34m***                                 move number [31m9[34m in [31mmin max:B[34m                                 [32m0.2282407283782959[34m            ***                                     
    [34m***                                 move number [31m10[34m in [31mmin max:B[34m                                [32m0.4063286781311035[34m            ***                                     
    [34m***                                 move number [31m11[34m in [31mmin max:B[34m                                [32m0.13660120964050293[34m           ***                                     
    [34m***                                 move number [31m12[34m in [31mmin max:B[34m                                [32m0.13772797584533691[34m           ***                                     
    [34m***                                 move number [31m13[34m in [31mmin max:B[34m                                [32m0.17061209678649902[34m           ***                                     
    [34m***                                 move number [31m14[34m in [31mmin max:B[34m                                [32m0.09305691719055176[34m           ***                                     
    [34m***                                 move number [31m15[34m in [31mmin max:B[34m                                [32m0.06393074989318848[34m           ***                                     
    [34m***                                 move number [31m16[34m in [31mmin max:B[34m                                [32m0.02207779884338379[34m           ***                                     
    [34m***                                 move number [31m17[34m in [31mmin max:B[34m                                [32m0.019346952438354492[34m          ***                                     
    [34m***                                 move number [31m18[34m in [31mmin max:B[34m                                [32m0.006260871887207031[34m          ***                                     
    [34m***                                 move number [31m19[34m in [31mmin max:B[34m                                [32m0.005793094635009766[34m          ***                                     
    [34m***                                 move number [31m20[34m in [31mmin max:B[34m                                [32m0.004127979278564453[34m          ***                                     
    [34m***                                 move number [31m21[34m in [31mmin max:B[34m                                [32m0.004498958587646484[34m          ***                                     
    [34m***                                 move number [31m22[34m in [31mmin max:B[34m                                [32m0.003900289535522461[34m          ***                                     
    [34m***                                 move number [31m23[34m in [31mmin max:B[34m                                [32m0.0016710758209228516[34m         ***                                     
    [34m***                                 move number [31m24[34m in [31mmin max:B[34m                                [32m0.004076242446899414[34m          ***                                     
    [34m***                                 move number [31m25[34m in [31mmin max:B[34m                                [32m0.00011897087097167969[34m        ***                                     
    Game over
    alpha betha wins
    CPU times: user 9.7 s, sys: 84.7 ms, total: 9.78 s
    Wall time: 10.4 s


### How Algorithm Works 
At first I inherited a Player from game and Player class. In this class I overrided `getMoves` and `generateMoves`.
In getMoves at first we call `minMax` method with depth 0 az maximizer. This method implement minimizer also. Runtime of them sperated by a flag which it's name is `maximum` (For maximizer it's true and for minimizer it's false). 


#### MIN-MAX
In min-max algorithm every time we check our successor and their min-max best value and select best value and move. This selection implied by max of successors for maximizer and min of them by minizer. 

#### Alpha Betha 
In Alpha Betha methodology we use two variables which their name is alpha and betha. This variables pruns part of tree which is useless. Alpha is max's best option on path to root and Betha is min's best option on path to root.

### Evaluation Function 
$$
EvaluationFunction(board)=0.05\times(numberOfSide - numberOfOpponent) + 10\times(numberOfMoves - numberOfOpponentMoves) + 2\times(numberOfPieceWhichCanMove - numberOfOpponentPieceWhichCanMove)
$$

We use number of remaining piece because if we have more of them it has more probabilty to have more moves in future. We also use moves because more moves have more selection in the future. And at the end if we have number of remaining piece which can move it is better at the future. 

### Does Alpha Betha moves are different from Min-Max ?
No it's not. alpha betha just is a pruning method which anticipate futrue by knowing that we will not select a movement in the future. Becasue if we be a maximizer we know less variables in subtree can not be usefull and so on for minimzer.

### Comparing Alpha-Betha And Min-Max Execution time

At first we play a game by two min-max with depth of 3 without pruning : 


```python
%%time
game = Game(8)
minMax1 = MinimaxPlayer(8)
minMax1.initialize('B',
                  name="min max",
                  pruning=False,
                  maxDepth=3)
minMax2 = MinimaxPlayer(8)
minMax2.initialize('W',
                  name="min max",
                  pruning=False,
                  maxDepth=3)

game.playNGames(1, minMax1, minMax2, False)
```

    Game 0
    Game over
    min max wins
    CPU times: user 10.5 s, sys: 63.1 ms, total: 10.5 s
    Wall time: 10.9 s


An then we play their game by pruning. According to the 3.3 alpha betha doesnt have compact on move selection. 


```python
%%time
game = Game(8)
alphaBetha1 = MinimaxPlayer(8)
alphaBetha1.initialize('B',
                  name="alpha betha",
                  pruning=True,
                  maxDepth=3)
alphaBetha2 = MinimaxPlayer(8)
alphaBetha2.initialize('W',
                  name="alpha betha",
                  pruning=True,
                  maxDepth=3)

game.playNGames(1, alphaBetha1, alphaBetha2, False)
```

    Game 0
    Game over
    alpha betha wins
    CPU times: user 4.09 s, sys: 32.8 ms, total: 4.12 s
    Wall time: 4.37 s


As you see min-max executed in 10.7 seconds but alpha betha executed in 4.24 seconds which is very better so :
$$
T(AlphaBetha) \leq T(MinMax)
$$
