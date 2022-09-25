<template>
    <div>
        <v-data-table
            :headers="headers"
            :items="users.data"
            disable-sort
            class="elevation-2"
            hide-default-footer
            disable-pagination
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
                    max-width="600px"
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
                            <v-form ref="editedForm">
                                <v-row>
                                    <v-col cols="12" sm="6" md="6">
                                        <v-text-field
                                            v-model="editedItem.name"
                                            label="Name"
                                            validate-on-blur
                                            :rules="nameRules"
                                            :error-messages="editedItemErrors.name"
                                        ></v-text-field>
                                    </v-col>
                                    <v-col cols="12" sm="6" md="6">
                                        <v-text-field
                                            v-model="editedItem.email"
                                            label="Email"
                                            validate-on-blur
                                            :rules="emailRules"
                                            :error-messages="editedItemErrors.email"
                                            @change="clearEmailErrors()"
                                        ></v-text-field>
                                    </v-col>
                                    <template v-if="editedIndex === -1">
                                        <v-col cols="12" sm="6" md="6">
                                            <v-text-field
                                                :append-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
                                                :type="showPassword ? 'text' : 'password'"
                                                class="input-group--focused"
                                                @click:append="showPassword = !showPassword"
                                                v-model="editedItem.password"
                                                label="Password"
                                                validate-on-blur
                                                :rules="passwordRules"
                                                :error-messages="editedItemErrors.password"
                                            ></v-text-field>
                                        </v-col>
                                        <v-col cols="12" sm="6" md="6">
                                            <v-text-field
                                                :append-icon="showPasswordConfirm ? 'mdi-eye' : 'mdi-eye-off'"
                                                :type="showPasswordConfirm ? 'text' : 'password'"
                                                class="input-group--focused"
                                                @click:append="showPasswordConfirm = !showPasswordConfirm"
                                                v-model="editedItem.password_confirmation"
                                                label="Password Confirm"
                                                validate-on-blur
                                                :rules="passwordConfirmRules"
                                                :error-messages="editedItemErrors.password_confirmation"
                                            ></v-text-field>
                                        </v-col>
                                    </template>
                                    <v-col cols="12" sm="12" md="12">
                                        <v-select
                                            :items="roles"
                                            item-value="id"
                                            item-text="role_name"
                                            v-model="editedItem.group_role"
                                            label="Group Role"
                                            validate-on-blur
                                            :rules="groupRoleRules"
                                            :error-messages="editedItemErrors.group_role"
                                        ></v-select>
                                    </v-col>
                                    <v-col cols="12" sm="6" md="6">
                                        <v-checkbox
                                            v-model="editedItem.is_active"
                                            label="Active User"
                                            :error-messages="editedItemErrors.is_active"
                                        ></v-checkbox>
                                    </v-col>
                                    <!-- <v-col cols="12" sm="6" md="6">
                                        <v-checkbox
                                            v-model="editedItem.is_delete"
                                            label="Delete User"
                                            :error-messages="editedItemErrors.is_delete"
                                        ></v-checkbox>
                                    </v-col> -->
                                </v-row>
                            </v-form>
                        </v-container>
                    </v-card-text>

                    <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn color="secondary" text @click="close">Cancel</v-btn>
                        <v-btn color="primary" text @click="save">Save</v-btn>
                    </v-card-actions>
                    </v-card>
                </v-dialog>
                <v-dialog v-model="dialogDelete" max-width="600px">
                    <v-card>
                    <v-card-title class="text-h5">Are you sure you want to delete this user?</v-card-title>
                    <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn color="secondary" text @click="closeDelete">Cancel</v-btn>
                        <v-btn color="primary" text @click="deleteItemConfirm">OK</v-btn>
                        <v-spacer></v-spacer>
                    </v-card-actions>
                    </v-card>
                </v-dialog>
                <v-dialog v-model="dialogDeactivate" max-width="600px">
                    <v-card>
                    <v-card-title class="text-h5">Are you sure you want to {{ inactiveStatus ? 'deactivate' : 'activate' }} this user?</v-card-title>
                    <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn color="secondary" text @click="closeDeactivate">Cancel</v-btn>
                        <v-btn color="primary" text @click="deactivateUserConfirm">OK</v-btn>
                        <v-spacer></v-spacer>
                    </v-card-actions>
                    </v-card>
                </v-dialog>
                </v-toolbar>
            </template>
            <template v-slot:[`item.actions`]="{ item }">
                <v-icon medium class="mx-1" color="accent" @click="editItem(item)">mdi-pencil</v-icon>
                <v-icon medium class="mx-1" color="warning" @click="deactivateUser(item)">{{ item.is_active === 1 ? 'mdi-account-lock' : 'mdi-account-lock-open' }}</v-icon>
                <v-icon medium color="error" @click="deleteItem(item)">mdi-delete</v-icon>
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
                    <b>Active</b>
                </v-chip>
                <v-chip v-else color="red">
                    <b>In-active</b>
                </v-chip>
            </template>
        </v-data-table>
        <v-row class="text-center px-4 align-center" wrap>
            <v-col class="text-truncate" cols="12" md="2">
                Total {{ users.total }} records
            </v-col>
            <v-col cols="12" md="5">
                <v-pagination
                    v-model="page"
                    :length="users.last_page">
                </v-pagination>
            </v-col>
            <v-col cols="6" md="3">
                <v-select
                    dense
                    outlined
                    hide-details
                    :value="itemsPerPage"
                    label="Items per page"
                    @change="itemsPerPage = parseInt($event, 10)"
                    :items="perPageChoices">
                </v-select>
            </v-col>
            <v-col cols="6" md="2">
                <v-text-field
                    v-model="page"
                    label="Go to page"
                    type="number"
                    outlined
                    hide-details
                    dense
                    min=1
                    :max="users.last_page"
                    @input="page = parseInt($event, 10); 
                            page = (page > users.last_page) ? users.last_page : (page < 1) ? 1 : page"
                ></v-text-field>
            </v-col>
        </v-row>
    </div>
