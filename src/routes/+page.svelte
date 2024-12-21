<script>
    import Carousel from 'svelte-carousel';

    async function getallfortniteskins() {
        const res = await fetch('https://fortnite-api.com/v2/shop');
        let data = await res.json();
        //console.log(data);
        //order same rank by regularPrice
        data.data.entries.sort((a, b) => (b.regularPrice > a.regularPrice) ? 1 : -1);
        //order same rank by sortPriority
        data.data.entries.sort((a, b) => (b.sortPriority > a.sortPriority) ? 1 : -1);
        //order same rank by regularPrice
        data.data.entries.sort((a, b) => (a.regularPrice > b.regularPrice) ? 1 : -1);
        //reorder data.data.entries by layout.rank
        data.data.entries.sort((a, b) => (b.layout.rank > a.layout.rank) ? 1 : -1);
        //remove all layout.name == 'Jam Tracks'
        data.data.entries = data.data.entries.filter(item => item.layout.name !== 'Jam Tracks' && item.layout.name !== 'OG Season Shop');
        console.log(data.data.entries);
        return data.data.entries;
    }

    function thaiDate(date) {
        const options = { year: 'numeric', month: 'long', day: 'numeric' };
        return new Date(date).toLocaleDateString('th-TH', options) + ' เวลา ' + new Date(date).toLocaleTimeString('th-TH').slice(0, 5) + ' น.';
    }

    function thaiDateShort(date) {
        //day/month/year format
        return new Date(date).toLocaleDateString('th-TH').slice(0, 10);
    }

    function thaiDateAndShortYear(date) {
        //day/month/year format
        return new Date(date).toLocaleDateString('th-TH').slice(0, 6) + '' + (new Date(date).getFullYear() + 543).toString().slice(2, 4);
    }
</script>

<!--h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://svelte.dev/docs/kit">svelte.dev/docs/kit</a> to read the documentation</p-->

