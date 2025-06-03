<template>
  <div class="page-wrapper">

    <!-- Background Gradient Layer -->
    <div class="hero-background"></div>

    <!-- Actual Page Content -->
    <div class="hero-gradient-content">
      <header class="header">
        <div class="header-content">
          <div class="spacer"></div>

          <!-- Title with masking -->
          <div class="title-mask-container">
            <!-- High-res image (masked to hide the rectangles) -->
            <img src="./assets/Title-HighRes.png" alt="Title High-Res" class="title-layer"
              style="mask: url(#highres-mask); -webkit-mask: url(#highres-mask);" />

            <!-- Pixelated version (masked to only show rectangles) -->
            <img src="./assets/Title-Pixelated.png" alt="Title Pixelated" class="title-layer pixelated-img"
              style="mask: url(#pixel-mask); -webkit-mask: url(#pixel-mask);" />

            <!-- SVG mask defs in visible DOM tree -->
            <svg class="title-mask-defs" viewBox="0 0 600 150" preserveAspectRatio="none">
              <defs>
                <!-- Pixelated mask -->
                <mask id="pixel-mask" maskUnits="userSpaceOnUse">
                  <rect width="800" height="200" fill="black" />
                  <rect v-for="(rect, i) in currentRects" :key="'pix-' + i" :x="rect.x" :y="rect.y" :width="rect.width"
                    :height="rect.height" fill="white" />
                </mask>

                <!-- High-res mask (inverse) -->
                <mask id="highres-mask" maskUnits="userSpaceOnUse">
                  <rect width="800" height="200" fill="white" />
                  <rect v-for="(rect, i) in currentRects" :key="'high-' + i" :x="rect.x" :y="rect.y" :width="rect.width"
                    :height="rect.height" fill="black" />
                </mask>
              </defs>
            </svg>
          </div>

          <div class="spacer"></div>
        </div>
        <div class="header-bar-container">
          <div class="menu">
            <a class="menu-item" href="#" style="--color: #000">CLASS OF '25</a>
            <a class="menu-item" href="#" style="--color: #999">CLASS OF '26</a>
            <a class="menu-item" href="#" style="--color: #999">CLASS OF '27</a>
          </div>

          <div class="header-text">
            <p>
              Upscaling Gameplay is een vak voor eerstejaars Game Design studenten aan de HKU.
              In deze module maken ze voor het eerst een geheel eigen game: van eerste idee tot gepubliceerd
              eindresultaat.
              Hier vind je hun creaties: kleurrijk, verrassend, soms absurd, maar allemaal met aandacht gemaakt.
              Klik, speel, ontdek en laat je verrassen door wat onze eerstejaars hebben neergezet!
            </p>
          </div>
        </div>
      </header>

      <main>
        <div class="grid-wrapper">
          <div class="grid-spacer"></div>
          <div class="grid-container">
            <GridItem
              v-for="(item, index) in items"
              :key="index"
              :title="item.title"
              :maker="item.maker"
              :image="item.image"
              :platforms="item.platforms"
              :link="item.link"
              :active="index === activeIndex"
            />
          </div>
          <!-- Add extra scroll space after the grid -->
          <div class="grid-bottom-spacer"></div>
          <div class="grid-spacer"></div>
        </div>
      </main>
    </div>
  </div>

</template>

<script>
import GridItem from './components/GridItem.vue'

