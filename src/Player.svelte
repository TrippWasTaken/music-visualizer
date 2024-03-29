<script lang="ts">
    
    import Player from "./Visuals.svelte";

    const songs = [
        {
            name: "Life and its end",
            artist: "Solyeong",
            src: "./media/1.mp3",
        },
        {
            name: "人生の不思議",
            artist: "solyeong",
            src: "./media/2.mp3",
        },
        {
            name: "Beyond It All, Lies Nothing But Isolation",
            artist: "Solyeong",
            src: "./media/3.mp3",
        },
        {
            name: "I MISS THE RAGE",
            artist: "Solyeong ft. Mario Judah",
            src: "./media/4.mp3",
        },
    ];

    let currSong = songs[0];
    let audioTag: HTMLMediaElement;
    let time: number = 0;
    let duration: any;
    let paused = true;
    let timer: number;
    let timerWidth: number;
    let curr: number = 0;

    const changeSong = (index: number) => {
        currSong = songs[index];
        curr = index;

        let canPlay = audioTag.play();
        if (canPlay != undefined) {
            canPlay.then(() => {
                audioTag.play();
            });
        }
    };

    const handleKey = (e: any) => {
        if (e.code === "Space" || e.key === " ") {
            if (paused) audioTag.play();
            else audioTag.pause();
        }
    };

    const handleClick = (e: any) => {
        if (paused) audioTag.play();
        else audioTag.pause();
    };

    const updateProgress = () => {
        timer = (time / duration) * 100;
        requestAnimationFrame(updateProgress);
    };

    const switchSong = () => {
        curr++;
        if (curr < songs.length) {
            changeSong(curr);
        } else {
            curr = 0;
            changeSong(curr);
        }
    };

    const changeTime = (e: any) => {
        const c = e.clientX;
        const change = c / timerWidth;
        time = duration * change;
    };
</script>

<svelte:window on:keydown={handleKey} />

<main>
    <section class="songSelect">
        {#each songs as song, index}
            <div class="songWrapper" on:click={() => changeSong(index)}>
                <span class="songArtist">{song.artist}</span> -
                <span class="songName">{song.name}</span>
            </div>
        {/each}
    </section>

    <audio
        id="songSource"
        src={currSong.src}
        bind:this={audioTag}
        bind:currentTime={time}
        bind:duration
        bind:paused
        on:playing={updateProgress}
        on:ended={switchSong}
    />
    {#if audioTag != null}
        <section
            class="visArea"
            on:click={handleClick}
            bind:clientWidth={timerWidth}
        >
            <Player audioDiv={audioTag} />
        </section>
    {:else}
        <h2>Loading player</h2>
    {/if}

    <section class="audioControls">
        <div class="progressBar" on:click={changeTime}>
            <div class="nowPlaying">{currSong.artist} - {currSong.name}</div>
            <div class="controlsProgress" style="width: {timer}%;" />
        </div>
    </section>
</main>

<style lang="scss">
    @import url("https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap");

    .visArea {
        position: fixed;
        width: 100vw;
        height: 100vh;
        z-index: 0;
    }
    .songWrapper {
        font-family: "Montserrat", sans-serif;
        color: white;
        font-size: 18px;
        font-weight: 500;
        padding: 1rem;
    }
    .songSelect {
        height: 100%;
        width: 20%;
        position: fixed;
        z-index: 1;
        top: 0;
        left: 0;
        opacity: 0;
        transition: 0.5s;
        transition-timing-function: cubic-bezier(0.5, 0, 1, 0.5);
        padding: 10px;
        transform: translateX(-95%);
        display: flex;
        flex-direction: column;
        justify-content: center;
        backdrop-filter: blur(10px);
        border-right: 1rem solid white;
        &:hover {
            transform: translateX(0);
            opacity: 1;
            transition-timing-function: cubic-bezier(0, 0.5, 0.5, 1);
        }
    }
    .songSelect:not(:hover){
        animation: anim 5s ease-out;
    }

    @keyframes anim{
        from {opacity: 100%;}
        to {opacity: 0;}
    }

    .audioControls {
        position: fixed;
        width: 100%;
        position: fixed;
        bottom: 0%;
        left: 0;
        color: white;
    }

    .progressBar {
        z-index: 1;
        opacity: 0.7;
    }

    .nowPlaying {
        text-align: center;
        font-size: 2rem;
        font-weight: bolder;
        opacity: 0.2;
    }

    .controlsProgress {
        height: 0.5rem;
        width: 0;
        background-color: white;
    }
</style>
