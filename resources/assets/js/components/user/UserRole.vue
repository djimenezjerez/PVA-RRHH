<template>
  <div>
    <RemoveItem :bus="bus"/>
    <v-data-table
      :headers="headers"
      :items="users"
      :search="search"
      :rows-per-page-items="[10,20,30,{text:'TODO',value:-1}]"
      disable-initial-sort
    >
      <template slot="items" slot-scope="props">
        <tr>
          <td class="text-xs-center">
            <v-tooltip right v-if="props.item.employee">
              <span slot="activator">
                {{ props.item.employee.first_name }} {{ props.item.employee.second_name }} {{ props.item.employee.last_name }} {{ props.item.employee.mothers_last_name }}
              </span>
              <span>
                <div>{{ props.item.name }}</div>
                <div>{{ props.item.position }}</div>
              </span>
            </v-tooltip>
            <span v-else>
              {{ props.item.username }}
            </span>
          </td>
          <td
            v-for="(role, index) in roles"
            v-bind:item="role"
            v-bind:index="index"
            v-bind:key="role.id"
            class="text-xs-center"
          >
            <v-layout row wrap>
              <v-flex xs6></v-flex>
              <v-checkbox @click.native="updateRole(props.item.id, role.id, verifyRole(props.item.roles, role))" :input-value="verifyRole(props.item.roles, role)" readonly></v-checkbox>
            </v-layout>
          </td>
          <td class="text-xs-center">
            <v-tooltip top>
              <v-btn medium slot="activator" flat icon color="red darken-3" @click.native="removeItem(props.item)">
                <v-icon>delete</v-icon>
              </v-btn>
              <span>Eliminar</span>
            </v-tooltip>
            <v-tooltip top>
              <v-btn medium slot="activator" flat icon color="orange darken-3" @click.native="dialog = true; inactive = props.item.id;">
                <v-icon>not_interested</v-icon>
              </v-btn>
              <span>Desactivar</span>
            </v-tooltip>
          </td>
        </tr>
      </template>
    </v-data-table>
    <v-dialog persistent v-model="dialog" max-width="30%" @keydown.esc="dialog = false">
      <v-card>
        <v-card-text>
          <div class="title font-weight-regular">¿Seguro que desea desactivar el usuario?</div>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="success" small @click="dialog = false"><v-icon small>check</v-icon> Cancelar</v-btn>
          <v-btn color="error" small @click="setInactive"><v-icon small>close</v-icon> Desactivar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import Vue from "vue"
import RemoveItem from "../RemoveItem"

export default {
  name: "userRole",
  components: {
    RemoveItem
  },
  data() {
    return {
      bus: new Vue(),
      dialog: false,
      inactive: null,
      roles: [],
      users: [],
      headers: [
        {
          align: "center",
          text: "Usuario",
          value: "username",
          sortable: true
        }
      ],
      search: ""
    }
  },
  mounted() {
    this.getRoles()
    this.getUsers()
    this.bus.$on("closeDialog", () => {
      this.$router.go(0)
    })
  },
  methods: {
    verifyRole(roles, role) {
      if (
        roles.find(obj => {
          return obj.id == role.id
        })
      ) {
        return true
      } else {
        return false
      }
    },
    async getRoles() {
      try {
        let res = await axios.get("/role")
        this.roles = res.data
        this.roles.forEach(role => {
          this.headers.push({
            text: role.display_name,
            value: "",
            sortable: false,
            align: "center"
          })
        })
        this.headers.push({
          text: "Acciones",
          value: "",
          sortable: false,
          align: "center"
        })
      } catch (e) {
        console.log(e)
      }
    },
    async getUsers() {
      try {
        let res = await axios.get("/user")
        this.users = res.data
      } catch (e) {
        console.log(e)
      }
    },
    async setInactive() {
      try {
        if (Number.isInteger(this.inactive)) {
          let res = await axios.get(`user/switch_active/${this.inactive}`)
          this.users = this.users.filter(o => o.id != res.data.id)
          this.inactive = null
          this.dialog = false
          this.toastr.success(`Usuario ${res.data.username} desactivado`)
        }
      } catch (e) {
        console.log(e)
        this.toastr.error('No se pudo desactivar el usuario')
      }
    },
    async updateRole(userId, roleId, status) {
      try {
        if (status) {
          await axios.delete(`/user/${userId}/role/${roleId}`)
        } else if (!status) {
          await axios.patch(`/user/${userId}/role/${roleId}`)
        }
        this.toastr.success("Actualizado correctamente")
      } catch (e) {
        console.log(e)
      } finally {
        this.getUsers()
      }
    },
    removeItem(user) {
      this.bus.$emit("openDialogRemove", `/user/${user.id}`)
    }
  }
}
</script>
