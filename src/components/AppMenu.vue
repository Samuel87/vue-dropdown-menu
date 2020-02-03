<template>
  <div
    :class="['menu-container', positionClasses, stateClasses]"
    @mouseenter="onMouseenter"
    @mouseleave="onMouseleave"
  >
    <span
      ref="activator"
      aria-haspopup="true"
      :aria-expanded="isActive ? 'true' : 'false'"
      class="menu-activator"
      @click.prevent="onClick"
      @keydown.enter.prevent="toggle"
      @keydown.down.prevent="toggle"
      @keydown.space.prevent="toggle"
    >
      <slot name="activator" v-bind="{ isActive, isVisible }"></slot>
    </span>

    <transition
      :name="transitionName"
      @before-enter="isVisible = true"
      @after-leave="isVisible = false"
    >
      <div v-show="isActive" ref="menu" class="menu" role="menu" :style="menuStyle">
        <span v-if="hasArrow" class="menu-arrow" :style="arrowStyle"></span>
        <slot></slot>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
    props: {
        positionX: {
            type: String,
            default: 'right',
            validator(type) {
                return ['left', 'center', 'right'].indexOf(type) !== -1;
            },
        },
        positionY: {
            type: String,
            default: 'bottom',
            validator(type) {
                return ['top', 'center', 'bottom'].indexOf(type) !== -1;
            },
        },
        // TODO: fix jumpy position change when using transition that uses transforms
        transition: {
            type: [Boolean, String],
            default: 'v-menu',
        },
        origin: {
            type: String,
            default: 'top left',
        },
        minWidth: {
            type: [Number, String],
            default: 120,
        },
        maxHeight: [Number, String],
        maxWidth: [Number, String],
        closeOnClick: Boolean,
        openOnHover: Boolean,
        hasArrow: Boolean,
        fixed: Boolean,
        relative: Boolean,
    },

    data: () => ({
        isActive: false,
        isVisible: false, // if transitioned or is transitioning
        pageWidth: 0,
        pageHeight: 0,
        pageYOffset: 0,
        dimensions: {
            activator: {
                top: 0,
                left: 0,
                bottom: 0,
                right: 0,
                width: 0,
                height: 0,
            },
            menu: {
                top: 0,
                left: 0,
                bottom: 0,
                right: 0,
                width: 0,
                height: 0,
            },
        },
    }),

    computed: {
        positionClasses() {
            const {
                computedPositionY,
                computedPositionX,
                maxHeight,
                fixed,
                relative,
            } = this;
            return [
                !relative && `x-${computedPositionX}`,
                !relative && `y-${computedPositionY}`,
                { 'y-full': fixed },
                { 'position-relative': relative },
                { 'overflow-scroll': maxHeight || fixed },
            ];
        },

        stateClasses() {
            return { 'is-open': this.isActive };
        },

        computedPositionX() {
            if (this.computedOverflow.right) return 'left';
            if (this.computedOverflow.left) return 'right';
            return this.positionX;
        },

        computedPositionY() {
            if (this.computedOverflow.bottom) return 'top';
            if (this.computedOverflow.top) return 'bottom';
            return this.positionY;
        },

        menuStyle() {
            const { maxHeight, maxWidth, minWidth, origin } = this;

            return {
                'max-height': maxHeight ? maxHeight + 'px' : null,
                'max-width': maxWidth ? maxWidth + 'px' : null,
                'min-width': minWidth ? minWidth + 'px' : null,
                'transition-origin': origin,
            };
        },

        arrowStyle() {
            const {
                dimensions: { activator, menu },
                computedPositionX,
                computedPositionY,
            } = this;
            const activatorHalf = activator.width / 2;
            let left = '50%';
            let top = '-10px';

            if (computedPositionX === 'left')
                left = `${menu.width - activatorHalf}px`;
            if (computedPositionX === 'right') left = `${activatorHalf}px`;
            if (computedPositionY === 'top') top = `${menu.height}px`;

            return {
                left,
                top,
                [`border-${this.computedPositionY}`]: '10px solid white',
            };
        },

        transitionName() {
            return this.transition ? this.transition : null;
        },

        computedOverflow() {
            const {
                pageWidth,
                pageHeight,
                pageYOffset,
                dimensions: { menu },
                isVisible,
            } = this;
            const toBottom = pageYOffset + pageHeight;

            return {
                top: isVisible ? pageYOffset > menu.top : false,
                left: isVisible ? 0 > menu.left : false,
                bottom: isVisible ? toBottom < menu.bottom : false,
                right: isVisible ? pageWidth < menu.right : false,
            };
        },
    },

    mounted() {
        document.addEventListener('click', this.handleClickaway, false);
    },

    beforeDestroy() {
        document.removeEventListener('click', this.handleClickaway, false);
    },

    methods: {
        toggle() {
            if (this.isActive) return (this.isActive = false);

            this.updateDimensions();
            requestAnimationFrame(() => {
                this.isActive = true;
            });
        },

        deactivate() {
            this.isActive = false;
        },

        sneakPeek(el, cb) {
            requestAnimationFrame(() => {
                el.style.display = 'block';
                cb(el);
                el.style.display = 'none';
            });
        },

        measureElement(el) {
            const rect = el.getBoundingClientRect();
            return {
                top: Math.round(rect.top),
                left: Math.round(rect.left),
                bottom: Math.round(rect.bottom),
                right: Math.round(rect.right),
                width: Math.round(rect.width),
                height: Math.round(rect.height),
            };
        },

        updateDimensions() {
            const { activator, menu } = this.$refs;
            const dimensions = {};

            this.pageWidth = document.documentElement.clientWidth;
            this.pageHeight = window.innerHeight;
            this.pageYOffset = window.pageYOffset;

            dimensions.activator = this.measureElement(activator);

            this.sneakPeek(menu, (el) => {
                dimensions.menu = this.measureElement(el);

                this.dimensions = dimensions;
            });
        },

        onMouseenter() {
            if (this.openOnHover && !this.isActive) this.toggle();
        },

        onMouseleave() {
            if (this.openOnHover) this.deactivate();
        },

        onClick() {
            if (!this.openOnHover) this.toggle();
        },

        handleClickaway(evt) {
            const { activator, menu } = this.$refs;
            const shouldClose = this.closeOnClick
                ? true
                : !menu.contains(evt.target);

            if (!activator.contains(evt.target) && shouldClose) {
                this.deactivate();
            }
        },
    },
};
</script>

