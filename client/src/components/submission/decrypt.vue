<!--
Copyright 2019 ODK Central Developers
See the NOTICE file at the top-level directory of this distribution and at
https://github.com/getodk/central-frontend/blob/master/NOTICE.

This file is part of ODK Central. It is subject to the license terms in
the LICENSE file found in the top-level directory of this distribution and at
https://www.apache.org/licenses/LICENSE-2.0. No part of ODK Central,
including this file, may be copied, modified, propagated, or distributed
except according to the terms contained in the LICENSE file.
-->
<template>
  <modal :state="state" hideable :large="managedKey != null" backdrop
    @hide="$emit('hide')"
    @shown="$refs.form.querySelector('input:not([disabled])').focus()">
    <template #title>{{ $t('title') }}</template>
    <template #body>
      <div id="submission-download-container">
        <form ref="form" @submit.prevent>
          <p id="submission-download-options-title">
            {{ $t('exportOptions') }}
          </p>
          <div class="checkbox"
            :class="{ disabled: splitSelectMultiplesTitle != null }">
            <label :title="splitSelectMultiplesTitle">
              <input v-model="splitSelectMultiples" type="checkbox"
                :disabled="splitSelectMultiplesTitle != null">
              {{ $t('field.splitSelectMultiples') }}
            </label>
          </div>
          <div class="checkbox">
            <label>
              <input v-model="removeGroupNames" type="checkbox">
              {{ $t('field.removeGroupNames') }}
            </label>
          </div>
          <div class="checkbox"
            :class="{ disabled: deletedFieldsTitle != null }">
            <label :title="deletedFieldsTitle">
              <input v-model="deletedFields" type="checkbox"
                :disabled="deletedFieldsTitle != null"
                aria-describedby="submission-download-deleted-fields-help">
              {{ $t('field.deletedFields') }}
            </label>
            <p id="submission-download-deleted-fields-help" class="help-block">
              {{ $t('deletedFieldsHelp') }}
            </p>
          </div>
          <template v-if="managedKey != null">
            <p id="submission-download-passphrase-help"
              class="modal-introduction">
              {{ $t('introduction[0]') }}
            </p>
            <form-group v-model="passphrase" type="password"
              :placeholder="$t('field.passphrase')" required
              autocomplete="off"/>
            <p v-if="managedKey.hint != null" class="modal-introduction">
              {{ $t('hint', managedKey) }}
            </p>
          </template>
        </form>
        <div id="submission-download-divider"></div>
        <!-- eslint-disable-next-line vuejs-accessibility/click-events-have-key-events -->
        <div id="submission-download-actions" @click="download">
          <div class="submission-download-action">
            <span class="submission-download-action-label">
              <span class="icon-file-o"></span>{{ $t('action.download.mainTable') }}
            </span>
            <div>
              <a :href="href('.csv')" class="btn btn-primary">
                <span class="icon-download"></span>.csv
              </a>
            </div>
          </div>
          <div class="submission-download-action"
            :class="{ disabled: noRepeat }">
            <span class="submission-download-action-label"
              :title="noRepeat ? $t('noRepeat') : null">
              <span class="icon-files-o"></span>{{ $t('action.download.allTables') }}
            </span>
            <div>
              <a :href="href('.csv.zip', { attachments: false })"
                class="btn btn-primary" :class="{ disabled: noRepeat }"
                :title="noRepeat ? $t('noRepeat') : null">
                <span class="icon-download"></span>.zip
              </a>
            </div>
          </div>
          <div class="submission-download-action">
            <span class="submission-download-action-label">
              <span class="icon-image"></span>{{ $t('action.download.withAttachments') }}
            </span>
            <div>
              <a :href="href('.csv.zip')" class="btn btn-primary">
                <span class="icon-download"></span>.zip
              </a>
            </div>
          </div>
        </div>
      </div>
      <div class="modal-actions">
        <button type="button" class="btn btn-primary" @click="$emit('hide')">
          {{ $t('action.close') }}
        </button>
      </div>
      <!-- We specify a Frontend page for src so that any cookies are sent when
      the iframe form is submitted. -->
      <!-- eslint-disable-next-line vuejs-accessibility/iframe-has-title -->
      <iframe v-show="false" ref="iframe" src="/blank.html"></iframe>
    </template>
  </modal>
</template>

