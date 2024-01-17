<script>
	import { onMount } from 'svelte';
	import LevelSelector from '$lib/components/level-selector.svelte';
	import HealthBar from '$lib/components/healthbar.svelte';
	import { statAtLevel, formatVal } from '$lib/utils';

	async function loadData() {
		const data = await fetch('/data');
		const json = await data.json();

		return json.champions;
	}

	/** @type {Object} */
	let data = {};

	onMount(async () => {
		data = await loadData();
	});

	let level = 1;
	let sort = 'name';
	let desc = 1;
	const columns = [
		['name', 'Champion'],
		['healthbar', 'Health Bar'],
		['health', 'Health'],
		['healthRegen', 'Health / 5'],
		['armor', 'Armor'],
		['magicResistance', 'MR'],
		['attackDamage', 'AD'],
		['attackSpeed', 'AS'],
		['mana', 'Mana'],
		['manaRegen', 'Mana / 5']
	];

	function setLevel(lvl) {
		level = lvl;
	}

	$: champions = Object.values(data)
		.map((champion) => {
			const {
				id,
				icon,
				name,
				stats: {
					health,
					healthRegen,
					armor,
					magicResistance,
					attackDamage,
					attackSpeed,
					mana,
					manaRegen
				}
			} = champion;

			return {
				id,
				icon,
				name,
				health: statAtLevel(health.flat, health.perLevel, level),
				healthRegen: statAtLevel(healthRegen.flat, healthRegen.perLevel, level),
				armor: statAtLevel(armor.flat, armor.perLevel, level),
				magicResistance: statAtLevel(magicResistance.flat, magicResistance.perLevel, level),
				attackDamage: statAtLevel(attackDamage.flat, attackDamage.perLevel, level),
				attackSpeed: statAtLevel(attackSpeed.flat, attackSpeed.perLevel, level),
				mana: statAtLevel(mana.flat, mana.perLevel, level),
				manaRegen: statAtLevel(manaRegen.flat, manaRegen.perLevel, level)
			};
		})
		.sort((a, b) => (sort === 'name' ? a.name.localeCompare(b[sort]) : a[sort] - b[sort]) * desc);

	function toggleSort(col) {
		if (sort === col) {
			desc = -desc;
			return;
		}
		sort = col;
		desc = 1;
	}

	$: console.log(data);
</script>

<LevelSelector {level} {setLevel} />

<table>
	<tr>
		{#each columns as column}
			<th on:click={() => toggleSort(column[0])} scope="col">{column[1]}</th>
		{/each}
	</tr>

	{#each champions as champion}
		{@const {
			icon,
			name,
			health,
			healthRegen,
			armor,
			magicResistance,
			attackDamage,
			attackSpeed,
			mana,
			manaRegen
		} = champion}
		<tr>
			<td class="name">
				<div class="info">
					<img src={icon} alt={name} loading="lazy" width="40" height="40" />
					<span>{name}</span>
				</div>
			</td>
			<td><HealthBar {health} /></td>
			<td>{formatVal(health)}</td>
			<td>{formatVal(healthRegen, 1)}</td>
			<td>{formatVal(armor)}</td>
			<td>{formatVal(magicResistance)}</td>
			<td>{formatVal(attackDamage)}</td>
			<td>{formatVal(attackSpeed, 2)}</td>
			<td>{formatVal(mana)}</td>
			<td>{formatVal(manaRegen)}</td>
		</tr>
	{/each}
</table>

<style lang="scss">
	td {
		padding: 0.25em;
		min-width: 10ch;
		text-align: center;
		border: 1px solid hsla(var(--c1-hsl) / 0.15);

		&.name {
			text-align: left;
		}
	}

	.info {
		display: flex;
		align-items: center;
		gap: 0.5rem;
	}
</style>
