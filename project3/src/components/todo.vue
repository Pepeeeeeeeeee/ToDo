<template class="tem2">
  <div class="tem1">
    <div><h1 class="h1_1">TO DO LIST</h1></div>

    <table class="table">
      <tr>
        <td class="td1">
          <input class="input" type="text" v-model="group" /><button
            class="button-64 b1"
            @click="addGroup"
          >
            Add Group
          </button>
        </td>
        <td>
          <p>Sort :</p>
          <select @change="onChange()" v-model="selected">
            <option disabled>Select first</option>
            <option>High->Low</option>
            <option>Medium->Low</option>
            <option>Low->High</option>
            <option>Done->Not Done</option>
            <option>Not Done->Done</option>
          </select>
        </td>
      </tr>
      <tr v-for="group in groups" :key="group">
        <td>
          <table>
            <tr>
              <td>
                <h3>{{ group.groupName }}</h3>
              </td>
              <td>
                <button class="button-64 button-65" @click="removeGroup(group)">
                  Delete
                </button>
              </td>
            </tr>
            <tr>
              <td><strong>Task</strong></td>
              <td><strong>Priority</strong></td>
              <td><strong>State</strong></td>
              <td></td>
            </tr>

            <tr v-for="row in group.rows" :key="row">
              <td><input class="input" type="text" v-model="row.task" /></td>
              <td>
                <input
                  class="input"
                  id="priority"
                  @click="changePriority(row)"
                  readonly
                  type="text"
                  v-model="row.priority"
                />
              </td>
              <td>
                <input
                  class="input"
                  @click="changeState(row)"
                  readonly
                  type="text"
                  v-model="row.state"
                />
              </td>
              <td>
                <button
                  class="button-64 button-65"
                  @click="removeRow(group, row)"
                >
                  Done
                </button>
              </td>
              <td>
                <input
                  type="text"
                  style="visibility: hidden"
                  v-model="row.id"
                />
              </td>
            </tr>
          </table>
          <div>
            <button class="button-64 button-65" @click="addRow(group)">
              Add Task
            </button>
          </div>
        </td>
      </tr>
    </table>
  </div>
  <div class="div1">
    <button class="button-64 button-65" @click="postList(groups)">Save</button>
  </div>
</template>

<script>
import axios from "axios";

