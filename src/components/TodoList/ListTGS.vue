<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>

        <v-card>
            <v-card-title>
                <v-text-field
                    v-model ="search"
                    append-icon="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details
                ></v-text-field>
                <v-spacer></v-spacer>
                <v-select
                            v-model="sort"
                            :items="['Penting','Tidak penting']"
                            single-line
                            hide-details
                            label="Sorting"
                            @change="sortedArray(sort)"
                ></v-select>
                <v-spacer></v-spacer>
                <v-btn color="success" dark @click="dialog =true">
                    Tambah
                </v-btn>
            </v-card-title>
            <v-data-table :headers="headers" 
                :items="todos" 
                :search="search"
                :single-expand="singleExpand"
                item-key="note"
                show-expand
                class="elevation-1"
            >
                <template v-slot:[`item.actions`]="{item}">
                    <v-btn small class="mr-2" @click="editItem(item)">
                            edit
                    </v-btn>
                    <v-btn small @click="deleteItem(item)">
                            delete
                    </v-btn>
                </template>
                <template v-slot:[`item.priority`]="{ item }">
                    <td>
                        <v-chip
                            v-if="item.priority == 'Penting'"
                            color="red"
                            label
                            outlined
                        >
                            {{ item.priority }}
                        </v-chip>
                        <v-chip
                            v-else-if="item.priority == 'Biasa'"
                            color="yellow"
                            label
                            outlined
                        >
                            {{ item.priority }}
                        </v-chip>
                        <v-chip
                            v-else
                            color="green" 
                            label
                            outlined
                        >
                            {{ item.priority }}
                        </v-chip>
                    </td>
                </template>
                <template v-slot:expanded-item="{ headers, item }">
                    <td :colspan="headers.length">
                        {{ item.note }}
                    </td>
                </template>
               <template v-slot:[`item.check`]="{ item }">
                    <v-checkbox
                        color="blue"
                        v-model="item.check"
                        multiple
                        :key="item" @click.capture.stop="checkItem(item)"
                    ></v-checkbox>
                </template>
            </v-data-table>
        </v-card>

        <v-dialog v-model="dialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Form Todo</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-text-field
                            v-model="formTodo.task"
                            label="Task"
                            required
                        ></v-text-field>

                        <v-select
                            v-model="formTodo.priority"
                            :items="['Penting','Biasa','Tidak penting']"
                            label="Priority"
                            required
                        ></v-select>

                        <v-textarea
                            v-model="formTodo.note"
                            label="Note"
                            required
                        ></v-textarea>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">
                        Cancel
                    </v-btn>
                    <v-btn v-if="edit==false" color="blue darken-1" text @click="save">
                        Save
                    </v-btn>
                     <v-btn v-if="edit==true" color="blue darken-1" text @click="saveEdit(formTodo)">
                        Save
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="headline">Are you sure you want to delete this item?</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="cancel">Cancel</v-btn>
              <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-divider></v-divider>
        <br>
        <v-card v-if="selected.length!=0" class="elevation-1">
            <v-card-title>Delete Multipe</v-card-title>
            <v-list-item v-for="(item, index) in selected" :key="index">
                   <li>{{item.task}}</li>
            </v-list-item>
            <v-card-actions>
                <v-btn color="error" class="pa-2 ma-2 float-left" dark @click="deleteAll">
                 Hapus Semua
                </v-btn>
            </v-card-actions>  
        </v-card>
    </v-main>
</template>

<script>
    export default{
        name:"List",
        data(){
            return{
                search: null,
                dialog: false,
                dialogDelete: false,
                tempItem:null,
                edit: false,
                headers:[
                    {
                        text:"Task",
                        align: "start",
                        sortable: true,
                        value: "task",
                    },
                    {text:"Priority", value:"priority"},
                    {text:"Actions", value:"actions"},
                    {text:"", value:"check"},
                ],
                todos:[
                    {
                        task:"bernafas",
                        priority:"Penting",
                        note: "hufffttttt",
                    },
                    {
                        task:"nongkrong",
                        priority:"Tidak Penting",
                        note: "bersama tman2",
                    },
                    {
                        task:"masak",
                        priority:"Biasa",
                        note: "masak air 500ml",
                    },
                ],
                formTodo:{
                    task:null,
                    priority:null,
                    note: null,
                },
                selected:[],
                sort:"unsort",
            };
        },
        methods: {
            save(){
                this.todos.push(this.formTodo);
                this.resetForm();
                this.dialog =false;
            },
            cancel(){
                this.resetForm();
                this.dialog = false;
                this.dialogDelete= false;
            },
            resetForm(){
                this.formTodo ={
                    task: null,
                    priority:null,
                    note:null,
                };
            },
            deleteItem(item){
                this.dialogDelete = true;
                this.tempItem = item;
            },
            deleteItemConfirm(){
                this.todos.splice(this.todos.indexOf(this.tempItem),1);
                this.dialogDelete = false;
            },
            editItem(item){
                this.edit=true;
                this.formTodo ={
                    task: item.task,
                    priority: item.priority,
                    note: item.note,
                },
                this.dialog =true;
                this.tempItem =item;
            },
            saveEdit(formTodo){
                this.tempItem.task = formTodo.task;
                this.tempItem.priority = formTodo.priority;
                this.tempItem.note = formTodo.note;
                this.resetForm();
                this.dialog= false;
                this.edit=false;
            },
            checkItem(item){
                if(this.selected.includes(item)){
                    this.selected.splice(this.selected.indexOf(item),1);
                }
                else{
                    this.selected.push(item);
                };
            },
            deleteAll(){
                for(var i = 0; i<this.selected.length; i++){
                    this.todos.splice(this.todos.indexOf(this.selected[i]),1);
                }
                this.selected=[];
            },
        },
    };
</script>