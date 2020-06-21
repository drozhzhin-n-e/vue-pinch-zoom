<template>
    <div class="pinch-zoom-wrapper" v-bind:style="styleObject">
        <div class="pinch-zoom-content" v-bind:class="{'pz-dragging': isDragging()}" ref="wrapper">
            <slot></slot>
        </div>

        <div class="pz-zoom-button pz-zoom-control-position-bottom" v-if="isControl()" v-bind:class="{'pz-zoom-button-out': isZoomedIn}" v-on:click="toggleZoom()"></div>
    </div>
</template>

<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator';
import { IvyPinch } from '../ivypinch';
import { Interfaces } from '../interfaces';
import { defaultProperties } from '../properties';

@Component
export default class PinchZoom extends Vue {
    @Prop() properties: Interfaces;
    @Prop() transitionDuration: number;
    @Prop() doubleTap: boolean;
    @Prop() doubleTapScale: number;
    @Prop() autoZoomOut: boolean;
    @Prop() limitZoom: number | "original image size";
    @Prop() disabled: boolean;
    @Prop() disablePan: boolean;
    @Prop() overflow: "hidden" | "visible";
    //@Prop() zoomControlScale: number;
    @Prop() disableZoomControl: "disable" | "never" | "auto";
    @Prop() backgroundColor: string;
    @Prop() limitPan: boolean;
    @Prop() minScale: number;
    @Prop() listeners: 'auto' | 'mouse and touch';
    @Prop() wheel: boolean;
    @Prop() autoHeight: boolean;
    @Prop() wheelZoomFactor: number;
    @Prop() draggableImage: boolean;

    _properties: Interfaces;
    ivyPinch: any;
    styleObject:any;
    isZoomedIn: boolean = false;

    get isTouchScreen() {
        var prefixes = ' -webkit- -moz- -o- -ms- '.split(' ');
        var mq = function(query:any) {
            return window.matchMedia(query).matches;
        }

        if (('ontouchstart' in window)) {
            return true;
        }

        // include the 'heartz' as a way to have a non matching MQ to help terminate the join
        // https://git.io/vznFH
        var query = ['(', prefixes.join('touch-enabled),('), 'heartz', ')'].join('');
        return mq(query);
    }

    created() {
        let changedOptions = this.getProperties(this.$props);
        this.applyOptionsDefault(defaultProperties, changedOptions);
        this.setStyles();
    }

    mounted() {
        this.init();
    }

    beforeDestroy() {
      this.ivyPinch.destroy();
    }

    isDragging() {
        if (!this.ivyPinch) {
            return undefined;
        }
        return  this.ivyPinch.isDragging();
    }

    isControl() {
        if (this._properties['disabled']) {
            return false;
        }

        if (!this._properties) { return undefined; }

        if (this._properties['disableZoomControl'] === "disable") {
            return false;
        }

        if (this.isTouchScreen && this._properties['disableZoomControl'] === "auto") {
            return false;
        }

        return true;
    }

    getScale() {
        if (!this.ivyPinch) { return undefined; }
        return this.ivyPinch.scale;
    }

    init() {
        if (this._properties['disabled']) {
            return;
        }

        this._properties['element'] = this.$refs.wrapper;
        this._properties['eventHandler'] = this.myEventHandler;
        this.ivyPinch = new IvyPinch(this._properties);

        this.pollLimitZoom();
    }

    getProperties(changes: any){
        let properties: any = {};

        for (var prop in changes) {
            if (changes[prop] !== undefined) {
                if (prop !== 'properties'){
                    properties[prop] = changes[prop];
                }
                if (prop === 'properties'){
                    properties = changes[prop];
                }
            }
        }
        return properties;
    }

    applyOptionsDefault(defaultOptions: any, options: any): void {
        this._properties = Object.assign({}, defaultOptions, options);
    }

    myEventHandler(event:any) {
        if (event.name === "wheel") {
            this.isZoomedIn = event.detail.scale > 1;
        }
    }

    toggleZoom() {
        this.ivyPinch.toggleZoom();
        this.isZoomedIn = this.getScale() > 1;
    }

    pollLimitZoom() {
        this.ivyPinch.pollLimitZoom();
    }

    setStyles() {
        this.styleObject = {
            'overflow': this._properties['overflow'],
            'background-color': this._properties['backgroundColor']
        };
    }
}
</script>

<style lang="sass">
.pinch-zoom-wrapper
    position: relative
    overflow: hidden
    display: block

.pinch-zoom-content
    height: inherit

.pz-dragging
    cursor: all-scroll

