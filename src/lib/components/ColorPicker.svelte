<script lang="ts">
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';

    let color = { r: 255, g: 0, b: 0 };
    let hue = 0;
    let dragging = false;
    let pickerX = 100;
    let pickerY = 100;

    let canvas: HTMLCanvasElement | null = null;

    let colorModel: 'RGB' | 'HSB' | 'LAB' = 'RGB';

    let hashColor = '#ff0000';

    const startDrag = (event: MouseEvent) => {
        dragging = true;
        movePicker(event);
    }

    const stopDrag = () => {
        dragging = false;
    }

    const movePicker = (event: MouseEvent) => {
        if (dragging && event.target instanceof HTMLElement) {
            pickerX = Math.min(Math.max(0, event.clientX), 255);
            pickerY = Math.min(Math.max(0, event.clientY), 255);
            const imageData = canvas?.getContext('2d')?.getImageData(pickerX, pickerY, 1, 1).data;
            if (imageData) {
                color.r = imageData[0];
                color.g = imageData[1];
                color.b = imageData[2];
                updateHashColor();
            }
        }
    }

    const updateHashColor = () => {
        hashColor = `#${((1 << 24) + (color.r << 16) + (color.g << 8) + color.b).toString(16).slice(1).toUpperCase()}`;
    }

    const updateOther = (input: 'hash' | 'hue' | 'rgb') => {
        if (input === 'hash') {
        } else if (input === 'hue') {
            color.r = Math.floor(255 * Math.abs(Math.sin(hue / 60)));
            color.g = Math.floor(255 * Math.abs(Math.sin((hue + 120) / 60)));
            color.b = Math.floor(255 * Math.abs(Math.sin((hue + 240) / 60)));
        } else if (input === 'rgb') {
            const r = color.r / 255;
            const g = color.g / 255;
            const b = color.b / 255;
            const max = Math.max(r, g, b);
            const min = Math.min(r, g, b);
            let h: number = 0;
            let s: number = 0;
            let l: number = 0;
            l = (max + min) / 2;
            if (max === min) {
                h = s = 0;
            } else {
                const d = max - min;
                s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
                switch (max) {
                    case r:
                        h = (g - b) / d + (g < b ? 6 : 0);
                        break;
                    case g:
                        h = (b - r) / d + 2;
                        break;
                    case b:
                        h = (r - g) / d + 4;
                        break;
                }
                h /= 6;
            }
            hue = Math.round(h * 360);
            pickerX = Math.round(hue / 360 * 255);
            pickerY = Math.round(s * 255);
        } else {
        }
    }

    const handleHashInput = (event: Event) => {
        if (!(event.target instanceof HTMLInputElement)) return;
        const hex = event.target.value;
        if (/^#[0-9A-F]{6}$/i.test(hex)) {
            color.r = parseInt(hex.slice(1, 3), 16);
            color.g = parseInt(hex.slice(3, 5), 16);
            color.b = parseInt(hex.slice(5, 7), 16);
        }
        updateHashColor();
    }

    $: if (canvas) {
        const ctx = canvas.getContext('2d');
        const width = canvas.width;
        const height = canvas.height;
        if (ctx) {
            const gradientX = ctx.createLinearGradient(0, 0, width, 0);
            gradientX.addColorStop(0, 'white');
            gradientX.addColorStop(1, `hsl(${hue}, 100%, 50%)`);
            ctx.fillStyle = gradientX;
            ctx.fillRect(0, 0, width, height);

            const gradientY = ctx.createLinearGradient(0, 0, 0, height);
            gradientY.addColorStop(0, 'rgba(0, 0, 0, 0)');
            gradientY.addColorStop(1, 'black');
            ctx.fillStyle = gradientY;
            ctx.fillRect(0, 0, width, height);
        }
    }

</script>

<div
    role="button"
    tabindex="0"
    class="relative"
    on:mousedown={startDrag}
    on:mousemove={movePicker}
    on:mouseup={stopDrag}
>
    <canvas
        bind:this={canvas}
        class="w-[255px] h-[255px] border border-gray-300"
    >
    </canvas>
    <div
        class="w-[20px] h-[20px] border-2 border-slate-900 rounded-full absolute"
        style="
            background: rgb({color.r}, {color.g}, {color.b});
            left:{pickerX}px;
            top:{pickerY}px;
            transform:translate(-50%, -50%);
        "
    ></div>
</div>

<select bind:value={colorModel}>
    <option value="RGB">RGB</option>
    <option value="HSB">HSB</option>
    <option value="LAB">LAB</option>
</select>

<input type="range" min="0" max="255" bind:value={hue} on:input={() => updateOther('hue')}>

{#if colorModel === 'RGB'}
    <div>
        <p>R</p><input type="range" min="0" max="255" bind:value={color.r} on:input={() => updateOther('rgb')}>
        <p>G</p><input type="range" min="0" max="255" bind:value={color.g} on:input={() => updateOther('rgb')}>
        <p>B</p><input type="range" min="0" max="255" bind:value={color.b} on:input={() => updateOther('rgb')}>
    </div>
{/if}

<div>
    <p>Hex:</p><input type="text" bind:value={hashColor} on:input={handleHashInput} maxlength="7">
</div>

<p>現在の色：rgb({color.r}, {color.g}, {color.b}) - {hashColor}</p>