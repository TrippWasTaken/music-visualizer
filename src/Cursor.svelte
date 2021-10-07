<script lang="ts">
    import { spring } from 'svelte/motion';
    let size = spring(1);

    let cursor
    
    const moveCur = (e) =>{
        const x =  e.x
        const y = e.y
        cursor.setAttribute("style", "top: " + (y - 10)+ "px; left: " + (x - 10) + "px;")
    }

    const mUp = (e) =>{
        cursor.classList.remove("expand")
    }

    const mDown = (e) =>{
        cursor.classList.add("expand")
    }
    
</script>

<svelte:window on:mousemove="{moveCur}" on:mouseup={mUp} on:mousedown={mDown}></svelte:window>
<main>
    <div class="cursor" bind:this="{cursor}"></div>
</main>

<style lang="scss">
    .cursor{
        z-index: 5;
        width: 20px;
        height: 20px;
        background-color: white;
        border-radius: 50%;
        position: absolute;
        pointer-events: none;
        opacity: 0.8;
    }

    :global(.expand){
        transform: scale(1.4);
        opacity: 1;
    }

</style>