<template>
    <a :href="link" target="_blank" class="grid-item" :class="{ active }">
        <img :src="image" alt="" class="banner-image" />
        <div class="grid-item-overlay">
            <h3>{{ title }}</h3>
            <div class="platforms">
                <span v-for="(icon, i) in platformIcons" :key="platforms[i]">
                    <img v-if="icon" :src="icon" :alt="platforms[i]" class="platform-icon" />
                </span>
            </div>
            <h4>{{ maker }}</h4>
        </div>
    </a>
</template>

<script>
const iconModules = import.meta.glob('/src/assets/PlatformIcons/*.svg', { eager: true });

export default {
    name: "GridItem",
    props: {
        title: String,
        image: String,
        maker: String,
        platforms: Array,
        link: String,
        active: {
            type: Boolean,
            default: false
        }
    },
    computed: {
        platformIcons() {
            const icons = this.platforms.map(platform => {
                const fileName = `${platform}.svg`;
                const lowerFileName = `${platform.toLowerCase()}.svg`;
                const iconPath = iconModules[`/src/assets/PlatformIcons/${fileName}`]
                    || iconModules[`/src/assets/PlatformIcons/${lowerFileName}`];
                return iconPath ? iconPath.default : null;
            });
            console.log('platformIcons:', icons);
            return icons;
        }
    }
}
</script>

<style scoped>
.grid-item {
    position: relative;
    overflow: hidden;
    cursor: pointer;
    width: 100%;
    aspect-ratio: 1024 / 400;
    display: block;
}

.grid-item .banner-image {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    transition: filter 0.3s;
    z-index: 1;
}





.grid-item-overlay {
    position: absolute;
    inset: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    opacity: 0;
    transition: opacity 0.3s;
    z-index: 2;
    pointer-events: none;
}





.grid-item h3 {
    color: #fff;
    margin: 0;
    font-size: 1.5em;
    text-shadow: 0 2px 8px rgba(0, 0, 0, 0.7);
    pointer-events: none;
    text-transform: uppercase;
    /* All caps */
}

.grid-item h4 {
    color: #fff;
    font-size: 1em;
    text-shadow: 0 2px 8px rgba(0, 0, 0, 0.7);
    pointer-events: none;
    margin: 0.3em 0 0 0;
    position: absolute;
    left: 0;
    right: 0;
    bottom: 20px;
    /* 20px from bottom */
    text-align: center;
}

.platforms {
    display: flex;
    flex-direction: column;
    /* Stack vertically */
    gap: 6px;
    pointer-events: none;
    position: absolute;
    right: 20px;
    top: 50%;
    transform: translateY(-50%);
    align-items: flex-end;
    margin-top: 0;
}

.platforms span {
    background: none;
    padding: 0;
}

.platform-icon {
    width: 28px !important;
    height: 28px !important;
    object-fit: contain;
    display: block;
    background: transparent;
    border-radius: 6px;
    padding: 2px;
}

@media (min-width: 601px) {
    .grid-item:hover .grid-item-overlay {
        opacity: 1;
    }

    .grid-item:hover .banner-image {
        filter: brightness(0.25) contrast(1.2);
    }
}

@media (max-width: 600px) {
    .grid-item.active .banner-image {
        filter: brightness(0.25) contrast(1.2);
    }

    .grid-item.active .grid-item-overlay {
    opacity: 1;
    pointer-events: auto;
}
}
</style>