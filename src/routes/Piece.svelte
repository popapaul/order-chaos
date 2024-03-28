<script lang="ts">
    const memes = Object.entries(
        import.meta.glob<{ default: string }>("../memes/*", { query: "?raw" }),
    );

    type Placement = -1 | 1;
    let pieceEl: HTMLElement;
    import { draggable, type DragEventData } from "@neodrag/svelte";
    let {
        piece,
        onPlacement,
    }: {
        piece: { value: Placement; position: any };
        onPlacement: (index: number, value: Placement) => void;
    } = $props();

    const onDragEnd = (event: DragEventData) => {
        const box = pieceEl.getBoundingClientRect();
        const xCenter = (box.left + box.right) / 2;
        const yCenter = (box.top + box.bottom) / 2;

        const square = document
            .elementsFromPoint(xCenter, yCenter)
            .find((x) => x.tagName == "SQUARE");
        if (!square) return;

        const squareBox = square.getBoundingClientRect();
        const style = window.getComputedStyle(pieceEl);
        const matrix = new DOMMatrix(style.transform);

        const move = {
            x: box.x - squareBox.x,
            y: box.y - squareBox.y,
        };

        animate();
        pieceEl.style.transform = `translate3d(${matrix.e - move.x}px, ${matrix.f - move.y}px, 0px)`;

        onPlacement(Number(square.getAttribute("index")), piece.value);
    };

    const animate = (duration = 300) => {
        pieceEl.style.transition = `transform ${duration}ms ease-in-out`;
        setTimeout(() => {
            pieceEl.style.transition = "";
        }, duration + 100);
    };

    $effect(() => {
        piece.position && animate(1000);
    });
</script>

<piece
    bind:this={pieceEl}
    value
    class:bg-red-600={piece.value == 1}
    class:bg-blue-600={piece.value == -1}
    on:dblclick={() => (piece.position = { x: 0, y: 0 })}
    use:draggable={{
        onDragEnd,
        position: piece.position,
    }}
>
    {#await memes[Math.floor(Math.random() * memes.length)]
        [1]()
        .then((x) => x.default) then meme}
        {@html meme}
    {/await}
</piece>

<style>
    piece {
        border-radius: 100%;
        aspect-ratio: 1/1;
        cursor: grab;
        display: flex;
        width: var(--size);
        margin: 2px;
        position: relative;
        z-index: 2;
        box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.5);
    }
    :global(piece svg) {
        margin: auto;
        scale: 1.3;
        filter: invert(100%) saturate(1352%) brightness(1219%) contrast(119%);
    }
    piece::after {
        content: "";
        position: absolute;
        width: 80%;
        border-radius: 100%;
        opacity: 0.5;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        height: 80%;
        border: 2px white dotted;
    }
</style>
