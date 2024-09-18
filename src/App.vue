<template>
  <div class="page">
    <div class="container">
      <h1>Todo app</h1>

      <Vueform size="lg" :endpoint="createTask">
        <TextElement
          name="task"
          placeholder="Add a task"
          floating="Task name"
          rules="required"
        />

        <RadiogroupElement
          name="type"
          :items="['Personal', 'Business']"
          view="tabs"
          default="Personal"
        />

        <ButtonElement name="button" align="right" submits>
          Submit
        </ButtonElement>
      </Vueform>

      <hr class="divider" />

      <Vueform v-model="tasksModel" sync>
        <ListElement
          name="tasks"
          :controls="{
            add: false,
          }"
          :add-class="{
            handle: 'task-sort-handle',
          }"
          sort
          @sort="syncToStorage"
          @remove="syncToStorage"
        >
          <template #default="{ index }">
            <ObjectElement
              :name="index"
              :add-class="[
                'task-container',
                tasksModel.tasks[index].type === 'Personal'
                  ? 'is-personal'
                  : 'is-business',
              ]"
            >
              <ButtonElement
                :label="`#${index + 1} - ${tasksModel.tasks[index].task}`"
                name="edit"
                align="right"
                :conditions="[['editing', '!=', index]]"
                :columns="{
                  label: 8,
                }"
                @click="edit(index)"
              >
                Edit
              </ButtonElement>

              <TextElement
                name="task"
                :conditions="[['editing', index]]"
                :columns="6"
              />

              <RadiogroupElement
                name="type"
                view="tabs"
                :conditions="[['editing', index]]"
                :columns="2"
                :items="{
                  Personal: 'P',
                  Business: 'B',
                }"
              />

              <ButtonElement
                name="cancel"
                :conditions="[['editing', index]]"
                :columns="2"
                danger
                full
                @click="cancel"
              >
                Cancel
              </ButtonElement>

              <ButtonElement
                name="save"
                :conditions="[['editing', index]]"
                :columns="2"
                full
                @click="save"
              >
                Save
              </ButtonElement>
            </ObjectElement>
          </template>
        </ListElement>

        <HiddenElement name="editing" />
      </Vueform>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";

const tasksModel = ref({
  tasks: [],
  editing: null,
});

const createTask = (data, form$) => {
  addToStorage(form$.data);
  syncFromStorage();

  form$.reset();
};

const addToStorage = (data) => {
  let storageData = localStorage.getItem("tasks");
  storageData = storageData ? JSON.parse(storageData) : [];

  storageData.push(data);
  localStorage.setItem("tasks", JSON.stringify(storageData));
};

const syncFromStorage = () => {
  let tasks = localStorage.getItem("tasks");

  tasksModel.value = {
    tasks: tasks ? JSON.parse(tasks) : [],
  };
};

const syncToStorage = () => {
  localStorage.setItem("tasks", JSON.stringify(tasksModel.value.tasks));
};

const edit = (index) => {
  tasksModel.value.editing = index;
};

const cancel = (index) => {
  tasksModel.value.editing = null;
  syncFromStorage();
};

const save = () => {
  syncToStorage();
  tasksModel.value.editing = null;
};

onMounted(() => {
  syncFromStorage();
});
</script>

<style lang="scss">
.page {
  background: #f1f5f9;
  width: 100%;
  min-height: 100vh;
  padding-top: 2rem;
}

.container {
  max-width: 600px;
  margin: 0 auto;
}

h1 {
  font-size: 48px;
  margin-bottom: 2rem;
  font-weight: 600;
}

.divider {
  margin: 2rem 0;
  border-color: #e2e8f0;
}

.task-container {
  background: #ffffff;
  padding: 1rem;

  &.is-personal {
    border-left: 3px solid green;
  }

  &.is-business {
    border-left: 3px solid purple;
  }
}

.task-wrapper {
  display: flex;
  align-items: center;
}

.vf-list-handle.task-sort-handle {
  top: 1rem;
}
</style>
