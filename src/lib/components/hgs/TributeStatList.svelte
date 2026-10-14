<script lang="ts">
    import type {Tribute} from "$lib/js/hgs.svelte";
    import Message from "$lib/components/hgs/Message.svelte";
    import TributeImage from "$lib/components/hgs/TributeImage.svelte";
    import Stripe from "$lib/components/Stripe.svelte";

    interface Props {
        tributes: Tribute[],
        grey_if_dead?: boolean,
        group_by_district?: boolean
    }

    let {tributes, grey_if_dead = false, group_by_district = false}: Props = $props();

    let districts = $derived.by((): {name: string | undefined; members: Tribute[]}[] => {
        if (!group_by_district) return [{name: undefined, members: tributes}];

        const groups = new Map<string, Tribute[]>();
        for (const tribute of tributes) {
            const district = tribute.district?.trim() || "Unassigned";
            const group = groups.get(district);
            if (group) group.push(tribute);
            else groups.set(district, [tribute]);
        }

        return Array.from(groups, ([name, members]) => ({name, members}));
    });
</script>

<div class="tribute-stat-list">
    {#each districts as district}
        {#if district.name}
            <Stripe tight>{district.name}</Stripe>
        {/if}
        <div class="flex flex-wrap justify-center gap-8">
            {#each district.members as tribute}
                <div class="flex flex-col items-center gap-2">
                    <p>
                        {#if tribute.died_in_round !== undefined}
                        {:else}
                            <span class="tribute-winner">⭐Winner⭐</span>
                        {/if}
                    </p>
                    <TributeImage {tribute} {grey_if_dead} />
                    <Message parts={[tribute.name]}/>
                    <p>Kills: {tribute.kills}</p>
                    <p>
                        {#if tribute.died_in_round !== undefined}
                            Died: Round {tribute.died_in_round.index}
                        {:else}
                            <span class="spacer"></span>
                        {/if}
                    </p>
                </div>
            {/each}
        </div>
    {/each}
</div>

<style lang="scss">
    .tribute-stat-list { display: flex; flex-direction: column; gap: 1.5rem; }

    :global(.tribute-stat-list > .stripe) {
        width: calc(100% + 8rem);
        margin-left: -4rem;
    }

    @media (max-width: 900px) {
        :global(.tribute-stat-list > .stripe) {
            width: calc(100% + 3rem);
            margin-left: -1.5rem;
        }
    }

    p { margin: 0; }
    .tribute-winner {
        color: var(--accentlight);
        font-variant: small-caps;
        font-weight: bold;
    }
    .spacer {
        display: block;
        height: 4rem;
    }
</style>
