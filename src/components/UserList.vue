<template>
    <v-data-table
        :headers="headers"
        :items="users"
        disable-sort
        class="elevation-2"
    >
        <template v-slot:top>
            <v-toolbar
                flat
            >
            <v-toolbar-title>List of Users</v-toolbar-title>
            <v-divider
                class="mx-4"
                inset
                vertical
            ></v-divider>
            <v-spacer></v-spacer>
            <v-dialog
                v-model="dialog"
                max-width="500px"
            >
                <template v-slot:activator="{ on, attrs }">
                <v-btn
                    color="primary"
                    dark
                    class="mb-2"
                    v-bind="attrs"
                    v-on="on"
                >
                    Add User
                </v-btn>
                </template>
                <v-card>
                <v-card-title>
                    <span class="text-h5">{{ formTitle }}</span>
                </v-card-title>

                <v-card-text>
                    <v-container>
                    <v-row>
                        <v-col
                        cols="12"
                        sm="6"
                        md="4"
                        >
                        <v-text-field
                            v-model="editedItem.name"
                            label="Dessert name"
                        ></v-text-field>
                        </v-col>
                        <v-col
                        cols="12"
                        sm="6"
                        md="4"
                        >
                        <v-text-field
                            v-model="editedItem.calories"
                            label="Calories"
                        ></v-text-field>
                        </v-col>
                        <v-col
                        cols="12"
                        sm="6"
                        md="4"
                        >
                        <v-text-field
                            v-model="editedItem.fat"
                            label="Fat (g)"
                        ></v-text-field>
                        </v-col>
                        <v-col
                        cols="12"
                        sm="6"
                        md="4"
                        >
                        <v-text-field
                            v-model="editedItem.carbs"
                            label="Carbs (g)"
                        ></v-text-field>
                        </v-col>
                        <v-col
                        cols="12"
                        sm="6"
                        md="4"
                        >
                        <v-text-field
                            v-model="editedItem.protein"
                            label="Protein (g)"
                        ></v-text-field>
                        </v-col>
                    </v-row>
                    </v-container>
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                    color="blue darken-1"
                    text
                    @click="close"
                    >
                    Cancel
                    </v-btn>
                    <v-btn
                    color="blue darken-1"
                    text
                    @click="save"
                    >
                    Save
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
        <template v-slot:[`item.actions`]="{ item }">
            <v-icon
            small
            class="mr-2"
            @click="editItem(item)"
            >
            mdi-pencil
            </v-icon>
            <v-icon
            small
            @click="deleteItem(item)"
            >
            mdi-delete
            </v-icon>
        </template>
        <template v-slot:no-data>
            <v-btn
            color="primary"
            @click="load"
            >
            Reset
            </v-btn>
        </template>
        <template v-slot:[`item.index`]="{ index }">
            {{ index + 1 }}
        </template>
        <template v-slot:[`item.group_role`]="{ item }">
            <p> {{ roleName(item.group_role) }}</p>
        </template>
        <template v-slot:[`item.is_active`]="{ item }">
            <v-chip v-if="item.is_active === 1" color="green">
                <b>Đang hoạt động</b>
            </v-chip>
            <v-chip v-else color="red">
                <b>Đã khóa</b>
            </v-chip>
        </template>
    </v-data-table>
</template>
<script>
export default {
    name: 'UserList',
    data() {
        return {
            roles: [],
            users: [],
            dialog: false,
            dialogDelete: false,
            headers: [
                { text: '#', value: 'index'},
                { text: 'Name', align: 'start', sortable: false, value: 'name'},
                { text: 'Email', value: 'email' },
                { text: 'Group Role', value: 'group_role' },
                { text: 'Status', value: 'is_active', align: 'center' },
                { text: 'Actions', value: 'actions', sortable: false },
            ],
            editedIndex: -1,
            editedItem: {
                name: '',
                calories: 0,
                fat: 0,
                carbs: 0,
                protein: 0,
            },
            defaultItem: {
                name: '',
                calories: 0,
                fat: 0,
                carbs: 0,
                protein: 0,
            },
        }
    },
    methods: {
        async load() {
            await this.$axios
            .get('http://127.0.0.1:8000/api/user/users/', {
                headers: {
                    Accept: 'application/json',
                    Authorization: 'Bearer ' + sessionStorage.getItem("access_token")
                }
            })
            .then(res => {
                if (res.status === 200) {
                    const DATA = res.data.data;
                    this.users = DATA.data;
                    this.roles = res.data.roles;
                }
            })
            .catch(err => {
                if (err.status !== 200) console.error(err.response.data.message);
            })
        },
        editItem (item) {
            this.editedIndex = this.desserts.indexOf(item)
            this.editedItem = Object.assign({}, item)
            this.dialog = true
        },

        deleteItem (item) {
            this.editedIndex = this.desserts.indexOf(item)
            this.editedItem = Object.assign({}, item)
            this.dialogDelete = true
        },

        deleteItemConfirm () {
            this.desserts.splice(this.editedIndex, 1)
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

        save () {
            if (this.editedIndex > -1) {
            Object.assign(this.desserts[this.editedIndex], this.editedItem)
            } else {
            this.desserts.push(this.editedItem)
            }
            this.close()
        },
        roleName(roleId) {
            return this.roles.find(x => x.id === parseInt(roleId)).role_name;
        },
    },
    computed: {
        formTitle () {
            return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
        },
    },

    watch: {
        dialog (val) {
            val || this.close()
        },
        dialogDelete (val) {
            val || this.closeDelete()
        },
    },
    mounted() {
        this.load();
    }
}
</script>