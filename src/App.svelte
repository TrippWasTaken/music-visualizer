<script lang="ts">
    import { fade } from 'svelte/transition';
import Cursor from './Cursor.svelte';
    import Player from "./player.svelte";

    let action = false
    const init = () =>{
        action = true
    }
</script>
<main>
    <Cursor />
    {#if !action}
    <div class="pageStart" on:click|once = {() => init()} transition:fade>
        <h1>Welcome</h1>
        <span><b>Click</b> on the screen to start the Visualizer</span>
        <span>Song select can be found by hovering to the left edge of the screen</span>
        <span>For best performace make sure Hardware <b>acceleration is turned on</b></span>
        
        <h3>Controls</h3>
        <span><b>Space/Mouse Click</b>: Play/Pause</span>
        <span>click on the bottom progressbar area to skip through song</span>
    </div>
    {:else}
    <section transition:fade>
        <Player />
    </section>
    {/if}
</main>

<style lang="scss">
    .pageStart{
        position: fixed;
        top: 0;
        left: 0;
        width: 100vw;
        height: 100vh;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        color:white;
    }

    :global(body){
        background-color: black;
        overflow: hidden;
        cursor: none;
    }
</style>
