<script lang="ts">
    interface Props {
        children?: import('svelte').Snippet;
    }

    let { children }: Props = $props();
</script>


<div class="wrapper">
    <div class="content">
        <div>
            {@render children?.()}
        </div>
    </div>
</div>
