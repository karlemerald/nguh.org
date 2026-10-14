<script lang="ts">
    import Stripe from "$lib/components/Stripe.svelte";
    import Dialog from "$lib/components/dialog/Dialog.svelte";
    import {
        Configuration,
        DownloadURL, type EventList,
        PronounSetting,
        type TributeCharacterSelectOptions
    } from '$lib/js/hgs.svelte';
    import SingleFileDialog from "$lib/components/dialog/SingleFileDialog.svelte";
    import MultiFileDialog from "$lib/components/dialog/MultiFileDialog.svelte";
    import Changelog from "$lib/components/hgs/Changelog.svelte";
    import SettingsDialog from '$lib/components/hgs/SettingsDialog.svelte';
    import {Err} from '$lib/js/dialog.svelte';
    import Checkbox from '$lib/components/Checkbox.svelte';
    interface Props {
        tributes: TributeCharacterSelectOptions[]
        start_game: () => void
        event_list: EventList,
    }

    let {tributes = $bindable(), event_list = $bindable(), start_game}: Props = $props();
    let get_image_dialog: SingleFileDialog
    let load_characters_dialog: SingleFileDialog
    let upload_images_dialog: MultiFileDialog
    let settings_dialog: SettingsDialog
    let crop_dialog: Dialog
    let editing_tribute: TributeCharacterSelectOptions | undefined = $state(undefined)
    let draft_crop_x = $state(50)
    let draft_crop_y = $state(50)
    let draft_crop_zoom = $state(1)

    function EditCrop(tribute: TributeCharacterSelectOptions) {
        editing_tribute = tribute
        draft_crop_x = tribute.crop_x ?? 50
        draft_crop_y = tribute.crop_y ?? 50
        draft_crop_zoom = tribute.crop_zoom ?? 1
        crop_dialog.open().ignore_cancellation()
    }

    function ApplyCrop() {
        if (!editing_tribute) return
        editing_tribute.crop_x = draft_crop_x
        editing_tribute.crop_y = draft_crop_y
        editing_tribute.crop_zoom = draft_crop_zoom
        crop_dialog.resolve()
    }

    /** Prompt the user for an image src to use for an <img>. */
    function GetImage(tribute: TributeCharacterSelectOptions) {
        get_image_dialog.open().and(res => { tribute.image_url = res.url })
    }

    /** Load characters from JSON. */
    async function LoadCharacters() {
        load_characters_dialog.open().and(async res => {
            try {
                tributes = await Configuration.LoadCharacters(res.data as object);
            } catch (e: any) {
                Err(e)
            }
        })
    }

    /** Save characters to JSON. */
    async function SaveCharacters() {
        try {
            const chars = await Configuration.SaveCharacters(tributes);
            DownloadURL('characters.json', URL.createObjectURL(new Blob([JSON.stringify(chars, null, 4)], {type: 'application/json'})))
        } catch (e: any) {
            Err(e)
        }
    }

    /** Load several images from disk. */
    async function UploadImages() {
        upload_images_dialog.open().and(res => {
            for (const f of res) {
                const file = f.data as File
                if (!file.type.startsWith('image')) continue
                tributes.push({
                    name: file.name.slice(0, file.name.lastIndexOf(".")),
                    image_url: URL.createObjectURL(file),
                    pronoun_option: PronounSetting.Common
                })
            }
        })
    }
</script>

<SingleFileDialog
    bind:this={get_image_dialog}
    title='Select an image'
    preserve_extern_urls={true}
    description={'You can input a local file or a public url.\nThe image can be in any format your browser supports.'}
    type='raw'
/>

<SingleFileDialog
    bind:this={load_characters_dialog}
    title='Load Game Setup'
    description={'Click below to upload a save.\nThis will override the current setup. Unsaved changes will be lost!'}
    type='json'
/>

<MultiFileDialog
    bind:this={upload_images_dialog}
    title='Upload Images'
    preserve_extern_urls={true}
    description='Click below to select multiple files to add them as tributes.'
    type='raw'
/>

