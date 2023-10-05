<script lang="ts">
  import toast from "svelte-french-toast"
  import { Button } from "$lib/components/ui/button"
  import { Input } from "$lib/components/ui/input"
  import * as Select from "$lib/components/ui/select"
  let selectedImage: File | null = null
  let selectedImage2: File | null = null
  let filteredImage: string | null = null
  let filtro: string = ""
  let threads: number = 0
  let parametro: number = 0.0
  let time: string | null = ""
  function changeThreads(value: any) {
    threads = value
  }
  function changeFilter(value: any) {
    filtro = value
  }
  function handleFileChange(event: Event) {
    const inputElement = event.target as HTMLInputElement
    const file = inputElement.files && inputElement.files[0]

    if (file) {
      selectedImage = file
    }
  }
  function handleFileChange2(event: Event) {
    const inputElement = event.target as HTMLInputElement
    const file = inputElement.files && inputElement.files[0]

    if (file) {
      selectedImage2 = file
    }
  }

  async function applyFilter() {
    if (!selectedImage || !filtro || threads === 0 || parametro < 0) {
      // Check if all required parameters are set
      toast.error("Rellena los parametros correctamente", {
        style: "border-radius: 200px; background: #222; color: #fff;",
        position: "bottom-center"
      })
      return
    }

    const formData = new FormData()
    if (filtro === "merge") {
      formData.append("file1", selectedImage)
      formData.append("file2", selectedImage2!)
    } else formData.append("file", selectedImage)
    formData.append("filtro", filtro)
    formData.append("threads", String(threads))
    formData.append("parametro", String(parametro))

    try {
      const response = await fetch(`http://localhost:8000/upload/${filtro === "merge" ? "merge" : ""}`, {
        method: "POST",
        body: formData
      })
      if (response.ok) {
        time = response.headers.get("x-time")
        const blob = await response.blob() // Get the response as a binary blob
        const url = URL.createObjectURL(blob)

        // Assuming the result contains the filtered image URL
        filteredImage = url
      } else {
        toast.error("Error: Checkea los parametros", {
          style: "border-radius: 200px; background: #222; color: #fff;",
          position: "bottom-center"
        })
      }
    } catch (error) {
      toast.error("Error: Checkea los parametros", {
        style: "border-radius: 200px; background: #222; color: #fff;",
        position: "bottom-center"
      })
    }
  }
</script>

<div class="pt-8 justify-center flex gap-4">
  <div class="grid w-full max-w-sm items-center">
    <h2>Picture</h2>
    <div class="flex gap-3">
      <Input id="picture" type="file" class="hover:cursor-pointer" accept="image/png, image/jpeg, image/webp" on:change={handleFileChange} />
      {#if filtro === "merge"}
        <Input id="picture" type="file" class="hover:cursor-pointer" accept="image/png, image/jpeg, image/webp" on:change={handleFileChange2} />
      {/if}
    </div>
  </div>
  <div class="flex flex-col justify-end mb-1">
    <label for="threads">Threads:</label>
    <Select.Root onSelectedChange={(value) => changeThreads(value?.value)}>
      <Select.Trigger class="w-[180px]">
        <Select.Value placeholder="Threads" on:change={(value) => console.log(value)} />
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
    <Input type="number" id="parametro" class="rounded-md" bind:value={parametro} />
  </div>
  <div class="flex flex-col justify-end mb-1">
    <label for="filtro">Filtro:</label>
    <Select.Root
      onSelectedChange={(value) => {
        changeFilter(value?.value)
        filteredImage = null
      }}
    >
      <Select.Trigger class="w-[180px]">
        <Select.Value placeholder="Filtro" />
      </Select.Trigger>
      <Select.Content>
        {#each ["merge", "shades", "edgeDetection", "vintage", "kaleidoscope", "emboss", "canvas", "brightness", "contrast", "plain", "boxBlur", "blackWhite"] as filterOption}
          <Select.Item value={filterOption}>{filterOption}</Select.Item>
        {/each}
      </Select.Content>
    </Select.Root>
  </div>
  <div class="flex flex-col justify-end mb-1">
    <Button class="px-2 py-1 rounded-md border" variant="default" on:click={applyFilter}>Aplicar</Button>
  </div>
</div>
<main class="flex pt-4 px-4 justify-center gap-8">
  <div class="flex flex-col gap-3 w-1/4">
    <div class="my-auto">
      <h1>Imagen sin filtro:</h1>

      <div class="border-2 border-gray-800 relative flex {!selectedImage && 'pb-[100%]'} h-fit">
        {#if selectedImage}
          <!-- svelte-ignore a11y-img-redundant-alt -->
          <img src={selectedImage && URL.createObjectURL(selectedImage)} class={`${filtro === "merge" && "w-1/2"}`} alt="Selected Image" />
          {#if filtro === "merge" && selectedImage2}
            <!-- svelte-ignore a11y-img-redundant-alt -->
            <img src={URL.createObjectURL(selectedImage2)} class="w-1/2 h-auto" alt="Selected Image 2" />
          {/if}
        {/if}
      </div>
    </div>
  </div>
  <div class="flex flex-col gap-3 w-1/4">
    <div class="my-auto">
      <h1>Imagen con filtro:</h1>

      <div class="border-2 border-gray-800 relative {!filteredImage && 'pb-[100%]'} h-fit">
        {#if filteredImage}
          <!-- svelte-ignore a11y-img-redundant-alt -->
          <img src={filteredImage} alt="Filtered Image" class="border-2 border-black" />
        {/if}
      </div>
    </div>
  </div>
</main>
<h1 class="text-center mt-4">{time && time}</h1>
