<script>
    import { Inertia } from '@inertiajs/inertia'
    import { InertiaLink } from '@inertiajs/inertia-svelte'
    import { route } from '@/utils'
    import pickBy from 'lodash/pickBy'
    import throttle from 'lodash/throttle'
    import Icon from '@/Shared/Icon.svelte'
    import Layout from '@/Shared/Layout.svelte'
    import Pagination from '@/Shared/Pagination.svelte'
    import SearchFilter from '@/Shared/SearchFilter.svelte'

    export let filters = {}
    export let contacts = []

    let form = {
        search: filters.search,
        trashed: filters.trashed,
    }

    $: search(form)

    function reset(filters) {
        Object.keys(form).forEach(key => (form[key] = null))
    }

    const search = throttle(form => {
        let query = pickBy(form)

        // TODO: fix pagination bug
        Inertia.replace(
            route('contacts', Object.keys(query).length ? query : {
                remember: 'forget'
            })
        )
    }, 150)

</script>

<Layout title="Contacts">
    <h1 class="mb-8 font-bold text-3xl">Contacts</h1>
    <div class="mb-6 flex justify-between items-center">
        <SearchFilter bind:value={form.search} class="w-full max-w-md mr-4" on:reset={reset} let:close={closeFilter}>
            <label for="trashed" class="block text-gray-700">Trashed:</label>
            <select id="trashed" bind:value={form.trashed} class="mt-1 w-full form-select" on:blur={closeFilter}>
                <option value={null} />
                <option value="with">With Trashed</option>
                <option value="only">Only Trashed</option>
            </select>
        </SearchFilter>
        <InertiaLink class="btn-indigo" href={route('contacts.create')}>
            <span>Create</span>
            <span class="hidden md:inline">Contact</span>
        </InertiaLink>
    </div>
    <div class="bg-white rounded shadow overflow-x-auto">
        <table class="w-full whitespace-no-wrap">
            <tr class="text-left font-bold">
                <th class="px-6 pt-6 pb-4">Name</th>
                <th class="px-6 pt-6 pb-4">Organization</th>
                <th class="px-6 pt-6 pb-4">City</th>
                <th class="px-6 pt-6 pb-4" colspan="2">Phone</th>
            </tr>
            {#each contacts.data as contact (contact.id)}
                <tr class="hover:bg-gray-100 focus-within:bg-gray-100">
                    <td class="border-t">
                        <InertiaLink class="px-6 py-4 flex items-center focus:text-indigo-500" href={route('contacts.edit', contact.id)}>
                            {contact.name}
                            {#if contact.deleted_at}
                                <Icon name="trash" class="flex-shrink-0 w-3 h-3 fill-gray-400 ml-2" />
                            {/if}
                        </InertiaLink>
                    </td>
                    <td class="border-t">
                        <InertiaLink class="px-6 py-4 flex items-center" href={route('contacts.edit', contact.id)} tabindex="-1">
                            {#if contact.organization}
                                {contact.organization.name}
                            {/if}
                        </InertiaLink>
                    </td>
                    <td class="border-t">
                        <InertiaLink class="px-6 py-4 flex items-center" href={route('contacts.edit', contact.id)} tabindex="-1">
                            {contact.city}
                        </InertiaLink>
                    </td>
                    <td class="border-t">
                        <InertiaLink class="px-6 py-4 flex items-center" href={route('contacts.edit', contact.id)} tabindex="-1">
                            {contact.phone}
                        </InertiaLink>
                    </td>
                    <td class="border-t w-px">
                        <InertiaLink class="px-4 flex items-center" href={route('contacts.edit', contact.id)} tabindex="-1">
                            <Icon name="cheveron-right" class="block w-6 h-6 fill-gray-400" />
                        </InertiaLink>
                    </td>
                </tr>
            {/each}

            {#if contacts.data.length === 0}
                <tr>
                    <td class="border-t px-6 py-4" colspan="4">No contacts found.</td>
                </tr>
            {/if}
        </table>
    </div>
    <Pagination links={contacts.links} />
</Layout>
