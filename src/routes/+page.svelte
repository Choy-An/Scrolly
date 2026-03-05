<script>
	import { onMount } from "svelte";

	let showText = false;
	let section;

	function handleScroll() {
		const rect = section.getBoundingClientRect();
		const viewH = window.innerHeight;

		// progress while section is pinned
		const progress = Math.min(
			Math.max((viewH - rect.top) / viewH, 0),
			1
		);

		showText = progress > 0.4;
	}

	onMount(() => {
		window.addEventListener("scroll", handleScroll, { passive: true });

		return () => {
			window.removeEventListener("scroll", handleScroll);
		};
	});

	let sliderSection;
let sliderValue = 50;

function startDrag(e) {
	e.preventDefault();
	window.addEventListener("mousemove", onDrag);
	window.addEventListener("mouseup", stopDrag);
	window.addEventListener("touchmove", onDrag);
	window.addEventListener("touchend", stopDrag);
}

function onDrag(e) {
	const rect = sliderSection.getBoundingClientRect();
	const clientX = e.touches ? e.touches[0].clientX : e.clientX;
	let percent = ((clientX - rect.left) / rect.width) * 100;
	sliderValue = Math.min(Math.max(percent, 0), 100);
}

function stopDrag() {
	window.removeEventListener("mousemove", onDrag);
	window.removeEventListener("mouseup", stopDrag);
	window.removeEventListener("touchmove", onDrag);
	window.removeEventListener("touchend", stopDrag);
}
/* ===============================
   STATUE DATA SECTION
================================ */

let statueSection;
let statueProgress = 0;

function handleStatueScroll() {
	if (!statueSection) return;

	const rect = statueSection.getBoundingClientRect();
	const viewH = window.innerHeight;
	const totalScroll = rect.height - viewH;

	const progress = (viewH - rect.top) / totalScroll;

	statueProgress = Math.min(Math.max(progress, 0), 1);
}


onMount(() => {
	window.addEventListener("scroll", handleStatueScroll, { passive: true });

	return () => {
		window.removeEventListener("scroll", handleStatueScroll);
	};
});


</script>

<!-- ===============================
     PINNED HERO VIDEO
================================ -->
<section bind:this={section} class="hero-pin">

	<div class="video-sticky">
		<video
			src="/videos/gangotri-dolly.mp4"
			autoplay
			muted
			playsinline
			preload="auto"
		></video>

		<div class="video-gradient"></div>

		<div class="title-overlay {showText ? 'visible' : ''}">
			<h1>When Rivers Lose Their Source</h1>
			<p>Gangotri Glacier and the Ganga headwaters</p>
		</div>
	</div>

</section>

<section class="reflection-section">
	<div class="reflection-inner">
		<p>
			Rivers are drawn as continuous lines, with clear beginnings and clear paths.
			Their sources are treated as fixed points on a map. But in mountain systems fed by glaciers, the origin of a river can shift,
			quietly, and far from where its effects are felt. This is one such place This is one such place, where significance and change quietly overlap.
		</p>
	</div>
</section>

<section class="image-fade-section">
	<div class="image-fade-wrapper">
		<img
			src="/images/gangotri-map.png"
			alt="Gangotri Glacier and Ganga headwaters"
		/>
	</div>
</section>

<section class="reflection-section">
	<div class="reflection-inner">
		<p>
			The Gangotri Glacier is not a single mass of ice, but a system, feeding multiple streams that converge to form the Bhagirathi, one of the main headwaters of the Ganga.
This networked structure is visible at the source itself.
		</p>
	</div>
</section>

<section class="map-bleed-section">
	<img
		src="/images/GIS Map.png"
		alt="Gangotri Glacier system and Ganga headwaters"
		class="map-bleed-image"
	/>
</section>

<section class="retreat-section">
	<div class="retreat-inner">
		<h3 class="retreat-title">The Retreat</h3>
		<p class="retreat-text">
			Over the past decades, the boundary of the Gangotri Glacier has shifted
			steadily upstream. This change is visible from above, where the extent of
			ice can be traced and compared over time.
			
			These images use MODIS Terra Snow Cover data derived from the Normalized Difference Snow Index (NDSI), which isolates snow and ice based on how they reflect light. Lighter areas represent snow, while darker greens and browns indicate exposed land and vegetation. As you move the slider, compare how continuous the snow belt appears and how far it extends downslope. Notice breaks in coverage, shrinking margins, and exposed terrain along the glacier’s edge, these visual shifts indicate changes in snow persistence and the glacier’s retreat over time.
		</p>
	</div>
