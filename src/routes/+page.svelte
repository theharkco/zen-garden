<script>
	import { onMount } from 'svelte';

	let garden = JSON.parse(localStorage.getItem('zenGarden')) || [];
	let isDragging = false;
	let dragIndex = -1;
	
	const elements = [
		{ type: 'stone', emoji: '🪨', color: '#8b7355' },
		{ type: 'stone', emoji: '🗿', color: '#6b5b45' },
		{ type: 'stone', emoji: '⛰️', color: '#9b8b7b' },
		{ type: 'lantern', emoji: '🏮', color: '#dc2626' },
		{ type: 'tree', emoji: '🎋', color: '#22c55e' },
		{ type: 'bowl', emoji: '🍲', color: '#a8a29e' },
		{ type: 'crystal', emoji: '💎', color: '#8b85d9' },
		{ type: 'bowl', emoji: '🥣', color: '#d4a574' },
	];

	function addElement() {
		const randomElement = elements[Math.floor(Math.random() * elements.length)];
		garden.push({
			...randomElement,
			id: Date.now(),
			x: Math.random() * 80 + 10,
			y: Math.random() * 80 + 10,
			scale: 1
		});
		saveGarden();
	}

	function saveGarden() {
		localStorage.setItem('zenGarden', JSON.stringify(garden));
	}

	function handleMouseDown(index, event) {
		isDragging = true;
		dragIndex = index;
	}

	function handleMouseMove(event) {
		if (!isDragging || dragIndex === -1) return;
		
		const rect = event.currentTarget.getBoundingClientRect();
		const x = ((event.clientX - rect.left) / rect.width) * 100;
		const y = ((event.clientY - rect.top) / rect.height) * 100;
		
		garden[dragIndex].x = Math.max(5, Math.min(95, x));
		garden[dragIndex].y = Math.max(5, Math.min(95, y));
		saveGarden();
	}

	function handleMouseUp() {
		isDragging = false;
		dragIndex = -1;
	}

	function removeElement(id) {
		garden = garden.filter(e => e.id !== id);
		saveGarden();
	}

	function clearGarden() {
		if (confirm('Clear your entire garden?')) {
			garden = [];
			saveGarden();
		}
	}

	function resetLayout() {
		garden = garden.map(item => ({
			...item,
			x: Math.random() * 80 + 10,
			y: Math.random() * 80 + 10
		}));
		saveGarden();
	}
</script>

<svelte:window 
	on:mouseup={handleMouseUp}
	on:mousemove={handleMouseMove}
/>

<div class="min-h-screen flex flex-col" style="background: linear-gradient(135deg, #f5f5f0 0%, #e8e8e3 100%);">
	<!-- Header -->
	<header class="text-center py-8 px-4">
		<h1 class="text-4xl md:text-5xl font-light mb-2 tracking-wider" style="color: #2d2d2a;">Zen Garden</h1>
		<p class="text-gray-600 text-sm tracking-wide">Create your peaceful sanctuary</p>
	</header>

	<!-- Garden Container -->
	<main class="flex-1 px-4 pb-4">
		<div 
			class="relative w-full max-w-4xl mx-auto h-[60vh] md:h-[70vh] rounded-2xl overflow-hidden shadow-2xl border-4"
			style="background: linear-gradient(to bottom, #f0ede5 0%, #e6dfcf 100%); border-color: #d4c5b0;"
			on:click|stopPropagation={(e) => {
				isDragging = false;
				dragIndex = -1;
			}}
		>
			<!-- Raked Sand Pattern -->
			<div class="absolute inset-0 opacity-30 pointer-events-none">
				<div class="w-full h-full" style="background: repeating-linear-gradient(0deg, transparent, transparent 2px, rgba(139, 115, 85, 0.1) 2px, rgba(139, 115, 85, 0.1) 4px);"></div>
			</div>

			<!-- Garden Elements -->
			{#each garden as element (element.id)}
				<div 
					class="absolute cursor-grab active:cursor-grabbing select-none transition-transform hover:scale-110"
					style={`
						left: ${element.x}%;
						top: ${element.y}%;
						transform: translate(-50%, -50%);
						font-size: ${element.scale * 4}rem;
					`}
					on:mousedown={(e) => handleMouseDown(garden.indexOf(element), e)}
					on:dblclick={(e) => {
						e.stopPropagation();
						removeElement(element.id);
					}}
				>
					<span class="drop-shadow-lg">{element.emoji}</span>
				</div>
			{/each}
		</div>
	</main>

	<!-- Controls -->
	<footer class="bg-white/80 backdrop-blur-sm border-t border-gray-200 p-4 md:p-6">
		<div class="max-w-4xl mx-auto space-y-4">
			<!-- Element Selector -->
			<div class="flex flex-wrap gap-2 justify-center">
				{#each elements as element}
					<button
						class="w-12 h-12 md:w-14 md:h-14 rounded-xl flex items-center justify-center text-2xl hover:scale-110 transition-all shadow-md hover:shadow-lg"
						style="background: white; border: 2px solid #e5e5e0;"
						on:click={addElement}
						title="Add {element.type}"
					>
						{element.emoji}
					</button>
				{/each}
			</div>

			<!-- Action Buttons -->
			<div class="flex flex-wrap gap-3 justify-center pt-4 border-t border-gray-200">
				<button
					class="px-6 py-2 rounded-full text-sm font-medium transition-all hover:scale-105 shadow-md"
					style="background: linear-gradient(135deg, #8b7355 0%, #a89f91 100%); color: white;"
					on:click={resetLayout}
				>
					🔄 Randomize
				</button>
				<button
					class="px-6 py-2 rounded-full text-sm font-medium transition-all hover:scale-105 shadow-md"
					style="background: #e8e8e3; color: #555;"
					on:click={clearGarden}
				>
					🗑️ Clear All
				</button>
				<button
					class="px-6 py-2 rounded-full text-sm font-medium transition-all hover:scale-105 shadow-md"
					style="background: #2d2d2a; color: white;"
					on:click={() => alert(`Your garden has ${garden.length} elements`)}
				>
					💾 {garden.length} elements
				</button>
			</div>
		</div>
	</footer>
</div>

<style>
	:global(.cursor-grab) {
		cursor: grab;
	}
	:global(.cursor-grabbing) {
		cursor: grabbing;
	}
</style>