{#await getallfortniteskins()}
	<p>กำลังโหลดข้อมูล...</p>
{:then posts}
<div class="container mx-auto p-4 text-white">
    <div class="grid grid-cols-4 gap-4">
        {#each posts as post}
            {#if post.bundle}
                {#if post.tileSize == 'Size_1_x_1'}
                    {#if post.colors.color2}
                        <div class="relative col-span-1 aspect-[.627] rounded-lg" style="background-image: url({post.bundle.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                            <div class="grid justify-items-stretch inline-grid grid-cols-10 w-full">
                                {#if post.banner?.backendValue == 'New'}
                                <span class="justify-self-start rounded-lg bg-yellow-300 m-2 px-2 py-1 text-lg font-bold text-black">มาใหม่!</span>
                                {:else if post.regularPrice-post.finalPrice > 0}
                                <span class="justify-self-start col-span-3 rounded-lg bg-white m-2 px-2 py-1 text-lg truncate font-bold text-black">ลดไป {post.regularPrice-post.finalPrice}</span>
                                <span class="justify-self-end col-span-7 rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDateAndShortYear(post.outDate)}</span>
                                {:else}
                                <span class="justify-self-end col-span-10 rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDateShort(post.outDate)}</span>
                                {/if}
                            </div>
                            <div class="absolute inset-x-0 bottom-0 py-4 rounded-b-lg backdrop-blur-md">
                                <span class="ml-4">
                                    {post.bundle.name}
                                </span>
                                <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                            </div>
                        </div>
                    {:else}
                        <div class="relative col-span-1 aspect-[.627] rounded-lg" style="background-image: url({post.bundle.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                            <div class="grid justify-items-stretch inline-grid grid-cols-10 w-full">
                                {#if post.banner?.backendValue == 'New'}
                                <span class="justify-self-start rounded-lg bg-yellow-300 m-2 px-2 py-1 text-lg font-bold text-black">มาใหม่!</span>
                                {:else if post.regularPrice-post.finalPrice > 0}
                                <span class="justify-self-start col-span-3 rounded-lg bg-white m-2 px-2 py-1 text-lg truncate font-bold text-black">ลดไป {post.regularPrice-post.finalPrice}</span>
                                <span class="justify-self-end col-span-7 rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDateAndShortYear(post.outDate)}</span>
                                {:else}
                                <span class="justify-self-end col-span-10 rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDateShort(post.outDate)}</span>
                                {/if}
                            </div>
                            <div class="absolute inset-x-0 bottom-0 py-4 rounded-b-lg backdrop-blur-md">
                                <span class="ml-4">
                                    {post.bundle.name}
                                </span>
                                {#if post.regularPrice-post.finalPrice > 0}
                                <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                                {:else}
                                <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                {/if}
                            </div>
                        </div>
                    {/if}
                {:else if post.tileSize == 'Size_2_x_1'}
                    {#if post.colors.color2}
                        <div class="relative col-span-2 aspect-[1/.76] rounded-lg" style="background-image: url({post.bundle.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                            <div class="grid justify-items-stretch inline-grid grid-cols-2 w-full">
                                {#if post.regularPrice-post.finalPrice > 0}
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">ลดไป {post.regularPrice-post.finalPrice} V-Bucks</span>
                                <span class="justify-self-end rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                {:else}
                                <span class="justify-self-end col-span-2 rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                {/if}
                            </div>
                            <div class="absolute inset-x-0 bottom-0 py-4 rounded-b-lg backdrop-blur-md">
                                <span class="ml-4">
                                    {post.bundle.name}
                                </span>
                                {#if post.regularPrice-post.finalPrice > 0}
                                <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                                {:else}
                                <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                {/if}
                            </div>
                        </div>
                    {:else}
                        <div class="relative col-span-2 aspect-[1/.76] rounded-lg" style="background-image: url({post.bundle.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                            <div class="grid justify-items-stretch inline-grid grid-cols-2 w-full">
                                {#if post.regularPrice-post.finalPrice > 0}
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">ลดไป {post.regularPrice-post.finalPrice} V-Bucks</span>
                                <span class="justify-self-end rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                {:else}
                                <span class="justify-self-end col-span-2 rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                {/if}
                            </div>
                            <div class="absolute inset-x-0 bottom-0 py-4 rounded-b-lg backdrop-blur-md">
                                <span class="ml-4">
                                    {post.bundle.name}
                                </span>
                                {#if post.regularPrice-post.finalPrice > 0}
                                <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                                {:else}
                                <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                {/if}
                            </div>
                        </div>
                    {/if}
                {:else if post.tileSize == 'Size_3_x_1'}
                    {#if post.colors.color2}
                        <div class="relative col-span-3 aspect-[1/.516] rounded-lg" style="background-image: url({post.bundle.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 50%;">
                            <div class="grid justify-items-stretch inline-grid grid-cols-2 w-full">
                                {#if post.regularPrice-post.finalPrice > 0}
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">ลดไป {post.regularPrice-post.finalPrice} V-Bucks</span>
                                <span class="justify-self-end rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                {:else}
                                <span class="justify-self-end col-span-2 rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                {/if}
                            </div>
                            <div class="absolute inset-x-0 bottom-0 py-4 rounded-b-lg backdrop-blur-md">
                                <span class="ml-4">
                                    {post.bundle.name}
                                </span>
                                {#if post.regularPrice-post.finalPrice > 0}
                                <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                                {:else}
                                <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                {/if}
                            </div>
                        </div>
                    {:else}
                        <div class="col-span-3 aspect-[1/.516] rounded-lg" style="background-image: url({post.bundle.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 50%;">
                            {post.bundle.name}
                        </div>
                    {/if}
                {:else}
                    {#if post.colors.color2}
                    <div class="relative col-span-4 w-full rounded-lg">
                        <div class="absolute z-10 inset-x-0 top-0 grid justify-items-stretch inline-grid grid-cols-2 w-full">
                            <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">ลดไป {post.regularPrice-post.finalPrice} V-Bucks</span>
                            <span class="justify-self-end rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                        </div>
                        <Carousel let:loaded autoplay dots={false} arrows={false} swiping={false} pauseOnFocus>
                            {#each post.newDisplayAsset.renderImages as bgimage}
                                <div class="aspect-[1/.38625] rounded-lg" style="background-image: url(https://img.gs/fhcphvsghs/quality=low/{bgimage.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                                </div>
                            {/each}
                        </Carousel>
                        <div class="absolute inset-x-0 bottom-0 py-4 rounded-b-lg backdrop-blur-md">
                            <span class="ml-4">
                                {post.bundle.name}
                            </span>
                            <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                        </div>
                    </div>
                        <!--div class="relative col-span-4 aspect-[1/.38625] rounded-lg" style="background-image: url({post.bundle.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                            <div class="grid justify-items-stretch inline-grid grid-cols-2 w-full">
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">ลดไป {post.regularPrice-post.finalPrice} V-Bucks</span>
                                <span class="justify-self-end rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                            </div>
                            <div class="absolute inset-x-0 bottom-0 py-4 rounded-b-lg backdrop-blur-md">
                                <span class="ml-4">
                                    {post.bundle.name}
                                </span>
                                <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                            </div>
                        </div-->
                    {:else}
                        <div class="relative col-span-4 w-full rounded-lg">
                            <div class="absolute z-10 inset-x-0 top-0 grid justify-items-stretch inline-grid grid-cols-2 w-full">
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">ลดไป {post.regularPrice-post.finalPrice} V-Bucks</span>
                                <span class="justify-self-end rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                            </div>
                            <Carousel let:loaded autoplay dots={false} arrows={false} swiping={false} pauseOnFocus>
                                {#each post.newDisplayAsset.renderImages as bgimage}
                                    <div class="aspect-[1/.38625] rounded-lg" style="background-image: url(https://img.gs/fhcphvsghs/quality=low/{bgimage.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                                    </div>
                                {/each}
                            </Carousel>
                            <div class="absolute inset-x-0 bottom-0 py-4 rounded-b-lg backdrop-blur-md">
                                <span class="ml-4">
                                    {post.bundle.name}
                                </span>
                                <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                            </div>
                        </div>
                        <!--div class="relative col-span-4 aspect-[1/.38625] rounded-lg" style="background-image: url({post.bundle.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                            <div class="grid justify-items-stretch inline-grid grid-cols-2 w-full">
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">ลดไป {post.regularPrice-post.finalPrice} V-Bucks</span>
                                <span class="justify-self-end rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                            </div>
                            <div class="absolute inset-x-0 bottom-0 py-4 rounded-b-lg backdrop-blur-md">
                                <span class="ml-4">
                                    {post.bundle.name}
                                </span>
                                <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                            </div>
                        </div-->
                    {/if}
                {/if}
            {:else}
                {#if post.newDisplayAsset}
                    {#if post.tileSize == 'Size_2_x_1'}
                        {#if post.colors.color2}
                            <div class="relative col-span-2 aspect-[1/.76] rounded-lg" style="background-image: url({post.newDisplayAsset.renderImages[0].image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                                <div class="grid justify-items-stretch inline-grid grid-cols-3 w-full">
                                    {#if post.banner?.backendValue == 'New'}
                                    <span class="justify-self-start rounded-lg bg-yellow-300 m-2 px-2 py-1 text-lg font-bold text-black">มาใหม่!</span>
                                    <span class="justify-self-end col-span-2 rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                    {:else}
                                    <span class="justify-self-end col-span-3 rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                    {/if}
                                </div>
                                <div class="absolute inset-x-0 bottom-0 py-4 rounded-b-lg backdrop-blur-md">
                                    <span class="ml-4">
                                        {#if post.brItems}
                                            {post.brItems[0].name}
                                        {/if}
                                        {#if post.instruments}
                                            {post.instruments[0].name}
                                        {/if}
                                    </span>
                                    <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                </div>
                            </div>
                        {:else}
                            <div class="relative col-span-2 aspect-[1/.76] rounded-lg" style="background-image: url({post.newDisplayAsset.renderImages[0].image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                                <div class="grid justify-items-stretch inline-grid grid-cols-3 w-full">
                                    {#if post.banner?.backendValue == 'New'}
                                    <span class="justify-self-start rounded-lg bg-yellow-300 m-2 px-2 py-1 text-lg font-bold text-black">มาใหม่!</span>
                                    <span class="justify-self-end col-span-2 rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                    {:else}
                                    <span class="justify-self-end col-span-3 rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                    {/if}
                                </div>
                                <div class="absolute inset-x-0 bottom-0 py-4 rounded-b-lg backdrop-blur-md">
                                    <span class="ml-4">
                                        {#if post.brItems}
                                            {post.brItems[0].name}
                                        {/if}
                                        {#if post.instruments}
                                            {post.instruments[0].name}
                                        {/if}
                                    </span>
                                    <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                </div>
                            </div>
                        {/if}
                    {:else}
                        {#if post.colors.color2}
                            <div class="relative h-full rounded-lg aspect-[.627]" style="background-image: url({post.newDisplayAsset.renderImages[0].image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                                <div class="grid justify-items-stretch inline-grid grid-cols-10 w-full">
                                    {#if post.banner?.backendValue == 'New'}
                                    <span class="justify-self-start col-span-3 rounded-lg bg-yellow-300 m-2 px-2 py-1 text-lg font-bold text-black">มาใหม่!</span>
                                    <span class="justify-self-end col-span-7 rounded-lg bg-white m-2 px-2 py-1 text-lg truncate font-bold text-black">อยู่จนถึงวันที่ {thaiDateShort(post.outDate)}</span>
                                    {:else}
                                    <span class="justify-self-end col-span-10 rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDateShort(post.outDate)}</span>
                                    {/if}
                                </div>
                                <div class="absolute inset-x-0 bottom-0 py-4 rounded-b-lg backdrop-blur-md">
                                    <span class="ml-4">
                                        {#if post.brItems}
                                            {post.brItems[0].name}
                                        {/if}
                                        {#if post.instruments}
                                            {post.instruments[0].name}
                                        {/if}
                                    </span>
                                    <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                </div>
                            </div>
                        {:else}
                            <div class="relative h-full rounded-lg aspect-[.627]" style="background-image: url({post.newDisplayAsset.renderImages[0].image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                                <div class="grid justify-items-stretch inline-grid grid-cols-10 w-full">
                                    {#if post.banner?.backendValue == 'New'}
                                    <span class="justify-self-start col-span-3 rounded-lg bg-yellow-300 m-2 px-2 py-1 text-lg font-bold text-black">มาใหม่!</span>
                                    <span class="justify-self-end col-span-7 rounded-lg bg-white m-2 px-2 py-1 text-lg truncate font-bold text-black">อยู่จนถึงวันที่ {thaiDateShort(post.outDate)}</span>
                                    {:else}
                                    <span class="justify-self-end col-span-10 rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDateShort(post.outDate)}</span>
                                    {/if}
                                </div>
                                <div class="absolute inset-x-0 bottom-0 py-4 rounded-b-lg backdrop-blur-md">
                                    <span class="ml-4">
                                        {#if post.brItems}
                                            {post.brItems[0].name}
                                        {/if}
                                        {#if post.instruments}
                                            {post.instruments[0].name}
                                        {/if}
                                    </span>
                                    <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                </div>
                            </div>
                        {/if}
                    {/if}
                {:else}
                    {#if post.colors }
                        <div class="h-full rounded-lg" style="background: linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: top;">
                            <img class="rounded-lg p-2" src={post.tracks[0].albumArt} alt={post.tracks[0].title} />
                            {post.tracks[0].title}
                        </div>
                    {:else}
                        <div class="aspect-[.627] rounded-lg" style="background-size: cover; background-position: top;">
                            <img class="rounded-lg p-2" src={post.tracks[0].albumArt} alt={post.tracks[0].title} />
                            {post.tracks[0].title}
                        </div>
                    {/if}
                {/if}
            {/if}
        {/each}
    </div>
</div>
{:catch error}
	<p style="color: red">{error.message}</p>
{/await}