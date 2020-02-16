<template>
    <div>
        <pinch-zoom>
            <img src="https://images.unsplash.com/photo-1544606402-e522c304b027?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1564&q=80">
        </pinch-zoom>
    </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { IvyPinch } from './ivypinch'; 

import PinchZoom from './components/PinchZoom.vue'; 

@Component({
    components: {
        'pinch-zoom': PinchZoom
    },
})
export default class myLib extends Vue {

}
</script>