<script>
import FormGroup from '../form-group.vue';
import Modal from '../modal.vue';

import useCallWait from '../../composables/call-wait';
import { apiPaths, isProblem } from '../../util/request';
import { useRequestData } from '../../request-data';

export default {
  // Ideally, this component would be named SubmissionDownload, but that would
  // cause messages to be moved in Transifex.
  name: 'SubmissionDecrypt',
  components: { FormGroup, Modal },
  inject: ['alert', 'logger'],
  props: {
    state: Boolean,
    formVersion: Object,
    odataFilter: String
  },
  emits: ['hide'],
  setup() {
    const { session, keys, fields } = useRequestData();
    const { callWait, cancelCall } = useCallWait();
    return { session, keys, fields, callWait, cancelCall };
  },
  data() {
    return {
      splitSelectMultiples: false,
      removeGroupNames: false,
      deletedFields: false,
      passphrase: ''
    };
  },
  computed: {
    managedKey() {
      return this.keys.dataExists ? this.keys.find(key => key.managed) : null;
    },
    splitSelectMultiplesTitle() {
      if (this.fields.dataExists &&
        !this.fields.some(({ selectMultiple }) => selectMultiple === true))
        return this.$t('noSelectMultiple');
      return this.formVersion.dataExists && this.formVersion.keyId != null
        ? this.$t('encryptedForm')
        : null;
    },
    deletedFieldsTitle() {
      return this.formVersion.dataExists && this.formVersion.publishedAt == null
        ? this.$t('deletedFieldsDisabledForDraft')
        : null;
    },
    noRepeat() {
      return this.fields.dataExists &&
        !this.fields.some(({ type }) => type === 'repeat');
    }
  },
  watch: {
    state(state) {
      if (!state) {
        // Reset the passphrase, but don't reset the other form fields.
        this.passphrase = '';
        this.cancelCall('checkForProblem');
      }
    }
  },
  methods: {
    href(extension, query = undefined) {
      if (!this.formVersion.dataExists) return '#';
      const fullQuery = {
        ...query,
        $filter: this.odataFilter,
        splitSelectMultiples: this.splitSelectMultiples,
        groupPaths: !this.removeGroupNames,
        deletedFields: this.deletedFields
      };
      return apiPaths.submissions(
        this.formVersion.projectId,
        this.formVersion.xmlFormId,
        this.formVersion.publishedAt == null,
        extension,
        fullQuery
      );
    },
    /*
    submitIframeForm() empties the iframe body, appends a form to it, then
    submits the form. We place a form in an iframe for a few reasons:

      - We want to have the browser handle everything about the download, which
        means that we cannot use an AJAX request.
      - Our two options are an <a> element and a <form> element. We use a form
        so that we can send a POST request. If we wish to securely pass the
        passphrase to Backend, then assuming that wire security is not an issue,
        we still need to ensure that the passphrase is not stored in the user's
        browser history. A POST request allows us to accomplish that.
      - However, submitting a form outside an iframe would navigate away from
        Frontend, at least if a Problem is returned. Thus, we place the form
        inside an iframe. The iframe may change pages, but that won't affect the
        rest of Frontend.

    Note that because the iframe may change pages after the form is submitted
    (if a Problem is returned), we recreate the form each time we submit it.
    */
    submitIframeForm(action) {
      const doc = this.$refs.iframe.contentWindow.document;
      doc.body.innerHTML = '';
      const form = doc.createElement('form');
      form.setAttribute('method', 'post');
      form.setAttribute('action', action);
      doc.body.appendChild(form);

      const passphraseInput = doc.createElement('input');
      // This might not be necessary (not sure).
      passphraseInput.setAttribute('type', 'password');
      passphraseInput.setAttribute('name', this.managedKey.id.toString());
      passphraseInput.setAttribute('autocomplete', 'off');
      form.appendChild(passphraseInput);

      const csrf = doc.createElement('input');
      csrf.setAttribute('type', 'password');
      csrf.setAttribute('name', '__csrf');
      csrf.setAttribute('autocomplete', 'off');
      form.appendChild(csrf);

      passphraseInput.value = this.passphrase;
      csrf.value = this.session.csrf;
      form.submit();
      // Ensure that the inputs' values are no longer in the DOM.
      form.reset();
    },
    checkForProblem() {
      const doc = this.$refs.iframe.contentWindow.document;
      // If Backend returns a Problem, the iframe changes pages. However, if the
      // form submission is successful, it seems that the iframe does not change
      // pages, and the form remains on the page.
      if (doc.querySelector('form') != null || doc.body == null) return false;
      let problem;
      try {
        // Note that the Problem may be wrapped in another element, for example,
        // a <pre> element.
        problem = JSON.parse(doc.body.textContent);
      } catch (e) {
        this.logger.log(doc.body.textContent);
        this.alert.danger(this.$t('alert.parseError'));
      }
      if (isProblem(problem)) this.alert.danger(problem.message);
      return true;
    },
    decrypt(action) {
      // Return immediately if the iframe is still loading. It would probably be
      // better to wait for the iframe to load, then continue the process then,
      // but there would be edge cases to consider in implementing that. (For
      // example, what if the user submits the form, but then closes the modal
      // before the iframe finishes loading?)
      if (this.$refs.iframe.contentWindow.document.readyState === 'loading') {
        this.alert.info('alert.unavailable');
        return;
      }

      this.submitIframeForm(action);

      this.cancelCall('checkForProblem');
      this.callWait('checkForProblem', this.checkForProblem, (tries) =>
        (tries < 300 ? 1000 : null));
    },
    download(event) {
      const a = event.target.closest('a');
      if (a == null) return;
      const willDownload = !a.classList.contains('disabled') &&
        this.$refs.form.reportValidity();
      if (this.managedKey == null) {
        if (!willDownload) event.preventDefault();
      } else {
        event.preventDefault();
        if (willDownload) this.decrypt(a.getAttribute('href'));
      }
      if (willDownload) this.alert.info(this.$t('alert.submit'));
    }
  }
};
</script>