export default {
  components: { GridItem },
  data() {
    return {
      currentRects: [
        { x: 50, y: 30, width: 80, height: 40 },
        { x: 200, y: 50, width: 60, height: 60 },
        { x: 300, y: 20, width: 100, height: 80 },
        { x: 450, y: 60, width: 70, height: 40 }
      ],
      animationKeyframes: [
        [{ x: 50, y: 30, width: 80, height: 40 },
        { x: 200, y: 50, width: 60, height: 60 },
        { x: 300, y: 20, width: 100, height: 80 },
        { x: 450, y: 60, width: 70, height: 40 }],
        [{ x: 80, y: 20, width: 70, height: 50 },
        { x: 250, y: 70, width: 60, height: 40 },
        { x: 350, y: 40, width: 120, height: 90 },
        { x: 420, y: 30, width: 60, height: 60 }],
        [{ x: 60, y: 50, width: 90, height: 30 },
        { x: 180, y: 60, width: 80, height: 80 },
        { x: 280, y: 10, width: 90, height: 70 },
        { x: 500, y: 50, width: 80, height: 50 }],
        [{ x: 40, y: 10, width: 60, height: 40 },
        { x: 210, y: 30, width: 70, height: 70 },
        { x: 330, y: 50, width: 90, height: 60 },
        { x: 460, y: 70, width: 50, height: 30 }]
      ],
      currentFrame: 0,
      animationDuration: 2000,
      easingFn: t => t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t,

      // ✅ This is the part you lost:
      items: [],
      activeIndex: 0
    }
  },
  async mounted() {
    this.startAnimationLoop()
    this.loadGames()
    window.addEventListener('scroll', this.updateActiveItem)
    window.addEventListener('resize', this.updateActiveItem)
    this.$nextTick(this.updateActiveItem)
  },
  beforeUnmount() {
    window.removeEventListener('scroll', this.updateActiveItem)
    window.removeEventListener('resize', this.updateActiveItem)
  },
  methods: {
    startAnimationLoop() {
      const loop = () => {
        const nextFrame = (this.currentFrame + 1) % this.animationKeyframes.length
        this.animateBetweenFrames(this.animationKeyframes[this.currentFrame], this.animationKeyframes[nextFrame], () => {
          this.currentFrame = nextFrame
          loop()
        })
      }
      loop()
    },

    animateBetweenFrames(fromRects, toRects, onComplete) {
      const startTime = performance.now()

      const step = (now) => {
        const elapsed = now - startTime
        const t = Math.min(elapsed / this.animationDuration, 1)
        const easedT = this.easingFn(t)

        this.currentRects = fromRects.map((from, i) => {
          const to = toRects[i]
          return {
            x: from.x + (to.x - from.x) * easedT,
            y: from.y + (to.y - from.y) * easedT,
            width: from.width + (to.width - from.width) * easedT,
            height: from.height + (to.height - from.height) * easedT
          }
        })

        if (t < 1) {
          requestAnimationFrame(step)
        } else {
          onComplete()
        }
      }

      requestAnimationFrame(step)
    },

    async loadGames() {
      // Dynamically import all JSON files in games-25
      const jsonModules = import.meta.glob('../src/games-25/*.json')
      const imageModules = import.meta.glob('../src/games-25/*.{png,jpg,jpeg}')

      const items = []

      for (const path in jsonModules) {
        const json = await jsonModules[path]()
        // Extract base name (without extension)
        const baseName = path.split('/').pop().replace('.json', '')
        // Find matching image
        let imagePath = null
        for (const imgPath in imageModules) {
          if (imgPath.includes(baseName)) {
            const imgModule = await imageModules[imgPath]()
            imagePath = imgModule.default // This is the resolved URL
            break
          }
        }
        if (imagePath) {
          items.push({
            title: json.title,
            platforms: json.platforms,
            maker: json.maker,
            link: json.link,
            image: imagePath
          })
        }
      }

      // Shuffle the items array
      for (let i = items.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1))
        ;[items[i], items[j]] = [items[j], items[i]]
      }

      this.items = items
    },

    updateActiveItem() {
      // Only run this logic on mobile (≤600px)
      if (window.innerWidth > 600) {
        this.activeIndex = -1 // No item is "active" on desktop
        return
      }
      this.$nextTick(() => {
        const items = Array.from(document.querySelectorAll('.grid-item'))
        if (!items.length) return
        const center = window.innerHeight / 2
        let minDist = Infinity
        let active = 0
        items.forEach((el, i) => {
          const rect = el.getBoundingClientRect()
          const itemCenter = rect.top + rect.height / 2
          const dist = Math.abs(center - itemCenter)
          if (dist < minDist) {
            minDist = dist
            active = i
          }
        })
        this.activeIndex = active
      })
    },
  }
}
</script>
