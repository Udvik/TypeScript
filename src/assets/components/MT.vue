<script setup lang="ts">
import { ref, watch, computed } from "vue";
import {
  InputText,
  Button,
  DataTable,
  Column,
  Dialog,
  IconField,
  InputIcon,
  Checkbox,
  Select,
  Tag,
} from "primevue";
import { FilterMatchMode } from "@primevue/core/api";

type Status = "Done" | "NotDone";

interface Task {
  id: number;
  title: string;
  checked: boolean;
  status: Status;
}

const task = ref<string>("");
const tasks = ref<Task[]>([]);
const editingRef = ref<Task | null>(null);
const editingTitle = ref<string>("");
const visible = ref<boolean>(false);

const filters = ref({
  global: { value: null, matchMode: FilterMatchMode.CONTAINS },
  title: { value: null, matchMode: FilterMatchMode.STARTS_WITH },
  status: { value: null, matchMode: FilterMatchMode.EQUALS },
});

let saved = localStorage.getItem("tasks");
if (saved) {
  tasks.value = JSON.parse(saved);
}

watch(
  tasks,
  (val) => {
    localStorage.setItem("tasks", JSON.stringify(val));
  },
  {
    deep: true,
  }
);

const displayedTasks = computed(() => {
  let items = tasks.value;

  return [...items];
});

const addTask = () => {
  tasks.value.push({
    id: Date.now(),
    title: task.value,
    checked: false,
    status: "NotDone",
  });

  task.value = "";
};

const deleteTask = (id) => {
  tasks.value = tasks.value.filter((item) => item.id != id);
};

const editTask = (data: Task) => {
  editingRef.value = data;
  editingTitle.value = data.title;
  visible.value = true;
};

const saveEdit = () => {
  editingRef.value!.title = editingTitle.value;
  editingRef.value = null;
  editingTitle.value = "";
  visible.value = false;
};

const statuses = ref(["Done", "NotDone"]);

const getSeverity = (status) => {
  switch (status) {
    case "Done":
      return "success";
    case "NotDone":
      return "danger";
  }
};

const onChange = (data: Task) => {
  data.checked = !data.checked;
  data.status = data.checked ? "Done" : "NotDone";
};
</script>

<template>
  <div class="flex justify-center items-center my-4 gap-2">
    <InputText v-model="task" placeholder="New Task" />
    <Button label="Add" severity="success" @click="addTask"></Button>
  </div>

  <div class="flex justify-center">
    <DataTable
      v-model:filters="filters"
      filterDisplay="row"
      :value="displayedTasks"
      data-key="id"
      style="max-width: 60rem"
      :globalFilterFields="['title']"
    >
      <template #header>
        <div class="flex justify-end">
          <IconField>
            <InputIcon>
              <i class="pi pi-search" />
            </InputIcon>
            <InputText
              v-model="filters['global'].value"
              placeholder="Keyword Search"
            />
          </IconField>
        </div>
      </template>
      <template #empty> No customers found. </template>
      <template #loading> Loading customers data. Please wait. </template>
      <Column>
        <template #body="{ data }">
          <Checkbox
            :checked="data.checked"
            binary
            @change="onChange(data)"
          ></Checkbox>
        </template>
      </Column>

      <Column header="Task" field="title" style="min-width: 12rem">
        <template #body="{ data }">
          {{ data.title }}
        </template>
        <template #filter="{ filterModel, filterCallback }">
          <InputText
            v-model="filterModel.value"
            type="text"
            @input="filterCallback()"
            placeholder="Search by name"
          />
        </template>
      </Column>
      <Column
        field="status"
        header="Status"
        :showFilterMenu="false"
        style="min-width: 12rem"
      >
        <template #body="{ data }">
          <Tag :value="data.status" :severity="getSeverity(data.status)" />
        </template>
        <template #filter="{ filterModel, filterCallback }">
          <Select
            v-model="filterModel.value"
            @change="filterCallback()"
            :options="statuses"
            placeholder="Select One"
            style="min-width: 12rem"
            :showClear="true"
          >
            <template #option="slotProps">
              <Tag
                :value="slotProps.option"
                :severity="getSeverity(slotProps.option)"
              />
            </template>
          </Select>
        </template>
      </Column>
      <Column header="Options" style="min-width: 12rem">
        <template #body="{ data }">
          <div class="flex gap-1">
            <Button
              label="delete"
              severity="danger"
              @click="deleteTask(data.id)"
            ></Button>
            <Button
              label="Edit"
              severity="info"
              @click="editTask(data)"
            ></Button>
          </div>
        </template>
      </Column>
    </DataTable>
  </div>

  <Dialog header="Edit Task" v-model:visible="visible">
    <div class="flex flex-col gap-2">
      <label for="edit">New Title</label>
      <InputText id="edit" v-model="editingTitle" />
    </div>
    <div class="my-3">
      <Button label="Save" severity="warn" @click="saveEdit"></Button>
    </div>
  </Dialog>
</template>