</template>
<script>
export default {
    name: 'UserList',
    data() {
        return {
            roles: [],
            users: {},
            dialog: false,
            dialogDelete: false,
            dialogDeactivate: false,
            headers: [
                { text: '#', value: 'index'},
                { text: 'Name', align: 'start', sortable: false, value: 'name'},
                { text: 'Email', value: 'email' },
                { text: 'Group Role', value: 'group_role' },
                { text: 'Status', value: 'is_active', align: 'center' },
                { text: 'Actions', value: 'actions', align: 'center' },
            ],
            page: 1,
            itemsPerPage: 10,
            perPageChoices: [
                {text:'5 records/page' , value: 5},
                {text:'10 records/page' , value: 10},
                {text:'20 records/page' , value: 20},
            ],
            editedIndex: -1,
            showPassword: false,
            showPasswordConfirm: false,
            editedItem: {
                name: '',
                email: '',
                password: '',
                password_confirmation: '',
                group_role: '',
                id: '',
                is_active: true, //True: 1 => Active,  False: 0 => Inactive
            },
            defaultItem: {
                name: '',
                email: '',
                password: '',
                password_confirmation: '',
                group_role: '',
                id: '',
                is_active: true, //True: 1 => Active,  False: 0 => Inactive
            },
            editedItemErrors: {
                name: [],
                email: [],
                password: [],
                password_confirm: [],
                group_role: [],
                is_active: [],
            },

            inactiveStatus: false,
            itemIdToDeactivate: '', 
            itemIdToDelete: '', 

            // Validation rule
            nameRules: [
                v => !!v || 'Name is required.',
                v => (v && v.length <= 255) || 'Name must be less than 255 characters.',
            ],
            emailRules: [
                v => !!v || 'Email is required.',
                v => (v && v.length <= 255) || 'Email must be less than 255 characters.',
                v => /^(([^<>()[\]\\.,;:\s@']+(\.[^<>()\\[\]\\.,;:\s@']+)*)|('.+'))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/.test(v) || 'E-mail must be valid.',
            ], 
            passwordRules: [
                v => !!v || 'Password is required.',
                v => (v && v.length >= 8) || 'Password must be more than 8 characters.',
            ],
            passwordConfirmRules: [
                v => !!v || 'Password Confirm is required',
                v => (v && v.length >= 8) || 'Password must be more than 8 characters.',
                v => v === this.editedItem.password || 'Password confirm must be match.',
            ],
            groupRoleRules: [
                v => !!v || 'Group role must be selected.',
            ],

            apiHeaders: {
                'Content-Type': 'application/json',
                'Accept': 'application/json',
                'Authorization': 'Bearer ' + sessionStorage.getItem("access_token")
            },
        }
    },
    methods: {
        async load(page, itemsPerPage) {
            await this.$axios
            .get(`${this.$backendUrl}user/users/`, {
                headers: this.apiHeaders,
                params: {
                    page: page,
                    per_page: itemsPerPage
                }
            })
            .then(res => {
                if (res.status === 200) {
                    const DATA = res.data.data;
                    this.users = DATA;
                    this.roles = res.data.roles;
                }
            })
            .catch(err => {
                if (err.status !== 200) console.error(err.response.data.message);
            })
        },
        editItem (item) {
            this.editedIndex = 1;
            item.group_role = parseInt(item.group_role);
            this.editedItem = Object.assign({}, item);
            this.dialog = true;
        },

        deleteItem (item) {
            this.itemIdToDelete = item.id;
            this.dialogDelete = true;
        },

        async deleteItemConfirm () {
            await this.$axios
            .delete(`${this.$backendUrl}user/users/${this.itemIdToDelete}`, {
                headers: this.apiHeaders
            })
            .then(res => {
                alert(res.data.message);
                this.load(this.page, this.itemsPerPage);
            })
            .catch(err => {
                alert(err.response.data.message);
            })

            this.closeDelete();
        },

        close () {
            this.editedItemErrors = {};
            this.$refs.editedForm.reset();
            this.dialog = false;
        },

        closeDelete () {
            this.dialogDelete = false;
            this.itemIdToDelete = '';
        },

        deactivateUser(item) {
            this.itemIdToDeactivate = item.id;
            this.dialogDeactivate = true;
            if(item.is_active === 1) this.inactiveStatus = true;
        },

        async deactivateUserConfirm() {
            await this.$axios
            .put(`${this.$backendUrl}user/users/deactivate/${this.itemIdToDeactivate}`, {} ,{
                headers: this.apiHeaders
            })
            .then(res => {
                alert(res.data.message);
                this.load(this.page, this.itemsPerPage);
            })
            .catch(err => {
                alert(err.response.data.message);
            })

            this.closeDeactivate();
        },

        closeDeactivate () {
            this.dialogDeactivate = false;
            this.itemIdToDeactivate = '';
            this.inactiveStatus = false;
        },

        async save () {
            if(!this.$refs.editedForm.validate()) return;

            let editedItem = this.editedItem;
            if (this.editedIndex > -1) {
                // Call to Edit axios
                await this.$axios
                .put(`${this.$backendUrl}user/users/${editedItem.id}`, {
                    name: editedItem.name,
                    email: editedItem.email,
                    group_role: editedItem.group_role.toString(),
                    is_active: editedItem.is_active ? 1 : 0,
                }, {
                    headers: this.apiHeaders
                })
                .then(res => {
                    alert(res.data.message);
                    this.load(this.page, this.itemsPerPage);
                    this.close();
                })
                .catch(err => {
                    this.editedItemErrors = err.response.data.error;
                    alert(err.response.data.message);
                })
            } else {
                // Call to Create axios
                await this.$axios
                .post(`${this.$backendUrl}user/users`, {
                    name: editedItem.name,
                    email: editedItem.email,
                    password: editedItem.password,
                    password_confirmation: editedItem.password_confirmation,
                    group_role: editedItem.group_role.toString(),
                    is_active: editedItem.is_active ? 1 : 0,
                }, {
                    headers: this.apiHeaders
                })
                .then(res => {
                    alert(res.data.message);
                    this.load(this.page, this.itemsPerPage);
                    this.close();
                })
                .catch(err => {
                    this.editedItemErrors = err.response.data.error;
                    alert(err.response.data.message);
                })
            }
        },
        roleName(roleId) {
            return this.roles.find(x => x.id === parseInt(roleId)).role_name;
        },
        clearEmailErrors() {
            this.editedItemErrors.email = [];
        }
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
        page(val) {
            this.load(val, this.itemsPerPage);
        },
        itemsPerPage(val) {
            this.load(this.page, val)
        }
    },
    mounted() {
        this.load(this.page, this.itemsPerPage);
    }
}
</script>