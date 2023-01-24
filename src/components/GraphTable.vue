<template>
  <v-card color="#fff" elevation="2">
    <v-data-table
      :headers="headers"
      :items="innerValue"
      :search="search"
      sort-by="id"
      :sort-desc="true"
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>
            <v-icon class="mr-2">mdi-table</v-icon>
            <span>Graph table</span>
          </v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-text-field
            v-model="search"
            append-icon="mdi-magnify"
            label="Search"
            single-line
            clearable
            hide-details
            class="mr-4"
          ></v-text-field>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on: dialogOn , attrs: dialogAttrs }">
              <v-tooltip bottom>
                <template v-slot:activator="{ on, attrs }">
                  <v-btn
                    color="primary mb-2"
                    dark
                    fab
                    small
                    v-bind="{...attrs, ...dialogOn}"
                    v-on="{...on, ...dialogOn}"
                  >
                    <v-icon>mdi-plus</v-icon>
                  </v-btn>
                </template>
                <span>New Node</span>
              </v-tooltip>
            </template>
            <v-card>
              <v-card-title>
                <v-icon class="mr-2" large>{{ formIcon }}</v-icon>
                <span class="text-h5">{{ formTitle }}</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12">
                      <v-text-field
                        v-model="editedItem.name"
                        label="Name*"
                        outlined
                        required
                        minlength="3"
                        hide-details="auto"
                        @change="nameInputIsValid"
                      ></v-text-field>
                      <transition name="slide">
                        <div name="slide" class="error-validation" v-if="notValid.nameIsRequired">The name field is required</div>
                      </transition>
                      <transition name="slide">
                        <div class="error-validation" v-if="notValid.nameIsRepetitive">the name is Repetitive</div>
                      </transition>
                      <transition name="slide">
                        <div class="error-validation" v-if="notValid.name">
                          Node name must be:
                          <ul>
                            <li>More than 2 characters</li>
                            <li>Not number</li>
                            <li>Not Special characters except Dash (-)</li>
                          </ul>
                        </div>
                      </transition>
                    </v-col>
                    <v-col cols="12">
                      <v-textarea
                        v-model="editedItem.description"
                        outlined
                        hide-details="auto"
                        label="Description"
                        counter
                        maxlength="199"
                        @change="descriptionInputIsValid"
                      ></v-textarea>
                      <div class="error-validation" v-if="notValid.description">Up to 199 characters are allowed</div>
                    </v-col>
                    <v-col cols="12">
                      <v-combobox
                        v-model="editedItem.children"
                        :items="nodes"
                        chips
                        clearable
                        label="Children"
                        hide-details="auto"
                        multiple
                        solo
                        outlined
                        @input="onInputChild"
                        @change="childrenInputIsValid"
                      >
                        <template
                          v-slot:selection="{ attrs, item, select, selected }"
                        >
                          <v-chip
                            v-bind="attrs"
                            :input-value="selected"
                            close
                            @click="select"
                            @click:close="removeChild(item)"
                          >
                            <span>Node</span>&nbsp;
                            <strong>{{ item }}</strong>
                          </v-chip>
                        </template>
                      </v-combobox>
                      <transition name="slide">
                        <div class="error-validation" v-if="notValid.children">
                          Child name must be:
                          <ul>
                            <li>More than 2 characters</li>
                            <li>Not number</li>
                            <li>Not Special characters except Dash (-)</li>
                          </ul>
                        </div>
                      </transition>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="red" text rounded @click="close">
                  <v-icon class="mr-1" small>mdi-close-outline</v-icon>
                  <span>Cancel</span>
                </v-btn>
                <v-btn color="primary" rounded @click="save" :disabled="!checkFormIsValid">
                  <v-icon class="mr-1" small>mdi-content-save</v-icon>
                  <span>Save</span>
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="text-h5">Are you sure you want to delete this item?</v-card-title>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
                <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:item.actions="{ item }">
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-icon small class="mr-2" @click="editItem(item)" v-on="on" v-bind="attrs">
              mdi-pencil
            </v-icon>
          </template>
          <span>Edit</span>
        </v-tooltip>
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
          <v-icon small @click="deleteItem(item)" v-on="on" v-bind="attrs">
            mdi-delete
          </v-icon>
          </template>
          <span>Delete</span>
        </v-tooltip>
      </template>
      <template v-slot:no-data>
        No Data
      </template>
    </v-data-table>
  </v-card>
</template>

