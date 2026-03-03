<script lang="ts">
 import type { SpatialUnit } from 'ochre-sdk';
 import { getPropertyValueByLabel } from 'ochre-sdk';
 
 const {data} = $props();
 const properties = $derived(data.observations[0]?.properties ?? []);
 $inspect('image:', data.image);

/*
image is not loading: the url to image is broken or in wrong format
data.image.url: "https://ochre.lib.uchicago.edu/ochre/v2/ochre.php?uuid=ed496a56-a601-49e6-931a-cdbbbcdebd49&preview"
url that works:https://ochre.lib.uchicago.edu/ochre?uuid=ed496a56-a601-49e6-931a-cdbbbcdebd49&preview
*/
//AI: use helper function to get correct url 
function getImageUrl(image: any): string | null {
    if (!image?.url) return null;
    try {
         const url = new URL(image.url);
         const uuid = url.searchParams.get('uuid');
         if (!uuid) return null;
         return `https://ochre.lib.uchicago.edu/ochre?uuid=${uuid}&preview`;
     } catch {
         return null;
     }
}

const imageUrl = $derived(getImageUrl(data.image));

</script>

<div class = "container mx-auto max-w-4xl p-10 space-y-6">
    <h1 class = "text-2xl font-bold">{data.identification?.label}</h1>
    <h2>{data.description}</h2>
    <!--- 
    {#if data.image?.url}
         <img src={data.image.url} alt={data.identification?.label} />
    {:else}
     <p>[ No image available ]</p>
    {/if}
    -->
    {#if imageUrl}
        <img src={imageUrl} alt={data.identification?.label} />
    {:else}
        <p>[ No image available ]</p>
    {/if}

    <!-- get dynamic properties no hard coding-->
    <ul>
        {#each properties as p}
            <!--property does not have content, content is in values: Array<PropertyValueContent<T>>;-->
            <!-- only have the first property value here -->
            <li><span class="font-bold">{p.label}</span>: {p.values[0]?.content ?? ''}</li>
        {/each}

    </ul>
</div>
<!-- 
<p>Museum Number: {getPropertyValueByLabel(data.observations[0]?.properties, "Museum Number") ?? ''}</p>
<p>Publicaton: {getPropertyValueByLabel(data.observations[0]?.properties, "Publication") ?? ''}</p>
<p>KTU: {getPropertyValueByLabel(data.observations[0]?.properties, "KTU") ?? ''}</p>
<p>Size: {getPropertyValueByLabel(data.observations[0]?.properties, "Size") ?? ''}</p>
<p>Material: {getPropertyValueByLabel(data.observations[0]?.properties, "Material") ?? ''}</p>
<p>Classification: {getPropertyValueByLabel(data.observations[0]?.properties, "Classification") ?? ''}</p>
<p>Associated text: {getPropertyValueByLabel(data.observations[0]?.properties, "Associated text") ?? ''}</p>
-->