<template>
  <div class="add-user">
    <div class="d-flex flex-row justify-content-end">
      <div class="p-2">
        <v-btn color="primary" @click="dialog = true"><v-icon>mdi-plus</v-icon> Add User</v-btn>
      </div>
    </div>
    <v-dialog persistent v-model="dialog" max-width="500px">
      <v-card>
        <v-card-title class=""
          ><h3>Add User</h3>
          <v-spacer></v-spacer>
          <v-icon @click="dialogClose()">mdi-close</v-icon></v-card-title
        >
        <v-form ref="form" v-model="valid" lazy-validation class="pa-3">
          <div class="d-flex justify-content-center flex-column">
            <div class="px-3">
              <v-text-field
                ref="firstName"
                v-model="firstName"
                label="Given Name"
                outlined
                required
                :rules="[() => !!firstName || 'Required']"
              ></v-text-field>
            </div>
            <div class="px-3">
              <v-text-field
                ref="familyName"
                v-model="familyName"
                label="Family Name"
                outlined
                required
                :rules="[() => !!familyName || 'Required']"
              ></v-text-field>
            </div>
          </div>
          <div class="d-flex justify-content-center flex-column">
            <div class="px-3">
              <v-text-field
                ref="email"
                v-model="email"
                label="Email"
                outlined
                required
                :rules="[() => !!email || 'Required', rules.email]"
              ></v-text-field>
            </div>
            <div class="px-3">
              <v-text-field
                ref="password"
                v-model="password"
                label="Password"
                outlined
                required
                autocomplete="off"
                :rules="[
                  () => (!!password && password.length >= 4) || 'Required at least 4 characters',
                ]"
              ></v-text-field>
            </div>
            <div class="px-3">
              <v-select
                ref="group"
                :items="items"
                v-model="group"
                label="Group"
                required
                outlined
              ></v-select>
            </div>
            <div class="px-3">
              <v-select
                label="Linked Instructor"
                :items="instructors"
                outlined
                v-model="linked_instructor"
                item-text="name"
                return-object
              >
                <template v-slot:selection="data">
                  {{ data.item._id ? "" : "-" }}{{ data.item.title }} {{ data.item.name }}
                </template>
                <template v-slot:item="data">
                  {{ data.item._id ? "" : "-" }}{{ data.item.title }} {{ data.item.name }}
                </template>
              </v-select>
            </div>
            <div class="d-flex justify-content-start align-items-center mx-4" v-if="linked_instructor ? linked_instructor.name ? true : false : false ">
              <!-- <label>Advisor</label> -->
              <v-switch
                class="v-input--reverse v-input--expand w-100"
                v-model="isAdvisor"
                hint="Activate Advisor will provide more features to this account"
                persistent-hint
              >
                <template #label>
                  <span class="me-3" style="color: rgba(0, 0, 0, 0.87)">Advisor:</span> {{ isAdvisor ? 'Yes' : 'No' }}
                </template>
              </v-switch>
            </div>
          </div>
          <v-card-actions class="mt-4">
            <v-spacer></v-spacer>
            <v-btn color="gray" text @click="clearText()">clear</v-btn>
            <v-btn class="py-3" style="min-width: 120px" color="primary" @click="validate()">submit</v-btn>
          </v-card-actions>
        </v-form>

        <v-dialog v-model="dialog1" max-width="450px">
          <v-card>
            <v-card-title class="headline grey lighten-2"> Confirm Add </v-card-title><br />
            <v-card-text
              >Are you sure you want to add: <br />First Name: <b>{{ input.firstName }}</b>
              <br />
              Family Name: <b>{{ input.familyName }}</b><br />
              Email: <b>{{ input.email }}</b> <br />
              Password: <b>{{ partialPassword(input.password) }}</b><br>
              Linked Instructor: <b>{{ input.linked_instructor ? input.linked_instructor.name ? input.linked_instructor.title + ' ' + input.linked_instructor.name : '-' : '-' }}</b><br>
              Advisor: <b>{{ input.isAdvisor ? "Yes" : "No" }}</b><br />

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn text color="gray" @click="dialog1 = false">No</v-btn>
                <v-btn color="primary" style="min-width: 100px" @click="addUser()">Yes</v-btn>
              </v-card-actions>
            </v-card-text>
          </v-card>
        </v-dialog>
        <v-snackbar centered v-model="snackbar" :timeout="1000">
          Form submission : success!
        </v-snackbar>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
