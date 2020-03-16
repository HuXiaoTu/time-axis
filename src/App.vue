<template>
    <div id="app">
        <TimeAxis @getDateMessage="getDateMessage"></TimeAxis>
    </div>
</template>

<script>
import TimeAxis from './timeAxis/timeAxis';
export default {
    name: 'app',
    data() {
        return {}
    },
    methods: {
        getDateMessage(data) {
            console.info('ws data>>>>', data);
        }
    },
    components: {
        TimeAxis,           // 时间轴
    },
}
</script>

<style lang="scss">
</style>
