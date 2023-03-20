# Games

This is a repo for a collection of in-browser games. Currently it includes:

- Snake
- Tic Tac Toe

---

## Snake

[Play in the browser](https://snake.mcbrid.es)

I wanted a reason to play around with Svelte, and snake in the browser seemed like a fun thing to build. Controls are pretty simple:

- Press Start Game and the snake will start moving.
- Press an arrow key to move the snake around.
- Try to eat the apple. You eat the apple, the snake gets longer.
- Don't run into yourself.
- Walls loop around to the other side.

### Things I'm considering

- [ ] Adding a toggle to make wall collisions end the game, if you want "hard mode". Wouldn't be hard to implement, but probably won't do it.
- [X] Adding a slider to adjust the speed of the snake. (Choose from slow/medium/fast)
- [X] Mobile controls, maybe a virtual keypad? (This is done, but lots of room for improvement)

---

## Tic Tac Toe

It's just tic-tac-toe. Player X starts, then player O goes next. When someone wins, the other person starts the next game. Total wins are kept track of on the page.
