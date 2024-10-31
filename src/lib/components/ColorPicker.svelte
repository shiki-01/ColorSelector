<script lang="ts">
    const rgbToHsv = (rgb: { r: number, g: number, b: number }) => {
        const r = rgb.r / 255;
        const g = rgb.g / 255;
        const b = rgb.b / 255;

        let max = Math.max(r, g, b), min = Math.min(r, g, b);

        let h: number = 0;
        let s: number = 0;
        let v: number = max;

        let d = max - min;
        s = max === 0 ? 0 : d / max;

        if (max === min) {
            h = 0;
        } else {
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

        return {h: Math.round(h * 360), s: Math.round(s * 100), v: Math.round(v * 100)};
    }

    const hsvToRgb = (hsv: { h: number, s: number, v: number }) => {
        const h = hsv.h / 360;
        const s = hsv.s / 100;
        const v = hsv.v / 100;

        let r: number = 0;
        let g: number = 0;
        let b: number = 0;

        let i = Math.floor(h / 60);
        let f = h / 60 - i;
        let p = v * (1 - s);
        let q = v * (1 - f * s);
        let t = v * (1 - (1 - f) * s);

        switch (i % 6) {
            case 0:
                r = v;
                g = t;
                b = p;
                break;
            case 1:
                r = q;
                g = v;
                b = p;
                break;
            case 2:
                r = p;
                g = v;
                b = t;
                break;
            case 3:
                r = p;
                g = q;
                b = v;
                break;
            case 4:
                r = t;
                g = p;
                b = v;
                break;
            case 5:
                r = v;
                g = p;
                b = q;
                break;
        }

        return {r: Math.round(r * 255), g: Math.round(g * 255), b: Math.round(b * 255)};
    }

    const rgbToCmyk = (rgb: { r: number, g: number, b: number }) => {
        const r = rgb.r;
        const g = rgb.g;
        const b = rgb.b;

        let c = 1 - (r / 255);
        let m = 1 - (g / 255);
        let y = 1 - (b / 255);
        let k = Math.min(c, m, y);

        if (k === 1) {
            return {c: 0, m: 0, y: 0, k: 100};
        }

        c = ((c - k) / (1 - k)) * 100;
        m = ((m - k) / (1 - k)) * 100;
        y = ((y - k) / (1 - k)) * 100;
        k *= 100;

        return {c: Math.round(c), m: Math.round(m), y: Math.round(y), k: Math.round(k)};
    }

    const cmykToRgb = (cmky: { c: number, m: number, y: number, k: number }) => {
        const c = cmky.c / 100;
        const m = cmky.m / 100;
        const y = cmky.y / 100;
        const k = cmky.k / 100;

        let r = (1 - c) * (1 - k);
        let g = (1 - m) * (1 - k);
        let b = (1 - y) * (1 - k);

        return {r: Math.round(r * 255), g: Math.round(g * 255), b: Math.round(b * 255)};
    }

    const rgbToHsl = (rgb: { r: number, g: number, b: number }) => {
        const r = rgb.r / 255;
        const g = rgb.g / 255;
        const b = rgb.b / 255;

        let max = Math.max(r, g, b), min = Math.min(r, g, b);

        let h: number = 0;
        let s: number = 0;
        let l: number = (max + min) / 2;

        let d = max - min;
        s = max === 0 ? 0 : d / (1 - Math.abs(2 * l - 1));

        if (max === min) {
            h = 0;
        } else {
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

        return {h: Math.round(h * 360), s: Math.round(s * 100), l: Math.round(l * 100)};
    }

    const hslToRgb = (hsl: { h: number, s: number, l: number }) => {
        const h = hsl.h / 360;
        const s = hsl.s / 100;
        const l = hsl.l / 100;

        let r: number = 0;
        let g: number = 0;
        let b: number = 0;

        if (s === 0) {
            r = g = b = l;
        } else {
            let hue2rgb = (p: number, q: number, t: number) => {
                if (t < 0) t += 1;
                if (t > 1) t -= 1;
                if (t < 1 / 6) return p + (q - p) * 6 * t;
                if (t < 1 / 2) return q;
                if (t < 2 / 3) return p + (q - p) * (2 / 3 - t) * 6;
                return p;
            }

            let q = l < 0.5 ? l * (1 + s) : l + s - l * s;
            let p = 2 * l - q;

            r = hue2rgb(p, q, h + 1 / 3);
            g = hue2rgb(p, q, h);
            b = hue2rgb(p, q, h - 1 / 3);
        }

        return {r: Math.round(r * 255), g: Math.round(g * 255), b: Math.round(b * 255)};
    }

    const rgbToHex = (rgb: { r: number, g: number, b: number }) => {
        return "#" + ((1 << 24) + (rgb.r << 16) + (rgb.g << 8) + rgb.b).toString(16).slice(1).toUpperCase();
    }

    const updateFromRgb = () => {
        hsv = rgbToHsv(rgb);
        hex = rgbToHex(rgb);
        hsl = rgbToHsl(rgb);
        cmyk = rgbToCmyk(rgb);
    }

    const updateFromHue = () => {
        rgb = hsvToRgb(hsv);
        hex = rgbToHex(rgb);
        hsl = rgbToHsl(rgb);
        cmyk = rgbToCmyk(rgb);
    }

    const updateFromHex = () => {
        rgb.r = parseInt(hex.substring(1, 3), 16);
        rgb.g = parseInt(hex.substring(3, 5), 16);
        rgb.b = parseInt(hex.substring(5, 7), 16);
        hsv = rgbToHsv(rgb);
        hsl = rgbToHsl(rgb);
        cmyk = rgbToCmyk(rgb);
    }

    const updateFromCmyk = () => {
        rgb = cmykToRgb(cmyk);
        hsv = rgbToHsv(rgb);
        hsl = rgbToHsl(rgb);
        hex = rgbToHex(rgb);
    }

    const updateFromHsl = () => {
        rgb = hslToRgb(hsl);
        hsv = rgbToHsv(rgb);
        hex = rgbToHex(rgb);
        cmyk = rgbToCmyk(rgb);
    }

    const updateFromHsv = () => {
        rgb = hsvToRgb(hsv);
        hex = rgbToHex(rgb);
        hsl = rgbToHsl(rgb);
        cmyk = rgbToCmyk(rgb);
    }

    const handleMouseDown = (e: MouseEvent) => {
        const handleMouseMove = (e: MouseEvent) => {
            hsv.s = Math.max(0, Math.min(100, Math.round((e.offsetX))));
            hsv.v = Math.max(0, Math.min(100, Math.round((e.offsetY))));
            updateFromHsv();
        }

        const handleMouseUp = () => {
            window.removeEventListener('mousemove', handleMouseMove);
            window.removeEventListener('mouseup', handleMouseUp);
        }

        window.addEventListener('mousemove', handleMouseMove);
        window.addEventListener('mouseup', handleMouseUp);
    }

    const handleMouseMove = (e: MouseEvent) => {
        if (e.buttons === 1) {
            hsv.s = Math.max(0, Math.min(100, Math.round((e.offsetX))));
            hsv.v = Math.max(0, Math.min(100, Math.round((e.offsetY))));
            updateFromHsv();
        }
    }

    const handleMouseUp = () => {
        window.removeEventListener('mousemove', handleMouseMove);
        window.removeEventListener('mouseup', handleMouseUp);
    }

    let rgb = {r: 66, g: 135, b: 245};
    let hsv = rgbToHsv(rgb);
    let cmyk = rgbToCmyk(rgb);
    let hsl = rgbToHsl(rgb);
    let hex = rgbToHex(rgb);

    let canvas: HTMLCanvasElement | null = null;

    $: if (canvas) {
        const ctx = canvas.getContext('2d');
        if (ctx) {
            const gradientX = ctx.createLinearGradient(0, 0, 255, 0);
            gradientX.addColorStop(0, 'white');
            gradientX.addColorStop(1, `hsl(${hsv.h}, 100%, 50%)`);
            ctx.fillStyle = gradientX;
            ctx.fillRect(0, 0, 255, 255);

            const gradientY = ctx.createLinearGradient(0, 0, 0, 255);
            gradientY.addColorStop(0, 'rgba(0, 0, 0, 0)');
            gradientY.addColorStop(1, 'black');
            ctx.fillStyle = gradientY;
            ctx.fillRect(0, 0, 255, 255);
        }
    }
</script>

<div class="">
    <div
        class="relative"
        role="button"
        tabindex="0"
    >
        <canvas
            width="255"
            height="255"
            bind:this={canvas}
            on:mousedown={handleMouseDown}
            on:mousemove={handleMouseMove}
            on:mouseup={handleMouseUp}
        ></canvas>
        <div
            class="absolute top-0 left-0 w-[20px] h-[20px] border-2 border-white rounded-full"
            style="
            transform: translate({hsv.s * 2.55}%, {hsv.v * 2.55}%);
            background-color: hsl({hsv.h}, 100%, 50%);
        "
        ></div>
    </div>

    <input type="range" min="0" max="360" bind:value={hsv.h} on:input={updateFromHue}/>

    <div class="flex flex-col gap-5">
        <div>
            <p>RGB:</p>
            <div class="flex flex-col">
                <input type="range" bind:value={rgb.r} on:input={updateFromRgb}/>
                <input type="range" bind:value={rgb.g} on:input={updateFromRgb}/>
                <input type="range" bind:value={rgb.b} on:input={updateFromRgb}/>
            </div>
        </div>

        <div>
            <p>HEX:</p>
            <input type="text" bind:value={hex} on:input={updateFromHex}/>
        </div>

        <div>
            <p>CMYK:</p>
            <div class="flex flex-col">
                <input type="range" bind:value={cmyk.c} on:input={updateFromCmyk}/>
                <input type="range" bind:value={cmyk.m} on:input={updateFromCmyk}/>
                <input type="range" bind:value={cmyk.y} on:input={updateFromCmyk}/>
                <input type="range" bind:value={cmyk.k} on:input={updateFromCmyk}/>
            </div>
        </div>
    </div>
</div>