export default {
  beforeMount() {
    this.getList();
  },

  data() {
    return {
      selected: "",
      groups: [
        {
          groupName: "default",
          rows: [
            { task: "Insert task", priority: "Select", state: "State", id: 0 },
          ],
        },
      ],
    };
  },

  methods: {
    addRow: function (group) {
      group.rows.push({ task: "", priority: "" });
    },
    removeRow: async function (group, row) {
      var index = group.rows.indexOf(row);
      group.rows.splice(index, 1);
      axios
        .delete(`http://127.0.0.1:8000/api/v1/mods/${row.id}`)
        .then((response) => {
          response.data;
          alert("deleted id : " + row.id);
        });
    },
    addGroup: function () {
      if (
        this.group === "undefined" ||
        this.group == null ||
        this.group.replace(/\s/g, "").length < 1
      ) {
        alert("e-e");
        return;
      }
      this.groups.push({
        groupName: this.group,
        rows: [{ task: "", priority: "" }],
      });
    },
    removeGroup: function (group) {
      for (var i = 0; i < group.rows.length; i++) {
        axios
          .delete(`http://127.0.0.1:8000/api/v1/mods/${group.rows[i].id}`)
          .then((response) => {
            response.data;
          });
      }

      var index = this.groups.indexOf(group);
      this.groups.splice(index, 1);
    },
    getList() {
      axios.get("http://127.0.0.1:8000/api/v1/mods/").then((response) => {
        response.data;

        for (var i = 0; i < response.data.length; i++) {
          var j = 0;
          var found = false;

          while (j < this.groups.length && !found) {
            if (this.groups[j].groupName == response.data[i].groupName) {
              found = true;
            } else {
              j++;
            }
          }
          if (found) {
            this.groups[j].rows.push({
              task: response.data[i].task,
              priority: response.data[i].priority,
              state: response.data[i].state,
              id: response.data[i].id,
            });
          } else {
            this.groups.push({
              groupName: response.data[i].groupName,
              rows: [
                {
                  task: response.data[i].task,
                  priority: response.data[i].priority,
                  state: response.data[i].state,
                  id: response.data[i].id,
                },
              ],
            });
          }
        }
      });
    },

    postList() {
      for (var i = 0; i < this.groups.length; i++) {
        for (var j = 0; j < this.groups[i].rows.length; j++) {
          var task = this.groups[i].rows[j];
          var groupName = this.groups[i].groupName;

          if (this.groups[i].rows[j].id > 0) {
            axios
              .put(
                `http://127.0.0.1:8000/api/v1/mods/${this.groups[i].rows[j].id}`,
                {
                  groupName: groupName,
                  task: task.task,
                  state: task.state,
                  priority: task.priority,
                }
              )
              .then((response) => {
                response.data;
              });
          } else {
            axios
              .post("http://127.0.0.1:8000/api/v1/mods/", {
                groupName: groupName,
                task: task.task,
                state: task.state,
                priority: task.priority,
              })
              .then((response) => {
                response.data;
              });
          }
        }
      }
      alert("done");
    },

    changePriority(row) {
      if (row.priority == "medium") {
        row.priority = "low";
        return;
      }

      if (row.priority == "high") {
        row.priority = "medium";
        return;
      }

      row.priority = "high";
    },

    changeState(row) {
      if (row.state == "not done") {
        row.state = "done";
        return;
      }

      row.state = "not done";
    },

    sortPriority(group, param1, param2) {
      var i = 0;
      var j = 0;
      if (group.rows.length == 0) {
        return;
      }
      var row = group.rows[0];
      while (i < group.rows.length) {
        while (i < group.rows.length && group.rows[i].priority == param1) {
          i++;
        }
        if (i < group.rows.length) {
          j = i;
          while (j < group.rows.length && group.rows[j].priority != param1) {
            j++;
          }
          if (j < group.rows.length) {
            row = group.rows[i];
            group.rows[i] = group.rows[j];
            group.rows[j] = row;
            i--;
          }
        }
        i++;
      }
      i = 0;
      while (i < group.rows.length && group.rows[i].priority == param1) {
        i++;
      }
      while (i < group.rows.length) {
        while (i < group.rows.length && group.rows[i].priority == param2) {
          i++;
        }
        if (i < group.rows.length) {
          j = i;
          while (j < group.rows.length && group.rows[j].priority != param2) {
            j++;
          }
          if (j < group.rows.length) {
            row = group.rows[i];
            group.rows[i] = group.rows[j];
            group.rows[j] = row;
            i--;
          }
        }
        i++;
      }
    },

    sortState(group, param1, param2) {
      var i = 0;
      var j = 0;
      if (group.rows.length == 0) {
        return;
      }
      var row = group.rows[0];
      while (i < group.rows.length) {
        while (i < group.rows.length && group.rows[i].state == param1) {
          i++;
        }
        if (i < group.rows.length) {
          j = i;
          while (j < group.rows.length && group.rows[j].state != param1) {
            j++;
          }
          if (j < group.rows.length) {
            row = group.rows[i];
            group.rows[i] = group.rows[j];
            group.rows[j] = row;
            i--;
          }
        }
        i++;
      }
      i = 0;
      while (i < group.rows.length && group.rows[i].state == param1) {
        i++;
      }
      while (i < group.rows.length) {
        while (i < group.rows.length && group.rows[i].state == param2) {
          i++;
        }
        if (i < group.rows.length) {
          j = i;
          while (j < group.rows.length && group.rows[j].state != param2) {
            j++;
          }
          if (j < group.rows.length) {
            row = group.rows[i];
            group.rows[i] = group.rows[j];
            group.rows[j] = row;
            i--;
          }
        }
        i++;
      }
    },

    onChange() {
      for (var i = 0; i < this.groups.length; i++) {
        switch (this.selected) {
          case "High->Low":
            this.sortPriority(this.groups[i], "high", "medium");
            break;

          case "Medium->Low":
            this.sortPriority(this.groups[i], "medium", "low");
            break;

          case "Low->High":
            this.sortPriority(this.groups[i], "low", "medium");
            break;

          case "Done->Not Done":
            this.sortState(this.groups[i], "done", "not done");
            break;

          case "Not Done->Done":
            this.sortState(this.groups[i], "not done", "done");
            break;
        }
      }
    },
  },
};
</script>


<style scoped>
.td1 {
  display: flex;
}
.tem1 {
  display: grid;
  justify-content: center;
}

.input {
  position: relative;

  font-size: 1.5em;

  background: linear-gradient(21deg, #10abff, #1beabd);

  padding: 3px;

  display: inline-block;

  border-radius: 9999em;
}

.button-64 {
  align-items: center;
  background-image: linear-gradient(144deg, #af40ff, #5b42f3 50%, #00ddeb);
  border: 0;
  border-radius: 8px;
  box-shadow: rgba(151, 65, 252, 0.2) 0 15px 30px -5px;
  box-sizing: border-box;
  color: #ffffff;
  display: flex;
  font-family: Phantomsans, sans-serif;
  font-size: 20px;
  justify-content: center;
  line-height: 1em;
  max-width: 40%;
  min-width: 50px;
  padding: 3px;
  text-decoration: none;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  white-space: nowrap;
  cursor: pointer;
  padding-top: 10px;
}

.button-64:hover {
  background: none;
  background-color: rgb(5, 6, 45);
  padding: 16px 24px;
  border-radius: 6px;
  transition: 300ms;
}

@media (min-width: 768px) {
  .button-64 {
    min-width: 196px;
  }
}

.div1 {
  padding-left: 50px;
}

.tem2 {
  background-color: rgb(66, 66, 66);
}
</style>
