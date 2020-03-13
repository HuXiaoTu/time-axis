<template>
    <div id="app">
        <TimeAxis></TimeAxis>
    </div>
</template>

<script>
import TimeAxis from './timeAxis/timeAxis';
export default {
    name: 'app',
    data() {
        return {
        }
    },
    components: {
        TimeAxis,           // 时间轴
    },
}
</script>

<style lang="scss">
</style>
