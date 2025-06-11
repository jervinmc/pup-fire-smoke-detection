<template>
    <v-container fluid class="pa-6">
      <!-- Create Entry Button -->
      <v-row class="mb-4">
        <v-col cols="12" class="d-flex justify-end">
          <v-btn color="primary" dark @click="openCreateDialog">
            <v-icon left>mdi-plus</v-icon>
            Create Entry
          </v-btn>
        </v-col>
      </v-row>
  
      <!-- Entries Table -->
      <v-card elevation="3" class="rounded-lg">
        <v-card-title class="font-weight-bold">📋 Recipient Entries</v-card-title>
        <v-data-table
          :headers="headers"
          :items="entries"
          class="elevation-1"
          dense
        >
          <template v-slot:item.actions="{ item }">
            <v-btn icon @click="openEditDialog(item)" color="blue">
              <v-icon>mdi-pencil</v-icon>
            </v-btn>
            <v-btn icon @click="deleteEntry(item)" color="red">
              <v-icon>mdi-delete</v-icon>
            </v-btn>
          </template>
        </v-data-table>
      </v-card>
  
      <!-- Create/Edit Dialog -->
      <v-dialog v-model="dialog" max-width="500">
        <v-card>
          <v-card-title>
            <span class="text-h6">{{ isEditing ? 'Edit Entry' : 'Add New Entry' }}</span>
          </v-card-title>
          <v-card-text>
            <v-form ref="form" v-model="valid">
              <v-text-field
                v-model="newEntry.fullname"
                label="Full Name"
                :rules="[v => !!v || 'Name is required']"
                required
              />
              <v-text-field
                v-model="newEntry.phone_number"
                label="Phone Number"
                :rules="[v => !!v || 'Number is required']"
                required
              />
              <v-text-field
                v-model="newEntry.email"
                label="Email"
                :rules="[
                  v => !!v || 'Email is required',
                  v => /.+@.+\..+/.test(v) || 'E-mail must be valid'
                ]"
                required
              />
            </v-form>
          </v-card-text>
          <v-card-actions>
            <v-spacer />
            <v-btn text @click="closeDialog">Cancel</v-btn>
            <v-btn color="primary" @click="saveEntry">
              {{ isEditing ? 'Update' : 'Save' }}
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-container>
  </template>
<script>
import axios from 'axios'

export default {
  data() {
    return {
      dialog: false,
      valid: false,
      isEditing: false,
      newEntry: {
        id: null,
        fullname: '',
        phone_number: '',
        email: ''
      },
      entries: [],
      headers: [
        { text: 'Full Name', value: 'fullname' },
        { text: 'Phone Number', value: 'phone_number' },
        { text: 'Email', value: 'email' },
        { text: 'Actions', value: 'actions', sortable: false }
      ]
    }
  },
  created() {
    this.fetchRecepients()
  },
  methods: {
    async fetchRecepients() {
      try {
        const response = await this.$axios.get('recipients/')
        this.entries = response.data
      } catch (error) {
        console.error('Error fetching recipients:', error)
      }
    },
    openCreateDialog() {
      this.isEditing = false
      this.dialog = true
      this.newEntry = {
        id: null,
        fullname: '',
        phone_number: '',
        email: ''
      }
    },
    openEditDialog(entry) {
      this.isEditing = true
      this.dialog = true
      this.newEntry = { ...entry }
    },
    closeDialog() {
      this.dialog = false
      this.$refs.form?.resetValidation()
    },
    async saveEntry() {
      const isValid = this.$refs.form.validate()
      if (!isValid) return

      try {
        if (this.isEditing) {
          await this.$axios.put(`recipients/${this.newEntry.id}/`, this.newEntry)
        } else {
          await this.$axios.post('recipients/', this.newEntry)
        }
        this.closeDialog()
        this.fetchRecepients()
      } catch (error) {
        console.error('Error saving entry:', error)
      }
    },
    async deleteEntry(entry) {
      if (!confirm(`Are you sure you want to delete ${entry.fullname}?`)) return
      try {
        await this.$axios.delete(`recipients/${entry.id}/`)
        this.fetchRecepients()
      } catch (error) {
        console.error('Error deleting entry:', error)
      }
    }
  }
}
</script>
<style scoped>
.v-data-table {
  border-radius: 8px;
}
</style>
  