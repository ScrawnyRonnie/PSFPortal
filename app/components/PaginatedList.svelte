<script>
	import { onMount } from 'svelte'
	import axios from 'axios'
	import Pagination from '../components/Pagination'

	export let setURLParam = false;
        export let URLSearchName = 'page';
	export let fetch;

	let data;
	let fetching = false;
	let pagination = { page: 1 };

	export async function refresh(newpage) {
		if (newpage !== undefined && typeof newpage == "number")
			await list_fetch(newpage)
		else
			await list_fetch(pagination.page)
	}

	onMount(async () => {
		const url = new URL(window.location.href)
		let initialPage = 1;

		if (setURLParam) {
			let param = parseInt(url.searchParams.get(URLSearchName))

			if (!isNaN(param))
				initialPage = param;
		}

		await list_fetch(initialPage);
	})

	async function pageChange(page) {
		if (pagination.page == page || fetching)
			return

		if (setURLParam) {
                   const url = new URL(window.location.href);
                   url.searchParams.set(URLSearchName, page);
                   history.replaceState(null, null,
                    url.pathname + url.search + url.hash)
                }

		await list_fetch(page);
	}

	async function list_fetch(page) {
		fetching = true;

		try {
			if (fetch != undefined) {
				const results = await fetch(page)

				if (results != undefined) {
					data = results[0];
					pagination = results[1]
				}
			}
		} finally {
			fetching = false;
		}
	}
</script>

{#if data}
<slot name="header" data={data} pagination={pagination}></slot>
{#if pagination.item_count > 0}
	{#if pagination.page_count > 1}
        <Pagination {pagination} {pageChange} {setURLParam} {URLSearchName} />
	{/if}

	<slot name="body" data={data} pagination={pagination}></slot>

	{#if pagination.page_count > 1}
        <Pagination {pagination} {pageChange} {setURLParam} {URLSearchName} />
	{/if}
{/if}
<slot name="footer" data={data} pagination={pagination}></slot>
{/if}
