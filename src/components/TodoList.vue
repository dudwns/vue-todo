<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { debounce } from 'lodash'
import Sortable from 'sortablejs'
import { useTodosStore } from '~/store/todos'
import TodoItem from '~/components/TodoItem.vue'
import TheIcon from '~/components/TheIcon.vue'
import TheBtn from '~/components/TheBtn.vue'

const todosStore = useTodosStore()
const todoListEl = ref<HTMLDivElement | null>(null)

const debounced = debounce((val: boolean) => {
  todosStore.updateCheckboxes(val)
}, 400)
const isAllChecked = computed({
  get() {
    return (
      !!todosStore.filteredTodos.length &&
      todosStore.filteredTodos.every((todo) => todo.done)
    )
  },
  set(val: boolean) {
    todosStore.todos.forEach((todo) => {
      todo.done = val
    })
    debounced(val)
  }
})

todosStore.fetchTodos()

onMounted(() => {
  initSortable()
})

function toggleAllCheckboxes() {
  isAllChecked.value = !isAllChecked.value
}

// 첫 번째 인수는 드래그 요소를 적용 할 요소, 두 번째 인수는 옵션
function initSortable() {
  if (todoListEl.value) {
    new Sortable(todoListEl.value, {
      handle: '.drag-handle', // 드래그 핸들러 요소
      animation: 0, // 애니메이션 효과
      forceFallback: true, // 브라우저에 의존하지 않고 자체적인 드래그 기능을 강제함
      onEnd: (event) => {
        const { oldIndex, newIndex } = event
        todosStore.reorderTodos({
          oldIndex: oldIndex as number,
          newIndex: newIndex as number
        })
      } // 목록의 순서를 바꾼 후 실행되는 함수
    })
  }
}
</script>

<template>
  <div class="todos-wrap shadow">
    <div class="todo-head">
      <TheIcon
        :active="isAllChecked"
        @click="toggleAllCheckboxes">
        done_all
      </TheIcon>
      <div class="btn-group">
        <TheBtn
          v-for="filter in todosStore.filters"
          :key="filter.name"
          :active="todosStore.filterStatus === filter.name"
          @click="todosStore.filterStatus = filter.name">
          {{ filter.label }}
        </TheBtn>
        <TheBtn
          danger
          @click="todosStore.deleteDoneTodos">
          완료만 삭제
        </TheBtn>
      </div>
    </div>
    <div
      ref="todoListEl"
      class="todo-list">
      <TodoItem
        v-for="todo in todosStore.filteredTodos"
        :key="todo.id"
        :todo="todo" />
    </div>
  </div>
</template>

<style scoped lang="scss">
.todos-wrap {
  border-radius: 6px;
  overflow: hidden;
}
</style>
