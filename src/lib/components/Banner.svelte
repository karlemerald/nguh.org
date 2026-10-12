<script lang="ts">
    // Compute the current season.
    function GetSeason() {
        let month = new Date().getMonth()
        switch (month) {
            // November, December, January, February
            case 10:
            case 11:
            case 0:
            case 1:
                return 'winter'

            // March, April, May
            case 2:
            case 3:
            case 4:
                return 'spring'

            // June, July, August
            case 5:
            case 6:
            case 7:
                return 'summer'

            // September, October
            case 8:
            case 9:
                return 'autumn'
        }

        // Should never be reached.
        return 'spring'
    }
</script>

<div id="banner" class="banner-{GetSeason()}"></div>

<style lang='scss'>
    #banner {
        height: var(--banner-height);
        box-shadow: inset 0 0 0 1000px rgba(0, 0, 0, .25), 0 2px 1rem black;
        background-repeat: no-repeat;
        background-size: cover;
    }

    :global(body.page-hide-banner #banner) {
        display: none !important;
    }

    img {
        height: 2rem;
        margin-left: 1rem;
        @media (max-width: 900px) { height: 1.5rem; }
    }

    .banner-spring { background-image: url("$lib/images/banner_spring.png"); }
    .banner-summer { background-image: url("$lib/images/banner_summer.jpg"); }
    .banner-autumn { background-image: url("$lib/images/banner_autumn.jpg"); }
    .banner-winter { background-image: url("$lib/images/banner_winter.jpg"); }
</style>