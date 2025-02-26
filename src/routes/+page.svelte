<script lang="ts">
	import type { CartProduct } from '$lib/types';
	import CartItem from './cart-item.svelte';
	import ShoppingCart from 'phosphor-svelte/lib/ShoppingCart';
	import X from 'phosphor-svelte/lib/X';

	let { data } = $props();

	let cartOpen = $state(false);
	let cartProducts = $state<CartProduct[]>([]);
	let cartStats = $derived.by(() => {
		let quantity = 0;
		let total = 0;
		for (const product of cartProducts) {
			quantity += product.quantity;
			total += product.quantity * product.product.price;
		}
		return { quantity, total };
	});

	const qualifiledForFreeShipping = $derived(cartStats.total >= 50);
	$effect(() => {
		if (qualifiledForFreeShipping) {
			alert('You have qualifiled for free Shipping.');
		}
	});
	function removeItem(id: string) {
		cartProducts = cartProducts.filter((product) => product.id !== id);
	}
</script>

<div class="flex items-center bg-gray-300 p-4">
	<span class="text-lg font-bold">SvelteMart</span>
	<div class="relative ml-auto flex items-center">
		<button
			onclick={() => (cartOpen = !cartOpen)}
			class="flex items-center rounded-full bg-sky-600 px-4 py-2 text-white hover:bg-sky-700"
		>
			<ShoppingCart class="mr-2 size-5" />
			<span>Cart ({cartStats.quantity})</span>
		</button>
		{#if cartOpen}
			<div class="absolute top-8 right-0 z-10 mt-2 w-80 rounded-lg bg-white shadow-xl">
				<div class="relative p-4">
					<h2 class="mb-4 text-lg font-semibold">Your Cart</h2>
					<button
						onclick={() => (cartOpen = false)}
						aria-label="close-cart"
						class="absolute top-4 right-4 rounded-full p-1 hover:bg-gray-200"
					>
						<X class="size-4" />
					</button>
					{#each cartProducts as _, i}
						<CartItem bind:cartProduct={cartProducts[i]} {removeItem} />
					{/each}
					<div class="mt-4 border-gray-200 pt-4">
						<p class="text-lg font-semibold">Total: ${cartStats.total.toFixed(2)}</p>
					</div>
				</div>
			</div>
		{/if}
	</div>
</div>

<div class="grid grid-cols-1 gap-6 bg-gray-100 p-8 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
	{#each data.products as product}
		<div class="overflow-hidden rounded-xl bg-white shadow-lg">
			<img src={product.thumbnail} alt={product.title} class="h-48 w-full object-cover" />
			<div class="p-4">
				<p class="mb-2 truncate overflow-hidden text-lg font-medium text-gray-800">
					{product.title}
				</p>
				<div class="flex items-center justify-between">
					<p class="text-xl font-bold">${product.price}</p>
					<button
						onclick={() => {
							cartProducts.push({
								id: crypto.randomUUID(),
								quantity: 1,
								product: product
							});
						}}
						class="rounded-full bg-sky-600 px-4 py-2 text-white transition-colors duration-300 hover:bg-sky-700"
					>
						Add to cart
					</button>
				</div>
			</div>
		</div>
	{/each}
</div>
