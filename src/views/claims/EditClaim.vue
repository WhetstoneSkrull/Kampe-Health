<template>
  <section class="admin-content" id="contact-search">
    <Navbar />

    <div class="bg-dark m-b-30">
      <div class="container">
        <div class="row p-b-60 p-t-60">
          <div class="col-md-6 text-center mx-auto text-white p-b-30">
            <h3 class="h3">Edit Claim</h3>
          </div>
        </div>
      </div>
    </div>
    <section class="pull-up">
      <div class="container">
        <div class="row list">
          <div class="col-lg-12 col-md-8">
            <div class="card m-b-30">
              <div class="card-body">
               

                <div class="form-row">
                  <div class="form-group col-md-12">
                    <p><label for="inputPassword4">Date of Care</label></p>
                    <date-picker
                      v-model="claimdetails.seen_date"
                      valueType="format"
                    ></date-picker>
                  </div>

                  <div class="form-group col-md-6">
                    <label for="inputCity">Enrollee KAMPE Number</label>
                    <input
                      type="text"
                      class="form-control"
                      id="inputEmail4"
                      :value="claimdetails.patient.id_card_number"
                      disabled
                    />
                  </div>

                  <div class="form-group col-md-6">
                    <label for="inputCity">Enrollee Full Name</label>
                    <input
                      type="text"
                      class="form-control"
                      id="inputEmail4"
                      :value="`${claimdetails.patient.firstname} ${claimdetails.patient.lastname}`"
                      disabled
                    />
                  </div>

                  <div class="form-group col-md-6">
                    <label for="inputCity">Enrollee Phone Number</label>
                    <input
                      type="text"
                      class="form-control"
                      id="inputEmail4"
                      :value="claimdetails.patient.phone_number"
                      disabled
                    />
                  </div>

                  <div class="form-group col-md-6">
                    <label for="inputCity"
                      >Select Diagnosis </label
                    >
                    <select
                      class="form-control"
                      v-model="claimdetails.diagnosis.id"
                    >
                      <option
                        v-for="dis in diseases"
                        v-bind:key="dis.id"
                        :value="dis.id"
                      >
                        {{ dis.name }}
                      </option>
                    </select>
                  </div>
                 </div>


                <div class="form-group col-md-12">
                  <label for="inputAddress">Treatment Summary</label>
                  <textarea
                    name="name"
                    rows="5"
                    cols="80"
                    class="form-control"
                    v-model="claimdetails.treatment"
                  ></textarea>
                </div>

                <div class="form-group">
                  <button
                    class="btn btn-outline-primary btn-block"
                    @click="updateClaim"
                  >
                    Update Claim <i class="fe fe-chevron-right"></i>
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="vld-parent">
        <loading
          :active.sync="isLoading"
          loader="dots"
          :can-cancel="true"
          :is-full-page="fullPage"
        ></loading>
      </div>
    </section>
  </section>
</template>

<script>
import Navbar from "@/views/Navbar.vue";
import ClassicEditor from "@ckeditor/ckeditor5-build-classic";

// Import component
import Loading from "vue-loading-overlay";
// Import stylesheet
import "vue-loading-overlay/dist/vue-loading.css";
// Init plugin
import DatePicker from "vue2-datepicker";

export default {
  components: {
    Navbar,
    Loading,
    DatePicker,
  },
  data() {
    return {
      editor: ClassicEditor,
      user: null,
      agencies: "",
      employees: "",
      clients: "",
      client: "",
      selected_user: "",
      claims: "",
      diseases: "",
      singleclaim: "",
      claimdetails: "",
      clientdetail: "",
      min: 100000000000000000,
      max: 1000000000000000000,
      edit: false,
      isLoading: false,
      fullPage: true,
      claim: {
        provider_id: "",
        client_id: "",
        diagnosis: "",
        treatment: "",
        client_name: "",
        seen_date: "",
        cost: "",
      },
    };
  },
  beforeMount() {
    this.user = JSON.parse(localStorage.getItem("user"));
  },
  computed: {
    randomNumber: function () {
      let authorization_code = Math.floor(
        Math.random() * (this.max - this.min + 1) + this.min
      );
      return authorization_code;
    },
  },
  methods: {
    getClaim() {
      this.user = JSON.parse(localStorage.getItem("user"));
      this.axios
        .get(`/api/v1/auth/detailedClaim/${this.$route.params.id}`)
        .then((response) => {
          this.singleclaim = response.data;
          this.comment = response.data.comments[0];
          this.claimdetails = response.data.singleclaim[0];
          //single client
          this.axios
            .get(
              `/api/v1/auth/user/${response.data.singleclaim[0].client_name}`
            )
            .then((response) => {
              this.clientdetail = response.data.user;
              console.log(response);
            });
          //end of single client

          // Prepared by
          this.axios
            .get(`/api/v1/auth/user/${response.data.singleclaim[0].user_id}`)
            .then((response) => {
              this.prepared_by = response.data.user;
              console.log(response);
            });
          //end of Prepared by
          //get facility
          this.axios
            .get(
              `/api/v1/auth/detailedProviderHmo/${response.data.singleclaim[0].provider_id}`
            )
            .then((response) => {
              this.facility = response.data.data[0];
              console.log(response);
            });
          //end of get facility
          console.log(response);
        })
        .catch((error) => {
          console.error(error);
        });
    },


    getDiseases() {
      this.user = JSON.parse(localStorage.getItem("user"));
      this.axios
        .get(`/api/v1/auth/diagnosis`)
        .then((response) => {
          this.diseases = response.data;
          console.log(response);
        })
        .catch((error) => {
          console.error(error);
        });
    },
    updateClaim() {
      this.user = JSON.parse(localStorage.getItem("user"));    
        this.isLoading = true;
        this.axios
          .post("/api/v1/auth/editClaim/" + this.claimdetails.id, {
            provider_id: this.claimdetails.provider_id,
            user_id: this.user.id,
            agency_id: 439078,
            client_name: this.claimdetails.client_name,
            seen_date: this.claimdetails.seen_date,
            diagnosis: this.claimdetails.diagnosis.id,
            treatment: this.claimdetails.treatment,
            cost: 0.0,
          })

          .then((response) => {
            console.log(response);
            this.isLoading = false;
            this.$toasted.info("updated Successfully", {
              position: "top-center",
              duration: 3000,
            });
             this.$router.push(`/service-processing-form/${this.claimdetails.id}`)
          })
          .catch((error) => {
            console.log(error.response);
            this.isLoading = false;
          });
    },
  },
  created() {
    this.getDiseases();
    this.getClaim();
  },
};
</script>
