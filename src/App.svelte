<script lang="ts">
    import { fade } from 'svelte/transition';
    import Player from "./player.svelte";

    let action = false
    const init = () =>{
        action = true
    }
</script>
<main>
    {#if !action}
    <div class="pageStart" on:click|once = {() => init()} transition:fade>
        <h1>Welcome</h1>
        <span><b>Click</b> on the screen to start the Visuazlier</span>
        <span>Song select can be found by hovering to the left edge of the screen</span>

        <h3>Controls</h3>
        <span><b>Space</b>: Play/Pause</span>
    </div>
    {:else}
    <section transition:fade>
        <Player></Player>
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
    }
</style>
