<template>
  <div class="w-screen space-y-10">
    <div class="flex w-full items-center justify-center bg-blue-600 py-5">
      <h1 class="text-center text-4xl text-gray-50">To Do List</h1>
    </div>
    <div class="flex w-full flex-col justify-center pb-6">
      <form @submit.prevent="onSubmit" class="mx-auto w-10/12 p-1 text-center">
        <input
          type="text"
          class="w-full rounded-md border-2 border-gray-500 p-3 pl-4 text-xl"
          v-model="todo"
          placeholder="輸入 To Do，按下 Enter 直接更新"
          aria-label="輸入 To Do，按下 Enter 直接更新"
        />
        <input type="submit" value="送出" v-show="false" />
      </form>
      <div class="flex items-center justify-end py-4 px-8">
        <button
          class="bg-gray-600 py-2 px-5 text-white"
          @click.stop="clearRecord"
        >
          清除
        </button>
      </div>
    </div>
    <div class="mx-auto w-9/12">
      <Container
        @drop="onDrop"
        orientation="vertical"
        class="flex w-full flex-col items-center justify-center gap-7"
      >
        <Draggable v-for="(item, index) in items" :key="item.id" class="w-full">
          <div
            class="flex w-full cursor-grab items-center justify-between rounded-md border-2 border-gray-600 py-4 px-5 active:cursor-grabbing"
          >
            <div class="break-words text-xl">
              <form @submit.prevent="onFixed(index)">
                <input
                  type="text"
                  class="border-2 border-gray-300 p-2 pl-3 outline-none read-only:select-none read-only:border-0 read-only:p-0"
                  v-model="item.text"
                  :readonly="item.readonly"
                />
                <input type="submit" v-show="false" />
              </form>
            </div>
            <div class="flex items-center justify-evenly gap-x-8">
              <span class="cursor-pointer" @click="fixedItem(index)">
                <i class="fa-solid fa-pen"></i>
              </span>
              <span class="cursor-pointer" @click="deleteItem(index)">
                <i class="fa-solid fa-trash"></i>
              </span>
            </div>
          </div>
        </Draggable>
      </Container>
    </div>
  </div>
</template>

<script>
import { onMounted, ref } from "vue";
import { Container, Draggable } from "vue3-smooth-dnd";
export default {
  setup() {
    const onDragClass = "scale-110 duration-100";
    const clearRecord = () => {
      items.value = [];
      window.sessionStorage.clear();
    };
    const items = ref([
      { id: 1, text: "test1", readonly: true },
      { id: 2, text: "test2", readonly: true },
      { id: 3, text: "test3", readonly: true },
    ]);
    const todo = ref("");
    const deleteItem = (index) => {
      items.value.splice(index, 1);
      window.sessionStorage.setItem("items", JSON.stringify(items.value));
    };
    const fixedItem = (index) => {
      items.value[index].readonly = false;
    };
    const onFixed = (index) => {
      items.value[index].readonly = true;
      window.sessionStorage.setItem("items", JSON.stringify(items.value));
    };
    const onSubmit = () => {
      items.value.unshift({
        id: new Date().getTime(),
        text: todo.value,
        readonly: true,
      });
      todo.value = "";
      window.sessionStorage.setItem("items", JSON.stringify(items.value));
    };
    const onDrop = (dropResult) => {
      items.value = applyDrag(items.value, dropResult);
      window.sessionStorage.setItem("items", JSON.stringify(items.value));
    };
    const applyDrag = (arr, dragResult) => {
      const { removedIndex, addedIndex } = dragResult;

      if (removedIndex === null || addedIndex === null) return arr;

      const result = [...arr];
      const dragItem = result[removedIndex];

      result.splice(removedIndex, 1);
      result.splice(addedIndex, 0, dragItem);

      return result;
    };
    onMounted(() => {
      if (window.sessionStorage.getItem("items")) {
        items.value = [...JSON.parse(window.sessionStorage.getItem("items"))];
      }
    });
    return {
      onDragClass,
      items,
      onDrop,
      todo,
      onSubmit,
      deleteItem,
      fixedItem,
      onFixed,
      clearRecord,
    };
  },
  components: {
    Container,
    Draggable,
  },
};
</script>

<style scoped>
.onDragClass {
  @apply scale-110 transition duration-100;
}
</style>
