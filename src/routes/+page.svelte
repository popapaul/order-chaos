<script lang="ts">
    import { fade } from "svelte/transition";
    import "../app.css";
    import Piece from "./Piece.svelte";
    type Placement = -1 | 1;

    class Placements {
        matrix = $state<(Placement | 0)[]>([]);
        size = 0;
        winner = $state<"Order" | "Chaos">();
        constructor(size = 6) {
            this.size = size;
            this.matrix = Array(size * size).fill(0);
        }

        playTurn(index: number, value: Placement) {
            placements.matrix[index] = value;
            this.hasGameEnded();
        }

        getDiagonals(matrix: number[][]) {
            let row = 0,
                x = 0,
                y = 0,
                output = [];
            for (row = 0; row < matrix.length; row++) {
                const diagonal = [];
                for (y = row, x = 0; y >= 0; y--, x++)
                    diagonal.push(matrix[y][x]);
                output.push(diagonal);
            }
            for (row = 1; row < matrix[0].length; row++) {
                const diagonal = [];
                for (
                    y = matrix.length - 1, x = row;
                    x < matrix[0].length;
                    y--, x++
                )
                    diagonal.push(matrix[y][x]);
                output.push(diagonal);
            }
            return output;
        }

        verifyRow(pieces: number[], filler = 0) {
            let sum = 0;
            let lastPiece = 0;
            for (const piece of pieces) {
                const value = piece || filler;
                if (lastPiece != value) sum = 0;
                lastPiece = value;
                sum += value;

                if (Math.abs(sum) === 5) return true;
            }

            return false;
        }

        hasGameEnded() {
            const rows = [...new Array(this.size)].map((_, row) =>
                this.matrix.slice(row * this.size, row * this.size + this.size),
            );

            const cols = [...new Array(this.size)].map((_, index) =>
                rows.map((row) => row[index]).reverse(),
            );

            const arrays = [
                ...this.getDiagonals(rows),
                ...this.getDiagonals(cols),
                ...cols,
                ...rows,
            ].filter((x) => x.length >= 5);

            const hasOrderWon = arrays.some((x) => this.verifyRow(x));
            if (hasOrderWon) this.winner = "Order";

            const canOrderStillWin =
                arrays.some((x) => this.verifyRow(x, 1)) ||
                arrays.some((x) => this.verifyRow(x, -1));

            if (!canOrderStillWin) this.winner = "Chaos";
        }
    }

    let placements = $state(new Placements());

    const onPlacement = (index: number, value: Placement) => {
        placements.playTurn(index, value);
    };
    let pieces = $state(
        [...Array(24).fill(-1), ...Array(24).fill(1)].map((value) => ({
            value,
            position: { x: 0, y: 0 },
        })),
    );

    let width = $state(90);
    setTimeout(() => {
        width = document.querySelector("square").clientWidth;
    });

    const resetGame = () => {
        placements = new Placements();
        placements.winner = null;
        pieces.forEach((piece) => (piece.position = { x: 0, y: 0 }));
    };
    import { Confetti } from "svelte-confetti";
    $inspect(placements.winner);
</script>

{#if placements.winner}
    <div
        transition:fade
        style="position: fixed; top: -50px; left: 0; z-index:20; height: 100vh; width: 100vw; display: flex; justify-content: center; overflow: hidden;"
    >
        <Confetti
            x={[-5, 5]}
            y={[0, 0.1]}
            size={20}
            delay={[0, 500]}
            infinite
            duration={5000}
            amount={200}
            fallDistance="100vh"
        />
        <dialog
            class="absolute mx-0 top-1/2 left-1/2 flex flex-col bg-slate-200 rounded-md p-4"
            style="transform:translate(-50%,-50%)"
        >
            <h2 class="bold">
                {placements.winner} has won!!!
            </h2>
            <div class="flex justify-center mt-4">
                <button
                    type="button"
                    class="p-1 px-3 relative z-20 text-white bg-green-500 rounded"
                    onclick={resetGame}>Reset</button
                >
            </div>
        </dialog>
    </div>
{/if}

<div class="bg-slate-200 relative" style="--size:{width}px">
    <div class="absolute">
        <button type="button" onclick={resetGame}>Reset</button>
    </div>
    <div
        class="container items-center flex flex-col mx-auto min-h-screen justify-around"
    >
        <section class="relative -left-10 h-40 mt-10 w-30 flex">
            {#each pieces.filter((x) => x.value > 0) as piece}
                <div class="w-5 h-5">
                    <Piece {piece} {onPlacement} />
                </div>
            {/each}
        </section>
        <board
            style="grid-template-columns:repeat({placements.size}, minmax(0, 1fr));grid-template-rows:repeat({placements.size}, minmax(0, 1fr));"
            class="grid aspect-square mx-auto grow h-0 max-w-[800px] gap-1 bg-white"
        >
            {#each placements.matrix as piece, index}
                <square
                    bind:clientWidth={width}
                    {index}
                    class="bg-slate-700 rounded relative flex items-center justify-center cursor-pointer"
                >
                    <span class="absolute text-white z-10 left-2 top-2">
                        {piece}</span
                    >
                </square>
            {/each}
        </board>
        <section class="relative -left-10 h-40 mt-10 w-30 flex">
            {#each pieces.filter((x) => x.value <= 0) as piece}
                <div class="h-5 w-5">
                    <Piece {piece} {onPlacement} />
                </div>
            {/each}
        </section>
    </div>
</div>