<script>
import Vue from "vue";
export default Vue.extend({
  name: 'GraphTable',
  model: {
    prop: 'value',
    event: 'change'
  },
  data: () => ({
    search: '',
    dialog: false,
    dialogDelete: false,
    headers: [
      { text: "Rows", value: "id" },
      { text: "Name", value: "name" },
      { text: "Description", value: "description" },
      { text: "Children", value: "children" },
      { text: "Actions", value: "actions", sortable: false },
    ],
    editedIndex: -1,
    editedItem: {
      id: null,
      name: "",
      description: "",
      children: [],
    },
    defaultItem: {
      id: null,
      name: "",
      description: "",
      children: [],
    },
    notValid: {
      name: false,
      nameIsRequired: false,
      nameIsRepetitive: false,
      description: false,
      children: false
    }
  }),
  props: {
    value: Array
  },
  computed: {
    innerValue: {
      get: function () {
        return this.value
      },
      set: function (val) {
        this.$emit('change', val)
      }
    },
    formTitle () {
      return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
    },
    formIcon () {
      return this.editedIndex === -1 ? 'mdi-shape-circle-plus' : 'mdi-circle-edit-outline'
    },
    nodes () {
      return this.innerValue.map(i => i.name).filter(i => i !== this.editedItem.name)
    },
    checkFormIsValid () {
      return this.nameInputIsValid() && this.descriptionInputIsValid() && this.childrenInputIsValid()
    }
  },
  watch: {
    dialog (val) {
      val || this.close()
    },
    dialogDelete (val) {
      val || this.closeDelete()
    }
  },
  methods: {
    editItem (item) {
      this.editedIndex = this.innerValue.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },
    deleteItem (item) {
      this.editedIndex = this.innerValue.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialogDelete = true
    },
    deleteItemConfirm () {
      this.innerValue.splice(this.editedIndex, 1)
      this.closeDelete()
    },
    close () {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },
    closeDelete () {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },
    removeChild (item) {
      if (this.editedItem.children.length) {
        this.editedItem.children.splice(this.editedItem.children.indexOf(item), 1)
        this.editedItem.children = [...this.editedItem.children]
        this.childrenInputIsValid()
      }
    },
    nodeNameIsValid (value) {
      return /^[a-zA-Z-]{2,}$/g.test(value)
    },
    checkValidationNameInput () {
      this.notValid.name = !this.nodeNameIsValid(this.editedItem.name)
      return !this.notValid.name
    },
    checkRequiredNameInput () {
      this.notValid.nameIsRequired = !this.editedItem.name.length
      return !this.notValid.nameIsRequired
    },
    nameInputIsValid () {
      return this.checkRequiredNameInput() && this.checkValidationNameInput()
    },
    descriptionInputIsValid () {
      this.notValid.description = !(/^[\S \n\t]{0,199}$/g.test(this.editedItem.description))
      return !this.notValid.description
    },
    childrenInputIsValid () {
      this.notValid.children = !!this.editedItem.children.length && this.editedItem.children.some(i => !this.nodeNameIsValid(i))
      return !this.notValid.children
    },
    save () {
      if (!this.checkFormIsValid) {
        return
      }
      if (this.editedIndex > -1) {
        const beforeItem = this.innerValue.find(i => i.id === this.editedItem.id)
        if (beforeItem && beforeItem.name !== this.editedItem.name) {
          this.innerValue.forEach((i, iIndex) => {
            i.children.forEach((j, jIndex) => { 
              if (j === beforeItem.name) {
                this.innerValue[iIndex].children[jIndex] = this.editedItem.name
              }
            })
          })
        }
        Object.assign(this.innerValue[this.editedIndex], this.editedItem)
      } else {
        if (this.nodes.includes(this.editedItem.name)) {
          this.notValid.nameIsRepetitive  = true
          return
        } else {
          this.notValid.nameIsRepetitive  = false
        }
        if (!this.editedItem.id) {
          this.editedItem.id = this.innerValue.length + 1
        }
        this.innerValue.push(this.editedItem)
      }
      this.close()
    },
    onInputChild (children) {
      const lastItem = children[children.length - 1]
      if (lastItem && lastItem.length && this.nodeNameIsValid(lastItem.name) && !this.nodes.includes(lastItem)) {
        this.innerValue.push({
          id: this.innerValue.length + 1,
          name: lastItem,
          description: '',
          children: []
        })
      }
    }
  }
})
</script>

<style lang="scss">
.error-validation {
    background-color: #ffebee;
    border-radius: 8px;
    padding: 12px;
    margin-top: 4px;
}

.slide-enter-active {
   -moz-transition-duration: 0.3s;
   -webkit-transition-duration: 0.3s;
   -o-transition-duration: 0.3s;
   transition-duration: 0.3s;
   -moz-transition-timing-function: ease-in;
   -webkit-transition-timing-function: ease-in;
   -o-transition-timing-function: ease-in;
   transition-timing-function: ease-in;
}

.slide-leave-active {
   -moz-transition-duration: 0.3s;
   -webkit-transition-duration: 0.3s;
   -o-transition-duration: 0.3s;
   transition-duration: 0.3s;
   -moz-transition-timing-function: cubic-bezier(0, 1, 0.5, 1);
   -webkit-transition-timing-function: cubic-bezier(0, 1, 0.5, 1);
   -o-transition-timing-function: cubic-bezier(0, 1, 0.5, 1);
   transition-timing-function: cubic-bezier(0, 1, 0.5, 1);
}

.slide-enter-to, .slide-leave {
   max-height: 100px;
   overflow: hidden;
}

.slide-enter, .slide-leave-to {
   overflow: hidden;
   max-height: 0;
}
</style>