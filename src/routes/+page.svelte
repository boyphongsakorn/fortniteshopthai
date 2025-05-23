<script>
    import { page } from '$app/stores';
    import Carousel from 'svelte-carousel';

    const out = $page.url.searchParams.get('out');
    let itemNoLayout

    async function getallfortniteskins() {
        // old
        const res = await fetch('https://fortnite-api.com/v2/shop');
        let data = await res.json();
        //console.log(data);
        //order same rank by regularPrice
        data.data.entries.sort((a, b) => (b.regularPrice > a.regularPrice) ? 1 : -1);
        //order same rank by sortPriority
        data.data.entries.sort((a, b) => (b.sortPriority > a.sortPriority) ? 1 : -1);
        //order same rank by regularPrice
        // data.data.entries.sort((a, b) => (a.regularPrice > b.regularPrice) ? 1 : -1);
        //order by banner.intensity == 'High' is first
        // data.data.entries.sort((a, b) => (b.banner?.intensity == 'High' && a.banner?.intensity == 'High') ? 1 : -1);
        //order by have bundle is first
        // data.data.entries.sort((a, b) => (b.bundle && a.bundle) ? 1 : -1);
        //order by layoutId
        data.data.entries.sort((a, b) => (a.layoutId > b.layoutId) ? 1 : -1);
        //get item don't have layout to new array
        //itemNoLayout = data.data.entries.filter(item => !item.layout);
        //remove data.data.entries don't have layout
        //data.data.entries = data.data.entries.filter(item => item.layout);
        //reorder data.data.entries by layout.rank
        data.data.entries.sort((a, b) => b.layout && a.layout ? (b.layout.rank > a.layout.rank) ? 1 : -1 : -1);
        //remove all layout.name == 'Jam Tracks' and 'OG Season Shop'
        data.data.entries = data.data.entries.filter(item => item.layout ? item.layout.name != 'Jam Tracks' && item.layout.name != 'OG Season Shop' && item.layout.name != 'Prove Your Power' : true);
        console.log(data.data.entries);
        try {
            const res2 = await fetch('https://localpost.teamquadb.in.th/fortniteitemshop', {signal : AbortSignal.timeout(12000)});
            let data2 = await res2.json();
            for (let i = 0; i < data.data.entries.length; i++) {
                //find webUrl by offerId in array data2.shop
                // data2.shop = data2.shop.filter(item => item.offerId == data.data.entries[i].offerId);
                // return data.shop.webURL;
                if (data2.shop.filter(item => item.offerId == data.data.entries[i].offerId).length > 0) {
                    data.data.entries[i].webURL = "https://www.fortnite.com"+data2.shop.filter(item => item.offerId == data.data.entries[i].offerId)[0].webURL+"?creator-code=boyalone99";
                    data.data.entries[i].video = data2.shop.filter(item => item.offerId == data.data.entries[i].offerId)[0].video.split('?')[0];
                } else {
                    data.data.entries[i].webURL = "https://www.fortnite.com/item-shop?creator-code=boyalone99";
                    data.data.entries[i].video = "";
                }
                //check data.data.entries[i].outDate is less than 24 hours add data.data.entries[i].outDateToday is true
                if (new Date(data.data.entries[i].outDate).getTime() - new Date().getTime() < 24 * 60 * 60 * 1000) {
                    data.data.entries[i].outDateToday = true;
                } else {
                    data.data.entries[i].outDateToday = false;
                }
            }
        } catch (error) {
            console.log(error);
            //add "https://www.fortnite.com/item-shop?creator-code=boyalone99" to every data.data.entries
            for (let i = 0; i < data.data.entries.length; i++) {
                data.data.entries[i].webURL = "https://www.fortnite.com/item-shop?creator-code=boyalone99";
                data.data.entries[i].video = "";
            }
        }
        if (out) {
            //get only item that outDate is today by outDate == today
            data.data.entries = data.data.entries.filter(item => item.outDate.slice(0, 10) == new Date().toISOString().slice(0, 10));
        }
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
        //return new Date(date).toLocaleDateString('th-TH').slice(0, 3) + '/' + (new Date(date).getFullYear() + 543).toString().slice(2, 4);
        return new Date(date).getDate() + '/' + (new Date(date).getMonth() + 1) + '/' + (new Date(date).getFullYear() + 543).toString().slice(2, 4);
    }

    function shopout() {
        //if less 7 hours to 00:00 (UTC time) return วันนี้
        if (new Date().getUTCHours() >= 17) {
            return 'วันนี้ (7 โมงเช้า)';
        } else {
            return 'พรุ่งนี้ (7 โมงเช้า)';
        }
    }

    function load() {
        window.location.href = '';
    }

    let innerWidth = 0
</script>
<svelte:window bind:innerWidth/>



<!--h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://svelte.dev/docs/kit">svelte.dev/docs/kit</a> to read the documentation</p-->

