<!--
Copyright 2017 ODK Central Developers
See the NOTICE file at the top-level directory of this distribution and at
https://github.com/getodk/central-frontend/blob/master/NOTICE.

This file is part of ODK Central. It is subject to the license terms in
the LICENSE file found in the top-level directory of this distribution and at
https://www.apache.org/licenses/LICENSE-2.0. No part of ODK Central,
including this file, may be copied, modified, propagated, or distributed
except according to the terms contained in the LICENSE file.
-->
<template>
  <modal id="field-key-new" :state="state" :hideable="!awaitingResponse"
    backdrop @hide="hideOrComplete" @shown="focusInput">
    <template #title>{{ $t('title') }}</template>
    <template #body>
      <template v-if="step === 0">
        <p class="modal-introduction">{{ $t('introduction[0]') }}</p>
        <form @submit.prevent="submit">
          <form-group ref="displayName" v-model.trim="displayName"
            :placeholder="$t('field.displayName')" required autocomplete="off"/>
          <div class="modal-actions">
            <button type="submit" class="btn btn-primary"
              :disabled="awaitingResponse">
              {{ $t('action.create') }} <spinner :state="awaitingResponse"/>
            </button>
            <button type="button" class="btn btn-link"
              :disabled="awaitingResponse" @click="hideOrComplete">
              {{ $t('action.cancel') }}
            </button>
          </div>
        </form>
      </template>
      <template v-else>
        <div id="field-key-new-success" class="modal-introduction">
          <p>
            <span class="icon-check-circle"></span>
            <strong>{{ $t('common.success') }}</strong>
            <sentence-separator/>
            <span>{{ $t('success[0]', created) }}</span>
          </p>
          <field-key-qr-panel :field-key="created" :managed="managed"/>
          <p>{{ $t('success[1]', created) }}</p>
          <i18n-t tag="p" keypath="success[2].full">
            <template #formAccessSettings>
              <router-link v-slot="{ href, navigate }"
                :to="projectPath('form-access')" custom>
                <a :href="href" @click="navigateToFormAccess(navigate, $event)">{{ $t('success[2].formAccessSettings') }}</a>
              </router-link>
            </template>
          </i18n-t>
        </div>
        <div class="modal-actions">
          <button type="button" class="btn btn-primary" @click="complete">
            {{ $t('action.done') }}
          </button>
          <button type="button" class="btn btn-link" @click="createAnother">
            {{ $t('action.createAnother') }}
          </button>
        </div>
      </template>
    </template>
  </modal>
</template>

<script>
import FormGroup from '../form-group.vue';
import Spinner from '../spinner.vue';
import Modal from '../modal.vue';
import FieldKeyQrPanel from './qr-panel.vue';
import SentenceSeparator from '../sentence-separator.vue';

import request from '../../mixins/request';
import routes from '../../mixins/routes';
import { afterNextNavigation } from '../../util/router';
import { apiPaths } from '../../util/request';
import { noop } from '../../util/util';
import { useRequestData } from '../../request-data';

export default {
  name: 'FieldKeyNew',
  components: { FormGroup, Spinner, Modal, FieldKeyQrPanel, SentenceSeparator },
  mixins: [request(), routes()],
  inject: ['alert'],
  props: {
    state: {
      type: Boolean,
      default: false
    },
    managed: {
      type: Boolean,
      default: false
    }
  },
  emits: ['hide', 'success'],
  setup() {
    // The modal assumes that this data will exist when the modal is shown.
    const { project, fieldKeys } = useRequestData();
    return { project, fieldKeys };
  },
  data() {
    return {
      awaitingResponse: false,
      // There are two steps/screens in the app user creation process. `step`
      // indicates the current step.
      step: 0,
      displayName: '',
      created: null
    };
  },
  watch: {
    state(state) {
      if (!state) {
        this.step = 0;
        this.displayName = '';
        this.created = null;
      }
    }
  },
  methods: {
    focusInput() {
      this.$refs.displayName.focus();
    },
    submit() {
      this.request({
        method: 'POST',
        url: apiPaths.fieldKeys(this.project.id),
        data: { displayName: this.displayName }
      })
        .then(({ data }) => {
          // Reset the form.
          this.alert.blank();
          this.displayName = '';

          this.step = 1;
          this.created = data;
        })
        .catch(noop);
    },
    complete() {
      this.$emit('success', this.created);
    },
    hideOrComplete() {
      if (this.created == null)
        this.$emit('hide');
      else
        this.complete();
    },
    navigateToFormAccess(navigate, event) {
      afterNextNavigation(this.$router, () => {
        // Clear this.fieldKeys so that the Form Access tab will fetch it again.
        this.fieldKeys.data = null;
      });
      navigate(event);
    },
    createAnother() {
      this.step = 0;
      // We do not reset this.created, because it will still be used once the
      // modal is hidden.
      this.$nextTick(this.focusInput);
    }
  }
};
</script>

