<template>
    <MainChart class="chart"/>
</template>

<script>
import { Bar } from 'vue-chartjs'

export default {
    extends: Bar,
    mixins: [mixins.reactiveProp],
    props:['score','oprions'],
    mounted () {
        this.renderChart(this.score,this.options)
    }
}
</script>