<script lang="ts">
    import { toast } from "@zerodevx/svelte-toast";
    import { onMount } from "svelte";

    let platforms = [];
    let versions = [];
    let versionsInfo = [];
    let selectedType = "paper";
    let selectedVersion = "";
    let selectedDownloadUrl = "";
    let isLoading = true;

    onMount(async () => {
        platforms = await getServerJarPlatforms();
        await updateVersions();
        updateSelectedDownloadUrl();
        isLoading = false;
    });

    async function updateVersions() {
        isLoading = true;
        versions = await getServerJarVersions(selectedType);
        selectedVersion = versions[0]?.version || "";
        updateSelectedDownloadUrl();
        versionsInfo = await getServerJarVersionsInfo(selectedType);
        isLoading = false;
    }

    function updateSelectedDownloadUrl() {
        const selectedPlatform = platforms.find(p => p.key === selectedType);
        if (selectedPlatform && selectedVersion) {
            selectedDownloadUrl = `${selectedPlatform.url}/${selectedVersion}/download`;
        } else {
            selectedDownloadUrl = "";
        }
    }

    async function getServerJarPlatforms() {
        const response = await fetch("/api/server-jars");
        const data = await response.json();
        return data.filter(item => item.key !== "spigot" && item.key !== "craftbukkit");
    }

    async function getServerJarVersions(type) {
        const response = await fetch(`/api/server-jars/${type}`);
        return await response.json();
    }

    async function getServerJarVersionsInfo(type) {
        const versions = await getServerJarVersions(type);
        const promises = versions.map(async (version) => {
            const response = await fetch(`/api/server-jars/${type}/${version.version}`);
            return await response.json();
        });

        return Promise.all(promises);
    }

    async function handleTypeChange() {
        selectedDownloadUrl = "";  // Clear the URL immediately
        await updateVersions();
        updateSelectedDownloadUrl();
    }

    async function handleVersionChange() {
        updateSelectedDownloadUrl();
    }

    function downloadSuccess() {
        toast.push('Downloaded successfully!', {
            theme: {
                '--toastColor': 'mintcream',
                '--toastBackground': 'rgba(72,187,120,0.9)',
                '--toastBarBackground': '#2F855A'
            }
        });
    }
</script>

<div class="place-items-center text-center items-start grid grid-cols-1 md:grid-cols-3 lg:grid-cols-3 xl:grid-cols-3 gap-10">
    <div class="flex flex-col">
        <h3 class="font-medium text-white text-[20px] text-left">Type</h3>
        <select bind:value={selectedType} on:change={handleTypeChange} id="type" class="w-[140px] scroll">
            {#each platforms as platform}
                <option value={platform.key} class="scroll-option">{platform.name}</option>
            {/each}
        </select>
    </div>
    <div class="flex flex-col">
        <h3 class="font-medium text-white text-[20px] text-left">Version</h3>
        <select bind:value={selectedVersion} on:change={handleVersionChange} id="underline_select2" class="w-[120px] scroll">
            {#each versions as version}
                <option value={version.version} class="scroll-option">{version.version}</option>
            {/each}
        </select>
    </div>
    <a href={selectedDownloadUrl} aria-label='Download Jar' class="self-end">
        <button class="button h-fit" on:click={downloadSuccess}>Download</button>
    </a>
</div>

