<script lang="ts">
import { onMount, src_url_equal } from "svelte/internal";

	import * as THREE from "three"

	const song = "./media/song2.mp3"
	let ref: Element
	let audioDiv: HTMLMediaElement
	const context = new window.AudioContext()

	const resumeContext = () =>{
		context.resume()
	}

	const getLoudness = (arr) =>{
		var sum = 0;
        for( var i = 0; i < arr.length; i ++ ) {
            sum += arr[i];
        }
        return sum / arr.length;
	}
	
	//Three set up
	const scene = new THREE.Scene()
	const camera = new THREE.PerspectiveCamera(100, window.innerWidth / window.innerHeight, 0.01, 5000)
	camera.position.set(0, 0, 800)
	const renderer = new THREE.WebGLRenderer({
		antialias: true,
		alpha: true,
		preserveDrawingBuffer: true,
	});
	renderer.setClearColor(0x000000, 0)
	renderer.setSize(window.innerWidth, window.innerHeight)
	const lines = new THREE.Group()
	
	onMount(()=>{
		//Audio set up
        const analyser = context.createAnalyser()
        const source = context.createMediaElementSource(audioDiv)
        source.connect(analyser)
        analyser.connect(context.destination)
        analyser.smoothingTimeConstant = 0.9
        analyser.minDecibels = -100
        analyser.maxDecibels = -10
        analyser.fftSize = 512
        const bufferLength = analyser.frequencyBinCount
        const dataArray = new Uint8Array(bufferLength)
		
		ref.appendChild(renderer.domElement)

		//main
		const addLines = () => {
            let z = 0
            for (let l = 0; l < 1000; l++) {
                const material = new THREE.LineBasicMaterial()
				const geometry = new THREE.BufferGeometry()
                const line = new THREE.Line(geometry, material)
                line.translateZ(z)
                lines.add(line)
                z -= 10
            }
        }
        scene.add(lines);
        const dataC = []

        const moveL = (dataAudio) => {
            const dataA = dataAudio
            const dataB = dataAudio.slice().reverse()
            const data = [...dataB, ...dataA]
            const positions = new Float32Array(data.length * 3)
            const loudness = Math.max(...dataAudio)
            for (let i = dataC.length - 1; i > 0; i--) {
                const current = lines.children[i].geometry.getAttribute('position')
                const l = (dataC.length - i) / dataC.length * loudness
                if (current) {
                    current.array = dataC[i - 1]
                    lines.children[i].material.color = new THREE.Color('hsl(' + l + ', 100%, 50%)')
                    current.needsUpdate = true
                } else {
                    lines.children[i].geometry.setAttribute('position', new THREE.BufferAttribute(dataC[i - 1], 3))
                }
            }

            for (let y = 0; y < data.length; y++) {
                positions[3 * y + 1] = data[y]//y
                positions[3 * y] = ((window.innerWidth) / (data.length) * y)   //x
            }

            lines.children[0].geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
            //lines.children[0].material.color = new THREE.Color('hsl(' + loudness + ', 100%, 50%)')

            dataC.unshift(positions)
            if (dataC.length > lines.children.length) {
                dataC.pop()
            }
        }

        let last = 0
        const animate = (now: any) => {
            if (!last || now - last >= 5) {
                analyser.getByteFrequencyData(dataArray)
                last = now
                moveL(dataArray)
            }
			camera.rotation.z += getLoudness(dataArray)/(4096 * 2)
            renderer.render(scene, camera)
            requestAnimationFrame(animate)
        };
        
        addLines()
		lines.translateX(-window.innerWidth/2)
		const lines2 = lines.clone();
        lines2.rotation.z = Math.PI;
		lines2.translateX(-window.innerWidth)
        scene.add(lines2);
        animate(last)
	})


</script>

<main>
	<section class="musicSect">
		<section class="audioCanvas" bind:this={ref}>

		</section>

		<div class="audioPlayer">
			<audio id="songSource" src={song} controls on:play="{resumeContext}" bind:this={audioDiv}>
			</audio>
		</div>
	</section>
</main>

<style type="scss">
	.musicSect{
		height: 100%;
		width: 100%;
		overflow: hidden;
		background: #C827B1;
		background: radial-gradient(ellipse at center, #C827B1, #825E39);
	}
	
	.audioCanvas{
		width: 100vw;
		height: 100vh;
	}

</style>