<script lang="ts">
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';

    let colors = writable<string[]>([]);
    let selectedColor = writable<string>('#ffffff');
    let mode = writable<string>('RGB');

    let colorInput: HTMLInputElement | null = null;

    function addColor(color: string) {
        colors.update((currentColors) => [...currentColors, color]);
    }

    function editColor(index: number, newColor: string) {
        colors.update((currentColors) => {
            currentColors[index] = newColor;
            return currentColors;
        });
    }

    function copyToClipboard(color: string) {
        navigator.clipboard.writeText(color);
        alert('Color copied to clipboard: ' + color);
    }

    function setMode(newMode: string) {
        mode.set(newMode);
    }

    onMount(() => {
        addColor('#ff0000');
        addColor('#00ff00');
        addColor('#0000ff');
    });

    import ColorPicker from "$lib/components/ColorPicker.svelte";
</script>

<div class="p-10">
    <ColorPicker />
</div>

<div>
    <h1>Color Picker</h1>

    <div>
        <label for="mode-select">Mode:</label>
        <select
                id="mode-select"
                on:change={(e: Event) => {
                if (e.target instanceof HTMLSelectElement)
                setMode(e.target.value);
            }}
        >
            <option value="RGB">RGB</option>
            <option value="HSB">HSB</option>
            <option value="LAB">LAB</option>
        </select>
    </div>
    <div>
        {#each $colors as color, index}
            <div>
                <input
                        type="color"
                        bind:value={color}
                        on:change={() => editColor(index, color)}
                />
                <button on:click="{() => copyToClipboard(color)}">Copy</button>
            </div>
        {/each}
    </div>
</div>