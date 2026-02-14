<template>
   <v-container id="checkout" class="py-16">
      <v-row justify="center">
         <v-col sm="12" md="6">
            <v-stepper class="steps" flat outlined v-model="steps">
               <v-stepper-header>
                  <v-stepper-step step="1" :complete="steps > 1" color="white">
                     {{ $t("checkout.steps.information") }}
                  </v-stepper-step>

                  <v-divider class="mx-3 divider mr-5"></v-divider>
                  <v-stepper-step step="2" :complete="steps > 2" color="white">
                     {{ $t("checkout.steps.pay") }}
                  </v-stepper-step>

                  <v-divider class="mx-3 divider ml-5"></v-divider>

                  <v-stepper-step step="3" :complete="steps > 3" color="white">
                     {{ $t("checkout.steps.ticket") }}
                  </v-stepper-step>
               </v-stepper-header>
            </v-stepper>
         </v-col>
      </v-row>

      <v-row class="mt-10">
         <v-col cols="12" md="6" :order="mobile ? '2' : '1'">
            <v-card
               width="100%"
               :class="['py-5', steps == 3 ? 'elevation-0' : '']"
            >
               <v-card-text>
                  <InformationForm
                     :class="[steps == 1 ? 'd-block' : 'd-none']"
                  ></InformationForm>
                  <!--<payment-form :class="[(steps==2) ? 'd-block' : 'd-none' ]"></payment-form>-->

                  <v-alert
                     v-if="showError"
                     border="right"
                     colored-border
                     type="error"
                     elevation="2"
                     class="mb-4"
                     dismissible
                  >
                     {{ paymentError }}
                  </v-alert>

                  <paypal
                     v-if="siteD === 'USD'"
                     :class="[steps == 2 ? 'd-block' : 'd-none']"
                     :clientId="clientId"
                     :total="total"
                  ></paypal>

                  <clip
                     v-if="siteD !== 'USD' && steps == 2"
                     :clientId="clientId"
                     :total="total"
                  >
                  </clip>

                  <confirmation
                     :class="[steps == 3 ? 'd-block' : 'd-none']"
                  ></confirmation>
               </v-card-text>
            </v-card>

            <v-row
               v-if="mobile"
               align="baseline"
               :class="!isCheckoutPage ? 'd-none' : 'd-flex mt-10 text-center'"
               no-gutters
            >
               <v-col cols="12" class="text-center">
                  <img
                     class="logo mx-4 d-inline-flex"
                     src="/images/layout/visa.svg"
                     :width="!mobile ? '18%' : '14%'"
                  />

                  <img
                     class="logo mx-4 d-inline-flex"
                     src="/images/layout/mastercard.svg"
                     :width="!mobile ? '8%' : '8%'"
                  />

                  <img
                     class="logo mx-4 d-inline-flex"
                     src="/images/layout/americanexpress.svg"
                     :width="!mobile ? '18%' : '14%'"
                  />

                  <img
                     class="logo mx-4 d-inline-flex"
                     src="/images/layout/paypal.svg"
                     :width="!mobile ? '18%' : '14%'"
                  />
               </v-col>
            </v-row>
            <v-row v-if="mobile" :class="!isCheckoutPage ? 'd-none' : 'd-flex'">
               <v-col cols="12" class="text-center">
                  <client-only>
                     <img
                        class="logo mx-auto"
                        :src="img"
                        :width="!mobile ? '82%' : '100%'"
                     />
                  </client-only>
               </v-col>
            </v-row>
         </v-col>
         <v-col cols="12" md="6" :order="mobile ? '1' : '2'">
            <TourDetail :tour="tourDetail"></TourDetail>

            <v-row
               v-if="!mobile"
               align="baseline"
               :class="!isCheckoutPage ? 'd-none' : 'd-flex mt-10 text-center'"
               no-gutters
            >
               <v-col cols="12" class="text-center">
                  <img
                     class="logo mx-4 d-inline-flex"
                     src="/images/layout/visa.svg"
                     :width="!mobile ? '18%' : '14%'"
                  />

                  <img
                     class="logo mx-4 d-inline-flex"
                     src="/images/layout/mastercard.svg"
                     :width="!mobile ? '8%' : '8%'"
                  />

                  <img
                     class="logo mx-4 d-inline-flex"
                     src="/images/layout/americanexpress.svg"
                     :width="!mobile ? '18%' : '14%'"
                  />

                  <img
                     class="logo mx-4 d-inline-flex"
                     src="/images/layout/paypal.svg"
                     :width="!mobile ? '18%' : '14%'"
                  />
               </v-col>
            </v-row>
            <v-row
               v-if="!mobile"
               :class="!isCheckoutPage ? 'd-none' : 'd-flex'"
            >
               <v-col cols="12" class="text-center">
                  <client-only>
                     <img
                        class="logo mx-auto"
                        :src="img"
                        :width="!mobile ? '82%' : '100%'"
                     />
                  </client-only>
               </v-col>
            </v-row>
         </v-col>
      </v-row>

      <v-dialog v-model="dialog" persistent width="300">
         <v-card color="primary" dark>
            <v-card-text class="pt-5">
               {{ $t("forms.payment.verifying") || "Verificando su pago..." }}
               <v-progress-linear
                  indeterminate
                  color="white"
                  class="mb-0 mt-3"
               ></v-progress-linear>
            </v-card-text>
         </v-card>
      </v-dialog>
   </v-container>
</template>

