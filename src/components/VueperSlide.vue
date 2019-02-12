<template>
<div class="vueperslide" :class="{ 'vueperslide--active': $parent.slides.activeUid === _uid, 'vueperslide--previous-slide': isPreviousSlide, 'vueperslide--next-slide': isNextSlide, 'vueperslide--visible': isSlideVisible }" :face="slideFace3d" :style="wrapperStyles"
    :aria-hidden="$parent.slides.activeUid === _uid || isSlideVisible ? 'false' : 'true'">
    <div class="vueperslide__image" v-if="image && $parent.conf.slideImageInside" :style="imageStyles"></div>
    <div class="vueperslide__content-wrapper" v-show="!$parent.conf.slideContentOutside && (title || hasTitleSlotData || content || hasContentSlotData)">
        <div class="vueperslide__title" v-show="title || hasTitleSlotData">
            <div v-show="!$parent.conf.slideContentOutside && !title">
                <slot name="slideTitle"></slot>
            </div>
            <div v-if="title" v-html="title"></div>
        </div>
        <div class="vueperslide__content" v-if="content || hasContentSlotData">
            <div v-show="!$parent.conf.slideContentOutside && !content">
                <slot name="slideContent"></slot>
            </div>
            <div v-if="content" v-html="content"></div>
        </div>
    </div>
</div>
</template>

<script>
export default {
  props: {
    clone: {
      type: Number,
      default: null
    },
    image: {
      type: String,
      default: ''
    },
    title: {
      type: String,
      default: ''
    },
    content: {
      type: String,
      default: ''
    }
  },
  data: () => ({
    index: 0
  }),
  created () {
    // vueperslide component has this useful attributes:
    // _uid, image, title, titleSlot, content, contentSlot, clone.
    this.index = this.$parent.addSlide(this)
  },
  // When removing a slide programmatically, remove it from the config so vueperslides
  // component is aware of the change.
  destroyed () {
    if (this.clone === null) this.$parent.removeSlide(this._uid)
  },
  computed: {
    wrapperStyles () {
      return {
        ...(!this.$parent.conf.slideImageInside && this.image && { backgroundImage: `url(${this.image})` }),
        ...(this.$parent.conf.visibleSlides > 1 && { width: 100 / this.$parent.conf.visibleSlides + '%' }),
        ...(this.$parent.conf.visibleSlides > 1 && this.$parent.conf.fade && { left: ((this.slideIndex % this.$parent.conf.visibleSlides) / this.$parent.conf.visibleSlides) * 100 + '%' })
      }
    },
    imageStyles () {
      return { ...(this.$parent.conf.slideImageInside && this.image && { backgroundImage: `url(${this.image})` }) }
    },
    hasTitleSlotData () {
      const { slideTitle } = this.$slots
      return slideTitle !== undefined
    },
    hasContentSlotData () {
      const { slideContent } = this.$slots
      return slideContent !== undefined
    },
    slideFace3d () {
      if (!this.$parent.conf['3d']) return false
      const faces = ['front', 'right', 'back', 'left']
      const slidesCount = this.$parent.slides.list.length
      const prevSlideIndex = (this.$parent.slides.current - 1 + slidesCount) % slidesCount
      const nextSlideIndex = (this.$parent.slides.current + 1) % slidesCount

      // Index starts at 1 so this.index-1.
      if (this.index - 1 === prevSlideIndex) return faces[(4 + this.$parent.slides.current - 1) % 4]
      else if (this.index - 1 === nextSlideIndex) return faces[(this.$parent.slides.current + 1) % 4]

      return faces[(this.index - 1) % 4]
    },
    isPreviousSlide () {
      if (!this.$parent.conf['3d']) return false
      const slidesCount = this.$parent.slides.list.length
      const prevSlideIndex = (this.$parent.slides.current - 1 + slidesCount) % slidesCount
      return this._uid === this.$parent.slides.list[prevSlideIndex]._uid
    },
    isNextSlide () {
      if (!this.$parent.conf['3d']) return false
      const slidesCount = this.$parent.slides.list.length
      const nextSlideIndex = (this.$parent.slides.current + 1) % slidesCount
      return this._uid === this.$parent.slides.list[nextSlideIndex]._uid
    },
    isSlideVisible () {
      const activeSlideUid = this.$parent.slides.activeUid
      const activeSlideIndex = this.slidesList.indexOf(activeSlideUid)
      const visibleSlidesCount = this.$parent.conf.visibleSlides

      return this.slideIndex >= activeSlideIndex && this.slideIndex < activeSlideIndex + visibleSlidesCount
    },
    slidesList () {
      return this.$parent.slides.list.map(slide => slide._uid)
    },
    slideIndex () {
      return this.slidesList.indexOf(this._uid)
    }
  }
}
</script>