<style lang="scss">
@import '../../assets/scss/variables';

#field-key-new-success {
  .icon-check-circle {
    color: $color-success;
    font-size: 32px;
    margin-right: 6px;
    vertical-align: middle;
  }

  .field-key-qr-panel {
    box-shadow: $box-shadow-popover;
    margin: 15px auto 30px;
  }
}
</style>

<i18n lang="json5">
{
  "en": {
    // This is the title at the top of a pop-up.
    "title": "Create App User",
    "introduction": [
      "This user will not have access to any Forms at first. You will be able to assign Forms after the user is created."
    ],
    "success": [
      "The App User ???{displayName}??? has been created.",
      // Clicking "See code" displays a QR code.
      "You can configure a mobile device for ???{displayName}??? right now, or you can do it later from the App Users table by clicking ???See code.???",
      {
        "full": "You may wish to visit this Project???s {formAccessSettings} to give this user access to Forms.",
        "formAccessSettings": "Form Access settings"
      }
    ],
    "action": {
      // This is the text of a button that is used to create another App User.
      "createAnother": "Create another"
    }
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "title": "Vytvo??it u??ivatele aplikace",
    "introduction": [
      "Tento u??ivatel nebude m??t nejprve p????stup k ????dn??m formul??????m. Po vytvo??en?? u??ivatele budete moci p??i??adit formul????e."
    ],
    "success": [
      "U??ivatel aplikace ???{displayName}??? byl vytvo??en.",
      "Mobiln?? za????zen?? pro ???{displayName}??? m????ete nakonfigurovat pr??v?? te??, nebo to m????ete ud??lat pozd??ji z tabulky U??ivatel?? aplikace kliknut??m na ???Zobrazit k??d???.",
      {
        "full": "Mo??n?? budete cht??t nav??t??vit toto {formAccessSettings} projektu a d??t tomuto u??ivateli p????stup k formul??????m.",
        "formAccessSettings": "Nastaven?? p????stupu k formul????i"
      }
    ],
    "action": {
      "createAnother": "Vytvo??it jin??ho"
    }
  },
  "de": {
    "title": "App-Benutzer erstellen",
    "introduction": [
      "Der Benutzer hat zun??chst keinen Zugriff auf Formulare. Sie k??nnen seinen Zugriff auf Formulare sp??ter erstellen."
    ],
    "success": [
      "Der App-Benutzer \"{displayName}\" ist erstellt worden.",
      "Sie k??nnen jetzt ein mobiles Ger??t f??r \"{displayName}\" konfigurieren oder Sie k??nnen es sp??ter in der App-Benutzer-Tabelle durch Klick auf \"Code anzeigen.\"",
      {
        "full": "Sie k??nnen dem Benutzer jetzt auf der Seite {formAccessSettings} Zugriff zu den Formularen geben.",
        "formAccessSettings": "Einstellungen Formular-Zugriff"
      }
    ],
    "action": {
      "createAnother": "Noch einen erstellen"
    }
  },
  "es": {
    "title": "Crear usuario m??vil",
    "introduction": [
      "Este usuario no tendr?? acceso a ning??n formulario al principio. Podr?? asignar formularios despu??s de crear el usuario."
    ],
    "success": [
      "Se ha creado el usuario m??vil {displayName}",
      "Puede configurar un dispositivo m??vil para \"{displayName}\" ahora mismo, o puede hacerlo m??s tarde desde la tabla de Usuarios m??viles de la aplicaci??n haciendo clic en \"Ver c??digo\".",
      {
        "full": "Es posible que desee visitar la {formAccessSettings} de este proyecto para dar acceso a este usuario a los formularios.",
        "formAccessSettings": "Configuraci??n de acceso a Formulario"
      }
    ],
    "action": {
      "createAnother": "Crear otro"
    }
  },
  "fr": {
    "title": "Cr??er un utilisateur mobile",
    "introduction": [
      "Cet utilisateur n'aura acc??s ?? aucun formulaire dans un premier temps. Vous pourrez lui en assigner quand il aura ??t?? cr????."
    ],
    "success": [
      "L'utilisateur mobile ???{displayName}??? a ??t?? cr????.",
      "Vous pouvez configurer un appareil mobile pour ???{displayName}??? d??s maintenant, ou vous pouvez le faire plus tard depuis le tableau des \"Utilisateurs mobiles\" en cliquant sur \"Voir le code\".",
      {
        "full": "Vous pourriez vouloir visiter les {formAccessSettings} de ce projet pour donner acc??s aux formulaires ?? cet utilisateur.",
        "formAccessSettings": "Param??tres d'acc??s aux formulaires"
      }
    ],
    "action": {
      "createAnother": "En cr??er un autre"
    }
  },
  "id": {
    "title": "Buat Pengguna Aplikasi",
    "introduction": [
      "Pengguna ini belum bisa mengakses formulir apapun. Anda bisa membagikan formulir setelah akun Pengguna dibuat."
    ],
    "success": [
      "Pengguna Aplikasi \"{displayName}\" telah dibuat.",
      "Kamu bisa mengkonfigurasi peranti seluler untuk ???{displayName}??? saat ini juga, atau bisa lain waktu melalui tabel Pengguna Apl dengan mengklik ???Lihat kode.???",
      {
        "full": "Anda mungkin ingin mengunjungi {formAccessSettings} proyek ini untuk memberikan Pengguna ini akses formulir.",
        "formAccessSettings": "pengaturan Akses Formulir"
      }
    ],
    "action": {
      "createAnother": "Buat baru"
    }
  },
  "it": {
    "title": "Crea un Utente App",
    "introduction": [
      "Inizialmente questo utente non avr?? accesso ad alcun formulario. Potrai assegnare i formulari solo dopo la creazione dell'utente."
    ],
    "success": [
      "L'Utente App \"{displayName}\" ?? stato creato.",
      "Puoi configurare un dispositivo mobile per \"{displayName}\" in questo momento, oppure puoi farlo in un secondo momento dalla tabella Utenti App facendo clic su \"Vedi codice\".",
      {
        "full": "Potresti voler visitare questo progetto {formAccessSettings} per dare a questo utente l'accesso ai formulari.",
        "formAccessSettings": "Configurazioni di accesso al form"
      }
    ],
    "action": {
      "createAnother": "Creare un altro"
    }
  },
  "ja": {
    "title": "??????????????????????????????",
    "introduction": [
      "????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????"
    ],
    "success": [
      "?????????????????????\"{displayName}\"??????????????????????????????",
      "???{displayName}????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????QR?????????????????????????????????????????????????????????????????????????????????",
      {
        "full": "???????????????????????????\"{formAccessSettings}\"??????????????????????????????????????????????????????????????????????????????????????????",
        "formAccessSettings": "?????????????????????????????????"
      }
    ],
    "action": {
      "createAnother": "?????????????????????"
    }
  },
  "sw": {
    "title": "Unda Mtumiaji wa Programu",
    "introduction": [
      "Mtumiaji huyu hatakuwa na ufikiaji wa Fomu zozote mwanzoni. Utaweza kukabidhi Fomu baada ya mtumiaji kuunda."
    ],
    "success": [
      "Mtumiaji wa Programu \"{displayName}\" ameundwa.",
      "Unaweza kusanidi kifaa cha mkononi kwa ajili ya \"{displayName}\" sasa hivi, au unaweza kuifanya baadaye kwenye jedwali la Watumiaji wa Programu kwa kubofya \"Angalia nambari ya kuthibitisha.\"",
      {
        "full": "Unaweza kutaka kutembelea {formAccessSettings} za Mradi huu ili kumpa mtumiaji huyu idhini ya kufikia Fomu.",
        "formAccessSettings": "Mipangilio ya Ufikiaji wa Fomu"
      }
    ],
    "action": {
      "createAnother": "Unda nyingine"
    }
  }
}
</i18n>