{#await getallfortniteskins()}
	<p>กำลังโหลดข้อมูล...</p>
{:then posts}
<div class="container mx-auto p-4 text-white">
    <div class="grid grid-cols-2">
        <div>
            {#if out}
            <a class="bg-blue-500 hover:bg-blue-700 text-sm text-white font-bold p-2 mb-2 rounded inline-flex" href="/" on:click="{() => window.location.href = '/'}">
                ของที่จะออก{shopout()}
                <svg fill="none" viewBox="0 0 24 24" height="20" width="20" xmlns="http://www.w3.org/2000/svg">
                    <path xmlns="http://www.w3.org/2000/svg" d="M5.29289 5.29289C5.68342 4.90237 6.31658 4.90237 6.70711 5.29289L12 10.5858L17.2929 5.29289C17.6834 4.90237 18.3166 4.90237 18.7071 5.29289C19.0976 5.68342 19.0976 6.31658 18.7071 6.70711L13.4142 12L18.7071 17.2929C19.0976 17.6834 19.0976 18.3166 18.7071 18.7071C18.3166 19.0976 17.6834 19.0976 17.2929 18.7071L12 13.4142L6.70711 18.7071C6.31658 19.0976 5.68342 19.0976 5.29289 18.7071C4.90237 18.3166 4.90237 17.6834 5.29289 17.2929L10.5858 12L5.29289 6.70711C4.90237 6.31658 4.90237 5.68342 5.29289 5.29289Z" fill="#0D0D0D"></path>
                </svg>
            </a>
            {:else}
            <a class="bg-blue-500 hover:bg-blue-700 text-sm text-white font-bold p-2 mb-2 rounded inline-flex" href="/?out=true" on:click="{() => window.location.href = '/?out=true'}">
                ของที่จะออก{shopout()}
            </a>
            {/if}
        </div>
        <div class="text-end">
            <a class="bg-slate-50 hover:bg-slate-100 text-sm text-black font-bold p-2 mb-2 rounded inline-flex" href="https://fortnite.com/@boyalone99" target="_blank">
                Favorite boyalone99 ใน Fortnite
            </a>
            <a class="bg-purple-700 hover:bg-purple-600 text-sm font-bold p-2 mb-2 rounded inline-flex" href="https://twitch.tv/boyalone99" target="_blank">
                ติดตาม boyalone99 บน Twitch
            </a>
        </div>
    </div>
    <div class="grid grid-cols-4 gap-4">
        {#each posts as post}
            {#if post.bundle}
                {#if post.tileSize == 'Size_1_x_1'}
                    {#if post.colors.color2}
                        <a class="relative col-span-2 md:col-span-1 max-md:aspect-[1/1] aspect-[.627] rounded-lg" href={post.webURL} style="background-image: url({post.bundle.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                            <div class="grid justify-items-stretch inline-grid grid-cols-10 w-full">
                                {#if post.banner?.backendValue == 'New'}
                                <span class="justify-self-start rounded-lg bg-yellow-300 m-2 px-2 py-1 text-lg font-bold text-black">มาใหม่!</span>
                                {:else if post.regularPrice-post.finalPrice > 0}
                                <span class="justify-self-start col-span-3 rounded-lg bg-white m-2 px-2 py-1 text-lg max-lg:text-sm truncate font-bold text-black">ลด {post.regularPrice-post.finalPrice}</span>
                                <span class="justify-self-end col-span-7 rounded-lg bg-white m-2 px-2 py-1 text-lg max-lg:text-sm truncate font-bold text-black">ออก {thaiDateAndShortYear(post.outDate)}</span>
                                {:else}
                                <span class="justify-self-end col-span-10 rounded-lg bg-white m-2 px-2 py-1 text-lg max-md:text-sm font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                {/if}
                            </div>
                            <div class="absolute inset-x-0 bottom-0 py-1 md:py-4 rounded-b-lg backdrop-blur-md">
                                <span class="ml-4">
                                    {post.bundle.name}
                                </span>
                                {#if post.regularPrice-post.finalPrice > 0}
                                <p class="text-sm md:text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                                {:else}
                                <p class="text-sm md:text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                {/if}
                            </div>
                        </a>
                    {:else}
                        <a class="relative col-span-2 md:col-span-1 max-md:aspect-[1/1] aspect-[.627] rounded-lg" href={post.webURL} style="background-image: url({post.bundle.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                            <div class="grid justify-items-stretch inline-grid grid-cols-10 w-full">
                                {#if post.banner?.backendValue == 'New'}
                                <span class="justify-self-start rounded-lg bg-yellow-300 m-2 px-2 py-1 text-lg font-bold text-black">มาใหม่!</span>
                                {:else if post.regularPrice-post.finalPrice > 0}
                                <span class="justify-self-start col-span-3 rounded-lg bg-white m-2 px-2 py-1 text-lg max-sm:text-sm truncate font-bold text-black">ลด {post.regularPrice-post.finalPrice}</span>
                                <span class="justify-self-end col-span-7 rounded-lg bg-white m-2 px-2 py-1 text-lg max-sm:text-sm font-bold text-black">ออก {thaiDateAndShortYear(post.outDate)}</span>
                                {:else}
                                <span class="justify-self-end col-span-10 rounded-lg bg-white m-2 px-2 py-1 text-lg max-md:text-sm font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                {/if}
                            </div>
                            <div class="absolute inset-x-0 bottom-0 max-sm:py-1 py-4 rounded-b-lg backdrop-blur-md">
                                <span class="max-sm:text-sm ml-4">
                                    {post.bundle.name}
                                </span>
                                {#if post.regularPrice-post.finalPrice > 0}
                                <p class="max-sm:text-sm text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                                {:else}
                                <p class="max-sm:text-sm text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                {/if}
                            </div>
                        </a>
                    {/if}
                {:else if post.tileSize == 'Size_2_x_1'}
                    {#if post.colors.color2}
                        <a class="relative col-span-2 max-md:aspect-[1/1] aspect-[1/.76] rounded-lg h-full w-full" href={post.webURL} style="background-image: url({post.bundle.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                            <div class="grid justify-items-stretch inline-grid grid-cols-2 w-full">
                                {#if post.regularPrice-post.finalPrice > 0}
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 max-md:hidden text-lg truncate font-bold text-black">ลด {post.regularPrice-post.finalPrice} V-Bucks</span>
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 text-sm md:hidden truncate font-bold text-black">ลด {post.regularPrice-post.finalPrice}</span>
                                <span class="justify-self-end rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg xl:hidden truncate font-bold text-black">ออก {thaiDateAndShortYear(post.outDate)}</span>
                                <span class="justify-self-end rounded-lg bg-white m-2 px-2 py-1 max-xl:hidden text-lg truncate font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                {:else}
                                <span class="justify-self-end col-span-2 rounded-lg bg-white m-2 px-2 py-1 max-md:text-sm md:hidden text-lg font-bold text-black">ออก {thaiDateAndShortYear(post.outDate)}</span>
                                <span class="justify-self-end col-span-2 rounded-lg bg-white m-2 px-2 py-1 max-md:hidden lg:text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                {/if}
                            </div>
                            <div class="absolute inset-x-0 bottom-0 max-sm:py-1 py-4 rounded-b-lg backdrop-blur-md">
                                <span class="max-sm:text-sm ml-4">
                                    {post.bundle.name}
                                </span>
                                {#if post.regularPrice-post.finalPrice > 0}
                                <p class="max-sm:text-sm text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                                {:else}
                                <p class="max-sm:text-sm text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                {/if}
                            </div>
                        </a>
                    {:else}
                        <a class="relative col-span-2 max-md:aspect-[1/1] aspect-[1/.76] rounded-lg h-full w-full" href={post.webURL} style="background-image: url({post.bundle.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                            <div class="grid justify-items-stretch inline-grid grid-cols-2 w-full">
                                {#if post.regularPrice-post.finalPrice > 0}
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 max-md:hidden text-lg truncate font-bold text-black">ลด {post.regularPrice-post.finalPrice} V-Bucks</span>
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 text-sm md:hidden truncate font-bold text-black">ลด {post.regularPrice-post.finalPrice}</span>
                                <span class="justify-self-end rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg xl:hidden truncate font-bold text-black">ออก {thaiDateAndShortYear(post.outDate)}</span>
                                <span class="justify-self-end rounded-lg bg-white m-2 px-2 py-1 max-xl:hidden text-lg truncate font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                {:else}
                                <span class="justify-self-end col-span-2 rounded-lg bg-white m-2 px-2 py-1 max-md:text-sm md:hidden text-lg font-bold text-black">ออก {thaiDateAndShortYear(post.outDate)}</span>
                                <span class="justify-self-end col-span-2 rounded-lg bg-white m-2 px-2 py-1 max-md:hidden lg:text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                {/if}
                            </div>
                            <div class="absolute inset-x-0 bottom-0 max-sm:py-1 py-4 rounded-b-lg backdrop-blur-md">
                                <span class="max-sm:text-sm text-ellipsis ml-4">
                                    {post.bundle.name}
                                </span>
                                {#if post.regularPrice-post.finalPrice > 0}
                                <p class="max-sm:text-sm text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                                {:else}
                                <p class="max-sm:text-sm text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                {/if}
                            </div>
                        </a>
                    {/if}
                {:else if post.tileSize == 'Size_3_x_1'}
                    {#if post.colors.color2}
                        <a class="relative max-md:col-span-2 col-span-3 max-md:aspect-[1/1] aspect-[1/.516] rounded-lg" href={post.webURL} style="background-image: url({post.bundle.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% {post.cars ? '50' : '10'}%;">
                            <div class="grid justify-items-stretch inline-grid grid-cols-2 w-full">
                                {#if post.regularPrice-post.finalPrice > 0}
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 max-md:hidden text-lg truncate font-bold text-black">ลด {post.regularPrice-post.finalPrice} V-Bucks</span>
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 md:hidden text-sm truncate font-bold text-black">ลด {post.regularPrice-post.finalPrice}</span>
                                <span class="justify-self-end rounded-lg bg-white m-2 px-2 py-1 max-md:hidden text-lg truncate font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                <span class="justify-self-end rounded-lg bg-white m-2 px-2 py-1 md:hidden text-sm truncate font-bold text-black">ออก {thaiDateAndShortYear(post.outDate)}</span>
                                {:else}
                                <span class="justify-self-end col-span-2 rounded-lg bg-white m-2 px-2 py-1 max-md:hidden text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                <span class="justify-self-end col-span-2 rounded-lg bg-white m-2 px-2 py-1 md:hidden text-sm font-bold text-black">ออก {thaiDateAndShortYear(post.outDate)}</span>
                                {/if}
                            </div>
                            <div class="absolute inset-x-0 bottom-0 max-sm:py-1 py-4 rounded-b-lg backdrop-blur-md max-sm:text-sm">
                                <span class="ml-4">
                                    {post.bundle.name}
                                </span>
                                {#if post.regularPrice-post.finalPrice > 0}
                                <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                                {:else}
                                <p class="text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                {/if}
                            </div>
                        </a>
                    {:else}
                        <a class="relative max-md:col-span-2 col-span-3 max-md:aspect-[1/1] aspect-[1/.516] rounded-lg" href={post.webURL} style="background-image: url({post.bundle.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% {post.cars ? '50' : '10'}%;">
                            <div class="grid justify-items-stretch inline-grid grid-cols-2 w-full">
                                {#if post.regularPrice-post.finalPrice > 0}
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 max-md:hidden text-lg truncate font-bold text-black">ลด {post.regularPrice-post.finalPrice} V-Bucks</span>
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 md:hidden text-sm truncate font-bold text-black">ลด {post.regularPrice-post.finalPrice}</span>
                                <span class="justify-self-end rounded-lg bg-white m-2 px-2 py-1 max-md:hidden text-lg truncate font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                <span class="justify-self-end rounded-lg bg-white m-2 px-2 py-1 md:hidden text-sm truncate font-bold text-black">ออก {thaiDateAndShortYear(post.outDate)}</span>
                                {:else}
                                <span class="justify-self-end col-span-2 rounded-lg bg-white m-2 px-2 py-1 max-md:hidden text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                <span class="justify-self-end col-span-2 rounded-lg bg-white m-2 px-2 py-1 md:hidden text-sm font-bold text-black">ออก {thaiDateAndShortYear(post.outDate)}</span>
                                {/if}
                            </div>
                            <div class="absolute inset-x-0 bottom-0 max-sm:py-1 py-4 rounded-b-lg backdrop-blur-md max-sm:text-sm">
                                <span class="ml-4">
                                    {post.bundle.name}
                                </span>
                                {#if post.regularPrice-post.finalPrice > 0}
                                <p class="max-sm:text-sm text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                                {:else}
                                <p class="max-sm:text-sm text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                {/if}
                            </div>
                        </a>
                    {/if}
                {:else}
                    {#if post.colors.color2}
                        <a class="relative col-span-2 md:col-span-4 w-full rounded-lg" href={post.webURL}>
                            <div class="max-sm:hidden absolute z-10 inset-x-0 top-0 grid justify-items-stretch inline-grid grid-cols-2 w-full">
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg truncate font-bold text-black">ลด {post.regularPrice-post.finalPrice} V-Bucks</span>
                                <span class="justify-self-end max-md:hidden rounded-lg bg-white m-2 px-2 py-1 text-lg truncate font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                <span class="justify-self-end md:hidden rounded-lg bg-white m-2 px-2 py-1 text-sm truncate font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                            </div>
                            <Carousel let:loaded autoplay dots={false} arrows={false} swiping={false} pauseOnFocus>
                                {#each post.newDisplayAsset.renderImages as bgimage}
                                    <div class="aspect-[1/1] md:aspect-[1/.38625] rounded-lg" style="background-image: url(https://img.gs/fhcphvsghs/quality=low/{bgimage.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                                    </div>
                                {/each}
                            </Carousel>
                            <div class="absolute inset-x-0 bottom-0 max-md:py-1 py-4 rounded-b-lg backdrop-blur-md">
                                <span class="max-sm:text-sm ml-4">
                                    {post.bundle.name}
                                </span>
                                <p class="max-sm:text-sm text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                            </div>
                        </a>
                        <!--div class="relative col-span-4 aspect-[1/.38625] rounded-lg" style="background-image: url({post.bundle.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                            <div class="grid justify-items-stretch inline-grid grid-cols-2 w-full">
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">ลด {post.regularPrice-post.finalPrice} V-Bucks</span>
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
                        <a class="relative col-span-2 md:col-span-4 w-full rounded-lg" href={post.webURL}>
                            <div class="max-sm:hidden absolute z-10 inset-x-0 top-0 grid justify-items-stretch inline-grid grid-cols-2 w-full">
                                {#if post.banner?.backendValue == 'New'}
                                    <span class="justify-self-start rounded-lg bg-yellow-300 m-2 px-2 py-1 text-sm md:text-lg font-bold text-black">มาใหม่!</span>
                                {:else}
                                    <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg truncate font-bold text-black">ลด {post.regularPrice-post.finalPrice} V-Bucks</span>
                                {/if}
                                <span class="justify-self-end max-md:hidden rounded-lg bg-white m-2 px-2 py-1 text-lg truncate font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                <span class="justify-self-end md:hidden rounded-lg bg-white m-2 px-2 py-1 text-sm truncate font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                            </div>
                            <Carousel let:loaded autoplay dots={false} arrows={false} swiping={false} pauseOnFocus>
                                {#each post.newDisplayAsset.renderImages as bgimage}
                                    <div class="aspect-[1/1] md:aspect-[1/.38625] rounded-lg" style="background-image: url(https://img.gs/fhcphvsghs/quality=low/{bgimage.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                                    </div>
                                {/each}
                            </Carousel>
                            <div class="absolute inset-x-0 bottom-0 max-md:py-1 py-4 rounded-b-lg backdrop-blur-md">
                                <span class="max-sm:text-sm ml-4">
                                    {post.bundle.name}
                                </span>
                                <!--p class="max-sm:text-sm text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p-->
                                {#if post.regularPrice-post.finalPrice > 0}
                                    <p class="text-sm md:text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice} <spin class="inline line-through">{post.regularPrice}</spin></p>
                                {:else}
                                    <p class="text-sm md:text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                {/if}
                            </div>
                        </a>
                        <!--div class="relative col-span-4 aspect-[1/.38625] rounded-lg" style="background-image: url({post.bundle.image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                            <div class="grid justify-items-stretch inline-grid grid-cols-2 w-full">
                                <span class="justify-self-start rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">ลด {post.regularPrice-post.finalPrice} V-Bucks</span>
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
                            <a class="relative col-span-2 aspect-[1/1] md:aspect-[1/.76] rounded-lg h-full w-full" href={post.webURL} style="background-image: url({post.brItems && innerWidth < 768 ? post.brItems[0].type.value == 'outfit' ? 'https://img.gs/fhcphvsghs/250x250,crop=0.5x0.2,quality=low/https://img.gs/fhcphvsghs/500x250,crop=top,quality=low/' : '' : ''}{post.newDisplayAsset.renderImages[0].image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                                <div class="grid justify-items-stretch inline-grid grid-cols-3 w-full">
                                    {#if post.banner?.backendValue == 'New'}
                                    <span class="justify-self-start max-sm:text-sm max-sm:truncate rounded-lg bg-yellow-300 m-2 px-2 py-1 md:text-lg font-bold text-black">มาใหม่!</span>
                                    <span class="justify-self-end max-md:hidden col-span-2 rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                    <span class="justify-self-end col-span-2 md:hidden rounded-lg bg-white m-2 px-2 py-1 text-sm truncate font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                    {:else}
                                    <span class="justify-self-end max-md:hidden col-span-3 rounded-lg bg-white m-2 px-2 py-1 lg:text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                    <span class="justify-self-end col-span-3 md:hidden rounded-lg bg-white m-2 px-2 py-1 text-sm truncate font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                    {/if}
                                </div>
                                <div class="absolute inset-x-0 bottom-0 max-md:py-1 py-4 rounded-b-lg backdrop-blur-md">
                                    <span class="max-md:text-sm ml-4">
                                        {#if post.brItems}
                                            {post.brItems[0].name}
                                        {/if}
                                        {#if post.instruments}
                                            {post.instruments[0].name}
                                        {/if}
                                    </span>
                                    <p class="max-md:text-sm text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                </div>
                            </a>
                        {:else}
                            <a class="relative col-span-2 aspect-[1/1] md:aspect-[1/.76] rounded-lg h-full w-full" href={post.webURL} style="background-image: url({post.brItems && innerWidth < 768 ? post.brItems[0].type.value == 'outfit' ? 'https://img.gs/fhcphvsghs/250x250,crop=0.5x0.2,quality=low/https://img.gs/fhcphvsghs/500x250,crop=top,quality=low/' : '' : ''}{post.newDisplayAsset.renderImages[0].image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                                <div class="grid justify-items-stretch inline-grid grid-cols-3 w-full">
                                    {#if post.banner?.backendValue == 'New'}
                                    <span class="justify-self-start max-sm:text-sm max-sm:truncate rounded-lg bg-yellow-300 m-2 px-2 py-1 md:text-lg font-bold text-black">มาใหม่!</span>
                                    <span class="justify-self-end max-md:hidden col-span-2 rounded-lg bg-white m-2 px-2 py-1 text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                    <span class="justify-self-end col-span-2 md:hidden rounded-lg bg-white m-2 px-2 py-1 text-sm truncate font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                    {:else}
                                    <span class="justify-self-end max-md:hidden col-span-3 rounded-lg bg-white m-2 px-2 py-1 lg:text-lg font-bold text-black">อยู่จนถึงวันที่ {thaiDate(post.outDate)}</span>
                                    <span class="justify-self-end col-span-3 md:hidden rounded-lg bg-white m-2 px-2 py-1 text-sm truncate font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                    {/if}
                                </div>
                                <div class="absolute inset-x-0 bottom-0 max-md:py-1 py-4 rounded-b-lg backdrop-blur-md">
                                    <span class="max-md:text-sm ml-4">
                                        {#if post.brItems}
                                            {post.brItems[0].name}
                                        {/if}
                                        {#if post.instruments}
                                            {post.instruments[0].name}
                                        {/if}
                                    </span>
                                    <p class="max-md:text-sm text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                </div>
                            </a>
                        {/if}
                    {:else}
                        {#if post.colors.color2}
                            <a class="relative h-full rounded-lg max-md:col-span-2 aspect-[1/1] md:aspect-[.627]" href={post.webURL} style="background-image: url({post.brItems && innerWidth < 768 ? post.brItems[0].type.value == 'outfit' ? 'https://img.gs/fhcphvsghs/250x250,crop=0.5x0.2,quality=low/https://img.gs/fhcphvsghs/500x250,crop=top,quality=low/' : '' : ''}{post.newDisplayAsset.renderImages[0].image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                                <div class="absolute z-10 inset-x-0 top-0 grid justify-items-stretch inline-grid grid-cols-10 w-full">
                                    {#if post.banner?.backendValue == 'New'}
                                    <span class="justify-self-start col-span-3 rounded-lg bg-yellow-300 m-2 px-2 py-1 text-sm lg:text-lg truncate font-bold text-black">มาใหม่!</span>
                                    <span class="justify-self-end col-span-7 rounded-lg bg-white m-2 px-2 py-1 text-sm lg:hidden truncate font-bold text-black">ออก {thaiDateAndShortYear(post.outDate)}</span>
                                    <span class="justify-self-end col-span-7 rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg max-lg:hidden truncate font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                    {:else}
                                    <span class="justify-self-end col-span-10 rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                    {/if}
                                </div>
                                <video autoplay muted loop class="min-w-full min-h-full object-cover object-top max-md:aspect-[1/1] rounded-lg">
                                    <source src={post.video} type="video/mp4">
                                    Your browser does not support HTML5 video.
                                </video>
                                <div class="absolute inset-x-0 bottom-0 max-md:py-1 py-4 rounded-b-lg backdrop-blur-md">
                                    <span class="max-md:text-sm ml-4">
                                        {#if post.brItems}
                                            {post.brItems[0].name}
                                        {/if}
                                        {#if post.instruments}
                                            {post.instruments[0].name}
                                        {/if}
                                        {#if post.legoKits}
                                            {post.legoKits[0].name}
                                        {/if}
                                    </span>
                                    <p class="max-md:text-sm text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                </div>
                            </a>
                        {:else}
                            <div class="relative h-full rounded-lg max-md:col-span-2 aspect-[1/1] md:aspect-[.627]" style="background-image: url({post.newDisplayAsset.renderImages[0].image}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                                <div class="absolute z-10 inset-x-0 top-0 grid justify-items-stretch inline-grid grid-cols-10 w-full">
                                    {#if post.banner?.backendValue == 'New'}
                                    <span class="justify-self-start col-span-3 rounded-lg bg-yellow-300 m-2 px-2 py-1 text-sm md:text-lg truncate font-bold text-black">มาใหม่!</span>
                                    <span class="justify-self-end col-span-7 rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg truncate font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                    {:else}
                                    <span class="justify-self-end col-span-10 rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                    {/if}
                                </div>
                                <video autoplay muted loop class="min-w-full min-h-full object-cover object-top max-md:aspect-[1/1] rounded-lg">
                                    <source src={post.video} type="video/mp4">
                                    Your browser does not support HTML5 video.
                                </video>
                                <div class="absolute inset-x-0 bottom-0 max-md:py-1 py-4 rounded-b-lg backdrop-blur-md">
                                    <span class="max-md:text-sm ml-4">
                                        {#if post.brItems}
                                            {post.brItems[0].name}
                                        {/if}
                                        {#if post.instruments}
                                            {post.instruments[0].name}
                                        {/if}
                                        {#if post.legoKits}
                                            {post.legoKits[0].name}
                                        {/if}
                                    </span>
                                    <p class="max-md:text-sm text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                </div>
                            </div>
                        {/if}
                    {/if}
                {:else}
                    {#if post.colors }
                        {#if post.colors.color2}
                            {#if post.tracks}
                                <div class="relative max-md:aspect-[1/1] max-md:col-span-2 h-full rounded-lg" style="background: linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: top;">
                                    <div class="absolute grid justify-items-stretch inline-grid grid-cols-10 w-full">
                                        {#if post.banner?.backendValue == 'New'}
                                        <span class="justify-self-start col-span-3 rounded-lg bg-yellow-300 m-2 px-2 py-1 text-sm md:text-lg truncate font-bold text-black">มาใหม่!</span>
                                        <span class="justify-self-end col-span-7 rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg truncate font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                        {:else}
                                        <span class="justify-self-end col-span-10 rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                        {/if}
                                    </div>
                                    <img class="rounded-lg p-2" src={post.tracks? post.tracks[0].albumArt : ""} alt={post.tracks?post.tracks[0].title: ""} />
                                    <div class="absolute inset-x-0 bottom-0 py-1 md:py-4 rounded-b-lg backdrop-blur-md">
                                        <span class="max-md:text-sm ml-4">
                                            {post.tracks?post.tracks[0].title: ""}
                                        </span>
                                        <p class="max-md:text-sm text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                    </div>
                                </div>
                            {:else}
                                <div class="relative h-full rounded-lg max-md:col-span-2 aspect-[1/1] md:aspect-[.627]" style="background-image: url({post.instruments ? post.instruments[0].images.large : post.brItems[0].images.icon}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color2} 50%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                                    <div class="absolute z-10 inset-x-0 top-0 grid justify-items-stretch inline-grid grid-cols-10 w-full">
                                        {#if post.banner?.backendValue == 'New'}
                                        <span class="justify-self-start col-span-3 rounded-lg bg-yellow-300 m-2 px-2 py-1 text-sm md:text-lg truncate font-bold text-black">มาใหม่!</span>
                                        <span class="justify-self-end col-span-7 rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg truncate font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                        {:else}
                                        <span class="justify-self-end col-span-10 rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                        {/if}
                                    </div>
                                    <video autoplay muted loop class="min-w-full min-h-full object-cover object-top max-md:aspect-[1/1] rounded-lg">
                                        <source src={post.video} type="video/mp4">
                                        Your browser does not support HTML5 video.
                                    </video>
                                    <div class="absolute inset-x-0 bottom-0 max-md:py-1 py-4 rounded-b-lg backdrop-blur-md">
                                        <span class="max-md:text-sm ml-4">
                                            {#if post.brItems}
                                                {post.brItems[0].name}
                                            {/if}
                                            {#if post.instruments}
                                                {post.instruments[0].name}
                                            {/if}
                                            {#if post.legoKits}
                                                {post.legoKits[0].name}
                                            {/if}
                                        </span>
                                        <p class="max-md:text-sm text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                    </div>
                                </div>
                            {/if}
                        {:else}
                            <!-- <div class="relative max-md:aspect-[1/1] max-md:col-span-2 h-full rounded-lg" style="background: linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: top;">
                                {#if post.tracks}
                                    <div class="absolute grid justify-items-stretch inline-grid grid-cols-10 w-full">
                                        {#if post.banner?.backendValue == 'New'}
                                        <span class="justify-self-start col-span-3 rounded-lg bg-yellow-300 m-2 px-2 py-1 text-sm md:text-lg truncate font-bold text-black">มาใหม่!</span>
                                        <span class="justify-self-end col-span-7 rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg truncate font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                        {:else}
                                        <span class="justify-self-end col-span-10 rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                        {/if}
                                    </div>
                                    <img class="rounded-lg p-2" src={post.tracks? post.tracks[0].albumArt : ""} alt={post.tracks?post.tracks[0].title: ""} />
                                    <div class="absolute inset-x-0 bottom-0 py-1 md:py-4 rounded-b-lg backdrop-blur-md">
                                        <span class="max-md:text-sm ml-4">
                                            {post.tracks?post.tracks[0].title: ""}
                                        </span>
                                        <p class="max-md:text-sm text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                    </div>
                                {:else}
                                    <div class="absolute grid justify-items-stretch inline-grid grid-cols-10 w-full">
                                        {#if post.banner?.backendValue == 'New'}
                                        <span class="justify-self-start col-span-3 rounded-lg bg-yellow-300 m-2 px-2 py-1 text-sm md:text-lg truncate font-bold text-black">มาใหม่!</span>
                                        <span class="justify-self-end col-span-7 rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg truncate font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                        {:else}
                                        <span class="justify-self-end col-span-10 rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                        {/if}
                                    </div>
                                    <img class="rounded-lg p-2" src={post.instruments[0].images.large} alt={post.instruments[0].name} />
                                    <div class="absolute inset-x-0 bottom-0 py-1 md:py-4 rounded-b-lg backdrop-blur-md">
                                        <span class="max-md:text-sm ml-4">
                                            {post.instruments[0].name}
                                        </span>
                                        <p class="max-md:text-sm text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                    </div>
                                {/if}
                            </div> -->
                            {#if post.tracks}
                                <div class="relative max-md:aspect-[1/1] max-md:col-span-2 h-full rounded-lg" style="background: linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: top;">
                                    <div class="absolute grid justify-items-stretch inline-grid grid-cols-10 w-full">
                                        {#if post.banner?.backendValue == 'New'}
                                        <span class="justify-self-start col-span-3 rounded-lg bg-yellow-300 m-2 px-2 py-1 text-sm md:text-lg truncate font-bold text-black">มาใหม่!</span>
                                        <span class="justify-self-end col-span-7 rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg truncate font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                        {:else}
                                        <span class="justify-self-end col-span-10 rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                        {/if}
                                    </div>
                                    <img class="rounded-lg p-2" src={post.tracks? post.tracks[0].albumArt : ""} alt={post.tracks?post.tracks[0].title: ""} />
                                    <div class="absolute inset-x-0 bottom-0 py-1 md:py-4 rounded-b-lg backdrop-blur-md">
                                        <span class="max-md:text-sm ml-4">
                                            {post.tracks?post.tracks[0].title: ""}
                                        </span>
                                        <p class="max-md:text-sm text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                    </div>
                                </div>
                            {:else}
                                <div class="relative h-full rounded-lg max-md:col-span-2 aspect-[1/1] md:aspect-[.627]" style="background-image: url({post.instruments ? post.instruments[0].images.large : post.brItems[0].images.icon}), linear-gradient(180deg, #{post.colors.color1} 0%, #{post.colors.color3} 100%); background-size: cover; background-position: 50% 10%;">
                                    <div class="absolute z-10 inset-x-0 top-0 grid justify-items-stretch inline-grid grid-cols-10 w-full">
                                        {#if post.banner?.backendValue == 'New'}
                                        <span class="justify-self-start col-span-3 rounded-lg bg-yellow-300 m-2 px-2 py-1 text-sm md:text-lg truncate font-bold text-black">มาใหม่!</span>
                                        <span class="justify-self-end col-span-7 rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg truncate font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                        {:else}
                                        <span class="justify-self-end col-span-10 rounded-lg bg-white m-2 px-2 py-1 text-sm md:text-lg font-bold text-black">ออก {thaiDateShort(post.outDate)}</span>
                                        {/if}
                                    </div>
                                    <video autoplay muted loop class="min-w-full min-h-full object-cover object-top max-md:aspect-[1/1] rounded-lg">
                                        <source src={post.video} type="video/mp4">
                                        Your browser does not support HTML5 video.
                                    </video>
                                    <div class="absolute inset-x-0 bottom-0 max-md:py-1 py-4 rounded-b-lg backdrop-blur-md">
                                        <span class="max-md:text-sm ml-4">
                                            {#if post.brItems}
                                                {post.brItems[0].name}
                                            {/if}
                                            {#if post.instruments}
                                                {post.instruments[0].name}
                                            {/if}
                                            {#if post.legoKits}
                                                {post.legoKits[0].name}
                                            {/if}
                                        </span>
                                        <p class="max-md:text-sm text-lg text-bold"><img class="ml-4 w-[25px] inline" src="https://fortnite-api.com/images/vbuck.png" />{post.finalPrice}</p>
                                    </div>
                                </div>
                            {/if}
                        {/if}
                    {:else}
                        <div class="aspect-[1/1] md:aspect-[.627] rounded-lg" style="background-size: cover; background-position: top;">
                            <img class="rounded-lg p-2" src={post.tracks? post.tracks[0].albumArt : ""} alt={post.tracks?post.tracks[0].title: ""} />
                            {post.tracks[0].title}
                        </div>
                    {/if}
                {/if}
            {/if}
        {/each}
    </div>
</div>
{:catch error}
	<p style="color: red">{error.message}</p> <button on:click={load}>โหลดใหม่</button>
{/await}

<!-- <div class="snowflakes" aria-hidden="true">
    <div class="snowflake">
      <div class="inner">❅</div>
    </div>
    <div class="snowflake">
      <div class="inner">❅</div>
    </div>
    <div class="snowflake">
      <div class="inner">❅</div>
    </div>
    <div class="snowflake">
      <div class="inner">❅</div>
    </div>
    <div class="snowflake">
      <div class="inner">❅</div>
    </div>
    <div class="snowflake">
      <div class="inner">❅</div>
    </div>
    <div class="snowflake">
      <div class="inner">❅</div>
    </div>
    <div class="snowflake">
      <div class="inner">❅</div>
    </div>
    <div class="snowflake">
      <div class="inner">❅</div>
    </div>
    <div class="snowflake">
      <div class="inner">❅</div>
    </div>
    <div class="snowflake">
      <div class="inner">❅</div>
    </div>
    <div class="snowflake">
      <div class="inner">❅</div>
    </div>
  </div>

<style>
    /* customizable snowflake styling */
    .snowflake {
      color: #fff;
      font-size: 1em;
      font-family: Arial, sans-serif;
      text-shadow: 0 0 5px #000;
    }
     
    .snowflake,.snowflake .inner{animation-iteration-count:infinite;animation-play-state:running}@keyframes snowflakes-fall{0%{transform:translateY(0)}100%{transform:translateY(110vh)}}@keyframes snowflakes-shake{0%,100%{transform:translateX(0)}50%{transform:translateX(80px)}}.snowflake{position:fixed;top:-10%;z-index:9999;-webkit-user-select:none;user-select:none;cursor:default;animation-name:snowflakes-shake;animation-duration:3s;animation-timing-function:ease-in-out}.snowflake .inner{animation-duration:10s;animation-name:snowflakes-fall;animation-timing-function:linear}.snowflake:nth-of-type(0){left:1%;animation-delay:0s}.snowflake:nth-of-type(0) .inner{animation-delay:0s}.snowflake:first-of-type{left:10%;animation-delay:1s}.snowflake:first-of-type .inner,.snowflake:nth-of-type(8) .inner{animation-delay:1s}.snowflake:nth-of-type(2){left:20%;animation-delay:.5s}.snowflake:nth-of-type(2) .inner,.snowflake:nth-of-type(6) .inner{animation-delay:6s}.snowflake:nth-of-type(3){left:30%;animation-delay:2s}.snowflake:nth-of-type(11) .inner,.snowflake:nth-of-type(3) .inner{animation-delay:4s}.snowflake:nth-of-type(4){left:40%;animation-delay:2s}.snowflake:nth-of-type(10) .inner,.snowflake:nth-of-type(4) .inner{animation-delay:2s}.snowflake:nth-of-type(5){left:50%;animation-delay:3s}.snowflake:nth-of-type(5) .inner{animation-delay:8s}.snowflake:nth-of-type(6){left:60%;animation-delay:2s}.snowflake:nth-of-type(7){left:70%;animation-delay:1s}.snowflake:nth-of-type(7) .inner{animation-delay:2.5s}.snowflake:nth-of-type(8){left:80%;animation-delay:0s}.snowflake:nth-of-type(9){left:90%;animation-delay:1.5s}.snowflake:nth-of-type(9) .inner{animation-delay:3s}.snowflake:nth-of-type(10){left:25%;animation-delay:0s}.snowflake:nth-of-type(11){left:65%;animation-delay:2.5s}
</style> -->