<style lang="scss">
@import '../../assets/scss/variables';

#submission-download-container {
  display: flex;
  justify-content: space-between;
}
#submission-download-options-title { font-weight: bold; }
#submission-download-container form, #submission-download-actions {
  margin-bottom: 1px;
  margin-top: 5px;
}
.modal-dialog:not(.modal-lg) #submission-download-container form {
  max-width: 305px;
}
#submission-download-passphrase-help { margin-top: 21px; }
#submission-download-container form :last-child { margin-bottom: 0; }
#submission-download-divider {
  border-left: 1px solid $color-subpanel-border-strong;
  margin: -1px 18px -5px 18px;
}
$label-icon-max-width: 15px;
$actions-padding-left: $label-icon-max-width + $margin-right-icon;
#submission-download-actions {
  margin-right: 12px;
  padding-left: $actions-padding-left;
}
.submission-download-action-label {
  // Instead of wrapping the element in a <div> and setting the margin on that,
  // we set `display` to inline-block: that way, the cursor will be shown as
  // not-allowed only over the text.
  display: inline-block;
  font-weight: bold;
  margin-bottom: 3px;
  text-indent: -$actions-padding-left;

  [class^="icon-"] {
    display: inline-block;
    margin-right: $margin-right-icon;
    min-width: $label-icon-max-width;
    text-indent: 0;
  }

  .submission-download-action.disabled & {
    color: $color-input-inactive;
    cursor: not-allowed;
  }
}
.submission-download-action + .submission-download-action { margin-top: 12px; }
</style>

