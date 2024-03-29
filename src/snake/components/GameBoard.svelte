<script>
    import Cell from "./Cell.svelte";
    import GameControls from "./GameControls.svelte";

    const SIZE = 20;
    const DIRECTIONS = {
        UP: { code: "ArrowUp", x: 0, y: -1, disallow: "ArrowDown" },
        DOWN: { code: "ArrowDown", x: 0, y: 1, disallow: "ArrowUp" },
        LEFT: { code: "ArrowLeft", x: -1, y: 0, disallow: "ArrowRight" },
        RIGHT: { code: "ArrowRight", x: 1, y: 0, disallow: "ArrowLeft" },
    };
    const ALLOWED_KEYS = ["ArrowUp", "ArrowDown", "ArrowLeft", "ArrowRight"];
    const KEYS = {
        ArrowUp: "UP",
        ArrowDown: "DOWN",
        ArrowLeft: "LEFT",
        ArrowRight: "RIGHT",
    };
    const STATUS = {
        STOPPED: "stopped",
        RUNNING: "running",
        GAME_OVER: "game over",
    };

    const cells = [...new Array(SIZE).keys()];
    $: currentDirection = DIRECTIONS.UP;
    $: currentStatus = STATUS.STOPPED;
    $: score = 0;
    $: speed = 150;

    $: snakeCoordinates = [
        { x: 10, y: 10 },
        { x: 10, y: 11 },
        { x: 10, y: 12 },
    ];

    $: fruitCoordinates = { x: 4, y: 4 };

    function sleep(ms) {
        return new Promise((resolve) => setTimeout(resolve, ms));
    }

    function startGame() {
        score = 0;
        currentStatus = STATUS.RUNNING;
        next();
    }

    function restartGame() {
        snakeCoordinates = [
            { x: 10, y: 10 },
            { x: 10, y: 11 },
            { x: 10, y: 12 },
        ];

        fruitCoordinates = { x: 4, y: 4 };
        currentDirection = DIRECTIONS.UP;
        startGame();
    }

    function stopGame(status = STATUS.STOPPED) {
        currentStatus = status;
    }

    async function next() {
        if (currentStatus === STATUS.RUNNING) {
            moveSnake();
            await sleep(speed);
            next();
        }
    }

    function isIntersectingSnake({ x, y }) {
        return snakeCoordinates.some((coord) => {
            return x === coord.x && y === coord.y;
        });
    }

    function getNextSnakeCoordinates() {
        let [head] = snakeCoordinates;

        let newHead = {
            x: head.x + currentDirection.x,
            y: head.y + currentDirection.y,
        };

        // allow looping to other side of board
        if (newHead.x === -1) {
            newHead.x = 19;
        }
        if (newHead.y === -1) {
            newHead.y = 19;
        }
        if (newHead.x === 20) {
            newHead.x = 0;
        }
        if (newHead.y === 20) {
            newHead.y = 0;
        }

        // if we hit the snake, we are done.
        if (isIntersectingSnake(newHead)) {
            return null;
        }

        // if we hit the fruit, move the fruit and add one cell to our snake
        if (
            newHead.x === fruitCoordinates.x &&
            newHead.y === fruitCoordinates.y
        ) {
            score++;
            moveFruit();
            return [newHead, ...snakeCoordinates];
        }

        // otherwise, just move every item up one position
        return [newHead, ...snakeCoordinates.slice(0, -1)];
    }

    async function moveSnake() {
        let newSnake = getNextSnakeCoordinates();
        if (!newSnake) {
            stopGame(STATUS.GAME_OVER);
        } else {
            snakeCoordinates = [...newSnake];
        }
    }

    function moveFruit() {
        let newCoordinates = {};
        let valid = false;
        while (!valid) {
            newCoordinates = {
                x: Math.floor(Math.random() * SIZE),
                y: Math.floor(Math.random() * SIZE),
            };

            valid =
                !isIntersectingSnake(newCoordinates) &&
                newCoordinates.x !== fruitCoordinates.x &&
                newCoordinates.y !== fruitCoordinates.y;
        }
        fruitCoordinates = newCoordinates;
    }

    function setSpeed(val) {
        speed = val;
    }

    function handleKeypress(event) {
        let keyCode = event.detail ? event.detail.code : event.code
        if (currentStatus === STATUS.RUNNING) {
            if (ALLOWED_KEYS.includes(keyCode)) {
                if (keyCode !== currentDirection.disallow) {
                    currentDirection = DIRECTIONS[KEYS[keyCode]];
                }
            }
        }
    }