/* Zoom button */
.pz-zoom-button
    position:  absolute
    z-index: 1000
    color: #fff
    background-image: url(data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyNCIgaGVpZ2h0PSIyNCIgc3R5bGU9IiI+PHJlY3QgaWQ9ImJhY2tncm91bmRyZWN0IiB3aWR0aD0iMTAwJSIgaGVpZ2h0PSIxMDAlIiB4PSIwIiB5PSIwIiBmaWxsPSJub25lIiBzdHJva2U9Im5vbmUiLz48ZyBjbGFzcz0iY3VycmVudExheWVyIiBzdHlsZT0iIj48dGl0bGU+TGF5ZXIgMTwvdGl0bGU+PHBhdGggZD0iTTE1LjUgMTRoLS43OWwtLjI4LS4yN0MxNS40MSAxMi41OSAxNiAxMS4xMSAxNiA5LjUgMTYgNS45MSAxMy4wOSAzIDkuNSAzUzMgNS45MSAzIDkuNSA1LjkxIDE2IDkuNSAxNmMxLjYxIDAgMy4wOS0uNTkgNC4yMy0xLjU3bC4yNy4yOHYuNzlsNSA0Ljk5TDIwLjQ5IDE5bC00Ljk5LTV6bS02IDBDNy4wMSAxNCA1IDExLjk5IDUgOS41UzcuMDEgNSA5LjUgNSAxNCA3LjAxIDE0IDkuNSAxMS45OSAxNCA5LjUgMTR6IiBpZD0ic3ZnXzEiIGNsYXNzPSIiIGZpbGw9IiNmZmZmZmYiIGZpbGwtb3BhY2l0eT0iMSIvPjxwYXRoIGQ9Ik0xMiAxMGgtMnYySDl2LTJIN1Y5aDJWN2gxdjJoMnYxeiIgaWQ9InN2Z18zIiBjbGFzcz0iIiBmaWxsPSIjZmZmZmZmIiBmaWxsLW9wYWNpdHk9IjEiLz48L2c+PC9zdmc+), url(data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyNCIgaGVpZ2h0PSIyNCI+PHJlY3QgaWQ9ImJhY2tncm91bmRyZWN0IiB3aWR0aD0iMTAwJSIgaGVpZ2h0PSIxMDAlIiB4PSIwIiB5PSIwIiBmaWxsPSJub25lIiBzdHJva2U9Im5vbmUiLz48ZyBjbGFzcz0iY3VycmVudExheWVyIiBzdHlsZT0iIj48dGl0bGU+TGF5ZXIgMTwvdGl0bGU+PHBhdGggZD0iTTE1LjUgMTRoLS43OWwtLjI4LS4yN0MxNS40MSAxMi41OSAxNiAxMS4xMSAxNiA5LjUgMTYgNS45MSAxMy4wOSAzIDkuNSAzUzMgNS45MSAzIDkuNSA1LjkxIDE2IDkuNSAxNmMxLjYxIDAgMy4wOS0uNTkgNC4yMy0xLjU3bC4yNy4yOHYuNzlsNSA0Ljk5TDIwLjQ5IDE5bC00Ljk5LTV6bS02IDBDNy4wMSAxNCA1IDExLjk5IDUgOS41UzcuMDEgNSA5LjUgNSAxNCA3LjAxIDE0IDkuNSAxMS45OSAxNCA5LjUgMTR6TTcgOWg1djFIN3oiIGlkPSJzdmdfMiIgY2xhc3M9IiIgZmlsbD0iI2ZmZmZmZiIgZmlsbC1vcGFjaXR5PSIxIi8+PC9nPjwvc3ZnPg==)
    background-color: rgba(0, 0, 0, .8)
    background-position: center, -1000px
    background-repeat: no-repeat, no-repeat
    background-size: 40px
    width: 56px
    height: 56px
    border-radius: 4px
    opacity: 0.5
    cursor: pointer
    transition: opacity .1s
    user-select: none

.pz-zoom-button-out
    background-position: -1000px, center

.pz-zoom-button:hover
    opacity: 0.7

.pz-zoom-button.pz-zoom-control-position-right
    right: 16px
    top: 50%
    margin-top: -28px

.pz-zoom-button.pz-zoom-control-position-right-bottom
    right: 16px
    bottom: 32px

.pz-zoom-button.pz-zoom-control-position-bottom
    bottom: 16px
    left: 50%
    margin-left: -28px

/* Zoom control */
.pz-zoom-control
    position: absolute
    background-color: rgba(0, 0, 0, .8)
    border-radius: 4px
    overflow: hidden

.pz-zoom-control.pz-zoom-control-position-right
    right: 16px
    top: 50%
    margin-top: -48px

.pz-zoom-control.pz-zoom-control-position-right-bottom
    right: 16px
    bottom: 32px

.pz-zoom-control.pz-zoom-control-position-bottom
    bottom: 16px
    left: 50%
    margin-left: -48px

.pz-zoom-in,
.pz-zoom-out
    width: 48px
    height: 48px
    background-position: center
    background-repeat: no-repeat
    opacity: 1
    cursor: pointer

.pz-zoom-in:hover,
.pz-zoom-out:hover
    background-color: rgba(255, 255, 255, 0.2)

.pz-zoom-control-position-bottom .pz-zoom-in,
.pz-zoom-control-position-bottom .pz-zoom-out
    float: right

.pz-disabled
    opacity: 0.5
    cursor: default

.pz-disabled:hover
    background-color: rgba(255, 255, 255, 0)

</style>
