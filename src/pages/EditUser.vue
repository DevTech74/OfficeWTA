<template>
  <q-page-container>
    <q-page>
      <div class="row q-pa-md q-gutter-sm">
        <div class="col-6">
          <span class="text-h4">
              User
            </span>
        </div>
      </div>
      <div class="row q-pa-md q-gutter-sm" v-if="user">
        <div class="col-6">
          <q-form class="q-pa-md q-gutter-sm">
            <q-input label="Name" v-model="userView.name"></q-input>
            <q-input label="Email" v-model="userView.email"></q-input>
            <q-input type="password" label="New Password (leave blank to not update)" v-model="userView.password"></q-input>
            <q-btn @click="deleteUser()" class="float-left" color="negative" label="Delete user"></q-btn>
            <!--            <q-btn class="float-left" label="Reset password"></q-btn>-->
            <q-btn @click="saveUser()" class="float-right" color="primary" label="Save"></q-btn>
          </q-form>

        </div>
      </div>
    </q-page>
  </q-page-container>
</template>
<script>
import {mapActions} from "vuex";

export default {
  name: 'EditUser',
  data: function () {
    return {
      user: null,
      userView: null,
    }
  },
  mounted() {
    const that = this;
    that.loadData({
      tableName: "user_account",
      params: {
        query: JSON.stringify([
          {
            column: 'email',
            operator: 'is',
            value: this.$route.params.emailId
          }
        ])
      }
    }).then(function (res) {
      console.log("Loaded user", res);
      if (!res.data || res.data.length === 0) {
        that.$q.notify({
          message: "User not found"
        });
        that.$router.back();
        return
      }
      that.userView = {
          name: res.data[0].name,
          email: res.data[0].email,
        };
        that.user = res.data[0];
      })
    },
    methods: {
      deleteUser(){
        const that = this;
        that.deleteRow({
          tableName: "user_account",
          reference_id: that.user.reference_id
        }).then(function (res) {
          console.log("Deleted user", res);
          that.$q.notify({
            message: "Deleted user"
          });
          that.$router.back();
        }).catch(function (error) {
          that.$q.notify({
            message: JSON.stringify(error)
          })
        })
      },
      saveUser() {
        console.log("Save user", this.user, this.userView);
        const that = this;
        let userObject = {
          tableName: 'user_account',
          id: that.user.reference_id,
          name: that.user.name,
          email: that.user.email,
        };

        if (this.userView.password) {
          userObject.password = this.userView.password;
        }
        that.updateRow(userObject).then(function (res) {
          that.$q.notify({
            message: "Saved user details"
          })
        }).catch(function (err) {
          console.log("Failed to save user details", err)
          that.$q.notify({
            message: "Failed to save user details"
          });

        })
      },
      ...mapActions(['loadData', 'updateRow', 'deleteRow'])
    }
  }
</script>