</section>

<section bind:this={sliderSection} class="comparison-section">

	<div class="comparison-years">
		<span>2005</span>
		<span>2025</span>
	</div>

	<div class="comparison-wrapper">

		<!-- Bottom Image -->
		<img src="/images/2005.png" class="comparison-image" />

		<!-- Top Image (cropped dynamically) -->
		<img
			src="/images/2025.png"
			class="comparison-image top-image"
			style="clip-path: inset(0 {100 - sliderValue}% 0 0);"
		/>

		<!-- Divider -->
		<div
			class="slider-line"
			style="left: {sliderValue}%"
			on:mousedown={startDrag}
			on:touchstart={startDrag}
		>
			<div class="slider-handle"></div>
		</div>

	</div>
	<div class="comparison-subtext">
	<p>
		Between 2005 and 2025, the visible extent of the Gangotri Glacier has
		receded along its terminus. Over the past five years alone, estimated
		mass loss corresponds to roughly <strong>0.72 cubic kilometres</strong>
		of ice, or nearly <strong>720 million cubic metres</strong>.
	</p>
</div>

</section>

<!-- ===============================
     STATUE VOLUME DATA SECTION
================================ -->

<section bind:this={statueSection} class="statue-section">

	<div class="statue-sticky">

		<!-- PHASE 1 -->
		<div
			class="statue-phase"
	style="opacity:
		{statueProgress < 0.4
			? 1
			: statueProgress < 0.5
				? 1 - (statueProgress - 0.4) * 10
				: 0}"
		>
			<h2>210,000 m³</h2>
			<p>Approximate concrete core volume of the Statue of Unity.</p>
		</div>

		<!-- PHASE 2 -->
		<div
			class="statue-phase"
	style="opacity:
		{statueProgress >= 0.4 && statueProgress < 0.7
			? 1
			: statueProgress >= 0.7 && statueProgress < 0.8
				? 1 - (statueProgress - 0.7) * 10
				: 0}"
		>
			<p>
				Between 2020 and 2025, the Gangotri Glacier is estimated to have
				lost approximately <strong>0.72 cubic kilometres</strong> of ice —
				nearly <strong>720 million cubic metres</strong>.
			</p>
		</div>

		<!-- PHASE 3 -->
		<div
			class="statue-phase"
	style="opacity:
		{statueProgress >= 0.7
			? Math.min((statueProgress - 0.7) * 5, 1)
			: 0}"
		>
			<h2>≈ 3,400</h2>
			<p>
				Statue-of-Unity core equivalents of ice lost in five years.
			</p>
			<small>
				Based on ~0.72 km³ glacier loss and ~210,000 m³ per statue core.
			</small>
		</div>

	</div>

</section>

<section class="conclusion-section">
	<div class="conclusion-inner">
		<h2>What shifts at the source</h2>

		<p>
			Glaciers are often imagined as permanent fixtures of the landscape,
			frozen origins from which rivers reliably emerge. Yet their boundaries
			are neither fixed nor guaranteed.
		</p>

		<p>
			The retreat of the Gangotri Glacier is not a singular event, but a
			continuing process. Measured in metres per year and cubic kilometres
			of ice, the change is gradual, cumulative, and largely invisible from
			the plains it ultimately feeds.
		</p>

		<p>
			When a river’s source shifts, the effects ripple far beyond the
			mountain valley. What appears distant and glacial is, in reality,
			part of an interconnected hydrological system, one that links
			high-altitude ice to cities, agriculture, and millions of lives
			downstream.
		</p>

		<p>
			The question is not whether the glacier has moved.
			It is how we understand and respond to the movement.
		</p>
	</div>
</section>


<style>
	:root {
	--title-color: #f7f5f2;
	--subtitle-color: #e6e4df;
}
/* ===============================
   BASE
================================ */
body {
	margin: 0;
	background: #000;
	color: #fff;
}

/* ===============================
   HERO PIN SECTION
================================ */
.hero-pin {
	height: 200vh; /* scroll space */
	position: relative;
}

.video-sticky {
	position: sticky;
	top: 0;
	height: 100vh;
	width: 100%;
	overflow: hidden;
}

video {
	width: 100%;
	height: 100%;
	object-fit: cover;
	display: block;
}

