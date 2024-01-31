<template>
    <div class="flex items-center justify-center h-screen">
        <div ref="sertifcont" class="relative w-full max-w-screen-md border">
            <canvas id="canvas">
                Hello World
            </canvas>
        </div>
    </div>
    <div class="fixed bottom-0 w-full flex justify-center p-6 gap-4">
        <UButton @click="downloadCanvas">Download Certificate</UButton>
        <UButton v-if="isDownloaded" @click="resetAll" color="red">New Certificate</UButton>
    </div>
</template>

<script setup lang="ts">
import jsPDF from 'jspdf'

const dataInput = [{
    name: 'name',
    placeholder: 'Your name...',
    x: 466,
    y: 270
}]
const dataModel = ref<string[]>([])
const isDownloaded = ref(false)

const resetAll = () => {
    // reload page
    window.location.reload()
}

const sertifcont = ref<HTMLDivElement>()
const makeInput = (input: { name: string, placeholder: string, x: number, y: number }) => {
    const inputEl = document.createElement('input')
    inputEl.type = 'text'
    inputEl.placeholder = input.placeholder
    // inputEl.classList.add('bg-white', 'bg-opacity-50', 'absolute', 'text-center', 'focus:outline-none', 'p-1', 'w-2/6')
    inputEl.style.top = `${input.y}px`
    inputEl.style.left = `${input.x}px`
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
    img.src = '/certif.jpg'
    // get image width and height
    img.onload = () => {
        const containerProps = sertifcont.value?.getBoundingClientRect()
        const imgHeight = img.height
        const imgWidth = img.width
        const coeficient = (containerProps?.width ?? 0) / imgWidth
        canvas.width = coeficient * imgWidth
        canvas.height = coeficient * imgHeight
        console.log(coeficient * imgWidth, coeficient * imgHeight)
        ctx.drawImage(img, 8, 8, coeficient * imgWidth - 16, coeficient * imgHeight - 16)
    }
    // add input
    dataInput.forEach(input => {
        const inputEl = makeInput(input)
        inputEl.id = `input-${input.name}`
        sertifcont.value?.appendChild(inputEl)
    })
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
        ctx.fillText(dataModel.value[dataInput.findIndex(v => v.name === input.name)] ?? '', input.x, input.y)
    })
    // hide all input element
    const inputs = document.querySelectorAll('input')
    inputs.forEach(input => input.style.display = 'none')
    // download canvas as pdf file make it landscape
    const pdf = new jsPDF('l', 'px', [canvas.width, canvas.height])
    pdf.addImage(canvas.toDataURL('image/png'), 'PNG', 0, 0, canvas.width, canvas.height)
    pdf.save('certificate.pdf')

    // download canvas as png
    // const link = document.createElement('a')
    // link.download = 'certificate.pdf'
    // link.href = canvas.toDataURL()
    // link.click()
}
</script>