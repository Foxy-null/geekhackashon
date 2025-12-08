<script setup lang="ts">
import { ref } from "vue";

interface Task {
  id: number;
  title: string;
  status: "todo" | "inProgress" | "done";
}

const tasks = ref<Task[]>([
  { id: 1, title: "プロジェクトの計画を立てる", status: "todo" },
  { id: 2, title: "デザインを作成する", status: "inProgress" },
  { id: 3, title: "開発環境をセットアップする", status: "done" },
]);

const newTaskTitle = ref("");
const nextId = ref(4);

const addTask = () => {
  if (newTaskTitle.value.trim()) {
    tasks.value.push({
      id: nextId.value++,
      title: newTaskTitle.value.trim(),
      status: "todo",
    });
    newTaskTitle.value = "";
  }
};

const moveTask = (
  taskId: number,
  newStatus: "todo" | "inProgress" | "done"
) => {
  const task = tasks.value.find((t) => t.id === taskId);
  if (task) {
    task.status = newStatus;
  }
};

const deleteTask = (taskId: number) => {
  tasks.value = tasks.value.filter((t) => t.id !== taskId);
};

const getTasksByStatus = (status: "todo" | "inProgress" | "done") => {
  return tasks.value.filter((task) => task.status === status);
};

const columns = [
  {
    id: "todo",
    title: "ステージ1",
    status: "todo" as const,
    bgColor: "bg-gray-100",
  },
  {
    id: "inProgress",
    title: "ステージ2",
    status: "inProgress" as const,
    bgColor: "bg-blue-100",
  },
  {
    id: "done",
    title: "ステージ3",
    status: "done" as const,
    bgColor: "bg-green-100",
  },
];
</script>

<template>
  <div class="mx-auto max-w-7xl p-6">
    <h1 class="mb-8 text-3xl font-bold text-gray-800">ロードマップボード</h1>

    <!-- タスク追加フォーム -->
    <div class="mb-8 rounded-lg bg-white p-4 shadow-md">
      <form @submit.prevent="addTask" class="flex gap-2">
        <input
          v-model="newTaskTitle"
          type="text"
          placeholder="新しいタスクを入力..."
          class="flex-1 rounded-md border border-gray-300 px-4 py-2 focus:border-blue-500 focus:outline-none focus:ring-2 focus:ring-blue-200"
        />
        <button
          type="submit"
          class="rounded-md bg-blue-500 px-6 py-2 font-semibold text-white transition hover:bg-blue-600"
        >
          追加
        </button>
      </form>
    </div>

    <!-- ロードマップボード -->
    <div class="grid grid-cols-1 gap-6 md:grid-cols-3">
      <div
        v-for="column in columns"
        :key="column.id"
        :class="column.bgColor"
        class="rounded-lg p-4 shadow-md"
      >
        <h2 class="mb-4 text-xl font-semibold text-gray-700">
          {{ column.title }}
          <span class="ml-2 text-sm text-gray-500"
            >({{ getTasksByStatus(column.status).length }})</span
          >
        </h2>

        <div class="space-y-3">
          <div
            v-for="task in getTasksByStatus(column.status)"
            :key="task.id"
            class="rounded-md bg-white p-4 shadow-sm transition hover:shadow-md"
          >
            <p class="mb-3 text-gray-800">{{ task.title }}</p>

            <div class="flex flex-wrap gap-2">
              <button
                v-if="task.status !== 'todo'"
                @click="moveTask(task.id, 'todo')"
                class="rounded bg-gray-500 px-3 py-1 text-xs text-white transition hover:bg-gray-600"
              >
                ← ステージ1
              </button>
              <button
                v-if="task.status !== 'inProgress'"
                @click="moveTask(task.id, 'inProgress')"
                class="rounded bg-blue-500 px-3 py-1 text-xs text-white transition hover:bg-blue-600"
              >
                {{ task.status === "todo" ? "ステージ2 →" : "← ステージ2" }}
              </button>
              <button
                v-if="task.status !== 'done'"
                @click="moveTask(task.id, 'done')"
                class="rounded bg-green-500 px-3 py-1 text-xs text-white transition hover:bg-green-600"
              >
                ステージ3 →
              </button>
              <button
                @click="deleteTask(task.id)"
                class="ml-auto rounded bg-red-500 px-3 py-1 text-xs text-white transition hover:bg-red-600"
              >
                削除
              </button>
            </div>
          </div>

          <div
            v-if="getTasksByStatus(column.status).length === 0"
            class="rounded-md border-2 border-dashed border-gray-300 p-4 text-center text-gray-400"
          >
            タスクがありません
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
