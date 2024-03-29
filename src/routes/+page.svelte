<script lang="ts">
	import SearchInput from '$lib/components/SearchInput.svelte';
	import HeaderInfo from '$lib/components/bip39/HeaderInfo.svelte';
	import LanguagePicker from '$lib/components/bip39/LanguagePicker.svelte';
	import Word from '$lib/components/bip39/Word.svelte';
	import type { Lang } from '$lib/models/bip39';
	import { isEnglish, normalize } from '$lib/utils';
	import Icon from '@iconify/svelte';
	import { words as wordSets } from './words';

	let lang: Lang = 'en';
	$: wordlist = wordSets[lang];

	let search = '';
	$: _search = search.toLowerCase().trim();
	const clearSearch = () => (search = '');

	const wordOrdinal = (word: string) => {
		return wordlist.indexOf(word) + 1;
	};

	const paddedOrdinal = (word: string) => {
		return wordOrdinal(word).toString().padStart(4, '0');
	};

	$: filteredWordlist = wordlist.filter((word: string) => {
		if (!isEnglish(_search)) {
			const nativeWordMatch = word.startsWith(_search);
			return nativeWordMatch;
		}

		const wordMatch = normalize(word).startsWith(normalize(_search));
		if (wordMatch) return true;

		const normalizedWordMatch = normalize(word).startsWith(normalize(_search));
		if (normalizedWordMatch) return true;

		const parsedNumber = parseInt(_search);
		console.log(parsedNumber);
		if (isNaN(parsedNumber)) return false;

		const parsedNumberString = parsedNumber.toString();

		const paddedOrdinalMatch = paddedOrdinal(word).startsWith(parsedNumberString);
		const ordinalMatch = wordOrdinal(word).toString().startsWith(parsedNumberString);

		return paddedOrdinalMatch || ordinalMatch;
	});

	const isFirstExampleOfLetter = (word: string, idx: number) => {
		if (['cn', 'cnt'].includes(lang)) return false;

		if (idx === 0) return true;

		const firstLetter = (word?: string) => word?.at(0);
		return firstLetter(word) !== firstLetter(filteredWordlist?.at(idx - 1));
	};
</script>

<div>
	<header class="mb-8">
		<LanguagePicker bind:lang {wordSets} />
		<div class="mt-6 flex flex-col items-start justify-between gap-8 sm:flex-row">
			<HeaderInfo />

			<SearchInput bind:value={search} />
		</div>
	</header>

	<section>
		{#if filteredWordlist?.length}
			<div class="cursor-crosshair columns-1 gap-8 sm:mt-16 sm:columns-3 md:columns-4">
				{#each filteredWordlist as word, i}
					<Word {word} ordinal={wordOrdinal(word)} highlighted={isFirstExampleOfLetter(word, i)} />
				{/each}
			</div>
		{:else}
			<div class="mt-20 flex flex-col items-center justify-center gap-5 text-neutral-50/20">
				<p class="px-2">no words found</p>
				<button
					on:click={clearSearch}
					class="flex items-center gap-1 rounded-sm bg-neutral-50/5 px-2 py-1 text-sm ring-neutral-600 hover:text-neutral-50/40 focus:outline-none focus:ring-2"
				>
					clear
					<Icon icon="ic:sharp-clear" class="text-base" />
				</button>
			</div>
		{/if}
	</section>

	<div class="mt-12 px-1">
		<a
			href="https://github.com/bitcoin/bips/blob/master/bip-0039/bip-0039-wordlists.md"
			target="_blank"
			rel="noreferrer"
			class="flex items-center gap-2 text-neutral-50/20 hover:underline"
		>
			source
			<Icon icon="ic:sharp-arrow-outward" />
		</a>
	</div>
</div>
