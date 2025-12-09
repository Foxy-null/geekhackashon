<script setup lang="ts">
import { ref } from "vue";

interface Task {
  id: number;
  title: string;
  columnId: string;
  completed: boolean;
  createdAt: number;
  updatedAt: number;
}

interface Column {
  id: string;
  title: string;
  bgColor: string;
  order: number;
}

const tasks = ref<Task[]>([
  {
    id: 1,
    title: "基本動詞0~100を覚える",
    columnId: "col-1",
    completed: true,
    createdAt: Date.now(),
    updatedAt: Date.now(),
  },
  {
    id: 2,
    title: "句動詞を例文で学習する",
    columnId: "col-2",
    completed: false,
    createdAt: Date.now(),
    updatedAt: Date.now(),
  },
  {
    id: 3,
    title: "覚えた単語で英作文を書く",
    columnId: "col-3",
    completed: false,
    createdAt: Date.now(),
    updatedAt: Date.now(),
  },
  {
    id: 4,
    title: "基本単語100~200を覚える",
    columnId: "col-1",
    completed: false,
    createdAt: Date.now(),
    updatedAt: Date.now(),
  },
  {
    id: 5,
    title: "応用単語0~100を覚える",
    columnId: "col-1",
    completed: false,
    createdAt: Date.now(),
    updatedAt: Date.now(),
  },
]);

const columns = ref<Column[]>([
  {
    id: "col-1",
    title: "基礎語彙",
    bgColor: "bg-gray-100",
    order: 1,
  },
  {
    id: "col-2",
    title: "文脈学習",
    bgColor: "bg-blue-100",
    order: 2,
  },
  {
    id: "col-3",
    title: "実践定着",
    bgColor: "bg-green-100",
    order: 3,
  },
]);

const newTaskTitle = ref("");
const newColumnTitle = ref("");
const nextTaskId = ref(4);
const nextColumnId = ref(4);

const bgColors = [
  "bg-gray-100",
  "bg-blue-100",
  "bg-green-100",
  "bg-yellow-100",
  "bg-purple-100",
  "bg-pink-100",
  "bg-indigo-100",
];

const addTask = () => {
  if (newTaskTitle.value.trim() && columns.value.length > 0) {
    const firstColumn = columns.value[0];
    tasks.value.push({
      id: nextTaskId.value++,
      title: newTaskTitle.value.trim(),
      columnId: firstColumn.id,
      completed: false,
      createdAt: Date.now(),
      updatedAt: Date.now(),
    });
    newTaskTitle.value = "";
  }
};

const addColumn = () => {
  if (newColumnTitle.value.trim()) {
    const newOrder = Math.max(...columns.value.map((c) => c.order), 0) + 1;
    const colorIndex = columns.value.length % bgColors.length;
    columns.value.push({
      id: `col-${nextColumnId.value++}`,
      title: newColumnTitle.value.trim(),
      bgColor: bgColors[colorIndex],
      order: newOrder,
    });
    newColumnTitle.value = "";
  }
};

const deleteColumn = (columnId: string) => {
  // カラムに属するタスクも削除
  tasks.value = tasks.value.filter((t) => t.columnId !== columnId);
  columns.value = columns.value.filter((c) => c.id !== columnId);
};

const moveTask = (taskId: number, newColumnId: string) => {
  const task = tasks.value.find((t) => t.id === taskId);
  if (task) {
    task.columnId = newColumnId;
    task.updatedAt = Date.now();
  }
};

const deleteTask = (taskId: number) => {
  tasks.value = tasks.value.filter((t) => t.id !== taskId);
};

const toggleTaskCompleted = (taskId: number) => {
  const task = tasks.value.find((t) => t.id === taskId);
  if (task) {
    task.completed = !task.completed;
    task.updatedAt = Date.now();
  }
};

const getTasksByColumn = (columnId: string) => {
  return tasks.value.filter((task) => task.columnId === columnId);
};

const getOtherColumns = (currentColumnId: string) => {
  return columns.value.filter((c) => c.id !== currentColumnId);
};

const sortedColumns = computed(() => {
  return [...columns.value].sort((a, b) => a.order - b.order);
});

const getColumnProgress = (columnId: string) => {
  const columnTasks = getTasksByColumn(columnId);
  if (columnTasks.length === 0) return 0;
  const completedCount = columnTasks.filter((t) => t.completed).length;
  return Math.round((completedCount / columnTasks.length) * 100);
};

const overallProgress = computed(() => {
  if (tasks.value.length === 0) return 0;
  const completedCount = tasks.value.filter((t) => t.completed).length;
  return Math.round((completedCount / tasks.value.length) * 100);
});
</script>

