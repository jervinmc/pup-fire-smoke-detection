<!-- Author: Jervin Macalawa -->
<!-- Added on: July 1, 2023 -->
<!-- Description: This section displays Event page -->
<!-- Updated: July 19, 2023: Adding DialogDeleteForm -->
<template>
    <!-- THIS WILL BE USING AFTER FEASIBILITY 2 -->
    <!-- <div class="pa-10">
        <div>
            <EventHeader @addItem="addItem" />
        </div>
        <div>
            <DialogAddForm :dialog="addItemDialog" @cancelAddDialog="cancelAddDialog" :feature="FEATURES.EVENT"
                @submit="handleSubmitEvent" />
        </div>
        <div>
            <DialogEditForm :dialog="editItemDialog" :data="selectedItem" @cancelEditDialog="cancelEditDialog" :feature="FEATURES.EVENT"
                @submit="handleUpdateEvent" />
        </div>
        <div>
            <DialogDeleteForm @deleteItem="confirmDelete" @cancelDelete="cancelDelete" :dialog="deleteItemDialog" />
        </div>
        <div>
            <EventBody @editItem="editItem" :event_data="event_data" @deleteItem="deleteItem" />
        </div>
        <div>
            <EventFooter />
        </div>
    </div> -->
    <div>
        <v-dialog v-model="addItemDialog" max-width="500">
            <v-card>
                <v-card-title class="headline">Add Restaurant</v-card-title>
                <v-card-text>
                    <v-form ref="form" v-model="valid">
                        <v-text-field v-model="register.restaurant_name" label="Restaurant Name" outlined
                            required></v-text-field>
                        <!-- <v-text-field v-model="image" label="Image URL" outlined required></v-text-field> -->

                        <input type="file" @change="onFileUpload" hidden ref="fileInput" accept="image/*">
                        <v-text-field v-model="register.restaurant_address" label="Restaurant Address" outlined
                            required></v-text-field>
                        <v-btn color="primary" @click="$refs.fileInput.click()">
                            <v-icon left>mdi-upload</v-icon>
                            Upload Image
                        </v-btn>
                    </v-form>
                </v-card-text>
                <v-card-actions>
                    <v-btn color="primary" @click="submitHandlerRegister">Save</v-btn>
                    <v-btn color="secondary" @click="close">Cancel</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        <div class="pa-10">
            <v-row>
                <v-spacer></v-spacer>
                <v-col align="end">
                    <v-btn @click="addItem" color="primary">
                        ADD NEW RESTAURANT
                    </v-btn>
                </v-col>
            </v-row>
        </div>
        <!-- THIS FUNCTION IS ONLY TEMPORARY -->
        <v-data-table class="pa-5" :headers="headers" :items="restaurant_data" :search="search" :loading="isLoading">
            <template v-slot:loading>
                <v-skeleton-loader v-for="n in 5" :key="n" type="list-item-avatar-two-line"
                    class="my-2"></v-skeleton-loader>
            </template>
            <template #[`item.opt`]="{ item }">
                <v-menu offset-y z-index="1">
                    <template v-slot:activator="{ attrs, on }">
                        <v-btn icon v-bind="attrs" v-on="on">
                            <v-icon>mdi-dots-horizontal</v-icon>
                        </v-btn>
                    </template>
                    <v-list dense>
                        <v-list-item @click.stop="viewMenu(item)">
                            <v-list-item-content>
                                <v-list-item-title>View Menu</v-list-item-title>
                            </v-list-item-content>
                        </v-list-item>
                        <v-list-item @click.stop="deleteMenu(item)">
                            <v-list-item-content>
                                <v-list-item-title>Delete</v-list-item-title>
                            </v-list-item-content>
                        </v-list-item>
                    </v-list>
                </v-menu>
            </template>
            <template #[`item.image`]="{ item }">
                <v-img :src="item.image" height="100" width="100">

                </v-img>
            </template>
        </v-data-table>

    </div>
</template>

<script>
import { mapState } from 'vuex';
var cloneDeep = require("lodash.clonedeep");
export default {
    computed: {
        ...mapState('restaurant', ['restaurant_data']),
    },
    data() {
        return {
            deleteItemDialog: false,
            addItemDialog: false,
            selectedItem: [],
            editItemDialog: false,
            file: '',
            register: {
                restaurant_name: '',
                restaurant_address: '',
                image: ''
            },
            headers: [
                { text: "ID", value: "id" },
                { text: "Restaurant", value: "restaurant_name" },
                { text: "Restaurant Address", value: "restaurant_address" },
                { text: "Image", value: "image" },
                { text: "Actions", value: "opt" },
                ,
            ],
        }
    },
    created() {
        this.$store.dispatch("restaurant/view")
    },
    methods: {
        deleteMenu(item){
            this.$store.dispatch("restaurant/delete",item)
            alert("SUCCESSFULLY DELETED")
        },
        close() {
            this.addItemDialog = false
        },
        viewMenu(item) {
            location = `/restaurant/menu?uuid=${item.restaurant_uuid}&id=${item.id}`
        },
        submitHandlerRegister() {
            let form_data = new FormData();
            form_data.append('restaurant_name', this.register.restaurant_name)
            form_data.append('restaurant_address', this.register.restaurant_address)
            form_data.append('image', this.register.image)
            this.$store.dispatch("restaurant/add", form_data)
            alert("SUCCESSFULLY ADDED")
        },
        onFileUpload(e) {
            this.file = e;
            e = e.target.files[0];
            if (e["name"].length > 100) {
                alert("255 characters exceeded filename.");
                return;
            }
            try {
                if (e.size > 16000000) {
                    alert("Only 15mb file can be accepted.");
                    return;
                }
            } catch (error) {
                alert(error);
                return;
            }
            this.register.image = e;
        },
        handleUpdateEvent(val) {
            this.$store.dispatch('events/edit', val)
        },
        cancelEditDialog() {

        },
        confirmDelete() {
            this.$store.dispatch("events/delete", this.selectedItem)
            alert('Succesfully Deleted!')
            this.deleteItemDialog = false
            // location.reload()
        },
        cancelDelete() {
            this.deleteItemDialog = false
        },
        deleteItem(item) {
            this.selectedItem = item
            this.deleteItemDialog = true
        },
        handleSubmitEvent(value) {
            const status = this.$addForm({
                action: "events/add",
                data: value
            })
            alert("Successfully Added!")
            this.addItemDialog = false
            // alert('Successfully Added!')
            // location.reload()


        },
        editItem(item) {
            console.log(item)
            this.selectedItem = cloneDeep(item)
            this.editItemDialog = true
        },
        addItem() {
            this.addItemDialog = true
        },
        cancelAddDialog() {
            this.addItemDialog = false
        }
    },
}

</script>

<style></style>