export default {
  name: "add_user",
  props: [],
  mounted() {
    this.getInstructors()
  },
  data() {
    return {
      firstName: "",
      familyName: "",
      email: "",
      password: "",
      linked_instructor: null,
      group: "",
      isAdvisor: false,
      items: ["lecturer", "program director", "coordinator"],
      dialog: false,
      dialog1: false,
      snackbar: false,
      input: {
        firstName: "",
        familyName: "",
        email: "",
        linked_instructor: {
          _id: '',
          title: '',
          name: '',
        },
        isAdvisor: false,
        password: "",
        group: "",
      },
      valid: true,
      rules: {
        counter: [(v) => v.length <= 4 || "Min 4 characters"],
        email: (value) => {
          const pattern = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
          return pattern.test(value) || "Invalid e-mail.";
        },
      },
      instructors: [{
        _id: null,
        name: "-",
        title: null,
      }],
    };
  },
  computed: {
    form() {
      return {
        firstName: this.firstName,
        familyName: this.familyName,
        email: this.email,
        linked_instructor: this.linked_instructor ? this.linked_instructor._id : '',
        password: this.password,
        group: this.group,
      };
    },
  },
  methods: {
    clearText() {
      (this.firstName = ""),
      (this.familyName = ""),
      (this.email = ""),
      (this.linked_instructor = null),
      (this.password = ""),
      (this.group = "");
    },
    validate() {
      this.$refs.form.validate();
      console.log(this.$refs.form.validate());
      if (this.$refs.form.validate()) {
        this.input.firstName = this.firstName;
        this.input.familyName = this.familyName;
        this.input.email = this.email;
        this.input.isAdvisor = this.linked_instructor ? this.linked_instructor.name ? this.isAdvisor : false : false;
        this.input.linked_instructor = this.linked_instructor;
        this.input.password = this.password;
        this.input.group = this.group;
        this.dialog1 = true;
      } else {
      }
    },
    dialogClose() {
      this.dialog = false;
      this.dialog1 = false;
      this.clearText();
    },
    addUser() {
      let query = `mutation {
                    createUser (userInput: {
                        given_name: "${this.input.firstName}",
                        family_name: "${this.input.familyName}",
                        password: "${this.input.password}",
                        email: "${this.input.email}",
                        isAdvisor: ${this.input.isAdvisor},
                        linked_instructor:  ${this.input.linked_instructor ? (this.input.linked_instructor._id ? '"' + this.input.linked_instructor._id + '"' : null) : null }
                        ${this.input.group ? ', group: "' + this.input.group + '"' : ''}
                    }) {
                      _id
                      display_name
                      username
                      given_name
                      family_name
                      email
                      group
                      createdAt
                      isAdvisor
                      linked_instructor {
                        _id
                        title
                        name
                      }
                      isActive
                      avatar {
                          small
                          medium
                          large
                      }
                    }
                }`;
      query = query.replace(/\s+/g, " ").trim();
      this.axios
        .post(process.env.VUE_APP_GRAPHQL_URL, { query }, { withCredentials: true })
        .then((res) => {
          let newuser = res.data.data.createUser
          newuser["name"] = [newuser.given_name, newuser.family_name].join(" ");
          this.$emit('addUser', newuser)
          this.dialogClose();
          this.snackbar = true;
        })
        .catch((err) => {
          console.log(err);
          this.snackbar = true;
        });
    },
    getInstructors() {
      let query = `
            {
              instructors {
                total
                instructors {
                  _id
                  title
                  name
                }
              }
            }
          `;
      query = query.replace(/\s+/g, ' ').trim()
      this.axios
        .post(process.env.VUE_APP_GRAPHQL_URL, { query }, { withCredentials: true })
        .then((res) => {
          this.instructors = [ this.instructors, ...res.data.data.instructors.instructors ]
        })
        .catch((err) => {
          console.log(err);
        });
    },
    partialPassword(pwd) {
      let hidden = ''
      for(var i = 0; i < pwd.length-2; i++) hidden += "*"
      return pwd.charAt(0) + hidden + pwd.charAt(pwd.length-1)
    }
    // clearForm() {
    //   this.userData = {};
    // },
  },
};
</script>
