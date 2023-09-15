<script lang="ts">
  import { Input } from "$lib/components/ui/input"
  import * as Select from "$lib/components/ui/select"
  let selectedImage: File | null = null
  let filteredImage: string | null = null
  let filtro: string = ""
  let threads: number = 0
  let parametro: number = 0.0
  function handleFileChange(event: Event) {
    const inputElement = event.target as HTMLInputElement
    const file = inputElement.files && inputElement.files[0]

    if (file) {
      selectedImage = file
    }
  }

  async function applyFilter() {
    if (!selectedImage || !filtro || threads === 0 || parametro === 0.0) {
      // Check if all required parameters are set
      console.error("Please fill in all parameters.")
      return
    }

    const formData = new FormData()
    formData.append("file", selectedImage)
    formData.append("filtro", filtro)
    formData.append("threads", String(threads))
    formData.append("parametro", String(parametro))

    try {
      const response = await fetch("http://localhost:5000/upload", {
        method: "POST",
        body: formData
      })

      if (response.ok) {
        const result = await response.json()
        // Assuming the result contains the filtered image URL
        filteredImage = result.filteredImageUrl
      } else {
        console.error("Failed to upload and process the image.")
      }
    } catch (error) {
      console.error("An error occurred while processing the request:", error)
    }
  }
</script>

<main class="flex pt-4 px-4 justify-center gap-8">
  <div class="flex flex-col gap-3 w-1/3">
    <div class="my-auto">
      <h1>Imagen sin filtro:</h1>

      <div class="border-2 border-black relative {!selectedImage && 'pb-[100%]'} h-fit">
        {#if selectedImage}
          <!-- svelte-ignore a11y-img-redundant-alt -->
          <img src={URL.createObjectURL(selectedImage)} alt="Selected Image" />
        {/if}
      </div>
    </div>
  </div>
  <div class="flex flex-col gap-3 w-1/3">
    <div class="my-auto">
      <h1>Imagen con filtro:</h1>

      <div class="border-2 border-black relative {!filteredImage && 'pb-[100%]'} h-fit">
        {#if filteredImage}
          <!-- svelte-ignore a11y-img-redundant-alt -->
          <img src={filteredImage} alt="Filtered Image" class="border-2 border-black" />
        {/if}
      </div>
    </div>
  </div>
</main>

<div class="mt-8 justify-center flex gap-4">
  <div class="grid w-full max-w-sm items-center gap-1.5">
    <h2>Picture</h2>
    <Input id="picture" type="file" class="hover:cursor-pointer" accept="image/png, image/jpeg, image/webp" on:change={handleFileChange} />
  </div>
  <div class="flex flex-col justify-end mb-1">
    <label for="threads">Threads:</label>
    <Select.Root>
      <Select.Trigger class="w-[180px]">
        <Select.Value placeholder="Threads" />
      </Select.Trigger>
      <Select.Content>
        {#each [...Array(12)].map((_, i) => i + 1) as threadNumber}
          <Select.Item value={threadNumber}>{threadNumber}</Select.Item>
        {/each}
      </Select.Content>
    </Select.Root>
  </div>
  <div class="flex flex-col justify-end mb-1">
    <label for="parametro">Parametro:</label>
    <input type="number" id="parametro" class="rounded-md border border-black" bind:value={parametro} />
  </div>
  <div class="flex flex-col justify-end mb-1">
    <label for="filtro">Filtro:</label>
    <Select.Root>
      <Select.Trigger class="w-[180px]">
        <Select.Value placeholder="Filtro" />
      </Select.Trigger>
      <Select.Content>
        {#each ["merge", "shades", "brightness", "contrast", "plain", "boxblur", "black&white"] as filterOption}
          <Select.Item value={filterOption}>{filterOption}</Select.Item>
        {/each}
      </Select.Content>
    </Select.Root>
  </div>
  <div class="flex flex-col justify-end mb-1">
    <button class="px-2 py-1 rounded-md border h-fit border-black hover:bg-slate-100" on:click={applyFilter}>Aplicar filtro</button>
  </div>
</div>
