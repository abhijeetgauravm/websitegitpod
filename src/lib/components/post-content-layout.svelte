<script lang="ts">
	import Avatars from '$lib/components/ui-library/avatars';
	import OpenGraph from './open-graph.svelte';
	import { authors, authorSocialMediaLinks } from '$lib/contents/authors';
	import '$lib/assets/markdown-commons.scss';
	import Share from '$lib/components/share.svelte';
	import Pill from './pill.svelte';
	import type { ShareLink } from '$lib/types/share-link';
	import RequestChanges from './ui-library/request-changes.svelte';
	import SubscribeRssFeed from './ui-library/subscribe-rss-feed.svelte';
	import '$lib/assets/prism-solarized-light.css';

	export let baseUrl: string;
	export let imagesDirectoryName: string;

	export let date: string = '';
	export let author: string = '';
	export let slug: string = '';
	export let title: string;
	export let image: string;
	export let teaserImage: string;
	export let excerpt: string;
	export let tags: string[] = [];

	let dateDisplay = new Date(Date.parse(date)).toLocaleDateString(undefined, {
		year: 'numeric',
		month: 'short',
		day: 'numeric',
	});

	const authorDisplayNames = Object.entries(authors).reduce(
		(displayNames, [username, profile]) => {
			displayNames[username] = profile.name;
			return displayNames;
		},
		{},
	);

	const hasATwitterProfile = (author: string) =>
		!!authors[author]?.socialProfiles.twitter;

	const writers = author.split(', ');

	const renderTwitterHandles = () => {
		let result = writers.reduce(
			(acc: string, current: string) =>
				acc +
				(hasATwitterProfile(current)
					? `@${current}`
					: authors[current]?.name) +
				', ',
			'',
		);
		result = result.substring(0, result.length - 2);
		return result;
	};

	const shareLinks: ShareLink[] = [
		{
			href: `https://twitter.com/intent/tweet?text=${encodeURIComponent(
				`${title} by ${renderTwitterHandles()} ${baseUrl}${slug}`,
			)}`,
			icon: {
				src: '/svg/brands/twitter.svg',
				alt: 'Twitter',
			},
			trackingName: 'twitter',
		},
		{
			href: `https://www.reddit.com/submit?url=${encodeURIComponent(
				`${baseUrl}${slug}&title=${title}`,
			)}`,
			icon: {
				src: '/svg/brands/reddit.svg',
				alt: 'Reddit',
			},
			trackingName: 'reddit',
		},
		{
			name: 'HackerNews',
			href: `https://news.ycombinator.com/submitlink?u=${encodeURIComponent(
				`${baseUrl}${slug}`,
			)}&t=${encodeURIComponent(title)}`,
			icon: {
				src: '/svg/brands/hackernews.svg',
				alt: 'HackerNews',
			},
			trackingName: 'hackernews',
		},
	];
</script>

<OpenGraph
	data={{
		description: excerpt,
		title,
		type: 'article',
		image: `images/${imagesDirectoryName}/${slug}/${image}`,
		imageTwitter: `images/${imagesDirectoryName}/${slug}/${image}`,
	}}
/>

<article>
	<div class="flex justify-center mt-small mb-8">
		<div class="w-full lg:w-[50rem] leading-[177.7%]">
			<img
				src="/images/{imagesDirectoryName}/{slug}/{teaserImage ||
					image}"
				alt={`${title}`}
				class="max-h-[540px] rounded-tl-2xl rounded-tr-[1.3rem]"
			/>
			<div
				class="content-blog prose prose-img:rounded-tl-2xl prose-img:rounded-tr-[1.3rem] max-w-none mt-xx-small"
			>
				<p
					class="{tags && tags.length > 0
						? '!mb-macro'
						: '!mb-macro'} mt-[1.875rem] text-body"
				>
					{dateDisplay}
				</p>
				<!-- {#if type === "digest"}
        <Pill text="DevX Digest" class="mb-micro" />
      {/if} -->
				{#if tags && tags.length > 0}
					<div class="flex mb-macro items-center gap-macro">
						{#each tags as tag}
							<a
								data-sveltekit-preload-data="hover"
								href="/blog?{new URLSearchParams({
									tag,
								}).toString()}"
								><Pill variant="gray" text={tag} /></a
							>
						{/each}
					</div>
				{/if}
				<header>
					<h1>{title}</h1>
					<p>
						<span
							><Avatars
								usernames={author}
								displayNames={authorDisplayNames}
								socialMediaLinks={authorSocialMediaLinks}
								socialMediaLinkClasses="inline-flex mr-2 px-2 bg-white dark:bg-card rounded-xl text-body focus:bg-card focus:text-important hover:bg-card hover:text-important"
								socialMediaImgClasses="mr-2 h-6 w-6 place-self-center"
							/></span
						>
					</p>
				</header>
				<slot />
			</div>
			<main>
				<div
					class="flex items-center md:flex-row justify-between md:items-baseline border-t border-solid border-divider pt-4 mt-small"
				>
					<Share text="Share this post" {shareLinks} />
					<div class="flex flex-row md:flex-row justify-between">
						{#if imagesDirectoryName === 'blog'}
							<SubscribeRssFeed href="/blog/rss.xml" />
							<RequestChanges
								href={`https://github.com/gitpod-io/website/edit/main/src/content/blog/${slug}.md`}
							/>
						{/if}
					</div>
					{#if imagesDirectoryName === 'guides'}
						<RequestChanges
							href={`https://github.com/gitpod-io/website/edit/main/src/content/guides/${slug}.md`}
						/>
					{/if}
				</div>
			</main>
		</div>
	</div>
</article>

<style lang="postcss">
	.prose :global(nav.toc) {
		@apply hidden;
	}
</style>
