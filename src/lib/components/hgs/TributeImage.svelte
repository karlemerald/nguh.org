<script lang="ts">
    import type {Tribute} from "$lib/js/hgs.svelte";

    interface Props {
        tribute: Tribute,
        grey_if_dead?: boolean
    }

    let {tribute, grey_if_dead = false}: Props = $props();
</script>

<div class="{grey_if_dead && tribute.died_in_round !== undefined ? 'tribute-dead-image-wrapper' : 'image-wrapper'} ml-0 mr-0">
    <img
        alt="{tribute.raw_name}"
        class="tribute-image"
        src="{tribute.image_src}"
    >
</div>