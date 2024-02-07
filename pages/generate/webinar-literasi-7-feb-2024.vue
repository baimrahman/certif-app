<template>
    <div class="flex items-center justify-center p-8 pt-24 bg-gray-300">
        <div ref="sertifcont" class="relative w-full max-w-screen-lg border">
            <canvas id="canvas" class="shadow-md" />
        </div>
    </div>
    <div class="fixed top-0 w-full flex justify-between items-center p-4 bg-white shadow-md">
        <h1 class="text-xl font-semibold">Certificate Generator</h1>
        <div class="flex gap-4">
            <UButton icon="i-heroicons-arrow-down-tray" v-if="dataModel.length" @click="downloadCanvas" color="blue">
                Download Certificate</UButton>
            <UButton icon="i-heroicons-plus" v-if="isDownloaded" @click="resetAll" color="yellow">New Certificate</UButton>
        </div>
    </div>
</template>

<script setup lang="ts">
useHead({
    title: 'Sertifikat Webinar Literasi | 7 Feb 2024',
    meta: [
        {
            name: 'description',
            content: 'Generate sertifikat webinar literasi'
        }
    ]
})

import jsPDF from 'jspdf'

const dataInput = [{
    name: 'name',
    placeholder: 'Your name...',
    x: 1020,
    y: 660
}]
const dataModel = ref<string[]>([])
const isDownloaded = ref(false)

let coeficient = 0

const resetAll = () => {
    // reload page
    window.location.reload()
}

const sertifcont = ref<HTMLDivElement>()
const makeInput = (input: { name: string, placeholder: string, x: number, y: number }) => {
    console.log(coeficient)
    const inputEl = document.createElement('input')
    inputEl.type = 'text'
    inputEl.placeholder = input.placeholder
    // inputEl.classList.add('bg-white', 'bg-opacity-50', 'absolute', 'text-center', 'focus:outline-none', 'p-1', 'w-2/6')
    inputEl.style.top = `${input.y * coeficient}px`
    inputEl.style.left = `${input.x * coeficient}px`
    inputEl.style.position = 'absolute'
    inputEl.style.backgroundColor = 'white'
    inputEl.style.outline = 'none'
    inputEl.style.textAlign = 'center'
    inputEl.style.padding = '4px'
    inputEl.style.transform = 'translate(-50%, -50%)'
    inputEl.style.width = '33%'
    inputEl.oninput = (e: any) => {
        dataModel.value[dataInput.findIndex(v => v.name === input.name)] = e.target.value
        console.log(dataModel.value)
    }

    return inputEl
}
onMounted(() => {
    const canvas = document.getElementById('canvas') as HTMLCanvasElement;
    const ctx = canvas.getContext('2d')
    if (!ctx || !sertifcont) return
    // add image from assets/img/certif.jpg
    const img = new Image()
    img.src = '/sertifikat-webinar-literasi.jpg'
    // get image width and height
    img.onload = () => {
        const containerProps = sertifcont.value?.getBoundingClientRect()
        const imgHeight = img.height
        const imgWidth = img.width
        coeficient = (containerProps?.width ?? 0) / imgWidth
        canvas.width = coeficient * imgWidth
        canvas.height = coeficient * imgHeight
        console.log(coeficient * imgWidth, coeficient * imgHeight)
        ctx.drawImage(img, 0, 0, coeficient * imgWidth, coeficient * imgHeight)
        // add input
        dataInput.forEach(input => {
            const inputEl = makeInput(input)
            inputEl.id = `input-${input.name}`
            sertifcont.value?.appendChild(inputEl)
            // make input active
            inputEl.focus()
        })
    }

})

const downloadCanvas = () => {
    isDownloaded.value = true
    const canvas = document.getElementById('canvas') as HTMLCanvasElement;
    // add text from dataInput to canvas with position x and y
    const ctx = canvas.getContext('2d')
    if (!ctx) return
    ctx.font = 'bold 32px Arial'
    ctx.fillStyle = 'black'
    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'
    dataInput.forEach(input => {
        ctx.fillText(dataModel.value[dataInput.findIndex(v => v.name === input.name)] ?? '', input.x * coeficient, input.y * coeficient)
    })
    // hide all input element
    const inputs = document.querySelectorAll('input')
    inputs.forEach(input => input.style.display = 'none')
    // download canvas as pdf file make it landscape high resolution
    const pdf = new jsPDF('l', 'mm', [canvas.width, canvas.height])
    pdf.addImage(canvas.toDataURL('image/png'), 'PNG', 0, 0, canvas.width, canvas.height)
    pdf.save(`sertifikat-${new Date().toISOString()}.pdf`)
}
</script>