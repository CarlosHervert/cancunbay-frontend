<template>
   <div>
      <v-row>
         <v-col cols="12" :class="[!mobile ? 'px-8' : '']">
            <v-checkbox
               required
               :error-messages="termsErrors"
               class="mt-0 mb-4 checkbox"
               v-model="checkboxTerms"
               :label="$t('forms.payment.accept')"
            >
            </v-checkbox>
            <p>{{ $t("forms.payment.terms_cond") }}</p>

            <div
               v-html="$t('book_politics.content')"
               class="overflow-y-auto politics_show"
            ></div>
         </v-col>
      </v-row>
      <v-row
         :class="[!checkboxTerms ? 'd-none' : 'd-block', 'mt-5']"
         justify="center"
      >
         <v-col cols="12" class="text-center">
            <v-btn
               color="orange"
               dark
               large
               :loading="loading"
               @click="payWithClip"
            >
               Pagar con Clip
               <v-icon right>mdi-credit-card</v-icon>
            </v-btn>
            <div v-if="error" class="mt-3 red--text">
               {{ error }}
            </div>
         </v-col>
      </v-row>
   </div>
</template>

<script>
import { validationMixin } from "vuelidate";

export default {
   mixins: [validationMixin],
   validations: {
      checkboxTerms: {
         checked(val) {
            return val;
         },
      },
   },
   props: {
      clientId: null,
      total: null,
   },
   data() {
      return {
         checkboxTerms: false,
         loading: false,
         error: null,
      };
   },

   computed: {
      mobile() {
         return this.isMobileDevice();
      },
      termsErrors() {
         const errors = [];
         if (!this.$v.checkboxTerms.$dirty) {
            return errors;
         }
         !this.$v.checkboxTerms.checked &&
            errors.push(this.$t("forms.payment.terms_condValidation"));
         return errors;
      },
      stateData() {
         return this.$store.getters["booking/getAllStore"];
      },
   },

   methods: {
      async payWithClip() {
         this.$v.$touch();
         if (this.$v.$invalid) {
            return;
         }

         this.loading = true;
         this.error = null;

         try {
            const response = await this.$axios.post("/clip/createPreference", {
               clientId: this.clientId,
               total: this.total,
               tourData: this.stateData.tours,
               baseUrl: window.location.href.split("?")[0],
            });

            if (
               response.data.status === "success" &&
               response.data.checkout_url
            ) {
               // Redirigir a Clip Checkout
               window.location.href = response.data.checkout_url;
            } else {
               this.error = "No se pudo generar el enlace de pago.";
            }
         } catch (e) {
            console.error("Clip Error:", e);
            this.error = "Error al conectar con Clip. Intente más tarde.";
         } finally {
            this.loading = false;
         }
      },
   },
};
</script>

<style lang="scss" scoped>
.politics_show {
   height: 20rem !important ;
}
</style>
