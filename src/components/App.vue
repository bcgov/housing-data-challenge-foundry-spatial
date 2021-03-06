<template>
<div id="app" class="container">

    <!-- map visualization  -->
    <map-viz></map-viz>

    <!-- list of active filters + their controls -->
    <section class="filters">
        <nav class="nav controls">
            <div class="nav-left">
                <div class="nav-item">
                    <button @click="showFilterPicker" class="button is-primary">Select filters</button>
                </div>
            </div>

            <div class="nav-right">
                <div class="nav-item">
                    <div class="control is-horizontal">
                        <div class="control-label">
                            <label class="label">Colors:</label>
                        </div>
                        <div class="control">
                            <div class="select is-fullwidth">
                                <map-color-picker></map-color-picker>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </nav>
        <div class="list">
            <div v-for="filter in enabledFilters" class="filter">
                <component :is="filter.component" :filterObject="filter"></component>
            </div>
            <div v-if="enabledFilters.length === 0" class="empty has-text-centered">
                <div class="block">
                    <h2 class="title">No filters selected</h2>
                    <p class="subtitle">Use filters to create meaningful visualizations</p>
                </div>
                <div>
                    <button class="button is-dark" @click="showFilterPicker">Select some filters</button>
                </div>
            </div>
        </div>
    </section>

    <!-- modal window for picking active filters -->
    <filter-picker></filter-picker>

</div>
</template>

<script>
// vuex store
import store from '../store';
// components
import FilterPicker from './FilterPicker.vue';
import ColorPicker from './MapColorPicker.vue';
import MapViz from './Map.vue';

export default {
    name: 'app',
    components: {
        'map-viz': MapViz,
        'filter-picker': FilterPicker,
        'map-color-picker': ColorPicker,
    },
    computed: {
        enabledFilters() {
            return store.state.enabledFilters;
        },
    },
    methods: {
        showFilterPicker() {
            store.commit('showFilterPicker', true);
        },
    },
};
</script>

<style lang="scss">
// customize bulma variables
$primary: #f26721; // Foundry Spatial orange

// import bulma css
@import '~bulma';

// base font size
html {
    font-size: 14px;
    @include tablet {
        font-size: 18px;
    }
}

// full-height layout
html, body {
    height: 100%;
    min-height:440px;
    background: $white-ter;
}

#app {
    height: 100%;
    display: flex;
    flex-flow: column;
}

.legend {
    background-color: #fff;
    border-radius: 3px;
    bottom: 30px;
    box-shadow: 0 1px 2px rgba(0,0,0,0.10);
    font: 12px/20px 'Helvetica Neue', Arial, Helvetica, sans-serif;
    padding: 10px;
    position: absolute;
    right: 10px;
    z-index: 1;
    top: 10px;
    height: 185px;

    h4 {
        margin: 0 0 10px;
        text-align:center;
        font-weight: bold;
    }
    table {
        td {
            text-align:right;
            &:last-child {
                text-align: left;
            }
            span {
                border-radius: 50%;
                display: inline-block;
                height: 10px;
                margin-right: 5px;
                width: 10px;
            }
        }
    }
}
.feature-info {
    position: absolute;
    top: 10px;
    left: 10px;
    background: rgba(255,255,255,0.7);
    padding: 4px;
    border-radius: 4px;
    max-width: 400px;
    max-height: 200px;
}

.filters {
    flex-grow: 1;
    display: flex;
    flex-direction: column;

    .controls {
        box-shadow: 0 2px 3px rgba(10, 10, 10, 0.1), 0 0 0 1px rgba(10, 10, 10, 0.1);
        border-width: 1px 0;
    }

    .list {
        flex-grow: 1;
        display: flex;
        flex-flow: column;
    }

    .filter {
        flex-grow: 1;
        &:not(:last-child) {
            margin-bottom: .25rem;
        }
        display: flex;
    }

    .empty {
        justify-content: center;
        display: flex;
        flex-direction: column;
        background: $grey-lighter;
        flex-grow: 1;
    }
}
</style>