/* ===============================
   GRADIENT
================================ */
.video-gradient {
	position: absolute;
	inset: 0;
	background: linear-gradient(
		to bottom,
		rgba(0,0,0,0.25),
		rgba(0,0,0,0.15),
		rgba(0,0,0,0.45)
	);
	pointer-events: none;
}

/* ===============================
   TITLE OVERLAY
================================ */
.title-overlay {
	position: absolute;
	inset: 0;
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;
	text-align: center;
	padding: 1.5rem;

	opacity: 0;
	transform: translateY(18px);
	transition: opacity 0.8s ease, transform 0.8s ease;
	pointer-events: none;
}

.title-overlay.visible {
	opacity: 1;
	transform: translateY(0);
}

.title-overlay h1 {
	font-size: clamp(2.8rem, 5vw, 4.5rem);
	font-weight: 700;
	margin-bottom: 0.9rem;
	color: var(--title-color);

	text-shadow:
		0 2px 6px rgba(0,0,0,0.45),
		0 10px 28px rgba(0,0,0,0.4);
}

.title-overlay p {
	font-size: clamp(1.2rem, 2vw, 1.6rem);
	opacity: 0.92;
	color: var(--subtitle-color);

	text-shadow:
		0 2px 6px rgba(0,0,0,0.45);
}

/* ===============================
   NEXT SECTION
================================ */
.next-section {
	min-height: 100vh;
	background: #f4f4f5;
	color: #111;
	display: flex;
	align-items: center;
	justify-content: center;
	font-size: 2rem;
}

/* ===============================
   REFLECTION SECTION
================================ */
.reflection-section {
	min-height: 100vh;
	background: #f4f4f5;
	display: flex;
	align-items: center;
}

.reflection-inner {
	max-width: 720px;
	margin: 0 auto;
	padding: 0 1.5rem;
}

.reflection-inner p {
	font-size: 1.25rem;
	line-height: 1.85;
	color: #222;
	margin-bottom: 0.6rem;
}

/* Slight typographic calm */
.reflection-inner p:last-child {
	margin-bottom: 0;
}

/* Mobile */
@media (max-width: 768px) {
	.reflection-inner p {
		font-size: 1.1rem;
		line-height: 1.75;
	}
}

/* ===============================
   IMAGE FADE SECTION
================================ */
.image-fade-section {
	background: #f4f4f5; /* same as previous section */
	padding: 0.1rem 1.5rem 3rem;
}

.image-fade-wrapper {
	max-width: 1000px;
	margin: 0 auto;
	position: relative;
}

/* The actual image */
.image-fade-wrapper img {
	width: 100%;
	display: block;

	/* SOFT EDGE FADE */
	-webkit-mask-image: radial-gradient(
		ellipse at center,
		rgba(0,0,0,1) 65%,
		rgba(0,0,0,0.9) 72%,
		rgba(0,0,0,0.6) 80%,
		rgba(0,0,0,0.3) 88%,
		rgba(0,0,0,0) 100%
	);
	mask-image: radial-gradient(
		ellipse at center,
		rgba(0,0,0,1) 65%,
		rgba(0,0,0,0.9) 72%,
		rgba(0,0,0,0.6) 80%,
		rgba(0,0,0,0.3) 88%,
		rgba(0,0,0,0) 100%
	);
}

/* ===============================
   FULL-WIDTH MAP BLEED
================================ */
.map-bleed-section {
	width: 100vw;
	position: relative;
	left: 50%;
	right: 50%;
	margin-left: -50vw;
	margin-right: -50vw;

	background: #f4f4f5;
	padding: 4rem 0 6rem;
}

.map-bleed-image {
	width: 100%;
	height: auto;
	display: block;

	/* TRUE ALL-SIDE EDGE BLEED */
	-webkit-mask-image:
		linear-gradient(to top,
			rgba(0,0,0,0) 0%,
			rgba(0,0,0,1) 12%
		),
		linear-gradient(to bottom,
			rgba(0,0,0,0) 0%,
			rgba(0,0,0,1) 12%
		),
		linear-gradient(to left,
			rgba(0,0,0,0) 0%,
			rgba(0,0,0,1) 12%
		),
		linear-gradient(to right,
			rgba(0,0,0,0) 0%,
			rgba(0,0,0,1) 12%
		);

	-webkit-mask-composite: intersect;
	mask-composite: intersect;
}
/* ===============================
   THE RETREAT SECTION
================================ */
.retreat-section {
	min-height: 100vh;
	background: #f4f4f5;
	display: flex;
	align-items: center;
	justify-content: center;
	padding: 0 1.5rem;
}