<style lang="scss" scoped>
.menu-container {
    display: inline-flex;
    position: relative;

    &.x-right {
        justify-content: flex-start;
    }
    &.x-center {
        justify-content: center;
    }
    &.x-left {
        justify-content: flex-end;
    }

    &.y-top .menu {
        top: 0;
        transform: translateY(-100%);
    }
    &.y-center .menu {
        top: 50%;
        transform: translateY(-50%);
    }
    &.y-bottom .menu {
        top: 100%;
    }
    &.y-full .menu {
        position: fixed;
        top: 1rem;
        bottom: 1rem;
    }

    &.position-relative .menu {
        position: relative;
        filter: none;
    }

    &.overflow-scroll .menu {
        overflow-y: auto;
        overflow-x: hidden;
    }
}

.menu {
    position: absolute;
    background-color: white;
    filter: drop-shadow(0px 3px 4px rgba(0, 0, 0, 0.25));
    z-index: 1030;
    will-change: transform;
}

.menu-activator {
    cursor: pointer;
}

.menu-arrow {
    position: absolute;
    width: 0;
    height: 0;
    border-left: 10px solid transparent;
    border-right: 10px solid transparent;
    transform: translateX(-50%);
}

.v-menu-enter-active {
    animation: fade 160ms;
}

.v-menu-leave-active {
    animation: fade 90ms reverse;
}

@keyframes fade {
    0% {
        opacity: 0;
    }

    100% {
        opacity: 1;
    }
}
</style>
