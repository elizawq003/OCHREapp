<script lang="ts">
 import * as Table from "$lib/components/ui/table/index.js";
 import type { SpatialUnit } from 'ochre-sdk';
 import type {Set} from 'ochre-sdk';
 import { getUniquePropertyLabels } from 'ochre-sdk';
 import { Input } from "$lib/components/ui/input/index.js";
 import { getPropertyValueByLabel } from 'ochre-sdk';
	import { isTemplateExpression } from "typescript";
import { MapLibre, DefaultMarker } from 'svelte-maplibre'; 
import {filterProperties} from 'ochre-sdk';
 
 

 const { data }: { data: Set}  = $props();
 const originalData = $derived(data);
 const spatialUnits = $derived(data.items as SpatialUnit[]); 
 const setItems = $derived(data.items);
 const propertyLabels = $derived(getUniquePropertyLabels(spatialUnits[0]!.properties));
 

 let search = $state(''); 
/*
 let filteredData: SpatialUnit[] = $derived(
    search === ''
    ? spatialUnits
    : spatialUnits.filter((item) => {
        const searchLower = search.toLocaleLowerCase();
        const nameMatch =  (item.identification?.label ?? '').toLocaleLowerCase().includes(searchLower);
        const propertyMatch = propertyLabels.some((label) =>
         String(getPropertyValueByLabel(item.properties, label) ?? '').toLocaleLowerCase().includes(searchLower));

        return nameMatch || propertyMatch

    })

 );
 */

// use helper function
let filteredData = $derived(
    search === ''
    ? spatialUnits
    : spatialUnits.filter((unit) => {
        const nameMatch =  (unit.identification?.label ?? '').toLocaleLowerCase().includes(search.toLocaleLowerCase());
        const propertyMatch = unit.properties.some((prop) =>
        filterProperties(
            prop,
            { label: 'all fields', value: search }, 
            { includeNestedProperties: true } )

    );
        return nameMatch || propertyMatch
    })
);

</script>

<div class = "p-8">
<h1 class = "text-2xl font-bold">Objects discovered outside the kingdom of Ugarit</h1>
<p> These 11 objetcs are related to the kingdom of Ugarit and were found **outside** the kingdom.</p>
<div class="container mt-8">
    <MapLibre
        zoom = {4}
        center={[33.9292, 36.0369]}
        class="h-[400px] rounded-lg shadow-2xl shadow-gray-300"
        style="https://basemaps.cartocdn.com/gl/voyager-gl-style/style.json"
        >
        {#each filteredData as item (item.uuid)}
            {#each item.coordinates as coord} 
                {#if coord.type == 'point'}
                    <DefaultMarker lngLat={[coord.longitude, coord.latitude]} />
                {/if}
            {/each}
        {/each}
    </MapLibre>

<Input
		type="text"
		bind:value={search}
		class="w-2/3 sm:w-96 rounded-sm border-2 mt-8 ml-20"
		placeholder="Filter by name, object type, and material "
	/>

</div>
<div class = "container mt-8">
    <h2 class = "mb-4 text-2xl font-bold">Objects</h2>
    <Table.Root>
        <Table.Header>
            <Table.Row>
                <Table.Head>Name</Table.Head>
                {#each propertyLabels as label}
                    <Table.Head>{label}</Table.Head>

                {/each}
            </Table.Row>
        </Table.Header>
        <Table.Body>
            {#each filteredData as item}
            <Table.Row>
                <Table.Cell>
                    <a href={`/${item.uuid}`} class = "text-blue-600 hover:underline ">
                        {item.identification?.label ?? ''}
                    </a>
                </Table.Cell>
                    {#each propertyLabels as label}
                    <Table.Cell>
                        { getPropertyValueByLabel(item.properties,label) ?? ''}
                    </Table.Cell>
                    {/each}
            </Table.Row>
            {/each}
        </Table.Body>
    </Table.Root>

</div>
</div>