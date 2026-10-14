<div id="banner"></div>

<style lang='scss'>
    #banner {
        height: var(--banner-height);
        box-shadow: inset 0 0 0 1000px rgba(0, 0, 0, .25), 0 2px 1rem black;
        background-repeat: no-repeat;
        background-size: cover;
        background-image: url("$lib/images/banner.jpg")
    }

    :global(body.page-hide-banner #banner) {
        display: none !important;
    }

    img {
        height: 2rem;
        margin-left: 1rem;
        @media (max-width: 900px) { height: 1.5rem; }
    }
</style>