<Dialog bind:this={crop_dialog} title="Edit portrait crop">
    {#snippet content()}
        <div class="crop-editor">
            <div class="crop-preview">
                <img
                    src={editing_tribute?.image_url ?? ''}
                    alt="Portrait crop preview"
                    style="object-fit: contain; translate: {50 - draft_crop_x}% {50 - draft_crop_y}%; transform: scale({draft_crop_zoom})"
                >
            </div>
            <label>
                Horizontal position: {draft_crop_x}%
                <input type="range" min="-100" max="200" bind:value={draft_crop_x}>
            </label>
            <label>
                Vertical position: {draft_crop_y}%
                <input type="range" min="-100" max="200" bind:value={draft_crop_y}>
            </label>
            <label>
                Zoom: {draft_crop_zoom.toFixed(2)}×
                <input type="range" min="0.5" max="5" step="0.01" bind:value={draft_crop_zoom}>
            </label>
        </div>

        <div class="crop-centering-controls">
            <button type="button" onclick={() => draft_crop_x = 50}>Center X</button>
            <button type="button" onclick={() => draft_crop_y = 50}>Center Y</button>
            <button
                type="button"
                onclick={() => {
                    draft_crop_x = 50;
                    draft_crop_y = 50;
                }}
            >Center</button>
        </div>
    {/snippet}
    {#snippet controls()}
        <button onclick={ApplyCrop}>Apply</button>
        <button onclick={() => crop_dialog.reject()}>Cancel</button>
    {/snippet}
</Dialog>

<SettingsDialog bind:this={settings_dialog} bind:event_list />

<Stripe>Choose your characters</Stripe>
<section>
    <div class="mb-4"><p>Current Players: {tributes.length}</p></div>

    <button onclick={() => settings_dialog.open()}>Settings</button>
    <button onclick={SaveCharacters}>Save Characters</button>
    <button onclick={LoadCharacters}>Load Characters</button>
    <button onclick={UploadImages}>Upload Images</button>

    <div class="mt-8 flex flex-wrap justify-between gap-y-8">
        {#each tributes as tribute, i}
            <form class="game-character flex flex-row items-end m-0">
                <div class="flex flex-col gap-3">
                    <div class="flex">
                        <button
                            type="button"
                            class="ml-auto danger-button"
                            onclick={() => tributes.splice(i, 1)}
                        >Delete Character</button>
                    </div>

                    <div class="flex">
                        <button
                            class="ml-auto"
                            type="button"
                            onclick={() => tribute.image_url = undefined}
                        >Remove Image</button>
                    </div>

                    <div class="flex">
                        <label>Name:</label>
                        <input
                            name="character"
                            type="text"
                            class="tribute-name"
                            bind:value={tribute.name}
                        >
                    </div>

                    <div class="flex">
                        <label for="district-{i}">District:</label>
                        <input
                            id="district-{i}"
                            name="district"
                            placeholder="Unassigned"
                            type="text"
                            bind:value={tribute.district}
                        >
                    </div>

                    <div
                        class="flex custom-gender-input-wrapper"
                        style="display: {
                            tribute.pronoun_option === PronounSetting.Custom
                            ? 'inherit'
                            : 'none'
                        };"
                    >   <label>Pronouns:</label>
                        <input
                            name="gender_input"
                            placeholder="they/them/their/themself"
                            type="text"
                            bind:value={tribute.custom_pronouns}
                        >
                    </div>

                    <div class="flex"><label>Pronouns:</label>
                        <select
                            name="gender_select"
                            bind:value={tribute.pronoun_option}
                        >   <!--
                                The values used here are already in a lot of save files,
                                so do NOT change these lightly!
                            -->
                            <option value="m">he/him/his/himself</option>
                            <option value="f">she/her/her/herself</option>
                            <option value="c">they/them/their/themself</option>
                            <option value="n">No Pronouns</option>
                            <option value="other">Other</option>
                        </select>
                    </div>
                </div>
                <div id="image-wrapper" class="image-wrapper">
                    <img
                        alt="Click to add an image"
                        class="non-previewable-icon tribute-image"
                        src={tribute.image_url ?? ''}
                        onclick={() => GetImage(tribute)}
                        style="object-fit: contain; translate: {50 - (tribute.crop_x ?? 50)}% {50 - (tribute.crop_y ?? 50)}%; transform: scale({tribute.crop_zoom ?? 1})"
                    >
                </div>
                <button type="button" disabled={!tribute.image_url} onclick={() => EditCrop(tribute)}>
                    Edit crop
                </button>
            </form>
        {/each}

        <div class="relative w-96 h-40 mr-auto mt-auto mb-auto">
            <button
                id="add-character"
                type="button"
                onclick={() => tributes.push({
                    name: "Player " + (tributes.length + 1),
                    pronoun_option: PronounSetting.Common,
                })}
            ></button>
        </div>
    </div>
    <div class="flex justify-center mt-8">
        <button onclick={start_game}>Start Game</button>
    </div>
</section>

<style lang="scss">
    #add-character {
        position: absolute;
        inset: 2rem;

        border: none;
        padding: 0;
        width: 0;

        box-shadow: none;

        &:hover::before { color: white; }

        &::before {
            position: absolute;
            top: -4rem;
            left: 10rem;

            content: '+';

            font-size: 10rem;
            color: var(--accentlight);
            transition: color .5s;
        }
    }
    .game-character {
        input { padding-block: .225rem !important; background-color: white; }

        select, option, input {
            color: black;
            width: 15rem;
            border: none;
        }

        select, option, input, button {
            height: 2rem;
            width: 16.5rem;
        }

        label {
            margin-right: 1rem;
            width: 6rem;
        }
    }
    .crop-editor {
        display: flex;
        flex-direction: column;
        gap: 1rem;
        min-width: min(18rem, 70vw);

        label { display: flex; flex-direction: column; gap: .25rem; }
        input[type="range"] { width: 100%; }
    }

    .crop-preview {
        width: min(18rem, 70vw);
        aspect-ratio: 1;
        overflow: hidden;
        background: black;

        img {
            width: 100%;
            height: 100%;
            object-fit: contain;
            transform-origin: center;
        }
    }
    .crop-centering-controls {
        display: flex;
        flex-wrap: wrap;
        justify-content: center;
        gap: .5rem;
    }
</style>