.retreat-inner {
	max-width: 760px;
	text-align: center;
}

/* Subtitle */
.retreat-title {
	font-size: 2.1rem;
	font-weight: 600;
	margin-bottom: 1.6rem;
	display: inline-block;
	position: relative;

	color: #111;

	/* subtle depth */
	text-shadow:
		0 1px 2px rgba(0,0,0,0.08),
		0 4px 12px rgba(0,0,0,0.06);
}

/* Light blue underline */
.retreat-title::after {
	content: "";
	position: absolute;
	left: 50%;
	transform: translateX(-50%);
	bottom: -0.5rem;

	width: 60%;
	height: 3px;
	border-radius: 2px;

	background: #9ecae1; /* light, cartographic blue */
}

/* Body text */
.retreat-text {
	font-size: 1.2rem;
	line-height: 1.8;
	color: #222;
	max-width: 680px;
	margin: 0 auto;
}

/* Mobile */
@media (max-width: 768px) {
	.retreat-section {
	min-height: 70vh;
}


	.retreat-text {
		font-size: 1.1rem;
		line-height: 1.7;
	}
}

/* ===============================
   BEFORE–AFTER COMPARISON
================================ */

.comparison-section {
	background: #f4f4f5;
	padding: 1.5rem 1.5rem 4rem;
	display: flex;
	flex-direction: column;
	align-items: center;
	margin-top: -rem;   /* pulls it upward slightly */
}


.comparison-years {
	width: 100%;
	max-width: 1100px;
	display: flex;
	justify-content: space-between;
	margin-bottom: 1rem;
	font-weight: 600;
	color: #333;
}

.comparison-wrapper {
	position: relative;
	width: 100%;
	max-width: 1100px;
	aspect-ratio: 16 / 9;
	overflow: hidden;
}

.comparison-image {
	position: absolute;
	inset: 0;
	width: 100%;
	height: 100%;
	object-fit: cover;
}

.comparison-overlay {
	position: absolute;
	top: 0;
	left: 0;
	height: 100%;
	overflow: hidden;
}

.slider-line {
	position: absolute;
	top: 0;
	height: 100%;
	width: 2px;
	background: white;
	transform: translateX(-1px);
	cursor: ew-resize;
	display: flex;
	align-items: center;
	justify-content: center;
	z-index: 10;
}

.slider-handle {
	width: 22px;
	height: 22px;
	border-radius: 50%;
	background: white;
	border: 2px solid #333;
}
.comparison-subtext {
	max-width: 800px;
	margin: 2.5rem auto 0;
	text-align: center;
}

.comparison-subtext p {
	font-size: 1.1rem;
	line-height: 1.8;
	color: #222;
}
/* ===============================
   STATUE DATA SECTION
================================ */

.statue-section {
	height: 450vh;
	background: #f4f4f5;
	position: relative;
}

.statue-sticky {
	position: sticky;
	top: 0;
	height: 100vh;
	display: flex;
	align-items: center;
	justify-content: center;
	text-align: center;
	padding: 0 1.5rem;
}

.statue-phase {
	position: absolute;
	max-width: 800px;
	transition: opacity 0.5s ease;
}

.statue-phase h2 {
	font-size: clamp(3rem, 8vw, 5rem);
	font-weight: 700;
	color: #111;
	margin-bottom: 1rem;
}

.statue-phase p {
	font-size: 1.2rem;
	line-height: 1.8;
	color: #222;
}

.statue-phase small {
	display: block;
	margin-top: 1rem;
	font-size: 0.85rem;
	color: #666;
}

/* ===============================
   CONCLUSION SECTION
================================ */

.conclusion-section {
	min-height: 100vh;
	background: #f4f4f5;
	display: flex;
	align-items: center;
	justify-content: center;
	padding: 6rem 1.5rem;
}

.conclusion-inner {
	max-width: 760px;
	text-align: center;
}

.conclusion-inner h2 {
	font-size: 2.2rem;
	font-weight: 600;
	margin-bottom: 2rem;
	color: #111;
}

.conclusion-inner p {
	font-size: 1.15rem;
	line-height: 1.9;
	color: #222;
	margin-bottom: 1.4rem;
}

/* ===============================
   MOBILE
================================ */
@media (max-width: 768px) {
	.title-overlay h1 {
		font-size: 2.2rem;
	}
}
</style>