<i18n lang="json5">
{
  "en": {
    // This is the title at the top of a pop-up.
    "title": "Download Submissions",
    // This text is shown above options for downloading Submissions.
    "exportOptions": "Export options",
    "field": {
      "splitSelectMultiples": "Split ???select multiple??? choices into columns",
      "removeGroupNames": "Remove group names",
      "deletedFields": "Include fields not in the published Form"
    },
    "deletedFieldsHelp": "Use this option if you need to see fields referenced in previous Form versions.",
    "noSelectMultiple": "This Form does not have any select multiple fields.",
    "encryptedForm": "Encrypted Forms cannot be processed in this way.",
    "deletedFieldsDisabledForDraft": "Draft Forms cannot be processed in this way.",
    "introduction": [
      "In order to download this data, you will need to provide your passphrase. Your passphrase will be used only to decrypt your data for download, after which the server will forget it again."
    ],
    // This text is shown if there is a passphrase hint. {hint} is the
    // passphrase hint.
    "hint": "Hint: {hint}",
    // "Repeats" refers to repeat groups.
    "noRepeat": "This Form does not have repeats.",
    "action": {
      "download": {
        // This is the text of a button. "Repeats" refers to repeat groups.
        "mainTable": "Main data table (no repeats)",
        "allTables": "All data tables",
        // "Attachments" refers to Submission Attachments.
        "withAttachments": "All data and Attachments"
      }
    },
    "alert": {
      "unavailable": "The data download is not yet available. Please try again in a moment.",
      "submit": "Your data download should begin soon. Once it begins, you can close this box. If you have been waiting and it has not started, please try again.",
      "parseError": "Something went wrong while requesting your data."
    }
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "title": "St??hnout p????sp??vky",
    "exportOptions": "Mo??nosti exportu",
    "field": {
      "splitSelectMultiples": "Rozd??len?? mo??nost?? \"vybrat v??ce\" do sloupc??",
      "removeGroupNames": "Odstran??n?? n??zv?? skupin",
      "deletedFields": "Zahrnout pole, kter?? nejsou ve zve??ejn??n??m formul????i"
    },
    "deletedFieldsHelp": "Tuto mo??nost pou??ijte, pokud pot??ebujete zobrazit pole odkazovan?? v p??edchoz??ch verz??ch formul????e.",
    "noSelectMultiple": "Tento formul???? nem?? ????dn?? pole pro v??b??r v??ce polo??ek.",
    "encryptedForm": "??ifrovan?? formul????e nelze t??mto zp??sobem zpracov??vat.",
    "deletedFieldsDisabledForDraft": "N??vrhy formul?????? nelze t??mto zp??sobem zpracov??vat.",
    "introduction": [
      "Abyste mohli tato data st??hnout, budete muset zadat va??e p????stupov?? heslo. Va??e p????stupov?? heslo bude pou??ito pouze k de??ifrov??n?? va??ich dat ke sta??en??, a pot?? ho server znovu zapomene."
    ],
    "hint": "N??pov??da: {hint}",
    "noRepeat": "Tento formul???? nem?? opakov??n??.",
    "action": {
      "download": {
        "mainTable": "Hlavn?? datov?? tabulka (bez opakov??n??)",
        "allTables": "V??echny datov?? tabulky",
        "withAttachments": "V??echna data a p????lohy"
      }
    },
    "alert": {
      "unavailable": "Data ke sta??en?? zat??m nejsou k dispozici. Zkuste to pros??m za chv??li znovu.",
      "submit": "Stahov??n?? dat by m??lo za????t brzy. Jakmile to za??ne, m????ete toto pole zav????t. Pokud jste ??ekali a neza??alo to, zkuste to znovu.",
      "parseError": "P??i vy????d??n?? va??ich dat se n??co pokazilo."
    }
  },
  "de": {
    "title": "??bermittlungen downloaden",
    "exportOptions": "Exportoptionen",
    "field": {
      "splitSelectMultiples": "Unterteilen Sie ???mehrere ausw??hlen???-Optionen in Spalten",
      "removeGroupNames": "Gruppennamen entfernen",
      "deletedFields": "Felder einschlie??en, die nicht in der ver??ffentlichten Version des Formulars enthalten sind"
    },
    "deletedFieldsHelp": "Verwenden Sie diese Option, um Felder aus fr??heren Versionen des Formulars anzuzeigen.",
    "noSelectMultiple": "Dieses Formular enth??lt keine Mehrfachauswahlfelder.",
    "encryptedForm": "Verschl??sselte Formulare k??nnen auf diese Weise nicht verarbeitet werden.",
    "deletedFieldsDisabledForDraft": "Formularentw??rfe k??nnen auf diese Weise nicht bearbeitet werden.",
    "introduction": [
      "Um die Daten herunterzuladen wird Ihre Passphrase ben??tigt. Diese wird nur zum Entschl??sseln der Daten f??r den Download benutzt und nicht gespeichert."
    ],
    "hint": "Hinweis: {hint}",
    "noRepeat": "Diese Form hat keine repeats.",
    "action": {
      "download": {
        "mainTable": "Hauptdatentabelle (keine repeats)",
        "allTables": "Alle Datentabellen"
      }
    },
    "alert": {
      "unavailable": "Der Datendownload ist noch nicht verf??gbar. Bitte versuchen Sie es gleich noch einmal.",
      "submit": "Ihr Daten-Download sollte bald beginnen. Sobald er beginnt, k??nnen Sie diese Box schlie??en. Wenn Sie gewartet haben und er hat nicht begonnen, versuchen Sie es erneut.",
      "parseError": "Beim Anfordern Ihrer Daten ist etwas nicht geklappt."
    }
  },
  "es": {
    "title": "Descargar datos enviados",
    "exportOptions": "Opciones de exportaci??n",
    "field": {
      "splitSelectMultiples": "Divida las opciones de \"selecci??n m??ltiple\" en columnas",
      "removeGroupNames": "Eliminar nombres de grupos",
      "deletedFields": "Incluir campos que no est??n en el Formulario publicado."
    },
    "noSelectMultiple": "Este formulario no tiene ning??n campo de selecci??n m??ltiple.",
    "encryptedForm": "Los formularios cifrados no se pueden procesar de esta manera.",
    "deletedFieldsDisabledForDraft": "Los borradores no se pueden procesar de esta manera.",
    "introduction": [
      "Para descargar la informaci??n, usted necesitar?? ingresar su contrase??a. Su contrase??a se usar?? para desencriptar la informaci??n descargada, al finalizar el proceso el servidor la olvidar??."
    ],
    "hint": "Pista: {hint}",
    "noRepeat": "Este formulario no tiene campos repetidos.",
    "action": {
      "download": {
        "mainTable": "Tabla de datos principal (sin repeticiones)",
        "allTables": "Todas las tablas de datos"
      }
    },
    "alert": {
      "unavailable": "La descarga de datos a??n no est?? disponible. Vuelva a intentarlo en un momento.",
      "submit": "Su descarga de los datos comenzar?? pronto. Una vez que inicie, puede cerrar este cuadro. Si ha estado esperando y no ha comenzado, por favor int??ntelo nuevamente.",
      "parseError": "Algo sali?? mal al solicitar tus datos."
    }
  },
  "fr": {
    "title": "T??l??charger les soumissions",
    "exportOptions": "Options d'exportation",
    "field": {
      "splitSelectMultiples": "S??parer le contenu des questions ?? choix multiples en plusieurs colonnes.",
      "removeGroupNames": "Supprimer les noms de groupe",
      "deletedFields": "Inclut les champs qui ne sont pas dans le formulaire publi??"
    },
    "deletedFieldsHelp": "Utilisez cette option si vous avez besoin de champs r??f??renc??s dans les versions pr??c??dentes du formulaire",
    "noSelectMultiple": "Ce formulaire ne contient aucun champ ?? choix multiples (de type \"select_multiple\").",
    "encryptedForm": "Les formulaires chiffr??s ne peuvent ??tre trait??s de cette mani??re.",
    "deletedFieldsDisabledForDraft": "Les ??bauches de formulaires ne peuvent ??tre utilis??es comme cela.",
    "introduction": [
      "Pour t??l??charger ces donn??es, vous devrez fournir votre phrase secr??te. Votre phrase secr??te sera utilis??e uniquement pour d??chiffrer vos donn??es pour le t??l??chargement, apr??s quoi le serveur l'oubliera ?? nouveau"
    ],
    "hint": "Indice : {hint}",
    "noRepeat": "Ce formulaire ne contient pas de structures \"repeat\"",
    "action": {
      "download": {
        "mainTable": "Principale table de donn??es (sans \"repeats\")",
        "allTables": "Toutes les tables de donn??es",
        "withAttachments": "Toutes donn??es et fichiers joints"
      }
    },
    "alert": {
      "unavailable": "Le t??l??chargement des donn??es n'est pas encore disponible. Merci de r??essayer dans un moment.",
      "submit": "Le t??l??chargement de vos donn??es devrait commencer bient??t. Une fois qu'il aura commenc??, vous pourrez fermer cette bo??te. Si vous avez attendu et que le t??l??chargement n'a pas commenc??, veuillez r??essayer.",
      "parseError": "Quelque chose s'est mal pass?? pendant la requ??te de vos donn??es"
    }
  },
  "id": {
    "introduction": [
      "Untuk mengunduh data ini, Anda harus menyediakan frasa sandi. Frasa sandi hanya akan digunakan untuk mendekripsi data anda untuk diunduh, dan akan terhapus dari server setelahnya."
    ],
    "hint": "Petunjuk: {hint}",
    "alert": {
      "submit": "Unduhan Anda akan segera dimulai. Setelah unduhan dimulai, Anda dapat menutup kotak ini. Apabila Anda sudah menunggu dan unduhan belum dimulai, silakan coba lagi."
    }
  },
  "it": {
    "title": "Scarica invii",
    "exportOptions": "Opzioni di esportazione",
    "field": {
      "splitSelectMultiples": "Dividi le scelte \"select multiple\" in colonne",
      "removeGroupNames": "Rimuovi i nomi dei gruppi",
      "deletedFields": "Includere i campi non nel formulario pubblicato"
    },
    "deletedFieldsHelp": "Utilizzare questa opzione se ?? necessario visualizzare i campi a cui si fa riferimento nelle versioni precedenti del formulario.",
    "noSelectMultiple": "Questo formulario non ha campi a scelta multipla selezionati.",
    "encryptedForm": "I moduli crittografati non possono essere elaborati in questo modo.",
    "deletedFieldsDisabledForDraft": "Le bozze non possono essere elaborate in questo modo.",
    "introduction": [
      "Per scaricare questi dati, dovrai fornire la tua passphrase. La tua passphrase verr?? utilizzata solo per decifrare i tuoi dati per il download, dopodich?? il server la canceller??."
    ],
    "hint": "Suggerimento: {hint}",
    "noRepeat": "Questo formulario non ha repeats.",
    "action": {
      "download": {
        "mainTable": "Tabella dati principale (no repeats)",
        "allTables": "Tutte le tabelle di dati"
      }
    },
    "alert": {
      "unavailable": "Il download dei dati non ?? ancora disponibile. Si prega di riprovare in un altro momento.",
      "submit": "Il download dei dati dovrebbe iniziare a breve. Una volta iniziato, puoi chiudere questa finestra. Se stai ancora aspettando e il download non ?? iniziato, riprova.",
      "parseError": "Qualcosa ?? andato storto durante la richiesta dei tuoi dati"
    }
  },
  "ja": {
    "title": "????????????????????????????????????????????????",
    "exportOptions": "????????????????????????",
    "field": {
      "splitSelectMultiples": "?????????????????????????????????????????????",
      "removeGroupNames": "??????????????????????????????"
    },
    "noSelectMultiple": "???????????????????????????????????????????????????????????????",
    "encryptedForm": "????????????????????????????????????????????????????????????",
    "deletedFieldsDisabledForDraft": "??????????????????????????????????????????????????????",
    "introduction": [
      "??????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????"
    ],
    "hint": "????????????{hint}",
    "noRepeat": "????????????????????????????????????????????????????????????",
    "action": {
      "download": {
        "mainTable": "????????????????????????????????????????????????",
        "allTables": "??????????????????"
      }
    },
    "alert": {
      "unavailable": "?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????",
      "submit": "??????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????"
    }
  },
  "sw": {
    "title": "Pakua Mawasilisho",
    "exportOptions": "Chaguzi za export",
    "field": {
      "splitSelectMultiples": "Gawanya chaguo za \"chagua nyingi\" kwenye safu wima",
      "removeGroupNames": "Ondoa majina ya kikundi"
    },
    "noSelectMultiple": "Fomu hii haina sehemu nyingi zilizochaguliwa",
    "encryptedForm": "Fomu Zilizosimbwa kwa njia fiche haziwezi kuchakatwa kwa njia hii",
    "deletedFieldsDisabledForDraft": "Rasimu za Fomu haziwezi kuchakatwa kwa njia hii.",
    "introduction": [
      "Ili kupakua data hii, utahitaji kutoa neno lako la siri. Kauli yako ya siri itatumika tu kusimbua data yako kwa ajili ya kupakua, na kisha seva itaisahau tena."
    ],
    "hint": "Kidokezo: {hint}",
    "noRepeat": "Fomu hii haina marudio",
    "action": {
      "download": {
        "mainTable": "Jedwali kuu la data (hakuna marudio)",
        "allTables": "Jedwali zote za data"
      }
    },
    "alert": {
      "unavailable": "Upakuaji wa data bado haupatikani. Tafadhali jaribu tena baada ya muda mfupi",
      "submit": "Upakuaji wa data yako unapaswa kuanza hivi karibuni. Mara tu inapoanza, unaweza kufunga kisanduku hiki. Ikiwa umekuwa ukingoja na haijaanza, tafadhali jaribu tena."
    }
  }
}
</i18n>
