<script>
	import { onMount } from "svelte";
	import Chart from "chart.js/auto";

	/* ======================================================
	   INTRO OVERLAY
	   ====================================================== */
	let introVisible = true;

	function hideIntro() {
		introVisible = false;
		window.removeEventListener("scroll", hideIntro);
	}

	onMount(() => {
		window.addEventListener("scroll", hideIntro, { passive: true });
	});

	/* ======================================================
	   ZOOM SEQUENCE LOGIC
	   ====================================================== */
	let canvas;
	let ctx;

	const startFrame = 0;
	const endFrame = 455;
	const frameCount = endFrame - startFrame + 1;

	const images = [];
	const frameSrc = (i) =>
		`/images/hunga_frames/hunga-zoom_${String(i).padStart(3, "0")}.jpeg`;

	// overlay state
	let showOverlay = false;
	let showOverlayA = false;
	let showOverlayB = false;

	onMount(() => {
		for (let i = startFrame; i <= endFrame; i++) {
			const img = new Image();
			img.src = frameSrc(i);
			images[i] = img;
		}

		ctx = canvas.getContext("2d");
		window.addEventListener("scroll", handleScroll);
		drawFrame(startFrame);
	});

	function drawFrame(i) {
		const img = images[i];
		if (!img) return;

		const cw = canvas.width;
		const ch = canvas.height;

		ctx.clearRect(0, 0, cw, ch);
		ctx.drawImage(img, 0, 0, cw, ch);
	}

	function handleScroll() {
		const scrollTop = window.scrollY;
		const zoomScrollLength = window.innerHeight * 4;

		// clamp 0–1
		const rawProgress = scrollTop / zoomScrollLength;
		const progress = Math.min(Math.max(rawProgress, 0), 1);

		const frameIndex =
			startFrame + Math.floor(progress * (frameCount - 1));

		drawFrame(frameIndex);

		// Overlay logic: only during zoom
		if (progress < 0.4 || progress >= 1) {
			// hidden before 40% and after zoom is done
			showOverlay = false;
			showOverlayA = false;
			showOverlayB = false;
		} else if (progress < 0.7) {
			// 40%–70% → first card
			showOverlay = true;
			showOverlayA = true;
			showOverlayB = false;
		} else {
			// 70%–100% → second card
			showOverlay = true;
			showOverlayA = false;
			showOverlayB = true;
		}
	}

	/* ======================================================
	   SEQUENCE SCROLL LOGIC (L1 → L8)
	   ====================================================== */

	let seqCanvas;
	let seqCtx;

	const seqFrames = 8;
	const seqImages = [];
	const seqSrc = (i) => `/images/hunga_frames/L${i}.png`;

	onMount(() => {
		for (let i = 1; i <= seqFrames; i++) {
			const img = new Image();
			img.src = seqSrc(i);
			seqImages[i] = img;
		}

		seqCtx = seqCanvas.getContext("2d");
		drawSeqFrame(1);

		window.addEventListener("scroll", handleSeqScroll);
	});

	function drawSeqFrame(i) {
		const img = seqImages[i];
		if (!img) return;

		const cw = seqCanvas.width;
		const ch = seqCanvas.height;

		seqCtx.clearRect(0, 0, cw, ch);
		seqCtx.drawImage(img, 0, 0, cw, ch);
	}

	function handleSeqScroll() {
		const section = document.querySelector(".sequence-wrapper");
		const rect = section.getBoundingClientRect();

		const viewH = window.innerHeight;
		const totalScroll = rect.height - viewH;

		const progress = Math.min(
			Math.max((viewH - rect.top) / totalScroll, 0),
			1
		);

		const frameIndex = 1 + Math.floor(progress * (seqFrames - 1));
		drawSeqFrame(frameIndex);
	}

	/* ======================================================
	   GLOBAL TEMPERATURE CHART DATA
	   ====================================================== */

	const tempLabels = [
		"Dec 21",
		"Jan 22",
		"Feb 22",
		"Mar 22",
		"Apr 22",
		"May 22",
		"Jun 22",
		"Jul 22",
		"Aug 22",
		"Sep 22",
		"Oct 22",
		"Nov 22",
		"Dec 22",
		"Jan 23"
	];

	const tempData = [
		0.83, 0.89, 0.85, 0.82, 0.8, 0.78, 0.76,
		0.78, 0.8, 0.82, 0.85, 0.87, 0.89, 0.87
	];

	const baselineData = [
		0.83, 0.834, 0.832, 0.836, 0.838, 0.84, 0.842,
		0.844, 0.846, 0.848, 0.85, 0.852, 0.854, 0.856
	];

	const touchpoints = [
		"5th warmest December on record (baseline: 1951–1980)",
		"Hunga Tonga eruption injects water vapor & aerosols into stratosphere",
		"Short-term aerosol cooling effect from eruption",
		"Continued minor cooling; global temps stabilize",
		"Close to baseline; volcano cooling still in effect",
		"Volcanic aerosol effects linger; delaying heat retention",
		"Minimal anomaly; lowest point post-eruption",
		"Begin warming rebound; El Niño hints emerging",
		"Water vapor starts exerting warming influence",
		"Slight warming; ENSO neutral conditions",
		"Warming continues; solar cycle influence",
		"Pre-El Niño oceanic warming becomes visible",
		"Strongest warming since eruption; amplified by vapor",
		"Start of 2023 warming surge leading to El Niño"
	];

	onMount(() => {
		const canvasEl = document.getElementById("tempChart");
		if (!canvasEl) return;

		const ctxChart = canvasEl.getContext("2d");

		const animatedData = Array(tempData.length).fill(null);

		const chart = new Chart(ctxChart, {
			type: "line",
			data: {
				labels: tempLabels,
				datasets: [
					{
						label: "Temperature anomaly (°C)",
						data: animatedData,
						borderColor: "#E63946",
						borderWidth: 2,
						tension: 0.35,
						pointRadius: 4,
						pointHoverRadius: 6,
						pointBackgroundColor: "#ffffff"
					},
					{
						label: "Baseline (°C)",
						data: baselineData,
						borderColor: "#555555",
						borderWidth: 2,
						borderDash: [6, 6],
						tension: 0,
						pointRadius: 0
					}
				]
			},
			options: {
				responsive: true,
				maintainAspectRatio: false,
				plugins: {
					legend: {
						labels: {
							color: "#333",
							font: { size: 13 }
						}
					},
					tooltip: {
						callbacks: {
							afterBody: (items) => {
								const index = items[0].dataIndex;
								return touchpoints[index] || "";
							}
						}
					}
				},
				scales: {
					x: {
						ticks: { color: "#333" },
						grid: { color: "rgba(0,0,0,0.08)" }
					},
					y: {
						title: {
							display: true,
							text: "Temperature anomaly (°C)"
						},
						ticks: { color: "#333" },
						grid: { color: "rgba(0,0,0,0.08)" }
					}
				}
			}
		});

		let i = 0;
		function animateChart() {
			if (i < tempData.length) {
				animatedData[i] = tempData[i];
				chart.update("none");
				i++;
				requestAnimationFrame(animateChart);
			}
		}
		animateChart();
	});