<template>
  <div class="mx-auto max-w-7xl p-6">
    <div class="mb-8 flex items-center gap-4">
      <h1 class="text-3xl font-bold text-gray-800">ロードマップボード</h1>
      <div
        v-if="tasks.length > 0"
        class="flex items-center gap-2 rounded-lg bg-blue-50 px-4 py-2"
      >
        <span class="text-sm font-semibold text-gray-600">全体進捗:</span>
        <span class="text-xl font-bold text-blue-600"
          >{{ overallProgress }}%</span
        >
        <span class="text-xs text-gray-500"
          >({{ tasks.filter((t) => t.completed).length }}/{{
            tasks.length
          }})</span
        >
      </div>
    </div>

    <!-- タスク追加フォーム -->
    <div class="mb-6 rounded-lg bg-white p-4 shadow-md">
      <form class="flex gap-2" @submit.prevent="addTask">
        <input
          v-model="newTaskTitle"
          type="text"
          placeholder="新しいタスクを入力..."
          class="flex-1 rounded-md border border-gray-300 px-4 py-2 focus:border-blue-500 focus:outline-none focus:ring-2 focus:ring-blue-200"
        />
        <button
          type="submit"
          :disabled="columns.length === 0"
          class="rounded-md bg-blue-500 px-6 py-2 font-semibold text-white transition hover:bg-blue-600 disabled:cursor-not-allowed disabled:opacity-50"
        >
          追加
        </button>
      </form>
    </div>

    <!-- カラム追加フォーム -->
    <div class="mb-8 rounded-lg bg-white p-4 shadow-md">
      <form class="flex gap-2" @submit.prevent="addColumn">
        <input
          v-model="newColumnTitle"
          type="text"
          placeholder="新しいカラムを入力..."
          class="flex-1 rounded-md border border-gray-300 px-4 py-2 focus:border-blue-500 focus:outline-none focus:ring-2 focus:ring-blue-200"
        />
        <button
          type="submit"
          class="rounded-md bg-green-500 px-6 py-2 font-semibold text-white transition hover:bg-green-600"
        >
          カラム追加
        </button>
      </form>
    </div>

    <!-- ロードマップボード -->
    <div
      v-if="columns.length > 0"
      class="grid gap-6"
      :class="{
        'grid-cols-1': columns.length === 1,
        'grid-cols-1 md:grid-cols-2': columns.length === 2,
        'grid-cols-1 md:grid-cols-3': columns.length === 3,
        'grid-cols-1 md:grid-cols-2 lg:grid-cols-4': columns.length >= 4,
      }"
    >
      <div
        v-for="column in sortedColumns"
        :key="column.id"
        :class="column.bgColor"
        class="rounded-lg p-4 shadow-md"
      >
        <div class="mb-4">
          <div class="mb-2 flex items-center justify-between">
            <h2 class="text-xl font-semibold text-gray-700">
              {{ column.title }}
            </h2>
            <button
              class="rounded bg-red-500 px-2 py-1 text-xs text-white transition hover:bg-red-600"
              title="カラムを削除"
              @click="deleteColumn(column.id)"
            >
              削除
            </button>
          </div>
          <div class="flex items-center gap-2">
            <span class="text-sm text-gray-500"
              >({{ getTasksByColumn(column.id).length }}個)</span
            >
            <span
              v-if="getTasksByColumn(column.id).length > 0"
              class="rounded bg-white px-2 py-1 text-sm font-semibold"
              :class="{
                'text-green-600': getColumnProgress(column.id) === 100,
                'text-blue-600':
                  getColumnProgress(column.id) > 0 &&
                  getColumnProgress(column.id) < 100,
                'text-gray-500': getColumnProgress(column.id) === 0,
              }"
            >
              {{ getColumnProgress(column.id) }}%
            </span>
          </div>
        </div>

        <div class="space-y-3">
          <div
            v-for="task in getTasksByColumn(column.id)"
            :key="task.id"
            class="rounded-md bg-white p-4 shadow-sm transition hover:shadow-md"
          >
            <!-- チェックボックスとタイトル -->
            <div class="mb-3 flex items-start gap-3">
              <input
                type="checkbox"
                :checked="task.completed"
                class="mt-1 size-5 cursor-pointer rounded border-gray-300 text-blue-600 focus:ring-2 focus:ring-blue-500"
                @change="toggleTaskCompleted(task.id)"
              />
              <p
                class="flex-1 text-gray-800"
                :class="{ 'line-through opacity-60': task.completed }"
              >
                {{ task.title }}
              </p>
            </div>

            <!-- アクションボタン -->
            <div class="flex flex-wrap gap-2">
              <select
                v-if="getOtherColumns(column.id).length > 0"
                class="rounded border border-gray-300 bg-white px-3 py-1 text-xs text-gray-700 transition hover:bg-gray-50 focus:border-blue-500 focus:outline-none focus:ring-2 focus:ring-blue-200"
                @change="
                  (e) => {
                    const target = e.target as HTMLSelectElement;
                    if (target.value) {
                      moveTask(task.id, target.value);
                      target.value = '';
                    }
                  }
                "
              >
                <option value="">移動先を選択...</option>
                <option
                  v-for="otherColumn in getOtherColumns(column.id)"
                  :key="otherColumn.id"
                  :value="otherColumn.id"
                >
                  {{ otherColumn.title }}
                </option>
              </select>
              <button
                class="ml-auto rounded bg-red-500 px-3 py-1 text-xs text-white transition hover:bg-red-600"
                @click="deleteTask(task.id)"
              >
                削除
              </button>
            </div>
          </div>

          <div
            v-if="getTasksByColumn(column.id).length === 0"
            class="rounded-md border-2 border-dashed border-gray-300 p-4 text-center text-gray-400"
          >
            タスクがありません
          </div>
        </div>
      </div>
    </div>

    <!-- カラムが存在しない場合のメッセージ -->
    <div
      v-else
      class="rounded-lg border-2 border-dashed border-gray-300 bg-gray-50 p-8 text-center"
    >
      <p class="text-gray-500">
        カラムがありません。上のフォームからカラムを追加してください。
      </p>
    </div>
  </div>
</template>
