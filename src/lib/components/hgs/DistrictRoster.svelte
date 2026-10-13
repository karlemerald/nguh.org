<script lang="ts">
    import type {Tribute} from "$lib/js/hgs.svelte";
    import Message from "$lib/components/hgs/Message.svelte";
    import TributeImage from "$lib/components/hgs/TributeImage.svelte";
    import Stripe from "$lib/components/Stripe.svelte";

    interface Props {
        tributes: Tribute[]
    }

    let {tributes}: Props = $props();

    let districts = $derived.by(() => {
        const groups = new Map<string, Tribute[]>();

        for (const tribute of tributes) {
            const district = tribute.district?.trim() || 'Unassigned';
            const group = groups.get(district);
            if (group) group.push(tribute);
            else groups.set(district, [tribute]);
        }

        return Array.from(groups, ([name, members]) => ({name, members}));
    });
</script>

<div class="district-roster">
    {#each districts as district}
        <Stripe tight>{district.name}</Stripe>
        <section class="district-group">
            <div class="flex flex-wrap justify-center gap-8">
                {#each district.members as tribute}
                    <div class="flex flex-col items-center gap-2">
                        <TributeImage {tribute} />
                        <Message parts={[tribute.name]} />
                    </div>
                {/each}
            </div>
        </section>
    {/each}
</div>

<style lang="scss">
    .district-roster {
        display: flex;
        flex-direction: column;
        gap: 2rem;
    }

    .district-group h3 { text-align: center; }

    :global(.district-roster .stripe) {
        width: calc(100% + 8rem);
        margin-left: -4rem;
    }

    @media (max-width: 900px) {
        :global(.district-roster .stripe) {
            width: calc(100% + 3rem);
            margin-left: -1.5rem;
        }
    }
</style>
