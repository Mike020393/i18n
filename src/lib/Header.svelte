<script lang="ts">
	import { base } from '$app/paths'
	import { browser } from '$app/environment'
	import { invalidateAll } from '$app/navigation'
	import { page } from '$app/stores'
	import { setLocale, locale } from '$i18n/i18n-svelte'
	import type { Locales } from '$i18n/i18n-types'
	import { locales,baseLocale } from '$i18n/i18n-util'
	import { loadLocaleAsync } from '$i18n/i18n-util.async'
	import { replaceLocaleInUrl } from '../utils.js'
    import LL from '$i18n/i18n-svelte'

	const switchLocale = async (newLocale: Locales, updateHistoryState = true) => {
		if (!newLocale || $locale === newLocale) return

		// load new dictionary from server
		await loadLocaleAsync(newLocale)

		// select locale
		setLocale(newLocale)

		if (updateHistoryState) {
			// update url to reflect locale changes
			history.pushState({ locale: newLocale }, '', replaceLocaleInUrl($page.url, newLocale))
		}

		// run the `load` function again
		invalidateAll()
	}

	// update `lang` attribute
	$: browser && document.querySelector('html')!.setAttribute('lang', $locale)

	// update locale when navigating via browser back/forward buttons
	const handlePopStateEvent = async ({ state }: PopStateEvent) => switchLocale(state.locale, false)

	// update locale when page store changes
	$: if (browser) {
		const lang = $page.params.lang as Locales
		switchLocale(lang, false)
		history.replaceState({ ...history.state, locale: lang }, '', replaceLocaleInUrl($page.url, lang))
	}
</script>

<header>
	<a href="{base}/{$locale}">
		<h1>typesafe-i18n</h1>
	</a>
    <ul>
        {#each locales as l}
            <li>
                <a class:active={l === $locale} href={replaceLocaleInUrl($page.url, l)}>
                    {l}
                </a>
            </li>
        {/each}
    </ul>
	<!-- <LocaleSwitcher /> -->
</header>
{#each locales as l}
	<link rel="alternate" hreflang={l} href={replaceLocaleInUrl($page.url, l, true)} />
{/each}
<link rel="alternate" hreflang="x-default" href={replaceLocaleInUrl($page.url, baseLocale, true)} />
<h2>
	{$LL.HI({ name: "test" })}
</h2>
