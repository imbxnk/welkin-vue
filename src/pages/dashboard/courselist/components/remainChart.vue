<template>
  <div>
    <div id="chart">
      <apexchart type="bar" :height="height" :options="chartOptions" :series="series"></apexchart>
    </div>
    <v-dialog v-model="dialog" width="500">
      <v-card>
        <v-card-title class="mb-n2">
          Student with ID {{ this.batch[this.index] }}<v-spacer></v-spacer>
          <v-icon @click="dialog = false">mdi-close</v-icon>
        </v-card-title>

        <v-divider class="mx-5"></v-divider>
        <v-card-text>
          Registered : {{ this.passed[this.index] }}<br />
          Remain : {{ this.total[this.index] }}<br /><br />
          <b>Remain Students: </b>
          <div class="wk-remain-std-wrap">
            <div class="wk-remain-std-box">
              <div v-for="(student, i) in unregistered" :key="i" class="my-2">
                {{ i+1 + ". " }} <router-link style="text-decoration: none" target="_blank" :to="{name: 'student_record', params: { sid: student.sid }}">{{ student.name }} {{ checkNull(student.nick_name) }}</router-link>
                <span style="float: right">{{ student.sid }} </span>
              </div>
              <div v-if="unregistered.length === 0">None</div>
            </div>
            <div class="cover-bar"></div>
          </div>
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>
<script>
export default {
  name: "remainChart",
  props: ["batch", "total", "passed", "code"],
  components: {},
  created() {},
  mounted() {
    this.height = 124 + this.batch.length * 35;
  },
  data() {
    return {
      index: 0,
      height: 200,
      registered: [],
      unregistered: [],
      dialog: false,
      series: [
        {
          name: "Registered",
          data: this.passed,
        },
        {
          name: "Remain",
          data: this.total,
        },
      ],
      chartOptions: {
        chart: {
          type: "bar",
          height: this.height,
          stacked: true,
          fontFamily: "Quicksand, sans-serif",
          toolbar: {
            show: false,
          },
          events: {
            dataPointSelection: (event, chartContext, config) => {
              this.getBatch(config.dataPointIndex);
            },
          },
        },
        plotOptions: {
          bar: {
            horizontal: true,
          },
        },
        stroke: {
          width: 1,
          colors: ["#fff"],
        },
        dataLabels: {
          style: {
            colors: ["#fff", "#000000de"],
          },
        },
        // colors: ["#3c84fb", "#97b8f0"],
        // colors: ["#3c84fb", "#000000de"],
        colors: ["#3c84fb", "#0000000e"],
        title: {
          text: "Registered Students",
          style: {
            fontSize: "16px",
            fontWeight: "600",
          },
        },
        xaxis: {
          categories: this.batch,
          labels: {
            formatter: function(val) {
              return val;
            },
          },
        },

        yaxis: {
          title: {
            text: undefined,
          },
        },
        tooltip: {
          y: {
            formatter: function(val) {
              return val;
            },
          },
          x: {
            formatter: function(val) {
              return "Students with ID: " + val;
            },
          },
        },
        fill: {
          opacity: 1,
        },
        legend: {
          position: "top",
          horizontalAlign: "left",
          offsetX: 10,
        },
      },
    };
  },
  methods: {
    checkNull(item) {
      if (item == " " || item == "null" || !item) return "";
      return "(" + item + ")";
    },
    getBatch(i) {
      this.index = i;
      this.dialog = true;
      console.log(this.code);
      this.getList(this.code, this.batch[this.index]);
      console.log(this.batch[this.index]);
    },
    getList(code, batch) {
      let query = `
             {
                countStudent(course_code:"${code}",batch:"${batch}"){
                  batch
                  course
                  total
                  students {
                    sid
                    grade
                  }
                  unregistered {
                    sid
                    nick_name
                    given_name
                    family_name
                  }
                }
              }
          `;
      query = query.replace(/\s+/g, ' ').trim()
      this.axios
        .post(process.env.VUE_APP_GRAPHQL_URL, { query }, { withCredentials: true })
        .then((res) => {
          this.registered = res.data.data.countStudent.students;
          this.unregistered = res.data.data.countStudent.unregistered;
          this.unregistered.forEach((student) => {
            student["name"] = [student.given_name, student.family_name].join(" ");
          });
        })
        .catch((err) => {
          console.log(err);
        });
    },
  },
};
</script>
<style scoped>
.remain-list {
  min-height: 50px;
  max-height: 200px;
}
.test {
  height: 100px;
}
</style>