</script>

<svelte:window on:keydown={handleKeypress} />

<h1 class="text-xl md:text-3xl font-bold mt-8 sm:mt-20 text-center">
    🐍 Snake 🐍
</h1>

<div class="mt-8 sm:mt-16 relative flex flex-col items-center justify-center">
    <span class="isolate inline-flex rounded-md shadow-sm mb-12">
      <button
            type="button"
            on:click={() => setSpeed(200)}
            class={`${speed === 200 ? 'bg-blue-600 text-white ring-blue-600' : 'bg-white text-gray-900 ring-gray-300 hover:bg-gray-50'} relative inline-flex items-center rounded-l-md px-3 py-2 text-sm font-semibold ring-1 ring-inset focus:z-10 focus:outline-none focus:ring-blue-600`}
        >Slow</button>
      <button
            type="button"
            on:click={() => setSpeed(150)}
            class={`${speed === 150 ? 'bg-blue-600 text-white ring-blue-600' : 'bg-white text-gray-900 ring-gray-300 hover:bg-gray-50'} -ml-px relative inline-flex items-center px-3 py-2 text-sm font-semibold ring-1 ring-inset focus:z-10 focus:outline-none focus:ring-blue-600`}
            >Medium</button>
      <button
            type="button"
            on:click={() => setSpeed(75)}
            class={`${speed === 75 ? 'bg-blue-600 text-white ring-blue-600' : 'bg-white text-gray-900 ring-gray-300 hover:bg-gray-50'} -ml-px relative inline-flex items-center rounded-r-md px-3 py-2 text-sm font-semibold ring-1 ring-inset focus:z-10 focus:outline-none focus:ring-blue-600`}
            >Fast</button>
    </span>

    {#each cells as y}
        <div class="flex items-center">
            {#each cells as x}
                <Cell
                    coordinate={{ x, y }}
                    snakeCoords={snakeCoordinates}
                    fruitCoords={fruitCoordinates}
                />
            {/each}
        </div>
    {/each}
    {#if currentStatus === "game over"}
        <div
            class="absolute top-1/2 transform -translate-y-1/2 -rotate-6 bg-red-600 text-white font-bold text-2xl px-6 py-2 uppercase"
        >
            Game Over
        </div>
    {/if}
</div>

<div class="mt-6 sm:mt-8 mx-auto flex flex-col items-center">
    <div class="mb-6">Score: {score}</div>
    {#if currentStatus === "stopped"}
        <button
            type="button"
            on:click={() => startGame()}
            class="inline-flex items-center rounded-md border border-transparent bg-emerald-100 px-3 py-2 text-sm font-medium leading-4 text-emerald-700 hover:bg-emerald-200 focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:ring-offset-2"
            >Start Game</button
        >
    {/if}
    {#if currentStatus === "running"}
        <GameControls on:click={handleKeypress} />
        <button
            type="button"
            on:click={() => stopGame()}
            class="mt-8 inline-flex items-center rounded-md border border-transparent bg-red-100 px-3 py-2 text-sm font-medium leading-4 text-red-700 hover:bg-red-200 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-offset-2"
            >Stop Game</button
        >
    {/if}
    {#if currentStatus === "game over"}
        <button
            type="button"
            on:click={() => restartGame()}
            class="inline-flex items-center rounded-md border border-transparent bg-emerald-100 px-3 py-2 text-sm font-medium leading-4 text-emerald-700 hover:bg-emerald-200 focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:ring-offset-2"
            >New Game</button
        >
    {/if}
</div>
