<template>
  <div id="app" class="d-flex justify-content-center">
    <div class="w-100" style="max-width: 1000px">
      Search Student
      <form class="input-group mb-3" v-on:submit.prevent="getStudent">
        <input
          type="text"
          pattern="\d*"
          onkeydown="return ( event.ctrlKey || event.altKey
                    || (47<event.keyCode && event.keyCode<58 && event.shiftKey==false)
                    || (95<event.keyCode && event.keyCode<106)
                    || (event.keyCode==8) || (event.keyCode==9)
                    || (event.keyCode>34 && event.keyCode<40)
                    || (event.keyCode==46) )"
          class="form-control"
          maxlength="7"
          v-model="student_ID"
          v-on:keyup.enter="getID"
          placeholder="You can only search using Student ID. Example: 6080001"
        />
        <div class="input-group-append">
          <button class="btn btn-outline-primary" type="button" @click="getID">Search</button>
        </div>
      </form>
      <!-- <v-card class="pa-5" style="height:485px;"> </v-card> -->

      <v-card class="" style=""
        ><simplebar data-simplebar-auto-hide="true">
          <div v-if="!check" class="d-flex justify-content-center m-4" style="color:red;">
            <v-icon x-large class="mr-5" color="red">mdi-alert</v-icon>
            <div class="d-flex justify-content-center pa-3">
              <div>
                <h6>Something went wrong..</h6>
                <p class="caption mt-n2 mb-0">
                  Please try again, make sure that you follow the format (Ex. 6080001)
                </p>
              </div>
            </div>
          </div>
          <div class="pa-3" v-else>
            <div v-if="loading" class="loading align-items-center pa-3">
              <v-progress-circular :size="50" color="primary" indeterminate></v-progress-circular>
              <p class="mt-3 primary--text">Searching for {{ search_ID }} ...</p>
            </div>
            <div v-else>
              <div v-if="showing == 0" class="grey--text pa-3">
                Please enter the student ID to search fot the student
              </div>
              <div v-if="showing == 1" class="pa-3">
                Sorry, there is no {{ search_ID }} in our database.
              </div>
              <div v-if="showing == 2" class="pa-3">
                <div class="row d-md-block">
                  <div class="col-md-4 order-1 float-left">
                    <v-card outlined elevation="0" class="text-center pa-3">
                      <v-img
                        :src="std_details.avatar_url || $config.defaultAvatar"
                        contain
                        max-width="230"
                        class="center"
                      />
                      <div class="overline">{{ std_details.sid }}</div>
                    </v-card>
                  </div>
                  <div class="col-md-8 order-2 float-right">
                    <v-card outlined elevation="0" class="profile-card">
                      <span class="overline">About this student</span>
                      <h3>{{ std_details.given_name }} {{ std_details.family_name }}</h3>
                      <v-divider></v-divider>
                      {{ std_details.entry_year }}T{{ std_details.entry_trimester }}<br />
                      Email: {{ checkNull(std_details.email) }}<br />
                      Phone: {{ checkNull(std_details.phone) }}<br />
                      <div
                        class="w-100 d-flex justify-content-end align-self-end  mt-sm-16 mt-md-16 "
                      >
                        <a
                          @click="
                            $router.push({
                              name: 'student_record',
                              params: { sid: std_details.sid },
                            })
                          "
                          class="primary--text"
                          ><b>... See More >></b></a
                        >
                      </div>
                    </v-card>
                  </div>
                  <div class="col-md-4 order-3 float-left">
                    <v-card outlined elevation="0" class="pa-3"
                      ><span class="overline">Taken courses</span>

                      <div class="d-flex justify-content-around flex-wrap flex-row">
                        <div class="order-1 text-center">
                          <p class="mt-n1">Core</p>
                          <h4 class="primary--text mt-n4">
                            {{ std_details.records.core_credits }}
                          </h4>
                          <p class="small mt-n2 ">credits</p>
                        </div>
                        <div class="order-2 text-center">
                          <p class="mt-n1">Required</p>
                          <h4 class="primary--text mt-n4">
                            {{ std_details.records.required_credits }}
                          </h4>
                          <p class="small mt-n2 ">credits</p>
                        </div>
                        <div class="order-3 text-center">
                          <p class="mt-n1">Elective</p>
                          <h4 class="primary--text mt-n4">
                            {{ std_details.records.elective_credits }}
                          </h4>
                          <p class="small mt-n2 ">credits</p>
                        </div>
                      </div>
                    </v-card>
                  </div>
                </div>
              </div>
            </div>
          </div></simplebar
        >
      </v-card>
    </div>
  </div>
</template>
<script>
import simplebar from "simplebar-vue";
import "simplebar/dist/simplebar.min.css";
export default {
  mounted() {
    if (this.$route.query.id) {
      this.student_ID = this.$route.query.id.replace(/[^0-9]+/g, "").substring(0, 7);
      this.getID();
    }
  },
  components: {
    simplebar,
  },
  data() {
    return {
      loading: false,
      check: true,
      showing: 0,
      student_ID: "",
      search_ID: "",
      deley: 2000,
      std_details: [],
    };
  },
  methods: {
    checkNull(item) {
      if (item == " ") {
        return " - ";
      } else if (item == "null") {
        return " - ";
      } else if (!item) {
        return " - ";
      } else {
        return item;
      }
    },
    getID() {
      if (this.student_ID.length == 7) {
        this.$router.replace({ name: "search_student", query: { id: this.student_ID } });
        this.search_ID = this.student_ID;
        this.loading = true;
        this.check = true;
        (this.showing = true), this.getStudent(this.search_ID);
      } else {
        this.check = false;
      }
    },
    async getStudent(id) {
      let query = ` {
                student (searchInput:{sid:"${id}"}) {

                    sid
                    program
                    prefix
                    given_name
                    family_name
                    entry_trimester
                    entry_year
                    nick_name
                    avatar_url
                    email
                    phone
                    lineID
                    lineUID
                    advisor{
                      name
                    }
                    status {
                      current
                    }
                    remarks{
                      message
                      user {
                        username
                      }
                      _id
                    }
                    records{
                      egci_cumulative_gpa
                      total_credits
                      core_credits
                      required_credits
                      elective_credits

                    }
                }
              }
      `;
      query = query.replace(/\s+/g, " ").trim();
      await this.axios
        .post(process.env.VUE_APP_GRAPHQL_URL, { query }, { withCredentials: true })
        .then((res) => {
          if (!res.data.data.student) {
            this.showing = 1;
            this.loading = false;
          } else {
            this.std_details = res.data.data.student;
            this.showing = 2;
            this.loading = false;
          }
        })
        .catch((err) => {
          console.log(err);
        });
    },
  },
};
</script>
<style lang="scss" scoped>
.center {
  display: block;
  margin-left: auto;
  margin-right: auto;
  width: 100%;
}
.profile-card {
  // height: 200px;
  padding: 1em;
}
.loading {
  display: flex;
  flex-direction: column;
  // position: absolute;
  // top: 35%;
  // left: 50%;
  // -ms-transform: translate(-50%, -50%);
  // transform: translate(-50%, -50%);
}
.wk-content-full-height {
  height: calc(var(--app-height) - 156px);
  /* height: var(--app-height); */
  /* height: auto; */
  overflow: auto;
}

input,
button {
  box-shadow: none !important;
}
// @media (max-width: 576px) {
//   .wk-content-full-height-list {
//     height: calc(var(--app-height) - 237px);
//   }
// }
</style>
