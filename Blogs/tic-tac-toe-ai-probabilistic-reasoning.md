
# Tic-Tac-Toe AI - Probabilistic Reasoning 

---

---

We have spent some part of our childhood playing the game of Tic-Tac-Toe in the back of out class copies. The idea of the game is for a player to get three of his/her character either vertically, horizontally or diagonally. The emulation of a player for this game can be easily implemented by using a random function to place the CPU's move in any blank space. But that would be no fun, since it wouldn't always be the smartest move. A way to tackle this problem would be to store every game played and refer to the stored game sets to avoid making the same mistake. This would be smart and the AI would be a learning one. However, this would be extensive since reaching the stage where the AI makes no mistake will be after a total of 255,168 unique games. 

> Fun fact: Out of 255,168 unique game, 131,184 are won by the first player, 77,904 are won by the second player, and 46,080 are drawn. [source](https://goo.gl/JY4XYq)
        
An alternate way to create a smarter CPU player is to use the <em>minimax algorithm</em> described in the <em>game theory</em>. Minimax is a sort of a backtracking algorithm that is commonly used in turn based game, like chess, checkers, etc. The idea behind this algorithm is very simple. There are two players, minimizer and and maximizer. The task of the minimizer to minimize the score and the maximizer to maximize the score. The rounds are played alternately, thus at the end, the maximizer has to maximize out of the minimized score. This give the CPU the best possible move for an optimized gameplay. The time complexity of this algorithm can be reduced by implementing the <em>alpha-beta pruning</em> which is a bounding technique to decrease the number of iterations.\n\nThe scoring is done as such; if the game state is terminated, i.e. no more moves are left, the game state is evaluated. If the maximizer wins the score is given some positive integer, otherwise if the minimizer wins a negative integer of equal magnitude is given to the score. If the game ends in a draw, the score assigned is zero. The algorithm of finding best move and minimax for tic-tac-toe:


```python
# Group: Algorithm_MinMax
# Title: Find best move 

findBestMove(game_state) :
    best_score = -infinity
    best_move = null
    for each move in game_state :
        new_game_state = mark move in game_state
        score = minimax(new_game_state,false)
        
        if score > best_score :
            best_score = score
            best_move = move
            unmark move in game_state

    return best_move
```

```python
# Group: Algorithm_MinMax
# Title: "Minimax"

minimax(game_state, isMaximizer) :
    if game_state is terminated :
        return score

    if isMaximizer :
        best_score = -infinity
        for every move in game_state :
            new_game_state = mark move in game_state
            score = minimax(new_game_state,false)
            if scorer &gt; best_score :
                best_score = score
                unmark move in game_state
                return best_score
            else :
                best_score = infinity
                for every move in game_state :
                    new_game_state = mark move in game_state
                    score = minimax(new_game_state,true)
                    if scorer &lt; best_score :
                        best_score = score
                        unmark move in game_state
        return best_score
```

After implementing this I found that for a given game state, at times, there are multiple moves with the same score. How then, am I to know which is the better move. The idea then was to calculate out the total number of victories (NOV), losses (NOL) and games (NOG) for each move in a given game state. With that I would be able to find the probability of victory (POV), loss (POL) and draw (POD) for each move in a given game state. Thus, after finding which, I had to only to check for the move with the lease POL and the move with the highest POV. By basic probability, it can be inferred that `~POL = POV+POD` for a move. Thus for a given set of moves with their POVs and POLs we can compute the best move as such:

```python
# Group: Algorithm_Probabilistic
# Title: Best Probabilistic Move

Let Move[] be an array of moves.
Let POV[] be the array of POVs for the respective moves.
Let POL[] be the array of POLs for the respective moves.

best_probabilistic_move(Move[], POV[], POL[]) :
    l = Move.length
    best_move = null
    for i in 1 to l:
        if 1-POV[i] > POV[i] :
            best_move = Move[i]
        else :
            best_move = Move[i]
    return best_move
```

From the above algorithm we can notice that, we only chose the move with the highest POV if it is greater than or equal to ~POL, because this marks the absolute surety of victory compared to ~POL, which is a combination of both POV and POD. It might so happen that there will be multiple POVs and POLs with the same values. If it occurs it will be best to select the least POL with the highest complimentary POV and, the highest POV with the least complimentary POL.

To further optimize the solution, it is necessary that the minimizer and the maximizer always plays the right move and avoids unnecessary moves. This involves the following points:

- Checking if there is a possibility to win with a single move.
- Checking if there is a possibility for the opponent to win with a single move and countering it.
- Checking if there is possibility to make a **fork** move.
- Checking if there is a possibility for the opponent to make a **fork**  move and countering it.

The possibility of victory can be easily checked by checking if two of the same characters are in sequence followed a black space, but what is a **fork** move? A fork is a move where a player gets two possible victory move by just playing one move, thus giving a sure shot victory at the game. This is a complicated method to check since it involves checking for several steps forwards. One way to implement this is by using brute force. With these enhancers, the game board evaluation can be optimized to a great precision.\n\nTo check out my project codes you can visit my Github Repo [here](https://github.com/SpandanBG/T3AI). The T3AI project is written in Java. The T3AI.java file can be easily implemented in any java application. The GameHandler.java file is a demo how it works. Please go through the documentation before implementing it in your own application.

To read more about the algorithms you can check out [minimax algorithm](http://www.geeksforgeeks.org/minimax-algorithm-in-game-theory-set-1-introduction) and [alpha-beta pruning](http://www.geeksforgeeks.org/minimax-algorithm-in-game-theory-set-4-alpha-beta-pruning).

---

Spandan Buragohain,
2017-04-22 18:30:15
    