</script>

<!-- ======================================================
     INTRO SCREEN
     ====================================================== -->
<div
	class="intro-screen"
	style="opacity: {introVisible ? 1 : 0}; pointer-events: {introVisible ? 'auto' : 'none'}"
>
	<div class="intro-text intro-longform">
		<h1 class="intro-headline">
			The volcano that warmed the world
		</h1>

		<p class="intro-dek">
			What happened beneath the South Pacific on January 15, 2022, and how a single
			underwater eruption disrupted the atmosphere across the planet.
		</p>

		<p class="intro-body">
			A massive explosion at the Hunga Tonga–Hunga Haʻapai volcanic arc sent shockwaves
			racing across continents and injected unprecedented amounts of water vapour into
			the stratosphere. Scientists later traced a measurable rise in global
			temperature to this single event, an eruption powerful enough to reshape the sky itself.
		</p>

		<div class="intro-arrow">↓ Scroll to begin ↓</div>
	</div>
</div>

<!-- ======================================================
     PAGE CONTAINER
     ====================================================== -->
<div class="page-container">

	<div class="scrolly-block">

		<div class="canvas-wrapper">
			<canvas
				bind:this={canvas}
				width={1920}
				height={1080}
				class="zoom-canvas"
			></canvas>
		</div>

		<!-- OVERLAY THAT FADES IN DURING ZOOM -->
		{#if showOverlay}
			<div class="zoom-overlay">
				<div class="zoom-overlay-box">
					{#if showOverlayA}
						<h2>Approaching the Tonga Arc</h2>
						<p>
							As we descend toward the South Pacific, the Tonga–Kermadec arc
							comes into view, one of Earth’s most active tectonic boundaries.
						</p>
					{:else if showOverlayB}
						<h2>The South Pacific Ocean</h2>
						<p>
							Hunga Tonga–Hunga Haʻapai lies along a 2,700 km chain of submarine
							volcanoes shaped by the collision of giant plates.
						</p>
						<p>
							Mostly submerged, the volcano remained a remote outcrop, until the
							2022 eruption reshaped it within minutes.
						</p>
					{/if}
				</div>
			</div>
		{/if}

		<div class="zoom-spacer"></div>
	</div>

	<!-- ======================================================
	     CHAPTER INTRO
	     ====================================================== -->
	<section class="chapter-intro">
		<h2 class="chapter-title">How the eruption unfolded</h2>

		<p class="chapter-text">
			A sequence of fast-building events: pressure surges, violent steam flashes,
			and the collapse of the volcanic structure, unfolded within minutes,
			intensifying the eruption into one of the most powerful atmospheric disturbances ever recorded.
		</p>

		<p class="chapter-text">
			Below is a simplified step-by-step reconstruction of how the eruption
			progressed, based on satellite imagery, pressure sensors, and
			ocean-surface observations.
		</p>
	</section>

	<!-- ======================================================
	     SEQUENCE SECTION
	     ====================================================== -->
<section class="sequence-wrapper">
    <div class="sequence-sticky">
        <canvas
            bind:this={seqCanvas}
            width={900}
            height={900}
            class="sequence-canvas"
        ></canvas>

        <div class="sequence-caption">
            Satellite reconstruction of the eruption sequence
        </div>
    </div>
</section>


	<section class="post-sequence-text">
		<p>
			What followed was a rapid escalation – pressure spikes, explosive steam flashes
			as magma hit seawater, and structural collapse within the volcano. Together,
			these forces amplified the eruption into a planet-shaking event.
		</p>
	</section>

	<section class="eruption-video-section">
		<h2 class="eruption-video-title">Satellite view of the eruption</h2>

		<p class="eruption-video-description">
			A geostationary satellite captured the eruption in real time, revealing the
			shockwave, plume expansion, and atmospheric ripples as they spread across
			the South Pacific within minutes.
		</p>

		<video class="eruption-video" controls autoplay muted playsinline>
			<source src="/videos/hunga-eruption.mp4" type="video/mp4" />
			Your browser does not support the video tag.
		</video>
	</section>

	<section class="facts-wrapper">
		<div class="fact-section left">
			<div class="fact-card">
				<h3>The volcano’s summit was just ~150 m below sea level</h3>
				<p>
					Most volcano summits rise hundreds to thousands of meters above sea level.
					Hunga Tonga’s unusually shallow summit allowed seawater to interact directly
					with erupting magma, supercharging the explosion.
				</p>
			</div>
		</div>

		<div class="fact-section right">
			<div class="fact-card">
				<h3>Water vapour from the eruption reached the mesosphere</h3>
				<p>
					Major eruptions typically inject material only into the stratosphere (10–50 km).
					This eruption sent moisture above 50 km, an extremely rare event showing how
					powerful the vertical blast was.
				</p>
			</div>
		</div>

		<div class="fact-section left">
			<div class="fact-card">
				<h3>~10% more water vapour entered the stratosphere</h3>
				<p>
					The stratosphere normally contains very little water vapour.
					A sudden 10% increase from one eruption temporarily altered
					Earth’s radiative balance and atmospheric chemistry.
				</p>
			</div>
		</div>

		<div class="fact-section right">
			<div class="fact-card">
				<h3>Estimated +0.035°C global temperature rise</h3>
				<p>
					That number seems small, but for a single eruption it’s measurable globally,
					and unusual, since major volcanic events typically cool the planet instead of warming it.
				</p>
			</div>
		</div>
	</section>

	<!-- ============================
	     RISE IN MONTHLY GLOBAL TEMPERATURE
	     ============================== -->
	<section id="chart-section" class="chart-section">
		<div class="chart-text-block">
			<h2>Rise in monthly global temperature</h2>
			<p>
				Monthly global temperature anomalies before and after the Hunga Tonga eruption.
				The solid line shows observed temperatures; the dashed line shows the long-term
				baseline used by climate scientists.
			</p>
		</div>

		<div class="chart-container">
			<canvas id="tempChart"></canvas>
		</div>
	</section>

	<!-- ======================================================
	     WHAT WAS DIFFERENT ABOUT HUNGA TONGA
	====================================================== -->
	<section class="difference-section">
		<div class="difference-inner">
			<h2 class="difference-title">What was so different about Hunga Tonga?</h2>

			<p class="difference-text">
				Unlike typical volcanic eruptions that cool the planet, Hunga Tonga blasted an
				unprecedented amount of water vapour straight into the stratosphere, enough to 
				temporarily alter Earth’s radiative balance. This wasn’t just an eruption; it was a 
				climate-altering event outside the patterns scientists normally expect from volcanoes.
			</p>
		</div>
	</section>

	<!-- ======================================================
	     ISOMETRIC — Volcano Steam vs Olympic Pools
	====================================================== -->
	<section class="iso-section">

		<!-- LEFT — Volcano -->
		<div class="iso-left">
			<div class="tooltip-wrapper">
				<img src="/images/hunga_frames/steamw.jpeg" alt="Volcano" class="iso-volcano fade-in" />
				<div class="tooltip">Steam released: 146,000,000,000 L</div>
			</div>
		</div>

		<!-- RIGHT — 58 Pool Icons -->
		<div class="iso-right">
			{#each Array(58) as _, i}
				<div class="tooltip-wrapper" style="animation-delay:{i * 0.03}s">
					<img 
						src="/images/hunga_frames/Pool.png"
						alt="Pool Icon"
						class="iso-pool fade-in"
					/>
					<div class="tooltip">2,500,000 L (per pool)</div>
				</div>
			{/each}
		</div>
	</section>

	<!-- ======================================================
	     LEGEND — Steam vs Pool Capacity
	====================================================== -->
	<section class="legend-section">

		<h2 class="legend-title">Understanding the scale</h2>

		<p class="legend-subtext">
			The eruption injected an extraordinary amount of water vapour into the atmosphere.
			Here’s how the total volume compares to a standard Olympic-sized swimming pool.
		</p>

		<div class="legend-cards">

			<!-- Steam Released -->
			<div class="legend-card">
				<h3>Steam released</h3>
				<p class="legend-value">146,000,000,000 L</p>
				<p class="legend-desc">
					Estimated total water vapour injected into the stratosphere by the January 15 eruption.
				</p>
			</div>

			<!-- One Pool -->
			<div class="legend-card">
				<h3>One Olympic swimming pool</h3>
				<p class="legend-value">2,500,000 L</p>
				<p class="legend-desc">
					Standard volume used for comparing large-scale water measurements.
				</p>
			</div>

		</div>

		<p class="legend-conclusion">
			That means the eruption released the equivalent of <strong>58 Olympic pools</strong> of water vapour, shooting higher into the atmosphere than any eruption ever recorded.
		</p>

	</section>
	<section class="comparison-section">
    <h2 class="comparison-title">Was Hunga Tonga the biggest?</h2>

    <p class="comparison-subtext">
        A look at how the 2022 eruption compares to some of the most powerful volcanic
        and explosive events in recorded history, measured in megatons of TNT.
    </p>

     <div class="comparison-image-wrapper">
        <img src="/images/hunga_frames/COMPARISON.png" 
             alt="Comparison of volcanic explosion energy"
             class="comparison-image" />
    </div>

	<p class="legend-comparison">
			Each ring represents <strong>10 TNT</strong> (a high explosive formed from toluene by substitution of three hydrogen atoms with nitro groups).
		</p>
</section>


	<!-- ============================
	     CONCLUSION SECTION
	============================ -->
	<section class="conclusion-section">
		<h2 class="conclusion-title">A violent eruption, a planet-scale disturbance</h2>

		<p class="conclusion-text">
			The Hunga Tonga eruption wasn’t just another volcanic event. It rewrote the rules of 
			how a single explosion can influence the atmosphere. By blasting an extraordinary 
			amount of water vapour into the stratosphere and beyond, the volcano created effects 
			normally seen only in climate-changing geological events.
		</p>

		<p class="conclusion-text">
			The shockwaves circled the planet. Satellites recorded atmospheric ripples stretching 
			across oceans. And the excess moisture trapped enough heat to register a measurable 
			rise in global temperature, a rare outcome, since major eruptions usually cool the planet instead.
		</p>

		<p class="conclusion-text">
			What unfolded in January 2022 is now regarded as one of the strongest natural atmospheric 
			disturbances in modern history. It was brief, but it left fingerprints across the 
			atmosphere, the climate system, and global scientific records.
		</p>

		<p class="conclusion-text">
			Hunga Tonga didn’t just erupt. <br />
			It <strong>reset</strong> what we thought a volcano could do.
		</p>
	</section>

</div>

<style>
	/* GLOBAL BACKGROUND COLOR */
	body {
		background: #F4F4F5;
	}

	/* Ensure the page container doesn't override body background */
	.page-container {
		background: transparent;
	}

	/* The intro screen stays white intentionally */
	.intro-screen {
		background: white;
	}

	/* INTRO OVERLAY */
	.intro-screen {
		position: fixed;
		inset: 0;
		background: white;
		display: flex;
		align-items: center;
		justify-content: center;
		z-index: 99999;
		transition: opacity 0.8s ease;
	}

	.intro-longform {
		max-width: 720px;
		text-align: left;
		padding: 0 1rem;
	}

	.intro-headline { font-size: 2.4rem; font-weight: 700; }
	.intro-dek { font-size: 1.25rem; margin-bottom: 1.5rem; }
	.intro-body { font-size: 1rem; margin-bottom: 2rem; }

	.intro-arrow {
		font-size: 1.2rem;
		animation: arrowBounce 1.6s infinite;
		font-weight: 600;
	}

	@keyframes arrowBounce {
		0%,20%,50%,80%,100% { transform: translateY(0); }
		40% { transform: translateY(-12px); }
		60% { transform: translateY(-6px); }
	}

	/* PAGE LAYOUT */
	.page-container {
		width: 100%;
		max-width: none; /* let zoom + background be truly full width */
		margin: 0 auto;
	}

	/* SCROLLY BLOCK */
	.scrolly-block {
		position: relative;
		width: 100%;
	}

	.canvas-wrapper {
		position: sticky;
		top: 0;
		display: flex;
		justify-content: center;
		z-index: 1;
	}

	.zoom-canvas {
		width: 100vw;      /* full viewport width */
		height: 100vh;     /* full viewport height */
		display: block;
		object-fit: contain;  /* no cropping, full frame visible */
	}

	.zoom-spacer {
		height: 350vh;
	}

	/* ZOOM OVERLAY CARDS */
	.zoom-overlay {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100vh;
		display: flex;
		justify-content: flex-start;
		align-items: center;
		pointer-events: none;
		z-index: 2; /* above canvas, below intro */
	}

	.zoom-overlay-box {
		margin-left: 10%;
		max-width: 440px;
		background: rgba(255,255,255,0.78);
		backdrop-filter: blur(14px);
		padding: 2rem 2.2rem;
		border-radius: 16px;
		box-shadow: 0 12px 38px rgba(0,0,0,0.18);
		border: 1px solid rgba(255,255,255,0.45);
		pointer-events: auto;
		animation: fadeSwap 0.35s ease;
	}

	@keyframes fadeSwap {
		from { opacity: 0; transform: translateY(12px); }
		to   { opacity: 1; transform: translateY(0); }
	}

	/* CHAPTER INTRO */
	.chapter-intro {
		max-width: 800px;
		margin: 14rem auto 8rem;
		padding: 0 1.5rem;
	}

	.chapter-title { font-size: 2.4rem; font-weight: 700; }

	.chapter-text {
		font-size: 1.15rem;
		line-height: 1.75;
		margin-bottom: 1.2rem;
	}

	/* SEQUENCE SECTION */
	.sequence-wrapper {
    height: 420vh; /* reduced from 600vh */
    margin-top: 8rem; 
    padding-top: 4rem;
    padding-bottom: 6rem; /* gives space below */
}


	.sequence-sticky {
		position: sticky;
		top: 0;
		height: 100vh;
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
	}

	.sequence-canvas {
    width: min(55vw, 650px);
    max-width: 100%;
    object-fit: contain;
}


	.sequence-caption {
		margin-top: 2rem;
		font-size: 1.1rem;
		opacity: 0.9;
		text-align: center;
	}

	.post-sequence-text {
		max-width: 800px;
		margin: 4rem auto 3rem;
		padding: 0 1.5rem;
		font-size: 1.15rem;
		line-height: 1.75;
		color: #333;
	}

	.eruption-video-section {
		max-width: 900px;
		margin: 4% auto 8rem;
		padding: 0 1.5rem;
		text-align: left;
	}

	.eruption-video-title {
		font-size: 2rem;
		font-weight: 700;
		margin-bottom: 1rem;
		color: #111;
	}

	.eruption-video-description {
		font-size: 1.15rem;
		color: #444;
		line-height: 1.75;
		margin-bottom: 2rem;
	}

	.eruption-video {
		width: 100%;
		border-radius: 12px;
		box-shadow: 0 10px 40px rgba(0,0,0,0.2);
	}

	/* FACT CARDS (left-right alternating pinned) */
	.facts-wrapper {
		margin-right: 10%;
		margin-top: 8rem;
		margin-bottom: 10rem;
	}

	.fact-section {
		height: 130vh;
		display: flex;
		align-items: flex-start;
		position: relative;
	}

	.fact-card {
		position: sticky;
		top: 25vh;
		max-width: 420px;
		background: rgba(255,255,255,0.75);
		backdrop-filter: blur(14px);
		padding: 2rem;
		border-radius: 16px;
		box-shadow: 0 12px 35px rgba(0,0,0,0.15);
		border: 1px solid rgba(255,255,255,0.5);
		opacity: 1;
		transform: translateY(0);
		transition: opacity 0.4s ease, transform 0.4s ease;
	}

	.fact-section.left {
		margin-left: 10%;
		justify-content: flex-start;
		padding-left: 2rem;
	}

	.fact-section.right {
		justify-content: flex-end;
		padding-right: 2rem;
	}

	.fact-card h3 {
		font-size: 1.6rem;
		font-weight: 700;
		margin-bottom: 1rem;
		color: #111;
	}

	.fact-card p {
		font-size: 1.05rem;
		line-height: 1.65;
		color: #333;
	}

	.fact-section:last-child {
		margin-bottom: 8rem;
	}

	/* CHART SECTION */
	.chart-section {
		margin-top: 8rem;
		margin-bottom: 10rem;
		padding: 0 1.5rem;
	}

	.chart-text-block {
		max-width: 720px;
		margin: 0 auto 3rem auto;
		text-align: left;
	}

	.chart-text-block h2 {
		font-size: 2rem;
		font-weight: 700;
		color: #111;
		margin-bottom: 1rem;
	}

	.chart-text-block p {
		font-size: 1.1rem;
		line-height: 1.7;
		color: #333;
	}

	.chart-container {
		max-width: 900px;
		height: 450px;
		margin: 0 auto;
		padding: 2rem;
		background: #ffffff;
		border-radius: 16px;
		box-shadow: 0 20px 45px rgba(0,0,0,0.1);
	}

	/* BASIC RESPONSIVE TWEAKS */
	@media (max-width: 768px) {
		.zoom-overlay-box {
			margin-left: 1rem;
			margin-right: 1rem;
			max-width: none;
		}

		.sequence-wrapper {
			height: 400vh;
		}

		.sequence-canvas {
    width: min(55vw, 650px);
}


		.fact-section {
			height: 110vh;
		}

		.fact-section.left,
		.fact-section.right {
			justify-content: center;
			padding-left: 1rem;
			padding-right: 1rem;
		}

		.fact-card {
			max-width: 90vw;
		}

		.chart-container {
			padding: 1.25rem;
			height: 360px;
		}
	}

	/* DIFFERENCE SECTION */
	.difference-section {
		margin: 8rem auto 8rem;
		padding: 0 1.5rem;
	}

	.difference-inner {
		max-width: 800px;
		margin: 0 auto;
	}

	.difference-title {
		font-size: 2.2rem;
		font-weight: 700;
		color: #111;
		margin-bottom: 1.2rem;
	}

	.difference-text {
		font-size: 1.15rem;
		line-height: 1.75;
		color: #333;
	}

	/* ======================================================
	   ISOMETRIC SECTION LAYOUT
	====================================================== */

	.iso-section {
		display: flex;
		justify-content: space-between;
		align-items: flex-start;
		gap: 4rem;
		padding: 6rem 1.5rem;
		max-width: 1400px;
		margin: 0 auto 0.02rem;
	}

	/* LEFT SIDE — volcano */
	.iso-left {
		flex: 1;
		display: flex;
		justify-content: center;
	}

	.iso-volcano {
		width: 100%;
		max-width: 420px;
		object-fit: contain;
	}

	/* RIGHT SIDE — grid of pools */
	.iso-right {
		flex: 1.4;
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(75px, 1fr));
		gap: 14px;
	}

	/* Pool icon */
	.iso-pool {
		width: 100%;
		object-fit: contain;
		opacity: 0.95;
		pointer-events: auto;
	}

	/* ======================================================
	   FADE-IN + STAGGER ANIMATION
	====================================================== */
	.fade-in {
		opacity: 0;
		transform: translateY(14px);
		animation: fadeRise 0.6s ease forwards;
	}

	@keyframes fadeRise {
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	/* Stagger applied via inline style: animation-delay */

	/* ======================================================
	   TOOLTIP STYLING
	====================================================== */

	.tooltip-wrapper {
		position: relative;
		display: inline-block;
	}

	/* Tooltip box */
	.tooltip {
		position: absolute;
		bottom: 110%;
		left: 50%;
		transform: translateX(-50%);
		background: rgba(0,0,0,0.78);
		color: white;
		padding: 6px 10px;
		font-size: 0.85rem;
		border-radius: 8px;
		opacity: 0;
		pointer-events: none;
		white-space: nowrap;
		transition: opacity 0.2s ease;
		z-index: 5;
	}

	/* Show on hover */
	.tooltip-wrapper:hover .tooltip {
		opacity: 1;
	}

	/* Small arrow */
	.tooltip::after {
		content: "";
		position: absolute;
		top: 100%;
		left: 50%;
		transform: translateX(-50%);
		border-width: 6px;
		border-style: solid;
		border-color: rgba(0,0,0,0.78) transparent transparent transparent;
	}

	/* ======================================================
	   RESPONSIVE
	====================================================== */

	@media (max-width: 700px) {
		.iso-section {
			flex-direction: column;
			align-items: center;
			gap: 2rem;
		}

		.iso-right {
			grid-template-columns: repeat(auto-fill, minmax(60px, 1fr));
			gap: 10px;
		}
	}

	/* ======================================================
	   LEGEND SECTION
	====================================================== */

	.legend-section {
		max-width: 900px;
		margin: 0.1rem auto 0.1rem;
		padding: 1.5rem;
		text-align: left;
	}

	.legend-title {
    font-size: 2.2rem;
    font-weight: 700;
    margin-bottom: 1.2rem;
}

	.legend-subtext {
		font-size: 1.1rem;
		line-height: 1.7;
		color: #444;
		margin-bottom: 2.5rem;
	}

	.legend-cards {
		display: flex;
		gap: 2rem;
		margin-bottom: 2.5rem;
		flex-wrap: wrap;
	}

	.legend-card {
		flex: 1;
		min-width: 260px;
		background: rgba(255,255,255,0.75);
		backdrop-filter: blur(10px);
		padding: 1.8rem 2rem;
		border-radius: 14px;
		box-shadow: 0 12px 35px rgba(0,0,0,0.1);
		border: 1px solid rgba(255,255,255,0.5);
	}

	.legend-card h3 {
		font-size: 1.3rem;
		font-weight: 700;
		margin-bottom: 0.4rem;
	}

	.legend-value {
		font-size: 1.8rem;
		font-weight: 700;
		color: #000;
		margin-bottom: 0.8rem;
	}

	.legend-desc {
		font-size: 1rem;
		color: #444;
		line-height: 1.6;
	}

	/* ======================================================
   COMPARISON SECTION
====================================================== */

.comparison-section {
    max-width: 960px;
    margin: 6rem auto 8rem;
    padding: 0 1.5rem;
    text-align: left;
}

.comparison-title {
    font-size: 2.1rem;
    font-weight: 700;
    color: #111;
    margin-bottom: 1rem;
}

.comparison-subtext {
    font-size: 1.15rem;
    line-height: 1.75;
    color: #444;
    margin-bottom: 2.5rem;
    max-width: 760px;
}

.comparison-image-wrapper {
    display: flex;
    justify-content: center;
}

.comparison-image {
    width: 100%;
    max-width: 1000px;
    height: auto;
    border-radius: 8px;
    box-shadow: 0 10px 38px rgba(0,0,0,0.18);
}
	.legend-conclusion {
		margin-top: 2rem;
		font-size: 1.15rem;
		line-height: 1.75;
		color: #222;
	}
/* Force the conclusion section back to centered alignment */
.conclusion-section {
    text-align: center;
}

.conclusion-section h2 {
    text-align: center;
}

.conclusion-section p {
    text-align: left;
	font-size: 1.15rem;
		line-height: 1.75;
    margin-left: auto;
    margin-right: auto;
    max-width: 760px;
}


</style>

