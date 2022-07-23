<template>
  <ListQuotes 
    :quotes="quotes" 
    :listenQuotes="listenQuotes"
    @unlisten="onUnlisten"
  />

  <div class="mt-2 text-right">
    <cite class="text-small">
      Atualizará novamente em <b>{{nextUpdateTime}} segundos</b>
    </cite>
  </div>
</template>


<script>
import api from '@/services/api';
import { onMounted, onUnmounted, reactive, ref, toRefs, watch } from 'vue'
import ListQuotes from './ListQuotes.vue'

const CURRENT_UPDATE_TIME = 10;

export default {
  components: { ListQuotes },
  props: {
    listenQuotes: { type: Array, required: true },
  },
  setup (props, {emit}) {
    const interval = ref(null);
    const quotes = ref({});
    const nextUpdateTime = ref(CURRENT_UPDATE_TIME);

    const methods = reactive({
      async refresh() {
        const { data } = await api.listen(props.listenQuotes);
        quotes.value = data;
      },
      onUnlisten(code) {
        emit('unlisten', code);
      },
      restartInterval() {
        clearInterval(interval.value);
        nextUpdateTime.value = CURRENT_UPDATE_TIME;
        interval.value = setInterval(() => {
          nextUpdateTime.value -= 1;
          if(nextUpdateTime.value <= 0){
            nextUpdateTime.value = CURRENT_UPDATE_TIME;
            this.refresh();
          }
        }, 1000);
      }
    });

    onMounted(() => {
      methods.refresh();
      methods.restartInterval();
    });

    onUnmounted(() => {
      clearInterval(interval.value);
    });

    watch(props, () => {
      methods.refresh();
      methods.restartInterval();
    });

    return { ...toRefs(methods), quotes, nextUpdateTime }
  },
  emits: ['unlisten']
}
</script>

<style scoped>

</style>