<script>
import InformationForm from "~/components/checkout/InformationForm";
// import PaymentForm from '~/components/checkout/PaymentForm.vue';
import TourDetail from "~/components/checkout/TourDetail.vue";
// import Categories from '~/components/General/Categories.vue';
// import SectionTitle from '~/components/General/SectionTitle.vue';
import Confirmation from "~/components/checkout/Confirmation.vue";
import Paypal from "~/components/checkout/paypal.vue";
import MercadoPago from "~/components/checkout/MercadoPago.vue";
import Clip from "~/components/checkout/Clip.vue";
export default {
   components: {
      InformationForm,
      TourDetail,
      Confirmation,
      MercadoPago,
      Paypal,
      Clip,
   },
   data() {
      return {
         steps: 1,
         total: 0,
         clientId: 0,
         img:
            this.$i18n.locale === "es"
               ? "/images/layout/100_reembolsable.svg"
               : "/images/layout/100_money-back.svg",
         siteD: this.$i18n.locale === "es" ? "MXN" : "USD",
         paymentError: "",
         showError: false,
         dialog: false,
      };
   },

   computed: {
      tourDetail() {
         return this.$store.getters["booking/tours"];
      },
      alto() {
         if (this.mobile) {
            return 500;
         } else {
            return 468;
         }
      },
      isCheckoutPage() {
         // console.log(this.$route.name);

         if (this.$route.name.includes("checkout")) {
            return true;
         }
         return false;
      },

      mobile() {
         // console.log(this.$route.name);
         return this.isMobileDevice();
      },

      languageCode() {
         return this.$store.getters["booking/language"];
      },
   },

   watch: {
      // Usamos el watch para asegurar que detectamos el cambio aunque el componente se reutilice
      "$route.query": {
         handler(query) {
            console.log("Watch detectó cambio en query:", query);
            const status = query.status;
            const clientId = query.clientId;

            if ((status === "success" || status === "error") && clientId) {
               console.log("Iniciando confirmación desde Watch");
               this.clientId = clientId;
               this.confirmClipPayment(clientId);
            }
         },
         immediate: true,
      },
   },

   mounted() {
      this.getTotalTour();

      this.$nuxt.$on("goPaymentEvent", (id) => {
         this.clientId = id.client;
         this.total = id.total;
         this.$store.dispatch("booking/setTotal", {
            usd: this.total,
            mxn: this.total,
         });
         this.steps = 2;
      });
      this.$nuxt.$on("confirmation", () => {
         this.steps = 3;
      });
   },
   methods: {
      confirmClipPayment(clientId) {
         this.dialog = true; // Mostrar diálogo de cargando si existe
         // Paso 1: Verificar el estado real en el backend consultando la API de Clip
         this.$axios
            .post("/clip/verify-payment-status", { clientId: clientId })
            .then((res) => {
               const finalStatus =
                  res.data.status === "success"
                     ? res.data.final_status
                     : "rejected";
               const paymentId =
                  res.data.payment_id || "CLIP-PAYMENT-" + clientId;

               // Paso 2: Actualizar el pago con el estado final (complet, pending o rejected)
               this.$axios
                  .post("/updatePayment", {
                     clientId: clientId,
                     authorization: paymentId,
                     status: finalStatus,
                     idioma: this.languageCode,
                     merch: "Clip",
                  })
                  .then((response) => {
                     if (finalStatus === "complet") {
                        this.$nuxt.$emit("confirmation");
                     } else if (finalStatus === "pending") {
                        // Tratar como pendiente (similar a ticket de Mercado Pago)
                        this.steps = 3;
                        this.$nuxt.$emit("confirmation");
                     } else {
                        // Rejected
                        this.showError = true;
                        this.paymentError =
                           "El pago no pudo ser procesado. Por favor intente de nuevo.";
                     }
                     this.dialog = false;
                  })
                  .catch((error) => {
                     console.log(error);
                     this.showError = true;
                     this.paymentError = this.$t("forms.payment.error");
                     this.dialog = false;
                  });
            })
            .catch((err) => {
               console.error("Error verificando Clip:", err);
               this.showError = true;
               this.paymentError = "No se pudo verificar el estado de su pago.";
               this.dialog = false;
            });
      },
      async getTotalTour() {
         try {
            if (!this.tourDetail.isPrivate) {
               await this.$axios
                  .post("/getTotal", {
                     id: this.tourDetail.tour_id,
                     adult: this.tourDetail.adultos,
                     child: this.tourDetail.ninos,
                     idioma: this.$store.getters["booking/language"],
                  })
                  .then((resp) => {
                     this.$store.dispatch("booking/setTotal", {
                        usd: resp.data.data,
                        mxn: resp.data.data_mxn,
                     });
                     this.total =
                        this.$i18n.locale === "es"
                           ? resp.data.data_mxn
                           : resp.data.data;
                  });
            } else {
               const rates = this.tourDetail.rates;

               const pos = rates
                  .map((element) => element.pax)
                  .indexOf(this.tourDetail.pax);

               if (pos !== -1) {
                  this.$store.dispatch("booking/setTotal", {
                     usd: parseFloat(rates[pos].real_price),
                     mxn: parseFloat(rates[pos].real_price_mxn),
                  });
                  this.total =
                     this.$i18n.locale === "es"
                        ? parseFloat(rates[pos].real_price_mxn)
                        : parseFloat(rates[pos].real_price);
               }
            }
         } catch (e) {
            this.error = e.response.data.message;
            console.log("error" + e);
         }
      },
   },
};
</script>
