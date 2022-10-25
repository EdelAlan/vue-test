<template>
  <Modal class="initial__wrapper" @close="close()">
    <div class="initial">
      <div class="initial__header">
        <div
          class="initial__header__label font-h1"
          v-text="$t('auth.initial.title')"
        ></div>
        <div class="initial__header__icon-close" @click="close()">
          <Icon icon="close" color="#9999A9" />
        </div>
      </div>

      <div class="initial__body">
        <Input
          v-show="isEmailType"
          v-bind="inputEmail"
          v-model="inputEmail.value"
          @keyup.native.enter="checkCredentials()"
        />

        <Input
          v-show="!isEmailType"
          v-bind="inputPhone"
          v-model="inputPhone.value"
          @keyup.native.enter="checkCredentials()"
        />

        <Button
          class="initial__body__btn"
          v-text="continueButtonText"
          @click="checkCredentials()"
        />

        <span v-text="$t('auth.initial.or')"></span>

        <div class="initial__body__social">
          <div
            class="initial__body__social__btn"
            v-for="soc in socialNets"
            :key="soc.code"
            :style="{
              backgroundImage: soc.imgUrl,
              backgroundSize: soc.size,
            }"
            @click="clickSocial(soc)"
          ></div>
        </div>

        <Button
          class="initial__body__btn"
          color="white"
          v-text="changeSigninTypeButtonText"
          @click="changeSigninType()"
        />
      </div>
    </div>
  </Modal>
</template>

<script>
import Icon from "@/components/icons/index.vue";
import Modal from "@/components/ui-kit/modal/index.vue";
import Input from "@/components/ui-kit/input/index.vue";
import Button from "@/components/ui-kit/button/index.vue";

import { axiosAuth } from "@/services/scopes/axios-auth.js";
import { validateEmail } from "@/assets/js/utils/validate-email.js";
import { validatePhone } from "@/assets/js/utils/validate-phone.js";
import { getBackendErrorMessage } from "@/assets/js/utils/get-backend-error-message";

export default {
  components: {
    Icon,
    Modal,
    Input,
    Button,
  },
  props: {
    socialNets: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      isEmailType: true,
      inputEmail: {
        value: "",
        placeholder: "E-mail",
        error: false,
        autocomplete: "email",
        name: "email",
      },
      inputPhone: {
        value: "",
        phone: true,
        error: false,
        autocomplete: "tel",
        name: "tel",
      },
    };
  },
  computed: {
    continueButtonText() {
      return this.isEmailType
        ? this.$t("auth.initial.continueWithEmail")
        : this.$t("auth.initial.continueWithPhone");
    },
    changeSigninTypeButtonText() {
      return this.isEmailType
        ? this.$t("auth.initial.loginWithPhone")
        : this.$t("auth.initial.loginWithEmail");
    },
  },
  methods: {
    async checkCredentials() {
      this.$emit("loading", true);
      if (this.isEmailType) {
        await this.processEmailType();
      } else {
        await this.processPhoneType();
      }
      this.$emit("loading", false);
    },
    async processEmailType() {
      const email = this.inputEmail.value.trim();
      const isValidEmail = this.validateEmailInput(email);

      if (!isValidEmail) return;

      const {
        data: { success: emailExists },
      } = await axiosAuth.checkAuthEmail(email).catch(() => {
        this.$emit("showSignup", { email });
      });

      emailExists && this.$emit("showPassword", email);
    },
    async processPhoneType() {
      const phone = this.inputPhone.value;
      const isValidPhone = this.validatePhoneInput(phone);

      if (!isValidPhone) return;

      const {
        data: { success: phoneExists },
      } = await axiosAuth.checkAuthPhone(phone).catch(() => {
        this.$emit("showSignup", { phone });
      });

      if (!phoneExists) return;

      const {
        data: { success: codeSent },
      } = await axiosAuth.sendAuthCode(phone).catch((error) => {
        this.inputPhone.error = this.getBackendErrorMessage(error);
      });

      codeSent &&
        this.$emit("showConfirmSms", {
          phone,
          validate: false,
        });
    },
    validateEmailInput(email) {
      const isValidEmail = email && validateEmail(email);

      this.inputEmail.error = isValidEmail
        ? false
        : this.$t("auth.initial.wrongEmail");

      return isValidEmail;
    },
    validatePhoneInput(phone) {
      const isValidPhone = phone && validatePhone(phone);

      this.inputPhone.error = isValidPhone
        ? false
        : this.$t("auth.initial.wrongPhone");

      return isValidPhone;
    },
    changeSigninType() {
      this.isEmailType = !this.isEmailType;
    },
    clickSocial(soc) {
      this.$emit("openPopup", soc);
    },
    close() {
      this.$emit("close");
    },
    getBackendErrorMessage,
  },
};
</script>

<style lang="scss" scoped>
.initial {
  &__wrapper >>> .custom-modal {
    width: 440px;
  }

  @include am.wrapper();

  &__header {
    @include am.header();
  }

  &__body {
    @include am.body();

    & > span {
      font-size: 15px;
      line-height: 18px;
      margin: 24px 0;
    }

    &__btn {
      margin-top: 16px;
      box-shadow: none !important;
      width: 100%;

      &__yandex {
        @extend .initial__body__btn;

        display: flex !important;
        align-items: center;
        justify-content: center;
        grid-gap: 10px;

        & > div {
          &:first-child {
            width: 24px;
            height: 24px;
            background: url("../../../assets/images/yandex.png") center
              no-repeat;
            background-size: contain;
            border-radius: 50%;
          }
        }
      }
    }

    &__social {
      display: flex;
      justify-content: space-between;
      grid-gap: 16px;

      & > div {
        width: 100%;
        height: 56px;

        border: 1px solid var.$gray-dark;
        border-radius: 8px;

        background-repeat: no-repeat;
        background-position: center;
        background-size: 28px auto;

        cursor: pointer;

        &:hover {
          border-color: var.$secondary;
        }
      }
    }
  }
}
</style>
