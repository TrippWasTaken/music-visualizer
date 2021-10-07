<script lang="ts">
    import { onMount } from "svelte/internal";
    import { EffectComposer } from "three/examples/jsm/postprocessing/EffectComposer.js";
    import { UnrealBloomPass } from "three/examples/jsm/postprocessing/UnrealBloomPass.js";
    import { RenderPass } from "three/examples/jsm/postprocessing/RenderPass.js";

    import * as THREE from "three";
    import { Vector2 } from "three";

    //export let song: any;
    let ref: Element;
    export let audioDiv: HTMLMediaElement;
    const context = new window.AudioContext();

    //Three set up
    const scene = new THREE.Scene();
    const width = window.innerWidth;
    const height = window.innerHeight;
    const camera = new THREE.PerspectiveCamera(100, width / height, 0.1, 5000);
    camera.position.set(0, 0, 250);

    const renderer = new THREE.WebGLRenderer({
        antialias: true,
        alpha: true,
        preserveDrawingBuffer: true,
    });

    renderer.setClearColor(0x000000, 0);
    renderer.setSize(width, height);
    const lines = new THREE.Group();

    export const resumeContext = () => {
        context.resume();
    };

    const getLoudness = (arr: any) => {
        let sum = 0;
        for (let i = 0; i < arr.length; i++) {
            sum += arr[i];
        }
        return sum / arr.length;
    };

    //console.log(audioDiv)
    onMount(() => {
        //Audio set up
        //console.log(audioDiv)
        const analyser = context.createAnalyser();
        const source = context.createMediaElementSource(audioDiv);
        source.connect(analyser);
        analyser.connect(context.destination);
        analyser.smoothingTimeConstant = 0.7;
        analyser.maxDecibels = -20;
        analyser.fftSize = 1024;
        const bufferLength = analyser.frequencyBinCount;
        const dataArray = new Uint8Array(bufferLength);
        ref.appendChild(renderer.domElement);

        //main
        const addLines = () => {
            let z = 0;
            for (let l = 0; l < 512; l++) {
                const material = new THREE.LineBasicMaterial();
                const geometry = new THREE.BufferGeometry();
                const line = new THREE.Line(geometry, material);
                line.frustumCulled = false;
                line.translateZ(z);
                lines.add(line);
                z -= 1;
            }
        };
        scene.add(lines);
        const dataC = [];

        const moveL = (dataAudio: any) => {
            const dataA = dataAudio;
            const dataB = dataAudio.slice().reverse();
            const data = [...dataB, ...dataA];
            const positions = new Float32Array(data.length * 3);
            const loudness = Math.max(...dataAudio);
            for (let i = dataC.length - 1; i > 0; i--) {
                const current =
                    lines.children[i].geometry.getAttribute("position");
                const l =
                    ((dataC.length - i) / dataC.length) *
                    (loudness / 255) *
                    360;
                if (current) {
                    current.array = dataC[i - 1];
                    lines.children[i].material.color = new THREE.Color(
                        "hsl(" + l + ", 70%, 20%)"
                    );
                    current.needsUpdate = true;
                } else {
                    lines.children[i].geometry.setAttribute(
                        "position",
                        new THREE.BufferAttribute(dataC[i - 1], 3)
                    );
                }
            }

            for (let y = 0; y < data.length; y++) {
                positions[3 * y + 1] = data[y] * 2; //y
                positions[3 * y] = (width / data.length) * y; //x
            }

            dataC.unshift(positions);
            if (dataC.length > lines.children.length) {
                dataC.pop();
            }
        };

        //Post Processing
        const composer = new EffectComposer(renderer);
        composer.setSize(width, height);
        composer.addPass(new RenderPass(scene, camera));
        const res = new Vector2(width, height);
        const bloomPass = new UnrealBloomPass(res, 1, 1, 0);
        bloomPass.exposure = 1;
        composer.addPass(bloomPass);

        const animate = () => {
            analyser.getByteFrequencyData(dataArray);
            moveL(dataArray);
            camera.rotation.z +=
                getLoudness(dataArray) / (dataArray.length * 60);
            composer.render();
            requestAnimationFrame(animate);
        };

        //Init
        addLines();
        lines.translateX(-width / 2);
        const lines2 = lines.clone();
        lines2.rotation.z = Math.PI;
        lines2.translateX(-width);
        scene.add(lines2);
        animate();
    });
</script>

<!-- 
<svelte:window on:keydown={toggleAudio} /> -->

<main>
    <section class="audioCanvas" bind:this={ref} />
</main>

<style type="scss">
    .audioCanvas {
        position: fixed;
        top: 0;
        left: 0;
    }
</style>
